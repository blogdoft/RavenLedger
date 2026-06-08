# [FEATURE] FT-02 — Recepção de AuditEvents via HTTP (Ingestion Service)

## Descrição

**Wave 1 — MVP | Lean Inception: Registro Automático**

Esta feature implementa o serviço `raven-ledger.ingestion`, porta de entrada do RavenLedger. Sistemas clientes (CRM, Financeiro, Estoque, etc.) enviam eventos de auditoria via HTTP. O serviço valida o payload, autentica o sistema cliente via KeyCloak, enfileira o evento no Kafka para processamento assíncrono e retorna confirmação ao sistema cliente em menos de 100ms.

O requisito de 100ms é crítico: sistemas clientes não podem ter sua performance degradada pela auditoria. Por isso, toda a lógica pesada (cálculo de hash, persistência no banco) é responsabilidade do `raven-ledger.register`, que consome a fila de forma assíncrona.

**Cenários de negócio:**
- **Happy path:** Sistema cliente envia AuditEvent válido com token KeyCloak → ingestion autentica, valida, enfileira, retorna `202 Accepted` com `X-Event-Id`.
- **Payload malformado:** Campo obrigatório ausente (ex: `metadata.source`) → `400 Bad Request` com campo inválido identificado.

- **Token ausente ou inválido:** Requisição sem token ou com token expirado → `401 Unauthorized`.
- **Broker indisponível:** Kafka fora do ar → `503 Service Unavailable`; sistema cliente deve retentar.
- **Evento duplicado enviado:** Mesmo `metadata.eventId` reenviado → `202 Accepted` (idempotente na entrada; deduplicação ocorre no register).

---

## Descrição Técnica

**Serviço:** `raven-ledger.ingestion` (.NET 10 — WebAPI)

**Contrato:** `docs/systemDesign/ingestion-api.openapi.yaml`

**Endpoint:**
```
POST /api/v1/events
Content-Type: application/json
Authorization: Bearer <token KeyCloak>
```

### Estrutura do payload

O corpo da requisição é dividido em duas seções:

- **`metadata`** — identifica e roteia o evento (idempotência, origem, timestamp de transmissão e correlação distribuída).
- **`event`** — conteúdo auditado (aplicação, usuário, operação e snapshot do dado).

### Autenticação

O sistema cliente deve obter um **token de serviço via KeyCloak** antes de enviar eventos. O `raven-ledger.ingestion` valida o token JWT a cada requisição. Requisições sem token ou com token inválido/expirado retornam `401 Unauthorized` antes de qualquer validação de payload.

### Validações obrigatórias (metadata)

- `metadata.eventId`: obrigatório, string não vazia. Padrão recomendado: `{app}-{domain}-{entity-key}`.
- `metadata.source`: obrigatório, string não vazia (URI do sistema de origem).
- `metadata.occurredAt`: obrigatório, ISO 8601.
- `metadata.correlationId`: opcional, string; quando informado, propaga o identificador de operação distribuída para vincular eventos emitidos por diferentes microserviços que fazem parte da mesma transação de negócio.

### Validações obrigatórias (event)

- `event.appInfo.appname`, `event.appInfo.domain`: obrigatórios.
- `event.user.id`: obrigatório.
- `event.operation.entity`: obrigatório.
- `event.operation.generatedAt`: obrigatório, ISO 8601.

### Resposta de sucesso

```
HTTP 202 Accepted
X-Event-Id: {metadata.eventId}
```

### Publicação no Kafka

Após validação, publica o payload completo no tópico `audit.events.received` usando **Confluent.Kafka** com publisher confirms (acks). O nome do tópico e o timeout de conexão com o broker são configurados via **RavenConfig** (banco mantido pelo `raven-ledger.api`).

### Gestão de Segredos

- **Ambiente produtivo:** segredos (credenciais Kafka, client secret KeyCloak) armazenados no **OpenBao**; a aplicação acessa o vault diretamente em startup.
- **Ambiente local:** segredos armazenados em arquivo `.env` (**nunca versionado**). Para cada `.env` esperado, existe um `.env.template` no mesmo nível, com as chaves sem valor real.

### Logs estruturados (Serilog — JSON)

- `event_received` → `event_id`, `source`, `domain`, `entity`, `correlation_id` (quando presente), `latency_ms`
- `event_validation_failed` → `event_id` (se presente), `errors[]`, `latency_ms`
- `broker_publish_failed` → `event_id`, `error`
- `auth_failed` → `reason`, `latency_ms`

### Métricas de estabilidade

- Latência P99 < 100ms
- Taxa de erro 4xx < 1% em operação normal
- Taxa de erro 5xx < 0,1%

### Configurações via RavenConfig (gerenciado pelo `raven-ledger.api`)

- Nome do tópico Kafka de destino
- Timeout de conexão com o broker
- URL do KeyCloak e realm de validação de token

---

## Critérios de Aceite

- [ ] `POST /api/v1/events` com payload válido e token KeyCloak retorna `202 Accepted`
- [ ] Header `X-Event-Id` presente na resposta de sucesso com o valor de `metadata.eventId`
- [ ] Requisição sem token ou com token inválido retorna `401 Unauthorized`
- [ ] Payload com campo obrigatório ausente retorna `400 Bad Request` com identificação do campo em dot-notation
- [ ] `event.operation.type` inválido retorna `400 Bad Request`
- [ ] Evento é publicado no tópico Kafka `audit.events.received` após aceite
- [ ] Broker indisponível retorna `503 Service Unavailable`
- [ ] Latência P99 < 100ms medida em carga de 50 req/s
- [ ] Log `event_received` emitido para cada evento aceito com campos corretos
- [ ] Segredos de acesso ao Kafka e KeyCloak não são versionados; `.env.template` presente no repositório
- [ ] Serviço não armazena dados em banco próprio (stateless)
- [ ] `metadata.correlationId` informado no payload é preservado e publicado no evento Kafka

---

## Cenários de Teste

```gherkin
# language: pt

@regression
Feature: Recepção de AuditEvents via HTTP
  Como sistema cliente autenticado
  Quero enviar eventos de auditoria ao RavenLedger
  Para que cada operação sobre dados seja registrada de forma confiável sem impactar minha performance

  Background:
    Given que o sistema cliente possui um token de serviço válido emitido pelo KeyCloak
    And que o broker Kafka está operacional

  # AC-1: Evento válido aceito e enfileirado
  @happy-path @ac-1 @ac-2
  Scenario: Envio de AuditEvent válido retorna confirmação de aceite com identificador
    Given que o payload contém metadata e event com todos os campos obrigatórios válidos
    When o sistema cliente envia o evento de auditoria
    Then o serviço confirma o aceite com o identificador do evento
    And o evento é publicado na fila para processamento assíncrono

  # AC-3: Campos obrigatórios de metadata ausentes
  @exception @ac-3
  Scenario Outline: Rejeição de evento com campo obrigatório de metadata ausente
    Given que o payload está sem o campo "<campo>"
    When o sistema cliente envia o evento de auditoria
    Then o serviço rejeita a requisição identificando o campo "<campo>" como inválido

    Examples:
      | campo                |
      | metadata.eventId     |
      | metadata.source      |
      | metadata.occurredAt  |

  # AC-3: Campos obrigatórios de event ausentes
  @exception @ac-3
  Scenario Outline: Rejeição de evento com campo obrigatório de event ausente
    Given que o payload está sem o campo "<campo>"
    When o sistema cliente envia o evento de auditoria
    Then o serviço rejeita a requisição identificando o campo "<campo>" como inválido

    Examples:
      | campo                        |
      | event.appInfo.appname        |
      | event.appInfo.domain         |
      | event.user.id                |
      | event.operation.entity       |
      | event.operation.generatedAt  |

  # AC-4: Tipo de operação inválido
  @exception @ac-4
  Scenario: Rejeição de evento com operation.type fora do domínio aceito
    Given que o payload contém o campo "event.operation.type" com o valor "read"
    When o sistema cliente envia o evento de auditoria
    Then o serviço rejeita a requisição identificando o campo "event.operation.type" como inválido

  # AC-1: Idempotência na entrada
  @happy-path @ac-1
  Scenario: Evento com identificador duplicado é aceito sem erro na entrada
    Given que um evento com o identificador "crm-sales-order-001" já foi enviado anteriormente
    When o sistema cliente envia novamente o evento com o identificador "crm-sales-order-001"
    Then o serviço confirma o aceite sem erro
    And a deduplicação ocorrerá no serviço de registro

  # AC-6: Broker indisponível
  @exception @ac-6
  Scenario: Broker indisponível resulta em resposta de serviço temporariamente indisponível
    Given que o broker Kafka está fora do ar
    When o sistema cliente envia um evento de auditoria válido
    Then o serviço informa que está temporariamente indisponível
    And o sistema cliente deve retentar a operação

  # Autenticação: token ausente
  @exception @regression
  Scenario: Requisição sem token de autenticação é rejeitada antes de qualquer validação
    Given que o sistema cliente não possui token de autenticação
    When o sistema cliente tenta enviar um evento de auditoria
    Then o serviço rejeita a requisição por falta de autenticação

  # Autenticação: token inválido ou expirado
  @exception @regression
  Scenario: Requisição com token expirado é rejeitada
    Given que o sistema cliente possui um token de autenticação expirado
    When o sistema cliente tenta enviar um evento de auditoria
    Then o serviço rejeita a requisição por falta de autenticação

  # AC-7: Latência P99 sob carga
  @boundary @ac-7
  Scenario: Latência P99 permanece abaixo do limite sob carga simultânea
    Given que 50 requisições são enviadas simultaneamente com payloads válidos
    When todas as requisições são processadas
    Then o percentil 99 de latência fica abaixo de 100 milissegundos

  # AC-8: Log de evento aceito
  @happy-path @ac-8
  Scenario: Log estruturado emitido para cada evento aceito contém os campos esperados
    Given que um evento de auditoria válido é enviado
    When o serviço confirma o aceite
    Then o log event_received é emitido com event_id, source, domain, entity e latência

  # AC-9: Serviço sem persistência própria
  @happy-path @ac-9
  Scenario: Serviço de ingestão não armazena dados em banco próprio
    Given que múltiplos eventos de auditoria foram aceitos pelo serviço de ingestão
    When o banco de dados interno do serviço é consultado
    Then nenhuma tabela de dados de eventos é encontrada

  # correlationId: campo repassado ao publicar
  @happy-path
  Scenario: Evento com correlationId é publicado na fila preservando o identificador de correlação
    Given que o payload contém o campo "metadata.correlationId" com o valor "op-7263"
    And que o payload contém metadata e event com todos os demais campos obrigatórios válidos
    When o sistema cliente envia o evento de auditoria
    Then o serviço confirma o aceite
    And o evento publicado na fila contém o campo correlationId com o valor "op-7263"
```

---

## Priority

1

## Risk

2 — Médio. O contrato REST define a interface com sistemas externos. Qualquer mudança de quebra impacta todos os sistemas clientes integrados. Versionar a API desde o início (`/v1/`) reduz o risco de evolução futura. A dependência do KeyCloak adiciona risco de disponibilidade na autenticação.

## Effort

2 — P
