# Déployer PDL, Incidents et les ACL dans le SI-RIE — pas à pas

Suivre les **étapes 1 à 12** dans l’ordre. Ce guide installe les deux moteurs
Python et le portail NiceGUI sur Ubuntu, en **mode `external`** : OpenSearch,
VictoriaMetrics, LDAP et ITSM-NG sont déjà fournis par le SI. Open WebUI reste
installé sur chaque poste utilisateur.

Le même parcours s’utilise dans une [sandbox préparée](reference/22-external-sandbox.md).
Les questions de l’étape 11 donnent des exemples copiables pour son corpus
fictif. Dans le SI réel, utiliser les paramètres et cas de recette validés par
ses responsables. Les [compléments](reference/README.md) détaillent les sujets
auxquels une étape renvoie.

## Avant de commencer

Télécharger l’[archive source](https://github.com/martinjoubert/alberto/raw/refs/heads/main/rie-si-rie-ubuntu-2026-11-18-si-deploy-r10.tar.gz)
et [son fichier SHA-256](https://github.com/martinjoubert/alberto/raw/refs/heads/main/rie-si-rie-ubuntu-2026-11-18-si-deploy-r10.tar.gz.sha256).
Le [paquet documentaire Wiki.js](https://github.com/martinjoubert/alberto/raw/refs/heads/main/rie-si-rie-wikijs-2026-11-18-si-deploy-r10.zip)
permet de conserver les pages et diagrammes hors du dépôt.

<a id="versions-à-utiliser-pour-cette-livraison"></a>
<a id="préparation-des-images--responsable-de-la-construction-avant-létape-1"></a>

### Éléments à obtenir

| Élément | À faire préparer par le responsable concerné |
| --- | --- |
| Serveur | Ubuntu 24.04 ou 26.04, Python 3.10 ou plus, Docker et Compose ; compte SSH avec `sudo`, espace disque adapté au déploiement et aux sauvegardes |
| Sources du PoC | Archive et SHA-256 correspondants, référence Git et manifeste du reçu de livraison |
| Images | Deux images serveur approuvées — moteurs et NiceGUI — disponibles dans la registry GitLab du SI, avec leur empreinte `sha256` et l’architecture du serveur |
| Raccordement au SI | [Fiche de raccordement](reference/26-si-connection-sheet.md) : adresses, certificats, comptes lecteurs, groupes, droits, fichiers de données et demandes de recette ; secrets remis séparément |
| Accès réseau | Lecture de la registry et des sources depuis Ubuntu ; URLs HTTPS des deux moteurs joignables depuis les postes et Open WebUI |
| Postes utilisateurs | Open WebUI 0.11.3 avec l’image cliente et l’adaptateur OAuth fournis, données sauvegardées, accès Albert et administrateur local disponible |
| Ticketing | ITSM-NG 1.6.17, API et accès individuels préparés pour les utilisateurs de recette |

La **fiche de raccordement** rassemble les valeurs à obtenir auprès du SI.
Elle sert à remplir l’assistant NiceGUI de l’étape 6. Les mots de passe et
jetons sont fournis dans des fichiers privés ; ils restent hors du wiki et
du compte rendu. La construction des images est confiée au mainteneur selon
la [procédure de publication](reference/27-development-and-images.md), avant l’installation.

Exécuter les commandes **une par une**, sur la machine indiquée, puis vérifier
l’attendu. Chaque ligne `#` décrit la commande suivante. Remplacer les valeurs
`REPLACE_...`, `SSH_LOGIN`, `UBUNTU_HOST` et `CLIENT_COMPOSE` avant exécution.
En cas d’erreur, conserver le message sans secret et corriger avant de poursuivre.

### Variables du terminal

Les `export` mémorisent les chemins et identifiants pour le terminal courant.
La configuration des services est enregistrée dans l’instance privée : fermer
le terminal ne l’efface pas et n’arrête pas les services. Pour reprendre une
session, voir [Reprendre après une déconnexion SSH](#reprendre-après-une-déconnexion-ssh).

| Étape | Action | Machine | Attendu avant de continuer |
| --- | --- | --- | --- |
| [1](#1-identifier-ubuntu-et-vérifier-le-si-existant) | Identifier le serveur et les fichiers reçus | Ubuntu | Bon serveur, Python et `sudo` disponibles |
| [2](#2-vérifier-et-extraire-le-paquet) | Vérifier et extraire les sources | Ubuntu | Archive `OK`, manifeste `passed: true` |
| [3](#3-choisir-instance-mode-et-administrateur) | Choisir l’instance et l’administrateur | Ubuntu | Destination neuve, compte LDAP habilité |
| [4](#4-vérifier-les-prérequis) | Vérifier Docker et le stockage | Ubuntu | Docker/Compose fonctionnels, espace disponible |
| [5](#5-préparer-linstance-privée) | Créer la configuration privée | Ubuntu | Permissions `700`, `700`, `600` |
| [6](#6-préparer-le-raccordement-avec-nicegui) | Renseigner et importer le raccordement | Navigateur et Ubuntu | Dossier vérifié, secrets installés |
| [7](#7-valider-récupérer-les-images-et-démarrer) | Télécharger les images et démarrer | Ubuntu | `pdl`, `incidents`, `portal` sains |
| [8](#8-revoir-et-approuver-les-mappings-pdl) | Approuver les correspondances PDL | Navigateur et Ubuntu | Catalogue utilisable, contrôle `ready` |
| [9](#9-raccorder-les-précédents-itsm-ng) | Activer la recherche de précédents ITSM | Ubuntu | Accès API individuel vérifié |
| [10](#10-connecter-nicegui-et-open-webui-local) | Connecter Open WebUI aux deux moteurs | Ubuntu et poste | Modèles visibles, autorisations personnelles actives |
| [11](#11-jouer-les-deux-uc-et-leurs-acl-avec-albert) | Vérifier les réponses et les droits | Open WebUI et NiceGUI | Résultats comparés aux sources, refus conformes |
| [12](#12-vérifier-lexploitation-et-consigner) | Vérifier sauvegarde et reprise | Ubuntu | Restauration réussie, compte rendu renseigné |

## Les services du PoC et leur utilisation

| Service | Usage |
| --- | --- |
| `pdl` | Recherche et rapprochement des logs depuis le modèle **SI-RIE — PDL** d’Open WebUI |
| `incidents` | Diagnostic, preuves et précédents depuis **SI-RIE — INCIDENTS** |
| `portal` | NiceGUI : mappings, ACL, documentation, démonstrations et rapports de tests |

Les ACL sont communes aux deux moteurs. Les tâches `initialize` et `operator`
sont exécutées automatiquement par les commandes du guide. Albert est joint
par API. L’[architecture](reference/03-architecture.md) donne les ports et flux détaillés.

## 1. Identifier Ubuntu et vérifier le SI existant

**Où : terminal SSH ouvert sur Ubuntu.** Comparer la machine et le compte à la fiche.

```bash
# Affiche le compte Unix connecté.
whoami
# Affiche le nom du serveur.
hostname
# Affiche la distribution Ubuntu et sa version.
cat /etc/os-release
# Affiche l’architecture : aarch64 pour ARM64, x86_64 pour AMD64.
uname -m
# Vérifie Python ; version 3.10 ou plus attendue.
python3 --version
# Vérifie les droits sudo ; résultat attendu : 0.
sudo id -u
# Désigne le dossier reçu avec les paramètres et fichiers du SI ; adapter son emplacement si nécessaire.
export RIE_SI="$HOME/si-connection"
# Vérifie l’existence de ce dossier sans afficher ses fichiers privés.
ls -ld "$RIE_SI"
```

**Attendu :** serveur et architecture prévus, Python disponible, `sudo` autorisé
et dossier de raccordement présent. Les connexions aux sources seront testées
après leur configuration.

## 2. Vérifier et extraire le paquet

**Où : même terminal Ubuntu.** Les sources et l'instance doivent rester
distinctes. Ne pas extraire par-dessus un dossier existant.

```bash
# Entre dans le dossier de réception du paquet ; adapter si le SI l'a remis ailleurs.
cd "$HOME/deploy"
# Désigne le nom exact de l'archive de cette livraison, à comparer au reçu.
export RIE_ARCHIVE=rie-si-rie-ubuntu-2026-11-18-si-deploy-r10.tar.gz
# Vérifie l'archive contre son fichier SHA-256 reçu ; attendre OK.
sha256sum -c "$RIE_ARCHIVE.sha256"
# Choisit un dossier neuf pour cette version des sources.
export RIE_SOURCES="$HOME/rie-deploy-sources-2026-11-18-r10"
# Crée le dossier ; s'il existe, choisir un autre nom avant extraction.
mkdir "$RIE_SOURCES"
# Extrait les sources ; aucun logiciel ni service n'est installé.
tar -xzf "$RIE_ARCHIVE" -C "$RIE_SOURCES"
# Se place à la racine des sources, utilisée pour les commandes suivantes.
cd "$RIE_SOURCES/rie-delivery"
# Vérifie la liste, les tailles et les empreintes des fichiers extraits.
python3 deploy/ubuntu/verify_delivery.py .
```

**Attendu :** archive `OK`, puis `passed: true` et `source.dirty: false`.
Comparer la référence Git et l’empreinte du manifeste au reçu de livraison.
`authenticity: not_verified_by_checksum` signifie que l’empreinte contrôle
l’intégrité ; l’origine se confirme par le canal de distribution approuvé.
Le [détail des champs](reference/01-installation.md#interpréter-le-résultat-de-vérification-du-paquet)
est disponible en complément.

## 3. Choisir instance, mode et administrateur

**Où : Ubuntu, racine des sources.** Remplacer l'identifiant LDAP avant exécution.

```bash
# Choisit le dossier privé de la nouvelle instance ; cette commande ne le crée pas.
export RIE_INSTANCE="$HOME/.local/share/dinum-rie-si"
# Raccorde les services déjà disponibles, sans créer leurs bases ou comptes.
export RIE_MODE=external
# Désigne le véritable administrateur des accès du PoC, fourni dans la fiche.
export RIE_ADMIN_LDAP=REPLACE_LDAP_ADMIN_LOGIN
# Affiche seulement ces choix non secrets, pour les relire avant préparation.
printf 'Instance : %s\nMode : %s\nAdministrateur LDAP : %s\n' "$RIE_INSTANCE" "$RIE_MODE" "$RIE_ADMIN_LDAP"
```

**Attendu :** destination neuve, mode `external` et véritable compte LDAP,
avec les droits d’administration du PoC confirmés dans la fiche. Ce compte
est distinct du compte Unix SSH et du lecteur technique de l’annuaire.

## 4. Vérifier les prérequis

**Où : Ubuntu.** Docker et Compose doivent être installés par l’exploitant.

```bash
# Vérifie que le client et le moteur Docker sont accessibles avec sudo.
sudo docker version
# Vérifie Docker Compose.
sudo docker compose version
# Affiche le dossier de stockage de Docker.
sudo docker info --format '{{.DockerRootDir}}'
# Affiche l’espace disponible ; repérer le disque contenant le dossier Docker indiqué ci-dessus.
df -h
```

**Attendu :** Docker et Compose répondent ; le stockage prévu pour les images,
rapports et sauvegardes dispose de la capacité convenue avec l’exploitant.

Si la registry exige une authentification, utiliser le compte de lecture fourni
par le SI. Le mot de passe ou jeton est saisi à l’invite de Docker :

```bash
# Authentifie l’opérateur Docker Ubuntu ; remplacer par le nom réel de la registry.
sudo docker login REPLACE_REGISTRY_HOST
```

**Attendu :** `Login Succeeded`, ou accès déjà préparé par l’exploitant.
Un problème de proxy ou de certificat se règle avec lui avant le téléchargement.
Le [manuel d’installation](reference/01-installation.md) décrit les variantes
qui construisent les images et leurs prérequis supplémentaires.

## 5. Préparer l'instance privée

**Où : Ubuntu, racine des sources, `sudo`.** Sur une destination neuve seulement.

```bash
# Crée les seuls dossiers parents dans le profil Unix, sans créer l'instance privée.
mkdir -p "$HOME/.local/share"
# Crée configuration initiale et secrets internes du PoC ; ne démarre aucun service.
sudo python3 deploy/ubuntu/manage.py prepare "$RIE_INSTANCE" --mode "$RIE_MODE"
# Active le portail NiceGUI pour les mappings, ACL et rapports ; son démarrage viendra à l'étape 7.
sudo python3 deploy/ubuntu/manage.py configure-ui "$RIE_INSTANCE" --enable-demo-ui
# Contrôle les permissions de l'instance, des secrets et de l'environnement.
sudo stat -c '%a %n' "$RIE_INSTANCE" "$RIE_INSTANCE/secrets" "$RIE_INSTANCE/.env"
```

**Attendu :** instance préparée en `external`, portail activé, permissions
`700`, `700`, `600`. Les services démarreront à l’étape 7. `prepare` est réservé
à une destination neuve ; les clés générées doivent être conservées.

<a id="6-renseigner-les-sources-droits-et-albert"></a>
<a id="préparer-avec-nicegui"></a>
<a id="copier-les-modèles-et-les-fichiers-remis"></a>
<a id="définir-les-paramètres-de-connexion"></a>
<a id="définir-les-index-pdl-et-le-ldap"></a>
<a id="comprendre-les-noms-ldap"></a>
<a id="choisir-explicitement-les-droits-initiaux"></a>

## 6. Préparer le raccordement avec NiceGUI

L’assistant NiceGUI prépare les fichiers de l’instance créée à l’étape 5.
Remplir ses écrans avec les valeurs de la fiche de raccordement.

### Préparer les fichiers à déposer

L’assistant s’ouvre dans le **navigateur du poste** : les fichiers à déposer
doivent donc être disponibles sur ce poste. Utiliser ceux remis par le SI.
S’ils sont seulement sur Ubuntu, les récupérer **avant d’ouvrir le tunnel**
dans un terminal B du poste, distinct du terminal SSH A.

Les chemins ci-dessous supposent le dossier `si-connection` de l’étape 1 ;
les adapter si le SI l’a placé ailleurs. Le compte SSH doit pouvoir lire ces
fichiers de configuration. Les secrets restent sur Ubuntu.

```bash
# Crée sur le poste un dossier privé pour les seuls fichiers à déposer dans l’assistant.
install -d -m 700 "$HOME/rie-connection-files"
# Récupère le certificat public de confiance LDAP.
scp -p SSH_LOGIN@UBUNTU_HOST:si-connection/ldap-ca.crt "$HOME/rie-connection-files/"
# Récupère le contrat des équipements et périodes Incidents.
scp -p SSH_LOGIN@UBUNTU_HOST:si-connection/incidents-corpus.json "$HOME/rie-connection-files/"
# Récupère la correspondance des mesures, labels et unités.
scp -p SSH_LOGIN@UBUNTU_HOST:si-connection/metric-mapping.json "$HOME/rie-connection-files/"
# Récupère le snapshot autorisé des alertes et traitements courants.
scp -p SSH_LOGIN@UBUNTU_HOST:si-connection/itsm-tickets.json "$HOME/rie-connection-files/"
```

**Seulement si le SI fournit un bundle de CA complémentaire :**

```bash
# Récupère le bundle de certificats publics, sans clé privée.
scp -p SSH_LOGIN@UBUNTU_HOST:si-connection/ca-sources.pem "$HOME/rie-connection-files/"
```

**Attendu :** fichiers présents dans `rie-connection-files` sur le poste,
prêts à être sélectionnés dans le navigateur. Aucun fichier de `private/` ou
de `secrets/` n’est à transférer pour cette saisie.

### Ouvrir l’assistant

**Ubuntu · terminal A**, à la racine des sources, avec les variables des étapes
précédentes. Remplacer le marqueur par l’image NiceGUI approuvée remise par le SI.

```bash
# Désigne l'image NiceGUI approuvée pour cette livraison, avec son digest complet.
export RIE_SETUP_IMAGE=REPLACE_APPROVED_PORTAL_IMAGE
# Récupère cette image depuis la registry autorisée.
sudo docker pull "$RIE_SETUP_IMAGE"
# Ouvre l'assistant de préparation sans dépendre de LDAP ; garde ce terminal ouvert.
sudo python3 deploy/ubuntu/manage.py deployment-ui --image "$RIE_SETUP_IMAGE"
```

**Attendu :** l’assistant est disponible sur le port **18091**.

**Poste du déployeur · terminal B** : après la récupération des fichiers, ouvrir
le tunnel ci-dessous en remplaçant
`SSH_LOGIN@UBUNTU_HOST` par l’accès fourni par le SI. Garder aussi ce terminal ouvert.

```bash
# Rend le port de préparation Ubuntu accessible uniquement sur la boucle locale de ce poste.
ssh -o ExitOnForwardFailure=yes -N -L 127.0.0.1:18091:127.0.0.1:18091 SSH_LOGIN@UBUNTU_HOST
```

**Navigateur du poste :** ouvrir <http://127.0.0.1:18091/deployment>.

### Compléter la fiche dans NiceGUI

Suivre les **cinq écrans** de l’assistant avec les valeurs validées par le SI :

| Écran | Ce qu’il faut renseigner |
| --- | --- |
| **1. Cible et images** | Environnement, contact, vrai compte LDAP administrateur et références des deux images serveur |
| **2. Annuaire LDAP** | Serveur, lecteur technique, base, attributs et groupes réels |
| **3. Sources et droits initiaux** | OpenSearch, VictoriaMetrics et index ; sélectionner pour chaque source les droits validés dans la fiche |
| **4. Albert et Open WebUI** | URL et modèles Albert ; laisser les champs OAuth vides si leur préparation est reportée à l’étape 10 |
| **5. Fichiers, vérification et export** | Déposer les fichiers ci-dessous, confirmer la revue, vérifier puis télécharger |

Les droits sont initialement fermés : laisser une source sur **Interdit à tous**
en empêchera la lecture, même par un administrateur du PoC. Choisir les règles
convenues avec le responsable des accès avant de vérifier le dossier.

Fichiers à déposer : **`ldap-ca.crt`**, **`incidents-corpus.json`**,
**`metric-mapping.json`**, **`itsm-tickets.json`**, et **`ca-sources.pem`** si le SI
utilise une CA qui n’est pas déjà reconnue. Les trois JSON sont des contrats
préparés avec les responsables des données, au format du PoC ; voir le
[détail des fichiers](reference/02-configuration.md#répartition-des-fichiers-privés).
Le responsable des alertes précise l’origine, la date d’export et le périmètre
du snapshot dans son champ `provenance`. Un export natif non adapté ne suffit
pas ; le [format du snapshot](reference/02-configuration.md#préparer-le-snapshot-des-alertes-courantes)
précise les éléments à faire préparer. Les secrets seront installés après l’import.

Cliquer **Vérifier le dossier**. Corriger les messages jusqu’à obtenir
**« Préparation locale vérifiée »**, puis cliquer **Télécharger le dossier privé**.

**Attendu :** téléchargement de **`rie-connection.zip`**, avec les paramètres
et fichiers préparés. La vérification porte sur leur cohérence ; les accès
aux sources seront testés après le démarrage.

### Transférer le dossier vers Ubuntu

Une fois le ZIP enregistré, arrêter l’assistant avec **Ctrl+C dans le terminal A**,
puis arrêter le tunnel avec **Ctrl+C dans le terminal B**. Garder les deux
terminaux ouverts : les variables du terminal A restent disponibles.

**Poste du déployeur · terminal B** : transférer le fichier téléchargé.
Adapter le dossier et le nom au **ZIP qui vient d’être téléchargé** ; le
navigateur peut ajouter un suffixe si un fichier du même nom existe déjà.
Le dossier Ubuntu `deploy` existe déjà depuis l’étape 2.

```bash
# Rend le ZIP de raccordement téléchargé lisible uniquement par son propriétaire.
chmod 600 "$HOME/Downloads/rie-connection.zip"
# Transfère le ZIP en conservant ses permissions privées ; remplacer l'accès SSH par celui du SI.
scp -p "$HOME/Downloads/rie-connection.zip" SSH_LOGIN@UBUNTU_HOST:deploy/rie-connection.zip
```

**Attendu :** le ZIP est présent dans `$HOME/deploy` sur Ubuntu et reste privé.

### Importer et ajouter les secrets

**Ubuntu · terminal A**, toujours à la racine des sources. Les quatre fichiers
privés ci-dessous sont ceux fournis séparément dans `RIE_SI`. Si OpenSearch ou
VictoriaMetrics n’exige pas de secret, le responsable fournit le fichier correspondant vide.

```bash
# Importe le dossier vérifié dans l'instance externe neuve de l'étape 5 ; ne démarre aucun service.
sudo python3 deploy/ubuntu/manage.py configure-connection "$RIE_INSTANCE" --input "$HOME/deploy/rie-connection.zip"
# Installe le secret du seul lecteur LDAP, sans l'afficher.
sudo install -m 600 "$RIE_SI/private/ldap-reader.password" "$RIE_INSTANCE/secrets/reader.password"
# Installe le secret lecteur OpenSearch fourni par le SI, éventuellement vide selon son authentification.
sudo install -m 600 "$RIE_SI/private/opensearch.password" "$RIE_INSTANCE/secrets/opensearch_password"
# Installe le token lecteur VictoriaMetrics fourni par le SI, éventuellement vide selon son authentification.
sudo install -m 600 "$RIE_SI/private/victoria.token" "$RIE_INSTANCE/secrets/victoria_token"
# Installe la clé Albert autorisée pour le serveur, sans l'afficher.
sudo install -m 600 "$RIE_SI/private/albert.key" "$RIE_INSTANCE/secrets/albert_key"
```

**Attendu :** l’import affiche `prepared: true`. Les copies des secrets
réussissent sans sortie. Aucun service n’est encore démarré.

**Passer à l’étape 7.** Les fichiers de connexion et les droits initiaux sont
prêts. Les mappings PDL et l’historique ITSM seront configurés aux étapes 8 et 9.

## 7. Valider, récupérer les images et démarrer

**Où : Ubuntu · terminal A**, à la racine des sources. L’assistant a enregistré
les deux références d’images approuvées :

| Image | Services lancés |
| --- | --- |
| Image commune des moteurs | `pdl` et `incidents`, avec leurs ACL |
| Image NiceGUI | `portal` |

```bash
# Vérifie les paramètres et fichiers privés avant le démarrage.
sudo python3 deploy/ubuntu/manage.py validate "$RIE_INSTANCE"
# Télécharge les images du projet ; réutilise celles déjà présentes.
sudo python3 deploy/ubuntu/manage.py pull "$RIE_INSTANCE"
# Démarre les deux moteurs et le portail.
sudo python3 deploy/ubuntu/manage.py up "$RIE_INSTANCE"
# Affiche leur état et les images utilisées.
sudo python3 deploy/ubuntu/manage.py status "$RIE_INSTANCE"
```

**Attendu :** validation réussie, images disponibles, puis `pdl`, `incidents`
et `portal` **healthy**. Vérifier les images dans la colonne `IMAGE`.
La tâche ponctuelle `initialize` peut apparaître terminée avec un code `0`.
Le catalogue sera approuvé à l’étape 8.

En cas d’échec, corriger le message indiqué avant de continuer. Pour examiner
un service, utiliser les [commandes d’état et de journaux](#lire-létat-les-logs-et-lespace).

## 8. Revoir et approuver les mappings PDL

La page **Mappings PDL** propose le bloc **« Exemple complet illustré ·
PFS, Backbone, PFAING et TACACS »**, fermé par défaut. Il montre les champs,
relations, validations et approbations.
Son [guide avec captures](reference/04-pdl.md#exemple-complet-illustré--des-champs-pdl-aux-accès-tacacs)
précise aussi la limite actuelle des flux TACACS uniquement textuels.

**Où : navigateur du poste et terminal Ubuntu.** Faire relire le sens des
champs et des rapprochements par le responsable métier, pas seulement leur type.
Pour accéder au portail, ouvrir **un second terminal sur le poste**, avec
les véritables paramètres SSH :

```bash
# Ouvre l'accès local au portail du serveur autorisé ; remplacer SSH_LOGIN et UBUNTU_HOST avant exécution, puis garder ce terminal ouvert.
ssh -o ExitOnForwardFailure=yes -N -L 127.0.0.1:28090:127.0.0.1:18090 SSH_LOGIN@UBUNTU_HOST
```

**Attendu :** session silencieuse active. Si le port local est occupé, convenir
d'un autre port et adapter les liens. Ne pas publier l'écoute sur `0.0.0.0`.

1. Ouvrir [Mappings PDL dans NiceGUI](http://127.0.0.1:28090/admin-mappings)
   et se connecter avec le compte LDAP administrateur réel.
2. Cliquer **Actualiser les sources**. Pour une première saisie sans fichier,
   choisir **Ajouter un index découvert**. Si une proposition ou un export
   précédent vous a été remis sur le poste, utiliser **Importer un fichier
   YAML/JSON**. Ce YAML contient les mappings à examiner ; il est distinct du
   ZIP de raccordement de l’étape 6. Reprendre la proposition déjà chargée si elle existe.
3. Choisir le profil/famille et les champs physiques des concepts. Leur type
   est contrôlé ; le sens métier et la casse des valeurs restent à confirmer
   sur les échantillons réels. Ceux-ci ne sont pas un dictionnaire exhaustif.
4. Dans **Identité et lecture avancée**, ne relier que des identités de même
   sens. Une IP de trafic n'est pas automatiquement une IP de gestion.
5. **Vérifier la proposition**, saisir un motif puis **Enregistrer une
   proposition**. Les revues sont `pending` ; aucun mapping n'est encore activé.
   **Exporter le YAML** conserve une copie de la proposition, sans l'approuver.
6. Pour chaque revue : **Examiner cette revue**, comparer la version active,
   faire confirmer le sens, cocher la confirmation, justifier et **Approuver
   ce mapping**. Toute modification de schéma ou de version impose un nouvel examen.

**Attendu :** les index requis du SI sont approuvés. Quatre familles ne signifient
pas forcément quatre index physiques ; comparer à l'inventaire de la fiche.
Le responsable doit approuver chaque source requise et ses relations.

Dans le terminal Ubuntu initial :

```bash
# Vérifie catalogue, dépendances et LDAP après les approbations, avec le vrai compte administrateur.
sudo python3 deploy/ubuntu/manage.py check "$RIE_INSTANCE" --username "$RIE_ADMIN_LDAP"
```

**Attendu :** `ready`, mappings nécessaires actifs et sources requises prêtes.
Ce contrôle vérifie les accès et la configuration ; les réponses Albert sont
vérifiées à l’étape 11. Si le contrôle échoue, corriger la source ou le mapping
signalé. Le [guide des mappings](reference/04-pdl.md#revoir-puis-approuver-les-mappings)
détaille le diagnostic et l’utilisation de la ligne de commande.

## 9. Raccorder les précédents ITSM-NG

**Où : Ubuntu, après le premier démarrage.** Le connecteur lit les précédents
et solutions dans l’API ITSM-NG **1.6.17**, avec les droits de chaque utilisateur.
Le fichier `itsm-tickets.json` de l’étape 6 décrit les alertes et traitements
courants ; il reste distinct de cet historique. Adapter les adresses et le
fuseau du modèle à la fiche. Pour les données fictives de la sandbox,
renseigner aussi `"synthetic": true` dans ce JSON pour signaler leur origine
fictive ; les données réelles utilisent le réglage `false` par défaut.

```bash
# Installe le modèle du lecteur historique dans un fichier privé à personnaliser.
sudo install -m 600 deploy/ubuntu/examples/itsm-history.json "$RIE_INSTANCE/config/itsm-history-candidate.json"
# Remplace les adresses API et portail ITSM et vérifie le fuseau selon la fiche ; conserve les autres champs préremplis.
sudoedit "$RIE_INSTANCE/config/itsm-history-candidate.json"
# Associe le compte LDAP de recette à ses propres identifiants ITSM fournis séparément.
sudo python3 deploy/ubuntu/manage.py itsm-user "$RIE_INSTANCE" --username "$RIE_ADMIN_LDAP" --input "$RIE_SI/private/$RIE_ADMIN_LDAP-itsm.json"
```

Répéter `itsm-user` pour **chaque utilisateur autorisé**, avec son compte LDAP
et son propre fichier d’accès ITSM. Le responsable ITSM fournit les identifiants
et droits de chaque titulaire.

**Si un App-Token est requis**, [installer le fichier fourni par ITSM](reference/15-itsm-precedents.md#ajouter-lapp-token-fourni-par-itsm)
avant de continuer. Cette opération complète les accès individuels déjà enregistrés.

### Valider et appliquer le raccordement ITSM

```bash
# Enregistre le raccordement relu, sans contacter la source ni activer une écriture ITSM.
sudo python3 deploy/ubuntu/manage.py configure-itsm-history "$RIE_INSTANCE" --input "$RIE_INSTANCE/config/itsm-history-candidate.json"
# Vérifie la configuration et les identifiants avant application.
sudo python3 deploy/ubuntu/manage.py validate "$RIE_INSTANCE"
# Recharge les seuls services PoC ; interruption courte prévue.
sudo python3 deploy/ubuntu/manage.py up "$RIE_INSTANCE"
# Vérifie la lecture API avec l'identité associée au compte LDAP de recette.
sudo python3 deploy/ubuntu/manage.py check "$RIE_INSTANCE" --username "$RIE_ADMIN_LDAP"
```

**Attendu :** `ready` et `incidents_itsm-ng-history: ok`. À l’étape 11,
retrouver un précédent connu et comparer deux utilisateurs ITSM : une API
joignable ne suffit pas à vérifier le contenu et les droits.

Le [guide ITSM](reference/15-itsm-precedents.md) précise les paramètres et les
limites du connecteur. Convenir avec le responsable de la source de la mise à
jour du snapshot courant `itsm-tickets.json`.

## 10. Connecter NiceGUI et Open WebUI local

La préparation du serveur et celle du poste peuvent être confiées à deux
personnes. Terminer les deux avant de connecter les utilisateurs.

### Sur Ubuntu : activer la connexion individuelle

```bash
# Conserve la proposition de l'assistant si elle existe ; sinon installe le modèle public OAuth à compléter.
sudo test -f "$RIE_INSTANCE/config/oauth-candidate.json" || sudo install -m 600 deploy/ubuntu/examples/oauth.json "$RIE_INSTANCE/config/oauth-candidate.json"
# Renseigne l’adresse publique PDL, les deux URLs MCP et les URLs de retour d’Open WebUI, selon la fiche.
sudoedit "$RIE_INSTANCE/config/oauth-candidate.json"
# Valide et enregistre ce raccordement, sans authentifier un utilisateur.
sudo python3 deploy/ubuntu/manage.py configure-client-auth "$RIE_INSTANCE" --input "$RIE_INSTANCE/config/oauth-candidate.json"
# Vérifie toute la configuration avant interruption.
sudo python3 deploy/ubuntu/manage.py validate "$RIE_INSTANCE"
# Recharge les seuls services PoC avec OAuth activé.
sudo python3 deploy/ubuntu/manage.py up "$RIE_INSTANCE"
# Vérifie sources, catalogue, LDAP et historique ITSM avec l'administrateur de recette.
sudo python3 deploy/ubuntu/manage.py check "$RIE_INSTANCE" --username "$RIE_ADMIN_LDAP"
```

**Attendu :** configuration acceptée, puis `ready`. Vérifier dans le fichier
les URLs HTTPS des moteurs et les URLs de retour d’Open WebUI fournies dans la
fiche. Les paramètres préremplis de durée peuvent être conservés.
Le responsable réseau vérifie leur accès depuis le navigateur **et** depuis
Open WebUI. Le [guide de connexion](reference/25-user-access.md#ce-que-prépare-le-déployeur)
détaille les champs, certificats et paramètres de proxy.

### Sur le poste : préparer le client local

**Prérequis :** Open WebUI local déjà installé, Docker/Compose, Python 3.10+,
sources publiques de la même version, compte administrateur local et fournisseur
Albert configuré. Les commandes partent de la racine des sources **sur le poste**,
pas du terminal SSH Ubuntu.

**Si le poste utilise déjà l’image cliente approuvée avec l’adaptateur OAuth**,
conserver cette installation et passer au contrôle de la connexion Albert
ci-dessous. Sinon, l’administrateur sauvegarde les données et fichiers de
lancement du client, puis conserve ses volumes, clés, réseau et réglages en
changeant l’image.
Pour un client Docker Compose, reprendre les mêmes fichiers, variables et nom
de projet. Les commandes ci-dessous supposent un service nommé `openwebui` ;
un autre lancement se prépare avec le [guide du poste](reference/25-user-access.md#ce-que-prépare-ladministrateur-du-poste).

```bash
# Désigne l'image cliente approuvée reçue du responsable de la registry ; remplacer la référence avant exécution.
export RIE_OPENWEBUI_IMAGE=REPLACE_APPROVED_OPENWEBUI_IMAGE
# Récupère cette image avec l'opérateur Docker local autorisé.
docker pull "$RIE_OPENWEBUI_IMAGE"
# Recrée uniquement le client local ; remplacer CLIENT_COMPOSE et conserver les paramètres du projet existant.
docker compose -f CLIENT_COMPOSE -f deploy/openwebui/compose.override.yaml up -d --no-deps openwebui
```

**Attendu :** Open WebUI accessible sur son port habituel, avec les comptes
et conversations conservés. L’opérateur Docker du poste doit avoir accès à la
registry ; l’accès Ubuntu de l’étape 4 ne remplace pas celui du poste.

Dans Open WebUI, vérifier la connexion Albert et relever l’identifiant complet
du modèle disponible qui servira de relais. Dans les paramètres de partage de
ce modèle, accorder sa lecture aux utilisateurs prévus pour le PoC, selon les
droits du poste. Vérifier cet accès avec un compte utilisateur ordinaire : les
deux modèles PoC ont besoin de ce relais pour fonctionner. Les ACL des sources
restent vérifiées séparément par les moteurs.
Renseigner ensuite ses URLs et cet identifiant dans le terminal du poste :

```bash
# Désigne l’adresse de base réelle du client, avec le même port que ses URLs de retour.
export RIE_WEBUI_URL=http://127.0.0.1:8080
# Désigne l'URL MCP publique PDL de la fiche, joignable par le navigateur et le client.
export RIE_PDL_URL=https://pdl.REPLACE.invalid/mcp
# Désigne l'URL MCP publique Incidents de la fiche.
export RIE_INCIDENTS_URL=https://incidents.REPLACE.invalid/gptoss/assistant/mcp
# Désigne l'identifiant complet du modèle Albert réellement disponible dans ce client.
export RIE_RELAY_MODEL=REPLACE_FULL_CLIENT_MODEL_ID
# Contrôle ces adresses sans modifier le client ni demander un secret.
python3 deploy/openwebui/configure.py --webui-url "$RIE_WEBUI_URL" --pdl-url "$RIE_PDL_URL" --incidents-url "$RIE_INCIDENTS_URL" --base-model "$RIE_RELAY_MODEL" --dry-run
# Installe les deux connexions, filtres et modèles ; demande le compte administrateur de ce client local.
python3 deploy/openwebui/configure.py --webui-url "$RIE_WEBUI_URL" --pdl-url "$RIE_PDL_URL" --incidents-url "$RIE_INCIDENTS_URL" --base-model "$RIE_RELAY_MODEL"
```

**Attendu :** `configured: true` et `user_authorization_pending: true`.
Conserver le chemin de sauvegarde affiché pour un éventuel retour arrière.
L’utilisateur autorise ensuite ses propres accès.

### Autoriser les deux sources dans le navigateur

1. Se connecter normalement à son Open WebUI avec un **compte utilisateur
   ordinaire**, pas seulement le compte administrateur du client.
2. Vérifier que **SI-RIE — PDL** et **SI-RIE — INCIDENTS** apparaissent dans
   le sélecteur de modèles.
3. Ouvrir [Autoriser PDL](http://127.0.0.1:8080/oauth/clients/mcp:dinum-pdl-agent/authorize),
   vérifier l'adresse du PoC puis saisir **son propre compte LDAP réel** et
   confirmer. Adapter l'origine du lien si le client utilise un autre port.
4. Ouvrir [Autoriser Incidents](http://127.0.0.1:8080/oauth/clients/mcp:obs-gptoss/authorize),
   vérifier l'identité LDAP affichée et confirmer. **Utiliser un autre compte**
   permet de corriger l'identité ; chaque source a sa propre autorisation.
5. Revenir dans Open WebUI pour tester les deux modèles à l’étape 11.

Chaque utilisateur conserve sa propre autorisation ; Open WebUI la renouvelle
pendant sa durée de validité. Une expiration ou une révocation impose une
nouvelle connexion. Les droits restent ceux du LDAP et des sources.

## 11. Jouer les deux UC et leurs ACL avec Albert

**Où : Open WebUI sur le poste utilisateur.** Vérifier l’autorisation personnelle
aux deux sources selon l’étape 10. Un nouveau fil signifie **une nouvelle
conversation**, pas une actualisation de la page. Attendre la réponse avant
d’envoyer le message suivant. Un état `ready` ne prouve aucune génération.

| Environnement de recette | Textes et attendus à utiliser |
| --- | --- |
| **Sandbox SI simulé préparée**, y compris en mode `external` | Les blocs copiables ci-dessous utilisent ses données fictives : `demo-alpha`, `edge-a`, `OBS-RTR-01` et les deux fenêtres datées. |
| **SI-RIE réel** | Faire compléter les [demandes de recette du SI](reference/26-si-connection-sheet.md#textes-open-webui-à-préparer-pour-le-si-réel). Remplacer équipements, périmètres, dates et attendus par les valeurs validées du SI, avec le modèle et le fil indiqués. |

Consigner le texte **effectivement envoyé**, l’identité, le modèle, la fenêtre
et son fuseau, le résultat, le `request_id` et les appels Albert exécutés.
Comparer les réponses réellement reçues aux attendus ci-dessous.

### PDL : clarification puis reprise du même fil

**Cas de référence de la sandbox :** compter les refus PFS de `demo-alpha`,
le **12 octobre 2026 de 08:00 à 09:00 UTC** ; attendu : **deux refus**.
Voici exactement comment tester l’omission de la période.

**PDL-1 — Nouveau fil**, modèle **SI-RIE — PDL**. Envoyer seulement :

```text
Compte les refus PFS dans demo-alpha.
```

**Attendu :** l’assistant demande la période. Il ne doit pas annoncer un nombre
comme si la recherche avait déjà été exécutée.

**PDL-2 — Même fil et même modèle**, après cette clarification. Envoyer :

```text
Le 12 octobre 2026 de 08:00 à 09:00 UTC.
```

**Attendu : deux refus PFS** dans `demo-alpha`, sur la période demandée.
Vérifier les index, le filtre de refus et les preuves de lecture.

**Comparaison NiceGUI :** dans [PDL](http://127.0.0.1:28090/demo-pdl-5min),
se connecter avec le même compte LDAP, remplacer **Question métier** par cette
**demande complète**, laisser la reprise décochée et cliquer **Investiguer la question** :

```text
Compte les refus PFS dans demo-alpha le 12 octobre 2026 de 08:00 à 09:00 UTC.
```

Comparer au même attendu ; dans le SI réel, utiliser sa demande préparée.

### PDL : quatre familles puis restriction des sources

**PDL-3 — Nouveau fil**, modèle **SI-RIE — PDL** :

```text
Montre les logs de l'équipement edge-a PFS BB PFAING TACACS dans demo-alpha du 12 octobre 2026 de 08:00 à 09:00 UTC.
```

**Attendu dans la sandbox : six documents**, deux PFS, un BB, un PFAING et
deux TACACS. Vérifier les rapprochements approuvés et l’exclusion des autres
équipements, périmètres et jours.

**PDL-4 — Même fil que PDL-3** :

```text
Uniquement TACACS pour ce même équipement, sur la même période.
```

**Attendu : deux documents TACACS**, liés aux IP de gestion IPv4 et IPv6.
Les sources d’identité PFS/BB peuvent être lues ; seuls les logs TACACS sont
attendus dans la réponse.

**PDL-5 — Toujours ce même fil** :

```text
Pour edge-a dans demo-alpha, sur la même période, montre PFS, BB et PFAING, sans TACACS.
```

**Attendu : quatre documents**, deux PFS, un BB et un PFAING, issus de nouvelles
lectures autorisées. La période et le périmètre restent ceux de PDL-3.
Les cas de [recherche par IP et de résultat vide](reference/16-pdl-tutorial.md#5-chercher-les-accès-internet-dune-ip)
complètent ce parcours si ces usages font partie de la réception.

Dans NiceGUI, **Lire mon catalogue approuvé** affiche les index autorisés
et approuvés, leurs versions et correspondances métier. Cette lecture permet
de vérifier la configuration avant les recherches.

<a id="incidents--diagnostic-et-précédents-natifs"></a>

### Incidents : diagnostic, suivi et précédents

**Cas de référence de la sandbox :** `OBS-RTR-01` (alias `R1`), site de Rennes,
le **28 septembre 2026 de 10:00 à 12:00, heure de Paris**.
Vérifier cette période avant l’envoi.

**INC-1 — Nouveau fil**, modèle **SI-RIE — INCIDENTS** :

```text
Diagnostique OBS-RTR-01 le 28 septembre 2026 de 10:00 à 12:00, heure de Paris, sur le site de Rennes. Recherche les précédents ITSM-NG disponibles et présente la solution la mieux étayée, ses conditions et les niveaux de confiance du diagnostic et de la solution. Cite les sources et distingue résolution technique et clôture du ticket.
```

**Attendu dans la sandbox :** cinq mesures de 120 points chacune et 120 logs ;
la résolution de l’alerte est à **11:30**, la clôture du ticket à **12:15**.
Comparer les mesures, couverture, logs et chronologie aux sources. L’historique
ITSM exige son raccordement et les droits natifs individuels de l’étape 9 :
son absence ou une recherche sans précédent doit être indiquée, pas remplacée
par des tickets inventés. Une solution reste conditionnelle, sourcée et à
valider par un opérateur ; sa confiance est distincte de celle du diagnostic.

Contrôler **rédaction et relecture Albert**, modèle retourné et tokens.
`partial`, synthèse non validée, appel absent ou fournisseur indisponible ne
valent pas réussite de la génération. Le budget IA est partagé par les
utilisateurs de l’instance. Attendre au moins **60 secondes après la réponse**
avant un nouveau diagnostic ou un échange de clarification Incidents. Un
suivi fermé sur un statut et une période déjà confirmée peut relire les faits
sans nouvel appel IA.

Si **« Budget IA : réessayez dans… »** apparaît, attendre le délai indiqué et
**renvoyer le même message** avant de poursuivre le dialogue. Ce message
signifie que la demande n’a pas encore été traitée ; passer directement à la
précision suivante ferait perdre le contexte de la demande refusée.

**INC-2 — Même fil que INC-1**, même modèle :

```text
À 12 h, heure de Paris, l'alerte était-elle résolue et le ticket déjà clôturé ? Cite les horodatages disponibles.
```

**Attendu :** à midi, l’alerte est résolue et le ticket pas encore clôturé.
Un statut final `closed` ne doit pas être appliqué rétroactivement à midi.
Si le rapport complet est replié sous **Détails du diagnostic**, le déplier
pour consulter les mesures, les précédents et toutes les sources.

**INC-3 — Nouveau fil**, modèle **SI-RIE — INCIDENTS**, pour vérifier aussi
sa clarification de période :

```text
Diagnostique OBS-RTR-01 sur le site de Rennes.
```

**Attendu :** demande de période, sans diagnostic sur des dates inventées.
**Dans ce même fil, attendre 60 secondes après la demande de période**, puis
répondre avec le texte ci-dessous. Sa compréhension utilise aussi l’IA et
partage le même budget.

```text
Le 28 septembre 2026 de 10:00 à 12:00, heure de Paris.
```

**Attendu :** le diagnostic conserve `OBS-RTR-01` et Rennes et relit les sources
sur cette fenêtre. Les [autres cas Incidents](reference/17-incidents-tutorial.md#3-comparer-les-équipements-du-site-de-rennes)
fournissent les demandes exactes pour comparer Rennes, constater le trou de
collecte de R3 et vérifier le refus de R4.

Dans la sandbox, **Diagnostiquer avec Albert API** sur la [page Incidents](http://127.0.0.1:28090/demo-incidents-5min)
permet de comparer ce même scénario. Le test des données réelles se fait
avec leur question dans Open WebUI.

### ACL : lecteurs, refus et rétablissement

Utiliser des sessions utilisateur distinctes et l’autorisation OAuth propre à
chaque titulaire. Une nouvelle conversation ou une autre fenêtre portant la
même autorisation ne change pas l’identité. Ne pas partager celle de l’administrateur.

| Profil réel à fournir | Vérification |
| --- | --- |
| Interne administrateur | Lectures selon politique et administration autorisée |
| Interne lecteur | Lectures autorisées, administration ACL/mappings/qualification refusée |
| Externe | PDL seulement si `authenticated` ; sources Incidents selon les règles réellement décidées |
| Non classé | Connexion et lectures protégées refusées |

**ACL-1 — Nouveau fil par utilisateur**, modèle **SI-RIE — PDL** :

```text
Quels sont mes index accessibles ?
```

**Attendu :** uniquement les index de ce titulaire, sans période nécessaire.
Tester aussi cette liste dans un fil PDL déjà utilisé : elle ne doit pas
reprendre la période ou l’équipement de son ancienne recherche.

**ACL-2 — Nouveau fil PDL par lecteur**, pour comparer l’accès TACACS :

```text
Montre les logs de l'équipement edge-a uniquement TACACS dans demo-alpha du 12 octobre 2026 de 08:00 à 09:00 UTC.
```

Dans la sandbox avec TACACS `authenticated`, Bruno et Eric reçoivent chacun
deux documents. Le [cycle TACACS complet](reference/16-pdl-tutorial.md#9-vérifier-les-acl-pdl)
précise la question de relance et les attendus après restriction et rétablissement.
Sur le SI, utiliser la source de recette et la politique convenues dans la fiche.

**ACL-3 — Nouveau fil Incidents par lecteur**, modèle **SI-RIE — INCIDENTS**,
avec le texte complet suivant dans la sandbox, ou la demande Incidents préparée du SI :

```text
Diagnostique OBS-RTR-01 le 28 septembre 2026 de 10:00 à 12:00, heure de Paris, sur le site de Rennes.
```

Selon la politique initiale de la sandbox, Bruno est autorisé et Eric refusé,
sans preuve interdite. Les attendus du SI réel dépendent de sa politique.

Pour le retrait/rétablissement, dans [Administration ACL](http://127.0.0.1:28090/admin-acces),
noter la révision et la règle initiale de la **source de recette convenue**.
Passer **Mesures VictoriaMetrics** à `blocked`, enregistrer et vérifier la
hausse de révision. **Dans chacun des fils Incidents déjà ouverts**, renvoyer
**le texte complet ACL-3 ci-dessus**, sans changer de compte ni d’autorisation :
refus même pour l’administrateur, sans preuve interdite ni ancien diagnostic
republié. Rétablir exactement la règle initiale, enregistrer et **renvoyer
encore le texte complet ACL-3 dans ces mêmes fils**, après admission : nouvelles
preuves attendues pour les seuls comptes autorisés. Répéter ce cycle pour
**Tickets et suivi ITSM**, en notant sa propre règle initiale. Restaurer chaque
règle avant de passer à la suivante.

Comparer aussi deux utilisateurs ayant des droits natifs ITSM différents :
entités, tickets, solutions et suivis privés ne doivent pas être élargis par le
PoC. Un compte administrateur local Open WebUI n’est pas une preuve du parcours
utilisateur ordinaire. Tout cas non joué reste **non qualifié**, avec sa raison.

## 12. Vérifier l'exploitation et consigner

**Où : Ubuntu, racine des sources, `sudo`.** Planifier l'interruption des seuls
services PoC. Les bases et l'annuaire du SI restent gérés par leurs responsables.

### Lire l'état, les logs et l'espace

```bash
# Affiche l'état des processus sans les arrêter.
sudo python3 deploy/ubuntu/manage.py status "$RIE_INSTANCE"
# Vérifie la disponibilité métier avec le compte de recette habilité.
sudo python3 deploy/ubuntu/manage.py check "$RIE_INSTANCE" --username "$RIE_ADMIN_LDAP"
# Lit les dernières lignes PDL pour retrouver les request_id de la recette.
sudo python3 deploy/ubuntu/manage.py logs "$RIE_INSTANCE" --service pdl --tail 100
# Lit les dernières lignes Incidents pour expliquer les échecs éventuels.
sudo python3 deploy/ubuntu/manage.py logs "$RIE_INSTANCE" --service incidents --tail 100
# Mesure l'occupation et les rapports, sans suppression automatique.
sudo python3 deploy/ubuntu/manage.py storage "$RIE_INSTANCE"
```

**Attendu :** services sains, `ready`, aucun échec inexpliqué dans les journaux
et espace disque disponible.
Examiner les logs en privé et convenir de l’archivage des rapports avec l’exploitant.

### Arrêter, sauvegarder et reprendre

```bash
# Choisit une destination de sauvegarde neuve, privée, hors sources et instance ; adapter au stockage approuvé.
export RIE_BACKUP="$HOME/rie-backups/$(date +%Y%m%d-%H%M%S)"
# Crée le seul parent privé des sauvegardes, pas la destination finale.
sudo install -d -m 700 "$HOME/rie-backups"
# Arrête les seuls services du PoC en conservant volumes et fichiers privés.
sudo python3 deploy/ubuntu/manage.py stop "$RIE_INSTANCE"
# Sauvegarde configuration, images et données locales avec manifeste d'intégrité.
sudo python3 deploy/ubuntu/manage.py backup "$RIE_INSTANCE" --output "$RIE_BACKUP"
# Redémarre la même instance sans réinitialiser les données ni réapprouver les mappings.
sudo python3 deploy/ubuntu/manage.py up "$RIE_INSTANCE"
# Vérifie la disponibilité et la conservation des règles/mappings après reprise.
sudo python3 deploy/ubuntu/manage.py check "$RIE_INSTANCE" --username "$RIE_ADMIN_LDAP"
```

**Attendu :** reçu de sauvegarde et SHA-256 du manifeste, puis `ready`. Si la
sauvegarde échoue, l'instance reste arrêtée ; corriger ou reprendre explicitement
par `up` et `check`. La sauvegarde contient des secrets : rester sur le circuit
privé de l'exploitant. En `external`, elle **n'inclut pas** les bases/LDAP du SI,
à sauvegarder séparément. La [procédure d’exploitation](reference/07-operations.md) détaille sauvegarde et mise à jour.

### Restaurer réellement dans une instance neuve

```bash
# Choisit une destination inexistante et distincte de l'originale.
export RIE_RESTORED="$HOME/.local/share/dinum-rie-si-restored-$(date +%Y%m%d-%H%M%S)"
# Vérifie les empreintes et prépare la copie ; l’originale reste disponible si cette restauration échoue.
sudo python3 deploy/ubuntu/manage.py restore "$RIE_RESTORED" --backup "$RIE_BACKUP"
# Arrête l’originale après la restauration réussie, pour libérer ses ports.
sudo python3 deploy/ubuntu/manage.py stop "$RIE_INSTANCE"
# Démarre la copie avec les images et fichiers sauvegardés.
sudo python3 deploy/ubuntu/manage.py up "$RIE_RESTORED"
# Vérifie sources et mappings dans la copie restaurée.
sudo python3 deploy/ubuntu/manage.py check "$RIE_RESTORED" --username "$RIE_ADMIN_LDAP"
```

**Attendu :** intégrité acceptée, nouvelle instance et `ready`. Comparer la
révision ACL, les mappings, audits et états des autorisations.
Dans Open WebUI, ouvrir **un nouveau fil SI-RIE — PDL** et envoyer le texte
complet **PDL-3** de l’étape 11 ; attendu de la sandbox : six documents des
quatre familles. Ouvrir **un nouveau fil SI-RIE — INCIDENTS** et envoyer le
texte complet **INC-1** ; comparer les faits aux références initiales et vérifier
les nouveaux appels Albert. En production, utiliser les demandes préparées du SI.
Avec un compte privé d’un droit, renvoyer aussi la demande complète **ACL-3** :
refus sans preuve interdite. Respecter le délai de synthèse et conserver les
nouveaux `request_id` ; une ancienne réponse affichée ne prouve pas la restauration.
Les bases et l’annuaire du SI restent gérés par leurs responsables.

Le dernier service actif est **l'instance restaurée** ; conserver l'originale
arrêtée. Une révocation postérieure à la sauvegarde doit être réappliquée avant
remise aux utilisateurs. L'expiration réelle reste opposable. Si le client a
renouvelé ses autorisations depuis la sauvegarde, une nouvelle autorisation
peut être nécessaire ; reconnecter alors le compte habilité depuis Open WebUI.

```bash
# Sélectionne désormais l'instance restaurée active pour les futures commandes d'exploitation.
export RIE_INSTANCE="$RIE_RESTORED"
```

### Clore la séance

Consigner date, opérateur, OS/architecture, commit, archive/manifeste, images
et digests, chemins, révision ACL, mappings, requêtes, modèles réellement
retournés et références des rapports. Pour chacune des douze étapes :
**réussi**, **échoué** ou **non fait**, avec preuve ou raison. Le responsable métier
valide séparément la pertinence des résultats.

Consulter les [rapports de tests dans NiceGUI](http://127.0.0.1:28090/qualification).
L’[import d’un rapport Allure](reference/13-test-reports.md#importer-puis-consulter-dans-nicegui)
est décrit en complément si un rapport vous a été remis.

La [matrice de réception SI-RIE](reference/14-si-rie-acceptance.md) précise les
vérifications à terminer sur les réseaux, identités, données et capacités réels.
Les [preuves de qualification](reference/08-qualification.md) donnent leur
version, leur environnement et leurs limites.

## Reprendre après une déconnexion SSH

**À utiliser seulement pour retrouver une installation déjà préparée.** Sur
Ubuntu, reprendre les chemins et le compte LDAP notés dans le compte rendu.
Après l’étape 12, choisir l’instance restaurée comme instance active.

```bash
# Retrouve le dossier contenant rie-delivery ; remplacer par le chemin enregistré.
export RIE_SOURCES=REPLACE_SOURCE_DIRECTORY
# Retrouve le dossier privé de l’instance active ; remplacer par le chemin enregistré.
export RIE_INSTANCE=REPLACE_ACTIVE_INSTANCE_DIRECTORY
# Retrouve l’administrateur LDAP habilité du PoC.
export RIE_ADMIN_LDAP=REPLACE_LDAP_ADMIN_LOGIN
# Revient à la racine des sources de cette version.
cd "$RIE_SOURCES/rie-delivery"
# Vérifie l’état des services de l’instance retrouvée.
sudo python3 deploy/ubuntu/manage.py status "$RIE_INSTANCE"
# Vérifie les sources et mappings de cette instance.
sudo python3 deploy/ubuntu/manage.py check "$RIE_INSTANCE" --username "$RIE_ADMIN_LDAP"
```

Reprendre ensuite au dernier contrôle réussi. La préparation initiale, l’import
et les approbations déjà réalisés restent conservés. Pour reprendre une étape
de déploiement inachevée, rétablir aussi les variables qu’elle utilise, notamment
`RIE_SI` si elle copie des fichiers reçus du SI.
