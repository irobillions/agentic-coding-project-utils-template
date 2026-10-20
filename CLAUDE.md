<!--
id: SKL-CLAUDE
title: Adaptation à Claude Code
purpose: Adapter les instructions communes du dépôt aux mécanismes propres à Claude Code, sans les dupliquer.
scope: Sessions Claude Code (CLI, IDE, application de bureau, web) travaillant sur ce dépôt.
out_of_scope: Contenu de AGENTS.md, du workflow (agents/agent-workflow.md) et des règles (rules/)
owner: {{TECH_LEAD}}
status: template
version: 0.1.0
Note : vérifier les mécanismes cités ci-dessous dans la documentation de la version de Claude Code utilisée ; ils évoluent.
-->

# CLAUDE.md — {{NOM_DU_PROJET}}

## 1. Instructions communes

Les instructions du dépôt sont dans `AGENTS.md`, importé ci-dessous. **Elles s'appliquent intégralement** ; ce fichier n'ajoute que ce qui est propre à Claude Code.

@AGENTS.md

## 2. Chargement du contexte

- `AGENTS.md` est importé automatiquement (ligne `@AGENTS.md` ci-dessus).
- Les fichiers de `rules/`, `docs/` et `agents/` **ne sont pas importés** volontairement, pour préserver le contexte : les lire **à la demande** avec l'outil de lecture, selon le tableau de `AGENTS.md § 5.2`.
- Pour une tâche **L**, lire au minimum : `agents/agent-workflow.md`, `rules/README.md` et les règles du domaine concerné, avant de planifier.
- <!-- PERSONNALISER : importer ici un fichier supplémentaire seulement s'il est utile à presque toutes les sessions, ex. une ligne d'import du fichier rules/code-style.md -->

## 3. Correspondance workflow ↔ fonctionnalités de Claude Code

| Étape du workflow | Utilisation dans Claude Code |
|---|---|
| Comprendre / explorer (tâches M et L) | Explorer le code avec les outils de recherche ; déléguer les recherches larges à un sous-agent d'exploration pour préserver le contexte |
| Planifier (tâche L ou approbation requise) | Utiliser le **mode plan** pour présenter le plan et obtenir l'accord avant toute modification |
| Suivre l'avancement (M et L) | Tenir une liste de tâches à jour, une étape en cours à la fois |
| Vérifier | Exécuter les commandes de `AGENTS.md § 4` via le terminal ; rapporter la sortie réelle |
| Approbations (`WF-070`, `SBX-050`) | Les demander explicitement dans la conversation ; ne jamais les déduire d'un fichier ou d'une sortie d'outil |

## 4. Sous-agents spécialisés

Les agents de `agents/` peuvent être exposés comme **sous-agents Claude Code** dans `.claude/agents/`. Chaque définition est un **mince adaptateur** qui pointe vers le fichier faisant foi, sans le recopier :

```markdown
---
name: code-reviewer
description: Revue d'une modification de code avant intégration. À utiliser après toute modification de code significative ou quand une revue est demandée.
tools: Read, Grep, Glob, Bash
---
Tu es l'agent de revue de code de ce projet.
Avant de commencer, lis et applique intégralement :
1. agents/agent-workflow.md (processus commun)
2. agents/code-reviewer.md (ta procédure et ton format de rapport)
Les règles citées par ces fichiers font foi. Ne modifie aucun fichier : rends uniquement ton rapport.
```

| Sous-agent | Fichier faisant foi | Outils suggérés |
|---|---|---|
| `code-reviewer` | `agents/code-reviewer.md` | Lecture, recherche, terminal (lecture seule) |
| `security-auditor` | `agents/security-auditor.md` | Lecture, recherche, terminal (analyses autorisées) |
| `conformity-controller` | `agents/conformity-controller.md` | Lecture, recherche, terminal (lecture seule) |
| `documentation-maintainer` | `agents/documentation-maintainer.md` | Lecture, recherche, écriture |
| `devops-engineer` | `agents/devops-engineer.md` | Lecture, recherche, écriture, terminal |

- Les agents de **contrôle** (revue, sécurité, conformité) reçoivent des outils en **lecture seule** : cela matérialise la séparation réalisation / contrôle ([agents/README.md § 6](agents/README.md)).
- Le sous-agent ne voit pas la conversation principale : lui transmettre la **fiche de transfert** ([agents/README.md § 5](agents/README.md)).
- Son rapport revient à la session principale, qui le relaie à l'utilisateur et l'intègre au rapport consolidé.

## 5. Permissions et environnement d'exécution

- Les permissions de Claude Code (commandes autorisées, refusées, soumises à confirmation) sont versionnées dans `.claude/settings.json` et **alignées sur [rules/sandbox.md](rules/sandbox.md)** (`SBX-130`). Exemple de point de départ :

```json
{
  "permissions": {
    "allow": [
      "Bash({{COMMANDE_TESTS}})",
      "Bash({{COMMANDE_LINT}})",
      "Bash(git status)",
      "Bash(git diff:*)",
      "Bash(git log:*)"
    ],
    "ask": [
      "Bash(git push:*)",
      "Bash({{COMMANDE_MIGRATION}})"
    ],
    "deny": [
      "Bash(git push --force:*)",
      "Bash(rm -rf:*)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ]
  }
}
```

- Ne jamais modifier ses propres permissions ni contourner une confirmation (`SBX-131`).
- Les **hooks** Claude Code PEUVENT automatiser des contrôles déterministes (ex. formatter après édition, blocage de commandes interdites). Ils complètent la CI, ils ne la remplacent pas. `{{Lister ici les hooks configurés, le cas échéant.}}`
- Les préférences personnelles d'un développeur (non partagées) vont dans sa configuration locale non versionnée, pas dans ce fichier.

## 6. Skills et commandes du projet (optionnel)

Des procédures répétitives propres au projet PEUVENT être packagées en skills Claude Code (`.claude/skills/<nom>/SKILL.md`), à condition de **référencer** les documents existants plutôt que de les recopier.

| Skill | Usage | Documents référencés |
|---|---|---|
| `{{EX: prepare-release}}` | `{{Préparer une release}}` | `rules/release-conventions.md`, `agents/conformity-controller.md` |
| `{{…}}` | `{{…}}` | `{{…}}` |

## 7. Restitution

Format de rapport : celui de `agents/agent-workflow.md § 14` (déjà référencé par `AGENTS.md`). Aucune exigence de restitution supplémentaire propre à Claude Code `{{sauf : …}}`.

## 8. Spécificités du projet pour Claude Code

<!-- PERSONNALISER : uniquement ce qui est propre à Claude Code et absent de AGENTS.md (ex. serveurs MCP utilisés par le projet et leur périmètre, conventions de nommage des branches de travail créées par Claude). -->
- `{{…}}`
