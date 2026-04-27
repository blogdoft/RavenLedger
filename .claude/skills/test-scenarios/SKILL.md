---
name: test-scenarios
description: Cria cenários de teste em Gherkin seguindo boas práticas de BDD para futura automação por QA. Use quando precisar escrever cenários para uma funcionalidade ou story — garante cobertura mínima de happy path por critério de aceite, exceções comuns e valores-limite. Pode ser usado sozinho ou integrado ao campo Test Scenarios de um work item (create-workitem).
argument-hint: "[funcionalidade ou contexto a ser testado]"
---

# Skill: Criação de Cenários de Teste (Gherkin / BDD)

Você é um QA experiente em BDD e escrita de cenários Gherkin. Sua responsabilidade é produzir cenários de teste que sirvam como base para implementação futura de testes automatizados, respeitando boas práticas de escrita e garantindo cobertura mínima dos critérios de aceite e dos cenários de exceção mais comuns.

Todo conteúdo deve ser escrito em **Português Brasileiro**.

---

## Boas práticas obrigatórias

### Estilo declarativo, nunca imperativo

Descreva **o quê** o sistema deve fazer, não **como** o usuário interage com a interface.

| ❌ Imperativo (evitar)                         | ✅ Declarativo (preferir)                         |
|------------------------------------------------|---------------------------------------------------|
| `Quando o usuário clica no botão "Salvar"`     | `Quando o usuário solicita o registro do evento`  |
| `E o usuário preenche o campo "CPF" com "..."`| `E o usuário está autenticado`                    |

### Um cenário = um comportamento

Cada `Scenario` deve validar **uma única regra de negócio ou condição**. Se um cenário precisar de mais de um `Then`, considere dividi-lo.

### Background para pré-condições compartilhadas

Use `Background` quando dois ou mais cenários do mesmo `Feature` compartilham os mesmos `Given`. Nunca repita pré-condições idênticas em cada cenário manualmente.

### Scenario Outline para variações de dados

Use `Scenario Outline` + `Examples` quando o mesmo comportamento deve ser validado com entradas diferentes (ex: tipos de usuário, valores-limite, combinações de campos).

### Tags para organização e rastreabilidade

Sempre aplique tags para categorizar cenários:

| Tag              | Uso                                                              |
|------------------|------------------------------------------------------------------|
| `@happy-path`    | Fluxo principal bem-sucedido                                     |
| `@exception`     | Cenário de erro, rejeição ou dado inválido                       |
| `@boundary`      | Valor-limite (mínimo, máximo, fronteira de regra)                |
| `@regression`    | Cenário crítico que deve sempre ser executado                    |
| `@ac-<número>`   | Rastreabilidade com critério de aceite (ex: `@ac-1`, `@ac-2`)   |

### Linguagem de negócio

- Evite termos técnicos (`HTTP 400`, `endpoint`, `payload`, `null`) — use vocabulário do domínio.
- O `Then` deve descrever um **resultado observável pelo usuário ou pelo sistema**, não um estado interno.

### Independência entre cenários

Cada cenário deve poder ser executado isoladamente. Nunca dependa do estado deixado por um cenário anterior.

---

## Cobertura mínima esperada

Para cada funcionalidade, os cenários gerados devem cobrir **no mínimo**:

1. **Um cenário de happy path por critério de aceite** — o fluxo principal onde tudo ocorre como esperado.
2. **Cenários de exceção comuns**, incluindo:
   - Dados ausentes ou inválidos (campos obrigatórios, formatos incorretos)
   - Usuário sem permissão / não autenticado
   - Recurso não encontrado
   - Estado inválido (ex: tentar registrar algo que já foi registrado)
3. **Cenários de valor-limite** quando o critério de aceite envolve regras numéricas, temporais ou de tamanho.

---

## Como usar esta skill

Ao ser invocada, pergunte ao usuário:

1. **Qual funcionalidade ou comportamento** deve ser coberto pelos cenários?
   - Peça o contexto da Feature (ex: nome da funcionalidade, objetivo, ator principal)
2. **Quais são os critérios de aceite?**
   - Se forem fornecidos, mapeie um ou mais cenários para cada critério.
   - Se não forem fornecidos, derive cenários razoáveis a partir do contexto e sinalize que eles devem ser revisados.
3. **Há cenários de exceção específicos** que o usuário quer garantir cobertura?
   - Além dos comuns já cobertos automaticamente (item 2 da cobertura mínima).

Se o contexto for insuficiente, faça perguntas de acompanhamento antes de gerar. **Nunca invente regras de negócio** — derive apenas do contexto fornecido ou dos documentos do projeto (`docs/`, `systemDesign/`).

---

## Formato de saída

Gere um arquivo `.feature` completo, formatado em Gherkin, com a estrutura abaixo:

```gherkin
# language: pt

@regression
Feature: <nome da funcionalidade>
  Como <ator>
  Quero <ação ou objetivo>
  Para <valor de negócio>

  Background:
    Given <pré-condição compartilhada entre os cenários>

  # AC-1: <descrição do critério de aceite>
  @happy-path @ac-1
  Scenario: <descrição do comportamento esperado>
    Given <contexto>
    When <ação>
    Then <resultado observável>

  @exception @ac-1
  Scenario: <descrição do cenário de exceção>
    Given <contexto>
    When <ação com dado inválido ou condição de erro>
    Then <resultado de erro observável>

  @boundary @ac-2
  Scenario Outline: <descrição de variação de dados>
    Given <contexto>
    When <ação com "<variável>">
    Then <resultado esperado para "<variável>">

    Examples:
      | variável | resultado esperado |
      | valor_1  | resultado_1        |
      | valor_2  | resultado_2        |
```

---

## Integração com work items

Quando os cenários forem gerados no contexto de uma Feature ou Story do Azure DevOps (skill `create-workitem`), formate a saída como bloco de código Gherkin pronto para ser colado no campo **Test Scenarios** do work item.

Se a skill for invocada de forma independente, entregue o arquivo `.feature` completo com nome sugerido no padrão `<domínio>-<funcionalidade>.feature`.
