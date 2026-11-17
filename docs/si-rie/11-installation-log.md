# Journal de l'installation humaine sur Ubuntu

Le 6 novembre 2026, le parcours manuel a été exécuté sur une VM Ubuntu neuve,
depuis les seuls paquets sources identifiés par leur manifeste. Des défauts ont
nécessité des corrections et des reprises. Les résultats réussis ci-dessous
concernent ces reprises corrigées ; la première tentative complète n'a pas été
réussie du premier coup.

Ce journal explique les écarts rencontrés et leur résolution. Pour déployer la
livraison actuelle, suivre [Installation](01-installation.md), puis
[Configuration](02-configuration.md). Le [bilan de qualification](08-qualification.md)
reste la référence pour les résultats de charge, de panne et de compatibilité.

## Cadre de l'essai

| Élément | Situation réellement utilisée |
| --- | --- |
| Hôte | MacBook, hyperviseur Lima/VZ |
| Machine invitée | Ubuntu 24.04.5 LTS ARM64, VM `rie-u24-manual` |
| Ressources | 8 vCPU, environ 16 Go de RAM, disque virtuel de 80 Go |
| Partage de fichiers Mac | Aucun ; configuration Lima `mounts: []` |
| Transfert | Archive source publique et manifeste, puis extractions et reprises identifiées |
| Compte Ubuntu | `dinum`, UID 1000 ; commandes Docker et système avec `sudo` |
| État initial | Docker absent ; installation depuis le dépôt APT officiel |
| Outils relevés après préparation | Docker 29.9.3, Compose 5.7.0, Buildx 0.38.1 |
| Sources métier | OpenSearch, VictoriaMetrics et LDAP réels, contenant uniquement des données fictives ; tickets simulés |
| Clients | NiceGUI par tunnel ; Open WebUI installé localement sur le Mac |

Le serveur n'a utilisé ni montage du checkout Mac, ni configuration d'une pile
de démonstration Mac. Les secrets de chaque instance sont restés dans ses
dossiers privés. Le test ne prétend pas avoir raccordé les sources du SI-RIE.

## Parcours réellement rejoué

Le parcours manuel a utilisé les commandes du manuel, sans `install.sh` :
préparation système, `prepare`, activation facultative de NiceGUI, `build`,
`pull`, `up`, puis `seed`, `check` et `acceptance`. La préparation d'instance
restait possible sous le compte utilisateur ; l'accès Docker était assuré par
`sudo` avec le même chemin absolu, sans ajout au groupe Docker.

Les sorties successives ont été conservées. Chaque correction a donné lieu à
une reprise des étapes concernées ; les services, données et politiques déjà
valides ont été conservés. Des paquets corrigés ont été retransférés dans la VM.
Cette démarche qualifie le parcours après correction, sans présenter les reprises
comme une exécution initiale sans incident.

Les paquets historiques de travail n'étaient pas une publication figée : leurs
manifestes indiquaient `dirty: true`, sur le commit
`95d5f5d784245d4866dd2174e03d796b4ecb6a82`. Le manifeste et l'empreinte du paquet
final remis doivent être conservés séparément ; ce commit seul ne suffit pas à
reconstituer ces archives.

## Défauts rencontrés et corrections

| Constat réel | Cause et correction | Vérification après correction |
| --- | --- | --- |
| Une suite de commandes envoyée par SSH s'interrompait après un `compose run`, sans exécuter toutes les lignes suivantes | Docker héritait de l'entrée standard du shell `bash -s`. Les sous-processus sans entrée métier utilisent désormais `stdin=DEVNULL` ; les conteneurs opérateur et d'initialisation passent `-T --interactive=false`. L'import de mappings conserve explicitement son entrée lorsqu'elle est nécessaire. | Reprise séparée du seed, du contrôle et de la recette, puis nouvelles exécutions complètes sans consommation du script SSH |
| Le code de l'image était illisible par l'utilisateur applicatif non root | Une préparation privée avec `umask 077` produisait des fichiers copiés dans l'image avec des modes trop restrictifs. Le contexte public Docker reçoit désormais les modes `0644` pour les fichiers et `0755` pour les dossiers. Les fichiers privés d'instance restent en `0600` et `0700`. Ce défaut avait été repéré pendant les essais préparatoires, avant la recette native Ubuntu. | Images construites et services applicatifs démarrés sous leur identité non root dans la VM native |
| La page d'accueil NiceGUI renvoyait une erreur alors que les processus étaient sains | Le module documentaire supposait une profondeur de checkout inexistante sous `/delivery/wiki_export.py`. La résolution du dossier documentaire accepte maintenant le layout de l'image et `RIE_DOCS_DIR=/documentation`. | Test d'import dans un layout court, puis ouverture réelle du portail déployé |
| Les sources draw.io étaient livrées, mais les figures SVG et PNG manquaient dans l'archive | La sélection des assets ne descendait pas dans `assets/diagrams`. Le paquet inclut désormais ce sous-dossier explicitement, avec un test sur l'archive extraite. | Le paquet v3 contient huit draw.io, huit SVG et huit PNG ; leurs exports ont été contrôlés par navigateur |
| L'inspection complémentaire d'Ubuntu 26.04 signale LDAP `unhealthy` et aucune sonde Docker VictoriaMetrics, malgré les 12 contrôles techniques et 14 contrôles Chrome réussis | La sonde LDAP échoue sur `localhost` mais réussit sur `127.0.0.1` et `ldap` avec la même CA et la validation TLS. La cible passe à `127.0.0.1`, sans attribuer de cause réseau non prouvée. Une sonde `/health` est ajoutée pour VictoriaMetrics. | Nouvelle exécution v5 réussie : six services `running/healthy`, aucun OOM et zéro redémarrage, puis 12 contrôles techniques et 14 contrôles Chrome. Le reçu historique v4 conserve son résultat global négatif |
| La configuration manuelle initiale des filtres de sortie Open WebUI n'était pas suffisamment reproductible | Les étapes de configuration du client sont précisées dans la page Configuration. | Les conversations Albert ont validé la configuration autorisée réellement utilisée ; un nouveau parcours humain complet du client après cette clarification documentaire n'est pas revendiqué |

Les modes publics du contexte Docker ne s'appliquent qu'aux sources autorisées
par sa liste d'inclusion. Aucun assouplissement des droits sur les clés, jetons,
mots de passe ou sauvegardes n'a été utilisé pour résoudre les erreurs.

### Identifier les versions intermédiaires

| Archive de travail | Fichiers déclarés au manifeste | draw.io | SVG | PNG |
| --- | ---: | ---: | ---: | ---: |
| `rie-ubuntu-qualification-v1.tar.gz` | 121 | 8 | 0 | 0 |
| `rie-ubuntu-qualification-v2.tar.gz` | 124 | 8 | 0 | 0 |
| `rie-ubuntu-qualification-v3.tar.gz` | 142 | 8 | 8 | 8 |

Ces valeurs décrivent les archives historiques inspectées, pas le nombre de
fichiers à attendre d'une livraison plus récente. Les corrections documentaires
et les outils de vérification ajoutés ensuite modifient ce nombre.

## Résultats observés après les reprises corrigées

- `check` : état `ready`, LDAP vérifié, quatre mappings PDL actifs, corpus
  Incidents initialisé et dépendances nécessaires disponibles.
- `acceptance` : douze contrôles réussis, aucun échec ; aucune génération Albert
  n'est revendiquée par cette commande.
- NiceGUI déployé dans la VM : quatorze contrôles Chrome réussis, dont les trois
  démonstrations, les refus et changements de droits, les liens et exports
  documentaires ; aucune erreur JavaScript relevée.
- Conversations distinctes dans deux clients Open WebUI locaux avec Albert réel
  `gpt-oss-120b` : six cas, sept envois et sept contrôles transverses réussis.
  Les cas couvrent PDL, clarification et reprise, rapprochement de sources,
  diagnostic Incidents avec synthèse et refus Incidents d'un compte externe.

Une exécution complémentaire Ubuntu 26.04.1 a conclu la compatibilité avec le
paquet v5, SHA-256
`496792903b4c47307f715961ac9210070fd5217da088d5bcd614d27878084ff9` :
reprise d'installation et douze contrôles techniques réussis, six services
sains, dix-sept contrôles ACL et quatorze contrôles Chrome sans erreur JavaScript.
La recette étend le contrôle de l'UUID LDAP aux sessions signées de dix minutes :
recréer le même compte invalide l'ancienne session sur les deux backends avec
`directory_identity_changed`, puis une nouvelle connexion fonctionne. Le compte
jetable est supprimé et les jetons de recette révoqués. Les pages et figures
ont été comparées au paquet v5 ; cette version reste distincte du paquet final.

Le corpus PDL de référence reste daté du 12 octobre 2026 ; les observations
Incidents sont celles du 28 septembre 2026. Une exécution réussie sur ces données
fictives ne valide ni un corpus réel, ni un dimensionnement de serveur AMD64.
Les résultats de la seconde installation automatisée, d'Ubuntu 26.04, de
l'endurance et des restaurations sont décrits dans le bilan de qualification,
avec leur propre état et leurs preuves.

## Incident de l'endurance v2

**Échoué sur v2.** La campagne a terminé ses 7 200,088 secondes avec
8 460 demandes et 1 539 erreurs inattendues, toutes dans PDL : 422 résultats
`error` et 1 117 `protocol_error`.

| Utilisateurs | Demandes | Erreurs PDL inattendues | Refus ACL attendus |
| --- | ---: | ---: | ---: |
| 1 | 360 | 0 | 0 |
| 5 | 2 700 | 171 | 270 |
| 10 | 5 400 | 1 368 | 540 |
| Total | 8 460 | 1 539 | 810 |

Incidents n'a produit aucune erreur inattendue. Les 810 refus ACL attendus
sont comptés séparément des erreurs. Le critère de zéro erreur inattendue
n'est pas satisfait : l'endurance v2 est échouée.

Deux défauts successifs sont établis par les journaux :

| Défaut | Première trace UTC | Marqueurs relevés |
| --- | --- | ---: |
| Suppression concurrente d'un état expiré, `FileNotFoundError` dans `StateStore.issue` | 15:44:15 | 1 175 |
| Plafond de 2 048 états atteint, `dialogue_capacity` | 16:15:20 | 1 059 |

Ces marqueurs d'exception ne sont pas des comptes de demandes. La trace v2
ne comporte pas le sous-code nécessaire pour répartir exactement les 1 539
erreurs entre ces deux causes.

Le diagnostic initial, vers 16:08 UTC, ne montrait pas encore de saturation.
Elle est apparue ensuite au palier de dix utilisateurs. Le plafond de 2 048
états est donc un second défaut effectivement rencontré, en plus de la course
au nettoyage qui explique le début des erreurs.

Le correctif v6 tolère la disparition concurrente d'un fichier et verrouille
les opérations d'émission sur le même stockage. Un état encore valide reste
conservé malgré un ancien `mtime`.

La capacité est désormais fixée explicitement à 8 192 états et reste bornée.
Les contrôles de signature, expiration et propriétaire restent actifs et sont
couverts par les tests. Les deux corrections ont été soumises à la campagne v6 décrite ci-dessous.

Les 421 échantillons système ne montrent aucune erreur de collecte, aucun
OOM et aucun redémarrage. L'absence de ces événements ne compense pas les
erreurs applicatives de la campagne.

Après charge, la comparaison intégrale a réussi en lecture seule : 100 000
événements PDL, 100 000 logs Incidents, dix documents, vingt séries et
4 200 points métriques. Aucune donnée n'a été réparée. Le vérificateur public
v6 a été utilisé dans l'ancienne image sans reconstruire les services.

Les trente jetons de charge ont été révoqués ; aucun ne reste actif. Seul
l'ancien projet de charge `rie-31ad896af2cc` a été arrêté après collecte.
L'instance d'endurance corrigée et les autres services ont été préservés.

## Résultat final de la campagne corrigée v6

Les deux heures v6 du 6 novembre ont atteint 8 460 demandes sans erreur
inattendue, avec 810 refus ACL attendus et corpus conservé. Six transitions
avaient une cadence trop courte ; le pilote corrigé a ensuite passé le contrôle
natif de 60,066 secondes, sans nouvelle endurance complète revendiquée.

Les mesures par palier, ressources, croissance des rapports, empreintes et
limites sont conservées une seule fois dans le [bilan canonique](08-qualification.md#résultat-final-de-lendurance-corrigée-v6).
Les reçus de travail restent sous `endurance-fixed-v6/` ; ne pas republier leurs
contenus privés en bloc. Cette conclusion concerne la version v6 gelée pendant
la campagne, pas les images ni les ACL de sources ajoutées le 7 novembre.

## Statut Incidents et conversations réelles v6

La correction de `/status` accepte l'absence du champ facultatif `prepared_at`
en renvoyant `null`, tout en conservant un refus 503 pour un manifeste invalide.
Les droits sur les deux index de logs et documents sont contrôlés avant lecture.
La recette HTTP réelle v6 a vérifié un interne en 200 avec état `ready` et
un externe en 403 sans liste d'équipements, sans génération demandée.
Le premier parcours complet Albert v6 a réussi cinq cas sur six en sept envois.
Le dernier cas externe a atteint la protection de cadence sans publier de
preuve, puis la boucle du client a été arrêtée.

Le harnais attendait depuis le début du cas précédent, au lieu de sa fin.
Après correction, le seul cas externe a été repris manuellement après le
délai complet : un envoi et sept contrôles réussis, sans changer modèle ou
configuration des backends.

Le total est de huit envois. Les six cas disposent de preuves positives mais
le parcours initial reste échoué ; aucun nouveau parcours complet corrigé
n'est revendiqué. Les clients ont été arrêtés en conservant conversations et
volumes, et les deux nouveaux jetons ont été révoqués.

## Retrouver les preuves sans exposer les secrets

Les preuves de travail sont rangées sous `.runtime/ci/` et les archives sous
`.runtime/delivery/`. Elles ne sont pas incluses en bloc dans le ZIP public :
certains sous-dossiers contiennent des reçus de conversation ou des secrets
individuels. Les comptes rendus partagés doivent se limiter aux résultats et
captures autorisés.

| Étape | Référence de travail à conserver avec la remise interne |
| --- | --- |
| État initial et prérequis | `ubuntu24-before-install.log`, `ubuntu24-manual-prerequisites.log` |
| Installation et reprises | `ubuntu24-manual-install.log`, `ubuntu24-manual-seed.log`, `ubuntu24-manual-check.log`, `ubuntu24-manual-acceptance.log` |
| Réintégration des assets v3 | `ubuntu24-portal-assets-v3.log` |
| Parcours navigateur corrigé | `ubuntu-qualification/browser-native24-v3/browser-results.json` et captures datées |
| Génération Albert réelle | `ubuntu-qualification/openwebui-real/real-conversation-results.json` ; les reçus bruts restent privés |
| Compatibilité et sessions Ubuntu 26.04 v5 | `ubuntu-qualification/acl-native26-v5/report.json`, `container-health.json` dans le même dossier, et `ubuntu-qualification/browser-native26-v5/browser-results.json` |
| Intégrité des sources | `DELIVERY-MANIFEST.json` dans chaque archive et son fichier `.tar.gz.sha256` |

Le vérificateur `verify_delivery.py`, ajouté au parcours remis, contrôle tous
les fichiers d'une extraction neuve avant construction. Il renforce la remise
au tiers ; il ne transforme pas rétroactivement les premières tentatives en
installations sans correction.

## Revue et reprises du 7 novembre 2026

La revue avant validation humaine a séparé les parcours d'installation scriptée
et manuelle, ajouté les modèles externes et clarifié l'exploitation. Le runtime
commun a été réduit ; NiceGUI et les outils de qualification restent facultatifs.
Les droits de sources UC2 sont ajoutés explicitement, sans ouverture des
politiques existantes.

Le premier candidat C1 a échoué au démarrage PDL sur les deux nouvelles
instances : une dépendance Python encore utilisée par le moteur avait été
exclue de la liste runtime. Les conteneurs C1 ont été arrêtés et leurs volumes
conservés. C2 rétablit ce module et vérifie les imports dans un contexte isolé,
puis pendant la construction sans réseau. Les nouvelles installations C2,
les ACL source par source, seize contrôles navigateur par VM, l'exploitation
et le contrôle court de charge ont ensuite réussi. Le [bilan canonique](08-qualification.md)
conserve les chiffres et leurs limites.

Lors de la recette de restauration, le contrôleur a mal décodé une sortie
contenant une ligne de commande avant le JSON. L'import produit avait réussi :
la reprise a comparé ses volumes inchangés avant tout démarrage, sans réimporter
la sauvegarde. Cette erreur de harnais est conservée, distincte d'une perte de
données ou d'un échec de restauration produit.

La vérification documentaire a aussi trouvé une ancienne ancre numérotée après
réorganisation du manuel, puis l'absence d'identifiants de titres dans le rendu
NiceGUI. Les liens de section ont été corrigés et testés hors réseau. Le portail
ajoute maintenant les ancres à l'affichage sans changer le Markdown téléchargé ;
le portail C3 a ensuite passé dix-sept contrôles navigateur sur chaque VM,
dont la section cible effectivement visible après navigation asynchrone.
Les captures du 7 novembre illustrent C2 ; elles ne sont pas réétiquetées comme
preuves de ce correctif ultérieur. Les fichiers runtime et manifestes/configs
d'exécution C2/C3 sont identiques ; le changement d'index OCI d'attestation
BuildKit est conservé comme métadonnée distincte.

## Reprendre le laboratoire conservé sur le MacBook

La remise de corrections du 7 novembre utilise de nouvelles instances C1,
avec groupe administrateur distinct et Bruno lecteur interne. Les anciennes
instances C2 et leurs preuves sont conservées sans migration implicite.
La [matrice SI-RIE](14-si-rie-acceptance.md) conserve les essais à mener sur le
réseau et les données réels ; les commandes du laboratoire ne les qualifient pas.

Cette section concerne les VM du MacBook, depuis son checkout d'origine.
Elle ne remplace pas l'installation sur un serveur du SI. Lima doit être
présent ; `RIE_LIMACTL` précise son exécutable et `RIE_VM_HOME` ses disques privés
si le checkout a changé de place. Ces disques ne font pas partie du paquet source.

Les références prévues pour la remise du 7 novembre sont les **nouvelles**
instances de corrections C1. Pour la remise, conserver leurs données et consulter le reçu
privé de fin de campagne qui atteste leur arrêt effectif. Ne pas relancer ces
commandes pendant une campagne active. Le paquet remis est vérifié au chemin
`/home/dinum/remise-20261107-corrections-final/rie-delivery` lors du transfert final ; conserver
ce reçu avec le manifeste de livraison.

| VM | Nouvelle instance de référence | Accès sur le Mac |
| --- | --- | --- |
| `rie-u24-auto` | `/home/dinum/.local/share/rie-corrections-20261107-c1` | PDL 29910, Incidents 29950, NiceGUI 28090 |
| `rie-u26-compat` | `/home/dinum/.local/share/rie-corrections-20261107-c1` | PDL 39910, Incidents 39950, NiceGUI 38090 |

La VM `rie-u24-manual`, son instance `/home/dinum/.local/share/rie-manual` et ses
sources `/home/dinum/remise-20261106/rie-delivery` restent une **référence historique
arrêtée du 6 novembre**. Elle ne reçoit pas automatiquement les nouvelles ACL.
Les anciennes instances des deux autres VM sont également conservées, pas
réinitialisées à la place des nouvelles.

Après clôture, depuis la racine du checkout sur **le Mac**, reprendre la
nouvelle référence principale :

```bash
python3 deploy/ubuntu/vm.py start --name rie-u24-auto
python3 deploy/ubuntu/vm.py shell --name rie-u24-auto -- sudo python3 /home/dinum/remise-20261107-corrections-final/rie-delivery/deploy/ubuntu/manage.py up /home/dinum/.local/share/rie-corrections-20261107-c1
python3 deploy/ubuntu/vm.py shell --name rie-u24-auto -- sudo python3 /home/dinum/remise-20261107-corrections-final/rie-delivery/deploy/ubuntu/manage.py check /home/dinum/.local/share/rie-corrections-20261107-c1
```

Attendu : même instance, données et droits, puis `status: ready`. Ouvrir
`http://127.0.0.1:28090`. Les redirections du laboratoire sont fournies par Lima ;
sur le serveur du SI, employer le tunnel du manuel. Pour 26.04, remplacer le
nom de VM et utiliser l'accès 38090 du tableau.

Les ports internes Ubuntu restent 19910/19950/18090 ; les décalages ci-dessus
concernent leur accès depuis le Mac. Pour l'Open WebUI local, employer donc
29910/29950 sur la référence 24.04 ou 39910/39950 sur 26.04 au lieu des URL internes
copiées dans une VM. Les projets de charge historiques et nouveaux sont séparés
et arrêtés après collecte ; ne pas leur appliquer le seed de référence.

Après utilisation, arrêter l'instance choisie puis la VM :

```bash
python3 deploy/ubuntu/vm.py shell --name rie-u24-auto -- sudo python3 /home/dinum/remise-20261107-corrections-final/rie-delivery/deploy/ubuntu/manage.py stop /home/dinum/.local/share/rie-corrections-20261107-c1
python3 deploy/ubuntu/vm.py stop --name rie-u24-auto
```

Les volumes sont conservés. Les jetons temporaires de qualification sont
révoqués ou retirés ; émettre un nouveau jeton personnel pour une nouvelle
séance Open WebUI. Sa connexion locale ne nécessite aucun serveur frontend
central supplémentaire.

## Sandbox externe du 16 novembre 2026 — certificat LDAP par adresse IP

La recette manuelle sur `rie-u24-sandbox-20261116` a validé les fichiers et
construit les images. Les trois processus du PoC sont devenus `healthy`, mais
le premier `check` a échoué avec `directory_unavailable_or_denied`. Les paramètres
LDAP, le compte lecteur et la CA correspondaient à la fiche ; les cinq services
SI restaient sains. Cette tentative demeure consignée en échec.

Le test depuis le conteneur PDL a confirmé TCP et la chaîne TLS avec Python,
puis reproduit un refus du vérificateur de nom de `ldap3` 2.9.1. Sous Python 3.12,
son repli historique n’utilisait pas les SAN de type adresse IP. La correction
confie la vérification DNS/IP et de la chaîne de confiance au contexte TLS
Python/OpenSSL sur la connexion effective, avant l’envoi de credentials.
Les contrôles d’identité, groupes, administration et politique restent les mêmes.

Neuf régressions utilisent de vrais échanges TLS sur des paires de sockets
locales, sans accès réseau : DNS, IPv4 et IPv6 autorisés, mauvais noms/adresses,
CA non reconnue et certificat expiré refusés. Les trois suites Ubuntu terminent
avec **1 302 tests réussis et un ignoré sur Mac** : source 533 réussis et un
ignoré, PDL 216 réussis, Incidents 553 réussis. Les résultats JUnit et Allure sont
conservés sous `.runtime/ci/ldap-ip-san-20261116`.

Un processus de qualification dans le conteneur réel, chargé avec le code corrigé
en mémoire, retrouve Alice administratrice interne, Bruno lecteur interne et Eric
externe. Camille non classée et un mauvais mot de passe sont refusés. Il ne
modifie ni image persistante ni données et n’appelle pas Albert. La qualification
des images reconstruites et la reprise de la recette humaine restent des preuves
séparées de cet essai ; la campagne conversationnelle et les deux heures
historiques ne sont pas automatiquement attribuées au correctif.

Le nouveau paquet `rie-si-rie-ubuntu-2026-11-16-ldap-tls-fix.tar.gz` doit être
vérifié et extrait dans un nouveau dossier. L’archive initiale et ses 275 fichiers
extraits sont conservés intacts. La configuration déjà saisie dans
`/home/dinum/.local/share/dinum-rie-si` est réutilisée ; aucune nouvelle préparation
ni approbation automatique de mapping n’est effectuée.


## Administration des mappings PDL — 16 novembre 2026

La difficulté de saisie manuelle du fichier opérateur a conduit à ajouter le
formulaire **Mappings PDL** dans le portail Ubuntu facultatif. Le fichier reste
un format d’échange ; l’utilisateur choisit les champs découverts, confirme les
identités communes, vérifie la proposition puis examine chaque revue avant
approbation. Le groupe LDAP d’administration et les droits d’index sont relus
à chaque opération. La décision est liée à l’état de la revue et de la version
active affichées, contrôlé sous le verrou du catalogue.

La recette navigateur réelle réussit ses **13 contrôles** sur la troisième
instance isolée, Ubuntu 24.04 ARM64, avec OpenLDAP et OpenSearch du SI sandbox.
Les deux premières instances de qualification et leurs échecs sont conservés,
arrêtés avec leurs données. Les défauts trouvés et les limites sont décrits dans
le [bilan de qualification](08-qualification.md#administration-guidée-des-mappings--16-novembre-2026).
Les 27 régressions ciblées couvrent aussi une saisie complète sans YAML et les
valeurs incompatibles. Les trois suites terminent avec **1 329 tests réussis
et un ignoré** : source 560 réussis et un ignoré, PDL 216, Incidents 553.
Les résultats JUnit/Allure sont sous `.runtime/ci/mapping-ui-20261116-release`. La CI les exécute avec les autres tests Ubuntu, PDL et
Incidents. Les preuves brutes restent sous `.runtime/`, les deux captures
publiques datées sous `docs/si-rie/assets/screenshots-2026-11-16/`.

Le paquet `rie-si-rie-ubuntu-2026-11-16-mapping-ui.tar.gz` remplace les images à
partir des sources vérifiées, en conservant l’instance existante : aucune
nouvelle commande `prepare`, aucun seed et aucune approbation automatique. Les
archives précédentes restent immuables. La documentation autonome et le ZIP
Wiki.js sont régénérés. Les contrôles sans IA ne valent pas validation Albert ;
la suite humaine reste l’examen des quatre revues, le raccordement historique
ITSM et les conversations réelles des deux UC.

Le contrôle du paquet extrait a aussi repéré l’absence des deux nouvelles
captures dans la liste explicite de livraison. Cette liste a été complétée et
le test vérifie désormais leur présence et leurs liens après export Wiki.js
hors dépôt ; le premier rapport en échec reste conservé.
