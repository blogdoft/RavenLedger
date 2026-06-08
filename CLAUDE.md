# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is a **documentation and content repository** — there is no application source code. It is a technical blog project (Blog do FT) that documents the complete process of building a software product from scratch, using **RavenLedger** as a fictional example product.

Blog content (`roteiro/`) is written in **Brazilian Portuguese (PT-BR)**. Technical documentation (`docs/`) and commit messages are written in **English**.

## RavenLedger — the example product

RavenLedger is a fictional audit microservice (SaaS or self-hosted) with the following product vision:

> Para **o perito designado pela justiça/perito compliance interno**
> cujas **investigações são difíceis de colher evidências válidas**
> o **RavenLedger** é um **micro-serviço SaaS ou self hosted**
> que **armazena evidências de manipulação de dados, identificáveis e de fácil acesso**.
> Diferente das **auditorias simplificadas**,
> nosso produto **garante a incorruptibilidade das evidências**.

Key constraints defined in the Lean Inception:
- **É / Faz**: Sistema de auditoria, repositório de evidências, armazena log de manipulações de entidade, identifica versões e autores, encadeia registros para validação de adulteração, detecta violação interna de maneira passiva.
- **Não é / Não faz**: Sistema de observabilidade/tracing, previne fraude, auditoria operacional/financeira, replay de eventos, correção de dados, determina fraude.

## Repository structure

- `roteiro/` — Blog chapters in Markdown. The narrative thread of the blog.
- `docs/Lean Inception/` — Excalidraw diagrams for each day of the Lean Inception (01–06).
- `docs/architecture.excalidraw` — Architecture diagram.
- `docs/Technical Constraints.md` — Cross-cutting technical constraints: languages (code/docs/commits in English, blog chapters in PT-BR), Conventional Commits rules, Git Flow branching strategy, static analysis, secrets policy, and diagram rules. **Read this file whenever branching, commit style, or project-wide conventions are referenced.**
- `docs/TechStack.md` — Full tech stack definition: backend (.NET 10), frontend (Angular LTS), messaging (Kafka), database (PostgreSQL), auth (KeyCloak), secrets (OpenBao), infra (K3D, ArgoCD, GitHub Actions) and observability (OpenTelemetry, Grafana stack). **Read this file whenever stack or library choices need to be referenced or discussed.**
- `systemDesign/` — System design artifacts: `sd.excalidraw` and `event.json` (CloudEvents 1.0 sample for `AuditEvent`).

## Working conventions

- To know what changed in a file, always use Git (`git log -- <file>`, `git diff`, `git show`). Never infer changes from file content alone.

## Content conventions

- Chapters follow the Lean Inception week structure (segunda → sexta-feira), then move into system design and implementation.
- The author explicitly acknowledges doing Lean Inception solo for teaching purposes — this is intentional, not an error.
- Existing diagrams are in Excalidraw format (`.excalidraw`) and are not meant to be edited as text.
- New diagrams generated for documentation or content must use **Mermaid** via the `diagram` skill. In Mermaid, line breaks inside labels must use `<br/>` — never `\n`.
- `event.json` uses the [CloudEvents 1.0 spec](https://cloudevents.io/) as the event envelope for audit events.
