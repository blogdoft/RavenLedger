# Tech Stack

Este documento define a stack de softwares utilizadas na construção do Raven Ledger e suas justificativas.

---

## Backend — .NET 10

A plataforma principal de desenvolvimento dos serviços é o **.NET 10**.

**Justificativa:** plataforma madura, com excelente suporte a aplicações HTTP de alta performance, ecossistema rico de bibliotecas e curva de aprendizado acessível para o público-alvo do blog.

### Bibliotecas de Aplicação

| Biblioteca | Papel |
|---|---|
| **Swashbuckle.AspNetCore** | Geração automática de documentação OpenAPI/Swagger a partir das controllers |
| **Serilog** | Logging estruturado em JSON — facilita ingestão por ferramentas de observabilidade |
| **Confluent.Kafka** | Client oficial do Kafka para .NET; conexão e consumo de tópicos |
| **Dapper** | Micro-ORM: mapeamento de queries SQL para objetos sem abstração excessiva |
| **Npgsql** | Driver ADO.NET nativo para PostgreSQL; usado como provider do Dapper |
| **FluentMigrator** | Migrations versionadas via código C#; cada serviço gerencia seu próprio banco |

### Bibliotecas de Teste

| Biblioteca | Papel |
|---|---|
| **Bogus** | Geração de dados aleatórios e realistas para fixtures de teste |
| **NSubstitute** | Framework de mocks e stubs com API fluente para .NET |
| **AutoBogus.NSubstitute** | Integração entre Bogus e NSubstitute para geração automática de fakes |
| **Coverlet.Collector** | Coleta de cobertura de código para integração com relatórios CI |
| **Shouldly** | Biblioteca de asserções com mensagens de erro legíveis |

### Análise Estática

| Ferramenta | Papel |
|---|---|
| **SonarQube** | Analisador estático principal — quality gate de CI/CD para C# e TypeScript; rastreia dívida técnica, cobertura e vulnerabilidades de segurança ao longo do tempo |
| **StyleCop** | Enforce de convenções de estilo e formatação de código C# em build-time via Roslyn |
| **Roslynator** | Análise de qualidade e sugestões de refatoração via Roslyn em build-time |

---

## Frontend — Angular (LTS)

O frontend de visualização de registros é construído com **Angular na versão LTS corrente**.

**Justificativa:** framework consolidado, tipagem forte via TypeScript e modelo de componentes que se alinha bem com a natureza tabular/consultiva da interface do RavenLedger.

---

## Mensageria — Apache Kafka

O broker de mensagens adotado é o **Apache Kafka**.

**Justificativa:** garantia de entrega (at-least-once), retenção configurável de mensagens, alta throughput e modelo de consumer groups — essencial para desacoplar `raven-ledger.ingestion` do `raven-ledger.register` sem perda de eventos de auditoria. O Kafka é acessado pelos serviços .NET via `Confluent.Kafka`.

---

## Banco de Dados — PostgreSQL

O banco relacional adotado é o **PostgreSQL**, operando em modo _append-only_ para o `LedgerDatabase`.

**Justificativa:** banco relacional maduro, mais acessível para novos colaboradores, suficiente para o MVP. Limitações conhecidas (sharding, full-text search, pesquisa em campos alterados) são aceitas conscientemente neste momento.

---

## Autenticação — KeyCloak

A autenticação entre sistemas clientes e o RavenLedger é feita via **KeyCloak**.

**Justificativa:** os sistemas clientes devem obter um token de serviço do KeyCloak para autenticar as chamadas ao endpoint de ingestão, garantindo a rastreabilidade de quem enviou cada evento.

---

## Gestão de Segredos — OpenBao

O armazenamento e a distribuição de segredos é responsabilidade do **OpenBao** (fork open-source do HashiCorp Vault).

**Justificativa:** nenhum segredo é versionado — nem em ambiente local. A política é:

- **Ambiente produtivo:** todos os segredos são fornecidos pelo OpenBao em runtime, independente da camada.
- **Ambiente local:** a estratégia varia por componente:

| Camada | Configuração | Segredos |
|---|---|---|
| **Serviços .NET** | `appsettings.Development.json` (versionado, apenas config não-sensível) | ASP.NET Core User Secrets (`dotnet user-secrets`) |
| **Angular** | Arquivo `.env` (nunca versionado) | Arquivo `.env` (nunca versionado) |
| **Docker Compose** | Arquivo `.env` (nunca versionado) | Arquivo `.env` (nunca versionado) |

Serviços .NET não utilizam arquivos `.env`. Para cada `.env` esperado existe um `.env.template` no mesmo nível, com os nomes das variáveis mas sem valores.

---

## Infraestrutura e Deploy

| Ferramenta | Papel |
|---|---|
| **K3D** | Cluster Kubernetes local (K3s dentro do Docker) para desenvolvimento e testes de infraestrutura |
| **ArgoCD** | GitOps — Continuous Deployment declarativo a partir do repositório |
| **GitHub Actions** | Pipelines de CI (build, test, análise estática, cobertura) |

---

## Observabilidade

> **Nota:** esta stack monitora a operação do próprio RavenLedger. Não se confunde com o que o RavenLedger entrega — ele é um sistema de auditoria de dados, não de observabilidade.

| Ferramenta | Papel |
|---|---|
| **OpenTelemetry .NET SDK** | Instrumentação de métricas, traces e logs nos serviços; desacopla a coleta do backend de destino |
| **Grafana Alloy** | Coletor OpenTelemetry (OTLP) e agente de ingestão; encaminha para os backends correspondentes |
| **Loki** | Armazenamento e consulta de logs estruturados — ingere o JSON gerado pelo Serilog |
| **Prometheus** | Armazenamento de séries temporais de métricas |
| **Tempo** | Rastreamento distribuído (distributed tracing) |
| **Grafana** | Visualização unificada de logs, métricas e traces |

**Justificativa:** o Serilog já emite JSON estruturado; com `Serilog.Sinks.OpenTelemetry` os logs saem via OTLP sem refatoração. O OpenTelemetry SDK instrumenta automaticamente ASP.NET Core e HTTP — sem código manual nas camadas de transporte. Todo o stack roda em Kubernetes via Helm charts oficiais, alinhado com o modelo self-hosted do RavenLedger.

## Testes de Sistema

### Testes End-to-End

| Ferramenta | Papel |
|---|---|
| **Selenium** | Automação de testes E2E sobre a interface Angular; valida fluxos completos do usuário via browser |

**Justificativa:** Selenium é a ferramenta consolidada para testes de browser com suporte a múltiplos drivers; permite validar os fluxos críticos da interface de visualização de registros de auditoria de ponta a ponta.

### Testes de Carga

| Ferramenta | Papel |
|---|---|
| **k6** | Testes de carga e performance nos endpoints HTTP; valida SLOs de latência e throughput sob pressão |

**Justificativa:** k6 executa cenários de carga escritos em JavaScript com API declarativa, integra-se ao GitHub Actions para execução em CI e exporta métricas diretamente para o Prometheus/Grafana — stack já adotada no projeto.

---

## Pipelines

### CI

Para o processo de CI, cada projeto irá utilizar um Git Hub Actions, que será responsável por enviar o código para validação durante a etapa de pull request. E também quando o merge em `develop` for executado. 

### CD

Para o processo de CD, um Git Hub Actions será responsável por alterar a versão do arquivo de deployment do repositório RavenLedger.Infrasctructure. Esse repositório irá armazenar todos os deployments dos serviços relativos ao RavenLedger, sendo sincronizado com o ArgoCD.