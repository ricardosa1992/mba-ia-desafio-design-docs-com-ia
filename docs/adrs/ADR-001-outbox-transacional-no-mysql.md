# ADR-001: Outbox transacional no MySQL

## Status

**Aceito**, decidido na reunião técnica de quinta-feira (redigido em 2026-10-09, revisado em 2026-10-18).

- **Decisores**: Diego (Engenheiro Sênior, Plataforma), Larissa (Tech Lead), Bruno (Engenheiro Pleno, Pedidos).
- **Consultados**: nenhum participante além dos decisores opinou sobre esta decisão.

## Contexto

O evento de mudança de status de um pedido precisa sair para um endpoint externo do cliente sem comprometer a própria mudança de status. O pedido de negócio está no [PRD](../PRD.md).

Hoje `OrderService.changeStatus` (`src/modules/orders/order.service.ts`) roda inteiro dentro de uma transação do banco: lê o pedido, valida a transição, atualiza o pedido e grava o histórico de status. Essa transação já é considerada pesada. Ela também mexe no estoque, mas só em duas transições: débito de PENDING para PAID e reposição de PAID ou PROCESSING para CANCELLED (`src/modules/orders/order.status.ts`).

O projeto não tem fila nem barramento de eventos: o `package.json` não declara dependência desse tipo e o `docker-compose.yml` sobe só o MySQL 8.0. O time é pequeno e não quer infraestrutura nova.

Neste ADR, "status do pedido" é o status de domínio (`OrderStatus` em `prisma/schema.prisma`). "Estado da outbox" é o estado técnico de cada linha da outbox, com os nomes dados na reunião: pendente, processando, falhou e entregue.

## Fatores de decisão

- Status do pedido e evento não podem divergir: numa mudança feita pelo `changeStatus` com pelo menos um webhook interessado, o evento existe se, e somente se, a mudança de status foi confirmada (ressalvas no CONS-09 e no [ADR-007](ADR-007-payload-snapshot-e-filtro-na-insercao.md)).
- Um cliente lento ou fora do ar não pode atrasar nem desfazer a mudança de status de nenhum pedido.
- Nenhuma infraestrutura nova para um time pequeno.

## Decisão

- O evento de mudança de status do pedido é registrado em uma tabela de outbox (`webhook_outbox`) no MySQL que o projeto já usa.
- A linha do evento é gravada dentro da mesma transação da mudança de status. Se a gravação falhar, a transação inteira é desfeita e o status do pedido não muda.
- A garantia vale para as mudanças feitas pelo fluxo de mudança de status. Uma mudança sem nenhum webhook interessado não gera linha, por causa do filtro na inserção do [ADR-007](ADR-007-payload-snapshot-e-filtro-na-insercao.md).
- A chamada HTTP ao cliente fica fora da transação, a cargo de um worker separado que lê a tabela ([ADR-002](ADR-002-worker-separado-com-polling.md)).

O ponto exato da chamada dentro do `changeStatus` e o contrato da função que grava o evento estão no FDD, seções 5.1 Criação do evento na outbox e 6.3 Contrato interno publishWebhookEvent.

## Alternativas Consideradas

### ADR-001-ALT-01: Disparo síncrono no changeStatus

- **Descrição**: chamar o endpoint do cliente dentro do service de pedidos, no momento em que o status do pedido muda.
- **Prós**: **Derivada:** nenhuma tabela nova nem processo extra, e nenhum atraso entre a mudança e o envio.
- **Contras**: um cliente lento travaria a mudança de status de outros pedidos. Com o cliente fora do ar, restaria desfazer a mudança de status, o que não faz sentido.
- **Motivo do descarte**: acopla a mudança de status à disponibilidade de um sistema externo.

### ADR-001-ALT-02: Redis Streams como fila

- **Descrição**: publicar os eventos em Redis Streams ou em uma fila similar.
- **Prós**: **Derivada:** fila dedicada, fora do banco transacional.
- **Contras**: exigiria subir e operar um Redis Cluster, infraestrutura nova para um time pequeno.
- **Motivo do descarte**: a outbox no MySQL existente resolve sem infraestrutura nova.

## Consequências

### Positivas

- **ADR-001-CONS-01**: Status do pedido e evento ficam consistentes nas mudanças feitas pelo `changeStatus` com pelo menos um webhook interessado: se a transação foi confirmada, o evento foi registrado; se foi desfeita, o evento some junto.
- **ADR-001-CONS-02**: Nenhuma infraestrutura nova: a fila é uma tabela no MySQL que o projeto já usa.
- **ADR-001-CONS-03**: A transação de mudança de status não faz chamada HTTP, então um cliente lento deixa de travar a mudança de status de outros pedidos.

### Negativas

#### Custo na transação e no banco

- **ADR-001-CONS-04**: A transação, já pesada, ganha mais uma escrita e uma nova causa de falha: um erro ao gravar na outbox desfaz a mudança de status do pedido.
- **ADR-001-CONS-05**: **Hipótese:** o rollback depende de o Prisma desfazer tudo o que foi gravado na transação interativa quando o código dentro dela lança erro. A documentação atual do Prisma afirma esse comportamento, mas não é a da versão 5.22.0 usada pelo projeto (`package.json`). Falta confirmar nessa versão, o que o teste da seção Confirmação faz.
- **ADR-001-CONS-06**: A tabela cresce com as linhas no estado da outbox entregue. O arquivamento ficou fora do escopo desta feature, sem prazo. Os índices no estado e na data de criação da outbox, previstos para a leitura do worker, também crescem com a tabela.
- **ADR-001-CONS-10**: **Derivada:** a outbox não cria um ponto de falha novo, porque usa o mesmo banco da mudança de status: se o MySQL cair, a mudança de status já falha. Ela cria, porém, um custo novo nesse banco compartilhado: uma escrita por evento na transação de status, a leitura periódica do worker ([ADR-002](ADR-002-worker-separado-com-polling.md)) e a leitura da configuração dos webhooks para o filtro ([ADR-007](ADR-007-payload-snapshot-e-filtro-na-insercao.md)). O volume esperado não foi estimado. Ver questão 3.
- **ADR-001-CONS-11**: **Derivada:** o limite de 64 KB do payload, com erro caso ultrapasse, é uma nova causa possível de falha na gravação: se o tamanho for checado ao montar o snapshot na inserção e o erro for uma exceção, a mudança de status do pedido é desfeita por causa do tamanho do payload. O conflito e as opções estão no [ADR-007](ADR-007-payload-snapshot-e-filtro-na-insercao.md) (CONS-09, questão 2).

#### Cobertura da garantia

- **ADR-001-CONS-07**: **Derivada:** a criação do pedido (`OrderService.create`, em `src/modules/orders/order.service.ts`) grava o pedido no status PENDING sem passar pelo `changeStatus`, único ponto de integração definido. Sem decisão contrária, o cliente não recebe evento de criação. Ver questão 1.
- **ADR-001-CONS-09**: **Derivada:** nada no código obriga toda mudança de status a passar pelo `changeStatus`. O seed (`prisma/seed.ts`) já grava pedidos e histórico direto no banco, e um script de manutenção ou um ajuste manual por SQL também mudaria o status sem gerar evento. A garantia do CONS-01 vale só para o fluxo da aplicação. Mitigações possíveis, sem decisão: documentar a regra para quem opera o banco ou criar uma checagem periódica. Ver questão 2.

### Trade-off

- **ADR-001-CONS-08**: Troca uma transação de status mais leve e o envio imediato por consistência sem infraestrutura nova. Ganha status do pedido e evento que são confirmados ou desfeitos juntos no fluxo de mudança de status. Paga com escrita extra e nova causa de falha na mudança de status, carga adicional no banco compartilhado e uma tabela que cresce até existir arquivamento.

## Confirmação

- **Proposta (a validar):** teste de integração no padrão de `tests/orders.test.ts` que confirma a linha na outbox depois de `PATCH /api/v1/orders/:id/status` e, quando a gravação falha, a ausência da linha com o status antigo do pedido mantido. O teste também confirma a hipótese do ADR-001-CONS-05.

## Referências

- Decisão (ADR-001):
  - `[09:06] Diego`: propõe a outbox na mesma transação que atualiza pedido e histórico, com consistência entre commit e rollback, e o worker separado.
  - `[09:08] Larissa`: fecha "outbox em MySQL".
  - `[09:08] Diego`: estados da outbox pendente, processando, falhou e entregue; índices em status e data de criação.
  - `[09:40] Bruno`: inserção na `webhook_outbox` dentro da transação do `changeStatus`; rollback se a inserção falhar; "não pode ter caso de status mudar e evento não sair".
  - `[09:41] Diego`: fora da transação "perde a garantia toda".
  - `[09:41] Bruno`: função que recebe o `tx` da transação, detalhada no FDD.
  - `[09:34] Bruno`: filtro na inserção, base da ressalva sobre mudança sem webhook interessado.
- Contexto e fatores: `[09:04] Bruno` (transação pesada, cliente lento trava outros pedidos), `[09:07] Diego` (time pequeno, sem infraestrutura nova).
- ADR-001-ALT-01: `[09:03] Larissa` (levanta síncrono ou outbox), `[09:04] Bruno` (descarte), `[09:06] Diego` ("síncrono está fora de questão").
- ADR-001-ALT-02: `[09:07] Larissa` (levanta Redis Streams), `[09:07] Diego` (descarte por overengineering).
- Consequências: CONS-01 `[09:06] Diego`; CONS-02 `[09:07] Diego`; CONS-03 `[09:04] Bruno`; CONS-04 `[09:04] Bruno` e `[09:40] Bruno`; CONS-05 `src/modules/orders/order.service.ts` (transação interativa), `package.json` (Prisma 5.22.0) e a documentação externa abaixo; CONS-06 `[09:07] Bruno` (acúmulo na tabela) e `[09:08] Diego` (índices; arquivamento fora do escopo); CONS-07 `[09:40] Bruno` (único ponto de integração citado) e `src/modules/orders/order.service.ts`; CONS-08 `[09:06] Diego`; CONS-09 `prisma/seed.ts` e `[09:40] Bruno`; CONS-10 `[09:40] Bruno` (escrita na transação de status), com a leitura do worker no ADR-002 e a do filtro no ADR-007; CONS-11 `[09:24] Larissa` (64 KB, erro caso ultrapasse) e `[09:40] Bruno` (rollback se a inserção falhar), com o ADR-007-CONS-09.
- Código:
  - `src/modules/orders/order.service.ts`: `OrderService.changeStatus` (transação) e `OrderService.create`.
  - `src/modules/orders/order.status.ts`: `shouldDebitStock` e `shouldReplenishStock`.
  - `prisma/seed.ts`: pedidos e histórico gravados direto no banco.
  - `prisma/schema.prisma`: datasource `mysql`, `enum OrderStatus` e `model OrderStatusHistory` (campo `fromStatus` opcional).
  - `package.json`: `@prisma/client` 5.22.0.
  - `docker-compose.yml`: só o serviço `mysql`.
  - `tests/orders.test.ts`: padrão de teste de integração.
- Documentação externa: Prisma, "Transactions and batch queries" (https://www.prisma.io/docs/orm/prisma-client/queries/transactions): a transação interativa é desfeita quando o callback lança erro (versão atual da documentação).
- ADRs relacionados: [ADR-002](ADR-002-worker-separado-com-polling.md) (worker), [ADR-003](ADR-003-retry-com-backoff-e-dlq.md) (retry e DLQ), [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md) (padrões reaproveitados), [ADR-007](ADR-007-payload-snapshot-e-filtro-na-insercao.md) (conteúdo e filtro da linha).
- Documentos: [FDD](../FDD.md), seções 5.1, 6.2, 6.3, 11 e 13; [PRD](../PRD.md).

## Questões em aberto

1. **A criação do pedido (status PENDING) deve gerar evento para o cliente?** Opções:
   - sim: exige um segundo ponto de integração em `OrderService.create`, com o mesmo filtro do [ADR-007](ADR-007-payload-snapshot-e-filtro-na-insercao.md), e um status de origem vazio no payload, como já acontece no histórico (`fromStatus` opcional em `prisma/schema.prisma`; contrato no FDD, seção 6.2);
   - não: o cliente só recebe mudanças de status feitas pelo `changeStatus`.

   Afeta detalhes, não o que foi decidido. Responder junto com a questão 2. Responde: produto (Marcos). Origem: CONS-07.
2. **Mudanças de status do pedido feitas fora do `changeStatus` (seed, scripts, SQL manual) devem gerar evento?** Opções:
   - aceitar que não gerem e documentar a regra para quem opera o banco;
   - checagem periódica que compara o histórico de status com a outbox. Exige distinguir uma mudança filtrada (sem webhook interessado, [ADR-007](ADR-007-payload-snapshot-e-filtro-na-insercao.md)) de uma mudança perdida, reaplicando o filtro na checagem, e excluir os dados do seed. Não cobre mudanças feitas sem gravar histórico, como um SQL manual que só altera o status do pedido;
   - trigger no banco que grava na outbox. Exige montar o snapshot do payload em SQL, fora da aplicação, o que conflita com a renderização feita pela aplicação no [ADR-007](ADR-007-payload-snapshot-e-filtro-na-insercao.md). Como a trigger também dispararia nas mudanças feitas pelo `changeStatus`, exige ainda retirar a gravação feita pela aplicação, o que mudaria o núcleo deste ADR.

   Afeta detalhes, não o que foi decidido, salvo a terceira opção. Responder junto com a questão 1. Responde: arquitetura (Larissa). Origem: CONS-09.
3. **É preciso estimar o volume de mudanças de status antes do go-live para dimensionar a carga da outbox no MySQL?** Sim ou não. Afeta operação, não o que foi decidido. A resposta também informa as questões 2 e 4 do [ADR-002](ADR-002-worker-separado-com-polling.md) (ciclo do worker e premissa de worker único). Responde: arquitetura (Larissa) com plataforma (Diego). Origem: CONS-10.
