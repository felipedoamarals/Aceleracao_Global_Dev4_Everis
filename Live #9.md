# Live #9 - Criando pipelines de dados eficientes - Parte 1 - Sparke Streaming

**[Apache Spark](https://pt.wikipedia.org/wiki/Apache_Spark)** é um framework de código fonte aberto para computação distribuída. Ele provê uma interface para programação de clusters com paralelismo e tolerância a falhas. <br>

Profº [**Marco Antonio Pereira**](https://www.linkedin.com/in/marcoap/) <br>

Requisitos Básicos
- Máquina Virtual com Apache Spark 2.4
- Conhecimentos básicos em Python ou Scala
- Paciência

### 1 - Sobre

Exemplos de RDD:

Python
~~~shell
rddArquivo = sc.textFile ("hdfs://meucluster/data/arquivo.txt")
~~~

Scala
~~~shell
val rddArquivo = sc.textFile("hdfs://meucluster/data/arquivo.txt")
~~~

Java
~~~shell
JavaRDD<String> rddArquivo = sc.textFile("hdfs://meucluster/data/arquivo.txt");
~~~

### 2 - Revisão

2.1 - Criar conta no Databricks Community: https://community.cloud.databricks.com 

2.2 - Criando SparkContext (SparkStreaming)
~~~Python
#cmd1
from pyspark.sql import SparkSession
spark_session = SparkSession.builder.enableHiveSupport().getOrCreate()
# Duas maneiras de acessar o contexto do spark a partir da sessão do spark
spark_context = spark_session._sc
spark_context = spark_session.sparkContext
~~~

2.3 - Exibindo
~~~Python
#cmd2
spark_context
#SparkContext
#Spark_UI
#Version
#    v3.0.1
#Master
#    local(8)
#AppName
#    Databricks Shell
~~~

2.4 - Teste de Streaming
~~~Python
from pyspark.streaming import StreamingContext

ssc = StreamingContext(sc, 1)
lines = ssc.socketTextStream('localhost', 9999)
counts = lines.flatMap(lambda line: line.split(" ")).map(lambda word: (word, 1)).reduceByKey(lambda a, b: a+b)
counts.pprint()
ssc.start
ssc.awaitTermination()

~~~

### Material
[Slide da aula](https://drive.google.com/file/d/1a38ZM9QxjOCtWuLz6q-hdxhWS3eaeFvD/view?usp=sharing)
