# Contrato de Dados — Top 10 Clientes
**Dono:** Analista de Dados — deriva de [02-requisitos.md]

## Fontes de Dados
- **clientes:** Formato JSON comprimido (`./data/input/dataset-json-clientes/data/clientes.json.gz`) — Grão: 1 linha por cliente.
- **pedidos:** Formato CSV comprimido com separador `;` (`./data/input/datasets-csv-pedidos/data/pedidos/pedidos-2026-01.csv.gz`) — Grão: 1 linha por item de pedido.

## Schema Relevante
- **clientes:**
  - `id`: Long
  - `nome`: String
- **pedidos:**
  - `ID_CLIENTE`: Long
  - `VALOR_UNITARIO`: Double
  - `QUANTIDADE`: Long / Integer

## Regras de Junção e Qualidade
- **Junção:** `pedidos.ID_CLIENTE == clientes.id` (Inner Join).
- Pedidos com `ID_CLIENTE` inexistente na base de clientes devem ser descartados da agregação final.
- Campos numéricos nulos em `VALOR_UNITARIO` ou `QUANTIDADE` devem ser tratados adequadamente para não corromper o somatório.
