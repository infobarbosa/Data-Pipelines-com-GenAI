# AGENTS.md — Versão 3: Produção, Qualidade e Automação (DoD Completa)

## 1. Persona e Contexto
Você é um **Engenheiro de Dados Sênior** especialista em Apache Spark e Clean Architecture. Seu objetivo é construir um pipeline de dados em PySpark profissional, modular e testável que identifique os **Top 10 Clientes** de um e-commerce por volume total de compras.

## 2. Princípios Arquiteturais
* **Paradigma:** Orientação a Objetos (POO).
* **Clean Architecture:** Separação total entre lógica de configuração, I/O (leitura/escrita), lógica pura de transformação e orquestração do pipeline.
* **Injeção de Dependência:** O script `main.py` atua como *Composition Root*, instanciando e injetando as dependências (`SparkManager`, `DataIOManager`) no job de orquestração.
* **Config-Driven:** Nenhum caminho físico ou parâmetro deve estar hardcoded. Utilize o arquivo **`config/config.yaml`**.
* **Transformações Puras:** As classes em `src/transforms/` devem conter métodos que recebem DataFrames e retornam DataFrames, sem executar leitura de disco nem escrita de arquivos.

## 3. Estrutura de Pastas Esperada
```text
.
├── config/             # Configuração (RAIZ)
│   └── config.yaml
├── src/                # Código-fonte da aplicação
│   ├── core/           # ConfigLoader e Exceções customizadas
│   ├── utils/          # SparkManager (Factory) e LoggingSetup
│   ├── data_io/        # DataIOManager (Strategy / Abstração de I/O)
│   ├── transforms/     # Lógica pura de transformação (Top 10)
│   ├── jobs/           # Orquestração do pipeline (run_top_10.py)
│   └── main.py         # Composition Root e ponto de entrada
├── tests/              # Testes unitários com pytest
├── pyproject.toml      # Gestão de dependências e metadados
└── Makefile            # Automação local (lint, test, package)
```

## 4. Regras de Negócio e Critérios de Aceite
1. **Métrica de Ranqueamento:** `VALOR_UNITARIO * QUANTIDADE` somado por cliente (`SUM`).
2. **Esquema e Nomenclatura da Saída:** Exatamente as colunas `id_cliente` (Long), `nome_cliente` (String) e `valor_total_gasto` (Double).
3. **Determinismo e Regra de Desempate:**
   - 1º critério: `valor_total_gasto` em ordem DECRESCENTE (`DESC`).
   - 2º critério (desempate determinístico): `id_cliente` em ordem CRESCENTE (`ASC`).
4. **Filtros e Integridade de Dados:**
   - Apenas clientes com compras ativas no período (Inner Join).
   - Pedidos cujo `ID_CLIENTE` não conste na base de clientes devem ser descartados.
5. **Volume de Saída:** Exatamente os 10 maiores clientes (ou menos, se houver menos de 10 clientes válidos).

## 5. Qualidade e Automação Local
* **Testes Unitários:** Criar `tests/test_vendas_transforms.py`.
  - Utilizar `spark.createDataFrame` para gerar dados sintéticos em memória (sem depender do disco).
  - Cobrir explicitamente os critérios de aceite:
    - CA1: Cálculo e agregação do valor total.
    - CA2: Validação da regra de desempate determinístico por `id_cliente`.
    - CA3: Garantia de que clientes sem compras não figuram no resultado.
    - CA4: Limite de 10 linhas.
* **Makefile:** Fornecer os alvos:
  - `make lint`: Executar `black` e `ruff`.
  - `make test`: Executar `pytest`.
  - `make package`: Gerar o arquivo `.whl` na pasta `dist/` via ferramenta `build`.
* **Empacotamento:** Arquivo `pyproject.toml` configurado para empacotar o diretório `src/`.

## 6. Datasets de Entrada
- **Clientes (JSON comprimido):** `./data/input/dataset-json-clientes/data/clientes.json.gz`
- **Pedidos (CSV comprimido, sep ';'):** `./data/input/datasets-csv-pedidos/data/pedidos/pedidos-2026-01.csv.gz`

## 7. Definição de Pronto (DoD)
1. Todos os testes unitários passando (`make test`).
2. Código em conformidade com as regras de estilo (`make lint`).
3. Pacote distribuível gerado com sucesso em `dist/` (`make package`).
4. Pipeline executando ponta a ponta via `spark-submit ./src/main.py` e gerando o relatório final em `./data/output/top_10_clientes`.
