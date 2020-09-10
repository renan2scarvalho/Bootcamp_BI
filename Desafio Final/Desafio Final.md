# Desafio Final

O Desafio Final do Bootcamp consistiu em aplicar todo o conhecimento adquirido ao longo do curso, exercitando os seguintes conceitos:
- Conceitos e fundamentos de BI;
- Ferramentas de BI;
- Processo ETL;
- Relatórios e Dashboards.

## O trabalho

O trabalho trata de uma empresa que vende mobílias e material de escritório. Os dados contém vendas entre os anos de 2011 e 2014, e o principal objetivo do trabalho é criar um dashboard para auxiliar a alta gestão na tomada de decisão.

Par a execução, foi utilizada uma base SQL com o seguinte esquema *snowflake*:

![snowflake](https://user-images.githubusercontent.com/63553829/92404035-09d25380-f109-11ea-86cd-3757aacf09c2.png)

E utilizado um arquivo *csv* com a avaliação dos clientes:

![rating](https://user-images.githubusercontent.com/63553829/92404218-6766a000-f109-11ea-9a7b-d1898655a61e.png)

Com o modelo relacional já pronto, iremos ao Power BI Desktop para realizarmos um relatório de vendas.
O primeiro passo é, portanto, realizar a conexão do PBI com o MySQL para inserirmos as tabelas do modelo *snowflake*. Após, adicionamos a tabela *ratings*. Aqui, entretanto, temos que realizar algumas transformações, uma vez que os valores possuem aspas duplas duplicadas. Após solucionarmos essa simples transformação, os relacionamentos no PBI ficam da seguinte maneira:

![pbi](https://user-images.githubusercontent.com/63553829/92509023-f5ad5580-f1df-11ea-8fbb-316f93af1ed9.png)

O relatório final pode ser encontrado no [link](https://app.powerbi.com/view?r=eyJrIjoiMjFlZTE3NWQtZWMwNi00MTI3LWIzMzMtYzgwNzdiYjEwMzg1IiwidCI6IjdlOTNlMjg2LWIyOWEtNDQ1NC1hNDFhLWU4NDE5ZWM5ZGViNSJ9&pageName=ReportSection)


## Highlights:

Como comentado no início, o desahboard consiste em vendas entre os anos de 2011 e 2014 de uma empresa que possui em seu portfólio de vendas mobílias e materiais para escritório.

- Gerais:
  - $6.56M em vendas, com lucro líquido de $0,60M e ticket médio de $5.81 (considerando negativos) para um total de 30k pedidos, e uma média de 3,74 produtos por pedido.
  
- Países:
  - País com maior lucro foi EUA. Entretanto, a Alemanha vem demonstrando grande crescimento entre 2012 e 2013, estável em 2014. Mesmo ocorreu com a Índia entre 2011 e 2012, e entre 2013 e 2014. Tais variações nestes países são importantes pois, como veremos abaixo, as melhorias nas avaliações pelos consumidores ocorreu também entre 2012 e 2013, e principalmente 2013 e 2014, fatos, portanto, que podem estar atrelados.

- Produtos:
  - Produto "Sauder Classic Bookcase" obteve maior lucro ($10,6k) com 113 vendas teve uma razão lucro-vendas de 94,44  i.e. para cada unidade vendida, o lucro foi de $94,44;
  - Produto "Hoover Stove" obteve maior razão lucro-vendas ($263,28), com lucro de $5k e 19 vendas, ou seja, é um produto mais caro;
  - Produto "Bevis Conference Table" obteve prejuízo de $4k, com 59 vendas, resultando em uma razão lucro-vendas de -$74,19, já próxima ao produto com maior lucro.
  
- Avaliações dos consumidores: entre 1 e 5
  - As avaliações melhoraram com o tempo, e o número de avaliações aumentou, ou seja, mais clientes dando feedback sobre as compras. Em 2011, de um total de 2,1k de avaliações, aproximadamente metade eram baixas (1 e 2), proporção mantida também em 2012, porém com total de 2,5k avaliações. Já em 2013, existiam somente avaliações 3, 4 e 5 em proporção 1:1:1,1, de um total de 3,1k. Em 2014, existiram apenas avaliações 4 e 5 de um total de 4k, em proporções 1:1.

- Pedidos e Avaliações:
  - De maneira geral, o número de pedidos aumento com os anos, assim como o número de avaliações dos consumidores, em taxas muito semelhantes, como pode ser notado pela razão entre número de pedidos e número de avaliações (app. 2,58). O número de avaliações precisa aumentar, mas, como vimos, o número de avaliações positivas cresceu juntamente com o número total de avaliações, o que é uma notícia positiva.

- Lucro Anual por Categoria:
  - O lucro por categoria foi semelhante em 2011 e 2012, porém a partir de 2013, a categoria de materiais para escritório vem se destacando.
  - Tratando de sazonalidades, a categoria mobílias tem uma tendência a um segundo trimestre fraco em vendas, com melhoria no terceiro e quarto trimestres, tendência semelha da categoria materiais para escritório, porém com menor intensidade. O ano de 2012 foge à regra, pois o lucro foi razoavelmente constanto ao longo de todo ano.

- Lucro Anual Total:
  - Como esperado, o lucro total segue a tendência apresentada pelas categorias, com forte aumento entre 2012 e 2013, e se manteve estável em 2014, com ligeiro aumento.
  
- Pedidos Anuais:
  - O número de pedidos anuais tem uma tendência sazonal de alta em Junho, redução até Agosto, e nova alta até o fim do ano. Essa tendência é importante caso sejam necessárias contratações de funcionários temporários para suprir a demanda. O início de todos os anos possui menor demanda, como comentado em relação ao lucro anual.
  
