# EXPLAIN / Query Plan — otimização

_Aula 6 de 9 · Performance — SQL Avançado_

> Como enxergar o que o banco de dados realmente faz por trás da sua query

---

`EXPLAIN` revela o plano de execução que o otimizador escolheu: qual estratégia de varredura usar, em que ordem fazer JOINs, e o custo estimado de cada etapa.

### EXPLAIN vs EXPLAIN ANALYZE

```sql
EXPLAIN ANALYZE
SELECT * FROM pedidos WHERE cliente_id = 42;
```

> ⚠️ EXPLAIN ANALYZE executa a query de verdade. Cuidado com queries de escrita — use uma transação com ROLLBACK se só quiser analisar.

### Seq Scan vs Index Scan

**Seq Scan** lê a tabela inteira. **Index Scan** usa um índice para pular direto às linhas relevantes. Seq Scan em tabela grande onde se esperava índice é o sinal mais comum de otimização faltando.

```sql
EXPLAIN SELECT * FROM pedidos WHERE cliente_id = 42;
-- Solução se mostrar Seq Scan:
CREATE INDEX idx_pedidos_cliente ON pedidos(cliente_id);
```

### Lendo o custo estimado

Números como `cost=0.29..8.31` são unidades relativas de custo, não milissegundos — servem para comparar planos entre si.

### Estratégias de JOIN no plano

  - **Nested Loop** — eficiente quando uma tabela é pequena ou bem indexada.

  - **Hash Join** — eficiente para tabelas grandes sem índice na coluna de junção.

  - **Merge Join** — quando ambas as tabelas já estão ordenadas pela coluna de junção.

### Índice existe mas não é usado

  - Tabela pequena — overhead do índice não compensa

  - Estatísticas desatualizadas — rode `ANALYZE nome_da_tabela`

  - Função aplicada sobre a coluna no WHERE: `WHERE LOWER(email) = 'x'` não usa índice simples em `email`

> 💡 Solução: índice funcional — CREATE INDEX ON clientes (LOWER(email)).

## Recapitulando

- EXPLAIN mostra o plano estimado; EXPLAIN ANALYZE executa de verdade
- Seq Scan em tabela grande onde se esperava índice é o sinal mais comum de otimização faltando
- Custos no plano são unidades relativas, não milissegundos
- Nested Loop, Hash Join e Merge Join são escolhidos automaticamente pelo otimizador
- Índice não usado: verificar estatísticas desatualizadas ou função aplicada sobre a coluna

## Aprofundar no NotebookLM

**Assunto do notebook:** EXPLAIN e otimização de queries: leitura de planos, Seq Scan vs Index Scan, estratégias de JOIN.

**Perguntas-guia para o Audio Overview:**

- Explique como ler um plano de EXPLAIN linha por linha.
- Por que EXPLAIN ANALYZE é potencialmente perigoso em queries de escrita?
- Quais os 3 motivos mais comuns para um índice existir mas não ser usado?
- Compare Nested Loop, Hash Join e Merge Join com exemplos.

**Fonte complementar sugerida:** Cruzar com a documentação oficial do PostgreSQL sobre EXPLAIN e Índices (postgresql.org/docs/current/using-explain.html).

---

**Teste rápido:** Você criou um índice em "cliente_id", mas o EXPLAIN ainda mostra Seq Scan. Qual NÃO é uma causa provável?

- [ ] A tabela é pequena e o otimizador prefere Seq Scan
- [ ] As estatísticas da tabela estão desatualizadas
- [x] O índice foi criado com o nome errado de coluna
- [ ] Uma função está sendo aplicada sobre a coluna filtrada

> **Explicação:** Nome de coluna errado geraria erro na criação ou índice em outra coluna — não explicaria Seq Scan na coluna correta.
