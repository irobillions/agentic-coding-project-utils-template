---
id: "DOC-REQ"
title: "Spécification des exigences — {{NOM_DU_PROJET}}"
purpose: "Source officielle des besoins, résultats attendus, exigences et critères de conformité du produit."
scope: "{{NOM_DU_PROJET}} — {{périmètre fonctionnel couvert par ce document}}"
out_of_scope:
  - "Procédures de test (rules/testing.md)"
  - "Conventions de programmation (rules/code-style.md)"
  - "Choix de conception non imposés (docs/architecture.md)"
references:
  - "architecture.md"
  - "../rules/README.md"
  - "../rules/testing.md"
  - "../agents/conformity-controller.md"
owner: "{{PRODUCT_OWNER}}"
status: "template"
version: "0.1.0"
---

# Spécification des exigences — {{NOM_DU_PROJET}}

> **Ce document dit QUOI, pas COMMENT.** Il décrit ce que le logiciel doit accomplir et les critères permettant de le vérifier. Il n'impose une solution technique que lorsqu'il s'agit d'une **contrainte** explicite (section 9).
>
> **Toutes les valeurs mesurables du produit** (latences, disponibilité, RPO/RTO, niveau d'accessibilité, volumétrie) sont définies **ici et uniquement ici** (`GOV-003`). Les autres documents référencent leur identifiant.

<!-- MODE D'EMPLOI
1. Remplacer les {{PLACEHOLDERS}} ; supprimer les exemples une fois les vraies exigences saisies.
2. Une exigence = un identifiant stable, jamais réutilisé. Une exigence retirée garde son ID avec le statut « Retirée ».
3. Chaque exigence doit être : nécessaire, non ambiguë, vérifiable, atomique, réalisable, cohérente, traçable (ISO/IEC/IEEE 29148).
4. Bannir les termes non vérifiables : « rapide », « convivial », « robuste », « sécurisé », « etc. », « si possible », « au moins dans la plupart des cas ».
-->

---

## 0. Contrôle du document

| Version | Date | Auteur | Changements | Approuvé par |
|---|---|---|---|---|
| 0.1.0 | `{{AAAA-MM-JJ}}` | `{{…}}` | Création | `{{…}}` |

**Statuts possibles d'une exigence :** `Proposée` → `Approuvée` → `Implémentée` → `Vérifiée` · `Différée` · `Rejetée` · `Retirée`.

---

## 1. Contexte, problème et objectifs

### 1.1. Contexte
`{{Situation actuelle, organisation, marché, historique. Pourquoi maintenant ?}}`

### 1.2. Problème à résoudre
`{{Énoncé du problème du point de vue des utilisateurs / du métier, avec éléments factuels (chiffres, irritants constatés).}}`

### 1.3. Objectifs et résultats métier

| ID | Objectif | Indicateur de résultat | Valeur actuelle | Cible | Échéance |
|---|---|---|---|---|---|
| `OBJ-001` | `{{EX: Réduire le délai de traitement des demandes}}` | `{{délai médian de traitement}}` | `{{5 jours}}` | `{{1 jour}}` | `{{AAAA-MM}}` |
| `OBJ-002` | `{{…}}` | `{{…}}` | `{{…}}` | `{{…}}` | `{{…}}` |

### 1.4. Vision produit (une phrase)
> Pour `{{utilisateurs cibles}}` qui `{{besoin}}`, `{{NOM_DU_PROJET}}` est `{{catégorie de produit}}` qui `{{bénéfice clé}}`. Contrairement à `{{alternative}}`, notre produit `{{différenciation}}`.

---

## 2. Parties prenantes et utilisateurs

### 2.1. Parties prenantes

| Partie prenante | Rôle / intérêt | Influence | Contact | Implication attendue |
|---|---|---|---|---|
| `{{EX: Direction métier}}` | Sponsor | Forte | `{{…}}` | Arbitrage des priorités |
| `{{EX: Service juridique / DPO}}` | Conformité | Forte | `{{…}}` | Validation des contraintes légales |
| `{{EX: Support client}}` | Exploitation fonctionnelle | Moyenne | `{{…}}` | Recette |

### 2.2. Profils utilisateurs (personas)

| ID | Profil | Description | Objectifs | Contexte d'usage (appareil, fréquence, environnement) | Besoins d'accessibilité / de langue |
|---|---|---|---|---|---|
| `USR-01` | `{{…}}` | `{{…}}` | `{{…}}` | `{{…}}` | `{{…}}` |
| `USR-02` | `{{…}}` | `{{…}}` | `{{…}}` | `{{…}}` | `{{…}}` |

---

## 3. Périmètre

### 3.1. Dans le périmètre
- `{{Capacité 1}}`
- `{{Capacité 2}}`

### 3.2. Hors périmètre (explicitement exclu)
- `{{EX: Application mobile native (prévue en phase 2)}}`

### 3.3. Hypothèses
| ID | Hypothèse | Impact si fausse | Responsable de la validation |
|---|---|---|---|
| `HYP-001` | `{{…}}` | `{{…}}` | `{{…}}` |

### 3.4. Dépendances externes
| ID | Dépendance | Nature | Criticité | Responsable |
|---|---|---|---|---|
| `DEP-001` | `{{EX: Fournisseur d'identité de l'entreprise}}` | `{{service}}` | `{{haute}}` | `{{…}}` |

---

## 4. Glossaire métier

> Source unique du vocabulaire métier : le code et les documents utilisent ces termes (`CS-022`).

| Terme | Définition | Synonymes interdits | Traduction dans le code |
|---|---|---|---|
| `{{EX: Commande}}` | `{{…}}` | `{{« ordre », « achat »}}` | `{{Order}}` |

---

## 5. Cas d'utilisation et parcours

### 5.1. Liste

| ID | Cas d'utilisation | Acteur(s) | Priorité | Exigences liées |
|---|---|---|---|---|
| `UC-001` | `{{…}}` | `{{USR-01}}` | `{{Must}}` | `{{FR-001, FR-002}}` |

### 5.2. Gabarit de description

```markdown
#### UC-<NNN> — <Titre>
- **Acteur principal :** USR-xx
- **Déclencheur :** …
- **Préconditions :** …
- **Scénario nominal :**
  1. …
  2. …
- **Scénarios alternatifs :** 2a. …
- **Scénarios d'erreur :** E1. …
- **Postconditions (succès) :** …
- **Exigences liées :** FR-…, NFR-…
- **Fréquence / volumétrie :** …
```

<!-- Alternative agile : user stories « En tant que <USR-xx>, je veux <action> afin de <bénéfice> », chacune reliée à un FR et munie de critères d'acceptation (section 11). -->

---

## 6. Exigences fonctionnelles

### 6.1. Gabarit

```markdown
#### FR-<NNN> — <Titre court>
- **Énoncé :** Le système DOIT <comportement observable> lorsque <condition>.
- **Justification / origine :** OBJ-…, UC-…, demande de <partie prenante>
- **Priorité :** Must | Should | Could | Won't (cette fois)
- **Criticité :** Critique | Élevée | Standard | Faible   (oriente la profondeur des tests, TST-001)
- **Critères d'acceptation :** AC-<NNN>-1, AC-<NNN>-2 (section 11)
- **Règles métier :** BR-…
- **Dépendances :** FR-…, INT-…
- **Statut :** Proposée | Approuvée | …
```

### 6.2. Exigences

<!-- Regrouper par domaine fonctionnel. -->

#### Domaine : `{{NOM_DU_DOMAINE}}`

#### FR-001 — `{{TITRE}}`
- **Énoncé :** Le système DOIT `{{…}}` lorsque `{{…}}`.
- **Justification / origine :** `{{OBJ-001, UC-001}}`
- **Priorité :** `{{Must}}` — **Criticité :** `{{Élevée}}`
- **Critères d'acceptation :** `AC-001-1`
- **Règles métier :** `{{BR-001}}`
- **Statut :** `Proposée`

---

## 7. Exigences non fonctionnelles

> Structure alignée sur **ISO/IEC 25010**. Chaque exigence est **mesurable** : métrique, cible, conditions de mesure, méthode de vérification. Supprimer les catégories sans objet **en l'indiquant** (« Non applicable : justification »).

### 7.1. Gabarit

```markdown
#### NFR-<CAT>-<NNN> — <Titre>
- **Énoncé :** <propriété attendue>
- **Métrique :** <ce qui est mesuré>
- **Cible :** <valeur + unité>      ← SEULE source de cette valeur
- **Conditions :** <charge, environnement, périmètre, fenêtre de mesure>
- **Tolérance :** <marge acceptée, s'il y en a>
- **Vérification :** test (TST-…) | mesure en production (SLI, OBS-050) | revue | audit
- **Priorité :** Must | Should | Could
```

### 7.2. Performance et efficacité (`NFR-PERF`)

#### NFR-PERF-001 — Latence des opérations interactives
- **Énoncé :** les opérations `{{liste ou « API synchrones »}}` répondent rapidement pour les utilisateurs.
- **Métrique :** latence côté serveur, percentile 95, mesurée à `{{la passerelle}}`.
- **Cible :** `{{EX: < 300 ms}}` (p95), `{{< 1 s}}` (p99).
- **Conditions :** `{{charge nominale NFR-CAP-001, données de volumétrie NFR-CAP-002}}`, fenêtre de 28 jours en production ; scénario de charge `{{…}}` en préproduction.
- **Vérification :** test de charge (`TST-080`) + SLI en production (`OBS-050`).
- **Priorité :** `{{Must}}`

#### NFR-PERF-002 — Performance perçue côté client (si interface)
- **Métrique / cible :** `{{EX: Core Web Vitals au 75e percentile : LCP ≤ 2,5 s, INP ≤ 200 ms, CLS ≤ 0,1 ; démarrage à froid mobile ≤ 2 s}}`
- **Conditions :** `{{appareils et réseau de référence}}`

### 7.3. Capacité et montée en charge (`NFR-CAP`)

#### NFR-CAP-001 — Charge nominale et pic
- **Cible :** `{{EX: 200 req/s nominales, 1 000 req/s en pic de 15 min, 10 000 utilisateurs simultanés}}`
- **Croissance prévue :** `{{EX: ×3 sur 24 mois}}`

#### NFR-CAP-002 — Volumétrie des données
- **Cible :** `{{EX: 50 M d'enregistrements à 3 ans, 2 To de fichiers}}`

### 7.4. Fiabilité, disponibilité et reprise (`NFR-REL`)

#### NFR-REL-001 — Disponibilité
- **Métrique :** SLI de disponibilité (`OBS-050`).
- **Cible (SLO) :** `{{EX: 99,9 % sur 28 jours glissants}}` (≈ `{{43 min}}` d'indisponibilité par mois).
- **Exclusions :** `{{maintenances planifiées annoncées ≥ 72 h, si contractuellement admises}}`.

#### NFR-REL-002 — Taux d'erreur
- **Cible :** `{{EX: ≥ 99,5 % des requêtes valides réussies}}`

#### NFR-REL-003 — Objectifs de reprise

| Actif / service | RPO (perte de données max.) | RTO (interruption max.) |
|---|---|---|
| `{{EX: données transactionnelles}}` | `{{EX: 5 min}}` | `{{EX: 1 h}}` |
| `{{EX: fichiers utilisateurs}}` | `{{EX: 24 h}}` | `{{EX: 8 h}}` |
| `{{EX: sinistre régional complet}}` | `{{…}}` | `{{…}}` |

- **Vérification :** tests de restauration et exercices de reprise (`BKP-080`, `BKP-082`, `BKP-100`).

#### NFR-REL-004 — Comportement dégradé
- **Énoncé :** en cas d'indisponibilité de `{{dépendance}}`, le système `{{EX: reste consultable en lecture seule et met en file les écritures}}`.

### 7.5. Sécurité (`NFR-SEC`)

> Les **mécanismes** obligatoires sont dans `rules/security-conventions.md`. Ici : les besoins **propres au produit**.

#### NFR-SEC-001 — Niveau de vérification
- **Cible :** `{{EX: OWASP ASVS niveau 2}}` ; `{{EX: test d'intrusion externe sans vulnérabilité Critique/Élevée ouverte avant mise en production}}`.

#### NFR-SEC-002 — Authentification
- **Énoncé :** `{{EX: MFA obligatoire pour les rôles Administrateur et Gestionnaire ; SSO via le fournisseur d'identité de l'entreprise pour les utilisateurs internes}}`.

#### NFR-SEC-003 — Rôles et permissions
- **Énoncé :** `{{matrice rôles × capacités, ou lien vers celle-ci}}`.

#### NFR-SEC-004 — Traçabilité des actions
- **Énoncé :** `{{EX: toute modification de donnée financière est historisée (qui, quand, avant/après) et consultable par un auditeur pendant 10 ans}}`.

### 7.6. Protection des données et vie privée (`NFR-PRIV`)

| Donnée | Catégorie (dont sensibles) | Finalité | Base légale | Durée de conservation | Localisation autorisée |
|---|---|---|---|---|---|
| `{{EX: e-mail client}}` | `{{personnelle}}` | `{{authentification, notification}}` | `{{contrat}}` | `{{durée du compte + 3 ans}}` | `{{UE}}` |

#### NFR-PRIV-001 — Droits des personnes
- **Énoncé :** `{{EX: une demande d'export ou d'effacement est traitée de bout en bout en ≤ 30 jours, dont l'exécution technique en ≤ 72 h}}`.

### 7.7. Utilisabilité et accessibilité (`NFR-USA`, `NFR-ACC`)

#### NFR-ACC-001 — Conformité d'accessibilité
- **Cible :** `{{EX: WCAG 2.2 niveau AA}}` `{{+ référentiel légal applicable : EN 301 549 / RGAA / ADA / EAA}}`.
- **Périmètre :** `{{toutes les interfaces utilisateur publiques et internes}}`.
- **Vérification :** `TST-082` + audit `{{périodicité}}`.

#### NFR-USA-001 — Efficacité d'usage
- **Métrique / cible :** `{{EX: 90 % des utilisateurs du panel réalisent UC-001 sans aide en < 3 min}}`.

### 7.8. Internationalisation et localisation (`NFR-I18N`)

#### NFR-I18N-001 — Langues et régions
- **Cible :** langues `{{EX: fr-FR, en-US}}` ; formats de dates, nombres et devises selon la locale ; support `{{RTL oui/non}}` ; fuseaux horaires `{{…}}` ; devises `{{…}}`.

### 7.9. Compatibilité et portabilité (`NFR-COMP`)

#### NFR-COMP-001 — Plateformes supportées

| Plateforme | Versions supportées |
|---|---|
| Navigateurs | `{{EX: 2 dernières versions de Chrome, Edge, Firefox, Safari}}` |
| Mobile | `{{EX: iOS 17+, Android 10+ (API 29)}}` |
| Systèmes / environnements d'exécution | `{{…}}` |
| Interopérabilité | `{{EX: import/export CSV RFC 4180, API compatibles v1 pendant 12 mois}}` |

### 7.10. Maintenabilité et évolutivité (`NFR-MAINT`)

#### NFR-MAINT-001 — `{{EX: Délai de mise en production d'un correctif critique}}`
- **Cible :** `{{EX: ≤ 24 h entre la validation du correctif et sa disponibilité en production}}`.

### 7.11. Observabilité et exploitabilité (`NFR-OBS`)

#### NFR-OBS-001 — Détection des incidents
- **Cible :** `{{EX: tout incident affectant la disponibilité est détecté et notifié à l'astreinte en ≤ 5 min}}`.

### 7.12. Composants d'IA (`NFR-AI`) — si applicable

#### NFR-AI-001 — Qualité des réponses
- **Métrique / cible :** `{{EX: ≥ 90 % de réponses jugées correctes et fondées sur les sources sur le jeu d'évaluation EVAL-01}}`.

#### NFR-AI-002 — Garde-fous
- **Énoncé :** `{{EX: aucune action irréversible sans confirmation explicite de l'utilisateur ; refus des demandes hors domaine}}`.

#### NFR-AI-003 — Coût et latence
- **Cible :** `{{EX: coût moyen ≤ 0,02 € par requête ; premier jeton en ≤ 1,5 s (p95)}}`.

#### NFR-AI-004 — Transparence
- **Énoncé :** `{{EX: l'utilisateur est informé qu'il interagit avec une IA ; obligations applicables (ex. AI Act) : …}}`.

---

## 8. Règles métier

| ID | Règle | Source (réglementation, politique interne, expert) | Exigences liées |
|---|---|---|---|
| `BR-001` | `{{EX: Une commande ne peut être annulée après expédition}}` | `{{…}}` | `{{FR-…}}` |

---

## 9. Contraintes

| ID | Type | Contrainte | Source | Négociable |
|---|---|---|---|---|
| `CON-001` | Légale / réglementaire | `{{EX: RGPD ; hébergement des données de santé certifié HDS}}` | `{{texte}}` | Non |
| `CON-002` | Contractuelle | `{{EX: SLA client de 99,9 %}}` | `{{contrat}}` | Non |
| `CON-003` | Technique imposée | `{{EX: intégration obligatoire avec l'ERP existant ; cloud imposé}}` | `{{…}}` | `{{…}}` |
| `CON-004` | Organisationnelle / budgétaire | `{{EX: mise en production avant le …}}` | `{{…}}` | `{{…}}` |
| `CON-005` | Normative / sectorielle | `{{EX: PCI DSS, ISO 27001, normes d'accessibilité}}` | `{{…}}` | Non |

---

## 10. Intégrations et échanges externes

| ID | Système externe | Sens | Données échangées | Protocole / format | Fréquence / volume | Disponibilité exigée | Propriétaire |
|---|---|---|---|---|---|---|---|
| `INT-001` | `{{…}}` | `{{entrant/sortant}}` | `{{…}}` | `{{REST/JSON, SFTP/CSV…}}` | `{{…}}` | `{{…}}` | `{{…}}` |

Pour chaque intégration : comportement attendu en cas d'indisponibilité, de données invalides ou de doublon `{{…}}`.

---

## 11. Critères d'acceptation

### 11.1. Format

Chaque critère est **vérifiable**, indépendant de l'implémentation, et rédigé de préférence en *Given / When / Then* :

```gherkin
# AC-001-1 (FR-001)
Scénario: {{titre}}
  Étant donné {{contexte initial}}
  Quand {{action}}
  Alors {{résultat observable}}
  Et {{résultat complémentaire}}
```

### 11.2. Critères

| ID | Exigence | Critère | Conditions de mesure / données | Tolérance | Méthode de vérification |
|---|---|---|---|---|---|
| `AC-001-1` | `FR-001` | `{{…}}` | `{{…}}` | `{{…}}` | `{{test automatisé E2E / revue / démonstration}}` |

### 11.3. Définition de « prêt » d'une exigence (Definition of Ready)
- [ ] Énoncé clair, non ambigu, validé par le métier
- [ ] Critères d'acceptation vérifiables
- [ ] Priorité et criticité fixées
- [ ] Dépendances identifiées
- [ ] Impacts sécurité / données personnelles / accessibilité évalués

---

## 12. Priorités, arbitrages et dépendances

### 12.1. Méthode de priorisation
`{{EX: MoSCoW par release ; WSJF pour l'ordonnancement}}`

### 12.2. Arbitrages de périmètre

| Date | Décision | Exigences concernées | Raison | Décideur |
|---|---|---|---|---|
| `{{…}}` | `{{EX: FR-014 différée en v2}}` | `{{…}}` | `{{…}}` | `{{…}}` |

### 12.3. Exigences en conflit
`{{Conflits identifiés entre exigences et leur résolution — voir GOV-010.}}`

---

## 13. Matrice de traçabilité

> Mise à jour à chaque changement d'exigence, d'architecture ou de test. Consommée par `agents/conformity-controller.md`. Peut être générée automatiquement à partir des références dans le code et les tests (`TST-002`).

| Exigence | Objectif / UC | Conception (ADR / composant) | Réalisation (module / PR) | Tests | Preuve (rapport, mesure) | Statut |
|---|---|---|---|---|---|---|
| `FR-001` | `OBJ-001`, `UC-001` | `{{ADR-003, composant X}}` | `{{chemin / PR}}` | `{{tests/…}}` | `{{lien rapport CI}}` | `{{Vérifiée}}` |
| `NFR-PERF-001` | `OBJ-…` | `{{ADR-…}}` | `{{…}}` | `{{test de charge}}` | `{{rapport + SLI}}` | `{{…}}` |

---

## 14. Gestion des changements

#### Procédure
1. Toute demande de changement est enregistrée (`{{outil}}`) avec : origine, justification, exigences impactées.
2. Analyse d'impact : architecture, coût/délai, tests, documentation, conformité, sécurité.
3. Décision par `{{PRODUCT_OWNER / comité}}` ; une exigence de sécurité ou légale ne peut être assouplie sans `{{AUTORITÉ}}`.
4. Mise à jour de ce document (version, statut), de la matrice de traçabilité et notification des consommateurs.

#### Journal des changements d'exigences

| ID changement | Date | Exigence(s) | Nature (ajout / modification / retrait) | Origine | Impact | Décision | Version du document |
|---|---|---|---|---|---|---|---|
| `CHG-001` | `{{…}}` | `{{…}}` | `{{…}}` | `{{…}}` | `{{…}}` | `{{…}}` | `{{…}}` |

---

## 15. Risques produit

| ID | Risque | Probabilité | Impact | Réponse | Responsable |
|---|---|---|---|---|---|
| `RSK-001` | `{{…}}` | `{{…}}` | `{{…}}` | `{{éviter / réduire / transférer / accepter}}` | `{{…}}` |

## 16. Questions ouvertes

| ID | Question | Posée par | Date | Responsable de la réponse | Échéance | Réponse |
|---|---|---|---|---|---|---|
| `Q-001` | `{{…}}` | `{{…}}` | `{{…}}` | `{{…}}` | `{{…}}` | `{{…}}` |
