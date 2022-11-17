# java-scale-data-architecture

## Processamento em lote aws
- utilizar s3 por questões de custo e escalabilidade (mais barato que o Efs)
- no processamento dos dados, temos:
  - spark em clusters EMR, onde existem 2 tipos:
    - transitório: executa uma tarefa e é excluido
    - persistente: fica no ar e executa tempos em tempos, com base em uma trigger pelo cronjob 
  - glue, serviço sem servidor que permite scripts scala ou python.
- cont Profiling the source data

## Processamento em tempo real
- quando precisamos dar uma resposta rápida ao nosso cliente e o fluxo de dados e muito grande, sugere-se uso do kafka para stream de dados.
- podemos ter no fluxo dados sujos e faz necessária a quebra entre topic de entrada(source), processamento e sáida (sink)
  - entrada: limpa os dados
  - processamento: executa a lógica necessária
  - saída: resultado do processamento
- para subir um server kafka local, seguir o procedimento abaixo:
````
subir zookeeper (nova versão não precisa mais dele)
bin/zookeeper-server-start.sh config/zookeeper.properties

subir o kafka
bin/kafka-server-start.sh config/server.properties

criar os tópicos
bin/kafka-topics.sh --create --topic landingTopic1 --bootstrap-server localhost:9092
bin/kafka-topics.sh --create --topic richedTopic1 --bootstrap-server localhost:9092

````
### KStream
- uma biblioteca kafka, destinada para processamento em tempo real
- atráves dela podemos criar uma pipeline de processo, como:
  - ouvir um tipic
  - processar a mensagem, aplicando regras de negócio, chamando uma api externa ou efetuando operações no banco de dados.
  - como resultado, enviar para outro tópic ou salvar em uma base de dados via plugin
- utilizamos sua DSL para escrever tal procedimento.

### Kafka connect
- podemos configurar plugins via kafka connect
  - por exemplo: salvar o resultado de um processamento no banco de dados
- pode-se efetuar uma configuração via cluster ou standalone
- o connector kafka possui 3 componentes:
  - conector: faz interface kafka com fontes externas, ele cuida de qualquer protocolo externo que essas fontes de dados e coletores precisam para se comunicar com o kafka.
  - conversor: utilizados para serializar e deserializar eventos
  - transformador: é uma propriedade opcional, utilizada para transformar levemente os dados para que estejam no formato correto para o destino.
- exemplo:
````
bin/connect-standalone.sh config/connect-standalone.properties connect-riskcalc-mongodb-sink.properties
````
- abaixo um exemplo da configuração, quando cair uma mensagem no topic enrichedtopic1, a mesma será salva no dynamodb.
````
name=mongo-sink
topics=enrichedTopic1
connector.class=com.mongodb.kafka.connect.MongoSinkConnector
tasks.max=1
# converter configs
key.converter=org.apache.kafka.connect.storage.StringConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
key.converter.schemas.enable=false
value.converter.schemas.enable=false
# Specific global MongoDB Sink Connector configuration
connection.uri=mongodb://root:root@localhost:27017
database=CRRD
collection=newloanrequest
max.num.retries=1
retries.defer.timeout=5000
document.id.strategy=com.mongodb.kafka.connect.sink.processor.id.strategy.PartialValueStrategy
document.id.strategy.partial.value.projection.list=id
document.id.strategy.partial.value.projection.type=AllowList
writemodel.strategy=com.mongodb.kafka.connect.sink.writemodel.strategy.ReplaceOneBusinessKeyStrategy

````
