# SELECT, WHERE, GROUP BY, ORDER BY

_Aula 1 de 9 · Fundamentos — SQL Avançado_

> A gramática básica de qualquer consulta — e os detalhes que a maioria ignora

---

Toda query SQL, por mais complexa que fique depois, nasce de quatro cláusulas: `SELECT` (o que eu quero ver), `FROM` (de onde), `WHERE` (quais linhas) e `ORDER BY` (em que ordem). Parece trivial, mas a ordem lógica de execução dessas cláusulas — que é diferente da ordem em que você as escreve — é a base para entender praticamente todo erro estranho que você vai encontrar em SQL mais complexo depois.

### A ordem real de execução

Você escreve `SELECT ... FROM ... WHERE ... GROUP BY ... HAVING ... ORDER BY`, mas o banco de dados processa nessa ordem: **FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY**. Isso explica por que você não pode usar um alias definido no SELECT dentro do WHERE — o WHERE roda antes do SELECT existir.

```sql
-- Isso NÃO funciona na maioria dos bancos:
SELECT preco * 1.1 AS preco_com_taxa
FROM produtos
WHERE preco_com_taxa > 100;  -- erro: preco_com_taxa ainda não existe aqui

-- Isso funciona:
SELECT preco * 1.1 AS preco_com_taxa
FROM produtos
WHERE preco * 1.1 > 100;
```

### WHERE vs HAVING — a confusão mais comum

`WHERE` filtra linhas _antes_ do agrupamento. `HAVING` filtra grupos _depois_ do agrupamento — por isso só faz sentido usá-lo junto de `GROUP BY`, e é o único lugar onde você pode filtrar por uma função de agregação como `COUNT()` ou `SUM()`.

```sql
SELECT regiao, COUNT(*) AS total_clientes
FROM clientes
WHERE ativo = true
GROUP BY regiao
HAVING COUNT(*) > 100;
```

> ⚠️ Um erro clássico de iniciante: tentar escrever WHERE COUNT(*) > 100. Isso sempre falha, porque no momento em que o WHERE é processado, a agregação ainda não existe.

### GROUP BY — a regra que ninguém explica direito

Toda coluna que aparece no SELECT e **não** está dentro de uma função de agregação precisa estar no GROUP BY. Bancos como PostgreSQL são rígidos com essa regra e recusam a query; o MySQL em modo não-estrito pode "deixar passar", retornando um valor arbitrário — o que costuma gerar bugs silenciosos.

```sql
-- Correto
SELECT cidade, COUNT(*) FROM clientes GROUP BY cidade;

-- Incorreto em bancos rígidos: 'nome' não está agregado nem no GROUP BY
SELECT cidade, nome, COUNT(*) FROM clientes GROUP BY cidade;
```

### ORDER BY — nulls e múltiplas colunas

Por padrão no PostgreSQL, valores `NULL` são ordenados por último em ordem ascendente — mas isso varia entre bancos, então especifique explicitamente quando o comportamento de NULL importa:

```sql
SELECT nome, data_ultimo_pedido
FROM clientes
ORDER BY data_ultimo_pedido DESC NULLS LAST;
```

Ordenar por múltiplas colunas funciona como um desempate em cascata:

```sql
SELECT * FROM funcionarios
ORDER BY departamento ASC, salario DESC;
```

### DISTINCT — remoção de duplicatas

`DISTINCT` remove linhas totalmente duplicadas. Quando aplicado a múltiplas colunas, considera a _combinação_ delas, não cada uma isoladamente.

```sql
SELECT DISTINCT cidade, estado FROM clientes;
```

### Por que isso importa em entrevista

Perguntas sobre esses fundamentos aparecem disfarçadas em cases mais complexos. Entender a ordem de execução lógica é o que permite debugar rapidamente, sem tentativa e erro.

## Recapitulando

- Ordem lógica de execução: FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
- WHERE filtra linhas antes de agrupar; HAVING filtra grupos já agregados
- Toda coluna não-agregada no SELECT precisa estar no GROUP BY
- ORDER BY com múltiplas colunas funciona como desempate em cascata
- DISTINCT remove combinações duplicadas, considerando todas as colunas selecionadas

## Aprofundar no NotebookLM

**Assunto do notebook:** Fundamentos de consultas SQL: ordem de execução lógica, WHERE vs HAVING, e as regras do GROUP BY.

**Perguntas-guia para o Audio Overview:**

- Por que a ordem em que escrevo uma query SQL é diferente da ordem em que ela é executada?
- Me dê 3 exemplos de queries com erro comum de GROUP BY e explique o que corrigir em cada uma.
- Quando devo usar HAVING em vez de WHERE? Crie um cenário de negócio para ilustrar.
- Como o comportamento de NULLS em ORDER BY muda entre PostgreSQL, MySQL e SQL Server?

**Fonte complementar sugerida:** Cruzar com a documentação oficial do PostgreSQL sobre a cláusula SELECT (postgresql.org/docs/current/sql-select.html).

---

**Teste rápido:** Por que a query "SELECT cidade, COUNT(*) as total FROM clientes WHERE total > 5 GROUP BY cidade" falha na maioria dos bancos?

- [ ] Porque falta um JOIN
- [x] Porque WHERE é processado antes do SELECT existir — o alias "total" ainda não foi calculado
- [ ] Porque COUNT(*) não pode ser usado com GROUP BY
- [ ] Porque cidade precisa estar entre aspas

> **Explicação:** A ordem lógica de execução processa WHERE antes do SELECT. Como "total" é um alias definido no SELECT, ele não existe ainda quando o WHERE é avaliado. O correto seria usar HAVING COUNT(*) > 5.
