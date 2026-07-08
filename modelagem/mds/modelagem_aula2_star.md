# Star Schema — fatos e dimensões

_Aula 2 de 7 · Modelagem dimensional — Modelagem de Dados_

> O modelo mais usado em Data Warehousing — e por que ele existe

---

Star Schema é o modelo dimensional mais usado em Data Warehousing, formalizado por Ralph Kimball. A ideia central: separar **o que aconteceu** (fato) de **o contexto sobre quem/onde/quando/como** (dimensão) — permitindo consultas analíticas rápidas e intuitivas.

### Tabela Fato

Contém as métricas numéricas do negócio (valor, quantidade, duração) e chaves estrangeiras apontando para as dimensões relevantes. Tende a ter muitas linhas — uma linha por evento de negócio (uma venda, um clique, uma transação).

```sql
-- fato_vendas
id_venda | id_produto (FK) | id_cliente (FK) | id_data (FK) | valor | quantidade
```

### Tabela Dimensão

Contém atributos descritivos usados para filtrar, agrupar e dar contexto às métricas do fato. Tende a ter menos linhas, mas mais colunas descritivas.

```sql
-- dim_produto
id_produto (PK) | nome | categoria | marca | preco_lista

-- dim_data
id_data (PK) | data_completa | dia_semana | mes | trimestre | ano | eh_feriado
```

### Por que "Star" (estrela)

O nome vem do desenho visual: uma tabela fato central conectada a várias dimensões ao redor, como os raios de uma estrela. Essa estrutura simples é o que torna as queries analíticas intuitivas — você praticamente sempre faz JOIN do fato com uma ou mais dimensões:

```sql
SELECT d.mes, p.categoria, SUM(f.valor) AS total_vendido
FROM fato_vendas f
JOIN dim_data d ON d.id_data = f.id_data
JOIN dim_produto p ON p.id_produto = f.id_produto
GROUP BY d.mes, p.categoria;
```

### Dimensão Data — um caso especial e onipresente

Praticamente todo Star Schema tem uma `dim_data`, mesmo que pareça redundante com um campo de data simples. Ela existe porque permite atributos derivados (dia da semana, trimestre, feriado) sem recalculá-los toda vez na query — e esses atributos são extremamente comuns em análises de negócio ("vendas por dia da semana", "comparação com o mesmo trimestre do ano anterior").

### Granularidade — a decisão mais importante do fato

Antes de desenhar uma tabela fato, é preciso definir sua granularidade: o que representa exatamente uma linha? "Um item de um pedido" é diferente de "um pedido inteiro" — a primeira granularidade permite analisar produtos individualmente dentro de um pedido; a segunda, não. Errar a granularidade é o erro de design mais caro de corrigir depois, porque frequentemente exige reconstruir o fato do zero.

> ⚠️ Regra prática: escolha sempre a granularidade mais fina que faça sentido para o negócio. É fácil agregar de fino para grosso numa query; é difícil (ou impossível, se o detalhe já foi perdido) desagregar de grosso para fino depois.

## Recapitulando

- Fato = métricas numéricas + FKs para dimensões; Dimensão = atributos descritivos
- O desenho em "estrela" reflete o padrão de JOIN: fato central ligado a dimensões ao redor
- dim_data é quase universal — evita recalcular atributos derivados (trimestre, feriado) em toda query
- Granularidade define o que representa uma linha do fato — a decisão de design mais cara de errar
- Prefira sempre a granularidade mais fina que fizer sentido; agregar depois é fácil, desagregar não é

## Aprofundar no NotebookLM

**Assunto do notebook:** Star Schema: tabelas fato e dimensão, granularidade, e o papel da dimensão de data.

**Perguntas-guia para o Audio Overview:**

- Explique com um exemplo de e-commerce a diferença entre granularidade "por pedido" e "por item de pedido".
- Por que praticamente todo Star Schema tem uma dim_data, mesmo parecendo redundante com um campo de data simples?
- Desenhe (em texto) um Star Schema simples para um sistema de streaming de música, com fato e pelo menos 3 dimensões.
- Quais são os custos de corrigir um erro de granularidade depois que o fato já está em produção?

**Fonte complementar sugerida:** Cruzar com o material do Kimball Group sobre The Data Warehouse Toolkit (kimballgroup.com/data-warehouse-business-intelligence-resources/books/data-warehouse-dw-toolkit/).

---

**Teste rapido:** Por que a decisão de granularidade da tabela fato é considerada a mais cara de errar?

- [ ] Porque muda o nome das colunas
- [x] Porque agregar de fino para grosso é fácil, mas desagregar de grosso para fino pode ser impossível se o detalhe já foi perdido
- [ ] Porque afeta apenas a performance, nunca a análise possível
- [ ] Granularidade não afeta o design do fato

> **Explicacao:** Se a tabela fato já agrega "por pedido" e depois você precisa analisar produtos individuais dentro do pedido, esse detalhe pode já ter sido perdido — exigindo reconstruir o fato do zero a partir da fonte original, se ainda existir.
