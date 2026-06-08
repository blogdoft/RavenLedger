# Segunda-feira — Visão do Produto + É/Não É / Faz/Não Faz

## Visão do Produto

**Para** peritos em geral / compliance
**cujo** interesse é identificar quem, o quê, quando e porquê determinada manipulação de dados ocorreu,
**o RavenLedger** é um Sistema de Auditoria de Dados/TI, via browser
**que** permite a identificação e localização de dados alterados por processos ou pessoas.
**Diferente do** EntityFramework,
**nosso produto** oferece uma interface amigável para o usuário e seus processos internos garantem a legitimidade dos dados auditados.

## Definição do Produto

**O que é?** Define características básicas do produto.
**Objetivo:** Delimitar escopo do produto.

### É

- MicroServiço
- SaaS
- OnPremisse
- Intuitivo
- Seguro
- Rápido
- WebApp
- Bonito
- OpenSource
- ReadOnly

### Não É

- Mobile
- Backup
- Stream de dados / Log
- Sistema Anti-Fraude
- Um data lake (analytics)
- Replicador de dados

## Faz / Não Faz

### FAZ

- Armazena histórico de modificações
- Identifica responsáveis pelas modificações
- Possibilita visualizar a cadeia de modificações
- Garante imutabilidade dos dados auditados
- Identifica possíveis alterações na auditoria
- Relatórios para auditoria/compliance
- Permite encontrar um registro com base em seus dados e metadados

### NÃO FAZ

- Não garante a transmissão de correlationId
- Replica informações para outras fontes
- Relatórios gerenciais
- Análise de dados
