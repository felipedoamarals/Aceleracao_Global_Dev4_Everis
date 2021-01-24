# Live #3 - Orquestrando ambientes de big data distribuídos com Zookeeper, Yarn e Sqoop

**[Zookeeper](https://www.cetax.com.br/apache-hadoop-tudo-o-que-voce-precisa-saber/)** é um serviço de coordenação distribuída para gerenciar grandes conjuntos de Clusters. <br>
**[Yarn](https://pt.wikipedia.org/wiki/Hadoop)** é uma plataforma do ecossistema Hadoop para gerenciamento de recursos responsável pelo gerenciamento dos recursos computacionais em cluster, assim como pelo agendamento dos recursos. <br>
**[Sqoop](https://www.cetax.com.br/apache-hadoop-tudo-o-que-voce-precisa-saber/)** é um projeto do ecossistema Hadoop, cuja responsabilidade é importar e exportar dados do banco de dados relacionais. <br>

### 1 - Alguns exemplos:
~~~shell
# Exemplo 1
sqoop import \
--connect jdbc:mysql://mysql.example.com/sqoop \
--username sqoop \
--password sqoop \
--table cities
--warehouse-dir /etl/input/ # Pemite especificar um diretório no HDFS como destino
--where "country = 'Brazil'" # Pa importar apenas um subconjunto de registros de uma tabela
-P ou --password-file my-sqoop-password
--as-sequencefile ou --as-avrodatafile # Para escrever o arquivo no HDFS em formato binário (Sequence ou Avro)
--compress # Comprime os blocos antes de gravar no HDFS em formato gzip por padrão
--compression-codec # Utilizar outros codecs de compressão, exemplo: org.apache.hadoop.io.compress.BZip2codec
--direct # Realiza import direto por meio das funcionalidades nativas do BD para melhorar a performance, exemplo: mysqldump ou pg_dump
--map-column-java c1=String # Especificar o tipo do campo
--num-mappers 10 # Especificar a quantidade de paralelismo para controlar o workload
--null-string '\\N'\
--null-non-string '\\N'
--incremental append ou lastmodified # Funcionalidade para incrementar os dados
			--check-column id ou last_update_date # Identifica a coluna que será verificada para incrementar novos dados
			--last-value 1 ou "2013-05-22 01:01:01" # Para especificar o último valor importado no Hadoop
			
# Exemplo 2 - Import da tabela accounts
sqoop import --table accounts \
--connect jdbc:mysql://dbhost/loudacre \
--username dbuser --password pw

# Exemplo 3 - Importa da tabela accounts utilizando um delimitador
sqoop import --table accounts \
--connect jdbc:mysql://dbhost/loudacre \
--username dbuser --password pw \
--fields-terminated-by "\t"

# Exemplo 4 - Import da tabela accounts limitando os resultados
sqoop import --table accounts \
--connect jdbc:mysql://dbhost/loudacre \
--username dbuser --password pw \
--where "state='CA'"

# Exemplo 5 - Import incremental baseado em um timestamp. Deve certificar-se de que esta coluna é atualizada quando os registros são atualizados ou adicionados
sqoop import --table invoices \
--connect jdbc:mysql://dbhost/loudacre \
--username dbuser --password pw \
--incremental lasmodified \
--check-column mod_dt \
--last-value '2015-09-30 16:00:00'

# Exemplo 6 - Import baseado no último valor de uma coluna específica
sqoop import --table invoices \
--connect jdbc:mysql://dbhost/loudacre \
--username dbuser --password pw \
--incremental append \
--check-column id \
--last-value 9878306
~~~

### 2 - Instalando o Sqoop
~~~shell
sudo yum install --assumeyes sqoop # Instalando
cd /tmp # Abre a pasta tmp
wget http://www.java2s.com/Code/JarDownload/java-json/java-json.jar.zip # Pega o arquivo zipado
unzip /tmp/java-json.jar.zip # Descompacta o arquivo
sudo mv /tmp/java-json.jar /usr/lib/sqoop/lib/ # Move o json para a lib do sqoop
sudo chown root: /usr/lib/sqoop/lib/java-json.jar # Altera o dono para o root
sqoop-version
~~~

### 3 - Criando base de dados de exemplo no Mysql
~~~shell
# Foi utilizado uma base do Pokemon. Base disponível no arquivo pokemon.sql
mysql -u root -h localhost -pEveris@2021 < pokemon.sql
~~~

### 4 - Iniciando serviços utilizados
~~~shell
sudo service hadoop-hdfs-namenode start
sudo service hadoop-hdfs-secondarynamenode start
sudo service hadoop-hdfs-datanode start
sudo service hadoop-yarn-nodemanager start
sudo service hadoop-yarn-resourcemanager start
sudo service hadoop-mapreduce-historyserver start
sudo service zookeeper-server start
~~~

### 5 - Importando base de dados Mysql para o HDFS
~~~shell
sh sqoop_import.sh
~~~

### 6 - Manipulando arquivos no HDFS
~~~shell
# Listando arquivos
hdfs dfs -ls /user/everis-bigdata/pokemon/

# Lendo um arquivo
hdfs dfs -text /user/everis-bigdata/pokemon/part-m-00000.gz | more
~~~

### 7 - Atividades
~~~shell
#1 - Todos os Pokemons lendários;
sudo -u hdfs sqoop import \ # Utilizando o usuário HDFS
--connect jdbc:mysql://localhost/trainning \ # String de conexão
--username root --password "Everis@2021" \ # Login
--direct \ # Executar no BD
--table pokemon \ # Terminando tabela
--target-dir /user/everis-bigdata/pokemon/1 \ # Onde será salvo
--where "Legendary=1" # Condição

# Resultado:
#65 Pokemons Lendários

#2 - Todos os Pokemons de apenas um tipo;
sudo -u hdfs sqoop import \
--connect jdbc:mysql://localhost/trainning \
--username root --password "Everis@2021" \
--direct \
--table pokemon \
--target-dir /user/everis-bigdata/pokemon/2 \ 
--where "(Type1 <> '' AND Type2 = '') OR (Type2 <> '' AND Type2 = '')"

#Resultado da Query
#SELECT * FROM trainning.pokemon where (Type1 <> '' AND Type2 = '') OR (Type2 <> '' AND Type2 = '');
#386 rows in set (0.01 sec)
#Resultado
#INFO mapreduce.ImportJobBase: Retrieved 386 records.

sudo -u hdfs sqoop import \
--connect jdbc:mysql://localhost/trainning \
--username root --password "Everis@2021" \
--fields-terminated-by "|" \
--split-by 1 \
--target-dir /user/everis-bigdata/pokemon/2 \
--query "SELECT * FROM pokemon WHERE \$CONDITIONS (Type1 <> '' AND Type2 = '') OR (Type2 <> '' AND Type2 = '')"

#3 - Os top 10 Pokemons mais rápidos;
sudo -u hdfs sqoop import \
--connect jdbc:mysql://localhost/trainning \
--username root --password "Everis@2021" \
--fields-terminated-by "|" \
--split-by 1 \
--target-dir /user/everis-bigdata/pokemon/3 \
--query 'SELECT * FROM pokemon WHERE $CONDITIONS ORDER BY SPEED DESC LIMIT 10' 

# Resultado da Query
#SELECT * FROM trainning.pokemon ORDER BY SPEED DESC LIMIT 10;
#+--------+---------------------+----------+--------+------+--------+---------+-------+-------+-------+------------+-----------+
#| Number | Name                | Type1    | Type2  | HP   | Attack | Defense | SpAtk | SpDef | Speed | Generation | Legendary |
#+--------+---------------------+----------+--------+------+--------+---------+-------+-------+-------+------------+-----------+
#|    432 | Deoxys Speed Forme  | Psychic  |        |   50 |     95 |      90 |    95 |    90 |   180 |          3 |         1 |
#|    316 | Ninjask             | Bug      | Flying |   61 |     90 |      45 |    50 |    50 |   160 |          3 |         0 |
#|    430 | DeoxysAttack Forme  | Psychic  |        |   50 |    180 |      20 |   180 |    20 |   150 |          3 |         1 |
#|    155 | Mega Aerodactyl     | Rock     | Flying |   80 |    135 |      85 |    70 |    95 |   150 |          1 |         0 |
#|    429 | Deoxys Normal Forme | Psychic  |        |   50 |    150 |      50 |   150 |    50 |   150 |          3 |         1 |
#|     72 | Mega Alakazam       | Psychic  |        |   55 |     50 |      65 |   175 |    95 |   150 |          1 |         0 |
#|    276 | Mega Sceptile       | Grass    | Dragon |   70 |    110 |      75 |   145 |    85 |   145 |          3 |         0 |
#|     20 | Mega Beedrill       | Bug      | Poison |   65 |    150 |      40 |    15 |    80 |   145 |          1 |         0 |
#|    679 | Accelgor            | Bug      |        |   80 |     70 |      40 |   100 |    60 |   145 |          5 |         0 |
#|    110 | Electrode           | Electric |        |   60 |     50 |      70 |    80 |    80 |   140 |          1 |         0 |
#+--------+---------------------+----------+--------+------+--------+---------+-------+-------+-------+------------+-----------+
#10 rows in set (0.00 sec)

# Resultado
#432|Deoxys Speed Forme|Psychic||50|95|90|95|90|180|3|true
#316|Ninjask|Bug|Flying|61|90|45|50|50|160|3|false
#430|DeoxysAttack Forme|Psychic||50|180|20|180|20|150|3|true
#155|Mega Aerodactyl|Rock|Flying|80|135|85|70|95|150|1|false
#429|Deoxys Normal Forme|Psychic||50|150|50|150|50|150|3|true
#72|Mega Alakazam|Psychic||55|50|65|175|95|150|1|false
#276|Mega Sceptile|Grass|Dragon|70|110|75|145|85|145|3|false
#20|Mega Beedrill|Bug|Poison|65|150|40|15|80|145|1|false
#679|Accelgor|Bug||80|70|40|100|60|145|5|false
#110|Electrode|Electric||60|50|70|80|80|140|1|false

#4 - Os top 50 Pokemons com menos HP;
sudo -u hdfs sqoop import \
--connect jdbc:mysql://localhost/trainning \
--username root --password "Everis@2021" \
--fields-terminated-by "|" \
--split-by 1 \
--target-dir /user/everis-bigdata/pokemon/4 \
--query 'SELECT * FROM pokemon WHERE $CONDITIONS ORDER BY HP ASC LIMIT 50' 

# Query
# SELECT * FROM trainning.pokemon ORDER BY HP ASC LIMIT 50; Exibi somente as 4 primeiras linha apenas
#+--------+------------+----------+---------+------+--------+---------+-------+-------+-------+------------+-----------+
#| Number | Name       | Type1    | Type2   | HP   | Attack | Defense | SpAtk | SpDef | Speed | Generation | Legendary |
#+--------+------------+----------+---------+------+--------+---------+-------+-------+-------+------------+-----------+
#|    317 | Shedinja   | Bug      | Ghost   |    1 |     90 |      45 |    30 |    30 |    40 |          3 |         0 |
#|     56 | Diglett    | Ground   |         |   10 |     55 |      25 |    35 |    45 |    95 |          1 |         0 |
#|    187 | Pichu      | Electric |         |   20 |     40 |      15 |    35 |    35 |    60 |          2 |         0 |
#|    389 | Duskull    | Ghost    |         |   20 |     40 |      90 |    30 |    90 |    25 |          3 |         0 |

#Resultado
#INFO mapreduce.ImportJobBase: Retrieved 50 records.
#317|Shedinja|Bug|Ghost|1|90|45|30|30|40|3|false
#56|Diglett|Ground||10|55|25|35|45|95|1|false
#187|Pichu|Electric||20|40|15|35|35|60|2|false
#389|Duskull|Ghost||20|40|90|30|90|25|3|false

#5 - Os top 100 Pokemon com maiores atributos
sudo -u hdfs sqoop import \
--connect jdbc:mysql://localhost/trainning \
--username root --password "Everis@2021" \
--fields-terminated-by "|" \
--split-by 1 \
--target-dir /user/everis-bigdata/pokemon/5 \
--query 'SELECT Number, Name, SUM(HP+Attack+Defense+SpAtk+SpDef+Speed) AS Total FROM pokemon WHERE $CONDITIONS GROUP BY Number ORDER BY Total DESC LIMIT 100' 

# Query | Exibi apenas as 4 primeiras linhas
# SELECT Number, Name, SUM(HP+Attack+Defense+SpAtk+SpDef+Speed) AS Total FROM trainning.pokemon GROUP BY Number ORDER BY Total DESC LIMIT 100;
#+--------+--------------------------+-------+
#| Number | Name                     | Total |
#+--------+--------------------------+-------+
#|    164 | Mega Mewtwo X            |   780 |
#|    427 | Mega Rayquaza            |   780 |
#|    165 | Mega Mewtwo Y            |   780 |
#|    425 | Primal Groudon           |   770 |

# Resultado
#164|Mega Mewtwo X|780
#427|Mega Rayquaza|780
#165|Mega Mewtwo Y|780
#425|Primal Groudon|770
~~~

### Adicionais
~~~shell
#Arquivo sh com os comandos para importar tabela para o HDFS(sqoop_import.sh)
#Arquivo utilizado para criar base de dados Pokemon(pokemon.sql)

#Conectar com o Mysql
mysql -u root -h localhost -pEveris@2021

#Contar linhas
hdfs dfs -cat /user/everis-bigdata/pokemon/1/* |wc -l
~~~

### Material
[Slide da aula](https://drive.google.com/file/d/1ZN53soEHPYiRS1hCtEN3L3lYSnuCZH9-/view)
