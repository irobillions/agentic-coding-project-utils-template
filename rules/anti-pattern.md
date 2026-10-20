---
id: "RUL-AP"
title: "Catalogue des anti-patterns"
purpose: "Fournir une base de diagnostic et de remédiation des pratiques problématiques, en renvoyant vers les règles propriétaires."
scope: "Code, architecture, données, API, tests, exploitation, concurrence et composants d'IA."
out_of_scope:
  - "Définition des règles de code, sécurité, test ou API (fichiers propriétaires)"
  - "Procédure de revue (agents/code-reviewer.md)"
references:
  - "README.md"
  - "code-style.md"
  - "security-conventions.md"
  - "testing.md"
  - "api-conventions.md"
  - "../docs/architecture.md"
owner: "{{TECH_LEAD}}"
status: "template"
version: "0.1.0"
---

# Catalogue des anti-patterns

> **Nature de ce fichier :** un outil de **diagnostic**, pas une seconde source de règles. Quand un anti-pattern viole une règle, l'entrée **cite l'identifiant** de la règle propriétaire ; c'est cette règle qui fait foi.
>
> **Un anti-pattern est un signal, pas un verdict.** Chaque entrée précise quand le problème est réel ; le contexte peut justifier un compromis (§ 10).

<!-- PERSONNALISER : ajouter les anti-patterns observés dans le projet (post-mortems, revues récurrentes) dans la plage AP-9NN. Retirer les catégories sans objet. -->

---

## 0. Gabarit d'une entrée

```markdown
#### AP-<NNN> — <Nom> (<nom anglais usuel>)
- **Symptômes :** signes observables dans le code ou le système.
- **Problème réel quand :** conditions dans lesquelles il cause un dommage.
- **Risques :** impacts (fiabilité, sécurité, coût de maintenance…).
- **Remédiation :** alternatives et approche de correction.
- **Règle propriétaire :** <ID> ou « aucune — diagnostic seul ».
- **Détection :** manuelle | statique (<outil/règle>) | runtime (<signal>).
```

Plages d'identifiants : `AP-1xx` code · `AP-2xx` architecture · `AP-3xx` données · `AP-4xx` API · `AP-5xx` tests · `AP-6xx` exploitation · `AP-7xx` concurrence · `AP-8xx` IA · `AP-9xx` spécifiques au projet.

---

## 1. Code

#### AP-101 — Objet dieu (God Object / God Class)
- **Symptômes :** classe/module de très grande taille, nombreuses dépendances, nom vague (`Manager`, `Service`, `Utils`), modifié par presque toutes les PR.
- **Problème réel quand :** il concentre plusieurs raisons de changer et devient un point de conflit.
- **Risques :** régressions, revues difficiles, tests lents et fragiles.
- **Remédiation :** extraire par responsabilité / concept métier ; introduire des *value objects* et services ciblés.
- **Règle propriétaire :** `CS-030`, `CS-040`, seuils `CS-032`.
- **Détection :** statique (taille, fan-in/fan-out, *churn* Git).

#### AP-102 — Erreur avalée (Swallowed Exception)
- **Symptômes :** `catch {}` vide, `except: pass`, erreur seulement journalisée puis ignorée sans décision.
- **Problème réel quand :** toujours, sauf récupération réellement intentionnelle et commentée.
- **Risques :** pannes silencieuses, corruption de données, diagnostic impossible.
- **Remédiation :** traiter, propager avec contexte, ou transformer à une frontière.
- **Règle propriétaire :** `CS-060`, `CS-061`.
- **Détection :** statique (règles de linter dédiées).

```text
// ❌ Anti-pattern
try { save(order) } catch (e) { }

// ✅ Correction
try { save(order) } catch (e) { throw new OrderPersistenceError(order.id, { cause: e }) }
```

#### AP-103 — Nombres et chaînes magiques (Magic Numbers / Strings)
- **Symptômes :** littéraux sans nom (`if status == 3`, `sleep(86400)`).
- **Risques :** incompréhension, modifications partielles.
- **Remédiation :** constantes nommées, enums, configuration.
- **Règle propriétaire :** `CS-080`.
- **Détection :** statique.

#### AP-104 — Copier-coller de connaissance (Copy-Paste Programming)
- **Symptômes :** même règle métier implémentée à plusieurs endroits avec de légères variations.
- **Problème réel quand :** les copies représentent **la même connaissance** (elles devront évoluer ensemble).
- **Risques :** corrections partielles, incohérences métier.
- **Remédiation :** extraire après la troisième occurrence ; ne pas fusionner deux règles qui se ressemblent par hasard.
- **Règle propriétaire :** principe DRY, `code-style.md § 6`.
- **Détection :** statique (détecteur de duplication).

#### AP-105 — Obsession des primitives (Primitive Obsession)
- **Symptômes :** emails, montants, identifiants, statuts manipulés en `string`/`int` nus ; validations répétées.
- **Risques :** états invalides, inversions de paramètres, erreurs d'unité.
- **Remédiation :** *value objects*, types dédiés, enums, unions discriminées.
- **Règle propriétaire :** `CS-052`, `CS-023`.
- **Détection :** manuelle.

#### AP-106 — Code en flèche (Arrow Code / Deep Nesting)
- **Symptômes :** imbrication `if/for/try` au-delà de 3 niveaux.
- **Remédiation :** clauses de garde, extraction de fonctions, polymorphisme.
- **Règle propriétaire :** `CS-042`, `CS-032`.
- **Détection :** statique (profondeur, complexité cognitive).

#### AP-107 — Paramètre booléen de contrôle (Flag Argument)
- **Symptômes :** `render(true, false)` ; la fonction a deux comportements.
- **Remédiation :** deux fonctions, enum ou objet d'options nommé.
- **Règle propriétaire :** `CS-043`.
- **Détection :** manuelle / statique.

#### AP-108 — État global mutable (Global Mutable State)
- **Symptômes :** singletons modifiables, variables de module réassignées, caches statiques.
- **Risques :** couplage caché, tests dépendants de l'ordre, bugs de concurrence.
- **Remédiation :** injection de dépendances, état encapsulé et immuable.
- **Règle propriétaire :** `CS-045`, `CS-070`.
- **Détection :** manuelle / statique.

#### AP-109 — Chirurgie au fusil (Shotgun Surgery)
- **Symptômes :** un changement fonctionnel simple impose de modifier de nombreux fichiers dispersés.
- **Remédiation :** regrouper la connaissance dispersée dans un module cohésif.
- **Règle propriétaire :** `CS-030`.
- **Détection :** analyse du *co-change* Git.

#### AP-110 — Jalousie de fonctionnalité (Feature Envy)
- **Symptômes :** méthode qui manipule surtout les données d'un autre objet.
- **Remédiation :** déplacer le comportement vers les données.
- **Règle propriétaire :** aucune — diagnostic seul.
- **Détection :** manuelle.

#### AP-111 — Optimisation prématurée (Premature Optimization)
- **Symptômes :** code complexifié pour la performance sans mesure ni exigence.
- **Problème réel quand :** aucune exigence `NFR-PERF-*` ni profilage ne la justifie.
- **Remédiation :** code simple d'abord ; optimiser sur mesure, avec benchmark avant/après ([testing.md](testing.md)).
- **Règle propriétaire :** principe KISS/YAGNI, `code-style.md § 6`.
- **Détection :** manuelle.

#### AP-112 — Généralisation spéculative (Speculative Generality)
- **Symptômes :** interfaces à une seule implémentation « pour plus tard », paramètres jamais utilisés, frameworks internes.
- **Remédiation :** supprimer ; réintroduire quand un second besoin réel apparaît.
- **Règle propriétaire :** YAGNI, `code-style.md § 6`.
- **Détection :** manuelle / code mort (statique).

#### AP-113 — Typage échappatoire (Stringly / Any-typed)
- **Symptômes :** `any`, `Object`, `Map<String, Object>` traversant le domaine ; casts multiples.
- **Remédiation :** typer à la frontière, valider, convertir.
- **Règle propriétaire :** `CS-050`, `CS-053`.
- **Détection :** statique.

---

## 2. Architecture

#### AP-201 — Grosse boule de boue (Big Ball of Mud)
- **Symptômes :** pas de frontières lisibles, dépendances dans tous les sens, aucun module remplaçable.
- **Risques :** coût de changement exponentiel.
- **Remédiation :** expliciter les frontières dans [docs/architecture.md](../docs/architecture.md), les faire respecter par des tests d'architecture, migrer progressivement (*strangler fig*).
- **Règle propriétaire :** ADR du projet ; `CS-030`.
- **Détection :** statique (graphe de dépendances, outils de règles d'architecture).

#### AP-202 — Dépendances cycliques (Cyclic Dependencies)
- **Symptômes :** A dépend de B qui dépend de A (modules, paquets ou services).
- **Remédiation :** inversion de dépendance, extraction d'un module commun, événements.
- **Règle propriétaire :** `CS-031` ; règles de dépendance de `docs/architecture.md`.
- **Détection :** statique.

#### AP-203 — Monolithe distribué (Distributed Monolith)
- **Symptômes :** services déployés ensemble, appels synchrones en chaîne, base partagée, versions couplées.
- **Risques :** complexité des microservices sans leurs bénéfices ; pannes en cascade.
- **Remédiation :** revenir à un monolithe modulaire ou découper selon les contextes métier ; communication asynchrone ; données possédées par un seul service.
- **Règle propriétaire :** ADR du projet.
- **Détection :** analyse des déploiements et traces distribuées.

#### AP-204 — Base de données partagée entre services (Shared Database Integration)
- **Symptômes :** plusieurs services écrivent dans les mêmes tables.
- **Remédiation :** un propriétaire par donnée ; intégration par API ou événements.
- **Règle propriétaire :** ADR du projet.
- **Détection :** revue d'architecture, droits d'accès en base.

#### AP-205 — Abstraction qui fuit (Leaky Abstraction)
- **Symptômes :** types de l'ORM, du SDK cloud ou du fournisseur exposés dans le domaine ou les API publiques.
- **Remédiation :** ports et adaptateurs ; mapping aux frontières.
- **Règle propriétaire :** règles de couches de `docs/architecture.md`.
- **Détection :** statique (imports interdits par couche).

#### AP-206 — Marteau doré (Golden Hammer)
- **Symptômes :** la même technologie imposée à tous les problèmes, sans évaluation.
- **Remédiation :** ADR avec alternatives comparées.
- **Règle propriétaire :** processus ADR de `docs/architecture.md`.
- **Détection :** revue d'architecture.

#### AP-207 — Couplage fournisseur non maîtrisé (Unmanaged Vendor Lock-in)
- **Symptômes :** API propriétaires appelées partout ; aucune stratégie de sortie.
- **Problème réel quand :** le coût de sortie n'a pas été accepté explicitement.
- **Remédiation :** décider consciemment (ADR), isoler derrière des interfaces là où le risque le justifie.
- **Règle propriétaire :** ADR du projet.
- **Détection :** revue d'architecture.

#### AP-208 — Point de défaillance unique (Single Point of Failure)
- **Symptômes :** un composant non redondé dont la panne arrête le service, sans exigence l'acceptant.
- **Remédiation :** redondance, dégradation gracieuse, ou acceptation explicite liée à `NFR-REL-*`.
- **Règle propriétaire :** `NFR-REL-*` de `docs/requirements.md`.
- **Détection :** revue d'architecture, tests de résilience.

---

## 3. Données

#### AP-301 — Requêtes N+1 (N+1 Queries)
- **Symptômes :** une requête par élément d'une liste ; nombre de requêtes proportionnel aux données.
- **Risques :** latence et charge base qui explosent avec le volume.
- **Remédiation :** chargement groupé (*join*, `IN`, *batch loader*, *eager loading* ciblé).
- **Règle propriétaire :** `NFR-PERF-*` ; méthode de mesure dans [testing.md](testing.md).
- **Détection :** runtime (compteur de requêtes par transaction, traces), tests.

```text
// ❌ N+1
for order in orders: order.customer = db.query("... WHERE id = ?", order.customerId)
// ✅ Chargement groupé
customers = db.query("... WHERE id IN (?)", orders.map(o => o.customerId))
```

#### AP-302 — Requête non bornée (Unbounded Query / Result Set)
- **Symptômes :** `SELECT *` sans `LIMIT`, export complet en mémoire, liste API non paginée.
- **Remédiation :** pagination, *streaming*, projection des colonnes utiles.
- **Règle propriétaire :** pagination : `api-conventions.md` ; capacité : `NFR-CAP-*`.
- **Détection :** statique / revue / runtime.

#### AP-303 — Flottant pour l'argent (Floating-Point Money)
- **Symptômes :** `float`/`double` pour des montants.
- **Risques :** erreurs d'arrondi, écarts comptables.
- **Remédiation :** décimal ou entier en plus petite unité + devise ISO 4217.
- **Règle propriétaire :** `CS-054`.
- **Détection :** statique / revue.

#### AP-304 — Dates sans fuseau (Naive Datetime)
- **Symptômes :** heures locales stockées sans fuseau ; comparaisons entre fuseaux.
- **Remédiation :** stocker en UTC (ou avec offset), convertir à l'affichage.
- **Règle propriétaire :** `CS-054`, `api-conventions.md` (format des dates).
- **Détection :** revue.

#### AP-305 — Migration destructive en une étape (Big-Bang Migration)
- **Symptômes :** renommage/suppression de colonne dans le même déploiement que le code qui l'utilise.
- **Risques :** indisponibilité, rollback impossible.
- **Remédiation :** *expand / migrate / contract*.
- **Règle propriétaire :** `release-conventions.md` (migrations).
- **Détection :** revue des migrations, linter de migrations.

#### AP-306 — Absence d'index sur les accès fréquents (Missing Index)
- **Symptômes :** *full scan* sur des requêtes fréquentes ; latence croissant avec le volume.
- **Remédiation :** index alignés sur les requêtes réelles ; vérification via plan d'exécution.
- **Règle propriétaire :** `NFR-PERF-*`.
- **Détection :** runtime (requêtes lentes), plan d'exécution.

#### AP-307 — Données sensibles en clair (Plaintext Sensitive Data)
- **Règle propriétaire :** `security-conventions.md` (protection des données, secrets). Entrée de diagnostic uniquement.

---

## 4. API

#### AP-401 — API bavarde (Chatty API)
- **Symptômes :** un écran ou un cas d'usage nécessite des dizaines d'appels séquentiels.
- **Remédiation :** ressources composites, *backend for frontend*, requêtes groupées.
- **Règle propriétaire :** `api-conventions.md`.
- **Détection :** traces, revue de contrat.

#### AP-402 — Rupture de contrat silencieuse (Silent Breaking Change)
- **Symptômes :** champ renommé/supprimé, type changé, statut modifié sans nouvelle version.
- **Remédiation :** évolution additive, versionnement, dépréciation annoncée.
- **Règle propriétaire :** `api-conventions.md` (compatibilité).
- **Détection :** statique (diff de contrat en CI).

#### AP-403 — Erreur déguisée en succès (200 OK with Error Body)
- **Symptômes :** statut de succès avec `{ "error": … }`.
- **Remédiation :** statut approprié + format d'erreur standard.
- **Règle propriétaire :** `api-conventions.md` (statuts, erreurs).
- **Détection :** tests de contrat.

#### AP-404 — Exposition du modèle interne (Anemic DTO = Entity)
- **Symptômes :** entités de persistance sérialisées telles quelles ; champs internes ou sensibles exposés.
- **Remédiation :** DTO / schémas de contrat dédiés.
- **Règle propriétaire :** `api-conventions.md` ; exposition de données : `security-conventions.md`.
- **Détection :** revue.

#### AP-405 — Opérations non idempotentes rejouées (Unsafe Retry)
- **Symptômes :** nouvelle tentative automatique sur un `POST` de paiement ou de création → doublons.
- **Remédiation :** clé d'idempotence, retries limités aux opérations idempotentes.
- **Règle propriétaire :** `api-conventions.md` (idempotence, retries).
- **Détection :** revue, tests d'erreur.

---

## 5. Tests

#### AP-501 — Test instable toléré (Flaky Test)
- **Symptômes :** test qui échoue de façon intermittente ; relances manuelles « jusqu'au vert ».
- **Risques :** perte de confiance, vrais défauts masqués.
- **Remédiation :** quarantaine tracée, correction de la cause (temps, ordre, concurrence, réseau).
- **Règle propriétaire :** `testing.md` (tests instables).
- **Détection :** historique CI.

#### AP-502 — Cornet de glace (Ice-Cream Cone)
- **Symptômes :** majorité de tests E2E/manuels, peu de tests unitaires.
- **Remédiation :** rééquilibrer selon la stratégie de test.
- **Règle propriétaire :** `testing.md` (stratégie).
- **Détection :** métriques de suite de tests.

#### AP-503 — Test des détails d'implémentation
- **Symptômes :** tests cassés par chaque refactorisation sans changement de comportement ; assertions sur méthodes privées ou nombre d'appels internes.
- **Remédiation :** tester le comportement observable via l'API publique.
- **Règle propriétaire :** `testing.md`.
- **Détection :** revue.

#### AP-504 — Sur-simulation (Mock Overuse)
- **Symptômes :** tests qui ne vérifient que des mocks ; mocks de types tiers non possédés.
- **Remédiation :** simuler aux frontières, *fakes* en mémoire, tests d'intégration ciblés, tests de contrat.
- **Règle propriétaire :** `testing.md` (doublures de test).
- **Détection :** revue.

#### AP-505 — Test sans assertion significative
- **Symptômes :** test qui exécute du code sans vérifier de résultat, ou assertion triviale (`assert true`, `not null`).
- **Remédiation :** assertions sur le résultat métier attendu ; *mutation testing* pour révéler ces tests.
- **Règle propriétaire :** `testing.md` (qualité des assertions).
- **Détection :** revue, mutation testing.

#### AP-506 — Tests dépendants de l'ordre ou de l'environnement
- **Symptômes :** passent seuls mais échouent en suite (ou l'inverse) ; dépendent de l'heure, du fuseau, d'un réseau réel.
- **Remédiation :** isolation des données, horloge et aléa injectés, exécution en ordre aléatoire.
- **Règle propriétaire :** `testing.md` (indépendance, reproductibilité).
- **Détection :** exécution aléatoire / parallèle.

#### AP-507 — Affaiblir un test pour le faire passer
- **Symptômes :** assertion supprimée, seuil relâché, test désactivé, sans justification ni ticket.
- **Risques :** contournement des contrôles (principe de non-contournement).
- **Remédiation :** corriger le code, ou documenter et faire approuver le changement d'attente.
- **Règle propriétaire :** `testing.md` ; `WF` (conduite permanente) ; `GOV-020`.
- **Détection :** revue du diff des tests.

---

## 6. Exploitation et livraison

#### AP-601 — Serveur flocon de neige (Snowflake Server)
- **Symptômes :** configuration manuelle non reproductible ; « ne pas toucher à ce serveur ».
- **Remédiation :** Infrastructure as Code, immutabilité, reconstruction automatisée.
- **Règle propriétaire :** `release-conventions.md` ; mise en œuvre : `agents/devops-engineer.md`.
- **Détection :** détection de dérive (*drift*).

#### AP-602 — Déploiement manuel non reproductible
- **Symptômes :** procédure de livraison dans la tête d'une personne ; artefacts reconstruits par environnement.
- **Remédiation :** pipeline automatisé, *build once, deploy many*.
- **Règle propriétaire :** `release-conventions.md`.
- **Détection :** revue du processus.

#### AP-603 — Fatigue d'alerte (Alert Fatigue)
- **Symptômes :** alertes nombreuses, non actionnables, ignorées.
- **Remédiation :** alerter sur les symptômes utilisateurs et les budgets d'erreur, avec runbook.
- **Règle propriétaire :** `observability.md` (alerting).
- **Détection :** statistiques d'alertes (taux d'acquittement sans action).

#### AP-604 — Journalisation de données sensibles
- **Règle propriétaire :** `observability.md` (protection de la télémétrie) et `security-conventions.md`. Diagnostic : rechercher tokens, mots de passe, données personnelles dans les logs.

#### AP-605 — Sauvegarde jamais restaurée (Schrödinger Backup)
- **Symptômes :** sauvegardes réussies mais aucun test de restauration.
- **Remédiation :** tests de restauration périodiques avec preuves.
- **Règle propriétaire :** `backup-conventions.md`.
- **Détection :** absence de preuve de restauration récente.

#### AP-606 — Déploiement sans retour arrière
- **Symptômes :** aucune procédure de rollback, migrations irréversibles, pas de critères de déclenchement.
- **Règle propriétaire :** `release-conventions.md` (rollback).
- **Détection :** revue de plan de release.

---

## 7. Concurrence et systèmes distribués

#### AP-701 — Vérifier-puis-agir non atomique (Check-Then-Act Race)
- **Symptômes :** `if (!exists) create()` sans verrou ni contrainte d'unicité.
- **Remédiation :** opérations atomiques, contraintes en base, verrouillage optimiste (version / ETag).
- **Règle propriétaire :** `CS-070`.
- **Détection :** revue, tests de concurrence.

#### AP-702 — Écriture double sans garantie (Dual Write)
- **Symptômes :** écrire en base **et** publier un message dans deux opérations indépendantes.
- **Risques :** incohérence en cas d'échec partiel.
- **Remédiation :** *transactional outbox*, *change data capture*, consommateurs idempotents.
- **Règle propriétaire :** ADR du projet.
- **Détection :** revue d'architecture.

#### AP-703 — Tempête de nouvelles tentatives (Retry Storm)
- **Symptômes :** retries immédiats et illimités à chaque niveau d'appel.
- **Remédiation :** backoff exponentiel avec *jitter*, budget de retries, *circuit breaker*.
- **Règle propriétaire :** `api-conventions.md` (retries).
- **Détection :** métriques de trafic lors d'incidents.

#### AP-704 — Supposer une livraison exactement-une-fois
- **Symptômes :** consommateur de messages non idempotent.
- **Remédiation :** idempotence côté consommateur (clé de déduplication).
- **Règle propriétaire :** `api-conventions.md` (événements).
- **Détection :** revue, tests de rejeu.

---

## 8. Composants d'IA (si applicable)

#### AP-801 — Confiance aveugle dans la sortie d'un modèle
- **Symptômes :** réponse d'un LLM exécutée, insérée en base, rendue en HTML ou utilisée dans une requête sans validation.
- **Risques :** injection, actions non autorisées, données erronées.
- **Remédiation :** traiter la sortie comme une **entrée non fiable** : schéma structuré, validation, encodage, autorisation de l'action côté serveur.
- **Règle propriétaire :** `security-conventions.md` (IA), `CS-053`.
- **Détection :** revue, tests adverses.

#### AP-802 — Instructions et données mélangées (Prompt Injection Exposure)
- **Symptômes :** contenu utilisateur ou externe concaténé dans le prompt système ; l'agent obéit à du texte trouvé dans un document.
- **Règle propriétaire :** `security-conventions.md` (IA) ; `sandbox.md` (agents).
- **Détection :** tests adverses, red teaming.

#### AP-803 — Absence d'évaluation (Vibe-Based Validation)
- **Symptômes :** changement de prompt ou de modèle validé « à l'œil » sur quelques exemples.
- **Remédiation :** jeux d'évaluation versionnés, métriques, seuils de non-régression.
- **Règle propriétaire :** `testing.md` (composants d'IA).
- **Détection :** absence d'artefact d'évaluation dans la PR.

#### AP-804 — Modèle non épinglé
- **Symptômes :** alias de modèle « latest » en production ; comportement qui change sans déploiement.
- **Remédiation :** version de modèle explicite, changement de version traité comme une release.
- **Règle propriétaire :** `release-conventions.md`.
- **Détection :** revue de configuration.

---

## 9. Anti-patterns spécifiques au projet

<!-- PERSONNALISER : alimenter à partir des post-mortems et des revues récurrentes. -->

#### AP-901 — `{{NOM}}`
- **Symptômes :** `{{…}}`
- **Problème réel quand :** `{{…}}`
- **Risques :** `{{…}}`
- **Remédiation :** `{{…}}`
- **Règle propriétaire :** `{{ID}}`
- **Détection :** `{{…}}`

## 10. Exceptions et compromis documentés

Un anti-pattern PEUT être accepté s'il résulte d'un compromis conscient (prototype jetable, contrainte de performance mesurée, dette planifiée). Dans ce cas :

- s'il viole une règle **DOIT** → dérogation formelle ([README.md § 6](README.md)) ;
- sinon → justification dans le code (commentaire `CS-090`) ou dans une ADR, et ticket de dette si une correction est prévue.

| Anti-pattern | Emplacement | Justification | Référence (ADR / dérogation / ticket) | Revue prévue |
|---|---|---|---|---|
| `{{AP-XXX}}` | `{{chemin}}` | `{{…}}` | `{{…}}` | `{{date}}` |
