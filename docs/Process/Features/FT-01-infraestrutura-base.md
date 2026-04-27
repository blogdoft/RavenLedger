# [FEATURE] FT-01 — Infraestrutura Base e Scaffolding dos Repositórios

## Descrição

Esta feature técnica provisiona toda a base necessária para que os serviços do RavenLedger possam operar em ambiente local e para que os repositórios estejam prontos para desenvolvimento com IA (Claude Code). Abrange a criação e inicialização dos quatro repositórios de serviço, o ambiente local completo via Docker Compose — cobrindo todas as dependências da stack definida — e a configuração inicial de análise estática, gestão de segredos e suporte ao Claude Code em cada repositório.

Sem esta feature, nenhum dos outros serviços consegue subir em ambiente local e nenhum repositório possui a estrutura mínima para iniciar o desenvolvimento. Ela é o pré-requisito de todas as demais features do MVP.

**Cenários de operação:**
- **Happy path:** `docker compose up` sobe todos os containers; bancos criados automaticamente e vazios; Kafka operacional com tópico criado; KeyCloak e OpenBao acessíveis; stack de observabilidade ativa.
- **Erro de configuração:** Variável de ambiente ausente → container não sobe com mensagem clara no log; o `.env.template` documenta o nome e o propósito da variável faltante.
- **Re-execução:** `docker compose up` é idempotente — reexecutar não recria nem corrompe dados existentes.

---

## Descrição Técnica

### Repositórios a criar

| Repositório | Propósito |
|---|---|
| `raven-ledger.infra` | Infraestrutura local: Docker Compose, configurações de ambiente e documentação de setup |
| `raven-ledger.ingestion` | Serviço de ingestão de eventos de auditoria — scaffolding inicial vazio |
| `raven-ledger.register` | Serviço de registro e persistência de eventos — scaffolding inicial vazio |
| `raven-ledger.api` | Serviço de API de consulta — scaffolding inicial vazio |

---

### `raven-ledger.infra` — Ambiente local (Docker Compose)

O `docker-compose.yml` deve cobrir todos os serviços necessários para o ambiente de desenvolvimento local, com healthchecks em todos os containers.

#### Dependências de aplicação

| Serviço | Imagem base | Configuração mínima |
|---|---|---|
| `postgres-ledger` | PostgreSQL | Banco `LedgerDatabase` criado automaticamente, vazio, sem tabelas |
| `postgres-config` | PostgreSQL | Banco `RavenConfig` criado automaticamente, vazio, sem tabelas |
| `kafka` | Apache Kafka (KRaft, sem Zookeeper) | Tópico `audit.events.received` criado automaticamente na inicialização |
| `keycloak` | KeyCloak | Realm inicial configurado para o RavenLedger; acessível para emissão de tokens de serviço |
| `openbao` | OpenBao | Modo dev; acessível para leitura de segredos em ambiente local |

> Os schemas de cada banco (tabelas, índices, permissões) são responsabilidade de cada serviço via FluentMigrator e serão definidos nas features correspondentes.

#### Stack de observabilidade

| Serviço | Propósito |
|---|---|
| `grafana-alloy` | Coletor OTLP — recebe métricas, traces e logs dos serviços e encaminha para os backends |
| `loki` | Armazenamento de logs estruturados em JSON (emitidos via Serilog) |
| `prometheus` | Armazenamento de séries temporais de métricas |
| `tempo` | Armazenamento de traces distribuídos |
| `grafana` | Visualização unificada; datasources Loki, Prometheus e Tempo pré-configurados |

#### Análise estática local

| Serviço | Propósito |
|---|---|
| `sonarqube` | Quality gate local — permite rodar análise estática antes do push, sem depender do CI |

#### Outros artefatos de `raven-ledger.infra`

- `.env.template` — documenta **todas** as variáveis de ambiente necessárias para todos os serviços, com descrição do propósito de cada variável e o container que a consome.
- `README.md` — instruções de como subir o ambiente local, pré-requisitos e troubleshooting comum.
- `CLAUDE.md` — instruções do repositório para o Claude Code, cobrindo: propósito do repositório, convenções do projeto, referência ao `docs/Technical Constraints.md` e ao `docs/TechStack.md`.

---

### Repositórios de serviço — scaffolding inicial

Cada um dos repositórios de serviço (`.ingestion`, `.register`, `.api`) deve ser inicializado com a estrutura mínima abaixo. **Nenhum código de negócio deve ser escrito nesta feature.**

#### Estrutura de solução .NET 10

Cada repositório recebe uma solução `.sln` vazia com a estrutura de pastas padrão do projeto:

```
src/
tests/
```

#### Análise estática — build-time (StyleCop + Roslynator)

- `Directory.Build.props` na raiz do repositório referenciando os pacotes **StyleCop.Analyzers** e **Roslynator.Analyzers** como dependências de análise, aplicadas a todos os projetos da solução.
- `.editorconfig` com as regras de estilo C# alinhadas às convenções do StyleCop.
- A build deve quebrar em qualquer violação de análise estática.

#### Análise estática — CI (SonarQube)

- `sonar-project.properties` (ou equivalente) configurado com a chave do projeto, nome e apontando para os diretórios de cobertura do Coverlet.
- A configuração deve ser suficiente para executar `dotnet sonarscanner begin` / `end` no pipeline de CI.

#### Gestão de segredos

- `.env.template` com as variáveis de ambiente esperadas pelo serviço (nomes e descrições, sem valores).
- `.gitignore` incluindo `.env` explicitamente.

#### Pipeline de CI (GitHub Actions)

Cada repositório de serviço recebe um arquivo `.github/workflows/ci.yml` como parte do scaffolding. A pipeline é o quality gate obrigatório antes de qualquer merge — nenhum PR pode ser integrado sem que ela passe.

**Gatilhos:**

| Evento | Branch alvo | Propósito |
|---|---|---|
| `pull_request` | `develop` | Valida o código antes do merge — bloqueia o PR se a pipeline falhar |
| `push` | `develop` | Valida o estado integrado após o merge |

**Etapas da pipeline (em ordem):**

| Etapa | Ferramenta | Critério de falha |
|---|---|---|
| Restore | `dotnet restore` | Dependências não resolvidas |
| Build | `dotnet build` | Erro de compilação ou violação de StyleCop/Roslynator |
| Test + Coverage | `dotnet test` + Coverlet | Falha em qualquer teste |
| SonarQube Analysis | `dotnet sonarscanner` | Quality gate do SonarQube não aprovado |

> A etapa de SonarQube engloba `begin` (antes do build) e `end` (após os testes), garantindo que a cobertura coletada pelo Coverlet seja enviada para análise.

**Secrets necessários no repositório GitHub:**

| Secret | Propósito |
|---|---|
| `SONAR_TOKEN` | Token de autenticação para o SonarQube |
| `SONAR_HOST_URL` | URL do servidor SonarQube (local em dev, instância dedicada em CI) |

> Os valores dos secrets devem ser documentados no `.env.template` do repositório, mas nunca versionados.

#### Suporte ao Claude Code

- `CLAUDE.md` na raiz do repositório, contendo:
  - Propósito do serviço (1–2 parágrafos).
  - Referência explícita ao `docs/Technical Constraints.md` e ao `docs/TechStack.md` do repositório de documentação.
  - Convenções específicas do serviço que o Claude Code deve respeitar (a serem detalhadas nas features de cada serviço).

---

## Critérios de Aceite

**Repositórios:**
- [ ] Repositórios `raven-ledger.infra`, `raven-ledger.ingestion`, `raven-ledger.register` e `raven-ledger.api` criados e inicializados no GitHub

**Ambiente local:**
- [ ] `docker compose up` sobe sem erros e todos os containers ficam healthy em < 60 segundos
- [ ] `LedgerDatabase` acessível e vazio (sem tabelas de negócio)
- [ ] `RavenConfig` acessível e vazio (sem tabelas de negócio)
- [ ] Kafka operacional com tópico `audit.events.received` criado
- [ ] KeyCloak acessível e com realm inicial configurado
- [ ] OpenBao acessível em modo dev
- [ ] Grafana acessível com datasources Loki, Prometheus e Tempo pré-configurados
- [ ] SonarQube acessível localmente
- [ ] `.env.template` documenta todas as variáveis de ambiente necessárias
- [ ] `README.md` descreve como subir o ambiente local

**Scaffolding dos serviços:**
- [ ] Cada repositório de serviço possui solução `.sln` com estrutura `src/` e `tests/`
- [ ] `Directory.Build.props` com StyleCop e Roslynator configurados — build quebra em violações
- [ ] `.editorconfig` com regras C# alinhadas ao StyleCop
- [ ] `sonar-project.properties` configurado para integração com SonarQube no CI
- [ ] `.env.template` criado em cada repositório de serviço
- [ ] `.gitignore` inclui `.env` explicitamente em cada repositório

**Pipeline de CI:**
- [ ] `.github/workflows/ci.yml` criado em cada repositório de serviço (`.ingestion`, `.register`, `.api`)
- [ ] Pipeline disparada em `pull_request` para `develop` e em `push` para `develop`
- [ ] Etapas configuradas na ordem: restore → build → test + coverage → SonarQube analysis
- [ ] PR bloqueado automaticamente quando a pipeline falha
- [ ] Secrets `SONAR_TOKEN` e `SONAR_HOST_URL` documentados no `.env.template` de cada repositório

**Claude Code:**
- [ ] `CLAUDE.md` criado em todos os quatro repositórios com propósito e referências às restrições do projeto

---

## Cenários de Teste

```gherkin
# language: pt

@regression
Feature: Infraestrutura base do ambiente local
  Como desenvolvedor do RavenLedger
  Quero subir o ambiente local com um único comando
  Para que todos os serviços possam operar de forma integrada

  Background:
    Given que todas as variáveis de ambiente obrigatórias estão configuradas

  # AC: Ambiente local sobe sem erros e todos os containers ficam healthy
  @happy-path
  Scenario: Subida do ambiente local a partir do zero
    Given que nenhum container do RavenLedger está em execução
    When o ambiente local é iniciado com docker compose up
    Then todos os containers ficam healthy em menos de 60 segundos

  # AC: LedgerDatabase acessível e vazio
  @happy-path
  Scenario: LedgerDatabase criado e acessível sem tabelas de negócio
    Given que o ambiente local está em execução
    When uma conexão é estabelecida com o LedgerDatabase
    Then o banco existe e não contém nenhuma tabela de negócio

  # AC: RavenConfig acessível e vazio
  @happy-path
  Scenario: RavenConfig criado e acessível sem tabelas de negócio
    Given que o ambiente local está em execução
    When uma conexão é estabelecida com o RavenConfig
    Then o banco existe e não contém nenhuma tabela de negócio

  # AC: Kafka operacional com tópico criado
  @happy-path
  Scenario: Tópico audit.events.received disponível no Kafka
    Given que o ambiente local está em execução
    When uma mensagem é publicada no tópico audit.events.received
    Then a mensagem está disponível para consumo

  # AC: KeyCloak acessível
  @happy-path
  Scenario: KeyCloak acessível e com realm configurado
    Given que o ambiente local está em execução
    When o endpoint de descoberta do KeyCloak é consultado
    Then o realm do RavenLedger está disponível

  # AC: Stack de observabilidade operacional
  @happy-path
  Scenario: Grafana acessível com datasources configurados
    Given que o ambiente local está em execução
    When o Grafana é acessado
    Then os datasources Loki, Prometheus e Tempo estão configurados e respondendo

  # AC: SonarQube acessível
  @happy-path
  Scenario: SonarQube acessível para análise local
    Given que o ambiente local está em execução
    When o SonarQube é acessado
    Then a interface está disponível e pronta para receber análises

  # AC: Variável de ambiente obrigatória ausente impede subida
  @exception
  Scenario: Variável de ambiente obrigatória ausente impede subida do container
    Given que uma variável de ambiente obrigatória não está configurada
    When o ambiente local é iniciado
    Then o container correspondente não sobe
    And o log exibe mensagem de erro identificando a variável ausente

  # AC: Re-execução do ambiente é idempotente
  @happy-path
  Scenario: Re-execução do ambiente não corrompe dados existentes
    Given que o ambiente local já está em execução com dados
    When o ambiente local é iniciado novamente com docker compose up
    Then o estado do ambiente permanece inalterado e a inicialização ocorre sem erros

  # AC: Build quebra em violação de análise estática
  @happy-path
  Scenario: Build de serviço falha em violação do StyleCop
    Given que um repositório de serviço está inicializado com StyleCop configurado
    When um arquivo C# com violação de estilo é adicionado e o build é executado
    Then o build falha com mensagem identificando a violação

  # AC: Pipeline de CI disparada em pull request para develop
  @happy-path
  Scenario: Pipeline de CI é executada ao abrir pull request para develop
    Given que um repositório de serviço possui o arquivo .github/workflows/ci.yml configurado
    And que um pull request é aberto com destino à branch develop
    When o pull request é submetido
    Then a pipeline de CI é disparada automaticamente
    And o resultado da pipeline é exibido como check no pull request

  # AC: PR bloqueado quando a pipeline falha
  @exception
  Scenario: Pull request bloqueado quando a pipeline de CI falha
    Given que um pull request está aberto com destino à branch develop
    And que a pipeline de CI falhou em alguma etapa
    When um revisor tenta aprovar o merge do pull request
    Then o merge é bloqueado pelo GitHub até que a pipeline passe

  # AC: Pipeline executada após merge em develop
  @happy-path
  Scenario: Pipeline de CI executada após merge em develop
    Given que um pull request foi aprovado e mergeado em develop
    When o push para develop é registrado
    Then a pipeline de CI é disparada automaticamente sobre o estado integrado da branch

  # AC: Quality gate do SonarQube integrado à pipeline
  @happy-path
  Scenario: Pipeline falha quando o quality gate do SonarQube não é aprovado
    Given que a pipeline de CI está em execução para um pull request
    When a etapa de análise do SonarQube detecta violação do quality gate
    Then a pipeline falha na etapa de SonarQube
    And o pull request permanece bloqueado para merge
```

---

## Priority

1

## Risk

3 — Médio. A topologia KRaft do Kafka e a configuração inicial do KeyCloak (realm, client, scopes) são os pontos de maior atenção: erros aqui impactam todas as features subsequentes. Os demais componentes são bem documentados e o risco é baixo.

## Effort

3 — M. O docker compose com stack completa (observabilidade + autenticação + segredos) é mais trabalhoso que o setup mínimo original. A configuração do SonarQube local e a inicialização dos quatro repositórios com scaffolding padronizado adicionam volume, mas o trabalho é mecânico e bem documentado.
