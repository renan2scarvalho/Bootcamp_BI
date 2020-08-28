# ETL

Nesse Desafio o objetivo foi realizar o processo completo do ETL no Pentaho Data Integration (PDI), a partir de sete tabelas *excel* e arquivos *csv*:

![tabelas](https://user-images.githubusercontent.com/63553829/91566283-cd894100-e919-11ea-910d-aa6f59225b61.png)

Essas tabelas seguem o esquema *snow flake*, como apresentado adiante:

![sf_mdl](https://user-images.githubusercontent.com/63553829/91566200-afbbdc00-e919-11ea-9722-d46bfaa58656.png)

Assim, a partir das tabelas de origem, o objetivo é modelar um Data Warehouse (DW) como o seguinte esquema *star*:

![star_mdl](https://user-images.githubusercontent.com/63553829/91566611-51dbc400-e91a-11ea-875e-2d944ea62019.png)

## Agora, ao trabalho!

## Extração

Agora que já sabemos quais são as tabelas e o esquema proposto, iremos realizar o primeiro passo, que é a criação de uma **staging area**, i.e. criação de uma área temporária e  extração das tabelas para a mesma, a fim de padronizá-las e seguir para o passo de transformação. Portanto, no MySQL Workbench, criamos a base de dados com o código a seguir, no qual também já criaremos a base de dados para o Data Warehouse (DW):

```javascript
CREATE DATABASE stg_desafio;
CREATE DATABASE dw_desafio;
```

Agora, com a *staging area* criada, iremos realizar a **extração** das tabelas, iniciando com as extensões *csv* no Pentaho (uma boa prática é realizar a extração de uma tabela individualmente, mas nesse caso iremos realizar a extração agrupada por extensões). Assim, somente devemos digitar *csv* na aba *Design* do Pentaho, e incluir *CSV input* para cada tabela. Aqui, caso queira, é possível adicionar notas clicando com o botão direto na tela um espaço em branco.

![ext_csv](https://user-images.githubusercontent.com/63553829/91573442-48a02680-e91d-11ea-899c-f9646c0b14e5.png)

O próximo passo, realizado para todas as tabelas, é adicionar o diretório do arquivo, delimitador, e buscar os campos (*Get Fields*). É importante notar que nesse processo não vamos abordar padronizações como formato, tamanho e precisão dos atributos. Mas aqui é importante notar as casas decimais e agrupamentos, que devem ser iguals ao seu OS, e aplicar corte (*Trim*) em ambos os lados (para que não existam espaços antes e depois dos valores), como na figura a seguir:

![ext_csv2](https://user-images.githubusercontent.com/63553829/91577004-818ccb00-e91e-11ea-8209-4786348ac68e.png)

Aqui, caso queira, é possível dar um *Preview* na tabela, localizado ao loado de *Get Fields*, inserir o número de linhas para visualização:

![ext_prev](https://user-images.githubusercontent.com/63553829/91577530-3626ec80-e91f-11ea-8fc5-9e8e746c9c11.png)

Como é possível notar, temos algumas linhas com valores faltantes, as quais deveremos tratar durante a transformação.
Após a realização da extração dos arquivos *csv*, agora iremos rapidamente realizar a extração dos arquivos *excel*, que envolve alguns passos a mais. Novamente, apenas adicionando *excel* na aba *Design* é possível adicionar as caixas de input. Novamente buscamos o diretório, adicionamos, até que apareça no campo *Selected Files*:

![ext_excel](https://user-images.githubusercontent.com/63553829/91578189-383d7b00-e920-11ea-9f93-c13c0c8959d6.png)

No passo anterior, é importante comentar sobre o *Spread sheet type (engine)*, que seleciona o arquivo *excel*. Aqui, recomenda-se que se utilize o *Excel 2007 XLSX (Apache POI)*, capaz de selecionar desde arquivos *excel* antigos (97-2003) a mais novos.
O próximo passo com arquivos excel é selectionar a planilha *Sheet* através de *Get sheetname(s)...* e selecionar as planilhas desejadas:

![ext_excel2](https://user-images.githubusercontent.com/63553829/91578525-aeda7880-e920-11ea-8d24-964c048ea184.png)

E por fim, selectionar os campos na aba *Fields*, através do *Get fields from header row...*, e realizar as padronizações necessárias, como nos arquivos *csv*, e, se necessário, visualizar a selecção através do *Preview rows*:

![ext_excel3](https://user-images.githubusercontent.com/63553829/91578970-4cce4300-e921-11ea-8d20-c8c116bc3c9a.png)

Após adicionar todos os inputs, devemos enviá-los à **staging area**, através de um *Table output*, também facilmente adicionado na aba *Design*. Esse processo é comum a todos os inputs, e, portanto, será demonstrado apenas uma vez. O primeiro passo que devemos realizar é criar uma nova conexão com nossa base de dados "stg_desafio" no MySQL criada no início. Daremos um nome à conexão para ser partilhada pelos demais inputs, e adicionamos os parâmetros do MySQL (*Host Name, Database Name, Port Number, Username, Password*), e podemos realizar um teste para avaliar se a conexão foi estabelecida:

![sql](https://user-images.githubusercontent.com/63553829/91585346-55774700-e92a-11ea-9de8-98051e5d9346.png)

Agora que conseguimos conectar com nossa *staging area* no MySQL, devemos dar um nome à tabela a ser inserida, e criá-la, onde existem duas possibilidades. A primeira consiste na criação da tabela dentro do próprio Workbench, ou criá-la através do Pentaho, como faremos. Nesse caso, ao clicar no botão "SQL", uma janela com a query necessária para a criação da tabela aparece, e temos que executá-la. Ao final, caso a query tenha sido executada com sucesso, uma nova janela aparecerá com o resultado:

![sql2](https://user-images.githubusercontent.com/63553829/91585961-231a1980-e92b-11ea-92fa-88fbf1d654a3.png)

Enfim, terminamos a programação do processo de **extração** das tabelas, e envio para a *staging area*. O próximo passo é darmos um preview para que possamos ver se existe algum erro no processo. Clicamos com o botão direito em algum dos processos e colocamos *Preview*. Caso nenhum erro tenha sido encontrado, realizaremos o processo através do botão *Run*. Caso o processo tenha sido executado, abaixo temos o aviso de transformação finalizada, como detalhado na figura:

![ext_finish](https://user-images.githubusercontent.com/63553829/91587688-7ee5a200-e92d-11ea-8495-71854715d54e.png)

Caso queira conferir, é possível executar uma query no MySQL Workbench:

```javascript
SELECT * FROM [TABLE]
```


## Transformação e Carga

As transformações serão realizadas em cada dimensão separadamente. O primeiro passo é inserir as tabelas da *staging area* atraves de um *Table input* na aba *Design*. Criaremos a conexão com o MySQL como feito anteriormente, e aplicamos uma query para seleção os atributos (aqui selectionaremos todos atributos). Como esse passo é comum a todas as dimensões, ele será apresentado apenas um vez:

![inp](https://user-images.githubusercontent.com/63553829/91590431-c9691d80-e931-11ea-8823-7333f457b218.png)

### 1. Dimensão Cliente

A figura a seguir apresenta a visão completa da Dimensão Cliente, e alguns blocos discutidos em detalhe 

![dim_cl1](https://user-images.githubusercontent.com/63553829/91591231-fec23b00-e932-11ea-9075-fdd04f6bd7dd.png)

Iniciamos com um *Sort rows"*, o qual realiza a ordenação da tabela em função de um atributo, normalmente a chave primária (PK). Entretanto, no caso das tabelas "Regiao" e "Territorio", como iremos aplicar um *Merge* em ambas, elas são ordenadas em função das chaves em comum. Após, aplicamos um *Merge join* para fundir duas tabelas em função de uma PK. Aqui, realizamos um *LEFT OUTER Join*, isso porque a tabela "Regiao" possui mais IDs que a tabela "Territorio":

![dim_cl2](https://user-images.githubusercontent.com/63553829/91592054-4eedcd00-e934-11ea-84a1-a039fa1f40c5.png)

A seguir, aplicamos um *Select values* para removemos o atributo *iDTerritorio*, uma vez que após o *Merge* temos dois IDs para o atributo Territorio:

![dim_cl3](https://user-images.githubusercontent.com/63553829/91603590-a182b580-e943-11ea-9bbe-8be53010326d.png)

Após realizar o primeiro *Merge*, aplicamos outro *Sort rows* agora no atributo *iDRegiao*, como no caso do input *Cliente*, passa que possamos realizar o segundo merge *Merge* entre as tabelas "Cliente" e "Regiao". Após, ordenamos pelo *iDClientes*, e removemos o atributo *iDRegiao* pois está duplicado:

![dim_cl4](https://user-images.githubusercontent.com/63553829/91603748-ed355f00-e943-11ea-93a0-8e4f2241f1ae.png)

Por fim, criaremos a Tabela Dimensão Cliente com o bloco *Dimension Lookup/Update*. Aqui estabelecemos uma nova conxão para a base de dados do **Data Warehouse** *dw_desafio* como realizado outras vezes, damos um nome para a tabela.
Na aba *Keys* preenchemos o campo dimensão e selecionamos o campo *iDClientes* no fluxo, e abaixo, preenchemos a [*Surrogate Key*](https://en.wikipedia.org/wiki/Surrogate_key) que irá para a Tabela Fato "Vendas", e selecionamos como auto-incremental, e na aba *Fields*, selecionamos os atributos, os quais podem ser selecionados com o botão *Get Fields*.

![dim_cl5](https://user-images.githubusercontent.com/63553829/91593798-2f0bd880-e937-11ea-88ad-f2139da51030.png)

Agora, criamos uma versão para o campo, utilizando uma data alternativa para início. Por fim, temos que criar a tabela que irá ser preenchida. Novamente, podemos criá-la no MySQL Workbench, ou podemos criá-la direto no Pentaho, procedimento o qual feramos. Ao executarmos o passo, uma janela aparecerá, nos dizendo o resultado:

![dim_cl6](https://user-images.githubusercontent.com/63553829/91594179-d557de00-e937-11ea-8332-fab18ff490f1.png)

Caso todos os passos estejam corretos, ao executarmos a **transformação**, assim como o processo de extração, o Pentaho apresenta uma mensagem de *transformação concluída!!* É importante comentar que ao realizar essa transformação, estamos realizando também a **carga** da dimensão na base de dados do DW, ou seja, aqui realizamos os dois últimos passos do processo de ETL.

![dim_cl7](https://user-images.githubusercontent.com/63553829/91594397-3da6bf80-e938-11ea-9af6-bf83f7b4c354.png)

Pronto! Ou quase! Concluímos nossa primeira transformação! Agora iremos para as próximas dimensões.


### 2. Dimensão Produto

Agora criaremos a Dimensão Produto. A figura a seguir apresenta o diagrama commpleto.

![dim_prod](https://user-images.githubusercontent.com/63553829/91595087-582d6880-e939-11ea-81fd-7f794b7ed53e.png)

Como anteriormente, ordenamos as tabelas de entrada "Produto" e "Categoria" em função do atributo *iDCategoria* (lembre-se, valor em comum), e aplicamos um *LEFT OUTER Merge Join*. A tabela "Categoria" possui valores nulos, que continuaram após o *Merge*:

![dim_prod2](https://user-images.githubusercontent.com/63553829/91595876-a0995600-e93a-11ea-87d3-7ca78a714c6c.png)

Nesse caso, aplicamos então o bloco *If field value is null*, que trabalha com valores nulos, substituindo-os por "ND" (Não Disponível):

![dim_prod3](https://user-images.githubusercontent.com/63553829/91596106-071e7400-e93b-11ea-9d6b-b418f3f54100.png)

Por fim, removemos o *iDCategoria* (duplicado), e criamos a Dimensão Produto, como feito na Dimensão Cliente. Lembre-se que nesse passo é necessário preencher os campos de chave, chave técnica (SK), campos da tabela, data, e criar a tabela no MySQL. Após feitos estes passos, caso todos estejam corretos, temos a mensagem de êxito da **transformação**, sendo realizada também a **carga** da dimensão no DW.

![dim_prod4](https://user-images.githubusercontent.com/63553829/91601976-eb1dd100-e940-11ea-967b-bfb6f7640969.png)


## 3. Dimensão Funcionário

Neste terceiro passo, criaremos a Dimensão Funcionário, como apresentado no diagrama a seguir. Neste caso, apenas ordenamos o atributo *matFuncionario*, e aplicamos a **transformação**, novamente preenchendo os campos necessários no bloco *Dimension Lookup/Update*, e realiazmos a **carga** da dimensão no DW:

![dim_func](https://user-images.githubusercontent.com/63553829/91602657-17861d00-e942-11ea-91c3-d23a300b4226.png)


## 4. Dimensão Data

Todo modelo dimensional deve ter uma Dimensão Data. Aqui, criaremos esta dimensão do zero, que, ao final, terá o seguinte diagrama:

![dim_data](https://user-images.githubusercontent.com/63553829/91607198-e27dc880-e949-11ea-9b93-424e04b18014.png)

Criamos a data inicial com um bloco *Generate rows*, aplicando um limite de 100.000 linhas, dando um nome à variável e um valor inicial, no caso 01/01/2010. Como o valor da data inicial é o mesmo para toda linha, a seguir adicionamos uma sequencia de dias somados a partir da data inicial, com incremento de 1 dia:

![dim_data2](https://user-images.githubusercontent.com/63553829/91605782-62566380-e947-11ea-9fd5-e0cdae4d6ae8.png)

O próximo passo é calcular as diversas datas para a dimensão através de um bloco *Calculator*. Iniciamos preenchendo a primeira linha com a SK da data em função dos blocos anteriores, alterando a *Conversion mask* para dd/MM/yyyy, notação final das datas, e, a seguir, calculamos as demais datas em função da SK criada:

![dim_data3](https://user-images.githubusercontent.com/63553829/91606100-db55bb00-e947-11ea-80dc-627e20875bb9.png)

Criaremos agora strings para as semanas do ano, utilizando o atributo criado no paso anteior e adicionamos "ª Semana":

![dim_data4](https://user-images.githubusercontent.com/63553829/91606475-81092a00-e948-11ea-8331-6b46cb4c5423.png)

A seguir, aplicamos diversos blocos *Value mapper* para adicionarmos colunas em função de atributos existentes, como na figura abaixo:

![dim_data5](https://user-images.githubusercontent.com/63553829/91606877-55d30a80-e949-11ea-8fc0-d9aa4e4d7d18.png)

Por fim, removemos os atributos *DataInicial* e *Dias*, criados no começo do processo, e criamos a Dimensão Data, aplicando os preenchimentos necessários, criando a tabela na base de dados do MySQL, e aplicamos a **transformação** juntamente com a **carga** da dimensão no DW. Esse processo leva um tempo maior que as transformações anteriores, devido aos cálculos realizados na criação da dimensão, em comparação com os processos das demais dimensões que são razoavelmente simples.

![dim_data6](https://user-images.githubusercontent.com/63553829/91607936-10afd800-e94b-11ea-8c3b-5847991f37a0.png)


## Fato Vendas

Após tanto trabalho, estamos finalizando nosso processo de ETL e criação do DW. Nos falta apenas a tabela *Fato Vendas*, a qual criaremos de acordo com o seguinte diagrama:

![fato](https://user-images.githubusercontent.com/63553829/91608180-6dab8e00-e94b-11ea-977e-8d3335f1dfcd.png)

Após inserir a tabela "Vendas" da *staging area*, utilizamos blocos *Database lookpu* para buscar as dimensões e adicioná-las à mesma. Aqui utilizamos o join entre PK das dimensões e a tabela fato como lookup, mas retornaremos as SK das dimensões na fato:

![fato2](https://user-images.githubusercontent.com/63553829/91608845-ac8e1380-e94c-11ea-8823-b94af5ad3d6c.png)
![fato3](https://user-images.githubusercontent.com/63553829/91608946-d810fe00-e94c-11ea-811a-5cf29953aa4b.png)





