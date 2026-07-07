# Window Functions (ROW_NUMBER, RANK, LAG, LEAD)

_Aula 3 de 9 · Análise avançada — SQL Avançado_

> O recurso que separa quem sabe SQL básico de quem sabe SQL analítico

---

Segundo a documentação oficial do PostgreSQL, uma window function executa um cálculo através de um conjunto de linhas relacionadas à linha atual — sem colapsar as linhas em uma única linha de saída, diferente de uma agregação com GROUP BY.

### A anatomia da cláusula OVER

```sql
função(coluna) OVER (
  [PARTITION BY coluna_de_agrupamento]
  [ORDER BY coluna_de_ordenação]
  [frame_clause]
)
```

`PARTITION BY` divide as linhas em grupos processados independentemente. `ORDER BY` dentro do OVER define a ordem de processamento — obrigatório para `ROW_NUMBER()`, `LAG()` e `LEAD()`.

### As três funções de ranking — a pegadinha clássica

```sql
SELECT nome, departamento, salario,
  ROW_NUMBER() OVER (PARTITION BY departamento ORDER BY salario DESC) AS rn,
  RANK()       OVER (PARTITION BY departamento ORDER BY salario DESC) AS rk,
  DENSE_RANK() OVER (PARTITION BY departamento ORDER BY salario DESC) AS drk
FROM funcionarios;
```

Com salários 5000, 5000, 4000 no mesmo departamento:

  - **ROW_NUMBER()** → 1, 2, 3 (nunca empata)

  - **RANK()** → 1, 1, 3 (empata, mas _pula_ a posição 2)

  - **DENSE_RANK()** → 1, 1, 2 (empata, _não pula_)

> ⚠️ Decore com um exemplo mental fixo: "RANK pula, DENSE_RANK não pula".

### LAG e LEAD — comparando com linhas vizinhas

```sql
SELECT mes, receita,
  LAG(receita, 1) OVER (ORDER BY mes) AS receita_mes_anterior,
  receita - LAG(receita, 1) OVER (ORDER BY mes) AS variacao
FROM receita_mensal;
```

### Agregações como window functions

```sql
SELECT produto, categoria, vendas,
  SUM(vendas) OVER (PARTITION BY categoria) AS total_categoria,
  ROUND(vendas * 100.0 / SUM(vendas) OVER (PARTITION BY categoria), 2) AS pct_da_categoria
FROM vendas_produtos;
```

### Running totals — soma acumulada

```sql
SELECT data, valor,
  SUM(valor) OVER (ORDER BY data) AS saldo_acumulado
FROM transacoes;
```

### Filtrando resultados de window functions

Window functions só são permitidas em `SELECT` e `ORDER BY`. Para filtrar, use uma CTE:

```sql
WITH ranking AS (
  SELECT nome, departamento, salario,
    RANK() OVER (PARTITION BY departamento ORDER BY salario DESC) AS rk
  FROM funcionarios
)
SELECT * FROM ranking WHERE rk <= 3;
```

## Recapitulando

- Window functions enriquecem linhas sem colapsá-las — diferente de GROUP BY
- PARTITION BY agrupa; ORDER BY dentro do OVER define ordem de processamento
- RANK() pula posições em empate; DENSE_RANK() não pula; ROW_NUMBER() nunca empata
- LAG/LEAD acessam linhas vizinhas — essenciais para cálculo de variação período a período
- Para filtrar o resultado de uma window function, use CTE + WHERE na query externa

## Aprofundar no NotebookLM

**Assunto do notebook:** Window functions em SQL: PARTITION BY, funções de ranking e funções de deslocamento (LAG, LEAD).

**Perguntas-guia para o Audio Overview:**

- Explique com uma tabela de exemplo a diferença exata entre ROW_NUMBER, RANK e DENSE_RANK.
- Como LAG e LEAD podem detectar anomalias em uma série temporal de vendas?
- Por que window functions não podem ser usadas diretamente em WHERE?
- Crie um exercício: top 3 produtos mais vendidos por categoria usando window functions.

**Fonte complementar sugerida:** Cruzar com a documentação oficial do PostgreSQL — Window Functions (postgresql.org/docs/current/tutorial-window.html e functions-window.html).

---

**Teste rápido:** Em uma partição com salários [5000, 5000, 4000, 3000], qual é a saída de RANK() OVER (ORDER BY salario DESC)?

- [ ] 1, 2, 3, 4
- [ ] 1, 1, 2, 3
- [x] 1, 1, 3, 4
- [ ] 1, 2, 2, 3

> **Explicação:** RANK() empata os dois primeiros com posição 1, mas PULA a posição 2 — o próximo valor recebe posição 3.
