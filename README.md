# agilista

Automações de agilidade para as squads da Ploomes, executadas pelo Claude Code com o conector Atlassian.

## Skills

| Skill | O que faz |
|---|---|
| [`retro-para-kaizen`](.claude/skills/retro-para-kaizen/SKILL.md) | Exporta os cards do Plano de Ação de uma retrospectiva do Dev Panel para o board Kaizen da squad no Jira, sem duplicar. |

### Como usar `retro-para-kaizen`

1. Abra a retro no Dev Panel e copie a coluna **Plano de Ação**.
2. Numa sessão do Claude Code neste repositório, peça algo como:
   > Exporte o plano de ação da retro da squad Plataforma para o Kaizen (história MTC-46): <link da retro> + cards colados
3. A skill cria um Kaizen por card, preenche o campo **Plano de ação (Kaizen)**, move os itens para **Tarefas pendentes** e comenta o resultado na história.

Para incluir uma nova squad, complete a tabela de squads em `SKILL.md`.
