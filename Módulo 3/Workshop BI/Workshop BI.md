# Workshop BI

A ideia do workshop foi demonstrar de maneira prática e objetiva como transformar um modelo de base transacional (OLTP) em um modelo dimensional (OLAP) para Data Warehouse (DW).O projeto utiliza quatro planilhas em formato *csv* de uma empresa de distribuição de artigos de tecnologia (hardware e software):
- customer
- orders
- products
- promotions
- salesrep

O desenvolvimento do trabalho consiste nos seguintes passos:
1. Identificar as fontes de dados e conhecer o modelo dimensional resultante;
2. Criar uma staging area e realizar a extração dos dados;
3. Implementar um modelo dimensional que se adeque à visão de negócio;
4. Criar estrutura das tabelas dimensão e fato (DW);
5. Organizar o fluxo do processo ETL.

As ferramentas utilizadas para esse processo ETL e criação de DW foram o Pentaho Data Integration (PDI) e o Sistema de Gerenciamento de Banco de Dados (SGBD) MySQL, ambos **open source**, o que nos permite um processos de qualidade com custo zero. Vamos então aos processos!


### 1. Modelo Transacional e Dimensional

O daataset consiste em um esquema clássico de varejo, com uma lista de produtos oferecidos pela empresa, os quais se enquadram em hierarquias e categorias.
Um pedido é constituído por vários itens, e é atrelado a um representante de vendas, e também pode estar atrelado a uma promoção.
O esquema abaixo representa o esquema transacional, onde cada Chave Primária (Primary Key - PK) são responsáveis pela identificação única de cada item, enquanto as Chaves Estrangeiras (Foreign Key - FK) são responsáveis por ligar as tabelas.

![trans_sch](https://user-images.githubusercontent.com/63553829/91753039-a8a50000-eb9d-11ea-9bd9-18bfd3425ccb.png)

Adentrando no modelo dimensional, as PK recebem outro nome quando no DW: Chave Substituta (Surrogate Key - SK), a qual nada mais é que uma chave artificial, e existe para manter a integridade das informações de uma dimensão, pois possui as características de uma PK. A PK, quando vem de um banco transacional, é também chamada de *Natural Key* ou *Business Key*. O diagrama estrela do modelo dimensional será o seguinte:

![star_sch](https://user-images.githubusercontent.com/63553829/91753371-2cf78300-eb9e-11ea-9395-eb857fd85a3b.png)


### 2. Staging Area

A *staging area* consiste na criação de uma área temporária e de extração das tabelas para a mesma, a fim de padronizá-las e seguir para o passo de transformação.
Como agora é o momento de fazer todo o tratamento e junções dos dados, podemos presumir que isto exigirá um bom custo de processamento dos dados. Processamento este que se fosse feito diretamente nos bancos de origem, provavelmente afetaria a performance dos sistemas de produção, pois exige bastante recurso computacional. Outro motivo é que em muitos casos é necessário cruzar informações existentes em bancos fisicamente separados, então a *staging area* deixa as tabelas no mesmo local para tal cruzamento.
A staging area nada mais é do que um banco relacional que serve de repositório para as informações extraídas do banco transacional para que possam ser trabalhos e tratados. Potanto, sua criação é realizada no MySQL, SGBD escolhido para esse trabalho:

```jaavscript
CREATE DATABASE stg_workshopbi;
```

Agroa, realizamos a extração dos dados dentro do PDI. Como o formato de todas as tabelas é *csv*, realizaremos a extração de todas as tabelas de uma vez (o ideal é realizar a extração de um input por vez), conforme a figura a seguir. Primeiramente adicionamos as caixas *CSV file input* para cada tabela, e adicionamos caixas *Table output* para cada entrada:

![ext](https://user-images.githubusercontent.com/63553829/91754286-88764080-eb9f-11ea-8556-bd2292a49f8a.png)

Assim, para cada entrada, buscamos o diretório do arquivo, e selecionamos os campos através do botão *Get Fields*. Nessa etapa é importante dar um *Preview* nos campos da tabela, para alterar/padronizar cada atributo caso seja necessário. É importante notar aqui que o foco não é no formato, tamanho e precisão de cada atributo, e sim na extração.

![ext2](https://user-images.githubusercontent.com/63553829/91755549-acd31c80-eba1-11ea-9dfd-b650377bff5a.png)

A seguir, configuraremos uma nova conexão com o MySQL para extração de dados para a *staging area*. Neste passo, é importante testar a conexão para ver se a mesma obteve êxito. Por fim, precisamos criar a tabela que recebera os dados na *staging area*. Existem duas possibilidades para a execução: criar a tabela no MySQL, ou criar a tabela no PDI, apss mais simples, como representado abaixo.

![ext3](https://user-images.githubusercontent.com/63553829/91756255-d80a3b80-eba2-11ea-8210-7bbf2ada440a.png)
![ext4](https://user-images.githubusercontent.com/63553829/91756804-b9f10b00-eba3-11ea-953c-c9ec7368c3b9.png)

O próximo seria realizar a extração. Entretanto, caso executemos tal passo, um erro aparecerá:

![ext5](https://user-images.githubusercontent.com/63553829/91757255-6501c480-eba4-11ea-9dd7-d53fde833c03.png)

Nesse caso, se dermos um preview no atributo *PRODUCT_DESCRIPTION* notaremos que realmente um *Data type TINYTEXT* não seria capaz de armazenar os dados e, portanto, alteramos durante a criação da tabela no código SQL o atributo de *TINYTEXT* para *LONGTEXT*.

![ext6](https://user-images.githubusercontent.com/63553829/91757414-a4c8ac00-eba4-11ea-961d-af6ec0d5bf38.png)

Agora, quando executamos a extração, o processe terá êxito:

![ext7](https://user-images.githubusercontent.com/63553829/91757776-46e89400-eba5-11ea-80fc-27107b88878b.png)

Pronto! Um passo foi dado: realizamos a **extração!** Caso queira, é possível executar uma query rapidamente no MySQL para ver o êxito da extração:

```javascript
SELECT * FROM table
```


### 3. Transformação para Adequação

Aplicaremos a visão de negócio (**sempre!**) para este caso para implementarmos o modelo dimensional. Os arquivos compreendem o banco transacional de uma empresa. Aplicando a técnica 5W2H já é possível ter uma visão geográfica, financeira, comportamental e temporal:
- (Who) Quem são os clientes?
- (What) O que eles compram?
- (Where) De onde eles compram?
- (Why) Por que eles compram?
- (When) Quando eles compram?
- (How much) Quanto eles pagam?
- (How) Quanto eles têm disponível para comprar?

Agora aplicaremos, portanto, as transformações necessárias em cada dimensão do modelo dimensional apresentado acima.

#### 1. Dimensão Produto

Iniciaremos com a Dimensão Produto, que é uma das mais simples de ser realizada, consistindo apenas em inserir a tabela utilizando a mesma conexão com o MMySQL anterior, e organizar as linhas de acordo com a PK *PRODUCT_ID*:

![dim_prod](https://user-images.githubusercontent.com/63553829/91759318-d98a3280-eba7-11ea-91d2-725d97dbb5c5.png)

Agora, após essa simples **transformação**, iremos realizar a **carga** da tabela no DW. Entretanto, ainda devemos criar o DW, executando o seguinte código no MySQL:

```javascript
CREATE DATABASE dw_workshopbi
```

A seguir, adicionamos o bloco *Dimension lookup/update*, e criamos uma nova conexão com DW no MySQL (lembre-se de testar a conexão):

![dim_prod2](https://user-images.githubusercontent.com/63553829/91759735-88c70980-eba8-11ea-9620-297b40e9e100.png)

Agora, iremos configurar a dimensão, iniciando pelo nome, o qual deve ser inserido no *Target table*, selecionamos a PK *PRODUCT_ID*, selecionamos os campos na aba *Fields* com o botão *Get Fields*. Agora, criaremos uma *Technical key*, que será a SK na tabela fato, a qual chamamos de *sk_product* com auto-incremento e uma versão:

![dim_prod3](https://user-images.githubusercontent.com/63553829/91760152-389c7700-eba9-11ea-87e2-472c2fb9b5a4.png)

Por fim, adicionamos uma data alternativa como início da SK em sua primeira versão. Mas ainda não temos uma tabela criada no MySQL para a Dimensão Produto. Devemos, portanto, 
executar o a query SQL para criação da tabela

![dim_prod4](https://user-images.githubusercontent.com/63553829/91760605-04758600-ebaa-11ea-8a34-86172ffa5ada.png)

Após realizados todos os procedimentos anteriores, podemos executar a pipeline da Dimensão Produto, a qual será realizada com êxito:

![dim_prod5](https://user-images.githubusercontent.com/63553829/91760779-54544d00-ebaa-11ea-95f5-9014f57d6e90.png)

Acabamos de realizar a primeira **transformação e carga** da primeira dimensão de nosso modelo relacional!

#### 2. Dimensão Promoção

A Dimensão Promoção possui exatamente a mesma pipeline de extração e carga da Dimensão Produto e, por isso, não será abordada com maior escrutínio.

![dim_prom](https://user-images.githubusercontent.com/63553829/91762478-c8432500-ebab-11ea-9536-be89ed7c35cc.png)

#### 3. Dimensão Consumidor

A Dimensão Consumidor se inicia com uma caixa de ordenação seguida por uma caixa de operações de texto, na qual o atributo *CUST_EMAIL* é passado para caixa baixa:

![dim_cons](https://user-images.githubusercontent.com/63553829/91763740-6a630d00-ebac-11ea-98dd-4e73181525eb.png)

A seguir, substituímos valores nulos de alguns atributos importantes com a caixa *If field value is null*:

![dim_cons2](https://user-images.githubusercontent.com/63553829/91763863-9e3e3280-ebac-11ea-9f5f-f837ecad1af4.png)

E selecionamos os atributos necessários de acordo com nossa visão de negócio:

![dim_cons3](https://user-images.githubusercontent.com/63553829/91764100-16a4f380-ebad-11ea-931f-ca737ff0294c.png)

Após essas *transformações*, podemos popular a tabela fato através da caixa dimensão, a qual deve ser preenchida como apresentado na Dimensão Produto, utilizando a mesma conexão com o MySQL. Ao final, teremos realizado nossa terceira **transformação e carga**.

#### 4. Dimensão Representante de Vendas

Agora iremos realizar a transformação e carga da Dimensão Representante de Vendas. Nesse caso, antes de ordenarmos, devemos realizar uma filtragem. Isso porque, quando aplicamos uma visualização dos dados, temos um *EMPLOYEE_ID* com valor nulo na primeira linha. Iniciamos, portanto, filtrando tal valor com uma caixa *Filter rows* aplicada a tal atributo, direcionando ao próximo passo do diagrama quando os valores são Falsos, i.e. quando *EMPLOYEE_ID* não é 0, e direcionando os valores iguais a 0 para uma caixa *Dummy*, a qual não realiza nada:

![dim_sr](https://user-images.githubusercontent.com/63553829/91764602-f9bcf000-ebad-11ea-8d1f-75f1221a28f5.png)

Trocamos o ID_DEPARTMENT para 80

![dim_sr](https://user-images.githubusercontent.com/63553829/91764982-9f705f00-ebae-11ea-96b7-d2ee37a0b5c0.png)







