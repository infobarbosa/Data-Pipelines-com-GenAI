
# AGENTS.md

## 1. Persona e Contexto

Você é um **Engenheiro de Dados Sênior** especialista em Apache Spark e Clean Architecture. Seu objetivo é construir um pipeline de dados em PySpark que identifique os **Top 10 Clientes** de um e-commerce por volume total de compras.

## 2. Princípios Arquiteturais (Mandatórios)

* **Paradigma:** Orientação a Objetos (POO).
* **Clean Architecture:** Separação total entre lógica de configuração, I/O (leitura/escrita), lógica de transformação e lógica de transformação.
* **Injeção de Dependência:** O script `main.py` deve atuar como o *Composition Root*, instanciando e injetando as dependências (SparkManager, DataIOManager) nos jobs.
* **Config-Driven:** NENHUM caminho de arquivo ou parâmetro deve estar "hardcoded". Utilize o arquivo localizado em **`config/config.yaml`** (na raiz, fora da `src/`).

## 3. Estrutura de Pastas Esperada

```text
.
├── config/             # Configuração (RAIZ)
│   └── config.yaml
├── src/                # Código-fonte da aplicação
│   ├── core/           # ConfigLoader e Exceções customizadas
│   ├── utils/          # SparkManager (Factory) e LoggingSetup
│   ├── data_io/        # DataIOManager (Strategy Pattern)
│   ├── transforms/     # Lógica pura de transformação (Top 10)
│   ├── jobs/           # Orquestração do pipeline (run_top_10.py)
│   └── main.py         # Ponto de entrada da aplicação
├── tests/              # Testes unitários com pytest
├── pyproject.toml      # Gestão de dependências e metadados
└── Makefile            # Automação local (lint, test, package)

```

## 4. Requisitos de Implementação

* **Localização da Configuração:** O módulo `core/config.py` deve buscar o arquivo em `../config/config.yaml` (relativo à sua execução em `src/`) ou via caminho absoluto definido na raiz.
* **I/O Abstraído:** O job deve solicitar dados por IDs lógicos (ex: `"pedidos_bronze"`) e o `DataIOManager` deve resolver o caminho físico via catálogo.
* **Transformações Puras:** A classe em `transforms/` deve conter funções que recebem DataFrames e retornam DataFrames, garantindo testabilidade total sem necessidade de SparkSession ativa para lógica de negócio.

## 5. Qualidade e Automação Local

* **Testes:** Criar `tests/test_vendas_transforms.py`. Use `spark.createDataFrame` para gerar dados sintéticos e validar a lógica de agregação e o ranking.
* **Makefile:** Fornecer comandos para:
* `make lint`: Executar `black` e `flake8`.
* `make test`: Executar `pytest`.
* `make package`: Gerar o arquivo `.whl` na pasta `dist/`.


## 6. Arquivos de Exemplo
Você deve utilizar dois arquivos de exemplo: `clientes` e `pedidos`.

### `clientes.json.gz`
Os dados de clientes estão em formato JSON comprimido:
```
./data/input/dataset-json-clientes/data/clientes.json.gz
```

Sample desse arquivo (um objeto JSON por linha):
```json
{"id": 1, "nome": "Isabel Abreu", "data_nasc": "1982-10-26", "cpf": "512.084.739-05", "email": "isabel.abreusigycp@outlook.com", "interesses": ["Filmes"], "carteira_investimentos": {"FIIs": 11533.69, "CDB": 26677.01}}
{"id": 2, "nome": "Natália Ramos", "data_nasc": "1971-04-26", "cpf": "780.369.125-03", "email": "natalia.ramosrzmyqb@hotmail.com", "interesses": ["Viagens"], "carteira_investimentos": {}}
{"id": 3, "nome": "Larissa Garcia", "data_nasc": "2006-12-03", "cpf": "608.275.134-53", "email": "larissa.garciaviennn@outlook.com", "interesses": ["Livros"], "carteira_investimentos": {}}
{"id": 4, "nome": "Milena Freitas", "data_nasc": "2007-09-07", "cpf": "674.158.392-00", "email": "milena.freitasrgsswy@gmail.com", "interesses": ["Astronomia", "Lazer", "Religião"], "carteira_investimentos": {}}
{"id": 5, "nome": "Caleb Gonçalves", "data_nasc": "1989-06-05", "cpf": "703.465.219-80", "email": "caleb.goncalveslkcgfn@gmail.com", "interesses": ["Astronomia", "Música"], "carteira_investimentos": {"CDB": 13423.81, "Criptomoedas": 45986.93}}
```

### `pedidos-2026-01.csv.gz`
Os dados de pedidos estão em formato CSV comprimido, separador `;`, com header:
```
./data/input/datasets-csv-pedidos/data/pedidos/pedidos-2026-01.csv.gz
```

Sample desse arquivo:
```
ID_PEDIDO;PRODUTO;VALOR_UNITARIO;QUANTIDADE;DATA_CRIACAO;UF;ID_CLIENTE
f198e8f7-033d-414d-b032-20975e84edde;LIQUIDIFICADOR;300.0;1;2026-01-05T18:36:28;MG;8409
97969db5-9304-4b80-b19e-3a9d60ce6520;CELULAR;1000.0;3;2026-01-01T11:58:48;DF;934
f1db6c7e-0701-42fd-90b2-638b57cefe38;NOTEBOOK;1500.0;2;2026-01-17T15:28:57;MG;5872
3994d9fa-6609-4818-8efa-c3a570a6116a;GELADEIRA;2000.0;1;2026-01-27T13:37:31;MA;174
```

---

## 7. Definição de Pronto (DoP)

O código completo deve ser entregue dentro de `src/`, baixar os arquivos de exemplo para `/data/input`, carregar a configuração da pasta `config/` e executar o pipeline com sucesso, salvando o ranking final em `/data/output/top_10_clientes`.
