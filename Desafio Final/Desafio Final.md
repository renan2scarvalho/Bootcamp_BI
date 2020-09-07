# Desafio Final

O Desafio Final do Bootcamp consistiu em aplicar todo o conhecimento adquirido ao longo do curso, exercitando os seguintes conceitos:
- Conceitos e fundamentos de BI;
- Ferramentas de BI;
- Processo ETL;
- Relatórios e Dashboards.

Par a execução, foi utilizada uma base SQL com o seguinte esquema *snowflake*:

![snowflake](https://user-images.githubusercontent.com/63553829/92404035-09d25380-f109-11ea-86cd-3757aacf09c2.png)

E utilizado um arquivo *csv* com a avaliação dos clientes:

![rating](https://user-images.githubusercontent.com/63553829/92404218-6766a000-f109-11ea-9a7b-d1898655a61e.png)

O esquema estreal resultante do modelo dimensional foi o seguinte:

![star](https://user-images.githubusercontent.com/63553829/92404946-f4f6bf80-f10a-11ea-8dd1-e8b76e008853.png)

Os *softwares* utilizados foram o Pentaho Data Integration (PDI) e o SGBD MySQL. Como início, as tabelas da base SQL no MySQL foram já consideradas como a *staging area*, sendo que, portanto, não foi realizado o processo de extração, mas somente transformação e carga. Vamos então aos processos!

## Transformação

### 1. Dimensão Order

A dimensão *order* consiste na junção das tabelas *country*, *state*, *city*, *priority*, *rating* e *torder*. A figura abaixo demonstra todos os passos realizados para chegarmos à tabela final:

![order](https://user-images.githubusercontent.com/63553829/92407704-b31d4780-f111-11ea-8915-c8ffbd25af4c.png)

Primeiramente, selecionamos a conexão com o MySQL *staging area*, onde se encontram as tabelas de entrada. Devemos criar a conexão, inserindo todos os parâmetros demandados, testá-la, e por fim executar a query de seleção dos campos. Esse procedimento é comum a todas as tabelas acima, fora a entrada *rating*, a qual será mencionada abaixo.

![order2](https://user-images.githubusercontent.com/63553829/92408940-5e7bcb80-f115-11ea-9fb5-be9686d6b7af.png)

O procedimento de junção das tabelas é comum a todas. Aplicados o *merge* nos *IDs* em comum entre as tabelas. Mas, para isso, é **sempre** necessário ordenar as linhas pelos *IDs* que serão aplicados no *merge*. No nosso caso, todos os *merge* realizados serão *INNER Join*, ou seja, *IDs* em comum entre as tabelas:

![order3](https://user-images.githubusercontent.com/63553829/92409271-6a1bc200-f116-11ea-910a-04420748b8b7.png)

Executaremos 5 *merges* seguidos, e agora removemos os *IDs* repetidos de cada *merge* antes do passo final, com a entrada *ratings*:

![order4](https://user-images.githubusercontent.com/63553829/92409358-b1a24e00-f116-11ea-81fe-b6609ed9e0f5.png)

Agora trataremos do input *ratings*. Aqui selecionamos o diretório do arquivo e os campos, e retiramos o *enclosure*.

![order5](https://user-images.githubusercontent.com/63553829/92409519-4a38ce00-f117-11ea-93fc-d39f1fed9ff0.png)

Entretanto, quando damos um *preview* dos dados, notamos que as aspas duplas estão duplicadas. 

![order6](https://user-images.githubusercontent.com/63553829/92409574-77857c00-f117-11ea-9ab6-e5fd179d2eeb.png)

Agora executaremos, portanto, alguns passos com as strings. Primeiramente selecionamos os campos, procuramos pelas aspas duplas, e "substituímos" por um vazio. Agora, após a substituição das aspas nos dados, alteramos os nomes dos campos, removendo as aspas duplas:

![order7](https://user-images.githubusercontent.com/63553829/92409805-448fb800-f118-11ea-8956-05046df7a0ba.png)

Após realizados os processos com as strings, realizamos o último *merge*, removemos os *IDs* duplicados, e criamos nossa dimensão. Para tal, criamos uma nova conexão com o MySQL, agora direcionada ao Data Warehouse (DW), nova base de dados que deve ser previamente criada. Após, selecionamos as chaves e os campos, definimos uma chave técnica (*Surrogate  Key- SK*) auto-incremental, aplicamos uma versão para esta, caso ocorram *updates* na tabela, com uma data alternativa de início, e criamos a tabela dimensão. Para este último passo, podemos criar a tabela no MySQL, ou diretamente no Pentaho, passo muito mais prático tendo em vista que estamos trabalho dentro deste ambiente. Executamos a query de criação da tabela clicando no botão SQL.

![order8](https://user-images.githubusercontent.com/63553829/92409999-fe872400-f118-11ea-9301-727dbfbba855.png)
![order9](https://user-images.githubusercontent.com/63553829/92410078-4f971800-f119-11ea-8dca-3fbcaa558dc3.png)

Após a execução da query, temos a mensagem de execução e criação da tabela:

![order10](https://user-images.githubusercontent.com/63553829/92410144-8e2cd280-f119-11ea-8505-57a9026413ed.png)

Pronto, ao final, quando executamos a pipelina, recebemos a mensagem de êxito:

![order11](https://user-images.githubusercontent.com/63553829/92410274-fd0a2b80-f119-11ea-897f-0c8c960f5777.png)

Finalizamos, portanto, nossa primeira **transformação e carga** de nossa primeira dimensão!


### 2. Dimensão Product

Os passos para a criação da dimensão *Product* são mais simples que os passos anteriores. Aqui, temos apenas 2 *merges* para realizar antes de sleecionar os valores finais. O diagrama completo do processo de *transformação e carga* será o seguinte:

![prod](https://user-images.githubusercontent.com/63553829/92410406-583c1e00-f11a-11ea-8d06-2b9e06fb93bd.png)

Como os passos são muito semelhantes aos anteriores, iremos prosseguir para a próxima dimensão.


### 3. Dimensão Date

A dimensão *Date* será criada do zero, então será vista com mais detalhes. O diagrama final é o seguinte:

![date1](https://user-images.githubusercontent.com/63553829/92411624-f7631480-f11e-11ea-83e5-190b26469f2a.png)

Iniciamos gerando linhas, e adicionando uma sequência de dias para esse primeiro passo, visto que a data inicial criada possui 100.000 linhas com o mesmo valor inicial (01/01/1990):

![date2](https://user-images.githubusercontent.com/63553829/92411700-56288e00-f11f-11ea-8ffc-8685dfe0105d.png)

Agora utilizamos uma caixa *Calculator* para calcular os valores de data. Aqui, é importante se atentar à correta parametrização dos campos *Field A*, *Field B*, *Value type*, e *Conversion mask*:

![date3](https://user-images.githubusercontent.com/63553829/92411718-68a2c780-f11f-11ea-8dbb-18e000da5c6e.png)

No passo anterior, todos os valores resultantes eram data ou valores inteiros. Agora aplicaremos uma fórmula que nos permitirá gerar datas como strings. Aqui, damos o nome ao campo, e ao clicarmos na aba *Formula*, preenchemos com a fórmula desejada, apresentada à direita:

![date4](https://user-images.githubusercontent.com/63553829/92411824-cb945e80-f11f-11ea-9edd-9c62721b2fa3.png)

Feitos esses passos, agora iremos mapear valores para aumentar os campos possíveis de data com nomes. Seelcionamos a base, e damos um nome ao campo antes de realizar o mapeamento dos valores:

![date5](https://user-images.githubusercontent.com/63553829/92412015-963c4080-f120-11ea-98df-2cae5b578281.png)

Por fim, removemos os dois campos criados nos dois primeiros passos, e realizamos a *carga* da dimensão *Date* no DW. Agora iremos criar nossa tabela fato *Sales*.


### 4. Fato Sales

Nosso último passo dentro do processo de *transformação e carga* é criar a tabela fato *Sales*. O diagrama completo segue na figura abaixo:

![fato](https://user-images.githubusercontent.com/63553829/92412238-883aef80-f121-11ea-89d8-49b62787ca3e.png)

Portanto, após adicionarmos a tabela de entrada *Sales*, adicionamos *lookups* a serem realizados nas dimensões presentes no DW. A figura a seguir demonstra como se configura a caixa. A ideia principal é utilizar a conexão com o MySQL DW, relacionando as chaves em comum nas dimensões e fato, retornando os valores das chaves técnicas (SKs): 

![fato2](https://user-images.githubusercontent.com/63553829/92412319-de0f9780-f121-11ea-8e3a-f97d9a4e1794.png)

Entretanto, algumas chaves técnicas são existentes somente na tabela fato. Temos então valores nulos para algumas SKs, como podemos ver no *preview* a seguir:

![fato3](https://user-images.githubusercontent.com/63553829/92412474-62fab100-f122-11ea-824f-25bfd9496645.png)

Nesse caso, aplicamos um valor simbólico -1 para as SKs, para evitarmos valores nulos. A seguir, removemos os *IDs* em comum, visto que temos as *SKs* para tal propósito:

![fato4](https://user-images.githubusercontent.com/63553829/92412546-a0f7d500-f122-11ea-9cf5-4c4da5f716dc.png)

Agora, criamos uma tabela de saída para o DW, a qual será nossa tabela fato *Sales*. Aqui, novamente devemos selecionar a conexão com o MySQL DW, dar nome e criar a tabela através do botão SQL:

![fato5](https://user-images.githubusercontent.com/63553829/92412612-def4f900-f122-11ea-8d29-b63cbe33eb1b.png)

Pronto! Após este último passo, finalizamos todo o processo de **ETL**. Temos agora nosso **DW** criado, e podemos partir para a criação de um relatório no *Power BI* para analisarmos os dados.







