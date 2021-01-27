# Live #5 - Explorando o poder do NoSQL com Apache Cassandra e Apache Hbase

**[NoSQL](https://pt.wikipedia.org/wiki/NoSQL)** é um termo genérico que representa os bancos de dados não relacionais. <br>
**[Apache Cassandra](https://pt.wikipedia.org/wiki/Apache_Cassandra)** é um projeto de sistema de banco de dados distribuído altamente escalável de segunda geração, que reúne a arquitetura do DynamoDB, da Amazon Web Services e modelo de dados baseado no BigTable, do Google. <br>
**[Apache Hbase](https://pt.wikipedia.org/wiki/HBase)** é um banco de dados distribuído open-source orientado a coluna(Column Family ou Wide Column), modelado a partir do Google BigTable e escrito em Java. <br>

Profº [**Valdir Sevaios**](https://www.linkedin.com/in/valdir-novo-sevaios-junior-8190a096/) <br>

### 1 - Live Demo
### 1.1 - Beeline
~~~shell
~~~

### 1.2 - HBase
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
create 'funcionario', 'pessoais', 'profssionais'

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
alter 'funcionario', NAME='hobby', VERSIONS=>5

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

### 1.3 - Cassandra
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
~~~
2. Inserir 10 cidades na tabela criada de cidades.
~~~shell
~~~
   
3. Realizar uma contagem de linhas na tabela.
~~~shell
~~~
   
4. Consultar só o código e nome da cidade.
~~~shell
~~~
   
5. Escolha uma cidade, consulte os dados dessa cidade em específico antes do próximo passo.
~~~shell
~~~
   
6. Altere para a cidade escolhida os dados de Prefeito, Vice Prefeito e nova data de Posse.
~~~shell
~~~
   
7. Consulte os dados da cidade alterada.
~~~shell
~~~
   
8. Consulte todas as versões dos dados da cidade alterada.
~~~shell
~~~

9. Exclua as três cidades com menor quantidade de habitantes e quantidade de eleitores.
~~~shell
~~~
   
10. Liste todas as cidades novamente.
~~~shell
~~~
    
11. Adicione na ColumnFamily “estatísticas”, duas novas colunas de “quantidade de partidos políticos” e “Valor em Reais à partidos” para as 2 cidades mais populosas cadastradas.
~~~shell
~~~
    
12. Liste novamente todas as cidades.
~~~shell
~~~

### 3 - Operações massivas

### 4 - Integrações NoSQL com ambiente Hadoop

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