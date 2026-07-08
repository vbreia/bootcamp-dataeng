# Snowflake Schema — quando usar vs Star

_Aula 3 de 7 · Variações de modelagem — Modelagem de Dados_

> A variação normalizada do Star Schema — e o trade-off que ela representa

---

Snowflake Schema é uma variação do Star Schema onde as tabelas de dimensão são normalizadas em sub-tabelas relacionadas, em vez de manter todos os atributos descritivos em uma única tabela plana.

### Star vs Snowflake — a diferença visual

```sql
-- STAR: dimensão desnormalizada (tudo em uma tabela)
dim_produto: id_produto | nome | categoria | subcategoria | marca

-- SNOWFLAKE: dimensão normalizada em sub-tabelas
dim_produto: id_produto | nome | id_categoria (FK) | id_marca (FK)
dim_categoria: id_categoria | nome_categoria | id_subcategoria (FK)
dim_subcategoria: id_subcategoria | nome_subcategoria
dim_marca: id_marca | nome_marca
```

No Snowflake, para saber a categoria de um produto, você precisa de um JOIN adicional através de `dim_categoria` — daí o nome, porque o desenho visual se ramifica como os cristais de um floco de neve, em vez de manter o formato simples de estrela.

### O trade-off central

Normalizar as dimensões reduz redundância de armazenamento (o nome de uma categoria fica escrito uma única vez, não repetido em cada produto), mas aumenta a quantidade de JOINs necessários em cada query analítica — o que geralmente piora a performance de leitura e a simplicidade das queries.

> 💡 Em Data Warehouses analíticos, onde o objetivo principal é performance de leitura para relatórios e dashboards, o Star Schema costuma ser preferido — o espaço extra de armazenamento raramente é um problema real com os custos de storage atuais, mas a complexidade de múltiplos JOINs em toda query analítica é um custo real e recorrente.

### Quando Snowflake ainda faz sentido

Snowflake pode ser justificado quando: a dimensão tem hierarquias profundas e claramente definidas que mudam com frequência independente (por exemplo, uma estrutura organizacional com muitos níveis), ou quando o volume de dados de dimensão é grande o suficiente para que a redundância do Star Schema realmente pese no armazenamento e na manutenção de consistência.

### Uma decisão que raramente é tudo-ou-nada

Na prática, muitos Data Warehouses usam um modelo híbrido: a maioria das dimensões desnormalizadas (Star), mas uma ou duas dimensões específicas com hierarquia complexa normalizadas (Snowflake) — otimizando caso a caso em vez de aplicar uma regra rígida ao esquema inteiro.

## Recapitulando

- Snowflake normaliza dimensões em sub-tabelas relacionadas; Star mantém tudo em uma tabela plana
- Trade-off: Snowflake reduz redundância de armazenamento, mas exige mais JOINs por query
- Star Schema é preferido quando performance de leitura analítica é a prioridade
- Snowflake se justifica em hierarquias profundas e voláteis, ou dimensões muito grandes
- Modelos híbridos (Star na maioria, Snowflake pontual) são comuns na prática

## Aprofundar no NotebookLM

**Assunto do notebook:** Snowflake Schema comparado ao Star Schema: normalização de dimensões e o trade-off de performance.

**Perguntas-guia para o Audio Overview:**

- Desenhe (em texto) a mesma dimensão de produto em versão Star e versão Snowflake, mostrando as tabelas resultantes.
- Por que o Star Schema costuma ser preferido em Data Warehouses analíticos, apesar da redundância?
- Em que cenário real um Snowflake Schema resolveria um problema que o Star Schema não resolveria bem?
- O que significa um "modelo híbrido" entre Star e Snowflake, e quando isso é uma boa prática?

**Fonte complementar sugerida:** Cruzar com material comparativo sobre Star Schema vs Snowflake Schema em dbt Docs (docs.getdbt.com/docs/build/models) e Kimball Group.

---

**Teste rapido:** Qual é o principal custo de usar um Snowflake Schema em vez de Star Schema?

- [ ] Ocupa mais espaço em disco
- [x] Exige mais JOINs em cada query analítica, o que geralmente piora performance de leitura
- [ ] Não permite usar chaves estrangeiras
- [ ] É incompatível com arquitetura Medallion

> **Explicacao:** A normalização das dimensões em sub-tabelas reduz redundância de armazenamento, mas exige que queries analíticas façam JOINs adicionais para reconstruir os atributos completos — o oposto do Star Schema, que já tem tudo em uma única tabela de dimensão.
