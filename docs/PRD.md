# PRD: OMS Sistema de Webhooks de Notificação de Pedidos

| Campo | Valor |
| --- | --- |
| Versão | 1.0 |
| Data | 2026-10-22 (redação; reunião técnica de quinta-feira) |
| Responsável | Marcos (Product Manager). **Proposta (a validar):** quem trouxe o pedido dos clientes (`[09:00] Marcos`) e definiu os requisitos de produto (`[09:31] Marcos`) |
| Documentos relacionados | [RFC](RFC.md), [FDD](FDD.md), [Decisões relacionadas (ADR-001 a ADR-007)](RFC.md#decisões-relacionadas) |

Convenções: **Proposta (a validar)** marca o que a reunião não definiu, com a fala que a motiva e a questão ligada; **Hipótese** marca leitura a confirmar, com o modo de confirmar. Vocabulário: **webhook** (cadastro de uma URL do cliente), **notificação** (aviso de mudança do **status do pedido**: PENDING, PAID, PROCESSING, SHIPPED, DELIVERED ou CANCELLED, em `prisma/schema.prisma`), **envio** (cada tentativa de entrega), **entregas esgotadas** (notificações que esgotaram o reenvio automático) e **administrador** (usuário com papel ADMIN). A implementação está no [RFC](RFC.md) e no [FDD](FDD.md).

## Resumo e contexto

A feature avisa o cliente, por webhook, a cada mudança de status de um pedido dele, só da plataforma para o cliente (`[09:02] Marcos`). Ela responde ao pedido formal de 3 clientes B2B, Atlas Comercial, MaxDistribuição e Nova Cargo, que querem ser notificados quando o status dos pedidos deles muda, porque hoje consultam a lista de pedidos de tempos em tempos e a integração ficou lenta e cara (`[09:00] Marcos`).

**Onde a feature será implantada**: no sistema de gestão de pedidos existente, como um módulo igual aos outros da mesma API (`[09:30] Larissa`), no banco MySQL que o projeto já usa, sem sistema novo nem infraestrutura nova (`[09:07] Diego`). A arquitetura proposta está no [RFC](RFC.md).

## Problema e motivação

- **PRD-PROB-01**: os clientes descobrem mudanças de status consultando `GET /orders` de tempos em tempos; o impacto é uma integração lenta e cara para eles (`[09:00] Marcos`).
- **PRD-PROB-02**: os 3 clientes B2B fizeram um pedido formal da notificação; o impacto é uma demanda formal e nomeada de clientes existentes (`[09:00] Marcos`).
- **PRD-PROB-03**: a Atlas sugeriu migrar para um concorrente se a notificação não sair até o fim do trimestre; o impacto é o risco de perder um dos 3 clientes (`[09:00] Marcos`).
- **PRD-PROB-04**: a data pedida pela Atlas é o fim de novembro; o impacto é um prazo externo para a primeira entrega (`[09:45] Marcos`).

O que os clientes chamam de "tempo real" é uma notificação abaixo de 10 s; o que importa é não ficar pendurado nem exigir atualização manual (`[09:02] Marcos`).

## Público-alvo e cenários de uso

### Público-alvo

- **PRD-PUB-01**: os 3 clientes B2B e os sistemas deles que recebem as notificações (`[09:00] Marcos`).
- **PRD-PUB-02**: os usuários que cadastram e consultam webhooks em nome do cliente, direto pela API, autenticados no sistema (`[09:32] Marcos`). **Hipótese:** são usuários comuns do sistema, porque não há vínculo entre usuário e cliente (`prisma/schema.prisma`); confirmar com Marcos e Bruno (PRD-DEP-05).
- **PRD-PUB-03**: os administradores, que reprocessam as entregas esgotadas (`[09:36] Larissa`).
- **PRD-PUB-04**: os desenvolvedores dos clientes, que leem no portal do desenvolvedor como integrar (`[09:40] Marcos`).

### Cenários de uso

- **PRD-PUB-05**: o sistema do cliente recebe a notificação de mudança de status e, quando precisa de detalhes como os itens, consulta o pedido na API (`[09:43] Diego`).
- **PRD-PUB-06**: o cliente escolhe só os status que interessam, por exemplo SHIPPED e DELIVERED (`[09:33] Marcos`).
- **PRD-PUB-07**: o cliente com vários webhooks sabe qual cadastro recebeu cada notificação (`[09:44] Sofia`).
- **PRD-PUB-08**: o cliente confere no histórico o que foi enviado, o que o sistema dele respondeu e em quanto tempo (`[09:34] Marcos`).
- **PRD-PUB-09**: o cliente fica fora do ar por horas, como na indisponibilidade de 2 h que já ocorreu (`[09:16] Diego`), e recebe as notificações depois, pelo reenvio automático dentro da janela aceita pelo produto (`[09:17] Marcos`).
- **PRD-PUB-10**: depois que o cliente se recupera, um administrador reprocessa uma entrega esgotada (`[09:18] Diego`).
- **PRD-PUB-11**: depois de um vazamento de secret, como o que já ocorreu em log de um cliente (`[09:22] Diego`), o cliente pede uma secret nova e tem 24 h para migrar os sistemas dele (`[09:21] Sofia`); a forma da assinatura nesse período está no PRD-FR-12.
- **PRD-PUB-12**: o cliente recebe a mesma notificação duas vezes e descarta a repetida pelo identificador da notificação (`[09:25] Diego`).

## Objetivos e métricas de sucesso

| ID | Objetivo | Métrica | Meta | Origem |
| --- | --- | --- | --- | --- |
| PRD-OBJ-01 | Notificar a mudança de status do pedido no que os clientes chamam de tempo real | Tempo entre a mudança de status e a chegada da notificação | Abaixo de 10 s. A meta só é atendida quando o primeiro envio dá certo, com atraso de até 2 s causado pela leitura periódica ([ADR-002](adrs/ADR-002-worker-separado-com-polling.md)); se vale também para notificações que precisam de reenvio é pergunta de produto para Marcos ([RFC-QA-07](RFC.md#questões-em-aberto)) | `[09:02] Marcos`, `[09:10] Larissa` |
| PRD-OBJ-02 | Deixar a feature disponível para os 3 clientes que pediram | Prazo da primeira entrega, revisão de segurança incluída | 3 sprints (`[09:47] Larissa`). A Atlas pediu o fim de novembro (`[09:45] Marcos`), e Marcos confirma o prazo com ela (`[09:47] Marcos`). A reunião não verificou se as 3 sprints cabem até essa data (lacuna, sem questão aberta no RFC) | `[09:47] Larissa`, `[09:45] Marcos`, `[09:47] Marcos` |
| PRD-OBJ-03 | Não perder a notificação quando o cliente fica fora do ar | Janela de reenvio automático antes de exigir reprocessamento manual | Quase 15 h, aceita pelo produto (`[09:17] Diego`, `[09:17] Marcos`). **Hipótese:** "5 tentativas" são o envio inicial mais 5 reenvios, a leitura que usa os cinco intervalos e dá cerca de 14,6 h; confirmação na [RFC-QA-08](RFC.md#questões-em-aberto), outra contagem na questão 1 do [ADR-003](adrs/ADR-003-retry-com-backoff-e-dlq.md) | `[09:17] Diego`, `[09:17] Marcos`, `[09:17] Larissa` |
| PRD-OBJ-04 | Dar ao cliente visibilidade das entregas | Envios mostrados no histórico de cada webhook | Os últimos 100 envios por webhook | `[09:34] Marcos` |
| PRD-OBJ-05 | Permitir ao cliente confirmar que a notificação veio da plataforma e não foi alterada | Notificações com assinatura verificável pelo cliente | Toda notificação enviada | `[09:19] Sofia` |

## Escopo

### Incluso

- **PRD-ESC-01**: notificação de mudança de status do pedido (`[09:00] Marcos`), só de saída, da plataforma para o cliente (`[09:02] Marcos`).
- **PRD-ESC-02**: cadastro de webhook (`[09:31] Marcos`), listagem dos webhooks de um cliente, edição e remoção (`[09:33] Bruno`).
- **PRD-ESC-03**: escolha, por webhook, dos status do pedido que ele recebe (`[09:33] Marcos`).
- **PRD-ESC-04**: assinatura verificável com secret por webhook e rotação da secret (`[09:22] Sofia`).
- **PRD-ESC-05**: reenvio automático das notificações que falharam (`[09:17] Larissa`) e guarda das entregas esgotadas (`[09:18] Diego`).
- **PRD-ESC-06**: reprocessamento manual de entrega esgotada por administrador (`[09:36] Larissa`).
- **PRD-ESC-07**: histórico de entregas por webhook (`[09:34] Marcos`).
- **PRD-ESC-08**: documentação para os clientes no portal do desenvolvedor, com o aviso de que a notificação pode chegar repetida (`[09:26] Marcos`) e o modo de integrar (`[09:40] Marcos`).

### Fora de escopo

- **PRD-FORA-01**: webhooks de entrada, com os clientes enviando para a plataforma; eles só querem receber (`[09:02] Marcos`).
- **PRD-FORA-02**: aviso por e-mail ao cliente sobre falhas do webhook, adiado para uma fase futura (`[09:37] Larissa`, `[09:38] Marcos`); não é um caminho de entrega da notificação.
- **PRD-FORA-03**: painel visual para o cliente acompanhar os webhooks, projeto separado do time de frontend (`[09:40] Larissa`).
- **PRD-FORA-04**: limite de taxa de envios por cliente, em observação para decidir depois ([RFC-QA-02](RFC.md#questões-em-aberto), `[09:39] Larissa`).
- **PRD-FORA-05**: arquivamento das notificações já entregues, sem prazo definido (`[09:08] Diego`).
- **PRD-FORA-06**: ordem global entre pedidos, que os clientes nunca pediram (`[09:14] Marcos`), e vários processos de envio em paralelo, deixados para o futuro ([RFC-QA-03](RFC.md#questões-em-aberto), `[09:13] Diego`).
- **PRD-FORA-07**: permissões mais restritas no cadastro de webhooks, para mais adiante ([RFC-QA-04](RFC.md#questões-em-aberto), `[09:37] Sofia`).
- **PRD-FORA-08**: **Proposta (a validar):** a criação do pedido não gera notificação nesta fase, só a mudança de status, único ponto de integração citado (`[09:40] Bruno`); questão 1 do [ADR-001](adrs/ADR-001-outbox-transacional-no-mysql.md).

## Requisitos funcionais

**Proposta (a validar):** a prioridade de cada requisito é estimativa da redação, porque a reunião não priorizou: **alta** para o que o pedido dos clientes e a segurança exigem na primeira entrega (cadastrar, escolher status, notificar, assinar, reenviar e guardar entregas esgotadas) e **média** para o que completa a operação (listar, editar, remover, reprocessar, histórico e rotacionar). Todas as operações exigem usuário autenticado, de qualquer papel nesta fase (`[09:37] Sofia`), salvo o reprocessamento, só para administrador (`[09:36] Larissa`).

#### PRD-FR-01: Cadastrar webhook

O usuário autenticado cadastra, para um cliente, a URL que recebe as notificações e a lista de status do pedido que ela quer receber; a secret é gerada pela plataforma e devolvida na criação (`[09:31] Marcos`).

**Fluxo principal**

1. O usuário informa o cliente (`[09:32] Larissa`), a URL e a lista de status do pedido (`[09:31] Marcos`).
2. O sistema confere que a URL usa https (`[09:23] Sofia`).
3. O sistema gera a secret do webhook e a devolve na resposta da criação (`[09:31] Marcos`).
4. As mudanças de status seguintes que o webhook escolheu passam a gerar notificação para ele (filtro definido em `[09:34] Bruno`; [ADR-007](adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md)).

**Fluxos alternativos e exceções**

- O cliente com vários webhooks recebe uma secret diferente para cada um (`[09:21] Sofia`).
- **Proposta (a validar):** a secret aparece só na resposta da criação e na da rotação, nunca na listagem, na edição ou no histórico; quem perde a secret obtém outra pela rotação (questão 5 do [ADR-004](adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md)) (`[09:31] Marcos`).
- **Hipótese:** o usuário que cadastra é um usuário comum do sistema que representa o cliente (`[09:32] Marcos`); confirmar com Marcos e Bruno (PRD-DEP-05).

**Erros previstos**

- URL sem https é recusada (`[09:23] Sofia`).
- **Proposta (a validar):** lista de status vazia, com repetição ou com status inexistente é recusada ([seção 7 do FDD](FDD.md#7-matriz-de-erros-previstos)) (`[09:33] Marcos`); lacuna, sem questão aberta no RFC.
- **Proposta (a validar):** cliente inexistente é recusado ([seção 7 do FDD](FDD.md#7-matriz-de-erros-previstos)) (`[09:32] Larissa`); lacuna, sem questão aberta no RFC.

**Prioridade:** alta

#### PRD-FR-02: Listar os webhooks de um cliente

O usuário autenticado lista os webhooks cadastrados para um cliente (`[09:33] Bruno`).

**Fluxo principal**

1. O usuário informa o cliente (`[09:33] Bruno`).
2. O sistema devolve os webhooks desse cliente, cada um com a URL e o estado ativo (`[09:21] Bruno`) e os status escolhidos (`[09:33] Marcos`), como no [FDD-CONTRATO-02](FDD.md#61-api-de-configuração-de-webhooks).

**Fluxos alternativos e exceções**

- **Proposta (a validar):** a listagem nunca mostra a secret (questão 5 do [ADR-004](adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md)) (`[09:31] Marcos`).
- **Proposta (a validar):** webhook removido não aparece na listagem; webhook desativado aparece, marcado como inativo (questão 3 do [ADR-007](adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md)) (`[09:33] Bruno`).

**Erros previstos**

- Chamada sem usuário autenticado é recusada (`[09:32] Larissa`).
- **Proposta (a validar):** listagem sem o cliente informado é recusada ([seção 7 do FDD](FDD.md#7-matriz-de-erros-previstos)) (`[09:32] Larissa`); lacuna, sem questão aberta no RFC.

**Prioridade:** média

#### PRD-FR-03: Editar webhook

O usuário autenticado altera a URL, a lista de status e o estado ativo de um webhook (`[09:33] Bruno`, `[09:21] Bruno`).

**Fluxo principal**

1. O usuário escolhe o webhook e informa o que muda: URL, lista de status ou estado ativo (`[09:33] Bruno`).
2. O sistema aplica as mesmas recusas do cadastro (`[09:23] Sofia`).
3. A nova lista de status vale para as mudanças de status feitas depois da edição (filtro definido em `[09:34] Bruno`; [ADR-007](adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md)).

**Fluxos alternativos e exceções**

- **Proposta (a validar):** desativar o webhook, pelo estado ativo do cadastro (`[09:21] Bruno`), suspende as notificações novas para ele, e notificações registradas antes da desativação ainda são entregues (questão 3 do [ADR-007](adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md)).
- **Proposta (a validar):** a resposta da edição não mostra a secret, que só muda pela rotação do PRD-FR-12 (questão 5 do [ADR-004](adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md)) (`[09:31] Marcos`).

**Erros previstos**

- URL sem https é recusada (`[09:23] Sofia`).
- **Proposta (a validar):** lista de status vazia, com repetição ou com status inexistente é recusada ([seção 7 do FDD](FDD.md#7-matriz-de-erros-previstos)) (`[09:33] Marcos`); lacuna, sem questão aberta no RFC.
- Webhook inexistente é recusado (`[09:28] Bruno`). **Proposta (a validar):** webhook removido também (questão 3 do [ADR-007](adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md)).

**Prioridade:** média

#### PRD-FR-04: Remover webhook

O usuário autenticado remove um webhook (`[09:33] Bruno`).

**Fluxo principal**

1. O usuário escolhe o webhook a remover (`[09:33] Bruno`).
2. O sistema confirma a remoção.
3. **Proposta (a validar):** o webhook removido deixa de receber notificações novas e some da listagem (questão 3 do [ADR-007](adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md)) (`[09:33] Bruno`).

**Fluxos alternativos e exceções**

- **Proposta (a validar):** notificações registradas antes da remoção ainda são entregues, inclusive pelo reenvio automático (questão 3 do [ADR-007](adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md)) (`[09:33] Bruno`).

**Erros previstos**

- Webhook inexistente é recusado (`[09:28] Bruno`). **Proposta (a validar):** webhook já removido também (questão 3 do [ADR-007](adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md)).

**Prioridade:** média

#### PRD-FR-05: Escolher os status do pedido que o webhook recebe

Cada webhook escolhe os status do pedido que quer receber (`[09:33] Marcos`) e só recebe notificação desses status (`[09:33] Bruno`).

**Fluxo principal**

1. No cadastro ou na edição, o usuário informa a lista de status do pedido que o webhook quer receber (`[09:33] Marcos`).
2. Uma mudança de status gera notificação só para os webhooks do cliente que escolheram o novo status (`[09:33] Bruno`).
3. Uma mudança para um status que nenhum webhook do cliente escolheu não gera notificação (`[09:34] Bruno`).

**Fluxos alternativos e exceções**

- **Proposta (a validar):** escolher PENDING é aceito, mas nunca gera notificação, porque nenhuma mudança de status leva a PENDING; a criação do pedido não gera notificação nesta fase, só a mudança de status (questão 1 do [ADR-001](adrs/ADR-001-outbox-transacional-no-mysql.md)) (`[09:40] Bruno`).
- **Proposta (a validar):** webhook desativado ou removido não recebe notificações novas, mesmo que tenha escolhido o status (questão 3 do [ADR-007](adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md)) (`[09:21] Bruno`).

**Erros previstos**

- **Proposta (a validar):** lista de status vazia, com repetição ou com status inexistente é recusada no cadastro e na edição ([seção 7 do FDD](FDD.md#7-matriz-de-erros-previstos)) (`[09:33] Marcos`); lacuna, sem questão aberta no RFC.

**Prioridade:** alta

#### PRD-FR-06: Notificar a mudança de status do pedido

A cada mudança de status de um pedido, a plataforma notifica o cliente (`[09:00] Marcos`), nos webhooks dele que escolheram o novo status (`[09:33] Bruno`), com os dados básicos do pedido e da mudança (`[09:43] Diego`).

**Fluxo principal**

1. Um usuário muda o status de um pedido pela operação normal da API (`[09:40] Bruno`).
2. A notificação fica registrada junto com a mudança: se a mudança é desfeita, a notificação some junto (`[09:06] Diego`).
3. No caminho feliz, a leitura periódica atrasa o primeiro envio em até 2 s (`[09:10] Larissa`).
4. A notificação leva o identificador da notificação, o tipo, a data e hora do evento, o pedido, o número do pedido, o status anterior e o novo, o cliente e o valor total, sem os itens (`[09:43] Diego`). **Proposta (a validar):** essa data e hora é o momento da mudança ([RFC-QA-05](RFC.md#questões-em-aberto)).
5. O cliente que quer mais detalhes consulta o pedido na API (`[09:43] Diego`).

**Fluxos alternativos e exceções**

- A notificação reflete o pedido no momento da mudança, mesmo que ele mude depois (`[09:52] Larissa`), e traz o identificador do webhook, para o cliente com vários webhooks saber qual cadastro a recebeu (`[09:44] Sofia`).
- A mesma notificação pode chegar mais de uma vez, com o mesmo identificador em todos os reenvios automáticos (`[09:25] Diego`); o descarte das repetidas está no PRD-RNF-19.
- Os clientes nunca pediram ordem global (`[09:14] Marcos`); a ordem por pedido vale com um único processo de envio, como decidido (`[09:13] Larissa`), e o reenvio e os demais casos do PRD-RNF-21 podem invertê-la.
- **Proposta (a validar):** mudança de status feita direto no banco, como carga inicial ou correção manual, não gera notificação, e a regra fica documentada para quem opera o banco (questão 2 do [ADR-001](adrs/ADR-001-outbox-transacional-no-mysql.md)) (`[09:40] Bruno`).

**Erros previstos**

- Notificação acima de 64 KB é erro (`[09:24] Larissa`). **Proposta (a validar):** o que acontece com ela está no PRD-RNF-22 (questão 2 do [ADR-007](adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md)).
- Envio que falha vai para o reenvio automático do PRD-FR-08 (`[09:42] Diego`).

**Prioridade:** alta

#### PRD-FR-07: Assinar as notificações

Toda notificação leva uma assinatura que permite ao cliente verificar que ela veio da plataforma e não foi alterada no caminho (`[09:19] Sofia`).

**Fluxo principal**

1. A plataforma assina o conteúdo de cada notificação com a secret do webhook de destino (`[09:22] Sofia`).
2. O cliente recebe a notificação e verifica a assinatura com a secret do próprio webhook (`[09:20] Sofia`).
3. A documentação do portal mostra ao cliente como integrar (`[09:40] Marcos`). **Proposta (a validar):** inclusive como verificar a assinatura; lacuna, sem questão aberta no RFC.

**Fluxos alternativos e exceções**

- Cada webhook tem a sua secret, para que o vazamento de uma não comprometa as outras (`[09:21] Sofia`).
- Depois de uma rotação, a secret anterior continua valendo por 24 h (`[09:21] Sofia`). **Proposta (a validar):** nesse período a notificação leva as duas assinaturas, e o cliente aceita qualquer uma das duas ([RFC-QA-09](RFC.md#questões-em-aberto)).

**Erros previstos**

- Assinatura que não confere indica notificação que não veio da plataforma ou foi alterada, e o cliente a descarta (`[09:19] Sofia`).

**Prioridade:** alta

#### PRD-FR-08: Reenviar automaticamente as notificações que falharam

Envio que falha é repetido automaticamente com intervalos crescentes de 1 min, 5 min, 30 min, 2 h e 12 h (`[09:17] Larissa`), por quase 15 h entre a primeira falha e o último envio (`[09:17] Diego`).

**Fluxo principal**

1. Um envio falha, por exemplo porque o sistema do cliente não respondeu em 10 s (`[09:42] Diego`).
2. A plataforma espera o próximo intervalo e envia de novo a mesma notificação, com o mesmo identificador (`[09:25] Diego`).
3. O reenvio para no primeiro envio que dá certo ou quando o teto se esgota (`[09:15] Diego`).
4. Esgotado o teto, a notificação passa para as entregas esgotadas do PRD-FR-09 (`[09:15] Diego`).

**Fluxos alternativos e exceções**

- O teto foi fechado como "5 tentativas" (`[09:17] Larissa`). **Hipótese:** são o envio inicial mais 5 reenvios, a leitura que usa os cinco intervalos e dá cerca de 14,6 h, a janela de quase 15 h dita na reunião (`[09:17] Diego`); confirmação na [RFC-QA-08](RFC.md#questões-em-aberto), outra contagem na questão 1 do [ADR-003](adrs/ADR-003-retry-com-backoff-e-dlq.md).
- Cliente fora do ar por mais tempo que a janela depende de reprocessamento manual, limite aceito pelo produto (`[09:17] Marcos`).
- **Proposta (a validar):** resposta fora da faixa de sucesso do HTTP e erro de rede também contam como falha (questão 2 do [ADR-003](adrs/ADR-003-retry-com-backoff-e-dlq.md)) (`[09:42] Diego`).

**Erros previstos**

- Sem resposta em 10 s, o envio conta como falha e vai para reenvio (`[09:42] Diego`); esse limite é o tempo de resposta do cliente, não a meta de notificação do PRD-RNF-01.

**Prioridade:** alta

#### PRD-FR-09: Guardar as entregas esgotadas

A notificação que esgotou o reenvio automático fica guardada com o conteúdo, o motivo da falha e o momento, à espera de reprocessamento manual (`[09:18] Diego`).

**Fluxo principal**

1. O último envio do teto falha (`[09:15] Diego`).
2. A plataforma guarda a notificação entre as entregas esgotadas, com o conteúdo, o motivo da falha e o momento (`[09:18] Diego`).
3. A notificação deixa de ser reenviada automaticamente e aguarda o reprocessamento manual do PRD-FR-10 (`[09:18] Diego`).

**Fluxos alternativos e exceções**

- **Proposta (a validar):** notificação acima de 64 KB vai direto para as entregas esgotadas, com o motivo, sem nenhum envio, e o reprocessamento dela é recusado (questão 2 do [ADR-007](adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md)) (`[09:24] Larissa`).
- **Proposta (a validar):** a operação consegue saber quantas entregas esgotadas aguardam reprocessamento ([FDD, seção 9](FDD.md#9-observabilidade)); quem as acompanha está em aberto (questão 4 do [ADR-003](adrs/ADR-003-retry-com-backoff-e-dlq.md)) (`[09:18] Diego`).

**Erros previstos**

- Cliente fora do ar além da janela de reenvio: a notificação só chega depois do reprocessamento manual (`[09:17] Marcos`).

**Prioridade:** alta

#### PRD-FR-10: Reprocessar manualmente uma entrega esgotada

O administrador pede o reprocessamento de uma entrega esgotada (`[09:36] Larissa`), que volta ao envio automático (`[09:18] Diego`).

**Fluxo principal**

1. Depois que o cliente se recupera, o administrador escolhe a entrega esgotada (`[09:18] Diego`).
2. O sistema confere que o usuário tem papel de administrador (`[09:36] Larissa`).
3. A notificação volta ao envio automático (`[09:18] Diego`).
4. O sistema registra quem fez o reprocessamento, para auditoria (`[09:36] Sofia`).

**Fluxos alternativos e exceções**

- **Proposta (a validar):** o reprocessamento reenvia a notificação com o mesmo identificador e o mesmo conteúdo; uma segunda tentativa de reprocessar a mesma entrega é recusada ([RFC-QA-11](RFC.md#questões-em-aberto)) (`[09:18] Diego`).
- **Proposta (a validar):** a notificação reprocessada é entregue como foi registrada, mesmo que o pedido já tenha notificações mais novas entregues; o cliente ordena pelo momento da mudança (questão 5 do [ADR-003](adrs/ADR-003-retry-com-backoff-e-dlq.md), [RFC-QA-05](RFC.md#questões-em-aberto)) (`[09:18] Diego`).

**Erros previstos**

- Usuário sem papel de administrador é recusado (`[09:36] Larissa`).
- **Proposta (a validar):** entrega já reprocessada é recusada ([RFC-QA-11](RFC.md#questões-em-aberto)) (`[09:18] Diego`).
- **Proposta (a validar):** entrega guardada por passar de 64 KB é recusada (questão 2 do [ADR-007](adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md)) (`[09:24] Larissa`).
- **Proposta (a validar):** entrega esgotada inexistente é recusada ([seção 7 do FDD](FDD.md#7-matriz-de-erros-previstos)) (`[09:18] Diego`); lacuna, sem questão aberta no RFC.

**Prioridade:** média

#### PRD-FR-11: Consultar o histórico de entregas

O cliente consulta os últimos 100 envios de cada webhook, com sucesso ou falha, o conteúdo enviado, a resposta e o tempo de resposta (`[09:34] Marcos`).

**Fluxo principal**

1. O usuário autenticado escolhe o webhook (`[09:34] Marcos`).
2. O sistema devolve no máximo os últimos 100 envios desse webhook, cada um com sucesso ou falha, conteúdo enviado, resposta e tempo de resposta (`[09:34] Marcos`).

**Fluxos alternativos e exceções**

- **Proposta (a validar):** os envios aparecem do mais recente para o mais antigo; nenhum prazo de retenção foi definido (lacuna, sem questão aberta no RFC) (`[09:34] Marcos`).
- **Proposta (a validar):** o histórico nunca mostra a secret (questão 5 do [ADR-004](adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md)) (`[09:31] Marcos`).

**Erros previstos**

- Webhook inexistente é recusado (`[09:28] Bruno`).
- Chamada sem usuário autenticado é recusada (`[09:32] Larissa`).

**Prioridade:** média

#### PRD-FR-12: Rotacionar a secret

O cliente pede uma secret nova pela API (`[09:21] Sofia`), e a anterior continua valendo por mais 24 h, para ele migrar os sistemas dele (`[09:21] Sofia`).

**Fluxo principal**

1. O usuário autenticado pede a rotação da secret de um webhook (`[09:21] Sofia`).
2. O sistema gera a secret nova. **Proposta (a validar):** e a devolve na resposta da rotação (questão 5 do [ADR-004](adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md)) (`[09:31] Marcos`).
3. A secret anterior continua valendo por 24 h e depois expira (`[09:21] Sofia`).

**Fluxos alternativos e exceções**

- O caso de uso típico é o vazamento da secret do lado do cliente, que já aconteceu (`[09:22] Diego`).
- **Proposta (a validar):** nas 24 h de convivência, a notificação leva as duas assinaturas, e o cliente aceita qualquer uma das duas ([RFC-QA-09](RFC.md#questões-em-aberto)) (`[09:21] Sofia`).

**Erros previstos**

- Webhook inexistente é recusado (`[09:28] Bruno`).

**Prioridade:** média

## Requisitos não funcionais

### Performance

- **PRD-RNF-01**: notificação abaixo de 10 s depois da mudança de status (`[09:02] Marcos`). A meta só é atendida quando o primeiro envio dá certo ([ADR-002](adrs/ADR-002-worker-separado-com-polling.md)); se ela vale também para notificações que precisam de reenvio é pergunta de produto para Marcos ([RFC-QA-07](RFC.md#questões-em-aberto)).
- **PRD-RNF-02**: a leitura periódica das notificações registradas causa um atraso de até 2 s no pior caso, aceito na reunião (`[09:10] Larissa`). **Proposta (a validar):** o desenho técnico pode somar um pequeno acréscimo a esses 2 s ([FDD-OBJ-02](FDD.md#2-objetivos-técnicos), [FDD-CA-04](FDD.md#12-critérios-de-aceite-técnicos)); lacuna, sem questão aberta no RFC.
- **PRD-RNF-03**: o sistema do cliente tem 10 s para responder a cada envio; depois disso o envio conta como falha e vai para reenvio (`[09:42] Diego`). Esse limite de 10 s não é a meta de notificação do PRD-RNF-01.

### Disponibilidade

A reunião não fixou percentual de disponibilidade da plataforma, e este PRD não fixa nenhum.

- **PRD-RNF-04**: a entrega das notificações não para quando a API reinicia, porque o envio roda em processo separado da API (`[09:11] Diego`).
- **PRD-RNF-05**: o reenvio automático cobre indisponibilidades do cliente como a de 2 h que já ocorreu (`[09:16] Diego`), dentro da janela de quase 15 h (`[09:17] Diego`), cuja contagem está em aberto no PRD-FR-08 ([RFC-QA-08](RFC.md#questões-em-aberto)).

### Segurança e autorização

- **PRD-RNF-06**: a URL do webhook só aceita https (`[09:23] Sofia`).
- **PRD-RNF-07**: toda notificação tem assinatura verificável pelo cliente (`[09:19] Sofia`).
- **PRD-RNF-08**: cada webhook tem uma secret única, para que o vazamento de uma não comprometa as outras (`[09:21] Sofia`).
- **PRD-RNF-09**: a secret é rotacionável, e a anterior continua valendo por 24 h depois da rotação (`[09:21] Sofia`).
- **PRD-RNF-10**: a secret é gerada pela plataforma e devolvida na criação (`[09:31] Marcos`). **Proposta (a validar):** ela é devolvida também na rotação e nunca na listagem, na edição ou no histórico (questão 5 do [ADR-004](adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md)).
- **PRD-RNF-11**: **Proposta (a validar):** a secret nunca aparece em log da plataforma, motivada pelo vazamento que já ocorreu em log de um cliente (`[09:22] Diego`); questão 3 do [ADR-004](adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md).
- **PRD-RNF-12**: o reprocessamento é só para administrador (`[09:36] Larissa`) e registra o usuário que o fez (`[09:36] Sofia`).
- **PRD-RNF-13**: as demais operações aceitam qualquer usuário autenticado nesta fase (`[09:37] Sofia`).
- **PRD-RNF-14**: o código de segurança passa por revisão antes da entrada em produção (`[09:46] Sofia`).

### Observabilidade

- **PRD-RNF-15**: o cliente vê o histórico de entregas de cada webhook, como no PRD-FR-11 (`[09:34] Marcos`).
- **PRD-RNF-16**: **Proposta (a validar):** a operação consegue saber quantas entregas esgotadas aguardam reprocessamento e se os envios pararam ([FDD, seção 9](FDD.md#9-observabilidade)) (`[09:18] Diego`, `[09:10] Larissa`); quem acompanha as entregas esgotadas está em aberto na questão 4 do [ADR-003](adrs/ADR-003-retry-com-backoff-e-dlq.md), e quem percebe que os envios pararam, nas questões 5 e 6 do [ADR-002](adrs/ADR-002-worker-separado-com-polling.md).
- **PRD-RNF-17**: o registro de quem reprocessou cada entrega fica disponível para auditoria (`[09:36] Sofia`).

### Confiabilidade e integridade de dados

- **PRD-RNF-18**: toda mudança de status de um pedido feita pela operação normal da API, que algum webhook do cliente quer receber, gera a notificação (`[09:40] Bruno`; [ADR-001](adrs/ADR-001-outbox-transacional-no-mysql.md)), e nenhuma notificação sai de uma mudança desfeita (`[09:06] Diego`). **Proposta (a validar):** as exceções são a notificação acima de 64 KB (PRD-RNF-22) e a mudança feita direto no banco, fora da operação normal da API, que não gera notificação; essa regra fica documentada para quem opera o banco (questão 2 do [ADR-001](adrs/ADR-001-outbox-transacional-no-mysql.md)).
- **PRD-RNF-19**: a entrega é feita pelo menos uma vez: a mesma notificação pode chegar mais de uma vez, e o cliente descarta as repetidas pelo identificador (`[09:25] Diego`), regra documentada com destaque no portal (`[09:26] Marcos`). O identificador é o mesmo em todos os webhooks que recebem a notificação, então o cliente com vários webhooks descarta pelo identificador junto com o webhook que recebeu ([ADR-005-CONS-07](adrs/ADR-005-entrega-at-least-once-com-x-event-id.md)). Se o reprocessamento mantém o identificador está em aberto ([RFC-QA-11](RFC.md#questões-em-aberto)).
- **PRD-RNF-20**: a notificação reflete o pedido no momento da mudança, mesmo que ele mude depois (`[09:52] Larissa`).
- **PRD-RNF-21**: os clientes nunca pediram ordem global (`[09:14] Marcos`); com um único processo de envio, a ordem por pedido vale, como decidido (`[09:13] Larissa`), mas o reenvio, o reprocessamento e os demais casos do [ADR-002](adrs/ADR-002-worker-separado-com-polling.md) podem invertê-la. **Proposta (a validar):** o cliente ordena pelo momento da mudança, que vem no conteúdo da notificação ([RFC-QA-05](RFC.md#questões-em-aberto)).
- **PRD-RNF-22**: cada notificação tem limite de 64 KB, com erro acima disso (`[09:24] Larissa`). **Proposta (a validar):** a mudança de status do pedido não é desfeita; a notificação não é enviada e fica entre as entregas esgotadas com o motivo, sem reenvio automático, e o reprocessamento dela é recusado (questão 2 do [ADR-007](adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md), [RFC-RISCO-06](RFC.md#riscos)).

### Compatibilidade

- **PRD-RNF-23**: a feature só altera o comportamento existente dentro da mudança de status do pedido (`[09:40] Bruno`), que mantém o contrato atual, e os clientes que continuarem consultando a lista de pedidos não são afetados ([FDD-DEP-10](FDD.md#10-dependências-e-compatibilidade)). **Proposta (a validar):** sem mudança na exclusão de cliente nesta fase, a exclusão de cliente que tem ou já teve webhook passa a ser recusada com erro genérico, como a hipótese do FDD prevê também para cliente que tem pedido ([FDD-RISCO-12](FDD.md#13-riscos-e-mitigação)); lacuna, sem questão aberta no RFC.

## Decisões e trade-offs

As decisões de arquitetura estão nos ADRs e no [RFC](RFC.md); aqui fica só o que importa para o produto.

### PRD-DEC-01: Notificação registrada junto com a mudança de status

- **Justificativa:** a outbox no MySQL existente registra a notificação na mesma transação da mudança de status, sem infraestrutura nova (`[09:08] Larissa`; [ADR-001](adrs/ADR-001-outbox-transacional-no-mysql.md)).
- **Trade-off:** sem infraestrutura nova, o MySQL compartilhado ganha carga contínua, com escrita na transação de status e leitura periódica ([ADR-001-CONS-10](adrs/ADR-001-outbox-transacional-no-mysql.md), [ADR-002-CONS-15](adrs/ADR-002-worker-separado-com-polling.md)).

### PRD-DEC-02: Envio em processo separado com leitura a cada 2 s

- **Justificativa:** um worker separado da API lê a outbox por polling a cada 2 s, o que atende a meta no caminho feliz e sobrevive ao reinício da API (`[09:10] Larissa`; [ADR-002](adrs/ADR-002-worker-separado-com-polling.md)).
- **Trade-off:** atraso de até 2 s e ordem por pedido que vale com um único processo de envio, mas pode se inverter no reenvio e nos demais casos do ADR ([ADR-002](adrs/ADR-002-worker-separado-com-polling.md), [RFC-QA-05](RFC.md#questões-em-aberto)).

### PRD-DEC-03: Reenvio com intervalos crescentes e entregas esgotadas guardadas à parte

- **Justificativa:** retry com backoff de 1 min a 12 h e DLQ em tabela separada, com reprocessamento manual por administrador (`[09:17] Larissa`; [ADR-003](adrs/ADR-003-retry-com-backoff-e-dlq.md)).
- **Trade-off:** a janela de reenvio é fixa; depois dela, a notificação só chega por reprocessamento manual ([ADR-003-CONS-06](adrs/ADR-003-retry-com-backoff-e-dlq.md)).

### PRD-DEC-04: Assinatura com secret por webhook

- **Justificativa:** HMAC-SHA256 sobre o corpo da notificação, secret por webhook e rotação com 24 h de convivência (`[09:22] Sofia`; [ADR-004](adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md)).
- **Trade-off:** o cliente precisa verificar a assinatura e guardar a secret, e a forma da assinatura durante a rotação está em aberto ([ADR-004-CONS-05](adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md), [RFC-QA-09](RFC.md#questões-em-aberto)).

### PRD-DEC-05: Entrega pelo menos uma vez

- **Justificativa:** at-least-once com identificador por notificação para o cliente deduplicar, sem coordenação a cada envio (`[09:26] Larissa`; [ADR-005](adrs/ADR-005-entrega-at-least-once-com-x-event-id.md)).
- **Trade-off:** o cliente precisa descartar as notificações repetidas ([ADR-005-CONS-04](adrs/ADR-005-entrega-at-least-once-com-x-event-id.md)).

### PRD-DEC-06: Reuso dos padrões do sistema existente

- **Justificativa:** o webhook é um módulo da mesma base, com os padrões de erro, validação, autenticação e log já existentes (`[09:30] Larissa`; [ADR-006](adrs/ADR-006-reuso-dos-padroes-do-projeto.md)).
- **Trade-off:** as rotas de webhook rodam e são implantadas junto com a API de pedidos, e consultas pesadas ao histórico concorrem com elas ([ADR-006-CONS-13](adrs/ADR-006-reuso-dos-padroes-do-projeto.md)).

### PRD-DEC-07: Conteúdo fixado na mudança e filtro por status antes de registrar

- **Justificativa:** o payload é guardado em snapshot no momento da mudança (`[09:52] Bruno`), e só é registrado se algum webhook do cliente quer o status (`[09:34] Bruno`) ([ADR-007](adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md)).
- **Trade-off:** a notificação não traz os itens do pedido, e o cliente consulta o pedido na API para os detalhes ([ADR-007](adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md)).

## Dependências

### PRD-DEP-01: Organizacional: Revisão de segurança

Sofia revisa o código de segurança por pelo menos 2 dias úteis antes da entrada em produção, com foco na assinatura e na geração de secret (`[09:46] Sofia`). A revisão está incluída no prazo de 3 sprints (`[09:47] Larissa`).

### PRD-DEP-02: Organizacional: Documentação no portal do desenvolvedor

Marcos documenta no portal do desenvolvedor como os clientes integram pela API (`[09:40] Marcos`) e avisa com destaque que a notificação pode chegar repetida e deve ser descartada pelo identificador (`[09:26] Marcos`).

### PRD-DEP-03: Externa: Confirmação do prazo com a Atlas

Marcos confirma o prazo com a Atlas (`[09:47] Marcos`) e atualiza os clientes (`[09:49] Marcos`), porque a data pedida é o fim de novembro e o prazo combinado é de 3 sprints (PRD-OBJ-02).

### PRD-DEP-04: Organizacional: Respostas de produto às questões em aberto

As questões [RFC-QA-05](RFC.md#questões-em-aberto) (ordem no reenvio), [RFC-QA-07](RFC.md#questões-em-aberto) (alcance da meta de 10 s) e [RFC-QA-08](RFC.md#questões-em-aberto) (contagem dos envios), em que o RFC pede a resposta ou o aceite de Marcos, precisam de resposta antes de codar. A sessão de revisão do documento de design antes de codar foi marcada só com Bruno e Diego (`[09:50] Larissa`). **Proposta (a validar):** Marcos responde nessa sessão, convidado, ou antes dela, como o RFC propõe para as lacunas; lacuna, sem questão aberta no RFC.

### PRD-DEP-05: Técnica: Usuários que representam o cliente

O cadastro de webhooks usa a autenticação da API, por usuários que representam o cliente (`[09:32] Marcos`). **Hipótese:** esses usuários são usuários comuns do sistema, porque não há vínculo entre usuário e cliente (`prisma/schema.prisma`); confirmar com Marcos e Bruno. Enquanto isso, qualquer usuário autenticado opera webhooks de qualquer cliente.

### PRD-DEP-06: Organizacional: Painel visual do time de frontend

O painel visual para o cliente é projeto separado do time de frontend, fora desta fase (`[09:40] Larissa`). Dependência futura, sem prazo: o painel dependeria do histórico de entregas do PRD-FR-11.

## Riscos e mitigação

**Proposta (a validar):** probabilidade e impacto de cada risco são estimativa da redação, não números da reunião.

### PRD-RISCO-01: Atraso na entrega e churn da Atlas

A Atlas sugeriu migrar para um concorrente se a notificação não sair até o fim do trimestre (`[09:00] Marcos`). A estimativa por bloco (1, 1, 0,5 e 0,5 sprint) já soma as 3 sprints antes do "mais um pouco" de assinatura, schemas e validações e da revisão de segurança (`[09:46] Larissa`; [RFC-RISCO-01](RFC.md#riscos)). A duração da sprint não foi dita (lacuna, sem questão aberta no RFC).

**Probabilidade**: alta.

**Impacto**: alto: perda de um dos 3 clientes que pediram a feature.

**Mitigação**:
- revisão de segurança já reservada dentro do prazo (`[09:47] Larissa`);
- confirmação do prazo com a Atlas (`[09:47] Marcos`);
- respostas de produto às questões em aberto antes de codar (PRD-DEP-04).

**Plano de contingência**: Marcos atualiza os clientes sobre o prazo (`[09:49] Marcos`).

### PRD-RISCO-02: Cliente que não descarta notificações repetidas

O cliente pode receber a mesma notificação duas vezes (`[09:25] Diego`), e a garantia joga essa responsabilidade para ele (`[09:25] Sofia`); quem não descarta processa a mudança duas vezes ([FDD-RISCO-07](FDD.md#13-riscos-e-mitigação)).

**Probabilidade**: média.

**Impacto**: médio, do lado do cliente.

**Mitigação**:
- aviso com destaque no portal do desenvolvedor (`[09:26] Marcos`);
- o mesmo identificador em todos os envios da notificação (`[09:25] Diego`);
- **Proposta (a validar):** o mesmo identificador também no reprocessamento ([RFC-QA-11](RFC.md#questões-em-aberto)).

**Plano de contingência**: mostrar ao cliente os envios repetidos pelo histórico de entregas do webhook (PRD-FR-11).

### PRD-RISCO-03: Vazamento da secret pelo cliente

Já houve cliente que vazou a secret em log da própria aplicação (`[09:22] Diego`); com a secret, terceiros conseguem assinar notificações falsas para aquele webhook ([FDD-RISCO-05](FDD.md#13-riscos-e-mitigação)).

**Probabilidade**: média.

**Impacto**: alto para o webhook afetado.

**Mitigação**:
- secret única por webhook, que limita o vazamento a um cadastro (`[09:21] Sofia`);
- **Proposta (a validar):** secret só nas respostas de criação e de rotação (questão 5 do [ADR-004](adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md));
- **Proposta (a validar):** secret nunca em log da plataforma (questão 3 do [ADR-004](adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md));
- revisão de segurança antes da entrada em produção (`[09:46] Sofia`).

**Plano de contingência**: o cliente rotaciona a secret, e a anterior ainda vale por 24 h enquanto ele migra (`[09:21] Sofia`), desde que a forma da assinatura nesse período permita verificar com as duas secrets ([RFC-QA-09](RFC.md#questões-em-aberto)).

### PRD-RISCO-04: Cliente fora do ar por mais tempo que a janela de reenvio

Um cliente fora do ar por mais de quase 15 h esgota o reenvio automático, e as notificações passam a depender de reprocessamento manual; o produto aceitou esse limite (`[09:17] Marcos`). **Hipótese:** a janela de quase 15 h vale com "5 tentativas" lidas como envio inicial mais 5 reenvios; a outra contagem, que encurta a janela, está na questão 1 do [ADR-003](adrs/ADR-003-retry-com-backoff-e-dlq.md) ([RFC-QA-08](RFC.md#questões-em-aberto)).

**Probabilidade**: baixa.

**Impacto**: médio: as notificações chegam só depois da ação de um administrador.

**Mitigação**:
- reenvio automático por quase 15 h (`[09:17] Diego`), com a contagem em aberto ([RFC-QA-08](RFC.md#questões-em-aberto));
- entregas esgotadas guardadas com o conteúdo e o motivo (`[09:18] Diego`);
- **Proposta (a validar):** a operação acompanha quantas entregas esgotadas aguardam reprocessamento (questão 4 do [ADR-003](adrs/ADR-003-retry-com-backoff-e-dlq.md)).

**Plano de contingência**: um administrador reprocessa as entregas esgotadas depois que o cliente se recupera (`[09:18] Diego`).

### PRD-RISCO-05: Notificações de um pedido chegando fora de ordem

Os clientes nunca pediram ordem global (`[09:14] Marcos`), e a ordem por pedido vale com um único processo de envio (`[09:13] Larissa`). Mesmo assim, com o reenvio em intervalos crescentes (`[09:17] Larissa`), uma notificação mais nova do mesmo pedido pode chegar antes da que espera reenvio, e o mesmo acontece no reprocessamento e nos demais casos do [ADR-002](adrs/ADR-002-worker-separado-com-polling.md) ([ADR-002-CONS-07](adrs/ADR-002-worker-separado-com-polling.md), [FDD-RISCO-08](FDD.md#13-riscos-e-mitigação), [RFC-QA-05](RFC.md#questões-em-aberto)).

**Probabilidade**: média, sempre que um envio falha.

**Impacto**: médio: um cliente que aplica a notificação sem conferir a ordem volta o pedido a um status antigo.

**Mitigação**:
- **Proposta (a validar):** o cliente ordena pelo momento da mudança, que vem no conteúdo da notificação ([RFC-QA-05](RFC.md#questões-em-aberto));
- documentação no portal de como integrar (`[09:40] Marcos`).

**Plano de contingência**: se a [RFC-QA-05](RFC.md#questões-em-aberto) escolher segurar as notificações seguintes de um pedido enquanto uma está em reenvio, opção da questão 1 do [ADR-002](adrs/ADR-002-worker-separado-com-polling.md), a inversão causada pelo reenvio deixa de ocorrer; a ordem por pedido ainda depende da [RFC-QA-06](RFC.md#questões-em-aberto) e dos demais casos do ADR-002.

### PRD-RISCO-06: Meta de 10 s não atendida quando o primeiro envio falha

A meta é notificar abaixo de 10 s (`[09:02] Marcos`), mas a notificação cujo primeiro envio falha só volta depois do primeiro intervalo de 1 min (`[09:17] Larissa`; [RFC-RISCO-05](RFC.md#riscos)).

**Probabilidade**: média.

**Impacto**: médio: a notificação chega pelo menos 1 min depois da mudança.

**Mitigação**:
- resposta de Marcos à [RFC-QA-07](RFC.md#questões-em-aberto) antes de codar (PRD-DEP-04);
- **Proposta (a validar):** acompanhamento do atraso até o primeiro envio ([FDD, seção 9](FDD.md#9-observabilidade)); lacuna, sem questão aberta no RFC.

**Plano de contingência**: se a resposta for que a meta vale também para notificações com reenvio, o desenho é revisto, como prevê a questão 3 do [ADR-002](adrs/ADR-002-worker-separado-com-polling.md).

Os riscos técnicos (envio parado, crescimento das tabelas, registro público que aceita administrador em `src/modules/auth/auth.schemas.ts`, entre outros) estão no [FDD, seção 13](FDD.md#13-riscos-e-mitigação) e no [RFC](RFC.md#riscos).

## Critérios de aceitação

Checklist objetivo que define se a feature está pronta. O detalhe técnico de cada verificação está na [seção 12 do FDD](FDD.md#12-critérios-de-aceite-técnicos).

- **PRD-CA-01**: toda mudança de status feita pela operação normal da API que um webhook do cliente escolheu gera notificação (`[09:40] Bruno`), e a mudança desfeita não gera nenhuma (`[09:06] Diego`) ([FDD-CA-01](FDD.md#12-critérios-de-aceite-técnicos), [FDD-CA-02](FDD.md#12-critérios-de-aceite-técnicos)).
- **PRD-CA-02**: webhook que não escolheu o novo status não recebe notificação (`[09:33] Bruno`) ([FDD-CA-03](FDD.md#12-critérios-de-aceite-técnicos)).
- **PRD-CA-03**: no caminho feliz, o primeiro envio sai em até 2 s depois da mudança, mais o tempo do envio (`[09:10] Larissa`); **Proposta (a validar):** mais o pequeno acréscimo do desenho técnico (PRD-RNF-02; lacuna, sem questão aberta no RFC); o critério não promete a meta de 10 s para notificação que precisa de reenvio ([RFC-QA-07](RFC.md#questões-em-aberto)) ([FDD-CA-04](FDD.md#12-critérios-de-aceite-técnicos)).
- **PRD-CA-04**: cadastro com URL sem https é recusado (`[09:23] Sofia`) ([FDD-CA-11](FDD.md#12-critérios-de-aceite-técnicos)).
- **PRD-CA-05**: **Proposta (a validar):** lista de status vazia, com repetição ou com status inexistente é recusada (`[09:33] Marcos`); lacuna, sem questão aberta no RFC ([FDD-CA-12](FDD.md#12-critérios-de-aceite-técnicos)).
- **PRD-CA-06**: sistema do cliente que não responde em 10 s gera envio com falha e recebe reenvio (`[09:42] Diego`) ([FDD-CA-05](FDD.md#12-critérios-de-aceite-técnicos)).
- **PRD-CA-07**: os reenvios seguem a progressão de intervalos de 1 min, 5 min, 30 min, 2 h e 12 h (`[09:17] Larissa`), e, esgotados os envios, a notificação fica entre as entregas esgotadas (`[09:18] Diego`). **Hipótese:** o teto é o envio inicial mais 5 reenvios, que usa os cinco intervalos ([RFC-QA-08](RFC.md#questões-em-aberto); outra contagem na questão 1 do [ADR-003](adrs/ADR-003-retry-com-backoff-e-dlq.md)) ([FDD-CA-06](FDD.md#12-critérios-de-aceite-técnicos), [FDD-CA-07](FDD.md#12-critérios-de-aceite-técnicos)).
- **PRD-CA-08**: o cliente verifica a assinatura com a secret do webhook (`[09:22] Sofia`). **Proposta (a validar):** nas 24 h depois de uma rotação, a verificação aceita a secret nova e a anterior ([RFC-QA-09](RFC.md#questões-em-aberto)) ([FDD-CA-08](FDD.md#12-critérios-de-aceite-técnicos)).
- **PRD-CA-09**: o identificador da notificação é o mesmo em todos os reenvios automáticos dela (`[09:25] Diego`); no reprocessamento, ver o PRD-CA-11 ([FDD-CA-09](FDD.md#12-critérios-de-aceite-técnicos)).
- **PRD-CA-10**: o reprocessamento é aceito só para administrador e registra o usuário que o fez (`[09:36] Larissa`, `[09:36] Sofia`) ([FDD-CA-13](FDD.md#12-critérios-de-aceite-técnicos)).
- **PRD-CA-11**: **Proposta (a validar):** o reprocessamento mantém o identificador e o conteúdo, e uma segunda tentativa sobre a mesma entrega é recusada ([RFC-QA-11](RFC.md#questões-em-aberto)) (`[09:18] Diego`) ([FDD-CA-14](FDD.md#12-critérios-de-aceite-técnicos)).
- **PRD-CA-12**: a secret aparece na resposta da criação (`[09:31] Marcos`). **Proposta (a validar):** e só nela e na da rotação (questão 5 do [ADR-004](adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md)) ([FDD-CA-15](FDD.md#12-critérios-de-aceite-técnicos)).
- **PRD-CA-13**: **Proposta (a validar):** a secret nunca aparece em log da plataforma (questão 3 do [ADR-004](adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md)) (`[09:22] Diego`) ([FDD-CA-16](FDD.md#12-critérios-de-aceite-técnicos)).
- **PRD-CA-14**: o histórico de um webhook mostra no máximo os últimos 100 envios (`[09:34] Marcos`) ([FDD-CA-17](FDD.md#12-critérios-de-aceite-técnicos)).
- **PRD-CA-15**: notificação acima de 64 KB é erro (`[09:24] Larissa`). **Proposta (a validar):** ela não desfaz a mudança de status do pedido e fica entre as entregas esgotadas (questão 2 do [ADR-007](adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md)) ([FDD-CA-10](FDD.md#12-critérios-de-aceite-técnicos)).
- **PRD-CA-16**: a revisão de segurança está concluída antes da entrada em produção (`[09:46] Sofia`); sem critério técnico correspondente no FDD, porque é verificação manual (PRD-TESTE-06).
- **PRD-CA-17**: a documentação para os clientes está publicada no portal do desenvolvedor, com o modo de integrar (`[09:40] Marcos`) e o aviso com destaque de que a notificação pode chegar repetida e deve ser descartada pelo identificador (`[09:26] Marcos`); **Proposta (a validar):** também como verificar a assinatura (lacuna, sem questão aberta no RFC); sem critério técnico correspondente no FDD.

## Estratégia de testes e validação

### Tipos de teste obrigatórios

- **PRD-TESTE-01**: testes de integração no padrão do repositório, com `vitest` e `supertest` contra o MySQL real (`package.json`, `tests/orders.test.ts`), cobrindo cadastro, escolha de status, notificação e reenvio ([seção 12 do FDD](FDD.md#12-critérios-de-aceite-técnicos)).
- **PRD-TESTE-02**: teste da mudança de status com falha forçada no registro da notificação, confirmando que a mudança é desfeita (`[09:40] Bruno`).
- **PRD-TESTE-03**: teste de verificação da assinatura com a secret do webhook (`[09:22] Sofia`).
- **PRD-TESTE-04**: teste de autorização do reprocessamento, recusado para quem não é administrador (`[09:36] Larissa`).
- **PRD-TESTE-05**: a suíte de testes existente continua passando (`package.json`, script `test`).

### Estratégia de validação

- **PRD-TESTE-06**: revisão de segurança manual de pelo menos 2 dias úteis, com foco na assinatura e na geração de secret (`[09:46] Sofia`).
- **PRD-TESTE-07**: revisão do documento de design com Bruno e Diego antes de codar (`[09:50] Larissa`).
- **PRD-TESTE-08**: **Proposta (a validar):** o Product Manager confere a documentação do portal antes de liberá-la aos clientes (`[09:40] Marcos`); lacuna, sem questão aberta no RFC.

**Proposta (a validar):** nenhum teste de carga entra nesta fase, porque a única meta de desempenho é a do PRD-RNF-01; lacuna, sem questão aberta no RFC.
