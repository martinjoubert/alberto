# Déployer PDL, Incidents et les ACL dans le SI-RIE — pas à pas

**Point d'entrée du déployeur : suivre ce document de haut en bas.** Il reprend
les douze étapes exercées le 16 novembre 2026 sur Ubuntu avec un SI simulé
déjà disponible. Il décrit une **première installation en mode `external`** :
les sources et l'annuaire existent ; seuls les deux moteurs et le portail
NiceGUI du PoC sont installés sur Ubuntu. Open WebUI reste sur chaque poste.

Le [manuel général](01-installation.md) conserve les variantes scriptées et
`demo` ; ses commandes ne sont pas un second parcours à exécuter ici. Pour
refaire l'exercice de laboratoire, utiliser les paramètres de la
[fiche sandbox](22-external-sandbox.md), avec la même numérotation. Les comptes,
index, dates et accès Lima/Colima du laboratoire ne sont pas ceux du SI-RIE.

## Avant de commencer

Le coordinateur remet au déployeur :

- La version identifiée : archive source, SHA-256 et référence Git complète du
  reçu de livraison, ou dépôt privé et commit complet. Ne pas déployer une branche mobile.
- La **[fiche de raccordement complétée](26-si-connection-sheet.md)** : adresses,
  certificats, groupes, index, contrats des données, cas de recette et contacts.
  Les secrets sont transmis séparément dans des fichiers privés.
- Les **deux images serveur approuvées**, runtime et NiceGUI, dans la registry
  GitLab autorisée, avec digest et architecture. Leur construction/promotion
  est expliquée en fin de guide ; elle précède l'installation sur le SI.
- Un compte SSH Ubuntu avec `sudo`, Docker/Compose/Buildx autorisés, les accès
  réseau des conteneurs et les URLs HTTPS du PoC préparées par le SI.
- Pour chaque poste de recette : Open WebUI installé, un administrateur local,
  l'image cliente approuvée et un accès Albert. La clé du serveur et le réglage
  Albert du client local sont deux configurations distinctes.

Une information manquante se traite avec le responsable indiqué dans la fiche.
Ne pas remplacer un paramètre inconnu par celui du laboratoire. Les services
existants OpenSearch, VictoriaMetrics, LDAP et ITSM-NG ne sont ni créés ni
initialisés par ce parcours. Aucun `seed` n'est à lancer en `external`.

Les lignes `#` expliquent la commande immédiatement suivante. Exécuter une
commande à la fois, contrôler l'attendu et s'arrêter en cas d'erreur. Les mots
de passe et tokens ne doivent pas apparaître dans le compte rendu. Les valeurs
`REPLACE_...` se remplacent avant exécution ; les variables définies au cours
du guide ne survivent pas à la fermeture du terminal.

| Étape | Action | Machine | Attendu avant de continuer |
| --- | --- | --- | --- |
| 1 | Confirmer le serveur et le dossier de raccordement | Ubuntu | Bon serveur, Python et sudo, fiche et fichiers reçus |
| 2 | Vérifier et extraire les sources | Ubuntu | Archive `OK`, manifeste `passed: true` |
| 3 | Choisir l'instance et le compte administrateur | Ubuntu | Mode `external`, vrai compte LDAP habilité |
| 4 | Vérifier les prérequis | Ubuntu | Docker, Compose, Buildx et capacité disponibles |
| 5 | Préparer l'instance privée | Ubuntu | Permissions privées, aucun service démarré |
| 6 | Renseigner sources, droits, Albert et images | Ubuntu | Paramètres revus, secrets privés |
| 7 | Valider, récupérer les images et démarrer | Ubuntu | Trois services PoC sains ; catalogue encore à approuver |
| 8 | Revoir et approuver les mappings | Ubuntu et navigateur | Sources requises prêtes, catalogue approuvé |
| 9 | Raccorder les précédents ITSM-NG | Ubuntu | Identités individuelles et lecture API vérifiée |
| 10 | Préparer le client puis autoriser les sources | Ubuntu et poste utilisateur | Deux connexions OAuth personnelles disponibles |
| 11 | Jouer les deux UC et leurs ACL avec Albert | Navigateur | Résultats de référence et refus conformes |
| 12 | Vérifier l'exploitation et consigner | Ubuntu et navigateur | Sauvegarde, reprise et restauration réelles |

## Les services du PoC et leur utilisation

| Service | Rôle | Utilisation | Écoute Ubuntu par défaut |
| --- | --- | --- | --- |
| `pdl` | Recherche, comptage et rapprochement des logs, mappings et ACL | Modèle **SI-RIE — PDL**, outil MCP `investigate_pdl` | `127.0.0.1:19910`, `/mcp` |
| `incidents` | Lecture des mesures/logs/documents/tickets, diagnostic Albert et relecture | Modèle **SI-RIE — INCIDENTS**, outil `diagnose_observability` | `127.0.0.1:19950`, `/gptoss/assistant/mcp` |
| `portal` | NiceGUI : administration des mappings/ACL, documentation, démos et rapports Allure | Navigateur, compte LDAP ; administration réservée aux habilités | `127.0.0.1:18090` |
| `initialize` | Copie des fichiers privés dans le volume et permissions | Tâche ponctuelle lancée automatiquement par `up` | Aucune page web |
| `operator` | Commandes `check`, mappings, jetons et opérations | Tâche ponctuelle lancée par `manage.py` | Aucune page web |

Les ACL sont communes aux deux moteurs ; il n'y a pas de serveur ACL séparé.
Les ports restent sur la boucle locale. Un tunnel SSH suffit à l'administration
NiceGUI ; les deux URLs MCP HTTPS sont rendues joignables par le circuit réseau
du SI. Albert est appelé par API, sans modèle local installé sur Ubuntu.

## 1. Identifier Ubuntu et vérifier le SI existant

**Où : terminal Ubuntu**, ouvert avec l'accès SSH fourni par le SI. Ne pas
utiliser un lanceur Lima du laboratoire pour joindre un serveur réel.

```bash
# Affiche le compte Unix de la session ; comparer au compte de la fiche.
whoami
# Affiche le serveur connecté ; comparer à l'hôte prévu.
hostname
# Vérifie la distribution ; Ubuntu 24.04 ou 26.04 est attendu.
cat /etc/os-release
# Affiche l'architecture ; elle doit correspondre aux images remises.
uname -m
# Vérifie Python ; 3.10 ou plus est nécessaire aux commandes opérateur.
python3 --version
# Vérifie les droits système ; attendu : 0, identifiant Unix de root.
sudo id -u
# Désigne le dossier reçu avec la fiche et les fichiers de raccordement ; adapter son emplacement si nécessaire.
export RIE_SI="$HOME/si-connection"
# Confirme que ce dossier existe, sans afficher les fichiers privés.
test -d "$RIE_SI"
```

**Attendu :** bonne machine, bonne architecture, Python disponible et sudo
autorisé. Le SI réel n'a pas à présenter cinq conteneurs locaux : ses sources
peuvent être sur d'autres serveurs. Le programme `verify-si.py` n'existe que
dans notre sandbox. Les accès réels seront sondés par `check`, puis comparés
aux cas métier ; une simple réponse réseau ne valide pas les données.

## 2. Vérifier et extraire le paquet

**Où : même terminal Ubuntu.** Les sources et l'instance doivent rester
distinctes. Ne pas extraire par-dessus un dossier existant.

```bash
# Entre dans le dossier de réception du paquet ; adapter si le SI l'a remis ailleurs.
cd "$HOME/deploy"
# Désigne le nom exact de l'archive de cette livraison, à comparer au reçu.
export RIE_ARCHIVE=rie-si-rie-ubuntu-2026-11-17-si-deploy.tar.gz
# Vérifie l'archive contre son fichier SHA-256 reçu ; attendre OK.
sha256sum -c "$RIE_ARCHIVE.sha256"
# Choisit un dossier neuf pour cette version des sources.
export RIE_SOURCES="$HOME/rie-deploy-sources-2026-11-17"
# Crée le dossier ; s'il existe, choisir un autre nom avant extraction.
mkdir "$RIE_SOURCES"
# Extrait les sources ; aucun logiciel ni service n'est installé.
tar -xzf "$RIE_ARCHIVE" -C "$RIE_SOURCES"
# Se place à la racine des sources, utilisée pour les commandes suivantes.
cd "$RIE_SOURCES/rie-delivery"
# Vérifie la liste, les tailles et les empreintes des fichiers extraits.
python3 deploy/ubuntu/verify_delivery.py .
```

**Attendu :** archive `OK`, puis `passed: true`. Comparer `source.commit` et
l'empreinte du manifeste au reçu de livraison. Le paquet final est construit
depuis un commit propre (`source.dirty: false`). Les anciens paquets de recette
`dirty: true` gardent leur date ; ne pas les présenter comme cette version finale.
`authenticity: not_verified_by_checksum` reste normal : les empreintes contrôlent
l'intégrité, l'origine se confirme par le canal de distribution approuvé ou
une signature vérifiée. Le [manuel](01-installation.md#interpréter-le-résultat-de-vérification-du-paquet)
explique chaque champ.

Si le SI remet un **dépôt privé au lieu d'une archive**, choisir cette seule
variante : cloner l'URL reçue, sélectionner `git checkout --detach COMMIT_REMIS`
avec le commit complet et contrôler `git rev-parse HEAD` et `git status --short`.
Les commandes suivantes partent alors de ce checkout propre. `verify_delivery.py`
concerne l'archive à manifeste ; il ne valide pas un clone contenant `.git`.

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

**Attendu :** chemin neuf, `external` et vrai compte LDAP, sans `REPLACE_...`.
Ce compte doit être interne et membre direct d'un groupe administrateur déclaré
dans `access.json`. Il n'a pas besoin d'être administrateur technique du LDAP.
Le compte Unix SSH et le lecteur annuaire sont deux autres identités. Dans la
sandbox seulement, l'administratrice de recette était `alice.martin`.

## 4. Vérifier les prérequis

**Où : Ubuntu, `sudo`.** Docker doit avoir été préparé par l'exploitant selon
les dépôts/proxy approuvés du SI.

```bash
# Vérifie le client et le moteur Docker accessibles à l'opérateur.
sudo docker version
# Vérifie le plugin Compose utilisé par les commandes manage.py.
sudo docker compose version
# Vérifie Buildx ; la livraison conserve cette vérification de prérequis.
sudo docker buildx version
# Vérifie OpenSSL utilisé pour les secrets locaux.
openssl version
# Affiche l'espace du disque Docker sans supprimer de fichier.
df -h /var/lib/docker
```

**Attendu :** commandes réussies et capacité approuvée pour images, rapports,
volume et sauvegarde. Les volumes du SI réel ne sont pas dimensionnés par les
mesures ARM64 du laboratoire. `vm.max_map_count` relève du serveur OpenSearch
existant ; le PoC externe n'en modifie pas le noyau.

Si un prérequis manque, le faire préparer par l'exploitant avant de poursuivre.
Le programme `deploy/ubuntu/prerequisites.py --install` est une **option explicite**
pour un Ubuntu de laboratoire autorisé à utiliser les dépôts publics ; ne pas
le lancer par défaut dans le SI filtré. Il peut modifier les paquets et le sysctl.

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

**Attendu :** mode `external`, aucun service démarré, portail activé, puis
permissions `700`, `700`, `600`. Ne pas réexécuter `prepare` pour corriger une
instance existante. Conserver les clés de session et de service générées.

## 6. Renseigner les sources, droits et Albert

**Où : Ubuntu, `sudo`.** Les paramètres viennent de la fiche complétée. Les
trois fichiers Incidents ci-dessous doivent avoir été préparés au format du
PoC à partir des données et contrats réels, puis relus avec leurs responsables.

### Copier les modèles et les fichiers remis

```bash
# Installe le modèle du moteur PDL ; il ne configure pas le serveur OpenSearch.
sudo install -m 600 deploy/ubuntu/examples/pdl.json "$RIE_INSTANCE/config/pdl.yaml"
# Installe le modèle d'identité du PoC ; il ne configure pas le serveur LDAP.
sudo install -m 600 deploy/ubuntu/examples/access.json "$RIE_INSTANCE/config/access.json"
# Installe les règles initialement fermées, à personnaliser avant démarrage.
sudo install -m 600 deploy/ubuntu/examples/policy.json "$RIE_INSTANCE/access/policy.json"
# Copie l'inventaire, les fenêtres et champs Incidents relus pour ce SI.
sudo install -m 600 "$RIE_SI/incidents-corpus.json" "$RIE_INSTANCE/config/incidents-corpus.json"
# Copie le contrat validé des séries, labels et unités.
sudo install -m 600 "$RIE_SI/metric-mapping.json" "$RIE_INSTANCE/config/metric-mapping.json"
# Copie le snapshot autorisé des alertes et traitements courants, distinct des précédents par API.
sudo install -m 600 "$RIE_SI/itsm-tickets.json" "$RIE_INSTANCE/config/itsm-tickets.json"
# Installe la CA approuvée de l'annuaire.
sudo install -m 600 "$RIE_SI/ldap-ca.crt" "$RIE_INSTANCE/config/ldap-ca.crt"
# Installe le bundle de confiance des sources et d'Albert.
sudo install -m 600 "$RIE_SI/ca-sources.pem" "$RIE_INSTANCE/config/ca-sources.pem"
# Installe le secret du lecteur annuaire sans l'afficher.
sudo install -m 600 "$RIE_SI/private/ldap-reader.password" "$RIE_INSTANCE/secrets/reader.password"
# Installe le secret du lecteur OpenSearch sans l'afficher.
sudo install -m 600 "$RIE_SI/private/opensearch.password" "$RIE_INSTANCE/secrets/opensearch_password"
# Installe le token lecteur des mesures sans l'afficher.
sudo install -m 600 "$RIE_SI/private/victoria.token" "$RIE_INSTANCE/secrets/victoria_token"
# Installe la clé Albert autorisée pour le serveur ; le client local a sa configuration propre.
sudo install -m 600 "$RIE_SI/private/albert.key" "$RIE_INSTANCE/secrets/albert_key"
```

**Attendu :** aucune sortie en cas de succès. Ces copies ne doivent pas être
rejouées sur une instance déjà personnalisée. Les exemples complets sont dans
`deploy/ubuntu/examples/`. Le marqueur `requires_operator_configuration` impose
une revue : le **supprimer après personnalisation**, pas le mettre à `false`.
Les fichiers restent privés et distincts des sources extraites.

### Définir les paramètres de connexion

```bash
# Ouvre l'environnement privé pour renseigner les seules lignes de la fiche, sans dupliquer les clés.
sudoedit "$RIE_INSTANCE/.env"
```

| Clé à renseigner | Valeur provenant de la fiche |
| --- | --- |
| `RIE_IMAGE`, `RIE_PORTAL_IMAGE` | Les deux références approuvées avec digest de la registry GitLab ; même version des sources |
| `OPENSEARCH_URL`, `OPENSEARCH_USER` | URL HTTPS du cluster et compte lecteur |
| `VICTORIAMETRICS_URL` | URL HTTPS des mesures ; son token est dans le fichier privé |
| `INCIDENTS_LOGS_INDEX`, `INCIDENTS_DOCS_INDEX` | Index physiques réels des logs et fiches Incidents |
| `INCIDENTS_MANIFEST` | `/data/config/incidents-corpus.json` |
| `SOURCE_CA_FILE` | `/data/config/ca-sources.pem` |
| `ALBERT_URL`, `ALBERT_MODEL` | API et modèle de planification PDL approuvés et disponibles |
| `OBS_MODEL_GPTOSS` | Modèle de rédaction Incidents ; le nom technique de ce preset n'impose pas la famille du modèle |
| `OBS_DIALOGUE_MODEL`, `OBS_REVIEW_MODEL` | Modèles d'interprétation et de relecture qualifiés |

Conserver `RIE_MODE=external`, les identifiants générés `RIE_PROJECT`,
`RIE_INSTANCE_ID`, `RIE_INSTANCE_DIR`, les chemins internes, ports et budgets.
Ne pas recopier le `.env` d'une autre instance. Ne pas mettre de secret dans
les URL. Le laboratoire utilise Gemma 4 pour PDL/rédaction et GPT-OSS pour
dialogue/relecture ; ces disponibilités et quotas restent à confirmer dans le SI.

### Définir les index PDL et le LDAP

**`config/pdl.yaml` est propre au PoC**, lu par le moteur PDL ; le fichier natif
du serveur OpenSearch est `opensearch.yml`, géré par son administrateur.
**`config/access.json` est également propre au PoC**, commun aux deux UC ;
son objet `ldap` explique comment interroger l'annuaire existant. Il ne crée
ni serveur LDAP, ni comptes ni groupes.

```bash
# Ouvre la configuration PDL pour renseigner les index et retirer le marqueur après revue.
sudoedit "$RIE_INSTANCE/config/pdl.yaml"
# Ouvre la configuration d'identité pour reporter annuaire et groupes réels.
sudoedit "$RIE_INSTANCE/config/access.json"
# Ouvre les ACL initiales pour décider chaque droit, puis retirer le marqueur après revue.
sudoedit "$RIE_INSTANCE/access/policy.json"
```

Dans `pdl.yaml`, renseigner `discovery.allowed_index_patterns` et
`required_index_patterns` avec les index réels des familles retenues. Le premier
limite la découverte ; le second impose leur présence avant `ready`. Les
concepts/champs seront revus à l'étape 8. Conserver TLS, chemins `/data`,
variables `${...}` et limites du modèle. Le fichier contient du JSON valide,
accepté comme YAML ; respecter les virgules.

| Champ de `access.json`, objet `ldap` | Valeur à renseigner et sens |
| --- | --- |
| `host`, `port` | DNS et port LDAPS réels, couverts par le certificat |
| `use_tls`, `validate_cert` | Conserver `true`, connexion chiffrée et certificat vérifié |
| `ca_file` | `/data/config/ldap-ca.crt`, vu depuis les conteneurs |
| `app_dn` | DN du seul lecteur technique de l'annuaire |
| `bind_password_file` | `/data/secrets/reader.password`, distinct du mot de passe utilisateur |
| `search_base` | DN de la branche contenant les utilisateurs à rechercher |
| `attribute_for_username`, `attribute_for_mail` | Noms des attributs de connexion et courriel ; pas les valeurs d'un utilisateur |
| `attribute_for_groups`, `attribute_for_subject` | Attribut des groupes et attribut stable unique ; noms à confirmer dans l'annuaire |
| `group_dns.internal`, `group_dns.external` | DN réels des groupes de classification |
| `administrator_group_dns` | Liste des DN administrateurs des accès du PoC ; appartenance directe d'un interne |

Conserver `allow_webui_assertions: false`. Un en-tête provenant du client ne
remplace pas une authentification LDAP. Les groupes imbriqués ne sont pas
développés automatiquement ; un compte non classé ou classé interne **et**
externe est refusé.

#### Comprendre les noms LDAP

`cn` nomme une entrée, `ou` une branche et `dc` un composant de la racine.
Un DN est leur chemin complet, par exemple le **fictif**
`cn=reader,ou=services,dc=example,dc=invalid`. Ce chemin est distinct du DNS
du serveur. Les DN du SI viennent de son administrateur ; ils ne se déduisent
pas de l'adresse électronique. Le lecteur recherche le compte, le PoC vérifie
le mot de passe de l'utilisateur, relit son identité stable et ses groupes,
puis applique les règles. Les mots de passe utilisateurs ne sont pas stockés.
La [fiche de raccordement](26-si-connection-sheet.md#identités-et-droits)
et le [glossaire](23-glossary.md) explicitent ces termes.

### Choisir explicitement les droits initiaux

Dans la politique, conserver `version: 1`, une révision initiale `1` et
`default: blocked`. Déclarer les **noms exacts** des index PDL, de logs et de
documents Incidents. Décider séparément `incidents.metrics` et
`incidents.tickets` dans `resources` :

| Valeur | Lecture autorisée |
| --- | --- |
| `blocked` | Personne, administrateur compris |
| `internal` | Comptes classés internes |
| `authenticated` | Comptes classés internes ou externes ; pas d'accès anonyme |

Un diagnostic complet exige les deux index Incidents **et** les deux ressources.
Les droits natifs ITSM s'ajoutent aux règles du PoC. Après cette préparation,
les modifications passent par l'administration ACL ; ne pas écraser les
révisions/audits actifs avec un ancien fichier de politique.

## 7. Valider, récupérer les images et démarrer

**Où : Ubuntu, `sudo`.** Ne pas lancer `build` dans ce parcours : les images
approuvées sont préparées en amont pour la registry filtrée.

```bash
# Valide fichiers privés, contrats et paramètres avant toute interruption ; ne génère aucune réponse Albert.
sudo python3 deploy/ubuntu/manage.py validate "$RIE_INSTANCE"
# Lit seulement la référence publique de l'image runtime depuis l'environnement privé.
export RIE_RUNTIME_IMAGE="$(sudo sed -n 's/^RIE_IMAGE=//p' "$RIE_INSTANCE/.env")"
# Lit seulement la référence publique de l'image NiceGUI.
export RIE_PORTAL_IMAGE="$(sudo sed -n 's/^RIE_PORTAL_IMAGE=//p' "$RIE_INSTANCE/.env")"
# Récupère l'image métier approuvée ; utiliser l'authentification registry préparée pour cet opérateur Docker.
sudo docker pull "$RIE_RUNTIME_IMAGE"
# Récupère l'image du portail approuvée de la même version.
sudo docker pull "$RIE_PORTAL_IMAGE"
# Démarre uniquement les services du PoC, après validation et disponibilité locale des images.
sudo python3 deploy/ubuntu/manage.py up "$RIE_INSTANCE"
# Affiche l'état des processus sans modifier leurs données.
sudo python3 deploy/ubuntu/manage.py status "$RIE_INSTANCE"
```

**Attendu :** validation réussie, images récupérées avec les digests attendus,
puis `pdl`, `incidents` et `portal` sains. L'authentification à la registry doit
être celle utilisée par `sudo docker`, pas seulement celle du compte sans sudo.
Ne pas passer son mot de passe en argument. Le compte et les droits registry
viennent du SI. Un refus de récupération se résout avant `up`.

Des processus `healthy` ne prouvent pas que le catalogue métier est prêt.
Avant l'étape 8, son absence est normale : ne pas lancer `seed` pour obtenir
artificiellement `ready`. La validation locale ne qualifie ni les lectures
réelles ni la génération Albert. En cas d'erreur, conserver le message sans
secret et examiner `status`/`logs` ; ne pas relancer `prepare`.

## 8. Revoir et approuver les mappings PDL

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
2. Cliquer **Actualiser les sources**. Choisir **Ajouter un index découvert**,
   ou importer le fichier initial si une proposition a été fournie. Une
   proposition déjà chargée n'est pas à recréer.
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

**Attendu :** état `ready`, mappings attendus actifs et chaque source requise
prête. Une erreur ITSM après activation se traite aussi avec les liaisons de
l'étape 9. L'administration autonome historique `rie_agent/admin_app.py`
n'est pas le portail Ubuntu. L'alternative CLI est décrite dans le
[guide mapping](04-pdl.md#revoir-puis-approuver-les-mappings) ; elle remplace
la saisie UI, sans seconde approbation. Utiliser un véritable identifiant de
revue retourné par l'outil, jamais le texte littéral `REVIEW_ID`.

## 9. Raccorder les précédents ITSM-NG

**Où : Ubuntu, `sudo`, après le premier démarrage.** La version qualifiée est
**1.6.17**. Le snapshot courant a été copié à l'étape 6 ; le connecteur ci-dessous
lit séparément les précédents et solutions historiques par API.

```bash
# Installe le modèle du lecteur historique dans un fichier privé à personnaliser.
sudo install -m 600 deploy/ubuntu/examples/itsm-history.json "$RIE_INSTANCE/config/itsm-history-candidate.json"
# Renseigne API HTTPS, URL des tickets, version, fuseau et provenance réelle ; conserve le chemin conteneur des credentials.
sudoedit "$RIE_INSTANCE/config/itsm-history-candidate.json"
# Lie le compte LDAP administrateur à ses seuls credentials ITSM fournis séparément.
sudo python3 deploy/ubuntu/manage.py itsm-user "$RIE_INSTANCE" --username "$RIE_ADMIN_LDAP" --input "$RIE_SI/private/$RIE_ADMIN_LDAP-itsm.json"
```

Répéter `itsm-user` pour **chaque utilisateur autorisé**, en remplaçant
`--username` et le fichier par ceux du titulaire. Le fichier privé contient
son `itsm_user_id` numérique et son `user_token`. Ne pas utiliser le compte
ITSM de l'administrateur pour représenter les autres utilisateurs.

**Seulement si le serveur ITSM exige un App-Token**, exécuter cette commande :

```bash
# Ajoute l'App-Token depuis le fichier privé, sans afficher sa valeur ni la passer en argument.
sudo python3 -c 'import json,pathlib,sys; p=pathlib.Path(sys.argv[1]); d=json.loads(p.read_text()); d["app_token"]=pathlib.Path(sys.argv[2]).read_text().strip(); p.write_text(json.dumps(d,indent=2)+"\n"); p.chmod(0o600)' "$RIE_INSTANCE/secrets/itsm-users.json" "$RIE_SI/private/itsm-app.token"
```

Puis, dans tous les cas :

```bash
# Enregistre le raccordement relu, sans contacter la source ni activer une écriture ITSM.
sudo python3 deploy/ubuntu/manage.py configure-itsm-history "$RIE_INSTANCE" --input "$RIE_INSTANCE/config/itsm-history-candidate.json"
# Valide l'instance et les credentials avant l'application.
sudo python3 deploy/ubuntu/manage.py validate "$RIE_INSTANCE"
# Recharge les seuls services PoC ; interruption courte prévue.
sudo python3 deploy/ubuntu/manage.py up "$RIE_INSTANCE"
# Vérifie la lecture API avec l'identité associée au compte LDAP de recette.
sudo python3 deploy/ubuntu/manage.py check "$RIE_INSTANCE" --username "$RIE_ADMIN_LDAP"
```

**Attendu :** credentials enregistrés sans valeur affichée, état `ready` et
`incidents_itsm-ng-history: ok`. Une recherche vide mais complète peut réussir
cette sonde ; retrouver un précédent connu et comparer deux profils à l'étape
11 reste obligatoire. Pour des données réelles, renseigner leur provenance et
`synthetic: false` ; modifier ce marqueur ne transforme pas des données fictives.
Un SSO ITSM, mTLS, autre version ou proxy imposé exige une recette propre.

L'API historique n'alimente pas le snapshot courant et n'écrit aucun ticket.
Convenir avec son responsable de la mise à jour de `itsm-tickets.json`.

## 10. Connecter NiceGUI et Open WebUI local

**Première installation : réaliser la préparation serveur puis la préparation
du poste, avant l'autorisation personnelle.** Aucune sous-partie n'est supposée
« déjà faite ». Le déployeur Ubuntu et l'administrateur du poste peuvent être
deux personnes différentes ; convenir de qui réalise chaque action.

### Sur Ubuntu : activer la connexion individuelle

```bash
# Installe le modèle public OAuth dans un fichier privé à compléter.
sudo install -m 600 deploy/ubuntu/examples/oauth.json "$RIE_INSTANCE/config/oauth-candidate.json"
# Renseigne l'émetteur, les deux URLs MCP HTTPS et les callbacks locaux exacts de la fiche.
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

**Attendu :** import et validation réussis, puis `ready`. L'émetteur OAuth est
l'origine publique du moteur PDL. Le SI expose ses routes MCP, OAuth, `/login`
et `/.well-known/...` sans réécrire les chemins ; l'origine Incidents expose
aussi ses métadonnées et son endpoint MCP. Le proxy conserve `Authorization`
et la réponse `WWW-Authenticate`. DNS, CA et accès doivent fonctionner depuis
le **navigateur et le backend Open WebUI**, pas uniquement depuis Ubuntu.

Les callbacks sont exactement les deux URLs de retour du client local, par
exemple sur son origine `http://127.0.0.1:8080` :
`/oauth/clients/mcp:dinum-pdl-agent/callback` et
`/oauth/clients/mcp:obs-gptoss/callback`. Un port différent impose des callbacks
différents. Ce fichier configure le PoC, pas un fournisseur SSO du SI.

### Sur le poste : préparer le client local

**Prérequis :** Open WebUI local déjà installé, Docker/Compose, Python 3.10+,
sources publiques de la même version, compte administrateur local et fournisseur
Albert configuré. Les commandes partent de la racine des sources **sur le poste**,
pas du terminal SSH Ubuntu.

Sauvegarder avant modification le volume `/app/backend/data`, la clé
`WEBUI_SECRET_KEY` et les fichiers de lancement. Conserver réseau, CA, paramètres
LDAP/Albert, volume persistant et clé stable. L'image fournie repose sur
Open WebUI **0.11.3**, avec l'adaptateur de renouvellement qualifié ; une autre
version exige ses tests avant remplacement. L'overlay conserve un seul worker.

Pour un client géré par Compose, remplacer les valeurs de la commande et
reprendre les fichiers `-f`, `--env-file` et nom de projet habituels :

```bash
# Désigne l'image cliente approuvée reçue du responsable de la registry ; remplacer la référence avant exécution.
export RIE_OPENWEBUI_IMAGE=registry.REPLACE.invalid/REPLACE_PROJECT/rie-openwebui:REPLACE_APPROVED_VERSION
# Récupère cette image avec l'opérateur Docker local autorisé.
docker pull "$RIE_OPENWEBUI_IMAGE"
# Recrée uniquement le client local ; remplacer CLIENT_COMPOSE et conserver les paramètres du projet existant.
docker compose -f CLIENT_COMPOSE -f deploy/openwebui/compose.override.yaml up -d --no-deps openwebui
```

**Attendu :** client accessible sur son port habituel, comptes et conversations
conservés. L'overlay suppose un service nommé `openwebui` : s'il diffère,
l'administrateur adapte une copie. Un client lancé autrement se recrée avec
ses mêmes paramètres par son administrateur ; ne pas improviser un volume neuf.
En cas d'échec, reprendre l'image/configuration précédentes avec la sauvegarde.

Ouvrir le client, configurer/contrôler sa connexion Albert et relever
l'**identifiant complet du modèle relais** dans ce client. Il peut comporter
un préfixe de connexion ; ne pas reprendre le préfixe du Mac de développement.
Renseigner ensuite, dans ce même terminal du poste :

```bash
# Désigne l'origine réelle du client local, identique à celle des callbacks.
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

**Attendu :** `configured: true`, `personal_mcp_token_required: false` et
`user_authorization_pending: true`. Un snapshot privé de retour arrière est
conservé hors sources. Les autres connexions sont préservées. Cette préparation
ne connecte aucun utilisateur au LDAP et ne demande pas de jeton MCP à copier.

### Autoriser les deux sources dans le navigateur

1. Se connecter normalement à son Open WebUI avec un **compte utilisateur
   ordinaire**, pas seulement le compte administrateur du client.
2. Vérifier que **SI-RIE — PDL** et **SI-RIE — INCIDENTS** sont visibles dans
   le sélecteur. Leurs identifiants sont `albert-pdl-rie` et
   `albert-observabilite-gptoss`. Les outils et profils sont partagés en lecture
   aux utilisateurs authentifiés du client ; cela n'accorde pas les droits sources.
3. Ouvrir [Autoriser PDL](http://127.0.0.1:8080/oauth/clients/mcp:dinum-pdl-agent/authorize),
   vérifier l'adresse du PoC puis saisir **son propre compte LDAP réel** et
   confirmer. Adapter l'origine du lien si le client utilise un autre port.
4. Ouvrir [Autoriser Incidents](http://127.0.0.1:8080/oauth/clients/mcp:obs-gptoss/authorize),
   vérifier l'identité LDAP affichée et confirmer. **Utiliser un autre compte**
   permet de corriger l'identité ; chaque source a sa propre autorisation.
5. Revenir dans Open WebUI. Les deux autorisations doivent ramener à ce même
   client local. Un outil seulement visible ou configuré par défaut ne suffit pas.

Les mots de passe LDAP ne sont pas conservés. Le client chiffre ses autorisations
et renouvelle les accès courts ; une révocation ou une échéance impose une nouvelle
autorisation. Le PoC relit les groupes et règles à chaque action. Le SSO d'une
autre application ne démontre pas un raccordement SSO de ce PoC. Le
[guide des accès](25-user-access.md) détaille durées, révocations et dépannage.

## 11. Jouer les deux UC et leurs ACL avec Albert

**Où : navigateur du poste.** Reprendre les cas **réels et attendus** de la fiche,
sans copier `demo-alpha`, les dates de 2026 ou les équipements du laboratoire.
Consigner demandes, identité, fenêtre/fuseau, résultat, `request_id` et appels
Albert réellement exécutés. Un état `ready` ne prouve aucune génération.

### PDL : clarification puis reprise du même fil

1. Dans un nouveau fil Open WebUI, choisir **SI-RIE — PDL**.
2. Poser le cas de référence en omettant seulement sa période.
3. Attendre la demande de période, sans résultat fabriqué.
4. Donner date complète, heures et fuseau dans **ce même fil**.
5. Comparer le résultat et la période aux données de référence OpenSearch.
   Contrôler les sources, les éventuelles limites et les appels Albert du rapport.
6. Dans [NiceGUI PDL](http://127.0.0.1:28090/demo-pdl-5min), se connecter avec
   ce même compte LDAP. Remplacer la **Question métier** préremplie par la
   **demande réelle complète**, période comprise ; laisser la reprise décochée
   puis cliquer **Investiguer la question**. Comparer au même attendu.
7. Cliquer **Lire mon catalogue approuvé** : `Résultat · ok`, liste `catalogue`
   des index accessibles approuvés, versions et `concepts` attendus. `mapping`
   peut être `null`, les correspondances sont dans `concepts`. Ce bouton lit
   la configuration ; il ne lance ni recherche de logs ni génération Albert.

**Attendu :** cas exact conforme, une action métier vérifiable par tour,
clarification et reprise cohérentes. Un outil seulement mentionné ou « aucun
traitement vérifiable » est un échec. Les quatre familles PFS, BB, PFAING et
TACACS sont à réceptionner sur les index réellement raccordés, avec le métier.

### Incidents : diagnostic et précédents natifs

Dans un nouveau fil Open WebUI, choisir **SI-RIE — INCIDENTS**. Demander le
diagnostic de l'équipement et de la fenêtre de référence réels, avec précédents,
solution proposée, conditions et confiance. Comparer cinq mesures, couverture,
logs et chronologie aux outils sources, puis un précédent connu à ITSM-NG.
La résolution technique doit rester distincte de la clôture administrative.
Une solution reste conditionnelle, sourcée et à valider par un opérateur.

Contrôler **rédaction et relecture Albert**, modèle retourné et tokens, ainsi
que les deux niveaux de confiance. `partial`, synthèse non validée, appel
absent ou fournisseur indisponible ne valent pas réussite de la génération.
Attendre au moins **60 secondes après la réponse** avant une nouvelle synthèse
Incidents. Une limitation explicite est consignée puis retentée manuellement ;
ne pas désactiver le budget pour faire passer la recette.

La page **démo Incidents cinq minutes de NiceGUI** garde son scénario et sa
fenêtre de laboratoire. Elle ne permet pas de choisir librement les données
réelles : ne pas la compter comme recette du SI réel. Le parcours métier réel
est celui d'Open WebUI/MCP ci-dessus ; NiceGUI porte aussi l'administration
et le rapport Allure. La fiche sandbox conserve ses attendus de démo.

### ACL : lecteurs, refus et rétablissement

Utiliser des sessions navigateur distinctes et les propres autorisations LDAP
de chaque titulaire. Ne pas réutiliser l'autorisation de l'administrateur.

| Profil réel à fournir | Vérification |
| --- | --- |
| Interne administrateur | Lectures selon politique et administration autorisée |
| Interne lecteur | Lectures autorisées, administration ACL/mappings/qualification refusée |
| Externe | PDL seulement si `authenticated` ; sources Incidents selon les règles réellement décidées |
| Non classé | Connexion et lectures protégées refusées |

Dans [Administration ACL](http://127.0.0.1:28090/admin-acces), noter la révision.
Passer **Mesures VictoriaMetrics** à `blocked`, enregistrer, vérifier la hausse
de révision et rejouer le cas Incidents réel : **refus même pour l'administrateur,
sans preuve interdite ni ancien diagnostic réutilisé**. Rétablir exactement
la règle initiale, enregistrer et refaire une demande après admission : nouvelles
preuves attendues si le compte est autorisé. Répéter pour **Tickets et suivi ITSM**.
Conserver les autres règles et les décisions initiales approuvées.

Comparer aussi deux utilisateurs ayant des droits natifs ITSM différents :
entités, tickets, solutions et suivis privés ne doivent pas être élargis par le
PoC. Un compte administrateur local Open WebUI n'est pas une preuve du parcours
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

**Attendu :** services sains, `ready`, erreurs comprises et espace disponible.
Examiner les logs en privé avant d'en joindre un extrait. Les rapports/audits
sont conservés ; convenir de leur archivage avec l'exploitant.

### Arrêter, sauvegarder et reprendre

```bash
# Choisit une destination de sauvegarde neuve, privée, hors sources et instance ; adapter au stockage approuvé.
export RIE_BACKUP="$HOME/rie-backups/si-deploy-20261117"
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
à sauvegarder séparément. Ne pas utiliser la fonction `acceptance` avec ses
attendus fictifs pour déclarer la recette réelle réussie.

### Restaurer réellement dans une instance neuve

```bash
# Choisit une destination inexistante et distincte de l'originale.
export RIE_RESTORED="$HOME/.local/share/dinum-rie-si-restored-20261117"
# Arrête l'originale pour libérer les ports ; toutes ses données restent conservées.
sudo python3 deploy/ubuntu/manage.py stop "$RIE_INSTANCE"
# Vérifie toutes les empreintes puis restaure dans la nouvelle instance, sans prepare préalable.
sudo python3 deploy/ubuntu/manage.py restore "$RIE_RESTORED" --backup "$RIE_BACKUP"
# Démarre la copie avec les images et fichiers sauvegardés.
sudo python3 deploy/ubuntu/manage.py up "$RIE_RESTORED"
# Vérifie sources et mappings dans la copie restaurée.
sudo python3 deploy/ubuntu/manage.py check "$RIE_RESTORED" --username "$RIE_ADMIN_LDAP"
```

**Attendu :** intégrité acceptée, nouvelle instance et `ready`. Comparer la
révision ACL, les mappings, audits et états des autorisations. Rejouer les deux
cas métier avec Albert, un refus et un accès personnel encore valide. Les
sources externes n'ont pas été remplacées. Une copie de sauvegarde corrompue
doit être refusée avant écriture ; ne jamais altérer la bonne archive pour ce test.

Le dernier service actif est **l'instance restaurée** ; conserver l'originale
arrêtée. Une révocation postérieure à la sauvegarde doit être réappliquée avant
remise aux utilisateurs. L'expiration réelle reste opposable. Si le client a
renouvelé ses autorisations depuis la sauvegarde, une nouvelle autorisation
peut être nécessaire ; ne pas contourner le refus de credentials périmés.

```bash
# Sélectionne désormais l'instance restaurée active pour les futures commandes d'exploitation.
export RIE_INSTANCE="$RIE_RESTORED"
```

### Clore la séance

Consigner date, opérateur, OS/architecture, commit, archive/manifeste, images
et digests, chemins, révision ACL, mappings, requêtes, modèles réellement
retournés et références des rapports. Pour chacune des douze étapes :
**réussi**, **échoué** ou **non fait**, avec preuve ou raison. L'approbation
métier humaine reste distincte des contrôles automatisés et de la santé Docker.

Les tests sont consultables dans [Qualification NiceGUI](http://127.0.0.1:28090/qualification).
Pour ajouter un rapport Allure reçu, utiliser la commande suivante avec le vrai
chemin du ZIP ; cette importation ne lance aucun test et ne crée aucune réussite :

```bash
# Importe le ZIP de qualification vérifié dans l'instance active ; remplacer REPORT_ZIP.
sudo python3 deploy/ubuntu/manage.py import-report "$RIE_INSTANCE" --input REPORT_ZIP
```

La [matrice de réception SI-RIE](14-si-rie-acceptance.md) attribue les contrôles
réseau, PKI, registry, schémas, identités, données, quotas Albert et charge
restant à effectuer dans le SI réel. La
[qualification datée](08-qualification.md) conserve les limites de l'ARM64 et
des campagnes historiques ; ce guide ne les transforme pas en mesures AMD64.

## Préparation des images — responsable de la construction, avant l'étape 1

Le responsable utilise **les mêmes sources figées** que le déployeur, dans un
environnement de construction autorisé. Il peut préparer une instance de
construction isolée en `external`, activer le portail puis lancer
`manage.py build` selon le [manuel de référence](01-installation.md). Cette
construction ne demande ni les secrets ni un accès aux sources réelles du SI.
Le runtime reste distinct de NiceGUI ; la documentation ne change pas le runtime.

Après construction, identifier les images produites dans le `.env` privé de
cette instance, puis les étiqueter/publier dans le projet GitLab **autorisé**
avec les commandes `docker tag`/`docker push` du circuit SI. Remettre les
références **par digest**, architecture, commit source et résultats de tests.
L'image cliente se construit avec `deploy/openwebui/Dockerfile` ; son argument
`OPENWEBUI_BASE` permet de choisir la base miroir approuvée. Le Dockerfile du
runtime, ses bases, le frontend BuildKit et les dépendances verrouillées doivent
être disponibles dans l'environnement de construction selon le circuit autorisé.

Aucune adresse de registry SI ni habilitation de publication n'est présumée
disponible depuis le Mac. La publication réelle est un prérequis de la fiche,
pas un résultat revendiqué par le paquet source ou ses empreintes.

## Reprendre après une déconnexion SSH

Retrouver dans le compte rendu les chemins et le compte de **cette installation**.
Redéfinir `RIE_SI`, `RIE_SOURCES`, `RIE_INSTANCE`, `RIE_MODE=external` et
`RIE_ADMIN_LDAP` avec ces valeurs, puis revenir à la racine des sources.
Si l'étape 12 a été réalisée, utiliser le dossier **restauré**, pas l'original.
Reprendre au dernier contrôle réussi ; ne pas rejouer `prepare`, l'extraction,
les copies de modèles ni les approbations déjà faites.

Pour l'exercice conservé du 16 novembre uniquement, l'instance active est
`/home/dinum/.local/share/dinum-rie-si-restored-20261116` ; le fichier privé de
reprise est `~/VALIDATION-STATE.md`. Ces chemins historiques ne sont pas ceux
d'un nouveau déploiement. Les accès Lima/Colima sont décrits dans la fiche
sandbox, hors du parcours SI réel.
