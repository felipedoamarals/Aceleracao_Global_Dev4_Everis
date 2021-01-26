# Live #4 - Como realizar consultas de maneira simples no ambiente complexo de Big Data com HIVE e Impala

**[Apache Hive](https://www.dezyre.com/article/impala-vs-hive-difference-between-sql-on-hadoop-components/180)** é uma infraestrutura de data warehouse construída sobre a plataforma Hadoop para realizar tarefas intensivas de dados, como consulta, análise, processamento e visualização. <br>
**[Apache Impala](https://medium.com/@markuvinicius/deep-diving-into-hadoop-world-hive-e-impala-11b1dca716ba)** O Impala traz uma tecnologia de banco de dados paralela e escalável para o Hadoop, permitindo que os usuários emitam consultas SQL de baixa latência a dados armazenados no HDFS ou HBase sem necessidade de movimentar ou transformar os dados. <br>

### 1 - Iniciando os serviços que serão utilizados
~~~shell
#Utilizando script (restart_all_service_hive.sh)
sh script_apoio/restart_all_service_hive.sh
~~~

### 2 - Acessando Hive e Impala
~~~shell
#Hive
hive

#Impala
impala-shell
~~~

### 3 - Manipulando bases de dados no Hive
~~~shell
#Exibindo as bases de dados
show databases;

#Criando base de dados
create database teste01;

#Criar base de dados caso já exista com o mesmo nome
create database if not exists teste01;

#Acessando uma base de dados
use teste01;

#Criando uma tabela
##Fora do banco de dados
create table teste01.teste01 (id int);

##Usando o banco de dados
create table teste02 (id int);

#Exibindo as tabelas
show tables;

#Exindo informações de criação das tabelas
show create table teste01;
 
#Personalizando terminal para exibir o header
set hive.cli.print.header=true;

#Personalizando terminal para exibir banco que tá sendo usado
set hive.cli.print.current.db=true;

#Inserindo dados na tabela
insert into table teste01 values(1);

#Diretório default do Hive. Obs: Sair do Hive
hdfs dfs -ls /user/hive/warehouse

#Acessando arquivos no diretório default do hive. Obs: Sair do Hive
##Base de dados
hdfs dfs -ls /user/hive/warehouse/teste01.db

##Tabela. Obs: Sair do Hive
hdfs dfs -ls /user/hive/warehouse/teste01.db/teste01

#Criando tabela do tipo externa
create external table teste03 (id int);
~~~

4 - Ingestão de base de dados
~~~shell
#Criando tabela no Hive
CREATE EXTERNAL TABLE TB_EXT_EMPLOYEE (
      id STRING,
      groups STRING,
      age STRING,
      active_lifestyle STRING,
      salary STRING)
      ROW FORMAT DELIMITED FIELDS
      TERMINATED BY '\;'
      STORED AS TEXTFILE
      LOCATION '/user/hive/warehouse/external/tabelas/employee'
      tblproperties ("skip.header.line.count"="1"); 
      
#Carregando base de dados em arquivo para o HDFS usando -PUT
hdfs dfs -put /home/everis/employee.txt /user/hive/warehouse/external/tabelas/employee

#Criando tabela formatada
CREATE TABLE TB_EMPLOYEE(
      id INT,
      groups STRING,
      age INT,
      active_lifestyle STRING,
      salary DOUBLE)
      PARTITIONED BY (dt_processamento STRING) #Coluna do tipo partição.
      ROW FORMAT DELIMITED FIELDS TERMINATED BY '|'
      STORED AS PARQUET TBLPROPERTIES ("parquet.compression"="SNAPPY"); #Armazenando em formato parquet Snappy
      
#Inserindo dados de outra tabela na tabela formatada
insert into table TB_EMPLOYEE partition (dt_processamento='20201118')
      select 
      id,
      groups,
      age,
      active_lifestyle,
      salary
      from TB_EXT_EMPLOYEE;
~~~

5 - Visualizando arquivo do tipo parquet
~~~shell
#Copiar arquivo para o disco local
hdfs dfs -copyToLocal /user/hive/warehouse/teste01.db/tb_employee/dt_processamento=20201118/000000_0 .

#Visualizar info do schema utilizando o parquet-tools
parquet-tools schema 000000_0
~~~

6 - Ingestão de base de dados II
~~~shell
#Criando tabela
create external table localidade(
      street string,
      city string,
      zip string,
      state string,
      beds string,
      baths string,
      sq__ft string,
      type string,
      sale_date string,
      price string,
      latitude string,
      longitude string)
      PARTITIONED BY (particao STRING)
      ROW FORMAT DELIMITED FIELDS TERMINATED BY ","
      STORED AS TEXTFILE
      location '/user/hive/warehouse/external/tabelas/localidade'
      tblproperties ("skip.header.line.count"="1");
      
#Carga na tabela pelo Hive
load data local inpath '/home/everis/base_localidade.csv'
into table teste01.localidade partition (particao='2021-01-21');

#Visuliando dados da tabela no Hive
select * from localidade;

#Visualizando arquivo no HDFS
hdfs dfs -ls /user/hive/warehouse/external/tabelas/localidade/particao=2021-01-21

#Visulizando conteúdo do arquivo no HDFS
hdfs dfs -cat /user/hive/warehouse/external/tabelas/localidade/particao=2021-01-21/base_localidade.csv

#Criando tabela formatada
create table tb_localidade_parquet(
      street string,
      city string,
      zip string,
      state string,
      beds string,
      baths string,
      sq__ft string,
      type string,
      sale_date string,
      price string,
      latitude string,
      longitude string)
      PARTITIONED BY (particao STRING)
      STORED AS PARQUET;

#Inserindo dados via partição na tabela formatada
INSERT into TABLE tb_localidade_parquet
      PARTITION(PARTICAO='01')
      SELECT
      street,
      city,
      zip,
      state,
      beds,
      baths,
      sq__ft,
      type,
      sale_date,
      price,
      latitude,
      longitude
      FROM localidade;
~~~

7 - Impala
~~~shell
#Acessando Apache Impala
impala-shell

#Exibindo bases de dados;
show databases;

#Atualizando bases de dados. Necessário atualizar o metastore com a base de dados e tabela que deseja invalidar.
INVALIDATE METADATA teste01.tb_localidade_parquet;
~~~

8 - Relacionamento entre tabelas no Hive
~~~shell
#Exemplo sem relação com as tabelas disponíveis durante a aula
select
      tab01.id,
      tab02.zip
      from tb_ext_employee tab01
      full outer join tb_localidade_parquet tab02
      on tab01.id = tab02.zip;

#Select com coluna com agrupamento de informações
select
      tab01.id,
      tab02.zip,
      "teste" col_fixa,
      concat(tab01.id, tab02.zip) as col_concatenada
      from tb_ext_employee tab01
      full outer join tb_localidade_parquet tab02
      on tab01.id = tab02.zip;
~~~
### Exemplo de Script 
~~~shell
#!/bin/bash

dt_processamento=$(date '+%Y-%m-%d')
path_file='/home/cloudera/hive/datasets/employee.txt'
table=beca.ext_p_employee
load=/home/cloudera/hive/load.hql

hive -hiveconf dt_processamento=${dt_processamento} -hiveconf table=${table} -hiveconf path_file=${path_file} -f $load 2>> log.txt

hive_status=$?

if [ ${hive_status} -eq 0 ];
then
        echo -e "\nScript executado com sucesso"
else
        echo -e "\nHouve um erro na ingestao do arquivo"
impala-shell -q 'INVALIDATE METADATA beca.ext_p_employee;'

fi 

LOAD DATA LOCAL INPATH '${hiveconf:path_file}' INTO TABLE ${hiveconf:table} PARTITION(dt_processamento='${hiveconf:dt_processamento}');
~~~

### Comandos adicionais
~~~shell
#Desabilitar Safe Mode no Hive
sudo -u hdfs hadoop dfsadmin -safemode leave

#Acessar informações de comandos do Hadoop
hadoop -h

#Acessar manual do Hadoop
man hadoop

#Acessar informações de comandos do HDFS
hdfs -h

#Acessar manual do HDFS
man hdfs

#Exemplo de execução do Hive sem acessá-lo
hive -S -e "select count(*) from teste01.localidade;"
~~~

### Material
[Máquina Virtual utilizada](https://hermes.digitalinnovation.one/files/acceleration/Everis_BigData-v3.ova) <br>
[Slide da aula*]() *slide não disponível até o momento