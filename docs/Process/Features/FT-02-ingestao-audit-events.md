# [FEATURE] FT-02 — AuditEvent Reception via HTTP (Ingestion Service)

## Description

**Wave 1 — MVP | Lean Inception: Automatic Registration**

This feature implements the `raven-ledger.ingestion` service, RavenLedger's entry point. Client systems (CRM, Finance, Inventory, etc.) send audit events via HTTP. The service validates the payload, authenticates the client system via KeyCloak, enqueues the event in Kafka for asynchronous processing, and returns confirmation to the client system in less than 100ms.

The 100ms requirement is critical: client systems must not have their performance degraded by auditing. Therefore, all heavy processing (hash calculation, database persistence) is the responsibility of `raven-ledger.register`, which consumes the queue asynchronously.

**Business scenarios:**
- **Happy path:** Client system sends a valid AuditEvent with a KeyCloak token → ingestion authenticates, validates, enqueues, returns `202 Accepted` with `X-Event-Id`.
- **Malformed payload:** Required field missing (e.g.: `metadata.source`) → `400 Bad Request` with the invalid field identified.
- **Token absent or invalid:** Request without token or with an expired token → `401 Unauthorized`.
- **Broker unavailable:** Kafka is down → `503 Service Unavailable`; client system should retry.
- **Duplicate event sent:** Same `metadata.eventId` resent → `202 Accepted` (idempotent at the entry point; deduplication occurs in the register service).

---

## Technical Description

**Service:** `raven-ledger.ingestion` (.NET 10 — WebAPI)

**Contract:** `docs/systemDesign/ingestion-api.openapi.yaml`

**Endpoint:**
```
POST /api/v1/events
Content-Type: application/json
Authorization: Bearer <KeyCloak token>
```

### Payload structure

The request body is divided into two sections:

- **`metadata`** — identifies and routes the event (idempotency, origin, transmission timestamp, and distributed correlation).
- **`event`** — audited content (application, user, operation, and data snapshot).

### Authentication

The client system must obtain a **service token via KeyCloak** before sending events. `raven-ledger.ingestion` validates the JWT token on every request. Requests without a token or with an invalid/expired token return `401 Unauthorized` before any payload validation.

### Required validations (metadata)

- `metadata.eventId`: required, non-empty string. Recommended pattern: `{app}-{domain}-{entity-key}`.
- `metadata.source`: required, non-empty string (source system URI).
- `metadata.occurredAt`: required, ISO 8601.
- `metadata.correlationId`: optional, string; when provided, propagates the distributed operation identifier to link events emitted by different microservices that are part of the same business transaction.

### Required validations (event)

- `event.appInfo.appname`, `event.appInfo.domain`: required.
- `event.user.id`: required.
- `event.operation.entity`: required.
- `event.operation.generatedAt`: required, ISO 8601.

### Success response

```
HTTP 202 Accepted
X-Event-Id: {metadata.eventId}
```

### Kafka publishing

After validation, publishes the full payload to the `audit.events.received` topic using **Confluent.Kafka** with publisher confirms (acks). The topic name and broker connection timeout are configured via **RavenConfig** (database maintained by `raven-ledger.api`).

### Secrets management

- **Production environment:** secrets (Kafka credentials, KeyCloak client secret) stored in **OpenBao**; the application accesses the vault directly at startup.
- **Local environment:** secrets stored in a `.env` file (**never committed**). For each expected `.env`, a `.env.template` exists at the same level, with keys but no real values.

### Structured logs (Serilog — JSON)

- `event_received` → `event_id`, `source`, `domain`, `entity`, `correlation_id` (when present), `latency_ms`
- `event_validation_failed` → `event_id` (if present), `errors[]`, `latency_ms`
- `broker_publish_failed` → `event_id`, `error`
- `auth_failed` → `reason`, `latency_ms`

### Stability metrics

- P99 latency < 100ms
- 4xx error rate < 1% under normal operation
- 5xx error rate < 0.1%

### Configuration via RavenConfig (managed by `raven-ledger.api`)

- Target Kafka topic name
- Broker connection timeout
- KeyCloak URL and token validation realm

---

## Acceptance Criteria

- [ ] `POST /api/v1/events` with valid payload and KeyCloak token returns `202 Accepted`
- [ ] `X-Event-Id` header present in the success response with the `metadata.eventId` value
- [ ] Request without token or with invalid token returns `401 Unauthorized`
- [ ] Payload with missing required field returns `400 Bad Request` with the field identified in dot-notation
- [ ] Invalid `event.operation.type` returns `400 Bad Request`
- [ ] Event is published to Kafka topic `audit.events.received` after acceptance
- [ ] Unavailable broker returns `503 Service Unavailable`
- [ ] P99 latency < 100ms measured under 50 req/s load
- [ ] `event_received` log emitted for each accepted event with correct fields
- [ ] Kafka and KeyCloak access secrets are not committed; `.env.template` present in the repository
- [ ] Service does not store data in its own database (stateless)
- [ ] `metadata.correlationId` provided in the payload is preserved and published in the Kafka event

---

## Test Scenarios

```gherkin
# language: en

@regression
Feature: AuditEvent reception via HTTP
  As an authenticated client system
  I want to send audit events to RavenLedger
  So that each data operation is reliably recorded without impacting my performance

  Background:
    Given the client system has a valid service token issued by KeyCloak
    And the Kafka broker is operational

  # AC-1: Valid event accepted and enqueued
  @happy-path @ac-1 @ac-2
  Scenario: Sending a valid AuditEvent returns an acceptance confirmation with an identifier
    Given the payload contains metadata and event with all required fields valid
    When the client system sends the audit event
    Then the service confirms acceptance with the event identifier
    And the event is published to the queue for asynchronous processing

  # AC-3: Missing required metadata fields
  @exception @ac-3
  Scenario Outline: Rejection of event with missing required metadata field
    Given the payload is missing the "<field>" field
    When the client system sends the audit event
    Then the service rejects the request identifying the "<field>" field as invalid

    Examples:
      | field                |
      | metadata.eventId     |
      | metadata.source      |
      | metadata.occurredAt  |

  # AC-3: Missing required event fields
  @exception @ac-3
  Scenario Outline: Rejection of event with missing required event field
    Given the payload is missing the "<field>" field
    When the client system sends the audit event
    Then the service rejects the request identifying the "<field>" field as invalid

    Examples:
      | field                        |
      | event.appInfo.appname        |
      | event.appInfo.domain         |
      | event.user.id                |
      | event.operation.entity       |
      | event.operation.generatedAt  |

  # AC-4: Invalid operation type
  @exception @ac-4
  Scenario: Rejection of event with operation.type outside the accepted domain
    Given the payload contains the field "event.operation.type" with the value "read"
    When the client system sends the audit event
    Then the service rejects the request identifying the field "event.operation.type" as invalid

  # AC-1: Idempotency at the entry point
  @happy-path @ac-1
  Scenario: Event with duplicate identifier is accepted without error at the entry point
    Given an event with the identifier "crm-sales-order-001" has already been sent previously
    When the client system sends the event again with the identifier "crm-sales-order-001"
    Then the service confirms acceptance without error
    And deduplication will occur in the registration service

  # AC-6: Broker unavailable
  @exception @ac-6
  Scenario: Unavailable broker results in a service temporarily unavailable response
    Given the Kafka broker is down
    When the client system sends a valid audit event
    Then the service informs that it is temporarily unavailable
    And the client system should retry the operation

  # Authentication: missing token
  @exception @regression
  Scenario: Request without authentication token is rejected before any validation
    Given the client system does not have an authentication token
    When the client system attempts to send an audit event
    Then the service rejects the request due to missing authentication

  # Authentication: invalid or expired token
  @exception @regression
  Scenario: Request with an expired token is rejected
    Given the client system has an expired authentication token
    When the client system attempts to send an audit event
    Then the service rejects the request due to missing authentication

  # AC-7: P99 latency under load
  @boundary @ac-7
  Scenario: P99 latency stays below the limit under simultaneous load
    Given 50 requests are sent simultaneously with valid payloads
    When all requests are processed
    Then the 99th percentile latency stays below 100 milliseconds

  # AC-8: Log of accepted event
  @happy-path @ac-8
  Scenario: Structured log emitted for each accepted event contains the expected fields
    Given a valid audit event is sent
    When the service confirms acceptance
    Then the event_received log is emitted with event_id, source, domain, entity, and latency

  # AC-9: Service without own persistence
  @happy-path @ac-9
  Scenario: Ingestion service does not store data in its own database
    Given multiple audit events have been accepted by the ingestion service
    When the service's internal database is queried
    Then no event data tables are found

  # correlationId: field forwarded on publish
  @happy-path
  Scenario: Event with correlationId is published to the queue preserving the correlation identifier
    Given the payload contains the field "metadata.correlationId" with the value "op-7263"
    And the payload contains metadata and event with all other required fields valid
    When the client system sends the audit event
    Then the service confirms acceptance
    And the event published to the queue contains the correlationId field with the value "op-7263"
```

---

## Priority

1

## Risk

2 — Medium. The REST contract defines the interface with external systems. Any breaking change impacts all integrated client systems. Versioning the API from the start (`/v1/`) reduces the risk of future evolution. The dependency on KeyCloak adds availability risk to authentication.

## Effort

S
