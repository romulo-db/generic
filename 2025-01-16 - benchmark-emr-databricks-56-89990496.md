
# Benchmark EMR x Databricks Classic x Databricks Serverless

## Escopo dos testes


Comparação de performance e custos entre EMR Job Cluster, Databricks Classic e Databricks Serverless.
Seleção de 3 tabelas representativas baseadas em distribuição de tamanho em zona Cleaned do datalake, frequência de atualização e importância do dado.

Teste anterior a ser utilizado como referência:
- https://docs.google.com/presentation/d/1w2MzOa8xNd8tTC8Px2Dmurtmaf2itI1zZOc8swhq9qU/

Notebook Databricks com código base de benchmark:
- https://picpay-principal.cloud.databricks.com/editor/notebooks/3050423313483307 (2024-11-18)
- https://picpay-principal.cloud.databricks.com/editor/notebooks/541018804030307 (2024-12-09)

#### Tabelas selecionadas
- Pequena: mysql/credit_cards_registration/credit_card_status (125 KB)
- Média: mysql/operations/operation_aud (21 MB)
- Grande: sqlserver/jd_npc_dda/tbjdnpcrcb_pag_pagador_sit (9.46 GB)

#### Características do processamento
- Fluxo de extração e limpeza de dados (bancos de dados -> raw -> cleaned)
- Serão incluídos apenas tabelas com carga Full
- Não haverão joins ou processamento complexo de dados

#### Serviços testados
Todos os clusters utilizados em cada ambiente serão de tipo, tamanho e características idênticas às do cluster original já utilizado nas execuções em produção.
- EMR Job Cluster (conta aws `picpay-bi-prd`)
- Databricks Classic (ambiente `picpay-lab`)
- Databricks Serverless (ambiente `picpay-lab`)

#### Ambiente de Execução
- Necessidade de replicação de permissões do EMR
- Configuração de secrets e credenciais

## Métricas de Avaliação

#### Custos
- EMR
	- Separação entre EMR e EC2
	- Separação entre inicialização e execução
- Databricks Classic
	- Separação entre DBUs e EC2
	- Separação entre inicialização e execução
- Databricks Serverless
	- Separação entre DBUs e EC2
	- Separação entre inicialização e execução

#### Performance
- Tempo de inicialização do cluster
- Tempo de processamento

## Progresso e desafios

#### :white_check_mark: Desenvolvimento do código
- Importações das bibliotecas do TrustMe
- Adaptação das bibliotecas para execução local
- Desenvolvimento de código funcional para os três ambientes

#### :white_check_mark: Problema de configuração de ambiente
- Necessidade de espelhar permissões do EMR nos testes do Databricks
- **Solução**: Atribuição da mesma role utilizada nos clusters EMR no Databricks Classic e Serverless

#### :white_check_mark: Problema de compatibilidade de bibliotecas
- Conflitos de versão do Pydantic com imagem Serverless
- **Solução**: Upgrade da imagem do Databricks Serverless para Client version 2 (Python 3.11, DBR 15.4.2)

#### :white_check_mark: Problema de compatibilidade de drivers JDBC
- Compatibilidade com drivers JDBC customizados da Databricks
- **Solução**: Customização da chamada `spark.read.format(...).options(...)` do TrustMe para suportar drivers do Databricks

#### :white_check_mark: Problema de gestão de Secrets
- Jobs no Databricks não conseguem acessar secrets da conta AWS `picpay-bi-prd`
- **Solução temporária:**
	- **Databricks Classic**: Upload manual da 12 secrets necessárias à conta `picpay-data`.
	- **Databricks Serverless**: Necessidade de implementar service credentials.
- **Solução**: Modificar roles nas contas `picpay-bi-pr` e `picpay-data`, criar service credential no Databricks `picpay-lab`

#### :x: Problema de configuração de redes
- Jobs no Databricks não conseguem acessar os bancos RDS por timeout de socket
- **Solução**:
	- Abrir GMUDs para realizar mudanças de redes:
		- Peering entre VPCs de cada banco de dados para a VPC da conta `picpay-data`

## Próximos Passos
- Resolver problemas pendentes:
	- Definir implementação da solução de secrets para serviços Databricks Classic e Serverless
	- Definir configuração de redes para acesso aos bancos
- Executar testes de benchmark em período noturno adequado para cada tabela
- Coletar métricas e sumarizar resultados

## Timeline

**2024-12-05:**
Reunião com o Romulo (Databricks) a ser marcada

**2024-12-09:**
Reunião com o Romulo realizada e detalhes/critérios de benchmark alinhados
Documento: https://docs.google.com/document/d/1GxmnRC6UJbcr-xQ4rbfTspnKBKghJIl7gC9e0JPORwo