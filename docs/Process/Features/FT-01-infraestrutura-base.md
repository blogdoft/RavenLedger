# [FEATURE] FT-01 — Base Infrastructure and Repository Scaffolding

## Description

This technical feature provisions all the necessary foundation for RavenLedger services to operate in a local environment and for repositories to be ready for AI-assisted development (Claude Code). It covers the creation and initialization of the four service repositories, the complete local environment via Docker Compose — covering all defined stack dependencies — and the initial configuration of static analysis, secrets management, and Claude Code support in each repository.

Without this feature, none of the other services can start in a local environment and no repository has the minimum structure to begin development. It is a prerequisite for all other MVP features.

**Operation scenarios:**
- **Happy path:** `docker compose up` starts all containers; databases created automatically and empty; Kafka operational with topic created; KeyCloak and OpenBao accessible; observability stack active.
- **Configuration error:** Missing environment variable → container fails to start with a clear log message; the `.env.template` documents the name and purpose of the missing variable.
- **Re-execution:** `docker compose up` is idempotent — re-running does not recreate or corrupt existing data.

---

## Technical Description

### Repositories to create

| Repository | Purpose |
|---|---|
| `raven-ledger.infra` | Local infrastructure: Docker Compose, environment configuration, and setup documentation |
| `raven-ledger.ingestion` | Audit event ingestion service — empty initial scaffolding |
| `raven-ledger.register` | Event registration and persistence service — empty initial scaffolding |
| `raven-ledger.api` | Query API service — empty initial scaffolding |

---

### `raven-ledger.infra` — Local environment (Docker Compose)

The `docker-compose.yml` must cover all services required for the local development environment, with healthchecks on all containers.

#### Application dependencies

| Service | Base image | Minimum configuration |
|---|---|---|
| `postgres-ledger` | PostgreSQL | LedgerDatabase created automatically, empty, no tables |
| `postgres-config` | PostgreSQL | RavenConfig created automatically, empty, no tables |
| `kafka` | Apache Kafka (KRaft, without Zookeeper) | Topic `audit.events.received` created automatically on startup |
| `keycloak` | KeyCloak | Initial realm configured for RavenLedger; accessible for service token issuance |
| `openbao` | OpenBao | Dev mode; accessible for reading secrets in the local environment |

> The schema for each database (tables, indexes, permissions) is the responsibility of each service via FluentMigrator and will be defined in the corresponding features.

#### Observability stack

| Service | Purpose |
|---|---|
| `grafana-alloy` | OTLP collector — receives metrics, traces, and logs from services and forwards them to the backends |
| `loki` | Structured JSON log storage (emitted via Serilog) |
| `prometheus` | Metrics time-series storage |
| `tempo` | Distributed traces storage |
| `grafana` | Unified visualization; Loki, Prometheus, and Tempo datasources pre-configured |

#### Other `raven-ledger.infra` artifacts

- `.env.template` — documents **all** required environment variables for all services, with a description of each variable's purpose and the container that consumes it.
- `README.md` — instructions on how to start the local environment, prerequisites, and common troubleshooting.
- `CLAUDE.md` — repository instructions for Claude Code, covering: repository purpose, project conventions, and references to `docs/Technical Constraints.md` and `docs/TechStack.md`.

---

### Service repositories — initial scaffolding

Each of the service repositories (`.ingestion`, `.register`, `.api`) must be initialized with the minimum structure below. **No business logic code should be written in this feature.**

#### .NET 10 solution structure

Each repository receives an empty `.sln` solution with the project's standard folder structure:

```
__tests__/
src/
```

#### Static analysis — build-time (StyleCop + Roslynator)

- `Directory.Build.props` at the repository root referencing **StyleCop.Analyzers** and **Roslynator.Analyzers** packages as analysis dependencies, applied to all projects in the solution.
- `.editorconfig` with C# style rules aligned to StyleCop conventions.
- The build must fail on any static analysis violation.

#### Secrets management

- `.env.template` with the environment variables expected by the service (names and descriptions, no values).
- `.gitignore` explicitly including `.env` and other files with secrets.

#### CI Pipeline (GitHub Actions)

Each service repository receives a `.github/workflows/ci.yml` file as part of the scaffolding. The pipeline is the mandatory quality gate before any merge — no PR can be integrated without it passing.

**Triggers:**

| Event | Target branch | Purpose |
|---|---|---|
| `pull_request` | `develop` | Validates code before merge — blocks the PR if the pipeline fails |
| `push` | `develop` | Validates the integrated state after merge |

**Pipeline stages (in order):**

| Stage | Tool | Failure criterion |
|---|---|---|
| Restore | `dotnet restore` | Unresolved dependencies |
| Build | `dotnet build` | Compilation error or StyleCop/Roslynator violation |
| Test + Coverage | `dotnet test` + Coverlet | Any test failure |
| SonarQube Analysis | `dotnet sonarscanner` | SonarQube quality gate not approved |

> The SonarQube step includes `begin` (before build) and `end` (after tests), ensuring that coverage collected by Coverlet is sent for analysis.

**Secrets required in the GitHub repository:**

| Secret | Purpose |
|---|---|
| `SONAR_TOKEN` | SonarQube authentication token |
| `SONAR_HOST_URL` | SonarQube server URL (local in dev, dedicated instance in CI) |

> Secret values must be documented in the repository's `.env.template`, but never committed.

#### Claude Code support

- `CLAUDE.md` at the repository root, containing:
  - Service purpose (1–2 paragraphs).
  - Explicit reference to `docs/Technical Constraints.md` and `docs/TechStack.md` from the documentation repository.
  - Service-specific conventions that Claude Code must follow (to be detailed in each service's features).

---

## Acceptance Criteria

**Repositories:**
- [ ] Repositories `raven-ledger.infra`, `raven-ledger.ingestion`, `raven-ledger.register`, and `raven-ledger.api` created and initialized on GitHub

**Local environment:**
- [ ] `docker compose up` starts without errors and all containers become healthy in < 60 seconds
- [ ] `LedgerDatabase` accessible and empty (no business tables)
- [ ] `RavenConfig` accessible and empty (no business tables)
- [ ] Kafka operational with topic `audit.events.received` created
- [ ] KeyCloak accessible with initial realm configured
- [ ] OpenBao accessible in dev mode
- [ ] Grafana accessible with Loki, Prometheus, and Tempo datasources pre-configured
- [ ] SonarQube locally accessible
- [ ] `.env.template` documents all required environment variables
- [ ] `README.md` describes how to start the local environment

**Service scaffolding:**
- [ ] Each service repository has a `.sln` solution with `src/` and `tests/` structure
- [ ] `Directory.Build.props` with StyleCop and Roslynator configured — build fails on violations
- [ ] `.editorconfig` with C# rules aligned to StyleCop
- [ ] `sonar-project.properties` configured for SonarQube integration in CI
- [ ] `.env.template` created in each service repository
- [ ] `.gitignore` explicitly includes `.env` in each repository

**CI Pipeline:**
- [ ] `.github/workflows/ci.yml` created in each service repository (`.ingestion`, `.register`, `.api`)
- [ ] Pipeline triggered on `pull_request` to `develop` and on `push` to `develop`
- [ ] Stages configured in order: restore → build → test + coverage → SonarQube analysis
- [ ] PR automatically blocked when the pipeline fails
- [ ] Secrets `SONAR_TOKEN` and `SONAR_HOST_URL` documented in each repository's `.env.template`

**Claude Code:**
- [ ] `CLAUDE.md` created in all four repositories with purpose and references to project constraints

---

## Test Scenarios

```gherkin
# language: en

@regression
Feature: Base local environment infrastructure
  As a RavenLedger developer
  I want to start the local environment with a single command
  So that all services can operate in an integrated manner

  Background:
    Given all required environment variables are configured

  # AC: Local environment starts without errors and all containers become healthy
  @happy-path
  Scenario: Local environment startup from scratch
    Given no RavenLedger container is running
    When the local environment is started with docker compose up
    Then all containers become healthy in less than 60 seconds

  # AC: LedgerDatabase accessible and empty
  @happy-path
  Scenario: LedgerDatabase created and accessible with no business tables
    Given the local environment is running
    When a connection is established with LedgerDatabase
    Then the database exists and contains no business tables

  # AC: RavenConfig accessible and empty
  @happy-path
  Scenario: RavenConfig created and accessible with no business tables
    Given the local environment is running
    When a connection is established with RavenConfig
    Then the database exists and contains no business tables

  # AC: Kafka operational with topic created
  @happy-path
  Scenario: audit.events.received topic available in Kafka
    Given the local environment is running
    When a message is published to the audit.events.received topic
    Then the message is available for consumption

  # AC: KeyCloak accessible
  @happy-path
  Scenario: KeyCloak accessible with realm configured
    Given the local environment is running
    When the KeyCloak discovery endpoint is queried
    Then the RavenLedger realm is available

  # AC: Observability stack operational
  @happy-path
  Scenario: Grafana accessible with datasources configured
    Given the local environment is running
    When Grafana is accessed
    Then the Loki, Prometheus, and Tempo datasources are configured and responding

  # AC: SonarQube accessible
  @happy-path
  Scenario: SonarQube accessible for local analysis
    Given the local environment is running
    When SonarQube is accessed
    Then the interface is available and ready to receive analyses

  # AC: Missing required environment variable prevents startup
  @exception
  Scenario: Missing required environment variable prevents container startup
    Given a required environment variable is not configured
    When the local environment is started
    Then the corresponding container does not start
    And the log displays an error message identifying the missing variable

  # AC: Re-running the environment is idempotent
  @happy-path
  Scenario: Re-running the environment does not corrupt existing data
    Given the local environment is already running with data
    When the local environment is started again with docker compose up
    Then the environment state remains unchanged and initialization completes without errors

  # AC: Build fails on static analysis violation
  @happy-path
  Scenario: Service build fails on StyleCop violation
    Given a service repository is initialized with StyleCop configured
    When a C# file with a style violation is added and the build is executed
    Then the build fails with a message identifying the violation

  # AC: CI pipeline triggered on pull request to develop
  @happy-path
  Scenario: CI pipeline is triggered when a pull request to develop is opened
    Given a service repository has the .github/workflows/ci.yml file configured
    And a pull request is opened targeting the develop branch
    When the pull request is submitted
    Then the CI pipeline is triggered automatically
    And the pipeline result is displayed as a check on the pull request

  # AC: PR blocked when pipeline fails
  @exception
  Scenario: Pull request blocked when the CI pipeline fails
    Given a pull request is open targeting the develop branch
    And the CI pipeline failed at some stage
    When a reviewer attempts to approve the pull request merge
    Then the merge is blocked by GitHub until the pipeline passes

  # AC: Pipeline executed after merge to develop
  @happy-path
  Scenario: CI pipeline executed after merge to develop
    Given a pull request was approved and merged into develop
    When the push to develop is registered
    Then the CI pipeline is triggered automatically on the integrated state of the branch

  # AC: SonarQube quality gate integrated into the pipeline
  @happy-path
  Scenario: Pipeline fails when SonarQube quality gate is not approved
    Given the CI pipeline is running for a pull request
    When the SonarQube analysis step detects a quality gate violation
    Then the pipeline fails at the SonarQube step
    And the pull request remains blocked for merge
```

---

## Priority

1

## Risk

2 — Medium. Kafka's KRaft topology and KeyCloak's initial configuration (realm, client, scopes) are the highest attention points: errors here impact all subsequent features. The remaining components are well documented and carry low risk.

## Effort

L. The docker compose with the full stack (observability + authentication + secrets) is more work than the minimum original setup. Configuring local SonarQube and initializing the four repositories with standardized scaffolding adds volume, but the work is mechanical and well documented.
