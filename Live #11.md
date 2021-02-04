# Live #11 - Orquestrando Big Data em Ambiente de Nuvem

**[AWS](https://pt.wikipedia.org/wiki/Amazon_Web_Services)** Amazon Web Services, também conhecido como AWS, é uma plataforma de serviços de computação em nuvem, que formam uma plataforma de computação na nuvem oferecida pela Amazon.com. <br>

Profº [**Edmilson Carmo**](https://www.linkedin.com/in/edm-carmo/) <br>

### Live demo - AWS

Caso de uso: Atuaremos na detecção de anomalias do fluxo de cliques em tempo real do [Amazon Kinesis](https://aws.amazon.com/pt/blogs/big-data/real-time-clickstream-anomaly-detection-with-amazon-kinesis-analytics/)

1 - Criar um grupo de serviço no Cloud Formation usando o template Lab-DIO.json;

2 - Após a criação dos recursos, pegar em Outputs a URL para chamada da geração de dados no KNG e abrir com o usuário criado no Cognito;

3 - Criar no Kinesis uma aplicação de data analytics;

4 - Realizar conexão do Streaming de dados via Kinesis Firehose Delivey Stream e escolher a função(role) IAM disponível;

5 - No SQL Editor do Real-time analytics, utilizar o script Kinesis_Analytics_anomaly_detection.sql;

6 - Em Destination criar a Entrega:
* Destination: AWS Lambda function | Seleciona a função criada
* In-application stream: DESTINATION_SQL_STREAM

7 - Realizar testes;

8 - Parar os serviços, deletar as funções, deletar o bucket utilizado e deletar a Stack.

### Material
[Slide da aula](https://drive.google.com/file/d/1Br7ZRezzb2gIDqZZSCbfYASecEOV72JC/view?usp=sharing)
