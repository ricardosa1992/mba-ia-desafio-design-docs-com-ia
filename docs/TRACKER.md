# Tracker de Rastreabilidade

Mapa de cada item dos documentos de design até a sua origem na reunião técnica de quinta-feira (`TRANSCRICAO.md`) ou no código existente.
Cada linha corresponde a um ID definido em um documento: `ADR-NNN` (decisão), `ADR-NNN-ALT-NN` (alternativa), `ADR-NNN-CONS-NN` (consequência); `PRD-*`, `RFC-*` e `FDD-*` seguem o mesmo padrão. Sufixo de letra (`-b`) indica a segunda fonte do mesmo item.
Fonte `TRANSCRICAO`: Localização é a fala `[hh:mm] Nome` que sustenta o item. Fonte `CODIGO`: Localização é o arquivo do repositório.
Resumo iniciado por "Proposta:", "Hipótese:" ou "Consequência derivada:" indica item que a reunião não decidiu diretamente; a Localização é a fala ou o arquivo que o motiva.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| ADR-001 | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Decisão | Outbox no MySQL gravada na mesma transação do `changeStatus`; falha na gravação desfaz a mudança de status | TRANSCRICAO | [09:08] Larissa |
| ADR-001-ALT-01 | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Alternativa Descartada | Disparo síncrono da chamada HTTP no `changeStatus` | TRANSCRICAO | [09:04] Bruno |
| ADR-001-ALT-02 | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Alternativa Descartada | Redis Streams como fila | TRANSCRICAO | [09:07] Diego |
| ADR-001-CONS-01 | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Consequência | Status do pedido e evento consistentes nas mudanças do `changeStatus` com webhook interessado | TRANSCRICAO | [09:06] Diego |
| ADR-001-CONS-02 | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Consequência | Nenhuma infraestrutura nova: fila em tabela do MySQL existente | TRANSCRICAO | [09:07] Diego |
| ADR-001-CONS-03 | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Consequência | Transação de status sem chamada HTTP; cliente lento não trava outros pedidos | TRANSCRICAO | [09:04] Bruno |
| ADR-001-CONS-04 | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Consequência | Escrita extra e nova causa de falha na transação do `changeStatus` | TRANSCRICAO | [09:40] Bruno |
| ADR-001-CONS-05 | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Consequência | Hipótese: rollback da transação interativa no Prisma 5.22.0, documentado só na versão atual | CODIGO | src/modules/orders/order.service.ts |
| ADR-001-CONS-06 | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Consequência | Tabela e índices crescem; arquivamento das linhas entregues adiado, sem prazo | TRANSCRICAO | [09:08] Diego |
| ADR-001-CONS-07 | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Consequência | Consequência derivada: criação do pedido não passa pelo `changeStatus` e não gera evento sem decisão contrária | CODIGO | src/modules/orders/order.service.ts |
| ADR-001-CONS-08 | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Trade-off | Troca transação mais leve e envio imediato por consistência sem infraestrutura nova | TRANSCRICAO | [09:06] Diego |
| ADR-001-CONS-09 | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Consequência | Consequência derivada: mudanças de status fora do `changeStatus` (seed, SQL manual) não geram evento | CODIGO | prisma/seed.ts |
| ADR-001-CONS-10 | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Consequência | Consequência derivada: custo novo no MySQL compartilhado; volume de eventos não estimado | TRANSCRICAO | [09:40] Bruno |
| ADR-001-CONS-11 | `docs/adrs/ADR-001-outbox-transacional-no-mysql.md` | Consequência | Consequência derivada: limite de 64 KB checado na inserção desfaria a mudança de status (ver ADR-007) | TRANSCRICAO | [09:24] Larissa |
| ADR-002 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Decisão | Worker em processo separado, polling de 2 s, worker único como premissa; ordem por pedido não garantida com reentregas, envio concorrente ou vários workers | TRANSCRICAO | [09:10] Larissa |
| ADR-002-ALT-01 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Alternativa Descartada | Acionamento reativo pelo banco (trigger do MySQL) | TRANSCRICAO | [09:09] Diego |
| ADR-002-ALT-02 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Alternativa Descartada | Worker no mesmo processo da API | TRANSCRICAO | [09:11] Diego |
| ADR-002-CONS-01 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Polling de 2 s atende a meta de 10 s no caminho feliz, aceito pelo produto | TRANSCRICAO | [09:10] Marcos |
| ADR-002-CONS-02 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Reinício da API não para a entrega de eventos | TRANSCRICAO | [09:11] Diego |
| ADR-002-CONS-03 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Sem infraestrutura nova nem mecanismo improvisado no banco | TRANSCRICAO | [09:09] Diego |
| ADR-002-CONS-04 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Atraso de até 2 s no pior caso causado pelo polling | TRANSCRICAO | [09:10] Larissa |
| ADR-002-CONS-05 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Sem ordem global entre pedidos; clientes não pediram | TRANSCRICAO | [09:13] Larissa |
| ADR-002-CONS-06 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Vários workers ao mesmo tempo perdem a ordem por pedido; particionamento adiado | TRANSCRICAO | [09:13] Diego |
| ADR-002-CONS-07 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Consequência derivada: retry reordena eventos do mesmo pedido mesmo com worker único | TRANSCRICAO | [09:12] Diego |
| ADR-002-CONS-16 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Consequência derivada: reprocessamento da DLQ também entrega fora de ordem (ver ADR-003) | TRANSCRICAO | [09:18] Diego |
| ADR-002-CONS-08 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Consequência derivada: segundo processo sem implantação nem supervisão definidas | TRANSCRICAO | [09:11] Larissa |
| ADR-002-CONS-09 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Trade-off | Troca reatividade imediata e ordem garantida por simplicidade no banco existente | TRANSCRICAO | [09:10] Larissa |
| ADR-002-CONS-10 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Consequência derivada: premissa de worker único não garantida em deploy, reinício ou execução manual | TRANSCRICAO | [09:12] Diego |
| ADR-002-CONS-11 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Consequência derivada: envio das linhas do ciclo uma a uma ou ao mesmo tempo não definido | TRANSCRICAO | [09:42] Diego |
| ADR-002-CONS-12 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Consequência derivada: meta de 10 s só atendida no caminho feliz | TRANSCRICAO | [09:17] Larissa |
| ADR-002-CONS-13 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Consequência derivada: worker parado não é percebido; sem health check nem métricas | CODIGO | src/app.ts |
| ADR-002-CONS-14 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Consequência derivada: desligamento no modelo atual deixa linha presa em processando | CODIGO | src/server.ts |
| ADR-002-CONS-15 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Consequência derivada: custo contínuo no MySQL e pool de conexões próprio por instância | CODIGO | src/config/database.ts |
| ADR-003 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Decisão | Retry com backoff 1m/5m/30m/2h/12h, teto com contagem em aberto, DLQ separada, reprocessamento ADMIN auditado | TRANSCRICAO | [09:17] Larissa |
| ADR-003-ALT-01 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Alternativa Descartada | Retry indefinido | TRANSCRICAO | [09:15] Diego |
| ADR-003-ALT-02 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Alternativa Descartada | 3 tentativas | TRANSCRICAO | [09:16] Diego |
| ADR-003-ALT-03 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Alternativa Descartada | Marcar "falhou" na própria outbox no lugar da DLQ | TRANSCRICAO | [09:18] Diego |
| ADR-003-CONS-01 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência | Janela cobre indisponibilidades de horas e foi aceita pelo produto | TRANSCRICAO | [09:17] Marcos |
| ADR-003-CONS-02 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência | Nenhum evento fica sendo reenviado para sempre | TRANSCRICAO | [09:15] Diego |
| ADR-003-CONS-03 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência | Outbox com leitura limpa; DLQ como evidência | TRANSCRICAO | [09:18] Diego |
| ADR-003-CONS-04 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência | Reprocessamento devolve o evento à entrega, com registro de quem o fez | TRANSCRICAO | [09:18] Diego |
| ADR-003-CONS-05 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência | Consequência derivada: 5 tentativas e cinco intervalos só fecham com 6 envios (876 min); com 5 envios, 156 min | TRANSCRICAO | [09:17] Larissa |
| ADR-003-CONS-06 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência | Cliente fora além da janela depende de reprocessamento por ADMIN | TRANSCRICAO | [09:17] Marcos |
| ADR-003-CONS-07 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência | Consequência derivada: significado do estado da outbox falhou não definido | TRANSCRICAO | [09:08] Diego |
| ADR-003-CONS-08 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência | Controle por papel ADMIN fraco: registro público aceita ADMIN | CODIGO | src/modules/auth/auth.routes.ts |
| ADR-003-CONS-09 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Trade-off | Troca entrega sem limite por janela finita com DLQ e reprocessamento manual | TRANSCRICAO | [09:18] Diego |
| ADR-003-CONS-10 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência | Consequência derivada: retry reordena eventos do mesmo pedido (cenário no ADR-002) | TRANSCRICAO | [09:17] Larissa |
| ADR-003-CONS-11 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência | Consequência derivada: só o timeout de 10 s foi definido como falha; respostas 4xx sem regra | TRANSCRICAO | [09:42] Diego |
| ADR-003-CONS-12 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência | Consequência derivada: nada avisa evento novo na DLQ; e-mail ao cliente adiado | TRANSCRICAO | [09:37] Larissa |
| ADR-003-CONS-13 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência | Consequência derivada: cliente fora do ar multiplica envios e ocupa o worker até 10 s por envio | TRANSCRICAO | [09:42] Diego |
| ADR-003-CONS-14 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência | Consequência derivada: reprocessamento entrega snapshot antigo após eventos mais novos | TRANSCRICAO | [09:18] Diego |
| ADR-003-CONS-15 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência | Consequência derivada: DLQ cresce sem retenção, com cópia do payload | TRANSCRICAO | [09:18] Diego |
| ADR-003-CONS-16 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência | Consequência derivada: contagem e próximo envio gravados na linha para sobreviver a reinício | TRANSCRICAO | [09:17] Larissa |
| ADR-004 | `docs/adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md` | Decisão | HMAC-SHA256 sobre o corpo em `X-Signature`, secret única por endpoint gerada pela plataforma, rotação com 24 h | TRANSCRICAO | [09:22] Sofia |
| ADR-004-ALT-01 | `docs/adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md` | Alternativa Descartada | Secret global da plataforma | TRANSCRICAO | [09:21] Sofia |
| ADR-004-CONS-01 | `docs/adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md` | Consequência | Cliente valida origem e integridade do corpo | TRANSCRICAO | [09:19] Sofia |
| ADR-004-CONS-02 | `docs/adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md` | Consequência | Vazamento de uma secret afeta um endpoint, não todos | TRANSCRICAO | [09:21] Sofia |
| ADR-004-CONS-03 | `docs/adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md` | Consequência | Rotação responde ao vazamento em log; tempo de migrar depende da forma da assinatura nas 24 h | TRANSCRICAO | [09:21] Sofia |
| ADR-004-CONS-05 | `docs/adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md` | Consequência | Cliente precisa implementar a verificação da assinatura | TRANSCRICAO | [09:20] Sofia |
| ADR-004-CONS-06 | `docs/adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md` | Consequência | Consequência derivada: forma da assinatura nas 24 h de convivência não definida | TRANSCRICAO | [09:21] Sofia |
| ADR-004-CONS-07 | `docs/adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md` | Consequência | Consequência derivada: `X-Timestamp` fora da assinatura; requisição capturada pode ser reenviada | TRANSCRICAO | [09:22] Sofia |
| ADR-004-CONS-08 | `docs/adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md` | Consequência | Consequência derivada: `redactPaths` do logger não mascara `secret` | CODIGO | src/shared/logger/index.ts |
| ADR-004-CONS-09 | `docs/adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md` | Trade-off | Troca secret única por isolamento de vazamento, com secret lida em claro, rotação e verificação no cliente | TRANSCRICAO | [09:21] Sofia |
| ADR-004-CONS-10 | `docs/adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md` | Consequência | Consequência derivada: secret lida em claro para assinar; guardada em claro, vazamento do banco expõe todas | TRANSCRICAO | [09:20] Sofia |
| ADR-004-CONS-11 | `docs/adrs/ADR-004-assinatura-hmac-sha256-por-endpoint.md` | Consequência | Consequência derivada: não definido se respostas além da criação devolvem a secret | TRANSCRICAO | [09:31] Marcos |
| ADR-005 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Decisão | At-least-once, UUID gerado na inserção e enviado em `X-Event-Id`, deduplicação pelo cliente | TRANSCRICAO | [09:26] Larissa |
| ADR-005-ALT-01 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Alternativa Descartada | Exactly-once | TRANSCRICAO | [09:25] Diego |
| ADR-005-CONS-01 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Consequência | Entrega sem coordenar cada envio com o cliente | TRANSCRICAO | [09:25] Diego |
| ADR-005-CONS-02 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Consequência | Consequência derivada: envios do retry deixam de ser problema de correção; reprocessamento depende de questão | TRANSCRICAO | [09:26] Larissa |
| ADR-005-CONS-03 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Consequência | Chave estável de deduplicação gerada na inserção na outbox | TRANSCRICAO | [09:25] Diego |
| ADR-005-CONS-04 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Consequência | Cliente pode receber o mesmo evento mais de uma vez | TRANSCRICAO | [09:24] Diego |
| ADR-005-CONS-05 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Consequência | Consequência derivada: deduplicação depende da documentação no portal | TRANSCRICAO | [09:26] Marcos |
| ADR-005-CONS-06 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Consequência | Consequência derivada: reprocessamento com identificador novo impede a deduplicação | TRANSCRICAO | [09:18] Diego |
| ADR-005-CONS-07 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Consequência | Consequência derivada: granularidade da linha por evento ou por webhook não definida; cliente deduplica por evento e endpoint | TRANSCRICAO | [09:34] Bruno |
| ADR-005-CONS-08 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Trade-off | Troca entrega única por simplicidade, com duplicatas tratadas pelo cliente | TRANSCRICAO | [09:25] Diego |
| ADR-005-CONS-09 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Consequência | Consequência derivada: duplicatas também vêm de workers sobrepostos e de linha presa reenviada | TRANSCRICAO | [09:12] Diego |
| ADR-005-CONS-10 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Consequência | Consequência derivada: janela de retenção dos identificadores no cliente não definida | TRANSCRICAO | [09:17] Diego |
| ADR-005-CONS-11 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Consequência | Consequência derivada: deduplicação é a única defesa do cliente contra reenvio capturado | TRANSCRICAO | [09:22] Sofia |
| ADR-005-CONS-12 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Consequência | Consequência derivada: opções em aberto do ADR-007 deixariam eventos sem envio, fora do at-least-once | TRANSCRICAO | [09:24] Larissa |
| ADR-006 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Decisão | Webhook como módulo da base existente, reuso das capacidades transversais e códigos com prefixo `WEBHOOK_` | TRANSCRICAO | [09:30] Larissa |
| ADR-006-ALT-03 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Alternativa Descartada | Logger próprio do módulo | TRANSCRICAO | [09:29] Bruno |
| ADR-006-ALT-04 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Alternativa Descartada | Hipótese: webhooks como serviço separado, alternativa plausível não discutida | TRANSCRICAO | [09:30] Larissa |
| ADR-006-CONS-01 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Consequência | Erros das rotas do módulo no mesmo formato, via `errorMiddleware` sem mudança | TRANSCRICAO | [09:29] Bruno |
| ADR-006-CONS-02 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Consequência | Nenhuma dependência nova para log, validação, erros ou autorização | TRANSCRICAO | [09:30] Larissa |
| ADR-006-CONS-03 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Consequência | Consequência derivada: mesma estrutura de módulo facilita a navegação | TRANSCRICAO | [09:27] Bruno |
| ADR-006-CONS-04 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Consequência | `NotFoundError` e `validate` têm código fixo; `WEBHOOK_*` não sai desses caminhos sem mudança | CODIGO | src/shared/errors/http-errors.ts |
| ADR-006-CONS-05 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Consequência | Middleware de erro trata só `P2002` e `P2025` e não atua no worker | CODIGO | src/middlewares/error.middleware.ts |
| ADR-006-CONS-06 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Consequência | Prefixo `WEBHOOK_` é convenção nova; nenhum código existente tem prefixo de módulo | CODIGO | src/modules/users/user.service.ts |
| ADR-006-CONS-09 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Consequência | Consequência derivada: Pino só na camada HTTP; módulo e worker acrescentam os próprios logs | CODIGO | src/shared/logger/index.ts |
| ADR-006-CONS-11 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Trade-off | Troca liberdade de desenho e isolamento por consistência com a base | TRANSCRICAO | [09:30] Larissa |
| ADR-006-CONS-12 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Consequência | Consequência derivada: `WEBHOOK_SECRET_REQUIRED` sem caso definido, pois a secret é gerada pela plataforma | TRANSCRICAO | [09:28] Bruno |
| ADR-006-CONS-13 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Consequência | Consequência derivada: rotas no processo da API, implantadas junto e concorrendo com pedidos | TRANSCRICAO | [09:30] Larissa |
| ADR-007 | `docs/adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md` | Decisão | Payload em snapshot na inserção, sem itens; filtro por status do pedido aplicado antes de gravar | TRANSCRICAO | [09:52] Bruno |
| ADR-007-ALT-01 | `docs/adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md` | Alternativa Descartada | Guardar só o id do pedido e montar o payload no envio | TRANSCRICAO | [09:52] Larissa |
| ADR-007-ALT-02 | `docs/adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md` | Alternativa Descartada | Filtrar os eventos na hora do envio | TRANSCRICAO | [09:34] Bruno |
| ADR-007-ALT-03 | `docs/adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md` | Alternativa Descartada | Mandar os itens do pedido no payload | TRANSCRICAO | [09:43] Diego |
| ADR-007-CONS-01 | `docs/adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md` | Consequência | Evento reflete o estado de quando o status do pedido mudou | TRANSCRICAO | [09:52] Larissa |
| ADR-007-CONS-02 | `docs/adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md` | Consequência | Outbox sem linhas que ninguém vai receber | TRANSCRICAO | [09:34] Bruno |
| ADR-007-CONS-03 | `docs/adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md` | Consequência | Consequência derivada: worker não lê o pedido no envio; retry manda o mesmo conteúdo, reprocessamento depende do ADR-005 | TRANSCRICAO | [09:52] Bruno |
| ADR-007-CONS-04 | `docs/adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md` | Consequência | Snapshot decidido sem Marcos e Sofia e fora do resumo confirmado | TRANSCRICAO | [09:52] Bruno |
| ADR-007-CONS-05 | `docs/adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md` | Consequência | Consequência derivada: leitura da configuração na transação; filtro vale só para eventos novos | TRANSCRICAO | [09:34] Bruno |
| ADR-007-CONS-06 | `docs/adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md` | Consequência | Consequência derivada: payload entregue pode estar defasado em relação ao pedido atual | TRANSCRICAO | [09:52] Larissa |
| ADR-007-CONS-07 | `docs/adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md` | Consequência | Consequência derivada: lista de dados básicos do pedido não fechada | TRANSCRICAO | [09:43] Diego |
| ADR-007-CONS-08 | `docs/adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md` | Trade-off | Troca payload atual e filtro flexível por fidelidade ao momento e outbox menor | TRANSCRICAO | [09:52] Larissa |
| ADR-007-CONS-09 | `docs/adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md` | Consequência | Consequência derivada: limite de 64 KB checado na inserção desfaria a mudança de status | TRANSCRICAO | [09:24] Larissa |
| ADR-007-CONS-10 | `docs/adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md` | Consequência | Consequência derivada: efeito do estado ativo no filtro e destino das linhas de webhook desativado ou removido não definidos | TRANSCRICAO | [09:21] Sofia |
| ADR-007-CONS-11 | `docs/adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md` | Consequência | Consequência derivada: filtro pede índice pelo cliente; snapshot ocupa outbox e DLQ | TRANSCRICAO | [09:34] Bruno |
