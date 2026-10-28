---
name: retro-para-kaizen
description: Exporta os cards da coluna "Plano de Ação" de uma retrospectiva do Dev Panel (ploomes-developers.web.app/agile/retro/...) para o board Kaizen da squad no Jira, criando um item do tipo Kaizen por card, sem duplicar. Use quando pedirem "exportar plano de ação da retro", "levar a retro para o Kaizen", "criar Kaizens da retrospectiva", ou citarem uma história do épico Retrospectiva (MTC-1), como MTC-44, MTC-45 e MTC-46.
---

# Retro → Kaizen

Leva os planos de ação de uma retrospectiva para o board Kaizen da squad no Jira.
Atende os RFs das histórias do épico MTC-1 (Retrospectiva): exportação individual ou em lote,
preservação das informações, squad correta e controle de duplicidade.

Ferramentas: conector Atlassian (Rovo ou Atlassian MCP), `cloudId` = `ploomes.atlassian.net`.

## Entradas

Pergunte só o que faltar:

1. **Squad**, para descobrir o projeto Jira na tabela de squads abaixo.
2. **Cards da coluna Plano de Ação.** O Dev Panel exige login Google @ploomes.com,
   então o agente não consegue ler a retro sozinho. Peça para a pessoa abrir a retro e colar
   a coluna aqui (texto ou print). Formato que o Dev Panel copia:
   ```
   ## Plano de ação (N cartões)
   ### <texto do card> (0 votos)
   _sem comentários_
   ```
3. **Link da retro**, para registrar a origem.
4. **História do Jira** que pediu a exportação (ex.: MTC-44). É opcional e serve para comentar o resultado.

## Squads

| Squad | Projeto Jira | Board Kaizen | Slug da retro |
|---|---|---|---|
| Enterprise / Analytics | `CORE` | [1015](https://ploomes.atlassian.net/jira/software/c/projects/CORE/boards/1015) | `enterprise-analytics` |
| N2 | _preencher_ | _preencher_ | _preencher_ |
| Plataforma | `PLAT` | _preencher_ | `plataforma` (label usada; confirmar slug no link da retro) |

Se a squad não estiver na tabela, pergunte o projeto. Depois da execução, complete a linha
neste arquivo. Nunca crie itens no projeto de outra squad sem confirmação explícita (RF03).

## Passo a passo

### 1. Validar o projeto

- `getJiraProjectIssueTypesMetadata(projectKey)`: confirme que existe o tipo **Kaizen**.
  Se não existir, pare e avise.
- Pegue um item Kaizen existente do projeto e leia `getJiraIssue(..., expand: "names,editmeta", fields: ["*all"])`
  para confirmar os campos editáveis. No CORE, eles são:
  - `customfield_11250`: **Plano de ação (Kaizen)**, na aba Geral. É onde o texto do card aparece.
  - `customfield_11248` Cenário Atual, `customfield_11249` Cenário Desejado,
    `customfield_11251` Impacto da Melhoria, `customfield_11246` Resultado da ação.
  - **Atenção:** a tela do Kaizen não exibe o campo Descrição. Texto gravado só na descrição
    fica invisível para o time. Sempre preencha `customfield_11250`.

### 2. Controle de duplicidade (RF04)

Antes de criar, busque o que já foi exportado:

```
project = <PROJ> AND issuetype = Kaizen AND labels = "retro-<slug>"
```

Compare também por similaridade de texto com os Kaizens abertos do projeto. Pule os cards já
exportados e avise quais foram pulados. Se só houver um item parecido de uma retro anterior,
crie normalmente e cite o item antigo como relacionado.

### 3. Criar um Kaizen por card

`createJiraIssue` com:

- `issueTypeName`: `Kaizen`
- `summary`: título curto, com até cerca de 80 caracteres e verbo no infinitivo, resumindo o card.
  Se o card tiver título próprio, use o título.
- `description` (markdown): texto integral, origem, responsável, prazo e data de exportação.
  Modelo:
  ```
  **Plano de ação da retrospectiva (<Squad>)**

  <texto integral do card>

  ---
  - **Origem:** [Retrospectiva <Squad>](<link da retro>) — coluna Plano de Ação (<n> votos, <comentários>)
  - **Responsável(is):** <nome ou "não definido na retro">
  - **Prazo:** <data ou "não definido na retro">
  - **Exportado em:** <dd/mm/aaaa> via <história>
  ```
- `additional_fields`: `{"labels": ["retrospectiva", "retro-<slug>"]}`
- Se o card tiver responsável, resolva o accountId com `lookupJiraAccountId` e passe `assignee_account_id`.
  Se tiver prazo, passe `duedate` (`YYYY-MM-DD`).

Depois, `editJiraIssue` com `{"customfield_11250": "<texto integral do card>"}`.

No PLAT, `customfield_11250` está na tela de criação e exige ADF: passe direto em `additional_fields`
como `{"type": "doc", "version": 1, "content": [{"type": "paragraph", "content": [{"type": "text", "text": "..."}]}]}`.
Texto puro é recusado com "não é um conteúdo válido do formato de documento da Atlassian (ADF)".

### 4. Mover para Tarefas pendentes

O item nasce em **Backlog**, que não aparece no board Kanban. Use `getTransitionsForJiraIssue`
e aplique a transição que leva a **Tarefas pendentes**. No CORE e no PLAT, a transição é a `7` ("Priorizar").
Em outros projetos o ID pode mudar.

### 5. Conferir e registrar

- Leia de volta com JQL `key in (...)`, pedindo `customfield_11250` e `status`, e confirme os N itens.
- Comente na história de origem a lista `CHAVE — título`, as labels usadas, os cards pulados por
  duplicidade e os campos que ficaram vazios (responsável e prazo).

## Resultado para a pessoa

Mostre uma tabela `Item | Plano de ação` com links, diga o que ficou sem responsável ou prazo,
e ofereça mover a história de origem para Feito.

## Histórico

| Data | Squad | Retro | História | Itens |
|---|---|---|---|---|
| 28/10/2026 | Enterprise / Analytics | [-P22oNBFpDAA_aV19UO3](https://ploomes-developers.web.app/agile/retro/enterprise-analytics/-P22oNBFpDAA_aV19UO3) | MTC-44 | CORE-4112 a CORE-4118 |
| 28/10/2026 | Plataforma | _link não informado_ | MTC-46 | PLAT-637 a PLAT-644 |
