# Project Template — Référentiel documentaire d'ingénierie logicielle

Un **template de documentation prêt à l'emploi** pour démarrer n'importe quel projet logiciel (web, mobile, API, SaaS, application métier, projet intégrant de l'IA) avec un socle solide de **qualité, sécurité, fiabilité, maintenabilité et traçabilité**.

Il est conçu pour être lu **par les humains et par les assistants IA** (Claude Code, Codex, Cursor, Copilot…), qui y trouvent les règles du projet, un workflow de travail commun et des agents spécialisés.

> **Principe :** une source pour définir, un workflow pour agir, des agents pour exécuter et vérifier, des preuves pour conclure.

---

## Pourquoi ce template ?

- **Les meilleures pratiques, déjà écrites.** Conventions de code, d'API, de sécurité, de tests, d'observabilité, de sauvegarde et de livraison inspirées des référentiels reconnus (OWASP, NIST, SLSA, OpenTelemetry, SRE, ISO 25010/29148/42010, SemVer, WCAG…).
- **Générique et personnalisable.** Les parties propres à un projet sont des `{{PLACEHOLDERS}}`, souvent avec une valeur d'exemple (`{{EX: 80 %}}`).
- **Sans duplication.** Chaque règle, seuil ou décision a **un seul document propriétaire** ; les autres le référencent par son identifiant.
- **Vérifiable.** Chaque règle a un identifiant stable (`SEC-061`, `TST-121`…), un niveau (DOIT / DEVRAIT / PEUT) et une méthode de vérification.
- **Pensé pour l'IA.** Un workflow commun, des agents spécialisés, des limites d'autonomie et des règles d'approbation explicites pour travailler en sécurité avec des assistants IA.

---

## Structure

```text
.
├── AGENTS.md                  # Point d'entrée commun pour tous les assistants IA
├── CLAUDE.md                  # Adaptation à Claude Code (importe AGENTS.md)
│
├── docs/                      # Que doit faire le logiciel et comment est-il conçu ?
│   ├── requirements.md        # Exigences, critères d'acceptation, traçabilité
│   ├── architecture.md        # Architecture (arc42 + C4), données, déploiement, ADR
│   └── design-UX-UI.md        # Identité visuelle, design system, principes et conventions UX/UI
│
├── rules/                     # Quelles politiques techniques respecter ?
│   ├── README.md              # Gouvernance : index, identifiants, autorité, dérogations, gravité
│   ├── code-style.md          # Conventions de code
│   ├── anti-pattern.md        # Catalogue de diagnostic et remédiation
│   ├── api-conventions.md     # REST, GraphQL, gRPC, événements, webhooks
│   ├── security-conventions.md# Sécurité applicative et chaîne d'approvisionnement
│   ├── testing.md             # Stratégie et méthodes de test
│   ├── backup-conventions.md  # Sauvegarde et restauration
│   ├── sandbox.md             # Exécution isolée, permissions, agents IA
│   ├── observability.md       # Logs, métriques, traces, SLI/SLO, alertes
│   └── release-conventions.md # Versionnement, CI/CD, Quality Gates, déploiement
│
├── agents/                    # Quel spécialiste intervient et comment ?
│   ├── README.md              # Catalogue et coordination des agents
│   ├── agent-workflow.md      # Cycle universel de traitement d'une tâche
│   ├── code-reviewer.md
│   ├── conformity-controller.md
│   ├── devops-engineer.md
│   ├── documentation-maintainer.md
│   └── security-auditor.md
│
└── skills/                    # Emplacement réservé aux futures skills d'assistant IA
```

| Famille | Question | Contenu |
|---|---|---|
| `docs/` | **Quoi** et **comment c'est conçu** | Besoins et décisions propres au projet |
| `rules/` | **Quelles règles** | Conventions, exigences générales, contrôles |
| `agents/` | **Qui fait quoi, comment** | Procédures, responsabilités, livrables |
| `AGENTS.md` / `CLAUDE.md` | **Comment l'IA s'oriente** | Contexte, commandes, points d'entrée |

---

## Démarrer un nouveau projet

1. **Copier le template** dans le nouveau dépôt (ou utiliser *Use this template* sur GitHub).
2. **Renseigner les sources officielles**, sans répliquer les valeurs ailleurs :

   | Information | Où |
   |---|---|
   | Nom, objectifs, utilisateurs, exigences, seuils (performance, disponibilité, RPO/RTO, accessibilité…) | `docs/requirements.md` |
   | Technologies, composants, données, déploiement, décisions (ADR) | `docs/architecture.md` |
   | Identité de marque, liens Figma, design system, parcours, conventions UX/UI | `docs/design-UX-UI.md` |
   | Valeurs graphiques (couleurs, polices, espacements…) | Fichiers de design tokens (localisés dans `docs/design-UX-UI.md`) |
   | Langages, versions, outils de lint/format | `rules/code-style.md` |
   | Commandes concrètes (build, tests, lint…) et structure du dépôt | `AGENTS.md` |
   | Propriétaires des règles, autorités de dérogation | `rules/README.md` |
   | Permissions des assistants IA | `rules/sandbox.md` + `.claude/settings.json` |

3. **Parcourir chaque fichier** et traiter les marqueurs :
   - `{{PLACEHOLDER}}` → remplacer ;
   - `<!-- PERSONNALISER : … -->` → adapter ou retirer ce qui ne s'applique pas ;
   - sections « si applicable » (IA, mobile, multi-locataire, GraphQL/gRPC…) → conserver ou marquer « Non applicable : justification ».
4. **Ajouter les règles propres au projet** dans la section « Spécificités du projet » de chaque fichier (identifiants `XXX-200` et suivants).
5. **Installer les fichiers IA** aux emplacements réellement lus par vos outils (voir ci-dessous).
6. **Valider avec la checklist** en fin de page.

Pour retrouver les marqueurs restants :

```bash
grep -rn "{{" --include=*.md .
```

---

## Conventions du référentiel

### Identifiants

| Préfixe | Document propriétaire |
|---|---|
| `FR`, `NFR-*`, `BR`, `UC`, `INT`, `CON` | `docs/requirements.md` |
| `ADR` | `docs/architecture.md` |
| `UX` | `docs/design-UX-UI.md` |
| `GOV` | `rules/README.md` |
| `CS`, `AP`, `API`, `SEC`, `TST`, `BKP`, `SBX`, `OBS`, `REL` | Fichier correspondant de `rules/` |
| `WF` | `agents/agent-workflow.md` |

Un identifiant n'est jamais réutilisé. Les autres documents **citent** l'identifiant au lieu de recopier la règle.

### Niveaux normatifs

**DOIT / NE DOIT PAS** (écart = dérogation formelle) · **DEVRAIT** (écart justifié par écrit) · **PEUT** (libre). Définitions : `rules/README.md § 1`.

### Gravité commune des constats

**S1 Bloquant** · **S2 Majeur** · **S3 Mineur** · **S4 Suggestion** — définie dans `rules/README.md § 11`, utilisée par les agents et les Quality Gates.

### Statuts de preuve

✅ **Vérifié** (exécuté et observé) · 🔎 **Déduit** (analyse sans exécution) · ⚠️ **Non vérifié**. Une vérification non exécutée n'est jamais présentée comme réussie.

---

## Utilisation avec les assistants IA

- **`AGENTS.md`** à la racine est lu par la plupart des assistants compatibles. Il oriente vers le workflow (`agents/agent-workflow.md`), les règles pertinentes et les agents spécialisés.
- **`CLAUDE.md`** importe `AGENTS.md` (`@AGENTS.md`) et n'ajoute que les spécificités de Claude Code : mode plan, sous-agents (`.claude/agents/`), permissions (`.claude/settings.json`), hooks, skills.
- **`skills/`** accueille les futures skills ; pour Claude Code, elles doivent être placées dans `.claude/skills/<nom>/SKILL.md` pour être chargées.
- Les mécanismes propres à chaque outil évoluent : vérifier la documentation de la version utilisée.

---

## Checklist avant adoption

- [ ] Les chemins et la casse des fichiers correspondent à la structure ci-dessus.
- [ ] Chaque fichier a un objectif, un périmètre et un hors périmètre explicites.
- [ ] Chaque domaine normatif a une seule source officielle.
- [ ] Les agents référencent les règles plutôt que de les dupliquer.
- [ ] Tous les agents appliquent `agents/agent-workflow.md`.
- [ ] `AGENTS.md` ne redéfinit pas le workflow ; `CLAUDE.md` ne contient que les adaptations propres à Claude.
- [ ] Les valeurs mesurables du produit ne figurent que dans `docs/requirements.md`.
- [ ] Les contradictions, dérogations et décisions sensibles ont une voie d'arbitrage.
- [ ] Aucun contrôle ne peut être déclaré réussi sans exécution ou preuve.
- [ ] Les fichiers d'instructions IA sont chargés par chaque outil cible.

---

## Référentiels utilisés

ISO/IEC 25010 · ISO/IEC/IEEE 29148, 42010, 12207 · NIST SSDF (SP 800-218), SP 800-63B · OWASP ASVS, Top 10, API Security Top 10, Top 10 for LLM Applications, MASVS · SLSA, CycloneDX/SPDX · OpenTelemetry, Google SRE · SemVer, Conventional Commits, Keep a Changelog · WCAG 2.2 · RFC 9457, RFC 3339 · arc42, C4.

L'utilisation de ces référentiels comme source d'inspiration ne vaut ni certification ni conformité automatique.

---

## Contribuer

Les suggestions d'amélioration peuvent être proposées via les *issues*. Toute modification d'une règle suit la procédure de `rules/README.md § 7` : motivation, analyse d'impact, version incrémentée.

## Licence

© 2026 Christ Bouka. **Tous droits réservés.**

Aucune licence d'utilisation n'est accordée : toute reproduction, modification, distribution ou utilisation, totale ou partielle, nécessite l'autorisation écrite préalable de l'auteur. Voir le fichier [LICENSE](LICENSE).
