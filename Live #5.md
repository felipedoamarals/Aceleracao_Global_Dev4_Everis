# Live #5 - Explorando o poder do NoSQL com Apache Cassandra e Apache Hbase

**[NoSQL](https://pt.wikipedia.org/wiki/NoSQL)** é um termo genérico que representa os bancos de dados não relacionais. <br>
**[Apache Cassandra](https://pt.wikipedia.org/wiki/Apache_Cassandra)** é um projeto de sistema de banco de dados distribuído altamente escalável de segunda geração, que reúne a arquitetura do DynamoDB, da Amazon Web Services e modelo de dados baseado no BigTable, do Google. <br>
**[Apache Hbase](https://pt.wikipedia.org/wiki/HBase)** é um banco de dados distribuído open-source orientado a coluna(Column Family ou Wide Column), modelado a partir do Google BigTable e escrito em Java. <br>

Profº [**Valdir Sevaios**](https://www.linkedin.com/in/valdir-novo-sevaios-junior-8190a096/) <br>

### 1 - Live Demo

### 1.1 - HBase
~~~shell
#Acessando
$ hbase shell

#Listando tabelas
list

#Caso esteja no safe mode. Executar no shell. Ele entra no safe mode quando os serviços do hadoop não foram parados antes de desligar a VM.
$ sudo -u hdfs hadoop dfsadmin -safemode leave

#Verificar o que está corrompido no HDFS. Check Yarn. Executar no shell
$ sudo -u hdfs hadoop fsck / | egrep -v '^\.+$' | grep -v eplica

#Caso exiba algum arquivo corrompido será necessário apagar ou recuperar.
$ sudo -u hdfs hdfs dfs -rm <caminho do arquivo>

#Criando uma tabela
create 'funcionario', 'pessoais', 'profissionais'

#Inserindo dados na tabela
put 'funcionario', '1', 'pessoais:nome', 'Maria'

#Visualizando os registros
scan 'funcionario'

#Inserindo dados na tabela II
put 'funcionario', '1', 'pessoais:cidade', 'Sao Paulo'

#Inserindo dados na tabela III
put 'funcionario', '2', 'profssionais:empresa', 'Everis'

#Alterando dados na tabela
##Desabilitando tabela
disable 'funcionario'

##Alterando tabela
alter 'funcionario', NAME=>'hobby', VERSIONS=>5

##Habilitando tabela
enable 'funcionario'

#Inserindo dados na tabela IV
put 'funcionario', '1', 'hobby:nome', 'Ler livros'
put 'funcionario', '1', 'hobby:nome', 'Pescar'

#Visualizando versionamento da tabela
scan 'funcionario', {VERSIONS=>3}

#Contar as rows keys
count 'funcionario'

#Deletar registros da tabela. Ele deleta a última versão do registro
delete 'funcionario', '1', 'hobby:nome'

#Criando Time To Live. O Exemplo abaixo o registro ficará disponível por 20 segundos
create 'ttl_exemplo', {'NAME'=>'cf', 'TTL' => 20}

#Inserindo registro de exemplo para o Time To Live
put 'ttl_exemplo', 'linha123', 'cf:desc', 'TTL Exemplo'
~~~

### 1.2 - Cassandra
~~~shell
#Acessando
cqlsh

#Ajuda sobre os comandos
help
help alter #Exemplo

#Criação do keyspace (schema, namespace)
CREATE KEYSPACE empresa
WITH replication = {'class':'SimpleStrategy', 'replication_factor' : 3};

#Criação de tabela
create table empresa.funcionario
  (
    empregadoid int primary key,
    empregadonome text,
    empregadocargo text,
  );
    
#Criação de índice secundário para consulta
CREATE INDEX rempresacargo ON empresa.funcionario (empregadocargo);

#Inserindo dados no keyspace
INSERT INTO
"empresa"."funcionario" (empregadoid, empregadonome, empregadocargo)
VALUES (1, 'Felipe', 'Engenheiro de Dados'); 

#Visualizando keyspace
SELECT * FROM "empresa"."funcionario";
~~~

### 2 - Atividades
1. Criar uma tabela que representa lista de Cidades e que permite armazenar até 5 versões na Column Famility com os seguintes campos:
~~~shell
Código da cidade como rowKey
Column Family=info
        Nome da Cidade
        Data de Fundação
        
Column Family=responsaveis
        Nome Prefeito
        Data de Posse do Prefeito
        Nome Vice prefeito
Column Family=estatisticas
        Data da última Eleição
        Quantidade de moradores
        Quantidade de eleitores
        Ano de fundação

#Criando Tabela
create 'cidade', 'info', 'responsaveis', 'estatisticas'

#Versionamento
alter 'cidade', NAME='info', VERSIONS =>5
alter 'cidade', NAME='responsaveis', VERSIONS =>5
alter 'cidade', NAME='estatisticas', VERSIONS =>5
~~~
2. Inserir 10 cidades na tabela criada de cidades.
~~~shell
#Exemplo utilizado
put 'cidade', '10', 'info:N_Cid', 'Alagoas'
put 'cidade', '10', 'info:Dt_fund', '2000-10-10'
put 'cidade', '10', 'responsaveis:N_Pref', 'Joao'
put 'cidade', '10', 'responsaveis:Dt_Pos', '2000-10-10'
put 'cidade', '10', 'responsaveis:N_Vice', 'Pedro'
put 'cidade', '10', 'estatisticas:Dt_ultel', '2000-10-10'
put 'cidade', '10', 'estatisticas:Qt_Mora', '993939'
put 'cidade', '10', 'estatisticas:Qt_el', '949351'
put 'cidade', '10', 'estatisticas:Ano_fund', '2000-10-10'
~~~
   
3. Realizar uma contagem de linhas na tabela.
~~~shell
count 'cidade'
#Resultado:
#hbase(main):127:0> count 'cidade'
#10 row(s) in 0.0200 seconds
#
#=> 10

~~~
   
4. Consultar só o código e nome da cidade.
~~~shell
get 'cidade', '1', 'info:N_Cid'
#Resultado
#hbase(main):133:0> get 'cidade', '1', 'info:N_Cid'
#COLUMN                         CELL                                                                                   
# info:N_Cid                    timestamp=1611793163883, value=Florianopolis
~~~
   
5. Escolha uma cidade, consulte os dados dessa cidade em específico antes do próximo passo.
~~~shell
get 'cidade', '1'
#COLUMN                         CELL                                                                                   
# estatisticas:Ano_fund         timestamp=1611793471968, value=2000-10-10                                              
# estatisticas:Dt_ultel         timestamp=1611793469699, value=2000-10-10                                              
# estatisticas:Qt_Mora          timestamp=1611793469730, value=993939                                                  
# estatisticas:Qt_el            timestamp=1611793469750, value=949351                                                  
# info:Dt_fund                  timestamp=1611793170666, value=2000-10-10                                              
# info:N_Cid                    timestamp=1611793163883, value=Florianopolis                                           
# responsaveis:Dt_Pos           timestamp=1611793182596, value=2000-10-10                                              
# responsaveis:N_Pref           timestamp=1611793176437, value=Joao                                                    
# responsaveis:N_Vice           timestamp=1611793188547, value=Pedro                                                   
#9 row(s) in 0.0070 seconds
~~~
   
6. Altere para a cidade escolhida os dados de Prefeito, Vice Prefeito e nova data de Posse.
~~~shell
put 'cidade', '1', 'responsaveis:N_Pref', 'Cezar'
put 'cidade', '1', 'responsaveis:N_Vice', 'Augusto'
~~~
   
7. Consulte os dados da cidade alterada.
~~~shell
get 'cidade', '1'
#COLUMN                         CELL                                                                                   
# estatisticas:Ano_fund         timestamp=1611793471968, value=2000-10-10                                              
# estatisticas:Dt_ultel         timestamp=1611793469699, value=2000-10-10                                              
# estatisticas:Qt_Mora          timestamp=1611793469730, value=993939                                                  
# estatisticas:Qt_el            timestamp=1611793469750, value=949351                                                  
# info:Dt_fund                  timestamp=1611793170666, value=2000-10-10                                              
# info:N_Cid                    timestamp=1611793163883, value=Florianopolis                                           
# responsaveis:Dt_Pos           timestamp=1611793182596, value=2000-10-10                                              
# responsaveis:N_Pref           timestamp=1611795251657, value=Cezar                                                   
# responsaveis:N_Vice           timestamp=1611795236842, value=Augusto                                                 
#9 row(s) in 0.0140 seconds
~~~
   
8. Consulte todas as versões dos dados da cidade alterada.
~~~shell
get 'cidade', '1', {COLUMNS=>['responsaveis'],VERSIONS=>5}
#hbase(main):203:0> get 'cidade', '1', {COLUMNS=>['responsaveis'],VERSIONS=>5}
#COLUMN                        CELL                                                                                
# responsaveis:Dt_Pos          timestamp=1611793182596, value=2000-10-10
# responsaveis:N_Pref           timestamp=1611795251657, value=Cezar                                                   
# responsaveis:N_Vice           timestamp=1611795236842, value=Augusto                                           
# responsaveis:N_Pref          timestamp=1611796161455, value=Davi                                                 
# responsaveis:N_Vice          timestamp=1611795236842, value=Augusto                                              
#5 row(s) in 0.0080 seconds
~~~

9. Exclua as três cidades com menor quantidade de habitantes e quantidade de eleitores.
~~~shell

~~~
   
10. Liste todas as cidades novamente.
~~~shell
scan 'cidade'
~~~
    
11. Adicione na ColumnFamily “estatísticas”, duas novas colunas de “quantidade de partidos políticos” e “Valor em Reais à partidos” para as 2 cidades mais populosas cadastradas.
~~~shell
put 'cidade', '1', 'responsaveis:Qt_Part', '39'
put 'cidade', '1', 'responsaveis:Valor_Part', '34556'
~~~
    
12. Liste novamente todas as cidades.
~~~shell
scan 'cidade'
~~~

### 3 - Operações massivas
3.1 Exemplo
~~~shell
#1. Confirmar que o arquivo employees.csv esteja em /home/everis/employee.csv
#2. Criar a tabela employees no Hbase com a column Familty: employee_data
create 'employees', 'employee_data'
#3. Criar uma pasta no HDFS pelo shell do Linux
hadoop fs -mkdir /test
#4. Copia os arquivos exportados para o HDFS pelo shell do Linux
hadoop fs -copyFromLocal /home/everis/employees.csv /test/employees.csv
#5. Executar a importação no shell do Linux
hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.separator=';' -Dimporttsv.columns=HBASE_ROW_KEY,employee_data:birth_date,employee_data:first_name,employee_data:last_name,employee_data:gender,employee_data:hire_date employees/test/employees.csv
~~~

3.2 Atividades

### 4 - Integrações NoSQL com ambiente Hadoop
~~~shell
#1. Vamos criar um novo schema no Hive.
CREATE DATABASE tabelas_hbases;

#2. Criar a tabela employee no Hive.
CREATE EXTERNAL TABLE tabelas_hbases.employees (
    emp_no INT,
    birth_date string,
    first_name string,
    last_name string,
    gender string,
    hire_date string)
    STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
    WITH SERDEPROPERTIES("hbase.columns.mapping"=":key, employee_data:birth_date, employee_data:first_name, employee_data:last_name,employee_data:gender,employee_data:hire_date")
    TBLPROPERTIES("hbase.table.name"="employees", "hbase.mapred.output.outputtable"="employees");
~~~

4.1 Atvidades

### Comandos Gerais
~~~shell
#Acionar o Hbase
$ hbse shell

#Visualizar detalhes sobre o sistemas
status #Variações: status'simple' | status 'summary' | status 'detailed'

#Exibir versão do HBase
version

#Exibir comandos que se referenciam a uma tabela
table_help

##Comandos utilizandos para operar as tabelas no HBase
create # Cria uma tabela.
list # Lista todas as tabelas no HBase independente do namespace.
disable # Desabilita uma tabela.
is_disabled # Checa se uma tabela está desabilitada.
enable # Habilita uma tabela.
is_enabled # Checa se uma tabela está habilitada.
describe # Exibe informações de definição de uma tabela.
alter # Realiza alterações em uma tabela.
exists # Verifica se uma tabela existe.
drop # Exclui um tabela do HBase.
drop_all # Exclue todas as que se aplicam a um padrão de nomes via regra de Regex.

#Criar uma namespace no HBase
create_namespace '<namespace>'{PROPRIEDADES}

#Criar tabela no HBase
create '[<namespace>]:<nome tabela>','<nome da column family>' {PROPRIEDADES}

#Visualizar todas as tabelas presenntes e criadas no HBase
list

#Exibe informações sobre as column families presente na tabela e outras informações HBase
describe

#Desabilita a tabela para delete ou exclusão no HBase
disable '<nome da tabela>'

#Desabilitar as tabelas que atendem dentro do critério Regex
disable_all '<prefixo da tabela>.*'

#Deletar tabela
drop '<nome da tabela>'

#Deletar todas as tabelas dentro do Regex
drop_all'<expressão regular>'

#Verificar se a tabela está habilitada ou não
is_enabled'<nome da tabela>'

#Alterar propriedades de tabelas ou acrescentar column family
alter '<nome da tabela>', NAME=><column familyname>, VERSIONS => 5

#Exibe o status das alterações realizadas de uma tabela
alter_status '<nome da tabela>'

##Comandos de manipulação
put # Insere/Atualiza um valor em uma determinada célula de uma específica linha de uma tabela.
get # Consulta o conteúdo de uma linha ou célula em uma tabela
# delete # Exclui um valor de uma célula em uma tabela.
deleteall # Exclui todas as células de uma linha em específico.
scan # Varre toda a tabela retornando as dados contidos.
count # Conta e retorna o número de linhas em uma tabela.
truncate # Desabilita, exclui e recria uma tabela em específico.
~~~

### Material
[Máquina Virtual utilizada](https://hermes.digitalinnovation.one/files/acceleration/Everis_BigData-v3.ova) <br>
[Slide da aula](https://drive.google.com/file/d/1nfzVpbwwB8d99RY8BEvjaQJ-vYx6o56X/view) 