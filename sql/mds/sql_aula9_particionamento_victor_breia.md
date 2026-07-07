# Particionamento de tabelas

_Aula 9 de 9 · Escalabilidade — SQL Avançado_

> Como bancos gigantes continuam rápidos — dividir para conquistar, literalmente

---

Particionamento divide uma tabela logicamente única em partições físicas menores, de forma transparente para quem consulta, com ganhos de performance e manutenção em grande volume.

### O problema que resolve

Tabelas que crescem indefinidamente enfrentam queries lentas mesmo com índices, e manutenção (VACUUM, backup) cada vez mais demorada.

### Particionamento por Range

```sql
CREATE TABLE pedidos (
  id SERIAL, data_pedido DATE NOT NULL, valor NUMERIC
) PARTITION BY RANGE (data_pedido);

CREATE TABLE pedidos_2026_01 PARTITION OF pedidos
  FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');
```

O otimizador elimina partições fora do intervalo filtrado — **partition pruning**, o principal ganho de performance.

### Particionamento por Lista

```sql
CREATE TABLE vendas (
  id SERIAL, regiao TEXT NOT NULL, valor NUMERIC
) PARTITION BY LIST (regiao);

CREATE TABLE vendas_sudeste PARTITION OF vendas
  FOR VALUES IN ('SP', 'RJ', 'MG', 'ES');
```

### Particionamento por Hash

```sql
CREATE TABLE logs (
  id SERIAL, usuario_id INT NOT NULL, evento TEXT
) PARTITION BY HASH (usuario_id);

CREATE TABLE logs_p0 PARTITION OF logs FOR VALUES WITH (MODULUS 4, REMAINDER 0);
```

### Ganho além de performance de query

Descartar dados antigos vira `DROP TABLE` de uma partição inteira — praticamente instantâneo comparado a um DELETE em massa.

> 💡 Relevante para Medallion: a camada Bronze, que acumula dados brutos indefinidamente, é candidata natural a particionamento por data.

### Quando NÃO particionar

Adiciona complexidade operacional real. Só vale para tabelas na casa de dezenas de milhões de linhas com padrão claro de filtro.

## Recapitulando

- Particionamento divide uma tabela lógica em partições físicas menores
- Partition pruning permite ao otimizador ignorar partições irrelevantes
- Range (data) é o mais comum; List para categorias; Hash para distribuição uniforme
- Descartar dados antigos vira DROP de partição, muito mais rápido que DELETE em massa
- Só vale a complexidade em tabelas muito grandes com padrão claro de filtro

## Aprofundar no NotebookLM

**Assunto do notebook:** Particionamento de tabelas: Range, List e Hash, e o conceito de partition pruning.

**Perguntas-guia para o Audio Overview:**

- Explique partition pruning com um exemplo de query e plano de execução.
- Quando Range é preferível a Hash?
- Como particionamento por data se conecta com retenção de dados em Medallion?
- Quais os custos operacionais reais de manter uma tabela particionada?

**Fonte complementar sugerida:** Cruzar com a documentação oficial do PostgreSQL sobre Table Partitioning (postgresql.org/docs/current/ddl-partitioning.html).

---

**Teste rápido:** O que é "partition pruning"?

- [ ] Remoção automática de partições vazias
- [x] Capacidade do otimizador de ignorar partições que não podem conter linhas relevantes
- [ ] Arquivamento de partições antigas
- [ ] Compressão automática de dados por partição

> **Explicação:** Partition pruning é o otimizador determinando quais partições podem conter dados relevantes para a query, ignorando as demais.
