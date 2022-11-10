# java-scale-data-architecture

## Processamento em lote aws
- utilizar s3 por questões de custo e escalabilidade (mais barato que o Efs)
- no processamento dos dados, temos:
  - spark em clusters EMR, onde existem 2 tipos:
    - transitório: executa uma tarefa e é excluido
    - persistente: fica no ar e executa tempos em tempos, com base em uma trigger pelo cronjob 
  - glue, serviço sem servidor que permite scripts scala ou python.
- cont Profiling the source data
