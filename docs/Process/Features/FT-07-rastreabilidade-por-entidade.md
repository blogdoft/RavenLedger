# [FEATURE] FT-07 — Rastreabilidade de Modificações por Entidade e por Correlação

## Descrição

**Wave 2 | Lean Inception: Rastreabilidade**

Esta feature enriquece a API e a interface web com capacidade de rastreabilidade em duas dimensões complementares:

**1. Rastreabilidade por entidade** (baseada no campo `index`): dado um registro específico de uma entidade (identificado pelo campo `index` do AuditEvent), exibe o histórico completo de operações sobre aquele objeto — quem alterou, o que foi alterado, quando e em que versão. Permite a De Marco e Letícia reconstruírem a linha do tempo de um dado específico dentro de um único sistema de origem.

**2. Rastreabilidade por correlação** (baseada no `correlationId`): dado um identificador de operação distribuída, exibe todos os eventos registrados em diferentes microserviços que fazem parte da mesma transação de negócio. Permite identificar o impacto de uma única ação de usuário que se propaga por CRM, Financeiro, Estoque e outros serviços, unificando evidências dispersas em uma única linha do tempo cross-serviço.

O campo `index` do contrato AuditEvent foi projetado para a rastreabilidade por entidade: o sistema cliente determina quais campos identificam unicamente o registro auditado (ex: `{"orderId": "9872"}`) — o RavenLedger usa esses campos para correlacionar operações sobre o mesmo objeto ao longo do tempo. Já o `correlationId` (campo de `metadata`) é gerado pelo sistema cliente para propagar o contexto de uma operação distribuída entre serviços.

**Cenários de negócio:**
- **Histórico por entidade:** De Marco informa a entidade "orders" e o índice `orderId=9872` → lista cronológica de todas as operações sobre aquele pedido, com diff entre versões.
- **Rastreabilidade distribuída:** De Marco informa o `correlationId` "op-7263" → todos os eventos de CRM, Financeiro e Estoque gerados na mesma transação, ordenados por `generated_at`.
- **Entidade sem histórico:** Índice não encontrado → `200 OK` com lista vazia.
- **Múltiplos campos de índice:** `customerId=123` + `productId=456` → correlação por combinação de campos.
- **Diff entre versões:** Comparação entre dois registros consecutivos da mesma entidade → campos alterados destacados.

**Indicadores de sucesso:** Reduzir o tempo de reconstrução de histórico de uma entidade de horas (manual) para segundos (busca automatizada). Permitir rastreabilidade cross-serviço sem que o investigador precise cruzar logs de diferentes sistemas.

## Descrição Técnica

**Serviços afetados:** `raven-ledger.api` (novos endpoints), `raven-ledger.register` (indexação), interface web (novas telas)

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

GET /api/v1/audit-entries/trace
  Query params:
    correlation_id (string, obrigatório) — identificador da operação distribuída
    page           (int, default 1)
    size           (int, default 100, max 500)
  Resposta:
    {
      "correlationId": "op-7263",
      "entriesCount": 5,
      "sources": ["https://crm.example.com", "https://financial.example.com"],
      "items": [...]  -- mesmos campos do GET /api/v1/audit-entries
    }
  400 se correlation_id ausente
```

**Indexação no LedgerDatabase** (migration a ser criada pelo `raven-ledger.register` como parte desta feature):

```sql
-- Índice GIN para pesquisa por campos do index_payload
CREATE INDEX idx_ledger_entries__index_payload ON ledger_entries
  USING GIN (index_payload)
  WHERE index_payload IS NOT NULL;

-- Índice composto para rastreabilidade por entidade
CREATE INDEX idx_ledger_entries__entity_domain ON ledger_entries (entity, domain, generated_at);
```

> O índice para `correlation_id` já foi criado na FT-03 (`idx_ledger_entries__correlation_id`). O endpoint `/trace` o utiliza diretamente.

**Lógica de diff:** Comparação campo a campo entre `data_payload` (JSONB) de dois registros consecutivos da mesma entidade-chave. O diff é calculado na API (não armazenado), pois depende de qual par de registros o usuário está comparando.

**Novas telas na interface web:**

- Tela "Histórico de Entidade" (`/audit-entries/history`):
  - Formulário: `entity`, `domain` (opcional), `index_field`, `index_value`
  - Timeline vertical com os eventos em ordem cronológica
  - Cada item: `operationType`, `userName`, `generatedAt`
  - Clique em um item da timeline: painel lateral com diff entre este e o anterior
  - Campos modificados destacados (verde = adicionado, vermelho = removido, amarelo = modificado)

- Tela "Rastreabilidade por Correlação" (`/audit-entries/trace`):
  - Campo: `correlationId` (obrigatório)
  - Timeline agrupada por serviço de origem (`source`/`domain`)
  - Cada entrada: serviço, entidade, tipo de operação, usuário, `generatedAt`
  - Permite ao perito enxergar o impacto distribuído de uma única transação de negócio

**Logs estruturados:**
- `entity_history_query` → `entity`, `index_field`, `result_count`, `latency_ms`
- `entry_diff_computed` → `entry_id_a`, `entry_id_b`, `changed_fields_count`, `latency_ms`
- `correlation_trace_query` → `correlation_id`, `result_count`, `sources_count`, `latency_ms`

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
- [ ] `GET /api/v1/audit-entries/trace?correlation_id=X` retorna todos os registros com esse `correlationId`, ordenados por `generated_at`
- [ ] Resposta do `/trace` inclui lista de `sources` distintas encontradas
- [ ] `correlation_id` ausente no `/trace` → `400 Bad Request`
- [ ] Resposta do `/trace` < 200ms para operações com até 500 registros
- [ ] Interface web exibe tela de rastreabilidade por correlação com timeline agrupada por serviço

## Cenários de Teste

```gherkin
# language: pt

@regression
Feature: Rastreabilidade de modificações por entidade e por correlação
  Como perito judicial ou analista de compliance
  Quero consultar o histórico de operações sobre um registro específico ou sobre uma transação distribuída
  Para reconstruir a linha do tempo de um dado ou rastrear o impacto cross-serviço de uma operação

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

  # correlationId: rastreabilidade cross-serviço
  @happy-path
  Scenario: Consulta de rastreabilidade por correlationId retorna eventos de múltiplos serviços
    Given que três eventos com correlationId "op-7263" foram registrados por dois serviços distintos
    When o perito consulta a rastreabilidade pelo correlationId "op-7263"
    Then os três eventos são retornados ordenados por generated_at
    And a resposta indica os dois serviços distintos que geraram eventos nessa operação

  # correlationId: correlationId obrigatório no /trace
  @exception
  Scenario: Consulta de rastreabilidade sem correlationId é rejeitada
    Given que o perito não informa o correlationId
    When a consulta de rastreabilidade é solicitada
    Then o serviço rejeita a requisição informando que o correlationId é obrigatório

  # correlationId: correlationId sem registros
  @happy-path
  Scenario: Consulta de rastreabilidade com correlationId sem registros retorna lista vazia
    Given que nenhum registro possui o correlationId "op-inexistente"
    When o perito consulta a rastreabilidade pelo correlationId "op-inexistente"
    Then a lista retornada é vazia e nenhum erro é sinalizado
```

## Priority

2

## Risk

2 — Médio. Pesquisa em campo JSONB (`index_payload`) pode ser custosa sem índice GIN adequado. Volumetria alta (milhões de registros) pode degradar performance se o índice não cobrir o padrão de acesso. O endpoint `/trace` depende do índice parcial `idx_ledger_entries__correlation_id` definido na FT-03 — qualquer degradação nesse índice afeta diretamente a rastreabilidade cross-serviço. Testar com volume representativo antes de habilitar em produção.

## Effort

3 — M
