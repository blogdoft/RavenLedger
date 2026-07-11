# [FEATURE] FT-04 — Audit Trail Query API (API Service)

## Description

**Wave 1 — MVP | Lean Inception: Audit Trail**

This feature implements the `raven-ledger.api` service, responsible for exposing LedgerDatabase records via a read-only REST API. It allows Letícia (compliance) and De Marco (judicial expert) to query the audit trail with filters by entity, period, user, and operation type, retrieving the record list and the detail of each one.

This service is the product's programmatic interface: it serves both the web application (FT-05) and direct integrations by technical experts.

**Business scenarios:**
- **Happy path:** Expert filters by entity "users" and period → paginated list of records returned in < 200ms.
- **Specific record:** Direct access by ID → full detail including the `data` snapshot.
- **No results:** Filters find no records → `200 OK` with empty list (not an error).
- **Invalid filter:** `from` later than `to` → `400 Bad Request` with descriptive message.
- **Non-existent ID:** `GET /audit-entries/{id}` with invalid ID → `404 Not Found`.

**Success indicators:** Allows Letícia to run a complete audit trail without needing direct database access — eliminating the DBA dependency for routine investigations.

## Technical Description

**Service:** `raven-ledger.api` (read-only WebAPI)

**Contract:** `docs/systemDesign/audit-api.openapi.yaml`

**Endpoints:**

```
GET /api/v1/audit-entries
  Query params:
    entity       (string, optional)
    domain       (string, optional)
    user_id      (string, optional)
    operation    (string, optional) — insert | update | delete
    from         (ISO 8601, optional) — filter on received_at
    to           (ISO 8601, optional) — filter on received_at
    correlation_id (string, optional) — filters by distributed operation identifier
    page         (int, default 1)
    size         (int, default 20, max 100)
  Response: { items: [...], total: int, page: int, size: int }

GET /api/v1/audit-entries/{id}
  Response: full ledger_entry object, including data snapshot
  404 if not found
```

**Response model (list item):**
```json
{
  "id": "uuid",
  "eventId": "app-domain-key",
  "source": "https://...",
  "correlationId": "op-7263",
  "domain": "sales",
  "entity": "users",
  "appName": "crm",
  "userId": "12345",
  "userName": "John Doe",
  "operationType": "update",
  "generatedAt": "2024-01-15T10:00:00Z",
  "transmittedAt": "2024-01-15T10:00:01Z",
  "receivedAt": "2024-01-15T10:00:02Z"
}
```

**Response model (detail):** all fields above + `data` (snapshot from `raw_payload.data`).

**Validations:**
- `from` and `to` must be valid ISO 8601
- If both present: `from` ≤ `to`
- `operation` must be one of `insert`, `update`, `delete`
- `size` maximum of 100

**Performance:** Queries use the indexes created in FT-01. For the default `size` (20), P99 < 200ms.

**Service is purely read-only:** No write endpoints. Database user has only `SELECT`.

**Structured logs:**
- `audit_query` → applied filters, `result_count`, `latency_ms`
- `audit_entry_viewed` → `entry_id`, `latency_ms`

**Stability metrics:**
- P99 latency < 200ms for queries with up to 1M records in the database
- Availability > 99.9%

## Acceptance Criteria

- [ ] `GET /api/v1/audit-entries` returns a paginated list with filters working individually and combined
- [ ] `GET /api/v1/audit-entries/{id}` returns the full record with `data` field (snapshot)
- [ ] Non-existent ID returns `404 Not Found`
- [ ] Filter `from` later than `to` returns `400 Bad Request`
- [ ] Pagination works: `page` and `size` control the result, `total` reflects the total number of records
- [ ] P99 response < 200ms on a database with up to 100,000 records
- [ ] No write endpoints exposed
- [ ] Application database user has only `SELECT` permission
- [ ] `audit_query` log emitted with filters used
- [ ] `GET /api/v1/audit-entries?correlation_id=X` returns only records with that `correlationId`
- [ ] `correlationId` field present in the response model (null when absent from the record)

## Test Scenarios

```gherkin
# language: en

@regression
Feature: Audit trail query API
  As an expert or compliance analyst
  I want to query audit records with flexible filters
  So that I can investigate operation trails without needing direct database access

  Background:
    Given LedgerDatabase contains audit records
    And the user is authenticated in the API

  # AC-1: Individual filters working
  @happy-path @ac-1
  Scenario Outline: Query with individual filter returns only matching records
    Given there are records with "<field>" equal to "<value>" and records with other values
    When the expert queries the trail filtering "<field>" by value "<value>"
    Then only the records matching the filter are returned

    Examples:
      | field     | value   |
      | entity    | users   |
      | domain    | sales   |
      | user_id   | usr-123 |
      | operation | delete  |

  # AC-1: Combined filters
  @happy-path @ac-1
  Scenario: Query with multiple filters returns the intersection of criteria
    Given there are records for entity "orders" with operation "update" and records with other combinations
    When the expert queries filtering by entity "orders" and operation "update"
    Then only the records satisfying both filters are returned

  # AC-1: Period filter
  @happy-path @ac-1
  Scenario: Query with period filter returns only records within the range
    Given there are records with dates inside and outside the period 2024-01-01 to 2024-01-31
    When the expert queries the trail filtering by the period 2024-01-01 to 2024-01-31
    Then only the records received within the period are returned

  # AC-2: Specific record detail
  @happy-path @ac-2
  Scenario: Query by identifier returns full detail with data snapshot
    Given there is a record with a known identifier
    When the expert queries the record detail by that identifier
    Then the full record is returned including the snapshot of the original operation data

  # AC-3: Non-existent record
  @exception @ac-3
  Scenario: Query for a record with a non-existent identifier informs that it was not found
    Given no record has the identifier "id-que-nao-existe"
    When the expert queries the detail by that identifier
    Then the service informs that the record was not found

  # AC-4: Invalid period
  @exception @ac-4
  Scenario: Period with start date later than end date is rejected
    Given the expert provides start date 2024-02-01 and end date 2024-01-01
    When the expert requests the trail query
    Then the service rejects the request informing that the period is invalid

  # AC-5: Pagination
  @happy-path @ac-5
  Scenario: Pagination controls the quantity and position of returned results
    Given there are 50 records in LedgerDatabase
    When the expert queries the second page with a size of 10 records
    Then 10 records corresponding to the second page are returned
    And the total reflects the actual number of available records

  # AC-5: Page size above the limit
  @boundary @ac-5
  Scenario: Page size above the maximum limit is rejected
    Given the expert requests 200 records per page
    When the query is performed
    Then the service rejects the request informing that the size exceeds the maximum allowed

  # AC-1: Query without filters returns first page with correct total
  @happy-path @ac-1 @ac-5
  Scenario: Query without filters returns the first page with the record total
    Given there are audit records in LedgerDatabase
    When the expert queries the trail without any filter
    Then the first 20 records are returned with the correct total record count

  # AC-6: Performance
  @boundary @ac-6
  Scenario: Query with default filter responds within the latency limit
    Given LedgerDatabase contains 100,000 records
    When the expert performs a query with the default page size
    Then the response is delivered in less than 200 milliseconds

  # AC-7: Read-only service
  @exception @ac-7
  Scenario: Attempt to write via a disallowed HTTP method is rejected
    Given the expert attempts a write operation via HTTP
    When the request is sent to the service
    Then the service rejects informing that the method is not allowed

  # AC-8: Read permission in the database
  @happy-path @ac-8
  Scenario: API application user does not have modification permission in the database
    Given the service is connected to LedgerDatabase with its application user
    When a record modification attempt is executed with that user
    Then the operation is denied by the database

  # correlationId: filter by distributed operation
  @happy-path
  Scenario: Filter by correlationId returns only records from the same distributed operation
    Given there are records with correlationId "op-7263" and records with a different correlationId or without correlationId
    When the expert queries the trail filtering by correlationId "op-7263"
    Then only the records with correlationId "op-7263" are returned
```

## Priority

1

## Risk

2 — Medium. The API response model defines the contract with the web interface (FT-05) and with external integrations. Breaking contract changes after FT-05 is implemented require versioning or coordinated deployment.

## Effort

S
