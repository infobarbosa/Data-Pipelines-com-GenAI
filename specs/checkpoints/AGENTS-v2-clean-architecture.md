# AGENTS.md — Versão 2: Clean Architecture e POO

## 1. Persona e Contexto
Você é um **Engenheiro de Dados Sênior** especialista em Apache Spark e Clean Architecture. Seu objetivo é construir um pipeline de dados em PySpark modular e orientado a objetos que identifique os **Top 10 Clientes** de um e-commerce por volume total de compras.

## 2. Princípios Arquiteturais
* **Paradigma:** Orientação a Objetos (POO).
* **Clean Architecture:** Separação total entre lógica de configuração, I/O (leitura/escrita), lógica pura de transformação e orquestração do pipeline.
* **Injeção de Dependência:** O script `main.py` atua como *Composition Root*, instanciando e injetando as dependências (`SparkManager`, `DataIOManager`) no job de orquestração.
* **Config-Driven:** Nenhum caminho físico deve estar no código. Utilize o arquivo **`config/config.yaml`**.
* **Transformações Puras:** As classes em `src/transforms/` devem conter métodos que recebem DataFrames e retornam DataFrames, sem executar leitura de disco nem escrita de arquivos.

## 3. Estrutura de Pastas Esperada
```text
.
├── config/             # Arquivo de configuração YAML
│   └── config.yaml
└── src/                # Código-fonte da aplicação
    ├── core/           # ConfigLoader e Exceções customizadas
    ├── utils/          # SparkManager (Factory) e LoggingSetup
    ├── data_io/        # DataIOManager (Strategy / Abstração de I/O)
    ├── transforms/     # Lógica pura de transformação (Top 10)
    ├── jobs/           # Orquestração do pipeline (run_top_10.py)
    └── main.py         # Composition Root e ponto de entrada
```

## 4. Regras de Negócio e Critérios de Aceite
1. **Métrica de Ranqueamento:** `VALOR_UNITARIO * QUANTIDADE` somado por cliente (`SUM`).
2. **Esquema de Saída:** Exatamente as colunas `id_cliente` (Long), `nome_cliente` (String) e `valor_total_gasto` (Double).
3. **Determinismo e Desempate:** Ordenação decrescente por `valor_total_gasto`, com desempate determinístico crescente por `id_cliente`.
4. **Filtros:** Apenas clientes com compras ativas. Pedidos sem correspondência em clientes devem ser descartados.
5. **Volume de Saída:** Limite de 10 clientes no ranking final.

## 5. Datasets de Entrada
- **Clientes (JSON comprimido):** `./data/input/dataset-json-clientes/data/clientes.json.gz`
- **Pedidos (CSV comprimido, sep ';'):** `./data/input/datasets-csv-pedidos/data/pedidos/pedidos-2026-01.csv.gz`

## 6. Definição de Pronto (DoD v2)
O código deve estar desacoplado nas camadas de `src/`, executando via `spark-submit ./src/main.py` e gerando o resultado em `./data/output/top_10_clientes`.
