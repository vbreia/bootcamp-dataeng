# Normalização (1FN, 2FN, 3FN)

_Aula 5 de 7 · Fundamentos de modelagem relacional — Modelagem de Dados_

> Como sistemas transacionais evitam redundância — e por que Data Warehouses fazem o oposto

---

Normalização é o processo de organizar tabelas em um banco relacional para reduzir redundância de dados, principalmente em sistemas transacionais (OLTP) como o backend de um e-commerce ou de um sistema de cadastro. As três primeiras formas normais são a base conceitual mais cobrada.

### Primeira Forma Normal (1FN)

Toda coluna deve conter um valor atômico — sem listas, arrays ou valores compostos em uma única célula.

```sql
-- VIOLA 1FN: telefones como lista numa única coluna
id_cliente | nome | telefones
1          | Ana  | "11999990000, 11988880000"

-- RESPEITA 1FN: uma linha por telefone
id_cliente | nome | telefone
1          | Ana  | 11999990000
1          | Ana  | 11988880000
```

### Segunda Forma Normal (2FN)

Além de estar em 1FN, toda coluna não-chave deve depender da **chave primária inteira** — relevante especificamente quando a chave primária é composta (mais de uma coluna).

```sql
-- VIOLA 2FN: 'nome_produto' depende só de id_produto, não da chave composta inteira
pedido_item: id_pedido, id_produto, nome_produto, quantidade
-- (chave composta: id_pedido + id_produto)

-- RESPEITA 2FN: nome_produto vai para uma tabela separada, dependendo só de id_produto
pedido_item: id_pedido, id_produto, quantidade
produto: id_produto, nome_produto
```

### Terceira Forma Normal (3FN)

Além de estar em 2FN, nenhuma coluna não-chave pode depender de _outra coluna não-chave_ — eliminando dependências transitivas.

```sql
-- VIOLA 3FN: cidade_estado depende de cidade, que não é chave
cliente: id_cliente, nome, cidade, cidade_estado

-- RESPEITA 3FN: informação de estado fica associada à cidade numa tabela própria
cliente: id_cliente, nome, id_cidade
cidade: id_cidade, nome_cidade, estado
```

### O propósito prático da normalização

Evitar anomalias de atualização: sem normalização, corrigir o nome de uma cidade exigiria atualizar múltiplas linhas de clientes que moram nela — esquecer uma linha gera inconsistência de dados. A normalização garante que cada fato seja armazenado em um único lugar.

### O ponto que costuma confundir: Data Warehouses são desnormalizados de propósito

> ⚠️ Um Star Schema (visto na Aula 2) é propositalmente desnormalizado em relação às regras acima — uma dimensão de produto guarda categoria, marca e outros atributos todos juntos, mesmo que isso repita informação. Essa é uma escolha consciente: sistemas OLTP priorizam integridade e evitar duplicação (muitas escritas pequenas); Data Warehouses priorizam performance de leitura analítica (poucas escritas, muitas leituras agregando grandes volumes) — para isso, menos JOINs é mais importante que evitar redundância.

### Como isso aparece em entrevista

Uma pergunta comum: "por que seu Data Warehouse não está na 3FN?". A resposta correta demonstra que você entende que normalização e modelagem dimensional resolvem problemas diferentes — não é um erro, é uma escolha de design apropriada ao contexto de uso.

## Recapitulando

- 1FN: valores atômicos, sem listas ou valores compostos em uma célula
- 2FN: 1FN + toda coluna não-chave depende da chave primária inteira (relevante em chaves compostas)
- 3FN: 2FN + nenhuma coluna não-chave depende de outra coluna não-chave (sem dependências transitivas)
- Normalização evita anomalias de atualização em sistemas transacionais (OLTP)
- Data Warehouses são desnormalizados de propósito — trade-off consciente para priorizar performance de leitura

## Aprofundar no NotebookLM

**Assunto do notebook:** Normalização de bancos de dados: 1FN, 2FN, 3FN, e por que Data Warehouses fazem o oposto de propósito.

**Perguntas-guia para o Audio Overview:**

- Dê um exemplo de tabela que viola cada uma das três formas normais, e mostre como corrigir cada uma.
- O que é uma "anomalia de atualização" e como a normalização evita esse problema?
- Por que um Star Schema em um Data Warehouse é considerado desnormalizado de propósito, e isso é um problema?
- Como eu explicaria essa diferença (OLTP normalizado vs DW desnormalizado) para um entrevistador de forma concisa?

**Fonte complementar sugerida:** Cruzar com material de referência acadêmica sobre Formas Normais em bancos de dados relacionais (qualquer material de banco de dados de graduação em Ciência da Computação é uma boa fonte complementar aqui).

---

**Teste rapido:** Por que um Star Schema em um Data Warehouse costuma violar a 3ª Forma Normal de propósito?

- [ ] Porque foi mal desenhado por erro
- [x] Porque desnormalizar as dimensões reduz a quantidade de JOINs necessários, priorizando performance de leitura analítica sobre economia de espaço
- [ ] Porque bancos analíticos não suportam chaves estrangeiras
- [ ] A 3FN não se aplica a nenhum tipo de banco de dados

> **Explicacao:** É uma escolha de design consciente: sistemas OLTP normalizam para evitar anomalias de atualização em muitas escritas pequenas; Data Warehouses desnormalizam para reduzir JOINs em consultas analíticas de grande volume, onde a leitura é a operação dominante.
