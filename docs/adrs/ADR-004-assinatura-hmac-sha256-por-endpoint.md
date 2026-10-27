# ADR-004: Assinatura HMAC-SHA256 com secret por endpoint

## Status

**Aceito**, decidido na reunião técnica de quinta-feira (redigido em 2026-10-09, revisado em 2026-10-18).

> Aceito quanto à assinatura HMAC-SHA256 sobre o corpo do request no header `X-Signature`, à secret única por endpoint gerada pela plataforma e à rotação com 24 h de convivência. Como a assinatura se comporta durante a convivência e se o `X-Timestamp` entra no que é assinado dependem das questões 1 e 2.

- **Decisores**: Sofia (Engenheira de Segurança), Marcos (Product Manager).
- **Consultados**: Bruno (Engenheiro Pleno, Pedidos), Diego (Engenheiro Sênior, Plataforma).

## Contexto

Os eventos levam dados de pedidos para um endpoint fora da infraestrutura da plataforma. O cliente precisa conseguir validar que a requisição veio da plataforma e que o corpo não foi adulterado no caminho. Já houve cliente que vazou secret no log da própria aplicação.

O código não tem HMAC nem geração de secret: a única assinatura existente é a do JWT, em `AuthService.signToken` (`src/modules/auth/auth.service.ts`). A exigência de URL https é validação de schema, não decisão arquitetural, e fica fora deste ADR.

Neste ADR, "secret do endpoint" é a chave compartilhada entre a plataforma e um endpoint de webhook do cliente; ela não tem relação com o `JWT_SECRET` da API (`src/config/env.ts`). "Timestamp do payload" é o campo `timestamp` dentro do corpo do evento; "`X-Timestamp`" é o header com o momento do envio. São valores diferentes.

## Fatores de decisão

- Padrão de mercado, com biblioteca disponível do lado do cliente.
- O vazamento de uma secret não pode expor todos os clientes.
- O cliente precisa conseguir trocar a secret sem parar a integração.

## Decisão

- A plataforma assina o corpo do request com HMAC-SHA256 e envia a assinatura no header `X-Signature`. O cliente verifica a assinatura do lado dele.
- A assinatura garante a origem e a integridade do corpo. Ela não garante frescor nem protege contra o reenvio de uma requisição capturada (CONS-07).
- Cada endpoint de webhook do cliente tem uma secret do endpoint única, gerada pela plataforma e devolvida na criação do webhook.
- A secret do endpoint é rotacionável pela API. Depois da rotação, a secret anterior continua válida por 24 h em paralelo com a nova e depois expira. Como essa convivência se reflete na assinatura enviada está em aberto (CONS-06, questão 1).

Formato do valor do header, geração da secret, endpoint de rotação e exemplo de verificação estão no FDD, seções 5.5 Rotação de secret, 6.1 API de configuração de webhooks e 6.2 Contrato de entrega (request enviado ao cliente).

## Alternativas Consideradas

### ADR-004-ALT-01: Secret global da plataforma

- **Descrição**: uma única secret da plataforma para assinar os eventos de todos os clientes.
- **Prós**: **Derivada:** uma secret só para gerar, guardar e rotacionar.
- **Contras**: se uma vaza, vazam todas as integrações.
- **Motivo do descarte**: o vazamento de uma secret global exporia todos os clientes.

## Consequências

### Positivas

- **ADR-004-CONS-01**: O cliente consegue validar a origem do request e a integridade do corpo.
- **ADR-004-CONS-02**: O vazamento de uma secret do endpoint afeta um endpoint, não todos.
- **ADR-004-CONS-03**: A rotação responde ao caso real de vazamento em log, e a convivência de 24 h foi pensada para dar ao cliente tempo de migrar os sistemas dele. Esse tempo só existe na prática se a forma da assinatura durante a convivência permitir validar com as duas secrets (CONS-06, questão 1). Uma segunda rotação dentro das 24 h também mexe nesse tempo, e o que acontece nela não foi definido (questão 6).

### Negativas

#### Assinatura e rotação

- **ADR-004-CONS-06**: **Derivada:** quem assina é a plataforma, então "a secret anterior continua válida" só tem efeito se a assinatura enviada durante as 24 h permitir ao cliente validar com qualquer uma das duas. A reunião fixou a convivência, mas não como a assinatura funciona nela. Ver questão 1.
- **ADR-004-CONS-07**: **Derivada:** a assinatura cobre só o corpo. Quem capturar uma requisição pode reenviá-la depois, com o mesmo corpo e a mesma `X-Signature`, e trocar o `X-Timestamp` à vontade sem invalidar a assinatura. O `X-Timestamp` foi pensado para o cliente detectar esse tipo de reenvio, mas, sem assinatura, não serve para isso. A defesa que resta ao cliente é deduplicar pelo `event_id`, que está dentro do corpo assinado ([ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md), CONS-11). Ver questão 2.

#### Guarda e exposição da secret

- **ADR-004-CONS-08**: **Derivada:** o logger do projeto mascara `authorization`, `cookie`, `password`, `passwordHash`, `token` e `accessToken`, mas não `secret` (`redactPaths` em `src/shared/logger/index.ts`). Se a configuração de um webhook for logada, a secret do endpoint sai em texto claro. Ver questão 3.
- **ADR-004-CONS-10**: **Derivada:** para assinar, a plataforma precisa ler a secret do endpoint em claro. Ela não pode guardar só um hash, como faz com senhas (`bcrypt` em `src/modules/users/user.service.ts`). Se a secret também for guardada em claro (questão 4 respondida com "não"), um vazamento do banco expõe todas as secrets ativas e as anteriores ainda dentro das 24 h. Ver questão 4.
- **ADR-004-CONS-11**: **Derivada:** a secret do endpoint é devolvida na criação; não foi definido se outras respostas da API (listagem, edição) também a devolvem. Se só a criação e a rotação a devolvem, o cliente que perde a secret só a recupera com uma rotação. Ver questão 5.

#### Lado do cliente

- **ADR-004-CONS-05**: O cliente precisa implementar a verificação da assinatura do lado dele.

### Trade-off

- **ADR-004-CONS-09**: Troca a simplicidade de uma secret única por isolamento de vazamento. Ganha um impacto limitado a um endpoint quando uma secret vaza. Paga com uma secret por cadastro que precisa ser lida em claro para assinar, um fluxo de rotação com 24 h de convivência ainda sem forma definida e a verificação implementada por cada cliente, sem proteção contra reenvio além da deduplicação.

## Confirmação

- Sofia revisa o código de segurança, com foco em HMAC e geração de secret, por pelo menos 2 dias úteis antes do deploy.
- **Proposta (a validar):** incluir na mesma revisão o risco de autorização do reprocessamento registrado no [ADR-003](ADR-003-retry-com-backoff-e-dlq.md) (CONS-08).
- **Proposta (a validar):** teste que recalcula o HMAC-SHA256 do corpo enviado com a secret do endpoint e compara com o `X-Signature`, incluindo um caso dentro das 24 h após a rotação, conforme a resposta da questão 1.

## Referências

- Decisão (ADR-004):
  - `[09:20] Sofia`: assinatura HMAC com secret compartilhada, header `X-Signature`, verificação pelo cliente; SHA-256 como padrão de mercado.
  - `[09:21] Sofia`: secret única por endpoint; secret rotacionável pela API; anterior válida por 24 h em paralelo.
  - `[09:22] Sofia`: fecha "HMAC-SHA256 sobre o corpo do request, secret por endpoint, rotação com grace period de 24h".
  - `[09:31] Marcos`: secret gerada pela plataforma e devolvida na criação.
- Contexto e fatores: `[09:19] Sofia` (origem e integridade), `[09:20] Sofia` (bibliotecas do cliente), `[09:21] Sofia` (vazamento de uma não expõe todos), `[09:22] Diego` (vazamento em log de cliente), `[09:23] Sofia` (https não é decisão arquitetural).
- ADR-004-ALT-01: `[09:21] Sofia` (descarte da secret global).
- Consequências: CONS-01 `[09:19] Sofia`; CONS-02 `[09:21] Sofia`; CONS-03 `[09:21] Sofia` e `[09:22] Diego`; CONS-05 `[09:20] Sofia`; CONS-06 `[09:21] Sofia`; CONS-07 `[09:22] Sofia` (assinatura sobre o corpo), `[09:43] Diego` (`event_id` e timestamp no payload) e `[09:44] Diego` (`X-Timestamp` do envio, para detectar replay); CONS-08 `[09:22] Diego`; CONS-10 `[09:20] Sofia` (secret compartilhada); CONS-11 `[09:31] Marcos`; CONS-09 `[09:21] Sofia`.
- Confirmação: `[09:46] Sofia` (revisão de pelo menos 2 dias úteis, foco em HMAC e geração de secret).
- Código:
  - `src/modules/auth/auth.service.ts`: `AuthService.signToken`, única assinatura existente.
  - `src/config/env.ts`: `JWT_SECRET`, sem relação com a secret do endpoint.
  - `src/shared/logger/index.ts`: `redactPaths`, sem `secret`.
  - `src/modules/users/user.service.ts`: senha guardada com `bcrypt`.
- ADRs relacionados: [ADR-003](ADR-003-retry-com-backoff-e-dlq.md) (risco de autorização do reprocessamento), [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md) (garantia de entrega e deduplicação), [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md) (logger reaproveitado).
- Documentos: [FDD](../FDD.md), seções 5.5, 6.1, 6.2, 9 e 10; [PRD](../PRD.md).

## Questões em aberto

1. **Como a assinatura funciona nas 24 h em que as duas secrets valem?** Opções:
   - enviar duas assinaturas, uma com cada secret. Exige definir um formato do `X-Signature` com mais de um valor (FDD, seção 6.2);
   - assinar só com a secret nova. A secret anterior não tem efeito na verificação: o cliente precisa trocar para a nova no momento da rotação, e a convivência de 24 h perde o sentido;
   - assinar com a secret anterior até o cliente confirmar a nova. Exige um passo de confirmação na API, que não foi discutido.

   Afeta o núcleo da decisão. Responder junto com a questão 2, porque as duas mexem no formato da assinatura. Responde: segurança (Sofia). Origem: CONS-06.
2. **O `X-Timestamp` deve entrar no que é assinado?** Opções:
   - sim: muda o que é assinado (corpo mais momento do envio) e exige definir esse formato no FDD, seção 6.2. Exige também uma tolerância de horário do lado do cliente;
   - não: a proteção contra reenvio fica só na deduplicação pelo `event_id` do lado do cliente, que depende da janela de retenção da questão 3 do [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md).

   Afeta o núcleo da decisão. Responder junto com a questão 1. Responde: segurança (Sofia). Origem: CONS-07.
3. **Como evitar que a secret do endpoint apareça em log?** Opções:
   - incluir caminhos de `secret` no `redactPaths` de `src/shared/logger/index.ts`, código compartilhado por toda a API;
   - garantir que o módulo nunca loga a configuração completa do webhook.

   Afeta detalhes, não o que foi decidido. Responder junto com a questão 4. Responde: segurança (Sofia). Origem: CONS-08.
4. **A secret do endpoint deve ser cifrada no banco?** Opções:
   - sim: exige uma chave de cifragem e um lugar para ela; hoje a única chave da aplicação é o `JWT_SECRET` em variável de ambiente (`src/config/env.ts`);
   - não: a proteção fica no controle de acesso ao banco.

   Afeta detalhes, não o que foi decidido. Responder junto com a questão 3. Responde: segurança (Sofia) com plataforma (Diego). Origem: CONS-10.
5. **Quais respostas da API devolvem a secret do endpoint?** Opções:
   - só a criação e a rotação;
   - também a listagem e a edição.

   Afeta detalhes, não o que foi decidido. Responde: segurança (Sofia) com produto (Marcos). Origem: CONS-11.
6. **O que acontece com uma nova rotação pedida dentro das 24 h de convivência?** Opções:
   - a secret mais antiga deixa de valer, e só duas secrets valem ao mesmo tempo. Encurta para essa secret a convivência de 24 h da Decisão;
   - recusar a nova rotação até a secret anterior expirar. Exige um erro novo na API de rotação (FDD, seção 7) e impede trocar de novo uma secret que vazou logo depois da rotação;
   - manter as três secrets válidas. Exige assinar com três valores durante a convivência, o que depende da questão 1.

   Afeta detalhes, não o que foi decidido, salvo a primeira opção, que encurta a convivência. Depende da questão 1. Responde: segurança (Sofia). Origem: CONS-03.
