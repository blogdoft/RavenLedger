# [FEATURE] FT-06 — API de Verificação de Integridade (Audit API)

## Descrição

**Wave 2 | Lean Inception: Garantir Inviolabilidade**

Esta feature adiciona ao serviço `raven-ledger.api` os endpoints de verificação criptográfica da cadeia de hashes construída pelo `raven-ledger.register` (FT-03). Peritos judiciais e analistas de compliance podem verificar a integridade de um registro específico ou de uma faixa de registros sem precisar acessar o banco de dados diretamente.

A cadeia de hashes é construída e mantida pela FT-03. Esta feature apenas a expõe como API verificável — o diferencial é que a prova de integridade deixa de ser um artefato interno e passa a ser consultável por partes externas com interesse legítimo (ex: perito designado pela justiça).

**Cenários de negócio:**
- **Verificação pontual:** Perito solicita verificação de um registro → API confirma `{valid: true}` ou identifica adulteração.
- **Adulteração detectada:** `raw_payload` foi modificado diretamente no banco → verificação retorna `{valid: false, reason: "hash_mismatch"}`.
- **Cadeia quebrada:** Hash de um registro foi modificado → registro seguinte na cadeia retorna `{valid: false, reason: "previous_hash_broken"}`.
- **Verificação em faixa:** Perito solicita verificação de um período → todos os ramos da árvore percorridos; primeiro ponto de quebra identificado.

**Indicadores de sucesso para hipótese de negócio:** Taxa de detecção de adulteração = 100% para qualquer modificação em campos mapeados ou em `hash` de registros existentes.

---

## Descrição Técnica

**Serviço afetado:** `raven-ledger.api` (expansão da FT-04)

**Contrato:** `docs/systemDesign/audit-api.openapi.yaml` (atualizado)

**Dependência:** FT-03 — cadeia de hashes construída e `hash`/`previous_hash`/`previous_id` populados em `ledger_entries`.

**Endpoints:**

```
GET /api/v1/audit-entries/{id}/verify
  Verifica integridade de um registro específico.
  Resposta 200: { valid: true,  entryId: "...", hash: "..." }
  Resposta 200: { valid: false, entryId: "...", reason: "hash_mismatch|previous_hash_broken" }
  Resposta 404: registro não encontrado

GET /api/v1/audit-entries/verify-range?from=&to=
  Verifica todos os registros do período (filtro em received_at).
  Percorre todos os ramos da árvore do genesis até as folhas.
  Para na primeira falha detectada.
  Resposta 200: { valid: true,  entriesChecked: 150 }
  Resposta 200: { valid: false, chainBrokenAt: "<uuid>", entriesChecked: 150, firstFailureAt: "..." }
  Resposta 400: parâmetros inválidos (ex: from posterior a to)
```

### Algoritmo de verificação (camada de aplicação)

Para cada registro verificado:

1. Ler o registro do banco, incluindo `hash`, `previous_hash` e `previous_id`
2. Ler o registro predecessor pelo `previous_id` (se existir)
3. Recalcular o hash segundo o algoritmo canônico definido na FT-03, usando `data_payload::text` e `index_payload::text` lidos do PostgreSQL e re-normalizados da mesma forma que o registro original, e o campo `correlation_id` (representado como string vazia quando NULL na concatenação canônica)
4. Comparar hash recalculado com o `hash` armazenado → divergência indica `hash_mismatch`
5. Comparar `previous_hash` do registro atual com o `hash` do predecessor lido no passo 2 → divergência indica `previous_hash_broken`

> **Atenção:** ao ler `data_payload` e `index_payload` do PostgreSQL para recalcular o hash, o JSONB normaliza a representação (ordena chaves, remove espaços). A aplicação de verificação deve reproduzir a mesma normalização aplicada no momento do INSERT original para que o hash recalculado coincida com o armazenado.

### Logs estruturados (Serilog — JSON)

- `integrity_verified` → `entry_id`, `valid`, `latency_ms`
- `integrity_violation_detected` → `entry_id`, `expected_hash`, `found_hash`

---

## Critérios de Aceite

- [ ] `GET /api/v1/audit-entries/{id}/verify` retorna `{valid: true}` para registro íntegro
- [ ] Modificação manual de qualquer campo mapeado no banco invalida a verificação: `{valid: false, reason: "hash_mismatch"}`
- [ ] Modificação manual do `hash` de um registro invalida o registro seguinte na cadeia: `{valid: false, reason: "previous_hash_broken"}`
- [ ] `GET /api/v1/audit-entries/{id}/verify` com ID inexistente retorna `404 Not Found`
- [ ] `GET /api/v1/audit-entries/verify-range` percorre todos os ramos da árvore de hashes no período e retorna `{valid: true}` quando nenhuma adulteração é encontrada
- [ ] `GET /api/v1/audit-entries/verify-range` identifica e retorna `chainBrokenAt` e `firstFailureAt` na primeira falha detectada
- [ ] `GET /api/v1/audit-entries/verify-range` com `from` posterior a `to` retorna `400 Bad Request`
- [ ] Log `integrity_violation_detected` emitido quando violação é detectada durante a verificação

---

## Cenários de Teste

```gherkin
# language: pt

@regression
Feature: API de verificação de integridade da trilha de auditoria
  Como perito judicial ou analista de compliance
  Quero verificar a integridade dos registros de auditoria via API
  Para garantir que nenhuma evidência foi adulterada após ser registrada no sistema

  Background:
    Given que o LedgerDatabase contém registros persistidos pelo serviço de registro
    And que o mecanismo de hash chain da FT-03 está ativo e os registros possuem hash calculado
    And que o usuário está autenticado na API com perfil de perito

  # AC-1: Verificação de registro íntegro
  @happy-path @ac-1 @regression
  Scenario: Verificação de registro sem adulteração confirma integridade
    Given que existe um registro no LedgerDatabase sem nenhuma alteração após a gravação
    When o perito solicita a verificação de integridade desse registro
    Then o resultado confirma que o registro é íntegro
    And o hash do registro é retornado na resposta

  # AC-2: Adulteração detectada no payload
  @exception @ac-2 @regression
  Scenario: Modificação direta de campo mapeado invalida a verificação do registro adulterado
    Given que o conteúdo de um campo mapeado de um registro foi modificado diretamente no banco de dados
    When o perito solicita a verificação de integridade desse registro
    Then o resultado indica que o registro foi adulterado com razão "hash_mismatch"

  # AC-3: Adulteração detectada via hash quebrado na cadeia
  @exception @ac-3 @regression
  Scenario: Modificação direta no hash de um registro invalida o registro seguinte na cadeia
    Given que o hash de um registro foi modificado diretamente no banco de dados
    When o perito solicita a verificação de integridade do registro imediatamente posterior na cadeia
    Then o resultado indica que a cadeia foi quebrada com razão "previous_hash_broken"

  # AC-4: Registro inexistente
  @exception @ac-4
  Scenario: Verificação de registro com identificador inexistente informa que não foi encontrado
    Given que nenhum registro possui o identificador "id-que-nao-existe"
    When o perito solicita a verificação de integridade por esse identificador
    Then o serviço informa que o registro não foi encontrado

  # AC-5: Verificação de faixa íntegra
  @happy-path @ac-5
  Scenario: Verificação de faixa sem adulterações confirma integridade de todos os registros
    Given que uma faixa de registros existe sem nenhuma alteração após a gravação
    When o perito solicita a verificação de integridade da faixa completa
    Then o resultado confirma que todos os registros são íntegros
    And informa quantos registros foram verificados

  # AC-6: Verificação de faixa com adulteração
  @exception @ac-6 @regression
  Scenario: Verificação de faixa com adulteração identifica exatamente o ponto de quebra
    Given que um registro em uma faixa foi adulterado após a gravação
    When o perito solicita a verificação de integridade da faixa
    Then o resultado indica que a cadeia foi quebrada
    And identifica o registro responsável pela quebra e o momento da primeira falha

  # AC-7: Período inválido no verify-range
  @exception @ac-7
  Scenario: Período com data inicial posterior à data final é rejeitado
    Given que o perito informa data inicial 2024-02-01 e data final 2024-01-01
    When o perito solicita a verificação de integridade da faixa
    Then o serviço rejeita a requisição informando que o período é inválido
```

---

## Priority

1

## Risk

2 — Médio. O algoritmo de verificação precisa reproduzir fielmente a normalização do JSON aplicada no momento do INSERT (FT-03). Qualquer divergência na re-serialização de `data_payload` ou `index_payload` produzirá falsos negativos — registros íntegros reportados como adulterados. Testar exaustivamente contra dados reais antes de habilitar em produção.

## Effort

2 — P
