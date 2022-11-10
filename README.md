# java-scale-data-architecture

## Processamento em lote aws
- utilizar s3 por questões de custo e escalabilidade (mais barato que o Efs)
- no processamento dos dados, temos:
  - spark em clusters EMR
  - glue, serviço sem servidor que permite scripts scala ou python.
