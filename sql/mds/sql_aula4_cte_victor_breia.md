# CTEs (WITH ... AS)

_Aula 4 de 9 · Organização de queries — SQL Avançado_

> Como estruturar queries complexas de forma legível — e resolver o filtro de window functions

---

Uma Common Table Expression, escrita com `WITH nome AS (...)`, cria um resultado nomeado e temporário que existe apenas durante a execução daquela query. É uma forma de dar nome e organizar uma subquery, tornando queries complexas muito mais legíveis.

### A motivação principal: legibilidade

```sql
WITH totais_por_cliente AS (
  SELECT c.nome, SUM(p.valor) AS total
  FROM clientes c JOIN pedidos p ON p.cliente_id = c.id
  GROUP BY c.nome
),
media_geral AS (
  SELECT AVG(total) AS media FROM totais_por_cliente
)
SELECT t.nome, t.total
FROM totais_por_cliente t, media_geral m
WHERE t.total > m.media;
```

Você pode rodar cada CTE isoladamente (trocando o SELECT final por `SELECT * FROM totais_por_cliente`) para verificar se aquele passo está correto.

### Múltiplas CTEs encadeadas

```sql
WITH vendas_diarias AS (
  SELECT data, SUM(valor) AS total_dia
  FROM pedidos GROUP BY data
),
media_movel AS (
  SELECT data, total_dia,
    AVG(total_dia) OVER (ORDER BY data ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS media_7dias
  FROM vendas_diarias
)
SELECT * FROM media_movel ORDER BY data;
```

### Resolvendo o filtro de window functions

```sql
WITH ranking AS (
  SELECT produto, vendas,
    DENSE_RANK() OVER (ORDER BY vendas DESC) AS posicao
  FROM produtos
)
SELECT * FROM ranking WHERE posicao  💡 CTEs recursivas raramente são cobradas em entrevistas jr/pleno, mas mencionar que você sabe que existem já demonstra profundidade.

### CTE vs Subquery vs View

  - **Subquery** — uma única transformação simples, aninhada.

  - **CTE** — múltiplos passos lógicos, ou reuso dentro da mesma query.

  - **View** — mesma lógica reutilizada entre queries diferentes.

## Recapitulando

- CTEs nomeiam resultados temporários, tornando queries complexas mais legíveis e debugáveis
- Múltiplas CTEs podem ser encadeadas, cada uma referenciando as anteriores
- CTE + WHERE na query externa é o padrão para filtrar resultados de window functions
- WITH RECURSIVE permite percorrer hierarquias (organogramas, árvores de categoria)
- Use View em vez de CTE quando a mesma lógica precisa ser reaproveitada entre queries diferentes

## Aprofundar no NotebookLM

**Assunto do notebook:** Common Table Expressions (CTEs) em SQL: sintaxe WITH, encadeamento e recursão.

**Perguntas-guia para o Audio Overview:**

- Reescreva uma subquery aninhada complexa como uma sequência de CTEs.
- Quando faz mais sentido usar uma CTE em vez de uma View?
- Explique passo a passo como uma CTE recursiva "anda" por uma hierarquia.
- Dê um exemplo de pipeline de 3 CTEs encadeadas para um cálculo de cohort de clientes.

**Fonte complementar sugerida:** Cruzar com a documentação oficial do PostgreSQL sobre WITH Queries (postgresql.org/docs/current/queries-with.html).

---

**Teste rápido:** Por que CTEs tornam mais fácil debugar uma query complexa?

- [ ] Porque elas rodam mais rápido que subqueries sempre
- [x] Porque você pode rodar cada CTE isoladamente para validar um passo intermediário
- [ ] Porque CTEs não permitem erros de sintaxe
- [ ] Porque o banco otimiza CTEs automaticamente em paralelo

> **Explicação:** A vantagem prática vem de poder isolar cada CTE para verificar se aquele passo específico está correto antes de seguir para o próximo.
