# Modifier le code, construire et publier les images

> Complément de référence — le [guide principal](../README.md) reste le seul parcours de déploiement à exécuter.

Cette page concerne le développeur et le responsable de construction. Le déployeur
reçoit des sources figées, des images approuvées par digest et un bilan de tests.
Le serveur d’exploitation ne devient pas l’environnement de développement.

## Sources modifiables et VS Code

Les sources Python sont modifiables. Pour une correction maintenue, utiliser le
**dépôt de développement complet**, avec Git, les suites des deux UC, les tests
ACL, les dépendances verrouillées et `ci/`. L’archive opérateur publiée sur GitHub
est une sélection vérifiable pour installer et exploiter ; elle ne contient pas
la totalité de ces tests et ne remplace pas le dépôt de maintenance.
L’URL du dépôt interne et les droits de contribution sont à remettre au mainteneur.
Le dépôt de développement actuel est `clients_neovelion_tech/dinum_rie_demo_albert_api`
sur GitLab ; son historique privé ne fait pas partie du dépôt public de distribution.

Ouvrir `deploy/ubuntu/rie-development.code-workspace` dans VS Code depuis le dépôt
complet. Cet espace conserve les chemins réels et fournit deux tâches : lancer
les non-régressions et contrôler la sélection du paquet opérateur. Choisir un
interpréteur Python **3.12** avec l’extension Python de VS Code, préparé avec les verrous de `ci/README.md`
dans le dépôt complet, avant de lancer la tâche de test. La commande du terminal
`python -B ci/runners/run_ubuntu_tests.py` reste utilisable sans VS Code.
Le même espace est joint à l’archive opérateur pour ouvrir ses sources dans
VS Code. Sa tâche de contrôle du paquet y fonctionne ; la tâche de non-régression
complète nécessite le dépôt de développement avec `ci/` et toutes les suites.
Aucun secret du SI ni configuration d’exploitation n’est ajouté à cet espace.

| Partie à modifier | Chemin dans le dépôt complet |
| --- | --- |
| Moteur PDL | `use-cases/pdl/agentic-pdl/rie_agent/` |
| Moteur Incidents et connecteur ITSM | `use-cases/incidents/observability-demo/obs_poc/` |
| Identités, ACL et OAuth | `core/access-control/src/dinum_access/` |
| Client Albert commun | `core/albert-client/src/dinum_albert/` |
| Commandes de déploiement et NiceGUI | `deploy/ubuntu/`, `deploy/ubuntu/portal/` |
| Documentation canonique | `docs/si-rie/README.md`, `docs/si-rie/reference/` |

Une modification de paramètres, de mapping ou d’ACL n’exige généralement pas une
modification du code. Passer par la configuration et les administrations prévues ;
une règle active garde sa révision et sa trace d’audit.

## Construire les trois images depuis la même version

**Où : machine de construction autorisée**, Linux natif de l’architecture cible,
Python et Docker disponibles. Le runtime serveur, NiceGUI et le client Open WebUI
sont trois images distinctes. Les bases, le frontend BuildKit et les dépendances
verrouillées doivent être accessibles par le circuit approuvé. Un proxy de
registry ne fournit pas à lui seul les dépendances Python nécessaires au build.

Le build seul fonctionne aussi depuis l’archive opérateur extraite : remplacer
les deux contrôles Git ci-dessous par `python3 deploy/ubuntu/verify_delivery.py .`,
attendre `passed: true` et relever le commit du manifeste. Pour la maintenance et
les non-régressions complètes, conserver le dépôt complet.

Les commandes ci-dessous illustrent une construction locale reproductible. Elles
ne publient rien et ne contactent pas LDAP, OpenSearch ou Albert. Remplacer le
chemin d’instance de construction si nécessaire ; il doit être neuf et hors dépôt.

```bash
# Relève le commit exact des sources ; attendre une version revue et figée.
git rev-parse HEAD
# Vérifie l'absence de modifications oubliées avant construction ; attendre une sortie vide.
git status --porcelain
# Choisit une instance isolée servant uniquement aux références de construction.
export RIE_BUILD_INSTANCE="$HOME/.local/share/rie-image-build"
# Prépare cette instance externe sans source ni secret réel du SI.
python3 deploy/ubuntu/manage.py prepare "$RIE_BUILD_INSTANCE" --mode external
# Demande aussi la construction de l'image NiceGUI.
python3 deploy/ubuntu/manage.py configure-ui "$RIE_BUILD_INSTANCE" --enable-demo-ui
# Construit le runtime commun et NiceGUI à partir des listes explicites de fichiers.
python3 deploy/ubuntu/manage.py build "$RIE_BUILD_INSTANCE"
```

**Attendu :** les images locales sont référencées par `RIE_IMAGE` et
`RIE_PORTAL_IMAGE` dans le `.env` privé de construction. Aucun `up`, `seed` ou
raccordement réel n’est nécessaire. La documentation et l’assistant n’entrent
pas dans l’image des moteurs. Pour une nouvelle version, choisir une nouvelle
instance de construction et conserver les reçus précédents.

## Publier dans la registry approuvée

**Où : même machine de construction**, même contexte Docker et utilisateur que
pour le build. Le responsable de la registry fournit l’hôte, le chemin du projet,
un compte autorisé à pousser et les miroirs de base. Remplacer les marqueurs
avant d’exécuter. Les identifiants ne sont jamais placés dans les sources.
Si Docker exige `sudo`, appliquer ce préfixe à toutes les commandes Docker et
à la lecture du `.env` appartenant à cet opérateur ; ne pas mélanger les contextes.

```bash
# Désigne l'hôte de la registry GitLab approuvée, sans schéma HTTPS.
export RIE_REGISTRY=REPLACE_REGISTRY_HOST
# Désigne le chemin complet du projet autorisé, incluant l'hôte précédent.
export RIE_IMAGE_PROJECT=REPLACE_REGISTRY_HOST/REPLACE_GROUP/REPLACE_PROJECT
# Utilise le commit complet et l’architecture native comme étiquette, sans écraser une autre architecture.
export RIE_VERSION="$(git rev-parse HEAD)-$(uname -m)"
# Authentifie l'opérateur par la saisie interactive prévue ; aucun mot de passe en argument.
docker login "$RIE_REGISTRY"
# Retrouve la référence locale publique du runtime construit.
export RIE_BUILT_RUNTIME="$(sed -n 's/^RIE_IMAGE=//p' "$RIE_BUILD_INSTANCE/.env")"
# Retrouve la référence locale publique de NiceGUI construit.
export RIE_BUILT_PORTAL="$(sed -n 's/^RIE_PORTAL_IMAGE=//p' "$RIE_BUILD_INSTANCE/.env")"
# Donne au runtime son nom de publication dans le projet approuvé.
docker tag "$RIE_BUILT_RUNTIME" "$RIE_IMAGE_PROJECT/rie-runtime:$RIE_VERSION"
# Publie le runtime commun aux deux UC et à l'opérateur.
docker push "$RIE_IMAGE_PROJECT/rie-runtime:$RIE_VERSION"
# Donne au portail son nom de publication de la même version.
docker tag "$RIE_BUILT_PORTAL" "$RIE_IMAGE_PROJECT/rie-portal:$RIE_VERSION"
# Publie NiceGUI et sa documentation.
docker push "$RIE_IMAGE_PROJECT/rie-portal:$RIE_VERSION"
# Affiche les digests du runtime reçu par la registry ; retenir le chemin du projet publié.
docker image inspect --format '{{json .RepoDigests}}' "$RIE_IMAGE_PROJECT/rie-runtime:$RIE_VERSION"
# Affiche les digests de NiceGUI pour la fiche de raccordement.
docker image inspect --format '{{json .RepoDigests}}' "$RIE_IMAGE_PROJECT/rie-portal:$RIE_VERSION"
```

**Attendu :** chaque push réussit et retourne un digest `sha256:...`. Remettre
les deux références `chemin@sha256:...`, l’architecture, le commit source, le paquet
correspondant et les résultats de tests. Ces deux références sont saisies dans
NiceGUI à l’étape 6. À l’étape 7, `manage.py pull` récupère les images absentes
du serveur ; `pdl` et `incidents` partagent le même runtime, `portal` utilise NiceGUI. Le serveur d’installation utilise ces
digests ; il n’a besoin que de droits de lecture sur les images.

Pour le **client Open WebUI**, sur une architecture correspondant aux postes :

```bash
# Désigne la base Open WebUI qualifiée, disponible dans le miroir approuvé du SI.
export RIE_OPENWEBUI_BASE=REPLACE_APPROVED_OPENWEBUI_BASE_WITH_DIGEST
# Construit le client avec l'adaptateur OAuth livré ; aucune configuration utilisateur n'entre dans l'image.
docker build --file deploy/openwebui/Dockerfile --build-arg "OPENWEBUI_BASE=$RIE_OPENWEBUI_BASE" --tag "$RIE_IMAGE_PROJECT/rie-openwebui:$RIE_VERSION" deploy/openwebui
# Publie l'image cliente destinée aux postes utilisateurs.
docker push "$RIE_IMAGE_PROJECT/rie-openwebui:$RIE_VERSION"
# Relève sa référence par digest pour les administrateurs des postes.
docker image inspect --format '{{json .RepoDigests}}' "$RIE_IMAGE_PROJECT/rie-openwebui:$RIE_VERSION"
```

La CI GitLab historique `build:arm64` / `build:amd64` publie le runtime PDL
historique. Elle ne publie pas automatiquement les trois images de cette livraison
Ubuntu. Les suites Ubuntu et le workflow GitHub contrôlent le code et les paquets ;
une automatisation interne des trois publications devra reprendre les mêmes cibles,
contrôles et protections de branches, avec des runners et accès fournis par le SI.
La publication dans **la registry réelle du SI-RIE** reste non qualifiée tant que
ces accès et le filtrage effectif ne sont pas disponibles.

## Corriger après le premier déploiement

1. Reproduire le défaut dans une branche du dépôt complet, avec des données
   fictives ou une extraction autorisée. Ne pas modifier les fichiers à l’intérieur
   d’un conteneur en service : ces changements ne constitueraient pas une livraison.
2. Corriger et ajouter une régression utile. Lancer les tests du composant puis
   `ci/runners/run_ubuntu_tests.py`. Actualiser les documents et inventaires concernés.
3. Figer un nouveau commit et construire/publier de nouvelles images. Qualifier
   le changement sur une instance de test, y compris les conversations Albert si
   leur traitement change. Une nouvelle base ou version Open WebUI se requalifie.
4. Fournir nouveau paquet, digests, bilan, éventuelles migrations et procédure de
   retour. Sauvegarder l’instance cible à froid avant l’intervention.
5. Depuis les **nouvelles sources vérifiées**, conserver le dossier d’instance,
   renseigner les nouveaux digests dans son `.env`, puis exécuter `validate`, `pull`
   et `up`. Ne pas refaire `prepare`, `configure-connection` ou `seed` sur cette instance.
6. Vérifier `check`, les droits et les cas métier, puis remettre les clients.
   En cas d’échec, suivre la [restauration vers une instance neuve](07-operations.md#mettre-à-jour-et-revenir-à-létat-sauvegardé)
   avec le paquet et les images de la sauvegarde.

Les modifications de données intervenues depuis la sauvegarde doivent être prises
en compte avant le retour ; elles ne sont pas fusionnées automatiquement.
