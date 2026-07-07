# JOINs (INNER, LEFT, RIGHT, FULL, CROSS)

_Aula 2 de 9 · Combinação de tabelas — SQL Avançado_

> A operação mais cobrada em testes técnicos — e como raciocinar sobre cada tipo

---

Um JOIN combina linhas de duas (ou mais) tabelas com base em uma condição de correspondência. É provavelmente a operação SQL mais testada em processos seletivos, porque avalia diretamente se você entende relacionamento entre entidades.

### INNER JOIN — a interseção

Retorna apenas as linhas que têm correspondência em **ambas** as tabelas.

```sql
SELECT c.nome, p.valor
FROM clientes c
INNER JOIN pedidos p ON p.cliente_id = c.id;
```

### LEFT JOIN — preservando o lado esquerdo

Retorna **todas** as linhas da tabela à esquerda, preenchendo com `NULL` quando não há correspondência.

```sql
SELECT c.nome, p.valor
FROM clientes c
LEFT JOIN pedidos p ON p.cliente_id = c.id;
```

> 💡 Padrão comum em entrevista: "encontre clientes que NUNCA fizeram pedido". LEFT JOIN + WHERE p.id IS NULL.

```sql
SELECT c.nome
FROM clientes c
LEFT JOIN pedidos p ON p.cliente_id = c.id
WHERE p.id IS NULL;
```

### RIGHT JOIN — o espelho

Funciona como o LEFT JOIN, mas preservando a tabela à direita. Raramente usado — qualquer RIGHT JOIN pode ser reescrito como LEFT JOIN invertendo a ordem das tabelas.

### FULL OUTER JOIN — a união completa

Retorna todas as linhas de ambas as tabelas, com `NULL` onde não há correspondência de nenhum lado. Útil para achar divergências entre duas fontes.

```sql
SELECT rh.nome, acessos.usuario
FROM funcionarios_rh rh
FULL OUTER JOIN acessos_sistema acessos ON acessos.cpf = rh.cpf;
```

### CROSS JOIN — produto cartesiano

Combina cada linha de uma tabela com **todas** as linhas da outra, sem condição de correspondência.

```sql
SELECT p.nome, t.tamanho
FROM produtos p
CROSS JOIN tamanhos_disponiveis t;
```

### Self JOIN — uma tabela consigo mesma

```sql
SELECT f.nome AS funcionario, g.nome AS gerente
FROM funcionarios f
LEFT JOIN funcionarios g ON f.gerente_id = g.id;
```

### O erro de performance mais comum

JOINs em colunas sem índice geram varreduras completas de tabela em ambos os lados. Garanta índice nas colunas usadas em `ON`.

## Recapitulando

- INNER JOIN retorna apenas matches; LEFT JOIN preserva todas as linhas da esquerda
- LEFT JOIN + WHERE ... IS NULL é o padrão para achar registros "órfãos"
- FULL OUTER JOIN é útil para comparar duas fontes de dados e achar divergências
- CROSS JOIN gera produto cartesiano — cuidado com tabelas grandes
- Self JOIN usa aliases diferentes para relacionar uma tabela com ela mesma

## Aprofundar no NotebookLM

**Assunto do notebook:** Tipos de JOIN em SQL: INNER, LEFT, RIGHT, FULL OUTER e CROSS, com foco em quando usar cada um.

**Perguntas-guia para o Audio Overview:**

- Crie 3 cenários de negócio reais onde cada tipo de JOIN seria a escolha certa.
- Por que RIGHT JOIN é raramente usado na prática?
- Explique o padrão "LEFT JOIN + WHERE IS NULL" passo a passo.
- Quais são os riscos de performance de um CROSS JOIN acidental em produção?

**Fonte complementar sugerida:** Cruzar com a documentação oficial do PostgreSQL sobre Joined Tables (postgresql.org/docs/current/queries-table-expressions.html).

---

**Teste rápido:** Qual query encontra corretamente clientes que NUNCA fizeram nenhum pedido?

- [ ] SELECT c.nome FROM clientes c INNER JOIN pedidos p ON p.cliente_id = c.id WHERE p.id IS NULL
- [x] SELECT c.nome FROM clientes c LEFT JOIN pedidos p ON p.cliente_id = c.id WHERE p.id IS NULL
- [ ] SELECT c.nome FROM clientes c, pedidos p WHERE p.cliente_id != c.id
- [ ] SELECT c.nome FROM clientes c RIGHT JOIN pedidos p ON p.cliente_id = c.id

> **Explicação:** INNER JOIN nunca produziria linhas com p.id NULL. O LEFT JOIN preserva todos os clientes, e o filtro isola os sem correspondência.
