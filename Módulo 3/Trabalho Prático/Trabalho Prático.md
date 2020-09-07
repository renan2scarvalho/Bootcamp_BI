# Trabalho Prático

O trabalho prático do Módulo 3 teve como objetivo exercitar os conceitos apresentados nas aulas, os quais consistiram em:
- Modelagem dimensional;
- Processo ETL;
- Ferramenta ETL;
- Processo de carga.

Nesse trabalho, foram utilizados arquivos de entrada com informações da CAPES, nos quais são feitas avaliações da pós-graduação *stritu sensu*. 
Essa avaliação tem como objetivo a certificação da qualidade da pós-graduação brasileira, bem como a identificação de assimetrias regionais e de áreas estratégicas 
do conhecimento. Ela é orientada pela Diretoria de Avaliação/CAPES, e realizada com a participação da comunidade acadêmico-científica por meio de consultores ad hoc. 
Tem como pilares a formação pós-graduada de docentes para todos os níveis de ensino e de profissionais de recursos humanos qualificados para o mercado, bem como o 
fortalecimento das bases científicas, tecnológicas e de inovação. As bases são:

![files](https://user-images.githubusercontent.com/63553829/92388585-eb129380-f0ed-11ea-8ef2-f63622cb241d.png)

Nesse trabalho foi utilizado o *software* open source Pentaho Data Integration (PDI) e o SGBD MySQL, também open source. 
Na figura abaixo temos todos os passos realizados no Pentaho. É possível notar que temos como saída um arquivo *.txt*, e duas tabelas no MySQL.

![total](https://user-images.githubusercontent.com/63553829/92394298-adb30380-f0f7-11ea-9974-36ef2594c6f5.png)

Iniciamos com os arquivos de entrada, um *csv* e outro *excel* No caso do arquivo *csv*, primeiramente selecionamos o diretório do arquivo, e selecionamos os campos desejados através do *Get Fields*:

![csv](https://user-images.githubusercontent.com/63553829/92389963-6d03bc00-f0f0-11ea-8e83-77f2c04c640d.png)

No caso do arquivo *excel* o procedimento é similar. Primeiramente, na aba *Files*, selecionamos o diretório e o adicionamos para que o arquivo apareça em *Selected files*:

![excel](https://user-images.githubusercontent.com/63553829/92390229-d8e62480-f0f0-11ea-855b-f19d39d7ab91.png)

Após, vamos à aba *Sheets*, e em *Get Sheetname(s)* selecionamos as Planilhas desejadas:

![excel2](https://user-images.githubusercontent.com/63553829/92390443-3d08e880-f0f1-11ea-9084-23e213942f7d.png)

Por fim, na aba *Fileds*, selecionamos os campos desejados, utilizando o botão *Get fileds from header now...* como facilitador:

![excel3](https://user-images.githubusercontent.com/63553829/92390535-704b7780-f0f1-11ea-8660-dc1a964725ec.png)

Nesse escopo, temos um relacionamento (1:n) entre as tabelas "Programas CAPES* e "Produção*, i.e. um programa possui diversos valores de produção. Nesse caso, aplicamos uma caixa *Stream lookup* no input *Programas*, buscando seu *ID* na tabela *Produção*, e selecionamos os campos desejados nesta última tabela:

![lookup](https://user-images.githubusercontent.com/63553829/92390692-c15b6b80-f0f1-11ea-894c-f37d53c1e861.png)

Aqui, quanto realizamos este passo, teremos o seguinte erro:

![erro](https://user-images.githubusercontent.com/63553829/92390997-4e062980-f0f2-11ea-8123-906b315ea00f.png)

Adicionamos, então, uma caixa *Dummy* para cuidar deste erro:

![dummy](https://user-images.githubusercontent.com/63553829/92391136-7f7ef500-f0f2-11ea-9b9f-b8f793a263b3.png)

Nosso próximo passo é selecionar os campos desejados, uma vez que para o resultado final, não são necessários todos os campos. Então utilizamos uma caixa *Select fields* para tal fim, selecionando os campos a seguir, modificando o nome do "Código IES":

![select](https://user-images.githubusercontent.com/63553829/92395956-975a7700-f0fa-11ea-8d5c-5b2edfa288a8.png)

Agora, após selecionados os campos, organizamos os mesmos a partir do "Código IES", e selecionamos os campos únicos, para não termos repetições:

![sort_unique](https://user-images.githubusercontent.com/63553829/92391528-3bd8bb00-f0f3-11ea-9a34-1fd26e1fd195.png)

Agora teremos nosso primeiro output: o arquivo *txt*. Aqui, selecionamos o diretório para o output, e na aba *Fields* selecionamos os campos através deo botão *Get fields*:

![txt](https://user-images.githubusercontent.com/63553829/92391671-78a4b200-f0f3-11ea-9c22-9738a60feeb8.png)
![txt2](https://user-images.githubusercontent.com/63553829/92391855-bf92a780-f0f3-11ea-9779-2026a6e09f54.png)

Esta mesma tabela será também adicionada ao MySQL. Assim, aqui teremos que configurar nossa conexão com o MySQL com uma base de dados já criada, e, caso a tabela ainda não tenha sido criada, devemos criá-la:

![mysql](https://user-images.githubusercontent.com/63553829/92392192-4778b180-f0f4-11ea-8fd4-d9e7dfae4d95.png)

Para configurar a conexão, vamos em *New* na figura acima, selecionamos *MySQL*, configuramos os parâmetros, e testamos a conexão. Caso tudo tenha sido configurado corretamente, teremos uma caixa de diálogo com a mensagem de êxito:

![conn](https://user-images.githubusercontent.com/63553829/92392059-139d8c00-f0f4-11ea-9867-01e7c7fecc20.png)

Na criação da tabela, podemos executar tal passo no MySQL, ou diretamente no Pentaho, através do botão inferior SQL, o qual abre uma nova janela com a query da tabela, a qual,m caso executada corretamente, terá como resposta a mensagem à direita:

![query](https://user-images.githubusercontent.com/63553829/92393439-49dc0b00-f0f6-11ea-85b4-5cb9b2e4eeb5.png)

Agora, para nossa próxima tabela, iremos remover os seguintes campos:

![select2](https://user-images.githubusercontent.com/63553829/92393570-7859e600-f0f6-11ea-928f-9e126a9fe79a.png)

Por fim, criamos nossa última tabela. Aqui, novamente utilizamos a conexão anterior com o MySQL, e novamente executamos a query de criação da tabela:

![mysql2](https://user-images.githubusercontent.com/63553829/92393788-d090e800-f0f6-11ea-8b5b-0b2bef771158.png)

Por fim, caso todos os passos anteriores tenham sido corretamente executados, após clicarmos em *Run*, temos a seguinte mensagem de sucesso da transformação:

![success](https://user-images.githubusercontent.com/63553829/92394475-01255180-f0f8-11ea-9573-eb67ad87d9d5.png)

Agora, após realizados os procedimentos da extração, iremos criar um *Job*, o qual visa automatizar o processo. Primeiramente adicionamos uma caixa de *Start*, e a caixa de transformação, procurando o arquivo de *Transformação* criado anteriormente. 

![job1](https://user-images.githubusercontent.com/63553829/92394677-56616300-f0f8-11ea-9e87-7832db018846.png)

É importante salientar aqui que na caixa *Start* temos a possibilidade de programar a transformação para determinados intervalos diários, semanais ou mensais(*Scheduling*). Outra possibilidade é também enviar um e-mail de confirmação da transformação:

![job2](https://user-images.githubusercontent.com/63553829/92395107-28305300-f0f9-11ea-81b8-d7c2f7bd2fbf.png)

Podemos inclusive adicionar comentários:

![job3](https://user-images.githubusercontent.com/63553829/92395177-4f872000-f0f9-11ea-9684-e6b385f8f612.png)

É interessante também adicionar uma caixa de sucesso do *Job*, caso tudo tenha sido executado corretamente.

Agora, vamos ao MySQL checar as tabelas. Primeiramente checaremos a tabela *dim_programa*, onde notamos pelo resultado da query que tudo ocorreu como esperado:

![mysql3](https://user-images.githubusercontent.com/63553829/92395617-ff5c8d80-f0f9-11ea-837a-5f18ef47674a.png)

Como comparação, vemos o arquivo *txt* também criado, que contém os mesmos valores:

![txtoutcome](https://user-images.githubusercontent.com/63553829/92395752-3894fd80-f0fa-11ea-942b-9d4458d4f0ad.png)

E por fim, vemos nossa tabela *dim_uf*, a qual contêm os dados de região contidos na tabela anterior:

![mysql4](https://user-images.githubusercontent.com/63553829/92395693-21561000-f0fa-11ea-94df-9530867931f0.png)



