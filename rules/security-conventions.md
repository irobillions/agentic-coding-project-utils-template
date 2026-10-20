---
id: "RUL-SEC"
title: "Conventions de sécurité logicielle"
purpose: "Définir les obligations techniques de sécurité applicables au logiciel et à son cycle de développement."
scope: "Code, configuration, dépendances, données, pipelines, environnements et composants d'IA du projet."
out_of_scope:
  - "Démarche détaillée d'audit (agents/security-auditor.md)"
  - "Restrictions des environnements isolés (rules/sandbox.md)"
  - "Choix d'architecture du projet (docs/architecture.md)"
  - "Exigences de sécurité métier propres au produit (docs/requirements.md, NFR-SEC-*)"
references:
  - "README.md"
  - "sandbox.md"
  - "observability.md"
  - "release-conventions.md"
  - "../docs/requirements.md"
  - "../docs/architecture.md"
owner: "{{SECURITY_LEAD}}"
status: "template"
version: "0.1.0"
---

# Conventions de sécurité logicielle

> **Principe directeur :** la sécurité est une propriété **conçue**, **vérifiée en continu** et **jamais présumée**. Toute entrée est hostile jusqu'à preuve du contraire ; tout accès est refusé jusqu'à autorisation explicite.

<!-- PERSONNALISER : niveau de vérification visé (§ 1), classification des données (§ 3), délais de correction (§ 13), réglementations (§ 8). Les exigences de sécurité propres au produit (ex. MFA obligatoire pour les administrateurs) vont dans docs/requirements.md sous NFR-SEC-*. -->

---

## 1. Niveau d'exigence visé

| Élément | Valeur projet |
|---|---|
| Référentiel de vérification | OWASP ASVS `{{5.0}}` niveau `{{1 / 2 / 3}}` |
| Mobile (si applicable) | OWASP MASVS `{{L1 / L2 / R}}` |
| Cadre de développement sécurisé | NIST SSDF (SP 800-218) |
| Chaîne d'approvisionnement | SLSA niveau `{{1 / 2 / 3}}` (build) |
| Exigences sectorielles | `{{EX: PCI DSS, HDS, SOC 2, ISO 27001, NIS2, DORA}}` |
| Exigences produit | `NFR-SEC-*` dans [requirements.md](../docs/requirements.md) |

## 2. Principes

#### SEC-001 — Principes fondamentaux
- **Niveau :** DOIT
- **Énoncé :** toute conception et implémentation applique :
  - **moindre privilège** (identités, services, jetons, comptes de base de données, pipelines) ;
  - **défense en profondeur** (aucun contrôle unique n'est supposé suffisant) ;
  - **sécurité par défaut** (configuration la plus sûre à l'installation ; les assouplissements sont explicites) ;
  - **refus par défaut** (*deny by default*) ;
  - **échec sûr** (*fail closed*) : une erreur dans un contrôle de sécurité refuse l'accès ;
  - **minimisation** de la surface d'attaque et des données ;
  - **séparation des privilèges et des tâches** ;
  - **pas de sécurité par l'obscurité** comme seul contrôle.

## 3. Classification des données

#### SEC-010 — Toute donnée est classée

| Classe | Exemples | Exigences minimales |
|---|---|---|
| **Publique** | Contenu marketing | Intégrité |
| **Interne** | Documentation interne, métriques | Accès authentifié |
| **Confidentielle** | Données personnelles, données clients | Chiffrement en transit et au repos, accès sur besoin d'en connaître, journalisation des accès |
| **Restreinte** | Secrets, données de paiement, santé, identifiants nationaux, biométrie | Confidentielle + chiffrement applicatif ou tokenisation, accès nominatif tracé, rétention minimale |

<!-- PERSONNALISER : aligner sur la politique de classification de l'organisation. L'inventaire concret des données et de leur classe figure dans docs/architecture.md (architecture des données). -->

## 4. Modélisation des menaces et analyse de risque

#### SEC-020 — Modélisation des menaces
- **Niveau :** DOIT
- **Énoncé :** une modélisation des menaces est réalisée et mise à jour : à la conception initiale ; lors de tout changement touchant une frontière de confiance, l'authentification, l'autorisation, des données Confidentielles/Restreintes, une nouvelle intégration externe ou un composant d'IA.
- **Méthode :** `{{EX: STRIDE par flux de données ; LINDDUN pour la vie privée}}`, à partir des diagrammes de [architecture.md](../docs/architecture.md).
- **Livrable :** menaces, contrôles retenus, risques résiduels — stocké dans `{{CHEMIN}}`.

#### SEC-021 — Cotation des risques
- **Niveau :** DOIT
- **Énoncé :** les risques sont cotés `{{EX: vraisemblance × impact, échelle 1–5}}` ; les vulnérabilités techniques avec **CVSS `{{4.0}}`**, complété par l'exploitabilité réelle (`{{EX: EPSS, catalogue CISA KEV}}`) et l'exposition. Un risque résiduel **Élevé ou Critique** ne peut être accepté que par `{{AUTORITÉ}}` (voir [README.md § 6](README.md)).

## 5. Authentification

#### SEC-030 — Ne pas réinventer l'authentification
- **Niveau :** DOIT
- **Énoncé :** utiliser un fournisseur d'identité ou une bibliothèque éprouvée, standards **OpenID Connect / OAuth 2.0** (profil OAuth 2.1 : *Authorization Code + PKCE*, pas de *implicit grant*, pas de *password grant*).

#### SEC-031 — Mots de passe (si gérés par le système)
- **Niveau :** DOIT
- **Énoncé :** conformément à NIST SP 800-63B :
  - longueur minimale `{{EX: 15 caractères si facteur unique, 8 si MFA}}`, maximale ≥ 64 ; tous caractères Unicode acceptés ;
  - **pas** de règles de composition arbitraires ni d'expiration périodique forcée (sauf compromission) ;
  - vérification contre une liste de mots de passe compromis ;
  - stockage par **Argon2id** (`{{paramètres, ex. m=19 MiB, t=2, p=1}}`) ou à défaut scrypt / bcrypt (coût ≥ 10) ; jamais de hachage rapide (MD5, SHA-*) ni de chiffrement réversible ;
  - protection contre le *credential stuffing* et la force brute (limitation, délais progressifs, détection).

#### SEC-032 — Authentification multifacteur
- **Niveau :** DOIT pour les comptes à privilèges et l'accès aux outils de production ; selon `NFR-SEC-*` pour les utilisateurs.
- **Énoncé :** préférer les facteurs résistants à l'hameçonnage (**passkeys / WebAuthn / FIDO2**) ; SMS à éviter comme seul second facteur.

#### SEC-033 — Messages et flux sans fuite
- **Niveau :** DOIT
- **Énoncé :** les messages d'échec de connexion, de réinitialisation et d'inscription ne révèlent pas l'existence d'un compte ; les jetons de réinitialisation sont à usage unique, aléatoires (CSPRNG), à courte durée de vie `{{EX: 15 min}}`.

#### SEC-034 — Authentification service-à-service
- **Niveau :** DOIT
- **Énoncé :** identités de charge de travail (*workload identity*), mTLS ou jetons à courte durée de vie ; pas de clés statiques partagées à longue durée de vie.

## 6. Sessions et jetons

#### SEC-040 — Cookies de session
- **Niveau :** DOIT
- **Énoncé :** attributs `Secure`, `HttpOnly`, `SameSite=Lax` (ou `Strict`), préfixe `__Host-` lorsque possible ; identifiant régénéré à l'authentification et à l'élévation de privilège ; invalidation côté serveur à la déconnexion.

#### SEC-041 — Durées de vie
- **Niveau :** DOIT
- **Énoncé :** expiration d'inactivité `{{EX: 30 min}}` et absolue `{{EX: 12 h}}` selon la sensibilité ; jetons d'accès courts `{{EX: ≤ 15 min}}` ; jetons de rafraîchissement rotatifs avec détection de réutilisation.

#### SEC-042 — Jetons JWT
- **Niveau :** DOIT
- **Énoncé :** algorithme imposé côté vérificateur (refus de `alg: none` et de la confusion d'algorithmes) ; vérification de la signature, `exp`, `nbf`, `iss`, `aud` ; pas de donnée sensible dans le contenu ; révocation possible pour les jetons à longue durée.

#### SEC-043 — Stockage côté client
- **Niveau :** DOIT
- **Énoncé :** pas de jeton d'accès à longue durée dans `localStorage` / `sessionStorage` des applications web ; sur mobile, utiliser le stockage sécurisé de la plateforme (Keychain, Keystore).

## 7. Autorisation et contrôle d'accès

#### SEC-050 — Contrôle côté serveur, à chaque requête
- **Niveau :** DOIT
- **Énoncé :** toute autorisation est vérifiée côté serveur, sur chaque requête, au plus près de la ressource. Masquer un bouton dans l'interface n'est pas un contrôle.

#### SEC-051 — Autorisation au niveau de l'objet et du champ
- **Niveau :** DOIT
- **Énoncé :** chaque accès à un objet vérifie que l'appelant y a droit (protection contre BOLA/IDOR) ; chaque champ modifiable est explicitement autorisé (protection contre l'assignation de masse / BOPLA).

#### SEC-052 — Modèle d'autorisation explicite
- **Niveau :** DOIT
- **Énoncé :** le modèle (`{{RBAC / ABAC / ReBAC}}`), les rôles et la matrice des permissions sont documentés `{{CHEMIN}}` et centralisés dans le code (pas de vérifications ad hoc dispersées).

#### SEC-053 — Isolation multi-locataire (si applicable)
- **Niveau :** DOIT
- **Énoncé :** l'identifiant de locataire provient du contexte authentifié, jamais d'un paramètre client ; l'isolation est appliquée systématiquement (filtre obligatoire, *row-level security* ou base dédiée) et couverte par des tests négatifs.

## 8. Validation des entrées, encodage des sorties et vulnérabilités courantes

#### SEC-060 — Validation par liste d'autorisation
- **Niveau :** DOIT
- **Énoncé :** toute entrée (utilisateur, API, fichier, message, en-tête, variable d'environnement, sortie de modèle d'IA) est validée côté serveur par schéma : type, format, longueur, plage, liste de valeurs autorisées. La validation côté client n'est qu'un confort.

#### SEC-061 — Requêtes paramétrées
- **Niveau :** DOIT
- **Énoncé :** aucune construction de requête (SQL, NoSQL, LDAP, XPath, commandes système, expressions de template) par concaténation d'entrées. Requêtes paramétrées / ORM ; pas d'appel au shell avec des arguments non maîtrisés.

#### SEC-062 — Encodage contextuel des sorties
- **Niveau :** DOIT
- **Énoncé :** les données sont encodées selon leur contexte de sortie (HTML, attribut, JavaScript, URL, CSS) par le moteur de templates ; tout rendu brut (`innerHTML`, `dangerouslySetInnerHTML`, `|safe`) passe par un assainisseur éprouvé et est justifié.

#### SEC-063 — Protections web
- **Niveau :** DOIT (applications web)
- **Énoncé :**
  - **CSRF** : jetons anti-CSRF ou `SameSite` + vérification d'origine pour toute requête modifiant l'état authentifiée par cookie ;
  - **CORS** : liste explicite d'origines ; jamais `*` avec des identifiants ;
  - **En-têtes** : `Strict-Transport-Security`, `Content-Security-Policy` (sans `unsafe-inline` pour les scripts ; nonces ou hachages), `X-Content-Type-Options: nosniff`, `Referrer-Policy`, `frame-ancestors` (anti-clickjacking), `Permissions-Policy` ;
  - **Redirections** : liste blanche des destinations.

#### SEC-064 — Requêtes sortantes (SSRF)
- **Niveau :** DOIT
- **Énoncé :** toute requête sortante dont la destination dépend d'une entrée est restreinte par liste d'autorisation ; accès bloqué aux adresses internes, de métadonnées cloud et aux schémas non HTTP(S) ; redirections revalidées.

#### SEC-065 — Fichiers téléversés
- **Niveau :** DOIT
- **Énoncé :** taille limitée, type vérifié par le contenu (pas seulement l'extension), nom régénéré, stockage hors racine web, analyse antimalware `{{si applicable}}`, téléchargement servi avec `Content-Disposition` adapté.

#### SEC-066 — Désérialisation et parseurs
- **Niveau :** DOIT
- **Énoncé :** pas de désérialisation native de données non fiables (formats binaires d'objets, YAML non sûr, `pickle`…) ; parseurs XML avec entités externes désactivées (XXE) ; limites contre les expressions régulières catastrophiques (ReDoS) et les bombes de décompression.

## 9. Cryptographie et gestion des clés

#### SEC-070 — Pas de cryptographie maison
- **Niveau :** DOIT
- **Énoncé :** uniquement des bibliothèques reconnues et des primitives modernes ; aucune implémentation ou composition cryptographique originale.

| Usage | Choix autorisés | Interdits |
|---|---|---|
| Transport | TLS 1.3 (TLS 1.2 avec suites AEAD tolérée) | SSL, TLS 1.0/1.1, suites sans confidentialité persistante |
| Chiffrement symétrique | AES-256-GCM, ChaCha20-Poly1305 | ECB, CBC sans MAC, DES/3DES, RC4 |
| Signature / échange | Ed25519, ECDSA P-256, RSA ≥ 3072 (PSS / OAEP), X25519 | RSA < 2048, PKCS#1 v1.5 pour du nouveau code |
| Hachage | SHA-256/384/512, SHA-3, BLAKE2/3 | MD5, SHA-1 pour la sécurité |
| Aléa | CSPRNG du système | Générateurs pseudo-aléatoires non cryptographiques |
| Mots de passe | voir `SEC-031` | — |

#### SEC-071 — Gestion des clés
- **Niveau :** DOIT
- **Énoncé :** clés générées et stockées dans un KMS/HSM `{{SERVICE}}` ; chiffrement par enveloppe ; rotation `{{PÉRIODICITÉ}}` et en cas de compromission ; séparation des clés par environnement et par usage ; accès journalisé.

#### SEC-072 — Agilité cryptographique
- **Niveau :** DEVRAIT
- **Énoncé :** les algorithmes et versions de clés sont identifiés dans les données chiffrées afin de permettre leur remplacement (préparation post-quantique `{{à évaluer selon la durée de vie des données}}`).

## 10. Secrets et certificats

#### SEC-080 — Aucun secret dans le code ni l'historique
- **Niveau :** DOIT
- **Énoncé :** aucun secret (mot de passe, clé, jeton, chaîne de connexion, certificat privé) dans le code, les fichiers de configuration versionnés, les images, les logs, les messages d'erreur, les tickets ou les prompts d'IA.
- **Vérification :** auto — détection de secrets en pre-commit et CI (`{{OUTIL}}`), bloquante.

#### SEC-081 — Coffre de secrets
- **Niveau :** DOIT
- **Énoncé :** secrets stockés dans `{{EX: Vault, AWS Secrets Manager, Azure Key Vault, GCP Secret Manager}}`, injectés à l'exécution, à portée minimale, avec rotation automatisée lorsque possible et identités fédérées (OIDC) pour la CI plutôt que des secrets statiques.

#### SEC-082 — Secret exposé = secret compromis
- **Niveau :** DOIT
- **Énoncé :** tout secret exposé (commit, log, capture) est **révoqué et remplacé immédiatement** ; la réécriture de l'historique ne suffit pas. Incident tracé.

#### SEC-083 — Certificats
- **Niveau :** DOIT
- **Énoncé :** certificats émis par une AC de confiance, renouvellement automatisé, surveillance d'expiration (alerte ≥ `{{EX: 21 jours}}` avant).

## 11. Protection des données et vie privée

#### SEC-090 — Minimisation et finalité
- **Niveau :** DOIT
- **Énoncé :** ne collecter que les données nécessaires aux finalités définies dans `docs/requirements.md` ; durées de conservation définies et appliquées automatiquement.

#### SEC-091 — Chiffrement
- **Niveau :** DOIT
- **Énoncé :** données Confidentielles et Restreintes chiffrées en transit et au repos ; données Restreintes protégées en plus au niveau applicatif (chiffrement de champ, tokenisation) ou justifiées par ADR.

#### SEC-092 — Droits des personnes et réglementation
- **Niveau :** DOIT si données personnelles
- **Énoncé :** le système permet d'exercer les droits applicables (`{{EX: RGPD : accès, rectification, effacement, portabilité, opposition}}`) ; analyse d'impact (AIPD/DPIA) réalisée lorsque requise ; transferts hors juridiction encadrés. Réglementations applicables : `{{LISTE}}`.

#### SEC-093 — Données hors production
- **Niveau :** DOIT
- **Énoncé :** aucune donnée personnelle réelle hors production sans anonymisation irréversible ou autorisation formelle (voir [sandbox.md](sandbox.md)).

## 12. Chaîne d'approvisionnement logicielle

#### SEC-100 — Analyse des dépendances (SCA)
- **Niveau :** DOIT
- **Énoncé :** les dépendances directes et transitives sont analysées en continu (CI et surveillance planifiée) ; mises à jour automatisées proposées (`{{EX: Renovate, Dependabot}}`).

#### SEC-101 — Provenance et intégrité
- **Niveau :** DOIT
- **Énoncé :** fichiers de verrouillage versionnés ; sources de paquets maîtrisées (registre officiel ou miroir interne) ; protection contre la confusion de dépendances (*namespace* réservé, portée explicite) ; vérification des sommes de contrôle.

#### SEC-102 — Nomenclature (SBOM)
- **Niveau :** DOIT pour tout artefact livré
- **Énoncé :** un SBOM au format **CycloneDX** ou **SPDX** est produit à chaque build de release et conservé avec l'artefact. Signature et provenance : [release-conventions.md](release-conventions.md).

#### SEC-103 — Pipelines durcis
- **Niveau :** DOIT
- **Énoncé :** actions/étapes tierces de CI épinglées par empreinte (SHA) ; permissions minimales des jetons de pipeline ; pas d'exécution de code non revu avec accès aux secrets (ex. PR de forks) ; protection des branches et revues obligatoires.

#### SEC-104 — Images de conteneurs
- **Niveau :** DOIT
- **Énoncé :** images de base minimales et maintenues, épinglées par digest, analysées ; exécution en utilisateur non-root ; système de fichiers en lecture seule lorsque possible.

#### SEC-105 — Licences
- **Niveau :** DOIT
- **Énoncé :** les licences des dépendances sont vérifiées contre la politique `{{EX: liste autorisée / à approuver / interdite}}`.

## 13. Configuration sécurisée

#### SEC-110 — Durcissement
- **Niveau :** DOIT
- **Énoncé :** mode debug, consoles d'administration, endpoints de diagnostic et comptes par défaut désactivés en production ; messages d'erreur génériques côté client ; services et ports non nécessaires fermés ; configuration alignée sur `{{EX: CIS Benchmarks}}`.

#### SEC-111 — Infrastructure as Code analysée
- **Niveau :** DOIT
- **Énoncé :** les définitions d'infrastructure sont analysées en CI (`{{EX: Checkov, tfsec, KICS}}`) ; pas de stockage public par défaut ; pas de règle réseau `0.0.0.0/0` sur des ports d'administration.

#### SEC-112 — Séparation des environnements
- **Niveau :** DOIT
- **Énoncé :** environnements (dev, test, préprod, prod) isolés : comptes/projets cloud, réseaux, secrets et identités distincts ; aucun accès de non-production vers la production.

## 14. Journalisation de sécurité

#### SEC-120 — Événements à journaliser
- **Niveau :** DOIT
- **Énoncé :** sont journalisés avec horodatage, acteur, action, cible, résultat et identifiant de corrélation : authentifications (succès/échecs), changements de privilèges et de rôles, accès refusés, actions d'administration, accès aux données Restreintes, modifications de configuration de sécurité, détection d'entrée malveillante, usage de jetons révoqués.
- **Format et transport :** [observability.md](observability.md).

#### SEC-121 — Jamais de données sensibles dans les journaux
- **Niveau :** DOIT
- **Énoncé :** pas de mots de passe, jetons, secrets, numéros de carte complets, données de santé ou identifiants nationaux dans les journaux ; masquage automatique en plus de la discipline des développeurs.

#### SEC-122 — Intégrité des journaux de sécurité
- **Niveau :** DEVRAIT
- **Énoncé :** journaux de sécurité centralisés, protégés contre l'altération (stockage immuable ou séparé), conservés `{{DURÉE}}`, et exploités par une détection `{{SIEM}}`.

## 15. Gestion des vulnérabilités

#### SEC-130 — Délais de correction

| Gravité (ajustée selon exploitabilité et exposition) | Délai maximal de correction ou d'atténuation | Bloque la release |
|---|---|---|
| Critique | `{{EX: 7 jours}}` (exploitée activement : `{{EX: 48 h}}`) | Oui |
| Élevée | `{{EX: 30 jours}}` | Oui |
| Moyenne | `{{EX: 90 jours}}` | Non (sauf décision) |
| Faible | `{{EX: 180 jours / meilleur effort}}` | Non |

Un délai dépassé impose une dérogation ([README.md § 6](README.md)).

#### SEC-131 — Triage
- **Niveau :** DOIT
- **Énoncé :** chaque constat est trié (vrai/faux positif, atteignabilité, exposition), assigné, suivi jusqu'à clôture ; un faux positif ou une non-applicabilité est justifié et documenté (ex. VEX).

#### SEC-132 — Divulgation coordonnée
- **Niveau :** DOIT pour les produits exposés
- **Énoncé :** une politique de divulgation est publiée (`/.well-known/security.txt`, RFC 9116) avec un contact `{{EMAIL_SÉCURITÉ}}` et un délai de réponse.

#### SEC-133 — Réponse aux incidents
- **Niveau :** DOIT
- **Énoncé :** une procédure de réponse aux incidents de sécurité existe `{{LIEN}}` : détection, confinement, éradication, restauration, notification (obligations légales `{{EX: 72 h RGPD}}`), retour d'expérience.

## 16. Contrôles de sécurité du cycle de développement

| Contrôle | Étape | Bloquant | Outil |
|---|---|---|---|
| Détection de secrets | pre-commit, CI | Oui | `{{…}}` |
| Analyse statique (SAST) | CI | Oui (Critique/Élevée) | `{{…}}` |
| Analyse des dépendances (SCA) | CI + planifiée | Selon `SEC-130` | `{{…}}` |
| Analyse IaC | CI | Oui (Critique/Élevée) | `{{…}}` |
| Analyse d'images | CI (build) + registre | Selon `SEC-130` | `{{…}}` |
| Analyse dynamique (DAST) | préprod | `{{…}}` | `{{…}}` |
| Revue de sécurité manuelle | PR touchant `SEC-020` | Oui | `security-auditor` |
| Test d'intrusion | `{{EX: annuel + changement majeur}}` | — | `{{PRESTATAIRE}}` |
| Modélisation des menaces | conception | Oui pour tâches `L` | — |

L'intégration de ces contrôles dans les Quality Gates est définie par [release-conventions.md](release-conventions.md).

## 17. Applications mobiles et clients (si applicable)

#### SEC-150 — Le client n'est pas de confiance
- **Niveau :** DOIT
- **Énoncé :** aucun secret embarqué dans une application cliente (web, mobile, desktop) ; toute logique de sécurité est appliquée côté serveur ; communications en TLS avec validation stricte des certificats (épinglage `{{selon analyse de risque}}`) ; données locales sensibles dans le stockage sécurisé de la plateforme.

## 18. Composants d'IA et agents (si applicable)

Référence : OWASP Top 10 for LLM Applications.

#### SEC-160 — Sortie de modèle = entrée non fiable
- **Niveau :** DOIT
- **Énoncé :** toute sortie d'un modèle est validée (schéma structuré), encodée et autorisée comme une entrée utilisateur avant d'être affichée, stockée, exécutée ou transmise.

#### SEC-161 — Séparation instructions / données
- **Niveau :** DOIT
- **Énoncé :** le contenu non fiable (saisie utilisateur, documents, pages web, résultats d'outils) est délimité et traité comme données ; le système suppose que l'injection de prompt **réussira** et limite les conséquences (moindre privilège des outils, confirmation humaine des actions sensibles, pas d'accès à des secrets).

#### SEC-162 — Outils et actions des agents
- **Niveau :** DOIT
- **Énoncé :** chaque outil exposé à un modèle a une portée minimale, une autorisation vérifiée côté serveur sur l'identité de l'utilisateur final, et une liste d'actions nécessitant confirmation humaine. Les environnements d'exécution suivent [sandbox.md](sandbox.md).

#### SEC-163 — Fuite de données vers les modèles
- **Niveau :** DOIT
- **Énoncé :** aucune donnée Restreinte ni secret envoyé à un fournisseur de modèle sans accord contractuel adapté (non-rétention, localisation) ; les prompts système ne contiennent pas d'information sensible (ils peuvent fuiter).

#### SEC-164 — Consommation non bornée
- **Niveau :** DOIT
- **Énoncé :** limites de jetons, de coût, de nombre d'appels d'outils et de durée par requête et par utilisateur.

## 19. Spécificités du projet

<!-- PERSONNALISER : identifiants SEC-2NN. -->

#### SEC-200 — `{{TITRE}}`
- **Niveau :** `{{…}}`
- **Énoncé :** `{{…}}`
- **Vérification :** `{{…}}`
