# Arquitetura Final — Bootcamp etransparente (sem login)

Substitui o plano anterior de login/senha + Azure Functions + Table Storage. Decisão final: manter o sistema **100% estático**, sem backend, hospedado no Azure Static Web Apps, com personalização leve e vídeos externos.

## Decisões finais (resumo)

| Item | Decisão |
|---|---|
| Hospedagem | Azure Static Web Apps, plano **Free**, como hospedagem pura de arquivo estático — sem API/Functions |
| Autenticação | **Nenhuma.** Sem login, sem senha, sem cadastro |
| Personalização | Modal pedindo o nome na primeira visita, salvo em `localStorage`, exibido como "Olá, {nome}" no header |
| Progresso | Continua em `localStorage`, como já funciona hoje |
| Vídeos | Azure Blob Storage (container público de leitura), referenciados por URL direta — **não** ficam no repositório nem no YouTube |
| Portabilidade de dados | Botões de **exportar/importar backup em `.json`**, cobrindo progresso de todos os módulos + nome salvo, para a pessoa levar entre dispositivos manualmente |

## Por que essa combinação faz sentido

- Hospedar como Static Web App resolve um problema que existia na versão 100% local: `localStorage` em arquivos abertos via `file://` **não é confiável entre navegadores** (Chrome trata cada arquivo local como origem isolada; Firefox às vezes compartilha por pasta). Hospedado num domínio real, todas as páginas do site compartilham o mesmo `localStorage` de forma consistente.
- Sem login, o Static Web App não precisa de API — é literalmente arquivo estático servido, o uso mais simples e barato possível do serviço.
- Vídeos no Blob Storage evitam o repositório Git inchado (ver conversa anterior: ~2GB em vídeos quebraria o fluxo de clone/fork) e evitam qualquer questão de política de conteúdo do YouTube — é só um espaço de arquivo público.
- Export/Import em JSON é o substituto direto de "sincronizar entre dispositivos" sem precisar de banco de dados — a pessoa faz isso manualmente quando quiser levar o progresso para outra máquina.

## Custo esperado

- Static Web Apps Free: **$0**
- Blob Storage: armazenamento (~1.9GB de vídeo quando tudo estiver gravado) ≈ **$0.04/mês**; egress coberto pelos 100GB grátis/mês da Azure para o volume esperado de voluntários
- Total: within o crédito de $2.000/ano do plano nonprofit, sem chegar perto do limite

## O que NÃO existe mais neste plano (comparado à primeira versão)

- Sem Azure Functions
- Sem Table Storage (nem qualquer banco de dados)
- Sem JWT, sem hash de senha, sem rotas de signup/login
- Sem pasta `api/` no projeto — só `app location`, sem `api location`

## Estrutura de pastas do projeto

```
etransparente-bootcamp/
├── src/                                   ← tudo que o Static Web App serve
│   ├── data-engineer-study-system.html
│   ├── modulo-01-sql.html
│   ├── modulo-02-modelagem.html
│   ├── modulo-03-airflow.html
│   ├── modulo-04-python.html
│   ├── modulo-05-cloud.html
│   └── modulo-06-processos.html
└── staticwebapp.config.json               ← configuração mínima, sem seção "auth"
```

Não há pasta `api/`. Os vídeos não moram no repositório — ficam no Blob Storage e são referenciados por URL absoluta.

### `staticwebapp.config.json` mínimo
```json
{
  "navigationFallback": {
    "rewrite": "/data-engineer-study-system.html"
  }
}
```

## Modelo de dados no `localStorage` (sem mudança na essência, só um item novo)

Chaves já existentes, mantidas como estão:
```
de_study_local_v1          → progresso do hub
de_module_sql_v1           → progresso do Módulo 1
de_module_modelagem_v1     → progresso do Módulo 2
de_module_airflow_v1       → progresso do Módulo 3
de_module_python_v1        → progresso do Módulo 4
de_module_cloud_v1         → progresso do Módulo 5
de_module_processos_v1     → progresso do Módulo 6
```

Chave nova:
```
de_user_name                → string simples com o nome digitado no modal
```

## Vídeos — referência por URL do Blob Storage

O campo `video` de cada lição passa a ser uma **URL completa** (não mais um ID do YouTube nem caminho de arquivo local):

```js
// Formato esperado:
video: 'https://SEUSTORAGEACCOUNT.blob.core.windows.net/videos/sql/sql1.Logica_de_Execucao_SQL.mp4',
```

E `renderVideoBlock()` volta a gerar uma tag `<video>` nativa (não mais `<iframe>` do YouTube):

```javascript
function renderVideoBlock(lesson, lessonNum) {
  if (lesson.video) {
    return `<div class="video-wrap">
      <video controls preload="metadata">
        <source src="${lesson.video}" type="video/mp4">
        Seu navegador não suporta vídeo HTML5.
      </video>
    </div>`;
  }
  return `<div class="video-placeholder">
    <div class="vp-icon">🎬</div>
    <div class="vp-title">Vídeo ainda não adicionado</div>
    <div class="vp-sub">Suba no Blob Storage e cole a URL completa no campo video: da aula</div>
  </div>`;
}
```

> Nomes de arquivo com acentos/caracteres especiais (como já usados: `Lógica_de_Execução_SQL.mp4`) funcionam no Blob Storage, mas a URL final vem URL-encoded automaticamente pelo Azure. Copie a URL exatamente como o portal mostrar após o upload — não digite manualmente.

## Passo a passo de implementação (ordem sugerida)

1. No Azure Portal (mesma conta do dashboard.etransparente.org): criar um Storage Account novo (ou reaproveitar um existente do projeto), com um container chamado `videos`, configurado para **acesso público de leitura em nível de blob** (não o Storage Account inteiro — só esse container).
2. Fazer upload dos 9 vídeos do módulo SQL nesse container (pode manter subpastas virtuais, ex: `videos/sql/sql1....mp4`, o Blob Storage aceita "/" no nome do blob como se fosse pasta).
3. Copiar a URL pública de cada blob e atualizar o campo `video:` das 9 lições em `modulo-01-sql.html`.
4. Implementar o modal de nome + export/import JSON (ver `PROMPTS-CLAUDE-CODE.md` — prompts prontos para isso).
5. Testar tudo localmente antes do deploy (abrir os HTMLs direto no navegador já deve funcionar, incluindo os vídeos do Blob, já que são URLs públicas).
6. Criar o recurso Azure Static Web App, conectar ao repositório GitHub, configurar `app location: /src`, sem `api location`.
7. Deploy. Testar a URL pública final com 2-3 voluntários antes de divulgar pra todos.
8. Conforme novos vídeos forem gravados: upload no Blob Storage → copiar URL → colar no campo `video:` da lição correspondente → commit/push (o Static Web App faz redeploy automático a cada push, via GitHub Actions).

## Regras importantes (mantidas do documento anterior)

- **Nunca usar `\'` dentro de strings JS de aspas simples** — já corrompeu o Módulo 1 uma vez. Prefira aspas duplas dentro de strings de aspas simples, ou template literals.
- Validar balanceamento de `{`, `(` e crases no `<script>` antes de considerar uma edição concluída.
- Manter os arquivos como HTML único autossuficiente por módulo.