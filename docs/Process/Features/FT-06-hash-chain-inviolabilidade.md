# [FEATURE] FT-06 — Integrity Verification API (Audit API)

## Description

**Wave 2 | Lean Inception: Guarantee Inviolability**

This feature adds to the `raven-ledger.api` service the cryptographic verification endpoints for the hash chain built by `raven-ledger.register` (FT-03). Judicial experts and compliance analysts can verify the integrity of a specific record or a range of records without needing direct database access.

The hash chain is built and maintained by FT-03. This feature merely exposes it as a verifiable API — the differentiator is that the integrity proof ceases to be an internal artifact and becomes queryable by external parties with a legitimate interest (e.g.: court-appointed expert).

**Business scenarios:**
- **Point verification:** Expert requests verification of a record → API confirms `{valid: true}` or identifies tampering.
- **Tampering detected:** `raw_payload` was modified directly in the database → verification returns `{valid: false, reason: "hash_mismatch"}`.
- **Broken chain:** Hash of a record was modified → the following record in the chain returns `{valid: false, reason: "previous_hash_broken"}`.
- **Range verification:** Expert requests verification of a period → all tree branches traversed; first breakpoint identified.

**Business hypothesis success indicators:** Tampering detection rate = 100% for any modification to mapped fields or to the `hash` of existing records.

---

## Technical Description

**Affected service:** `raven-ledger.api` (expansion of FT-04)

**Contract:** `docs/systemDesign/audit-api.openapi.yaml` (updated)

**Dependency:** FT-03 — hash chain built and `hash`/`previous_hash`/`previous_id` populated in `ledger_entries`.

**Endpoints:**

```
GET /api/v1/audit-entries/{id}/verify
  Verifies the integrity of a specific record.
  Response 200: { valid: true,  entryId: "...", hash: "..." }
  Response 200: { valid: false, entryId: "...", reason: "hash_mismatch|previous_hash_broken" }
  Response 404: record not found

GET /api/v1/audit-entries/verify-range?from=&to=
  Verifies all records in the period (filter on received_at).
  Traverses all tree branches from genesis to leaves.
  Stops at the first detected failure.
  Response 200: { valid: true,  entriesChecked: 150 }
  Response 200: { valid: false, chainBrokenAt: "<uuid>", entriesChecked: 150, firstFailureAt: "..." }
  Response 400: invalid parameters (e.g.: from later than to)
```

### Verification algorithm (application layer)

For each verified record:

1. Read the record from the database, including `hash`, `previous_hash`, and `previous_id`
2. Read the predecessor record by `previous_id` (if it exists)
3. Recalculate the hash according to the canonical algorithm defined in FT-03, using `data_payload::text` and `index_payload::text` read from PostgreSQL and re-normalized in the same way as the original record, and the `correlation_id` field (represented as an empty string when NULL in the canonical concatenation)
4. Compare the recalculated hash with the stored `hash` → divergence indicates `hash_mismatch`
5. Compare the `previous_hash` of the current record with the `hash` of the predecessor read in step 2 → divergence indicates `previous_hash_broken`

> **Note:** when reading `data_payload` and `index_payload` from PostgreSQL to recalculate the hash, JSONB normalizes the representation (sorts keys, removes spaces). The verification application must reproduce the same normalization applied at the time of the original INSERT so that the recalculated hash matches the stored one.

### Structured logs (Serilog — JSON)

- `integrity_verified` → `entry_id`, `valid`, `latency_ms`
- `integrity_violation_detected` → `entry_id`, `expected_hash`, `found_hash`

---

## Acceptance Criteria

- [ ] `GET /api/v1/audit-entries/{id}/verify` returns `{valid: true}` for an intact record
- [ ] Manual modification of any mapped field in the database invalidates the verification: `{valid: false, reason: "hash_mismatch"}`
- [ ] Manual modification of a record's `hash` invalidates the next record in the chain: `{valid: false, reason: "previous_hash_broken"}`
- [ ] `GET /api/v1/audit-entries/{id}/verify` with non-existent ID returns `404 Not Found`
- [ ] `GET /api/v1/audit-entries/verify-range` traverses all hash tree branches in the period and returns `{valid: true}` when no tampering is found
- [ ] `GET /api/v1/audit-entries/verify-range` identifies and returns `chainBrokenAt` and `firstFailureAt` at the first detected failure
- [ ] `GET /api/v1/audit-entries/verify-range` with `from` later than `to` returns `400 Bad Request`
- [ ] `integrity_violation_detected` log emitted when a violation is detected during verification

---

## Test Scenarios

```gherkin
# language: en

@regression
Feature: Audit trail integrity verification API
  As a judicial expert or compliance analyst
  I want to verify the integrity of audit records via API
  So that I can ensure no evidence has been tampered with after being recorded in the system

  Background:
    Given LedgerDatabase contains records persisted by the registration service
    And the FT-03 hash chain mechanism is active and records have a calculated hash
    And the user is authenticated in the API with an expert profile

  # AC-1: Verification of intact record
  @happy-path @ac-1 @regression
  Scenario: Verification of a record without tampering confirms integrity
    Given a record exists in LedgerDatabase without any changes after being written
    When the expert requests the integrity verification of that record
    Then the result confirms that the record is intact
    And the record's hash is returned in the response

  # AC-2: Tampering detected in payload
  @exception @ac-2 @regression
  Scenario: Direct modification of a mapped field invalidates the verification of the tampered record
    Given the content of a mapped field of a record was modified directly in the database
    When the expert requests the integrity verification of that record
    Then the result indicates that the record was tampered with, with reason "hash_mismatch"

  # AC-3: Tampering detected via broken chain hash
  @exception @ac-3 @regression
  Scenario: Direct modification of a record's hash invalidates the next record in the chain
    Given the hash of a record was modified directly in the database
    When the expert requests the integrity verification of the immediately following record in the chain
    Then the result indicates that the chain was broken with reason "previous_hash_broken"

  # AC-4: Non-existent record
  @exception @ac-4
  Scenario: Verification of a record with a non-existent identifier informs that it was not found
    Given no record has the identifier "id-que-nao-existe"
    When the expert requests the integrity verification by that identifier
    Then the service informs that the record was not found

  # AC-5: Intact range verification
  @happy-path @ac-5
  Scenario: Verification of a range without tampering confirms the integrity of all records
    Given a range of records exists without any changes after being written
    When the expert requests the integrity verification of the full range
    Then the result confirms that all records are intact
    And informs how many records were verified

  # AC-6: Range verification with tampering
  @exception @ac-6 @regression
  Scenario: Range verification with tampering identifies exactly the breakpoint
    Given a record in a range was tampered with after being written
    When the expert requests the integrity verification of the range
    Then the result indicates that the chain was broken
    And identifies the responsible record and the moment of the first failure

  # AC-7: Invalid period in verify-range
  @exception @ac-7
  Scenario: Period with start date later than end date is rejected
    Given the expert provides start date 2024-02-01 and end date 2024-01-01
    When the expert requests the range integrity verification
    Then the service rejects the request informing that the period is invalid
```

---

## Priority

1

## Risk

2 — Medium. The verification algorithm must faithfully reproduce the JSON normalization applied at the time of INSERT (FT-03). Any divergence in the re-serialization of `data_payload` or `index_payload` will produce false negatives — intact records reported as tampered. Thoroughly test against real data before enabling in production.

## Effort

M
