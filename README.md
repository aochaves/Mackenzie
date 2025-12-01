**Explicação das Decisões Arquiteturais**
Abaixo detalho as decisões técnicas encontradas no código e suas justificativas:
1. Camada Raw (Bronze) em Parquet
Decisão: Os dados originais em CSV (que possuem metadados nas primeiras linhas e encoding latin1) foram lidos, convertidos para um DataFrame Spark e salvos no formato Parquet com compressão Snappy.
Justificativa: O formato Parquet é colunar e muito mais performático para leitura do que CSV. A compressão Snappy oferece um excelente equilíbrio entre taxa de compressão e velocidade de descompressão, ideal para a camada de entrada de um Data Lake. Além disso, o código trata a extração do cabeçalho de metadados (UF, Estação, Latitude) transformando-o em uma coluna JSON, preservando a informação original sem quebrar a estrutura tabular.
2. Processamento com Spark (PySpark)
Decisão: Utilização do Apache Spark para realizar a tipagem de dados (casting), tratamento de nulos e criação de novas colunas (como DataHora).
Justificativa: O Spark permite processamento distribuído e escalável. No notebook, ele foi usado para corrigir problemas comuns de arquivos brasileiros (como trocar vírgula por ponto em decimais) e unificar colunas de data e hora em um timestamp único, facilitando análises temporais futuras.
3. Camada Curated (Silver) em Delta Lake
Decisão: A tabela final aula.default.clima foi salva no formato Delta.
Justificativa: O Delta Lake adiciona uma camada de transacionalidade (ACID) sobre o Data Lake. Isso é essencial para permitir as operações de UPDATE e DELETE demonstradas no notebook, algo que não seria possível se os dados estivessem apenas em arquivos Parquet puros.
4. Particionamento por UF
Decisão: A escrita da tabela final utilizou .partitionBy("UF").
Justificativa: O particionamento melhora drasticamente a performance de consultas que filtram por estado (ex: WHERE UF = 'DF'). Ao separar fisicamente os arquivos em pastas por UF, o motor de busca lê apenas os dados necessários, ignorando o restante.
5. Uso de Time Travel (Viagem no Tempo)
Decisão: O código executa um DESCRIBE HISTORY e posteriormente um RESTORE TABLE.
Justificativa: Esta é uma funcionalidade exclusiva do Delta Lake que permite versionamento dos dados. No exemplo, após deletar os dados do Distrito Federal ('DF'), a arquitetura permitiu "voltar no tempo" e restaurar a tabela para uma versão anterior (versão 1), recuperando os dados excluídos sem a necessidade de backups externos complexos.
