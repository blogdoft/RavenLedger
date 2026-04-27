# [FEATURE] FT-07 — Rastreabilidade de Modificações por Entidade

## Descrição

**Wave 2 | Lean Inception: Rastreabilidade**

Esta feature enriquece a API e a interface web com capacidade de rastreabilidade: dado um registro específico de uma entidade (identificado pelo campo `index` do AuditEvent), exibe o histórico completo de operações sobre aquele objeto — quem alterou, o que foi alterado, quando e em que versão. Permite a De Marco e Letícia reconstruírem a linha do tempo de um dado específico sem precisar cruzar tabelas ou depender de conhecimento do schema da aplicação cliente.

O campo `index` do contrato AuditEvent foi projetado exatamente para este caso: o sistema cliente determina quais campos identificam unicamente o registro auditado (ex: `{"orderId": "9872"}`) — o RavenLedger usa esses campos para correlacionar operações sobre o mesmo objeto ao longo do tempo.

**Cenários de negócio:**
- **Happy path:** De Marco informa a entidade "orders" e o índice `orderId=9872` → lista cronológica de todas as operações sobre aquele pedido, com diff entre versões.
- **Entidade sem histórico:** Índice não encontrado → `200 OK` com lista vazia.
- **Múltiplos campos de índice:** `customerId=123` + `productId=456` → correlação por combinação de campos.
- **Diff entre versões:** Comparação entre dois registros consecutivos da mesma entidade → campos alterados destacados.

**Indicadores de sucesso:** Reduzir o tempo de reconstrução de histórico de uma entidade de horas (manual) para segundos (busca automatizada).

## Descrição Técnica

**Serviços afetados:** `raven-ledger.api` (novos endpoints), `raven-ledger.register` (indexação), interface web (nova tela)

**Novos endpoints em `raven-ledger.api`:**

```
GET /api/v1/audit-entries/history
  Query params:
    entity       (string, obrigatório)
    domain       (string, opcional)
    index_field  (string, obrigatório) — nome do campo no objeto index
    index_value  (string, obrigatório) — valor do campo
  Resposta: lista ordenada por generated_at (asc), com campo diff entre registros consecutivos

GET /api/v1/audit-entries/{id}/diff/{other_id}
  Compara dois registros específicos
  Resposta: { added: {...}, removed: {...}, modified: { field: { from: ..., to: ... } } }
```

**Indexação no LedgerDatabase:**
```sql
-- Índice para pesquisa por campos do index
CREATE INDEX idx_ledger_index_fields ON ledger_entries
  USING GIN ((raw_payload -> 'data' -> 'index'));

-- Índice composto para rastreabilidade
CREATE INDEX idx_ledger_entity_domain ON ledger_entries (entity, domain, generated_at);
```

**Lógica de diff:** Comparação campo a campo entre `raw_payload.data.data` de dois registros consecutivos da mesma entidade-chave. O diff é calculado na API (não armazenado), pois depende de qual par de registros o usuário está comparando.

**Nova tela na interface web:**

- Tela "Histórico de Entidade" (`/audit-entries/history`):
  - Formulário: `entity`, `domain` (opcional), `index_field`, `index_value`
  - Timeline vertical com os eventos em ordem cronológica
  - Cada item: `operationType`, `userName`, `generatedAt`
  - Clique em um item da timeline: painel lateral com diff entre este e o anterior
  - Campos modificados destacados (verde = adicionado, vermelho = removido, amarelo = modificado)

**Logs estruturados:**
- `entity_history_query` → `entity`, `index_field`, `result_count`, `latency_ms`
- `entry_diff_computed` → `entry_id_a`, `entry_id_b`, `changed_fields_count`, `latency_ms`

## Critérios de Aceite

- [ ] `GET /api/v1/audit-entries/history?entity=X&index_field=Y&index_value=Z` retorna lista ordenada por `generated_at`
- [ ] Registros retornados pertencem apenas à combinação entidade + campo índice + valor fornecidos
- [ ] Entidade sem histórico retorna lista vazia (sem erro)
- [ ] `GET /api/v1/audit-entries/{id}/diff/{other_id}` retorna diff estruturado entre dois registros
- [ ] Índice GIN criado e utilizado na query de histórico (sem full table scan)
- [ ] Resposta < 200ms para entidades com até 1.000 registros
- [ ] Interface web exibe timeline com diff entre versões consecutivas
- [ ] Campos modificados são destacados visualmente na interface
- [ ] `entity` e `index_field` são obrigatórios → `400 Bad Request` se ausentes

## Cenários de Teste

```gherkin
# language: pt

@regression
Feature: Rastreabilidade de modificações por entidade
  Como perito judicial ou analista de compliance
  Quero consultar o histórico completo de operações sobre um registro específico
  Para reconstruir a linha do tempo de um dado sem precisar cruzar tabelas manualmente

  Background:
    Given que o LedgerDatabase contém registros de auditoria de diferentes entidades e índices
    And que o serviço de API está operacional

  # AC-1: Histórico retornado em ordem cronológica
  @happy-path @ac-1
  Scenario: Consulta de histórico retorna operações em ordem cronológica crescente
    Given que cinco eventos foram registrados para a entidade "orders" com orderId igual a "9872"
    When o perito consulta o histórico da entidade "orders" com o índice orderId igual a "9872"
    Then os cinco registros são retornados ordenados do mais antigo para o mais recente

  # AC-2: Isolamento por combinação entidade + índice
  @happy-path @ac-2
  Scenario: Histórico retorna apenas os registros da combinação entidade e índice solicitados
    Given que existem registros de "orders" com orderId "9872" e registros de outras entidades e índices
    When o perito consulta o histórico filtrando por entidade "orders" e orderId "9872"
    Then apenas os registros que correspondem exatamente à combinação solicitada são retornados

  # AC-3: Entidade sem histórico
  @happy-path @ac-3
  Scenario: Consulta de entidade sem histórico retorna lista vazia sem sinalizar erro
    Given que nenhum registro existe para a entidade "products" com o índice productId "inexistente"
    When o perito consulta o histórico dessa entidade e índice
    Then a lista retornada é vazia e nenhum erro é sinalizado

  # AC-4: Diff entre versões consecutivas
  @happy-path @ac-4
  Scenario: Comparação entre insert e update identifica os campos alterados
    Given que existe um registro de insert e um registro de update para a mesma entidade e índice
    When o perito solicita o diff entre os dois registros
    Then os campos alterados entre as duas versões são identificados e apresentados de forma estruturada

  # AC-5: Parâmetros obrigatórios ausentes
  @exception @ac-5
  Scenario Outline: Consulta sem parâmetro obrigatório é rejeitada informando o campo faltante
    Given que o perito não informa o parâmetro "<parametro>"
    When a consulta de histórico é solicitada
    Then o serviço rejeita a requisição informando que "<parametro>" é obrigatório

    Examples:
      | parametro   |
      | entity      |
      | index_field |

  # AC-6: Performance com volume representativo
  @boundary @ac-6
  Scenario: Consulta de histórico com 1.000 registros responde dentro do limite de latência
    Given que 1.000 registros estão associados à mesma entidade e índice
    When o perito consulta o histórico completo dessa entidade
    Then a resposta é entregue em menos de 200 milissegundos com o índice de pesquisa ativo

  # AC-7: Interface com timeline e diff lateral
  @happy-path @ac-7
  Scenario: Interface exibe timeline e campos modificados destacados ao selecionar evento
    Given que a tela de histórico é acessada com entidade e índice válidos que possuem múltiplos eventos
    When o perito seleciona um evento da timeline
    Then o painel lateral exibe os campos modificados em relação à versão anterior com destaque visual
```

## Priority

2

## Risk

2 — Médio. Pesquisa em campo JSONB (`raw_payload.data.index`) pode ser custosa sem índice GIN adequado. Volumetria alta (milhões de registros) pode degradar performance se o índice não cobrir o padrão de acesso. Testar com volume representativo antes de habilitar em produção.

## Effort

3 — M
