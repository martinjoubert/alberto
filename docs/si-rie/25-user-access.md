# Connexion individuelle depuis Open WebUI, sans jeton à copier

L'utilisateur ouvre **son Open WebUI local**, puis autorise les sources PDL et
Incidents avec son compte LDAP. Open WebUI conserve ensuite les autorisations
et renouvelle automatiquement les accès courts. Aucun fichier de jeton MCP
n'est remis à l'utilisateur. NiceGUI conserve sa connexion LDAP habituelle.

Il s'agit d'une **autorisation OAuth des sources après connexion à Open WebUI**.
Ce n'est pas encore une connexion unique au fournisseur SSO du SI. Le SSO
d'OpenProject ne permet pas d'en déduire le protocole : le responsable identité
doit confirmer OIDC ou SAML, le fournisseur et les paramètres autorisés. Le
raccordement OIDC commun à Open WebUI et au PoC sera une évolution distincte.

## Ce que prépare le déployeur

Le serveur d'autorisation est intégré au moteur `pdl`, avec un état privé
commun aux deux moteurs. Aucun service supplémentaire n'est nécessaire.
`allow_webui_assertions` reste `false` : un poste ne peut pas choisir librement
le nom, le courriel ou le rôle de l'utilisateur. L'identité LDAP stable et les
groupes sont relus à chaque accès, y compris après renouvellement.

| Paramètre de `config/oauth.json`, spécifique au PoC | Valeur à faire confirmer |
| --- | --- |
| `issuer` | Origine publique du moteur PDL, sans chemin ni `/` final ; elle sert l'autorisation et la connexion LDAP |
| `resources.pdl` | URL complète du MCP PDL, terminée par `/mcp` |
| `resources.incidents` | URL complète du MCP Incidents, terminée par `/gptoss/assistant/mcp` |
| `redirect_uris` | URL de retour exacte de chaque connexion sur les postes Open WebUI ; conserver les identifiants `dinum-pdl-agent` et `obs-gptoss` |
| `access_seconds` | Durée de l'accès court : `600` secondes par défaut |
| `refresh_seconds` | Durée maximale de l'autorisation : `604800` secondes, soit sept jours ; le renouvellement ne repousse pas cette échéance |
| `login_seconds` | Session du navigateur sur la page LDAP : `28800` secondes, soit huit heures ; elle permet une confirmation sans ressaisir le mot de passe |

**Les URL des sources doivent être joignables à la fois par le navigateur et
par le backend Open WebUI.** Utiliser les DNS et certificats HTTPS validés du
SI, avec proxy et CA configurés sur les postes. Une URL accessible dans le
navigateur seulement ne suffit pas. Le reverse proxy doit transmettre aussi
`/authorize`, `/login`, `/token`, `/register`, `/revoke` et `/.well-known/`,
sans remplacer les en-têtes d'identité ou désactiver TLS. L'URL de retour doit
correspondre exactement à celle utilisée pour ouvrir Open WebUI, par exemple
`http://127.0.0.1:8080/oauth/clients/mcp:dinum-pdl-agent/callback`.

L'HTTP est accepté uniquement sur la boucle locale pour la sandbox et ses
tunnels. En Docker, `127.0.0.1` désigne l'espace réseau du conteneur : faire
vérifier le chemin réseau des deux côtés avant de configurer le client. Les
adresses de la sandbox ne sont pas des adresses de production.

**Sur Ubuntu, racine du paquet vérifié, droits `sudo` :**

```bash
# Copie le modèle public dans un fichier privé à adapter, sans activer OAuth.
sudo install -m 600 deploy/ubuntu/examples/oauth.json "$RIE_INSTANCE/config/oauth-candidate.json"
# Renseigne les URL réelles, les callbacks et les durées convenus avec le SI.
sudo nano "$RIE_INSTANCE/config/oauth-candidate.json"
# Vérifie et enregistre ce raccordement, sans démarrer ni contacter les sources.
sudo python3 deploy/ubuntu/manage.py configure-client-auth "$RIE_INSTANCE" --input "$RIE_INSTANCE/config/oauth-candidate.json"
# Vérifie la configuration privée avant tout redémarrage.
sudo python3 deploy/ubuntu/manage.py validate "$RIE_INSTANCE"
# Recharge les seuls services du PoC ; courte interruption prévue.
sudo python3 deploy/ubuntu/manage.py up "$RIE_INSTANCE"
# Vérifie les sources et les mappings avec un véritable administrateur LDAP du PoC.
sudo python3 deploy/ubuntu/manage.py check "$RIE_INSTANCE" --username "$RIE_ADMIN_LDAP"
```

**Attendu :** import accepté, validation réussie, puis `ready`. Un modèle
contenant encore `REPLACE.invalid` est refusé. Ni `validate` ni `check` ne
qualifient une conversation Albert ou une autorisation navigateur.

## Ce que prépare l'administrateur du poste

L'Open WebUI local de référence est **0.11.3**, avec stockage persistant,
`WEBUI_SECRET_KEY` stable et un seul worker. L'image dérivée fournie corrige
uniquement le renouvellement des deux connexions du PoC : sérialisation des
appels concurrents et conservation de l'autorisation lors d'une panne
temporaire. Les autres connexions utilisent le comportement d'Open WebUI.
Le script de construction refuse un code amont différent de la version
vérifiée. Une mise à jour d'Open WebUI nécessite une nouvelle qualification.

**Sur le poste utilisateur, racine des sources vérifiées :**

```bash
# Construit l'image locale à partir de la version Open WebUI figée et de l'adaptateur fourni.
docker build -t dinum/rie-openwebui:oauth -f deploy/openwebui/Dockerfile deploy/openwebui
```

**Attendu :** image construite, contrôle d'empreinte amont accepté. Si le SI
filtre les images via sa registry GitLab, publier/importer cette image et sa
base dans le circuit autorisé avant l'installation. Ne pas contourner ce filtre.

L'administrateur remplace l'image de **son client local** en conservant son
volume `/app/backend/data`, sa clé stable, ses réglages Albert/LDAP, ses
montages CA et son réseau. Sauvegarder ces éléments avant remplacement.
Le serveur Ubuntu ne reçoit aucun Open WebUI. Un volume ou une clé perdus
imposent de réautoriser les sources ; ils ne doivent pas être recréés à chaque
redémarrage. Le mot de passe du compte administrateur local Open WebUI n'est
pas transmis au PoC par le programme de configuration.

Pour un client déjà géré par Docker Compose, ajouter l'overlay fourni aux
fichiers de lancement existants, sans modifier le nom du projet ni les volumes.
L'exemple suivant suppose un seul fichier initial `CLIENT_COMPOSE` à remplacer
par son chemin réel ; reprendre tous les `-f` et le `--env-file` du poste :

```bash
# Recrée uniquement le client local avec l'image validée, sans démarrer ses dépendances.
docker compose -f CLIENT_COMPOSE -f deploy/openwebui/compose.override.yaml up -d --no-deps openwebui
```

**Attendu :** même client sur le même port, comptes et conversations conservés.
Garder l'overlay dans la commande habituelle de reprise du client. La variable
`RIE_OPENWEBUI_IMAGE` peut désigner l'image approuvée dans la registry GitLab.

Albert doit déjà être disponible dans le client, avec le modèle relais
`gpt-oss-120b`. Si la connexion fournisseur ajoute un préfixe au nom du modèle,
passer son identifiant complet à `--base-model`, tel qu'affiché dans Open WebUI.
Par exemple, le client de développement de cette sandbox expose
`recette-20261109.gpt-oss-120b` ; les clients neufs de qualification utilisent
le nom sans préfixe. Le programme refuse un modèle absent avant de modifier
les réglages. Préparer les deux URL MCP convenues, puis lancer :

```bash
# Vérifie les arguments publics sans contacter le client ni modifier sa configuration.
python3 deploy/openwebui/configure.py --webui-url http://127.0.0.1:8080 --pdl-url https://pdl-poc.REPLACE.invalid/mcp --incidents-url https://incidents-poc.REPLACE.invalid/gptoss/assistant/mcp --dry-run
```

Remplacer les deux exemples `REPLACE.invalid` **avant exécution** ; ils sont
volontairement refusés. Relancer ensuite la même commande **sans `--dry-run`**.
Le programme demande le compte et le mot de passe de l'administrateur de cet
Open WebUI. Il sauvegarde les anciens réglages dans un fichier privé hors des
sources, préserve les autres intégrations, inscrit les deux connexions OAuth,
installe leurs filtres et prépare les modèles `albert-pdl-rie` et
`albert-observabilite-gptoss`. Aucune clé MCP individuelle n'est demandée.

**Attendu :** `configured: true`, `personal_mcp_token_required: false` et
`user_authorization_pending: true`. Conserver le chemin `rollback_snapshot`.
Cette étape configure le client ; elle ne connecte personne au LDAP.

Le programme rend les deux modèles et leurs outils visibles **en lecture aux
utilisateurs authentifiés d'Open WebUI**. Ce partage porte sur les profils
du client, sans accorder de droit sur les données. Chaque utilisateur doit
encore autoriser ses sources ; LDAP, les ACL et les droits ITSM individuels
restent opposables. Vérifier la liste avec un compte local de rôle `user` :
les modèles privés d'un administrateur peuvent fonctionner lors d'un test
administrateur tout en restant invisibles aux utilisateurs ordinaires.

## Ce que fait l'utilisateur

1. Ouvrir son Open WebUI et se connecter normalement.
2. Ouvrir les connexions d'outils dans les paramètres et lancer
   **Autoriser OAuth** pour **SI-RIE — PDL**. Le client local a son propre
   administrateur ; sur une installation partagée, faire préparer ce parcours
   par cet administrateur. Le menu `+` des outils peut également proposer
   l'autorisation lors de la première sélection.
3. Sur **Connecter Open WebUI aux sources**, vérifier l'adresse du PoC, le
   client de destination et saisir son véritable identifiant et mot de passe
   LDAP. Cliquer sur **Autoriser la connexion**.
4. Autoriser **SI-RIE — INCIDENTS**. La page affiche le compte LDAP déjà
   connecté : le vérifier et confirmer, ou choisir **Utiliser un autre compte**.
5. Revenir dans Open WebUI et ouvrir les modèles PDL puis Incidents. Jouer les
   questions de l'[étape 11](24-deploy-walkthrough.md#11-jouer-les-deux-uc-et-leurs-acl-avec-albert).

Sur le client local ouvert à `http://127.0.0.1:8080`, les liens directs sont
[autoriser PDL](http://127.0.0.1:8080/oauth/clients/mcp:dinum-pdl-agent/authorize)
et [autoriser Incidents](http://127.0.0.1:8080/oauth/clients/mcp:obs-gptoss/authorize).
Se connecter d'abord à ce client. Pour un autre port ou nom, adapter l'origine
du lien et le callback déclaré ; ne pas transmettre ce lien à un tiers comme
une preuve d'accès, chacun doit autoriser ses propres sources.

**Attendu :** retour au bon Open WebUI, outils disponibles pour cette identité
et rapport étayé. La présence d'un outil par défaut dans un modèle ne déclenche
pas à elle seule la première autorisation interactive. Un test de découverte
des métadonnées ne prouve pas qu'un utilisateur est connecté aux sources.

Open WebUI chiffre ses autorisations dans son stockage local. Le PoC conserve
des empreintes des secrets OAuth, l'identité stable et les échéances ; aucun
mot de passe LDAP n'est conservé. Le renouvellement automatique ne modifie
jamais les groupes ou la politique. Au bout de sept jours, après révocation
ou suppression du compte, reconnecter la source si le compte reste habilité.
Une panne temporaire refuse la demande courante et conserve l'autorisation
afin de permettre la reprise ; elle ne doit pas produire un diagnostic inventé.

## Retirer un accès et diagnostiquer

**Sur Ubuntu, administrateur des accès du PoC :**

```bash
# Liste uniquement les identités, ressources, échéances et identifiants de révocation, sans secrets.
sudo python3 deploy/ubuntu/manage.py list-client-sessions "$RIE_INSTANCE" --username "$RIE_ADMIN_LDAP"
# Demande l'identifiant non secret de l'autorisation examinée dans la liste précédente.
read -r -p 'Identifiant de session à révoquer : ' RIE_CLIENT_SESSION
# Révoque seulement cette autorisation ; sa valeur doit être l'id de session, jamais un jeton.
sudo python3 deploy/ubuntu/manage.py revoke-client-session "$RIE_INSTANCE" --username "$RIE_ADMIN_LDAP" --id "$RIE_CLIENT_SESSION"
```

**Attendu :** la session choisie est révoquée ; son prochain accès est refusé.
Les autres utilisateurs restent connectés. Les sauvegardes à froid du volume
runtime conservent cet état et la révocation. Les anciens jetons personnels
`issue-token` restent compatibles pour scripts/opérateurs ; leur révocation
est distincte. Ne pas en créer pour remplacer le parcours OAuth utilisateur.

| Symptôme | Vérification utile |
| --- | --- |
| Pas de page LDAP | Accès au MCP et aux métadonnées depuis le backend client ; callback exact et inscription OAuth |
| Identifiants refusés | Véritable compte LDAP, TLS/CA du PoC, groupes interne/externe et identité stable ; aucun mot de passe dans les logs |
| Mauvais compte affiché | Choisir **Utiliser un autre compte** ; ne pas partager une session navigateur entre utilisateurs |
| Connexion refusée après retour | Même origine Open WebUI que le callback, cookies, clé locale stable et horloges ; relancer l'autorisation |
| Accès source refusé après connexion | Politique `blocked`, droits et identité LDAP actuels, droits ITSM propres au titulaire ; l'autorisation OAuth ne les élargit pas |
| Échec pendant une panne | Rétablir la dépendance, relancer la demande ; si l'autorisation est expirée/révoquée, la recréer depuis Open WebUI |

La recette doit distinguer les tests automatisés, les parcours navigateur et
les conversations Albert. Le raccordement au SSO et aux certificats, réseaux,
postes et annuaires réels du SI reste à réceptionner dans la
[matrice SI-RIE](14-si-rie-acceptance.md).

## Qualification du 16 novembre 2026

La recette utilise Ubuntu 24.04 ARM64, les vrais services du SI fictif de la
sandbox et deux Open WebUI locaux isolés. Le runtime testé est
`runtime-03763e37a65820d831cc` ; l'image client dérivée est
`sha256:ac6dfa6d4e27…`, construite à partir de l'image amont figée.

| Contrôle | Résultat mesuré |
| --- | --- |
| Suites sans réseau du dépôt | 1 362 réussites, un test Linux ignoré sur Mac : source/ACL/client/opérateur 593, PDL 216, Incidents 553 |
| Autorisations natives par navigateur | Quatre réussites : deux sources pour Alice et Éric ; première saisie LDAP, puis confirmation explicite de l'identité |
| Conversations natives avec Albert | PDL : six documents, puis deux TACACS dans le même fil ; Incidents : diagnostic validé avec précédents ITSM ; Éric : refus sans preuves interdites |
| Fournisseur réellement appelé | Planification PDL et rédaction Incidents : `gemma-4-31b-it` ; relecture Incidents : `gpt-oss-120b`, avec usage retourné ; relais de dialogue également vérifié |
| Renouvellement et concurrence | Accès effectivement expirés renouvelés par Open WebUI ; cinq lectures natives concurrentes réussies |
| Panne et reprise | Autorité de qualification arrêtée : accès courant refusé et sessions conservées ; reprise sans ressaisie LDAP |
| Redémarrage du client | Même volume et clé, autorisations réutilisées après redémarrage |
| Révocation | Retrait du seul accès PDL d'Alice ; son accès Incidents et ceux d'Éric restent actifs |
| Sauvegarde/restauration neuve | 45 fichiers du volume identiques avant démarrage, base OAuth incluse ; révocation et accès restant vérifiés depuis Open WebUI après reprise |
| Configuration répétée | Inscriptions OAuth et accès natifs d'Éric conservés, sans nouvelle autorisation |

Les preuves de navigateur sont produites par
`deploy/ubuntu/tests/browser_oauth.py`, avec comptes privés sur l'entrée standard,
reçus expurgés et captures Allure. Les contrôles déterministes sont exécutés
par `ci/runners/run_ubuntu_tests.py`. Les preuves locales de cette campagne
restent sous `.runtime/sandbox-si-20261116/oauth-browser-release-c4/` et les
résultats des suites sous `.runtime/ci/oauth-release-20261116-c3/`.
Le portail NiceGUI présente le rapport importé `user-access-20261116`.

### Correctif de visibilité pour les utilisateurs ordinaires

La campagne initiale utilisait les administrateurs locaux de deux clients
isolés. Elle vérifiait les identités LDAP internes/externes côté sources, mais
ne couvrait pas la visibilité des profils pour un compte Open WebUI de rôle
`user`. La recette manuelle a révélé que les profils et outils restaient privés
à l'administrateur qui les avait installés.

Le programme de configuration partage désormais les deux profils et leurs
outils **en lecture avec les comptes authentifiés du client**. Aucun droit
d'administration du client ou d'accès aux sources n'est ajouté. La qualification
navigateur `browser_oauth.py` exige désormais un compte de rôle `user`, pour
qu'un administrateur ne puisse pas masquer à nouveau ce défaut lors du test.

La correction est vérifiée sur le compte réel de démonstration d'Alice, qui
reste `user` dans Open WebUI : les deux profils et outils deviennent visibles,
les deux autorisations personnelles restent actives et l'administration des
connexions reste refusée. Un échange PDL par navigateur avec Albert retrouve
les deux refus attendus et conserve la réponse vérifiée du serveur. Le chat
technique créé par ce contrôle a été retiré ; les conversations de l'utilisateur
sont conservées. Les inscriptions OAuth et les autres connexions restent
identiques. Un nouveau diagnostic Incidents sous ce rôle n'a pas été demandé
dans ce contrôle ; il reste à jouer pendant la recette métier.

Les 1 364 non-régressions passent, avec un test Linux ignoré sur Mac. Deux
régressions empêchent la qualification navigateur d'accepter un compte
administrateur ou en attente. Les preuves du correctif sont sous
`.runtime/sandbox-si-20261116/alice-model-visibility/` et
`.runtime/ci/model-visibility-20261116/`, campagne Allure
`model-visibility-20261116`.

Les premiers essais ont révélé un refus du formulaire Chrome lié à
`Referrer-Policy: no-referrer`, corrigé sans retirer le contrôle d'origine/CSRF,
et une particularité HTTP 401 des fonctions absentes d'Open WebUI, couverte par
une revérification du rôle administrateur. La signature native des appels
positionnels et les autres fournisseurs sont protégés par régression.
Un collecteur provisoire lançant un second processus applicatif a provoqué
un OOM sur le client isolé limité à 1,2 Go ; il a été abandonné au profit des
routes HTTP natives. Cet échec de collecteur est conservé dans les traces et
ne fait pas partie des huit parcours navigateur réussis. Le client a été
redémarré et les six contrôles opérationnels ont été rejoués par HTTP.

La campagne ne requalifie pas deux heures d'endurance, Ubuntu 26.04/AMD64,
le SSO réel ou le réseau du SI. Les preuves historiques conservent leurs
versions et limites. La recette métier sur l'instance principale reste à
effectuer par l'utilisateur après activation de ses propres autorisations.
