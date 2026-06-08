---
name: create-workitem
description: Cria work items para o Azure DevOps (Epic, Feature ou Story) seguindo a estrutura definida para o projeto RavenLedger. Use quando precisar redigir ou estruturar um work item — o usuário fornece o tipo e o contexto, a skill gera o documento completo em Português Brasileiro pronto para colar no Azure DevOps.
argument-hint: "[tipo: epic | feature | story]"
---

# Skill: Create Azure DevOps Work Item

You are an expert in product management, processes, and agility. Your task is to help write Work Items for Azure DevOps following the project's defined structure.

All content must be written in **Brazilian Portuguese**.

## Work Item Types

### EPIC

Delimits a large deliverable — typically one of the functionalities in the Lean Inception sequencer, or the entire MVP depending on complexity.

**Required fields:**

**Description**
Describe in detail what the epic represents in terms of product value and why it is relevant.

**Priority**
Priority relative to other WIs. Range: 1 (most critical) to 4 (least critical).

**Risk**
3 = Low, 2 = Medium, 1 = High. Consider: integrations, rule complexity, number of services involved.

**Effort**
T-shirt sizing: 1- PP | 2- P | 3- M | 4- G | 5- GG

**Classification**
`Architectural` — driven by engineering concerns.
`Business` — driven by business/product concerns.

---

### FEATURE

Delimits a functionality (or group of functionalities) that can be delivered to production without causing usability, security, or availability issues. Features may be delivered partially when activated via feature flags.

**Required fields:**

**Description**
Detailed business motivations and rules to be implemented. Must cover: happy path scenarios, exception/error scenarios with user-facing messages, and (when applicable) business success indicators/metrics.

**Technical Description**
Code changes required, services to be created, logs to be emitted, components to be created/used, and any technical constraints. Include or reference diagrams, code examples, and attachments. Include stability/success metrics when relevant.

When the Technical Description requires a database schema (tables, columns, indexes, constraints), use the `db-conventions` skill to produce it. That skill enforces all naming conventions, column patterns, constraint naming, and `COMMENT ON` documentation. Do not inline schema rules here — delegate entirely to that skill.

**Acceptance Criteria**
Conditions for the feature to be considered done. Prefer a checklist. Include engineering acceptance criteria when necessary.

**Test Scenarios**
List all test scenarios ensuring every business rule is covered.

**Priority**
Range: 1 (most critical) to 4 (least critical).

**Risk**
3 = Low, 2 = Medium, 1 = High.

**Effort**
T-shirt sizing: 1- PP | 2- P | 3- M | 4- G | 5- GG

---

### STORY

**User Story** — describes user behavior derived from its parent feature.
**Technical Story** — describes code changes not necessarily tied to a user flow.

**Required fields:**

**Business Description**
For User Stories: describe from the user/business perspective — scope and business rules affected.
For Technical Stories: justify the engineering need and how it enables the business — scope and affected rules.

**Technical Description**
Code changes, services to create, logs to emit, components to create/use, and any technical constraints. Include or reference diagrams and code examples. Include stability metrics when relevant.

**Acceptance Criteria**
Conditions for the story to be considered done. Prefer a checklist. Include engineering acceptance criteria when necessary.

**Test Scenarios**
List all test scenarios ensuring business rule coverage.

**Story Points**
Fibonacci number representing implementation complexity. Values: 1, 2, 3, 5, 8, ∞ (anything above 8 = infinite, meaning the story must be split).

**Risk**
3 = Low, 2 = Medium, 1 = High.

**Priority**
Range: 1 (most critical) to 4 (least critical).

---

## How to use this skill

When invoked, ask the user:
1. **Which work item type** to create: Epic, Feature, or Story?
2. **Context**: What is this work item about? (brief description of the functionality or change)
3. For Stories: **Which Feature** does this story belong to? (for context and alignment)

Before generating, read the following files to ground the work item in the project's real context:
- `docs/Lean Inception/` — product goals, personas, and feature sequencing
- `docs/systemDesign/` — architecture and event design
- `CLAUDE.md` — product vision, constraints (É/Faz and Não é/Não faz)

Then generate the complete work item with all required fields filled in, formatted as a markdown document ready to be copied into Azure DevOps.

**Skill integrations:**
- For any diagram in Technical Description, use the `diagram` skill to produce a Mermaid diagram.
- For database schema (tables, indexes, constraints) in Technical Description, use the `db-conventions` skill.
- For the Test Scenarios field, use the `test-scenarios` skill to produce Gherkin scenarios with full BDD coverage.

## Output format

Output each work item as a structured markdown document using `##` headers for each field. At the top, include a suggested **Title** for the Azure DevOps work item (concise, action-oriented).

Example structure:
```
# [TYPE] Title of the Work Item

## Description
...

## Priority
...

## Risk
...

## Effort
...

## Classification (Epics only)
...
```

If the user provides incomplete context, ask follow-up questions before generating. Never invent business rules — only derive from provided context or existing project documents.
