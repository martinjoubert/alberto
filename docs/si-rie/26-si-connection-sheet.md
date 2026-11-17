# Fiche de raccordement SI-RIE — à compléter avant installation

Cette fiche rassemble les paramètres à remettre au déployeur pour suivre
le [guide pas à pas](24-deploy-walkthrough.md). Elle décrit le SI cible ; ce
n'est ni une configuration LDAP/OpenSearch standard, ni un fichier à exécuter.
Les responsables des sources renseignent leurs parties et le déployeur prépare
les fichiers du PoC dans les formats indiqués. Une valeur inconnue reste
**à fournir** : ne pas la remplacer par celle de la sandbox.

**Ne mettre aucun secret dans cette page ou le wiki.** Indiquer uniquement
le nom du fichier privé remis séparément, son propriétaire et sa destination.

## Identification et décisions de déploiement

| Élément | Valeur du SI cible / décision | Responsable et validation |
| --- | --- | --- |
| Environnement, date et contact de coordination | À fournir | Responsable SI |
| Ubuntu et architecture | À fournir : 24.04 ou 26.04, ARM64 ou AMD64 | Exploitant ; qualification complète actuelle sur 24.04 ARM64 |
| Serveur SSH et compte Unix avec sudo | À fournir ; clé/hôte vérifiés par le circuit SI | Exploitant |
| Racine des sources / dossier privé d'instance | À fournir ; hors l'un de l'autre, sans espaces | Déployeur |
| Origine de distribution et commit complet | À fournir depuis le reçu de livraison | Préparateur de la livraison |
| Archive, SHA-256 et manifeste source | À fournir depuis le reçu ; même version que les images | Préparateur de la livraison |
| Mode | `external` | Déployeur ; aucun service du SI à créer ou initialiser |
| Portail NiceGUI | Activé pour la recette ; décider son maintien ensuite | Exploitant |
| Image commune PDL/Incidents/opérateur | Référence GitLab approuvée `registry/chemin@sha256:...`, architecture et provenance | Responsable des images ; renseigne `RIE_IMAGE` |
| Image NiceGUI | Référence GitLab approuvée `registry/chemin@sha256:...`, même source | Responsable des images ; renseigne `RIE_PORTAL_IMAGE` |
| Image cliente Open WebUI | Référence approuvée, basée sur la version qualifiée 0.11.3 et l'adaptateur fourni | Responsable des images ; renseigne `RIE_OPENWEBUI_IMAGE` sur le poste |
| Réseau, proxy, CA et accès à la registry | À fournir pour le serveur, les conteneurs et les postes | Responsables réseau/PKI/registry |
| Émetteur OAuth et URLs HTTPS des deux moteurs | À fournir, joignables par le navigateur et le backend Open WebUI | Réseau/PKI ; aucun en-tête utilisateur ne remplace LDAP |
| Callbacks Open WebUI locaux | Origine/port réels de chaque poste et deux chemins exacts | Administrateur du poste ; saisie dans `oauth.json` |
| Exploitation et sauvegardes | Répertoire, capacité, destinataire et conservation | Exploitant ; archives privées contenant des secrets |

Les images du serveur ne lancent que le PoC. Les images OpenSearch,
VictoriaMetrics, LDAP et ITSM de la sandbox ne sont pas à installer dans le SI.
La construction/promotion des images précède l'intervention du déployeur ;
le guide utilise les références validées, sans dépendre d'un téléchargement
Docker Hub ou PyPI sur le serveur cible.

## Identités et droits

| Paramètre | Valeur à fournir | Fichier du PoC |
| --- | --- | --- |
| Hôte, port LDAPS et CA | DNS couvert par le certificat, port et chaîne approuvée | `config/access.json`, `config/ldap-ca.crt` |
| DN du lecteur annuaire | Véritable compte de service en lecture | `ldap.app_dn` |
| Secret du lecteur | Nom du fichier privé remis ; aucune valeur ici | `secrets/reader.password` |
| Base de recherche | DN de la branche contenant les utilisateurs | `ldap.search_base` |
| Attributs identifiant / courriel / groupes / sujet stable | Confirmer les quatre noms ; ne pas supposer ceux du laboratoire | `ldap.attribute_for_*` |
| Groupes interne / externe | DN réels ; comptes dans les deux groupes ou aucun refusés | `ldap.group_dns` |
| Groupes administrateurs des accès du PoC | DN réels ; appartenance directe d'un interne | `ldap.administrator_group_dns` |
| Compte de recette administrateur | Identifiant réel ; aucun mot de passe dans la fiche | Variable `RIE_ADMIN_LDAP` |
| Comptes de recette lecteur interne / externe / non classé | Identifiants et attendus validés par le responsable des accès | Recette de l'étape 11 |
| Règle de chaque index PDL, des logs et des documents Incidents | `blocked`, `internal` ou `authenticated` ; défaut `blocked` | `access/policy.json` |
| Mesures et tickets Incidents | Décision séparée pour chaque ressource | `incidents.metrics`, `incidents.tickets` dans la politique |

`access.json` configure le PoC pour consulter l'annuaire existant. Il ne crée
ni comptes ni groupes. `cn` nomme notamment une entrée, `ou` une branche et
`dc` les composants de la racine ; leur enchaînement constitue un DN. Par
exemple, `cn=reader,ou=services,dc=example,dc=invalid` est un **exemple fictif**,
pas une valeur du SI. Les DN réels viennent de l'administrateur LDAP.
Le PoC recherche l'utilisateur avec son lecteur, vérifie le mot de passe de
celui-ci, relit ses groupes puis applique les ACL. Les groupes imbriqués ne
sont pas développés automatiquement. Un administrateur reste soumis à
`blocked` et aux droits natifs ITSM.

## Sources et contrats des données

| Source | Informations et preuves à fournir | Destination |
| --- | --- | --- |
| OpenSearch PDL | URL HTTPS, CA, lecteur, noms physiques des index, schémas, échantillons, concepts et sens des relations d'identité | `.env`, `config/pdl.yaml`, mappings à approuver |
| OpenSearch Incidents | Index logs et documents, champs de filtre/lecture, équipements et fenêtre connue | `.env`, `config/incidents-corpus.json` |
| VictoriaMetrics | URL HTTPS, authentification lecteur, séries, labels, unités, cadences et une série par équipement/mesure | `.env`, `config/metric-mapping.json` |
| Inventaire Incidents | Équipements, alias, périmètres, interfaces, dates ISO avec fuseau et provenance | `config/incidents-corpus.json` |
| Alertes et traitements courants | Export complet autorisé, horodaté et borné à l'inventaire ; procédure de mise à jour et propriétaire | `config/itsm-tickets.json` |
| Précédents ITSM-NG | Version 1.6.17 confirmée, API, URL des tickets, CA, fuseau des dates naïves et App-Token si exigé | `config/itsm-history.json`, `secrets/itsm-users.json` |
| Chaque utilisateur ITSM | Liaison de son sujet LDAP stable à son compte ITSM et à son propre token | Fichier privé avec `itsm_user_id` numérique et `user_token` ; commande `itsm-user` |
| Albert | URL approuvée, modèles disponibles, limites/quota, proxy/CA et autorisation d'envoi des données | `.env`, `secrets/albert_key` ; configuration séparée du client local |

Le contrat métrique utilise des jauges : CPU, pertes et utilisations en %, et
latence en ms. Les compteurs, `rate`, CPU par cœur et séries multiples exigent
un prétraitement ou une adaptation à qualifier. Un nom différent peut être
mappé ; une sémantique différente ne se corrige pas en renommant un champ.

Le snapshot courant et les précédents historiques ont deux rôles distincts.
Le connecteur ITSM livré lit les précédents et solutions selon les droits
individuels ; il ne synchronise pas les alertes courantes et n'écrit aucun
ticket. Le format technique `simulator: ITSM-NG` du snapshot reste nécessaire,
même pour un export réel autorisé : la provenance et `synthetic: false`
décrivent alors les données réelles, sans prétendre à une connexion directe.
Voir les [contrats détaillés](02-configuration.md) et le
[raccordement ITSM](15-itsm-precedents.md#fiche-de-raccordement-itsm-à-faire-compléter).

## Fichiers à remettre dans le dossier `RIE_SI`

Par convention, le guide utilise `$HOME/si-connection` ; son emplacement réel
peut changer. Les fichiers de configuration du PoC sont préparés à partir des
modèles publics `deploy/ubuntu/examples/`, relus avec les responsables puis
copiés dans l'instance privée. Ils ne sont pas des exports natifs universels.

| Fichier attendu | Préparateur / revue | Mode et contenu |
| --- | --- | --- |
| Copie complétée de cette fiche | Coordination SI | Sans secrets ; conserver hors dépôt si elle contient des informations internes |
| `incidents-corpus.json` | Déployeur + responsables données | Inventaire, fenêtre connue, champs, cadences, `status: ready` après revue |
| `metric-mapping.json` | Déployeur + responsable métriques | Cinq mesures, labels et unités validés |
| `itsm-tickets.json` | Responsable des alertes + déployeur | Export complet au format attendu et provenance de sa fraîcheur |
| `ldap-ca.crt`, `ca-sources.pem` | Responsable PKI | CA LDAP et bundle des CA des sources/Albert ; pas de clé privée |
| `private/ldap-reader.password` | Responsable LDAP | `0600`, secret du seul lecteur |
| `private/opensearch.password` | Responsable OpenSearch | `0600`, secret lecteur |
| `private/victoria.token` | Responsable métriques | `0600`, token lecteur selon l'authentification retenue |
| `private/albert.key` | Responsable du fournisseur | `0600`, clé autorisée pour les appels du serveur |
| `private/itsm-app.token` | Responsable ITSM | `0600`, seulement si le serveur exige un App-Token |
| `private/LDAP_LOGIN-itsm.json` | Utilisateur / responsable ITSM | `0600`, ses seuls `itsm_user_id` et `user_token` |
| `mappings.yaml` | Déployeur + métier, facultatif | Propositions PDL ; la saisie NiceGUI peut remplacer ce fichier, jamais l'approbation |

Les vrais mots de passe LDAP des utilisateurs ne sont pas remis au déployeur
dans ces fichiers. Chaque titulaire les saisit dans la page d'autorisation
LDAP ou NiceGUI. Les secrets des CA, administrateurs LDAP/MariaDB et comptes
techniques privilégiés n'ont pas leur place dans le PoC.

## Cas de recette à convenir avant démarrage

| Cas | Demande et résultat de référence à compléter | Validateur |
| --- | --- | --- |
| PDL | Famille, périmètre, filtre, fenêtre/fuseau et résultat exact vérifié dans OpenSearch | Responsable métier PDL |
| PDL, reprise | Même cas sans période, clarification puis complément dans le même fil | Métier + déployeur |
| Incidents | Équipement, fenêtre, cinq mesures, logs, résolution et clôture distinctes | Hyperviseur |
| Historique ITSM | Précédent connu, solution acceptée et droits différents entre deux utilisateurs | Responsable ITSM |
| ACL | Comptes autorisés/refusés, sources interdites et rétablissements à tester | Responsable des accès |
| Sauvegarde/restauration | Destination privée, fenêtre d'interruption et instance de restauration neuve | Exploitant |

**Prêt à installer** lorsque les accès, fichiers, images et cas de référence
sont disponibles et validés par leurs responsables. **Prêt à utiliser** après
les étapes 11 et 12 et la [réception du SI réel](14-si-rie-acceptance.md).
La fiche reste à compléter dans le SI ; les données du laboratoire ne la valident pas.
