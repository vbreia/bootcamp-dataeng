# Índices e performance

_Aula 7 de 9 · Performance — SQL Avançado_

> Como um índice funciona por dentro — e por que ele não é gratuito

---

Um índice é uma estrutura auxiliar que permite localizar linhas sem varrer a tabela inteira — como o índice remissivo de um livro.

### Como um índice B-Tree funciona

```sql
CREATE INDEX idx_pedidos_data ON pedidos(data_pedido);
SELECT * FROM pedidos WHERE data_pedido BETWEEN '2026-01-01' AND '2026-01-31';
```

### O trade-off que todo índice representa

> ⚠️ Todo índice acelera leitura mas encarece escrita (INSERT/UPDATE/DELETE precisam atualizar o índice também). A pergunta certa: "essa coluna é filtrada com frequência suficiente para justificar isso?"

### Índice composto — ordem importa

```sql
CREATE INDEX idx_composto ON pedidos(cliente_id, data_pedido);

-- Usa eficientemente:
SELECT * FROM pedidos WHERE cliente_id = 42;
SELECT * FROM pedidos WHERE cliente_id = 42 AND data_pedido > '2026-01-01';

-- NÃO usa eficientemente:
SELECT * FROM pedidos WHERE data_pedido > '2026-01-01';
```

> 💡 Coloque primeiro a coluna mais usada em filtros de igualdade, por último a mais usada em intervalos ou ordenação.

### Índice único — integridade

```sql
CREATE UNIQUE INDEX idx_email_unico ON usuarios(email);
```

### Índices parciais

```sql
CREATE INDEX idx_pedidos_pendentes ON pedidos(cliente_id)
WHERE status = 'pendente';
```

### Quando NÃO indexar

  - Colunas de baixa cardinalidade (poucos valores distintos)

  - Tabelas pequenas o suficiente para caber em memória

  - Colunas raramente usadas em WHERE, JOIN ou ORDER BY

## Recapitulando

- Índices B-Tree permitem busca logarítmica em vez de varredura linear
- Todo índice tem custo de escrita — decisão sempre é trade-off leitura vs escrita
- Em índices compostos, a ordem das colunas determina quais queries se beneficiam
- Índice único garante integridade além de performance
- Índices parciais cobrem só o subconjunto relevante, mais leves de manter

## Aprofundar no NotebookLM

**Assunto do notebook:** Índices em bancos relacionais: B-Tree, trade-off leitura/escrita, compostos e parciais.

**Perguntas-guia para o Audio Overview:**

- Explique com analogia como uma B-Tree acelera buscas.
- Por que "criar índice em tudo" é má prática?
- Como a ordem das colunas em índice composto afeta quais queries se beneficiam?
- Quando um índice parcial é preferível a um completo?

**Fonte complementar sugerida:** Cruzar com a documentação oficial do PostgreSQL sobre Índices (postgresql.org/docs/current/indexes.html).

---

**Teste rápido:** Você tem um índice composto (cliente_id, status). Qual query NÃO se beneficia eficientemente dele?

- [ ] WHERE cliente_id = 42
- [ ] WHERE cliente_id = 42 AND status = "pago"
- [x] WHERE status = "pago"
- [ ] WHERE cliente_id = 42 ORDER BY status

> **Explicação:** Um índice composto é ordenado primeiro por cliente_id. Buscar só por status não aproveita essa ordenação.
