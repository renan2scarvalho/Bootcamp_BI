## ETL

Nesse Desafio o objetivo foi realizar o processo completo do ETL no Pentaho Data Integration (PDI), a partir de sete tabelas *excel* e arquivos *csv*:

![tabelas](https://user-images.githubusercontent.com/63553829/91566283-cd894100-e919-11ea-910d-aa6f59225b61.png){: .mx-auto.d-block :}

Essas tabelas seguem o esquema *snow flake*, como apresentado adiante:

![sf_mdl](https://user-images.githubusercontent.com/63553829/91566200-afbbdc00-e919-11ea-9722-d46bfaa58656.png)

Assim, a partir das tabelas de origem, o objetivo é modelar um Data Warehouse (DW) como o seguinte esquema *star*:

![star_mdl](https://user-images.githubusercontent.com/63553829/91566611-51dbc400-e91a-11ea-875e-2d944ea62019.png)

## Agora, ao trabalho!

Agora que já sabemos quais são as tabelas e o esquema proposto, iremos realizar o primeiro passo, que é a criação de uma **staging area**, i.e. criação de uma área temporária e  extração das tabelas para a mesma, a fim de padronizá-las e seguir para o passo de transformação. Portanto, no MySQL Workbench, criamos a base de dados com o código a seguir:

```javascript
CREATE DATABASE stg_desafio;
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



