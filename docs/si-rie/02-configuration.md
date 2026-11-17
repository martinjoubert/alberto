# Configurer l'instance et raccorder les sources

Une instance rassemble ses paramètres, secrets et volumes. Le code et le wiki
ne doivent contenir ni clé Albert, ni mot de passe annuaire, ni jeton utilisateur.
Les exemples fictifs du mode `demo` ne deviennent pas une configuration du SI réel.
Toutes les modifications décrites ici se font sur le serveur Ubuntu, dans le
dossier privé `RIE_INSTANCE`, par l'opérateur retenu à l'installation. Employer
`sudoedit` si les fichiers appartiennent à root ; conserver les dossiers en
`0700` et les fichiers en `0600`. Les chemins `/data/...` désignent le conteneur,
pas un répertoire à créer sur le poste utilisateur.

## Choisir le mode

| Mode | Sources | Initialisation |
| --- | --- | --- |
| `demo` | OpenSearch, VictoriaMetrics et LDAP privés, tickets fictifs | Corpus versionné ; mappings de démonstration activés explicitement par le seed |
| `external` | URLs et comptes lecteurs fournis par les responsables du SI | Aucun seed ; portées, mappings et règles à relire |

La simulation utilise de vrais protocoles et bases ; ses équipements, adresses,
identités et tickets sont fictifs. Un snapshot de tickets n'est pas une API
ITSM-NG réelle raccordée. La recherche historique dispose désormais d'un
[connecteur facultatif ITSM-NG 1.6.17](15-itsm-precedents.md), activé séparément
avec des credentials individuels. La [topologie simulée](03-architecture.md#simulation)
permet de reconnaître les limites de la recette.

## Raccorder des sources externes

Ce parcours concerne **une instance externe neuve**, sur le serveur Ubuntu,
avec l'opérateur `sudo` retenu à l'installation. Il ne modifie aucune source du
SI : seuls des fichiers locaux et le catalogue applicatif sont préparés.

**`config/pdl.yaml` configure le moteur PDL du PoC** : accès lecteur à OpenSearch,
index à découvrir, limites de recherche et appels Albert. Il est issu du modèle
`deploy/ubuntu/examples/pdl.json`. La configuration native du serveur OpenSearch
est `opensearch.yml`, gérée par son administrateur ; ces deux fichiers ont des
rôles distincts. Les commandes de ce parcours modifient la configuration du PoC.

**`config/access.json` est également un fichier propre au PoC**, issu du modèle
`deploy/ubuntu/examples/access.json`. Il configure le composant d’identité et
d’accès commun aux deux UC. Son objet `ldap` indique comment interroger
l’annuaire existant ; il ne configure pas le serveur LDAP et ne crée ni comptes
ni groupes. Le [tableau commenté du guide](24-deploy-walkthrough.md#définir-les-index-pdl-et-le-ldap)
explique les valeurs, puis les [noms LDAP et le fonctionnement de la connexion](24-deploy-walkthrough.md#comprendre-les-noms-ldap).

Les responsables des services fournissent une **fiche de raccordement** :
adresses, ports, certificats, comptes, groupes et contrats de données. Elle
sert de référence pour renseigner les fichiers ci-dessous ; les secrets sont
transmis par le canal privé prévu. Voir sa
[définition dans le README](README.md#fiche-de-raccordement),
le [glossaire](23-glossary.md#fiche-de-raccordement) et les
[valeurs de la sandbox](22-external-sandbox.md#fiche-de-raccordement-de-cet-exercice).

1. Faire fournir par les responsables les URLs, CA, comptes lecteurs, groupes,
   index physiques, schémas, séries et export de tickets autorisés.
2. Préparer un nouveau dossier, puis copier les modèles publics livrés :

```bash
# Crée le dossier privé, la configuration initiale et les secrets du PoC ; ne démarre aucun service.
sudo python3 deploy/ubuntu/manage.py prepare "$RIE_INSTANCE" --mode external
# Copie le modèle à adapter pour sources, périmètres et budgets PDL ; à faire sur l'instance neuve avant personnalisation.
sudo cp deploy/ubuntu/examples/pdl.json "$RIE_INSTANCE/config/pdl.yaml"
# Copie le modèle à adapter pour LDAP et identité des utilisateurs ; à faire sur l'instance neuve avant personnalisation.
sudo cp deploy/ubuntu/examples/access.json "$RIE_INSTANCE/config/access.json"
# Copie le modèle à adapter pour règles ACL initiales ; à faire sur l'instance neuve avant personnalisation.
sudo cp deploy/ubuntu/examples/policy.json "$RIE_INSTANCE/access/policy.json"
# Copie le modèle à adapter pour équipements, fenêtres et provenance Incidents ; à faire sur l'instance neuve avant personnalisation.
sudo cp deploy/ubuntu/examples/incidents-corpus.json "$RIE_INSTANCE/config/incidents-corpus.json"
# Copie le modèle à adapter pour séries et unités des mesures ; à faire sur l'instance neuve avant personnalisation.
sudo cp deploy/ubuntu/examples/metric-mapping.json "$RIE_INSTANCE/config/metric-mapping.json"
# Copie le modèle à adapter pour export des alertes et traitements courants ; à faire sur l'instance neuve avant personnalisation.
sudo cp deploy/ubuntu/examples/itsm-tickets.json "$RIE_INSTANCE/config/itsm-tickets.json"
# Copie le modèle à adapter pour propositions de correspondance entre concepts PDL et champs physiques ; à faire sur l'instance neuve avant personnalisation.
sudo cp deploy/ubuntu/examples/mappings.yaml "$RIE_INSTANCE/config/mappings.yaml"
```

Les fichiers cibles existent déjà, à l'exception du mapping opérateur : leur
mode privé est conservé. Pour le nouveau mapping, appliquer
`sudo chmod 600 "$RIE_INSTANCE/config/mappings.yaml"`.
Le guide `deploy/ubuntu/examples/README.md` explique chaque champ et le format
exact du snapshot. Ces exemples sont dans le paquet source, pas dans l'image.

3. Adapter les fichiers avec `sudoedit`. Remplacer chaque `REPLACE_...` et
   `replace-...`, retirer `requires_operator_configuration` **après revue**,
   déclarer les index exacts et choisir les visibilités des deux ressources UC2.
   Conserver `default: blocked`. Un droit absent reste refusé.
4. Renseigner `.env` et les secrets selon les tableaux ci-dessous. Ne pas
   remplacer les clés de session et de service générées par `prepare`.
5. Lancer `validate`, puis le parcours scripté externe ou les phases manuelles
   `build`, `pull`, `up`. Aucun seed ni recette fictive sur cette instance.
6. Faire le `refresh` et la [revue des mappings](04-pdl.md#revoir-puis-approuver-les-mappings),
   puis `check --username LDAP_ADMIN_USERNAME`. Le catalogue vide est attendu
   avant cette revue ; ne pas annoncer `ready` pour le contourner.

Le diagnostic Incidents complet nécessite les deux index configurés **et**
`incidents.metrics` et `incidents.tickets`. Les quatre décisions se configurent
indépendamment. Le mode `external` n'ouvre pas automatiquement ces droits.

La [revue des écarts externes](14-si-rie-acceptance.md#écarts-concrets-à-lever-ou-à-accepter-explicitement)
fait partie de ce raccordement. Le manifeste Incidents fournit `anchor_date`,
`timezone`, `start`, `end` et `interval_seconds`. Adapter sa fenêtre de contrôle
aux données réellement présentes, avec bornes ISO et fuseau ; ne pas conserver
les dates fictives du modèle. Le validateur local contrôle ce contrat avant
l'arrêt des services. La présence des données se vérifie ensuite par `check`
et par les questions métier de la matrice.

### Paramètres de raccordement dans `.env`

| Paramètre | Valeur à fournir |
| --- | --- |
| `OPENSEARCH_URL` | URL HTTP(S) du cluster approuvé, sans identifiants dans l'URL |
| `OPENSEARCH_USER` | Compte lecteur limité aux index utiles, ou vide si le mécanisme autorisé n'en utilise pas |
| `VICTORIAMETRICS_URL` | URL HTTP(S) du fournisseur de métriques |
| `INCIDENTS_LOGS_INDEX` | Nom de l'index de logs Incidents, à déclarer dans la politique |
| `INCIDENTS_DOCS_INDEX` | Nom de l'index documentaire Incidents, à déclarer dans la politique ; défaut historique `obs-network-docs` |
| `INCIDENTS_MANIFEST` | Conserver `/data/config/incidents-corpus.json` en externe |
| `SOURCE_CA_FILE` | Chemin conteneur d'une CA source, par exemple `/data/config/ca-sources.pem`, ou vide pour la confiance système approuvée |
| `ALBERT_URL`, `ALBERT_MODEL` | Fournisseur Albert commun et modèle PDL ; défaut des nouvelles instances : `gemma-4-31b-it`. Absence d’accès = génération non qualifiée |
| `OBS_MODEL_GPTOSS`, `OBS_MODEL_GEMMA`, `OBS_MODEL_DEEPSEEK` | Modèle d’un preset Incidents. Les nouvelles instances définissent `OBS_MODEL_GPTOSS=gemma-4-31b-it` pour la rédaction. La relecture est configurée séparément avec `OBS_REVIEW_MODEL`. Si ce champ est absent dans une ancienne instance, le fallback reste `ALBERT_MODEL` |
| `OBS_DIALOGUE_MODEL` | Interprétation du dialogue Incidents : `gpt-oss-120b` explicite dans les nouvelles instances. Si absent, modèle du preset sélectionné |
| `OBS_REVIEW_MODEL` | Relecteur Incidents : `gpt-oss-120b` explicite dans les nouvelles instances. Si absent, le rédacteur est aussi relecteur ; ce profil doit être qualifié séparément |
| `OBS_MIN_TURN_INTERVAL_SECONDS` | Intervalle entre admissions IA Incidents par processus, défaut `60`, valeur entre `0` et `86400` ; `0` exige un choix explicite |

Ne pas modifier `RIE_PROJECT`, `RIE_INSTANCE_ID` ni `RIE_INSTANCE_DIR`. Les ports
peuvent changer en conservant trois valeurs distinctes si NiceGUI est activé.

| Secret privé | Utilisation |
| --- | --- |
| `secrets/reader.password` | Mot de passe du compte lecteur LDAP |
| `secrets/opensearch_password` | Mot de passe du compte lecteur OpenSearch si requis |
| `secrets/victoria_token` | Bearer VictoriaMetrics si requis |
| `secrets/albert_key` | Clé du fournisseur IA configuré sur le serveur |

Éditer ces fichiers avec `sudoedit`, sans placer leurs valeurs dans une commande,
une URL, le wiki ou une sortie de recette. La CA LDAP se dépose séparément dans
`config/ldap-ca.crt` et doit correspondre au nom du serveur LDAP.

## Répartition des fichiers privés

| Fichier dans l'instance | Contenu à vérifier |
| --- | --- |
| `.env` | Projet, ports, images, URLs, modèle et chemins publics de CA |
| `config/pdl.yaml` | Configuration propre au moteur PDL du PoC : index autorisés, limites, sources, TLS et Albert ; distincte du `opensearch.yml` du serveur OpenSearch |
| `config/access.json` | Configuration propre au composant d’identité et d’accès du PoC : raccordement LDAP, base, attributs, groupes et chemins des secrets ; ce n’est pas la configuration native du serveur LDAP |
| `access/policy.json` | Politique initiale des index concrets et ressources UC2, utilisée à la première initialisation |
| `config/incidents-corpus.json` | Inventaire et périmètre Incidents externe |
| `config/metric-mapping.json` | Noms des séries, labels et conversions VictoriaMetrics |
| `config/itsm-tickets.json` | Snapshot de tickets compatible, explicitement identifié |
| `config/itsm-history.json` | Raccordement facultatif aux précédents par API ITSM-NG 1.6.17, URL et fuseau ; activé par `configure-itsm-history` |
| `secrets/itsm-users.json` | Correspondances entre sujets LDAP stables et credentials ITSM individuels, App-Token si nécessaire ; provisionner avec `itsm-user`, pas avec un compte partagé |
| `secrets/` | Clés et mots de passe, dans des fichiers privés |

Configuration et secrets sont recopiés dans le volume propre à l'instance lors
de `up`. Cette commande arrête les consommateurs puis les recrée avec la
configuration ; prévoir une interruption, même lors d'une simple modification.
La politique active et les données métier sont conservées ; après
initialisation, employer les commandes `policy` ou l'administration NiceGUI
pour changer les droits.

## Borner les états de conversation

La livraison Ubuntu configure explicitement `DINUM_DIALOGUE_MAX_STATES=8192`
dans `.env`, pour chacun des deux backends. Cette limite porte sur les états
de reprise conservés par le backend, tous utilisateurs confondus ; elle ne
correspond pas au nombre d'utilisateurs simultanés. La durée de validité reste
d'une heure. Chaque nouveau tour peut créer un état ; à 10 utilisateurs et un
tour toutes les cinq secondes, un backend peut recevoir 7 200 tours par heure.
Cette réserve de fichiers ne constitue pas une mesure de capacité du SI-RIE.

L'opérateur peut choisir un entier entre `1` et `32768`. Le profil Ubuntu
applique aussi `8192` lors de la reprise d'une ancienne instance dont `.env`
ne contient pas encore ce paramètre. Hors de ce profil, le moteur historique
conserve sa limite de `2048` sans configuration explicite. Après une
modification de `.env`, exécuter `validate` puis `up` sur la même instance ;
les processus rechargent le paramètre. Une limite atteinte refuse une nouvelle
reprise ; aucun ancien état encore valide n'est supprimé pour faire de la place.
L'expiration et la liaison des états à leur propriétaire restent contrôlées.

## Préparer le LDAP réel

Utiliser LDAPS avec validation du certificat et de son nom. Fournir le compte
lecteur, son mot de passe, la base de recherche et les attributs d'identifiant,
de mail, de groupes et d'identité stable.

La connexion vérifie la chaîne de confiance et l’identité du serveur avec le
contexte TLS Python/OpenSSL, pour un nom DNS comme pour une adresse IP présente
dans les SAN du certificat. Le correctif du 16 novembre évite le vérificateur
historique de `ldap3` qui ignorait les SAN IP sous Python 3.12. Il conserve la
validation TLS avant l’envoi des identifiants ; aucune option de contournement
n’est nécessaire dans `access.json`.

L'attribut stable est `entryUUID` pour un OpenLDAP compatible ou `objectGUID`
pour Active Directory, après confirmation de l'administrateur. Il doit être
unique et lisible. Les deux groupes `ldap.group_dns.internal` et `external`
classent les lecteurs. Définir séparément `ldap.administrator_group_dns`, liste
de DN de groupes administrateurs, distincts des deux groupes précédents.
Seul un interne membre d'un de ces groupes peut administrer ; liste absente ou
vide = aucun administrateur implicite. Le déployeur doit faire confirmer au moins
un administrateur avant les commandes opérateur et suivre la [migration des
rôles](06-acl.md) pour une instance existante.
Un compte non classé ou membre des deux groupes est refusé ; les groupes
imbriqués ne sont pas développés automatiquement.

Ne pas distribuer de clé de signature Open WebUI aux postes. La configuration
conserve `allow_webui_assertions: false` et utilise les jetons individuels
émis côté serveur. Voir [ACL et identités](06-acl.md).

## Définir le PDL et ses droits

Renseigner les motifs de découverte dans `config/pdl.yaml`, puis les noms
concrets autorisés dans la politique initiale. Ne pas élargir une portée à `*`
pour faire disparaître un refus. Les comptes lecteurs natifs OpenSearch doivent
aussi être limités aux index utiles.

Adapter les concepts, champs et relations du fichier de mappings avec l'expert
métier. Valider, importer des revues en attente, puis approuver leurs identifiants
précis. Un type de champ ne suffit pas à prouver son sens métier ; une IP de
trafic n'est pas automatiquement une IP de gestion. Voir [Technique PDL](04-pdl.md).

## Définir les sources Incidents

Les paramètres `INCIDENTS_LOGS_INDEX` et `INCIDENTS_DOCS_INDEX` désignent les
index physiques de logs et de documents. Le manifeste décrit les équipements
admis et leurs périmètres. Le mapping de
métriques associe les concepts CPU, latence, pertes et utilisation de liens aux
séries et labels du fournisseur. Valider unités, cadence et périodes avec le
responsable de collecte.

Le manifeste accepte `source_fields.logs` et `source_fields.documents` : clés
canoniques du modèle vers chemins réels. Une chaîne emploie le même chemin pour
la lecture et la requête ; un objet distingue, par exemple,
`{"source": "host.name", "query": "host.name.keyword"}`. Les recherches, lecteurs
et liens de preuve emploient ce même mapping. Il n'autorise ni script ni DSL.
Les chemins imbriqués simples sont supportés ; les tableaux `nested` nécessitent
une adaptation. Faire vérifier types, date et sens métier par le propriétaire.

La cadence est choisie par métrique : `hosts[].metric_intervals_seconds`, puis
`metric_intervals_seconds` du manifeste, puis `interval_seconds`. Elle s'applique
aux calculs courants et de référence. Le mapping VictoriaMetrics permet des
`bindings` et `fixed_labels` propres à chaque métrique, en plus des défauts
communs. Le lecteur attend une seule série exacte et des valeurs de type gauge ;
il n'invente ni conversion de compteur par `rate`, ni somme de CPU par cœur.
Normaliser ces cas en amont ou faire développer et qualifier un adaptateur.

La fenêtre du manifeste sert aux sondes de disponibilité ; elle ne limite pas
les dates d'investigation demandées et n'atteste pas une collecte récente.
L'inventaire et le snapshot sont des fichiers locaux relus, sans découverte
automatique du SI. Définir leur mise à jour et vérifier la fraîcheur en réception.

Le corpus externe doit remplacer les marqueurs volontairement incomplets créés
par `prepare --mode external`. Le programme ne doit pas prendre les équipements
fictifs pour inventaire réel. Les précédents ITSM-NG peuvent provenir d'un
snapshot documenté ou du connecteur API en lecture seule. Le
[guide ITSM-NG](15-itsm-precedents.md) décrit le raccordement ; l'accès à l'API
et les droits du SI réel restent à qualifier.

## Raccorder Albert ou une IA interne

Configurer `ALBERT_URL`, `ALBERT_MODEL` et `secrets/albert_key`, valider puis
relancer `up`. La livraison prépare désormais Gemma 4 sur Albert pour la
rédaction Incidents avec `OBS_MODEL_GPTOSS=gemma-4-31b-it` et la relecture
avec `OBS_REVIEW_MODEL=gpt-oss-120b`. La planification et le dialogue PDL utilisent
`ALBERT_MODEL=gemma-4-31b-it` ; le dialogue Incidents conserve
`OBS_DIALOGUE_MODEL=gpt-oss-120b`. Le nom du preset et ses
URL MCP restent compatibles. Le rapport indique le modèle effectivement retourné.
Les instances existantes ne sont pas migrées automatiquement : appliquer ces
valeurs explicitement après sauvegarde. Vérifier l’accès et les quotas de la
clé de déploiement pour chacun des modèles ; leur présence au catalogue seule
ne qualifie pas cet accès.
Les autres presets et le modèle d'interprétation peuvent être remplacés avec
les paramètres du tableau. Vérifier le modèle demandé dans le résultat structuré
et le modèle renvoyé par le fournisseur dans `usage` ; un alias configurable ne
prouve pas les capacités du modèle. Le modèle conversationnel du client Open
WebUI est une configuration distincte.

Le fournisseur du moteur PDL doit prendre en charge les appels de fonction
structurés avec choix imposé (`tools` et `tool_choice`), utilisés par la
correction bornée d'une ambiguïté d'identité. Cette capacité est vérifiée avec
`gpt-oss-120b` sur Albert. Elle s'ajoute aux réponses JSON du planificateur ;
un autre modèle ou fournisseur doit être testé sur ces deux contrats. Une
réponse mal formée n'est pas transformée en lecture par un repli silencieux.

L'intervalle est mesuré depuis l'admission du tour, et non depuis la fin de sa réponse.
L'admission est commune à HTTP `/diagnose`, aux outils MCP historiques et à
l'assistant. Elle précède l'interprétation IA et les lectures d'un diagnostic
avec synthèse. Une admission couvre le tour entier, qui peut comporter plusieurs
appels IA ; ce n'est pas un compteur de tokens ou une garantie de quota fournisseur.
HTTP avec `synthesize: false` n'effectue aucun appel IA et reste utilisable pendant
l'intervalle. Le budget est partagé entre utilisateurs d'un processus, se
réinitialise au redémarrage et n'est pas distribué entre plusieurs serveurs.
Un résultat `rate_limited` indique le délai ; ne pas relancer automatiquement.

Une machine IA interne ne remplace pas automatiquement Albert : vérifier appels
d'outils, réponses structurées, modèle et délais. Une dérogation Internet ne
constitue un accès autorisé qu'après la décision obtenue. La recette technique
sans génération peut être réalisée avant cette disponibilité.

Les CA publiques internes se déposent dans `config/`. Référencer leur chemin
conteneur, par exemple `/data/config/ca-interne.pem`, dans `SOURCE_CA_FILE`.
Conserver la vérification TLS ; ne pas placer d'identifiants dans les URLs.
Le bundle `SOURCE_CA_FILE` est transmis comme `CA_BUNDLE` aux clients des sources
et au fournisseur IA, interprétation comprise. Un bundle personnalisé doit
couvrir toutes ces origines HTTPS, y compris Albert si celui-ci reste public ;
il remplace leur magasin de confiance par défaut. Faire préparer un bundle
approuvé complet plutôt que désactiver la vérification.

## Configurer le client Open WebUI local qualifié

**Parcours courant sans jeton à copier :** suivre la
[connexion individuelle LDAP et OAuth](25-user-access.md). Elle prépare les
deux connexions, filtres et modèles depuis l'API du client local, puis laisse
chaque utilisateur autoriser ses sources dans son navigateur. La procédure
Bearer ci-dessous reste la variante compatible pour scripts/opérateurs et
la description des preuves historiques du 6 novembre ; ne pas l'enchaîner
avec la configuration OAuth du même client.

Ces actions se font **sur le poste de chaque utilisateur**, avec les droits
d'administration de son Open WebUI local. Elles complètent les connexions MCP
du [raccordement du poste utilisateur](01-installation.md#raccorder-le-poste-utilisateur--apr%C3%A8s-a-ou-b).
Le client qualifié le 6 novembre 2026 est **Open WebUI stock 0.11.3-slim**, sur
Mac ARM64 avec Colima, et le modèle Albert `gpt-oss-120b`. Le digest de son image
ARM64 est `sha256:98b795650b5323614d9b1cd7abd20cd1b62245b02fe81671f46f37acea8c6668`.
Une autre version de client ou de fournisseur demande de rejouer cette recette.
La qualification de ce client ne vaut pas qualification d'un serveur AMD64.

### 1. Vérifier réseau, fournisseur et identité

Garder le tunnel SSH ouvert. L'URL MCP doit être accessible **depuis le backend
du client**, et non seulement depuis le navigateur :

| Emplacement du backend Open WebUI | Adresse du serveur MCP après tunnel |
| --- | --- |
| Processus natif sur le poste | `http://127.0.0.1:19910/mcp` et `http://127.0.0.1:19950/gptoss/assistant/mcp` |
| Conteneur Docker Desktop sur Mac | Remplacer `127.0.0.1` par `host.docker.internal` et vérifier la connexion |
| Conteneur Colima de la recette Mac | Ajouter au conteneur l'alias `host.docker.internal` vers l'adresse de `host.lima.internal`, puis vérifier ; cette adresse dépend du profil Colima |
| Conteneur Docker Engine sur Linux | Faire configurer un accès au tunnel du poste ; `--add-host=host.docker.internal:host-gateway` crée un nom mais ne rend pas un tunnel lié à `127.0.0.1` accessible sur la passerelle |

Sous Linux, le réseau hôte est une possibilité pour un client local dédié,
après validation de l'administrateur ; les URL restent alors celles de la
première ligne. Ne pas publier le tunnel sur `0.0.0.0` pour résoudre un refus.
La recette sur ce Mac a utilisé deux clients séparés ; les mots de passe et
jetons de ces clients de qualification ne sont pas des paramètres à recopier.

Dans les paramètres administrateur des connexions OpenAI compatibles, déclarer
l'URL Albert autorisée et la clé du titulaire selon les règles internes. Vérifier
que `gpt-oss-120b` apparaît dans les modèles. Cette clé fournisseur est distincte
du jeton personnel d'accès aux UC et de la clé Albert conservée sur Ubuntu.
Sans accès fournisseur, la génération reste **non qualifiée** ; la recette HTTP
sans synthèse des backends demeure disponible.

Créer ou vérifier ces deux connexions de type **MCP**, authentification
**Bearer**, dans les connexions d'outils du client :

| Identifiant de connexion exact | Filtre des fonctions autorisées | Jeton |
| --- | --- | --- |
| `dinum-pdl-agent` | `investigate_pdl` | Jeton personnel émis sur Ubuntu pour ce titulaire |
| `obs-gptoss` | `diagnose_observability` | Jeton personnel du même titulaire |

Ces identifiants techniques ne sont pas seulement les libellés affichés : le
filtre PDL attend `dinum-pdl-agent_investigate_pdl`. Laisser les en-têtes d'identité
libres vides. Ne pas envoyer `X-OpenWebUI-User-JWT`, de mail ou de rôle comme
autorisation. Conserver `ENABLE_FORWARD_USER_INFO_HEADERS=false` dans la
configuration du client et les outils serveurs activés (`ENABLE_TOOL_SERVERS=true`).
Le bouton de vérification doit retrouver exactement la fonction attendue.
Une vérification de schéma réussie ne prouve pas l'autorisation d'une lecture.

### 2. Importer les deux filtres fournis

Dans **Espace de travail / Workspace → Fonctions / Functions**, créer une
fonction de type filtre à partir du contenu de chacun des fichiers suivants du
**paquet source remis**. Lire ce code avant de l'activer : les fonctions Open
WebUI s'exécutent dans le backend du client. Enregistrer les identifiants exacts,
puis activer les fonctions ; ne pas les rendre globales pour tous les modèles.

| Identifiant exact de fonction | Fichier source à ouvrir/copier |
| --- | --- |
| `pdl_rie_dialogue` | `use-cases/pdl/agentic-pdl/rie_agent/openwebui_dialogue.py` |
| `incident_report_display` | `use-cases/incidents/observability-demo/openwebui/report_display.py` |

Les deux fichiers sont autonomes et inclus dans le manifeste de livraison.
Aucun module d'observabilité ni ancien filtre du portail Mac n'est nécessaire.
Si l'interface génère un identifiant depuis le nom, contrôler l'identifiant
enregistré avant l'association au modèle.

Le filtre PDL capture le message courant avant l'appel et impose son transfert
intégral. Il reprend uniquement le dernier `continuation_token` d'un résultat
d'outil vérifié du **même fil**, puis affiche la réponse serveur. Le filtre
Incidents impose l'appel natif de l'unique outil sélectionné, empêche un second
appel après son résultat, puis affiche le rapport serveur avec ses citations.
Sans résultat natif vérifiable, il affiche un échec explicite. Il ne calcule ni
date, ni périmètre, ni diagnostic. Le modèle transmet
seulement le message courant et le jeton du résultat vérifié du tour précédent ;
le relais protège ces deux arguments sans retransmettre les anciens rapports
au modèle de dialogue. Le backend reste responsable du contrôle du jeton.
Les droits sont relus à chaque nouvel appel, quel que soit
l'historique local.

### 3. Créer les deux modèles dédiés

Dans **Espace de travail / Workspace → Modèles / Models**, créer deux modèles
personnalisés basés sur `gpt-oss-120b`. Conserver leurs identifiants exacts :

| Identifiant de modèle | Connexion outil seule | Filtre seul |
| --- | --- | --- |
| `albert-pdl-rie` | `dinum-pdl-agent` | `pdl_rie_dialogue` |
| `albert-observabilite-gptoss` | `obs-gptoss` | `incident_report_display` |

Dans les paramètres avancés, choisir **appels d'outils natifs / Native**, avec
`function_calling=native`, température `0` et maximum de sortie `4096` tokens.
Désactiver les outils intégrés sur ces modèles. Les identifiants d'outils stockés
dans les métadonnées sont respectivement `server:mcp:dinum-pdl-agent` et
`server:mcp:obs-gptoss` ; les `filterIds` sont ceux de la dernière colonne.
Conserver un seul filtre par modèle : aucune ambiguïté d'ordre n'est alors
introduite par d'autres filtres de reformulation ou d'affichage.

Utiliser ce message système sur les deux modèles :

```text
Pour chaque message utilisateur, appelle exactement une fois l’outil MCP sélectionné.
Transmets question comme copie intégrale du message courant. Utilise le dernier continuation_token de ce fil, vide pour une nouvelle conversation.
Le serveur interprète le dialogue et relit les droits et les preuves. Après l’appel, restitue sa réponse, ses preuves et ses limites sans les inventer.
Un refus reste un refus ; une clarification reste une question. N’expose aucun jeton dans la réponse. Ne relance pas automatiquement en cas de limitation.
```

Le client de référence limite les itérations à
`CHAT_RESPONSE_MAX_TOOL_CALL_ITERATIONS=1`, sans appels parallèles pour PDL.
Ces réglages bornent la boucle d'outils du client ; ils ne remplacent pas les
limites du serveur ou les étapes IA internes du diagnostic. La génération de
titre, tags, suggestions, la revue mémoire, la compaction automatique et le
repli sur un autre modèle étaient désactivés pour la recette. Sur un client
existant, faire appliquer les réglages utiles par son administrateur et
redémarrer son backend si sa configuration d'environnement le demande ; ne pas
remplacer l'ensemble de ses paramètres par ceux d'un client de laboratoire.

### 4. Contrôler le résultat dans une nouvelle conversation

Sélectionner explicitement le modèle PDL. Demander « Compte les refus PFS dans
demo-alpha. », puis répondre dans **le même fil** « Le 12 octobre 2026 de 08:00
à 09:00 UTC. ». Attendu sur la simulation : clarification sans preuve au premier
tour, puis deux refus PFS après précision, avec un seul appel métier par tour.
Ouvrir ensuite un nouveau fil avec le modèle Incidents et demander R1 le
28 septembre 2026 de 10:00 à 12:00, heure de Paris. Un interne doit obtenir les
cinq mesures, 120 logs et les citations du rapport ; un externe doit obtenir un
refus sans preuves. Attendre au moins 60 secondes **après la réponse** de la
synthèse Incidents précédente avant d'en demander une autre.
Les autres scénarios et leurs limites sont dans [Démonstrations](09-demonstrations.md).

Si le filtre PDL ne trouve pas l'outil natif, vérifier l'identifiant de modèle,
l'identifiant de connexion, les outils natifs et l'association du filtre. Une
réponse sans appel MCP vérifiable ne constitue pas une recette réussie. Si un
suivi perd le contexte, garder le même fil et vérifier que la dernière sortie
d'outil contient bien le jeton ; ne pas recopier celui d'un autre fil. Si le
rapport Incidents visible diffère du champ `answer` du résultat d'outil, vérifier
le filtre de sortie et retirer les autres filtres de ce modèle.

La campagne du 6 novembre a utilisé la configuration automatisée de
`deploy/ubuntu/qualification_openwebui.py`. Son parcours initial v6 a réussi
cinq cas sur six en sept soumissions ; le dernier refus externe a ensuite
réussi en une reprise ciblée, soit huit soumissions au total. Ce script crée des clients fictifs isolés ; ce n'est
pas un installateur destiné à transformer un client existant. La procédure
humaine ci-dessus décrit cette configuration attestée ; elle n'est pas présentée
comme une seconde installation manuelle déjà rejouée. Les chaînes de connexion
au SI réel et les performances AMD64 restent non qualifiées.

## Contrôler le passage au réel

Avant de construire et après toute modification, depuis la racine du code sur
Ubuntu, contrôler la configuration privée :

```bash
# Contrôle les fichiers et paramètres privés avant application ; ne démarre rien et n'appelle pas Albert.
sudo python3 deploy/ubuntu/manage.py validate "$RIE_INSTANCE"
```

Attendu : `Private instance configuration validated; no generation performed.`
Un échec indique notamment un marqueur externe non remplacé, une portée PDL
vide, une politique sans index, une URL contenant des identifiants, des ports
identiques ou une configuration LDAP incomplète. Corriger le fichier indiqué,
puis rejouer `validate`. Ce contrôle local ne contacte pas les sources et ne
prouve pas encore le sens métier des mappings.

Après `build`, `pull` et `up`, contrôler la disponibilité avec un identifiant
réel classé interne et habilité à l'administration par le LDAP :

```bash
# Sonde les sources, le LDAP et les mappings ; ready signifie disponibilité métier, sans validation de génération Albert.
sudo python3 deploy/ubuntu/manage.py check "$RIE_INSTANCE" --username LDAP_USERNAME
```

Attendu après initialisation métier : `"status": "ready"` et code de retour 0.
Un catalogue sans mapping actif ou un manifeste Incidents sans `"status": "ready"`
reste non initialisé. Ne pas ajouter ce statut pour masquer une source en panne :
faire préparer le manifeste et les mappings réels avec le responsable métier.
En cas de code 2, lire les dépendances et l'authentification indiquées ; la
disponibilité d'Albert est testée séparément.

Exécuter la recette fictive sur une instance distincte, puis faire relire les
accès et mappings réels. Documenter ce qui a été testé : transport, source,
droit de lecture, question métier et génération. Ne jamais annoncer un SI-RIE
raccordé à partir d'une capture ou d'une réponse du simulateur.
