---
id: "RUL-API"
title: "Conventions d'API"
purpose: "Définir les conventions de conception, de représentation et d'évolution des interfaces programmatiques."
scope: "Toute API exposée (publique, partenaire ou interne) : HTTP/REST, GraphQL, gRPC, événements et webhooks."
out_of_scope:
  - "Mécanismes de sécurité (rules/security-conventions.md)"
  - "Méthodes de test (rules/testing.md)"
  - "Liste exhaustive des endpoints métier (contrats du projet)"
references:
  - "README.md"
  - "security-conventions.md"
  - "testing.md"
  - "observability.md"
  - "../docs/requirements.md"
  - "../docs/architecture.md"
owner: "{{API_LEAD}}"
status: "template"
version: "0.1.0"
---

# Conventions d'API

> **Principe directeur :** une API est un **produit** et un **contrat**. Elle est conçue pour ses consommateurs, prévisible, documentée par un schéma faisant foi, et elle évolue **sans casser** ceux qui en dépendent.

<!-- PERSONNALISER : § 1 (styles retenus) et les choix marqués {{…}}. Supprimer les sections GraphQL / gRPC / événements si non utilisées. -->

---

## 1. Styles d'API et périmètre

| Style | Utilisé | Cas d'usage dans le projet | Format de contrat faisant foi |
|---|---|---|---|
| REST / HTTP JSON | `{{oui/non}}` | `{{EX: API publique, front web}}` | OpenAPI `{{3.1}}` |
| GraphQL | `{{oui/non}}` | `{{EX: agrégation pour clients riches}}` | SDL |
| gRPC | `{{oui/non}}` | `{{EX: communication interne service-à-service}}` | Protobuf (`proto3`) |
| Événements / messages | `{{oui/non}}` | `{{EX: intégration asynchrone}}` | AsyncAPI `{{3.x}}` / CloudEvents |
| Webhooks | `{{oui/non}}` | `{{EX: notifications partenaires}}` | OpenAPI (webhooks) |

| Audience | Exigences de stabilité |
|---|---|
| **Publique** | Contrat strict, versionnement, politique de dépréciation publiée (§ 9). |
| **Partenaire** | Idem publique, préavis négocié contractuellement : `{{DURÉE}}`. |
| **Interne** | Compatibilité requise pendant les déploiements ; évolution coordonnée possible. |

## 2. Principes généraux

#### API-001 — Contrat d'abord (Contract-First)
- **Niveau :** DOIT
- **Énoncé :** le schéma (OpenAPI, SDL, `.proto`, AsyncAPI) est écrit et revu **avant ou avec** l'implémentation, versionné dans le dépôt, et constitue la source de vérité. Code, clients et documentation en sont dérivés ou validés contre lui.
- **Vérification :** auto (lint du contrat + validation des réponses en tests).

#### API-002 — Cohérence avant préférence
- **Niveau :** DOIT
- **Énoncé :** une nouvelle API suit les conventions de ce fichier et celles des API existantes du projet, même si une autre approche semble préférable localement.

#### API-003 — Conception orientée consommateur
- **Niveau :** DEVRAIT
- **Énoncé :** les ressources et opérations reflètent les concepts métier et les cas d'usage des consommateurs (`UC-*`), pas la structure des tables ni des classes internes.

#### API-004 — Linter de contrat
- **Niveau :** DOIT
- **Énoncé :** un linter (`{{EX: Spectral, Redocly, buf lint, graphql-schema-linter}}`) vérifie automatiquement les règles de ce fichier en CI.

## 3. Nommage (HTTP/REST)

#### API-010 — Ressources

| Élément | Convention | Exemple |
|---|---|---|
| Collection | nom pluriel, `kebab-case` | `/purchase-orders` |
| Élément | collection + identifiant | `/purchase-orders/{orderId}` |
| Sous-ressource | relation de composition uniquement, profondeur ≤ 2 | `/purchase-orders/{orderId}/lines` |
| Action non-CRUD | verbe en sous-ressource, `POST` | `/purchase-orders/{orderId}:cancel` ou `/purchase-orders/{orderId}/cancellation` `{{CHOISIR UNE FORME}}` |
| Paramètres de requête | `{{camelCase / snake_case}}` | `?createdAfter=` |
| Champs JSON | `{{camelCase / snake_case}}` — **un seul style** pour toute l'API | `"createdAt"` |
| En-têtes personnalisés | sans préfixe `X-` (RFC 6648) | `Idempotency-Key` |

- Pas de verbes dans les chemins de ressources (`/getOrders` ✗).
- Pas d'extension de format dans l'URL (`/orders.json` ✗) ; négociation par `Accept`.
- URL sans barre oblique finale ; casse sensible traitée de manière cohérente.

## 4. Représentation des données

#### API-020 — Types et formats

| Type | Convention | Exemple |
|---|---|---|
| Date-heure | RFC 3339 / ISO 8601, **UTC** avec `Z` (ou offset explicite) | `"2026-10-20T14:30:00Z"` |
| Date seule | ISO 8601 `YYYY-MM-DD` | `"2026-10-20"` |
| Durée | ISO 8601 (`PT30S`) ou entier avec unité dans le nom | `"timeoutSeconds": 30` |
| Montant | chaîne décimale + devise ISO 4217, ou entier en unité mineure | `{"amount": "19.99", "currency": "EUR"}` |
| Identifiant | chaîne **opaque** (UUID, ULID…) ; jamais d'ID séquentiel exposé si énumérable | `"ord_01J9Z…"` |
| Énumération | chaîne `UPPER_SNAKE_CASE` `{{ou autre}}` ; les clients DOIVENT tolérer des valeurs inconnues | `"PENDING_PAYMENT"` |
| Booléen | `true` / `false`, nom affirmatif | `"isActive": true` |
| Langue / pays | BCP 47 / ISO 3166-1 alpha-2 | `"fr-FR"`, `"FR"` |
| Grands entiers (> 2^53) | chaîne | `"9007199254740993"` |

#### API-021 — Absence de valeur
- **Niveau :** DOIT
- **Énoncé :** la sémantique entre champ **absent**, `null` et valeur vide est définie dans le contrat et appliquée uniformément (`{{EX: absent = non fourni ; null = effacé explicitement}}`).

#### API-022 — Enveloppe de réponse
- **Niveau :** DOIT
- **Énoncé :** les réponses d'élément renvoient la ressource directement ; les collections utilisent `{{EX: { "data": [...], "page": {...} } }}`. Le choix est unique dans toute l'API.

#### API-023 — Encodage
- **Niveau :** DOIT
- **Énoncé :** UTF-8 ; `Content-Type` explicite (`application/json`, `application/problem+json`…).

## 5. Sémantique des opérations HTTP

| Méthode | Usage | Sûre | Idempotente | Statuts de succès typiques |
|---|---|---|---|---|
| `GET` | Lire | Oui | Oui | `200`, `304` |
| `HEAD` | Métadonnées | Oui | Oui | `200` |
| `POST` | Créer / action | Non | **Non** (sauf clé d'idempotence) | `201` + `Location`, `202`, `200` |
| `PUT` | Remplacer intégralement | Non | Oui | `200`, `204` |
| `PATCH` | Modifier partiellement (`application/merge-patch+json` RFC 7396 ou JSON Patch RFC 6902) | Non | Non par défaut | `200`, `204` |
| `DELETE` | Supprimer | Non | Oui | `204`, `202` |

#### API-030 — Pas d'effet de bord sur `GET`
- **Niveau :** DOIT
- **Énoncé :** une méthode sûre ne modifie aucun état métier.

#### API-031 — Codes de statut

| Code | Usage |
|---|---|
| `200` / `201` / `202` / `204` | Succès (lecture/MAJ ; création ; accepté pour traitement asynchrone ; sans contenu) |
| `400` | Requête mal formée (syntaxe) |
| `401` | Non authentifié |
| `403` | Authentifié mais non autorisé |
| `404` | Ressource inexistante **ou** non visible pour l'appelant (ne pas révéler l'existence) |
| `409` | Conflit d'état (doublon, transition invalide) |
| `412` | Précondition échouée (`If-Match`) |
| `415` / `406` | Type de contenu non supporté / non acceptable |
| `422` | Requête bien formée mais sémantiquement invalide (validation métier) |
| `429` | Limite de débit dépassée (+ `Retry-After`) |
| `500` | Erreur interne inattendue (aucun détail interne exposé) |
| `502` / `503` / `504` | Dépendance en échec / service indisponible (+ `Retry-After` si connu) / délai dépassé |

#### API-032 — Pas de succès déguisé
- **Niveau :** DOIT
- **Énoncé :** une erreur n'est jamais renvoyée avec un statut `2xx` (voir `AP-403`).

## 6. Erreurs

#### API-040 — Format d'erreur standard (RFC 9457 Problem Details)
- **Niveau :** DOIT
- **Énoncé :** toutes les erreurs HTTP utilisent `application/problem+json` :

```json
{
  "type": "https://{{DOMAINE}}/problems/insufficient-funds",
  "title": "Insufficient funds",
  "status": 422,
  "detail": "The account balance is lower than the requested amount.",
  "instance": "/transfers/trf_01J9Z...",
  "code": "INSUFFICIENT_FUNDS",
  "traceId": "4bf92f3577b34da6a3ce929d0e0e4736",
  "errors": [
    { "pointer": "/amount", "code": "MAX_EXCEEDED", "detail": "Must be ≤ 1000.00" }
  ]
}
```

- `code` : identifiant **stable** et documenté, exploitable par programme ; `title`/`detail` : lisibles, localisables via `Accept-Language`.
- `traceId` : corrélation avec la télémétrie ([observability.md](observability.md)).
- Aucune trace d'appel, requête SQL, chemin interne ou donnée sensible ([security-conventions.md](security-conventions.md)).

#### API-041 — Catalogue des codes d'erreur
- **Niveau :** DOIT
- **Énoncé :** les codes d'erreur métier sont listés dans le contrat (ou `{{CHEMIN_CATALOGUE}}`) ; supprimer ou changer le sens d'un code est une **rupture de compatibilité**.

## 7. Validation des requêtes et réponses

#### API-050 — Validation contre le schéma
- **Niveau :** DOIT
- **Énoncé :** les requêtes entrantes sont validées contre le contrat (types, formats, bornes, champs requis, tailles maximales) avant toute logique métier ; toutes les erreurs de validation sont renvoyées ensemble (`errors[]`).

#### API-051 — Champs inconnus
- **Niveau :** DOIT
- **Énoncé :** politique explicite pour les champs inconnus en entrée : `{{EX: rejet (400) / ignorés}}`. Les **clients** DOIVENT ignorer les champs inconnus en sortie (robustesse à l'évolution additive).

#### API-052 — Limites de taille
- **Niveau :** DOIT
- **Énoncé :** taille de corps, longueur des chaînes, nombre d'éléments des tableaux et profondeur d'imbrication sont bornés dans le contrat.

## 8. Collections : pagination, filtrage, tri, recherche

#### API-060 — Pagination obligatoire
- **Niveau :** DOIT
- **Énoncé :** toute collection potentiellement non bornée est paginée, avec une taille par défaut `{{EX: 20}}` et maximale `{{EX: 100}}`.
- **Méthode :** pagination par **curseur opaque** RECOMMANDÉE (`?pageSize=&pageToken=` → `nextPageToken`) ; pagination par offset tolérée pour de petits volumes stables.

#### API-061 — Filtrage et tri
- **Niveau :** DEVRAIT
- **Énoncé :** filtres par paramètres nommés (`?status=ACTIVE&createdAfter=…`) ; tri via `?sort=createdAt,-amount` (`-` = décroissant). Seuls les champs indexés/documentés sont filtrables et triables. Ordre par défaut **stable et documenté**.

#### API-062 — Recherche
- **Niveau :** PEUT
- **Énoncé :** recherche plein texte via `?q=` ou ressource dédiée (`/search`), documentée séparément.

#### API-063 — Champs partiels
- **Niveau :** PEUT
- **Énoncé :** sélection de champs via `?fields=` si le besoin de performance est avéré.

## 9. Versionnement, compatibilité et dépréciation

#### API-070 — Stratégie de version
- **Niveau :** DOIT
- **Énoncé :** version majeure portée par `{{CHOISIR : chemin /v1 | en-tête | type média}}`. Seules les ruptures de compatibilité créent une nouvelle version majeure. Les versions mineures (additives) ne changent pas l'URL.

#### API-071 — Changements compatibles vs ruptures

| Compatible (pas de nouvelle version) | **Rupture** (nouvelle version majeure ou interdit) |
|---|---|
| Ajouter un endpoint, une opération | Supprimer/renommer un endpoint, un champ, un paramètre |
| Ajouter un champ **optionnel** en entrée | Rendre un champ obligatoire, ajouter un champ obligatoire |
| Ajouter un champ en sortie | Changer le type, le format ou la sémantique d'un champ |
| Ajouter une valeur d'enum en sortie **si** les clients sont tenus de tolérer l'inconnu (`API-020`) | Retirer une valeur d'enum ; ajouter une valeur d'enum en entrée attendue |
| Assouplir une validation | Durcir une validation, réduire une limite |
| Ajouter un code d'erreur nouveau sur un cas nouveau | Changer le statut HTTP ou le `code` d'un cas existant |
| | Changer l'authentification, l'ordre par défaut, la pagination |

#### API-072 — Détection automatique des ruptures
- **Niveau :** DOIT
- **Énoncé :** un diff de contrat (`{{EX: oasdiff, buf breaking, graphql-inspector}}`) s'exécute en CI et bloque toute rupture non accompagnée d'une nouvelle version majeure approuvée.

#### API-073 — Dépréciation
- **Niveau :** DOIT
- **Énoncé :** un élément déprécié est marqué `deprecated` dans le contrat, signalé par les en-têtes `Deprecation` (RFC 9745) et `Sunset` (RFC 8594) avec un lien de migration, et maintenu au minimum `{{EX: 6 mois pour une API publique}}`. L'usage résiduel est mesuré avant retrait.

## 10. Fiabilité : idempotence, délais, débit, concurrence

#### API-080 — Clé d'idempotence
- **Niveau :** DOIT pour les opérations non idempotentes à effet financier ou irréversible ; DEVRAIT pour les autres créations.
- **Énoncé :** le client fournit `Idempotency-Key` (UUID) ; le serveur mémorise la réponse pendant `{{EX: 24 h}}` et renvoie la même réponse pour la même clé et le même corps ; `409`/`422` si la clé est réutilisée avec un corps différent.

#### API-081 — Délais d'expiration
- **Niveau :** DOIT
- **Énoncé :** tout appel sortant a un délai explicite ; les délais se propagent (*deadline propagation*) et le délai d'un appelant est supérieur à celui de ses appels internes.

#### API-082 — Nouvelles tentatives
- **Niveau :** DOIT
- **Énoncé :** retries uniquement sur opérations idempotentes (ou munies d'une clé) et erreurs transitoires (`408`, `429`, `502`, `503`, `504`, erreurs réseau) ; **backoff exponentiel avec jitter**, nombre maximal `{{EX: 3}}`, respect de `Retry-After` ; pas de retries empilés à chaque couche (voir `AP-703`).

#### API-083 — Disjoncteur et dégradation
- **Niveau :** DEVRAIT
- **Énoncé :** les dépendances critiques sont protégées par un *circuit breaker* et/ou un *bulkhead* ; le comportement dégradé est documenté.

#### API-084 — Limitation de débit
- **Niveau :** DOIT pour les API exposées hors du système
- **Énoncé :** quotas par consommateur ; dépassement → `429` + `Retry-After` ; en-têtes `RateLimit` / `RateLimit-Policy` (`{{norme IETF retenue}}`) exposés. Valeurs : `{{RÉFÉRENCE NFR-CAP-*}}`.

#### API-085 — Concurrence optimiste
- **Niveau :** DEVRAIT
- **Énoncé :** les ressources modifiables exposent un `ETag` ; les mises à jour acceptent `If-Match` et renvoient `412` en cas de conflit.

#### API-086 — Opérations longues
- **Niveau :** DOIT
- **Énoncé :** une opération dépassant `{{EX: 10 s}}` est asynchrone : `202 Accepted` + `Location` vers une ressource d'opération (`status`, `progress`, `result`/`error`), ou notification par webhook.

## 11. Événements et webhooks

#### API-090 — Format d'événement
- **Niveau :** DEVRAIT
- **Énoncé :** enveloppe **CloudEvents** (`id`, `source`, `type`, `time`, `specversion`, `datacontenttype`, `data`) ; `type` nommé `{{EX: com.example.order.created.v1}}` (passé composé, versionné).

#### API-091 — Garanties de livraison
- **Niveau :** DOIT
- **Énoncé :** la garantie est documentée (généralement *at-least-once*) ; les consommateurs sont **idempotents** (déduplication par `id`) et tolèrent le désordre, sauf ordre garanti par clé de partition documentée.

#### API-092 — Webhooks
- **Niveau :** DOIT
- **Énoncé :** livraison signée (mécanisme : [security-conventions.md](security-conventions.md)), horodatée, avec retries à backoff sur `{{DURÉE}}`, file des échecs consultable, et possibilité de rejouer.

#### API-093 — Évolution des schémas d'événements
- **Niveau :** DOIT
- **Énoncé :** mêmes règles de compatibilité que § 9 ; registre de schémas `{{EX: Schema Registry}}` si disponible.

## 12. Spécificités GraphQL

- Schéma décrit en SDL ; types `PascalCase`, champs `camelCase`, enums `UPPER_SNAKE_CASE`.
- Mutations nommées `verbeNom` (`createOrder`) avec un type d'entrée unique (`CreateOrderInput`) et un type de retour dédié (`CreateOrderPayload`) incluant les erreurs métier.
- Pagination selon la spécification *Connections* (Relay).
- Évolution sans version : dépréciation via `@deprecated(reason: …)`, jamais de suppression sans période de dépréciation.
- Limites **obligatoires** de profondeur, complexité et coût de requête ; introspection désactivée en production pour les API non publiques `{{à arbitrer}}` ; *persisted queries* RECOMMANDÉES pour les clients propriétaires.
- Résolution groupée (*DataLoader*) pour éviter le N+1 (`AP-301`).

## 13. Spécificités gRPC / Protobuf

- Respect du guide de style Protobuf et des règles `buf lint` `{{configuration}}`.
- Paquets versionnés (`company.product.v1`).
- **Ne jamais réutiliser ni renuméroter** un numéro de champ ; réserver (`reserved`) les numéros et noms supprimés.
- Codes d'erreur gRPC canoniques ; détails d'erreur via `google.rpc.Status` / `ErrorInfo`.
- *Deadlines* obligatoires côté client.

## 14. Documentation et exemples

#### API-100 — Documentation générée et enrichie
- **Niveau :** DOIT
- **Énoncé :** la documentation de référence est générée depuis le contrat ; chaque opération a une description, au moins un exemple de requête et de réponse (succès et erreur principale), les codes d'erreur possibles et les exigences d'authentification.

#### API-101 — Guides
- **Niveau :** DEVRAIT pour les API publiques/partenaires
- **Énoncé :** guide de démarrage, authentification, pagination, erreurs, limites de débit, changelog d'API et guides de migration entre versions majeures.

## 15. Tests de contrat

#### API-110 — Conformité au contrat vérifiée
- **Niveau :** DOIT
- **Énoncé :** l'implémentation est testée contre le contrat (validation des réponses, tests de contrat consommateur/fournisseur pour les intégrations internes). La méthode est définie dans [testing.md](testing.md).

## 16. Spécificités du projet

<!-- PERSONNALISER : conventions additionnelles (ex. multi-tenant via en-tête, conventions BFF, SDK clients générés). Identifiants API-2NN. -->

#### API-200 — `{{TITRE}}`
- **Niveau :** `{{…}}`
- **Énoncé :** `{{…}}`
