# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is a **documentation and content repository** — there is no application source code. It is a technical blog project (Blog do FT) that documents the complete process of building a software product from scratch, using **RavenLedger** as a fictional example product.

All blog content is written in **Brazilian Portuguese**.

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
- `systemDesign/` — System design artifacts: `sd.excalidraw` and `event.json` (CloudEvents 1.0 sample for `AuditEvent`).

## Content conventions

- Chapters follow the Lean Inception week structure (segunda → sexta-feira), then move into system design and implementation.
- The author explicitly acknowledges doing Lean Inception solo for teaching purposes — this is intentional, not an error.
- Diagrams are in Excalidraw format (`.excalidraw`) and are not meant to be edited as text.
- `event.json` uses the [CloudEvents 1.0 spec](https://cloudevents.io/) as the event envelope for audit events.
