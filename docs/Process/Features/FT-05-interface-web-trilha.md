# [FEATURE] FT-05 — Web Interface for Audit Trail Visualization

## Description

**Wave 1 — MVP | Lean Inception: Audit Trail (UX)**

This feature implements the Web application (Angular) that allows Letícia (compliance) and De Marco (judicial expert) to query the audit trail directly from the browser, without the need for database access or technical API knowledge. The interface prioritizes clarity and functionality over aesthetics — the MVP goal is to enable real investigations.

The MVP Canvas defines "via browser" as the platform for both personas, but the Lean Inception assigns only ❤️ (1 heart) of UX for Audit Trail — signaling that a functional interface is sufficient; visual polish can come in later iterations.

**Business scenarios:**
- **Happy path:** Letícia accesses the application, filters records by entity and period, views the list, and opens the detail of a suspicious record.
- **No results:** Applied filters find no data → clear message "No records found for the selected filters."
- **Record detail:** Click on a record → full view including the `data` snapshot of the original payload.
- **API error:** `raven-ledger.api` service unavailable → friendly error message, no blank screen.

## Technical Description

**Application:** Standalone Angular, no own backend. Consumes exclusively `raven-ledger.api`.

**Screens:**

**1. Listing screen (`/audit-entries`)**
- Filter form: `entity`, `domain`, `user_id`, `operation` (select), `correlationId`, `from` (datepicker), `to` (datepicker)
- Results table with columns: `entity`, `domain`, `userName`, `operationType`, `generatedAt`, `receivedAt`
- Pagination (next/prev) synchronized with query parameters
- Clear filters button
- Loading state (skeleton or spinner)
- Empty state with descriptive message

**2. Detail screen (`/audit-entries/:id`)**
- All record fields: CloudEvent metadata + audit fields
- Display of the `data` field (entity snapshot) formatted as JSON with syntax highlighting
- "Back to listing" button that preserves previous filters
- 404 displayed in a friendly way if ID not found

**API integration:**
- Consumes `GET /api/v1/audit-entries` with filters mapped from URL query params
- Consumes `GET /api/v1/audit-entries/{id}` for detail
- API URL configurable via environment variable (`RAVEN_API_URL`)
- No authentication in MVP (self-hosted/local)

**Responsiveness:** Desktop first, minimum 1280px. Mobile is not an MVP requirement.

**No authentication state** in MVP — application directly accessible.

## Acceptance Criteria

- [ ] Listing screen displays records consuming the API with all filters working
- [ ] Filters for `entity`, `domain`, `user_id`, `operation`, and period work individually and combined
- [ ] Pagination works: navigates between pages and displays total record count
- [ ] Detail screen displays all record fields including formatted `data` snapshot
- [ ] Empty state displays the message "No records found for the selected filters"
- [ ] API error displays a friendly message (no blank screen or technical error)
- [ ] "Back" button on the detail screen preserves the previous listing filters
- [ ] API URL configurable via environment variable
- [ ] Application renders correctly at 1280px width
- [ ] Filters are reflected in the URL (query params) to allow link sharing
- [ ] Filter by `correlationId` works and returns only records from the same distributed operation

## Test Scenarios

```gherkin
# language: en

@regression
Feature: Web interface for audit trail visualization
  As a compliance analyst or judicial expert
  I want to query the audit trail from the browser
  So that I can investigate data operations without relying on technical database access

  Background:
    Given the web application is accessible in the browser
    And the raven-ledger.api service is operational with available records

  # AC-1: Listing with records
  @happy-path @ac-1
  Scenario: Listing displays records with the expected columns
    Given LedgerDatabase contains audit records
    When the analyst accesses the listing screen
    Then the table displays the records with the columns entity, domain, userName, operationType, generatedAt, and receivedAt

  # AC-2: Filter by entity
  @happy-path @ac-2
  Scenario: Filter by entity updates the listing with only the matching records
    Given there are records for entity "users" and for other entities
    When the analyst applies the entity filter with the value "users"
    Then the table displays only the records for entity "users"

  # AC-2: Filter by operation type
  @happy-path @ac-2
  Scenario Outline: Filter by operation type displays only the matching operations
    Given there are records of different operation types
    When the analyst filters by operation "<operation>"
    Then the table displays only records with operation "<operation>"

    Examples:
      | operation |
      | insert    |
      | update    |
      | delete    |

  # AC-3: Pagination
  @happy-path @ac-3
  Scenario: Pagination correctly navigates between pages of results
    Given there are more records than the size of one page
    When the analyst navigates to the next page
    Then different records are displayed and the total record count remains correctly shown

  # AC-4: Record detail
  @happy-path @ac-4
  Scenario: Selecting a record displays the full detail with snapshot formatted as JSON
    Given the listing displays audit records
    When the analyst selects a specific record
    Then all record fields are displayed including the data snapshot formatted as JSON

  # AC-5: Empty state
  @happy-path @ac-5
  Scenario: Filters with no result display a descriptive message to the analyst
    Given no record matches the applied filters
    When the analyst applies the filters and waits for the response
    Then the interface displays the message "No records found for the selected filters"

  # AC-6: API error
  @exception @ac-6
  Scenario: API unavailability displays a friendly message without exposing technical details
    Given the raven-ledger.api service is unavailable
    When the analyst accesses or refreshes the listing
    Then the interface displays a friendly error message without a blank screen or technical information

  # AC-7: Filter preservation on back navigation
  @happy-path @ac-7
  Scenario: Returning from the detail screen preserves the previously applied listing filters
    Given the analyst applied filters in the listing and opened the detail of a record
    When the analyst returns to the listing
    Then the previously applied filters are preserved

  # AC-9: Non-existent record displays friendly page
  @exception @ac-4
  Scenario: Accessing the detail of a non-existent record displays a friendly message
    Given the analyst navigates to the detail address of a non-existent record
    When the detail screen attempts to load the record
    Then the interface displays a friendly message informing that the record was not found

  # AC-10: Filters reflected in the URL
  @happy-path @ac-10
  Scenario: Applied filters appear in the URL to allow link sharing
    Given the analyst applies entity and period filters in the listing
    When the filters are applied
    Then the parameters corresponding to the filters appear in the page URL

  # correlationId: distributed operation filter
  @happy-path
  Scenario: Filter by correlationId displays only records from the same distributed operation
    Given there are records with correlationId "op-7263" and records with a different correlationId or without correlationId
    When the analyst applies the correlationId filter with the value "op-7263"
    Then the table displays only the records with correlationId "op-7263"
```

## Priority

2

## Risk

3 — Low. Purely read-only interface, no complex business rules. Consumes stable API (FT-04). Highest risk is UX, not data.

## Effort

M
