# Tracker de Rastreabilidade

Mapa de cada item dos documentos de design até a sua origem na reunião técnica de quinta-feira (`TRANSCRICAO.md`) ou no código existente.
Cada linha corresponde a um ID definido em um documento. PRD, RFC e FDD usam prefixo do documento, tipo e número de 2 dígitos (`PRD-FR-01`, `RFC-QA-03`, `FDD-CONTRATO-02`; o significado do tipo está na coluna Tipo); nos ADRs, `ADR-NNN` é a decisão, `ADR-NNN-ALT-NN` uma alternativa e `ADR-NNN-CONS-NN` uma consequência.
Ordem: PRD, RFC, ADR-001 a ADR-007 e FDD; dentro de cada documento, por tipo (na ordem das seções) e por número. Número que falta na sequência é item removido numa revisão: IDs não são renumerados nem reaproveitados. Sufixo `b` (`FDD-INT-01b`) é a segunda fonte do mesmo item, logo depois da linha base.
Fonte `TRANSCRICAO`: Localização é a fala `[hh:mm] Nome` que sustenta o item. Fonte `CODIGO`: Localização é o arquivo do repositório.
Resumo iniciado por "Proposta:", "Hipótese:" ou "Consequência derivada:" indica item que a reunião não decidiu diretamente; a Localização é a fala ou o arquivo que o motiva.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| PRD-PROB-01 | `docs/PRD.md` | Problema | Clientes consultam a lista de pedidos de tempos em tempos; integração lenta e cara | TRANSCRICAO | [09:00] Marcos |
| PRD-PROB-02 | `docs/PRD.md` | Problema | Pedido formal dos 3 clientes B2B pela notificação de mudança de status | TRANSCRICAO | [09:00] Marcos |
| PRD-PROB-03 | `docs/PRD.md` | Problema | Atlas sugeriu migrar para concorrente se não sair até o fim do trimestre | TRANSCRICAO | [09:00] Marcos |
| PRD-PROB-04 | `docs/PRD.md` | Problema | Data pedida pela Atlas: fim de novembro | TRANSCRICAO | [09:45] Marcos |
| PRD-PUB-01 | `docs/PRD.md` | Público-alvo | Os 3 clientes B2B e os sistemas deles que recebem as notificações | TRANSCRICAO | [09:00] Marcos |
| PRD-PUB-02 | `docs/PRD.md` | Público-alvo | Usuários que cadastram webhooks em nome do cliente pela API; serem usuários comuns é hipótese | TRANSCRICAO | [09:32] Marcos |
| PRD-PUB-03 | `docs/PRD.md` | Público-alvo | Administradores que reprocessam entregas esgotadas | TRANSCRICAO | [09:36] Larissa |
| PRD-PUB-04 | `docs/PRD.md` | Público-alvo | Desenvolvedores dos clientes que leem o portal do desenvolvedor | TRANSCRICAO | [09:40] Marcos |
| PRD-PUB-05 | `docs/PRD.md` | Público-alvo | Cenário: cliente recebe a notificação e consulta o pedido na API para detalhes | TRANSCRICAO | [09:43] Diego |
| PRD-PUB-06 | `docs/PRD.md` | Público-alvo | Cenário: cliente escolhe só os status que interessam | TRANSCRICAO | [09:33] Marcos |
| PRD-PUB-07 | `docs/PRD.md` | Público-alvo | Cenário: cliente com vários webhooks sabe qual cadastro recebeu a notificação | TRANSCRICAO | [09:44] Sofia |
| PRD-PUB-08 | `docs/PRD.md` | Público-alvo | Cenário: cliente confere no histórico o que foi enviado e o que respondeu | TRANSCRICAO | [09:34] Marcos |
| PRD-PUB-09 | `docs/PRD.md` | Público-alvo | Cenário: cliente fora do ar por horas recebe depois pelo reenvio automático | TRANSCRICAO | [09:16] Diego |
| PRD-PUB-10 | `docs/PRD.md` | Público-alvo | Cenário: administrador reprocessa entrega esgotada após o cliente se recuperar | TRANSCRICAO | [09:18] Diego |
| PRD-PUB-11 | `docs/PRD.md` | Público-alvo | Cenário: cliente troca a secret após vazamento e tem 24 h para migrar | TRANSCRICAO | [09:21] Sofia |
| PRD-PUB-12 | `docs/PRD.md` | Público-alvo | Cenário: cliente descarta notificação repetida pelo identificador | TRANSCRICAO | [09:25] Diego |
| PRD-OBJ-01 | `docs/PRD.md` | Objetivo | Notificação abaixo de 10 s; só atendida quando o primeiro envio dá certo (RFC-QA-07) | TRANSCRICAO | [09:02] Marcos |
| PRD-OBJ-02 | `docs/PRD.md` | Objetivo | Prazo da primeira entrega: 3 sprints, com a revisão de segurança incluída | TRANSCRICAO | [09:47] Larissa |
| PRD-OBJ-03 | `docs/PRD.md` | Objetivo | Reenvio automático por quase 15 h antes de exigir ação manual; contagem em aberto (RFC-QA-08) | TRANSCRICAO | [09:17] Diego |
| PRD-OBJ-04 | `docs/PRD.md` | Objetivo | Histórico com os últimos 100 envios por webhook | TRANSCRICAO | [09:34] Marcos |
| PRD-OBJ-05 | `docs/PRD.md` | Objetivo | Toda notificação com assinatura verificável pelo cliente | TRANSCRICAO | [09:19] Sofia |
| PRD-ESC-01 | `docs/PRD.md` | Escopo | Notificação de mudança de status do pedido, só de saída | TRANSCRICAO | [09:02] Marcos |
| PRD-ESC-02 | `docs/PRD.md` | Escopo | Cadastro, listagem por cliente, edição e remoção de webhooks | TRANSCRICAO | [09:31] Marcos |
| PRD-ESC-03 | `docs/PRD.md` | Escopo | Escolha dos status do pedido por webhook | TRANSCRICAO | [09:33] Marcos |
| PRD-ESC-04 | `docs/PRD.md` | Escopo | Assinatura verificável com secret por webhook e rotação | TRANSCRICAO | [09:22] Sofia |
| PRD-ESC-05 | `docs/PRD.md` | Escopo | Reenvio automático e guarda das entregas esgotadas | TRANSCRICAO | [09:17] Larissa |
| PRD-ESC-06 | `docs/PRD.md` | Escopo | Reprocessamento manual por administrador | TRANSCRICAO | [09:36] Larissa |
| PRD-ESC-07 | `docs/PRD.md` | Escopo | Histórico de entregas por webhook | TRANSCRICAO | [09:34] Marcos |
| PRD-ESC-08 | `docs/PRD.md` | Escopo | Documentação no portal do desenvolvedor, com aviso de repetição | TRANSCRICAO | [09:40] Marcos |
| PRD-FORA-01 | `docs/PRD.md` | Fora de escopo | Webhooks de entrada; clientes só querem receber | TRANSCRICAO | [09:02] Marcos |
| PRD-FORA-02 | `docs/PRD.md` | Fora de escopo | Aviso por e-mail sobre falhas, adiado; não é caminho de entrega | TRANSCRICAO | [09:37] Larissa |
| PRD-FORA-03 | `docs/PRD.md` | Fora de escopo | Painel visual para o cliente, projeto do time de frontend | TRANSCRICAO | [09:40] Larissa |
| PRD-FORA-04 | `docs/PRD.md` | Fora de escopo | Limite de taxa de envios por cliente, em observação (RFC-QA-02) | TRANSCRICAO | [09:39] Larissa |
| PRD-FORA-05 | `docs/PRD.md` | Fora de escopo | Arquivamento das notificações entregues, sem prazo | TRANSCRICAO | [09:08] Diego |
| PRD-FORA-06 | `docs/PRD.md` | Fora de escopo | Ordem global entre pedidos e vários processos de envio em paralelo (RFC-QA-03) | TRANSCRICAO | [09:13] Diego |
| PRD-FORA-07 | `docs/PRD.md` | Fora de escopo | Permissões mais restritas no cadastro de webhooks (RFC-QA-04) | TRANSCRICAO | [09:37] Sofia |
| PRD-FORA-08 | `docs/PRD.md` | Fora de escopo | Proposta: criação do pedido não gera notificação nesta fase (questão 1 do ADR-001) | TRANSCRICAO | [09:40] Bruno |
| PRD-FR-01 | `docs/PRD.md` | Requisito Funcional | Cadastrar webhook com URL https e lista de status; secret gerada e devolvida na criação | TRANSCRICAO | [09:31] Marcos |
| PRD-FR-02 | `docs/PRD.md` | Requisito Funcional | Listar os webhooks de um cliente, sem a secret (proposta) | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-03 | `docs/PRD.md` | Requisito Funcional | Editar URL, lista de status e estado ativo, com as recusas do cadastro | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-04 | `docs/PRD.md` | Requisito Funcional | Remover webhook; efeito sobre notificações já registradas é proposta | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-05 | `docs/PRD.md` | Requisito Funcional | Cada webhook escolhe os status do pedido e só recebe esses | TRANSCRICAO | [09:33] Marcos |
| PRD-FR-06 | `docs/PRD.md` | Requisito Funcional | Notificar a mudança de status com dados básicos do pedido, sem itens, refletindo o momento da mudança | TRANSCRICAO | [09:00] Marcos |
| PRD-FR-07 | `docs/PRD.md` | Requisito Funcional | Assinar cada notificação com a secret do webhook para o cliente verificar origem e integridade | TRANSCRICAO | [09:22] Sofia |
| PRD-FR-08 | `docs/PRD.md` | Requisito Funcional | Reenviar com intervalos 1m/5m/30m/2h/12h por quase 15 h; contagem em aberto | TRANSCRICAO | [09:17] Larissa |
| PRD-FR-09 | `docs/PRD.md` | Requisito Funcional | Guardar entregas esgotadas com conteúdo, motivo e momento | TRANSCRICAO | [09:18] Diego |
| PRD-FR-10 | `docs/PRD.md` | Requisito Funcional | Reprocessar entrega esgotada, só administrador, com registro de quem fez | TRANSCRICAO | [09:18] Diego |
| PRD-FR-11 | `docs/PRD.md` | Requisito Funcional | Histórico com os últimos 100 envios, sucesso ou falha, conteúdo, resposta e tempo | TRANSCRICAO | [09:34] Marcos |
| PRD-FR-12 | `docs/PRD.md` | Requisito Funcional | Rotacionar a secret pela API; anterior vale por 24 h | TRANSCRICAO | [09:21] Sofia |
| PRD-RNF-01 | `docs/PRD.md` | Requisito Não Funcional | Notificação abaixo de 10 s, só atendida quando o primeiro envio dá certo (RFC-QA-07) | TRANSCRICAO | [09:02] Marcos |
| PRD-RNF-02 | `docs/PRD.md` | Requisito Não Funcional | Atraso de até 2 s no pior caso causado pela leitura periódica | TRANSCRICAO | [09:10] Larissa |
| PRD-RNF-03 | `docs/PRD.md` | Requisito Não Funcional | Cliente tem 10 s para responder; depois é falha e reenvio; não é a meta de 10 s | TRANSCRICAO | [09:42] Diego |
| PRD-RNF-04 | `docs/PRD.md` | Requisito Não Funcional | Entrega não para quando a API reinicia; envio em processo separado | TRANSCRICAO | [09:11] Diego |
| PRD-RNF-05 | `docs/PRD.md` | Requisito Não Funcional | Reenvio cobre indisponibilidades como a de 2 h, dentro da janela de quase 15 h | TRANSCRICAO | [09:16] Diego |
| PRD-RNF-06 | `docs/PRD.md` | Requisito Não Funcional | URL do webhook só https | TRANSCRICAO | [09:23] Sofia |
| PRD-RNF-07 | `docs/PRD.md` | Requisito Não Funcional | Toda notificação com assinatura verificável | TRANSCRICAO | [09:19] Sofia |
| PRD-RNF-08 | `docs/PRD.md` | Requisito Não Funcional | Secret única por webhook | TRANSCRICAO | [09:21] Sofia |
| PRD-RNF-09 | `docs/PRD.md` | Requisito Não Funcional | Secret rotacionável com 24 h de convivência | TRANSCRICAO | [09:21] Sofia |
| PRD-RNF-10 | `docs/PRD.md` | Requisito Não Funcional | Secret gerada pela plataforma e devolvida na criação; demais respostas são proposta | TRANSCRICAO | [09:31] Marcos |
| PRD-RNF-11 | `docs/PRD.md` | Requisito Não Funcional | Proposta: secret nunca em log da plataforma (questão 3 do ADR-004) | TRANSCRICAO | [09:22] Diego |
| PRD-RNF-12 | `docs/PRD.md` | Requisito Não Funcional | Reprocessamento só por administrador, com registro do usuário | TRANSCRICAO | [09:36] Larissa |
| PRD-RNF-13 | `docs/PRD.md` | Requisito Não Funcional | Demais operações para qualquer usuário autenticado nesta fase | TRANSCRICAO | [09:37] Sofia |
| PRD-RNF-14 | `docs/PRD.md` | Requisito Não Funcional | Revisão de segurança antes da entrada em produção | TRANSCRICAO | [09:46] Sofia |
| PRD-RNF-15 | `docs/PRD.md` | Requisito Não Funcional | Cliente vê o histórico de entregas | TRANSCRICAO | [09:34] Marcos |
| PRD-RNF-16 | `docs/PRD.md` | Requisito Não Funcional | Proposta: operação sabe quantas entregas esgotadas aguardam e se os envios pararam | TRANSCRICAO | [09:18] Diego |
| PRD-RNF-17 | `docs/PRD.md` | Requisito Não Funcional | Registro de quem reprocessou disponível para auditoria | TRANSCRICAO | [09:36] Sofia |
| PRD-RNF-18 | `docs/PRD.md` | Requisito Não Funcional | Mudança de status desejada feita pela operação normal da API gera notificação; mudança desfeita não gera; exceções propostas | TRANSCRICAO | [09:40] Bruno |
| PRD-RNF-19 | `docs/PRD.md` | Requisito Não Funcional | Entrega pelo menos uma vez; cliente descarta repetidas pelo identificador (com vários webhooks, junto com o webhook) | TRANSCRICAO | [09:25] Diego |
| PRD-RNF-20 | `docs/PRD.md` | Requisito Não Funcional | Notificação reflete o pedido no momento da mudança | TRANSCRICAO | [09:52] Larissa |
| PRD-RNF-21 | `docs/PRD.md` | Requisito Não Funcional | Sem ordem global; ordem por pedido vale com processo de envio único; reenvio, reprocessamento e demais casos do ADR-002 podem invertê-la (RFC-QA-05) | TRANSCRICAO | [09:13] Larissa |
| PRD-RNF-22 | `docs/PRD.md` | Requisito Não Funcional | Limite de 64 KB com erro; destino da notificação acima do limite é proposta | TRANSCRICAO | [09:24] Larissa |
| PRD-RNF-23 | `docs/PRD.md` | Requisito Não Funcional | Só a mudança de status do pedido muda por dentro, com o contrato mantido; na proposta de não mudar a exclusão de cliente, a de cliente com webhook passa a ser recusada | TRANSCRICAO | [09:40] Bruno |
| PRD-RNF-23b | `docs/PRD.md` | Requisito Não Funcional | Na proposta de não mudar a exclusão de cliente, a de cliente com webhook é recusada como a de cliente com pedido (chave estrangeira com `ON DELETE RESTRICT`) | CODIGO | prisma/migrations/20260519182739_init/migration.sql |
| PRD-DEC-01 | `docs/PRD.md` | Decisão | Outbox no MySQL na transação da mudança de status, sem infra nova (ADR-001) | TRANSCRICAO | [09:08] Larissa |
| PRD-DEC-02 | `docs/PRD.md` | Decisão | Worker separado com polling de 2 s (ADR-002) | TRANSCRICAO | [09:10] Larissa |
| PRD-DEC-03 | `docs/PRD.md` | Decisão | Retry com backoff, DLQ separada e reprocessamento por administrador (ADR-003) | TRANSCRICAO | [09:17] Larissa |
| PRD-DEC-04 | `docs/PRD.md` | Decisão | HMAC-SHA256 sobre o corpo, secret por webhook, rotação com 24 h (ADR-004) | TRANSCRICAO | [09:22] Sofia |
| PRD-DEC-05 | `docs/PRD.md` | Decisão | At-least-once com identificador para deduplicação no cliente (ADR-005) | TRANSCRICAO | [09:26] Larissa |
| PRD-DEC-06 | `docs/PRD.md` | Decisão | Reuso dos padrões do projeto, webhook como módulo da mesma base (ADR-006) | TRANSCRICAO | [09:30] Larissa |
| PRD-DEC-07 | `docs/PRD.md` | Decisão | Snapshot na inserção e filtro por status antes de registrar (ADR-007) | TRANSCRICAO | [09:52] Bruno |
| PRD-DEP-01 | `docs/PRD.md` | Dependência | Revisão de segurança de pelo menos 2 dias úteis, incluída no prazo | TRANSCRICAO | [09:46] Sofia |
| PRD-DEP-02 | `docs/PRD.md` | Dependência | Documentação no portal: como integrar e aviso de repetição | TRANSCRICAO | [09:40] Marcos |
| PRD-DEP-03 | `docs/PRD.md` | Dependência | Marcos confirma o prazo com a Atlas e atualiza os clientes | TRANSCRICAO | [09:47] Marcos |
| PRD-DEP-04 | `docs/PRD.md` | Dependência | Respostas de Marcos às RFC-QA-05, RFC-QA-07 e RFC-QA-08 antes de codar; sessão de revisão marcada só com Bruno e Diego | TRANSCRICAO | [09:50] Larissa |
| PRD-DEP-05 | `docs/PRD.md` | Dependência | Hipótese: usuários que representam o cliente são usuários comuns, sem vínculo com cliente | TRANSCRICAO | [09:32] Marcos |
| PRD-DEP-06 | `docs/PRD.md` | Dependência | Painel visual do time de frontend, fora desta fase, dependeria do histórico | TRANSCRICAO | [09:40] Larissa |
| PRD-RISCO-01 | `docs/PRD.md` | Risco | Atraso na entrega e churn da Atlas; blocos já somam 3 sprints antes da revisão | TRANSCRICAO | [09:00] Marcos |
| PRD-RISCO-02 | `docs/PRD.md` | Risco | Cliente que não descarta notificações repetidas | TRANSCRICAO | [09:25] Diego |
| PRD-RISCO-03 | `docs/PRD.md` | Risco | Vazamento da secret pelo cliente | TRANSCRICAO | [09:22] Diego |
| PRD-RISCO-04 | `docs/PRD.md` | Risco | Cliente fora do ar por mais que a janela de reenvio depende de reprocessamento manual | TRANSCRICAO | [09:17] Marcos |
| PRD-RISCO-05 | `docs/PRD.md` | Risco | Consequência derivada: notificações de um pedido fora de ordem no reenvio e no reprocessamento | TRANSCRICAO | [09:17] Larissa |
| PRD-RISCO-06 | `docs/PRD.md` | Risco | Meta de 10 s não atendida quando o primeiro envio falha | TRANSCRICAO | [09:02] Marcos |
| PRD-CA-01 | `docs/PRD.md` | Critério de Aceitação | Mudança de status desejada feita pela operação normal da API gera notificação; mudança desfeita não gera | TRANSCRICAO | [09:40] Bruno |
| PRD-CA-02 | `docs/PRD.md` | Critério de Aceitação | Webhook que não escolheu o status não recebe | TRANSCRICAO | [09:33] Bruno |
| PRD-CA-03 | `docs/PRD.md` | Critério de Aceitação | Primeiro envio até 2 s após a mudança no caminho feliz, sem prometer 10 s com reenvio | TRANSCRICAO | [09:10] Larissa |
| PRD-CA-04 | `docs/PRD.md` | Critério de Aceitação | URL sem https recusada | TRANSCRICAO | [09:23] Sofia |
| PRD-CA-05 | `docs/PRD.md` | Critério de Aceitação | Proposta: lista de status inválida recusada | TRANSCRICAO | [09:33] Marcos |
| PRD-CA-06 | `docs/PRD.md` | Critério de Aceitação | Sem resposta em 10 s é falha e recebe reenvio | TRANSCRICAO | [09:42] Diego |
| PRD-CA-07 | `docs/PRD.md` | Critério de Aceitação | Progressão de intervalos 1m/5m/30m/2h/12h e entregas esgotadas ao fim; hipótese de envio inicial mais 5 reenvios (RFC-QA-08) | TRANSCRICAO | [09:17] Larissa |
| PRD-CA-08 | `docs/PRD.md` | Critério de Aceitação | Cliente verifica a assinatura com a secret; 24 h após rotação é proposta (RFC-QA-09) | TRANSCRICAO | [09:22] Sofia |
| PRD-CA-09 | `docs/PRD.md` | Critério de Aceitação | Mesmo identificador da notificação em todos os reenvios automáticos | TRANSCRICAO | [09:25] Diego |
| PRD-CA-10 | `docs/PRD.md` | Critério de Aceitação | Reprocessamento só por administrador, registrado com o usuário | TRANSCRICAO | [09:36] Sofia |
| PRD-CA-11 | `docs/PRD.md` | Critério de Aceitação | Proposta: reprocessamento mantém identificador; segunda tentativa recusada (RFC-QA-11) | TRANSCRICAO | [09:18] Diego |
| PRD-CA-12 | `docs/PRD.md` | Critério de Aceitação | Secret na resposta da criação; só nela e na rotação é proposta | TRANSCRICAO | [09:31] Marcos |
| PRD-CA-13 | `docs/PRD.md` | Critério de Aceitação | Proposta: secret nunca em log da plataforma | TRANSCRICAO | [09:22] Diego |
| PRD-CA-14 | `docs/PRD.md` | Critério de Aceitação | Histórico com no máximo os últimos 100 envios | TRANSCRICAO | [09:34] Marcos |
| PRD-CA-15 | `docs/PRD.md` | Critério de Aceitação | 64 KB é erro; não desfazer a mudança de status é proposta | TRANSCRICAO | [09:24] Larissa |
| PRD-CA-16 | `docs/PRD.md` | Critério de Aceitação | Revisão de segurança concluída antes da entrada em produção | TRANSCRICAO | [09:46] Sofia |
| PRD-CA-17 | `docs/PRD.md` | Critério de Aceitação | Documentação publicada no portal: modo de integrar e aviso de notificação repetida | TRANSCRICAO | [09:40] Marcos |
| PRD-TESTE-01 | `docs/PRD.md` | Estratégia de Teste | Testes de integração com vitest e supertest contra o MySQL real, no padrão do repositório | CODIGO | tests/orders.test.ts |
| PRD-TESTE-02 | `docs/PRD.md` | Estratégia de Teste | Falha forçada no registro da notificação desfaz a mudança de status | TRANSCRICAO | [09:40] Bruno |
| PRD-TESTE-03 | `docs/PRD.md` | Estratégia de Teste | Verificação da assinatura com a secret do webhook | TRANSCRICAO | [09:22] Sofia |
| PRD-TESTE-04 | `docs/PRD.md` | Estratégia de Teste | Autorização do reprocessamento só para administrador | TRANSCRICAO | [09:36] Larissa |
| PRD-TESTE-05 | `docs/PRD.md` | Estratégia de Teste | Suíte existente continua passando (script test) | CODIGO | package.json |
| PRD-TESTE-06 | `docs/PRD.md` | Estratégia de Teste | Revisão de segurança manual de pelo menos 2 dias úteis | TRANSCRICAO | [09:46] Sofia |
| PRD-TESTE-07 | `docs/PRD.md` | Estratégia de Teste | Revisão do documento de design com Bruno e Diego antes de codar | TRANSCRICAO | [09:50] Larissa |
| PRD-TESTE-08 | `docs/PRD.md` | Estratégia de Teste | Proposta: Product Manager confere a documentação do portal | TRANSCRICAO | [09:40] Marcos |
| RFC-PROP-01 | `docs/RFC.md` | Proposta Técnica | Evento gravado na outbox na mesma transação do `changeStatus`; falha na gravação desfaz a mudança (ADR-001) | TRANSCRICAO | [09:08] Larissa |
| RFC-PROP-02 | `docs/RFC.md` | Proposta Técnica | Filtro por status do pedido antes de gravar e payload em snapshot, sem itens (ADR-007) | TRANSCRICAO | [09:52] Bruno |
| RFC-PROP-03 | `docs/RFC.md` | Proposta Técnica | Worker em processo separado lê pendentes a cada 2 s; worker único é premissa (ADR-002) | TRANSCRICAO | [09:10] Larissa |
| RFC-PROP-03b | `docs/RFC.md` | Proposta Técnica | Worker único dá ordem por pedido, salvo reentrega e envio concorrente (RFC-QA-05, RFC-QA-06) | TRANSCRICAO | [09:13] Larissa |
| RFC-PROP-04 | `docs/RFC.md` | Proposta Técnica | Envio com HMAC-SHA256 do corpo em `X-Signature`, secret por endpoint rotacionável com 24 h, forma da assinatura na convivência em aberto, e UUID em `X-Event-Id` (ADR-004, ADR-005) | TRANSCRICAO | [09:22] Sofia |
| RFC-PROP-05 | `docs/RFC.md` | Proposta Técnica | Retry com backoff 1m/5m/30m/2h/12h, timeout HTTP de 10 s conta como falha, teto "5 tentativas" com contagem em aberto (ADR-003) | TRANSCRICAO | [09:17] Larissa |
| RFC-PROP-06 | `docs/RFC.md` | Proposta Técnica | Esgotados os envios, evento vai para `webhook_dead_letter`; reprocessamento por ADMIN com registro de quem fez (ADR-003) | TRANSCRICAO | [09:18] Diego |
| RFC-PROP-07 | `docs/RFC.md` | Proposta Técnica | Entrega at-least-once com deduplicação pelo cliente via `X-Event-Id`; no reprocessamento, depende da RFC-QA-11 (ADR-005) | TRANSCRICAO | [09:26] Larissa |
| RFC-PROP-08 | `docs/RFC.md` | Proposta Técnica | Módulo `src/modules/webhooks/` (novo) no padrão dos módulos, reuso dos padrões transversais e prefixo `WEBHOOK_` (ADR-006) | TRANSCRICAO | [09:30] Larissa |
| RFC-PROP-09 | `docs/RFC.md` | Proposta Técnica | Capacidades da API de configuração e consulta: cadastro com secret gerada e URL https, listar, editar, remover, status de interesse, últimos 100 envios, rotação, reprocessamento ADMIN; demais operações para qualquer autenticado | TRANSCRICAO | [09:31] Marcos |
| RFC-PROP-10 | `docs/RFC.md` | Proposta Técnica | Fora desta proposta: webhooks de entrada, aviso por e-mail, painel visual, arquivamento sem prazo, múltiplos workers | TRANSCRICAO | [09:40] Larissa |
| RFC-ALT-01 | `docs/RFC.md` | Alternativa Descartada | Disparo síncrono no `changeStatus` | TRANSCRICAO | [09:04] Bruno |
| RFC-ALT-02 | `docs/RFC.md` | Alternativa Descartada | Redis Streams | TRANSCRICAO | [09:07] Diego |
| RFC-ALT-03 | `docs/RFC.md` | Alternativa Descartada | Trigger do MySQL para acordar o worker | TRANSCRICAO | [09:09] Diego |
| RFC-ALT-04 | `docs/RFC.md` | Alternativa Descartada | Worker no mesmo processo da API | TRANSCRICAO | [09:11] Diego |
| RFC-ALT-05 | `docs/RFC.md` | Alternativa Descartada | Exactly-once, descartado por exigir coordenação dos dois lados | TRANSCRICAO | [09:25] Diego |
| RFC-QA-01 | `docs/RFC.md` | Questão em Aberto | `customer_id` do cadastro no body ou no path (não decidido) | TRANSCRICAO | [09:32] Larissa |
| RFC-QA-02 | `docs/RFC.md` | Questão em Aberto | Rate limiting de envios por cliente ("observar e decidir depois", gatilho "se virar problema") | TRANSCRICAO | [09:39] Larissa |
| RFC-QA-03 | `docs/RFC.md` | Questão em Aberto | Escala para vários workers: particionar por pedido ou lock pessimista (adiado) | TRANSCRICAO | [09:13] Diego |
| RFC-QA-04 | `docs/RFC.md` | Questão em Aberto | Endurecer as permissões do CRUD de configuração (adiado) | TRANSCRICAO | [09:37] Sofia |
| RFC-QA-05 | `docs/RFC.md` | Questão em Aberto | Lacuna: ordem por pedido durante o retry, reordenar ou segurar (ADR-002, questão 1) | TRANSCRICAO | [09:12] Diego |
| RFC-QA-06 | `docs/RFC.md` | Questão em Aberto | Lacuna: envio das linhas do ciclo uma de cada vez ou ao mesmo tempo (ADR-002, questão 2) | TRANSCRICAO | [09:42] Diego |
| RFC-QA-07 | `docs/RFC.md` | Questão em Aberto | Lacuna: meta de 10 s vale só para o primeiro envio (ADR-002, questão 3) | TRANSCRICAO | [09:02] Marcos |
| RFC-QA-08 | `docs/RFC.md` | Questão em Aberto | Lacuna: "5 tentativas" com ou sem o envio inicial (ADR-003, questão 1) | TRANSCRICAO | [09:17] Larissa |
| RFC-QA-09 | `docs/RFC.md` | Questão em Aberto | Lacuna: forma da assinatura nas 24 h de convivência das secrets (ADR-004, questão 1) | TRANSCRICAO | [09:21] Sofia |
| RFC-QA-10 | `docs/RFC.md` | Questão em Aberto | Lacuna: `X-Timestamp` dentro ou fora do que é assinado (ADR-004, questão 2) | TRANSCRICAO | [09:44] Diego |
| RFC-QA-11 | `docs/RFC.md` | Questão em Aberto | Lacuna: reprocessamento mantém ou não o identificador do evento (ADR-005, questão 1) | TRANSCRICAO | [09:18] Diego |
| RFC-QA-12 | `docs/RFC.md` | Questão em Aberto | Lacuna: prefixo `WEBHOOK_` nos erros que saem de caminhos com código fixo (ADR-006, questão 1) | TRANSCRICAO | [09:29] Larissa |
| RFC-RISCO-01 | `docs/RFC.md` | Risco | Estimativa por bloco já fecha 3 sprints antes do "mais um pouco" e da revisão de segurança | TRANSCRICAO | [09:46] Larissa |
| RFC-RISCO-02 | `docs/RFC.md` | Risco | Reprocessamento por ADMIN fraco: registro público aceita ADMIN (ADR-003-CONS-08) | CODIGO | src/modules/auth/auth.routes.ts |
| RFC-RISCO-03 | `docs/RFC.md` | Risco | Outbox e DLQ sem política de retenção (ADR-001-CONS-06, ADR-003-CONS-15) | TRANSCRICAO | [09:08] Diego |
| RFC-RISCO-04 | `docs/RFC.md` | Risco | Worker único sem garantia e worker parado sem detecção (ADR-002-CONS-10, ADR-002-CONS-13) | TRANSCRICAO | [09:12] Diego |
| RFC-RISCO-05 | `docs/RFC.md` | Risco | Meta de 10 s só atendida no caminho feliz; primeiro envio que falha espera pelo menos 1 min (ADR-002-CONS-12) | TRANSCRICAO | [09:17] Larissa |
| RFC-RISCO-06 | `docs/RFC.md` | Risco | Consequência derivada: se o limite de 64 KB for checado na inserção com erro, a mudança de status é desfeita (ADR-007-CONS-09) | TRANSCRICAO | [09:24] Larissa |
| RFC-IMP-01 | `docs/RFC.md` | Impacto | `changeStatus` ganha escrita na outbox, leitura da configuração e nova causa de rollback (ADR-001-CONS-04, ADR-007-CONS-05) | TRANSCRICAO | [09:40] Bruno |
| RFC-IMP-02 | `docs/RFC.md` | Impacto | Segundo processo para implantar, supervisionar e monitorar; carga contínua no MySQL (ADR-002-CONS-08, ADR-002-CONS-15) | TRANSCRICAO | [09:11] Larissa |
| RFC-IMP-03 | `docs/RFC.md` | Impacto | Cliente verifica assinatura, deduplica e lida com eventos fora de ordem; documentação no portal | TRANSCRICAO | [09:26] Marcos |
| RFC-IMP-04 | `docs/RFC.md` | Impacto | Prazo de 3 sprints com revisão de segurança de pelo menos 2 dias úteis no fim; Atlas pede fim de novembro; Marcos confirma o prazo | TRANSCRICAO | [09:47] Larissa |
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
| ADR-002 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Decisão | Worker em processo separado, polling de 2 s, worker único como premissa | TRANSCRICAO | [09:10] Larissa |
| ADR-002b | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Decisão | Ordem por pedido com worker único, como limitação conhecida; reentregas, envio concorrente e vários workers podem invertê-la | TRANSCRICAO | [09:13] Larissa |
| ADR-002-ALT-01 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Alternativa Descartada | Acionamento reativo pelo banco (trigger do MySQL) | TRANSCRICAO | [09:09] Diego |
| ADR-002-ALT-02 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Alternativa Descartada | Worker no mesmo processo da API | TRANSCRICAO | [09:11] Diego |
| ADR-002-CONS-01 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Polling de 2 s atende a meta de 10 s no caminho feliz, aceito pelo produto | TRANSCRICAO | [09:10] Marcos |
| ADR-002-CONS-02 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Reinício da API não para a entrega de eventos | TRANSCRICAO | [09:11] Diego |
| ADR-002-CONS-03 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Sem infraestrutura nova nem mecanismo improvisado no banco | TRANSCRICAO | [09:09] Diego |
| ADR-002-CONS-04 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Atraso de até 2 s no pior caso causado pelo polling | TRANSCRICAO | [09:10] Larissa |
| ADR-002-CONS-05 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Sem ordem global entre pedidos; clientes não pediram | TRANSCRICAO | [09:13] Larissa |
| ADR-002-CONS-06 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Vários workers ao mesmo tempo perdem a ordem por pedido; particionamento adiado | TRANSCRICAO | [09:13] Diego |
| ADR-002-CONS-07 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Consequência derivada: retry reordena eventos do mesmo pedido mesmo com worker único | TRANSCRICAO | [09:17] Larissa |
| ADR-002-CONS-08 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Consequência derivada: segundo processo sem implantação nem supervisão definidas | TRANSCRICAO | [09:11] Larissa |
| ADR-002-CONS-09 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Trade-off | Troca reatividade imediata e ordem em todos os casos por simplicidade no banco existente | TRANSCRICAO | [09:10] Larissa |
| ADR-002-CONS-10 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Consequência derivada: premissa de worker único não garantida em deploy, reinício ou execução manual | TRANSCRICAO | [09:12] Diego |
| ADR-002-CONS-11 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Consequência derivada: envio das linhas do ciclo uma a uma ou ao mesmo tempo não definido | TRANSCRICAO | [09:42] Diego |
| ADR-002-CONS-12 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Consequência derivada: meta de 10 s só atendida no caminho feliz | TRANSCRICAO | [09:17] Larissa |
| ADR-002-CONS-13 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Consequência derivada: worker parado não é percebido; sem health check nem métricas | CODIGO | src/app.ts |
| ADR-002-CONS-14 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Consequência derivada: desligamento no modelo atual deixa linha presa em processando | CODIGO | src/server.ts |
| ADR-002-CONS-15 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Consequência derivada: custo contínuo no MySQL e segundo pool de conexões sem `connection_limit`; hipótese de soma abaixo do limite em produção | CODIGO | src/config/database.ts |
| ADR-002-CONS-16 | `docs/adrs/ADR-002-worker-separado-com-polling.md` | Consequência | Consequência derivada: reprocessamento da DLQ também entrega fora de ordem (ver ADR-003) | TRANSCRICAO | [09:18] Diego |
| ADR-003 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Decisão | Retry com backoff 1m/5m/30m/2h/12h, teto de 5 tentativas (hipótese: envio inicial mais 5 retentativas), DLQ separada, reprocessamento ADMIN auditado | TRANSCRICAO | [09:17] Larissa |
| ADR-003-ALT-01 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Alternativa Descartada | Retry indefinido | TRANSCRICAO | [09:15] Diego |
| ADR-003-ALT-02 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Alternativa Descartada | 3 tentativas | TRANSCRICAO | [09:16] Diego |
| ADR-003-ALT-03 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Alternativa Descartada | Marcar "falhou" na própria outbox no lugar da DLQ | TRANSCRICAO | [09:18] Diego |
| ADR-003-CONS-01 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência | Janela cobre indisponibilidades de horas e foi aceita pelo produto | TRANSCRICAO | [09:17] Marcos |
| ADR-003-CONS-02 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência | Nenhum evento fica sendo reenviado para sempre | TRANSCRICAO | [09:15] Diego |
| ADR-003-CONS-03 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência | Outbox com leitura limpa; DLQ como evidência | TRANSCRICAO | [09:18] Diego |
| ADR-003-CONS-04 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência | Reprocessamento devolve o evento à entrega, com registro de quem o fez | TRANSCRICAO | [09:18] Diego |
| ADR-003-CONS-05 | `docs/adrs/ADR-003-retry-com-backoff-e-dlq.md` | Consequência | Hipótese: 5 tentativas são envio inicial mais 5 retentativas, única contagem que fecha os cinco intervalos (876 min); outra contagem na questão 1 | TRANSCRICAO | [09:17] Larissa |
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
| ADR-007-CONS-10 | `docs/adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md` | Consequência | Consequência derivada: efeito do estado ativo no filtro e destino das linhas de webhook desativado ou removido não definidos | TRANSCRICAO | [09:21] Bruno |
| ADR-007-CONS-11 | `docs/adrs/ADR-007-payload-snapshot-e-filtro-na-insercao.md` | Consequência | Consequência derivada: filtro pede índice pelo cliente; snapshot ocupa outbox e DLQ | TRANSCRICAO | [09:34] Bruno |
| FDD-OBJ-01 | `docs/FDD.md` | Objetivo Técnico | Linha da outbox existe se e somente se a mudança de status pelo `changeStatus` com webhook interessado foi confirmada, para payload de até 64 KB (ADR-001) | TRANSCRICAO | [09:06] Diego |
| FDD-OBJ-02 | `docs/FDD.md` | Objetivo Técnico | Primeiro envio até 2 s depois da gravação no caminho feliz, para a meta de notificação abaixo de 10 s (alcance em RFC-QA-07) | TRANSCRICAO | [09:10] Larissa |
| FDD-OBJ-03 | `docs/FDD.md` | Objetivo Técnico | Reentrega com backoff 1m/5m/30m/2h/12h até o teto de 5 tentativas (contagem em aberto), timeout HTTP de 10 s como falha e DLQ ao esgotar (ADR-003) | TRANSCRICAO | [09:17] Larissa |
| FDD-OBJ-04 | `docs/FDD.md` | Objetivo Técnico | Todo envio com HMAC-SHA256 do corpo no `X-Signature`, secret por endpoint (ADR-004) | TRANSCRICAO | [09:22] Sofia |
| FDD-OBJ-05 | `docs/FDD.md` | Objetivo Técnico | Mesmo identificador em `X-Event-Id` e `event_id` em todos os envios da linha (ADR-005) | TRANSCRICAO | [09:25] Diego |
| FDD-OBJ-06 | `docs/FDD.md` | Restrição | Só MySQL e stack existentes; reuso de módulo, erros, Zod, autenticação e Pino (ADR-006) | TRANSCRICAO | [09:30] Larissa |
| FDD-ESC-01 | `docs/FDD.md` | Escopo | Gravação do evento na outbox dentro do `changeStatus` por `publishWebhookEvent` | TRANSCRICAO | [09:40] Bruno |
| FDD-ESC-02 | `docs/FDD.md` | Escopo | Worker em processo próprio, `src/worker.ts` (novo) e `npm run worker` | TRANSCRICAO | [09:11] Larissa |
| FDD-ESC-03 | `docs/FDD.md` | Escopo | Retry com backoff, DLQ separada e reprocessamento por ADMIN com registro | TRANSCRICAO | [09:17] Larissa |
| FDD-ESC-04 | `docs/FDD.md` | Escopo | Assinatura HMAC-SHA256, secret por endpoint gerada pela plataforma, rotação com 24 h | TRANSCRICAO | [09:22] Sofia |
| FDD-ESC-05 | `docs/FDD.md` | Escopo | API de configuração (cadastrar, listar, editar, remover) e histórico de entregas | TRANSCRICAO | [09:33] Bruno |
| FDD-ESC-06 | `docs/FDD.md` | Escopo | Módulo `src/modules/webhooks/` (novo) na estrutura dos módulos existentes | TRANSCRICAO | [09:27] Bruno |
| FDD-ESC-07 | `docs/FDD.md` | Escopo | Excluído: webhooks de entrada | TRANSCRICAO | [09:02] Marcos |
| FDD-ESC-08 | `docs/FDD.md` | Escopo | Excluído: aviso por e-mail de falhas, fase futura, não é caminho de entrega | TRANSCRICAO | [09:37] Larissa |
| FDD-ESC-09 | `docs/FDD.md` | Escopo | Excluído: painel visual, projeto do time de frontend | TRANSCRICAO | [09:40] Larissa |
| FDD-ESC-10 | `docs/FDD.md` | Escopo | Excluído: rate limiting de envios, em observação (RFC-QA-02) | TRANSCRICAO | [09:39] Larissa |
| FDD-ESC-11 | `docs/FDD.md` | Escopo | Excluído: arquivamento das linhas entregues, sem prazo | TRANSCRICAO | [09:08] Diego |
| FDD-ESC-12 | `docs/FDD.md` | Escopo | Excluído: vários workers ao mesmo tempo (RFC-QA-03) | TRANSCRICAO | [09:13] Diego |
| FDD-ESC-13 | `docs/FDD.md` | Escopo | Excluído: permissões mais restritas no CRUD (RFC-QA-04) | TRANSCRICAO | [09:37] Sofia |
| FDD-ESC-14 | `docs/FDD.md` | Escopo | Proposta: criação do pedido não gera evento nesta fase (questão 1 do ADR-001) | TRANSCRICAO | [09:40] Bruno |
| FDD-ESC-15 | `docs/FDD.md` | Escopo | Proposta: mudanças de status fora do `changeStatus` (seed, SQL) não geram evento e a regra é documentada (questão 2 do ADR-001) | TRANSCRICAO | [09:40] Bruno |
| FDD-DADOS-01 | `docs/FDD.md` | Modelo de Dados | Tabela `webhook_endpoints` com url, secret, cliente, estado ativo, dados de rotação e lista de status em `Json`; proposta de remoção lógica | TRANSCRICAO | [09:21] Bruno |
| FDD-DADOS-02 | `docs/FDD.md` | Modelo de Dados | Tabela `webhook_outbox` com UUID, `eventId`, snapshot, estado, índices em status e criação; proposta de uma linha por webhook, envios, próximo envio e entrada em processando | TRANSCRICAO | [09:08] Diego |
| FDD-DADOS-03 | `docs/FDD.md` | Modelo de Dados | Proposta: tabela `webhook_deliveries`, uma linha por envio, com payload, status HTTP, corpo truncado, tempo e resultado | TRANSCRICAO | [09:34] Marcos |
| FDD-DADOS-04 | `docs/FDD.md` | Modelo de Dados | Tabela `webhook_dead_letter` com payload, motivo e timestamp; proposta de `eventId`, envios e dados do reprocessamento | TRANSCRICAO | [09:18] Diego |
| FDD-DADOS-05 | `docs/FDD.md` | Modelo de Dados | Proposta: enum `WebhookOutboxStatus` com pendente, processando, falhou (aguarda próximo envio) e entregue, e transições | TRANSCRICAO | [09:08] Diego |
| FDD-DADOS-06 | `docs/FDD.md` | Modelo de Dados | Proposta: sketch Prisma dos quatro modelos e do enum, no padrão do schema atual, em migração nova | CODIGO | prisma/schema.prisma |
| FDD-FLUXO-01 | `docs/FDD.md` | Fluxo | Criação do evento no `changeStatus`: filtro por status, snapshot, `eventId` por `uuid`, uma linha por webhook, rollback em erro | TRANSCRICAO | [09:40] Bruno |
| FDD-FLUXO-02 | `docs/FDD.md` | Fluxo | Proposta: payload acima de 64 KB vai direto para a DLQ sem lançar erro (questão 2 do ADR-007) | TRANSCRICAO | [09:24] Larissa |
| FDD-FLUXO-03 | `docs/FDD.md` | Fluxo | Ciclo do worker: entry-point, `PrismaClient` próprio, linhas presas, lote com `FOR UPDATE SKIP LOCKED` via `$queryRaw`, envio sequencial com `fetch` e timeout de 10 s, desligamento | TRANSCRICAO | [09:10] Larissa |
| FDD-FLUXO-04 | `docs/FDD.md` | Fluxo | Proposta: 6 envios (876 min) pela hipótese do ADR-003-CONS-05; tabela dos envios, cálculo do próximo envio, o que conta como falha | TRANSCRICAO | [09:17] Larissa |
| FDD-FLUXO-05 | `docs/FDD.md` | Fluxo | Proposta: envios esgotados copiam a linha para a DLQ e a removem da outbox na mesma transação, com o motivo | TRANSCRICAO | [09:18] Diego |
| FDD-FLUXO-06 | `docs/FDD.md` | Fluxo | Reprocessamento por ADMIN em `POST /api/v1/admin/webhooks/dead-letter/:id/replay`, com log de auditoria; proposta de mesmo `eventId` e recusa de segundo reprocessamento | TRANSCRICAO | [09:35] Diego |
| FDD-FLUXO-07 | `docs/FDD.md` | Fluxo | Rotação: secret nova por `crypto.randomBytes`, anterior válida por 24 h, proposta de dois valores no `X-Signature` e de nova rotação dentro das 24 h (questão 6 do ADR-004) | TRANSCRICAO | [09:21] Sofia |
| FDD-CONTRATO-01 | `docs/FDD.md` | Contrato | `POST /api/v1/webhooks` (caminho proposto): cadastro com URL https, status e `customerId` no body (proposta, RFC-QA-01); devolve a secret | TRANSCRICAO | [09:31] Marcos |
| FDD-CONTRATO-02 | `docs/FDD.md` | Contrato | `GET /api/v1/webhooks?customerId=` (caminho e filtro propostos, RFC-QA-01): listagem paginada por cliente, sem secret | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-03 | `docs/FDD.md` | Contrato | `PATCH /api/v1/webhooks/:id` (caminho proposto): edição parcial de URL, status e estado ativo | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-04 | `docs/FDD.md` | Contrato | `DELETE /api/v1/webhooks/:id` (caminho proposto): proposta de remoção lógica, `204` | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-05 | `docs/FDD.md` | Contrato | Proposta: `POST /api/v1/webhooks/:id/secret/rotate` devolve a secret nova e a expiração da anterior | TRANSCRICAO | [09:21] Sofia |
| FDD-CONTRATO-06 | `docs/FDD.md` | Contrato | `GET /api/v1/webhooks/:id/deliveries`: últimos 100 envios, sem paginação (proposta); histórico de webhook removido consultável (proposta) | TRANSCRICAO | [09:34] Marcos |
| FDD-CONTRATO-07 | `docs/FDD.md` | Contrato | `POST /api/v1/admin/webhooks/dead-letter/:id/replay` com `requireRole('ADMIN')` | TRANSCRICAO | [09:36] Larissa |
| FDD-CONTRATO-08 | `docs/FDD.md` | Contrato | Request de entrega: headers `Content-Type`, `X-Event-Id`, `X-Signature`, `X-Timestamp`, `X-Webhook-Id`, payload e verificação no cliente | TRANSCRICAO | [09:44] Diego |
| FDD-CONTRATO-09 | `docs/FDD.md` | Contrato | `publishWebhookEvent(tx, order, fromStatus, toStatus)` com `Prisma.TransactionClient`, chamada no `changeStatus` | TRANSCRICAO | [09:41] Bruno |
| FDD-ERRO-01 | `docs/FDD.md` | Erro | Proposta: `WEBHOOK_VALIDATION_ERROR` (400, `BadRequestError` pelo `validate` com código padrão) para erros de schema fora de `url` e `statuses` (RFC-QA-12) | TRANSCRICAO | [09:29] Larissa |
| FDD-ERRO-02 | `docs/FDD.md` | Erro | `WEBHOOK_INVALID_URL` (400, `BadRequestError`) para URL inválida ou sem https; proposta: lançado pelo `validate` com código do campo `url` | TRANSCRICAO | [09:28] Bruno |
| FDD-ERRO-03 | `docs/FDD.md` | Erro | Proposta: `WEBHOOK_INVALID_EVENT_FILTER` (400) para `statuses` vazio, repetido ou fora de `OrderStatus` | TRANSCRICAO | [09:33] Marcos |
| FDD-ERRO-04 | `docs/FDD.md` | Erro | `WEBHOOK_NOT_FOUND` (404) para webhook inexistente; proposta: subclasse de `AppError` em `webhook.errors.ts` (novo) e webhook removido também | TRANSCRICAO | [09:28] Bruno |
| FDD-ERRO-05 | `docs/FDD.md` | Erro | Proposta: `WEBHOOK_CUSTOMER_NOT_FOUND` (404) para `customerId` sem cliente no cadastro | TRANSCRICAO | [09:32] Larissa |
| FDD-ERRO-06 | `docs/FDD.md` | Erro | Proposta: `WEBHOOK_DEAD_LETTER_NOT_FOUND` (404) para entrada da DLQ inexistente no reprocessamento | TRANSCRICAO | [09:18] Diego |
| FDD-ERRO-07 | `docs/FDD.md` | Erro | Proposta: `WEBHOOK_DEAD_LETTER_ALREADY_REPLAYED` (409, `ConflictError`) para entrada já reprocessada (RFC-QA-11) | TRANSCRICAO | [09:18] Diego |
| FDD-ERRO-08 | `docs/FDD.md` | Erro | Proposta: `WEBHOOK_PAYLOAD_TOO_LARGE` (422, `UnprocessableEntityError`) no reprocessamento de entrada gravada por payload acima de 64 KB | TRANSCRICAO | [09:24] Larissa |
| FDD-ERRO-09 | `docs/FDD.md` | Erro | Proposta: motivo gravado `WEBHOOK_DELIVERY_TIMEOUT` para envio sem resposta em 10 s, no formato `CÓDIGO: detalhe` | TRANSCRICAO | [09:42] Diego |
| FDD-ERRO-10 | `docs/FDD.md` | Erro | Proposta: motivo gravado `WEBHOOK_DELIVERY_HTTP_ERROR` para resposta fora de 2xx | TRANSCRICAO | [09:42] Diego |
| FDD-ERRO-11 | `docs/FDD.md` | Erro | Proposta: motivo gravado `WEBHOOK_DELIVERY_NETWORK_ERROR` para DNS, conexão recusada ou TLS | TRANSCRICAO | [09:42] Diego |
| FDD-ERRO-12 | `docs/FDD.md` | Erro | Proposta: `WEBHOOK_SECRET_REQUIRED` como motivo gravado pelo worker sem secret utilizável, tentativa sem chamada HTTP; nunca resposta da API; alternativa de descartar (questão 2 do ADR-006) | TRANSCRICAO | [09:28] Bruno |
| FDD-ERRO-13 | `docs/FDD.md` | Erro | Proposta: motivo `WEBHOOK_PAYLOAD_TOO_LARGE` gravado por `publishWebhookEvent` na DLQ, com `attempts` zero | TRANSCRICAO | [09:24] Larissa |
| FDD-RES-01 | `docs/FDD.md` | Resiliência | Timeout HTTP de 10 s com `AbortSignal.timeout`, contado como falha; distinto da meta de 10 s | TRANSCRICAO | [09:42] Diego |
| FDD-RES-02 | `docs/FDD.md` | Resiliência | Retry com backoff 1m/5m/30m/2h/12h; proposta de 6 envios (RFC-QA-08) e falha = fora de 2xx, rede ou timeout | TRANSCRICAO | [09:17] Larissa |
| FDD-RES-03 | `docs/FDD.md` | Resiliência | DLQ ao esgotar os envios; proposta de cópia e remoção na mesma transação e falhou = aguarda próximo envio | TRANSCRICAO | [09:18] Diego |
| FDD-RES-04 | `docs/FDD.md` | Resiliência | Reprocessamento por ADMIN devolve à outbox como pendente; proposta de mesmo `eventId`, snapshot como está e recusa do segundo | TRANSCRICAO | [09:18] Diego |
| FDD-RES-05 | `docs/FDD.md` | Resiliência | Proposta: linha em processando há mais de 5 min volta a pendente sem contar envio (questão 7 do ADR-002) | CODIGO | src/server.ts |
| FDD-RES-06 | `docs/FDD.md` | Resiliência | Proposta: desligamento espera o ciclo em andamento e desconecta (questão 8 do ADR-002) | CODIGO | src/server.ts |
| FDD-RES-07 | `docs/FDD.md` | Resiliência | Proposta: erro no ciclo loga `webhook_cycle_failed` e não encerra; falha de conexão no início encerra com `bootstrap_failed` | CODIGO | src/server.ts |
| FDD-RES-08 | `docs/FDD.md` | Resiliência | Invariante (a): linha gravada se e somente se a mudança pelo `changeStatus` foi confirmada com webhook interessado, com as exceções de 64 KB, fora do `changeStatus` e reprocessamento | TRANSCRICAO | [09:40] Bruno |
| FDD-RES-09 | `docs/FDD.md` | Resiliência | Invariante (b): nenhuma linha sai da outbox sem entrega ou cópia para a DLQ na mesma transação; entregues ficam (sem arquivamento) | TRANSCRICAO | [09:18] Diego |
| FDD-RES-10 | `docs/FDD.md` | Resiliência | Invariante (c): todos os envios de uma linha com o mesmo `eventId` e o mesmo corpo | TRANSCRICAO | [09:25] Diego |
| FDD-RES-11 | `docs/FDD.md` | Resiliência | Fallback atual = DLQ e reprocessamento manual; e-mail é fase futura e não é fallback | TRANSCRICAO | [09:37] Larissa |
| FDD-RES-12 | `docs/FDD.md` | Resiliência | Proposta: envio de uma linha de cada vez faz endpoint lento segurar o ciclo até 10 s por linha (RFC-QA-06) | TRANSCRICAO | [09:42] Diego |
| FDD-OBS-01 | `docs/FDD.md` | Observabilidade | Proposta: eventos de log `webhook_*` em snake_case com campos em camelCase, quem loga e nível | CODIGO | src/server.ts |
| FDD-OBS-01b | `docs/FDD.md` | Observabilidade | Campos de log em camelCase, como `requestId` do request logger | CODIGO | src/middlewares/request-logger.middleware.ts |
| FDD-OBS-02 | `docs/FDD.md` | Observabilidade | Proposta: logs do worker por `logger.child({ component: 'webhook-worker' })`, sem alterar `base.service` | CODIGO | src/shared/logger/index.ts |
| FDD-OBS-03 | `docs/FDD.md` | Observabilidade | Proposta: `secret`, `previousSecret` e os mesmos caminhos com `*.` no `redactPaths`; nenhum log do cadastro completo nem dos headers (questão 3 do ADR-004) | TRANSCRICAO | [09:22] Diego |
| FDD-OBS-04 | `docs/FDD.md` | Observabilidade | Auditoria do reprocessamento com o usuário; proposta: log `webhook_dead_letter_replayed` | TRANSCRICAO | [09:36] Sofia |
| FDD-OBS-05 | `docs/FDD.md` | Observabilidade | Proposta: `publishWebhookEvent` não loga dentro da transação; entrada de 64 KB percebida pela consulta da DLQ | TRANSCRICAO | [09:40] Bruno |
| FDD-OBS-06 | `docs/FDD.md` | Observabilidade | Proposta: métricas por consultas SQL às tabelas novas e contagens do log do ciclo, sem biblioteca de métricas | TRANSCRICAO | [09:34] Marcos |
| FDD-OBS-07 | `docs/FDD.md` | Observabilidade | Proposta: biblioteca de métricas fora desta fase, proposta futura | TRANSCRICAO | [09:30] Larissa |
| FDD-OBS-08 | `docs/FDD.md` | Observabilidade | Correlação pelo `eventId` (tabelas, logs, `X-Event-Id`) e pelo `orderId` com o log `http_request` | TRANSCRICAO | [09:25] Diego |
| FDD-OBS-09 | `docs/FDD.md` | Observabilidade | Proposta: `requestId` não gravado na outbox para não mudar a assinatura de `publishWebhookEvent` | TRANSCRICAO | [09:41] Bruno |
| FDD-OBS-10 | `docs/FDD.md` | Observabilidade | Proposta: tracing distribuído (OpenTelemetry) fora desta fase | CODIGO | package.json |
| FDD-OBS-11 | `docs/FDD.md` | Observabilidade | Proposta: worker parado detectado pelo log `webhook_cycle_completed` e idade da linha elegível com limiar de 1 min (questão 6 do ADR-002) | TRANSCRICAO | [09:10] Larissa |
| FDD-OBS-12 | `docs/FDD.md` | Observabilidade | Proposta: DLQ acompanhada pelo log `webhook_dead_lettered` e pela consulta das entradas sem reprocessamento (questão 4 do ADR-003) | TRANSCRICAO | [09:18] Diego |
| FDD-DEP-01 | `docs/FDD.md` | Dependência | Node >=20; hipótese: `fetch`, `AbortSignal.timeout` e `crypto` (`createHmac`, `randomBytes`, `timingSafeEqual`) nativos, sem dependência nova, a confirmar em teste | CODIGO | package.json |
| FDD-DEP-02 | `docs/FDD.md` | Dependência | MySQL que o projeto já usa; `SKIP LOCKED` como hipótese a confirmar em teste | TRANSCRICAO | [09:07] Diego |
| FDD-DEP-02b | `docs/FDD.md` | Dependência | MySQL 8.0, imagem `mysql:8.0` do `docker-compose.yml` | CODIGO | docker-compose.yml |
| FDD-DEP-03 | `docs/FDD.md` | Dependência | Prisma 5.22.0: `$queryRaw` na transação (hipótese), migração nova, `PrismaClient` próprio no worker com a mesma `DATABASE_URL` | TRANSCRICAO | [09:30] Bruno |
| FDD-DEP-03b | `docs/FDD.md` | Dependência | Prisma 5.22.0 e migração por `npm run db:migrate` (`prisma migrate dev`) | CODIGO | package.json |
| FDD-DEP-04 | `docs/FDD.md` | Dependência | `uuid` 11.0.3, `zod` 3.23.8 e `pino` 9.5.0 já declarados | TRANSCRICAO | [09:25] Diego |
| FDD-DEP-05 | `docs/FDD.md` | Dependência | Nenhuma dependência nova | TRANSCRICAO | [09:30] Larissa |
| FDD-DEP-06 | `docs/FDD.md` | Dependência | Worker importa `src/config/env.ts` e exige as mesmas variáveis da API, inclusive `JWT_SECRET` | CODIGO | src/config/env.ts |
| FDD-DEP-07 | `docs/FDD.md` | Dependência | Segundo pool de conexões; `DATABASE_URL` sem `connection_limit`; hipótese de soma abaixo do limite do MySQL de produção | CODIGO | .env.example |
| FDD-DEP-08 | `docs/FDD.md` | Dependência | Proposta: sem mudança nesta fase; hipótese: exclusão de cliente com webhook falha pela FK `ON DELETE RESTRICT` e responde 500 | CODIGO | prisma/migrations/20260519182739_init/migration.sql |
| FDD-DEP-09 | `docs/FDD.md` | Dependência | Proposta: parâmetros como constantes em `webhook.config.ts` (novo), sem variável de ambiente nova | CODIGO | src/config/env.ts |
| FDD-DEP-10 | `docs/FDD.md` | Dependência | Compatibilidade: nenhuma rota existente muda; `PATCH /api/v1/orders/:id/status` ganha leitura e gravação na transação; proposta: exclusão de cliente com webhook passa a falhar | TRANSCRICAO | [09:40] Bruno |
| FDD-INT-01 | `docs/FDD.md` | Integração | `order.service.ts`: `changeStatus` chama `publishWebhookEvent` depois do histórico e antes da releitura | TRANSCRICAO | [09:40] Bruno |
| FDD-INT-01b | `docs/FDD.md` | Integração | `order.service.ts`: ponto de chamada dentro do callback de `$transaction` | CODIGO | src/modules/orders/order.service.ts |
| FDD-INT-02 | `docs/FDD.md` | Integração | `order.status.ts` sem alteração; define os `to_status` que geram evento | CODIGO | src/modules/orders/order.status.ts |
| FDD-INT-03 | `docs/FDD.md` | Integração | `app-error.ts` sem alteração; base das subclasses do módulo | TRANSCRICAO | [09:28] Bruno |
| FDD-INT-04 | `docs/FDD.md` | Integração | `http-errors.ts` sem alteração; classes que aceitam código reaproveitadas e subclasses próprias para 404 | CODIGO | src/shared/errors/http-errors.ts |
| FDD-INT-05 | `docs/FDD.md` | Integração | Proposta: `validate.middleware.ts` ganha parâmetro opcional com código padrão e por campo | TRANSCRICAO | [09:29] Larissa |
| FDD-INT-06 | `docs/FDD.md` | Integração | `auth.middleware.ts` sem alteração; `authenticate` no módulo e `requireRole('ADMIN')` no reprocessamento | TRANSCRICAO | [09:36] Larissa |
| FDD-INT-06b | `docs/FDD.md` | Integração | `auth.middleware.ts`: `requireRole` existente reaproveitado | CODIGO | src/middlewares/auth.middleware.ts |
| FDD-INT-07 | `docs/FDD.md` | Integração | `error.middleware.ts` sem alteração; não atua no worker | TRANSCRICAO | [09:29] Bruno |
| FDD-INT-08 | `docs/FDD.md` | Integração | `request-logger.middleware.ts` sem alteração; correlação pelo `http_request` | CODIGO | src/middlewares/request-logger.middleware.ts |
| FDD-INT-09 | `docs/FDD.md` | Integração | Proposta: `logger/index.ts` com os caminhos de `secret` e `previousSecret` no `redactPaths`; worker por `logger.child` | TRANSCRICAO | [09:22] Diego |
| FDD-INT-10 | `docs/FDD.md` | Integração | `server.ts` sem alteração; modelo do `src/worker.ts` (novo) | TRANSCRICAO | [09:11] Larissa |
| FDD-INT-10b | `docs/FDD.md` | Integração | `server.ts`: `bootstrap`, sinais e `bootstrap_failed` como modelo | CODIGO | src/server.ts |
| FDD-INT-11 | `docs/FDD.md` | Integração | Proposta: `app.ts` monta repository, service e controller do módulo | TRANSCRICAO | [09:27] Bruno |
| FDD-INT-11b | `docs/FDD.md` | Integração | `app.ts`: `buildControllers` como ponto de montagem | CODIGO | src/app.ts |
| FDD-INT-12 | `docs/FDD.md` | Integração | Proposta: `routes/index.ts` monta `/webhooks` e `/admin/webhooks` | TRANSCRICAO | [09:35] Diego |
| FDD-INT-13 | `docs/FDD.md` | Integração | `database.ts` sem alteração; proposta: worker importa o `prisma`, que ganha instância própria no seu processo | TRANSCRICAO | [09:30] Bruno |
| FDD-INT-13b | `docs/FDD.md` | Integração | `database.ts`: `prisma` criado na carga do módulo | CODIGO | src/config/database.ts |
| FDD-INT-14 | `docs/FDD.md` | Integração | `env.ts` sem alteração; worker exige as mesmas variáveis | CODIGO | src/config/env.ts |
| FDD-INT-15 | `docs/FDD.md` | Integração | `schema.prisma` ganha o enum, os quatro modelos e o campo de relação em `Customer` | TRANSCRICAO | [09:08] Diego |
| FDD-INT-16 | `docs/FDD.md` | Integração | `package.json` ganha o script `worker` | TRANSCRICAO | [09:11] Larissa |
| FDD-INT-17 | `docs/FDD.md` | Integração | Proposta: `tests/setup.ts` apaga as tabelas novas antes de `customer` | CODIGO | tests/setup.ts |
| FDD-INT-18 | `docs/FDD.md` | Integração | `tests/orders.test.ts` sem alteração; padrão dos testes novos | CODIGO | tests/orders.test.ts |
| FDD-INT-19 | `docs/FDD.md` | Integração | Proposta: `factories.ts` ganha `createTestWebhook` gravando direto no banco | CODIGO | tests/helpers/factories.ts |
| FDD-INT-20 | `docs/FDD.md` | Integração | Proposta: `customer.service.ts` sem alteração nesta fase; exclusão de cliente com webhook responde 500 | CODIGO | src/modules/customers/customer.service.ts |
| FDD-INT-21 | `docs/FDD.md` | Integração | Proposta: `prisma/seed.ts` apaga as tabelas novas antes de `customer` | CODIGO | prisma/seed.ts |
| FDD-CA-01 | `docs/FDD.md` | Critério de Aceitação | Mudança pelo `changeStatus` com webhook interessado grava linha pendente; proposta: uma por webhook, com o mesmo `eventId` | TRANSCRICAO | [09:40] Bruno |
| FDD-CA-02 | `docs/FDD.md` | Critério de Aceitação | Falha na gravação da outbox mantém o status antigo e nenhuma linha; confirma a hipótese do rollback | TRANSCRICAO | [09:40] Bruno |
| FDD-CA-03 | `docs/FDD.md` | Critério de Aceitação | Status que nenhum webhook do cliente quer não grava nada; proposta: sem linha para webhook que não quer, inativo ou removido | TRANSCRICAO | [09:34] Bruno |
| FDD-CA-04 | `docs/FDD.md` | Critério de Aceitação | Primeiro envio até 2 s depois da gravação, mais o tempo do envio, no caminho feliz | TRANSCRICAO | [09:10] Larissa |
| FDD-CA-05 | `docs/FDD.md` | Critério de Aceitação | Sem resposta em 10 s, o envio falha; proposta: motivo `WEBHOOK_DELIVERY_TIMEOUT`, estado falhou e próximo envio em 1 min | TRANSCRICAO | [09:42] Diego |
| FDD-CA-06 | `docs/FDD.md` | Critério de Aceitação | Proposta: intervalos da seção 5.3 contados do fim do envio, com 6 envios e falha do envio 6 levando à DLQ | TRANSCRICAO | [09:17] Larissa |
| FDD-CA-07 | `docs/FDD.md` | Critério de Aceitação | Envios esgotados: entrada na DLQ; proposta: linha fora da outbox | TRANSCRICAO | [09:18] Diego |
| FDD-CA-08 | `docs/FDD.md` | Critério de Aceitação | `X-Signature` verificável com a secret do endpoint; proposta: dois valores nas 24 h após rotação | TRANSCRICAO | [09:22] Sofia |
| FDD-CA-09 | `docs/FDD.md` | Critério de Aceitação | `X-Event-Id` igual ao `event_id` e estável entre envios da linha | TRANSCRICAO | [09:25] Diego |
| FDD-CA-10 | `docs/FDD.md` | Critério de Aceitação | Proposta: payload acima de 64 KB não desfaz a mudança de status e vai para a DLQ com `attempts` zero | TRANSCRICAO | [09:24] Larissa |
| FDD-CA-11 | `docs/FDD.md` | Critério de Aceitação | URL `http` recusada com 400; código `WEBHOOK_INVALID_URL` na proposta da seção 7 (RFC-QA-12) | TRANSCRICAO | [09:23] Sofia |
| FDD-CA-12 | `docs/FDD.md` | Critério de Aceitação | Proposta: `statuses` inválido recusado com 400 `WEBHOOK_INVALID_EVENT_FILTER` | TRANSCRICAO | [09:33] Marcos |
| FDD-CA-13 | `docs/FDD.md` | Critério de Aceitação | Reprocessamento sem ADMIN recusado com 403; com ADMIN registra o usuário em log; proposta: 200 e evento `webhook_dead_letter_replayed` | TRANSCRICAO | [09:36] Larissa |
| FDD-CA-14 | `docs/FDD.md` | Critério de Aceitação | Proposta: reprocessamento mantém `eventId` e payload; segundo reprocessamento recusado com 409 | TRANSCRICAO | [09:18] Diego |
| FDD-CA-15 | `docs/FDD.md` | Critério de Aceitação | Proposta: secret só nas respostas de cadastro e rotação | TRANSCRICAO | [09:31] Marcos |
| FDD-CA-16 | `docs/FDD.md` | Critério de Aceitação | Proposta: secret nunca em log; chave `secret` sai como `[REDACTED]` | TRANSCRICAO | [09:22] Diego |
| FDD-CA-17 | `docs/FDD.md` | Critério de Aceitação | Histórico devolve no máximo 100 envios, do mais recente para o mais antigo (ordem proposta) | TRANSCRICAO | [09:34] Marcos |
| FDD-CA-18 | `docs/FDD.md` | Critério de Aceitação | Proposta: linha presa há mais de 5 min volta a pendente sem contar envio | CODIGO | src/server.ts |
| FDD-CA-19 | `docs/FDD.md` | Critério de Aceitação | `npm test` continua passando, com os testes existentes sem alteração e a limpeza de `tests/setup.ts` ajustada | CODIGO | package.json |
| FDD-RISCO-01 | `docs/FDD.md` | Risco | Endpoint lento segura o ciclo com o envio de uma linha de cada vez | TRANSCRICAO | [09:42] Diego |
| FDD-RISCO-02 | `docs/FDD.md` | Risco | Linha presa em processando após encerramento no meio do ciclo | CODIGO | src/server.ts |
| FDD-RISCO-03 | `docs/FDD.md` | Risco | Dois workers rodando, premissa sem garantia (RFC-RISCO-04) | TRANSCRICAO | [09:12] Diego |
| FDD-RISCO-04 | `docs/FDD.md` | Risco | Worker parado sem ninguém perceber (sem health check no worker) | CODIGO | src/app.ts |
| FDD-RISCO-05 | `docs/FDD.md` | Risco | Vazamento da secret por log, banco ou cliente | TRANSCRICAO | [09:22] Diego |
| FDD-RISCO-06 | `docs/FDD.md` | Risco | Crescimento sem retenção de outbox, histórico e DLQ; arquivamento sem prazo | TRANSCRICAO | [09:08] Diego |
| FDD-RISCO-07 | `docs/FDD.md` | Risco | Cliente que não deduplica pelo `X-Event-Id` (parte técnica; risco de produto no PRD-RISCO-02) | TRANSCRICAO | [09:25] Diego |
| FDD-RISCO-08 | `docs/FDD.md` | Risco | Perda de ordem por pedido no retry e no reprocessamento (RFC-QA-05) | TRANSCRICAO | [09:17] Larissa |
| FDD-RISCO-09 | `docs/FDD.md` | Risco | Registro público que aceita ADMIN enfraquece `requireRole` no reprocessamento | CODIGO | src/modules/auth/auth.schemas.ts |
| FDD-RISCO-10 | `docs/FDD.md` | Risco | Proposta: evento acima de 64 KB fora do at-least-once | TRANSCRICAO | [09:24] Larissa |
| FDD-RISCO-11 | `docs/FDD.md` | Risco | Transação do `changeStatus` mais longa com leitura de webhooks e gravação na outbox | TRANSCRICAO | [09:40] Bruno |
| FDD-RISCO-12 | `docs/FDD.md` | Risco | Proposta: exclusão de cliente com webhook responde 500 (hipótese do FDD-DEP-08) | CODIGO | src/middlewares/error.middleware.ts |
| FDD-PARAM-01 | `docs/FDD.md` | Parâmetro | Polling de 2 s (decidido); proposta: contado a partir do fim do ciclo | TRANSCRICAO | [09:10] Larissa |
| FDD-PARAM-02 | `docs/FDD.md` | Parâmetro | 1 worker (decidido) | TRANSCRICAO | [09:12] Diego |
| FDD-PARAM-03 | `docs/FDD.md` | Parâmetro | Proposta: 10 linhas por ciclo ("batch pequeno") | TRANSCRICAO | [09:08] Diego |
| FDD-PARAM-04 | `docs/FDD.md` | Parâmetro | Proposta: limite de 5 min em processando | CODIGO | src/server.ts |
| FDD-PARAM-05 | `docs/FDD.md` | Parâmetro | Timeout HTTP de 10 s (decidido) | TRANSCRICAO | [09:42] Diego |
| FDD-PARAM-06 | `docs/FDD.md` | Parâmetro | Proposta: teto de 6 envios por linha (RFC-QA-08) | TRANSCRICAO | [09:17] Larissa |
| FDD-PARAM-07 | `docs/FDD.md` | Parâmetro | Intervalos 1 min, 5 min, 30 min, 2 h, 12 h (decidido) | TRANSCRICAO | [09:17] Larissa |
| FDD-PARAM-08 | `docs/FDD.md` | Parâmetro | Limite de 64 KB (decidido); proposta: medido em bytes do JSON serializado | TRANSCRICAO | [09:24] Larissa |
| FDD-PARAM-09 | `docs/FDD.md` | Parâmetro | Convivência de 24 h da secret anterior (decidido) | TRANSCRICAO | [09:22] Sofia |
| FDD-PARAM-10 | `docs/FDD.md` | Parâmetro | Proposta: secret de 32 bytes em hexadecimal | CODIGO | package.json |
| FDD-PARAM-11 | `docs/FDD.md` | Parâmetro | 100 envios no histórico (decidido) | TRANSCRICAO | [09:34] Marcos |
| FDD-PARAM-12 | `docs/FDD.md` | Parâmetro | Proposta: corpo da resposta truncado em 4 KB | TRANSCRICAO | [09:34] Marcos |
| FDD-PARAM-13 | `docs/FDD.md` | Parâmetro | Proposta: `pageSize` padrão 20, como `listOrdersQuerySchema` | CODIGO | src/modules/orders/order.schemas.ts |
| FDD-PARAM-14 | `docs/FDD.md` | Parâmetro | Proposta: `pageSize` máximo 100, como `listOrdersQuerySchema` | CODIGO | src/modules/orders/order.schemas.ts |
| FDD-PARAM-15 | `docs/FDD.md` | Parâmetro | Proposta: limiar de 1 min para worker parado (questão 6 do ADR-002) | TRANSCRICAO | [09:10] Larissa |
