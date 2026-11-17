# Glossaire du déploiement et de l'exploitation

Ce glossaire définit le vocabulaire du parcours Ubuntu pour les deux cas
d'usage PDL et Incidents et leurs ACL. Le [README](README.md) donne l'ordre de
lecture, le [manuel](01-installation.md) les commandes et la
[configuration](02-configuration.md) les champs à renseigner.

## Fiche de raccordement

Document de paramètres fourni au déployeur par les responsables des services
du SI. Il décrit les adresses et ports, certificats, comptes lecteurs, groupes,
index, formats et autres contrats de données nécessaires au raccordement.
Le déployeur s'en sert pour renseigner `.env` et les fichiers de configuration
de son instance du PoC. Par exemple, l'URL OpenSearch fournie renseigne
`OPENSEARCH_URL`.

La fiche référence les fichiers ou canaux privés de remise des mots de passe
et jetons. Sa version publique reste sans secret. Dans la sandbox, les
[paramètres de l'exercice](22-external-sandbox.md#fiche-de-raccordement-de-cet-exercice)
sont aussi fournis dans `$HOME/si-connection/connection.json`, et les accès
privés sous `$HOME/si-connection/private`. Dans le SI réel, les responsables
des services fournissent ou confirment ces informations.

## Préparer et déployer

| Terme | Sens dans ce projet |
| --- | --- |
| SI | Système d'information : services, données, identités et réseau auxquels le PoC doit se raccorder. Le SI-RIE est le SI cible ; la sandbox en simule une partie. |
| PoC | Preuve de concept : réalisation permettant d'évaluer les cas d'usage et leurs limites avant une éventuelle mise en production. |
| UC | Cas d'usage. Les deux UC livrés sont la recherche PDL, ou puits de logs, et le diagnostic Incidents. |
| Sandbox / SI simulé | Environnement isolé d'essai. Ici, de vrais logiciels et protocoles servent des données et identités fictives. |
| VM Ubuntu | Machine virtuelle qui représente le serveur de déploiement. Une VM peut héberger plusieurs services et plusieurs instances du PoC. |
| Instance du PoC | Ensemble identifié de configuration, secrets, volumes et services du PoC. Son dossier est désigné par `RIE_INSTANCE` ; il reste distinct du code et des services du SI. |
| `config/pdl.yaml` | Configuration propre au moteur PDL du PoC, issue du modèle `pdl.json` : accès lecteur à OpenSearch, index, limites et Albert. Le serveur OpenSearch possède sa propre configuration `opensearch.yml`, gérée séparément par son administrateur. |
| `config/access.json` | Configuration propre au composant d’identité et d’accès commun aux deux UC du PoC. Son objet `ldap` décrit le raccordement à l’annuaire existant ; ce fichier ne configure pas le serveur LDAP et ne crée aucun compte ni groupe. |
| Mode `demo` | L'installation du PoC prépare aussi ses propres services sources et leur corpus fictif. |
| Mode `external` | Le PoC se raccorde aux services déjà disponibles et configurés par leurs responsables, dans le SI réel ou dans une sandbox. |
| Paquet d'installation / delivery | Archive des sources du PoC, modèles et documentation, fournie avec son fichier SHA-256. La sandbox la place dans `$HOME/deploy`. Les images sont construites ensuite ; les paramètres SI et secrets sont fournis séparément. |
| Manifeste / SHA-256 | Le manifeste décrit les fichiers attendus et leurs empreintes. Le contrôle SHA-256 détecte une modification des octets ; la confiance dans le canal de remise reste à établir séparément. |
| Commit / `source.dirty` | Le commit identifie la version Git de base du préparateur. `dirty: true` signale des modifications non commitées ou fichiers non suivis à la fabrication ; `false` un dépôt propre ; `null` une information indisponible. Le manifeste identifie les fichiers réellement livrés. |
| Intégrité / authenticité | L'intégrité compare le contenu à une référence. L'authenticité établit son origine. `not_verified_by_checksum` indique que le vérificateur compare les empreintes sans vérifier de signature d'éditeur ; voir les [résultats de vérification](01-installation.md#interpréter-le-résultat-de-vérification-du-paquet). |
| Image / conteneur Docker | L'image contient le logiciel et ses dépendances. Le conteneur est une exécution de cette image avec sa configuration et ses volumes. |
| Volume Docker | Stockage persistant d'un service. Il conserve les données au-delà de l'arrêt ou de la recréation d'un conteneur. |
| Docker Compose | Outil qui décrit et gère ensemble les services d'un même projet Docker. Les sources de cette sandbox et le PoC appartiennent à deux projets distincts. |
| Registry | Service de distribution d'images de conteneurs. La registry GitLab filtrée du SI-RIE fait partie des raccordements à vérifier dans le SI cible. |

## Se connecter et contrôler les droits

| Terme | Sens dans ce projet |
| --- | --- |
| Source / compte lecteur | Une source fournit logs, mesures, documents ou tickets. Un compte lecteur possède les droits nécessaires pour les consulter ; la politique du PoC contrôle aussi les lectures autorisées à chaque utilisateur. |
| LDAP / groupe LDAP | LDAP est le protocole d'annuaire utilisé pour retrouver les identités et leurs groupes. Ici, les groupes distinguent internes, externes et administrateurs des accès. |
| `RIE_ADMIN_LDAP` | Identifiant d'un compte du LDAP cible, interne et membre direct d'un groupe d'administration des accès du PoC déclaré dans `ldap.administrator_group_dns`. En production, utiliser un véritable compte habilité ; Alice est fictive et propre à la sandbox. Ce rôle est distinct de l'administrateur technique LDAP, du lecteur d'annuaire et du compte Unix avec `sudo`. |
| DN / `entryUUID` | Le DN est le nom complet d'une entrée LDAP. `entryUUID` identifie durablement cette entrée dans l'annuaire de l'exercice ; il sert à lier les accès à l'identité stable. |
| ACL / politique d'accès | Règles qui autorisent ou refusent les lectures. Le PoC protège les index ainsi que les ressources de mesures et tickets. La présence dans un groupe LDAP ne crée pas à elle seule une règle ACL. |
| Credentials | Identifiants et secrets d'accès : selon le service, compte et mot de passe, clé API ou jeton. Ils sont remis et conservés dans des fichiers ou canaux privés. |
| Jeton / Bearer | Secret présenté lors d'un appel pour authentifier l'accès. Le jeton du PoC est individuel, avec expiration et révocation ; les jetons des autres services ont leur propre usage. |
| OAuth / renouvellement | Autorisation individuelle d'un client à lire une source. Ici, l'utilisateur s'identifie au LDAP sur la page du PoC ; Open WebUI reçoit et renouvelle les accès courts sans copier de jeton. La durée totale reste bornée et la révocation reste opposable. |
| SSO / OIDC / SAML | Le SSO permet une connexion commune à plusieurs applications. OIDC et SAML sont deux protocoles possibles. Le SSO d'une application existante ne prouve pas qu'OIDC est disponible pour le PoC ; le fournisseur du SI doit être identifié. |
| TLS / LDAPS / CA | TLS protège et authentifie la connexion au serveur ; LDAPS l'applique à LDAP. La CA est l'autorité de certification dont le certificat permet de vérifier celui du serveur. Le fichier de confiance remis contient des certificats publics. |
| Tunnel SSH | Redirection chiffrée entre le poste et le serveur. Elle permet au client local d'atteindre les ports du PoC liés à la boucle locale du serveur. |

## Comprendre les données et les réponses

| Terme | Sens dans ce projet |
| --- | --- |
| Index / schéma OpenSearch | L'index regroupe des documents ; son schéma décrit les champs et leurs types. Un nom d'index seul ne décrit pas le sens métier des données. |
| Mapping PDL | Correspondance entre concepts métier et champs physiques, avec leurs types, valeurs et relations. Son approbation doit porter sur le sens des champs et leurs preuves. |
| Catalogue approuvé | Ensemble des mappings actifs autorisés pour les recherches. La disponibilité d'un index et son droit d'accès ne suffisent pas à approuver son mapping. |
| Corpus / fixture | Le corpus est l'ensemble des données étudiées. Une fixture est un jeu préparé pour reproduire un scénario de test ; celles de la sandbox sont fictives. |
| Snapshot / précédents ITSM | Le snapshot est l'export daté des alertes et traitements courants. Les précédents sont des tickets historiques recherchés séparément, ici par l'API native ITSM-NG. |
| MCP | Protocole utilisé par le client pour appeler les outils publiés par les moteurs. Les outils métier conservent les contrôles d'identité, d'arguments et de droits. |
| Backend / Open WebUI / NiceGUI | Les backends Python traitent les demandes sur Ubuntu. Open WebUI porte la conversation sur le poste utilisateur. NiceGUI est le portail facultatif de démonstration, documentation et administration. |
| Génération Albert | Appel réel au fournisseur IA pour interpréter, planifier, rédiger ou relire selon le parcours. Les calculs déterministes et contrôles techniques restent distincts de cette génération. |

## Vérifier et réceptionner

| Terme | Sens dans ce projet |
| --- | --- |
| Sonde de santé / `healthy` | Contrôle du fonctionnement d'un processus ou service. Son succès ne valide pas à lui seul le corpus, les droits ou une réponse IA. |
| Disponibilité métier / `ready` | Résultat de contrôles sur les dépendances, l'identité et le catalogue nécessaires au PoC. La génération Albert conserve sa recette distincte. |
| Recette / qualification | La recette vérifie les scénarios et leurs attendus. La qualification rassemble les résultats, versions, environnements, preuves et limites ; la réception du SI réel reste une validation propre à ce SI. |
| CI / Allure | La CI exécute les contrôles automatisés lors de la validation du code. Allure présente leurs résultats et pièces de diagnostic ; NiceGUI donne accès aux publications importées. |
