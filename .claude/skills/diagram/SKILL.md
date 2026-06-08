---
name: diagram
description: Gera diagramas usando Mermaid sempre que um diagrama for necessário — fluxos, sequências, entidade-relacionamento, arquitetura, etc. Aplica obrigatoriamente a convenção de quebra de linha com `<br/>` em vez de `\n` dentro de labels e textos do Mermaid.
argument-hint: "[tipo de diagrama e contexto — ex: fluxo de autenticação, diagrama de sequência do evento de auditoria]"
---

# Skill: Diagramas com Mermaid

Você é especialista em comunicação técnica visual. Sempre que um diagrama for necessário — seja para documentação, explicação de fluxo, arquitetura, modelo de dados ou qualquer outra finalidade — use **Mermaid** como formato padrão.

Todo conteúdo textual dos diagramas deve seguir o idioma do contexto: **Português Brasileiro** para documentação deste projeto.

---

## Regra obrigatória: quebras de linha

Dentro de **qualquer texto, label ou nota** em Mermaid, use **`<br/>`** para representar quebras de linha.

**Nunca** use `\n` — o Mermaid não interpreta `\n` como quebra de linha em labels.

```
✅ correto:   A["Linha 1<br/>Linha 2"]
❌ incorreto: A["Linha 1\nLinha 2"]
```

Esta regra se aplica a todos os tipos de nó, edge label, nota (`Note`), legenda, ou qualquer string dentro do bloco Mermaid.

---

## Tipos de diagrama e quando usar

| Tipo                  | Keyword Mermaid       | Usar quando                                                        |
|-----------------------|-----------------------|--------------------------------------------------------------------|
| Fluxo / Processo      | `flowchart`           | Decisões, ramificações, pipelines, fluxos de negócio              |
| Sequência             | `sequenceDiagram`     | Interações entre serviços, chamadas de API, protocolos            |
| Entidade-Relacionamento | `erDiagram`         | Modelo de dados, schema de banco                                   |
| Classes               | `classDiagram`        | Hierarquias de domínio, contratos de interface                    |
| Estado                | `stateDiagram-v2`     | Máquinas de estado, ciclo de vida de entidades                    |
| Gantt                 | `gantt`               | Cronogramas, planejamento de sprints                              |
| Git / Branches        | `gitGraph`            | Estratégias de branching, histórico de releases                   |
| Mindmap               | `mindmap`             | Mapeamentos conceituais, brainstorming                            |
| Quadrante             | `quadrantChart`       | Priorização, análise comparativa                                  |
| Bloco (C4-like)       | `block-beta`          | Arquitetura de alto nível, visão de componentes                   |

Escolha o tipo que melhor representa a informação — prefira simplicidade e legibilidade a diagramas sobrecarregados.

---

## Boas práticas

### Legibilidade acima de completude
Um diagrama com menos elementos mas fácil de ler é mais valioso do que um diagrama completo e incompreensível. Omita detalhes que não agregam ao ponto que está sendo comunicado.

### Nomeie os nós com o vocabulário do domínio
Use os termos do negócio, não nomes técnicos de implementação, a não ser que o diagrama seja explicitamente técnico.

### Direcionalidade em `flowchart`
- `flowchart TD` — top-down (padrão para fluxos de processo)
- `flowchart LR` — left-right (bom para pipelines horizontais)

### Agrupamentos com `subgraph`
Use `subgraph` para agrupar responsabilidades ou contextos delimitados (bounded contexts, camadas de arquitetura, serviços). Inclua um título descritivo.

### Estilos (`style`, `classDef`)
Use `classDef` para aplicar estilos consistentes a grupos de nós (ex: destaque de componentes externos, entidades críticas). Evite estilos desnecessários que poluam o código.

---

## Como usar esta skill

Ao ser invocada, pergunte ao usuário (se não fornecido no argumento):

1. **O que deve ser representado?** — descrição do fluxo, modelo ou arquitetura.
2. **Qual o nível de detalhe esperado?** — visão geral (alto nível) ou detalhamento técnico?
3. **Há restrições de escopo?** — ex: apenas um serviço, apenas a camada de persistência.

Se o contexto for suficiente (argumento ou conversa anterior), gere diretamente sem perguntas adicionais.

**Nunca invente comportamentos ou componentes** que não tenham sido mencionados pelo usuário ou que não constem nos documentos do projeto (`docs/`, `systemDesign/`, `roteiro/`).

---

## Formato de saída

Entregue o diagrama em um bloco de código Mermaid:

````markdown
```mermaid
<tipo>
  <conteúdo do diagrama>
```
````

Quando relevante, inclua uma breve descrição em prosa (1–3 frases) **antes** do bloco explicando o que o diagrama representa e qual decisão ou trade-off ele ilustra.

Se o diagrama for destinado a um work item do Azure DevOps (campo **Technical Description**), formate-o pronto para colar, com o bloco Mermaid e a descrição contextual.
