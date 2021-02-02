# Live #8 - Processando grandes conjuntos de dados de forma paralela e distribuída com Spark

**[Apache Spark](https://pt.wikipedia.org/wiki/Apache_Spark)** é um framework de código fonte aberto para computação distribuída. Ele provê uma interface para programação de clusters com paralelismo e tolerância a falhas. <br>

Profº [**Ivan Falcão**](https://www.linkedin.com/in/ivanpfalcao/) <br>

### Requisitos básicos:
- Conhecimentos básicos de Shellscript
- Conhecimentos básicos de Linux
- Conhecimentos básicos de linguagens de programação

### 1 - Instalação e execução
1.1 - Baixar a versão desejada no site: https://spark.apache.org/downloads.html

1.2 - Descompactar o arquivo
~~~shell
tar -zxvf spark-2.4.7-bin-hadoop2.7.tgz
~~~

1.3 - Mover para o local desejado;
~~~shell
mv caminho pasta
~~~

1.4 - Configurar a variável de ambiente SPARK_HOME, para a pasta onde se encontra o Spark;

1.5 - Configurar no PATH do sistema a pasta bin, dentro do diretório do Spark;
~~~shell
#No caso do Linux, adicionar ao arquivo /etc/bash.bashrc (Debian) ou /etc/bashrc (RHEL):
export SPARK_HOME="/opt/spark-2.3.1-bin-hadoop2.7"
PATH="$PATH:$SPARK_HOME/bin"
export PYSPARK_PYTHON="python3"
~~~

### 2 - Acessando Spark

2.1 - Via Spark Shell
~~~shell
#Baixando base de exemplo
$ wget https://raw.githubusercontent.com/fivethirtyeight/data/master/avengers/avengers.csv

#Acessando Spark Shell
$ spark-shell

#Lendo arquivo csv local
val insurance = spark.read.format("csv").option("sep",",").option("header","true").load("file:///home/everis/avengers.csv")

#Visualizar dataframe
insurance.show(10, false) #(10) número de linhas para exibir | (false) função truncate

#Criando outro dataframe com dados tratados
val ds2 = insurance.select("URL")

#Visualizando novo dataframe
ds2.show()
~~~

2.2 - Via Pyspark shell
~~~shell
#acessando
$ pyspark

#Lendo um arquivo csv local
insurance = spark.read.format("csv").option("sep",",").option("header","true").load("file:///home/everis/avengers.csv")

#Visualizando dataframe
insurance.show
~~~

2.3 - Via Spark SQL
~~~shell
#Acessando Spark SQL
$ spark-sql

#Lendo um arquivo local
SELECT * FROM csv.`file:///home/everis/avengers.csv`;
~~~

2.4 - Via Spark R shell
~~~shell
#O Spark R depende do R estar instalado no sistema
$ sparkR 
~~~

2.5 - Pode ser via Jupyter notebook ou Zeppelin Notebook

### 3 - SparkSQL
Criando Spark Session:
~~~shell
import org.apache.spark.sql.SparkSession

val spark = SparkSession
  .builder()
  .appName("Spark SQL")
  .config("configuracao","valor da configuracao")
  .getOrCreate()
~~~

Algumas formas de leitura de dados no SparkSQL:
~~~shell
#json
val dfJson = spark.read.json("file:///home/spark/Downloads/people.json")

#Parquet
val dfParquet = spark.read.format("parquet").load("file:///home/spark/Downloads/people.parquet")

#CSV
val peopleDFCsv = spark.read.format("csv").option("sep",",").option("header","true").load("file:///home/spark/Downloads/FL_insurance_sample.csv")

#JDBC
val jdbcDF = spark.read
  .format("jdbc")
  .option("url","jdbc:postgresql:dbserver")
  .option("dbtable","schema.tablename")
  .option("user","username")
  .option("password","password")
  .option("driver","com.driver.MyDriver")
  .load()
~~~

Operações de Dataframes
~~~shell
#Visualizar a estrutura da tabela
df.printSchema()

#Visualizar os dados da tabela
df.show(50.false)

#Consulta em campos específicos 
df.select("field1","field2").show()

#Realizando operações
df.select($"field1",$"field2"+1).show()

#Filtrando dados
df.filter($"age">21).show()

#Agrupamento
df.groupBy("age").count().show()

#Renomear coluna
df.withColumn("new_column_name",col("old_column_name").cast("long")).show()

#
df.withColumn("new_column_name",col("old_column_name").cast("long")).show()

#Média
df.avg("age").show()

#Soma
df.sum("sales").show()

#Maior valor
df.max("age").show()
~~~

### 4 - Atividade
Baixar o arquivo: https://raw.githubusercontent.com/shankarmsy/practice_Pandas/master/FL_insurance_sample.csv
~~~shell
$ wget https://raw.githubusercontent.com/shankarmsy/practice_Pandas/master/FL_insurance_sample.csv
~~~
Obter a média do campo eq_site_limit, agrupado por construction
~~~shell
#Lendo dataframe
val dfFL = spark.read.format("csv").option("sep",",").option("header","true").load("file:///home/everis/FL_insurance_sample.csv")

#Criado Dataframe no contexto
dfFL.createOrReplaceTempView("av")

#Realizando consulta
spark.sql("SELECT SUM(eq_site_limit) FROM av GROUP BY construction").show(10, false)

#Resultado
#+----------------------------------+                                            
#|sum(CAST(eq_site_limit AS DOUBLE))|
#+----------------------------------+
#|5.2160724E9                       |
#|5.1695028629999834E8              |
#|1.625976E10                       |
#|1.794733355699995E9               |
#|3.0094492608000007E9              |
#+----------------------------------+
~~~
### Material
[Máquina Virtual utilizada](https://hermes.digitalinnovation.one/files/acceleration/Everis_BigData-v3.ova) <br>
[Slide da aula](https://drive.google.com/file/d/1sLEOp1T2nmH74yT0CLHDNzLcHLtVqIm3/view)