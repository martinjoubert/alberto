# Installer le PoC sur un SI simulé déjà disponible

> Complément de référence — le [guide principal](../README.md) reste le seul parcours de déploiement à exécuter.

## État conservé et utilisation du guide principal

Le [guide pas à pas](../README.md) est désormais rédigé pour le SI
réel. Cette page conserve les **valeurs de substitution du laboratoire** et
ses résultats attendus, sans introduire un second parcours à exécuter.

La recette du 16 novembre est terminée sur les parties ACL et exploitation :
112 résultats Allure réussis, zéro échec. L'instance restaurée
`/home/dinum/.local/share/dinum-rie-si-restored-20261116` reste l'instance à
reprendre ; l'originale est conservée arrêtée. Le fichier `~/VALIDATION-STATE.md`
et la page [Qualification](08-qualification.md) donnent le point de reprise.
Les sections ci-dessous décrivent aussi le **départ d'un nouvel exercice** :
elles ne demandent pas de réinstaller l'instance déjà validée.

| Étape du guide principal | Valeurs ou particularités du laboratoire du 16 novembre |
| --- | --- |
| 1, accès Ubuntu | Depuis le dépôt Mac, `.runtime/sandbox-si-20261116/connect.command`. Le fichier SSH est généré localement ; il ne fait pas partie du paquet SI. Compte `dinum`, Ubuntu 24.04 ARM64. `verify-si.py` sous `~/si-connection` contrôle les sources fictives. |
| 2, version | Pour rejouer la preuve historique : archive `rie-si-rie-ubuntu-2026-11-16-user-access-r8.tar.gz`, sources `~/rie-deploy-sources-2026-11-16-user-access-r8/rie-delivery`. Pour une nouvelle recette, utiliser le paquet et le reçu effectivement remis. |
| 3, identités | Administratrice `alice.martin`, lecteur interne `bruno.lecteur`, externe `eric.durand.ext`, non classée `camille.inconnue`. Mots de passe propres à cette sandbox, dans les fichiers privés du seul compte concerné. |
| 6, sources | Voir la fiche ci-dessous : adresses `172.17.0.1`, CA privées de laboratoire et fichiers de `~/si-connection`. La clé serveur existe dans l'instance privée ; ne pas la remplacer pour reprendre une recette. |
| 7, images | Images de laboratoire déjà construites localement ; aucune publication dans la registry SI n'est revendiquée. Pour une première installation de laboratoire, le responsable peut utiliser `manage.py build` au lieu de la récupération d'images SI. |
| 8, catalogue | Quatre index physiques `rie-pfs-fixture`, `rie-bb-fixture`, `rie-pfaing-fixture`, `rie-tacacs-fixture`, approuvés humainement, version 1. Ne pas réapprouver pour reprendre l'existant. |
| 9, ITSM | API native de laboratoire 1.6.17, données fictives, `synthetic: true`. Alice et Bruno ont chacun leurs credentials et des droits natifs différents. |
| 10, réseau | Tunnel Mac : ports `28090`, `29910`, `29950` vers `18090`, `19910`, `19950` Ubuntu. Les origins OAuth de laboratoire sont HTTP loopback ; le SI réel exige ses URLs HTTPS. |
| 10, client Mac | Open WebUI `http://127.0.0.1:8080`, modèle relais `recette-20261109.gpt-oss-120b`. Ce préfixe appartient à ce client historique, pas au fournisseur. Sur Colima, `.runtime/sandbox-si-20261116/client-tunnels.command` fournit les redirections supplémentaires ; ce lanceur reste local au Mac. |
| 11, PDL | Textes copiables [PDL-1 à PDL-5](../README.md#pdl--clarification-puis-reprise-du-même-fil). « Compte les refus PFS dans demo-alpha », puis « Le 12 octobre 2026 de 08:00 à 09:00 UTC ». Attendu : deux refus PFS. NiceGUI accepte la question complète dans le champ Question métier. |
| 11, Incidents | Textes copiables [INC-1 à INC-3](../README.md#incidents--diagnostic-suivi-et-précédents). `OBS-RTR-01`, 28/09/2026 de 10:00 à 12:00 Europe/Paris ; cinq mesures de 120 points, 120 logs, congestion uplink0, résolution 11:30 et clôture 12:15 distinctes. La démo NiceGUI utilise ce scénario fixe. |
| 11, ACL | PDL `authenticated` pour les quatre index ; index Incidents, mesures et tickets `internal`. Deux cycles d'interdiction/rétablissement ont abouti à la révision 5, avec les règles d'origine rétablies. |
| 12, reprise | Sauvegarde `~/rie-backups/sandbox-20261116`, puis instance restaurée ci-dessus. Les sources SI n'ont pas été arrêtées ou remplacées. Aucun nouveau test de deux heures n'est déduit de cette recette. |

Un nouveau démarrage Lima peut changer son port SSH. Le préparateur actualise
le fichier local depuis la configuration de cette VM ; ne pas réutiliser en
production la désactivation de vérification d'hôte du lanceur de laboratoire.

Cette sandbox prépare le **rôle du déployeur du PoC**, avec un Ubuntu neuf et
des sources préparées séparément par l'administrateur du SI simulé. Le PoC
s'installe en **mode `external`**, comme pour le raccordement au SI-RIE réel.
Les données restent fictives. Le mode décrit la responsabilité des services,
pas l'authenticité des données ni le fournisseur IA.

Suivre le **[déroulé unique de déploiement, étapes 1 à 12](../README.md)**,
dont le point d'entrée est remis dans `$HOME/README.md` sur Ubuntu. La présente page est la fiche
de paramètres et de limites de l’exercice ; elle ne constitue pas un second
parcours à exécuter. Configuration, droits et approbations restent à effectuer
par le déployeur.

## Ce qui est prêt et ce qui reste à installer

La séance du 16 novembre 2026 utilise la VM `rie-u24-sandbox-20261116` :
Ubuntu 24.04 ARM64, 8 vCPU, 16 Gio de RAM, disque de 80 Gio et aucun partage
des dossiers du Mac. Docker et les outils de contrôle sont déjà installés
pour héberger le SI simulé ; leur présence doit être vérifiée à l’étape 4 du déroulé.

| Services du SI simulé déjà disponibles | Raccordement proposé au PoC |
| --- | --- |
| OpenSearch et six index de référence | HTTPS, lecteur authentifié, découverte, schémas et recherches ; écritures refusées |
| VictoriaMetrics et séries réseau | HTTPS, Bearer lecteur ; écritures refusées |
| OpenLDAP, quatre comptes et trois groupes | LDAPS avec validation du certificat, lecteur annuaire et identités individuelles |
| ITSM-NG 1.6.17 avec MariaDB | API HTTPS native, App-Token et deux identités individuelles, droits natifs distincts |

Les cinq conteneurs du projet `rie-si-sandbox-20261116` appartiennent au SI
simulé. Une passerelle Nginx sur Ubuntu assure HTTPS et les restrictions de
lecture. Cette préparation est indépendante des images et volumes du PoC.

**À installer par le déployeur :** moteurs PDL et Incidents, leurs ACL et,
pour cet exercice, NiceGUI. Aucun moteur, catalogue approuvé ni jeton utilisateur
du PoC n'est créé avant cette installation. Open WebUI reste sur le poste
utilisateur ; ses connexions vers cette nouvelle instance restent à configurer.

## Vérifier le départ sur Ubuntu

Se connecter avec l'accès SSH remis pour la nouvelle VM, puis :

```bash
# Affiche le compte Unix de cette session ; attendu sur la sandbox : dinum.
whoami
# Affiche le nom du serveur connecté pour confirmer que vous agissez sur le bon Ubuntu.
hostname
# Mémorise le dossier de la fiche de raccordement et des accès fournis par le SI simulé.
export RIE_SI="$HOME/si-connection"
# Contrôle les sources et identités du SI simulé, sans installer le PoC ni appeler Albert ; attendre passed: true.
python3 "$RIE_SI/verify-si.py"
# Liste les conteneurs actifs et leur état ; au départ, seuls les cinq services du SI simulé sont attendus.
sudo docker ps --format 'table {{.Names}}\t{{.Status}}'
```

**Attendu :** compte `dinum`, serveur `lima-rie-u24-sandbox-20261116`,
`"passed": true` et cinq conteneurs du SI simulé démarrés et `healthy`. Le vérificateur
consulte les sources et identités ; il n'installe rien et n'appelle pas Albert.
Les sessions de contrôle ITSM sont fermées, sans modification de ticket.

Pour vérifier les accès HTTPS sans saisir ni afficher les secrets :

```bash
# Vérifie l'accès HTTPS à OpenSearch avec le fichier privé fourni ; affiche HTTP 200 en cas de succès.
curl --config "$RIE_SI/private/opensearch-curl.conf" --write-out '\nHTTP %{http_code}\n'
# Vérifie l'accès HTTPS à VictoriaMetrics avec le fichier privé fourni ; affiche HTTP 200 en cas de succès.
curl --config "$RIE_SI/private/metrics-curl.conf" --write-out '\nHTTP %{http_code}\n'
```

**Attendu :** HTTP `200` et code de sortie `0` pour chaque commande. En cas de
refus TLS, d'identité ou de source indisponible, résoudre cette cause avant
l'installation. Les fichiers cURL contiennent des credentials privés ; leur
contenu n'est pas à joindre au compte rendu.

## Fiche de raccordement de cet exercice

Cette fiche est le **document de paramètres remis par le SI au déployeur**.
Elle décrit les services déjà disponibles et les accès et données à utiliser
pour configurer le PoC en `external`. Voir la
[définition du README](README.md#fiche-de-raccordement) et le
[glossaire](23-glossary.md#fiche-de-raccordement).

Ces adresses sont accessibles **depuis Ubuntu et ses conteneurs**. Elles ne
désignent ni les adresses du Mac, ni les services du SI-RIE réel. Le fichier
`$RIE_SI/connection.json` reprend les paramètres sans secret.

| Paramètre | Valeur de la sandbox |
| --- | --- |
| Mode du PoC et nouveau dossier d'instance | `external`, `$HOME/.local/share/dinum-rie-si` |
| Compte LDAP prévu pour l'administration des accès du PoC | `alice.martin`, fictif dans cette sandbox ; utiliser un véritable compte habilité du LDAP cible pour le SI réel |
| OpenSearch | `https://172.17.0.1:19443`, lecteur `poc-reader` |
| VictoriaMetrics | `https://172.17.0.1:19444` |
| LDAP | `172.17.0.1`, port `1636`, TLS et validation du certificat activés |
| API historique ITSM-NG | `https://172.17.0.1:19445/apirest.php` |
| Racine publique ITSM de laboratoire | `https://172.17.0.1:19445` |
| Fuseau des dates ITSM sans offset | `Europe/Paris` |

Les DN du lecteur, de la base et des groupes ainsi que les attributs sont dans
`connection.json`. Le sujet LDAP stable est `entryUUID`. Alice est interne et
administratrice des accès ; Bruno interne lecteur ; Eric externe ; Camille
sans groupe de classification. Leurs mots de passe sont **nouveaux et propres
à cette sandbox**. Les règles ACL du PoC restent à choisir explicitement :
les groupes LDAP ne les créent pas automatiquement.

| Fichier remis sous `RIE_SI` | Utilisation après revue par l'installateur |
| --- | --- |
| `ldap-ca.crt` | Copier dans `config/ldap-ca.crt` de l'instance |
| `ca-sources.pem` | Bundle comprenant CA de laboratoire et CA publiques pour Albert ; copier dans `config/ca-sources.pem`, puis définir `SOURCE_CA_FILE=/data/config/ca-sources.pem` |
| `private/ldap-reader.password` | Copier dans `secrets/reader.password` |
| `private/opensearch.password` | Copier dans `secrets/opensearch_password` |
| `private/victoria.token` | Copier dans `secrets/victoria_token` |
| `private/LDAP_USERNAME.json` | Identifiant et mot de passe du seul compte concerné pour NiceGUI |
| `metric-mapping.json` | Contrat des séries/unités à relire puis copier dans `config/metric-mapping.json` |
| `incidents-corpus.json` | Équipements, fenêtres et provenance à relire puis copier dans `config/incidents-corpus.json` |
| `itsm-tickets.json` | Snapshot des alertes/traitements courants, à copier dans `config/itsm-tickets.json`, distinct des précédents par API |
| `pdl-mappings-candidates.yaml` | Proposition des quatre familles et relations à revoir, importer et approuver ; aucun catalogue n'est préapprouvé |
| `itsm-history-candidate.json` | Proposition du raccordement API natif, avec `synthetic: true` pour signaler ses données fictives ; activation explicite par l'opérateur |
| `private/itsm-app.token` et `private/LDAP_USERNAME-itsm.json` | App-Token et credentials natifs à raccorder selon le [guide ITSM-NG](15-itsm-precedents.md#raccordement-sur-ubuntu), pour Alice et Bruno individuellement |

Les fichiers et dossiers privés restent en `0600` et `0700`. Pour l'instance
créée avec `sudo`, copier un fichier relu avec
`sudo install -m 600 SOURCE_PATH DESTINATION_PATH`. Les mots de passe
administrateur LDAP/MariaDB et les clés privées de CA ne sont pas remis au PoC.

## Poursuivre l'installation external

Continuer dans le [déroulé unique](../README.md), à l’étape en cours :
**4 après les variables**, **6 pour renseigner les sources**, **8 pour approuver
les mappings**, **9 pour l’historique ITSM**, **10 pour le poste**, **11 pour les
conversations Albert et ACL**, puis **12 pour l’exploitation**. Les commandes
y sont regroupées ; ne pas reprendre l’extraction ni une autre installation.

## Portée de la répétition

Les protocoles, TLS, identités, bases et l'API ITSM-NG sont réels ; les données
et équipements sont fictifs. Les alertes et traitements courants restent un
snapshot. La passerelle de lecture, le réseau Docker et les images publiques
du laboratoire ne reproduisent pas le proxy, la registry GitLab filtrée,
la PKI, les ACL natives OpenSearch, les schémas ou volumes réels du SI-RIE.
Ces points restent dans la [matrice de réception](14-si-rie-acceptance.md).

La disponibilité du SI simulé ne valide pas le PoC encore absent, ses ACL,
les conversations Albert ou la charge. La nouvelle installation et sa recette
seront consignées séparément dans la [fiche humaine](12-manual-acceptance.md).
