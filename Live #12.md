# Live #12 - Scala: o poder de uma linguagem multiparadigma

**[Scala](https://pt.wikipedia.org/wiki/Scala_(linguagem_de_programa%C3%A7%C3%A3o))** é uma linguagem de programação de propósito geral, diga-se multiparadigma, projetada para expressar padrões de programação comuns de uma forma concisa, elegante e type-safe. Ela incorpora recursos de linguagens orientadas a objetos e funcionais. Também é plenamente interoperável com Java. <br>

Profº [**Ivan Falcão**](https://www.linkedin.com/in/ivanpfalcao/) <br>

### Parte 1 - Conhecendo e compilando em Scala

1 - Instalando o Scala
~~~shell
#Instalar o open JDK:
sudo apt-get install openjdk-8-jdk

##Distros Linux baseadas em apt-get
sudo yum install java-1.8.0-openjdk

#Instalar Scala:
sudo apt-get install scala

##Distro Linux baseadas em yum:
wget https://downloads.lightbend.com/scala/2.11.12/scala-2.11.12.rpm
sudo rpm -i scala-2.11.12.rpm
~~~

2 - Instalando o Apache Maven
~~~shell
#apt-get:
sudo apt-get install maven

#yum:
sudo yum install maven
~~~

3 - Archetypes
~~~shell
#Gerar estruturas
mvn archetype:generate -DarchetypeGroupId=net.alchim31.maven -DarchetypeArtifactId=scala-archetype-simple -DarchetypeVersion=1.6

#Preencher os valores solicitados
~~~

4 - Iniciar o programa na IDE deseja
~~~shell
#Iniciando com VScode
#Abrir a pasta do projeto e iniciar o VScode
~/ProgramaScala$ code .

#Recomenda-se utilizar a extensão Metals no VScode
~~~

5 - Arquivo pom.xml
~~~xhtml
<!--Editar os campos recomendados-->
    <properties>
        <maven.compiler.source>1.6</maven.compiler.source>
        <maven.compiler.target>1.6</maven.compiler.target>
        <encoding>UTF-8</encoding>
        <scala.version>2.11.12</scala.version> <!---Versão 12 para compatibilidade com Metals-->
        <scala.compat.version>2.11</scala.compat.version>
    </properties>

<!--Remover a linha Make:transitive em build-->
       <arg>-make:transitive</arg>
~~~

6 - Remover pasta SAMPLE em src/test/scala/

7 - Compilando o app de exemplo do projeto
~~~shell
#Na pasta raiz da sua aplicação utilize o comando:
mvn package
~~~

8 - Executando o programa:
~~~shell
scala -classpath target/ProgramaScala-1.0.jar br.com.felipedoamaral.App
~~~

9 - Criando um uberjar/fatjar:
~~~shell
#Adicionar no pom.xml como último elemento dentro da tag plugins.
<!-- Maven Shade Plugin -->
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <configuration>
          <createDependencyReducedPom>false</createDependencyReducedPom>
          <filters>
            <filter>
              <artifact>*:*</artifact>
              <excludes>
                <exclude>META-INF/*.SF</exclude>
                <exclude>META-INF/*.DSA</exclude>
                <exclude>META-INF/*.RSA</exclude>
              </excludes>
            </filter>
          </filters>
        </configuration>
        <executions>
          <execution>
            <phase>package</phase>
            <goals>
              <goal>shade</goal>
            </goals>
            <configuration>
              <shadedArtifactAttached>true</shadedArtifactAttached>
              <shadedClassifierName>shaded</shadedClassifierName>
              <transformers>
                <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer" />
                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                  <mainClass>com.everis.evaAnalyticsETL.EvaAnalyticsETL</mainClass>
                </transformer>
              </transformers>

            </configuration>
          </execution>
        </executions>
      </plugin>

~~~

10 - Rodando o programa criado
~~~Shell
java -classpath target/ProgramaScala.jar br.com.felipedoamaral.App

#Com o plugin que instalamos o fat jar ficará:
ProgramaScala-jar-with-dependencies.jar

~~~

### Parte 2: A sintaxe do Scala

Object: armazena métodos que serão utilizados sem a necessidade de instanciá-los.
~~~Shell
package com.everis

object App {
  
  def main(args : Array[String]) {
    println( "Hello World!" )
  }

}
~~~

### Material
[Maven Repository](https://mvnrepository.com/)

[Slide da aula](https://drive.google.com/file/d/1fz8gxS6PKvEflnhtaQwlCv7qK3fhH2OV/view?usp=sharing)

[Site Scala](https://docs.scala-lang.org/?_ga=2.219152154.265858086.1612118441-215605817.1612018441)

[Site Alvin Alexander](https://alvinalexander.com/scala)

[Livro](https://www.amazon.com.br/Learning-Scala-Practical-Functional-Programming-ebook/dp/B00QW1RQ94/ref=sr_1_3?__mk_pt_BR=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=Scala+language&qid=1612117681&sr=8-3)
