---
id: "RUL-BKP"
title: "Sauvegarde et restauration"
purpose: "Définir les procédés de protection et de récupération des données et des états importants du système."
scope: "Bases de données, stockages de fichiers/objets, files persistantes, configurations, secrets, code d'infrastructure et données hébergées chez des tiers (SaaS)."
out_of_scope:
  - "Fixation des RPO/RTO métier (docs/requirements.md, NFR-REL-*)"
  - "Politique de release et rollback applicatif (rules/release-conventions.md)"
  - "Mise en œuvre technique (agents/devops-engineer.md)"
references:
  - "README.md"
  - "security-conventions.md"
  - "observability.md"
  - "testing.md"
  - "../docs/requirements.md"
  - "../docs/architecture.md"
owner: "{{SRE_LEAD / OPS_LEAD}}"
status: "template"
version: "0.1.0"
---

# Sauvegarde et restauration

> **Principe directeur :** une sauvegarde n'a de valeur que par la **restauration** qu'elle permet. Une sauvegarde jamais restaurée est une hypothèse, pas une garantie (`AP-605`).
>
> **Objectifs de reprise :** les RPO (perte de données maximale admissible) et RTO (durée maximale d'interruption) sont fixés dans [requirements.md](../docs/requirements.md) (`NFR-REL-*`). Ce fichier définit **comment** les atteindre et **le démontrer**.

<!-- PERSONNALISER : inventaire (§ 1), paramètres par niveau (§ 2), outils et emplacements. -->

---

## 1. Inventaire et classification des actifs

#### BKP-001 — Inventaire tenu à jour
- **Niveau :** DOIT
- **Énoncé :** tout actif contenant un état non reconstructible est inventorié avec son propriétaire, son niveau (§ 2) et l'exigence de reprise associée. L'inventaire est mis à jour à chaque ajout de stockage (revue d'architecture).

| Actif | Type | Classe de données ([SEC-010](security-conventions.md)) | Niveau | Exigence liée | Propriétaire | Méthode | Emplacement de la sauvegarde |
|---|---|---|---|---|---|---|---|
| `{{EX: base principale}}` | `{{PostgreSQL}}` | `{{Confidentielle}}` | `{{T1}}` | `{{NFR-REL-003}}` | `{{équipe}}` | `{{PITR + snapshot quotidien}}` | `{{compte de sauvegarde isolé, région B}}` |
| `{{EX: stockage objets utilisateurs}}` | `{{S3 / Blob}}` | `{{…}}` | `{{T2}}` | `{{…}}` | `{{…}}` | `{{versionnement + réplication}}` | `{{…}}` |
| `{{EX: configuration et IaC}}` | Git | Interne | `{{T2}}` | — | `{{…}}` | Dépôt + miroir | `{{…}}` |
| `{{EX: secrets / clés KMS}}` | `{{coffre}}` | Restreinte | `{{T1}}` | — | `{{…}}` | `{{export chiffré / réplication native}}` | `{{…}}` |
| `{{EX: données SaaS (CRM, tickets)}}` | `{{SaaS}}` | `{{…}}` | `{{T3}}` | — | `{{…}}` | `{{export API}}` | `{{…}}` |

#### BKP-002 — Ce qui n'a pas besoin d'être sauvegardé est explicite
- **Niveau :** DEVRAIT
- **Énoncé :** les éléments reconstructibles (artefacts de build, caches, environnements éphémères) sont listés comme **exclus**, avec la procédure de reconstruction.

## 2. Niveaux de protection

#### BKP-010 — Paramètres par niveau

| Paramètre | T1 — Critique | T2 — Important | T3 — Standard |
|---|---|---|---|
| RPO / RTO cibles | Selon `NFR-REL-*` | Selon `NFR-REL-*` | Selon `NFR-REL-*` |
| Méthode | Continue (PITR / journaux) + complète périodique | Incrémentale + complète | Complète périodique |
| Fréquence minimale | `{{EX: continue + quotidienne}}` | `{{EX: quotidienne}}` | `{{EX: hebdomadaire}}` |
| Copies hors site | Oui, autre région | Oui | `{{…}}` |
| Copie immuable | Oui | Oui | `{{…}}` |
| Test de restauration | `{{EX: mensuel}}` | `{{EX: trimestriel}}` | `{{EX: semestriel}}` |
| Exercice de reprise complet | `{{EX: annuel}}` | `{{EX: annuel}}` | — |

## 3. Stratégies de sauvegarde

#### BKP-020 — Règle 3-2-1-1-0
- **Niveau :** DOIT pour T1 et T2
- **Énoncé :** au moins **3** copies des données, sur **2** supports/services distincts, dont **1** hors site (autre région ou fournisseur), **1** hors ligne ou immuable, et **0** erreur lors de la vérification de restauration.

#### BKP-021 — Méthode adaptée au type d'actif
- **Niveau :** DOIT
- **Énoncé :**
  - **Bases de données :** sauvegarde cohérente (outil natif ou snapshot coordonné) + journaux pour restauration à un instant donné (PITR) lorsque le RPO l'exige ; une copie de fichiers d'une base en cours d'exécution n'est pas une sauvegarde valide.
  - **Stockage objet :** versionnement + verrouillage d'objet + réplication inter-région ; la réplication seule **n'est pas** une sauvegarde (elle réplique aussi les suppressions et les corruptions).
  - **Volumes / VM :** snapshots cohérents au niveau applicatif.
  - **SaaS :** export régulier par API ; le modèle de responsabilité partagée du fournisseur est vérifié.

#### BKP-022 — Redondance ≠ sauvegarde
- **Niveau :** DOIT
- **Énoncé :** RAID, réplicas, haute disponibilité et multi-zone protègent contre la panne matérielle, pas contre la suppression, la corruption logique ou le rançongiciel. Ils ne remplacent pas une sauvegarde.

## 4. Fréquence et automatisation

#### BKP-030 — Automatisation intégrale
- **Niveau :** DOIT
- **Énoncé :** les sauvegardes sont planifiées et exécutées automatiquement, configurées en Infrastructure as Code ; aucune sauvegarde ne dépend d'une action manuelle.

#### BKP-031 — Fréquence dérivée du RPO
- **Niveau :** DOIT
- **Énoncé :** l'intervalle entre deux points de restauration est inférieur au RPO de l'actif, avec une marge couvrant la durée de la sauvegarde elle-même.

## 5. Rétention, archivage et suppression

#### BKP-040 — Politique de rétention

| Type de point | Rétention | Justification |
|---|---|---|
| Journaux PITR | `{{EX: 7–35 jours}}` | Restauration fine récente |
| Quotidiennes | `{{EX: 30 jours}}` | — |
| Hebdomadaires | `{{EX: 12 semaines}}` | — |
| Mensuelles | `{{EX: 12 mois}}` | — |
| Annuelles / légales | `{{EX: selon obligation}}` | `{{RÉFÉRENCE LÉGALE}}` |

#### BKP-041 — Rétention compatible avec la vie privée
- **Niveau :** DOIT
- **Énoncé :** la rétention respecte les durées de conservation des données (`SEC-090`) ; la procédure de gestion des demandes d'effacement vis-à-vis des sauvegardes est documentée (`{{EX: expiration naturelle + réapplication des effacements après restauration}}`).

#### BKP-042 — Suppression sécurisée
- **Niveau :** DOIT
- **Énoncé :** les sauvegardes expirées sont supprimées automatiquement ; la destruction des supports ou des clés de chiffrement (*crypto-shredding*) est tracée.

## 6. Stockage, isolation et résistance aux altérations

#### BKP-050 — Isolation
- **Niveau :** DOIT
- **Énoncé :** les sauvegardes sont stockées dans un domaine de sécurité distinct de la production (compte/abonnement/projet dédié, identités distinctes) : la compromission de la production ne doit pas permettre de supprimer les sauvegardes.

#### BKP-051 — Immuabilité
- **Niveau :** DOIT pour T1 et T2
- **Énoncé :** au moins une copie est immuable (WORM, verrouillage d'objet en mode conformité, coffre de sauvegarde verrouillé) pendant la durée de rétention minimale.

#### BKP-052 — Indépendance géographique
- **Niveau :** DOIT pour T1
- **Énoncé :** une copie est conservée dans une région (ou chez un fournisseur) différente, en respectant les contraintes de localisation des données `{{RÉFÉRENCE CON-*}}`.

## 7. Chiffrement et contrôle d'accès

#### BKP-060 — Chiffrement
- **Niveau :** DOIT
- **Énoncé :** sauvegardes chiffrées en transit et au repos selon [security-conventions.md](security-conventions.md) (`SEC-070`, `SEC-071`).

#### BKP-061 — Clés récupérables indépendamment
- **Niveau :** DOIT
- **Énoncé :** les clés nécessaires au déchiffrement sont elles-mêmes protégées et récupérables même si l'environnement de production est perdu ; la procédure est testée avec la restauration.

#### BKP-062 — Séparation des tâches
- **Niveau :** DOIT
- **Énoncé :** les droits de suppression ou de modification de la rétention sont restreints, nominatifs, soumis à double validation `{{selon outil}}` et journalisés ; les comptes d'administration de la production ne les possèdent pas.

## 8. Restauration et validation d'intégrité

#### BKP-070 — Procédures de restauration écrites
- **Niveau :** DOIT
- **Énoncé :** chaque actif T1/T2 dispose d'un runbook de restauration testé, stocké **hors** du système qu'il permet de restaurer, selon le gabarit :

```markdown
### Runbook — Restauration de {{ACTIF}}
- **Déclencheurs :** {{corruption, suppression, sinistre régional, rançongiciel…}}
- **Décideur :** {{RÔLE}} — **Exécutants :** {{RÔLES}}
- **Prérequis :** accès, clés, outils, capacité cible
- **Étapes :** 1. … 2. … (commandes exactes)
- **Choix du point de restauration :** {{critères, ex. dernier point antérieur à l'incident}}
- **Vérifications post-restauration :** intégrité, cohérence, comptages, contrôles applicatifs
- **Retour en service :** {{procédure, communication}}
- **Durée mesurée au dernier test :** {{…}} — **Date :** {{…}}
```

#### BKP-071 — Vérification d'intégrité
- **Niveau :** DOIT
- **Énoncé :** chaque sauvegarde fait l'objet d'une vérification automatique (sommes de contrôle, statut de l'outil, taille cohérente avec l'historique) ; une sauvegarde non vérifiée est considérée comme **échouée**.

#### BKP-072 — Cohérence inter-systèmes
- **Niveau :** DEVRAIT
- **Énoncé :** lorsque plusieurs stockages doivent rester cohérents (base + objets + index de recherche), la procédure définit l'ordre de restauration et la réconciliation.

## 9. Tests périodiques de restauration

#### BKP-080 — Restauration réelle et mesurée
- **Niveau :** DOIT
- **Énoncé :** selon la fréquence du § 2, une restauration réelle est effectuée dans un environnement isolé, suivie de contrôles applicatifs. Sont mesurés : durée totale (comparée au RTO), point de données obtenu (comparé au RPO), anomalies.

#### BKP-081 — Preuve conservée

| Date | Actif | Point restauré | Durée mesurée | RPO atteint | RTO atteint | Contrôles | Résultat | Opérateur | Lien vers le rapport |
|---|---|---|---|---|---|---|---|---|---|
| `{{…}}` | `{{…}}` | `{{…}}` | `{{…}}` | `{{oui/non}}` | `{{oui/non}}` | `{{…}}` | `{{succès/échec}}` | `{{…}}` | `{{…}}` |

#### BKP-082 — Exercice de reprise après sinistre
- **Niveau :** DOIT pour les systèmes T1
- **Énoncé :** au moins `{{EX: une fois par an}}`, un exercice simule la perte d'un environnement complet (région, compte) ; il inclut la communication, la décision et le retour à la normale. Les écarts donnent lieu à un plan d'action.

## 10. Surveillance, alertes et échecs

#### BKP-090 — Supervision des sauvegardes
- **Niveau :** DOIT
- **Énoncé :** sont surveillés : succès/échec de chaque tâche, âge du dernier point valide (alerte si > RPO), taille et durée anormales, désactivation d'une politique, échec de vérification. Conventions d'alerte : [observability.md](observability.md).

#### BKP-091 — Traitement des échecs
- **Niveau :** DOIT
- **Énoncé :** un échec de sauvegarde T1 est traité comme un incident de gravité `{{EX: élevée}}` ; deux échecs consécutifs d'un même actif sont escaladés à `{{RÔLE}}`.

## 11. Démonstration du respect des objectifs de reprise

#### BKP-100 — Preuves de conformité
- **Niveau :** DOIT
- **Énoncé :** la conformité à chaque `NFR-REL-*` relatif à la reprise est démontrée par :
  1. la configuration effective (fréquence, rétention, immuabilité) vérifiée automatiquement contre l'inventaire ;
  2. le dernier test de restauration réussi dans la période requise, avec durée et point obtenus ;
  3. l'absence d'alerte non résolue sur l'âge des sauvegardes ;
  4. le dernier exercice de reprise et le suivi de son plan d'action.

Ces preuves sont consommées par [agents/conformity-controller.md](../agents/conformity-controller.md).

## 12. Spécificités du projet

<!-- PERSONNALISER : identifiants BKP-2NN (ex. sauvegarde des modèles d'IA entraînés, des index vectoriels, contraintes de souveraineté). -->

#### BKP-200 — `{{TITRE}}`
- **Niveau :** `{{…}}`
- **Énoncé :** `{{…}}`
