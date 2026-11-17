# Remise Ubuntu et CI GitLab — 13 novembre 2026

> Preuve historique de la qualification du 13 novembre 2026. Ses résultats gardent leur version et leur périmètre ; consulter le [bilan de qualification](08-qualification.md) pour les suites et le [guide principal](../README.md) pour déployer.

Cette campagne vérifie la livraison des UC PDL et Incidents avec leurs ACL.
Elle complète les [96 échanges Albert du 12 novembre](19-openwebui-corrections.md)
par une installation neuve, des essais de reprise et une nouvelle compatibilité
Ubuntu 26.04. Les résultats portent sur le laboratoire ARM64 du MacBook.
La [réception des sources et de la machine du SI-RIE](14-si-rie-acceptance.md)
reste à effectuer.

**Bilan :** installation, reprise, pannes, CI et compatibilité réussissent.
L'endurance de **7 200,087 secondes** termine avec **8 459 demandes, zéro erreur
inattendue, 810 refus ACL attendus et zéro violation de cadence**. Le corpus
complet et les données ITSM sont préservés. L'import de l'artifact CI privé,
la couverture complète du journal d'accès ITSM et la réception du SI réel
restent distinctement non qualifiés.

## Versions et séparation des essais

| Élément | Version ou portée |
| --- | --- |
| Moteurs PDL, Incidents et commandes opérateur | `dinum/rie-delivery:runtime-5afa381bc374efc4c4c7` |
| Portail NiceGUI facultatif | `dinum/rie-portal:13f1915c0329f4994bf2` |
| Image facultative de charge | `dinum/rie-qualification:d7f64e439f44ffecb17e` |
| Source du candidat C1 | Commit `7f2a0f6145c61a75da41940d78a70a6abe9ddad5`, 265 fichiers vérifiés |
| Qualification principale | VM Ubuntu 24.04.5 ARM64 neuve, 8 vCPU, 16 Gio RAM, disque 80 Gio ; aucun partage des dossiers du Mac |
| Compatibilité | Nouvelle instance sur Ubuntu 26.04.1 ARM64 ; Docker déjà installé sur cette VM |
| Sources | OpenSearch, VictoriaMetrics, LDAP réels et ITSM-NG natif 1.6.17 ; données, comptes et secrets de laboratoire indépendants de l'ancienne démo |
| Client conversationnel | Open WebUI sur le Mac, campagne Albert du 12 novembre distincte de ces essais d'exploitation |

L'installation neuve 24.04 commence avec Docker absent. Elle utilise le paquet
source du commit `9e1bc97`, puis les outils de qualification du candidat C1 dans
une extraction distincte. Les changements intermédiaires concernent les runners
CI, leurs fixtures et le pilote de pannes fournisseur. Les contextes des images
métier et de charge gardent les empreintes ci-dessus. Le pilote navigateur
accepte ensuite des ports personnalisés pour contrôler la nouvelle instance 26.04.
Ces évolutions de qualification ne constituent pas une nouvelle image métier.

## CI GitLab effectivement exécutée

La [pipeline 3065317225](https://gitlab.com/clients_neovelion_tech/dinum_rie_demo_albert_api/-/pipelines/3065317225)
du candidat C1 termine ses **sept jobs avec succès**. Une première pipeline verte,
[3065227003](https://gitlab.com/clients_neovelion_tech/dinum_rie_demo_albert_api/-/pipelines/3065227003),
la précède. Les premiers essais en échec restent conservés : ancien tag sans
runner disponible, absence de Git dans les images Python slim, assertion de
dry-run confondant `/builds` et la commande `build`, fixture de transport
asynchrone Incidents non alignée avec le lecteur actuel.

Les corrections installent Git seulement dans les jobs qui en ont besoin,
utilisent les runners hébergés GitLab.com pour la validation et conservent les
runners dédiés pour la construction protégée. Les tests n'appellent pas Albert.
Les suites se recouvrent : leurs compteurs ne sont pas additionnés comme des
scénarios indépendants. La branche de travail ne déclenche aucune publication
d'image protégée dans une registry SI-RIE.

Le gate Ubuntu C1 sur Linux exécute 513 contrôles source/opérateur, 216 PDL et
553 Incidents, soit **1 282 réussites sans saut**. Le rejeu local final sur le
Mac utilise les mêmes suites : **1 281 réussites et un contrôle Linux ignoré**.
Les 28 contrôles documentaires et d'extraction sont inclus dans cette couverture.

Le job `report:ubuntu` produit réellement Allure. Son artifact est réservé
aux Maintainers. L'autorisation de lire l'identifiant GitLab enregistré sur le
Mac a été obtenue, mais le gestionnaire n'a retourné aucun credential HTTPS
pour GitLab. **L'import de cet artifact CI dans NiceGUI reste non qualifié**
tant qu'un téléchargement authentifié n'est pas disponible. Les nouveaux
rapports locaux portent leur propre provenance ; ils ne remplacent pas
l'artifact CI. Voir [le parcours d'import](13-test-reports.md).

## Installation et exploitation sur Ubuntu 24.04

| Essai réalisé | Résultat observé |
| --- | --- |
| Paquet seul dans une VM neuve | Vérification des empreintes et de la sélection ; aucun accès à `.migration/`, aux environnements Python du Mac ou à un dossier partagé |
| Installation scriptée | Dry-run sans création d'instance, puis installation avec option explicite de prérequis système et portail facultatif ; disponibilité métier et 12 contrôles techniques réussis |
| Validation avant reprise | Configuration JSON invalide, port invalide et image absente refusés avant tout arrêt, nouveau conteneur ou volume ; services et fichiers préservés |
| Opérations courantes | Lecture des journaux et occupation disque sans mutation ; fichier de mapping root `0600` importé avec les droits opérateur, sans approbation implicite |
| Contrôles répétés | Cinq contrôles de disponibilité successifs : registre des jetons identique par empreinte ; les jetons utilisateurs ne sont pas supprimés |
| Sauvegarde froide | Cinq images et quatre volumes, configuration, CA et liaisons ITSM incluses dans le périmètre applicatif privé |
| Archive corrompue | Restauration refusée avant toute création de destination ou de volume ; l'archive originale reste intacte |
| Restauration neuve | Quatre nouveaux volumes ; égalité de tous les fichiers, octets, modes, UID et GID avant démarrage ; disponibilité, 12 contrôles et état du jeton personnel préservés |
| Changements de sources | Huit contrôles de fraîcheur : nouvelles mesures et nouveau snapshot de ticket visibles dans de nouveaux rapports ; données initiales rétablies, ancien rapport inchangé |
| Redémarrage Docker réel | Configuration, catalogue, mappings, politique et jetons préservés ; corpus et données ITSM comparés, puis disponibilité et 12 contrôles réussis |
| Reboot Ubuntu réel | Identifiant de démarrage du noyau différent ; mêmes comparaisons d'intégrité et 12 contrôles réussis après reprise |

Le corpus de référence comprend 13 documents PDL, 960 logs Incidents, 10 fiches
et 20 séries représentant 4 200 points. Les comparaisons de reprise portent sur
leurs identifiants, contenus et valeurs, pas seulement sur un compteur de documents.
La durée totale de reprise n'a pas été mesurée comme un RTO contractuel.

Le serveur ITSM natif du laboratoire est un supplément explicitement transféré,
avec quatre tickets et deux lecteurs fictifs. La restauration applicative conserve
ses URL, sa CA et les liaisons individuelles ; elle ne sauvegarde pas la base
du service ITSM externe. Le laboratoire redémarre explicitement cette API et sa
base, et raccorde l'API au réseau de l'instance restaurée. Dans le SI, la
sauvegarde et la disponibilité d'ITSM relèvent de son exploitant.

## Pannes contrôlées et refus

| Famille d'essais | Résultat observé |
| --- | --- |
| Arrêt de PDL puis d'Incidents | Service arrêté inaccessible, autre UC encore disponible selon ses propres dépendances ; reprise après rétablissement |
| LDAP indisponible | Refus fermé pour les deux UC ; aucune preuve protégée publiée ; retour à la disponibilité après reprise |
| OpenSearch indisponible | Résultat partiel explicite ; anciens logs et documents non réutilisés comme observations fraîches |
| VictoriaMetrics indisponible | Résultat partiel explicite ; anciennes mesures absentes, autres sources conservées |
| API ITSM puis base ITSM indisponibles | Deux pannes et huit critères vérifiés par panne : historique indisponible explicite, pas d'anciens précédents ni solution, métriques encore disponibles, refus externe sans preuve, disponibilité en échec puis reprise |
| Identité native ITSM | Treize contrôles avant désactivation, pendant révocation et après rétablissement : droits d'entité, suivis privés et solutions respectés |
| Fournisseur IA contrôlé | Huit cas : connexion refusée, erreurs 500 et 429, JSON invalide et délais effectifs de 60 s dans l'interprétation ou le planner ; échec explicite sans ancienne preuve ni ancien identifiant de rapport |
| Admission IA Incidents | Premier diagnostic sur fournisseur fictif accepté, demande suivante limitée pendant la fenêtre de 60 s, sans nouvel appel de génération ; aucune désactivation silencieuse |

Le pilote fournisseur a d'abord échoué parce qu'il attendait un délai de 30 s
alors que le contrat courant est de 60 s, et parce que son propre planner ne
reconnaissait pas la requête sérialisée actuelle. Le reçu d'échec est conservé.
Le pilote corrigé est rejoué et réussit ; les moteurs, les délais métier et la
limite d'admission sont inchangés. Ces injections utilisent un fournisseur
local fictif, sans prétendre provoquer une panne réelle du service Albert.

## Endurance : protocole et résultats mesurés

L'instance de charge est distincte du corpus de référence. Elle contient
**100 000 événements PDL et 100 000 logs Incidents**, générés de façon
reproductible, puis vérifiés intégralement sans réparation. Les vingt séries
de mesures et dix fiches restent de petite taille. Dix comptes LDAP distincts
sont liés à dix nouvelles identités ITSM natives ; deux profils sont externes.
Le compte opérateur qui contrôle la disponibilité possède sa propre liaison ITSM.

Le test court préalable dure 60,075 s : 73 demandes et aucune erreur inattendue.
Il reste identifié comme un test court. La campagne d'endurance applique
30 minutes à un utilisateur, 45 minutes à cinq, puis 45 minutes à dix, avec
au plus un début de demande par identité toutes les cinq secondes, y compris
au changement de palier. Chaque client attend sa réponse avant sa prochaine
demande ; il n'existe pas de file de demandes client illimitée.

La campagne est terminée : **13 critères finaux réussissent**, dont la comparaison
intégrale des 100 000 événements PDL, 100 000 logs, dix fiches et 4 200 points,
sans réensemencement ni réparation. Les tickets, solutions, suivis et identités
ITSM de référence sont inchangés ; les dix identités natives de charge sont
toujours actives. Aucun test de panne, redémarrage ou changement de configuration
n'est effectué pendant les deux heures. Les refus restent sans preuve protégée.

| Utilisateurs | Durée mesurée | Demandes | Refus attendus | Débit total, demandes/s |
| --- | --- | --- | --- | --- |
| 1 | 1 800,009 s | 360 | 0 | 0,2000 |
| 5 | 2 700,006 s | 2 700 | 270 | 1,0000 |
| 10 | 2 700,004 s | 5 399 | 540 | 1,9996 |

Les latences suivantes sont en millisecondes ; les trois valeurs désignent
**p50 / p95 / p99**. Les refus sont séparés des diagnostics autorisés.

| Utilisateurs | PDL autorisé | Incidents interne autorisé | Refus Incidents externe |
| --- | --- | --- | --- |
| 1 | 27.267 / 30.393 / 32.041 | 540.532 / 590.782 / 612.943 | Aucun externe dans ce palier |
| 5 | 32.128 / 45.365 / 55.513 | 749.098 / 1253.466 / 1269.816 | 55.792 / 78.762 / 83.129 |
| 10 | 39.805 / 68.230 / 80.978 | 1196.367 / 1553.960 / 1626.621 | 73.105 / 85.031 / 96.554 |

Les 8 449 intervalles entre débuts de demandes sont vérifiés avec l'horloge
monotone native, y compris aux transitions : minimum **5,000011 s**, zéro
violation. Le test fermé n'ajoute pas de demande de rattrapage : une demande
de moins que le maximum théorique est mesurée au dernier palier. Les durées
monotones font foi ; les horodatages UTC servent au recoupement des traces.

PDL utilise le fournisseur fictif de qualification ; Incidents lit ses sources
sans demander de synthèse. L'historique est lu dans le vrai ITSM-NG de laboratoire.
Les refus externes sont contrôlés sans fuite de preuve. Les temps sont séparés
entre PDL, Incidents interne et refus attendus : une latence mélangée ne mesure
pas le seul diagnostic autorisé. Les ressources des cinq services de charge,
de l'API et de la base ITSM sont échantillonnées, ainsi que CPU/mémoire de la VM,
disque, croissance des fichiers et files OpenSearch. Aucune pile d'observabilité
n'est installée pour cette campagne.

Le collecteur principal fournit **421 échantillons sans erreur** ; sa pause
cible de 15 s donne un intervalle maximal mesuré de 17,241 s, collecte comprise.
Le collecteur complémentaire fournit 422 instantanés, dont ceux d'encadrement.
Les valeurs par palier ci-dessous ne retiennent que la fenêtre du palier.
Ce sont des maxima échantillonnés, pas des maxima continus.

| Utilisateurs | CPU VM p95 / maximum, sur les huit vCPU | RAM VM utilisée maximale |
| --- | --- | --- |
| 1 | 2,317 % / 2,745 % | 2,25 Gio |
| 5 | 6,825 % / 7,189 % | 2,29 Gio |
| 10 | 15,872 % / 24,489 % | 2,43 Gio |

Le maximum mémoire Docker observé est d'environ 139 Mio pour Incidents,
88 Mio pour PDL, 1,17 Gio pour OpenSearch, 30 Mio pour VictoriaMetrics,
7 Mio pour LDAP, 113 Mio pour l'API ITSM et 211 Mio pour sa base. Le CPU Docker
d'Incidents atteint ponctuellement 309,07 % : **100 % représente un seul CPU
logique**, avec une fenêtre différente du CPU moyen de la VM. Aucune hausse
de compteur de redémarrage ni OOM n'est détectée sur les sept services.

Les files OpenSearch `get`, `search` et `write` restent à zéro aux instants
échantillonnés et leurs compteurs de rejet n'augmentent pas. Les autres files
serveur ne sont pas instrumentées. Le maximum de demandes client simultanées,
le modèle fermé et les mesures détaillées restent dans les reçus.

Le volume applicatif passe de 5 424 361 à 414 855 040 octets. Dans la fenêtre
de campagne, 4 230 nouveaux fichiers de rapport PDL et 3 419 fichiers Incidents
représentent environ **346 Mio** ; le reste comprend notamment audits, états et
trace de charge. Aucun rapport n'est supprimé automatiquement. Le minimum de
disque libre reste d'environ 66,9 Gio, sur un disque qui contient aussi les
images, sauvegardes et volumes de référence arrêtés. Voir
[l'archivage et le contrôle d'occupation](07-operations.md).

### Journaux : couverture acquise et limite conservée

Les journaux Docker conservés ne contiennent aucun marqueur d'erreur critique,
de traceback, d'exception non gérée, d'OOM ou d'avertissement pendant les périodes
examinées. Pour les moteurs PDL et Incidents, une capture intermédiaire et la
capture finale se recouvrent : la fenêtre des requêtes est couverte.

Le serveur **ITSM supplémentaire du laboratoire** utilise une rotation de
deux fichiers de 5 Mio, contre trois de 10 Mio pour les services Ubuntu livrés.
Sa capture initiale finit à 17:15:55 UTC ; son journal encore disponible en fin
de campagne commence à 17:46:46 UTC. Environ **31 minutes de journal d'accès
ITSM ne sont plus disponibles**. Le contrôle de continuité l'a détecté et son
échec est conservé. Les 8 459 assertions de réponse, les mesures de ressources
et les comparaisons de données réussissent indépendamment ; ils ne recréent
pas ces lignes perdues. La couverture complète de ce journal reste non qualifiée.

Pour une nouvelle campagne, capturer les journaux en continu **avant la charge**,
en conservant la rotation activée. Sur le serveur Ubuntu, dans un terminal
opérateur, choisir le nom depuis `manage.py status`, un dossier privé et un
fichier neuf ; le dossier appartient à l'opérateur qui effectue la redirection :

```bash
export RIE_LOG_DIR="$HOME/rie-journaux/campagne-a-nommer"
install -d -m 700 "$RIE_LOG_DIR"
set -o noclobber
sudo docker logs --follow --timestamps NOM_DU_CONTENEUR > "$RIE_LOG_DIR/service.log" 2>&1
```

Maintenir une collecte par service, puis l'interrompre avec `Ctrl+C` après la
campagne. Les journaux bruts restent privés ; le paquet de preuves remet les
compteurs, empreintes et limites, sans corps de requête ni secret sélectionné.

## Compatibilité Ubuntu 26.04 et NiceGUI

Sur une nouvelle instance Ubuntu 26.04.1 ARM64, l'installation automatisée et
les douze contrôles techniques réussissent. Les **149 contrôles ACL** couvrent
cycle de vie d'identité et de jeton, catégories et accès aux sources des deux UC.
L'historique utilise une nouvelle API native ITSM-NG 1.6.17 avec ses propres secrets.

Les **22 contrôles Chrome** réussissent sur les services réellement déployés :
trois démos, catalogue et identité MCP, précédents et solution, deux confiances,
sessions isolées, refus externe et non classé, lecteur non administrateur,
restriction/rétablissement des sources dans la même session, diagrammes,
liens documentaires et exports. Les ports publiés sont personnalisés pour éviter l'ancienne
instance de recette. Ce parcours lit les preuves sans appeler Albert ; il ne
réexécute pas les 96 conversations du 12 novembre sur Ubuntu 26.04.

Les captures et diagnostics datés sont conservés avec cette campagne, séparément
des captures historiques. Les traces complètes et entrées privées ne font pas
partie des paquets portables. La VM 26.04 est conservée arrêtée, avec ses volumes.

## Limites de la remise

- Les données de charge sont largement des répétitions d'un petit corpus.
  Elles ne couvrent pas la diversité réelle des schémas, shards, périodes,
  équipements, profils ITSM ni la charge des autres clients du SI.
- Quatre tickets fictifs ne qualifient pas la recherche dans un grand historique
  ITSM. Les droits natifs et la pertinence des solutions doivent être reçus avec
  des tickets représentatifs, connus autorisés et interdits.
- La charge avec fournisseur fictif ne mesure ni le quota, ni les délais, ni
  la capacité simultanée d'Albert. Les générations réelles ont leurs propres
  reçus et modèles dans la campagne conversationnelle.
- Les mesures de VM ARM64 sur le MacBook ne dimensionnent pas un serveur AMD64.
  Le test fermé ne cherche pas le point de saturation et ne définit aucun SLA.
- Les alertes de la chronologie restent un snapshot simulé. L'observabilité,
  le routage automatique entre UC, Dify et n8n restent hors périmètre.
- L'import de l'artifact CI restreint dans NiceGUI et la réception du SI réel
  restent explicitement non qualifiés. Une CI verte ne clôture pas ces points.
- La rotation a effacé une partie du journal d'accès de l'ITSM supplémentaire ;
  l'absence d'erreur dans ces lignes manquantes ne peut pas être attestée.

Les reçus privés de campagne se trouvent sous
`.runtime/ci/final-qualification-20261113/`. Le paquet de preuves exportable
contient une sélection contrôlée de résultats et d'empreintes ; il exclut les
secrets, fichiers d'identité, cookies, sessions et traces privées.
