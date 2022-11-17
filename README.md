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
  - como resultado, enviar para outro tópic
- utilizamos sua DSL para escrever tal procedimento.
