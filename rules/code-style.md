---
id: "RUL-CS"
title: "Conventions de code"
purpose: "Définir les conventions d'implémentation et de maintenabilité du code à l'échelle des fonctions, classes et modules."
scope: "Tout code source, script, configuration-as-code et code de test du dépôt."
out_of_scope:
  - "Architecture globale et frontières de modules (docs/architecture.md)"
  - "Catalogue d'anti-patterns (rules/anti-pattern.md)"
  - "Procédure de revue (agents/code-reviewer.md)"
  - "Méthodes de test (rules/testing.md)"
  - "Règles de sécurité (rules/security-conventions.md)"
references:
  - "README.md"
  - "anti-pattern.md"
  - "../docs/architecture.md"
owner: "{{TECH_LEAD}}"
status: "template"
version: "0.1.0"
---

# Conventions de code

> **Principe directeur :** le code est écrit une fois et lu des dizaines de fois. On optimise pour le **lecteur suivant**, la **modification sûre** et la **vérification automatique**.

<!-- PERSONNALISER : compléter § 1 avec les langages réels ; supprimer les sections sans objet (ex. § 9 si aucun code concurrent). Les règles génériques restent valables quel que soit le langage. -->

---

## 1. Langages, versions et guides de référence

| Langage / techno | Version minimale supportée | Guide idiomatique de référence | Formatter | Linter / analyse statique |
|---|---|---|---|---|
| `{{EX: TypeScript}}` | `{{EX: 5.x}}` | `{{EX: guide interne / Google TS Style}}` | `{{EX: Prettier}}` | `{{EX: ESLint + typescript-eslint}}` |
| `{{EX: Python}}` | `{{EX: 3.12}}` | `{{EX: PEP 8, PEP 257}}` | `{{EX: Ruff format}}` | `{{EX: Ruff, mypy --strict}}` |
| `{{EX: Go / Java / Kotlin / C# / Rust / Swift…}}` | `{{…}}` | `{{…}}` | `{{…}}` | `{{…}}` |
| `{{SQL / IaC / Shell}}` | `{{…}}` | `{{…}}` | `{{…}}` | `{{EX: shellcheck, tflint}}` |

#### CS-001 — Le guide idiomatique s'applique par défaut
- **Niveau :** DOIT
- **Énoncé :** pour tout point non couvert par ce fichier, le guide idiomatique du langage listé ci-dessus s'applique. Ce fichier ne documente que les choix **propres au projet** ou les arbitrages.
- **Vérification :** revue.

## 2. Formatage

#### CS-010 — Formatage automatique, non négociable
- **Niveau :** DOIT
- **Énoncé :** tout fichier est formaté par l'outil configuré du langage. Le formatage n'est **pas** un sujet de revue humaine.
- **Vérification :** auto — pre-commit et CI (`{{COMMANDE_FORMAT_CHECK}}`), bloquant.

#### CS-011 — Configuration partagée
- **Niveau :** DOIT
- **Énoncé :** la configuration de l'éditeur (`.editorconfig`), du formatter et du linter est versionnée dans le dépôt. Encodage **UTF-8**, fins de ligne **LF** (sauf contrainte de plateforme documentée), retour à la ligne final.

#### CS-012 — Changements de formatage isolés
- **Niveau :** DEVRAIT
- **Énoncé :** un reformatage de masse se fait dans un commit dédié, sans changement fonctionnel, et est ajouté à `.git-blame-ignore-revs`.

## 3. Nommage

#### CS-020 — Noms révélant l'intention
- **Niveau :** DOIT
- **Énoncé :** un nom décrit **ce que représente** l'élément ou **ce que fait** l'opération, dans le vocabulaire du domaine (glossaire de `docs/requirements.md`). Interdits : noms d'une lettre hors indices/lambdas triviales, abréviations non usuelles, noms trompeurs, suffixes vides (`Manager`, `Helper`, `Data`, `Info`) sans justification.

#### CS-021 — Conventions de casse

| Élément | Convention | Exemple |
|---|---|---|
| Variables, fonctions | `{{EX: camelCase / snake_case selon langage}}` | `{{…}}` |
| Types, classes, interfaces | `PascalCase` | `InvoiceRepository` |
| Constantes | `{{EX: UPPER_SNAKE_CASE}}` | `MAX_RETRY_COUNT` |
| Fichiers | `{{EX: kebab-case / snake_case}}` | `{{…}}` |
| Booléens | préfixe interrogatif (`is`, `has`, `can`, `should`) | `isActive` |
| Branches Git, commits | voir [release-conventions.md](release-conventions.md) | — |

#### CS-022 — Langue du code
- **Niveau :** DOIT
- **Énoncé :** identifiants, commentaires techniques et messages de log en **`{{EX: anglais}}`**. Les termes métier conservent la traduction fixée dans le glossaire du projet ; un même concept porte **un seul nom** dans tout le code.

#### CS-023 — Unités et sémantique dans les noms
- **Niveau :** DEVRAIT
- **Énoncé :** lorsqu'un type ne porte pas l'unité, le nom l'indique (`timeoutMs`, `sizeBytes`, `amountCents`). Préférer des types dédiés (`Duration`, `Money`) quand le langage le permet.

## 4. Organisation des fichiers et modules

#### CS-030 — Cohésion
- **Niveau :** DOIT
- **Énoncé :** un fichier/module regroupe des éléments qui changent pour la même raison. Les frontières entre modules et le sens des dépendances sont ceux de [docs/architecture.md](../docs/architecture.md) ; ils ne sont pas redéfinis ici.

#### CS-031 — Imports
- **Niveau :** DOIT
- **Énoncé :** imports triés automatiquement ; pas d'import inutilisé ; pas d'import « wildcard » sauf idiome du langage ; pas de dépendance circulaire.
- **Vérification :** auto.

#### CS-032 — Taille indicative
- **Niveau :** DEVRAIT
- **Énoncé :** au-delà des seuils suivants, un découpage est envisagé et, s'il n'est pas fait, justifié.

| Indicateur | Seuil d'alerte | Outil |
|---|---|---|
| Lignes par fonction | `{{EX: 50}}` | `{{…}}` |
| Lignes par fichier | `{{EX: 400}}` | `{{…}}` |
| Paramètres par fonction | `{{EX: 4}}` (au-delà : objet paramètre) | `{{…}}` |
| Complexité cyclomatique par fonction | `{{EX: 10}}` | `{{…}}` |
| Complexité cognitive par fonction | `{{EX: 15}}` | `{{…}}` |
| Profondeur d'imbrication | `{{EX: 3}}` | `{{…}}` |

## 5. Fonctions et classes

#### CS-040 — Responsabilité unique et niveau d'abstraction homogène
- **Niveau :** DOIT
- **Énoncé :** une fonction fait une chose, à un seul niveau d'abstraction. Les effets de bord sont explicites dans le nom ou la signature.

#### CS-041 — Séparation calcul / effets
- **Niveau :** DEVRAIT
- **Énoncé :** la logique métier est isolée dans des fonctions pures ou des objets du domaine ; les entrées/sorties (réseau, disque, base, horloge, aléa) sont injectées ou confinées aux bords (*functional core, imperative shell*). Cela rend le code testable sans mocks lourds.

#### CS-042 — Retours anticipés
- **Niveau :** DEVRAIT
- **Énoncé :** préférer les clauses de garde (*early return*) à l'imbrication profonde.

#### CS-043 — Pas de paramètres booléens de contrôle
- **Niveau :** DEVRAIT
- **Énoncé :** un booléen qui change le comportement d'une fonction est remplacé par deux fonctions, un enum ou un objet d'options nommé.

#### CS-044 — Composition avant héritage
- **Niveau :** DEVRAIT
- **Énoncé :** l'héritage est réservé aux vraies relations « est-un » respectant le principe de substitution de Liskov ; sinon composition / interfaces.

#### CS-045 — Immutabilité par défaut
- **Niveau :** DEVRAIT
- **Énoncé :** variables, champs et structures sont immuables sauf besoin justifié. Pas d'état global mutable.

## 6. Principes de conception locale

| Principe | Application attendue |
|---|---|
| **SOLID** | Appliqué avec discernement : interfaces petites et ciblées, dépendances vers des abstractions aux frontières du domaine. |
| **DRY** | Dupliquer une **connaissance** est interdit ; dupliquer du code ressemblant mais représentant deux règles distinctes est acceptable. Règle de trois avant d'abstraire. |
| **KISS / YAGNI** | Pas de généricité, d'extensibilité ou de configuration « au cas où ». |
| **Cohésion / couplage** | Forte cohésion interne, couplage faible et explicite entre modules. |
| **Loi de Déméter** | Éviter les chaînes `a.b().c().d()` traversant plusieurs objets. |
| **Principe de moindre surprise** | Un nom, une signature et un comportement cohérents avec les conventions du langage et du projet. |

## 7. Typage et contrats

#### CS-050 — Typage le plus strict disponible
- **Niveau :** DOIT
- **Énoncé :** le mode strict du système de types est activé (`{{EX: strict: true, mypy --strict, -Werror…}}`). Les types « échappatoires » (`any`, `Object`, `interface{}`, `dynamic`) sont interdits sauf aux frontières, avec validation immédiate.

#### CS-051 — Nullabilité explicite
- **Niveau :** DOIT
- **Énoncé :** l'absence de valeur est modélisée explicitement (types optionnels, `Option`, `Maybe`, nullable annotés). Pas de valeur sentinelle magique (`-1`, `""`) pour signifier « absent ».

#### CS-052 — Rendre les états invalides non représentables
- **Niveau :** DEVRAIT
- **Énoncé :** utiliser enums, unions discriminées, *value objects* et constructeurs validants plutôt que des chaînes et primitives libres (voir `AP` « obsession des primitives »).

#### CS-053 — Validation aux frontières
- **Niveau :** DOIT
- **Énoncé :** toute donnée entrant dans le système (API, fichier, file de messages, variable d'environnement, réponse d'un modèle d'IA) est validée et convertie en type du domaine **à la frontière** ; l'intérieur du système manipule des données déjà valides. Exigences de sécurité associées : [security-conventions.md](security-conventions.md).

#### CS-054 — Valeurs numériques sensibles
- **Niveau :** DOIT
- **Énoncé :** les montants monétaires n'utilisent **jamais** de flottant binaire (décimal ou entier en plus petite unité). Les dates/heures sont manipulées avec un type portant le fuseau ou en UTC ; conversion locale uniquement à l'affichage.

## 8. Erreurs, exceptions et ressources

#### CS-060 — Ne jamais avaler une erreur
- **Niveau :** DOIT
- **Énoncé :** une erreur est soit **traitée** (récupération réelle), soit **propagée** avec contexte, soit **journalisée et transformée** à une frontière. Un `catch` vide ou un `except: pass` est interdit.

#### CS-061 — Erreurs typées et contextualisées
- **Niveau :** DOIT
- **Énoncé :** les erreurs métier attendues sont des types/codes explicites ; les erreurs techniques conservent la cause d'origine (*chaining*). Le message aide au diagnostic sans exposer de donnée sensible.

#### CS-062 — Échouer tôt
- **Niveau :** DOIT
- **Énoncé :** une précondition violée ou une configuration invalide provoque une erreur immédiate et explicite, de préférence au démarrage.

#### CS-063 — Libération déterministe des ressources
- **Niveau :** DOIT
- **Énoncé :** fichiers, connexions, verrous, transactions sont libérés via les mécanismes idiomatiques (`try-with-resources`, `using`, `with`, `defer`, RAII), y compris en cas d'erreur.

#### CS-064 — Appels externes bornés
- **Niveau :** DOIT
- **Énoncé :** tout appel réseau ou E/S potentiellement bloquant a un **délai d'expiration** explicite. Les politiques de nouvelle tentative suivent [api-conventions.md](api-conventions.md).

## 9. Concurrence et asynchronisme

#### CS-070 — État partagé minimal
- **Niveau :** DOIT
- **Énoncé :** l'état partagé mutable entre fils/tâches est évité ; à défaut, il est protégé et sa stratégie de synchronisation est documentée au point de déclaration.

#### CS-071 — Pas d'asynchronisme orphelin
- **Niveau :** DOIT
- **Énoncé :** toute tâche asynchrone lancée est attendue, supervisée ou explicitement détachée avec gestion d'erreur. Annulation et délais propagés (`context`, `CancellationToken`, `AbortSignal`…).

#### CS-072 — Pas d'appel bloquant dans une boucle d'événements
- **Niveau :** DOIT
- **Énoncé :** dans un runtime asynchrone, les opérations bloquantes sont déportées.

## 10. Configuration, constantes et internationalisation

#### CS-080 — Pas de nombres ni chaînes magiques
- **Niveau :** DOIT
- **Énoncé :** les valeurs ayant un sens sont nommées (constante, enum, configuration).

#### CS-081 — Configuration externalisée
- **Niveau :** DOIT
- **Énoncé :** ce qui varie entre environnements est injecté par configuration (variables d'environnement, fichiers, service de configuration), validé au démarrage, jamais codé en dur. Secrets : [security-conventions.md](security-conventions.md).

#### CS-082 — Textes utilisateur internationalisables
- **Niveau :** `{{DOIT si NFR-I18N existe, sinon DEVRAIT}}`
- **Énoncé :** aucun texte destiné à l'utilisateur n'est codé en dur ; dates, nombres, devises et pluriels sont formatés selon la locale (ICU / CLDR). Pas de concaténation de fragments traduits.

## 11. Commentaires et documentation du code

#### CS-090 — Commenter le « pourquoi »
- **Niveau :** DOIT
- **Énoncé :** les commentaires expliquent l'intention, les contraintes non évidentes, les compromis et les références (`NFR-…`, `ADR-…`, ticket). Ils ne paraphrasent pas le code. Le code commenté (mort) est supprimé.

#### CS-091 — API publiques documentées
- **Niveau :** DOIT
- **Énoncé :** toute API publique d'un module (exportée, réutilisable) a une docstring/commentaire de documentation : objet, paramètres, retour, erreurs, effets de bord, exemple si non trivial. Format : `{{EX: JSDoc / TSDoc / Google docstrings / KDoc / rustdoc}}`.

#### CS-092 — Marqueurs de dette tracés
- **Niveau :** DOIT
- **Énoncé :** tout `TODO`/`FIXME`/`HACK` référence un ticket : `TODO({{TICKET-123}}): …`. Un marqueur sans ticket est refusé en revue.

## 12. Dépendances

#### CS-100 — Ajout de dépendance justifié
- **Niveau :** DOIT
- **Énoncé :** toute nouvelle dépendance tierce est justifiée (besoin, alternatives, maintenance active, licence compatible avec `{{POLITIQUE_LICENCES}}`, taille, sécurité). Les contrôles de sécurité de la chaîne d'approvisionnement sont définis dans [security-conventions.md](security-conventions.md).

#### CS-101 — Versions verrouillées
- **Niveau :** DOIT
- **Énoncé :** un fichier de verrouillage (`lockfile`) est versionné pour toute application déployable.

## 13. Refactorisation

#### CS-110 — Règle du campeur, bornée
- **Niveau :** DEVRAIT
- **Énoncé :** laisser le code touché un peu plus propre, **dans le périmètre de la tâche**. Une refactorisation large fait l'objet d'une tâche/PR distincte.

#### CS-111 — Refactoriser sous filet de tests
- **Niveau :** DOIT
- **Énoncé :** une refactorisation ne modifie pas le comportement observable et s'appuie sur des tests existants ou ajoutés au préalable ([testing.md](testing.md)).

Déclencheurs de refactorisation : dépassement des seuils § 4, anti-pattern identifié ([anti-pattern.md](anti-pattern.md)), duplication d'une connaissance, difficulté avérée à tester ou modifier.

## 14. Vérification automatique et dérogations

| Contrôle | Commande | Étape | Bloquant |
|---|---|---|---|
| Formatage | `{{COMMANDE}}` | pre-commit, CI | Oui |
| Lint | `{{COMMANDE}}` | pre-commit, CI | Oui (erreurs) |
| Typage | `{{COMMANDE}}` | CI | Oui |
| Complexité / duplication | `{{EX: SonarQube, jscpd, radon}}` | CI | `{{Oui / Non}}` |

#### CS-120 — Suppressions locales justifiées
- **Niveau :** DOIT
- **Énoncé :** la désactivation ponctuelle d'une règle de linter (`// eslint-disable-next-line`, `# noqa`, `@SuppressWarnings`) est ciblée sur une ligne/règle précise et accompagnée d'une justification. La désactivation globale d'une règle relève de [README.md § 7](README.md).

#### CS-121 — Le code généré est exclu, pas ignoré
- **Niveau :** DEVRAIT
- **Énoncé :** le code généré est isolé dans des répertoires identifiés, exclu des linters, et régénérable par une commande documentée dans [AGENTS.md](../AGENTS.md).

## 15. Spécificités du projet

<!-- PERSONNALISER : conventions propres au framework (ex. structure des composants UI, conventions ORM, patterns imposés par l'architecture). Chaque règle prend un identifiant CS-2NN. -->

#### CS-200 — `{{TITRE}}`
- **Niveau :** `{{…}}`
- **Énoncé :** `{{…}}`
- **Vérification :** `{{…}}`
