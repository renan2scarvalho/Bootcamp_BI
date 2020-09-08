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

Com o modelo relacional já pronto, iremos ao Power BI Desktop para realizarmos um relatório de vendas.
O primeiro passo é, portanto, realizar a conexão do PBI com o MySQL para inserirmos as tabelas do modelo *snowflake*. Após, adicionamos a tabela *ratings*. Aqui, entretanto, temos que realizar algumas transformações, uma vez que os valores possuem aspas duplas duplicadas. Após solucionarmos essa simples transformação, os relacionamentos no PBI ficam da seguinte maneira:

![pbi](https://user-images.githubusercontent.com/63553829/92509023-f5ad5580-f1df-11ea-8fbb-316f93af1ed9.png)

O relatório final pode ser encontrado no [link](https://app.powerbi.com/view?r=eyJrIjoiMjFlZTE3NWQtZWMwNi00MTI3LWIzMzMtYzgwNzdiYjEwMzg1IiwidCI6IjdlOTNlMjg2LWIyOWEtNDQ1NC1hNDFhLWU4NDE5ZWM5ZGViNSJ9), ou através do QR code a seguir:

![Desafio Final](https://user-images.githubusercontent.com/63553829/92519120-d4546580-f1ef-11ea-96e5-349e7a9698b2.jpg)

