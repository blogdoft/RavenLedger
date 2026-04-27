# Process

Você é um especialista em gestão de produtos, processos e agilidade.

Com base na lean inception e no system design, crie para mim os épicos, features e stories que servirão de Workitems de implementação do produto.

Eu estou seguindo um modelo ágil de desenvolvimento.

Os worktitems são: Epic, Feature e Stories:

## Epic

Delimitam um grande entregável. Olhando para o Lean Inception, pode ser uma das funcionalidades presentes no sequenciador. Ou até mesmo delimitar toda a entrega do MVP, a depender da complexidade envolvida.

Ele é composto dos campos:

### Description

Descreve detalhadamente o que aquele épico representa em termos de valor para o produto e porque ele é relevante.

### Priority

Qual a prioridade desse WI em relação aos demais.  Varia de 1 até 4, onde 1 é o mais prioritário e o 4 é o menos.

### Risk

Um número entre 1 e 3 (Low, Medium e High) que representa o risco geral da implementação dessa funcionalidade. Aqui são considerados integrações, complexidade das regras, quantidade de serviços envolvidos e etc.

### Effort

Vamos usar a técnica de "Tamanho de camisa" para mensurar o esforço nessa fase do projeto. Os valores aceitáveis são:
- PP 
- P
- M
- G
- GG
com base no lean inception, crie arquivos MD com os
  Epic possíveis dentro de ./docs/Process/Epics. Se você achar que só um    
  Epic é necessário, tudo bem.

### Classification

Classifica o artefato como "Architectural" ou "Business" dependendo do foco desse workitem ou do motivador da sua construção (Engenharia ou Negócio).

## Feature

Delimita uma funcionalidade ou conjunto de funcionalidades que podem ser entregues em produção, sem causar problemas de usabilidade, segurança ou indisponibilidade de ambiente. Funcionalidades podem ser entregues de maneira parcial, desde que sejam ativadas por feature flags.


### Description

Descreve detalhadamente as motivações de negócio e as regras que devem ser implementadas. 
Deve conter explicações para os cenários felizes e os cenários de exceção, bem como mensagens de erro apresentadas ao usuário.
Também deve incluir, se necessário, como será medido o sucesso da feature através de indicadores de negócio.

### Technical Description

Descreve as alterações que deverão ser efetuadas na base de código, serviços a serem criados, logs que devem ser emitidos, componentes a serem criados/utilizados e qualquer outra limitação técnica. Documentação como diagramas, exemplos de código e afins devem vir nesse campo ou referenciados como arquivos anexos.
Quando necessário, descreve as métricas de sucesso/estabilidade da funcionalidade.

### Acceptance Criteria

Critérios de aceite para que a feature seja considerada como pronta para entrega. Pode ser uma lista de itens (preferível) ou cenários de uso obrigatórios.
Se necessário, deve incluir também critérios de aceite de engenharia.

### Test Scenarios

Lista os cenários de teste que garantem que todas regras de negócio estão cobertas.

### Priority

Qual a prioridade desse WI em relação aos demais.  Varia de 1 até 4, onde 1 é o mais prioritário e o 4 é o menos.

### Risk

Um número entre 1 e 3 (Low, Medium e High) que representa o risco geral da implementação dessa funcionalidade. Aqui são considerados integrações, complexidade das regras, quantidade de serviços envolvidos e etc.

### Effort

Vamos usar a técnica de "Tamanho de camisa" para mensurar o esforço nessa fase do projeto. Os valores aceitáveis são:
- PP 
- P
- M
- G
- GG

## Story

As "User Story" descrevem comportamentos do usuário à partir da feature a qual as US estão ligadas. As "Technical Story" descrevem alterações na base de código que, não necessariamente, serão atendidas por um fluxo de usuário.

### Business Description

Descreve a User Story à partir do ponto de vista do usuário/time de negócio. Na Technical Story, justifica a necessidade que engenharia vê para que o negócio seja habilitado. Ambos os casos devem descrever escopo e regras de negócio que serão afetadas.

### Technical Description

Descreve as alterações que deverão ser efetuadas na base de código, serviços a serem criados, logs que devem ser emitidos, componentes a serem criados/utilizados e qualquer outra limitação técnica. Documentação como diagramas, exemplos de código e afins devem vir nesse campo ou referenciados como arquivos anexos.
Quando necessário, descreve as métricas de sucesso/estabilidade da funcionalidade.

### Acceptance Criteria

Critérios de aceite para que a story seja considerada como pronta para entrega. Pode ser uma lista de itens (preferível) ou cenários de uso obrigatórios.
Se necessário, deve incluir também critérios de aceite de engenharia.

### Test Scenarios

Lista os cenários de teste que garantem que todas regras de negócio estão cobertas.

### Story Point

Um número da sequência fibonacci que representa a complexidade de implementação. Acima de oito, o valor é infinito.

### Risk

Um número entre 1 e 3 (Low, Medium e High) que representa o risco geral da implementação dessa funcionalidade. Aqui são considerados integrações, complexidade das regras, quantidade de serviços envolvidos e etc.

### Priority

Qual a prioridade desse WI em relação aos demais.  Varia de 1 até 4, onde 1 é o mais prioritário e o 4 é o menos.
