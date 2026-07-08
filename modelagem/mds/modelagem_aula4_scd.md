# SCD Tipo 1, 2 e 3

_Aula 4 de 7 · Histórico de dimensões — Modelagem de Dados_

> O que fazer quando um atributo de dimensão muda com o tempo

---

Slowly Changing Dimensions (SCD) resolve um problema muito comum: o que fazer quando um atributo de uma dimensão muda — por exemplo, um cliente muda de cidade, ou um produto muda de categoria. A resposta depende de uma pergunta de negócio: você precisa saber o valor histórico (como era antes) ou só o valor atual importa?

### SCD Tipo 1 — sobrescrever

A abordagem mais simples: quando o atributo muda, você simplesmente atualiza o valor na linha existente, perdendo o valor anterior.

```sql
-- Antes: id_cliente=1, nome=Ana, cidade=Rio
UPDATE dim_cliente SET cidade = 'São Paulo' WHERE id_cliente = 1;
-- Depois: id_cliente=1, nome=Ana, cidade=São Paulo (Rio não existe mais em lugar nenhum)
```

> ⚠️ Toda análise histórica que já foi feita usando esse atributo passa a "mentir" retroativamente — se você somar vendas por cidade do ano passado, as vendas antigas da Ana em "Rio" agora aparecem como se sempre tivessem sido em "São Paulo".

### SCD Tipo 2 — preservar histórico completo

A abordagem mais usada quando o histórico importa: em vez de atualizar, você insere uma **nova linha** com o valor novo, e marca a linha antiga como "não mais atual", geralmente com colunas de data de início/fim e uma flag.

```sql
id_cliente | nome | cidade | data_inicio  | data_fim    | atual
1          | Ana  | Rio    | 2024-01-01   | 2025-06-01  | false
1          | Ana  | SP     | 2025-06-01   | NULL        | true
```

Agora, uma query que junta o fato de vendas com essa dimensão usando a data da venda (em vez de sempre pegar a linha "atual") reflete corretamente o histórico: vendas antigas continuam associadas a "Rio", vendas novas a "SP".

```sql
SELECT dc.cidade, SUM(f.valor)
FROM fato_vendas f
JOIN dim_cliente dc ON dc.id_cliente = f.id_cliente
  AND f.data_venda BETWEEN dc.data_inicio AND COALESCE(dc.data_fim, '9999-12-31')
GROUP BY dc.cidade;
```

### SCD Tipo 3 — manter só o valor anterior

Um meio-termo raramente usado: em vez de criar uma nova linha, você adiciona uma coluna extra para guardar apenas o valor _imediatamente anterior_, sem histórico completo de todas as mudanças.

```sql
id_cliente | nome | cidade_atual | cidade_anterior
1          | Ana  | SP           | Rio
```

Isso só funciona bem quando você sabe, de antemão, que só precisa comparar "antes vs depois" da mudança mais recente — qualquer mudança adicional sobrescreve a anterior, perdendo o histórico mais antigo.

### Escolhendo entre os três

> 💡 Pergunta prática para decidir: "se esse atributo mudar, alguém vai querer analisar dados históricos considerando o valor de na época, ou só o valor atual importa daqui pra frente?" Se a resposta é "histórico importa", SCD Tipo 2 é quase sempre a escolha certa em produção.

## Recapitulando

- SCD Tipo 1 sobrescreve o valor antigo — simples, mas perde histórico e distorce análises passadas
- SCD Tipo 2 cria uma nova linha com datas de validade, preservando o histórico completo
- SCD Tipo 3 mantém apenas o valor "anterior" — meio-termo raramente usado na prática
- Joins com dimensões SCD Tipo 2 devem usar a data do evento, não sempre a linha "atual"
- A pergunta certa para decidir o tipo: "análises históricas precisam refletir o valor de na época?"

## Aprofundar no NotebookLM

**Assunto do notebook:** Slowly Changing Dimensions (SCD): Tipo 1, 2 e 3, e como preservar histórico em dimensões.

**Perguntas-guia para o Audio Overview:**

- Explique com um exemplo numérico como SCD Tipo 1 distorce retroativamente uma análise histórica.
- Desenhe o fluxo completo de como uma nova linha SCD Tipo 2 é criada quando um atributo muda.
- Por que o JOIN entre fato e uma dimensão SCD Tipo 2 precisa considerar a data do evento, e não a flag "atual"?
- Em que cenário raro o SCD Tipo 3 seria suficiente, sem precisar do histórico completo do Tipo 2?

**Fonte complementar sugerida:** Cruzar com o material do Kimball Group sobre Slowly Changing Dimensions (kimballgroup.com).

---

**Teste rapido:** Por que um JOIN entre uma tabela fato e uma dimensão SCD Tipo 2 deve considerar a data do evento em vez de sempre usar a linha marcada como "atual"?

- [ ] Porque a linha "atual" nunca existe em SCD Tipo 2
- [x] Porque usar sempre a linha atual faria vendas antigas aparecerem associadas ao valor mais recente do atributo, distorcendo o histórico
- [ ] Porque SCD Tipo 2 não permite JOIN com tabela fato
- [ ] Porque a coluna "atual" é apenas decorativa

> **Explicacao:** O objetivo do SCD Tipo 2 é justamente preservar o valor histórico correto para cada período. Se o JOIN sempre pegasse a linha "atual", perderíamos exatamente o benefício de ter capturado o histórico — as vendas antigas apareceriam com o valor mais recente do atributo.
