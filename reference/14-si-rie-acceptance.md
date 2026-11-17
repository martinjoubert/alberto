# Raccordement SI-RIE : limites, prérequis et réception

> Complément de référence — le [guide principal](../README.md) reste le seul parcours de déploiement à exécuter.

Cette page fixe le périmètre de la remise et les vérifications restant à faire
dans le **SI-RIE réel**. Elle accompagne le développeur tiers, l'exploitant et
les responsables des sources. La qualification du laboratoire ne vaut pas
réception de cette intégration. Au moment de cette revue, aucun des contrôles
SI-RIE ci-dessous n'est déclaré exécuté avec les sources et identités réelles.

## Ce que démontre le laboratoire

De vrais services OpenSearch, VictoriaMetrics et LDAP ont été utilisés sur des
VM Ubuntu ARM64, avec équipements, comptes, documents et tickets fictifs.
Installation, lectures, contrôles d'accès, pannes locales et restauration ont
été exercés ; leurs versions et résultats sont dans le [bilan](08-qualification.md).
Les alertes et leur traitement courant restent un snapshot compatible. La
recherche de précédents utilise désormais l'API native ITSM-NG 1.6.17 du
laboratoire avec des données fictives et des credentials individuels ; voir
le [lot ITSM](15-itsm-precedents.md). Aucune connexion au SI réel n'est revendiquée.
Les tests déterministes et les épreuves Allure ne sont pas des essais dans le SI.

La [connexion utilisateur sans copie de jeton](25-user-access.md) utilise OAuth
MCP et LDAP. Avant réception dans le SI, confirmer DNS/HTTPS des deux moteurs,
CA/proxy du navigateur et du backend Open WebUI, callbacks exacts des postes,
version/image client, stockage persistant et clé locale stable. Rejouer la
première autorisation, le renouvellement, l'expiration, la révocation et les
refus des deux UC avec les véritables identités. L'existence du SSO OpenProject
ne qualifie pas à elle seule un raccordement OIDC ou SAML du PoC : fournisseur,
protocole et habilitations restent à faire confirmer au responsable identité.

## Écarts conversationnels du 11 novembre 2026

La nouvelle matrice du 12 novembre sur `runtime-5afa381bc374efc4c4c7` termine avec **96 / 96 échanges conformes et 9 / 9 fils longs conformes**. OW-01 à OW-08 sont clôturés sur cette matrice du laboratoire. Les 149 contrôles ACL sont rejoués séparément. Cette qualification ne clôture aucun prérequis ou contrôle de réception du SI réel.

Le [bilan des corrections](19-openwebui-corrections.md) identifie le candidat
courant, ses tests par lot et la nouvelle campagne Albert. La grille ci-dessous
conserve les constats initiaux et leurs critères de clôture ; ses compteurs
ne sont pas ceux des images corrigées. La réception du SI réel reste distincte.

La grille Open WebUI avec Albert a été exécutée trois fois, sur la même image
et les mêmes données fictives. **50 messages sur 96 sont conformes à tous les
contrôles ; aucun des neuf fils de dix tours ne l’est entièrement.** Dans ce passage initial, ces défauts
du laboratoire devaient être traités avant de présenter le parcours comme
stable ; ils ne dépendent pas tous du raccordement au SI réel.
Le [rapport complet et les preuves](18-openwebui-stability.md) conservent les
fréquences, questions, écarts et critères de clôture.
Le [suivi des corrections](19-openwebui-corrections.md) conserve les validations
par lot et la nouvelle campagne, sans modifier les compteurs historiques.

| ID | Constat initial et portée | Critère de clôture |
| --- | --- | --- |
| OW-01 | PDL et Incidents : certaines heures deviennent une journée entière ; 15 lectures sur une mauvaise période | Corriger les deux parseurs identiques et refuser un élargissement silencieux ; trois répétitions de chaque formulation et données contrastées hors fenêtre |
| OW-02 | Clarifications superflues, suivi R3 refusé et retour implicite à l’ancien sujet non assuré | Rejouer les dix tours en conservant équipement, période et sources ; distinguer précédents ITSM et comparaison temporelle |
| OW-03 | Comparaison Rennes : trois dépassements du budget local de relecture | Borner le contexte sans perdre les preuves ni désactiver le contrôle factuel ; comparaison à deux équipements qualifiée |
| OW-04 | Refus serveur explicite rendu comme erreur générique ; un dépassement du nombre d’appels d’outil | Rendre refus et fenêtre vide compréhensibles, avec un seul appel métier pour une demande simple |
| OW-05 | Mauvais profil : question de périmètre inadaptée ou lecture du site par défaut hors sujet | Guider avant lecture ; le routage automatique entre UC reste une décision séparée |
| OW-06 | Explication de limite de comptage non étayée, deux occurrences détectées | Faire correspondre le texte aux limites effectives et à l’opération réellement exécutée |
| OW-07 | Six synthèses exigées non validées ; un autre refus de synthèse sur un constat factuel R3 | Améliorer la rédaction et les reprises bornées ; conserver les refus légitimes et qualifier la disponibilité du diagnostic |
| OW-08 | Deux comptages conservent demo-beta au lieu du nouveau demo-alpha ; un texte prétend une correction inexistante | Priorité à la dernière demande et restitution fidèle du filtre exécuté ; données contrastées entre périmètres |

Les refus observés ne publient pas de preuves métier interdites. Cette campagne
utilise une seule identité interne et **ne requalifie pas les ACL externes**.
Les services sains et les anciennes recettes réussies ne clôturent pas les
écarts ci-dessus. Les corrections AUD-01 à AUD-03 conservent leur statut et
leurs preuves historiques ci-dessous.

## Points ouverts après l'audit du 10 novembre 2026

**Les trois défauts de code confirmés ont reçu des corrections et des régressions.** La clôture
précédente portait sur les scénarios fictifs examinés, sans preuve d'absence de
défaut dans les deux UC et les ACL. Le rapport d'audit initial et ses échecs
restent conservés. Le correctif utilise une nouvelle image ; les preuves de
sa revalidation sont distinguées de celles de l'image antérieure.

| ID | Périmètre et défaut confirmé | Reproduction et impact | État et critère de clôture |
| --- | --- | --- | --- |
| AUD-01 | Incidents / précédents ITSM : le classement réinterprétait les dates naïves en Europe/Paris, même si le lecteur natif était configuré en UTC | Une résolution native à 09 h 30 UTC devenait 07 h 30 UTC ; une solution créée à 10 h 30 UTC pouvait être retenue avant une fenêtre débutant à 10 h UTC | Corrigé : fuseau transmis par le lecteur, résolution et dates des solutions normalisées en UTC ; UTC, Paris, New York, offsets explicites et transitions d'heure testés. Les solutions postérieures sont exclues |
| AUD-02 | Incidents / transport HTTP OpenSearch : le délai par lecture ne garantissait pas le budget total | Un serveur local délivrant 41 octets progressivement terminait avec succès en environ 0,57 s pour un budget configuré de 0,12 s | Corrigé : annulation asynchrone pendant connexion, en-têtes et corps. Le rejeu s'arrête vers 0,12 s avec `deadline_exceeded`. Réponses bornées à 2 Mo, fermeture vérifiée et erreurs de source préservées |
| AUD-03 | Incidents / contrôle de la synthèse : traitement particulier de `uplink0` | Une autre interface documentée comme `uplink7` était rejetée comme quantité non vérifiée | Corrigé : identifiants issus des champs structurés ou des mentions explicites d'interface/port des preuves autorisées. Plusieurs formats passent ; interfaces inconnues, correspondances partielles et mesures inventées restent refusées |

L'audit initial conserve quatre échecs et un contrôle positif sous
`.runtime/ci/open-items-20261110/`, campagne Allure
`audit-questions-ouvertes-20261110`. Les cinq contre-exemples rejoués après
correction réussissent sous `.runtime/ci/fix-aud-20261110/audit-replay/`.
Le rejeu transporte la métadonnée de fuseau désormais émise par le lecteur ;
une régression publique vérifie aussi la chaîne configuration UTC, lecture
native simulée, puis classement. Le contrôle HTTP utilise un vrai petit serveur
sur 127.0.0.1. Les doubles de génération isolent le garde-fou et ne valent pas
validation Albert.

Trente-sept nouvelles régressions Incidents sont intégrées à la CI. Les suites
complètes exécutées après correction comptent **1 151 réussites et un saut Linux
sur Mac** : 480 source/opérateur/ACL/client/CI, 163 PDL et 508 Incidents.
Les reçus sont dans `.runtime/ci/fix-aud-20261110/suites-r1/`. La revalidation
Ubuntu utilise l'image du correctif et distingue les contrôles de disponibilité,
les douze contrôles techniques et les conversations Albert effectivement rejouées.

Sur Ubuntu 24.04 ARM64, l'image corrigée
`dinum/rie-delivery:runtime-f94c456528fe0e67adbc` passe les contrôles de
disponibilité et les douze contrôles techniques. Trois parcours frais avec
Albert réussissent : Incidents dans NiceGUI et Open WebUI, puis PDL dans
Open WebUI. La rédaction Incidents utilise Gemma 4, sa relecture et le dialogue
GPT-OSS ; PDL conserve GPT-OSS. Les modèles retournés et les usages sont conservés.

Un contrôle séparé de génération réelle réussit avec `GigabitEthernet0/1` sur
une copie explicitement adaptée des preuves fictives. Ce contexte figé n'est
ni une nouvelle collecte ni un scénario MCP utilisateur. Les données déployées
ne sont pas réécrites pour ce contrôle, et les limites des interfaces métier
restent actives. Les fichiers de preuve sont sous `.runtime/ci/fix-aud-20261110/`.
La campagne Allure du correctif est `corrections-aud-20261110` ; le rapport initial
en échec reste accessible séparément.

### Fonctionnalités disponibles et extensions conditionnelles

La revue n'a pas confirmé de nouveau défaut PDL ni de fuite ACL dans son
périmètre ; cela ne prouve pas leur absence générale. Le PDL conserve ses
quatre familles, mappings approuvés et lectures bornées. Les ACL des deux UC
couvrent les index et ressources, les identités individuelles et les trois
visibilités. Les droits natifs ITSM restent opposables.

| Point à décider selon le besoin du SI | État réel et conséquence |
| --- | --- |
| Alertes et traitement courants depuis une API ITSM réelle | Non livrés par les quatre US de recherche de précédents. La chronologie utilise un snapshot ; une alimentation automatisée ou un connecteur courant demande un travail distinct si exigé au déploiement |
| ACL par équipement, site, tenant ou période | Non implémentées dans les sources standard. L'inventaire global n'est pas une règle individuelle ; développement et recette nécessaires si cette granularité est demandée |
| Compteurs, `rate`, CPU par cœur, plusieurs interfaces ou séries à agréger | Le contrat Incidents attend des jauges et une série exacte. Prétraitement amont ou adaptation du lecteur si les sources réelles diffèrent ; ce n'est pas garanti par un changement de nom de métrique |
| Schémas PDL et texte brut du SI | Mappings et cas représentatifs à valider ; aucune adaptation universelle automatique au texte brut ou aux nouveaux sens métier n'est livrée |
| Ancienne réserve SEC-01 de lecture Victoria via Grafana | Réserve historique distincte : Grafana est absent de la livraison Ubuntu actuelle. Sa réactivation exige de traiter ou de requalifier le filtrage côté source ; elle n'est pas déclarée corrigée par les tests MCP |

Les réserves de migration, Ollama, Wiki.js/OpenProject et anciens parcours du
backlog du 12 octobre (`docs/handover/backlog.md` dans le dépôt) conservent leur date et leur
périmètre. Elles ne sont ni toutes des bugs de la livraison Ubuntu, ni toutes
clôturées par les nouveaux essais. L'observabilité, Dify et n8n restent hors
périmètre de cette remise.

### Validations de laboratoire et réception encore ouverte

La [matrice Albert du 12 novembre](19-openwebui-corrections.md) couvre désormais
96 échanges, trois répétitions et neuf fils longs, avec changement de sujet,
clarification, reprise et refus. Elle clôture les écarts OW-01 à OW-08 sur les
données fictives testées. Les affirmations injectées au relecteur restent une
épreuve distincte de ces conversations intégralement générées.

La [remise du 13 novembre](20-final-ubuntu-qualification.md) identifie les
nouvelles preuves d'installation neuve 24.04, de compatibilité 26.04 et
d'exploitation : sauvegarde/restauration, fichiers et identités ITSM, pannes,
redémarrage Docker et reboot Ubuntu. La CI distante du candidat termine ses
sept jobs avec succès. L'endurance de cette campagne dure 7 200,087 secondes :
8 459 demandes, zéro erreur inattendue, 810 refus attendus et zéro violation de
cadence, avec corpus intégralement préservé. La couverture du journal d'accès
ITSM reste partielle après rotation ; elle est déclarée non qualifiée. Les
résultats historiques ci-dessous conservent leurs versions et leurs limites.

Dans le SI réel, la grille conversationnelle doit être rejouée sur les sources,
formulations et droits des hyperviseurs, avec la clé Albert effectivement
autorisée. Le navigateur habituel, la machine AMD64, les schémas, les profils
natifs, la volumétrie et les sources réelles restent à recevoir. Les succès du
laboratoire ne clôturent aucun contrôle de réception du SI réel.

L'endurance historique v6 porte sur 100 000 événements PDL et 100 000 logs
Incidents, largement construits par répétition du petit corpus de référence.
Elle ne reproduit pas la diversité des schémas, valeurs, équipements, shards,
droits natifs ou charges concurrentes du SI-RIE. Les métriques comportent vingt
séries et 4 200 points ; les fiches documentaires sont peu nombreuses.

| Preuve acquise | Portée exacte |
| --- | --- |
| Endurance historique de deux heures | 8 460 demandes, zéro erreur inattendue, 810 refus ACL attendus ; fournisseur PDL fictif et Incidents sans synthèse ; six écarts de cadence aux transitions restent documentés dans le bilan |
| Palier historique de dix utilisateurs | Environ deux demandes/s au total ; p95 mélangé PDL/Incidents/refus de 152,970 ms, réseau local et requêtes répétées |
| Runtime minimal C2, avant les corrections ci-dessous | Contrôle court de 60,078 s et 73 demandes ; les anciennes deux heures ne sont pas réattribuées aux nouvelles images |
| Conversations Albert historiques | Parcours et reprise ciblée documentés ; pas de qualification de la charge IA réelle sur les nouvelles images |
| Images et machine cible du SI | À identifier et qualifier ; les mesures ARM64 du MacBook ne dimensionnent pas un serveur AMD64 |

Ces valeurs ne constituent ni un SLA ni une promesse de dix utilisateurs en
conversation IA simultanée. Le test était fermé : chaque client attendait sa
réponse, sans recherche du point de saturation ni mesure de file serveur.

## Répartition des contributions

| Acteur à nommer avant le raccordement | Contribution attendue |
| --- | --- |
| Responsable de la réception SI-RIE | Arrêter le périmètre, les utilisateurs, les seuils de service et les décisions sur les écarts |
| Développeur tiers / mainteneur des UC | Installer la version identifiée, vérifier les contrats, adapter ce qui diffère et fournir les preuves techniques |
| Responsables OpenSearch, supervision et ticketing | Fournir interfaces, droits lecteurs, schémas, unités, cadence, inventaire et échantillons autorisés |
| Référent métier PDL / réseau / incidents | Fournir les résultats attendus et valider le sens des mappings, rapprochements et diagnostics |
| Exploitant, équipes réseau et sécurité | Fournir machine, flux, DNS, CA, LDAP, règles d'accès, sauvegarde et conditions des essais |
| Responsable Albert / fournisseur IA | Confirmer modèle, API, accès autorisé, quotas, délais et règles de consommation |

Le déployeur ne peut pas déduire seul le sens des données, les droits à accorder
ou les capacités du SI depuis les fixtures. Une donnée ou une habilitation
manquante est inscrite comme prérequis non fourni ; elle n'est pas remplacée
par une hypothèse silencieuse.

## Écarts concrets à lever ou à accepter explicitement

Les défauts de la revue précédente ont reçu des corrections ciblées ; les
trois réserves AUD-01 à AUD-03 ci-dessus disposent désormais de corrections
et de régressions. Leur portée de recette ne remplace pas les contrôles du SI réel.
Leurs régressions et le rejeu Ubuntu sont identifiés dans le
[bilan de qualification](08-qualification.md). Une correction de code ne vaut
pas réception d'un raccordement : les vérifications de la dernière colonne
restent à exécuter avec les données, identités et services réels.

| ID | Correction dans la nouvelle livraison | Vérification restant à faire dans le SI-RIE |
| --- | --- | --- |
| E01 | Modèle de manifeste complet et validation partagée de la fenêtre, de l'inventaire et des cadences avant démarrage | Fournir une fenêtre réelle contenant les données ; vérifier statut et lectures. Retirer les marqueurs ne suffit pas |
| E02 | La cadence configurée, éventuellement par équipement/métrique, est transmise aux calculs courants et de référence | Confirmer la cadence réelle et comparer couverture/chiffres avec les outils de supervision |
| E03 | `ldap.administrator_group_dns` sépare l'administration des lecteurs internes ; rôle relu au LDAP, absence de groupe = aucun admin implicite | Faire décider les administrateurs, adapter les anciennes instances et tester le refus des lecteurs avec le LDAP réel |
| E04 | Une admission commune couvre HTTP, MCP historique et assistant avant leurs appels IA ; elle couvre le tour entier | Définir intervalle, routes accessibles et quotas fournisseur ; tester plusieurs utilisateurs. Budget par processus, pas une limitation distribuée de tokens/requêtes |
| E05 | Le lecteur VictoriaMetrics refuse avant connexion si le budget est épuisé et borne ouverture/lecture par le temps restant | Injecter les délais du réseau réel et vérifier la durée du diagnostic entier ; la correction de ce lecteur n'est pas une borne bout en bout de tous les connecteurs |
| E06 | Champs logs/documents configurables avec normalisation interne ; bindings et labels propres à chaque métrique ; identifiants documentaires bornés moins restrictifs | Confirmer types et sens métier. Une seule série gauge exacte est attendue ; compteurs nécessitant `rate`, CPU par cœur, `nested` ou interfaces à agréger demandent normalisation amont ou adaptation. La chronologie courante reste un snapshot ; l'API native concerne les précédents historiques |
| E07 | Limites de découverte explicites, lecture d'identité séparée des logs affichés, contexte IA réduit aux champs approuvés et disponibilité par motif requis | Calibrer rollover, schémas, identité bavarde et fenêtre du modèle sur des cas représentatifs. Le scan d'identité reste borné, avec refus explicite au-delà ; aucun support illimité ni adaptation automatique |
| E08 | Modèles Incidents configurables par preset ; `gptoss` reprend `ALBERT_MODEL` et l'interprétation suit le preset sauf surcharge | Vérifier le modèle demandé/renvoyé et les réponses structurées avec le fournisseur réel ; un changement d'alias ne qualifie pas une IA interne |
| E09 | Sous-ensemble PDL optionnel protégé par identifiants exacts d'hôtes Zabbix et dashboards Grafana, avec projection des résultats ; les autres opérations restent refusées | Connecteurs désactivés dans Ubuntu. S'ils sont demandés, renseigner les règles par objet, contrôler les droits natifs et qualifier les opérations précises ; pas de compatibilité générale annoncée |

Exemple de E02 : dix points sur dix minutes représentent 100 % d'une collecte
à 60 s, mais seulement 16,67 % d'une collecte attendue à 10 s. Le calcul applique
maintenant la cadence déclarée ; la justesse de cette déclaration reste une
responsabilité de raccordement. Une source joignable ne prouve pas son sens métier.

Repères pour le mainteneur, chemins conservés dans la remise :
`obs_poc/domain.py` (`aggregate`), `obs_poc/service.py`, `obs_poc/server.py`,
`obs_poc/assistant.py`, `obs_poc/telemetry.py`,
`obs_poc/clients.py`, `obs_poc/opensearch_links.py`,
`core/access-control/src/dinum_access/security.py`,
`rie_agent/inventory.py`, `rie_agent/equipment.py` et `rie_agent/planner.py`.
Les dossiers des deux moteurs sont indiqués dans l'[architecture](03-architecture.md).

## Matrice de vérification dans le SI-RIE

Pour chaque ID, renseigner le résultat, la date, les acteurs et la preuve dans
le compte rendu ci-dessous. L'état initial est **non fait dans le SI-RIE**.
Nommer un responsable unique par ID ; les autres acteurs sont contributeurs.
Les tests de panne et de charge sont planifiés avec les responsables des sources.

| ID | Vérification et prérequis | Résultat attendu / preuve | Acteurs principaux |
| --- | --- | --- | --- |
| S01 | Version exacte, Ubuntu, architecture, CPU/RAM/disque, droits Docker et prérequis du tiers | Manifeste et empreintes, configuration effective, installation neuve rejouée par le tiers ; écarts au laboratoire consignés | Déployeur + exploitant |
| S02 | Accès aux sources du code, APT, registres et paquets Python, ou miroirs/paquet hors ligne préparés | Construction et démarrage possibles depuis le réseau cible ; aucune dépendance implicite au Mac ou à une sortie Internet non autorisée | Exploitant + réseau + déployeur |
| S03 | DNS, routage, pare-feu, certificats, CA et mécanismes d'authentification de chaque source et fournisseur IA | Lectures réelles depuis les conteneurs, puis refus 401/403, CA invalide et délai correctement distingués ; matrice de flux conservée sans secrets | Réseau + sécurité + propriétaires des sources |
| S04 | Proxy obligatoire, éventuel mTLS, SSO ou authentification différente de celle des exemples | Compatibilité démontrée ou adaptation identifiée. Plusieurs clients utilisent `trust_env=False` : `HTTPS_PROXY` seul ne démontre pas le raccordement | Développeur + réseau + sécurité |
| S05 | LDAP réel : attribut stable, DN, groupes, comptes internes/externes/non classés, éventuels groupes imbriqués | Identité unique et fraîche ; double appartenance, changement de groupe, objet recréé, expiration/révocation et annuaire indisponible testés ; rôles d'administration décidés selon E03 | Administrateur LDAP + sécurité + déployeur |
| S06 | Droits applicatifs et natifs des deux UC, API et lecteurs web ; besoin éventuel par équipement/tenant ; données autorisées dans les prompts et rapports | Refus sans lecture ni preuve interdite, isolation des utilisateurs et des reprises, droits après changement ; périmètre **par index/source** accepté ou granularité complémentaire réalisée ; extraits transmis à l'IA/exportés relus selon les règles du SI | Sécurité + propriétaires des sources + développeur |
| P01 | Noms physiques, alias, rollover, rétention, mappings et `_field_caps` PDL réels | Toutes les familles prévues sont découvertes ; contraintes E07 satisfaites ou adaptées ; pas de succès fondé sur un seul index actif | Administrateur OpenSearch + mainteneur |
| P02 | Échantillons PFS/Backbone/PFAING/TACACS représentatifs, valeurs d'actions, types, dates, champs manquants/imbriqués et messages bruts | Recherches, comptages et agrégations comparés aux requêtes manuelles ; mappings approuvés par le métier ; les données de `event.original` ne sont pas supposées automatiquement normalisées. TACACS dont le seul lien est textuel : fournir un champ IP structuré ou conserver le flux non qualifié ; voir la [limite exercée](04-pdl.md#7-cas-avancé-et-limite-actuelle--tacacs-ne-fournit-quun-message-texte) | Expert métier + ingestion + développeur |
| P03 | Relations entre équipements, IP de gestion/trafic, IPv6, alias, doublons, IP partagées et plus de 50 correspondances d'identité | Résultat manuel retrouvé pour les cas résolubles ; refus pour une ambiguïté réelle ou un accès interdit. Un cas métier requis bloqué par une limite technique reste un écart à résoudre ou une limitation explicitement acceptée | Expert réseau + mainteneur PDL |
| P04 | Changement de type, ajout/suppression/rotation d'index et changement de valeurs métier sans changement de schéma | Révision/empreinte et nouvelle approbation lorsque nécessaire ; surveillance des valeurs métier distincte de la dérive de types. Le refresh est d'une heure par défaut | Exploitant + administrateur OpenSearch + métier |
| I01 | Inventaire Incidents, équipements/interfaces, alias, portée, période de contrôle et propriétaire de leur actualisation | Inventaire réel relu, pas d'équipement fictif ; nouvelles observations retrouvées sur une fenêtre récente. Le statut ne doit pas seulement sonder une ancienne fenêtre de fixture | Supervision + exploitant + développeur |
| I02 | Champs et identifiants des logs et documents, dates, types, droits, références et fiches réellement disponibles | Au moins un log et une fiche connus retrouvés ; recherche vide expliquée, citations ouvrables avec l'identité de l'utilisateur ; contrat E06 respecté | OpenSearch / documentation + métier |
| I03 | Les **cinq** séries CPU/latence/pertes/entrées/sorties, labels communs et unités ; jauges ou compteurs ; cadence et trous de collecte | Calculs rapprochés d'un export de référence, une série exacte sélectionnée ou lecteur adapté ; unités et couverture correctes ; E02 résolu, séries absentes/non finies/ambiguës testées | Supervision + expert réseau + développeur |
| I04 | Snapshot ticketing ou API réelle attendue, format, complétude et fréquence d'actualisation | Provenance/fraîcheur visibles, ticket réel connu retrouvé ; résolution technique distincte de la clôture administrative ; absence de ticket non assimilée à un export incomplet | Responsable ITSM + métier + développeur |
| I05 | Incidents connus, cas sains, données manquantes, périodes de référence et périmètres limites | Faits et diagnostic comparés à la référence métier ; hypothèses distinguées des causes démontrées ; fenêtres de sept jours et dix équipements maximum acceptées ou adaptées | Expert réseau / incidents + développeur |
| I06 | Précédents via l’API ITSM-NG 1.6.17 : client API activé, comptes individuels, profils/entités natifs, contrat de recherche, tickets et solutions connus | Après `incidents_itsm-ng-history: ok`, retrouver un ticket résolu/clos antérieur au début de l’incident, sa solution et les suivis autorisés ; comparer à l’interface native avec deux comptes de droits différents, tester suivi privé, absence de précédent et révocation. Une sonde complète mais vide ne clôture pas ce contrôle ; chronologie courante toujours traitée séparément en I04 | Responsable ITSM + sécurité + métier + déployeur |
| A01 | Modèle réellement accessible, API, JSON structuré/appels d'outils, taille du contexte, quotas, délais et contenus contradictoires ou porteurs d'instructions | Conversations réelles, clarification et suivi dans les deux UC ; chiffres/citations et refus conservés malgré un contenu trompeur ; nombre d'appels/tokens et erreurs 429 mesurés ; E04/E05/E08 résolus sur le périmètre retenu | Responsable IA + développeur + métier |
| U01 | Open WebUI installé sur chaque poste, version, filtres/presets, tunnel et connexion **depuis son backend** | Deux utilisateurs distincts jouent les deux UC ; appels MCP vérifiables, citations correctes et reprise isolée ; accès du navigateur seul insuffisant, aucune adresse frontend centrale nécessaire | Déployeur client + utilisateurs pilotes |
| C01 | Taille et diversité réalistes, cardinalités, shards/répliques, charge déjà présente et mélange de questions simples/complexes | Corpus autorisé et profil documentés ; résultats par UC et type de requête, sans mélanger refus rapides et diagnostics longs dans une seule promesse | Responsables données + métier + exploitant |
| C02 | Utilisateurs, débit, rafales, concurrence IA, démarrage à froid et seuils de latence décidés avant l'essai | Charge progressive puis pics, caches froids/chauds ; p50/p95/p99, débit, attentes/queues, 429, erreurs, CPU/RAM/disque ; point de saturation et admission connus ; aucun objectif déduit des 153 ms du laboratoire | Exploitant + développeur + responsable IA |
| C03 | Endurance sur **images finales et machine cible**, avec données et parcours représentatifs | Durée et seuils convenus, au moins la campagne de deux heures à rejouer ; aucun OOM, redémarrage spontané, fuite ACL ou perte ; données comparées sans réensemencement | Exploitant + développeur |
| X01 | Pannes et lenteurs réseau, LDAP, OpenSearch, VictoriaMetrics, ticketing et IA ; réponses partielles, shards défaillants | Refus fermé, absence de preuve périmée, erreurs interprétables et reprise vérifiée ; temps de rétablissement mesurés dans le SI | Exploitant + réseau + propriétaires + développeur |
| X02 | Croissance des rapports/audits/états, saturation des capacités de conversation, disque et archivage | Capacité/rétention et seuils disque choisis ; reprises des deux UC, expiration et propriétaire vérifiés sous charge ; pas de purge implicite | Exploitant + développeur |
| X03 | Sauvegarde froide, restauration neuve, reboot, mise à jour/retour arrière et révocations post-sauvegarde | Catalogue/politiques/audits/jetons comparés ; révocations réappliquées avant réouverture ; RPO/RTO mesurés. En `external`, la sauvegarde applicative n'inclut pas les données des sources : leurs propriétaires conservent leur propre procédure | Exploitant + sécurité + propriétaires des sources |
| X04 | Journaux, rapports Allure, accès opérateur et procédure d'escalade avant observabilité complète | Un échec réel de recette est diagnostiquable et attribué à une étape/source ; preuve expurgée exportable ; interlocuteur et procédure d'incident identifiés | Exploitant + déployeur |

`check`, `/healthz`, `/readiness` et un HTTP 200 restent utiles mais insuffisants
pour cette matrice. Le PDL exige un index résolu avec mapping approuvé pour
chaque motif requis ; un motif trop large peut être satisfait par un seul index
et ne garantit pas tous les rollovers. La sonde Incidents ne vérifie pas à elle seule les cinq séries et les résultats métier.
Un cas positif connu doit passer sur chaque source et chaque famille prévue.
Un refus sûr prouve une protection ; il ne valide pas à lui seul l'utilité d'un
cas métier qui devait pouvoir aboutir.
`acceptance` est réservé à `demo` : ne pas l'utiliser ou en retirer la protection
pour faire une recette sur les sources réelles. Construire les lectures de
référence avec leurs propriétaires, sans seed dans le SI.

## Compte rendu à compléter, sans résultat prérempli

Renseigner avant la séance : archive/manifeste `________`, commit et état des
sources `________`, empreinte de configuration hors secrets `________`, date
`________`, machine/architecture `________`, responsables nommés `________`,
sources et versions `________`, modèle IA `________`, périmètre d'utilisateurs
`________`, débit et latences visés `________`, fenêtre autorisée `________`.

Pour chaque ID E/S/P/I/A/U/C/X, recopier une ligne :

| ID et périmètre exact | Résultat : réussi / échoué / non fait / non applicable motivé | Preuve datée, version et résultat attendu/obtenu | Écart, action, responsable et échéance | Décision du responsable de réception |
| --- | --- | --- | --- | --- |
| À compléter | | | | |

Un résultat non fait reste non qualifié. Une exception ou limitation acceptée
est écrite avec son périmètre et sa décision ; elle ne devient pas un test
réussi. Toute exigence supplémentaire est enregistrée avec son impact sur les
données, le code, la charge, les délais et les preuves à rejouer.

La réception distingue **installation technique**, **raccordements et résultats
métier**, **charge/exploitation**, puis **génération IA**. Si Albert manque,
la recette technique peut avancer ; le parcours conversationnel complet reste
non qualifié. Une extension du périmètre, des sources, des droits ou du modèle
demande la recette correspondante. Voir aussi la [fiche humaine](12-manual-acceptance.md)
et la [conservation des rapports](13-test-reports.md).
