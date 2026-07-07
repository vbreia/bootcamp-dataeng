# Subqueries correlacionadas

_Aula 5 de 9 · Consultas aninhadas — SQL Avançado_

> Quando uma query "conversa" com a linha externa — e o cuidado de performance que isso exige

---

Uma subquery correlacionada é uma consulta aninhada que referencia uma coluna da query externa. Diferente de uma subquery independente, ela é reexecutada uma vez para cada linha da query externa em muitas implementações.

### Independente vs correlacionada

```sql
-- INDEPENDENTE: roda uma única vez
SELECT nome, salario FROM funcionarios
WHERE salario > (SELECT AVG(salario) FROM funcionarios);

-- CORRELACIONADA: referencia 'f' da query externa
SELECT nome, salario FROM funcionarios f
WHERE salario > (
  SELECT AVG(salario) FROM funcionarios
  WHERE departamento = f.departamento
);
```

### EXISTS — a forma mais eficiente de checar existência

```sql
SELECT c.nome
FROM clientes c
WHERE EXISTS (
  SELECT 1 FROM pedidos p
  WHERE p.cliente_id = c.id AND p.valor > 1000
);
```

> 💡 Convenção usar SELECT 1 dentro do EXISTS — o valor selecionado é irrelevante, só a existência da linha importa.

### NOT EXISTS — o padrão para "nunca aconteceu"

```sql
SELECT c.nome
FROM clientes c
WHERE NOT EXISTS (
  SELECT 1 FROM pedidos p WHERE p.cliente_id = c.id
);
```

> ⚠️ Resolve o mesmo problema do "LEFT JOIN + WHERE IS NULL", geralmente com performance melhor em tabelas grandes.

### IN vs EXISTS

```sql
-- IN: mais legível para casos simples
SELECT nome FROM clientes
WHERE id IN (SELECT cliente_id FROM pedidos WHERE valor > 1000);

-- EXISTS: geralmente mais eficiente em tabelas grandes
SELECT nome FROM clientes c
WHERE EXISTS (
  SELECT 1 FROM pedidos p WHERE p.cliente_id = c.id AND p.valor > 1000
);
```

### Subquery escalar no SELECT

```sql
SELECT c.nome,
  (SELECT COUNT(*) FROM pedidos p WHERE p.cliente_id = c.id) AS total_pedidos
FROM clientes c;
```

> ⚠️ Para tabelas grandes, uma window function ou JOIN com GROUP BY costuma performar melhor que subquery escalar linha a linha.

### Quando preferir JOIN

Se você só precisa filtrar, EXISTS/NOT EXISTS é suficiente. Se precisa trazer dados de outra tabela, use JOIN.

## Recapitulando

- Subquery correlacionada referencia a linha da query externa — recalculada para cada linha
- EXISTS retorna apenas verdadeiro/falso e para na primeira correspondência
- NOT EXISTS é o padrão preferido para "nunca aconteceu"
- IN é mais legível para listas pequenas; EXISTS tende a performar melhor em tabelas grandes
- Prefira JOIN quando precisar trazer dados; EXISTS quando só precisar filtrar

## Aprofundar no NotebookLM

**Assunto do notebook:** Subqueries correlacionadas: diferença para independentes, EXISTS/NOT EXISTS, e comparação com IN e JOIN.

**Perguntas-guia para o Audio Overview:**

- Explique como uma subquery correlacionada é reexecutada linha a linha.
- Quando EXISTS tem performance melhor que IN?
- Compare LEFT JOIN + IS NULL, NOT EXISTS e NOT IN para achar "clientes sem pedido" — riscos com NULL?
- Crie 2 exercícios de subquery correlacionada com dificuldade crescente.

**Fonte complementar sugerida:** Cruzar com a documentação oficial do PostgreSQL sobre Subquery Expressions (postgresql.org/docs/current/functions-subquery.html).

---

**Teste rápido:** Por que "SELECT 1" é usado por convenção dentro de um EXISTS?

- [ ] Porque SELECT * causa erro de sintaxe dentro de EXISTS
- [x] Porque o valor selecionado é irrelevante — só a existência da linha importa
- [ ] Porque SELECT 1 é mais rápido para o banco processar
- [ ] Porque EXISTS só aceita valores numéricos

> **Explicação:** EXISTS avalia apenas se a subquery retorna alguma linha — o conteúdo selecionado nunca é usado. SELECT 1 é convenção de legibilidade.
