# Explicação das Decisões Arquiteturais

Abaixo estão detalhadas as decisões técnicas encontradas no código e suas respectivas justificativas, ideais para o README.md de um repositório.

## 1. Camada Raw (Bronze) em Parquet com Snappy
Decisão: Os dados originais em CSV (com metadados nas primeiras linhas e encoding latin1) foram lidos, convertidos para um DataFrame Spark e salvos no formato Parquet com compressão Snappy.

### Justificativa:

Parquet: É um formato de armazenamento colunar, muito mais performático para leitura analítica do que o CSV.

Snappy: Oferece um excelente equilíbrio entre taxa de compressão e velocidade de descompressão, sendo ideal para a camada de entrada (Raw/Bronze) de um Data Lake.

Preservação de Metadados: O código trata a extração de metadados do cabeçalho (UF, Estação, Latitude), transformando-os em uma coluna JSON, preservando a informação original sem quebrar a estrutura tabular.

## 2. Processamento Distribuído com Spark (PySpark)
Decisão: Utilização do Apache Spark para realizar tipagem de dados (casting), tratamento de nulos e criação de novas colunas (como DataHora).

### Justificativa:

Escalabilidade: O Spark permite processamento distribuído e escalável, crucial para lidar com volumes crescentes de dados.

Data Cleaning: Foi essencial para corrigir problemas comuns de arquivos brasileiros (e.g., trocar vírgula por ponto em decimais) e unificar colunas de data e hora em um timestamp único (DataHora), facilitando análises temporais futuras.

## 3. Camada Curated (Silver) com Delta Lake
Decisão: A tabela final aula.default.clima foi salva no formato Delta Lake.

### Justificativa:

Transacionalidade (ACID): O Delta Lake adiciona uma camada de transacionalidade (ACID) sobre o Data Lake (Parquet).

DML: Isso é essencial para permitir operações de UPDATE e DELETE (Data Manipulation Language) demonstradas no notebook, algo que não seria possível se os dados estivessem apenas em arquivos Parquet puros.

## 4. Particionamento Físico por UF
Decisão: A escrita da tabela final utilizou .partitionBy("UF").

### Justificativa:

Performance de Consulta: O particionamento melhora drasticamente a performance de consultas que filtram por estado (ex: WHERE UF = 'DF').

Otimização de Leitura: Ao separar fisicamente os arquivos em pastas por UF, o motor de busca lê apenas os dados necessários (partition pruning), ignorando o restante e reduzindo a latência da consulta.

## 5. Uso de Time Travel (Viagem no Tempo)
Decisão: O código executa um DESCRIBE HISTORY e posteriormente um RESTORE TABLE.

### Justificativa:

Versionamento Nativo: Esta é uma funcionalidade exclusiva do Delta Lake que permite o versionamento dos dados.

Recuperação de Desastres: No exemplo, após deletar os dados do Distrito Federal ('DF'), a arquitetura permitiu "voltar no tempo" e restaurar a tabela para uma versão anterior (versão 1), recuperando os dados excluídos sem a necessidade de backups externos complexos.
