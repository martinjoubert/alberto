# Référence d'installation — modes et variantes

> **Pour déployer dans le SI-RIE, commencer par [Guide principal](../README.md).**
> Il n'est pas nécessaire de lire ni d'exécuter cette page 01 avant le guide principal.
> Ne pas faire « cette référence puis le guide » : ce seraient deux parcours d'installation qui se recoupent.

Cette page est une **référence complémentaire** pour choisir un autre mode,
automatiser l'installation ou construire les images. Le numéro `01` est un
identifiant documentaire conservé dans ce répertoire de compléments.

| Document | À quoi sert-il ? | Pour le déploiement SI-RIE actuel |
| --- | --- | --- |
| **[Guide principal](../README.md)** | Une procédure continue : préparation, installation, raccordement, Open WebUI, deux UC, ACL et exploitation | **Seul parcours à dérouler, de l'étape 1 à l'étape 12** |
| **01 — Cette référence** | Les variantes `demo`/`external`, l'installateur scripté, la construction locale et les commandes techniques complémentaires | À consulter ponctuellement si le guide y renvoie ou si une variante est explicitement choisie |

Les deux pages utilisent les mêmes composants et certaines commandes communes.
Elles ne décrivent pas exactement la même méthode : le guide principal utilise les
services SI existants et les **images approuvées de la registry GitLab** ;
les variantes ci-dessous peuvent créer un SI de démonstration ou **construire
les images localement**. Ne pas mélanger leurs commandes.

Le serveur Ubuntu héberge PDL, Incidents et leurs ACL. **Open WebUI reste sur
chaque poste utilisateur.** NiceGUI est un portail facultatif d'administration,
de démonstration et de qualification. La [fiche de raccordement](26-si-connection-sheet.md)
se complète avant installation ; la [réception SI-RIE](14-si-rie-acceptance.md)
décrit les contrôles à effectuer dans le SI réel.

## Quand utiliser les variantes de cette référence

Pour une première installation externe ou la sandbox avec SI déjà disponible, utiliser le
[déroulé continu](../README.md), qui fournit une seule numérotation
du début à la fin. Ce manuel détaille les variantes **scriptée ou manuelle**,
**demo ou external** ; ses sections A/B sont des références, pas des étapes
supplémentaires à exécuter après le guide principal.

Utiliser la suite de cette page uniquement pour une variante retenue avec le
responsable du déploiement : démonstration autonome, automatisation ou préparation
des images. La configuration des sources externes précède leur démarrage.

<a id="choisir-le-parcours-avant-de-commencer"></a>

## Choisir une variante avant de l'exécuter

| Choix | Quand l'utiliser | Suite à suivre |
| --- | --- | --- |
| `demo` | Vérifier l'installation avec données et identités fictives | Corpus embarqué, sans accès au SI réel ni Albert nécessaire |
| `external` | Raccorder les sources et le LDAP du SI | Paramètres relus par leurs responsables, aucun seed de démonstration |
| Installation scriptée | Variante d'automatisation explicitement choisie | Préparation commune, puis **A** ; ne pas exécuter aussi **B** ni le guide principal en parallèle |
| Installation manuelle | Examiner chaque phase ou qualifier le manuel | Préparation commune, puis **B** ; ne pas rejouer ensuite **A** |

Exécuter un bloc à la fois et vérifier son résultat avant le suivant. À la
première erreur, arrêter le parcours, corriger la cause puis reprendre la phase
concernée. Conserver les sorties techniques, sans mots de passe ni jetons.

## Préparation commune — serveur Ubuntu

**Droits :** compte installateur avec `sudo` et accès SSH. Les commandes
administratives ci-dessous utilisent `sudo` de façon cohérente. Elles conservent
un chemin d'instance développé par le shell utilisateur, jamais un chemin
implicite sous `/root`. Un opérateur déjà autorisé Docker peut omettre `sudo`
s'il reste propriétaire des fichiers privés ; ne pas alterner les propriétaires.

**Réseau :** prévoir APT Ubuntu, `download.docker.com`, les registres Docker et
PyPI, ou leurs miroirs/proxy/CA approuvés. Ne pas désactiver TLS. L'accès Albert
est distinct et n'est pas requis pour la recette technique. Le laboratoire est
ARM64 ; les performances AMD64 et le raccordement aux sources du SI restent à
qualifier.

### Récupérer une version identifiée

Pour une nouvelle installation, utiliser la [livraison indiquée au début du guide](../README.md#avant-de-commencer). Les versions datées des preuves restent historiques.

Depuis le dossier `$HOME/deploy` contenant le paquet d'installation et son
empreinte, choisir une seule fois le nom exact de l'archive reçue :

```bash
# Se place dans le dossier contenant le paquet d'installation et son fichier SHA-256.
cd "$HOME/deploy"
# Mémorise le nom exact de l'archive fournie ; adapter cette valeur au fichier effectivement reçu.
export RIE_ARCHIVE=rie-si-rie-ubuntu-2026-11-19-si-deploy-r12.tar.gz
# Compare l'empreinte de l'archive au fichier SHA-256 reçu ; attendre OK avant toute extraction.
sha256sum -c "$RIE_ARCHIVE.sha256"
# Choisit un dossier neuf pour extraire les sources ; cette ligne ne crée pas le dossier.
export RIE_SOURCES="$HOME/rie-deploy-sources-2026-11-19-r12"
# Crée le dossier d'extraction ; si ce nom existe déjà, choisir un dossier neuf avant de continuer.
mkdir "$RIE_SOURCES"
# Décompresse les sources dans le dossier choisi ; n'installe et ne démarre aucun service.
tar -xzf "$RIE_ARCHIVE" -C "$RIE_SOURCES"
# Se place à la racine des sources extraites, d'où lancer les commandes deploy/ubuntu.
cd "$RIE_SOURCES/rie-delivery"
# Vérifie les fichiers extraits contre leur manifeste ; attendre passed: true avant de les utiliser.
python3 deploy/ubuntu/verify_delivery.py .
```

**Attendu :** archive `OK`, puis `"passed": true`. Adapter le nom de l'archive
au paquet effectivement reçu ; choisir un dossier d'extraction neuf. Le vérificateur
contrôle chaque fichier, taille et SHA-256, et refuse ajouts, absences et liens.
Le lancer avant toute construction. Un écart impose une nouvelle remise, pas
une modification du manifeste. Sur Mac, le contrôle préalable équivalent est
`shasum -a 256 -c "$RIE_ARCHIVE.sha256"` ; le refaire après transfert sur Ubuntu.
Une empreinte ne remplace pas la confiance dans le canal de remise.

#### Interpréter le résultat de vérification du paquet

| Champ affiché | Ce qu'il indique |
| --- | --- |
| `passed: true` | Tous les fichiers reçus correspondent au manifeste : liste, taille et SHA-256 ; aucun ajout, manque ou lien n'a été accepté |
| `source.commit` | Commit Git de base du dépôt ayant servi à fabriquer le paquet ; si `dirty` est vrai, le manifeste décrit les fichiers supplémentaires ou modifiés effectivement livrés |
| `source.dirty: true` | Au moment de fabriquer l'archive, le dépôt d'origine avait des modifications non commitées ou des fichiers non suivis ; cela ne décrit pas l'état de la VM Ubuntu |
| `source.dirty: false` | Le dépôt d'origine était propre par rapport au commit indiqué ; cela ne prouve pas que les scénarios métier ont été validés |
| `source.dirty: null` | L'état Git n'était pas disponible lors de la fabrication ; l'intégrité des fichiers reste vérifiable par le manifeste |
| `authenticity: not_verified_by_checksum` | La comparaison des empreintes ne vérifie pas de signature de l'éditeur ; l'origine doit être établie par le canal de distribution et sa référence de confiance |

**Pourquoi `dirty` valait `true` dans les anciennes recettes ?** Ces paquets comprenaient les
renommages et adaptations documentaires en cours de préparation, avant leur
commit final. Ce statut est enregistré honnêtement. Le commit affiché est la
base du travail, tandis que le manifeste identifiait précisément le contenu du
paquet de recette. La livraison figée indique `dirty: false`. Les anciens
fichiers extraits déjà vérifiés restent à conserver
avec leur manifeste ; les paramètres du SI se modifient dans `RIE_INSTANCE`.

**Intégrité et authenticité sont deux contrôles distincts.** Les empreintes
permettent de comparer les octets au référentiel reçu. Pour confirmer qui a
fourni ce référentiel, utiliser le canal de distribution approuvé et une
empreinte de référence obtenue par un canal de confiance indépendant, ou une
signature vérifiée. Ce vérificateur ne traite pas de signature : la mention
`not_verified_by_checksum` reste donc affichée même lorsque `passed` est vrai.

La livraison finale au SI doit identifier une version figée avec un commit
propre et un nouveau paquet cohérent avec son manifeste. Le paquet de travail
de cette séance reste une preuve distincte, avec ses métadonnées réelles.

Si la remise fournit **un dépôt GitHub et un commit complet** plutôt qu'une
archive, cloner ce dépôt et sélectionner ce commit avec `git checkout --detach
COMMIT_REMIS`, puis vérifier `git rev-parse HEAD`. Ne pas déployer une branche
mobile. `verify_delivery.py` concerne l'archive avec manifeste, pas un checkout
Git qui contient `.git`. Dans les deux cas, rester à la racine des sources.

```bash
# Affiche la distribution et sa version ; attendu pour cette recette : Ubuntu 24.04 LTS.
cat /etc/os-release
# Affiche l'architecture du processeur : aarch64 sur cette VM, x86_64 sur un serveur AMD64.
uname -m
# Affiche la version de Python ; l'installateur exige Python 3.10 ou plus.
python3 --version
```

**Attendu :** Ubuntu 24.04/26.04 LTS, `aarch64` ou `x86_64`, Python 3.10 ou plus.
Si Python manque, le faire installer avant le script. Conserver le manifeste
ou le commit avec le compte rendu. Prévoir pour la recette 8 vCPU, 16 Go RAM et
80 Go disque ; ces valeurs décrivent le laboratoire, pas le dimensionnement du SI.
L'instance doit rester hors du code, dans un chemin absolu sans espace.

Choisir **un seul** des deux blocs suivants et conserver ces variables dans la
session qui exécutera A ou B. Pour les sources réelles du **SI-RIE**, remplacer
`LDAP_ADMIN_USERNAME` par **le véritable identifiant d'un compte existant dans
le LDAP cible**, habilité à administrer les accès du PoC :

```bash
# Choisit le dossier privé de la future instance du PoC, séparé des sources ; ne le crée pas encore.
export RIE_INSTANCE="$HOME/.local/share/dinum-rie-si"
# Choisit external : le PoC utilisera les services SI déjà disponibles, sans les recréer.
export RIE_MODE=external
# Désigne un vrai compte du LDAP cible habilité à administrer les accès du PoC ; remplacer LDAP_ADMIN_USERNAME.
export RIE_ADMIN_LDAP='LDAP_ADMIN_USERNAME'
```

Pour un **apprentissage isolé avec données fictives** uniquement :

```bash
# Choisit le dossier privé du PoC ; exemple demo à adapter au chemin de votre instance.
export RIE_INSTANCE="$HOME/.local/share/dinum-rie-demo"
# Choisit demo : l'installation créera aussi les sources et identités fictives.
export RIE_MODE=demo
# Utilise Alice, compte LDAP fictif prévu pour l'administration des accès du PoC dans la simulation.
export RIE_ADMIN_LDAP=alice.martin
```

`RIE_MODE` et `RIE_ADMIN_LDAP` sont ici des variables de commande. Le compte
administrateur n'est pas le compte à partager avec les utilisateurs. Pour
passer de `demo` à `external`, préparer une autre instance ; ne pas changer le
mode d'un dossier existant. Si les deux tournent sur le même serveur, prévoir
des ports distincts selon la configuration.

Le compte désigné par `RIE_ADMIN_LDAP` doit être **interne et membre direct d'un
groupe de `ldap.administrator_group_dns`**. La configuration LDAP et les règles
actives portent ses habilitations ; définir la variable indique quel compte
utiliser. Ce rôle concerne les accès du PoC et ne nécessite pas les droits
d'administration technique de l'annuaire. Le compte Unix avec `sudo` et le
compte lecteur LDAP sont distincts de ce rôle. Les lectures restent soumises
aux ACL des sources. Voir la [liste des valeurs à adapter](26-si-connection-sheet.md).

Avant `prepare`, il s'agit d'un **choix**, pas d'un mode déjà actif sur Ubuntu.
Après préparation, la référence est `RIE_MODE` dans `$RIE_INSTANCE/.env`, lisible
sans afficher les autres paramètres avec
`sudo grep '^RIE_MODE=' "$RIE_INSTANCE/.env"`.
Voir [choisir ou retrouver le mode](02-configuration.md#choisir-le-mode).

## A. Variante d’installation scriptée

### A1. Vérifier les phases prévues

```bash
# Affiche les phases prévues de l'installation scriptée ; ne crée aucun fichier ni service.
sudo bash deploy/ubuntu/install.sh --instance "$RIE_INSTANCE" --mode "$RIE_MODE" --username "$RIE_ADMIN_LDAP" --dry-run
```

**Attendu :** phases affichées, aucun service ou fichier d'instance créé.
Ajouter `--with-demo-ui` si NiceGUI est souhaité ; l'omettre dans une installation
sans portail. Le portail dispose de son image facultative, construite seulement
lorsqu'il est activé.

### A2. Installer et contrôler

**En `external`, avant de lancer l'installation :** exécuter les étapes 1 à 4
de [Configuration — raccorder des sources externes](02-configuration.md#raccorder-des-sources-externes).
Elles préparent l'instance neuve et renseignent ses fichiers avec les valeurs
du SI. Lancer ensuite `sudo python3 deploy/ubuntu/manage.py validate "$RIE_INSTANCE"`.
Le script conserve cette instance préparée. Pour une reprise, ne pas rejouer
`prepare` ni recopier les modèles par-dessus la configuration existante.

Sur un Ubuntu neuf :

```bash
# Exécute l'installation scriptée et autorise explicitement la préparation des paquets système et de Docker.
sudo bash deploy/ubuntu/install.sh --instance "$RIE_INSTANCE" --mode "$RIE_MODE" --username "$RIE_ADMIN_LDAP" --install-prerequisites
```

Ajouter `--with-demo-ui` pour les démonstrations. Si Docker, Compose, Buildx et
les outils système sont déjà présents, omettre `--install-prerequisites`.
Cette option autorise explicitement l'installation des prérequis ; le script
ne crée ni compte système, ni groupe Docker, ni règle de pare-feu.

**Attendu en `demo` :** préparation privée, images, démarrage, initialisation des corpus,
`"status": "ready"`, puis recette `"passed": true`. Ces contrôles sont déjà
faits par le script : ne pas rejouer les phases manuelles. Les quatre familles
PDL sont PFS, BB, PFAING et TACACS ; Incidents utilise le 28 septembre 2026.

**Reprise :** corriger l'étape en erreur et relancer la même commande avec le
même chemin. Les secrets, règles, mappings et données sont conservés. Ne pas
supprimer l'instance ou des volumes pour résoudre un refus d'identité.

**Attendu en `external` :** le script refuse les marqueurs incomplets et ne lance
ni `seed` ni la recette fictive. Sur une instance externe neuve, le dernier `check` retourne normalement
le code **2** et `healthy_not_initialized` tant que le catalogue n'a pas été revu :
les services sont démarrés, l'installation métier reste à terminer. Si le refus
concerne l'identité, le réseau ou un fichier invalide, corriger d'abord cette cause.
Avec l'administrateur LDAP indiqué, suivre [la revue des mappings](04-pdl.md#revoir-puis-approuver-les-mappings),
puis exécuter `sudo python3 deploy/ubuntu/manage.py check "$RIE_INSTANCE" --username "$RIE_ADMIN_LDAP"`.
Attendre `ready` et code 0 avant de passer au **raccordement du poste utilisateur**.
Il n'est pas nécessaire de reconstruire les images pour terminer cette revue.

## B. Installation manuelle — alternative au script

### B1. Préparer Docker et les outils

Sur un hôte déjà équipé, vérifier Docker et ses plugins sans réinstaller :

```bash
# Vérifie l'accès au moteur Docker et affiche les versions du client et du serveur.
sudo docker version
# Vérifie que le plugin Docker Compose nécessaire au démarrage est installé.
sudo docker compose version
# Vérifie que le plugin Buildx nécessaire à la construction des images est installé.
sudo docker buildx version
# Affiche l'espace disque occupé et disponible sur le stockage Docker ; ne supprime rien.
df -h /var/lib/docker
# Lit la limite mémoire virtuelle du noyau pour OpenSearch ; ne modifie aucun réglage.
sysctl vm.max_map_count
```

**Attendu :** accès au daemon, Compose avec `--wait` et `run --interactive`,
Buildx disponible et espace suffisant. Les versions relevées le 6 novembre
2026 sont Docker 29.9.3, Compose 5.7.0, Buildx 0.38.1 ; ce sont des relevés,
pas une consigne de désactiver leurs mises à jour de sécurité.

Sur un Ubuntu neuf, l'administrateur applique les mêmes prérequis que le script :

```bash
# Installe les prérequis système manquants et Docker, puis vérifie le réglage OpenSearch ; ne déploie pas le PoC.
sudo python3 deploy/ubuntu/prerequisites.py --install
```

Cette commande installe les outils APT et le dépôt Docker officiel si absent,
active Docker et conserve une installation existante. Pour examiner les commandes
système une par une, lire `prerequisites.py` et la [procédure officielle Docker
Ubuntu](https://docs.docker.com/engine/install/ubuntu/). L'outil ne réduit jamais
un `vm.max_map_count` déjà supérieur au minimum OpenSearch de 262144 et persiste
le réglage uniquement si nécessaire. Un échec Docker se résout avant Albert.

### B2. Préparer l'instance et ses options

```bash
# Crée le dossier privé, la configuration initiale et les secrets du PoC ; ne démarre aucun service.
sudo python3 deploy/ubuntu/manage.py prepare "$RIE_INSTANCE" --mode "$RIE_MODE"
```

**Attendu :** instance préparée, aucun service démarré ; dossiers `0700`, fichiers
privés `0600`. La commande refuse de réinitialiser un dossier rempli. Pour une
reprise, conserver cette instance et passer à la phase en échec.

Si le portail NiceGUI est souhaité :

```bash
# Active le portail NiceGUI facultatif dans la configuration ; ne le démarre pas encore.
sudo python3 deploy/ubuntu/manage.py configure-ui "$RIE_INSTANCE" --enable-demo-ui
```

Les ports par défaut sont PDL 19910, Incidents 19950 et NiceGUI 18090, tous liés
à la boucle locale. Adapter `.env` avec `sudoedit` s'ils sont occupés. En
`external`, remplir maintenant la
[configuration externe](02-configuration.md#raccorder-des-sources-externes)
avant B3 : l'instance vient d'être préparée, donc ne pas rejouer sa commande
`prepare`. Commencer par la copie des modèles de son étape 2, puis suivre les
étapes 3 et 4. Pour une reprise, conserver les fichiers déjà configurés.

### B3. Valider, construire et démarrer

```bash
# Contrôle les fichiers et paramètres privés avant application ; ne démarre rien et n'appelle pas Albert.
sudo python3 deploy/ubuntu/manage.py validate "$RIE_INSTANCE"
# Construit les images nécessaires au profil choisi, dont NiceGUI si activé ; ne démarre aucun service.
sudo python3 deploy/ubuntu/manage.py build "$RIE_INSTANCE"
# En demo, récupère OpenSearch/VictoriaMetrics ; en external, réutilise les images du projet construites ci-dessus.
sudo python3 deploy/ubuntu/manage.py pull "$RIE_INSTANCE"
# Démarre ou relance les services de cette instance et recharge sa configuration en conservant ses données.
sudo python3 deploy/ubuntu/manage.py up "$RIE_INSTANCE"
```

**Attendu :** configuration validée, images disponibles, processus sains.
La construction peut prendre plusieurs minutes. Le contexte Docker contient
les seuls fichiers publics sélectionnés. Le runtime et les outils opérateur
utilisent l'image minimale ; NiceGUI et les outils de qualification sont des
cibles distinctes. `up` ne télécharge ni ne construit implicitement les images.
Avant initialisation, des processus sains ne signifient pas encore `ready`.

**Dépannage :** image absente → reprendre `build`/`pull` ; identité Docker ou
volume différente → retrouver le bon contexte. Ne pas effacer les volumes.

### B4. Initialiser et recetter la simulation

**Uniquement en mode `demo` :**

```bash
# En demo uniquement, initialise les données fictives et active les mappings embarqués ; aucun appel Albert.
sudo python3 deploy/ubuntu/manage.py seed "$RIE_INSTANCE"
# Sonde les sources, le LDAP et les mappings ; ready signifie disponibilité métier, sans validation de génération Albert.
sudo python3 deploy/ubuntu/manage.py check "$RIE_INSTANCE"
# En demo uniquement, exécute la recette technique et ACL et enregistre son rapport ; sans génération Albert.
sudo python3 deploy/ubuntu/manage.py acceptance "$RIE_INSTANCE"
```

**Attendu :** corpus cohérent, quatre mappings PDL actifs, `ready`, recette
réussie sans appel Albert. Le nombre de contrôles dépend de la version remise ;
le bilan de qualification l'identifie. Un contenu divergent est refusé, pas
remplacé silencieusement. Avant le seed, `healthy_not_initialized` est normal.

**En mode `external` :** ne lancer ni `seed` ni `acceptance`. Suivre les revues
[PDL](04-pdl.md#revoir-puis-approuver-les-mappings), les droits
[ACL](06-acl.md), puis
`sudo python3 deploy/ubuntu/manage.py check "$RIE_INSTANCE" --username "$RIE_ADMIN_LDAP"`.
Attendre `ready` avant le raccordement du poste. Une génération IA
réelle conserve sa recette séparée.

## Raccorder le poste utilisateur — après A ou B

**Parcours utilisateur courant :** préparer la
[connexion LDAP et OAuth sans jeton à copier](25-user-access.md), puis jouer
les deux UC. L'utilisateur autorise les sources dans son navigateur ; Open
WebUI conserve cette autorisation et renouvelle les accès courts. Sur la
sandbox, reprendre directement l'[étape 10 du déroulé](../README.md#10-connecter-nicegui-et-open-webui-local).
La variante Bearer ci-dessous est conservée pour scripts/opérateurs et
installations historiques. Choisir une seule authentification par connexion.

Sur le serveur, remplacer `LDAP_USERNAME` par le compte LDAP réel
de l'utilisateur, puis émettre **son** jeton :

```bash
# Émet un jeton pour ce seul titulaire LDAP et le sauvegarde dans un fichier privé ; ne l'affiche pas.
sudo python3 deploy/ubuntu/manage.py issue-token "$RIE_INSTANCE" --username LDAP_USERNAME
```

**Attendu :** chemin d'un fichier privé, secret jamais affiché. Remettre ce fichier au seul titulaire
par le canal interne prévu. En `demo`, les mots de passe NiceGUI aléatoires se
récupèrent compte par compte selon [ACL — préparer une connexion fictive](06-acl.md#pr%C3%A9parer-une-connexion-fictive).
Le lecteur fictif `bruno.lecteur` convient pour les questions de démonstration ;
il ne désigne pas un compte à créer ou rechercher dans le LDAP du SI.

L'administrateur prépare un compte SSH de tunnel distinct du compte Docker,
avec les seules redirections autorisées par le SI. Sur le **poste utilisateur** :

```bash
# Sur le poste utilisateur, ouvre les tunnels PDL/Incidents et garde cette session active ; ne lance aucune commande distante.
ssh -N -o ExitOnForwardFailure=yes -L 19910:127.0.0.1:19910 -L 19950:127.0.0.1:19950 SSH_USERNAME@UBUNTU_SERVER
```

Garder cette session ouverte. Si NiceGUI est activé, ajouter
`-L 18090:127.0.0.1:18090` à la commande, puis ouvrir `http://127.0.0.1:18090`.

Dans l'Open WebUI local, créer les deux connexions MCP HTTP, Bearer individuel :

| Connexion | URL vue depuis le backend Open WebUI |
| --- | --- |
| PDL | `http://127.0.0.1:19910/mcp` |
| Incidents | `http://127.0.0.1:19950/gptoss/assistant/mcp` |

Pour un client conteneurisé, `127.0.0.1` désigne son conteneur : employer le
raccordement local décrit dans [Configuration Open WebUI](02-configuration.md#configurer-le-client-open-webui-local-qualifi%C3%A9).
Ne pas ouvrir le tunnel sur `0.0.0.0` pour résoudre ce point.
Importer les deux filtres et modèles dédiés de cette même page ; les connexions
MCP seules ne reproduisent pas le suivi et l'affichage qualifiés.

**Contrôle de connexion :** dans NiceGUI, si activé, se connecter avec le compte
de l'utilisateur et lire ses droits et son catalogue approuvé. Vérifier les
connexions MCP depuis le backend Open WebUI selon le guide client ; la seule
ouverture du portail ne valide pas ce raccordement. Le mot de passe NiceGUI
authentifie sa session ; le jeton personnel authentifie les appels MCP.

## Exécuter les deux UC et vérifier les ACL

### Avec les sources réelles du SI-RIE

Suivre cette séquence en mode `external`, après configuration du
[fournisseur IA](02-configuration.md#raccorder-albert-ou-une-ia-interne) et du
client. Les questions sont des modèles : remplacer tous les champs entre
crochets par les valeurs préparées avec l'équipe du SI.

1. **PDL :** suivre [Utiliser vos sources SI-RIE](16-pdl-tutorial.md#utiliser-vos-sources-si-rie).
   Jouer un comptage puis une recherche d'équipement dans les familles
   raccordées. Comparer filtres, chiffres et preuves à une recherche manuelle
   connue sur la même période. Tester aussi une clarification dans le même fil.
2. **Incidents :** suivre [Utiliser vos sources SI-RIE](17-incidents-tutorial.md#utiliser-vos-sources-si-rie).
   Diagnostiquer un équipement réel sur une période connue, puis poser une
   question de suivi. Comparer mesures, logs, documents et tickets aux sources ;
   distinguer les données absentes, les hypothèses et les causes confirmées.
   Les boutons NiceGUI fixés sur R1 et les dates fictives ne valident pas cet
   incident : utiliser la question adaptée dans Open WebUI.
3. **ACL dans les deux UC :** rejouer les questions avec les comptes de recette
   autorisé et non autorisé, chacun avec son jeton personnel. Comparer accès et
   refus à la politique active. Pour Incidents, vérifier les deux index et les
   ressources `incidents.metrics` et `incidents.tickets`. Les essais de retrait
   et de rétablissement des droits utilisent une instance ou des sources de
   recette convenues avec l'administrateur, puis restaurent la règle initiale.
4. **Génération IA :** relever le fournisseur et le modèle réellement utilisés,
   contrôler les chiffres et citations, puis consigner erreurs et limites.
   Une lecture technique sans synthèse ne valide pas une réponse IA. Si le
   fournisseur est indisponible, noter « génération non qualifiée ».

Aucun compte, équipement, total ni horaire du corpus fictif n'est un attendu
pour ces essais. Une identité interne ou externe ne prédit pas à elle seule
le résultat ACL : la politique effectivement configurée fait référence.

### Avec le corpus fictif de démonstration

En mode `demo`, après initialisation et recette technique réussies, suivre
les parcours [PDL](16-pdl-tutorial.md#avant-de-commencer-la-démonstration) et
[Incidents](17-incidents-tutorial.md#avant-de-commencer-la-démonstration), y
compris leur étape ACL. Les comptes, dates et chiffres fournis s'appliquent à
ce seul corpus. Le [guide de démonstration](09-demonstrations.md) organise une
présentation courte. Noter ces résultats comme des essais sur données fictives.

## Consigner la réception et transmettre à l'exploitation

Compléter la [fiche de recette manuelle](12-manual-acceptance.md) et la
[matrice SI-RIE](14-si-rie-acceptance.md#matrice-de-vérification-dans-le-si-rie)
avec version, mode, machine, comptes de test, sources, fenêtres, questions,
résultats observés et preuves sans secrets. Identifier les essais non exécutés
et leurs responsables. Le statut `ready` ou une démonstration réussie ne vaut
pas acceptation des données, droits, capacités IA et charges réelles.

Pour le quotidien, poursuivre avec [Exploitation](07-operations.md).
Le [journal historique](11-installation-log.md)
explique les reprises de la campagne du 6 novembre ; il n'est pas une liste
supplémentaire de tâches à exécuter.
