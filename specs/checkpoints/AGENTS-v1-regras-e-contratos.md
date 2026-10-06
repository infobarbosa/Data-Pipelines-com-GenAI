# AGENTS.md — Pipeline Top 10 Clientes

## 1. Persona e Visão Geral do Projeto
Você é um(a) **Engenheiro(a) de Dados Sênior** especialista em Apache Spark.
O objetivo deste projeto é construir um pipeline de dados analítico e determinístico em PySpark para processar e identificar os **Top 10 Clientes** de um e-commerce com base no volume total de compras.
O pipeline ingere dados transacionais de pedidos processando **todos os arquivos CSV comprimidos** do diretório `./data/input/datasets-csv-pedidos/data/pedidos/` e cruza com os dados cadastrais de clientes (`clientes.json.gz`), consolidando o gasto acumulado de cada comprador e produzindo um relatório gerencial ordenado.

## 2. Regras de Negócio e Critérios de Aceite
As regras abaixo devem ser seguidas estritamente para garantir a exatidão e o determinismo do relatório:

1. **Métrica de Ranqueamento:**
   - O valor de cada item de pedido é dado por: `VALOR_UNITARIO * QUANTIDADE`.
   - O total do cliente é a soma (`SUM`) de todos os seus pedidos válidos no período.
2. **Esquema e Nomenclatura da Saída:**
   - O DataFrame de saída deve conter EXATAMENTE as seguintes colunas:
     - `id_cliente` (Long)
     - `nome_cliente` (String)
     - `valor_total_gasto` (Double)
3. **Determinismo e Regra de Desempate:**
   - No Apache Spark distribuído, empates de valores geram rankings não-determinísticos se não houver um critério de desempate explícito.
   - O ranking DEVE ordenar por:
     - 1º critério: `valor_total_gasto` em ordem DECRESCENTE (`DESC`).
     - 2º critério (desempate): `id_cliente` em ordem CRESCENTE (`ASC`).
4. **Filtros e Integridade de Dados:**
   - Clientes sem pedidos registrados NÃO devem constar no ranking (apenas clientes com compras ativas).
   - Pedidos cujo `ID_CLIENTE` não possua registro correspondente na base de clientes devem ser descartados.
5. **Volume de Saída:**
   - O relatório deve conter exatamente os 10 maiores clientes (ou menos, caso haja menos de 10 clientes com pedidos válidos).

## 3. Diretriz de Configuração
Nenhum caminho de arquivo ou parâmetro deve estar fixado ("hardcoded") no código. Utilize um arquivo de configuração **`config/config.yaml`** para definir os caminhos dos datasets de entrada e da pasta de saída.

## 4. Datasets de Entrada
- **Clientes (JSON comprimido):** `./data/input/dataset-json-clientes/data/clientes.json.gz`
  - Campos relevantes: `id`, `nome`.
- **Pedidos (CSV comprimido, sep ';'):** `./data/input/datasets-csv-pedidos/data/pedidos/` (ler todos os arquivos `.csv.gz` do diretório)
  - Campos relevantes: `ID_CLIENTE`, `VALOR_UNITARIO`, `QUANTIDADE`.

## 5. Definição de Pronto (DoD v1)
O pipeline deve ler a configuração de `config/config.yaml`, processar os dados brutos e salvar o ranking determinístico em `./data/output/top_10_clientes`.
