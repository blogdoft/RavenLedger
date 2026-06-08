# Technical Constraints

This file defines cross-cutting technical constraints for the RavenLedger project.

---

## Languages

### Source Code

Code identifiers (variables, methods, classes, namespaces, properties) must be written in **English**.

**Rationale:** universal industry convention; ensures readability in technical contexts regardless of the reader's native language.

### Technical Documentation

All technical documentation (`docs/`) and code comments must be written in **English**.

**Rationale:** aligns with code language, maximizes accessibility for international contributors, and maintains consistency across all technical artifacts.

### Commit Messages

Commit messages must be written in **English**, following the Semantic Commits standard defined below.

### Process Documentation

Process documents — Azure Work Items and feature/story files under `docs/Process/` — must be written in **Brazilian Portuguese (PT-BR)**.

**Rationale:** these documents primarily serve product and business stakeholders who define and validate requirements in PT-BR; using the team's shared language reduces friction in requirement reviews and acceptance criteria validation.

---

## Semantic Commits

This project follows the [Conventional Commits](https://www.conventionalcommits.org/) specification.

### Format

```
<type>(<optional scope>): <short description in English>

[optional body]

[optional footer]
```

### Allowed Types

| Type | When to use |
|---|---|
| `feat` | New feature or new blog chapter |
| `fix` | Error correction in content or configuration |
| `docs` | Changes exclusively in supporting documentation |
| `refactor` | Restructuring without behavior or content change |
| `chore` | Maintenance tasks (CI, dependencies, configuration) |
| `style` | Formatting changes, no content or logic change |
| `wip` | Use when your commit or PR partially implements the Work Item |


### Examples

```
feat(roteiro): add system design chapter

docs(techstack): document FluentMigrator library

chore(ci): add markdown lint pipeline

fix(roteiro): correct sequence diagram in chapter 3
```

---

## Git Workflow — Git Flow

This project adopts **Git Flow** as its branching strategy.

### Main Branches

| Branch | Purpose |
|---|---|
| `main` | Stable / published content — always production-ready |
| `develop` | Continuous integration of new features and chapters |

### Supporting Branches

| Prefix | Purpose | Originates from | Merges into |
|---|---|---|---|
| `feature/*` | New features, chapters, or artifacts | `develop` | `develop` via PR |
| `bugfix/*` | Bug fixes found in production or during validations | `develop` or `release/*` | `develop` and `release/*` via PR |
| `release/*` | Release preparation and stabilization | `develop` | `main` and `develop` via PR |
| `hotfix/*` | Urgent fixes on published content | `main` | `main` and `develop` via PR |

### Rules

- Never commit directly to `main` or `develop`.
- Pull Requests are mandatory for merges into main branches.
- Branch names must be descriptive and kebab-cased: `feature/<work-item-number>_-_<brief-context>`.
- Supporting branches must be deleted after the PR is merged.
- `release/*` branches must follow semantic versioning: `release/1.2.0`.

---

## Static Analysis

### SonarQube — Primary Analyzer

**SonarQube** is the project's main static analysis platform. It runs as a mandatory CI quality gate and covers all languages in the stack (C# and TypeScript).

Responsibilities:
- Code quality and technical debt tracking over time.
- Security vulnerability detection (OWASP, CWE).
- Code coverage enforcement — the pipeline fails if coverage drops below the defined threshold.
- Duplication and complexity metrics.

A Pull Request may only be merged if the SonarQube quality gate passes.

### Build-Time Analyzers (C# only)

StyleCop and Roslynator run at compile time via Roslyn and enforce style and refactoring rules locally, before code reaches CI.

- The build breaks on any violation — no warnings are silently ignored.
- Warning suppressions (`#pragma warning disable`, `[SuppressMessage]`) are forbidden without a documented justification in the same file.

---

## Secrets and Environment Variables

- **No secret is versioned** — not even in local environments.
- **In production, all secrets are always retrieved from OpenBao at runtime** — no exceptions, regardless of the layer.

### Local development — per layer

| Layer | Configuration | Secrets |
|---|---|---|
| **Backend (.NET services)** | `appsettings.Development.json` (versioned, non-sensitive only) | ASP.NET Core User Secrets (`dotnet user-secrets`) |
| **Frontend (Angular)** | `.env` file | `.env` file |
| **Docker Compose** | `.env` file | `.env` file |

Additional rules:
- `appsettings.Development.json` must never contain secrets — only non-sensitive configuration overrides.
- `.NET` services do **not** use `.env` files.
- `.env` files must never be versioned; they must be listed in the `.gitignore` of the corresponding project.
- For every expected `.env` file, a `.env.template` must exist at the same level with keys but no values.

---

## Testing

### End-to-End Tests

**Selenium** is the mandatory tool for end-to-end browser tests against the Angular frontend.

- E2E tests must cover the critical user flows of the audit record viewing interface.
- Tests run as part of the CI pipeline on every Pull Request targeting `develop`.

### Load Tests

**k6** is the mandatory tool for load and performance tests against HTTP endpoints.

- Load test scenarios must validate the SLOs defined in each service's feature spec (e.g., P99 latency < 100ms at 50 req/s for `raven-ledger.ingestion`).
- k6 scripts are version-controlled alongside the service they test.
- k6 metrics are exported to Prometheus and visualised in Grafana.

---

## Diagrams

- New diagrams generated for documentation must use **Mermaid**.
- In Mermaid, line breaks inside labels use `<br/>` — never `\n`.
