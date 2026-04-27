# [FEATURE] FT-04 — API de Consulta da Trilha de Auditoria (API Service)

## Descrição

**Wave 1 — MVP | Lean Inception: Trilha de Auditoria**

Esta feature implementa o serviço `raven-ledger.api`, responsável por expor os registros do LedgerDatabase via API REST de somente-leitura. Permite que Letícia (compliance) e De Marco (perito judicial) consultem a trilha de auditoria com filtros por entidade, período, usuário e tipo de operação, obtendo a lista de registros e o detalhe de cada um.

Este serviço é a interface programática do produto: ele serve tanto a aplicação web (FT-05) quanto integrações diretas por parte de peritos técnicos.

**Cenários de negócio:**
- **Happy path:** Perito filtra por entidade "users" e período → lista paginada de registros retornada em < 200ms.
- **Registro específico:** Acesso direto por ID → detalhe completo incluindo snapshot `data`.
- **Sem resultados:** Filtros não encontram registros → `200 OK` com lista vazia (não é erro).
- **Filtro inválido:** `from` posterior a `to` → `400 Bad Request` com mensagem descritiva.
- **ID inexistente:** `GET /audit-entries/{id}` com ID inválido → `404 Not Found`.

**Indicadores de sucesso:** Permite que Letícia execute uma trilha de auditoria completa sem precisar acessar o banco diretamente — eliminando a dependência de DBA para investigações de rotina.

## Descrição Técnica

**Serviço:** `raven-ledger.api` (WebAPI somente-leitura)

**Endpoints:**

```
GET /api/v1/audit-entries
  Query params:
    entity       (string, opcional)
    domain       (string, opcional)
    user_id      (string, opcional)
    operation    (string, opcional) — insert | update | delete
    from         (ISO 8601, opcional) — filtro em received_at
    to           (ISO 8601, opcional) — filtro em received_at
    page         (int, default 1)
    size         (int, default 20, max 100)
  Resposta: { items: [...], total: int, page: int, size: int }

GET /api/v1/audit-entries/{id}
  Resposta: objeto completo do ledger_entry, incluindo data snapshot
  404 se não encontrado
```

**Modelo de resposta (item da lista):**
```json
{
  "id": "uuid",
  "eventId": "app-domain-key",
  "source": "https://...",
  "domain": "sales",
  "entity": "users",
  "appName": "crm",
  "userId": "12345",
  "userName": "John Doe",
  "operationType": "update",
  "generatedAt": "2024-01-15T10:00:00Z",
  "transmittedAt": "2024-01-15T10:00:01Z",
  "receivedAt": "2024-01-15T10:00:02Z"
}
```

**Modelo de resposta (detalhe):** todos os campos acima + `data` (snapshot do `raw_payload.data`).

**Validações:**
- `from` e `to` devem ser ISO 8601 válidos
- Se ambos presentes: `from` ≤ `to`
- `operation` deve ser um de `insert`, `update`, `delete`
- `size` máximo de 100

**Performance:** Queries usam os índices criados na FT-01. Para `size` padrão (20), P99 < 200ms.

**Serviço é puramente de leitura:** Nenhum endpoint de escrita. Usuário de banco possui apenas `SELECT`.

**Logs estruturados:**
- `audit_query` → filtros aplicados, `result_count`, `latency_ms`
- `audit_entry_viewed` → `entry_id`, `latency_ms`

**Métricas de estabilidade:**
- Latência P99 < 200ms para queries com até 1M de registros no banco
- Disponibilidade > 99,9%

## Critérios de Aceite

- [ ] `GET /api/v1/audit-entries` retorna lista paginada com filtros funcionando individualmente e combinados
- [ ] `GET /api/v1/audit-entries/{id}` retorna registro completo com campo `data` (snapshot)
- [ ] ID inexistente retorna `404 Not Found`
- [ ] Filtro `from` posterior a `to` retorna `400 Bad Request`
- [ ] Paginação funciona: `page` e `size` controlam o resultado, `total` reflete o total de registros
- [ ] Resposta P99 < 200ms em banco com até 100.000 registros
- [ ] Nenhum endpoint de escrita exposto
- [ ] Usuário de banco de dados da aplicação possui apenas permissão `SELECT`
- [ ] Log `audit_query` emitido com filtros utilizados

## Cenários de Teste

```gherkin
# language: pt

@regression
Feature: API de consulta da trilha de auditoria
  Como perito ou analista de compliance
  Quero consultar registros de auditoria com filtros flexíveis
  Para investigar trilhas de operações sem precisar acessar o banco diretamente

  Background:
    Given que o LedgerDatabase contém registros de auditoria
    And que o usuário está autenticado na API

  # AC-1: Filtros individuais funcionando
  @happy-path @ac-1
  Scenario Outline: Consulta com filtro individual retorna apenas registros correspondentes
    Given que existem registros com "<campo>" igual a "<valor>" e registros com outros valores
    When o perito consulta a trilha filtrando "<campo>" pelo valor "<valor>"
    Then apenas os registros que correspondem ao filtro são retornados

    Examples:
      | campo     | valor   |
      | entity    | users   |
      | domain    | sales   |
      | user_id   | usr-123 |
      | operation | delete  |

  # AC-1: Filtros combinados
  @happy-path @ac-1
  Scenario: Consulta com múltiplos filtros retorna a interseção dos critérios
    Given que existem registros de entidade "orders" com operação "update" e registros com outras combinações
    When o perito consulta filtrando por entidade "orders" e operação "update"
    Then apenas os registros que satisfazem ambos os filtros são retornados

  # AC-1: Filtro por período
  @happy-path @ac-1
  Scenario: Consulta com filtro de período retorna apenas registros dentro do intervalo
    Given que existem registros com datas dentro e fora do período de 2024-01-01 a 2024-01-31
    When o perito consulta a trilha filtrando pelo período de 2024-01-01 a 2024-01-31
    Then apenas os registros recebidos dentro do período são retornados

  # AC-2: Detalhe de registro específico
  @happy-path @ac-2
  Scenario: Consulta por identificador retorna detalhe completo com snapshot de dados
    Given que existe um registro com um identificador conhecido
    When o perito consulta o detalhe do registro por esse identificador
    Then o registro completo é retornado incluindo o snapshot dos dados da operação original

  # AC-3: Registro inexistente
  @exception @ac-3
  Scenario: Consulta de registro com identificador inexistente informa que não foi encontrado
    Given que nenhum registro possui o identificador "id-que-nao-existe"
    When o perito consulta o detalhe por esse identificador
    Then o serviço informa que o registro não foi encontrado

  # AC-4: Período inválido
  @exception @ac-4
  Scenario: Período com data inicial posterior à data final é rejeitado
    Given que o perito informa data inicial 2024-02-01 e data final 2024-01-01
    When o perito solicita a consulta da trilha
    Then o serviço rejeita a requisição informando que o período é inválido

  # AC-5: Paginação
  @happy-path @ac-5
  Scenario: Paginação controla quantidade e posição dos resultados retornados
    Given que existem 50 registros no LedgerDatabase
    When o perito consulta a segunda página com tamanho de 10 registros
    Then são retornados 10 registros correspondentes à segunda página
    And o total reflete o número real de registros disponíveis

  # AC-5: Tamanho de página acima do limite
  @boundary @ac-5
  Scenario: Tamanho de página acima do limite máximo é rejeitado
    Given que o perito solicita 200 registros por página
    When a consulta é realizada
    Then o serviço rejeita a requisição informando que o tamanho excede o máximo permitido

  # AC-1: Consulta sem filtros retorna primeira página com total correto
  @happy-path @ac-1 @ac-5
  Scenario: Consulta sem filtros retorna a primeira página com o total de registros
    Given que existem registros de auditoria no LedgerDatabase
    When o perito consulta a trilha sem nenhum filtro
    Then os primeiros 20 registros são retornados com o total de registros correto

  # AC-6: Performance
  @boundary @ac-6
  Scenario: Consulta com filtro padrão responde dentro do limite de latência
    Given que o LedgerDatabase contém 100.000 registros
    When o perito realiza uma consulta com tamanho de página padrão
    Then a resposta é entregue em menos de 200 milissegundos

  # AC-7: Serviço somente-leitura
  @exception @ac-7
  Scenario: Tentativa de escrita via método HTTP não permitido é rejeitada
    Given que o perito tenta realizar uma operação de escrita via HTTP
    When a requisição é enviada ao serviço
    Then o serviço rejeita informando que o método não é permitido

  # AC-8: Permissão de leitura no banco
  @happy-path @ac-8
  Scenario: Usuário de aplicação da API não possui permissão de modificação no banco
    Given que o serviço está conectado ao LedgerDatabase com seu usuário de aplicação
    When uma tentativa de modificação de registro é executada com esse usuário
    Then a operação é negada pelo banco de dados
```

## Priority

1

## Risk

2 — Médio. O modelo de resposta da API define o contrato com a interface web (FT-05) e com integrações externas. Mudanças de quebra de contrato após a FT-05 ser implementada exigem versionamento ou coordenação de deploy.

## Effort

2 — P
