---
id: "DOC-UX"
title: "Identité visuelle et expérience utilisateur — {{NOM_DU_PROJET}}"
purpose: "Référentiel central du design UX/UI : identité de marque, ressources officielles, design system, principes UX et conventions de conception des interfaces."
scope: "Toutes les interfaces utilisateur du projet (web, mobile, desktop, e-mails, documents générés) et leurs ressources graphiques."
out_of_scope:
  - "Exigences mesurables d'utilisabilité et d'accessibilité (docs/requirements.md, NFR-USA-*, NFR-ACC-*)"
  - "Architecture technique frontend (docs/architecture.md)"
  - "Conventions d'implémentation du code frontend (rules/code-style.md)"
  - "Méthodes et obligations de tests UX/UI et d'accessibilité (rules/testing.md)"
  - "Valeurs exactes des couleurs, polices, espacements, dimensions (fichiers de design tokens)"
  - "Contenu des maquettes et bibliothèques (Figma ou outil équivalent)"
references:
  - "requirements.md"
  - "architecture.md"
  - "../rules/README.md"
  - "../rules/code-style.md"
  - "../rules/testing.md"
  - "../agents/agent-workflow.md"
owner: "{{DESIGN_LEAD / PRODUCT_DESIGNER}}"
status: "template"
version: "0.1.0"
---

# Identité visuelle et expérience utilisateur — {{NOM_DU_PROJET}}

> **Ce document indique où se trouvent les références de design, comment les utiliser et quelles règles UX/UI respecter.** Il ne recopie ni le contenu des maquettes ni les valeurs des design tokens.
>
> **Deux sources, deux rôles :**
> - **Figma** (ou outil équivalent) = référence de **conception** (intention, maquettes, prototypes) ;
> - **les design tokens versionnés** = référence **d'implémentation** (valeurs consommées par le code).
>
> Toute divergence entre les deux se résout selon la procédure du § 15, jamais par un choix arbitraire.

<!-- MODE D'EMPLOI
- Remplacer les {{PLACEHOLDERS}}. Une ressource qui n'existe pas encore est marquée « Non disponible » (UX-002), jamais remplacée par une valeur inventée.
- Les règles de ce document portent le préfixe UX (UX-NNN). Règles propres au projet : UX-2NN (§ 17).
- Supprimer ce qui ne s'applique pas en l'indiquant (ex. « Pas d'application mobile : non applicable »).
-->

---

## 0. Contrôle du document

| Version | Date | Auteur | Changements | Approuvé par |
|---|---|---|---|---|
| 0.1.0 | `{{AAAA-MM-JJ}}` | `{{…}}` | Création | `{{…}}` |

### Répartition des responsabilités

| Document | Responsabilité |
|---|---|
| [requirements.md](requirements.md) | Exigences d'utilisabilité et d'accessibilité **mesurables** (niveau WCAG visé, objectifs d'efficacité) |
| [architecture.md](architecture.md) | Architecture technique frontend et ses contraintes |
| [code-style.md](../rules/code-style.md) | Conventions d'implémentation du code frontend |
| [testing.md](../rules/testing.md) | Méthodes et obligations de tests UX/UI et d'accessibilité |
| **design-UX-UI.md** | Identité, références visuelles, parcours, conventions de conception de l'interface |
| Fichiers de design tokens | **Valeurs** exactes (couleurs, typographies, espacements, rayons, ombres, durées…) |
| [agent-workflow.md](../agents/agent-workflow.md) | Traitement d'une tâche de design ou de développement par un agent |

---

## 1. Identité de marque

| Élément | Description |
|---|---|
| Nom du produit | `{{…}}` (graphie officielle, règles d'écriture : `{{EX: toujours en un mot, majuscule initiale}}`) |
| Positionnement | `{{Pour qui, quelle promesse, face à quelles alternatives}}` |
| Personnalité | `{{EX: 3 à 5 adjectifs — fiable, chaleureux, précis}}` |
| Valeurs | `{{…}}` |
| Image à transmettre | `{{EX: professionnel sans être austère}}` |
| À éviter | `{{EX: ton familier, visuels génériques de banque d'images}}` |

### 1.1. Ton et voix

| Principe | Nous sommes… | Nous ne sommes pas… | Exemple |
|---|---|---|---|
| `{{Clair}}` | `{{direct, concret}}` | `{{technique, jargonneux}}` | `{{« Votre paiement est confirmé. »}}` |
| `{{Bienveillant}}` | `{{…}}` | `{{…}}` | `{{…}}` |

Le ton s'adapte au contexte (erreur, succès, onboarding, contexte légal) sans perdre sa cohérence.

---

## 2. Ressources officielles

#### UX-001 — Liens faisant foi
- **Niveau :** DOIT
- **Énoncé :** les ressources ci-dessous sont les **seules** références officielles. Une capture d'écran, une copie locale ou un fichier non listé n'a pas valeur de référence.

#### UX-002 — Ressource absente déclarée, jamais inventée
- **Niveau :** DOIT
- **Énoncé :** une ressource qui n'existe pas encore est indiquée `Non disponible` (avec, si connue, la date prévue ou la personne responsable). Aucune valeur, maquette ou ressource n'est inventée pour combler l'absence.

### 2.1. Figma (ou outil de conception équivalent)
- Maquettes : `{{FIGMA_DESIGN_URL | Non disponible}}`
- Design system / bibliothèque : `{{FIGMA_LIBRARY_URL | Non disponible}}`
- Prototypes : `{{FIGMA_PROTOTYPE_URL | Non disponible}}`
- Parcours / flows (FigJam ou équivalent) : `{{FLOWS_URL | Non disponible}}`

### 2.2. Identité visuelle
- Logo : `{{LOGO_PATH | Non disponible}}`
- Variantes du logo : `{{LOGO_VARIANTS_PATH | Non disponible}}`
- Charte graphique : `{{BRAND_GUIDELINES_PATH | Non disponible}}`
- Polices : `{{FONT_REFERENCE | Non disponible}}` (licence : `{{…}}`)
- Icônes : `{{ICON_LIBRARY | Non disponible}}`
- Illustrations / photos : `{{ASSETS_PATH | Non disponible}}`

### 2.3. Design tokens
- Couleurs : `{{COLOR_TOKENS_PATH | Non disponible}}`
- Typographie : `{{TYPOGRAPHY_TOKENS_PATH | Non disponible}}`
- Espacements : `{{SPACING_TOKENS_PATH | Non disponible}}`
- Rayons, ombres, bordures : `{{SHAPE_TOKENS_PATH | Non disponible}}`
- Mouvement (durées, courbes) : `{{MOTION_TOKENS_PATH | Non disponible}}`
- Thèmes (clair, sombre, contraste élevé, marques) : `{{THEME_TOKENS_PATH | Non disponible}}`

### 2.4. Composants
- Bibliothèque UI (code) : `{{UI_COMPONENTS_PATH | Non disponible}}`
- Documentation interactive : `{{STORYBOOK_URL | Non disponible}}`
- Bibliothèque tierce utilisée : `{{EX: nom + version | Aucune}}`

---

## 3. Identité visuelle

> Les **valeurs** (codes couleur, tailles, graisses) sont dans les tokens et la charte. Ce chapitre fixe les **règles d'usage**.

### 3.1. Logo
- Variantes et contexte d'usage :

| Variante | Usage | Fichier |
|---|---|---|
| `{{Principale}}` | `{{fond clair}}` | `{{…}}` |
| `{{Négatif}}` | `{{fond sombre}}` | `{{…}}` |
| `{{Symbole seul}}` | `{{favicon, icône d'application}}` | `{{…}}` |

- Zone de protection, taille minimale : voir `{{charte}}`.
- **Interdits :** déformer, recolorer hors variantes, ajouter des effets, placer sur un fond nuisant à la lisibilité.

### 3.2. Couleurs
- Rôles sémantiques utilisés dans les interfaces : `{{EX: primary, secondary, surface, background, text, border, success, warning, danger, info, focus}}`.

#### UX-010 — Utiliser les rôles, pas les valeurs
- **Niveau :** DOIT
- **Énoncé :** les interfaces utilisent les **tokens sémantiques** (`{{EX: color.text.primary}}`), jamais les valeurs brutes ni les tokens de palette primitive (`{{EX: blue.500}}`), afin de permettre thèmes et évolutions.

#### UX-011 — La couleur ne porte jamais seule l'information
- **Niveau :** DOIT
- **Énoncé :** un statut, une erreur ou une sélection est aussi signalé par un texte, une icône ou une forme (WCAG 1.4.1).

### 3.3. Typographie
- Familles, échelle typographique et graisses : tokens `{{TYPOGRAPHY_TOKENS_PATH}}`.
- Rôles : `{{EX: display, heading 1–4, body, body-small, caption, code}}`.
- Les tailles s'expriment en unités relatives côté web (`rem`) pour respecter les réglages de l'utilisateur ; prise en charge de la taille de texte dynamique sur mobile.

### 3.4. Iconographie
- Bibliothèque unique : `{{ICON_LIBRARY}}` ; style `{{EX: contour, 1,5 px, angles arrondis}}`.
- Icône décorative → masquée aux technologies d'assistance ; icône porteuse de sens sans texte → nom accessible obligatoire.

### 3.5. Illustrations et photographies
- Style : `{{…}}` ; sources autorisées et droits : `{{…}}`.
- Texte alternatif pour toute image porteuse d'information ; représentations diverses et inclusives.

---

## 4. Design tokens

#### UX-020 — Source unique des valeurs
- **Niveau :** DOIT
- **Énoncé :** les valeurs graphiques (couleurs, typographies, espacements, rayons, ombres, tailles, points de rupture, durées et courbes d'animation, z-index) sont définies **uniquement** dans les fichiers de tokens versionnés (§ 2.3). Aucun autre document ni composant ne les recopie (`GOV-002`).

#### UX-021 — Pas de valeur codée en dur
- **Niveau :** DOIT
- **Énoncé :** le code d'interface n'utilise aucune valeur graphique littérale (`#3366FF`, `13px`, `250ms`) lorsqu'un token existe. Un besoin non couvert donne lieu à une **demande de nouveau token** (§ 15), pas à une valeur locale.
- **Vérification :** `{{EX: règle de linter de styles interdisant les valeurs littérales}}`.

### 4.1. Organisation des tokens

| Niveau | Rôle | Exemple de nommage | Utilisé par |
|---|---|---|---|
| **Primitifs** | Palette brute | `{{color.blue.500}}` | Tokens sémantiques uniquement |
| **Sémantiques** | Intention | `{{color.text.danger}}`, `{{space.inset.md}}` | Composants et écrans |
| **Composants** | Spécifiques à un composant | `{{button.primary.background}}` | Le composant concerné |

- Format : `{{EX: JSON au format W3C Design Tokens (DTCG)}}` ; génération vers les plateformes via `{{EX: Style Dictionary}}` → `{{CSS variables, Swift, Kotlin…}}`.
- Commande de génération : `{{…}}` (référencée dans [AGENTS.md](../AGENTS.md)).

---

## 5. Design system

#### UX-030 — Réutiliser avant de créer
- **Niveau :** DOIT
- **Énoncé :** avant de concevoir ou coder un élément d'interface, vérifier s'il existe dans la bibliothèque (§ 2.4). Ordre de préférence : **(1)** composant existant tel quel → **(2)** variante existante → **(3)** composition de composants existants → **(4)** évolution d'un composant (validée, § 15) → **(5)** nouveau composant (validé, § 15).

#### UX-031 — Pas de composant concurrent
- **Niveau :** DOIT
- **Énoncé :** il n'existe qu'un composant par besoin (un seul bouton, un seul champ de saisie, une seule modale…). Une copie locale modifiée d'un composant du design system est interdite.

### 5.1. Inventaire

| Composant | Variantes | États documentés | Figma | Code | Statut |
|---|---|---|---|---|---|
| `{{Button}}` | `{{primary, secondary, tertiary, danger ; sm, md, lg}}` | `{{§ 10}}` | `{{lien}}` | `{{chemin}}` | `{{stable / bêta / déprécié}}` |
| `{{TextField}}` | `{{…}}` | `{{…}}` | `{{…}}` | `{{…}}` | `{{…}}` |

<!-- Si l'inventaire est maintenu dans Storybook ou Figma, le remplacer par un lien (UX-001) plutôt que le dupliquer. -->

### 5.2. Fiche d'un composant (attendus minimaux)
Pour chaque composant, la documentation (Storybook / Figma) indique : objectif et cas d'usage, quand **ne pas** l'utiliser, anatomie, variantes, états (§ 10), comportement responsive, accessibilité (rôle, nom accessible, clavier, annonces), contenu (longueurs, troncature), exemples à faire / à ne pas faire.

### 5.3. Règles de composition
- Grille et espacements uniquement via tokens ; rythme vertical cohérent.
- Une seule action principale par vue ou par groupe ; actions secondaires visuellement subordonnées.
- `{{Règles propres au projet : ex. placement des actions de formulaire, densité}}`.

---

## 6. Principes UX

| Principe | Application attendue |
|---|---|
| **Simplicité** | Chaque écran sert un objectif principal ; supprimer ce qui ne sert pas cet objectif. |
| **Cohérence** | Même problème → même solution, même composant, même libellé, partout. Respect des conventions de la plateforme (web, iOS, Android). |
| **Prévisibilité** | Les éléments se comportent comme leur apparence le suggère ; pas d'action surprise ni d'effet de bord caché. |
| **Hiérarchie de l'information** | Le plus important d'abord ; titres, taille, contraste et position reflètent l'importance. |
| **Navigation facile** | L'utilisateur sait où il est, d'où il vient et où il peut aller. |
| **Charge cognitive réduite** | Reconnaître plutôt que se souvenir, valeurs par défaut pertinentes, divulgation progressive, découpage des tâches longues. |
| **Contrôle et tolérance aux erreurs** | Prévenir les erreurs, permettre d'annuler, confirmer les actions destructives, ne jamais perdre la saisie. |
| **Feedback** | Toute action produit une réponse perceptible et rapide (§ 10). |
| **Inclusion** | Conçu pour la diversité des capacités, langues, appareils et contextes (§ 9). |
| **Respect de l'utilisateur** | Aucun *dark pattern* (fausse urgence, consentement forcé, désabonnement caché, confirmshaming). |

#### UX-040 — Principes appliqués et justifiés
- **Niveau :** DOIT
- **Énoncé :** une conception qui s'écarte d'un principe ci-dessus est justifiée par écrit (maquette annotée, ticket ou ADR de design).

---

## 7. Navigation et parcours

### 7.1. Architecture de l'information
- Arborescence / plan du produit : `{{lien ou schéma}}`
- Modèle de navigation : `{{EX: barre latérale + fil d'Ariane (web), barre d'onglets (mobile)}}`
- Nommage des rubriques : termes du glossaire de [requirements.md § 4](requirements.md).

### 7.2. Parcours utilisateurs clés

| Parcours | Cas d'utilisation (`UC-*`) | Profil (`USR-*`) | Flow / prototype | Étapes clés | Critère de réussite |
|---|---|---|---|---|---|
| `{{Inscription}}` | `{{UC-001}}` | `{{USR-01}}` | `{{lien}}` | `{{…}}` | `{{renvoi AC-* / NFR-USA-*}}` |

#### UX-050 — Parcours respectés
- **Niveau :** DOIT
- **Énoncé :** l'implémentation respecte les parcours, l'ordre des étapes et les points de décision des flows validés. Tout ajout ou suppression d'étape est une modification de parcours soumise à validation (§ 15).

### 7.3. Comportements attendus
- URL partageables et état restaurable (web) ; bouton/geste retour fonctionnel ; pas de perte de saisie en navigation.
- Liens externes et ouvertures dans un nouvel onglet signalés.
- Actions destructives : confirmation explicite ou annulation possible (*undo*), selon `{{règle projet}}`.
- Opérations longues : progression visible et possibilité de continuer à naviguer si pertinent.

---

## 8. Responsive design

| Classe d'appareil | Points de rupture | Grille / colonnes | Particularités |
|---|---|---|---|
| Mobile | tokens `{{breakpoint.*}}` | `{{…}}` | Interaction tactile, une colonne, navigation basse |
| Tablette | `{{…}}` | `{{…}}` | Portrait / paysage |
| Ordinateur | `{{…}}` | `{{…}}` | Largeur de contenu maximale, survol disponible |

#### UX-060 — Conception mobile d'abord, contenu équivalent
- **Niveau :** DEVRAIT
- **Énoncé :** concevoir du plus petit au plus grand ; les fonctionnalités et contenus essentiels sont disponibles sur toutes les tailles ciblées (`NFR-COMP-001`).

#### UX-061 — Robustesse de la mise en page
- **Niveau :** DOIT
- **Énoncé :** pas de défilement horizontal de la page à 320 px de largeur CSS (WCAG 1.4.10), textes agrandis à 200 % sans perte de contenu (1.4.4), respect des zones de sécurité (encoches), orientation non imposée (1.3.4), contenus longs et traduits sans casse (prévoir +30 à +50 % de longueur selon les langues).

---

## 9. Accessibilité

> Le **niveau visé** est fixé par `NFR-ACC-001` dans [requirements.md](requirements.md). Les **méthodes de test** sont dans `TST-082` ([testing.md](../rules/testing.md)). Ce chapitre traduit l'exigence en **règles de conception**.

#### UX-070 — Accessible dès la conception
- **Niveau :** DOIT
- **Énoncé :** chaque maquette précise : ordre de lecture et de focus, noms accessibles des éléments non textuels, structure des titres, états de focus, alternatives textuelles, comportement au zoom.

| Thème | Règle de conception | Référence WCAG |
|---|---|---|
| Contrastes | Paires de couleurs texte/fond et composants d'interface validées dans les tokens pour tous les thèmes | 1.4.3, 1.4.11 |
| Clavier | Tout est utilisable au clavier, sans piège, dans un ordre logique | 2.1.1, 2.1.2, 2.4.3 |
| Focus | Indicateur de focus toujours visible, non masqué par des éléments fixes | 2.4.7, 2.4.11 |
| Lisibilité | Tailles relatives, interlignage et espacements ajustables par l'utilisateur sans perte | 1.4.4, 1.4.12 |
| Technologies d'assistance | Sémantique native d'abord ; rôles, noms et états exposés ; changements dynamiques annoncés | 1.3.1, 4.1.2, 4.1.3 |
| Cibles interactives | Taille minimale définie par token `{{size.target.min}}`, au moins conforme à 2.5.8 et aux guides des plateformes | 2.5.8 |
| Alternatives | Texte alternatif, sous-titres, transcriptions selon le média | 1.1.1, 1.2.x |
| Temps et mouvement | Pas de limite de temps sans contrôle ; mouvement réduit respecté (§ 12) | 2.2.1, 2.3.3 |
| Saisie | Labels visibles, aides, erreurs identifiées et suggérées (§ 11) | 3.3.x |

---

## 10. Interactions et états

#### UX-080 — Tous les états sont conçus
- **Niveau :** DOIT
- **Énoncé :** tout composant interactif et toute vue alimentée par des données ont leurs états conçus **avant** implémentation. Un état non maquetté est signalé, pas improvisé.

### 10.1. États des composants

| État | Signification | Règles |
|---|---|---|
| Par défaut | Repos | — |
| Survol (*hover*) | Pointeur au-dessus | Jamais seul vecteur d'information (absent sur tactile) |
| Focus | Cible clavier | Visible et distinct du survol (§ 9) |
| Actif / pressé | Activation en cours | Retour immédiat |
| Sélectionné / coché | Choix effectué | Non porté par la couleur seule |
| Désactivé | Indisponible | Contraste lisible ; préférer expliquer pourquoi plutôt que désactiver sans indication |
| Lecture seule | Consultable, non modifiable | Distinct de désactivé |

### 10.2. États des vues et des opérations

| État | Règles |
|---|---|
| **Chargement** | Squelettes pour les contenus structurés, indicateur pour les actions ; pas d'indicateur pour les opérations quasi instantanées (`{{seuil : token}}`) ; annoncé aux technologies d'assistance |
| **Succès** | Confirmation proportionnée (message en ligne, toast, écran de confirmation) ; annoncée |
| **Erreur** | Ce qui s'est passé, pourquoi si utile, que faire ; conservation de la saisie ; action de reprise |
| **Vide** | Explique l'absence de contenu et propose l'action suivante (premier usage, aucun résultat, tout traité) |
| **Partiel / dégradé** | Contenu disponible affiché, partie manquante signalée (`NFR-REL-004`) |
| **Hors ligne** (si applicable) | Statut visible, actions possibles ou mises en file |
| **Permissions** | Contenu non autorisé masqué ou expliqué, selon `{{règle projet}}` |

---

## 11. Formulaires et contenus

### 11.1. Formulaires

#### UX-090 — Conventions de formulaire
- **Niveau :** DOIT
- **Énoncé :**
  - label **visible** et persistant pour chaque champ (le *placeholder* n'est pas un label) ;
  - champs obligatoires ou optionnels signalés de façon cohérente (`{{convention : ex. marquer les optionnels}}`) ;
  - type de champ et clavier adaptés, saisie automatique (`autocomplete`) renseignée ;
  - formats attendus indiqués avant la saisie ; formats tolérants (espaces, tirets) ;
  - validation à la sortie du champ ou à la soumission — pas à chaque frappe pour les erreurs ;
  - message d'erreur près du champ, lié programmatiquement, avec récapitulatif focalisé pour les longs formulaires ;
  - aucune saisie perdue en cas d'erreur ;
  - la validation côté client est un confort : la validation de référence est côté serveur (`SEC-060`).

### 11.2. Microcopy et terminologie
- Termes : glossaire de [requirements.md § 4](requirements.md) ; un concept = un mot.
- Boutons : verbes d'action explicites (`{{« Enregistrer le brouillon »}}` plutôt que `{{« OK »}}`).
- Messages d'erreur : sans blâme, sans code technique seul, avec solution.

| Situation | Modèle | Exemple |
|---|---|---|
| Erreur de saisie | `{{Ce qui ne va pas}} + {{comment corriger}}` | `{{« Saisissez une date au format JJ/MM/AAAA. »}}` |
| Erreur système | `{{Ce qui s'est passé}} + {{action}}` | `{{« Impossible d'enregistrer pour le moment. Réessayez dans quelques instants. »}}` |
| Confirmation destructive | `{{Conséquence}} + {{action nommée}}` | `{{« Supprimer le projet “Alpha” ? Cette action est définitive. » [Supprimer le projet]}}` |

### 11.3. Localisation
- Langues : `NFR-I18N-001`. Aucun texte dans les images ; textes externalisés (`CS-082`).
- Prévoir l'expansion des textes, les langues RTL `{{si applicable}}`, les formats de date/nombre/devise selon la locale.
- Le ton (§ 1.1) est adapté par langue, pas traduit mot à mot.

---

## 12. Animations et transitions

#### UX-100 — Le mouvement a une fonction
- **Niveau :** DOIT
- **Énoncé :** une animation sert à orienter (d'où vient / où va un élément), à confirmer une action, à signaler un changement d'état ou à faire patienter. Pas d'animation purement décorative sur les parcours fréquents.

#### UX-101 — Préférences de mouvement réduit
- **Niveau :** DOIT
- **Énoncé :** lorsque l'utilisateur demande la réduction des animations (`prefers-reduced-motion`, réglages système), les animations non essentielles sont supprimées ou remplacées par des fondus ; aucun effet de parallaxe, zoom ou défilement automatique.

#### UX-102 — Durées et courbes via tokens
- **Niveau :** DOIT
- **Énoncé :** durées et courbes d'accélération proviennent des tokens de mouvement (§ 2.3). Animations interruptibles, sans blocage de l'interaction ; aucun clignotement au-delà de 3 fois par seconde (WCAG 2.3.1) ; tout contenu animé de plus de 5 s peut être mis en pause (2.2.2).

---

## 13. Cohérence visuelle

#### UX-110 — Conformité aux références
- **Niveau :** DOIT
- **Énoncé :** l'implémentation respecte les maquettes validées, les tokens et les composants du design system. Un écart volontaire (contrainte technique, amélioration) est documenté et validé par `{{DESIGN_LEAD}}` ; un écart involontaire est un défaut.

#### UX-111 — Tolérance de fidélité
- **Niveau :** DEVRAIT
- **Énoncé :** la comparaison aux maquettes porte sur la structure, la hiérarchie, les tokens utilisés et les comportements, pas sur des pixels isolés. Tolérance : `{{EX: tokens identiques ; écarts de rendu dus aux plateformes acceptés}}`.

---

## 14. Vérification UX/UI

> Les **méthodes** et obligations de test (outils, fréquence, caractère bloquant) sont dans [testing.md](../rules/testing.md) et [release-conventions.md](../rules/release-conventions.md). Ce chapitre liste les **vérifications de conception** attendues.

| Vérification | Quand | Par qui | Preuve attendue |
|---|---|---|---|
| Revue de design (maquettes) | Avant développement d'une fonctionnalité | `{{Design + Produit}}` | Maquette validée (statut dans Figma) |
| Revue de conformité visuelle (code vs maquette) | Dans la PR | `{{Designer / code-reviewer}}` | Captures ou lien de prévisualisation / Storybook |
| Vérification responsive | Dans la PR | `{{…}}` | Captures aux points de rupture |
| Accessibilité | Dans la PR + audit `{{périodicité}}` | `{{…}}` | Résultats `TST-082` |
| Tests visuels de non-régression | CI `{{si outillé}}` | Automatique | `{{EX: Chromatic, Playwright snapshots}}` |
| Tests d'utilisabilité | `{{EX: avant chaque lancement majeur}}` | `{{…}}` | Synthèse : tâches, taux de réussite, problèmes classés, actions |
| Revue des états (§ 10) | Dans la PR | `{{…}}` | Stories ou captures de chaque état |

---

## 15. Évolution du design

### 15.1. Synchronisation Figma ↔ code

#### UX-120 — Sens de synchronisation défini
- **Niveau :** DOIT
- **Énoncé :** le flux officiel est `{{CHOISIR : Figma → tokens versionnés → code (recommandé) | tokens versionnés → Figma}}`, via `{{EX: export de variables Figma, plugin Tokens Studio, pipeline de génération}}`. Les tokens sont modifiés **uniquement** par ce flux.

#### UX-121 — Résolution des divergences
- **Niveau :** DOIT
- **Énoncé :** lorsque Figma, les tokens et le code divergent :
  1. **le code suit les tokens versionnés** en attendant la résolution ;
  2. la divergence est signalée (`{{outil/ticket}}`) à `{{DESIGN_LEAD}}` avec les éléments concernés ;
  3. le designer décide de la valeur correcte ; la correction passe par le flux de `UX-120` ;
  4. **aucun agent ne choisit arbitrairement** l'une des sources ni n'introduit une valeur intermédiaire.

### 15.2. Processus de modification

| Type de changement | Exemples | Validation | Trace |
|---|---|---|---|
| Mineur | Correction d'alignement, libellé, nouvel état d'un composant existant | `{{Designer}}` | PR / ticket |
| Composant | Nouvelle variante, nouveau composant | `{{Design lead + tech lead}}` | Fiche composant + changelog du design system |
| Tokens | Ajout, modification, dépréciation | `{{Design lead}}` | Changelog des tokens, version |
| Parcours | Ajout/suppression d'étape, changement de navigation | `{{Product owner + design lead}}` | Flow mis à jour, exigences liées vérifiées |
| Identité | Logo, couleurs de marque, typographie, ton | `{{Direction / marque}}` | Charte mise à jour, ADR de design si structurant |

### 15.3. Versions et dépréciation
- Le design system et les tokens sont versionnés selon SemVer (`REL-001`) ; changelog dédié : `{{chemin}}`.
- Un composant ou token déprécié est marqué, a un remplaçant indiqué et une date de retrait ; son usage résiduel est suivi.

---

## 16. Règles de conduite pour les agents IA

> Complément du workflow commun ([agent-workflow.md](../agents/agent-workflow.md)) pour toute tâche touchant une interface.

#### UX-130 — Règles pour les agents
- **Niveau :** DOIT
- **Énoncé :** un agent travaillant sur une interface :
  1. **consulte** les maquettes, la charte graphique et les composants existants (§ 2) avant toute modification ;
  2. **réutilise** prioritairement le design system et les composants disponibles (`UX-030`) ;
  3. **n'invente pas** de couleurs, typographies ou styles concurrents lorsqu'une référence officielle existe (`UX-020`, `UX-021`) ; si une ressource est absente, il le signale (`UX-002`) ;
  4. **respecte** les parcours, la hiérarchie visuelle et les interactions attendues (`UX-050`) ;
  5. **prévoit** les états de chargement, succès, erreur et vide (`UX-080`) ;
  6. **respecte** les exigences responsive et d'accessibilité du projet (§ 8, § 9) ;
  7. **vérifie** le rendu et les interactions sur les supports ciblés, et rapporte ce qui a été effectivement vérifié (statuts de preuve `WF-081`) ;
  8. **demande une validation** avant toute modification importante de l'identité visuelle, du design system ou des parcours (§ 15.2 ; `WF-070`) ;
  9. en cas de divergence Figma / tokens / code, **applique `UX-121`** sans trancher.

---

## 17. Spécificités du projet

<!-- PERSONNALISER : identifiants UX-2NN (ex. règles de marque blanche, multi-marques, contraintes d'affichage TV/kiosque, conventions de data visualisation). -->

#### UX-200 — `{{TITRE}}`
- **Niveau :** `{{…}}`
- **Énoncé :** `{{…}}`
