# Agregação condicional (FILTER, CASE WHEN)

_Aula 8 de 9 · Análise de dados — SQL Avançado_

> Como calcular múltiplas métricas condicionais em uma única passada pela tabela

---

É comum precisar de métricas segmentadas na mesma query. A agregação condicional resolve isso combinando agregação com lógica condicional, processando a tabela uma única vez.

### CASE WHEN dentro de agregação

```sql
SELECT
  regiao,
  COUNT(CASE WHEN status = 'ativo' THEN 1 END) AS clientes_ativos,
  COUNT(CASE WHEN status = 'inativo' THEN 1 END) AS clientes_inativos,
  SUM(CASE WHEN status = 'ativo' THEN valor_mensal ELSE 0 END) AS receita_ativos
FROM clientes
GROUP BY regiao;
```

> ⚠️ CASE WHEN sem ELSE retorna NULL, que COUNT() ignora. Usar ELSE 0 no COUNT contaria TODAS as linhas — para SUM o ELSE 0 é necessário, para COUNT deve ser omitido.

### FILTER — a alternativa mais moderna

```sql
SELECT
  regiao,
  COUNT(*) FILTER (WHERE status = 'ativo') AS clientes_ativos,
  COUNT(*) FILTER (WHERE status = 'inativo') AS clientes_inativos,
  SUM(valor_mensal) FILTER (WHERE status = 'ativo') AS receita_ativos
FROM clientes
GROUP BY regiao;
```

### Pivotização manual

```sql
SELECT
  produto,
  SUM(CASE WHEN mes = 1 THEN vendas ELSE 0 END) AS jan,
  SUM(CASE WHEN mes = 2 THEN vendas ELSE 0 END) AS fev,
  SUM(CASE WHEN mes = 3 THEN vendas ELSE 0 END) AS mar
FROM vendas_mensais
GROUP BY produto;
```

### Combinando com AVG para percentuais

```sql
SELECT
  regiao,
  AVG(CASE WHEN status = 'ativo' THEN 1.0 ELSE 0.0 END) * 100 AS pct_ativos
FROM clientes
GROUP BY regiao;
```

> 💡 Aqui o ELSE é necessário porque usamos AVG, não COUNT.

### Por que preferir a múltiplas queries

Rodar tudo em uma passada é mais eficiente do que N queries separadas, especialmente em tabelas grandes.

## Recapitulando

- CASE WHEN sem ELSE retorna NULL, que COUNT() ignora
- Para SUM o ELSE 0 é necessário; para COUNT deve ser omitido
- FILTER (WHERE ...) é a alternativa mais legível e moderna
- Agregação condicional é a base para pivotizar dados manualmente
- Rodar múltiplas métricas em uma query é mais eficiente que múltiplas queries

## Aprofundar no NotebookLM

**Assunto do notebook:** Agregação condicional: CASE WHEN dentro de agregações e a cláusula FILTER.

**Perguntas-guia para o Audio Overview:**

- Por que COUNT(CASE WHEN x THEN 1 END) funciona mas com ELSE 0 não conta corretamente?
- Reescreva 3 exemplos de CASE WHEN usando FILTER equivalente.
- Como agregação condicional simula PIVOT em bancos sem essa função nativa?
- Crie um exercício de e-commerce (pedidos por status e método de pagamento).

**Fonte complementar sugerida:** Cruzar com a documentação oficial do PostgreSQL sobre Aggregate Expressions e FILTER.

---

**Teste rápido:** Por que SUM(...ELSE 0 END) precisa do ELSE, mas o COUNT equivalente não deveria ter ELSE?

- [ ] Não há diferença real
- [x] SUM precisa de valor neutro (0); COUNT ignora NULL, que já funciona como neutro
- [ ] ELSE é obrigatório sempre
- [ ] SUM não aceita CASE WHEN sem ELSE

> **Explicação:** SUM precisa somar algo em cada linha (0 é neutro para adição). COUNT conta não-nulos — NULL do CASE sem ELSE já exclui corretamente.
