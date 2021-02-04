# Live #10 - Criando pipelines de dados eficientes - Parte 2 - Spark SQL - PySpark

**[Apache Spark](https://pt.wikipedia.org/wiki/Apache_Spark)** é um framework de código fonte aberto para computação distribuída. Ele provê uma interface para programação de clusters com paralelismo e tolerância a falhas. <br>

Profº [**Marco Antonio Pereira**](https://www.linkedin.com/in/marcoap/) <br>

### 1 - SparkSQL

1.1 - Registrar no Kaggle e baixar a base: https://www.kaggle.com/gpreda/covid-world-vaccination-progress 

1.2 - Criar uma nova tabela no Databricks e fazer upload da base baixada: Data > Create Table

1.3 - Copiar caminho de origem da base no Databricks: DBFS > FileStore > tables > country_vaccinations.csv
    
    Exemplo: /FileStore/tables/country_vaccinations.csv

1.4 - Criar um notebook
~~~Python
#cmd1--------
# Path - dataset1
path_dataset1 = "/FileStore/tables/country_vaccinations.csv"

# Path - RDD
path_rdd = "/FileStore/tables/arquivo_rdd.txt"

#cmd2-------
# Leitura de Dataframe

## Opção 1
df1 = spark.read.format("csv").option("header","true").load(path_dataset1)

## Opção 2
#df1 = spark.read.csv(path_dataset1)
#df1 = spark.read.option("header","true").option("inferSchema","true").csv(path_dataset1)

## Exibindo dataframe
df1.show(2)

#cmd3-------
#Salvando dataframe em Parquet. Camada Rawzone
df1.write.format("parquet").save("/FileStore/tables/RAW_ZONE_PARQUET/")

#Salvando dataframe, dividindo o arquivo e subscrevendo
df1.repartition(2).write.format("parquet").mode("overwrite").save("/FileStore/tables/RAW_ZONE_PARQUET/")

#Visualiza linha de dataframe como objeto
df1.take(1)

#cmd4-------
## Outras formas de leitura de arquivos com PySpark

path = "/../../arquivoXPTO"

# Criando um dataframe a partir de um JSON
dataframe = spark.read.json(path)

# Criando um dataframe a partir de um ORC
dataframe = spark.read.orc(path)

# Criando um dataframe a partir de um PARQUET
dataframe = spark.read.parquet(path)

#cmd5-------
## Imprimindo tipos de campos

df1.dtypes
#df1.printSchema

#cmd6-------
# Leitura de um RDD

rdd = sc.textFile(path_rdd)
#rdd.show() = Errado, não é possível exibir um SHOW() de um RDD, somente um Dataframe
rdd.collect()

#cmd7-------
# Criando uma tabela temporária
nome_tabela_temporiaria = "tempTableDataFrame1"
df1.createOrReplaceTempView(nome_tabela_temporiaria)

#cmd8-------
# Lendo a tabela temporaria opcao 1
spark.read.table(nome_tabela_temporiaria).show()

#cmd9-------
spark.sql("SELECT * FROM tempTableDataFrame1").show()

#cmd10-------
# Visualização do Databricks
display(spark.sql("SELECT * FROM tempTableDataFrame1"))

#cmd11-------
# Scala
#import org.apache.spark.sql.functions._

# Python
from pyspark.sql.functions import col, column

# Usando function col ou column
df1.select(col("country"), col("date"), column("iso_code")).show()

#cmd12-------
df1.selectExpr("country", "date", "iso_code").show()

#cmd13-------
# Scala import
# org.apache.spark.sql.types._

# Criando um Schema manualmente no PySpark
from pyspark.sql.types import *

dataframe_ficticio = StructType([
                      StructField("col_String_1", StringType()),
                      StructField("col_Integer_2", IntegerType()),
                      StructField("col_Decimal_3", DecimalType())
                              ])
dataframe_ficticio

#cmd14-------
# Função para gerar Schema (campos/colunas/nomes de colunas)

'''
# Scala

org.apache.spark.sql.types._

def getSchema(fields : Array[StructField]) : StructType = {
  new StructType(fields)
}
'''

# PySpark
def getSchema(fields):
  return StructType(fields)
  
schema = getSchema([StructField("coluna1", StringType()), StructField("coluna2", StringType()), StructField("coluna3", StringType())])

#cmd15-------
schema

#cmd16-------
# Gravando um novo CSV

path_destino="/FileStore/tables/CSV/"
nome_arquivo="arquivo.csv"
path_geral= path_destino + nome_arquivo
df1.write.format("csv").mode("overwrite").option("sep", "\t").save(path_geral)

#cmd17-------
# Gravando um novo JSON

path_destino="/FileStore/tables/JSON/"
nome_arquivo="arquivo.json"
path_geral= path_destino + nome_arquivo
df1.write.format("json").mode("overwrite").save(path_geral)

#cmd18-------
# Gravando um novo PARQUET

path_destino="/FileStore/tables/PARQUET/"
nome_arquivo="arquivo.parquet"
path_geral= path_destino + nome_arquivo
df1.write.format("parquet").mode("overwrite").save(path_geral)

#cmd19-------
# Gravando um novo ORC

path_destino="/FileStore/tables/ORC/"
nome_arquivo="arquivo.orc"
path_geral= path_destino + nome_arquivo
df1.write.format("orc").mode("overwrite").save(path_geral)

#cmd20-------
# Outros tipos de SELECT

#Diferentes formas de selecionar uma coluna

from pyspark.sql.functions import *

df1.select("country").show(5)
df1.select('country').show(5)
df1.select(col("country")).show(5)
df1.select(column("country")).show(5)
df1.select(expr("country")).show(5)

#cmd21-------
# Define uma nova coluna com um valor constante
df2 = df1.withColumn("nova_coluna", lit(1))

# Adicionar coluna
teste = expr("total_vaccinations < 40")
df1.select("country", "total_vaccinations").withColumn("teste", teste).show(5)

# Renomear uma coluna
df1.select(expr("total_vaccinations as total_de_vacinados")).show(5)
df1.select(col("country").alias("pais")).show(5)
df1.select("country").withColumnRenamed("country", "pais").show(5)

# Remover uma coluna
df3 = df1.drop("country")
df3.columns

#cmd22-------
# Filtrando dados e ordenando
# where() é um alias para filter().

# Seleciona apenas os primeiros registros da coluna "total_vaccinations"
df1.filter(df1.total_vaccinations > 55).orderBy(df1.total_vaccinations).show(2)

# Filtra por país igual Argentina
df1.select(df1.total_vaccinations, df1.country).filter(df1.country == "Argentina").show(5)

# Filtra por país diferente Argentina
df1.select(df1.total_vaccinations, df1.country).where(df1.country != "Argentina").show(5) # python type

# Mostra valores únicos
df1.select("country").distinct().show()

# Especificando vários filtros em comando separados
filtro_vacinas = df1.total_vaccinations < 100
filtro_pais = df1.country.contains("Argentina")
df1.select(df1.total_vaccinations, df1.country, df1.vaccines).where(df1.vaccines.isin("Sputnik V", "Sinovac")).filter(filtro_vacinas).show(5)
df1.select(df1.total_vaccinations, df1.country, df1.vaccines).where(df1.vaccines.isin("Sputnik V", "Sinovac")).filter(filtro_vacinas).withColumn("filtro_pais", filtro_pais).show(5)

#cmd23-------
"""#######################################################################################################################
Convertendo dados
#######################################################################################################################"""

df5 = df1.withColumn("PAIS", col("country").cast("string").alias("PAIS"))
df5.select(df5.PAIS).show(2)

"""#######################################################################################################################
Trabalhando com funções
#######################################################################################################################"""

# Usando funções
df1.select(upper(df1.country)).show(3)
df1.select(lower(df1.country)).show(4)

#cmd24-------
# Criando um dataframe genérico

d = [{'name': 'Alice', 'age': 1}]
df_A = spark.createDataFrame(d)
df_A.show()

#cmd25-------
rdd1 = [{"nome": "Marco","idade": 33,"status": 'true'},
{"nome": "Antonio","idade":33,"status": 'true'},
{"nome":"Pereira","idade":33,"status": 'true'},
{"nome":"Helena","idade":30,"status": 'true'},
{"nome":"Fernando","idade":35,"status": 'true'},
{"nome":"Carlos","idade":28,"status": 'true'},
{"nome":"Lisa","idade":26,"status": 'true'},
{"nome":"Candido","idade":75,"status": 'false'},
{"nome":"Vasco","idade":62,"status": 'true'}
]
dff1 = spark.createDataFrame(rdd1)
dff1.show()


rdd2 = [
{"nome":"Marco","PaisOrigem":"Brasil"},
{"nome":"Helena","PaisOrigem":"Brasil"},
{"nome":"Gabriel","PaisOrigem":"Brasil"},
{"nome":"Vasco","PaisOrigem":"Portugal"},
{"nome":"Medhi","PaisOrigem":"Marocco"}]

dff2 = spark.createDataFrame(rdd2)
dff2.show()

'''
join_type = "inner"

+------+-----+------+------+----------+
|  nome|idade|status|  nome|PaisOrigem|
+------+-----+------+------+----------+
| Vasco|   62|  true| Vasco|  Portugal|
| Marco|   33|  true| Marco|    Brasil|
|Helena|   30|  true|Helena|    Brasil|
+------+-----+------+------+----------+
'''

'''val join_type = "left_semi"

+------+-----+------+
|  nome|idade|status|
+------+-----+------+
| Vasco|   62|  true|
| Marco|   33|  true|
|Helena|   30|  true|
+------+-----+------+
'''

'''val join_type = "right_outer"

+------+-----+------+-------+----------+
|  nome|idade|status|   nome|PaisOrigem|
+------+-----+------+-------+----------+
| Vasco|   62|  true|  Vasco|  Portugal|
| Marco|   33|  true|  Marco|    Brasil|
|  null| null|  null|Gabriel|    Brasil|
|Helena|   30|  true| Helena|    Brasil|
|  null| null|  null|  Medhi|   Marocco|
+------+-----+------+-------+----------+
'''

'''val join_type = "left_outer"

+--------+-----+------+------+----------+
|    nome|idade|status|  nome|PaisOrigem|
+--------+-----+------+------+----------+
| Antonio|   33|  true|  null|      null|
|   Vasco|   62|  true| Vasco|  Portugal|
|   Marco|   33|  true| Marco|    Brasil|
| Pereira|   33|  true|  null|      null|
|  Carlos|   28|  true|  null|      null|
|Fernando|   35|  true|  null|      null|
| Candido|   75| false|  null|      null|
|  Helena|   30|  true|Helena|    Brasil|
|    Lisa|   26|  true|  null|      null|
+--------+-----+------+------+----------+
'''

'''join_type = "full_outer"

+--------+-----+------+-------+----------+
|    nome|idade|status|   nome|PaisOrigem|
+--------+-----+------+-------+----------+
| Antonio|   33|  true|   null|      null|
|   Vasco|   62|  true|  Vasco|  Portugal|
|   Marco|   33|  true|  Marco|    Brasil|
| Pereira|   33|  true|   null|      null|
|  Carlos|   28|  true|   null|      null|
|    null| null|  null|Gabriel|    Brasil|
|Fernando|   35|  true|   null|      null|
| Candido|   75| false|   null|      null|
|  Helena|   30|  true| Helena|    Brasil|
|    Lisa|   26|  true|   null|      null|
|    null| null|  null|  Medhi|   Marocco|
+--------+-----+------+-------+----------+
'''

'''join_type = "left_anti"

+--------+-----+------+
|    nome|idade|status|
+--------+-----+------+
| Antonio|   33|  true|
| Pereira|   33|  true|
|  Carlos|   28|  true|
|Fernando|   35|  true|
| Candido|   75| false|
|    Lisa|   26|  true|
+--------+-----+------+
'''

join_type = "left_anti"
join_condition = dff1.nome == dff2.nome
df3 = dff1.join(dff2, join_condition, join_type)
df3.show()

#df1.groupBy("status").agg(countDistinct(col("idade"))).show()
~~~

### Material
[dontpad](dontpad.com/PySparkParte2)
[notebook](https://databricks-prod-cloudfront.cloud.databricks.com/public/4027ec902e239c93eaaa8714f173bcfc/5006635778553324/115959838911770/5180509889353222/latest.html)
[Slide da aula](https://drive.google.com/file/d/1Hl7NaISZ8_u8es8q5cK2U6VZ_mzNbD35/view?usp=sharing)

