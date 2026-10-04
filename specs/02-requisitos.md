# Requisitos — Top 10 Clientes
**Dono:** Analista de Negócios — deriva de [01-brief.md]

## Definição
"Top 10" = os 10 clientes com maior **valor total de compras** no período disponível.
`valor_total_gasto = Σ(VALOR_UNITARIO × QUANTIDADE)` de todos os pedidos válidos do cliente.

## Saída Esperada
Ranking estruturado com: `id_cliente` (Long), `nome_cliente` (String) e `valor_total_gasto` (Double).

## Critérios de Aceite (Critérios de Teste)
- **CA1 (Cálculo da Métrica):** O valor total deve corresponder com precisão à soma de `VALOR_UNITARIO * QUANTIDADE`.
- **CA2 (Determinismo e Desempate):** Em caso de empate no valor total, a ordenação DEVE ser estritamente determinística pelo `id_cliente` em ordem crescente (`ASC`).
- **CA3 (Filtro de Inativos):** Clientes sem pedidos registrados NÃO devem aparecer no ranking (mesmo que a base tenha menos de 10 clientes).
- **CA4 (Volume Limite):** O ranking deve conter exatamente 10 linhas quando houver 10 ou mais clientes com compras válidas.
