# [FEATURE] FT-06 — Garantia de Inviolabilidade via Hash Chain

## Descrição

**Wave 2 | Lean Inception: Garantir Inviolabilidade**

Esta feature implementa o mecanismo de encadeamento de registros por hash (hash chain) no serviço `raven-ledger.register`. Após a persistência de cada AuditEvent, é calculado um hash do registro combinado com o hash do registro imediatamente anterior, formando uma cadeia criptográfica verificável. Qualquer adulteração posterior de um registro — mesmo por um administrador de banco de dados — invalida todos os hashes subsequentes, detectando violação de forma passiva e automática.

Este é o diferencial técnico do RavenLedger frente a auditorias simplificadas: não basta que o dado seja append-only por controle de permissão. O hash chain oferece prova matemática de integridade, com validade para uso como evidência jurídica.

**Cenários de negócio:**
- **Happy path:** Registro gravado → hash calculado e encadeado → `hash` e `previous_hash` populados na linha.
- **Verificação de integridade:** Perito solicita verificação de um registro → API confirma `{valid: true}`.
- **Adulteração detectada:** Alguém modifica `raw_payload` diretamente no banco → verificação retorna `{valid: false, chain_broken_at: "<id>"}`.
- **Verificação em cadeia:** Verificar integridade de uma faixa de registros → identificar exatamente onde a cadeia foi quebrada.

**Indicadores de sucesso para hipótese de negócio:** Taxa de detecção de adulteração = 100% para qualquer modificação em `raw_payload` ou `hash` de registros existentes.

## Descrição Técnica

**Serviço afetado:** `raven-ledger.register` (expansão da FT-03)
**API afetada:** `raven-ledger.api` (novos endpoints de verificação)

**Algoritmo de hash:**
```
hash_input = SHA-256( event_id + "|" + raw_payload_canonical + "|" + previous_hash )
```
- `raw_payload_canonical`: serialização JSON determinística (chaves ordenadas, sem espaços)
- `previous_hash`: hash do registro imediatamente anterior no LedgerDatabase (ordenado por `received_at`)
- Primeiro registro da cadeia: `previous_hash = "0000...0000"` (64 zeros)
- Algoritmo: SHA-256, resultado em hexadecimal lowercase (64 chars)

**Fluxo no `raven-ledger.register` (após FT-03):**
1. Persiste registro (INSERT) — `hash` e `previous_hash` ficam NULL temporariamente
2. Em transação separada:
   a. Busca o `hash` do registro com `received_at` imediatamente anterior
   b. Calcula hash do registro atual
   c. Atualiza colunas `hash` e `previous_hash` no registro recém-inserido

**Importante:** O cálculo ocorre fora do caminho crítico da ingestão (não afeta SLA de 100ms). Registros sem hash ainda são válidos e imutáveis — o hash é uma camada adicional de verificação.

**Novos endpoints em `raven-ledger.api`:**

```
GET /api/v1/audit-entries/{id}/verify
  Resposta: { valid: true, entry_id: "...", hash: "..." }
  Ou:       { valid: false, entry_id: "...", reason: "hash_mismatch|previous_hash_broken" }

GET /api/v1/audit-entries/verify-range?from=&to=
  Verifica todos os registros do período
  Resposta: { valid: true, entries_checked: 150 }
  Ou: { valid: false, chain_broken_at: "<id>", entries_checked: 150, first_failure_at: "..." }
```

**Logs estruturados:**
- `hash_computed` → `event_id`, `hash` (primeiros 8 chars), `previous_hash` (primeiros 8 chars), `compute_time_ms`
- `integrity_verified` → `entry_id`, `valid`, `latency_ms`
- `integrity_violation_detected` → `entry_id`, `expected_hash`, `found_hash`

**Configuração:** Algoritmo de hash configurável via RavenConfig (para futura evolução para SHA-3 etc.).

## Critérios de Aceite

- [ ] Cada registro persistido recebe `hash` e `previous_hash` calculados após a gravação (não bloqueia ingestão)
- [ ] `hash` é calculado sobre `event_id + raw_payload_canonical + previous_hash`
- [ ] `previous_hash` do primeiro registro é `"0000...0000"` (64 zeros)
- [ ] `previous_hash` de qualquer outro registro aponta para o `hash` do registro anterior (por `received_at`)
- [ ] `GET /api/v1/audit-entries/{id}/verify` retorna `{valid: true}` para registro íntegro
- [ ] Modificação manual de `raw_payload` no banco invalida verificação: `{valid: false}`
- [ ] `GET /api/v1/audit-entries/verify-range` identifica `chain_broken_at` corretamente
- [ ] Cálculo de hash não impacta latência da ingestão (P99 < 100ms mantido na FT-02)
- [ ] Log `integrity_violation_detected` emitido quando violação é detectada na verificação

## Cenários de Teste

```gherkin
# language: pt

@regression
Feature: Garantia de inviolabilidade via hash chain
  Como perito judicial ou analista de compliance
  Quero verificar a integridade dos registros de auditoria
  Para garantir que nenhuma evidência foi adulterada após ser registrada no sistema

  Background:
    Given que o LedgerDatabase contém registros persistidos pelo serviço de registro
    And que o mecanismo de hash chain está ativo

  # AC-1 e AC-2: Hash calculado e encadeado corretamente
  @happy-path @ac-1 @ac-2
  Scenario: Sequência de registros possui hash e encadeamento calculados corretamente
    Given que três eventos de auditoria foram persistidos em sequência
    When os campos de integridade de cada registro são verificados
    Then cada registro possui hash calculado sobre seu conteúdo combinado com o hash do registro anterior
    And o previous_hash de cada registro corresponde ao hash do registro imediatamente anterior

  # AC-3: Hash do primeiro registro
  @boundary @ac-3
  Scenario: Primeiro registro da cadeia possui previous_hash preenchido com zeros
    Given que o LedgerDatabase não continha nenhum registro antes
    When o primeiro AuditEvent é persistido
    Then o previous_hash do registro é preenchido com 64 zeros

  # AC-4: Verificação de registro íntegro
  @happy-path @ac-4 @regression
  Scenario: Verificação de registro sem adulteração confirma integridade
    Given que existe um registro no LedgerDatabase sem nenhuma alteração após a gravação
    When o perito solicita a verificação de integridade desse registro
    Then o resultado confirma que o registro é íntegro

  # AC-5: Adulteração detectada no payload
  @exception @ac-5 @regression
  Scenario: Modificação direta no raw_payload invalida a verificação do registro adulterado
    Given que o conteúdo de um registro foi modificado diretamente no banco de dados
    When o perito solicita a verificação de integridade desse registro
    Then o resultado indica que o registro foi adulterado

  # AC-6: Adulteração detectada via hash quebrado na cadeia
  @exception @ac-6 @regression
  Scenario: Modificação direta no hash de um registro invalida o registro seguinte na cadeia
    Given que o hash de um registro foi modificado diretamente no banco de dados
    When o perito solicita a verificação da cadeia de integridade
    Then o resultado identifica o registro onde a cadeia foi quebrada

  # AC-7: Verificação de faixa íntegra
  @happy-path @ac-7
  Scenario: Verificação de faixa sem adulterações confirma integridade de todos os registros
    Given que uma faixa de registros existe sem nenhuma alteração após a gravação
    When o perito solicita a verificação de integridade da faixa completa
    Then o resultado confirma que todos os registros são íntegros
    And informa quantos registros foram verificados

  # AC-7: Verificação de faixa com adulteração
  @exception @ac-7 @regression
  Scenario: Verificação de faixa com adulteração identifica exatamente o ponto de quebra
    Given que um registro em uma faixa foi adulterado após a gravação
    When o perito solicita a verificação de integridade da faixa
    Then o resultado indica que a cadeia foi quebrada
    And identifica o registro responsável pela quebra

  # AC-8: Hash não impacta SLA de ingestão
  @boundary @ac-8 @regression
  Scenario: Cálculo de hash não degrada a latência do serviço de ingestão
    Given que o mecanismo de hash chain está ativo
    When 50 requisições de ingestão são enviadas simultaneamente
    Then o percentil 99 de latência da ingestão permanece abaixo de 100 milissegundos
```

## Priority

1

## Risk

1 — Alto. Implementação incorreta do hash chain (ex: ordenação não determinística, serialização JSON não canônica) gera hashes inconsistentes que comprometem toda a cadeia e, por consequência, a validade jurídica das evidências. Testar exaustivamente antes de habilitar em produção com dados reais.

## Effort

3 — M
