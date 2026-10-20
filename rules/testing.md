---
id: "RUL-TST"
title: "Stratégie et méthodes de test"
purpose: "Définir la méthode de vérification logicielle et la qualité attendue des tests."
scope: "Tous les tests automatisés et manuels du projet, de l'unitaire aux tests non fonctionnels, y compris l'évaluation des composants d'IA."
out_of_scope:
  - "Exigences métier et seuils produit (docs/requirements.md)"
  - "Critères bloquants de livraison (rules/release-conventions.md)"
  - "Contrôles de sécurité obligatoires (rules/security-conventions.md)"
references:
  - "README.md"
  - "code-style.md"
  - "api-conventions.md"
  - "security-conventions.md"
  - "release-conventions.md"
  - "../docs/requirements.md"
owner: "{{QA_LEAD / TECH_LEAD}}"
status: "template"
version: "0.1.0"
---

# Stratégie et méthodes de test

> **Principe directeur :** un test sert à **donner confiance dans un comportement** et à **détecter rapidement une régression**. Un bon test est rapide, déterministe, isolé, lisible et échoue pour une seule raison compréhensible.
>
> **Distinction :** ce fichier dit **comment vérifier**. [release-conventions.md](release-conventions.md) dit **quelles vérifications bloquent une livraison**. [requirements.md](../docs/requirements.md) dit **ce qui doit être atteint**.

<!-- PERSONNALISER : outillage (§ 2), répartition (§ 3), seuils de couverture (§ 13) et sections non fonctionnelles pertinentes (§ 10). -->

---

## 1. Stratégie fondée sur les risques

#### TST-001 — Effort proportionné au risque
- **Niveau :** DOIT
- **Énoncé :** la profondeur des tests dépend de la criticité de l'exigence (`docs/requirements.md`), de la probabilité de défaut (complexité, nouveauté, historique) et de l'impact d'une défaillance (financier, sécurité, données, réputation).

| Criticité de la zone | Exemples | Attendu minimal |
|---|---|---|
| **Critique** | Paiement, authentification, autorisation, calculs réglementaires, migrations de données | Unitaires exhaustifs (cas limites, négatifs), intégration, E2E du parcours, tests de propriétés `{{ou mutation}}`, revue des tests |
| **Élevée** | Règles métier centrales, intégrations externes | Unitaires + intégration + contrat |
| **Standard** | Fonctionnalités courantes | Unitaires + intégration ciblée |
| **Faible** | Outils internes, code d'affichage simple | Unitaires sur la logique, tests de fumée |

#### TST-002 — Toute exigence a un moyen de vérification
- **Niveau :** DOIT
- **Énoncé :** chaque `FR-*` et `NFR-*` approuvé est relié à au moins un test ou une méthode de vérification dans la matrice de traçabilité de [requirements.md](../docs/requirements.md). Les tests référencent l'identifiant (nom, tag ou annotation : `{{EX: @req FR-012}}`).

## 2. Outillage

| Niveau | Outil / framework | Commande | Emplacement |
|---|---|---|---|
| Unitaire | `{{…}}` | `{{…}}` | `{{EX: à côté du code / tests/unit}}` |
| Intégration | `{{EX: Testcontainers}}` | `{{…}}` | `{{…}}` |
| Contrat | `{{EX: Pact, Schemathesis, validation OpenAPI}}` | `{{…}}` | `{{…}}` |
| E2E / UI | `{{EX: Playwright, Cypress, Espresso, XCUITest}}` | `{{…}}` | `{{…}}` |
| Performance | `{{EX: k6, Gatling, JMeter, Locust}}` | `{{…}}` | `{{…}}` |
| Accessibilité | `{{EX: axe-core}}` + tests manuels | `{{…}}` | `{{…}}` |
| Mutation | `{{EX: Stryker, PIT, mutmut}}` | `{{…}}` | — |
| Propriétés | `{{EX: fast-check, Hypothesis, jqwik}}` | — | — |
| Évaluation IA | `{{…}}` | `{{…}}` | `{{…}}` |

Les commandes concrètes sont également listées dans [AGENTS.md](../AGENTS.md).

## 3. Répartition des tests

#### TST-010 — Forme de la suite
- **Niveau :** DEVRAIT
- **Énoncé :** la majorité des vérifications se situe aux niveaux les plus rapides capables de détecter le défaut visé. Répartition indicative : `{{EX: pyramide — ~70 % unitaires, ~20 % intégration/contrat, ~10 % E2E}}` ou `{{« trophée » : accent sur l'intégration pour les applications front-end}}`. Éviter le « cornet de glace » (`AP-502`).

#### TST-011 — Budget de temps
- **Niveau :** DEVRAIT
- **Énoncé :** suite exécutée sur chaque PR en moins de `{{EX: 10 min}}` ; tests unitaires locaux en moins de `{{EX: 1 min}}`. Les suites longues (performance, E2E complets) tournent sur un rythme dédié (§ 15).

## 4. Tests unitaires

#### TST-020 — Définition
- **Niveau :** DOIT
- **Énoncé :** un test unitaire vérifie une unité de comportement sans E/S réelle (réseau, disque, base, horloge système, aléa non maîtrisé). Il s'exécute en millisecondes.

#### TST-021 — Structure
- **Niveau :** DOIT
- **Énoncé :** structure **Arrange / Act / Assert** (ou *Given / When / Then*) visible ; un comportement par test ; pas de logique conditionnelle ni de boucle dans le test.

#### TST-022 — Nommage
- **Niveau :** DOIT
- **Énoncé :** le nom décrit le comportement attendu et la condition : `{{EX: should_reject_transfer_when_balance_is_insufficient}}` ou `{{describe/it en langage naturel}}`. Un échec doit être compréhensible **sans lire le code du test**.

#### TST-023 — Tester le comportement, pas l'implémentation
- **Niveau :** DOIT
- **Énoncé :** les tests passent par l'API publique de l'unité ; ils ne dépendent pas de méthodes privées ni de l'ordre d'appels internes (voir `AP-503`).

## 5. Tests d'intégration

#### TST-030 — Dépendances réelles ou fidèles
- **Niveau :** DOIT
- **Énoncé :** les tests d'intégration vérifient l'interaction avec une dépendance réelle ou une émulation fidèle (base de données du même moteur et de la même version majeure via conteneurs, file de messages, stockage). Une base en mémoire d'un **autre** moteur n'est pas une preuve d'intégration.

#### TST-031 — Isolation des données
- **Niveau :** DOIT
- **Énoncé :** chaque test crée ses propres données et ne dépend pas de celles d'un autre test (transaction annulée, schéma dédié, identifiants uniques).

#### TST-032 — Services tiers
- **Niveau :** DOIT
- **Énoncé :** les services tiers externes sont simulés au niveau HTTP (`{{EX: WireMock, MSW}}`) ou via leur bac à sable officiel ; aucun test automatisé ne dépend d'un service de production tiers.

## 6. Tests de contrat et d'interface

#### TST-040 — Contrats d'API vérifiés
- **Niveau :** DOIT
- **Énoncé :** les réponses du fournisseur sont validées contre le contrat faisant foi ([api-conventions.md](api-conventions.md), `API-110`). Pour les intégrations internes entre équipes, des tests de contrat pilotés par le consommateur (*consumer-driven*) sont RECOMMANDÉS.

#### TST-041 — Tests de propriétés sur les entrées
- **Niveau :** DEVRAIT
- **Énoncé :** les API exposées sont soumises à des tests générés à partir du schéma (*fuzzing* contractuel) pour détecter les erreurs `5xx` et les violations de contrat.

## 7. Tests fonctionnels et de bout en bout

#### TST-050 — Parcours critiques uniquement
- **Niveau :** DOIT
- **Énoncé :** les tests E2E couvrent les parcours utilisateurs critiques (`UC-*` prioritaires) et non toutes les combinaisons ; les variations sont testées aux niveaux inférieurs.

#### TST-051 — Robustesse des tests E2E
- **Niveau :** DOIT
- **Énoncé :** sélecteurs stables (rôles d'accessibilité, `data-testid`), attentes explicites sur des conditions (jamais de `sleep` fixe), données préparées par API plutôt que par l'interface, captures et traces conservées en cas d'échec.

#### TST-052 — Critères d'acceptation exécutables
- **Niveau :** DEVRAIT
- **Énoncé :** les critères d'acceptation rédigés en *Given/When/Then* dans `requirements.md` sont automatisés lorsque c'est rentable.

## 8. Tests de régression

#### TST-060 — Un défaut, un test
- **Niveau :** DOIT
- **Énoncé :** toute correction de défaut s'accompagne d'un test qui **échoue avant** la correction et **passe après**, au niveau le plus bas reproduisant le défaut. Le test référence le ticket.

#### TST-061 — Suite de régression complète
- **Niveau :** DOIT
- **Énoncé :** la totalité des tests automatisés pertinents est exécutée avant fusion sur la branche principale.

## 9. Cas limites et scénarios négatifs

#### TST-070 — Couverture des chemins d'échec
- **Niveau :** DOIT
- **Énoncé :** pour toute logique non triviale, les tests couvrent, selon pertinence :
  - valeurs limites (min, max, min−1, max+1, zéro, vide, un élément, très grand) ;
  - entrées invalides, nulles/absentes, mal formées, de mauvais type ;
  - Unicode, emojis, caractères de contrôle, textes longs, langues RTL ;
  - fuseaux horaires, changements d'heure, années bissextiles, fin de mois ;
  - erreurs des dépendances (délai, `5xx`, réponse partielle, réponse invalide) ;
  - concurrence (double soumission, mises à jour simultanées) ;
  - droits insuffisants et accès à l'objet d'autrui (tests négatifs d'autorisation, `SEC-051`).

#### TST-071 — Tests basés sur les propriétés
- **Niveau :** DEVRAIT pour la logique algorithmique ou de transformation
- **Énoncé :** utiliser des tests de propriétés (invariants, aller-retour sérialisation/désérialisation, idempotence) pour explorer l'espace des entrées.

## 10. Tests non fonctionnels

Les **cibles** sont définies dans `docs/requirements.md` (`NFR-*`). Ce fichier définit uniquement le **protocole**.

#### TST-080 — Performance et charge
- **Niveau :** DOIT si une exigence `NFR-PERF-*` ou `NFR-CAP-*` existe
- **Protocole :**
  1. Référencer l'exigence visée et son scénario (profil de charge, mix d'opérations, volume de données).
  2. Environnement représentatif et documenté (taille, version, données) ; jamais sur un poste de développeur pour conclure.
  3. Phase de montée en charge et de chauffe exclue des mesures.
  4. Mesurer des **percentiles** (p50, p95, p99) et le taux d'erreur, jamais la seule moyenne ; durée suffisante pour la stabilité statistique.
  5. Types de tests selon besoin : charge nominale, stress (point de rupture), endurance (fuites), pic.
  6. Comparer à une référence (*baseline*) ; conserver résultats bruts et configuration.
- **Tests de micro-performance :** les *benchmarks* de code utilisent un outil dédié (`{{EX: JMH, BenchmarkDotNet, pytest-benchmark}}`).

#### TST-081 — Sécurité
- **Niveau :** DOIT
- **Énoncé :** les tests de sécurité requis (SAST, SCA, DAST, tests négatifs d'autorisation…) sont définis par [security-conventions.md § 16](security-conventions.md). Ce fichier n'en fixe pas la liste.

#### TST-082 — Accessibilité
- **Niveau :** DOIT pour toute interface utilisateur si `NFR-ACC-*` existe
- **Énoncé :** contrôles automatisés (`{{EX: axe}}`) sur les pages/écrans clés **et** vérifications manuelles (navigation clavier complète, ordre de focus, lecteur d'écran `{{EX: NVDA, VoiceOver, TalkBack}}`, zoom 200–400 %, contrastes, réduction des animations). Les outils automatisés ne détectent qu'une partie des problèmes ; le contrôle manuel n'est pas optionnel.

#### TST-083 — Résilience
- **Niveau :** DEVRAIT pour les services avec `NFR-REL-*`
- **Énoncé :** injection de fautes (latence, indisponibilité d'une dépendance, perte d'instance, saturation) dans un environnement contrôlé, avec hypothèse formulée, rayon d'impact limité et critère d'arrêt. Vérifie délais, retries, disjoncteurs et dégradation gracieuse.

#### TST-084 — Compatibilité
- **Niveau :** DOIT si `NFR-COMP-*` existe
- **Énoncé :** matrice de navigateurs / OS / appareils / versions d'API définie dans les exigences, testée selon `{{EX: dernière version + N-1}}`.

#### TST-085 — Migrations et restauration
- **Niveau :** DOIT
- **Énoncé :** les migrations de données sont testées sur un jeu représentatif (aller, et retour si prévu) ; les tests de restauration de sauvegardes suivent [backup-conventions.md](backup-conventions.md).

## 11. Données de test, doublures et reproductibilité

#### TST-090 — Données synthétiques
- **Niveau :** DOIT
- **Énoncé :** les tests utilisent des données synthétiques ou anonymisées de façon irréversible ; jamais de données personnelles réelles (`SEC-093`).

#### TST-091 — Fabriques plutôt que gros jeux partagés
- **Niveau :** DEVRAIT
- **Énoncé :** données construites par des fabriques/*builders* avec valeurs par défaut valides, en ne précisant que ce qui compte pour le test.

#### TST-092 — Doublures de test
- **Niveau :** DOIT
- **Énoncé :**
  - simuler aux **frontières** du système (réseau, horloge, aléa, services externes), pas les collaborateurs internes du domaine ;
  - préférer *fakes* en mémoire et *stubs* aux *mocks* vérifiant des appels ;
  - ne pas simuler directement un type tiers non possédé : l'envelopper dans un adaptateur, testé par un test d'intégration ;
  - une doublure doit respecter le contrat de l'original.

#### TST-093 — Déterminisme
- **Niveau :** DOIT
- **Énoncé :** horloge, fuseau, locale, aléa (graine fixée et journalisée), ordre des collections et identifiants générés sont contrôlés. Aucun test ne dépend du réseau public.

## 12. Organisation et indépendance

#### TST-100 — Indépendance
- **Niveau :** DOIT
- **Énoncé :** chaque test peut s'exécuter seul, dans n'importe quel ordre et en parallèle. L'exécution en ordre aléatoire est activée `{{si l'outil le permet}}`.

#### TST-101 — Le code de test est du code
- **Niveau :** DOIT
- **Énoncé :** les tests respectent [code-style.md](code-style.md) (lisibilité, nommage, pas de duplication de connaissance) avec une tolérance pour la répétition explicite qui améliore la lisibilité.

#### TST-102 — Organisation
- **Niveau :** DOIT
- **Énoncé :** emplacement, suffixes (`{{EX: *.test.ts, test_*.py, *Test.java}}`) et tags (`unit`, `integration`, `e2e`, `slow`) conformes au § 2, afin de permettre une exécution sélective.

## 13. Couverture et qualité des assertions

#### TST-110 — La couverture est un indicateur, pas un objectif
- **Niveau :** DOIT
- **Énoncé :** la couverture mesure le code **non** testé ; un pourcentage élevé ne prouve pas la qualité. Le code nouveau ou modifié respecte les seuils suivants (sauf code généré et exclusions justifiées) :

| Métrique | Seuil sur le code modifié | Seuil global (non-régression) |
|---|---|---|
| Lignes | `{{EX: 80 %}}` | `{{EX: ne pas baisser}}` |
| Branches | `{{EX: 70 %}}` | `{{EX: ne pas baisser}}` |
| Score de mutation (zones critiques) | `{{EX: 60 %}}` | — |

<!-- Si la couverture est une exigence contractuelle du produit, la valeur est dans docs/requirements.md et ce tableau la référence (GOV-003). -->

#### TST-111 — Assertions significatives
- **Niveau :** DOIT
- **Énoncé :** chaque test vérifie un résultat observable précis (valeur, état, événement émis, erreur typée), pas seulement l'absence d'exception (`AP-505`). Les *snapshots* sont réservés aux sorties stables et revus à chaque mise à jour.

## 14. Tests instables et non-contournement

#### TST-120 — Traitement des tests instables
- **Niveau :** DOIT
- **Énoncé :** un test instable est, dans les `{{EX: 24 h}}` : corrigé, ou mis en **quarantaine** (exclu des contrôles bloquants) avec ticket, propriétaire et échéance `{{EX: 14 jours}}`. La relance automatique sans traçage est interdite ; le taux d'instabilité est suivi.

#### TST-121 — Ne jamais affaiblir un test pour obtenir un résultat favorable
- **Niveau :** DOIT
- **Énoncé :** supprimer une assertion, élargir une tolérance, désactiver ou marquer `skip` un test, ou modifier une valeur attendue n'est permis que si l'attente était **fausse** ; la justification figure dans la PR et la modification est revue explicitement. Sinon, c'est le code qui est corrigé (`AP-507`).

## 15. Rythme d'exécution

| Moment | Suites | Bloquant |
|---|---|---|
| Local / pre-commit | Unitaires rapides du périmètre modifié | — |
| Chaque PR | Unitaires, intégration, contrat, lint, E2E de fumée | Voir [release-conventions.md](release-conventions.md) |
| Fusion sur branche principale | Suite complète | Idem |
| Planifié `{{EX: nuit}}` | E2E complets, mutation, compatibilité | Alerte |
| Avant release | Performance, résilience, accessibilité manuelle `{{selon risque}}` | Idem |

## 16. Rapports, preuves et classification des échecs

#### TST-130 — Rapports exploitables
- **Niveau :** DOIT
- **Énoncé :** chaque exécution CI produit un rapport lisible par machine (`{{EX: JUnit XML}}`), le rapport de couverture et, pour les échecs, journaux, captures et traces. Les rapports sont conservés `{{DURÉE}}` et rattachés au commit.

#### TST-131 — Preuve d'exécution
- **Niveau :** DOIT
- **Énoncé :** un test est déclaré réussi uniquement s'il a été **exécuté** et que le résultat est disponible. « Devrait passer », « non exécuté localement » ou « compile » ne sont pas des preuves (voir statuts de preuve dans [agents/agent-workflow.md](../agents/agent-workflow.md)).

#### TST-132 — Classification des échecs

| Catégorie | Description | Action |
|---|---|---|
| **Défaut produit** | Le code ne respecte pas le comportement attendu | Corriger le code |
| **Défaut de test** | Le test est faux ou obsolète | Corriger le test (`TST-121`) |
| **Environnement** | Infrastructure, réseau, ressources | Corriger l'environnement ; ne pas masquer |
| **Données** | Données de test invalides ou partagées | Isoler / corriger |
| **Instable** | Résultat non déterministe | `TST-120` |

## 17. Composants d'IA (si applicable)

#### TST-140 — Évaluations versionnées
- **Niveau :** DOIT
- **Énoncé :** tout comportement dépendant d'un modèle (prompt, modèle, paramètres, outils, RAG) est évalué sur un jeu d'évaluation versionné, représentatif et incluant des cas adverses (injection, hors-sujet, données sensibles).

#### TST-141 — Métriques et tolérance
- **Niveau :** DOIT
- **Énoncé :** métriques définies (`{{EX: exactitude, fidélité aux sources, taux de refus approprié, respect du format}}`) avec seuils de non-régression ; tolérance statistique explicite pour le non-déterminisme (plusieurs exécutions, intervalle de confiance). Les évaluations automatiques par un modèle-juge sont calibrées sur un échantillon annoté par des humains.

#### TST-142 — Sorties structurées testées de façon déterministe
- **Niveau :** DOIT
- **Énoncé :** le code qui consomme les sorties d'un modèle est testé de façon déterministe avec des réponses simulées, y compris des réponses invalides ou malveillantes.

## 18. Spécificités du projet

<!-- PERSONNALISER : identifiants TST-2NN. -->

#### TST-200 — `{{TITRE}}`
- **Niveau :** `{{…}}`
- **Énoncé :** `{{…}}`
