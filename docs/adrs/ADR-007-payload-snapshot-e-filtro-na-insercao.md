# ADR-007: Payload em snapshot e filtro por status na inserção da outbox

## Status

**Aceito**, decidido na reunião técnica de quinta-feira (redigido em 2026-10-09, revisado em 2026-10-18).

- **Decisores**: Marcos (Product Manager), Bruno (Engenheiro Pleno, Pedidos), Larissa (Tech Lead), Diego (Engenheiro Sênior, Plataforma).
- **Consultados**: nenhum participante além dos decisores opinou sobre esta decisão.

## Contexto

A outbox do [ADR-001](ADR-001-outbox-transacional-no-mysql.md) recebe o evento dentro da transação do `changeStatus`. Este ADR registra o que a outbox guarda e em que condição o evento é gravado, pontos fechados em dois momentos da reunião: cada endpoint de webhook escolhe quais eventos quer receber, e o pedido pode mudar depois do evento.

Neste ADR, "status do pedido" é o status de domínio (`OrderStatus` em `prisma/schema.prisma`, com as transições válidas em `transitions`, `src/modules/orders/order.status.ts`). A lista de status que um webhook quer ouvir usa esses valores. "Estado da outbox" é o estado técnico da linha (pendente, processando, falhou, entregue). "Payload" é o corpo JSON enviado ao cliente. "Estado ativo" é o campo da configuração do webhook que indica se ele está ligado. O cenário do cliente que só quer alguns status está no [PRD](../PRD.md).

## Fatores de decisão

- O evento deve refletir o estado do pedido no momento da mudança de status.
- Não gravar linhas que nenhum webhook vai receber.
- Payload enxuto.

## Decisão

- A linha da outbox guarda o payload já montado no momento da inserção, como snapshot.
- O payload leva os dados básicos do pedido, sem os itens. Quem quiser detalhes consulta o pedido pela API que já existe.
- Cada webhook tem a lista dos status do pedido que quer ouvir.
- O filtro é aplicado na inserção: se nenhum webhook do cliente quer aquele status do pedido, o evento não é gravado.

Se a outbox terá uma linha por evento ou uma por webhook de destino está em aberto no [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md) (questão 2). A lista de campos do payload e a regra do filtro estão no FDD, seções 5.1 Criação do evento na outbox e 6.2 Contrato de entrega (request enviado ao cliente).

## Alternativas Consideradas

### ADR-007-ALT-01: Guardar só o id do pedido e montar o payload no envio

- **Descrição**: a outbox guardaria só o identificador do pedido, e o worker montaria o payload na hora do envio.
- **Prós**: **Derivada:** linha menor na outbox.
- **Contras**: se o pedido mudar depois, o evento deixa de refletir o estado de quando o status do pedido mudou.
- **Motivo do descarte**: o snapshot na inserção evita esse caso.

### ADR-007-ALT-02: Filtrar os eventos na hora do envio

- **Descrição**: gravar todo evento e decidir no envio quais webhooks o recebem.
- **Prós**: **Derivada:** uma mudança na lista de status do webhook valeria também para eventos já gravados.
- **Contras**: grava linhas que nenhum webhook vai receber.
- **Motivo do descarte**: filtrar na inserção economiza linhas na tabela.

### ADR-007-ALT-03: Mandar os itens do pedido no payload

- **Descrição**: incluir os itens do pedido no payload do evento.
- **Prós**: **Derivada:** o cliente não precisaria consultar o pedido para ver os itens.
- **Contras**: infla o payload.
- **Motivo do descarte**: os detalhes ficam em `GET /api/v1/orders/:id`, e o payload fica enxuto.

## Consequências

### Positivas

- **ADR-007-CONS-01**: O evento reflete o estado de quando o status do pedido mudou, mesmo que o pedido mude depois.
- **ADR-007-CONS-02**: A outbox não recebe linhas que ninguém vai receber.
- **ADR-007-CONS-03**: **Derivada:** o worker não precisa ler o pedido no envio, e os envios repetidos do retry mandam o mesmo conteúdo. O reprocessamento só manda o mesmo conteúdo se a questão 1 do [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md) for respondida com "mesmo identificador", porque o `event_id` está dentro do payload.

### Negativas

#### Conteúdo e frescor do payload

- **ADR-007-CONS-06**: **Derivada:** o payload entregue pode estar defasado em relação ao pedido atual, inclusive depois de horas de retry ([ADR-003](ADR-003-retry-com-backoff-e-dlq.md)). O cliente busca o estado atual em `GET /api/v1/orders/:id`.
- **ADR-007-CONS-07**: **Derivada:** a lista de dados básicos do pedido não foi fechada; só um campo de valor foi citado como exemplo. Ver questão 1.
- **ADR-007-CONS-09**: **Derivada:** o limite de 64 KB do payload, com erro caso ultrapasse, encontra o snapshot e a regra do [ADR-001](ADR-001-outbox-transacional-no-mysql.md): se o tamanho for checado ao montar o snapshot na inserção e o erro for uma exceção, a transação é desfeita e a mudança de status do pedido não acontece por causa do tamanho do payload. Onde o limite é checado e o destino do evento que ultrapassa não foram definidos. As opções que mandam o evento para a DLQ sem envio deixam esse evento fora da garantia at-least-once ([ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md), CONS-12). O conflito também está registrado no [ADR-001](ADR-001-outbox-transacional-no-mysql.md) (CONS-11). Ver questão 2.

#### Filtro e configuração

- **ADR-007-CONS-05**: **Derivada:** a inserção passa a ler a configuração dos webhooks do cliente dentro da transação do `changeStatus`, e uma mudança na lista de status de um webhook só vale para eventos gravados depois dela.
- **ADR-007-CONS-10**: **Derivada:** a configuração do webhook tem um estado ativo, mas não foi definido se um webhook já desativado conta para o filtro na inserção. Também não foi definido o que acontece com as linhas de um webhook desativado ou removido depois da gravação, que podem estar no estado da outbox pendente ou à espera do próximo envio do retry ([ADR-003](ADR-003-retry-com-backoff-e-dlq.md), questão 3): entregar ou descartar. Descartar deixa o evento fora da garantia at-least-once ([ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md), CONS-12). Ver questão 3.
- **ADR-007-CONS-11**: **Derivada:** o filtro consulta a configuração pelo cliente a cada mudança de status, o que pede um índice pelo cliente na tabela de configuração (FDD, seção 4). O snapshot ocupa espaço em cada linha da outbox e é copiado para a DLQ quando os envios se esgotam ([ADR-003](ADR-003-retry-com-backoff-e-dlq.md)).

#### Participação na decisão

- **ADR-007-CONS-04**: O snapshot foi decidido depois da saída de Marcos e Sofia e fora do resumo que todos confirmaram. A decisão vale, fechada por Bruno com Larissa e Diego, mas não passou por produto nem por segurança.

### Trade-off

- **ADR-007-CONS-08**: Troca payload sempre atual e filtro flexível por fidelidade ao momento da mudança e por uma outbox menor. Ganha um evento que reflete o estado de quando o status do pedido mudou e nenhuma linha sem destinatário. Paga com uma leitura de configuração a mais dentro da transação do `changeStatus`, um filtro que não se aplica a eventos já gravados e um payload que pode chegar defasado.

## Referências

- Decisão (ADR-007):
  - `[09:33] Marcos`: filtro como lista de status que o webhook quer ouvir, aplicado na inserção.
  - `[09:34] Bruno`: filtro na inserção; sem webhook interessado, não insere.
  - `[09:43] Diego`: payload com dados básicos do pedido, sem itens; detalhes em `GET /orders/:id`.
  - `[09:52] Larissa`: prefere o payload montado na inserção, para refletir o estado da mudança.
  - `[09:52] Diego`: concorda com o snapshot.
  - `[09:52] Bruno`: fecha "snapshot. Decidido".
- Contexto e fatores: `[09:33] Bruno` (cada endpoint escolhe os eventos), `[09:44] Bruno` (payload enxuto), `[09:21] Bruno` e `[09:21] Sofia` (configuração com estado ativo).
- ADR-007-ALT-01: `[09:51] Bruno` (levanta guardar só o id), `[09:52] Larissa` (descarte).
- ADR-007-ALT-02: `[09:34] Diego` (pergunta inserção ou envio), `[09:34] Bruno` (descarte).
- ADR-007-ALT-03: `[09:43] Diego` (descarte), `[09:44] Bruno` (apoia o payload enxuto).
- Consequências: CONS-01 `[09:52] Larissa`; CONS-02 `[09:34] Bruno`; CONS-03 `[09:52] Bruno`; CONS-04 `[09:48] Larissa` (resumo confirmado antes do snapshot) e `[09:52] Bruno`; CONS-05 `[09:34] Bruno` e `[09:40] Bruno`; CONS-06 `[09:52] Larissa` e `[09:43] Diego`; CONS-07 `[09:43] Diego`; CONS-09 `[09:24] Larissa` (64 KB, erro caso ultrapasse) e `[09:40] Bruno` (rollback se a inserção falhar); CONS-10 `[09:21] Sofia` (estado ativo) e `[09:33] Bruno` (DELETE para remover); CONS-11 `[09:34] Bruno` (filtro pelos webhooks do cliente) e `[09:18] Diego` (DLQ com o payload); CONS-08 `[09:52] Larissa`.
- Código:
  - `prisma/schema.prisma`: `enum OrderStatus` e `model Order`.
  - `src/modules/orders/order.status.ts`: `transitions`.
  - `src/modules/orders/order.routes.ts`: `buildOrderRouter`, rota `GET /:id`.
- ADRs relacionados: [ADR-001](ADR-001-outbox-transacional-no-mysql.md) (inserção na transação e rollback), [ADR-003](ADR-003-retry-com-backoff-e-dlq.md) (retry e DLQ), [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md) (granularidade da linha).
- Documentos: [FDD](../FDD.md), seções 4, 5.1 e 6.2; [PRD](../PRD.md).

## Questões em aberto

1. **Quais campos formam os dados básicos do pedido no payload?** Opções:
   - só os campos citados na reunião: identificador do evento, tipo do evento, timestamp, identificador e número do pedido, status do pedido de origem e de destino, cliente e valor total;
   - esses campos mais outros campos simples de `Order` (`prisma/schema.prisma`), a escolher.

   Afeta detalhes, não o que foi decidido. A definição do campo timestamp também responde à opção (a) da questão 1 do [ADR-002](ADR-002-worker-separado-com-polling.md). Responde: produto (Marcos) com arquitetura (Larissa). Origem: CONS-07.
2. **Onde o limite de 64 KB é checado e o que acontece com o evento que ultrapassa?** Opções:
   - na inserção, lançando erro: a mudança de status do pedido é desfeita (CONS-09);
   - na inserção, gravando o evento direto na DLQ com o motivo, sem desfazer a mudança. Exige que a DLQ aceite eventos que nunca foram enviados ([ADR-003](ADR-003-retry-com-backoff-e-dlq.md));
   - no envio, marcando o evento como falha permanente e mandando para a DLQ. Exige um caminho para a DLQ sem esgotar os envios, que o [ADR-003](ADR-003-retry-com-backoff-e-dlq.md) não prevê.

   As duas últimas opções deixam o evento sem envio, fora da garantia do [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md). Afeta detalhes, não o que foi decidido, mas a primeira opção muda o comportamento do [ADR-001](ADR-001-outbox-transacional-no-mysql.md). Responder junto com a questão 2 do ADR-003. Responde: arquitetura (Larissa) com segurança (Sofia). Origem: CONS-09.
3. **Como o estado ativo do webhook afeta o filtro e as linhas já gravadas?** Duas partes: um webhook desativado conta para o filtro na inserção (sim ou não)? E as linhas de um webhook desativado ou removido depois da gravação são entregues? Opções para a segunda parte:
   - entregar mesmo assim. Para webhook removido, exige manter URL e secret depois da remoção (remoção lógica) ou copiá-las para a linha da outbox (FDD, seção 4), porque a linha guarda só o payload;
   - descartar no envio. Exige que o worker consulte o estado ativo do webhook antes de cada envio, e o evento descartado fica fora da garantia do [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md).

   Afeta detalhes, não o que foi decidido. Responder junto com a questão 2 do [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md), porque a granularidade da linha define quais linhas são afetadas. Responde: produto (Marcos) com arquitetura (Larissa). Origem: CONS-10.
