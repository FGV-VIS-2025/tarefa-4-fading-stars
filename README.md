# Fading Stars

*Desenvolvido por Pedro Santos Tokar e Vitor Matheus do Nascimento Moreira.*

-------------------------------------------------------------------------------

## Sobre o projeto

Este repositório contém a nossa submissão da tarefa 4 da disciplina de Visualização
de Dados, do 5º período da graduação de Ciência de Dados e Inteligência Artificial
da FGV-EMAp. O trabalho consiste em implementar, com a biblioteca
[D3.js](https://d3js.org/), uma visualização que permite a exploração interativa
ou storytelling de um conjunto de dados da escolha do grupo.

No nosso trabalho, optamos por usar conjuntos de dados relacionados
às estrelas e às constelações. Informações sobre a fonte dos dados podem ser
encontradas no final da página da visualização. Para facilitar o desenvolvimento
e permitir uma melhor modularização de nossa base de código, optamos por usar o
framework [Svelte](https://svelte.dev/).

Os principais objetivos da visualização são:

1. Mostrar, de forma interativa e usando um diagrama já conhecido, como é o
ciclo de vida das estrelas e como suas classificações funcionam;
2. Mostrar, também de forma interativa, como o céu observado da terra sofre
interferência de diversos fatores, como latitude do observador e poluição
lumionsa do local em que está sendo feita a observação.

## Como acessar a visualização.

A página que abriga a visualização e suas ferramentas está hospedada no
[Github Pages](https://fgv-vis-2025.github.io/tarefa-4-fading-stars/). Para rodar
o servidor localmente e fazer mudanças, é necessário primeiro clonar esse repositório:

```bash
$ git clone git@github.com:FGV-VIS-2025/tarefa-4-fading-stars.git
cd tarefa-4-fading-stars
```

Após isso, instalar as dependências usando o `npm`:

```bash
$ npm install
```

E por último, executar o servidor svelte:

```bash
$ npm run build && npm run dev -- --open
```
Após executar, o navegador será automaticamente aberto na visualização. Também será
possível acessar usando o link localhost exposto no terminal.

## Decisões de design

Nosso projeto conta com duas visualizações principais para o dataset de estrelas:
um **diagrama HR** e uma **esfera celeste interativa**.

### Sobre o diagrama

O **diagrama HR** foi escolhido por ser uma visualização sobre as estrelas já
consolidada no estudo da Astronomia, que narra de forma visual como é o ciclo de
vida das estrelas e elucida as características dos seus diferentes tipos. Ele consiste
em um scatterplot das estrelas, atrelado a 4 variáveis, sendo elas dois pares de
variáveis relacionadas entre si. Nós estendemos o diagrama por meio de duas
ferramentas de interatividade:

- Um retículo, que acompanha o ponteiro do mouse na visualização e mostra quais
valores dos eixos correspondem a um certo ponto. Isso facilita inspecionar quais
são as caracteristicas de algum ponto na visualização, já que uma linha auxiliar
é traçada e ajuda na leitura dos eixos. Como há vários pontos bem pequenos nesse
diagrama, consideramos essa interação mais efetiva que um tooltip convencional
após testar o convencional.

- Filtros atrelados ao texto lateral, que auxiliam no story telling filtrando
quais estrelas são mostradas no diagrama HR. Dessa forma, o texto explicativo
consegue ter controle sobre o diagrama, tornando o storytelling mais fluído e
interessante, já que os filtros se relacionam com a explicação fornecida no texto.

### Sobre a esfera celeste

A **esfera celeste interativa** foi escolhida por ser uma forma de visualizar
a informação espacial das estrelas: por meio de uma projeção, é possível simular
a visão que um observador tem do céu à noite, sob diferentes posicionamentos do
simulador e do céu. As estrelas e constelações são plotadas usando sua localização
real como base. Com o globo, o leitor da visualização pode explorar o céu
sob diferentes perspectivas e simulando diferentes posicionamentos. As seguintes
ferramentas de interatividade auxiliam a visualização:

- Um mecanismo de arrasto, que permite que a esfera celeste seja posicionada
com diferentes valores de ascensão e declinação. Como estamos lidando com uma
esfera, consideramos ser mais efetivo permitir a translação dela do que projetá-la
para um plano 2d, e também porque uma projeção perderia o propósito de simular o
céu visível. Ao interagir, a posição projetada das estrelas e constelações é
calculada de acordo com a nova posição.

- Animações atreladas ao texto lateral, que explicam e demonstram efetivamente
como são os movimentos possíveis a serem feitos na esfera. Dessa forma, enquanto
introduzimos as opções de movimentação, elas são mostradas na prática. Essas mesmas
animações de movimento são reutilizadas nas demais ferramentas do globo, já que
elas podem focar em diferentes ângulos dele.

- Um tooltip, que exibe informações detalhadas sobre alguma estrela que recebe
atenção do mouse. Dessa forma, informações mais detalhadas e que não são codificadas
na visualização podem ser inspecionadas por um leitor mais curioso.

- Um filtro de constelações, que permite tanto ocultar algumas constelações
específicas do mapa (ou todas de uma vez), quanto permite focar uma em específico.
Quando o foco é aplicado, a esfera rotaciona para centralizar a constelação,
simulando um observador que está abaixo dela no céu. Dessa forma, constelações
indesejadas podem ser removidas do globo, tornando a visão mais focada nas estrelas.

- Um filtro de magnitude, que permite filtrar quais estrelas estão visiveis na
esfera (bem como constelações atreladas a elas) de acordo com o valor da magnitude
aparente delas. Esse filtro permite com que o usuário simule diferentes níveis
de poluição luminosa do céu, observando o impacto desses níveis no céu observado
durante a noite. Em conjunto com a ferramenta explicada abaixo, é possível com que
o nível seja definido automaticamente.

- Uma ferramenta que permite ajustar a posição do globo para mostrar o céu visível
de uma cidade específica. Com o auxilio da API pública do
[Nominatin](https://nominatim.org/release-docs/develop/api/Search/), a visualização
se ajusta para a latitude da cidade pesquisada e escolhida pelo usuário. Além disso,
quando uma cidade é escolhida, se torna visível a opção de ajustar o filtro anterior
de acordo com a poluição luminosa da cidade escolhida.

Essas duas últimas ferramentas auxiliam, de certa forma, no storytelling da
visualização, já que dão enfase na questão do céu observável ser diferente em
diferentes localizações e sob diferentes condições de poluição luminosa.

Todas as ferramentas são sinalizadas e explicadas ou por meio do texto lateral
(para o caso de ferramentas que são usadas diretamente no globo) ou por meio de
caixas de ajuda (para o caso de ferramentas presentes na lateral), assim evitando
que uma ferramenta em específico seja passada batida por um observador. Assim,
esperamos deixar claro tudo o que a visualização oferece.

### Histogramas

Além das duas visualizações principais, também foram adicionados dois histogramas,
que mostram a distribuição de features específicas das estrelas, para aquelas
visíveis na esfera exibida. Dessa forma, a visualização do globo se torna também
um filtro, já que sua rotação muda o conjunto das estrelas que serão usadas
para o cálculo dos histogramas.

## Processo de desenvolvimento

TODO.

## Uso de GPT

Durante o trabalho, LLMs foram úteis principalmente para ter ideias de como
implementar certas ferramentas, e em como fazer certos efeitos com CSS e JavaScript.
Como essas linguagens tem uma ampla documentação e uma ampla gama de recursos, o
GPT foi uma boa ferramenta para ajudar a localizar quais desses recursos poderiam
ser usados para implementar alguma coisa específica, e para sumarizar a documentação
delas.
