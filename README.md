# Data Engineer Study System

Sistema de estudos local (100% HTML/CSS/JS estático, sem backend) para preparação de Victor Breia em vagas de Data Engineer (Estágio → Jr → Pleno). Construído para uso pessoal, com progresso salvo em `localStorage` do navegador, exportação de aulas em Markdown (para NotebookLM), e vídeos gravados localmente embutidos por aula.

## Objetivo do sistema

Cobrir 6 áreas de estudo com aulas completas por subtópico (não só por tema), cada aula funcionando como uma "apostila" individual com:
- Conteúdo teórico fundamentado (800-1200 palavras)
- Exemplos de código
- Teste de fixação (1 pergunta de múltipla escolha)
- Vídeo-aula gravado pelo usuário (feito no NotebookLM a partir do `.md` exportado)
- Seção "Aprofundar no NotebookLM" com perguntas-guia para gerar Audio Overview

## Estrutura de arquivos

```
dataengineer/
├── data-engineer-study-system.html   ← painel geral (hub)
├── modulo-01-sql.html                ← Módulo 1: SQL Avançado (9 aulas)
├── modulo-02-modelagem.html          ← Módulo 2: Modelagem de Dados (7 aulas)
├── modulo-03-airflow.html            ← Módulo 3: Apache Airflow (9 aulas)
├── modulo-04-python.html             ← Módulo 4: Python para Dados (9 aulas)
├── modulo-05-cloud.html              ← Módulo 5: Cloud Azure/AWS/GCP (9 aulas)
├── modulo-06-processos.html          ← Módulo 6: Processos Seletivos (6 aulas)
├── sql/                              ← vídeos do Módulo 1 (já preenchido)
│   ├── sql1.Lógica_de_Execução_SQL.mp4
│   ├── sql2.Masterclass_SQL_JOINs.mp4
│   ├── ... (sql3 a sql9)
│   └── mds/                          ← pasta onde os .md exportados são salvos manualmente
├── modelagem/                        ← vídeos do Módulo 2 (vazio, placeholder ativo)
├── airflow/                          ← vídeos do Módulo 3 (vazio, placeholder ativo)
├── python/                           ← vídeos do Módulo 4 (vazio, placeholder ativo)
├── cloud/                            ← vídeos do Módulo 5 (vazio, placeholder ativo)
└── processos/                        ← vídeos do Módulo 6 (vazio, placeholder ativo)
```

Todos os arquivos são independentes e autossuficientes (HTML+CSS+JS inline, sem build step). Basta abrir direto no navegador — não precisa de servidor local, exceto que **vídeos locais só carregam se o HTML for aberto via `file://` no mesmo disco/pasta relativa**, o que já é o caso aqui.

## Anatomia de um arquivo de módulo (`modulo-0X-*.html`)

Cada módulo segue a mesma estrutura interna:

### 1. Header fixo (topo da página)
Breadcrumb de volta ao hub, título do módulo, botão "Exportar módulo completo (.md)", barra de progresso geral do módulo.

### 2. Layout de duas colunas
- **Sidebar esquerda** (`#sidebar`) — sumário clicável: "Visão geral" + lista de aulas com checkbox de progresso e tag de prioridade (crítico/importante/bônus/tenho)
- **Conteúdo direito** (`#content`) — renderiza a aula ativa ou a visão geral, via JS (`currentLesson` = índice, 0 = visão geral)

### 3. Array `LESSONS` (fonte única de verdade de cada módulo)
Cada aula é um objeto JS com esta forma exata:

```js
{
  id: 'slug_curto',                    // usado em storage, DOM ids, nome de arquivo exportado
  tag: 'critical' | 'important' | 'bonus' | 'owned',
  title: 'Título da Aula',
  eyebrow: 'Aula N de TOTAL · Categoria',
  sub: 'Subtítulo de uma linha',
  video: 'pasta/arquivo.mp4',          // OPCIONAL — se ausente, mostra placeholder de vídeo
  body: `<p>...</p><h3>...</h3>...`,   // HTML da aula (não Markdown!)
  recap: ['ponto 1', 'ponto 2', ...],  // 5 bullets de recapitulação
  nblm: {
    subject: 'resumo de 1 frase do assunto',
    questions: ['pergunta 1', 'pergunta 2', 'pergunta 3', 'pergunta 4'],
    source: 'fonte oficial complementar sugerida'
  },
  quiz: {
    q: 'pergunta',
    code: 'trecho de código opcional',  // OPCIONAL
    options: ['opção A', 'opção B', 'opção C', 'opção D'],
    correct: 1,                          // índice (0-based) da resposta certa
    explanation: 'por que essa é a resposta certa'
  }
}
```

**Importante sobre o campo `body`:** é HTML puro embutido em template literal JS (crases), não Markdown. Usa tags como `<h3>`, `<p>`, `<ul><li>`, `<pre><code>`, `<strong>`, `<code>`, e divs de callout: `<div class="callout tip">`, `<div class="callout warn">`, `<div class="callout example">` (só no Módulo 6).

### 4. Sistema de vídeo por aula
Função `renderVideoBlock(lesson, lessonNum)` (definida em cada arquivo, logo antes de `renderContent()`):
- Se `lesson.video` existe → renderiza `<video controls>` apontando para `VIDEO_FOLDER + '/' + lesson.video`
- Se não existe → renderiza placeholder visual (`.video-placeholder`) informando o nome de arquivo esperado, usando `VIDEO_FOLDER` e `VIDEO_PREFIX` (constantes no topo do `<script>`, uma por módulo)

Convenção de nomenclatura de vídeo por módulo (a mesma usada pelo NotebookLM ao exportar):

| Módulo | Pasta | Prefixo | Exemplo |
|---|---|---|---|
| 01 SQL | `sql/` | `sql` | `sql1.Lógica_de_Execução_SQL.mp4` |
| 02 Modelagem | `modelagem/` | `mod` | `mod1.Titulo.mp4` |
| 03 Airflow | `airflow/` | `air` | `air1.Titulo.mp4` |
| 04 Python | `python/` | `py` | `py1.Titulo.mp4` |
| 05 Cloud | `cloud/` | `cloud` | `cloud1.Titulo.mp4` |
| 06 Processos | `processos/` | `proc` | `proc1.Titulo.mp4` |

**Se o padrão real de nomenclatura divergir disso** (como aconteceu — o usuário pode preferir manter `sql`-style completo em vez de abreviado), ajustar as constantes `VIDEO_FOLDER`/`VIDEO_PREFIX` no topo do `<script>` de cada módulo, e o campo `video:` de cada lição já existente.

### 5. Persistência (localStorage)
Cada módulo usa uma `STORAGE_KEY` própria (`de_module_sql_v1`, `de_module_modelagem_v1`, etc.), guardando:
```js
{ done: { 'lesson_id': true/false, ... }, quizResults: { 'lesson_id': true/false, ... } }
```
Isso é **local ao navegador**, não sincroniza entre dispositivos nem entre arquivos. Não depende de nenhuma API externa.

### 6. Exportação em Markdown
Cada aula tem botão "Exportar esta aula (.md)" e cada módulo tem "Exportar módulo completo (.md)". A função `htmlToMarkdown()` (regex-based, definida em cada arquivo) converte o HTML do campo `body` para Markdown limpo. **Não usa PDF/html2pdf** — isso foi removido de propósito porque gerava quebras de página ruins; Markdown é o formato de entrada esperado pelo NotebookLM.

O fluxo de trabalho do usuário é: exportar `.md` → subir no NotebookLM → gerar Audio Overview usando as perguntas-guia da seção NBLM → gravar vídeo a partir disso → salvar o `.mp4` na pasta do módulo com o nome convencionado → o placeholder vira player automaticamente.

## Anatomia do hub (`data-engineer-study-system.html`)

Estrutura mais simples que os módulos: cada tema (`TOPICS` array) tem um objeto com `id`, `subtopics` (checklist simples, sem aula completa), `defaultSources` (links), e `questions` (teste geral do tema, não por subtópico). O conteúdo de aula "resumida" de cada tema fica no objeto `LESSONS` (chave = `topic.id`), como um blob HTML simples (sem a estrutura rica dos módulos).

Cada card de tema no hub tem um botão condicional que linka para o módulo completo correspondente:
```js
${topic.id === 'sql' ? `<a href="modulo-01-sql.html">📘 Abrir módulo completo...</a>` : ''}
```
Um bloco desses existe por módulo já criado (sql, modeling→modulo-02, airflow→modulo-03, python→modulo-04, cloud→modulo-05, process→modulo-06).

## Paleta de cores por módulo

Cada módulo tem uma cor de destaque (`--accent`) diferente, usada no header, active states, e no eyebrow das aulas:

| Módulo | Accent |
|---|---|
| SQL | `#ef4444` (vermelho) |
| Modelagem | `#8b5cf6` (roxo) |
| Airflow | `#10b981` (verde) |
| Python | `#3b82f6` (azul) |
| Cloud | `#0ea5e9` (azul claro) |
| Processos | `#f59e0b` (âmbar) |

## Convenções de conteúdo já estabelecidas

- Tom didático tipo "apostila" — introdução conceitual, seções `<h3>` numeradas conceitualmente, exemplos de código reais, 1-2 callouts de aviso/dica por aula, recapitulação de 5 pontos no final.
- Fundamentação em documentação oficial sempre que possível (citada na caixa NBLM em "Fonte complementar sugerida").
- Tags de prioridade (`critical`/`important`/`bonus`/`owned`) refletem o quanto aquele subtópico é cobrado em processos seletivos, calibradas com base no perfil real do usuário (ver seção "Contexto do usuário" abaixo).

## Contexto do usuário (para calibrar tom e exemplos)

Victor Breia é estudante de Engenharia de Software/Dados no Brasil, com dois projetos reais em produção:
- **pipeline-etransparente** — pipeline ETL open source (Python, Airflow, Docker, Azure VM, ADLS Gen2, arquitetura Medallion) que monitora transparência de ONGs brasileiras
- **Automação de Parcerias (IDC)** — sistema event-driven em Google Apps Script para o Instituto de Direito Coletivo

Exemplos e analogias no conteúdo das aulas frequentemente se conectam a esses dois projetos reais — isso é intencional e deve ser mantido ao criar novo conteúdo.

## Tarefas pendentes / próximos passos possíveis

1. Adicionar vídeos reais aos módulos 02-06 conforme forem gravados (só trocar/adicionar `video:` no objeto da lição correspondente — o placeholder já mostra o nome de arquivo esperado)
2. Confirmar se a convenção de nome de arquivo (prefixos abreviados `mod`, `air`, `py`, `cloud`, `proc`) é a que o usuário realmente vai usar, ou se prefere nomes completos como no padrão `sql`
3. Possível: testes cruzados entre módulos, ou um teste geral final cobrindo os 6 módulos
4. Possível: sincronizar o design entre hub e módulos (hoje o hub tem uma versão mais simples de "aula" por tema, sem a estrutura rica de subtópicos individuais)

## Regras importantes ao editar

- **Nunca usar `\\'` (barra invertida + aspas simples) dentro de strings JS de aspas simples** — já causou um bug de corrupção total do Módulo 1 (JS quebrado, página inteira travava). Prefira aspas duplas `"texto"` dentro de strings de aspas simples `'...'`, ou template literals com crase.
- Antes de considerar uma edição concluída, validar balanceamento de `{`, `(` e crases no bloco `<script>` (chaves e parênteses devem somar zero de diferença entre abertura/fechamento; contagem de crases deve ser par).
- Não reintroduzir dependência de `html2pdf.js` ou qualquer geração de PDF — a decisão de usar `.md` foi deliberada.
- Manter os arquivos como HTML único autossuficiente por módulo (sem imports entre arquivos, exceto os links `<a href>` de navegação entre hub e módulos).
