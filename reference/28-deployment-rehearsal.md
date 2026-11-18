# Répétition complète du déploiement — 18 novembre 2026

Les **douze étapes du [guide principal](../README.md)** ont été rejouées sur un
serveur Ubuntu neuf, en mode `external`, en utilisant un SI fictif déjà disponible
sur une autre VM. **Le parcours corrigé fonctionne pour les deux UC et les ACL**,
y compris après une vraie restauration. La première exécution a révélé cinq
écarts ; ils sont détaillés ci-dessous, avec leurs reprises.

Cette preuve prépare la réception dans le SI-RIE. Elle ne qualifie pas ses
données, son réseau, son annuaire ni ses performances.

## Environnement et version réellement testés

| Élément | Essai du 18 novembre |
| --- | --- |
| Serveur du PoC | VM Lima `rie-u24-deploy-20261118`, Ubuntu 24.04 ARM64, 8 vCPU, 16 Go, disque de 80 Go, aucun dossier Mac monté |
| Services installés | Seulement `pdl`, `incidents`, `portal` et tâches opérateur ; aucun serveur Open WebUI centralisé ni base SI créée sur cette VM |
| SI existant | Autre VM : OpenSearch, VictoriaMetrics, OpenLDAP TLS et ITSM-NG **natif 1.6.17** avec données et identités fictives |
| Tickets courants | Snapshot fictif explicitement tracé ; distinct des précédents lus par API native ITSM-NG |
| Distribution des images | Registry de laboratoire séparée, TLS, CA privée, authentification et récupération par digest ; ce n’est pas la registry GitLab filtrée du SI |
| Poste utilisateur | Open WebUI local existant avec adaptateur OAuth approuvé, navigateur Chrome piloté par Playwright |
| Identités | Comptes Open WebUI ordinaires dédiés ; autorisations LDAP/OAuth personnelles Alice, Bruno et Eric ; Camille non classée pour le refus |
| IA réelle | Albert API : planification PDL et rédaction Incidents avec `gemma-4-31b-it` ; interprétation/relecture avec `gpt-oss-120b` ; modèles retournés et usage présents dans les résultats natifs |
| État initial | Archive publique r9, manifeste propre du commit `f5ed1279dab6d91b00533af8cd7dbee0168233bc`, 312 fichiers vérifiés |
| Correction exercée | Source candidate sur cette base, identifiée `dirty: true` ; contexte applicatif moteurs `ced86293f609c10b37ac`, différent du runtime initial `03763e37a65820d831cc` |
| Remise corrigée | r10 : sources propres et documentation mises à jour ; égalité du contexte applicatif avec la candidate vérifiée séparément à la remise |

Le serveur a été préparé avant l’étape 1. Une erreur de premier démarrage Lima
(`systemd-logind` et activation de la session persistante) a nécessité une mise
à jour des paquets système Ubuntu et un reboot. Ensuite, système `running`,
cloud-init terminé sans erreur, Docker 29.10.0 et Compose 5.8.0. Il s’agit d’un
incident de préparation d’hôte, consigné sans l’attribuer au PoC.

Les accès privés ont été transférés uniquement vers la VM de répétition avec
l’autorisation explicite de leur propriétaire. Aucun secret n’est inclus dans
ce document, les captures publiées ou les paquets. Les services de développement
d’origine ont conservé leurs images et leurs conteneurs. Leurs réglages clients
et droits de lecture du modèle relais ont été restaurés après les essais.

## Résultats par étape du guide

| Étape | Exécution et résultat |
| --- | --- |
| 1 — Identifier le serveur | Compte, OS, architecture, Python, sudo et dossier de raccordement vérifiés sur le serveur neuf |
| 2 — Vérifier le paquet | Archive téléchargée depuis GitHub sans session privée ; SHA-256 et manifeste de tous les fichiers vérifiés avant extraction/utilisation |
| 3 — Choisir le mode | Destination neuve, `external`, administratrice LDAP fictive habilitée ; aucun compte SI créé par le PoC |
| 4 — Prérequis | Client/moteur Docker, Compose, Buildx et espace disque contrôlés ; services sources gérés sur l’autre serveur |
| 5 — Préparer | Instance créée sans service ; NiceGUI activé ; droits de dossiers `700`, configuration `600` |
| 6 — Raccordement | Assistant NiceGUI rempli champ par champ dans le navigateur, cinq fichiers chargés, contrôle et ZIP réellement produits puis importés : 10 contrôles réussis après correction de la provenance du snapshot |
| 7 — Images et démarrage | Validation avant arrêt, images approuvées récupérées par digest sur la registry TLS authentifiée, démarrage des seuls trois services du PoC |
| 8 — Mappings | Découverte, import de la proposition, examen et approbation séparés des quatre revues dans NiceGUI : 15 contrôles ; aucun catalogue actif copié de l’ancienne instance |
| 9 — ITSM | Deux credentials individuels associés à des sujets LDAP stables, App-Token privé, raccordement natif 1.6.17 ; contrôle métier `ready` avec historique disponible |
| 10 — Clients | Vérification de l’adaptateur existant, configuration des deux modèles, accès au relais accordé aux seuls comptes de recette, connexion ordinaire et deux autorisations personnelles OAuth par utilisateur ; aucun jeton personnel à copier |
| 11 — Deux UC et ACL | Questions exactes du guide dans Open WebUI, comparaison NiceGUI, refus et rétablissements décrits ci-dessous |
| 12 — Exploitation | État, contrôles, journaux, stockage, arrêt/sauvegarde/reprise puis restauration réelle dans une autre destination : 12 contrôles réussis, dont refus d’une sauvegarde corrompue avant création de cible |

Les commandes du guide ont été enregistrées et exécutées depuis les sources
extraites. Les formulaires ont réellement été utilisés. Pour les fichiers
normalement remplis avec un éditeur, l’automatisation a saisi les mêmes valeurs
validées de la fiche. Les résultats, captures et traces restent dans les preuves
privées. Dans **NiceGUI → Qualification**, les rapports sont séparés :
`deploy-rehearsal-20261118` pour les reprises corrigées,
`deploy-rehearsal-initial-20261118` pour les constats initiaux et
`deploy-regressions-20261118` pour les suites déterministes.

## Conversations et contrôles d’accès

| Parcours | Preuve obtenue |
| --- | --- |
| PDL, 5 messages du guide | Clarification sans lecture, reprise donnant deux refus PFS ; six documents des quatre familles ; restriction TACACS donnant deux événements IPv4/IPv6 ; exclusion TACACS donnant quatre documents ; planificateur Albert réel |
| Incidents, 4 messages | Diagnostic réel avec rédaction et relecture Albert ; cinq mesures de 120 points et 120 logs ; suivi à midi : alerte résolue à 11:30, ticket pas encore clôturé (12:15) ; nouveau fil incomplet puis reprise sur la bonne fenêtre |
| ACL, 3 profils et 9 messages | Périmètre exact de chaque titulaire ; TACACS autorisé selon la politique ; diagnostic interne autorisé et externe refusé sans preuves interdites |
| Deux cycles Interdit/rétablissement | Mesures puis tickets : 12 messages supplémentaires dans les mêmes fils, avec les mêmes autorisations ; refus de tous, administrateur inclus ; nouvelles preuves après rétablissement pour les seuls internes ; 23 contrôles, révision 1 → 5, règles initiales rétablies |
| Administration NiceGUI | Refus lecteur et externe pour ACL, mappings et qualification ; connexion non classée refusée : 7 contrôles |
| Comparaison NiceGUI | Catalogue PDL des quatre familles, recherche PFS donnant deux refus avec Albert ; diagnostic Incidents réel dont mesures, logs et deux horloges concordent avec Open WebUI |
| Après restauration | Deux UC avec Albert et refus externe rejoués : 7 contrôles sur 3 messages ; autorisations OAuth conservées, aucune nouvelle connexion aux sources nécessaire |

Un appel de métadonnées ou un refus ne constitue pas une génération. Le suivi
fermé à midi relit les faits sans IA ; les nouveaux diagnostics appellent
réellement Albert. Les traces gardent les modèles retournés et leur consommation.
La limitation IA reste active : aucune baisse du délai ni contournement de quota
n’a servi à faire passer la recette.

La liste de tous les index autorisés contient **six index pour un interne**
(quatre PDL, logs Incidents et documentation) et quatre pour l’externe selon la
politique d’essai. Le catalogue PDL approuvé affiche ses quatre familles. Ces
listes ont des rôles distincts et leurs ensembles exacts ont été contrôlés.

## Écarts rencontrés, corrections et reprises

| Écart initial | Correction et vérification |
| --- | --- |
| Snapshot courant ITSM sans provenance : refus du formulaire | Le fichier fourni dans la nouvelle VM a reçu une provenance fictive explicite ; les quatre tickets sont inchangés et l’original est conservé. Le contrat de fichier est précisé dans la fiche et la référence de configuration. Assistant/import rejoués. |
| Procédure imposant une reconstruction du client déjà conforme | Le guide permet de conserver une image cliente et son adaptateur déjà approuvés ; la procédure de mise à jour reste disponible pour les autres postes. Le fichier et le hook de l’adaptateur réellement installé ont été vérifiés. |
| Modèles PoC visibles mais relais Albert privé : `Model not found` pour les utilisateurs ordinaires | Partage limité aux trois comptes de recette, sans modifier les ACL des sources ; contrôle en compte ordinaire ajouté au guide. Réglages initiaux restaurés à la fin. |
| « Cite les horodatages disponibles » faisait sortir une question de statut du chemin factuel, puis rencontrer le budget IA | Correction générique des seules clauses finales demandant sources, dates ou heures ; aucune date ni équipement codé pour ce scénario. Les questions mixtes de cause, action ou solution restent hors de ce chemin. Neuf cas de régression supplémentaires ; suivi réel rejoué. |
| Budget IA décrit seulement pour les synthèses | Le guide explique qu’il est partagé par serveur et couvre aussi interprétation et clarification. Attente entre demande de période et réponse ; après un refus de budget, renvoi du même message avant de poursuivre. Dialogue complet rejoué sans désactiver la limite. |

Les premières tentatives sont conservées avec leurs statuts. Deux erreurs
d’observation du banc ont aussi été corrigées : attendre la publication finale
Open WebUI après la fin de l’outil, et comparer les index aux droits du profil
plutôt qu’attendre quatre index pour tout le monde. Elles n’ont pas nécessité
de correction produit. Les appels réels déjà exécutés ont été relus dans
l’historique natif et le rendu navigateur, sans inventer de nouvelles exécutions.

## Sauvegarde, restauration et conservation

La sauvegarde froide du mode externe contient la configuration privée, deux
images du projet et une archive de données locales. Les bases du SI existant
ne sont pas sauvegardées par le PoC. Les **78 fichiers de données** ont les
mêmes empreintes avant démarrage de la copie restaurée, notamment catalogue,
politique, audits, jetons et base OAuth. Configuration et credentials sont
également conservés. L’instance initiale reste arrêtée, avec ses données.

Une copie volontairement altérée de la sauvegarde a été refusée avant création
de sa destination. La sauvegarde originale reste intacte et privée dans la VM ;
elle n’a pas été téléchargée sur le Mac ni publiée. Les deux UC et le refus
externe ont ensuite été testés sur l’instance restaurée.

Les **43 contrôles des sources** passent avant et après : TLS, identités,
lectures, quantités et schémas de référence, droits natifs ITSM, refus des
lectures anonymes, des mauvais accès et des écritures. Aucune donnée métier du
SI simulé n’a été modifiée par cette campagne.

## Régressions et limites de la conclusion

Les suites déterministes ont réussi **1 397 tests, avec un ignoré**, après la
correction applicative. Elles couvrent livraison, portail, PDL, Incidents, ACL
et client Albert. Les résultats hors réseau ne remplacent pas les conversations
réelles décrites ci-dessus ; ces deux preuves sont séparées dans Allure.

**Réussi après corrections :** installation manuelle du PoC en mode externe,
raccordement aux services fictifs existants, données de référence, conversations
réelles Albert des deux UC, droits individuels, refus/rétablissement et reprise
sur sauvegarde. La documentation corrigée reste à dérouler avec les valeurs du
SI réel.

**Échecs initiaux corrigés :** entrées sans provenance, accès insuffisant au
relais, suivi factuel traité comme nouveau diagnostic, consignes d’attente
insuffisantes. Les preuves de ces tentatives ne sont pas transformées en succès.

**Non qualifié par cette répétition :**

- Données, sémantique des mappings, cardinalités, rétention et droits réels du SI-RIE ; en particulier TACACS uniquement textuel, dont la limite reste décrite dans la [référence PDL](04-pdl.md#exemple-complet-illustré--des-champs-pdl-aux-accès-tacacs).
- Registry GitLab filtrée, PKI, proxy, pare-feu et URLs HTTPS publiques du SI ; les tunnels et la registry TLS de laboratoire ne les remplacent pas.
- Autres versions d’ITSM-NG, configuration de ses profils et entités, API et éventuel SSO du SI ; les droits individuels doivent être vérifiés sur place.
- Nouvelle endurance de deux heures, nouveaux plafonds de charge, performances AMD64 et compatibilité Ubuntu 26.04 des images corrigées ; les campagnes historiques gardent leurs versions et leurs réserves.
- Open WebUI d’une autre version ou sans l’adaptateur approuvé ; disponibilité, quotas et comportement des futurs modèles Albert.
- Import dans la version Wiki.js et l’éditeur draw.io réellement déployés dans le SI.

La [fiche de réception SI-RIE](14-si-rie-acceptance.md) fournit les contrôles et
responsables à renseigner. Un point non exécuté doit y rester **non qualifié**.
