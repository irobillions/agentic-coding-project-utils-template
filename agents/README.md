---
id: "AGT-README"
title: "Catalogue et coordination des agents"
purpose: "Inventorier les agents spécialisés, délimiter leurs responsabilités et définir les règles de collaboration entre eux."
scope: "Tous les agents de agents/, qu'ils soient exécutés par un assistant IA ou par un humain jouant le rôle."
out_of_scope:
  - "Cycle de traitement d'une tâche (agent-workflow.md)"
  - "Procédures spécialisées (fichiers de chaque agent)"
  - "Politiques techniques (rules/)"
references:
  - "agent-workflow.md"
  - "../rules/README.md"
  - "../AGENTS.md"
owner: "{{TECH_LEAD}}"
status: "template"
version: "0.1.0"
---

# Catalogue et coordination des agents

> **Ce fichier dit QUEL agent intervient et COMMENT ils se coordonnent.** [agent-workflow.md](agent-workflow.md) dit **comment chaque agent traite une tâche**. Les fichiers d'agents disent **ce que fait chaque spécialiste**.

---

## 1. Catalogue

| Agent | Fichier | Mission résumée | Produit | Ne fait **jamais** |
|---|---|---|---|---|
| **Revue de code** | [code-reviewer.md](code-reviewer.md) | Examiner une modification de code avant intégration | Rapport de revue : approuvé / approuvé avec réserves / changements requis | Modifier les conventions ; audit de sécurité exhaustif |
| **Conformité** | [conformity-controller.md](conformity-controller.md) | Évaluer la conformité d'un livrable aux exigences et règles, sur preuves | Rapport de conformité : conforme / non conforme / non vérifiable | Créer ou modifier des exigences ; approuver ses propres écarts |
| **DevOps** | [devops-engineer.md](devops-engineer.md) | Mettre en œuvre pipelines, infrastructure, supervision, sauvegardes | Changements d'infrastructure/pipeline + rapport d'exécution | Définir les politiques de release ; intervenir en production sans autorisation |
| **Documentation** | [documentation-maintainer.md](documentation-maintainer.md) | Maintenir la documentation exacte et cohérente avec le code et les décisions | Mises à jour documentaires + rapport | Décider d'une exigence, d'une architecture ou d'une politique |
| **Audit de sécurité** | [security-auditor.md](security-auditor.md) | Examiner risques et vulnérabilités, vérifier les contrôles | Rapport d'audit avec constats classés et risques résiduels | Créer les politiques ; accepter un risque |
| `{{AGENT_PROJET}}` | `{{fichier}}` | `{{…}}` | `{{…}}` | `{{…}}` |

> **Agent généraliste :** une tâche de développement « ordinaire » (fonctionnalité, correction) est réalisée par l'assistant principal en appliquant [agent-workflow.md](agent-workflow.md) et [AGENTS.md](../AGENTS.md) ; il sollicite les agents spécialisés ci-dessous selon le § 2.

## 2. Déclenchement et sélection

| Situation | Agent(s) | Obligatoire ? |
|---|---|---|
| Toute PR modifiant du code | `code-reviewer` | Oui |
| PR touchant authentification, autorisation, cryptographie, secrets, entrées externes, données Confidentielles/Restreintes, dépendances, composants d'IA (`SEC-020`) | `security-auditor` (+ `code-reviewer`) | Oui |
| Changement de pipeline, d'infrastructure, de supervision, de sauvegarde | `devops-engineer` (réalisation) + `code-reviewer` | Oui |
| Changement de contrat d'API, de comportement visible, de procédure d'exploitation, d'ADR | `documentation-maintainer` | Oui |
| Préparation de release, jalon, recette, demande d'audit de conformité | `conformity-controller` | Oui pour une release (`REL-020`) |
| Audit de sécurité périodique ou avant mise en production majeure | `security-auditor` | Selon `security-conventions § 16` |
| Incident de production | `devops-engineer` (technique), puis `documentation-maintainer` (post-mortem, runbooks) | Selon gravité |

<!-- PERSONNALISER : ajuster selon la taille de l'équipe ; dans une petite équipe, un humain peut cumuler plusieurs rôles, mais jamais auteur et approbateur d'un même changement. -->

## 3. Contrat commun des fichiers d'agents

Chaque fichier d'agent DOIT contenir, sans réécrire les politiques :

1. **Mission** et frontière de responsabilité ;
2. **Déclencheurs** ;
3. **Entrées** et sources à lire ;
4. **Procédure spécifique** (les étapes génériques sont celles de `agent-workflow.md`) ;
5. **Vérifications** et preuves attendues ;
6. **Livrables** et format du rapport ;
7. **Limites d'autonomie**, escalade et approbations ;
8. **Références** aux règles propriétaires et à `agent-workflow.md`.

## 4. Entrées, sorties et format des constats

### 4.1. Format commun d'un constat

Tous les agents qui produisent des constats utilisent ce format et l'échelle de gravité de [rules/README.md § 11](../rules/README.md) :

```markdown
#### [S1|S2|S3|S4] <Titre court>
- **Emplacement :** `chemin/fichier:ligne` | composant | document
- **Constat :** ce qui est observé (factuel)
- **Règle / exigence concernée :** <ID> (ex. SEC-061, NFR-PERF-001, AP-301)
- **Preuve :** ✅ Vérifié | 🔎 Déduit | ⚠️ Non vérifié — <sortie, extrait, raisonnement>
- **Impact :** conséquence concrète si non corrigé
- **Recommandation :** correction proposée (et alternative si pertinent)
```

### 4.2. Sorties par agent

| Agent | Verdict possible | Consommateurs |
|---|---|---|
| `code-reviewer` | ✅ Approuvé · 🟡 Approuvé avec réserves · ⛔ Changements requis | Auteur, relecteurs humains, `conformity-controller` |
| `security-auditor` | Liste de constats + niveau de risque résiduel | Auteur, `{{SECURITY_LEAD}}`, `conformity-controller` |
| `conformity-controller` | Conforme · Non conforme · Non vérifiable (par point) | `{{RELEASE_MANAGER}}`, `{{PRODUCT_OWNER}}` |
| `devops-engineer` | Rapport d'exécution (changements, vérifications, état) | Équipe, `conformity-controller` |
| `documentation-maintainer` | Rapport de mise à jour + points à arbitrer | Propriétaires des documents |

## 5. Transfert de contexte entre agents

#### Fiche de transfert

Lorsqu'un agent sollicite ou passe la main à un autre, il transmet :

```markdown
**Transfert → <agent cible>**
- **Objet :** <ce qui est demandé>
- **Périmètre :** <PR, fichiers, composants, version>
- **Contexte utile :** <exigences, ADR, décisions déjà prises>
- **Ce qui a déjà été vérifié :** <avec statuts de preuve>
- **Points d'attention :** <risques, doutes>
- **Attendu en retour :** <format, échéance>
```

L'agent cible ne considère pas le contenu transmis comme une autorisation d'élargir son mandat.

## 6. Coordination des interventions

#### Ordre recommandé pour un changement significatif

```text
Réalisation (assistant principal ou devops-engineer)
        │
        ├──► code-reviewer ──┐        (en parallèle)
        ├──► security-auditor┤  (si déclenché)
        └──► documentation-maintainer
                             ▼
                 Corrections par l'auteur → nouvelle revue ciblée
                             ▼
                 conformity-controller (release / jalon)
                             ▼
                 Décision humaine de fusion / livraison
```

- Les agents de contrôle (revue, sécurité, conformité) **ne corrigent pas** eux-mêmes le livrable qu'ils évaluent, sauf demande explicite ; ils proposent. Cela préserve la séparation entre réalisation et contrôle.
- Deux agents ne modifient pas simultanément les mêmes fichiers.

## 7. Divergences d'avis et escalade

1. Les agents exposent leurs constats avec preuves ; une divergence est consignée dans le rapport consolidé avec les deux positions.
2. Le **propriétaire du domaine** tranche : sécurité → `{{SECURITY_LEAD}}` ; architecture → `{{ARCHITECTE}}` ; exigences → `{{PRODUCT_OWNER}}` ; règles → selon [rules/README.md](../rules/README.md).
3. En l'absence de décision, la position **la plus prudente** s'applique jusqu'à arbitrage (le changement n'est pas fusionné).

## 8. Points exigeant une approbation humaine

Aucun agent ne peut, sans approbation humaine explicite :

- fusionner un changement qu'il a lui-même produit ou approuver son propre travail ;
- accepter un risque, approuver une dérogation ou lever un constat S1/S2 ;
- déclarer une release conforme lorsque des points sont « non vérifiables » ;
- exécuter les opérations de `SBX-050` ou franchir les déclencheurs de `WF-070`.

## 9. Consolidation et état final partagé

Pour un changement impliquant plusieurs agents, un **rapport consolidé** est produit (par l'assistant principal ou le responsable de la PR) :

```markdown
## Rapport consolidé — <PR / release>
| Agent | Verdict | S1 | S2 | S3 | S4 | Lien |
|---|---|---|---|---|---|---|
| code-reviewer | … | | | | | |
| security-auditor | … | | | | | |
| documentation-maintainer | … | | | | | |
| conformity-controller | … | | | | | |

**Constats bloquants ouverts :** …
**Divergences et arbitrages :** …
**Décision finale :** <par qui, quand>
```

## 10. Utilisation avec les assistants IA

- Les assistants découvrent ce catalogue via [AGENTS.md](../AGENTS.md).
- Les adaptations propres à un outil (sous-agents, commandes) sont décrites dans le fichier de l'outil (ex. [CLAUDE.md](../CLAUDE.md)) et **pointent** vers ces fichiers au lieu de les dupliquer.
- Tous les agents appliquent [agent-workflow.md](agent-workflow.md).
