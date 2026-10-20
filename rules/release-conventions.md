---
id: "RUL-REL"
title: "Versionnement et livraison"
purpose: "Définir les conditions à satisfaire pour fabriquer, valider, publier, déployer et récupérer une version."
scope: "Gestion de version du code, intégration, construction des artefacts, promotion entre environnements, déploiement, rollback et clôture de release."
out_of_scope:
  - "Scripts et implémentation des pipelines (agents/devops-engineer.md)"
  - "Politique de sauvegarde (rules/backup-conventions.md)"
  - "Méthodes de test (rules/testing.md)"
references:
  - "README.md"
  - "code-style.md"
  - "testing.md"
  - "security-conventions.md"
  - "api-conventions.md"
  - "observability.md"
  - "backup-conventions.md"
  - "../docs/requirements.md"
owner: "{{RELEASE_MANAGER}}"
status: "template"
version: "0.1.0"
---

# Versionnement et livraison

> **Principe directeur :** livrer **souvent, petit, de manière reproductible et réversible**. Un artefact est construit **une seule fois**, vérifié, signé, puis promu sans modification d'un environnement à l'autre. Chaque déploiement a un plan de retour arrière.

<!-- PERSONNALISER : modèle de branches (§ 2), Quality Gates (§ 5), environnements (§ 8), approbateurs (§ 14). Pour une bibliothèque ou une application mobile, adapter § 8–12 (pas de déploiement continu, publication sur registre / store). -->

---

## 1. Versionnement

#### REL-001 — Versionnement sémantique
- **Niveau :** DOIT pour les bibliothèques, SDK, API et tout artefact consommé par un tiers ; DEVRAIT pour les applications.
- **Énoncé :** format **SemVer 2.0.0** `MAJEUR.MINEUR.CORRECTIF[-préversion][+build]` :
  - `MAJEUR` : rupture de compatibilité de l'interface publique (définition des ruptures d'API : `API-071`) ;
  - `MINEUR` : fonctionnalité rétrocompatible ;
  - `CORRECTIF` : correction rétrocompatible.
- **Alternative pour les applications déployées en continu :** `{{EX: CalVer AAAA.MM.JJ-N ou SemVer + SHA}}` — à choisir et documenter ici.

#### REL-002 — Interface publique définie
- **Niveau :** DOIT
- **Énoncé :** ce qui constitue l'interface publique (API, schémas d'événements, CLI, format de fichiers, configuration, schéma de base partagé) est listé `{{CHEMIN / LISTE}}` ; la compatibilité s'évalue sur ce périmètre.

#### REL-003 — Version traçable
- **Niveau :** DOIT
- **Énoncé :** chaque artefact déployé expose sa version et le SHA de commit (endpoint d'information, métadonnées, `service.version` en télémétrie).

## 2. Branches, commits, tags et pull requests

#### REL-010 — Modèle de branches
- **Niveau :** DOIT
- **Énoncé :** `{{RECOMMANDÉ : développement sur tronc (trunk-based) — branches courtes (< {{2}} jours) fusionnées dans main ; fonctionnalités incomplètes masquées par feature flags}}` / `{{ALTERNATIVE : branches de release maintenues pour supporter plusieurs versions}}`.

| Branche | Rôle | Protection |
|---|---|---|
| `main` | Toujours livrable | Fusion via PR uniquement, CI verte, revue obligatoire, historique linéaire `{{oui/non}}`, pas de push forcé |
| `{{feat|fix|chore}}/<ticket>-<description>` | Travail | — |
| `release/<version>` | `{{si maintenance de versions}}` | Comme `main` |
| `hotfix/<ticket>` | Correctif urgent | Comme `main` |

#### REL-011 — Messages de commit
- **Niveau :** DOIT
- **Énoncé :** format **Conventional Commits** : `type(portée optionnelle): description` — types `feat`, `fix`, `perf`, `refactor`, `docs`, `test`, `build`, `ci`, `chore`, `revert` ; rupture signalée par `!` ou pied `BREAKING CHANGE:`. Référence du ticket `{{EX: Refs: PROJ-123}}`.
- **Vérification :** auto (`{{EX: commitlint}}`).

#### REL-012 — Commits signés
- **Niveau :** `{{DOIT / DEVRAIT}}`
- **Énoncé :** commits et tags signés (`{{GPG / SSH / Sigstore gitsign}}`) et vérifiés par la forge.

#### REL-013 — Pull requests
- **Niveau :** DOIT
- **Énoncé :** une PR :
  - a un objectif unique et une taille raisonnable (`{{EX: < 400 lignes modifiées hors code généré}}`, sinon justifiée ou découpée) ;
  - décrit le **pourquoi**, le **quoi**, les risques, les vérifications effectuées et référence les exigences/tickets ;
  - est approuvée par au moins `{{EX: 1}}` relecteur(s) ne l'ayant pas rédigée (`{{EX: 2 pour les zones critiques, propriétaires de code via CODEOWNERS}}`) ;
  - passe toutes les vérifications bloquantes (§ 5) ;
  - ne contient pas de changement hors périmètre.

#### REL-014 — Tags de version
- **Niveau :** DOIT
- **Énoncé :** chaque release est identifiée par un tag immuable `v<version>` sur le commit exact construit ; un tag publié n'est jamais déplacé ni réutilisé.

## 3. Classification des changements

| Classe | Exemples | Exigences supplémentaires |
|---|---|---|
| **Standard** | Fonctionnalité derrière flag, correctif, refactorisation | Quality Gates standard |
| **Rupture de compatibilité** | Changement d'API publique, de schéma partagé, de format | Version majeure, guide de migration, dépréciation préalable (`API-073`), approbation `{{RÔLE}}` |
| **Migration de données** | Changement de schéma, réécriture de données | § 10 + sauvegarde vérifiée avant exécution |
| **Sensible sécurité** | Authentification, autorisation, cryptographie, secrets | Revue `security-auditor` (`SEC-020`) |
| **Infrastructure** | Réseau, IAM, capacité | Plan de changement, revue `devops-engineer`, fenêtre `{{…}}` |
| **Urgence (hotfix)** | Correctif d'incident ou de vulnérabilité critique | § 15 |
| **Modèle d'IA** | Changement de modèle, de version ou de prompt système | Évaluation `TST-140` ; traité comme une release |

## 4. Conditions d'entrée en préparation de release

#### REL-020 — Prête à livrer
- **Niveau :** DOIT
- **Énoncé :** une version entre en préparation lorsque :
  - [ ] le périmètre est figé et tracé (tickets, exigences couvertes) ;
  - [ ] toutes les PR incluses ont satisfait les gates de PR ;
  - [ ] aucun défaut ouvert de gravité S1 ([README.md § 11](README.md)) ni vulnérabilité hors délai (`SEC-130`) sans dérogation ;
  - [ ] les migrations et la stratégie de rollback sont identifiées ;
  - [ ] la documentation impactée est à jour (`agents/documentation-maintainer.md`) ;
  - [ ] la politique de budget d'erreur autorise le déploiement (`OBS-061`) ;
  - [ ] `{{CONDITION SPÉCIFIQUE}}`.

## 5. Quality Gates

#### REL-030 — Contrôles bloquants
- **Niveau :** DOIT
- **Énoncé :** les contrôles suivants bloquent la progression. Les **critères détaillés** appartiennent aux règles propriétaires ; ce tableau fixe uniquement **où** ils bloquent.

| Contrôle | Règle propriétaire | PR | Fusion `main` | Release | Production |
|---|---|---|---|---|---|
| Formatage, lint, typage | `CS-010`, `CS-050` | ✔ | ✔ | — | — |
| Tests unitaires, intégration, contrat | `TST-*` | ✔ | ✔ | ✔ | — |
| Seuils de couverture | `TST-110` | ✔ | ✔ | — | — |
| Aucun test instable non tracé | `TST-120` | ✔ | ✔ | ✔ | — |
| Diff de contrat sans rupture non approuvée | `API-072` | ✔ | ✔ | ✔ | — |
| Détection de secrets | `SEC-080` | ✔ | ✔ | ✔ | — |
| SAST / SCA / IaC / images selon gravité | `SEC-130`, § 16 de `security-conventions` | ✔ | ✔ | ✔ | — |
| SBOM produit, artefact signé | `SEC-102`, `REL-041` | — | — | ✔ | ✔ (vérification) |
| E2E des parcours critiques | `TST-050` | `{{fumée}}` | ✔ | ✔ | — |
| Tests non fonctionnels requis | `TST-080`…`TST-085` | — | — | `{{selon risque}}` | — |
| Évaluations IA sans régression | `TST-140` | `{{…}}` | ✔ | ✔ | — |
| Documentation et changelog | `REL-075` | — | — | ✔ | — |
| Vérification post-déploiement | `REL-073` | — | — | — | ✔ |

#### REL-031 — Gates non contournables
- **Niveau :** DOIT
- **Énoncé :** un gate ne peut être désactivé, ignoré ou forcé (fusion administrateur, `--no-verify`, relance jusqu'au vert) qu'avec une dérogation approuvée ([README.md § 6](README.md)), tracée dans la PR ou la release.

## 6. Construction reproductible, provenance et intégrité

#### REL-040 — Construire une fois
- **Niveau :** DOIT
- **Énoncé :** l'artefact (image, paquet, binaire, bundle) est construit **une seule fois** par la CI à partir d'un commit identifié, puis promu tel quel ; la configuration propre à l'environnement est injectée au déploiement, jamais intégrée au build.

#### REL-041 — Intégrité et provenance
- **Niveau :** DOIT
- **Énoncé :** chaque artefact de release est :
  - construit dans un environnement de CI éphémère et contrôlé, jamais sur un poste ;
  - identifié par son empreinte (digest) ;
  - **signé** (`{{EX: Sigstore cosign, signature du registre}}`) ;
  - accompagné d'une attestation de provenance (`{{SLSA niveau visé, cf. SEC § 1}}`) et d'un SBOM (`SEC-102`).
  Le déploiement en production vérifie la signature et refuse un artefact non signé `{{via contrôle d'admission}}`.

#### REL-042 — Reproductibilité
- **Niveau :** DEVRAIT
- **Énoncé :** dépendances verrouillées, outils de build versionnés, images de base épinglées par digest ; deux builds du même commit produisent un résultat fonctionnellement identique.

#### REL-043 — Conservation
- **Niveau :** DOIT
- **Énoncé :** les artefacts de release, SBOM, attestations et rapports de gates sont conservés `{{EX: durée de support + 1 an}}`.

## 7. Étapes logiques du pipeline CI/CD

#### REL-050 — Séquence minimale

```text
1. Déclenchement (PR / fusion / tag)
2. Préparation : récupération du code, restauration des caches, vérification des dépendances verrouillées
3. Vérifications statiques : format, lint, typage, secrets, SAST, IaC, licences
4. Build : compilation, empaquetage, image
5. Tests : unitaires → intégration → contrat → (E2E de fumée)
6. Analyses de l'artefact : SCA, image, SBOM
7. Signature et attestation de provenance        [fusion / release]
8. Publication dans le registre d'artefacts        [fusion / release]
9. Déploiement en environnement de validation      [fusion / release]
10. Tests post-déploiement / E2E / non fonctionnels
11. Promotion vers l'environnement suivant (§ 8)
12. Vérification post-déploiement en production (§ 11)
```

L'implémentation (outils, fichiers de pipeline) relève de [agents/devops-engineer.md](../agents/devops-engineer.md).

## 8. Promotion entre environnements

| Environnement | Rôle | Condition d'entrée | Approbation |
|---|---|---|---|
| `{{dev / preview}}` | Intégration continue | Fusion / PR | Automatique |
| `{{staging / préprod}}` | Validation représentative | Gates « Fusion » verts | Automatique |
| `{{production}}` | Utilisateurs réels | Gates « Release » verts + vérifications staging | `{{Automatique / RELEASE_MANAGER}}` |

#### REL-055 — Parité des environnements
- **Niveau :** DEVRAIT
- **Énoncé :** l'environnement de pré-production est aussi proche que possible de la production (versions, topologie, configuration) ; les écarts connus sont documentés.

#### REL-056 — Gels de changements
- **Niveau :** PEUT
- **Énoncé :** périodes de gel `{{EX: événements commerciaux}}` pendant lesquelles seuls les correctifs d'urgence sont autorisés.

## 9. Stratégies de déploiement

#### REL-057 — Choix de la stratégie

| Stratégie | Principe | Quand l'utiliser |
|---|---|---|
| **Progressif (rolling)** | Remplacement graduel des instances | Changements standard, compatibles entre versions N et N-1 |
| **Bleu / vert** | Bascule du trafic entre deux environnements complets | Besoin de bascule et de retour instantanés |
| **Canari** | Exposition progressive (`{{EX: 1 % → 10 % → 50 % → 100 %}}`) avec analyse automatique des SLI | Changements à risque, fort trafic |
| **Feature flags** | Code déployé inactif, activé indépendamment | Découpler déploiement et mise à disposition ; tests en production contrôlés |
| **Publication sur store / registre** | Publication progressive (*staged rollout*) | Applications mobiles, bibliothèques |

#### REL-058 — Compatibilité N / N-1
- **Niveau :** DOIT
- **Énoncé :** pendant un déploiement, deux versions coexistent : chaque version est compatible avec le schéma de données, les messages et les API de la version précédente et suivante.

#### REL-059 — Hygiène des feature flags
- **Niveau :** DEVRAIT
- **Énoncé :** chaque flag a un propriétaire, une date de retrait prévue, et est supprimé après généralisation ; les flags de sécurité ou d'urgence (*kill switch*) sont documentés.

## 10. Migrations de données

#### REL-065 — Expand / migrate / contract
- **Niveau :** DOIT
- **Énoncé :** les changements de schéma incompatibles se font en étapes déployées séparément : **(1) étendre** (ajouter, sans casser l'ancien code) → **(2) migrer** (code double-écriture / backfill par lots, idempotent, reprenable) → **(3) contracter** (retirer l'ancien, une fois plus aucune version ne l'utilise).

#### REL-066 — Sécurité des migrations
- **Niveau :** DOIT
- **Énoncé :** migrations versionnées, revues, testées sur un volume représentatif (`TST-085`), sans verrou long sur les tables à fort trafic, précédées d'un point de sauvegarde vérifié (`BKP-*`) lorsqu'elles modifient ou suppriment des données. Une migration destructive est une opération soumise à approbation.

## 11. Rollback et récupération

#### REL-070 — Plan de retour arrière
- **Niveau :** DOIT
- **Énoncé :** chaque release documente comment revenir à l'état précédent (redéploiement de la version N-1, désactivation de flag, bascule bleu/vert) et les limites (migrations irréversibles → *roll-forward* planifié).

#### REL-071 — Déclencheurs
- **Niveau :** DOIT
- **Énoncé :** le retour arrière est déclenché **sans attendre de diagnostic complet** lorsque :
  - un SLO lié est en consommation rapide de budget (`OBS-080`) après le déploiement ;
  - le taux d'erreurs ou la latence dépassent `{{seuils relatifs à la version précédente}}` lors de l'analyse canari ;
  - une fonctionnalité `Must` est indisponible ;
  - une vulnérabilité ou une corruption de données est détectée ;
  - `{{AUTRE}}`.
  Le retour arrière est automatisé lorsque la stratégie le permet.

#### REL-072 — Rollback testé
- **Niveau :** DEVRAIT
- **Énoncé :** la procédure de rollback est exercée régulièrement (pas seulement documentée).

## 12. Vérification post-déploiement

#### REL-073 — Vérification et surveillance renforcée
- **Niveau :** DOIT
- **Énoncé :** après chaque déploiement en production :
  - tests de fumée automatisés sur les parcours critiques ;
  - comparaison des SLI à la version précédente pendant `{{EX: 30 min}}` ;
  - vérification de la version effectivement déployée (`REL-003`) ;
  - surveillance renforcée pendant `{{EX: 24 h}}` pour les changements à risque, avec responsable identifié.

## 13. Changelog, notes de version et traçabilité

#### REL-075 — Changelog
- **Niveau :** DOIT
- **Énoncé :** `CHANGELOG.md` au format **Keep a Changelog** (`Added`, `Changed`, `Deprecated`, `Removed`, `Fixed`, `Security`), généré ou vérifié à partir des Conventional Commits, rédigé pour les **consommateurs**.

#### REL-076 — Notes de version
- **Niveau :** DOIT pour toute version publique
- **Énoncé :** les notes indiquent : nouveautés, ruptures et guide de migration, dépréciations, correctifs de sécurité (avec identifiants CVE le cas échéant), problèmes connus.

#### REL-077 — Traçabilité
- **Niveau :** DOIT
- **Énoncé :** il est possible de remonter de la version déployée → tag → commits → PR → tickets/exigences → rapports de gates.

## 14. Approbations, responsabilités et clôture

| Décision | Responsable |
|---|---|
| Release standard en production | `{{RELEASE_MANAGER / automatique}}` |
| Rupture de compatibilité | `{{TECH_LEAD + PRODUCT_OWNER}}` |
| Dérogation à un gate | Selon [README.md § 6](README.md) |
| Rollback | `{{Personne d'astreinte — sans approbation préalable}}` |
| Hotfix | `{{…}}` |

#### REL-080 — Clôture
- **Niveau :** DOIT
- **Énoncé :** une release est close lorsque : vérification post-déploiement réussie, période de surveillance écoulée sans déclencheur de rollback, changelog/notes publiés, tickets clos, dérogations éventuelles enregistrées, artefacts et preuves archivés.

## 15. Correctifs d'urgence (hotfix)

#### REL-085 — Procédure accélérée, contrôles maintenus
- **Niveau :** DOIT
- **Énoncé :** un hotfix part de la version en production, contient le changement minimal, passe les gates **automatiques** (aucun n'est sauté ; seuls les délais d'approbation peuvent être raccourcis par `{{RÔLE}}`), est reporté sur `main`, et fait l'objet d'une revue a posteriori sous `{{EX: 2 jours ouvrés}}`.

## 16. Spécificités du projet

<!-- PERSONNALISER : identifiants REL-2NN (ex. publication sur App Store / Play Store, versions LTS, fenêtres de maintenance contractuelles). -->

#### REL-200 — `{{TITRE}}`
- **Niveau :** `{{…}}`
- **Énoncé :** `{{…}}`
