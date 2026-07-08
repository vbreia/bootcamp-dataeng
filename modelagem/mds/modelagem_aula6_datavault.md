# Data Vault 2.0 — conceitos básicos

_Aula 6 de 7 · Modelagem avançada — Modelagem de Dados_

> Uma alternativa ao Star Schema para ambientes com múltiplas fontes e mudanças frequentes de escopo

---

Data Vault 2.0 é uma metodologia de modelagem de dados voltada para Data Warehouses que precisam integrar múltiplas fontes de dados heterogêneas e evoluir rapidamente sem quebrar o histórico já carregado. É um tema mais avançado — mencionar que você conhece o conceito já demonstra visão além do Star Schema tradicional.

### O problema que o Data Vault resolve

O Star Schema tradicional assume um modelo de negócio relativamente estável. Em ambientes onde novas fontes de dados aparecem com frequência, ou onde regras de negócio mudam rapidamente, alterar um Star Schema já em produção pode ser custoso e arriscado. Data Vault separa estrutura de negócio (que muda pouco) de contexto descritivo (que muda com frequência), tornando o modelo mais resiliente a mudanças.

### Os três blocos de construção

  - **Hub** — armazena apenas a lista de chaves de negócio únicas de uma entidade (por exemplo, todos os IDs de cliente que já existiram), sem nenhum atributo descritivo.

  - **Link** — representa relacionamentos entre Hubs (por exemplo, a relação entre cliente e pedido).

  - **Satellite** — armazena os atributos descritivos e o histórico de mudanças de um Hub ou Link, com data de carregamento e origem da informação.

```sql
hub_cliente:      hash_key_cliente | id_cliente_negocio | data_carga | origem
sat_cliente_info: hash_key_cliente | nome | email | data_carga | origem
hub_pedido:       hash_key_pedido | id_pedido_negocio | data_carga | origem
link_cliente_pedido: hash_key_cliente | hash_key_pedido | data_carga
```

### Vantagem central: auditoria e rastreabilidade nativas

Toda informação carregada num Satellite carrega metadados de origem e data — isso torna o modelo naturalmente auditável, o que é valioso em contextos regulatórios (bancário, saúde) onde é preciso provar exatamente de onde cada dado veio e quando mudou.

### O custo: complexidade e volume de tabelas

> ⚠️ Data Vault gera significativamente mais tabelas do que um Star Schema equivalente, e queries analíticas diretas sobre o Data Vault tendem a ser mais verbosas e exigir mais JOINs. Por isso, na prática, o Data Vault costuma servir como uma camada intermediária de integração — e uma camada Star Schema (a "camada de apresentação") é construída em cima dele especificamente para consumo por dashboards e relatórios.

### Quando vale considerar

Faz sentido em organizações grandes, com múltiplas fontes de dados legadas sendo integradas continuamente, ou em setores fortemente regulados que exigem auditoria detalhada de origem de dados. Para a maioria dos times pequenos ou pipelines com poucas fontes bem definidas, um Star Schema direto costuma ser suficiente e mais simples de manter.

## Recapitulando

- Data Vault separa estrutura de negócio (Hub/Link) de contexto descritivo (Satellite), tornando o modelo resiliente a mudanças
- Hub armazena apenas chaves de negócio únicas; Link representa relacionamentos; Satellite guarda atributos e histórico
- Vantagem central: auditoria e rastreabilidade nativas de origem e data de cada dado
- Custo: mais tabelas e queries mais verbosas — geralmente serve de camada de integração, não de consumo direto
- Faz mais sentido em organizações grandes com múltiplas fontes ou setores fortemente regulados

## Aprofundar no NotebookLM

**Assunto do notebook:** Data Vault 2.0: os conceitos de Hub, Link e Satellite, e quando essa metodologia é preferível ao Star Schema.

**Perguntas-guia para o Audio Overview:**

- Explique com um diagrama textual como Hub, Link e Satellite se relacionam para modelar clientes e pedidos.
- Por que o Data Vault é considerado mais resiliente a mudanças de escopo do que o Star Schema?
- Como a rastreabilidade nativa do Data Vault ajuda em contextos regulatórios (bancário, saúde)?
- Por que, na prática, muitas organizações constroem um Star Schema "por cima" de um Data Vault em vez de usar apenas um dos dois?

**Fonte complementar sugerida:** Buscar documentação e artigos introdutórios sobre Data Vault 2.0 (o conceito foi formalizado por Dan Linstedt) como fonte complementar externa.

---

**Teste rapido:** Qual é a principal razão para separar Hub, Link e Satellite em Data Vault, em vez de manter tudo em uma única tabela como no Star Schema?

- [ ] Para economizar espaço de armazenamento
- [x] Para separar estrutura de negócio (que muda pouco) de contexto descritivo (que muda com frequência), tornando o modelo mais resiliente a mudanças de escopo
- [ ] Porque bancos relacionais não suportam tabelas com muitas colunas
- [ ] Não há vantagem real — é apenas uma preferência estilística

> **Explicacao:** Essa separação é o que permite adicionar novas fontes de dados ou mudar atributos descritivos sem precisar redesenhar a estrutura central de relacionamento entre entidades — o principal ponto de dor que o Data Vault resolve em ambientes de integração complexos.
