# [FEATURE] FT-05 — Interface Web para Visualização da Trilha de Auditoria

## Descrição

**Wave 1 — MVP | Lean Inception: Trilha de Auditoria (UX)**

Esta feature implementa a aplicação Web (Angular) que permite a Letícia (compliance) e De Marco (perito judicial) consultarem a trilha de auditoria diretamente pelo browser, sem necessidade de acesso ao banco de dados ou conhecimento técnico de APIs. A interface prioriza clareza e funcionalidade sobre estética — o objetivo do MVP é viabilizar investigações reais.

O MVP Canvas define "via browser" como plataforma de ambas as personas, mas a Lean Inception atribui apenas ❤️ (1 coração) de UX para Trilha de Auditoria — sinalizado que uma interface funcional é suficiente; polimento visual pode vir em iterações posteriores.

**Cenários de negócio:**
- **Happy path:** Letícia acessa a aplicação, filtra registros por entidade e período, visualiza a lista e abre o detalhe de um registro suspeito.
- **Sem resultados:** Filtros aplicados não encontram dados → mensagem clara "Nenhum registro encontrado para os filtros selecionados."
- **Detalhe do registro:** Clique em um registro → visualização completa incluindo o snapshot `data` do payload original.
- **Erro de API:** Serviço `raven-ledger.api` indisponível → mensagem de erro amigável, sem tela em branco.

## Descrição Técnica

**Aplicação:** Angular standalone, sem backend próprio. Consome exclusivamente `raven-ledger.api`.

**Telas:**

**1. Tela de Listagem (`/audit-entries`)**
- Formulário de filtros: `entity`, `domain`, `user_id`, `operation` (select), `from` (datepicker), `to` (datepicker)
- Tabela de resultados com colunas: `entity`, `domain`, `userName`, `operationType`, `generatedAt`, `receivedAt`
- Paginação (next/prev) sincronizada com parâmetros da query
- Botão de limpar filtros
- Estado de carregamento (skeleton ou spinner)
- Estado vazio com mensagem descritiva

**2. Tela de Detalhe (`/audit-entries/:id`)**
- Todos os campos do registro: metadados CloudEvent + campos de auditoria
- Exibição do campo `data` (snapshot da entidade) formatado como JSON com syntax highlight
- Botão "Voltar para listagem" que preserva filtros anteriores
- 404 exibido de forma amigável se ID não encontrado

**Integração com API:**
- Consome `GET /api/v1/audit-entries` com filtros mapeados de query params da URL
- Consome `GET /api/v1/audit-entries/{id}` para detalhe
- URL da API configurável via variável de ambiente (`RAVEN_API_URL`)
- Sem autenticação no MVP (self-hosted/local)

**Responsividade:** Desktop first, mínimo 1280px. Mobile não é requisito do MVP.

**Sem estado de autenticação** no MVP — aplicação acessível diretamente.

## Critérios de Aceite

- [ ] Tela de listagem exibe registros consumindo a API com todos os filtros funcionando
- [ ] Filtros de `entity`, `domain`, `user_id`, `operation` e período funcionam individualmente e combinados
- [ ] Paginação funciona: navega entre páginas e exibe total de registros
- [ ] Tela de detalhe exibe todos os campos do registro incluindo snapshot `data` formatado
- [ ] Estado vazio exibe mensagem "Nenhum registro encontrado para os filtros selecionados"
- [ ] Erro de API exibe mensagem amigável (não tela em branco ou erro técnico)
- [ ] Botão "Voltar" na tela de detalhe preserva filtros da listagem anterior
- [ ] URL da API configurável via variável de ambiente
- [ ] Aplicação renderiza corretamente em 1280px de largura
- [ ] Filtros são refletidos na URL (query params) para permitir compartilhamento de link

## Cenários de Teste

```gherkin
# language: pt

@regression
Feature: Interface web para visualização da trilha de auditoria
  Como analista de compliance ou perito judicial
  Quero consultar a trilha de auditoria pelo navegador
  Para investigar operações sobre dados sem depender de acesso técnico ao banco

  Background:
    Given que a aplicação web está acessível no navegador
    And que o serviço raven-ledger.api está operacional com registros disponíveis

  # AC-1: Listagem com registros
  @happy-path @ac-1
  Scenario: Listagem exibe registros com as colunas esperadas
    Given que o LedgerDatabase contém registros de auditoria
    When o analista acessa a tela de listagem
    Then a tabela exibe os registros com as colunas entity, domain, userName, operationType, generatedAt e receivedAt

  # AC-2: Filtro por entidade
  @happy-path @ac-2
  Scenario: Filtro por entidade atualiza a listagem com apenas os registros correspondentes
    Given que existem registros para a entidade "users" e para outras entidades
    When o analista aplica o filtro de entidade com o valor "users"
    Then a tabela exibe apenas os registros da entidade "users"

  # AC-2: Filtro por tipo de operação
  @happy-path @ac-2
  Scenario Outline: Filtro por tipo de operação exibe apenas as operações correspondentes
    Given que existem registros de diferentes tipos de operação
    When o analista filtra por operação "<operacao>"
    Then a tabela exibe apenas registros com operação "<operacao>"

    Examples:
      | operacao |
      | insert   |
      | update   |
      | delete   |

  # AC-3: Paginação
  @happy-path @ac-3
  Scenario: Paginação navega corretamente entre páginas de resultados
    Given que existem mais registros do que o tamanho de uma página
    When o analista navega para a próxima página
    Then registros diferentes são exibidos e o total de registros permanece apresentado corretamente

  # AC-4: Detalhe do registro
  @happy-path @ac-4
  Scenario: Seleção de registro exibe detalhe completo com snapshot formatado como JSON
    Given que a listagem exibe registros de auditoria
    When o analista seleciona um registro específico
    Then todos os campos do registro são exibidos incluindo o snapshot de dados formatado como JSON

  # AC-5: Estado vazio
  @happy-path @ac-5
  Scenario: Filtros sem resultado exibem mensagem descritiva ao analista
    Given que nenhum registro corresponde aos filtros aplicados
    When o analista aplica os filtros e aguarda a resposta
    Then a interface exibe a mensagem "Nenhum registro encontrado para os filtros selecionados"

  # AC-6: Erro de API
  @exception @ac-6
  Scenario: Indisponibilidade da API exibe mensagem amigável sem expor detalhes técnicos
    Given que o serviço raven-ledger.api está indisponível
    When o analista acessa ou atualiza a listagem
    Then a interface exibe uma mensagem de erro amigável sem tela em branco ou informações técnicas

  # AC-7: Preservação de filtros ao voltar
  @happy-path @ac-7
  Scenario: Retorno da tela de detalhe preserva os filtros anteriormente aplicados na listagem
    Given que o analista aplicou filtros na listagem e abriu o detalhe de um registro
    When o analista retorna para a listagem
    Then os filtros aplicados anteriormente são preservados

  # AC-9: Registro inexistente exibe página amigável
  @exception @ac-4
  Scenario: Acesso a detalhe de registro inexistente exibe mensagem amigável
    Given que o analista navega para o endereço de detalhe de um registro inexistente
    When a tela de detalhe tenta carregar o registro
    Then a interface exibe uma mensagem amigável informando que o registro não foi encontrado

  # AC-10: Filtros refletidos na URL
  @happy-path @ac-10
  Scenario: Filtros aplicados aparecem na URL para permitir compartilhamento de link
    Given que o analista aplica filtros de entidade e período na listagem
    When os filtros são aplicados
    Then os parâmetros correspondentes aos filtros aparecem na URL da página
```

## Priority

2

## Risk

3 — Baixo. Interface puramente de leitura, sem regras de negócio complexas. Consome API estável (FT-04). Risco maior é de UX, não de dados.

## Effort

3 — M
