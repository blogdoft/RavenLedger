# Terça-feira — Personas e Features

## O que são Personas?

Elenca usuários e objetivos desse usuário. Abre caminho para discutir jornadas e features focadas nos usuários do produto e nas necessidades identificadas.

---

## Persona 1: Letícia

**Perfil:** 36 anos, sem filhos. Perfeccionista, analítica e resiliente (paciente). Conhece os processos internos da empresa detalhadamente. Pragmática e muito comprometida com o trabalho. Não coloca amizades acima dos seus deveres. Não abre exceções. Documenta absolutamente tudo.

**Papel:** Auditora de compliance interna.

### Objetivos (O que precisa fazer?)
- Garantir conformidade absoluta com normas internas e externas
- Verificar rastreabilidade completa dos dados
- Identificar inconsistências, falhas de controle e desvios
- Produzir evidências documentais robustas
- Formalizar não conformidades
- Evitar riscos regulatórios e reputacionais
- Manter histórico detalhado

### O que ela vê?
- Processos que nem sempre seguem o padrão definido
- Sistemas que às vezes não registram tudo
- Pressão por "agilidade" acima de controle
- Pessoas tentando justificar exceções
- Falhas de documentação
- Dados inconsistentes entre sistemas
- Retrabalho causado por ausência de governança

### O que ela escuta?
- "Isso sempre foi feito assim."
- "Foi só dessa vez."
- "Depois a gente regulariza."
- "Não precisa formalizar isso."
- "Confia em mim."
- "O sistema não deixa extrair isso."

### O que ela faz?
- Documenta absolutamente tudo
- Solicita evidências formais
- Cruza dados entre sistemas
- Analisa logs e históricos
- Questiona inconsistências
- Rejeita justificativas sem base documental
- Formaliza apontamentos de auditoria
- Mantém registros organizados e rastreáveis

### O que ela fala?
- "Onde está a evidência?"
- "Isso está documentado?"
- "Qual é o procedimento oficial?"
- "Não podemos abrir exceção."
- "Preciso da trilha completa."
- "Sem registro, não aconteceu."
- "Mostre o log."

### Dores
- Irritação com informalidade
- Frustração com falta de rastreabilidade
- Desconfiança quando não há evidência
- Sobrecarga por ter que validar tudo manualmente
- Risco reputacional por falhas que não são dela
- Sensação de que controles são vistos como "burocracia"
- Auditorias levam muito tempo para serem executadas

### Ganhos
- Segurança quando há trilha auditável clara
- Satisfação ao identificar e corrigir falhas
- Confiança em sistemas que registram tudo automaticamente
- Redução de retrabalho
- Clareza e padronização de processos
- Autoridade técnica reconhecida

---

## Persona 2: De Marco

**Perfil:** 35 anos, pai de 1 filho. Perfeccionista e analítico. Boa experiência com tecnologia. Profissional liberal, ganha por perícia. Formação universitária.

**Papel:** Perito judicial (profissional liberal).

### Objetivos (O que precisa fazer?)
- Entender a demanda judicial, identificando escopo e riscos técnicos
- Coletar e preservar evidências (bancos de dados, logs e demais fontes)
- Garantir cadeia de custódia das evidências catalogadas
- Fazer análise técnica dos dados
- Produzir o laudo técnico e gerenciar sua própria agenda

### O que ele vê?
- Judiciário lento e sobrecarregado
- Empresas com sistemas desatualizados e repletos de vulnerabilidades (complexa coleta de evidências)
- Tecnologia desenvolvendo rapidamente, com soluções nem sempre fáceis de usar
- Concorrência que utiliza métodos variados

### O que ele escuta?
- Judiciário pedindo clareza e assertividade sobre o laudo e a fundamentação técnica
- Cobranças sobre prazo
- Questionamentos sobre metodologia, isenção e nexo causal
- Desculpas para não fornecer dados necessários ("mudou ERP", "dados foram sobrescritos", "sistema é caixa preta")

### O que ele faz?
- Organiza o caos de dados com apoio de planilhas e outras ferramentas, normalizando e documentando inconsistências
- Cria validações próprias, submetendo dados a scripts ou validadores feitos por ele mesmo
- Documenta o passo-a-passo da metodologia aplicada
- Questiona formalmente as partes

### O que ele fala?
- "Minha análise é baseada exclusivamente nos dados apresentados."
- "A metodologia está descrita no laudo."
- "Qualquer parte pode reproduzir os cálculos."
- "Eu trabalho com evidência, não com opinião."
- "Se o dado não estiver íntegro, eu registro essa limitação."
- "O sistema da empresa não mantém histórico."

### Dores
- Medo de ter laudo impugnado
- Erro metodológico que comprometa reputação
- Não existe trilha de auditoria, logs ou insumos para construir um caso
- Pressão sobre prazos e assertividade

### Ganhos
- Segurança técnica para sustentar o laudo
- Reprodutividade dos dados
- Reduzir tempo de investigação manual
- Padronizar estrutura de laudos
- Ser visto como perito moderno, antenado nas novidades tecnológicas
- Ter controle total dos dados periciados
- Ter mais tempo com a família

---

## Features

O que o produto deve fazer para atingir os objetivos de negócio e as necessidades das personas?

| Feature                  | Valor Gerado                                                  | Personas Impactadas    |
|--------------------------|---------------------------------------------------------------|------------------------|
| Trilha de auditoria      | Analises rápidas de log de auditoria                         | Letícia                |
| Rastreabilidade          | Contexto de modificação e mapeamento de efeitos colaterais   | Letícia, De Marco      |
| Registro Automático      | Armazenar histórico de pesquisa, tornando mais ágil sua reprodução | De Marco          |
| Registro Imutável        | Confiabilidade nos dados auditados                           | —                      |
| Garantir Inviolabilidade | Permite responsabilizar o autor das modificações             | Letícia                |
| Abertura de investigação | Ter mais de um histórico armazenado; Anexar evidências       | Letícia, De Marco      |
