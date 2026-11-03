# ADR-002: Worker separado com polling de 2 segundos

## Status

**Aceito**, decidido na reunião técnica de quinta-feira (redigido em 2026-10-09, revisado em 2026-10-18).

> Aceito quanto ao worker em processo separado da API, ao polling de 2 s sobre o MySQL existente, ao worker único nesta fase e à ordem por pedido que ele dá. Manter essa ordem nas reentregas e no envio do ciclo e atender a meta de 10 s dependem das questões 1, 2 e 3.

- **Decisores**: Diego (Engenheiro Sênior, Plataforma), Larissa (Tech Lead).
- **Consultados**: Bruno (Engenheiro Pleno, Pedidos), Marcos (Product Manager).

## Contexto

Com a outbox do [ADR-001](ADR-001-outbox-transacional-no-mysql.md), algum componente precisa ler as linhas da outbox que aguardam envio e chamar o endpoint de cada cliente. Hoje não há processo de background no projeto: a única entry-point é `src/server.ts`, que sobe a API HTTP.

O MySQL não tem um mecanismo nativo para avisar um processo externo de que uma linha foi inserida: uma trigger só executa SQL.

Neste ADR, "status do pedido" é o status de domínio do pedido (`OrderStatus` em `prisma/schema.prisma`: PENDING, PAID, PROCESSING, SHIPPED, DELIVERED, CANCELLED). "Estado da outbox" é o estado técnico de cada linha da outbox, com os nomes dados na reunião: pendente, processando, falhou e entregue. O status do pedido PROCESSING e o estado da outbox processando não têm relação entre si.

## Fatores de decisão

- Meta de produto: notificar o cliente em menos de 10 s. A meta não diz se vale para o caso típico, para todo evento ou para um percentil; ver CONS-12.
- Um reinício da API não pode parar a entrega de eventos.
- Mesmo banco e mesma stack da API, sem infraestrutura nova.

## Decisão

- O worker roda como processo próprio, separado da API, no mesmo banco e na mesma stack.
- O worker lê a outbox por polling: a cada 2 segundos busca as linhas mais antigas no estado da outbox pendente (ou à espera do próximo envio, conforme a questão 3 do [ADR-003](ADR-003-retry-com-backoff-e-dlq.md)), envia e grava o estado resultante da linha. Cada execução dessa busca é um ciclo.
- Nesta fase roda um único worker. Isso é uma premissa do desenho, não uma garantia do sistema (CONS-10).
- Com um único worker, as linhas são lidas na ordem em que entraram na outbox e a ordem de entrega vale por pedido, como limitação conhecida: não há ordem global entre pedidos (CONS-05), e com mais de um worker rodando ao mesmo tempo a ordem por pedido se perde (CONS-06). **Derivada:** mesmo com worker único, retry (CONS-07), reprocessamento da DLQ (CONS-16), linha presa em processando devolvida a pendente (CONS-14) e envio de várias linhas ao mesmo tempo dentro do ciclo (CONS-11) podem inverter a ordem de um pedido.

Entry-point, script de execução, instância do cliente de banco e loop de leitura estão no FDD, seções 5.2 Processamento pelo worker e 10. Dependências e compatibilidade.

## Alternativas Consideradas

### ADR-002-ALT-01: Acionamento reativo a partir do banco

- **Descrição**: usar uma trigger do MySQL para avisar o worker a cada inserção, no estilo do NOTIFY/LISTEN do Postgres.
- **Prós**: worker mais reativo, sem o atraso do intervalo de polling.
- **Contras**: o MySQL não tem listener nativo e a trigger não notifica processo externo. Avisar o worker exigiria um improviso, como escrever em arquivo ou chamar um endpoint a partir do banco.
- **Motivo do descarte**: o polling de 2 s já atende a meta de 10 s no caminho feliz, sem improviso.

### ADR-002-ALT-02: Worker no mesmo processo da API

- **Descrição**: rodar o loop de leitura dentro do processo da API.
- **Prós**: **Derivada:** um processo só para implantar e supervisionar.
- **Contras**: se a API reinicia, o worker para junto.
- **Motivo do descarte**: a entrega de eventos não pode parar a cada reinício da API.

## Consequências

### Positivas

- **ADR-002-CONS-01**: O polling de 2 s atende a meta de 10 s quando a primeira tentativa dá certo e o cliente responde rápido. O intervalo foi aceito pelo produto (`[09:10] Marcos`).
- **ADR-002-CONS-02**: Um reinício da API não para a entrega de eventos.
- **ADR-002-CONS-03**: Nenhuma infraestrutura nova nem mecanismo improvisado no banco.

### Negativas

#### Ordem de entrega

- **ADR-002-CONS-05**: Não há ordem global entre pedidos. Os clientes não pediram ordem global, só saber se cada pedido mudou (`[09:14] Marcos`).
- **ADR-002-CONS-06**: Com vários workers rodando ao mesmo tempo, a ordem por pedido se perde. Particionar por pedido ou usar lock pessimista ficou para o futuro, sem prazo.
- **ADR-002-CONS-07**: **Derivada:** o retry do [ADR-003](ADR-003-retry-com-backoff-e-dlq.md) reordena eventos do mesmo pedido mesmo com um único worker. Cenário: o evento da mudança para o status do pedido PAID falha e a linha espera 1 min pela próxima tentativa; segundos depois o pedido vai para o status PROCESSING, a nova linha entra no estado da outbox pendente, é lida no ciclo seguinte e entregue; o cliente recebe PROCESSING antes de PAID. Isso só não acontece se o worker segurar as linhas posteriores de um pedido enquanto houver uma em retry, o que não foi decidido. Ver questão 1.
- **ADR-002-CONS-16**: **Derivada:** o reprocessamento da DLQ também entrega fora de ordem: ele devolve à outbox um evento antigo de um pedido que pode já ter eventos mais novos entregues ([ADR-003](ADR-003-retry-com-backoff-e-dlq.md), CONS-14). Ver a questão 5 do ADR-003.

#### Latência e meta de 10 s

- **ADR-002-CONS-04**: O polling introduz um atraso de até 2 s no pior caso, aceito na reunião (`[09:10] Marcos`). Esse valor é o atraso causado pelo intervalo, não a latência total da entrega.
- **ADR-002-CONS-11**: **Derivada:** não foi definido se o worker envia as linhas de um ciclo uma de cada vez ou várias ao mesmo tempo (processamento concorrente dentro do ciclo). Uma de cada vez, um endpoint que não responde segura o ciclo por até 10 s por linha (timeout do [ADR-003](ADR-003-retry-com-backoff-e-dlq.md)), e as linhas de outros clientes que estão atrás passam da meta de 10 s. Com processamento concorrente, a ordem de leitura deixa de ser a ordem de entrega, mesmo com um único worker. Ver questão 2.
- **ADR-002-CONS-12**: **Derivada:** a meta de 10 s só é atendida no caminho feliz. Um evento cuja primeira tentativa falha volta só depois do primeiro intervalo de retry (1 min, [ADR-003](ADR-003-retry-com-backoff-e-dlq.md)) e passa da meta por desenho. O mesmo vale para o envio uma linha de cada vez descrito no CONS-11. Ver questão 3.

#### Operação do processo novo

- **ADR-002-CONS-08**: **Derivada:** a operação ganha um segundo processo para implantar e supervisionar. O `docker-compose.yml` atual só sobe o MySQL, sem a API nem outro serviço, e nenhum gerenciador de processos ou orquestrador foi definido. Se o worker cair e ninguém o reiniciar, nenhum evento é entregue. Ver questão 5.
- **ADR-002-CONS-10**: **Derivada:** nada garante a premissa de worker único. Ela quebra em deploy com sobreposição (versão nova sobe antes de a antiga parar), em reinício com o processo antigo ainda vivo e em execução manual de um segundo worker. Com dois workers, as mesmas linhas no estado da outbox pendente podem ser lidas duas vezes: o cliente recebe duplicatas (toleradas pelo [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md) se ele deduplicar) e a ordem por pedido se perde. Mitigações possíveis, sem decisão: supervisor que garanta uma única instância, trava no banco na leitura das linhas pendentes ou o lock pessimista já citado para o futuro. Ver questão 4.
- **ADR-002-CONS-13**: **Derivada:** um worker parado ou travado não é percebido. Sem servidor HTTP, ele não tem health check (só a API tem `GET /health`, em `src/app.ts`), o projeto não tem biblioteca de métricas e a API continua aceitando mudanças de status do pedido. As linhas se acumulam no estado da outbox pendente em silêncio. Mitigações possíveis, sem decisão: log periódico de "vivo", alerta pela idade da linha pendente mais antiga ou endpoint de saúde próprio do worker. Ver questão 6.
- **ADR-002-CONS-14**: **Derivada:** o desligamento atual de `src/server.ts` desconecta do banco e encerra o processo sem esperar o trabalho em andamento. Se o worker seguir esse modelo, uma linha que estava no estado da outbox processando fica presa nesse estado, e uma entrega que o cliente já recebeu, mas que ainda não foi gravada como entregue, é reenviada depois (duplicata tolerada pelo [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md)). Não foi definido como uma linha presa em processando volta a pendente nem se o desligamento espera o ciclo em andamento. Ver questões 7 e 8.
- **ADR-002-CONS-15**: **Derivada:** o worker impõe custo contínuo ao MySQL compartilhado: uma consulta a cada 2 s mesmo sem eventos, apoiada nos índices previstos em status e data de criação da outbox ([ADR-001](ADR-001-outbox-transacional-no-mysql.md)), e um segundo pool de conexões, somado ao da API, porque o worker tem a própria instância de `PrismaClient`. Nem `src/config/database.ts` nem a `DATABASE_URL` do `.env.example` limitam o pool (não há `connection_limit`). **Hipótese:** a soma dos dois pools cabe no limite de conexões do MySQL de produção, cuja configuração não está no repositório; confirmar contando as conexões abertas no MySQL com a API e o worker rodando, junto com a questão 5. O tamanho do lote por ciclo não foi definido e fica para o FDD, seção 5.2.

### Trade-off

- **ADR-002-CONS-09**: Troca reatividade imediata e ordem em todos os casos por simplicidade sobre o banco existente. Ganha um worker sem infraestrutura nova que sobrevive ao reinício da API. Paga com até 2 s de atraso no caminho feliz, ordem por pedido que vale só com um único worker, envio de uma linha de cada vez e nenhuma reentrega (retry, reprocessamento ou linha presa), e um processo novo que precisa ser implantado, supervisionado e monitorado.

## Referências

- Decisão (ADR-002):
  - `[09:08] Diego`: estados da outbox pendente, processando, falhou e entregue; índice em status e data de criação.
  - `[09:09] Diego`: polling em loop a cada 2 s, buscando os pendentes mais antigos.
  - `[09:10] Larissa`: registra "Worker em polling, 2s" como decisão.
  - `[09:11] Diego`: processo separado, não dentro da instância da API; mesmo banco e mesma stack.
  - `[09:11] Larissa`: entry-point própria nos moldes de `src/server.ts` e script de execução (detalhados no FDD).
  - `[09:12] Diego`: worker único processa em ordem de criação; vários workers perdem a garantia.
  - `[09:13] Larissa`: ordem só por pedido e com worker único, como limitação conhecida.
  - `[09:30] Bruno`: worker com instância própria do cliente de banco e mesma `DATABASE_URL` (detalhado no FDD).
- Contexto e fatores: `[09:02] Marcos` (meta de 10 s), `[09:09] Diego` (MySQL sem listener; trigger só executa SQL), `[09:11] Diego` (reinício da API).
- ADR-002-ALT-01: `[09:09] Bruno` (sugere trigger), `[09:09] Diego` (descarte).
- ADR-002-ALT-02: `[09:11] Diego` (descarte).
- Consequências: CONS-01 `[09:09] Diego` e `[09:10] Marcos`; CONS-02 `[09:11] Diego`; CONS-03 `[09:09] Diego`; CONS-04 `[09:10] Larissa` e `[09:10] Marcos`; CONS-05 `[09:13] Larissa` e `[09:14] Marcos`; CONS-06 `[09:12] Diego` e `[09:13] Diego`; CONS-07 `[09:12] Diego` e `[09:17] Larissa` (backoff); CONS-08 `[09:11] Larissa` e `docker-compose.yml`; CONS-10 `[09:12] Diego` (worker único; vários workers perdem a garantia) e `[09:13] Diego` (lock pessimista no futuro); CONS-13 `src/app.ts` e `package.json` (sem biblioteca de métricas); CONS-14 `src/server.ts`; CONS-16 `[09:18] Diego` (reprocessamento recoloca como pendente), com o ADR-003-CONS-14; CONS-11 e CONS-12 `[09:42] Diego` (timeout de 10 s) e `[09:17] Larissa` (primeiro intervalo de 1 min); CONS-15 `[09:08] Diego` (índices; "batch pequeno"), `[09:29] Diego` (pool do Prisma), `[09:30] Bruno` (instância nova por processo), `src/config/database.ts` e `.env.example` (pool sem limite configurado); CONS-09 `[09:10] Larissa`.
- Código e configuração:
  - `src/server.ts`: `bootstrap` e `shutdown`, modelo da entry-point e do desligamento.
  - `src/app.ts`: `GET /health`, só na API.
  - `src/config/database.ts`: `createPrismaClient`, sem configuração de pool.
  - `.env.example`: `DATABASE_URL` sem `connection_limit`.
  - `prisma/schema.prisma`: `enum OrderStatus`.
  - `package.json`: `@prisma/client` e `prisma` 5.22.0; `scripts`, sem script do worker hoje.
  - `docker-compose.yml`: só o serviço `mysql`, imagem `mysql:8.0`.
- Documentação externa (só como apoio da hipótese da questão 4, a confirmar por teste):
  - MySQL 8.0 Reference Manual, "Locking Reads" (https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-reads.html): `SELECT ... FOR UPDATE SKIP LOCKED` dentro de transação.
- ADRs relacionados: [ADR-001](ADR-001-outbox-transacional-no-mysql.md) (outbox), [ADR-003](ADR-003-retry-com-backoff-e-dlq.md) (retry, intervalos e timeout), [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md) (duplicatas), [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md) (padrões reaproveitados), [ADR-007](ADR-007-payload-snapshot-e-filtro-na-insercao.md) (conteúdo do payload).
- Documentos: [FDD](../FDD.md), seções 4, 5.2, 6.2, 8, 9 e 10; [PRD](../PRD.md).

## Questões em aberto

1. **A ordem de entrega por pedido deve ser mantida durante o retry?** Opções:
   - (a) aceitar a reordenação e documentar que o cliente ordena os eventos de um pedido pelo campo `timestamp` do payload. Exige definir esse campo, citado na reunião, como o momento da mudança de status do pedido e com precisão suficiente para separar mudanças próximas (contrato no FDD, seção 6.2). O header `X-Timestamp` não serve, porque é o momento do envio.
   - (b) o worker segura as linhas posteriores de um pedido enquanto houver uma em retry. Exige que a leitura das linhas pendentes consiga agrupar por pedido (modelo da outbox no FDD, seção 4). O atraso das linhas seguradas chega a 876 min, cerca de 14,6 h (1 + 5 + 30 + 120 + 720 min, intervalos do [ADR-003](ADR-003-retry-com-backoff-e-dlq.md)), na leitura da contagem que é hipótese no ADR-003-CONS-05; a outra contagem possível está na questão 1 do ADR-003. Exige também decidir o que acontece com as linhas seguradas quando a linha em retry vai para a DLQ.

   Afeta o núcleo da decisão. Depende da questão 3 e da questão 1 do [ADR-003](ADR-003-retry-com-backoff-e-dlq.md); responder junto com a questão 2, com a questão 5 do [ADR-003](ADR-003-retry-com-backoff-e-dlq.md) (ordem no reprocessamento) e com a questão 1 do [ADR-007](ADR-007-payload-snapshot-e-filtro-na-insercao.md). Responde: produto (Marcos) e arquitetura (Larissa). Origem: CONS-07.
2. **O worker envia as linhas de um ciclo uma de cada vez ou várias ao mesmo tempo?** Opções: uma de cada vez; ou várias ao mesmo tempo, com um limite a definir. Afeta o núcleo da decisão, porque muda a ordem de entrega e o atendimento da meta. Responder junto com as questões 1 e 3 e com a questão 3 do [ADR-001](ADR-001-outbox-transacional-no-mysql.md). Responde: arquitetura (Larissa) com plataforma (Diego). Origem: CONS-11.
3. **A meta de 10 s vale só para eventos entregues na primeira tentativa?** Sim (retry fica fora da meta) ou não (é preciso rever o desenho). Afeta o núcleo da decisão. É a resposta que delimita as opções das questões 1 e 2. Responde: produto (Marcos). Origem: CONS-12.
4. **Como a premissa de worker único é garantida?** Opções:
   - supervisor que mantém uma única instância (depende da questão 5);
   - trava no banco na leitura das linhas pendentes, com `SELECT ... FOR UPDATE SKIP LOCKED` dentro de uma transação. Evita que dois workers leiam a mesma linha, mas não preserva a ordem por pedido com dois workers. **Hipótese:** o MySQL 8.0 do `docker-compose.yml` aceita essa leitura e pula as linhas já travadas por outra transação, e o Prisma 5.22.0 do `package.json`, sem leitura com trava na API do cliente, a executa por SQL bruto (`$queryRaw`) no cliente da transação; confirmar com um teste de integração contra esse banco, com duas transações concorrentes (FDD, seção 5.2);
   - aceitar duplicatas e perda de ordem nos momentos de sobreposição.

   Afeta operação, não o que foi decidido. Responder junto com a questão 5 e com a questão 3 do [ADR-001](ADR-001-outbox-transacional-no-mysql.md). Responde: plataforma (Diego). Origem: CONS-10.
5. **Onde o worker roda e quem o supervisiona?** Opções:
   - serviço no `docker-compose.yml`. Hoje o arquivo só sobe o MySQL e a API também não está nele, então a opção exige definir antes como a própria API é implantada;
   - orquestrador do ambiente de produção. Exige informação sobre esse ambiente, que não está no repositório;
   - gerenciador de processos. Nenhum está nas dependências hoje.

   Afeta operação, não o que foi decidido. Responder junto com a questão 4. Responde: plataforma (Diego). Origem: CONS-08.
6. **Como detectar um worker parado ou atrasado?** Opções:
   - log periódico de "vivo" pelo logger Pino que já existe. Exige alguém ou algo que leia esses logs;
   - alerta pela idade da linha pendente mais antiga. Exige ferramenta de alerta, que o projeto não tem;
   - endpoint de saúde próprio do worker. Exige um servidor HTTP no worker e um supervisor que o consulte (depende da questão 5).

   Afeta operação, não o que foi decidido. Depende da questão 5. Responde: plataforma (Diego). Origem: CONS-13.
7. **Como uma linha presa no estado da outbox processando volta a pendente?** Opções:
   - limite de tempo em processando. Exige guardar o momento em que a linha entrou nesse estado (modelo da outbox no FDD, seção 4);
   - devolução a pendente ao iniciar o worker. Só é segura se a premissa de worker único estiver garantida (depende da questão 4).

   Afeta operação, não o que foi decidido. Depende da questão 4; responder junto com a questão 8. Responde: arquitetura (Larissa). Origem: CONS-14.
8. **O desligamento do worker deve esperar o ciclo em andamento terminar?** Sim ou não. Se sim, a duração do ciclo depende da questão 2 (uma linha de cada vez, até 10 s por linha) e o supervisor da questão 5 precisa dar esse tempo antes de encerrar o processo. Se não, as linhas do ciclo interrompido caem na questão 7. Afeta operação, não o que foi decidido. Depende das questões 2 e 5; responder junto com a questão 7. Responde: arquitetura (Larissa) com plataforma (Diego). Origem: CONS-14.

## Recomendações

- O modelo de concorrência do worker (instância única, envio das linhas de um ciclo uma de cada vez ou ao mesmo tempo, ordem por pedido) é decidido aqui só de forma implícita e concentra as questões 1, 2 e 4. Recomenda-se avaliar uma ADR própria quando essas questões forem respondidas.
