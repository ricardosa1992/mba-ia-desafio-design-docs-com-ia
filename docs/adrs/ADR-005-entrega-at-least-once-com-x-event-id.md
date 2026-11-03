# ADR-005: Entrega at-least-once com deduplicação por X-Event-Id

## Status

**Aceito**, decidido na reunião técnica de quinta-feira (redigido em 2026-10-09, revisado em 2026-10-18).

> Aceito quanto à garantia at-least-once, ao identificador UUID gerado na inserção na outbox e enviado no header `X-Event-Id`, e à deduplicação feita pelo cliente. Se o reprocessamento da DLQ preserva o identificador, e portanto se a deduplicação cobre o reprocessamento, depende da questão 1.

- **Decisores**: Diego (Engenheiro Sênior, Plataforma), Larissa (Tech Lead).
- **Consultados**: Bruno (Engenheiro Pleno, Pedidos), Sofia (Engenheira de Segurança), Marcos (Product Manager).

## Contexto

O worker do [ADR-002](ADR-002-worker-separado-com-polling.md) repete o envio do que não teve sucesso, e um envio sem resposta em 10 s conta como falha mesmo que o cliente tenha processado o evento ([ADR-003](ADR-003-retry-com-backoff-e-dlq.md)). Com retry e reprocessamento, a plataforma precisava dizer qual garantia de entrega oferece e como o cliente lida com a repetição.

Neste ADR, "evento" é uma mudança de status do pedido registrada na outbox; "linha da outbox" é o registro técnico que o worker lê; "envio" é cada chamada HTTP ao endpoint do cliente. O identificador do evento é o mesmo valor no campo `event_id` do payload e no header `X-Event-Id`.

## Fatores de decisão

- Não exigir coordenação entre plataforma e cliente a cada envio.
- Seguir o modelo usado por provedores de webhook conhecidos, como Stripe e GitHub.

## Decisão

- A plataforma garante entrega at-least-once: cada evento gravado na outbox é enviado até ter sucesso ou esgotar os envios, e o cliente pode receber o mesmo evento mais de uma vez. Algumas opções ainda em aberto no [ADR-007](ADR-007-payload-snapshot-e-filtro-na-insercao.md) (questões 2 e 3) deixariam eventos sem nenhum envio, fora dessa garantia (CONS-12).
- Cada evento recebe um identificador UUID gerado quando entra na outbox, único por evento, enviado no header `X-Event-Id` e no campo `event_id` do payload. Um cliente com vários webhooks recebe o mesmo identificador em cada destino.
- Todos os envios de uma mesma linha da outbox, no retry, levam o mesmo identificador. Se o reprocessamento da DLQ preserva esse identificador está em aberto (questão 1).
- O cliente deduplica pelo identificador do lado dele.

Geração do UUID, biblioteca usada e semântica do header no contrato de entrega estão no FDD, seções 5.1 Criação do evento na outbox, 6.2 Contrato de entrega (request enviado ao cliente) e 10. Dependências e compatibilidade.

## Alternativas Consideradas

### ADR-005-ALT-01: Exactly-once

- **Descrição**: garantir que cada evento chega ao cliente exatamente uma vez.
- **Prós**: **Derivada:** o cliente não precisaria deduplicar.
- **Contras**: exigiria coordenação entre os dois lados a cada entrega e é muito mais complexo.
- **Motivo do descarte**: a coordenação com cada cliente custa mais do que a deduplicação pelo identificador.

## Consequências

### Positivas

- **ADR-005-CONS-01**: A plataforma entrega sem coordenar cada envio com o cliente.
- **ADR-005-CONS-02**: **Derivada:** os envios repetidos do retry deixam de ser problema de correção para a plataforma, porque o cliente tem o `X-Event-Id` para descartar a repetição. Para o reprocessamento, isso só vale se a questão 1 for respondida com "mesmo identificador".
- **ADR-005-CONS-03**: O cliente recebe uma chave estável para deduplicar, gerada na inserção na outbox e única por evento.

### Negativas

#### Responsabilidade do cliente

- **ADR-005-CONS-04**: O cliente pode receber o mesmo evento mais de uma vez e precisa estar preparado. A escolha transfere responsabilidade para o cliente.
- **ADR-005-CONS-05**: **Derivada:** a deduplicação depende de o cliente ler a documentação no portal; um cliente que não deduplica processa a repetição duas vezes.
- **ADR-005-CONS-10**: **Derivada:** para deduplicar, o cliente precisa guardar os identificadores já recebidos por algum tempo. O retry dura até 876 min, cerca de 14,6 h, na contagem que é hipótese no [ADR-003](ADR-003-retry-com-backoff-e-dlq.md) (CONS-05; a outra opção está na questão 1 dele). O reprocessamento manual pode acontecer a qualquer momento depois. Nenhuma janela de retenção foi definida para o cliente. Ver questão 3.
- **ADR-005-CONS-11**: **Derivada:** como o `event_id` está dentro do corpo assinado ([ADR-004](ADR-004-assinatura-hmac-sha256-por-endpoint.md)), a deduplicação também é a única defesa do cliente contra o reenvio de uma requisição capturada (ADR-004-CONS-07). A eficácia dessa defesa depende da mesma janela de retenção do CONS-10.

#### Identidade do evento

- **ADR-005-CONS-06**: **Derivada:** o reprocessamento devolve o evento à outbox no estado pendente ([ADR-003](ADR-003-retry-com-backoff-e-dlq.md)). Se isso gerar um identificador novo, o cliente não consegue reconhecer a repetição. Cenário: o cliente processa o evento, mas responde depois dos 10 s; os envios se esgotam e o evento vai para a DLQ; um ADMIN o reprocessa com identificador novo; o cliente processa o mesmo evento de novo. Ver questão 1.
- **ADR-005-CONS-07**: **Derivada:** não foi definido se a outbox terá uma linha por evento ou uma por webhook de destino. Como o identificador é o mesmo em todos os destinos, o cliente com vários webhooks deduplica por evento e endpoint, e a granularidade da linha decide se o retry de um destino afeta os outros. Ver questão 2.

#### Eventos sem envio

- **ADR-005-CONS-12**: **Derivada:** a garantia vale para eventos gravados e mantidos na outbox. Se o [ADR-007](ADR-007-payload-snapshot-e-filtro-na-insercao.md) adotar a gravação direta na DLQ do evento acima de 64 KB, a falha permanente no envio por tamanho ou o descarte das linhas de webhook desativado ou removido, esses eventos terão zero envios e ficarão fora do at-least-once. O mesmo vale se a questão 2 do [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md) usar o código `WEBHOOK_SECRET_REQUIRED` para webhook sem secret utilizável no momento de assinar: o evento não é enviado sem assinatura e pode esgotar o teto sem nenhuma chamada HTTP. Ver as questões 2 e 3 do ADR-007.

#### Origens de duplicata

- **ADR-005-CONS-09**: **Derivada:** além do retry, duplicatas surgem quando dois workers rodam ao mesmo tempo ([ADR-002](ADR-002-worker-separado-com-polling.md), CONS-10) e quando uma linha que ficou presa no estado da outbox processando é reenviada ([ADR-002](ADR-002-worker-separado-com-polling.md), CONS-14). Em todos os casos o identificador é o mesmo, então a deduplicação do cliente cobre essas repetições.

### Trade-off

- **ADR-005-CONS-08**: Troca a garantia de entrega única por simplicidade da plataforma. Ganha entrega sem coordenação entre os dois lados. Paga com duplicatas possíveis, que cada cliente precisa tratar pelo `X-Event-Id` e por uma janela de retenção que ainda não foi definida.

## Referências

- Decisão (ADR-005):
  - `[09:24] Diego`: garantia at-least-once; o cliente pode receber o mesmo evento duas vezes.
  - `[09:25] Diego`: `X-Event-Id` com UUID gerado quando o evento entra na outbox, único por evento; deduplicação pelo cliente.
  - `[09:26] Larissa`: fecha "At-least-once com X-Event-Id pra dedup do lado do cliente".
  - `[09:26] Marcos`: documenta com destaque no portal de desenvolvedor.
  - `[09:43] Diego`: `event_id` no payload JSON.
- Contexto e fatores: `[09:42] Diego` (timeout de 10 s conta como falha), `[09:25] Diego` (modelo de Stripe e GitHub; exactly-once exige coordenação), `[09:25] Bruno` (como o cliente diferencia).
- ADR-005-ALT-01: `[09:25] Sofia` (objeção à responsabilidade do cliente), `[09:25] Diego` (descarte do exactly-once).
- Consequências: CONS-01 `[09:25] Diego`; CONS-02 `[09:26] Larissa`; CONS-03 `[09:25] Diego`; CONS-04 `[09:24] Diego` e `[09:25] Sofia`; CONS-05 `[09:26] Marcos` (documentação no portal); CONS-06 `[09:18] Diego` (reprocessamento recoloca como pendente) e `[09:42] Diego`; CONS-07 `[09:25] Diego` (único por evento), `[09:34] Bruno` (filtro por webhook do cliente) e `[09:44] Sofia` (`X-Webhook-Id` para cliente com vários cadastros); CONS-08 `[09:25] Diego`; CONS-09 `[09:12] Diego` (worker único como premissa), com os cenários do ADR-002-CONS-10 e ADR-002-CONS-14; CONS-10 `[09:17] Diego` (janela do retry); CONS-11 `[09:22] Sofia` (assinatura sobre o corpo), `[09:43] Diego` (`event_id` no payload) e ADR-004-CONS-07; CONS-12 `[09:24] Larissa` (64 KB com erro), `[09:21] Sofia` (estado ativo do webhook) e as questões 2 e 3 do ADR-007.
- ADRs relacionados: [ADR-002](ADR-002-worker-separado-com-polling.md) (workers sobrepostos e linha presa), [ADR-003](ADR-003-retry-com-backoff-e-dlq.md) (retry, janela e reprocessamento), [ADR-004](ADR-004-assinatura-hmac-sha256-por-endpoint.md) (corpo assinado e reenvio capturado), [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md) (uso do código `WEBHOOK_SECRET_REQUIRED`), [ADR-007](ADR-007-payload-snapshot-e-filtro-na-insercao.md) (filtro por webhook na inserção).
- Documentos: [FDD](../FDD.md), seções 4, 5.1, 5.4, 6.2 e 10; [PRD](../PRD.md).

## Questões em aberto

1. **O reprocessamento da DLQ mantém o identificador original do evento?** Opções:
   - mesmo identificador: viável, porque a DLQ guarda o payload do evento, que contém o `event_id` ([ADR-003](ADR-003-retry-com-backoff-e-dlq.md)); exige que a linha devolvida à outbox reaproveite esse valor;
   - identificador novo: o cliente trata o reprocessamento como evento novo e pode processá-lo duas vezes (CONS-06). Exige reescrever o `event_id` dentro do snapshot vindo da DLQ antes de devolvê-lo à outbox, para header e corpo assinado continuarem iguais ([ADR-004](ADR-004-assinatura-hmac-sha256-por-endpoint.md), [ADR-007](ADR-007-payload-snapshot-e-filtro-na-insercao.md)).

   Afeta o núcleo da decisão. Responder junto com a questão 5 do [ADR-003](ADR-003-retry-com-backoff-e-dlq.md). Responde: arquitetura (Larissa). Origem: CONS-06.
2. **Com vários webhooks do mesmo cliente, a outbox tem uma linha por evento ou uma por webhook de destino?** Opções:
   - uma linha por webhook de destino, todas com o mesmo identificador: o retry de cada destino é independente, e o cliente deduplica por endpoint, que ele identifica pelo header `X-Webhook-Id`;
   - uma linha por evento, enviada a todos os destinos: o retry fica acoplado, e uma falha em um destino repete o envio para os outros;
   - um identificador por destino: exige rever "único por evento", decidido neste ADR.

   Afeta detalhes do modelo de dados, não o que foi decidido, salvo a terceira opção. Responder junto com a questão 3 do [ADR-007](ADR-007-payload-snapshot-e-filtro-na-insercao.md). Responde: arquitetura (Larissa). Origem: CONS-07.
3. **Por quanto tempo o cliente deve guardar os identificadores recebidos, segundo a documentação do portal?** Opções:
   - a duração do retry (cerca de 14,6 h na hipótese do ADR-003-CONS-05; ver a questão 1 do [ADR-003](ADR-003-retry-com-backoff-e-dlq.md)), deixando o reprocessamento manual fora da garantia;
   - sem limite, cobrindo o reprocessamento a qualquer momento.

   Afeta detalhes, não o que foi decidido. Depende da questão 1 deste ADR e da questão 1 do [ADR-003](ADR-003-retry-com-backoff-e-dlq.md); responder junto com a questão 2 do [ADR-004](ADR-004-assinatura-hmac-sha256-por-endpoint.md). Responde: produto (Marcos). Origem: CONS-10.
