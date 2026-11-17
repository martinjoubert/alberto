# Fiche de raccordement SI-RIE — à compléter avant installation

> Complément de référence — le [guide principal](../README.md) reste le seul parcours de déploiement à exécuter.

Cette page est la **liste des informations à obtenir auprès des responsables
du SI**, avec leurs validations et les cas de recette. Le déployeur saisit les
paramètres dans l’espace **Déploiement** de NiceGUI, selon l’[étape 6 du guide](../README.md#6-préparer-le-raccordement-avec-nicegui).
L’assistant produit une fiche technique et les fichiers de configuration du PoC ;
il n’est pas nécessaire de compléter tous ces tableaux Markdown avant la saisie.
Les validations de responsabilité et les cas de recette restent à convenir avec le SI.

La fiche décrit le SI cible ; ce n’est ni une configuration LDAP/OpenSearch
standard, ni un fichier à exécuter. Une valeur inconnue reste **à fournir** ;
les paramètres fictifs de la sandbox ne remplacent pas ceux du SI réel.
Les secrets sont remis séparément. Les tests de connexion suivent la préparation.

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

## Textes Open WebUI à préparer pour le SI réel

Le **cas de référence** est une demande complète dont les faits attendus ont
été vérifiés dans les sources par le responsable métier. Le déployeur reçoit
**le texte exact à envoyer et son attendu**, pas seulement « faire un diagnostic ».
Les blocs ci-dessous sont des modèles : remplacer **tous les champs entre
crochets**, faire valider le texte résultant, puis le copier dans Open WebUI.
Ne pas envoyer les crochets et ne pas remplacer les valeurs manquantes par
`edge-a`, `demo-alpha`, R1 ou les dates de la sandbox.

### PDL : comptage et clarification

**Modèle SI-RIE — PDL, nouveau fil : demande complète de référence.** Exemple
pour PFS si cette famille et la notion de refus sont réellement raccordées :

```text
Compte les refus PFS dans [PERIMETRE_AUTORISE] le [DATE_PDL] de [HEURE_DEBUT_PDL] à [HEURE_FIN_PDL], [FUSEAU_PDL].
```

Noter le **nombre exact**, les index, le filtre du refus et la requête manuelle
qui établissent l’attendu. Si ce comptage n’est pas disponible, le responsable
PDL doit fournir une autre demande de comptage prise en charge et son résultat.

**Autre nouveau fil, même modèle : première demande sans la période.**

```text
Compte les refus PFS dans [PERIMETRE_AUTORISE].
```

Attendre la demande de période. **Dans ce même fil**, envoyer :

```text
Le [DATE_PDL] de [HEURE_DEBUT_PDL] à [HEURE_FIN_PDL], [FUSEAU_PDL].
```

Attendu : clarification avant lecture métier, puis le résultat du comptage
complet préparé, avec les mêmes filtres. L’absence de période ne doit pas être
compensée par celle d’une ancienne conversation.

### PDL : rapprochement et suivi des sources

**Nouveau fil, modèle SI-RIE — PDL.** Fournir la liste des familles réellement
raccordées et leurs documents attendus :

```text
Montre les logs de l'équipement [EQUIPEMENT_PDL] dans les familles [FAMILLES_RACCORDEES] du périmètre [PERIMETRE_AUTORISE], le [DATE_PDL] de [HEURE_DEBUT_PDL] à [HEURE_FIN_PDL], [FUSEAU_PDL].
```

**Même fil**, en choisissant une famille effectivement incluse et autorisée :

```text
Pour ce même équipement, sur la même période et dans le même périmètre, garde uniquement la famille [FAMILLE_CIBLE].
```

Noter les documents de la seule cible, les sources servant à l’identité et
les leurres à exclure. Si un résultat vide est à tester, fournir aussi une
fenêtre **vérifiée vide** dans la source et sa question complète.

### Incidents : diagnostic, précédents et suivi

**Nouveau fil, modèle SI-RIE — INCIDENTS : demande complète.**

```text
Diagnostique [EQUIPEMENT_INCIDENT] le [DATE_INCIDENT] de [HEURE_DEBUT_INCIDENT] à [HEURE_FIN_INCIDENT], [FUSEAU_INCIDENT], sur le site [SITE_AUTORISE]. Recherche les précédents ITSM-NG disponibles et présente la solution la mieux étayée, ses conditions et les niveaux de confiance du diagnostic et de la solution. Cite les sources et distingue résolution technique et clôture du ticket.
```

Fournir les mesures/couverture, logs, ticket courant, horodatages et un précédent
connu accessible au compte testé. Si l’historique ITSM n’est pas raccordé, le
noter ; aucune solution historique n’est attendue par défaut.

**Même fil**, pour une heure de référence connue à l’intérieur de la fenêtre :

```text
À [HEURE_DE_REFERENCE], [FUSEAU_INCIDENT], l'alerte de cet équipement était-elle résolue et le ticket déjà clôturé ? Cite les horodatages disponibles.
```

Le responsable renseigne l’attendu de cette heure, ou l’absence de date source.
Ne pas reprendre 11:30 ou 12:15 de la simulation comme faits du SI.

**Autre nouveau fil, même modèle**, pour tester la clarification Incidents :

```text
Diagnostique [EQUIPEMENT_INCIDENT] sur le site [SITE_AUTORISE].
```

Après la demande de période, **dans ce même fil** :

```text
Le [DATE_INCIDENT] de [HEURE_DEBUT_INCIDENT] à [HEURE_FIN_INCIDENT], [FUSEAU_INCIDENT].
```

Respecter l’intervalle minimal de synthèse et tout délai annoncé par le
fournisseur avant une nouvelle génération ; ne pas désactiver ces contrôles.

### ACL : réutiliser des textes identifiés

Pour comparer les utilisateurs, créer un nouveau fil **pour chaque compte**,
avec sa propre autorisation OAuth. Pour tester un retrait ou rétablissement,
conserver le fil de ce compte et **renvoyer la demande complète**, période
comprise. Ne pas utiliser un simple « recommence » qui ajoute une ambiguïté.

| Texte à remettre au déployeur | Attendu à compléter selon la politique du SI |
| --- | --- |
| `Quels sont mes index accessibles ?`, modèle PDL, sans période | Liste autorisée de chaque compte, avant/après modification |
| Demande PDL complète sur la famille de recette | Compte autorisé, compte refusé, références permises et preuves absentes en refus |
| Demande Incidents complète ci-dessus | Diagnostic selon les quatre droits, refus après `blocked`, nouvelle lecture après rétablissement pour les seuls comptes autorisés |
| Demande Incidents avec précédents, pour deux utilisateurs ITSM | Identifiants des tickets accessibles à chacun ; tickets, solutions et suivis privés qui ne doivent pas apparaître |

Conserver avec chaque texte l’identité de test, les règles initiales à restaurer,
les sources, la période/fuseau et la validation métier. La recette suit ensuite
l’[étape 11](../README.md#11-jouer-les-deux-uc-et-leurs-acl-avec-albert).

**Prêt à installer** lorsque les accès, fichiers, images et cas de référence
sont disponibles et validés par leurs responsables. **Prêt à utiliser** après
les étapes 11 et 12 et la [réception du SI réel](14-si-rie-acceptance.md).
La fiche reste à compléter dans le SI ; les données du laboratoire ne la valident pas.
