---
id: "RUL-OBS"
title: "Observabilité et diagnostic"
purpose: "Définir les conventions d'instrumentation, de mesure, de supervision, d'alerte et de diagnostic."
scope: "Tous les composants exécutés (services, tâches planifiées, applications clientes, pipelines de données, composants d'IA)."
out_of_scope:
  - "Fixation des objectifs SLO et seuils produit (docs/requirements.md)"
  - "Installation concrète de l'infrastructure de supervision (agents/devops-engineer.md)"
  - "Liste des événements de sécurité obligatoires (rules/security-conventions.md)"
references:
  - "README.md"
  - "security-conventions.md"
  - "api-conventions.md"
  - "../docs/requirements.md"
  - "../docs/architecture.md"
owner: "{{SRE_LEAD}}"
status: "template"
version: "0.1.0"
---

# Observabilité et diagnostic

> **Principe directeur :** on doit pouvoir répondre à **toute** question sur le comportement du système en production — y compris celles qu'on n'avait pas prévues — **sans déployer de nouveau code**. Les signaux sont corrélés, structurés, utiles et respectueux des données.
>
> **Standard d'instrumentation :** **OpenTelemetry** (API, SDK, conventions sémantiques, protocole OTLP), sauf ADR contraire.

<!-- PERSONNALISER : backend de télémétrie (§ 0), champs spécifiques (§ 1), métriques métier (§ 3), rétention (§ 11). -->

---

## 0. Pile technique

| Signal | Collecte | Stockage / backend | Visualisation |
|---|---|---|---|
| Logs | `{{EX: OTel Collector}}` | `{{…}}` | `{{…}}` |
| Métriques | `{{…}}` | `{{…}}` | `{{…}}` |
| Traces | `{{…}}` | `{{…}}` | `{{…}}` |
| Erreurs / crashs clients | `{{…}}` | `{{…}}` | `{{…}}` |
| Alerting / astreinte | — | — | `{{…}}` |

## 1. Journaux structurés

#### OBS-001 — Format structuré
- **Niveau :** DOIT
- **Énoncé :** les journaux applicatifs sont émis en **JSON** (ou modèle de log OTel), une entrée par événement, sur la sortie standard ou via le SDK ; jamais de texte libre multi-lignes non structuré en production.

#### OBS-002 — Champs obligatoires

| Champ | Description | Exemple |
|---|---|---|
| `timestamp` | RFC 3339, UTC, précision ≥ milliseconde | `2026-10-20T14:30:00.123Z` |
| `severity` | Niveau (§ 2) | `ERROR` |
| `message` | Description stable, sans donnée variable interpolée si possible | `Payment authorization failed` |
| `service.name` / `service.version` | Composant et version déployée | `checkout` / `1.4.2` |
| `deployment.environment` | Environnement | `production` |
| `trace_id` / `span_id` | Corrélation (§ 4) | `4bf92f…` |
| `event.name` | Identifiant stable de l'événement | `payment.authorization.failed` |
| `error.type` / `exception.*` | Si erreur : type, message, pile | — |
| `{{tenant.id}}` | `{{si multi-locataire}}` | — |

Les attributs suivent les **conventions sémantiques OpenTelemetry** (`http.request.method`, `db.system`…) ; les attributs propres au projet sont préfixés `{{app.}}`.

#### OBS-003 — Message stable, contexte en attributs
- **Niveau :** DEVRAIT
- **Énoncé :** les valeurs variables sont placées dans des attributs, pas concaténées dans le message, afin de permettre l'agrégation et la recherche.

#### OBS-004 — Ce qu'on ne journalise pas
- **Niveau :** DOIT
- **Énoncé :** aucun secret, jeton, mot de passe, donnée Restreinte ni donnée personnelle non nécessaire (`SEC-121`). Corps de requêtes/réponses non journalisés par défaut. Un filtre de masquage est appliqué dans le pipeline de collecte **en plus** de la discipline au niveau du code.

## 2. Niveaux de gravité

| Niveau | Usage | Exemple | En production |
|---|---|---|---|
| `TRACE` / `DEBUG` | Diagnostic détaillé de développement | Valeurs intermédiaires | Désactivé par défaut, activable dynamiquement et temporairement |
| `INFO` | Événement métier ou cycle de vie significatif | Démarrage, commande créée | Oui, avec parcimonie |
| `WARN` | Situation anormale gérée, sans impact utilisateur immédiat | Retry réussi, dépréciation utilisée | Oui |
| `ERROR` | Échec d'une opération nécessitant attention | Requête échouée de façon inattendue | Oui |
| `FATAL` | Arrêt du processus | Configuration invalide au démarrage | Oui |

#### OBS-010 — Une erreur journalisée une fois
- **Niveau :** DOIT
- **Énoncé :** une erreur est journalisée à l'endroit où elle est **traitée**, pas à chaque niveau de propagation. Une erreur attendue et gérée (validation, `404`) n'est pas un `ERROR`.

## 3. Métriques

#### OBS-020 — Méthodes de couverture
- **Niveau :** DOIT
- **Énoncé :** chaque composant expose au minimum :
  - **services orientés requêtes — méthode RED :** débit (*Rate*), erreurs (*Errors*), durée (*Duration*, en histogramme) ;
  - **ressources — méthode USE :** utilisation, saturation, erreurs (CPU, mémoire, pools de connexions, files, disques) ;
  - **files et traitements asynchrones :** profondeur, âge du plus ancien message, débit, échecs, messages en file d'erreur ;
  - **tâches planifiées :** dernière exécution réussie, durée, résultat.

#### OBS-021 — Nommage et unités
- **Niveau :** DOIT
- **Énoncé :** noms selon les conventions sémantiques OTel (`http.server.request.duration`) ; unités UCUM (`s`, `By`, `{request}`) ; préfixe projet `{{app.}}` pour les métriques propres.

#### OBS-022 — Histogrammes pour les latences
- **Niveau :** DOIT
- **Énoncé :** les durées sont enregistrées en histogrammes (permettant les percentiles), jamais en moyennes seules ; les bornes de seaux couvrent les seuils des `NFR-PERF-*`.

#### OBS-023 — Cardinalité maîtrisée
- **Niveau :** DOIT
- **Énoncé :** aucune étiquette à cardinalité non bornée sur une métrique (ID utilisateur, ID de requête, URL brute, adresse e-mail) ; utiliser la route modèle (`/orders/{id}`). Ces informations vont dans les traces ou les journaux.

#### OBS-024 — Métriques métier
- **Niveau :** DEVRAIT
- **Énoncé :** les indicateurs métier clés sont instrumentés :

| Métrique | Définition | Lien avec l'exigence |
|---|---|---|
| `{{EX: app.orders.created}}` | `{{…}}` | `{{FR-… / objectif métier}}` |
| `{{…}}` | `{{…}}` | `{{…}}` |

## 4. Traces et propagation du contexte

#### OBS-030 — Traçage distribué
- **Niveau :** DOIT pour les systèmes à plusieurs composants
- **Énoncé :** chaque requête entrante crée ou poursuit une trace ; les spans couvrent les frontières (requêtes entrantes, appels sortants HTTP/gRPC, base de données, files, appels de modèles d'IA) et les opérations métier significatives.

#### OBS-031 — Propagation standard
- **Niveau :** DOIT
- **Énoncé :** propagation du contexte via **W3C Trace Context** (`traceparent`, `tracestate`) et **W3C Baggage** pour les attributs autorisés ; propagation à travers les messages asynchrones (en-têtes de message). Le `trace_id` est renvoyé aux clients dans les erreurs (`API-040`).

#### OBS-032 — Échantillonnage
- **Niveau :** DOIT
- **Énoncé :** stratégie d'échantillonnage documentée : `{{EX: tête 10 % + conservation en queue de 100 % des erreurs et des requêtes lentes}}`. Les métriques ne sont jamais calculées à partir de traces échantillonnées.

#### OBS-033 — Pas de données sensibles dans les attributs
- **Niveau :** DOIT
- **Énoncé :** mêmes règles que `OBS-004` pour les attributs de spans et le *baggage* (ce dernier est propagé aux services tiers).

## 5. Sondes de santé

#### OBS-040 — Sondes distinctes

| Sonde | Question | Vérifie | Ne vérifie **pas** |
|---|---|---|---|
| **Liveness** (`{{/livez}}`) | Le processus doit-il être redémarré ? | Le processus répond, pas d'interblocage | Les dépendances externes (sinon redémarrages en cascade) |
| **Readiness** (`{{/readyz}}`) | Peut-il recevoir du trafic ? | Initialisation terminée, dépendances indispensables joignables | — |
| **Startup** (`{{/startupz}}`) | L'initialisation est-elle finie ? | Démarrage lent terminé | — |

#### OBS-041 — Sondes légères et protégées
- **Niveau :** DOIT
- **Énoncé :** les sondes répondent rapidement, sans effet de bord, et n'exposent aucune information sensible ; un détail de diagnostic éventuel est réservé au réseau interne authentifié.

## 6. Indicateurs de niveau de service (SLI)

#### OBS-050 — Définition par événements bons / valides
- **Niveau :** DOIT
- **Énoncé :** un SLI est un ratio `événements bons / événements valides` sur une fenêtre, mesuré **au plus près de l'utilisateur** (équilibreur, passerelle ou client), pas sur des métriques d'infrastructure.

| SLI | Événements valides | Événements bons | Source de mesure | Exigence liée |
|---|---|---|---|---|
| Disponibilité | Requêtes HTTP hors `4xx` imputables au client | Réponses non `5xx` | `{{passerelle}}` | `{{NFR-REL-001}}` |
| Latence | Requêtes réussies | Durée < seuil défini dans l'exigence | `{{…}}` | `{{NFR-PERF-001}}` |
| Fraîcheur (données) | `{{…}}` | `{{…}}` | `{{…}}` | `{{…}}` |
| Exactitude (traitement) | `{{…}}` | `{{…}}` | `{{…}}` | `{{…}}` |

Les **valeurs cibles** (ex. 99,9 %, 300 ms) ne figurent pas ici : elles sont dans [requirements.md](../docs/requirements.md) (`GOV-003`).

## 7. Objectifs de niveau de service (SLO) et budgets d'erreur

#### OBS-060 — Suivi des SLO
- **Niveau :** DOIT pour chaque SLI lié à une exigence
- **Énoncé :** chaque SLO est suivi sur une fenêtre glissante `{{EX: 28 ou 30 jours}}` avec : conformité courante, budget d'erreur restant (`1 − objectif`), vitesse de consommation (*burn rate*).

#### OBS-061 — Politique de budget d'erreur
- **Niveau :** DEVRAIT
- **Énoncé :** une politique décrit les actions lorsque le budget est épuisé (`{{EX: gel des déploiements non correctifs, priorisation de la fiabilité}}`), approuvée par `{{PRODUCT_OWNER + TECH_LEAD}}`. L'application aux releases est définie dans [release-conventions.md](release-conventions.md).

## 8. Tableaux de bord

#### OBS-070 — Conventions
- **Niveau :** DEVRAIT
- **Énoncé :**
  - tableaux de bord **versionnés en code** (*dashboards as code*) ;
  - hiérarchie : vue d'ensemble des SLO → vue par service (RED/USE) → vues détaillées de diagnostic ;
  - chaque graphique a un titre explicite, une unité, et indique le seuil cible s'il existe ;
  - lien depuis chaque alerte vers le tableau de bord et le runbook concernés ;
  - annotations des déploiements et incidents sur les graphiques.

## 9. Alertes

#### OBS-080 — Alerter sur les symptômes
- **Niveau :** DOIT
- **Énoncé :** les alertes **qui réveillent** (astreinte) portent sur des symptômes vécus par les utilisateurs ou une menace imminente sur un SLO, de préférence via des **alertes de consommation du budget d'erreur multi-fenêtres** (ex. fenêtres courtes et longues combinées). Les causes (CPU élevé…) alimentent les tableaux de bord ou des tickets, pas l'astreinte.

#### OBS-081 — Toute alerte est actionnable
- **Niveau :** DOIT
- **Énoncé :** chaque alerte possède : description de l'impact, gravité, propriétaire, lien vers le tableau de bord, **runbook** (diagnostic et actions). Une alerte sans action possible est supprimée ou déclassée (`AP-603`).

#### OBS-082 — Gravité et routage

| Gravité | Critère | Routage | Délai de prise en charge |
|---|---|---|---|
| `P1` — Critique | Service indisponible ou budget consommé très rapidement ; perte de données | Astreinte (page) | `{{EX: 15 min}}` |
| `P2` — Élevée | Dégradation significative, consommation rapide du budget | Astreinte heures ouvrées / page selon `{{…}}` | `{{EX: 1 h}}` |
| `P3` — Modérée | Risque à moyen terme | Ticket | `{{EX: 1 jour ouvré}}` |
| `P4` — Information | Tendance | Tableau de bord / rapport | — |

#### OBS-083 — Revue des alertes
- **Niveau :** DEVRAIT
- **Énoncé :** revue `{{EX: mensuelle}}` : alertes déclenchées, faux positifs, alertes manquantes révélées par les incidents.

## 10. Diagnostic et corrélation

#### OBS-090 — Navigation entre signaux
- **Niveau :** DOIT
- **Énoncé :** depuis une métrique ou une alerte, il est possible d'atteindre les traces exemplaires (*exemplars*), puis les journaux de la trace via `trace_id`. Les signaux partagent les mêmes attributs de ressource (`service.name`, `service.version`, `deployment.environment`).

#### OBS-091 — Erreurs clients et crashs
- **Niveau :** DOIT pour les applications web/mobiles
- **Énoncé :** les erreurs et crashs côté client sont collectés avec version, plateforme et symboles de débogage (*source maps*, dSYM, mapping) gérés de façon sécurisée ; les métriques d'expérience réelle (`{{EX: Core Web Vitals, temps de démarrage}}`) sont mesurées si `NFR-PERF-*` les vise.

#### OBS-092 — Post-mortems
- **Niveau :** DEVRAIT
- **Énoncé :** les incidents `P1`/`P2` donnent lieu à un post-mortem sans recherche de coupable, incluant les lacunes d'observabilité constatées et les actions correctives.

## 11. Protection, accès, rétention et qualité de la télémétrie

#### OBS-100 — Accès et rétention

| Signal | Rétention | Accès |
|---|---|---|
| Journaux applicatifs | `{{EX: 30 jours}}` | `{{équipes produit}}` |
| Journaux de sécurité/audit | `{{EX: 1 an}}` (`SEC-122`) | `{{sécurité}}` |
| Métriques | `{{EX: 13 mois agrégées}}` | `{{…}}` |
| Traces | `{{EX: 7–14 jours}}` | `{{…}}` |

#### OBS-101 — Télémétrie = données
- **Niveau :** DOIT
- **Énoncé :** la télémétrie est soumise aux mêmes règles de classification, de localisation et de protection que les autres données (`SEC-010`) ; transport chiffré ; accès authentifié et journalisé.

#### OBS-102 — Maîtrise des volumes et coûts
- **Niveau :** DEVRAIT
- **Énoncé :** volumes et coûts de télémétrie suivis par service ; niveaux de log et échantillonnage ajustables sans redéploiement.

## 12. Tests de l'instrumentation et des alertes

#### OBS-110 — L'instrumentation est testée
- **Niveau :** DEVRAIT
- **Énoncé :** les métriques et événements critiques (en particulier ceux qui alimentent un SLI ou une métrique métier) sont couverts par des tests vérifiant leur émission et leurs attributs.

#### OBS-111 — Les alertes sont testées
- **Niveau :** DOIT pour les alertes `P1`/`P2`
- **Énoncé :** les règles d'alerte sont versionnées, validées syntaxiquement en CI et testées (tests unitaires de règles `{{EX: promtool}}` ou déclenchement contrôlé en préproduction) ; le routage vers l'astreinte est vérifié périodiquement.

## 13. Composants d'IA (si applicable)

#### OBS-120 — Signaux spécifiques
- **Niveau :** DOIT
- **Énoncé :** sont mesurés par appel de modèle : modèle et version, latence (dont délai du premier jeton si streaming), jetons consommés en entrée/sortie, coût estimé, erreurs et refus, appels d'outils. Contenus des prompts et réponses : non journalisés par défaut ; si nécessaire au diagnostic ou à l'évaluation, collecte échantillonnée, masquée et à rétention courte, conformément aux conventions sémantiques OTel pour l'IA générative.

#### OBS-121 — Qualité en production
- **Niveau :** DEVRAIT
- **Énoncé :** suivi d'indicateurs de qualité (retours utilisateurs, évaluations en ligne échantillonnées, taux de sorties invalides) pour détecter les dérives.

## 14. Spécificités du projet

<!-- PERSONNALISER : identifiants OBS-2NN. -->

#### OBS-200 — `{{TITRE}}`
- **Niveau :** `{{…}}`
- **Énoncé :** `{{…}}`
