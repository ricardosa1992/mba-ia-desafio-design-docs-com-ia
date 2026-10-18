# ADR-006: Reuso dos padrões do projeto no módulo de webhooks

## Status

**Aceito**, decidido na reunião técnica de quinta-feira (redigido em 2026-10-09, revisado em 2026-10-18).

> Aceito quanto ao webhook como módulo da base de código existente, ao reuso das capacidades transversais que a API já tem e ao formato dos códigos de erro. Se todos os erros do módulo terão de fato o prefixo `WEBHOOK_` depende da questão 1.

- **Decisores**: Bruno (Engenheiro Pleno, Pedidos), Larissa (Tech Lead).
- **Consultados**: Diego (Engenheiro Sênior, Plataforma).

## Contexto

O módulo de webhooks entra em uma base que já tem padrão claro: cada domínio é um módulo em `src/modules` com controller, service, repository, routes e schemas. O padrão tem exceções: `src/modules/auth/` não tem repository (usa `UserRepository`) e `src/modules/orders/` tem o arquivo extra `order.status.ts`. As capacidades transversais ficam fora dos módulos:

| Capacidade | Onde está hoje |
| --- | --- |
| Erro base e classes de erro HTTP | `src/shared/errors/app-error.ts` (`AppError`), `src/shared/errors/http-errors.ts` |
| Formato de resposta de erro | `src/middlewares/error.middleware.ts` (`errorMiddleware`) |
| Validação de entrada | `src/middlewares/validate.middleware.ts` (`validate`), com schemas Zod como `src/modules/orders/order.schemas.ts` |
| Autenticação e controle de papel | `src/middlewares/auth.middleware.ts` (`authenticate`, `requireRole`) |
| Log | `src/shared/logger/index.ts` (`logger`, Pino) |

Neste ADR, "código de erro" é o identificador em texto da resposta de erro (campo `code`, como `INVALID_STATUS_TRANSITION`); "status HTTP" é o número da resposta (404, 409). "Módulo" é uma pasta de domínio em `src/modules`.

## Fatores de decisão

- Reuso máximo do que já existe.
- Nenhum logger novo.
- Seguir o padrão de módulos que a base já tem.
- Time pequeno, sem infraestrutura nova ([ADR-001](ADR-001-outbox-transacional-no-mysql.md)).

## Decisão

- O webhook é um módulo da mesma base de código: suas rotas rodam no processo da API e seu worker é um segundo processo da mesma base ([ADR-002](ADR-002-worker-separado-com-polling.md)). Não é um serviço separado.
- O módulo segue a estrutura dos módulos existentes e reaproveita as capacidades transversais da tabela acima: erro base e classes de erro, formato de resposta de erro, validação por schema Zod, autenticação e controle de papel, e o logger Pino. Nenhuma biblioteca nova entra para essas funções.
- Os códigos de erro do módulo seguem o formato existente (letras maiúsculas separadas por sublinhado) com o prefixo `WEBHOOK_`. Os exemplos citados foram `WEBHOOK_NOT_FOUND`, `WEBHOOK_INVALID_URL` e `WEBHOOK_SECRET_REQUIRED`. Hoje, os erros que saem dos caminhos reaproveitados (validação de schema, recurso não encontrado, autenticação e papel, erros do banco tratados pelo middleware) têm códigos genéricos fixos (CONS-04); como o prefixo chega a esses casos está em aberto (questão 1).

Pasta e arquivos do módulo, registro das rotas, tipo de id das tabelas novas e matriz de códigos estão no FDD, seções 4 Modelo de dados, 7. Matriz de erros previstos e 11. Integração com o sistema existente.

## Alternativas Consideradas

### ADR-006-ALT-03: Logger próprio do módulo

- **Descrição**: trazer um logger novo para o módulo ou para o worker.
- **Prós**: **Derivada:** liberdade para escolher uma ferramenta de log específica para o worker.
- **Contras**: o projeto já tem o Pino como logger.
- **Motivo do descarte**: Pino já é o logger do projeto e nada novo entra.

### ADR-006-ALT-04: Webhooks como serviço separado

- **Descrição**: **Hipótese:** alternativa plausível, não discutida na reunião. O envio de webhooks seria um serviço próprio, com base de código, implantação e acesso a dados separados da API de pedidos.
- **Prós**: **Derivada:** isola falhas e carga do envio de webhooks da API de pedidos e permite implantar cada um no seu ritmo.
- **Contras**: **Derivada:** exige infraestrutura e operação novas, e a outbox precisaria continuar sendo gravada na transação do `changeStatus`, dentro da API, o que mantém o acoplamento pelo banco.
- **Motivo do descarte**: contraria o reuso máximo e a regra de não subir infraestrutura nova; o webhook fica como módulo igual aos outros.

## Consequências

### Positivas

- **ADR-006-CONS-01**: Os erros das rotas do módulo saem no mesmo formato de resposta das demais rotas, pelo `errorMiddleware`, sem mudança nele.
- **ADR-006-CONS-02**: Nenhuma dependência nova para log, validação ou erros, nem para autorização, que reaproveita o controle de papel existente.
- **ADR-006-CONS-03**: **Derivada:** quem conhece os módulos atuais encontra os mesmos papéis no novo, porque a estrutura é a mesma.

### Negativas

#### Contrato de erro

- **ADR-006-CONS-04**: os caminhos reaproveitados têm códigos de erro fixos, sem prefixo: `NotFoundError` sai com `NOT_FOUND` e o `validate` sempre devolve `VALIDATION_ERROR` (`src/shared/errors/http-errors.ts`, `src/middlewares/validate.middleware.ts`); `authenticate` e `requireRole` lançam `UNAUTHORIZED` e `FORBIDDEN` (`src/middlewares/auth.middleware.ts`); o middleware de erro responde `CONFLICT`, `NOT_FOUND` e `INTERNAL_SERVER_ERROR` para erros do banco (`src/middlewares/error.middleware.ts`). Sem mudança, `WEBHOOK_NOT_FOUND` e `WEBHOOK_INVALID_URL` não saem desses caminhos, e "prefixo para tudo do módulo" não vale para esses códigos. Ver questão 1.
- **ADR-006-CONS-05**: O middleware de erro trata do Prisma só os códigos `P2002` e `P2025`; outro erro do banco numa rota do módulo vira status HTTP 500 com `INTERNAL_SERVER_ERROR` (`src/middlewares/error.middleware.ts`). Ele é um middleware Express e não atua no worker, que não tem HTTP de entrada. **Derivada:** o worker trata e loga os próprios erros.
- **ADR-006-CONS-06**: O prefixo `WEBHOOK_` é convenção nova: nenhum código de erro existente tem prefixo de módulo, como `EMAIL_ALREADY_USED` em `src/modules/users/user.service.ts`. O padrão reaproveitado é o formato, não o prefixo.
- **ADR-006-CONS-12**: **Derivada:** o código `WEBHOOK_SECRET_REQUIRED` sugere uma secret informada pelo cliente, mas a secret é gerada pela plataforma ([ADR-004](ADR-004-assinatura-hmac-sha256-por-endpoint.md)). O caso em que esse código seria usado não está definido. Ver questão 2.

#### Acoplamento e operação

- **ADR-006-CONS-09**: **Derivada:** o Pino é o logger do projeto, mas hoje só `src/server.ts` e dois middlewares o usam; services não logam e `pino-http` está no `package.json` sem ser importado (`src/shared/logger/index.ts`). O módulo e o worker vão precisar acrescentar os próprios logs. A configuração atual identifica todo log como da API, o que exige um ajuste para separar os logs do worker (FDD, seção 9).
- **ADR-006-CONS-13**: **Derivada:** por ser módulo da mesma base, as rotas de webhook rodam no processo da API e são implantadas junto com ela. Uma carga alta nessas rotas, como consultas ao histórico de entregas, concorre com as rotas de pedidos, e um defeito no módulo exige implantar a API inteira.

### Trade-off

- **ADR-006-CONS-11**: Troca a liberdade de desenho e o isolamento de um serviço próprio por consistência com a base. Ganha um módulo sem dependência nova e com o mesmo formato de erro. Paga com adaptações onde o padrão não cobre o caso (códigos fixos nas classes de erro, worker fora do middleware de erro, prefixo novo) e com rotas que compartilham processo e implantação com a API.

## Confirmação

- **Proposta (a validar):** testes do módulo no padrão de `tests/orders.test.ts`, conferindo `res.body.error.code` com os códigos de erro `WEBHOOK_*` definidos na matriz do FDD.

## Referências

- Decisão (ADR-006):
  - `[09:27] Bruno`: cada domínio é um módulo com controller, service, repository, routes e schemas; webhook segue igual.
  - `[09:28] Diego`: concorda com o módulo.
  - `[09:28] Bruno`: segue o padrão de erros com `AppError` e códigos; exemplos `WEBHOOK_NOT_FOUND`, `WEBHOOK_INVALID_URL`, `WEBHOOK_SECRET_REQUIRED`.
  - `[09:29] Larissa`: prefixo `WEBHOOK_` para tudo do módulo.
  - `[09:29] Bruno`: Pino, nada novo; middleware de erro trata `AppError`, Zod e Prisma.
  - `[09:30] Larissa`: reuso máximo do que existe; webhook como módulo igual aos outros.
  - `[09:36] Larissa`: reaproveita o controle de papéis existente no reprocessamento.
- Contexto e fatores: `[09:27] Bruno` (padrão de módulos), `[09:29] Bruno` (nenhum logger novo), `[09:30] Larissa` (reuso máximo), `[09:07] Diego` (time pequeno).
- ADR-006-ALT-03: `[09:29] Bruno` (descarte do logger novo).
- ADR-006-ALT-04: `[09:30] Larissa` ("Webhook fica como módulo igual aos outros") e `[09:07] Diego` (sem infraestrutura nova); a alternativa em si não foi discutida.
- Consequências: CONS-01 `[09:29] Bruno`; CONS-02 `[09:30] Larissa` e `[09:36] Larissa`; CONS-03 `[09:27] Bruno`; CONS-06 `[09:29] Larissa`; CONS-12 `[09:28] Bruno` e `[09:31] Marcos` (secret gerada pela plataforma); CONS-13 `[09:30] Larissa` (módulo igual aos outros) e `[09:34] Marcos` (histórico de entregas); CONS-11 `[09:30] Larissa`. CONS-04, CONS-05, CONS-06 e CONS-09 vêm do código, com os caminhos citados no próprio item.
- Código:
  - os caminhos e símbolos da tabela do Contexto;
  - `src/modules/auth/`: módulo sem repository;
  - `src/modules/orders/order.status.ts`: arquivo extra do módulo de pedidos;
  - `src/modules/users/user.service.ts`: `EMAIL_ALREADY_USED`, código sem prefixo;
  - `src/server.ts`: um dos três arquivos que usam o logger;
  - `package.json`: `pino-http` declarado.
- ADRs relacionados: [ADR-001](ADR-001-outbox-transacional-no-mysql.md) (outbox na transação de pedidos), [ADR-002](ADR-002-worker-separado-com-polling.md) (worker), [ADR-003](ADR-003-retry-com-backoff-e-dlq.md) (reprocessamento com papel ADMIN), [ADR-004](ADR-004-assinatura-hmac-sha256-por-endpoint.md) (secret gerada pela plataforma e redaction).
- Documentos: [FDD](../FDD.md), seções 4, 7, 9 e 11.

## Questões em aberto

1. **Como os códigos `WEBHOOK_NOT_FOUND` e `WEBHOOK_INVALID_URL` chegam à resposta?** Opções:
   - classes de erro no módulo, com código próprio, e a regra de https checada no service, fora do schema Zod. Exige aceitar que a regra de https saia do schema ou seja checada duas vezes;
   - mudar o `validate` compartilhado para aceitar código de erro por schema. Exige alterar um middleware usado por todos os módulos e, para `WEBHOOK_NOT_FOUND`, ainda uma classe própria ou `AppError` direto;
   - aceitar os códigos genéricos dos caminhos reaproveitados (`NOT_FOUND`, `VALIDATION_ERROR`, `UNAUTHORIZED`, `FORBIDDEN`, `CONFLICT`, `INTERNAL_SERVER_ERROR`) e limitar o prefixo aos erros de domínio do módulo. Exige rever a decisão de prefixo para tudo do módulo.

   Afeta o núcleo da decisão. Responder junto com a questão 2. Responde: arquitetura (Larissa) com engenharia de Pedidos (Bruno). Origem: CONS-04.
2. **O código `WEBHOOK_SECRET_REQUIRED` deve existir?** Opções:
   - descartar o código, já que a secret é gerada pela plataforma;
   - manter o código para outro caso, a definir na matriz de erros (FDD, seção 7).

   Afeta detalhes, não o que foi decidido. Responder junto com a questão 1. Responde: arquitetura (Larissa) com engenharia de Pedidos (Bruno). Origem: CONS-12.

## Recomendações

- A escolha mais arquitetural deste ADR é a fronteira: webhook como módulo da base existente, e não serviço separado. O título fala em "reuso dos padrões" e não deixa isso visível. Recomenda-se tornar a fronteira explícita no título, numa revisão futura, ou registrá-la em ADR própria se a alternativa de serviço separado voltar a ser considerada.
