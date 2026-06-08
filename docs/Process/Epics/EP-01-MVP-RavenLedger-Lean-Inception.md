# [EPIC] Implementar o MVP do RavenLedger conforme Lean Inception

## Descrição

O RavenLedger é um microsserviço de auditoria — SaaS ou self-hosted — cujo propósito é armazenar evidências de manipulação de dados de forma incorruptível, identificável e de fácil acesso para peritos designados pela justiça ou por equipes internas de compliance.

Esta Epic representa **todo o escopo definido na Lean Inception**, consolidado no MVP Canvas. O produto endereça a ausência de evidências válidas e invioláveis em investigações de conformidade e perícia forense digital.

**Público-alvo:** Perito designado pela justiça / perito compliance interno.

**Problema central:** Investigações carecem de evidências de manipulação de dados que sejam juridicamente válidas, rastreáveis e impossíveis de adulterar.

**Valor entregue:** O RavenLedger garante a incorruptibilidade das evidências, diferenciando-se de auditorias simplificadas que não provam *quem* alterou *o quê* e *quando*.

---

**Escopo delimitado pela Lean Inception:**

**É / Faz:**
- Sistema de auditoria e repositório de evidências
- Armazena log de manipulações de entidade com identificação de versão e autor
- Encadeia registros para validação de adulteração (hash chain)
- Detecta violação interna de maneira passiva
- Rastreia efeitos colaterais de operações
- Configura entidades de domínio auditáveis
- Válido como prova jurídica; somente leitura; inviolável
- Disponível como SaaS e on-premise

**Não é / Não faz:**
- Não é sistema de observabilidade/tracing ou monitoramento operacional
- Não previne fraude nem determina fraude
- Não realiza replay de eventos nem correção de dados
- Não substitui auditor humano
- Não é auditoria financeira, operacional ou de processos

---

**Personas validadas:**
- **Perito forense designado pela justiça** — precisa de evidências sólidas, cadeia de custódia digital e exportação de relatórios para processo judicial.
- **Perito compliance interno** — necessita de rastreabilidade ágil para investigações internas, com acesso filtrado por entidade e período.

---

## Priority

2

## Risk

1 — Alto. Envolve múltiplos serviços (ingestão, armazenamento imutável, API de consulta), algoritmo de hash chain para garantia de inviolabilidade, modelo de deployment dual (SaaS / on-premise) e requisitos implícitos de conformidade jurídica.

## Effort

5- GG — Escopo completo do MVP, cobrindo ingestão de eventos, armazenamento imutável, hash chain, API de consulta, configuração de entidades e modelo de entrega SaaS/on-premise.

## Classification

`Business` — Derivada diretamente do problema de negócio identificado na Lean Inception: ausência de evidências invioláveis em investigações de conformidade e perícia forense.

---

> Esta Epic deve ser decomposta em Features seguindo o sequenciador da Lean Inception
> (disponível em `docs/Lean Inception/04 - Quinta-feira.excalidraw` e `06 - Sexta-feira.excalidraw`).
