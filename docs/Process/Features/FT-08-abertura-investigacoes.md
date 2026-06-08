# [FEATURE] FT-08 — Investigation Opening and Management

## Description

**Wave 3 | Lean Inception: Investigation Opening**

This feature allows Letícia and De Marco to create formal investigations within RavenLedger, grouping relevant audit records, adding annotations, and managing the lifecycle of the investigative case. An investigation is the formal starting point of an audit process — whether for internal compliance or for judicial expert proceedings.

The value of this feature lies in transforming scattered audit records into an organized and documented case, with a history of investigator actions — addressing De Marco's pain points ("fear of having the report challenged") and Letícia's ("producing robust documentary evidence").

**Business scenarios:**
- **Happy path:** Letícia opens investigation "CRM Audit Q1/2024", associates 10 relevant records, adds an annotation explaining the context → investigation saved and accessible.
- **Closed investigation:** De Marco finalizes the case after completing the analysis → status changes to "Closed", `closed_at` filled in, new records cannot be associated.
- **Reopen investigation:** New records emerge after closure → investigation reopened for new analysis.
- **Attempt to associate a record with a closed investigation:** `409 Conflict` with explanatory message.
- **Investigation without records:** Creation without immediately associating records is allowed — the investigator can populate it throughout the analysis.

**Success indicators:**
- Reduce evidence organization time from hours to minutes.
- Increase legal success rate by 10% (MVP Canvas hypothesis).

## Technical Description

**Affected service:** `raven-ledger.api` (new write endpoints) + investigations database

**Contract:** `docs/systemDesign/investigations-api.openapi.yaml`

**Database schema** (stored in RavenConfig or a separate `investigations_db` database):

```sql
CREATE TABLE investigations (
  id           UUID         PRIMARY KEY DEFAULT gen_random_uuid(),
  title        VARCHAR(255) NOT NULL,
  description  TEXT,
  status       VARCHAR(20)  NOT NULL DEFAULT 'open',  -- open | closed
  created_by   VARCHAR(255) NOT NULL,
  created_at   TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
  closed_at    TIMESTAMPTZ,
  closed_by    VARCHAR(255)
);

CREATE TABLE investigation_entries (
  investigation_id UUID REFERENCES investigations(id),
  ledger_entry_id  UUID NOT NULL,  -- reference to ledger_entries
  added_at         TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  added_by         VARCHAR(255) NOT NULL,
  PRIMARY KEY (investigation_id, ledger_entry_id)
);

CREATE TABLE investigation_notes (
  id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  investigation_id UUID REFERENCES investigations(id),
  content          TEXT NOT NULL,
  created_by       VARCHAR(255) NOT NULL,
  created_at       TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

**Endpoints in `raven-ledger.api`:**

```
POST   /api/v1/investigations
       Body: { title, description, created_by }
       → 201 Created with the investigation object

GET    /api/v1/investigations
       Query: status (open|closed), page, size
       → paginated list

GET    /api/v1/investigations/{id}
       → detail with associated entries and notes

POST   /api/v1/investigations/{id}/entries
       Body: { ledger_entry_id, added_by }
       → 201 if added; 409 if investigation is closed; 404 if ledger_entry does not exist

DELETE /api/v1/investigations/{id}/entries/{ledger_entry_id}
       → Removes association (only if investigation is open)

POST   /api/v1/investigations/{id}/notes
       Body: { content, created_by }
       → 201 Created

PATCH  /api/v1/investigations/{id}/status
       Body: { status: "closed" | "open", changed_by }
       → Updates status; records closed_at and closed_by when closing
```

**Validations:**
- `title` required and non-empty
- `created_by` required (in MVP, free-form string — authentication comes later)
- Closed investigation: `POST /entries` returns `409 Conflict`
- `ledger_entry_id` must exist in LedgerDatabase before being associated

**Web interface — new screens:**
- `/investigations` — listing with status filter
- `/investigations/new` — creation form
- `/investigations/{id}` — detail: list of associated entries, notes, close/reopen button
- "Associate records" button opens a search modal (reuses filters from FT-04/FT-05, including the `correlationId` filter — allows locating all records from a distributed transaction to include as evidence in a single step)
- Inline annotation field with save button

**Structured logs:**
- `investigation_created` → `investigation_id`, `title`, `created_by`
- `investigation_entry_added` → `investigation_id`, `ledger_entry_id`, `added_by`
- `investigation_closed` → `investigation_id`, `closed_by`
- `investigation_note_added` → `investigation_id`, `note_id`, `created_by`

## Acceptance Criteria

- [ ] `POST /api/v1/investigations` creates an investigation with status "open"
- [ ] `POST /api/v1/investigations/{id}/entries` associates a ledger_entry with an open investigation
- [ ] Associating an entry with a **closed** investigation returns `409 Conflict`
- [ ] `POST /api/v1/investigations/{id}/notes` saves an annotation with timestamp and author
- [ ] `PATCH /api/v1/investigations/{id}/status` with `"closed"` fills in `closed_at` and `closed_by`
- [ ] `PATCH` with `"open"` reopens the investigation (clears `closed_at`)
- [ ] `GET /api/v1/investigations/{id}` returns the investigation with associated entries and notes
- [ ] Web interface allows creating, viewing, associating records, annotating, and closing investigations
- [ ] Attempt to associate a non-existent `ledger_entry_id` returns `404`
- [ ] Investigation listing filters by status
- [ ] Record search modal supports filtering by `correlationId` to locate all evidence from a distributed transaction in a single step

## Test Scenarios

```gherkin
# language: en

@regression
Feature: Investigation opening and management
  As a compliance analyst or judicial expert
  I want to create and manage formal investigations by grouping relevant audit records
  So that I can organize evidence in a documented and traceable manner

  Background:
    Given the investigations service is operational
    And audit records are available in LedgerDatabase

  # AC-1: Investigation creation with valid data
  @happy-path @ac-1
  Scenario: Investigation creation with valid data results in an open investigation
    Given the expert provides a title and their author identifier
    When the expert requests the opening of a new investigation
    Then the investigation is created with status "open" and the provided data is preserved

  # AC-1: Creation without required title
  @exception @ac-1
  Scenario: Investigation creation without a title is rejected
    Given the expert does not provide the investigation title
    When the expert requests the opening of a new investigation
    Then the service rejects the request informing that the title is required

  # AC-2: Record association with an open investigation
  @happy-path @ac-2
  Scenario: Multiple audit records are associated with an open investigation
    Given an investigation with status "open" exists
    When the expert associates three audit records with the investigation
    Then the three records appear in the investigation's evidence list

  # AC-3: Association blocked on a closed investigation
  @exception @ac-3 @regression
  Scenario: Attempt to associate a record with a closed investigation is rejected
    Given an investigation with status "closed" exists
    When the expert attempts to associate a new record with the investigation
    Then the service rejects the operation informing that the investigation is closed

  # AC-4: Adding an annotation
  @happy-path @ac-4
  Scenario: Added annotation is available in the investigation detail with author and date
    Given an open investigation exists
    When the expert adds an annotation with analysis context
    Then the annotation is visible in the investigation detail with creation date and author recorded

  # AC-5: Investigation closure
  @happy-path @ac-5
  Scenario: Investigation closure records the date and person responsible for closing
    Given an investigation with status "open" exists
    When the expert requests the closure of the investigation providing their identifier
    Then the investigation transitions to status "closed" with the closure date and responsible party recorded

  # AC-6: Investigation reopening
  @happy-path @ac-6
  Scenario: Closed investigation can be reopened for new analysis
    Given an investigation with status "closed" exists
    When the expert requests the reopening of the investigation
    Then the investigation returns to status "open"
    And new records can be associated with it

  # AC-7: Detail with evidence and annotations
  @happy-path @ac-7
  Scenario: Investigation detail displays all associated evidence and annotations
    Given an investigation has three associated records and two annotations
    When the expert queries the investigation detail
    Then the three records and the two annotations are displayed

  # AC-8: Non-existent record cannot be associated
  @exception @ac-8
  Scenario: Association of a non-existent record in LedgerDatabase is rejected
    Given the expert provides a record identifier that does not exist in LedgerDatabase
    When they attempt to associate that identifier with an open investigation
    Then the service rejects the operation informing that the record was not found

  # AC-9: Listing filtered by status
  @happy-path @ac-9
  Scenario Outline: Listing filtered by status returns only matching investigations
    Given there are open investigations and closed investigations
    When the expert lists investigations filtering by status "<status>"
    Then only investigations with status "<status>" are returned

    Examples:
      | status  |
      | open    |
      | closed  |

  # AC-2: Interface for investigation creation and record association
  @happy-path @ac-2
  Scenario: Interface allows creating an investigation and associating records via search modal
    Given the expert accesses the new investigation form in the web interface
    When the expert fills out the form and searches for records to associate via the search modal
    Then the investigation is created with the selected records already associated

  # correlationId: block location of distributed transaction evidence
  @happy-path
  Scenario: Search modal filters by correlationId to locate distributed transaction evidence
    Given an open investigation exists
    And multiple records from different services share the correlationId "op-7263"
    When the expert opens the record search modal and filters by correlationId "op-7263"
    Then all records with that correlationId are displayed in the modal for selection
```

## Priority

2

## Risk

2 — Medium. The investigation data model is independent from the audit core (does not write to LedgerDatabase). The main risk is in modeling: the `investigation_entries` relationship is a soft reference to `ledger_entry_id` (no cross-database FK), which requires manual validation.

## Effort

L
