# Capítulo 1 - Definindo o produto

<!-- TOC -->

- [Capítulo 1 - Definindo o produto](#capítulo-1---definindo-o-produto)
  - [O que é um produto?](#o-que-é-um-produto)
  - [Por que Lean Inception?](#por-que-lean-inception)
  - [O que é Lean Inception?](#o-que-é-lean-inception)
    - [Quem participa?](#quem-participa)
    - [A agenda da Lean Inception](#a-agenda-da-lean-inception)
      - [Segunda-feira](#segunda-feira)
      - [Terça-feira](#terça-feira)
      - [Quarta-feira](#quarta-feira)
      - [Quinta-feira](#quinta-feira)
      - [Sexta-feira](#sexta-feira)
  - [Qual é o problema?](#qual-é-o-problema)
    - [Uma feature ou um produto?](#uma-feature-ou-um-produto)
    - [Antes de começarmos](#antes-de-começarmos)
  - [Manhã do primeiro dia: Visão do produto](#manhã-do-primeiro-dia-visão-do-produto)
    - [Público alvo](#público-alvo)
    - [Cujo problema](#cujo-problema)
    - [Nome do produto](#nome-do-produto)
    - [É um](#é-um)
    - [que](#que)
    - [Diferente do produto tal, nosso produto é](#diferente-do-produto-tal-nosso-produto-é)
    - [Visão do produto final](#visão-do-produto-final)
  - [Tarde do primeiro dia: É-não é. Faz-não faz](#tarde-do-primeiro-dia-é-não-é-faz-não-faz)
    - [Como aplicamos a dinâmica?](#como-aplicamos-a-dinâmica)
    - [Aplicando a dinâmica](#aplicando-a-dinâmica)
  - [Manhã do segundo dia: Personas](#manhã-do-segundo-dia-personas)

<!-- /TOC -->

## O que é um produto?

Nos anos 90, algumas emissoras locais exibiam um tipo curioso de programação aos sábados pela manhã: programas de propaganda. O conteúdo se resumia a alguém, com retórica irretocável, mostrando como determinado produto poderia tornar sua vida melhor, mais simples e mais feliz. Eu gostava desses programas porque falavam do meu maior sonho de infância: computadores. Falavam muito sobre computadores e seus kits multimídia. Foi assim que minha mãe me comprou o primeiro computador, e o resto é história.

O que esse pessoal já sabia nos anos 90 é que só podemos chamar de produto algo que veio para resolver um problema real, gerando benefício para as partes envolvidas (quem utiliza e quem produz). Ainda que esse possa parecer um detalhe óbvio, na área de software Pode parecer óbvio, mas há quem invista tempo e dinheiro em empreitadas sem saber qual é o benefício real que aquilo vai gerar, ou sem saber se o benefício compensa o esforço despendido. Assim aumenta a chance de fracasso: se o usuário não perceber valor, ele não utiliza; se o produtor não consegue equilibrar os custos, a operação tende a falir (mesmo com apoio de venture capital).

É preciso, portanto, ter o mínimo de clareza sobre o problema que vamos resolver e sobre as pessoas que vamos ajudar. Note que o foco não está no produto a ser criado, mas sim nas pessoas e no problema-alvo. O produto deve emergir a partir do encontro dessas duas partes e resultar em uma solução. O produto, portanto, é a materialização adequada de uma solução adequada. Por isso, ao iniciar o processo de definição de produto, você pode entrar com a ideia de fazer um aplicativo e concluir com uma planilha, cheia de gráficos gerados automaticamente; começar com uma ideia envolvendo inteligência artificial e terminar com uma fórmula de regressão simples.

Algo subentendido no que foi dito é que a solução só é solução de verdade quando está nas mãos do usuário. Caso contrário, é apenas um projeto - e projetos não entregam benefício. Por isso a segunda preocupação de quem produz é que o produto chegue às mãos do usuário, no menor tempo viável (não no menor tempo possível). Essa estratégia acelera a entrega de valor e permite validar a solução em fases iniciais, aumentando as chances de sucesso a cada iteração. Quem deve indicar o caminho de avanço de qualquer produto é a necessidade do usuário.

## Por que Lean Inception?

Como você já deve ter percebido, não dá pra gente criar um produto baseado apenas na inspiração e no *insight*. Inspiração e *insight* geram apenas bons *story tellings*. O que cria bons produtos é metodologia. E para isso existem várias.

De modo geral, as metodologias recentes envolvem reunir o grupo mais diverso possível de pessoas e colocá-las para pensar o problema, idealizar uma solução e rascunhar o produto. E por diversidade, você até pode pensar, em primeiro lugar, de cargos e funções. No entanto, se esse produto conseguir reproduzir a diversidade de grupos sociais, o alcance dele tende a ser maior.

Dentre as metodologias possíveis nesta etapa, considero a [Lean Inception](https://caroli.org/livro/lean-inception-como-alinhar-pessoas-e-construir-o-produto-certo/) a que mais se alinha à definição de produto apresentada. Seu foco é definir o problema e encontrar uma solução, alinhando as pessoas envolvidas. A documentação gerada permite que qualquer pessoa, ao olhar o quadro de anotações, recrie e compreenda o fluxo de pensamentos que levaram o produto até ali. Falo por experiência: dos produtos que ajudei a construir, os desenvolvidos a partir de Lean Inception foram os mais bem-sucedidos.  E a documentação nos ajudou a manter o produto nos trilhos.

Antes de prosseguir, uma ressalva: a forma como apresento esse processo não está 100% em conformidade com o material do Paulo Caroli, e essa nunca foi a minha pretensão. Tão pouco pretendo criar uma nova síntese do trabalho dele. Minha intenção é oferecer a perspectiva de quem é responsável pela engenharia do produto. Imagino que designers, stakeholders, POs, PMs, usuários e outros grupos tenham percepções e prioridades próprias. Aqui apresento as nossas.

## O que é Lean Inception?

Como já dito, Lean Inception é uma metodologia de definição de produto com foco no MVP (definindo o quê deve ser feito), deixando para outro momento a construção de histórias, estimativas, arquitetura de software e afins (o "como" será feito).

O processo leva cinco dias. Pode parecer muito, mas não é. E é fundamental que esses dias ocorram sem interrupção. Nada de "só à tarde" ou "todas as segundas-feiras". Esse espaçamento prolonga o trabalho, tira as pessoas do clima, altera o contexto e pode minar o sucesso do processo.

### Quem participa?

Reserve espaço na agenda. Idealmente todo o time participaria, do desenvolvedor ao presidente, mas isso raramente é possível. Recomendo, no mínimo, a presença durante os cinco dias de:

- Especialistas de domínio;
- Time de Produto (PM, PO ou cargos equivalentes);
- Agilistas (coordenadores, staff engineers e afins);
- Time de desenvolvimento (arquitetura, tech lead, desenvolvimento).

Os stakeholders podem participar no início e no final do processo: no início, para entender a agenda da semana; no final, para estar a par (e de acordo) com o que será construído nos próximos meses. Afinal, é o dinheiro deles que está em jogo.

### A agenda da Lean Inception

Os cinco dias ficam divididos assim:

|      Dia      | Manhã                                    | Tarde                  |
| :-----------: | :--------------------------------------- | :--------------------- |
| Segunda-feira | Introdução, kick-off, visão do produto   | É - Não é; Faz - Não faz |
| Terça-feira   | Personas                                 | Features               |
| Quarta-feira  | Revisões técnica, de UX e de negócio     | Jornadas do usuário    |
| Quinta-feira  | Alinhar features e jornadas              | Sequenciar features    |
| Sexta-feira   | Construir o MVP Canvas                   | Apresentação da Inception |

#### Segunda-feira

Apresentamos a metodologia, a agenda, conhecemos as pessoas (uma dinâmica inicial ajuda a criar vínculos) e alinhamos a visão do produto. Com a visão definida, delimita-se o escopo. A dinâmica "É - Não é; Faz - Não faz" evita que o produto vire um balaio de gato.

#### Terça-feira

A definição de personas foca em entender o perfil médio dos usuários, seus anseios e necessidades, para então pensarmos em funcionalidades que os atendam.

> Esse processo gera muita discussão. Muitas boas ideias surgem, mas podem estar fora do escopo ou não se relacionar ao MVP. Anote esses pontos no "parking lot" (estacionamento de ideias) para consulta futura.

#### Quarta-feira

Agora que conhecemos as funcionalidades, a manhã está reservada para entender - considerado o nível de incerteza que temos - o que é mais difícil de ser construído, o grau de incerteza que temos sobre alguns pontos, o quanto acreditamos que aquilo é importante para a experiência desse usuário.

Já na parte da tarde, vamos entender as jornadas do usuário. Desde quando ele sai de casa até as tarefas que precisam ser executadas no dia-a-dia.

#### Quinta-feira

Pela manhã, consolidamos funcionalidades com as jornadas. É crucial haver sinergia entre jornada e funcionalidades; caso contrário, podemos reavaliar a solução. À tarde, definimos a ordem de construção das features.

#### Sexta-feira

Começamos o último dia construindo o Canvas MVP. O momento mais crítico de todo o processo, uma vez que consolida o que será construído efetivamente pelo time.

E por fim, a apresentação do Canvas MVP para os interessados no produto.

> Ao final de cada dia de trabalho, fazemos uma breve revisão do dia para consolidar o caminho até aqui. Não é um momento para debate (se for necessário, que fique para amanhã), mas para revisão.

A pessoa que facilita esse processo deve ter experiência em construção de produtos, mantendo sempre o foco das discussões centradas no MVP. Caso contrário, corremos o risco de não explorar a diversidade presente e de levar o produto para "praias do engano". Se uma proposta não enriquece o MVP, deve ser cortada.

## Qual é o problema?

Para que possamos dar início ao nosso processo de definição de produto, antes precisamos ter claro qual é o problema que estamos buscando endereçar.

O caso a seguir é totalmente fictício. Qualquer similaridades com fatos reais é mera coincidência.

O Setor Financeiro identificou uma possível fraude nas contas da empresa. O montante de pouco mais de um milhão de reais aparenta recair sobre a folha de pagamento. Contudo, ao vistoriar os dados, tudo parece em conformidade com o esperado. No entanto, os extratos bancários da empresa dizem o contrário.

Um processo administrativo foi iniciado. A perícia bancária aponta que depósitos incomuns foram feitos para um funcionário, coincidentemente responsável pelas rotinas de folha de pagamento. Mas até aqui, tudo é circunstancial. O funcionário pode alegar, por exemplo, que foi o software que calculou errado o pagamento, ou até mesmo o banco que processou os dados errôneamente. O ônus da prova recai sobre a empresa, que por sua vez aciona a provedora de software.

Após analisar as rotinas de pagamento, o analista responsável percebeu uma brecha no algoritmo: Quando a rotina da folha de pagamento é acionada mais de **uma** vez, o sistema atualiza todos os dados de pagamento. No entanto, o gateway de comunicação com o banco não gera ordens de pagamento diferentes para o mesmo funcionário em menos de vinte e quatro horas. Ou seja: Basta rodar a folha com um valor alto, da primeira vez; atualizar o salário para um valor real e rodar novamente a folha de pagamento. Agora você tem dinheiro infinito.

Cabe a empresa provar que foi realmente isso que aconteceu, através de evidências sólidas e o menos circunstânciais possível. Afinal de contas, convicção não deve servir de argumento para condenação de ninguém. Como você resolveria isso?

### Uma feature ou um produto?

Neste momento é possível que você esteja pensando que a auditoria está parecendo mais uma feature do que um produto em si. Esse tipo de questionamento faz total sentido, especialmente na fase de definição do produto. Contudo, vamos considerar primeiro a complexidade envolvida para a auditoria?

Você já deve ter se deparado com aqueles costumeiros campos `createdBy` e `updatedBy`, que povoam as tabelas do banco de dados. Esse dados simples é capaz de te dar uma informação simples: Quem criou e quem foi a última pessoa a alterar esse registro. A informação "quais pessoas alteraram" e "quais dados foram alterados" se perdeu. Qualquer pessoa poderia questionar a qualidade dessas informações, uma vez que não é possível provar quais dados efetivamente foram modificados por essa última pessoa.

Uma auditoria real precisa armazenar o histórico de modificações, apontando o que foi modificado em cada uma das intervenções e principalmente quem efetuou essas intervenções nos dados. Também é imprescindível que essas evidências sejam invioláveis, o que exclui a possibilidade, por exemplo, que as evidências residam no mesmo banco em que estão os dados auditados.

Requisitos não-funcionais e transversais (também conhecidos como *cross-cutting*) são candidatos fortes ao processo de produtificação. Eles abstraem um conjunto significativo de complexidade e permitem uma fácil reutilização. Um exemplo bastante conhecido de um Requisito não-funcional que virou produto é o KeyCloak. Ao lado do Entra e Okta, são exemplo de produtos que endereçam soluções de autenticação. Auditoria tem bastante potencial para seguir esse mesmo caminho.

### Antes de começarmos

Nenhum processo de Product Design pode ser feito por uma pessoa sozinha. Não faça isso. Você até dizer para si mesmo que está adiantando um pouco das discussões. Apesar da sua boa vontade, não faça isso. A diversidade de ideias é um dos pilares que garantem o sucesso de um produto.

Contudo, não posso me dar a esse luxo. E por mais que eu não goste e nem recomende, farei todo o processo de Lean Inception sozinho. Mas somente porque não estamos criando um produto real. Lembre-se: o interesse aqui é apenas mostrar o processo de construção de software sob uma perspectiva. A ideia aqui não é construir um produto real. Até porque para isso, eu precisaria contratar especialistas na área do direito digital (advogados e peritos) para me apoiar na construção desse software. E não temos tempo (nem dinheiro) pra isso.

Por tanto, se você utilizar em produção o resultado da nossa construção, é por sua conta e risco.

## Manhã do primeiro dia: Visão do produto

Como nós já apresentamos a metodologia e discutimos sobre o produto, vamos direto a Visão do produto.

Essa dinâmica consiste em construir duas frases que definem o produto e nos auxiliarão na tarefa de construí-lo. Essa visão de produto deveria ficar acessível e visível para todos os participantes. E sempre que algo não fizer sentido com a visão, ou alteramos a visão (prefira não fazer isso) ou deixamos a ideia para mais tarde.

Basicamente, você precisa preencher as lacunas:

> Para [público alvo]
> cujo [qual o problema a ser resolvido]
> o [nome do produto] é um [categoria de produto]
> que [Benefício principal/razão para aquisição].
> Diferente do [principal alternativa de mercado],
> nosso produto [principal diferença entre eles].

### Público alvo

Quem é o público alvo do nosso produto? Alguém pode argumentar é o CTO ou qualquer pessoa com poder de decisão em uma empresa. E isso faz sentido, em qualquer empresa, é a gestão que efetivamente compra um produto. Então vamos definir - apenas à título de experimento mental - que é o CTO o nosso público alvo. Dessa forma, todas as demais respostas devem reagir a isso. Vamos ver como ficou:

> Para **CTO**
> cujas **plataformas sob sua responsabilidade não possuem auditoria**
> o **[nome do produto]** é um **micro-serviço SaaS ou self hosted**
> que **armazena de forma fácil e escalável dados de auditoria**.
> Diferente do [principal alternativa de mercado],
> nosso produto [principal diferença entre eles].

Parece funcionar bem. Mas **eu** fico profundamente incomodado com essa resposta. Tudo bem que o produto final precisa ser atrativo para a gestão de uma empresa. Contudo, na maior parte dos casos, os gestores não utilizarão essa ferramenta diariamente. Tão pouco serão eles a escrever possíveis integrações, manter os serviços no ar e outras demandas cotidianas. Esse não é o **papel** deles, de forma geral.

Um MVP para ter sucesso, precisa conquistar o seu usuário direto em primeiro lugar. Ter um pitch que seja atrativo é importante. Ter um produto fácil de ser integrado é decisivo. Mas um MVP pode (será que pode mesmo?) se dar ao luxo de ser incompleto. Ou você vai gastar algumas sprints escrevendo imagens docker, disponibilizando elas em um *Container Registry* - que você vai ter que pagar - e torcer para que o produto dê certo.

> Desta maneira, quando pensar em "público alvos", foque nas pessoas que vão utilizar o sistema em primeiro lugar.

Quem usa o nosso sistema, então? Ora, o perito! E aqui nós temos dois peritos: O perito designado pelo juiz e um perito designado pelo time de compliance (que pode ser apenas uma pessoa sem formação específica). E tem também aqueles usuários ocasionais, como um Product Manager, que vai checar se o que Fulaninho disse é verdade sobre não ter alterado a informação "X". Nesse momento, vamos focar apenas nos peritos. Já é trabalho suficiente. Vamos ver como o texto fica?

> Para **o perito designado pela justiça/perito compliance interno**
> cujas **investigações são difíceis de colher evidências válidas**
> o **[nome do produto]** é um **micro-serviço SaaS ou self hosted**
> que **armazena dados de auditoria de maneira incorruptível e de fácil acesso**.
> Diferente do [principal alternativa de mercado],
> nosso produto [principal diferença entre eles].

Agora sim, parece que estamos falando de um produto. Nós apenas listamos dois usuários que vão efetivamente usar o produto. Consegue identificar o que mudou dessa visão para a anterior?

### Cujo problema

Pode parecer óbvio, mas não custa reforçar: O problema a ser resolvido deve estar sempre sob a ótica de quem usa o sistema. Que sentido faria elicitar um problema que não dialoga com o público alvo? O time estaria afirmando, literalmente, que irá resolver um problema que não é dos seus usuários. Ou que seus usuários podem até ter um problema mas, o software, não irá resolver. Acho que você entendeu. Eu só peço para que não escreva uma monografia nessa linha. Se o problema não estiver claro, discuta mais com seus pares e quando estiver claro pra todo mundo, descreva.

### Nome do produto

Eu chamaria mais de "apelido", ou ainda melhor: codinome. Talvez você esteja escrevendo um módulo dentro de uma plataforma. Então você chamaria "módulo que faz tal coisa". Mas sinceramente? Eu iria de um codinome simples. A ideia aqui não é sair com logotipo, logomarca e *jingles* para vender. É apenas ter uma forma de chamar o produto durante a inception. Se o nome for bom e colar, pode até ser que permaneça. Mas não tem essa obrigação.

### É um

Que produto você está vendendo? É um aplicativo? É uma ferramenta agrícola? É um animal da raça Malinois? Não se preocupe com aquilo que, comercialmente, o produto não é. Teremos uma etapa só pra isso.

### que

Aqui é a parte importante do pitch: você precisa dizer em poucas palavras qual é o super-poder da sua solução. Pense que o seu produto é o Superman:

> Para **um sociedade assustada**
> cujas **vidas são assoladas por crimes e problemas galáticos**
> o **Superman** é um **super-herói alienígena**
> que **que é indestrutível, super-forte e incorruptível**.
> Diferente do [principal alternativa de mercado],
> nosso produto [principal diferença entre eles].

Pegou a ideia?

### Diferente do produto tal, nosso produto é

Não importa o que você esteja fazendo, alguém já fez algo parecido. Mesmo quando há disrupção. Imagine o Uber:

> Diferente do **Disk-Taxi**,
> nosso produto **otimiza tempo e custo, escolhendo sempre os motoristas mais próximos via aplicativo**.

Ou seja: Antes do Uber já existia o Disk-Taxi. Antes do iFood, existiam as páginas amarelas. Antes do... Acho que você já pegou a ideia.

### Visão do produto final

> Para **o perito designado pela justiça/perito compliance interno**
> cujas **investigações são difíceis de colher evidências válidas**
> o **RavenLedger** é um **micro-serviço SaaS ou self hosted**
> que **armazena evidências de manipulação de dados, identificáveis e de fácil acesso**.
> Diferente das **auditorias simplificadas**,
> nosso produto **garante a incorruptibilidade das evidências**.

## Tarde do primeiro dia: É-não é. Faz-não faz

Enquanto a visão do produto nos oferece uma visão mais ampla daquilo que desejamos entregar, a dinâmica da tarde refina um pouco mais a definição. O interesse aqui é definir um foco e estabelecer fronteiras para que fique muito bem delimitado o escopo do produto. Isso inclui questões um pouco mais distantes da engenharia - como posicionamento de mercado, estratégia de marketing e afins - e outras estritamente correlatas - como apresentação, domínio do produto e etc.

### Como aplicamos a dinâmica?

Basicamente, você precisa desenhar um plano cartesiano, com os dizeres **É**, **Não é**, **Faz**, **Não Faz** nos quadrantes 1, 2, 3 e 4. As pessoas serão convidadas a preencher um quadrante por vez, adicionando *post-it* com a definição escrita em cada um. 

Eu indicaria fazer um quadrante por vez para desestimular o preenchimento por simples antônimos. Por exemplo: **_É_** *um aplicativo web*, **_Não é_** *um app de celular*. O antônimo talvez faça sentido quando uma ideia precisa ser reforçada. Mas muitos, é só *post-it* sendo mal usado.

Uma outra sugestão seria a de orientar as pessoas a não utilizarem apenas adjetivos vagos. Por exemplo: *é fácil de usar*. São raros os sistemas onde a usabilidade não seja um requisito básico. Não ser fácil de usar é uma exceção à regra. Talvez uma boa definição seria: *usuários de AutoCad devem achar fácil de usar*, se o nosso **público alvo** é composto por usuários dessa ferramenta.

### Aplicando a dinâmica

- **É**: "Sistema de auditoria", "Repositório de evidências", "Válido como prova jurídica", "Inviolável", "Somente leitura", "SaaS", "on-premisse", "Sistema Web", "Auditoria de integridade de dados";
- **Não é**: "Sistema de observabilidade e tracing", "Log persistido", "Power BI", "Data Warehouse", "Previne Fraude", "Auditoria Operacional", "Auditoria Financeira", "Auditoria de processos", "Monitoramento".

É possível que ao final de cada quadrante, seja necessário remover alguns *post-it*. Faça isso consultando o time antes. Você está lidando com pessoas em um momento de profunda insegurança: Elas estão levando suas ideias ao escrutínio de outras pessoas - e que em alguns casos acabaram de se conhecer. Tenha cuidado para não perder alguém. Entenda que se algo está destoante, existem duas possibilidades: A pessoa teve uma ideia realmente fora da caixa e vale a pena ela explicar sua linha de raciocínio. Ou ela tem uma visão diferente do produto - o que é esperado para este primeiro dia e essa dinâmica visa justamente alinhar essas perspectivas.

No nosso caso, pode ter acontecido alguma discussão sobre que tipo de auditoria teríamos. Alguém que já trabalhou com sistema financeiro, pode acreditar que essa seria uma ótima base de dados para plugar um motor anti-fraude, acelerando a identificação e até mesmo prevenindo fraudes no sistema. Após alguma discussão, o time pode ter entendido que essa foi uma ótima ideia, mas que ela ficaria estacionada para um momento futuro ou para um produto futuro. Alguém pode ter sugerido, também, que esta seria uma ótima oportunidade para mensurar se os usuários estão adicionando as informações no sistema conforme o manual de boas práticas ou acertando tudo ao final do mês. Por isso alguém disse que o produto não é uma Auditoria de processos.

É importante discutir as afirmações (umas com discussões maiores, outras menores) para que elas tenham lastro, tenham o suporte do time e que fique clara a motivação por trás de cada uma. Por isso é melhor focar em qualidade do que em quantidade.

- **Faz**: "Armazena log de manipulações de entidade", "Identifica versões de um registro", "Identifica autor das manipulações", "Rastreia efeitos colaterais", "Configura entidades de domínio", "Armazena log de eventos", "Detecta violação interna de maneira passiva", "Encadeia registros para validação de adulteração";
- **Não faz**: "Replay de eventos", "Correção de dados", "Substituição de auditor humano", "Determina fraude", "Log de aplicações", "acesso de pessoas externas", "relatórios não ligados ao processo de compliance".

Na dinâmica *faz-não faz*, grande parte do *não faz* vai surgir enquanto consideramos o *faz*. A diversidade nos encaminha a isso. É preciso saber que existem duas categorias de *não faz*: Aqueles que não tem relação com a proposta do produto e aqueles que, um dia, serão implementados. Ambos ficam no *não faz*? Você que sabe. A documentação é sua! Apenas deixa fácil de lembrar quem é o "não faz agora" e o "jamais faremos".

O *faz* por sua vez, poderia contar com alguns detalhes técnicos - se já previamente conhecidos. Nosso produto de auditoria, com certeza absoluta, possui muitos requisitos técnicos que **já devem ser de conhecimento geral**. Adequações com processos ISO e SOX, por exemplo, e quais são esses processos deveriam fazer parte dessa discussão no mundo real (mas não fará parte desse nosso laboratório). Para garantir a inviolabilidade (e esse, por si só já é um *faz*), será necessário algum algoritmo de *hash*. Então especificar isso no *faz* não é um problema. As estratégias de venda - SaaS e on-premise - **precisam** estar declaradas já de início. O impacto dessa decisão *a posteriori* poderia exigir uma reescrita completa do software. **Então, sim, detalhes técnicos podem fazer sentido nesse momento**. 

Isso pode não ser verdade em produtos mais genéricos - como uma plataforma de eCommerce - onde a estratégia de desenvolvimento ainda poderia ser discutida conforme os requisitos ficam mais claros. Não se apresse a dizer se algo será escrito como micro serviços, ou *serverless*. Ao final você pode sair com um monolito "e tá tudo bem".

Agora é hora de fazer a consolidação do dia e partir para descansar.

## Manhã do segundo dia: Personas

Essa é, para mim, uma das partes mais importantes desse processo. O time tira os olhos do que ele quer como time e passa a olhar para os usuários do sistema. E você vai perceber que esse olhar busca ser o mais holístico possível. Nós queremos entender as pressões sob as quais o usuário está submetido, qual é a sua rotina, o que o motiva a fazer algo certo (ou pode motivar a fazer algo fora do especificação técnica) e assim por diante. 

Isso vai servir de base para definir, por exemplo, quais os valores padrões de filtros em pesquisas, quais visualizações ficam mais a esquerda, iconografia, enfim, grande parte da UX será construída tempo por base a persona construída. Podem parecer detalhes simples, mas farão total diferença na taxa de assertividade de processos, conversão de vendas e assim por diante.

Além da UX, a construção das personas auxiliará o time a definir quais são os problemas mais urgentes para o usuário. Você vai perceber que todas as pessoas do time tem as suas expectativas sobre o produto. Desejam essa ou aquela funcionalidade. Nós, da área de engenharia, queremos por exemplo todas as telas de dados mestres já no primeiro dia. Ou ainda que o usuário tenha uma opção de automatizar re-processamento de mensagens enfim. Agora o foco é no usuário. Vamos deixar as nossas expectativas para outro momento.
