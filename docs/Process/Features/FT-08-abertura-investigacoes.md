# [FEATURE] FT-08 — Abertura e Gestão de Investigações

## Descrição

**Wave 3 | Lean Inception: Abertura de Investigação**

Esta feature permite a Letícia e De Marco criarem investigações formais dentro do RavenLedger, agrupando registros de auditoria relevantes, adicionando anotações e gerenciando o ciclo de vida do caso investigativo. Uma investigação é o ponto de partida formal de um processo de auditoria — seja para compliance interna ou para instrução de perícia judicial.

O valor desta feature está em transformar registros dispersos de auditoria em um caso organizado e documentado, com histórico de ações do investigador — atendendo às dores de De Marco ("medo de ter laudo impugnado") e de Letícia ("produzir evidências documentais robustas").

**Cenários de negócio:**
- **Happy path:** Letícia abre investigação "Auditoria CRM Q1/2024", associa 10 registros relevantes, adiciona anotação explicando o contexto → investigação salva e acessível.
- **Investigação encerrada:** De Marco finaliza o caso após concluir a análise → status muda para "Encerrada", `closed_at` preenchido, novos registros não podem ser associados.
- **Reabrir investigação:** Novos registros surgem após encerramento → investigação reaberta para nova análise.
- **Tentativa de associar registro a investigação fechada:** `409 Conflict` com mensagem explicativa.
- **Investigação sem registros:** Criação sem associar registros imediatamente é permitida — o investigador pode populá-la ao longo da análise.

**Indicadores de sucesso:**
- Reduzir tempo de organização de evidências de horas para minutos.
- Aumentar taxa de sucesso jurídico em 10% (hipótese do MVP Canvas).

## Descrição Técnica

**Serviço afetado:** `raven-ledger.api` (novos endpoints de escrita) + banco de dados de investigações

**Schema de banco de dados** (armazenado no RavenConfig ou banco separado `investigations_db`):

```sql
CREATE TABLE investigations (
  id           UUID         PRIMARY KEY DEFAULT gen_random_uuid(),
  title        VARCHAR(255) NOT NULL,
  description  TEXT,
  status       VARCHAR(20)  NOT NULL DEFAULT 'open',  -- open | closed
  created_by   VARCHAR(255) NOT NULL,
  created_at   TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
  closed_at    TIMESTAMPTZ,
  closed_by    VARCHAR(255)
);

CREATE TABLE investigation_entries (
  investigation_id UUID REFERENCES investigations(id),
  ledger_entry_id  UUID NOT NULL,  -- referência ao ledger_entries
  added_at         TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  added_by         VARCHAR(255) NOT NULL,
  PRIMARY KEY (investigation_id, ledger_entry_id)
);

CREATE TABLE investigation_notes (
  id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  investigation_id UUID REFERENCES investigations(id),
  content          TEXT NOT NULL,
  created_by       VARCHAR(255) NOT NULL,
  created_at       TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

**Endpoints em `raven-ledger.api`:**

```
POST   /api/v1/investigations
       Body: { title, description, created_by }
       → 201 Created com objeto da investigação

GET    /api/v1/investigations
       Query: status (open|closed), page, size
       → lista paginada

GET    /api/v1/investigations/{id}
       → detalhe com entries associadas e notas

POST   /api/v1/investigations/{id}/entries
       Body: { ledger_entry_id, added_by }
       → 201 se adicionado; 409 se investigação fechada; 404 se ledger_entry não existe

DELETE /api/v1/investigations/{id}/entries/{ledger_entry_id}
       → Remove associação (somente se investigação aberta)

POST   /api/v1/investigations/{id}/notes
       Body: { content, created_by }
       → 201 Created

PATCH  /api/v1/investigations/{id}/status
       Body: { status: "closed" | "open", changed_by }
       → Atualiza status; registra closed_at e closed_by ao fechar
```

**Validações:**
- `title` obrigatório e não vazio
- `created_by` obrigatório (no MVP, string livre — autenticação vem após)
- Investigação fechada: `POST /entries` retorna `409 Conflict`
- `ledger_entry_id` deve existir no LedgerDatabase antes de ser associado

**Interface web — novas telas:**
- `/investigations` — listagem com filtro por status
- `/investigations/new` — formulário de criação
- `/investigations/{id}` — detalhe: lista de entries associadas, notas, botão de fechar/reabrir
- Botão "Associar registros" abre modal de busca (reutiliza filtros da FT-04/FT-05)
- Campo de anotação inline com botão salvar

**Logs estruturados:**
- `investigation_created` → `investigation_id`, `title`, `created_by`
- `investigation_entry_added` → `investigation_id`, `ledger_entry_id`, `added_by`
- `investigation_closed` → `investigation_id`, `closed_by`
- `investigation_note_added` → `investigation_id`, `note_id`, `created_by`

## Critérios de Aceite

- [ ] `POST /api/v1/investigations` cria investigação com status "open"
- [ ] `POST /api/v1/investigations/{id}/entries` associa ledger_entry à investigação aberta
- [ ] Associar entry a investigação **fechada** retorna `409 Conflict`
- [ ] `POST /api/v1/investigations/{id}/notes` salva anotação com timestamp e autor
- [ ] `PATCH /api/v1/investigations/{id}/status` com `"closed"` preenche `closed_at` e `closed_by`
- [ ] `PATCH` com `"open"` reabre investigação (limpa `closed_at`)
- [ ] `GET /api/v1/investigations/{id}` retorna investigação com entries e notas associadas
- [ ] Interface web permite criar, visualizar, associar registros, anotar e fechar investigações
- [ ] Tentativa de associar `ledger_entry_id` inexistente retorna `404`
- [ ] Listagem de investigações filtra por status

## Cenários de Teste

```gherkin
# language: pt

@regression
Feature: Abertura e gestão de investigações
  Como analista de compliance ou perito judicial
  Quero criar e gerenciar investigações formais agrupando registros de auditoria relevantes
  Para organizar evidências de forma documentada e rastreável

  Background:
    Given que o serviço de investigações está operacional
    And que existem registros de auditoria disponíveis no LedgerDatabase

  # AC-1: Criação de investigação com dados válidos
  @happy-path @ac-1
  Scenario: Criação de investigação com dados válidos resulta em investigação aberta
    Given que o perito informa título e seu identificador de autor
    When o perito solicita a abertura de uma nova investigação
    Then a investigação é criada com status "aberta" e os dados informados são preservados

  # AC-1: Criação sem título obrigatório
  @exception @ac-1
  Scenario: Criação de investigação sem título é rejeitada
    Given que o perito não informa o título da investigação
    When o perito solicita a abertura de uma nova investigação
    Then o serviço rejeita a requisição informando que o título é obrigatório

  # AC-2: Associação de registro a investigação aberta
  @happy-path @ac-2
  Scenario: Múltiplos registros de auditoria são associados a uma investigação aberta
    Given que existe uma investigação com status "aberta"
    When o perito associa três registros de auditoria à investigação
    Then os três registros aparecem na lista de evidências da investigação

  # AC-3: Associação bloqueada em investigação fechada
  @exception @ac-3 @regression
  Scenario: Tentativa de associar registro a investigação fechada é rejeitada
    Given que existe uma investigação com status "fechada"
    When o perito tenta associar um novo registro à investigação
    Then o serviço rejeita a operação informando que a investigação está encerrada

  # AC-4: Adição de anotação
  @happy-path @ac-4
  Scenario: Anotação adicionada fica disponível no detalhe da investigação com autor e data
    Given que existe uma investigação aberta
    When o perito adiciona uma anotação com contexto da análise
    Then a anotação é visível no detalhe da investigação com data de criação e autor registrados

  # AC-5: Encerramento de investigação
  @happy-path @ac-5
  Scenario: Encerramento de investigação registra data e responsável pelo fechamento
    Given que existe uma investigação com status "aberta"
    When o perito solicita o encerramento da investigação informando seu identificador
    Then a investigação passa para status "fechada" com data e responsável pelo encerramento registrados

  # AC-6: Reabertura de investigação
  @happy-path @ac-6
  Scenario: Investigação encerrada pode ser reaberta para nova análise
    Given que existe uma investigação com status "fechada"
    When o perito solicita a reabertura da investigação
    Then a investigação retorna ao status "aberta"
    And novos registros podem ser associados a ela

  # AC-7: Detalhe com evidências e anotações
  @happy-path @ac-7
  Scenario: Detalhe da investigação exibe todas as evidências e anotações associadas
    Given que uma investigação possui três registros associados e duas anotações
    When o perito consulta o detalhe da investigação
    Then os três registros e as duas anotações são exibidos

  # AC-8: Registro inexistente não pode ser associado
  @exception @ac-8
  Scenario: Associação de registro inexistente no LedgerDatabase é rejeitada
    Given que o perito informa um identificador de registro que não existe no LedgerDatabase
    When tenta associar esse identificador a uma investigação aberta
    Then o serviço rejeita a operação informando que o registro não foi encontrado

  # AC-9: Listagem filtrada por status
  @happy-path @ac-9
  Scenario Outline: Listagem filtrada por status retorna apenas investigações correspondentes
    Given que existem investigações abertas e investigações fechadas
    When o perito lista investigações filtrando pelo status "<status>"
    Then apenas investigações com o status "<status>" são retornadas

    Examples:
      | status  |
      | open    |
      | closed  |

  # AC-2: Interface de criação e associação de registros
  @happy-path @ac-2
  Scenario: Interface permite criar investigação e associar registros via modal de busca
    Given que o perito acessa o formulário de nova investigação na interface web
    When o perito preenche o formulário e busca registros para associar via modal de busca
    Then a investigação é criada com os registros selecionados já associados
```

## Priority

2

## Risk

2 — Médio. Modelo de dados de investigação é independente do core de auditoria (não toca no LedgerDatabase em escrita). Risco principal é de modelagem: o relacionamento `investigation_entries` é uma referência soft ao `ledger_entry_id` (sem FK cross-database), o que exige validação manual.

## Effort

4 — G
