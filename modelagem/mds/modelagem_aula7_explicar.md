# Explicar escolhas de modelagem em entrevista

_Aula 7 de 7 · Comunicação técnica — Modelagem de Dados_

> De nada adianta saber modelar se você não consegue justificar a escolha para quem está avaliando

---

Em processos seletivos de nível jr a pleno, raramente pedem para você desenhar um modelo do zero sob pressão de tempo — mas é muito comum pedirem para você **explicar e justificar** escolhas de modelagem já feitas em um projeto seu, ou reagir a um modelo hipotético proposto pelo entrevistador. Comunicar bem essas escolhas é uma habilidade tão importante quanto saber fazer a modelagem em si.

### O roteiro de resposta que funciona

Um padrão eficaz de resposta para "por que você modelou assim?" segue três passos: **(1)** qual problema essa escolha resolve, **(2)** por que essa opção específica e não uma alternativa óbvia, **(3)** qual trade-off você aceitou conscientemente.

> 💡 Exemplo aplicado ao seu pipeline-etransparente: "Usei Medallion Architecture porque preciso reprocessar sem re-extrair dos sites das ONGs — scraping é custoso e às vezes as páginas mudam de estrutura. A camada Bronze me dá esse seguro, mesmo custando espaço extra de storage, que no meu caso é barato o suficiente para não ser um problema real."

### Antecipando a pergunta "e se fosse diferente?"

Entrevistadores frequentemente seguem sua explicação com uma variação: "e se o volume de dados fosse 100x maior, você modelaria diferente?" ou "e se precisasse de dado em tempo real?". Preparar-se para essas variações — mesmo sem uma resposta perfeita — demonstra que você entende que modelagem é sempre uma resposta a um contexto específico, não uma verdade universal.

### Erros comuns ao explicar modelagem em entrevista

  - **Falar só em jargão sem aterrar no problema** — dizer "usei Star Schema" sem explicar por quê soa como repetição de termo, não como entendimento.

  - **Não admitir trade-offs** — toda escolha de modelagem tem um custo. Fingir que sua escolha não tem desvantagem nenhuma é um sinal de alerta para quem está avaliando.

  - **Não conectar com o negócio** — a melhor resposta técnica ainda conecta a decisão a um impacto real (custo, velocidade de consulta, facilidade de manutenção), não só à ferramenta em si.

### Analogias ajudam — mas precisam ser precisas

Para explicar Medallion Architecture a um entrevistador não-técnico ou a um stakeholder de negócio: "é como organizar uma cozinha profissional em três estações — recebimento (ingredientes brutos, do jeito que chegam), preparo (ingredientes limpos e cortados) e finalização (o prato pronto). Cada estação existe separada porque, se algo der errado no preparo, você não quer ter que comprar tudo de novo — só refazer aquela etapa."

### Praticando antes da entrevista

Um exercício útil: escolha 2-3 decisões de modelagem reais dos seus próprios projetos e escreva (ou grave em áudio) a explicação de 30-45 segundos para cada uma, seguindo o roteiro problema → escolha → trade-off. Isso transforma conhecimento passivo em resposta pronta, sem hesitação no momento da entrevista.

## Recapitulando

- Roteiro de resposta: qual problema resolve → por que essa opção e não outra → qual trade-off foi aceito
- Prepare-se para variações tipo "e se o volume fosse 100x maior?" — modelagem é sempre resposta a um contexto
- Nunca finja que uma escolha de modelagem não tem trade-off — isso é sinal de alerta para quem avalia
- Conecte a explicação técnica a um impacto de negócio real (custo, velocidade, manutenção)
- Pratique explicações de 30-45 segundos para 2-3 decisões reais dos seus próprios projetos antes da entrevista

## Aprofundar no NotebookLM

**Assunto do notebook:** Como comunicar e justificar decisões de modelagem de dados em entrevistas técnicas.

**Perguntas-guia para o Audio Overview:**

- Monte 3 perguntas de entrevista prováveis sobre modelagem de dados e um roteiro de resposta para cada.
- Como eu explico a escolha de Medallion Architecture para um entrevistador técnico vs. para um stakeholder de negócio, de forma diferente mas igualmente precisa?
- Quais são os sinais de alerta que um entrevistador percebe quando alguém não entende de verdade sua própria escolha de modelagem?
- Pratique comigo: eu descrevo um cenário de negócio, e você me ajuda a formular a resposta ideal de modelagem com trade-offs explícitos.

**Fonte complementar sugerida:** Sem fonte externa formal necessária para esta aula — o conteúdo é sobre comunicação técnica aplicada aos conceitos das aulas anteriores deste módulo.

---

**Teste rapido:** Qual é o principal problema em responder "usei Star Schema porque é o padrão de mercado" numa entrevista?

- [ ] Star Schema não é realmente um padrão de mercado
- [x] A resposta não demonstra entendimento real do trade-off ou do problema específico que a escolha resolveu
- [ ] Star Schema está incorreto e nunca deveria ser usado
- [ ] Entrevistadores nunca aceitam essa resposta, independente do contexto

> **Explicacao:** Repetir um termo sem explicar o raciocínio por trás da escolha — qual problema resolve, por que essa opção e não outra, qual trade-off foi aceito — não demonstra que você realmente entende a decisão, apenas que você a conhece de nome.
