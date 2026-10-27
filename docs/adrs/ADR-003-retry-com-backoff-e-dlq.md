# ADR-003: Retry com backoff exponencial e DLQ em tabela separada

## Status

**Aceito**, decidido na reunião técnica de quinta-feira (redigido em 2026-10-09, revisado em 2026-10-18).

> Aceito quanto ao retry com backoff exponencial nos intervalos 1 min, 5 min, 30 min, 2 h e 12 h, à DLQ em tabela separada e ao reprocessamento manual restrito a ADMIN, com auditoria. O número de envios por evento e, com ele, a janela efetiva de reentrega dependem da questão 1.

- **Decisores**: Diego (Engenheiro Sênior, Plataforma), Larissa (Tech Lead), Sofia (Engenheira de Segurança).
- **Consultados**: Bruno (Engenheiro Pleno, Pedidos), Marcos (Product Manager).

## Contexto

O worker do [ADR-002](ADR-002-worker-separado-com-polling.md) vai encontrar clientes fora do ar. Já houve cliente indisponível por 2 h em manutenção planejada. O código atual não tem retry nem backoff em `src/`.

Neste ADR, "envio" é cada chamada HTTP ao endpoint do cliente e "retentativa" é um envio feito depois de uma falha. "Tentativa" é a palavra usada na reunião, com contagem ambígua (CONS-05), e aparece só entre aspas ou no nome das alternativas. "Estado da outbox" é o estado técnico da linha da outbox, com os nomes dados na reunião: pendente, processando, falhou e entregue. "DLQ" é a tabela `webhook_dead_letter`, separada da outbox; o estado da outbox falhou não é a DLQ.

## Fatores de decisão

- Cobrir indisponibilidades de horas do cliente, como a de 2 h já vista.
- Nenhum evento pode ficar sendo reenviado para sempre.
- A leitura da outbox deve continuar simples.
- A janela de quase 15 h entre a primeira falha e o último envio foi aceita pelo produto (`[09:17] Marcos`).

## Decisão

- Um envio que falha é repetido com backoff exponencial, nos intervalos 1 min, 5 min, 30 min, 2 h e 12 h.
- O teto foi fechado como "5 tentativas". Só a leitura "envio inicial mais 5 retentativas" usa os cinco intervalos e fecha a janela de quase 15 h; a contagem exata está em aberto (CONS-05, questão 1).
- Um envio sem resposta em 10 s (timeout HTTP) conta como falha. As demais respostas que contam como falha não foram definidas (CONS-11).
- Esgotados os envios, o evento é falha permanente e vai para a DLQ, que guarda o evento e o motivo da falha.
- O reprocessamento é manual, por um endpoint administrativo que devolve o evento à outbox no estado pendente. Ele é restrito a usuários com papel ADMIN, pelo controle de papéis que a API já tem, e registra quem o fez, para auditoria.

Caminho do endpoint, cálculo do momento do próximo envio e tabela da DLQ estão no FDD, seções 4 Modelo de dados, 5.3 Retry com backoff, 5.4 DLQ e replay manual e 6.1 API de configuração de webhooks.

## Alternativas Consideradas

### ADR-003-ALT-01: Retry indefinido

- **Descrição**: repetir o envio com backoff, sem teto.
- **Prós**: **Derivada:** nenhum evento sai da entrega automática enquanto o cliente não voltar.
- **Contras**: o evento fica sendo reenviado para sempre se o cliente sumiu.
- **Motivo do descarte**: um teto com DLQ encerra a entrega automática de forma previsível.

### ADR-003-ALT-02: 3 tentativas

- **Descrição**: teto de "3 tentativas", na palavra da reunião.
- **Prós**: mais agressivo. **Derivada:** o evento chega antes à DLQ.
- **Contras**: na fala, três retentativas caberiam em cerca de 30 min (1 + 5 + 30 = 36 min, com os três primeiros intervalos), menos que a indisponibilidade de 2 h já vista.
- **Motivo do descarte**: 3 é pouco para cobrir indisponibilidades reais.

### ADR-003-ALT-03: Marcar "falhou" na própria outbox

- **Descrição**: deixar o evento que esgotou os envios na outbox, no estado falhou, em vez de usar uma tabela separada.
- **Prós**: **Derivada:** uma tabela a menos para modelar e migrar.
- **Contras**: a leitura da outbox fica menos limpa e o evento perde um lugar próprio de evidência.
- **Motivo do descarte**: uma tabela separada deixa a outbox mais limpa e serve de evidência para depuração e reprocessamento.

## Consequências

### Positivas

- **ADR-003-CONS-01**: A janela cobre indisponibilidades de horas, como a de 2 h já vista, e foi aceita pelo produto (`[09:17] Marcos`).
- **ADR-003-CONS-02**: Nenhum evento fica sendo reenviado para sempre.
- **ADR-003-CONS-03**: A outbox tem leitura mais limpa e a DLQ guarda evidência para depuração e reprocessamento.
- **ADR-003-CONS-04**: Um evento na DLQ volta à entrega sem intervenção direta no banco, pelo reprocessamento, com registro de quem o fez. Exceção em aberto: evento acima de 64 KB guardado na DLQ, que voltaria à outbox acima do limite (questão 2 do [ADR-007](ADR-007-payload-snapshot-e-filtro-na-insercao.md)).

### Negativas

#### Contagem e janela

- **ADR-003-CONS-05**: **Derivada:** "5 tentativas" e "cinco intervalos com quase 15 h" só fecham juntos com 6 envios. Com 5 envios no total, só os quatro primeiros intervalos são usados e a janela cai para 156 min, cerca de 2,6 h (1 + 5 + 30 + 120 min). Com envio inicial mais 5 retentativas, são 6 envios e 876 min, cerca de 14,6 h (1 + 5 + 30 + 120 + 720 min). A janela também delimita por quanto tempo o cliente guarda os identificadores para deduplicar ([ADR-005-CONS-10](ADR-005-entrega-at-least-once-com-x-event-id.md)). Ver questão 1.
- **ADR-003-CONS-06**: Um cliente fora do ar por mais tempo que a janela perde a entrega automática: o evento vai para a DLQ e só volta por reprocessamento de um ADMIN. O produto aceitou esse limite para a janela de quase 15 h (`[09:17] Marcos`); se a resposta da questão 1 for 5 envios no total, a janela cai para cerca de 2,6 h, valor que não foi aceito.

#### Semântica da falha

- **ADR-003-CONS-07**: **Derivada:** a reunião citou o estado da outbox falhou sem definir o significado, e a DLQ é uma tabela separada. Não está definido se uma linha no estado falhou aguarda o próximo envio ou se é um estado final antes de ir para a DLQ. Ver questão 3.
- **ADR-003-CONS-11**: **Derivada:** só o timeout de 10 s foi definido como falha. Não está definido se uma resposta de erro do cliente, como um 4xx de URL errada ou de autenticação recusada, deve ser repetida. Repetir um erro que não vai mudar gasta toda a janela e ocupa o worker. Ver questão 2.

#### Ordem e conteúdo reentregue

- **ADR-003-CONS-10**: **Derivada:** o retry reordena eventos do mesmo pedido mesmo com um único worker: enquanto um evento espera o próximo intervalo, um evento posterior do mesmo pedido pode ser entregue antes. O cenário e a questão estão no [ADR-002](ADR-002-worker-separado-com-polling.md) (CONS-07, questão 1).
- **ADR-003-CONS-14**: **Derivada:** o reprocessamento entrega o snapshot gravado na inserção ([ADR-007](ADR-007-payload-snapshot-e-filtro-na-insercao.md)), que pode ser bem mais antigo que eventos do mesmo pedido já entregues. Um cliente que aplica o evento sem conferir a ordem pode voltar o pedido a um status do pedido antigo. Se o reprocessamento mantém o identificador do evento está em aberto no [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md) (questão 1). A perda de ordem que isso causa está no [ADR-002-CONS-16](ADR-002-worker-separado-com-polling.md). Ver questão 5.

#### Operação do retry e da DLQ

- **ADR-003-CONS-12**: **Derivada:** nada avisa que um evento entrou na DLQ. O projeto não tem biblioteca de métricas nem ferramenta de alerta, e o e-mail de aviso ao cliente ficou para uma fase futura. O evento fica parado até alguém consultar a tabela ou o cliente reclamar. Ver questão 4.
- **ADR-003-CONS-13**: **Derivada:** um cliente fora do ar multiplica a carga: cada evento dele passa por todos os envios do teto. Se o endpoint não responde, cada envio ocupa o worker por até 10 s, o que agrava o efeito do envio uma linha de cada vez descrito no [ADR-002](ADR-002-worker-separado-com-polling.md) (CONS-11).
- **ADR-003-CONS-15**: **Derivada:** a DLQ cresce sem política de retenção, como a outbox ([ADR-001](ADR-001-outbox-transacional-no-mysql.md), CONS-06), e guarda uma cópia do payload de cada evento.
- **ADR-003-CONS-16**: **Derivada:** o agendamento do próximo envio tem de sobreviver a um reinício do worker. O número de envios feitos e o momento do próximo precisam ficar gravados na linha da outbox (FDD, seção 4), e não só na memória do processo; senão, um reinício zera a contagem ou perde o agendamento.

#### Autorização do reprocessamento

- **ADR-003-CONS-08**: O controle por papel ADMIN não é forte no reprocessamento: `POST /api/v1/auth/register` é público (`src/modules/auth/auth.routes.ts`, `buildAuthRouter`) e aceita `role: 'ADMIN'` (`src/modules/auth/auth.schemas.ts`, `registerSchema`), então qualquer pessoa consegue um usuário ADMIN. O risco fica registrado para a revisão de segurança, sem decisão aqui. Ver questão 6.

### Trade-off

- **ADR-003-CONS-09**: Troca a entrega automática sem limite por uma janela finita com DLQ. Ganha eventos que nunca ficam sendo reenviados para sempre e evidência separada para reprocessar. Paga com uma tabela a mais, reprocessamento manual para o cliente que ficar fora além da janela, ordem por pedido que o retry não preserva e uma janela cujo tamanho real ainda depende da contagem dos envios.

## Confirmação

- **Proposta (a validar):** incluir o risco do ADR-003-CONS-08 na revisão de segurança antes do deploy, cujo foco definido na reunião é HMAC e geração de secret ([ADR-004](ADR-004-assinatura-hmac-sha256-por-endpoint.md)).
- **Proposta (a validar):** teste que faz todos os envios falharem e confirma o evento na `webhook_dead_letter` e fora das linhas pendentes da outbox, segundo a contagem que a questão 1 definir.

## Referências

- Decisão (ADR-003):
  - `[09:15] Diego`: backoff exponencial, teto de tentativas, falha permanente vai para a DLQ; sugere 5.
  - `[09:17] Diego`: progressão 1 min, 5 min, 30 min, 2 h, 12 h; quase 15 h entre primeira falha e última tentativa.
  - `[09:17] Larissa`: fecha "5 tentativas, backoff 1m/5m/30m/2h/12h".
  - `[09:18] Diego`: tabela `webhook_dead_letter` com payload, motivo e timestamp; reprocessamento manual por endpoint admin que recoloca na outbox como pendente.
  - `[09:19] Larissa`: anota o endpoint de reprocessamento.
  - `[09:36] Sofia`: papel ADMIN e log de quem fez o reprocessamento.
  - `[09:36] Larissa`: fecha ADMIN reaproveitando o controle de papéis existente.
  - `[09:42] Diego`: sem resposta em 10 s conta como falha e vai para retry.
- Contexto e fatores: `[09:14] Larissa` (cliente offline), `[09:16] Diego` (indisponibilidade de 2 h), `[09:08] Diego` (estados da outbox), `[09:17] Marcos` (aceite da janela).
- ADR-003-ALT-01: `[09:15] Diego` (descarte do retry indefinido).
- ADR-003-ALT-02: `[09:16] Bruno` (propõe 3, "mais agressivo"), `[09:16] Diego` (descarte), `[09:16] Larissa` (fica com cinco).
- ADR-003-ALT-03: `[09:17] Larissa` (levanta "failed" na outbox), `[09:18] Diego` (descarte), `[09:18] Bruno` (apoia a tabela separada).
- Consequências: CONS-01 `[09:16] Diego`, `[09:17] Diego` e `[09:17] Marcos`; CONS-02 `[09:15] Diego`; CONS-03 e CONS-04 `[09:18] Diego` e `[09:36] Sofia` (CONS-04 com a questão 2 do ADR-007 e `[09:24] Larissa`, limite de 64 KB); CONS-05 `[09:17] Diego` e `[09:17] Larissa`, com o ADR-005-CONS-10; CONS-06 `[09:17] Marcos` e `[09:36] Larissa`; CONS-07 `[09:08] Diego` e `[09:18] Diego`; CONS-08 `src/modules/auth/auth.routes.ts` e `src/modules/auth/auth.schemas.ts`; CONS-10 `[09:17] Larissa` (backoff), com o ADR-002-CONS-07; CONS-11 `[09:42] Diego`; CONS-12 `[09:37] Larissa` (e-mail fora desta fase) e `package.json` (sem biblioteca de métricas); CONS-13 `[09:42] Diego`; CONS-14 `[09:18] Diego` ("Recoloca na outbox como pendente"), com o snapshot do ADR-007 e o ADR-002-CONS-16; CONS-15 `[09:18] Diego` (DLQ com o payload); CONS-16 `[09:17] Larissa` (backoff com intervalos crescentes); CONS-09 `[09:18] Diego`.
- Código:
  - `src/middlewares/auth.middleware.ts`: `requireRole`, controle de papéis existente.
  - `src/modules/auth/auth.routes.ts`: `buildAuthRouter`, registro sem autenticação.
  - `src/modules/auth/auth.schemas.ts`: `registerSchema`, aceita ADMIN.
  - `package.json`: sem biblioteca de métricas, alerta ou retry.
- ADRs relacionados: [ADR-001](ADR-001-outbox-transacional-no-mysql.md) (outbox), [ADR-002](ADR-002-worker-separado-com-polling.md) (worker, ordem e envio do ciclo), [ADR-004](ADR-004-assinatura-hmac-sha256-por-endpoint.md) (revisão de segurança), [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md) (reentregas e identificador no reprocessamento), [ADR-007](ADR-007-payload-snapshot-e-filtro-na-insercao.md) (snapshot).
- Documentos: [FDD](../FDD.md), seções 4, 5.3, 5.4, 6.1 e 9; [PRD](../PRD.md).

## Questões em aberto

1. **"5 tentativas" são 5 envios no total ou envio inicial mais 5 retentativas?** Opções:
   - 5 envios no total: usa os quatro primeiros intervalos e a janela fica em 156 min, cerca de 2,6 h, abaixo das quase 15 h aceitas pelo produto;
   - envio inicial mais 5 retentativas: usa os cinco intervalos e a janela fica em 876 min, cerca de 14,6 h.

   Afeta o núcleo da decisão. A resposta muda o atraso máximo da opção (b) da questão 1 do [ADR-002](ADR-002-worker-separado-com-polling.md) e a janela de retenção da questão 3 do [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md). Responde: arquitetura (Larissa), com aceite de produto (Marcos) sobre a janela. Origem: CONS-05.
2. **Quais respostas do cliente contam como falha que deve ser repetida?** Opções:
   - toda resposta fora da faixa 2xx e todo timeout são repetidos até o teto;
   - parte das respostas 4xx vai direto para a DLQ, sem repetir. Exige listar quais códigos (FDD, seção 5.3).

   Afeta detalhes, não o que foi decidido. Responder junto com a questão 3 e com a questão 2 do [ADR-007](ADR-007-payload-snapshot-e-filtro-na-insercao.md), que também discute um caminho para a DLQ sem esgotar os envios. Responde: arquitetura (Larissa). Origem: CONS-11.
3. **O que significa o estado da outbox falhou?** Opções:
   - a linha falhou no último envio e aguarda o próximo intervalo;
   - estado final da linha que esgotou os envios, mantido na outbox depois da cópia para a DLQ;
   - o estado não é usado. Exige reconciliar com os estados citados na reunião (modelo no FDD, seção 4).

   Afeta detalhes, não o que foi decidido. Responder junto com a questão 2. Responde: arquitetura (Larissa). Origem: CONS-07.
4. **Quem acompanha a DLQ e como fica sabendo de um evento novo nela?** Opções:
   - consulta periódica manual à tabela por alguém da operação;
   - log no logger Pino a cada evento movido para a DLQ. Exige alguém ou algo que leia esses logs;
   - alerta automático. Exige ferramenta de alerta, que o projeto não tem.

   Afeta operação, não o que foi decidido. Responde: produto (Marcos) e plataforma (Diego). Origem: CONS-12.
5. **O reprocessamento deve entregar um snapshot antigo mesmo que o pedido já tenha eventos mais novos entregues?** Opções:
   - entregar como está e documentar que o cliente ordena os eventos de um pedido. Exige o campo de momento da mudança no payload (questão 1, opção (a), do [ADR-002](ADR-002-worker-separado-com-polling.md));
   - recusar ou sinalizar o reprocessamento quando o pedido já tem evento mais novo entregue. Exige consultar as entregas do pedido antes de reprocessar.

   Afeta detalhes, não o que foi decidido. Depende da questão 1 do [ADR-002](ADR-002-worker-separado-com-polling.md) e da questão 1 do [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md). Responde: produto (Marcos) e arquitetura (Larissa). Origem: CONS-14.
6. **O registro público que aceita papel ADMIN deve ser fechado antes do go-live desta feature?** Opções:
   - aceitar o risco nesta fase e registrar na revisão de segurança;
   - fechar o registro de ADMIN. Exige mudar o módulo de autenticação (`src/modules/auth/`), fora do escopo desta feature.

   Afeta operação e segurança, não o que foi decidido. Responde: segurança (Sofia). Origem: CONS-08.

## Recomendações

- O reprocessamento manual da DLQ e a sua autorização (papel ADMIN, auditoria, risco do registro público) aparecem aqui como parte do retry, mas têm questões próprias (5 e 6). Recomenda-se avaliar uma ADR própria para o reprocessamento quando essas questões forem respondidas.
