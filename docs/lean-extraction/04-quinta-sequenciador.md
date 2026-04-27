# Quinta-feira — Features × Jornada e Sequenciador

## Features × Jornada

Em cada etapa da jornada, liga-se uma funcionalidade que possa ser útil ao usuário naquele momento.

| Etapa da Jornada      | Features relevantes                                      |
|-----------------------|----------------------------------------------------------|
| Acionamento           | Abertura de investigação                                 |
| Apreciação do Caso    | Trilha de Auditoria, Rastreabilidade                    |
| Análise de Histórico  | Registro Automático, Rastreabilidade                    |
| Coleta de evidência   | Garantir Inviolabilidade, Registro Imutável             |
| Construção do Laudo   | Trilha de Auditoria, Rastreabilidade, Registro Imutável |

---

## Sequenciador

O sequenciador organiza as features em **waves** (ondas de entrega), respeitando as seguintes regras:

- **Regra 1:** Uma wave pode conter no máximo três cards.
- **Regra 2:** Uma wave não pode conter mais de um card vermelho.
- **Regra 3:** Uma wave não pode conter três cards se nenhum for verde.
- **Regra 4:** A quantidade de esforço nos cards não pode exceder cinco Es.
- **Regra 5:** A soma do valor dos cards de uma wave não pode ser apenas $.

### Waves definidas

#### Wave 1 — MVP
| Feature             | Esforço | Negócio | UX       |
|---------------------|---------|---------|----------|
| Registro Automático | EEE     | $$$     | ❤️❤️❤️ |
| Registro Imutável   | E       | $$$     | ❤️❤️❤️ |
| Trilha de Auditoria | E       | $$$     | ❤️      |

#### Wave 2
| Feature                  | Esforço | Negócio | UX       |
|--------------------------|---------|---------|----------|
| Garantir Inviolabilidade | EE      | $$$     | ❤️❤️❤️ |
| Rastreabilidade          | EE      | $       | ❤️❤️❤️ |

#### Wave 3
| Feature                  | Esforço | Negócio | UX       |
|--------------------------|---------|---------|----------|
| Abertura de investigação | EEE     | $$      | ❤️❤️❤️ |
