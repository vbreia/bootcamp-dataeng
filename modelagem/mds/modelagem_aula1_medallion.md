# Medallion Architecture (Bronze/Silver/Gold)

_Aula 1 de 7 · Arquitetura de camadas — Modelagem de Dados_

> O padrão que organiza um Data Lake em estágios de confiabilidade crescente

---

Medallion Architecture é um padrão de organização de dados dentro de um Data Lake em três camadas progressivas de qualidade e confiabilidade, popularizado pela comunidade Databricks/Spark, mas aplicável em qualquer stack — incluindo Azure Data Lake com Airflow, como no seu pipeline-etransparente.

### As três camadas, em detalhe

**Bronze** — dado bruto, exatamente como chegou da fonte (API, scraping, upload manual). Sem nenhuma transformação, sem limpeza, sem validação de schema. Serve como registro histórico auditável e ponto de reprocessamento: se uma regra de negócio mudar ou um bug for encontrado na Silver, você reprocessa a partir da Bronze sem precisar re-extrair da fonte original.

**Silver** — dado limpo, validado e padronizado. Tipos corrigidos, duplicatas removidas, nulos tratados, nomes de coluna padronizados. Ainda granular (uma linha por evento/registro), mas já confiável o suficiente para ser consultado por outras equipes técnicas.

**Gold** — dado agregado e modelado especificamente para consumo: dashboards, relatórios, modelos de Machine Learning. Geralmente já estruturado como Star Schema (fatos e dimensões), otimizado para leitura analítica.

```sql
storage/
  bronze/
    ongs_raw_2026_01.json      -- exatamente como veio do scraping
  silver/
    ongs_clean.parquet         -- tipado, sem duplicatas, validado
  gold/
    score_transparencia.parquet -- agregado, pronto para o dashboard
```

### Por que não processar tudo direto na camada final?

> 💡 Essa é a pergunta mais comum em entrevista sobre o tema. A resposta: separar em camadas permite reprocessamento sem re-extrair da fonte, isolamento de falhas (um bug na transformação não corrompe o dado bruto), e auditoria histórica de exatamente o que foi recebido em cada momento.

### Idempotência entre camadas

Um pipeline bem desenhado entre camadas é idempotente: rodar o processo Bronze→Silver duas vezes para o mesmo período não deve duplicar dados nem gerar resultado diferente. Isso geralmente é resolvido com estratégias de _upsert_ (inserir ou atualizar) em vez de _append_ puro, ou particionando por data de processamento e sobrescrevendo a partição inteira a cada execução.

### Onde a Medallion se conecta com Orquestração

Cada camada tipicamente corresponde a uma ou mais tasks dentro de uma DAG do Airflow — uma task de extração escreve na Bronze, uma task de limpeza lê da Bronze e escreve na Silver, e assim por diante. Isso naturalmente cria pontos de checkpoint: se a task de limpeza falhar, você não perde o dado bruto já extraído.

### Medallion não é exclusivo de nenhuma cloud ou ferramenta

Apesar de populaizada pelo ecossistema Databricks, a arquitetura em si é um padrão conceitual — pode ser implementada com Azure Data Lake + Airflow + Pandas (como no seu caso), com AWS S3 + Glue, ou com qualquer outra combinação de storage e processamento.

## Recapitulando

- Bronze = dado bruto, sem transformação; Silver = limpo e validado; Gold = agregado para consumo
- Separar em camadas permite reprocessamento sem re-extrair da fonte e isola falhas
- Idempotência entre camadas evita duplicação ao reprocessar o mesmo período
- Cada camada geralmente corresponde a uma ou mais tasks numa DAG de orquestração
- Medallion é um padrão conceitual, não exclusivo de nenhuma cloud ou ferramenta específica

## Aprofundar no NotebookLM

**Assunto do notebook:** Medallion Architecture: as camadas Bronze, Silver e Gold, e o raciocínio de design por trás da separação.

**Perguntas-guia para o Audio Overview:**

- Por que separar dados em Bronze/Silver/Gold em vez de processar tudo direto na camada final?
- Como a Medallion Architecture se conecta com a orquestração de pipelines no Airflow?
- O que significa idempotência entre camadas, e como isso é implementado na prática?
- Compare a Medallion Architecture com a alternativa mais simples de "uma camada só" — quais problemas reais ela evita?

**Fonte complementar sugerida:** Cruzar com o material oficial da Databricks sobre Medallion Architecture (databricks.com/glossary/medallion-architecture).

---

**Teste rapido:** Qual é o principal argumento para manter a camada Bronze mesmo depois de já ter a Silver processada?

- [ ] Bronze é mais rápida de consultar
- [x] Bronze permite reprocessamento sem re-extrair da fonte original, servindo como histórico auditável
- [ ] Bronze ocupa menos espaço que a Silver
- [ ] Bronze é obrigatória por lei em dados sensíveis

> **Explicacao:** A Bronze funciona como um seguro: se uma regra de transformação mudar ou um bug for encontrado, você reprocessa a partir dela sem precisar re-extrair (que pode ser caro, como em scraping) da fonte original.
