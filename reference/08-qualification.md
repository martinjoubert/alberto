# Qualification : résultats, scénarios et limites

> Complément de référence — le [guide principal](../README.md) reste le seul parcours de déploiement à exécuter.

Cette page distingue les résultats, les versions et les essais encore en cours.
La recette manuelle du 9 novembre complète les campagnes des 6 et 7 novembre.
Chaque campagne conserve ses sources, images et manifestes ; l'ancienne recette
Colima garde aussi sa propre configuration.

## Recette externe, ACL et restauration — 16 novembre 2026

La préparation du 17 novembre remet le guide en douze étapes au format SI réel
et ajoute la fiche de raccordement à compléter. Les **1 364 non-régressions**
passent, avec un seul contrôle Linux ignoré sur Mac. Le contenu du contexte
runtime reste `03763e37a65820d831cc`, identique au moteur éprouvé ci-dessous ;
les documents et l'image du portail évoluent séparément. Le reçu de livraison
identifie le commit figé, les nouvelles archives et leurs empreintes. Les
preuves du 16 novembre gardent leurs versions et ne sont pas antidatées sur
ces nouvelles archives ; aucune nouvelle endurance n'est revendiquée.

La campagne Allure `sandbox-acl-operations-20261116`, consultable dans la
qualification NiceGUI de la sandbox, comporte **112 résultats réussis, zéro
échec** et trois périmètres explicitement non qualifiés. Ces résultats
comprennent 19 contrôles ACL par navigateur, 14 contrôles d'exploitation,
17 contrôles après restauration, 13 observations HTTP/MCP/santé, 43 contrôles
des sources, un cas PDL externe réel et cinq captures/bilans. Ce total n'est
pas une nouvelle suite de 112 tests unitaires.

Les quatre identités ont été vérifiées séparément. Bruno conserve ses droits
natifs ITSM ; Eric lit PDL mais reste refusé sur Incidents ; le compte non
classé est refusé. Les deux ressources mesures/tickets sont interdites puis
rétablies dans NiceGUI, sans preuves réutilisées lors du refus ; révision finale
**5**, règles initiales rétablies. Les lectures directes de tickets et readiness,
ainsi que les en-têtes d'identité usurpés, restent refusés.

La sauvegarde à froid, le redémarrage, le refus d'une copie corrompue et la
restauration dans une instance neuve passent. Les **53 fichiers** du volume
sont identiques avant démarrage, y compris audits et base OAuth. Mappings,
configuration privée, jeton personnel et autorisations OAuth sont conservés.
Les deux UC sont rejoués avec **Albert réel dans Open WebUI, compte ordinaire** :
PDL compte deux refus, Incidents rédige et relit un diagnostic validé avec
précédents natifs et chronologie distincte. Les sources externes restent intactes.

Cette preuve porte sur Ubuntu 24.04 ARM64, runtime
`dinum/rie-delivery:runtime-03763e37a65820d831cc`, portail
`dinum/rie-portal:322d5ce0dcf83e693e48` et paquet source historique
`rie-si-rie-ubuntu-2026-11-16-user-access-r8.tar.gz`, SHA-256
`d4a4a9bb05b8cd6ea5b88cc66dc7f1b34d39bf71f5acaa1034f7d9bfc19d99b7`.
L'archive Allure exportée a l'empreinte
`b197651bb26d8856b35eb338ba2277936448f64f12a28c49b7f159d4fdf1a5f8`.
Le manifeste de sauvegarde privé a l'empreinte
`f0340c8b7d3b15fbe8b54fda4fc62baf7f831df14b7d633b10feb6b5d198ba10`.

L'approbation métier humaine, la réception du SI réel et une nouvelle endurance
de deux heures/mesure AMD64 ne sont pas déduites de cette campagne. Les anciens
paquets et preuves restent conservés. Le [guide remis au tiers](../README.md)
reprend ces opérations avec les paramètres réels à faire compléter dans la
[fiche de raccordement](26-si-connection-sheet.md).

## Accès utilisateur sans copie de jeton — 16 novembre 2026

La [qualification LDAP et OAuth](25-user-access.md#qualification-du-16-novembre-2026)
réunit 1 362 non-régressions réussies, quatre autorisations natives et quatre
échanges par navigateur avec Albert/ACL. Renouvellement après expiration,
concurrence, panne/reprise, redémarrage client et révocation individuelle
passent sur l'instance dédiée. Une restauration neuve conserve les 45 fichiers
du runtime et les autorisations/révocations, vérifiées depuis les clients.
La répétition de la configuration préserve les sessions actives.

Le runtime est `runtime-03763e37a65820d831cc`. Le bilan précise aussi les
échecs intermédiaires du formulaire et d'un collecteur auxiliaire ayant causé
un OOM sur un client isolé ; ils ne sont pas effacés des preuves. Le parcours
courant utilise les routes natives HTTP et le navigateur. Les deux heures
d'endurance historiques, la compatibilité 26.04 et le SSO du SI ne sont pas
automatiquement attribués à ce nouveau parcours.

## Clôture des réserves et limite de mesure — 14 novembre 2026

Le [bilan du 14 novembre](21-evidence-reservations.md) vérifie l'import du
véritable rapport CI et les sept captures continues sur deux heures :
8 460 demandes sans erreur métier, 810 refus ACL attendus et corpus intact.
Le verdict global initial reste en échec pour un comptage de stockage interrompu
par l'expiration d'un cache. Le collecteur corrigé passe ses régressions et un
smoke distinct, sans nouvelle campagne de deux heures. Les résultats et cette
limite restent explicitement séparés.

## Remise Ubuntu et CI GitLab — 13 novembre 2026

La [qualification de la remise du 13 novembre](20-final-ubuntu-qualification.md)
rassemble les nouvelles preuves sur `runtime-5afa381bc374efc4c4c7` : installation
24.04 neuve, sauvegarde/restauration avec configuration ITSM, pannes, reprise
Docker et Ubuntu, ainsi que compatibilité 26.04 et parcours NiceGUI. La CI
GitLab distante termine ses sept jobs avec succès. La nouvelle endurance dure
7 200,087 secondes : 8 459 demandes, zéro erreur inattendue, 810 refus attendus,
zéro violation de cadence et corpus intégralement préservé. La couverture du
journal d'accès ITSM reste partielle après rotation ; ce point et l'import
de l'artifact CI privé sont consignés séparément dans le bilan.

Les conversations Albert du 12 novembre et les campagnes historiques ci-dessous
restent identifiées séparément. La charge utilise un fournisseur PDL fictif et
Incidents sans synthèse ; elle ne mesure pas dix conversations Albert simultanées.

## Conversations longues avec Albert — 11 novembre 2026

La version corrigée `runtime-5afa381bc374efc4c4c7` termine les trois répétitions avec **96 échanges conformes sur 96 et 9 fils longs conformes sur neuf**. Les 1 279 non-régressions, 149 contrôles ACL et douze contrôles techniques passent ; un contrôle Linux reste ignoré sur le Mac. Le [bilan des corrections](19-openwebui-corrections.md) fournit les mesures, reçus, limites et échecs intermédiaires. La campagne initiale ci-dessous conserve ses résultats et son image.

### Campagne initiale, conservée comme référence

La campagne complète compte **96 messages navigateur réels**, avec trois
répétitions de chacun des trois jeux de dix questions, plus six demandes au
mauvais profil. Sur l’image `runtime-93f90467b650ea8578a6`, **50 messages sur
96** satisfont l’ensemble des contrôles ; aucun des neuf fils de dix tours
n’est entièrement conforme. Les 52 réussites fonctionnelles comprennent deux
textes dont les explications de limites ne sont pas étayées.

Le [rapport complet](18-openwebui-stability.md) documente notamment les périodes
élargies à une journée, le périmètre précédent conservé malgré une correction,
les reprises de contexte, les refus mal restitués et le budget de relecture.
Le fournisseur est Albert réel : Gemma 4 rédige Incidents, GPT-OSS assure son
dialogue et sa relecture, ainsi que la planification PDL. Les profils restent
figés pendant les trois passages. Les données sont celles du SI simulé.

La campagne Allure `openwebui-baseline-redacted-20261111` conserve **50 réussites,
46 échecs et cinq portées non qualifiées**. Les treize tests déterministes de
l’oracle et la vérification du lecteur Allure restent des contrôles séparés.
Les services sont sains au point final, sans OOM ni redémarrage signalé ; cela
ne suffit pas à qualifier le comportement conversationnel. Les défauts OW-01
à OW-08 de cette image nécessitent une nouvelle recette de remise. Le
[suivi des corrections du 11 novembre](19-openwebui-corrections.md) distingue
les régressions déterministes des nouvelles conversations Albert ; la campagne
initiale ci-dessus conserve ses résultats.

## Correctifs de l’audit du 10 novembre 2026

**Correctif après l'audit du 10 novembre :** les trois défauts confirmés
(fuseau des précédents ITSM, délai total HTTP OpenSearch et validation d'une
interface autre que `uplink0`) disposent de corrections et de régressions.
Les cinq contre-exemples locaux passent. Les suites complètes comptent
1 151 réussites et un saut Linux sur Mac ; les conversations et contrôles Ubuntu
du nouveau candidat sont identifiés séparément dans leurs reçus. Les échecs
initiaux restent conservés. Voir
[l'inventaire AUD-01 à AUD-03 et les extensions conditionnelles](14-si-rie-acceptance.md#points-ouverts-après-laudit-du-10-novembre-2026).

Le correctif est installé sur Ubuntu 24.04 ARM64 dans
`runtime-f94c456528fe0e67adbc` : disponibilité et douze contrôles techniques
réussissent, ainsi que trois nouveaux parcours Albert réels (NiceGUI Incidents,
Open WebUI Incidents et Open WebUI PDL). Une génération réelle sur preuves
fictives adaptées valide séparément une autre interface, sans nouvelle collecte.
Les reçus sont sous `.runtime/ci/fix-aud-20261110/`, campagne
`corrections-aud-20261110`. Les anciennes deux heures ne sont pas attribuées
à cette image.

**La réception du SI-RIE réel reste non qualifiée.** Ses prérequis, écarts de
code/contrat et vérifications à mener sont consignés dans la
[matrice SI-RIE](14-si-rie-acceptance.md), avec responsables et preuves attendues.
Les résultats du laboratoire et les compteurs de tests ne remplacent pas cette
réception sur les données, l'annuaire, le réseau et la machine cibles.

Pour examiner les tests cas par cas, consulter **Qualification** dans NiceGUI.
Le [parcours des rapports Allure](13-test-reports.md) explique leur génération
en qualification/CI, leur import et leurs états. Ce catalogue complète le bilan
ci-dessous ; les campagnes historiques restent attachées à leurs versions.

## Mutualisation documentaire PDL — 11 novembre 2026

Les deux moteurs utilisent les mêmes primitives de lecture `_source` et
d'empreinte JSON dans `rie_agent/documents.py`. Les deux formats d'empreintes
persistées restent distincts et inchangés ; aucun catalogue ou mapping n'est
migré. Les contrôles portent notamment sur les champs plats prioritaires,
les parents absents/non objets, les listes, les valeurs nulles et les empreintes
Unicode conservées. Le module commun s'importe avec la seule bibliothèque
standard, sans transport, configuration ou annuaire.

Les suites déterministes rejouées comptent 1 161 réussites Ubuntu et un test
Linux ignoré sur Mac, 450 contrôles avec les dépendances minimales et 202 contrôles
du dialogue/revue Mac. Le complément Mac découvre 281 cas : 259 réussissent et
22 recettes live restent explicitement désactivées. Ces suites se recouvrent ;
leurs totaux ne sont pas des scénarios indépendants à additionner.

L'inventaire retrouve tous les fichiers Python publics actuels, y compris les
outils Ubuntu non encore indexés. La CI contrôle la couverture du paquet
opérateur et l'absence d'import des autres PoC depuis les deux entrées MCP.
L'image métier embarque désormais 50 fichiers Python du projet, avec sept
fichiers supplémentaires pour le portail facultatif. Ce regroupement vise
l'entretien des algorithmes communs ; il n'annonce pas une fusion des moteurs
ni une baisse artificielle du nombre de fichiers de tests.

La campagne de cette passe est `pdl-simplification-20261111`. Les reçus de
construction, de recette et de génération précisent les essais effectivement
exécutés. Les deux heures historiques et les conversations antérieures gardent
leurs dates et leur version ; elles ne qualifient pas automatiquement ce candidat.

Deux exécutions de la même question inter-index dans NiceGUI ont réellement
sollicité GPT-OSS sur Albert. La première a demandé une précision inutile sur
PFAING ; la seconde a retrouvé les six documents attendus dans les quatre
familles, sans modification du code ni de la question entre les essais.
Le rapport conserve les deux résultats : la génération n'est pas déclarée
uniformément réussie. Les sources des prompts, du dialogue et du compilateur
restent identiques à celles de la remise précédente ; cette comparaison ne
remplace pas une campagne diversifiée de conversations dans le SI.

## Lot ITSM-NG 1.6.17 — 10 novembre 2026

### Cohérence après le rangement du dépôt — 10 novembre 2026

La réconciliation du checkout partagé retrouve les 257 fichiers de la remise
`corrections-aud-final` à l'identique avant les précisions documentaires suivantes.
Les trois correctifs AUD-01 à AUD-03 sont donc conservés. Les nouveaux rejeux
réussissent : 1 151 tests Ubuntu, un test Linux ignoré sur Mac, 440 tests du socle,
354 tests des signalements agentiques déplacés et 102 tests du portail Mac.
Ces suites se recouvrent ; leurs nombres ne doivent pas être additionnés comme
des scénarios indépendants. Aucun nouvel appel Albert n'est déduit de ces tests.

Le tutoriel de mapping distingue désormais le parcours LDAP `manage.py pdl`
de l'interface autonome historique : pas d'édition JSON ni de rejet par bouton
dans la CLI Ubuntu. Les commandes de sauvegarde/édition utilisent les droits
des fichiers privés et le retrait d'un index vise `config/pdl.json`. Les schémas
PlantUML associés et leur copie exportable sont synchronisés. Le code des
images métier reste celui déjà qualifié ; les dates et limites des preuves
Albert et d'endurance précédentes restent inchangées.

### Résultats du lot initial

La [fiche des quatre US et de recette](15-itsm-precedents.md) décrit la nouvelle
recherche historique et la proposition de solution. Les suites complètes
réussissent : 471 source/opérateur/ACL/client, 163 PDL et 471 Incidents ;
un cas Linux est ignoré sur Mac. Les 52 nouveaux tests Incidents couvrent
le contrat REST, les refus avant lecture, les droits des sous-objets, la
pagination incomplète, les redirections, la taille et la durée des réponses,
la révocation pendant une lecture, le classement, la contradiction et le rendu.
Cinq nouveaux contrôles opérateur couvrent les imports privés. Les contrôles de synthèse
couvrent aussi les dates et identifiants effectivement retrouvés, les valeurs
inconnues et la transmission du motif précis d'un rejet à la correction Albert.
La qualification des générations refuse les réponses sans exécution du modèle,
sans relecture ou sans preuve d'usage du fournisseur.
La revue humaine ultérieure a relevé une attribution erronée malgré un verdict IA
valide. Son correctif ajoute dix régressions et cinq contrôles du relecteur Albert
avec affirmations bonnes/mauvaises, puis deux diagnostics intégralement générés
dans NiceGUI et Open WebUI. Voir [le périmètre exact du correctif](15-itsm-precedents.md#revue-des-attributions-de-sources--10-novembre).
La campagne d'attribution est `itsm-citations-20261110` ; la validation métier du
nouveau rendu reste à consigner.

Un vrai ITSM-NG 1.6.17, avec quatre tickets et deux identités fictives,
confirme la restriction native d'entité, les suivis privés et les solutions.
La désactivation d'un utilisateur le refuse dès la recherche suivante ;
son rétablissement conserve ces restrictions. Les quatre tickets, quatre
solutions et huit suivis sont inchangés par empreinte. Les reçus datés sont
sous `.runtime/ci/itsm-20261110/` ; les essais de construction et de réseau
intermédiaires ne sont pas présentés comme des succès.

Le serveur ITSM fonctionne maintenant dans Ubuntu, en HTTPS avec CA de laboratoire.
Il ne remplace pas la réception du SI : URL, certificats,
identités, profils, données, pertinence métier et charge réelles restent à
qualifier. La revue du rendu et de la solution proposée, suivie de l'accord de
poursuite sur les cas fictifs discutés, est consignée dans la fiche du lot.
Les résultats du 9 novembre ci-dessous
gardent leur version et leur portée.

Les parcours de génération sont désormais les actions principales NiceGUI.
La connexion des modèles Open WebUI de recette pointe explicitement vers
`albert.api.etalab.gouv.fr`, avec GPT-OSS 120B. Les lectures de santé, les
contrôles ACL, le catalogue et le contrôle technique des sources restent
déterministes ; leur succès ne valide aucune génération. En particulier,
`manage.py check` peut annoncer `albert_generation: not_tested` après une
conversation réussie : cette commande ne lance pas Albert et n'atteste pas
les conversations précédentes. Le rapport de campagne en conserve les preuves.


Les preuves initiales de génération du lot distinguent les parcours PDL avec
interpréteur/planner Albert des diagnostics Incidents avec rédaction et relecture
validées. Le fournisseur réel est GPT-OSS 120B sur Albert ; l'historique ITSM est
lu dans le vrai serveur 1.6.17 du laboratoire Ubuntu, sur données fictives.
Consulter [les usages observés et leurs limites](15-itsm-precedents.md#preuves-de-génération-réelle-du-laboratoire).
Les essais rejetés restent dans le rapport historique ; la campagne initiale
`itsm-albert-reel-20261110` garde ses propres résultats. Le correctif fait l’objet
de la campagne distincte `itsm-citations-20261110`.

La nouvelle recette NiceGUI de 12:21 UTC a produit un refus factuel justifié,
conservé comme résultat partiel. La correction bornée après ce refus est
qualifiée séparément sous `itsm-relecture-20261110` : mêmes preuves, une correction
locale et une factuelle, avec relecture obligatoire et au plus cinq appels
pour la synthèse. Le rejeu contrôlé du brouillon
refusé est distingué des conversations intégralement générées. Les essais et
motifs restent tracés ; le périmètre SI-RIE réel n'en est pas déduit.

Le profil final sépare rédaction et relecture : `gemma-4-31b-it` rédige sur
Albert, `gpt-oss-120b` relit ; PDL et l'interprétation du dialogue conservent
GPT-OSS. Sept contrôles du relecteur réel et le rejeu contrôlé du brouillon
erroné réussissent. Deux nouveaux diagnostics complets du 10 novembre
(NiceGUI 13:31 UTC, Open WebUI 13:33 UTC) réussissent avec les modèles retournés,
les usages et les essais conservés. Les échecs intermédiaires sont publiés
séparément sous `itsm-relecture-essais-20261110`, dont l'acceptation erronée du
brouillon humain par Gemma utilisé comme seul relecteur. Ces petits échantillons
ne prouvent aucune supériorité générale d'un modèle. L'accord de poursuite après
revue humaine porte sur les deux rendus fictifs discutés. L'endurance des
nouvelles images, Ubuntu 26.04 sur cette version et le SI réel restent non qualifiés.

Les deux rapports ensuite fournis par l'utilisateur (lectures à 13:33 et
13:48 UTC) ont passé dix-sept recoupements techniques et treize contrôles de
comparaison de leurs données structurées. La
[fiche comparative](15-itsm-precedents.md#comparaison-guidée-des-deux-rendus)
distingue les formulations et citations des observations et conseils identiques.
Ces contrôles ne relancent ni collecte ni Albert et ne valent pas validation
métier. Le détail du rapport est conservé pour les hyperviseurs ; l'aide à la
comparaison est propre à la recette. L'utilisateur a ensuite donné son accord
pour poursuivre, après explication de l'hypothèse et des précautions. Ce retour
ne vaut pas réception métier des vrais incidents.

Huit contrôles supplémentaires ouvrent les tickets sources dans l'interface
native ITSM-NG avec les deux identités fictives : connexion requise, lectures
autorisées, refus de l'entité protégée et sessions isolées. TLS est vérifié
strictement avant la navigation ; Chrome utilise une exception de clé publique
limitée au serveur du laboratoire dans un profil isolé, sans modifier le trousseau
du Mac. La confiance du navigateur habituel reste distincte et non qualifiée.
Les deux premiers pilotes en échec sont conservés ; les corrections portent
sur les sélecteurs du formulaire et le rendu lecteur, sans élargir les droits.

## Recette manuelle du 9 novembre 2026 — Ubuntu 24.04 ARM64

Un opérateur a rejoué les commandes dans une VM Ubuntu dédiée, sans partage
des dossiers du Mac. Cette campagne utilise le moteur
`dinum/rie-delivery:runtime-893deef36d9e3057713e` et de vrais services
OpenSearch, VictoriaMetrics et LDAP alimentés par les seules données fictives.
Open WebUI fonctionne sur le Mac ; les tickets restent un simulateur ITSM.

| Contrôle exécuté | Résultat et portée |
| --- | --- |
| Installation humaine et initialisation | Réussies : six services sains, quatre mappings PDL actifs, corpus Incidents prêt et douze contrôles techniques réussis |
| Conversations Open WebUI avec Albert | Parcours PDL et Incidents vérifiés avec GPT-OSS ; clarification, reprise et distinction résolution de l'alerte/clôture du ticket. Le suivi fermé de statut relit les sources sans seconde synthèse backend |
| Identités et administration | Alice administratrice, Bruno lecteur interne, Éric externe ; Camille non classée refusée sur le portail et l'administration. Lecteur interne et externe refusés en administration |
| Changements de politique | Restriction puis rétablissement de TACACS pour l'externe et des mesures Incidents pour le lecteur interne. Continuité des sessions confirmée par l'opérateur ; les autres sources Incidents n'ont pas été modifiées individuellement dans ce cycle manuel |
| Révocation du jeton d'essai | Avant : deux MCP accessibles et diagnostic Incidents complet. Après : HTTP 403 sur les trois routes, sans preuve ; autre jeton personnel et politique conservés. Même refus après restauration |
| Arrêt puis reprise applicative | Réussis : configurations, secrets, règles, mappings, rapports et corpus comparés par empreintes ; six services revenus sains |
| Panne VictoriaMetrics | Diagnostic `ok`, puis `partial` avec zéro mesure et cinq erreurs de cette source, puis `ok` avec cinq séries. Les 120 logs restent disponibles pendant la panne ; aucune ancienne mesure n'est réutilisée |
| Sauvegarde froide | Vérifiée : 70 fichiers au manifeste, cinq images et quatre archives de données, avec permissions privées |
| Copie volontairement corrompue | Commande de restauration refusée avant création de destination ou de volume ; sauvegarde originale et images préservées |
| Restauration dans une instance neuve | Quatre nouveaux volumes disjoints, six services sains et douze contrôles réussis. Configuration, catalogue, audits et état des jetons comparés à la sauvegarde ; révision 7 conservée |

Les corpus de référence sont conservés : 13 documents PDL, 960 logs Incidents,
10 documents techniques et 20 séries totalisant 4 200 points de métriques.
Ce volume de démonstration ne mesure pas la capacité du SI-RIE.

Les dix-huit contrôles Chrome complémentaires des trois démos et des exports
réussissent sur le portail corrigé. Ils restreignent puis rétablissent des règles
de test : la révision augmente, mais le contenu initial de la politique est
restauré. Les [captures datées](09-demonstrations.md#captures-du-9-novembre-2026)
montrent la même instance Ubuntu restaurée.

Après restauration, les audits peuvent recevoir de nouvelles lignes et le reçu
de recette ainsi que l'inventaire sont régénérés. Quatre anciens états PDL
étaient déjà expirés avant la restauration : leur nettoyage lors de nouvelles
émissions est attendu. Les autres fichiers archivés sont conservés. Une
restauration ne prolonge aucune échéance de jeton ou de conversation.

Le défaut d'affichage constaté sur une session NiceGUI inactive a motivé quatre
régressions : lecture et enregistrement après expiration, conflit de révision,
retrait du rôle administrateur. Elles utilisent les vrais composants NiceGUI et
des sessions signées, avec un annuaire isolé et une horloge de vérification
avancée dans le seul test. Elles appartiennent à la suite source et sont
distinctes d'un parcours navigateur ou d'une campagne sur le LDAP réel du SI.

Les archives finales, les exports et leur validation humaine sont consignés
séparément dans la [fiche de recette](12-manual-acceptance.md). Les essais
intermédiaires de contrôle restent conservés ; ils ne sont pas comptés comme
succès lorsqu'une attente du harnais a dû être corrigée.

**Non qualifiés par cette campagne :** nouvelle endurance de deux heures,
autres pannes de backends, redémarrage du daemon Docker ou reboot Ubuntu,
compatibilité 26.04 des nouvelles images, performances AMD64 et intégration
au SI-RIE réel. Les résultats antérieurs ci-dessous gardent leur propre portée.
Les raccordements et essais du SI restent à réaliser selon la
[matrice de réception](14-si-rie-acceptance.md).

## Corrections du 7 novembre 2026 — nouveau runtime

Le jalon GitLab `3a526b4` précède les corrections E01–E09 de la
[matrice SI-RIE](14-si-rie-acceptance.md). La nouvelle campagne utilise deux
instances vierges, avec données fictives, groupe administrateur distinct et
Bruno lecteur interne. Elle ne met pas à jour les anciennes instances C2.

Le candidat source C1 a pour SHA-256
`0f2fa687bbcd56f293e56454c58eb65bf9a9e27036acfd5276099ea543e3e43a`.
Le runtime est `dinum/rie-delivery:runtime-634f4b8b5993ade3b211` sur les deux VM.
La remise finale actualise les documents et le portail ; le reçu final compare
séparément l'identité d'exécution du runtime à ce candidat. Les images ont changé
par rapport à C2 : les anciens chiffres d'endurance ne leur sont pas attribués.

| Vérification de la campagne de corrections | Résultat et portée |
| --- | --- |
| Suites locales hors réseau | 908 réussites : 448 source/ACL/client Albert, 152 PDL, 308 Incidents ; un cas Linux ignoré sur Mac |
| Contrôles Linux | Sept cas exécutés sans skip dans chaque VM, dont surveillance inotify pour vérifier l'absence de lecture interdite |
| Installation scriptée neuve 24.04 et 26.04 ARM64 | Réussie, douze contrôles métier par VM et six services sains après démarrage |
| Identités et rôles sur LDAP réel simulé | Même jeton et même session relus après ajout/retrait du groupe administrateur ; lecteur interne, externe, ambiguïté, remplacement d'objet LDAP et expiration contrôlés sur chaque VM |
| NiceGUI | Dix-huit contrôles Chrome par VM : les deux UC, ACL, Bruno lecteur sans administration ni rapports, refus/rétablissement et exports exacts |
| Rapport Allure courant | Huit contrôles Chrome par VM : 908 réussites, un saut, deux périmètres non qualifiés affichés distinctement ; SHA des exports, iframe isolée et révocation à la déconnexion vérifiés |
| Contrats externes sur vrais OpenSearch/VictoriaMetrics du laboratoire 24.04 | Champs imbriqués et multifields keyword, identifiant documentaire `knowledge:router-1`, bindings métriques propres ; 120 points sur 720 attendus à 10 s donnent 16,67 %. Diagnostic partiel attendu, aucun appel IA |
| Pannes/reprises sur 26.04 | Cinq cas : PDL, Incidents, LDAP, OpenSearch, VictoriaMetrics ; refus ou résultat partiel attendu, indisponibilité signalée puis reprise contrôlée |
| Exploitation et restauration sur 24.04 | Neuf critères réussis : précontrôles sans mutation, imports privés, cinq contrôles sans croissance du registre de jetons, sauvegarde froide, refus de corruption et restauration neuve comparée fichier par fichier |
| Nouvelle endurance et charge IA réelle | Non rejouées sur ces images ; aucune capacité SI-RIE, performance AMD64 ou compatibilité d'une IA interne n'en est déduite |
| Validation humaine et réception SI-RIE | Non qualifiées ; compléter les fiches 12 et 14 avec les responsables nommés |

Le contrôle des contrats a créé puis supprimé seulement ses deux index et leurs
règles temporaires. Les compteurs des index de démonstration, les échantillons
VictoriaMetrics et les configurations d'origine sont restés inchangés. Ce test
réel de protocoles utilise des données synthétiques ; il ne remplace pas des
échantillons autorisés du SI.

Les délais de connexion, d'en-têtes et de corps VictoriaMetrics sont injectés
avec un transport HTTP de test dans les régressions. Le budget coupe ce lecteur,
y compris un flux lent, avant de lancer d'autres demandes hors délai. Cela ne
qualifie pas une deadline bout en bout de tous les connecteurs dans le réseau cible.
Le budget IA est testé sur les neuf couples de routes HTTP/MCP ; un tour peut
faire plusieurs appels fournisseur et son admission reste propre au processus.

Les reçus privés sont sous `.runtime/ci/si-rie-corrections-20261107/` :
`validated-tests/`, `native/`, `browser-rie-u24-auto/`, `browser-rie-u26-compat/`,
`allure-contracts/` et les deux dossiers `allure-browser-*`. Les anciennes preuves
restent dans leurs dossiers datés. Deux erreurs de harnais ont été conservées :
un précontrôle Docker initial sans `sudo`, puis un lecteur JSON qui ne gérait pas
la sortie multi-ligne. La reprise corrigée a passé les neuf critères sans
modification du produit ; ces erreurs ne sont pas comptées comme succès produit.
Le rapport Allure exportable de cette campagne
est un fichier distinct des paquets source et Wiki.js.

## Revue du 7 novembre 2026 — revalidation technique réussie

Le code et le parcours humain ont été revus avant la validation manuelle :
image runtime/opérateur minimale, portail et qualification facultatifs,
modèles externes annotés, logs/stockage en lecture seule, credentials fictifs
ciblés et import de mappings par fichier privé. Les droits UC2 ajoutent
`incidents.metrics` et `incidents.tickets` aux deux index ; les anciennes
politiques ne sont pas ouvertes automatiquement. Voir [ACL et migration](06-acl.md).

**La revalidation technique est réussie sur les configurations décrites.**
Le portail C3 a passé dix-sept contrôles navigateur sur chaque version Ubuntu,
y compris le clic interpage qui rend réellement la section cible visible sous
la navigation. Les seize succès C2 antérieurs restent identifiés séparément.
La validation humaine du nouveau déploiement reste à réaliser ; l'arrêt du
laboratoire et l'intégrité des paquets remis sont consignés dans leurs reçus
finaux distincts, pas déduits de ces tests.

Le candidat C2 est identifié par SHA-256
`bce98b54253057496a3b36c349be58d730971840b2c9fdce540fca33d366f255`.
Le candidat C3 du portail est identifié par SHA-256
`f44f6108b9e173117965b2f9d5f4fb5f08c16fe115b15fcdca280cc9914afe40`.
Ses 52 fichiers documentaires ont été comparés au paquet avant les essais.
Les 54 fichiers immuables du runtime et les manifestes/configurations
**d'exécution** sont identiques à C2 sur les deux VM. L'index OCI peut changer
avec l'attestation BuildKit ; cette métadonnée n'est pas présentée comme une
modification du moteur. La correction C3 porte sur le portail et sa navigation.
Le candidat C2 a été transféré seul dans les VM Ubuntu 24.04 et 26.04 ARM64, sans partage
de dossiers Mac. Les installations scriptées ont créé de **nouvelles instances**,
sans réutiliser les volumes de la campagne du 6 novembre.

| Vérification du 7 novembre | Résultat acquis et portée |
| --- | --- |
| Installation C2 Ubuntu 24.04 et 26.04 | Réussie ; douze contrôles techniques sur chaque VM, six services sains et runtime importable sans NiceGUI |
| ACL des quatre sources UC2 | 126 contrôles et 35 sondes sur chaque VM ; les 25 refus de chaque campagne n'ont effectué aucune lecture de fichier source, recherche OpenSearch ni création de rapport |
| Tests spécifiques Linux | Sept cas exécutés sur chaque VM, sans skip, avec observation inotify pour les refus avant lecture |
| Portail C2 | Seize contrôles Chrome réussis sur chaque VM, aucune erreur JavaScript ; refus/rétablissement des métriques depuis la même session, exports comparés aux fichiers exacts de C2 |
| Contrats hors réseau | 383 tests collectés au dernier reçu source, dont un skip Mac explicitement couvert par les cas Linux natifs ; pas d'échec. PDL : 135 ; Incidents : 253 séparément |
| Exploitation C2 Ubuntu 24.04 | Neuf critères réussis : précontrôle sans mutation, logs/stockage, fichier mapping root privé, jetons temporaires, backup, corruption, intégrité restaurée, recette de reprise et conservation finale |
| Charge courte C2 Ubuntu 26.04 | `smoke` de 60,078 s : 73 demandes, sept refus attendus, zéro erreur inattendue ; 63 intervalles monotoniques, minimum 5,000046 s, zéro violation |
| Génération Albert et endurance de deux heures nouvelles images | Non rejouées ; les conversations et deux heures du 6 novembre restent historiques |
| Portail C3 et navigation interpage vers une section | Dix-sept contrôles Chrome réussis par VM, zéro erreur JavaScript ; section réellement visible après navigation asynchrone |
| Validation humaine du nouveau déploiement | À faire avec la [fiche vierge](12-manual-acceptance.md), depuis une instance neuve |

Les sondes ACL ont conservé les services, images et règles initiales ; la
révision a progressé de 1 à 11 lors des changements contrôlés. Les registres de
jetons sont restés de zéro à zéro. Ces compteurs ne remplacent pas les décisions
source par source : chaque refus attendu est vérifié avec absence de lecture
et de preuve. La granularité par équipement/tenant/période reste hors livraison.

### Exploitation, conservation et reprise C2

Trois configurations invalides — port, JSON et image absente — ont été refusées
**avant** arrêt ou écriture : mêmes conteneurs, images, heures de démarrage,
fichiers runtime et ressources Docker. `logs` et `storage` ont aussi conservé
ces éléments. Un mapping public adapté au seul index fictif a été validé avec
`--input` depuis un fichier root `0600`, sans créer ni approuver de revue.

Cinq `check` successifs ont conservé le registre à **un enregistrement**,
octet pour octet, ainsi que le jeton personnel présent. Les identités de contrôle
temporaires ne font pas croître ce registre. La sauvegarde froide contient cinq
images et quatre volumes. Avant le premier démarrage de la restauration neuve,
les 261 fichiers des volumes ont été comparés par contenu SHA-256, mode et
propriétaire UID/GID : tous identiques, politique/catalogue/audits/jetons compris.
La copie restaurée puis la source ont chacune passé douze contrôles techniques.
Une archive corrompue a été refusée sans création de destination ou volume.

Le contrôleur de recette a d'abord échoué à analyser une sortie JSON mêlée à
une ligne de commande. La restauration produit avait réussi. La reprise du
**harnais** a vérifié les volumes déjà importés, sans refaire l'import et sans
modifier le code produit ; les reçus initiaux sont conservés. La source a été
rétablie saine et la copie restaurée arrêtée, volumes conservés.

### Images minimales et limites des mesures C2

Sur la VM 24.04, Docker a relevé 297 839 861 octets pour le runtime, contre
426 433 271 pour l'ancien monolithe du 6 novembre, et 415 784 325 pour le portail
C2. Le runtime contient 37 distributions Python, le portail 68. Ce sont les
valeurs **non compressées `image.Size`**, avec couches parentes partagées ; elles
ne s'additionnent pas pour calculer l'espace réellement occupé ni le transfert.
Le portail final pourra changer après sa correction documentaire.

Le runtime n'installe plus NiceGUI ni les extensions de `uvicorn[standard]` du
portail. Le choix de boucle serveur peut donc différer de l'ancien monolithe.
Les performances des deux heures du 6 novembre ne mesurent **pas** cette nouvelle
image. Le `smoke` C2 utilise le fournisseur PDL fictif et Incidents sans synthèse :
ce n'est ni une endurance ni une qualification de performance Albert ou AMD64.

Le corpus de charge séparé a conservé, avant/après le contrôle court, chaque
identifiant et contenu des 100 000 événements PDL, 100 000 logs Incidents, dix
documents, vingt séries et 4 200 points métriques. Aucune donnée réparée. Ses
jetons ont été révoqués, son seul projet arrêté et ses volumes conservés ; la
référence de démonstration 26.04 est restée identique.

### Défaut du premier candidat et traçabilité

C1 a échoué au démarrage PDL sur les deux VM : `openwebui_dialogue.py`, encore
importé par une fonction métier, avait été omis de la liste runtime. Son archive
et ses reçus négatifs restent conservés. C2 rétablit cette dépendance utile et
ajoute un import depuis le contexte exporté isolé ainsi qu'un contrôle Docker
`RUN --network=none`. Le succès C2 ne réécrit pas l'échec C1.

Les preuves privées du 7 novembre se trouvent sous
`.runtime/ci/ubuntu-hardening-20261107/` : `native-install-c2/`,
`uc2-source-acl-c2/`, `operations-c2/`, `load-smoke-c2-native26/` et
`browser-candidate24/26/`, `browser-c3-24/26/` et `image-identity/`. Elles
n'entrent pas en bloc dans les archives publiques.
Les captures sélectionnées sont dans la [galerie datée](09-demonstrations.md#galerie-de-qualification-du-7-novembre-2026).
Les nouvelles références du laboratoire sont décrites dans le
[journal de reprise](11-installation-log.md#reprendre-le-laboratoire-conservé-sur-le-macbook).
Le reçu privé de fin de campagne indique l'état d'arrêt effectif des VM et la
conservation de leurs volumes. Les commandes de reprise du journal s'emploient
hors campagne active ; les tests ci-dessus ne constituent pas ce reçu d'arrêt.

## Campagne historique du 6 novembre 2026

## Matrice de la campagne

| Domaine | Recette prévue | État de la campagne nouvelle |
| --- | --- | --- |
| Installation humaine | Ubuntu 24.04 ARM64 neuve, paquet seul, parcours manuel et reprises documentées | Réussi après corrections ; voir le journal d'installation |
| Installation scriptée | Seconde Ubuntu 24.04 neuve, reprise après erreur et réexécution | Réussi ; recette technique de 12 contrôles réussie après reprise |
| Compatibilité | Installation et recette fonctionnelle Ubuntu 26.04 | Réussi avec v5 : 12 contrôles techniques, 17 contrôles ACL, 14 contrôles Chrome et six services sains |
| Fonctionnement | Tests pertinents des deux UC, HTTP/MCP et ACL, cas autorisés/refusés | Réussi pour les suites et scénarios exécutés ; détail ci-dessous |
| Navigateur NiceGUI | Trois démos reliées à la VM, diagrammes et téléchargements | Réussi ; 14 contrôles Chrome, sans erreur JavaScript |
| Conversations Albert | Deux clients Open WebUI locaux, six cas métier et suivis | V6 : parcours initial échoué 5/6, puis refus externe réussi en reprise ciblée ; aucun parcours complet corrigé revendiqué |
| Pannes de dépendances | PDL, Incidents, LDAP, OpenSearch et VictoriaMetrics | Réussi ; cinq pannes et reprises contrôlées |
| Sauvegarde et reprise | Restauration neuve, corruption, redémarrage Docker et Ubuntu | Réussi ; détails ci-dessous |
| Cycle ACL réel | Groupes frais, objet LDAP recréé, révocation, expiration et sessions courtes | Réussi ; dix contrôles Ubuntu 24.04, puis 17 contrôles Ubuntu 26.04 v5 |
| Fraîcheur des sources | Modifier des mesures et tickets fictifs, relire puis restaurer | Réussi ; huit contrôles, puis nouvelle recette technique de 12 contrôles |
| Pannes du fournisseur IA | Délais, erreurs, récupération et protection de cadence | Réussi ; huit pannes, reprise et admission Incidents de 60 secondes vérifiées |
| Endurance | Deux heures sur instance de charge distincte, paliers 1/5/10 utilisateurs | V2 échouée ; v6 réussie sur erreurs/ACL/corpus/stabilité, avec six écarts de cadence aux transitions documentés |
| Statut Incidents v6 | Autorisation, manifeste et réponse HTTP sans génération | Réussi : interne 200, externe 403 sans équipements ; manifeste invalide couvert en 503 par les tests |
| Documentation | ZIP extrait, liens, draw.io éditable et Wiki.js de référence | Réussi ; huit modèles et huit contrôles navigateur Wiki.js, détails ci-dessous |

La remise finale conserve les états **réussi**, **échoué** ou **non qualifié**,
avec version, machine, date et rapport associé. Un essai encore en cours n'est
pas un succès. Un test sauté conserve sa raison. Aucun mot de passe ni jeton
n'entre dans les pièces publiques.

## Versions et machines qualifiées

| Exécution | Version identifiée | Portée de la preuve |
| --- | --- | --- |
| Installation manuelle Ubuntu 24.04.5 ARM64 | Paquets intermédiaires corrigés, dont v2/v3 | Parcours humain, initialisation et recette après reprises ; [journal détaillé](11-installation-log.md) |
| Endurance Ubuntu 24.04.5 ARM64 | Sources v2 gelées pendant la campagne | Échoué après 7 200,088 secondes : 1 539 erreurs PDL sur 8 460 demandes ; corpus conservé et projet arrêté |
| Endurance corrigée Ubuntu 24.04.5 ARM64 | Paquet v6 gelé, instance neuve `rie-load-fixed` sur la seconde VM | 7 200,069 secondes, 8 460 demandes, zéro erreur inattendue, corpus intégral conservé ; écart de cadence publié |
| Compatibilité Ubuntu 26.04.1 ARM64 | Paquet v5, après les corrections des sondes et des sessions | Installation automatisée reprise, fonctionnement, ACL et navigateur ; aucun Albert appelé sur cette VM |
| Outils et documentation remis | Version du manifeste final de livraison | Inclut des corrections postérieures au gel de l'endurance ; ne constitue pas l'image exacte déjà soumise aux deux heures |

La VM 26.04 utilise le noyau `7.0.0-34-generic`, Docker 29.9.3 et Compose 5.7.0.
Le paquet v5 identifié par SHA-256
`496792903b4c47307f715961ac9210070fd5217da088d5bcd614d27878084ff9`
a passé la reprise d'installation automatisée, douze contrôles techniques,
dix-sept contrôles ACL et quatorze contrôles Chrome sans erreur JavaScript.
Les six services sont actifs et `healthy`, sans OOM ni redémarrage enregistré.
Les pages et figures du navigateur ont été comparées au contenu exact de v5.

Le reçu historique v4 garde son résultat global négatif : il signalait LDAP
`unhealthy` et aucune sonde Docker VictoriaMetrics malgré les succès métier.
La nouvelle exécution v5 conclut les corrections, sans réécrire ce reçu.
L'arrêt et la conservation des VM sont documentés par un reçu distinct.
V5 reste un paquet de qualification antérieur à la remise finale ; les
modifications postérieures au gel v2 ne lui attribuent pas rétroactivement
les deux heures d'endurance.

Le paquet corrigé v6 contient 174 fichiers, SHA-256
`f898bed5092fd72cd364335df6ed58406c806afea37af7eaaec9e2723bf27b65`.
Au gel, 112 fichiers de code et configuration ont été comparés octet pour
octet avec le checkout, sans écart.

Le code des moteurs est resté gelé pendant les deux heures. Le harnais Open WebUI
et son test ont été corrigés après v6 pour le délai entre les cas ; cette
modification d'outil ne change pas les moteurs soumis à l'endurance.

Le pilote `qualification.py` a aussi été corrigé après v6, avec le nouveau
test `test_qualification_cadence.py`, pour conserver les cinq secondes entre
demandes d'un même utilisateur aux transitions. Les deux heures publiées ont
été mesurées avant ce changement ; elles ne sont pas revendiquées pour la
nouvelle version de cet outil.

Les documents peuvent évoluer et le manifeste final identifiera les pièces
effectivement remises.

## Résultats déjà exécutés

Le PDL portable a passé 135 tests et le moteur Incidents 253 tests de sa suite
complète. La vérification finale regroupe **322 contrôles réussis avec sockets
bloquées** : ACL 60, Ubuntu 188, client Albert 54 et contrôles CI 20.

La suite de compatibilité historique a également passé 165 tests. Ces résultats
hors réseau sont distincts de la génération Albert réelle.

Les recettes techniques natives vérifient des lecteurs et compilateurs réels,
sans confondre leurs intentions contrôlées avec une génération.

Les conversations Albert ont exercé le comptage PFS, une clarification suivie,
le rapprochement des six documents `edge-a`, un diagnostic R1 avec synthèse,
cinq métriques et 120 logs, puis les droits de l'externe. Les contrôles des
retours MCP, citations et refus sont conservés avec les conversations de recette.
Le parcours complet Albert v6 a réussi cinq cas sur six en sept envois. Le
dernier cas, refus Incidents d'un externe, a atteint la protection de cadence
sans publier de preuve, puis la boucle du client a été arrêtée.

Le harnais attendait depuis le début du cas précédent au lieu d'attendre
depuis sa fin. Après correction du harnais, le seul cas externe a été rejoué
manuellement après le délai complet : un envoi, sept contrôles réussis, modèle
et configuration des backends inchangés.

Les six cas disposent ainsi de preuves positives sur v6, avec huit envois au
total. Le parcours initial reste échoué ; un nouveau parcours complet avec
le harnais corrigé n'a pas été exécuté.

Les deux clients locaux ont été arrêtés en conservant volumes et conversations.
Les deux nouveaux jetons de recette ont été révoqués. Ce résultat reste séparé
de l'endurance v6 mesurée sans Albert.

Le point HTTP Incidents `/status` vérifie les droits sur les deux index
configurés, logs et documents, avant de lire le manifeste. Dans v6, un
`prepared_at` absent est rendu par `null` ; un manifeste invalide conserve une
réponse 503. Les tests hors réseau couvrent ces cas. La recette HTTP réelle
v6 a vérifié un interne en 200 avec état `ready`, puis un externe en 403 sans
liste d'équipements, sans demander de génération.

Les cinq pannes contrôlées ont signalé l'indisponibilité puis repris après
rétablissement, avec des délais observés de 0,686 à 9,197 secondes dans cette
configuration. Ces mesures ne sont pas des engagements de reprise du SI réel.

La campagne fournisseur a utilisé des serveurs fictifs sur la boucle locale
des deux backends, sans appeler Albert. Huit cas ont réussi : connexion refusée,
erreurs HTTP 500/429 et JSON invalide de classification, HTTP 500 et JSON
invalide du planificateur, puis délais réels de classification et planification.
Les délais observés étaient de **30,036 s** et **60,066 s**. Aucune ancienne
preuve n'a été réutilisée et chaque étape a effectué une seule tentative.
Après rétablissement, une nouvelle recherche PDL a produit son propre rapport.
Les demandes anonymes ou avec identité usurpée ont été refusées avant le fournisseur.

La synthèse Incidents fictive a exercé classification, rédaction et revue.
La demande immédiatement suivante a renvoyé `rate_limited` avec 60 secondes,
sans nouvelles lectures métier ni rédaction/revue. **La classification du
second message reste un appel IA** : cette protection borne la synthèse,
pas tous les appels d'interprétation. Aucun paramètre de cadence n'a été diminué.
La configuration initiale a été restaurée octet pour octet ; `check` et les
douze contrôles techniques ont ensuite réussi. Les six services Ubuntu 24.04
ont également passé leurs sondes Docker corrigées, sans changement d'image.

La restauration a recréé une identité d'instance et des volumes neufs, comparé
les quatre volumes avant démarrage et conservé catalogue, politique, identités
LDAP et état des jetons. Une archive corrompue a été refusée avant création de
volume. Le redémarrage Docker et un véritable reboot Ubuntu ont été suivis de
recettes techniques réussies ; le changement d'identifiant de démarrage du
guest confirme le reboot.

Le cycle annuaire a vérifié avec le même Bearer le passage interne vers externe,
le refus d'une double appartenance, le rétablissement, puis le refus de l'ancien
jeton après recréation de l'objet LDAP. Le nouveau jeton et l'expiration contrôlée
d'un enregistrement ont également été testés, sans changer l'horloge de la VM.
Sur Ubuntu 26.04 v5, la recette a été étendue aux sessions signées de dix minutes
du portail. Les dix-sept contrôles réussis vérifient notamment qu'après
suppression et recréation d'un compte avec les mêmes identifiant, adresse mail
et mot de passe, son UUID LDAP change : l'ancien jeton individuel et l'ancienne
session sont refusés. Les deux backends renvoient `directory_identity_changed`
pour l'ancienne session ; une nouvelle connexion et un nouveau jeton fonctionnent.
Le compte temporaire a été supprimé et les jetons de recette révoqués.

La recette de fraîcheur a ajouté vingt séries de métriques isolées, fait passer
la latence maximale attendue de 80 à 85, modifié un ticket fictif, puis vérifié
que de nouvelles demandes lisaient ces changements avec des identifiants de
requête distincts. Les huit contrôles ont réussi : les anciens rapports n'ont
pas été modifiés, les 120 logs de référence ont été conservés et aucune synthèse
IA n'a été appelée. Les fichiers initiaux ont été restaurés et les vingt séries
temporaires supprimées. Le seed de cohérence, `check` et les douze contrôles
techniques ont ensuite de nouveau réussi.

Les huit figures ont été ouvertes et réexportées dans le véritable éditeur
draw.io 32.6.10 auto-hébergé dans la VM. Les identifiants, libellés et connexions
natifs sont conservés. Une boîte et un point de routage d'un connecteur ont été
modifiés sur une copie, puis enregistrés, réouverts et réexportés avec leurs
changements. Les sources canoniques de la documentation restent inchangées.

Le Wiki.js de référence est en version 2.5.315. Trois pages représentatives
(installation, architecture et ACL) et 24 pièces jointes ont été importées.
Les PNG et fichiers `.drawio` ont conservé leurs empreintes ; les SVG affichés
ont conservé leur géométrie visible après assainissement, avec retrait du modèle
embarqué lors d'un upload générique.

La variante Markdown native a ensuite passé huit contrôles navigateur : lecture,
ouverture par **Edit Diagram**, modification d'une boîte et d'un connecteur,
retour à l'éditeur, sauvegarde de la page, réexport de la source et réouverture
du modèle modifié. Aucune erreur JavaScript n'a été observée. Pour cette recette,
la requête du plugin vers son éditeur externe par défaut a été redirigée par le
navigateur de test vers la copie locale, sans envoi à un service externe.
Ce dispositif ne constitue pas une configuration du wiki du SI ; la version
et le raccordement de son éditeur restent à confirmer.

## Recette fonctionnelle

Tester les recherches PDL simples et croisées, la sélection des familles, les
bornes temporelles, les leurres hors périmètre, la revue d'un mapping et son
invalidation après changement de schéma. Le DSL, les identifiants de documents
et les faits attendus sont vérifiés indépendamment des résultats produits.

Pour Incidents, tester R1, R2, le trou de collecte R3, l'équipement exclu, la
chronologie des tickets, le diagnostic sans synthèse et les clarifications.
Une lecture de mesures ne doit pas dépendre d'anciens identifiants Zabbix lorsque
le fournisseur est VictoriaMetrics.

Pour les ACL, tester interne, externe, non classé, groupes ambigus, identité
absente, jeton expiré/révoqué, compte recréé, index non administré/interdit,
changement de groupe et de politique, usurpation par en-tête et isolation des
conversations et sessions navigateur. Chaque nouveau tour relit les droits.

## Recette IA réelle et simulation

Le fournisseur fictif permet des intentions contrôlées et des pannes reproductibles
sans quota Albert. Ses sorties sont identifiées comme simulées. Elles ne prouvent
ni la qualité d'un modèle réel, ni sa capacité de raisonnement ou son débit.

Si Albert est accessible avec les autorisations retenues, tester les vrais
dialogues, clarifications, suivis, abstentions et citations dans Open WebUI.
Sinon, consigner **génération non qualifiée** et la cause. Le modèle client et
les modèles utilisés par les backends doivent tous être identifiés.

## Résilience et restauration

Arrêter puis rétablir chaque backend et dépendance, injecter délais et erreurs
IA, redémarrer Docker puis Ubuntu. Vérifier refus fermé, délais bornés, réponses
partielles explicites et reprise après rétablissement, sans nouvelle autorisation
implicite. Restaurer à froid dans une instance neuve et comparer corpus, mappings,
politique, audits et état des jetons. Une archive corrompue doit être refusée
avant d'écrire la destination.

## Endurance et performance

La campagne principale dure deux heures hors injections de panne :

| Palier | Durée | Charge maximale prévue |
| --- | --- | --- |
| 1 utilisateur simulé | 30 minutes | Une demande par utilisateur toutes les cinq secondes au maximum |
| 5 utilisateurs simulés | 45 minutes | Même cadence individuelle |
| 10 utilisateurs simulés | 45 minutes | Même cadence individuelle |

Le corpus de charge est distinct : 100 000 événements PDL et 100 000 logs
Incidents reproductibles. Les mesures enregistrent p50/p95/p99, débit, erreurs
attendues/inattendues, CPU, mémoire, disque, croissance des rapports et files
d'attente lorsqu'elles sont mesurables. La protection de cadence de synthèse est
vérifiée séparément ; elle ne doit pas être supprimée pour obtenir un bon chiffre.

La campagne exige zéro fuite ACL, zéro erreur inattendue hors scénarios prévus,
aucune perte de données, aucun OOM ni redémarrage spontané. La capacité est publiée
par palier avec la configuration exacte. En l'absence d'objectif SI convenu, une
latence de laboratoire ne devient pas un engagement de production.

### Reproduire l'endurance depuis une instance neuve

Sur l'hôte Ubuntu, depuis la racine du **paquet source extrait et vérifié**,
utiliser Python 3.10 ou plus et les prérequis Docker du manuel. Les commandes
ci-dessous ne dépendent pas de Git ni de paquets Python sur l'hôte. Elles
utilisent `sudo` pour l'accès Docker et les fichiers privés ; conserver ce
choix pour tout le parcours.

Choisir une instance et un dossier de résultats **neufs**, hors des sources,
distincts de la démonstration et de toute campagne précédente :

```bash
export RIE_LOAD_INSTANCE="$HOME/.local/share/rie-charge-NEUF"
export RIE_LOAD_RESULTS="$HOME/qualification-results/rie-charge-2h-NEUF"
sudo python3 deploy/ubuntu/manage.py prepare "$RIE_LOAD_INSTANCE" --mode demo
sudo python3 deploy/ubuntu/load_seed.py --prepare-accounts "$RIE_LOAD_INSTANCE"
```

Attendu : instance créée sans service, puis dix identités fictives préparées
sans afficher leurs mots de passe. `--prepare-accounts` doit précéder la toute
première opération Docker de cette instance, donc aussi `build`, `pull` ou
`status`. Une instance déjà associée à Docker est refusée par cette préparation.

Définir explicitement les paramètres publics de la campagne :

```bash
sudo python3 - "$RIE_LOAD_INSTANCE" <<'PY'
from pathlib import Path
import sys
sys.path.insert(0, 'deploy/ubuntu')
import manage
instance = Path(sys.argv[1]).resolve()
values = manage.load(instance)
values.update({
    'RIE_LOAD_QUALIFICATION': 'true',
    'RIE_SIMULATED_ALBERT': 'true',
    'ALBERT_URL': 'http://127.0.0.1:8123/v1',
    'ALBERT_MODEL': 'qualification-fixture',
    'PDL_PORT': '20010',
    'INCIDENTS_PORT': '20050',
    'RIE_ENABLE_DEMO_UI': 'false',
    'DINUM_DIALOGUE_MAX_STATES': '8192',
})
manage.save_env(instance, values)
PY
sudo python3 deploy/ubuntu/manage.py validate "$RIE_LOAD_INSTANCE"
sudo python3 deploy/ubuntu/manage.py build "$RIE_LOAD_INSTANCE" --with-qualification
sudo python3 deploy/ubuntu/manage.py pull "$RIE_LOAD_INSTANCE"
sudo python3 deploy/ubuntu/manage.py up "$RIE_LOAD_INSTANCE"
sudo python3 deploy/ubuntu/manage.py seed "$RIE_LOAD_INSTANCE"
sudo python3 deploy/ubuntu/manage.py check "$RIE_LOAD_INSTANCE"
sudo python3 deploy/ubuntu/manage.py acceptance "$RIE_LOAD_INSTANCE"
```

Vérifier que les ports 20010 et 20050 sont libres avant le démarrage. Le
fournisseur fictif reste en boucle locale du backend PDL, sans port hôte et
sans appel Albert réel. La capacité de 8 192 états est explicite et bornée ;
elle ne désactive ni l'expiration ni les contrôles d'identité. NiceGUI est
désactivé pour cette instance. L’image de qualification est demandée explicitement ; le profil de charge
l’utilise pour PDL et l’opérateur, sans ajouter ses outils au runtime habituel.
Les phases doivent réussir le corpus de référence, `check` et la recette de la
version courante **avant** le chargement volumineux.

Charger ensuite les 100 000 événements PDL et les 100 000 logs Incidents dans
cette seule instance. La commande de campagne ci-dessous capture les journaux
des cinq services locaux dès le départ. Si le laboratoire utilise aussi une
API ITSM-NG native et sa base, ajouter leurs noms Docker exacts avec
`--log-container` **avant de lancer la campagne**, comme indiqué dans
[le parcours de capture continue](13-test-reports.md#conserver-les-journaux-pendant-une-endurance).
Les journaux d'une source externe doivent être obtenus auprès de son exploitant ;
leur couverture ne peut pas être déduite de celle des moteurs.

```bash
sudo python3 - "$RIE_LOAD_INSTANCE" <<'PY'
from pathlib import Path
import sys
sys.path.insert(0, 'deploy/ubuntu')
import manage
instance = Path(sys.argv[1]).resolve()
values = manage.load(instance)
manage.guard(instance, values)
manage.tool(instance, values, 'python', '/delivery/load_seed.py')
PY
sudo python3 deploy/ubuntu/manage.py check "$RIE_LOAD_INSTANCE"
sudo python3 deploy/ubuntu/qualification_logs.py "$RIE_LOAD_INSTANCE" --output "$RIE_LOAD_RESULTS" --duration 7200 --period 5
```

Attendu : corpus volumineux chargé, instance prête et deux heures réelles de
demandes aux trois paliers. Une durée raccourcie reste un `smoke`. Le dossier
de résultats doit être inexistant au départ ; les rapports et échantillons
système sont privés. Conserver la source et l'image gelées pendant la mesure.
`report.json` porte le verdict combiné ; `endurance/` conserve les mesures et
`logs/` les flux privés et leurs empreintes. Une capture interrompue ne valide
pas la campagne, même si les demandes terminées étaient correctes.

Même si l'endurance échoue, exécuter ensuite la comparaison complète en lecture
seule et conserver son reçu :

```bash
sudo python3 - "$RIE_LOAD_INSTANCE" "$RIE_LOAD_RESULTS/post-corpus.json" <<'PY'
from pathlib import Path
import os
import sys
sys.path.insert(0, 'deploy/ubuntu')
import manage
os.umask(0o077)
instance = Path(sys.argv[1]).resolve()
values = manage.load(instance)
manage.guard(instance, values)
result = manage.tool(instance, values, 'python', '/delivery/load_seed.py', '--verify', capture=True)
with Path(sys.argv[2]).open('x') as receipt:
    receipt.write(result.stdout)
print('Contrôle du corpus réussi ; reçu privé conservé.')
PY
```

Attendu : `passed: true`, `read_only: true`, chaque identifiant et document
comparé, valeurs et dates des métriques vérifiées, `source_data_repaired: false`.
Un écart fait échouer le contrôle. Après le chargement volumineux et après
l'endurance, ne pas relancer `seed`, `install.sh` ou `acceptance` pour prouver
l'absence de perte : leurs compteurs portent sur le corpus de référence et
une initialisation pourrait réparer des données au lieu de constater l'écart.

### Échec de l'endurance v2 et nouvelle campagne

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

### Résultat final de l’endurance corrigée v6

**Réussi sur les critères applicatifs et de conservation, avec un écart de
cadence documenté aux transitions.** Le rapport natif et la collecte vérifiée
concluent les 7 200,069 secondes : 8 460 demandes, zéro erreur inattendue et
810 refus ACL attendus, sans fuite de preuve interdite relevée.

| Utilisateurs | Durée | Demandes | p50 (ms) | p95 (ms) | p99 (ms) | Débit (demandes/s) |
| --- | --- | ---: | ---: | ---: | ---: | ---: |
| 1 | 30 min | 360 | 43,842 | 50,466 | 55,513 | 0,2 |
| 5 | 45 min | 2 700 | 67,277 | 77,651 | 94,084 | 1,0 |
| 10 | 45 min | 5 400 | 119,739 | 152,970 | 165,401 | 2,0 |

Tous les paliers ont zéro erreur inattendue. Les refus attendus se répartissent
en 0, 270 et 540. Les latences regroupent PDL, Incidents et ces refus ; elles
mesurent les échanges HTTP/MCP de cette simulation.

Le fournisseur PDL est fictif et les diagnostics Incidents sont sans synthèse.
Aucun appel Albert réel n'entre dans ces chiffres de charge.

#### Cadence et limites de mesure

Dans chaque palier, les intervalles reconstruits par utilisateur vont de
5,000016 à 5,031373 secondes. **Six transitions de palier raccourcissent
l'intervalle entre deux demandes à 3,942136–3,958843 secondes.** La borne
stricte de cinq secondes n'a donc pas été tenue à ces frontières.

Ces deux heures restent la mesure de l'outil antérieur à la correction de
cadence. Le harnais de qualification est corrigé séparément ; une nouvelle
campagne complète de deux heures avec cette correction n'est pas revendiquée.
Ces corrections de harnais n’ont pas modifié les moteurs soumis à la campagne v6.

Le pilote corrigé a passé six tests de cadence et un contrôle natif court de
60,066 secondes : 73 demandes sans erreur, 63 intervalles mesurés avec une
horloge monotonique, minimum 5,000053 secondes et zéro violation. Les six
transitions contrôlées vont de 5,000531 à 5,056887 secondes.

Le corpus est resté intégralement identique et les preuves v6 sont conservées
octet pour octet. Après ce contrôle supplémentaire, les trente jetons de
charge sont tous révoqués. Le reçu est
`endurance-fixed-v6/cadence-transition-v7/transition-control.json`.

Les futurs rapports du pilote corrigé exigent zéro violation de cadence pour
conclure `passed`. Ce contrôle court ne vaut pas une nouvelle endurance de
deux heures.

Le maximum côté client est de dix demandes simultanées, avec une demande par
client et sans file illimitée. La profondeur de file interne du serveur n'est
pas instrumentée. Aucun SLA ni dimensionnement du SI-RIE n'est déduit de ces
mesures sur la VM ARM64.

#### Ressources observées

Les 421 échantillons ne montrent ni erreur de collecte, ni OOM, ni arrêt
spontané, ni redémarrage. L'écart maximal entre deux échantillons est de
20,246 secondes. Le CPU est exprimé en pourcentage d'un CPU logique ; la
mémoire est le working set rapporté par Docker.

| Service | CPU p95 (%) | CPU maximum (%) | Mémoire maximale (octets) |
| --- | ---: | ---: | ---: |
| PDL | 32,23 | 146,91 | 113 875 354 |
| Incidents | 27,87 | 192,41 | 133 064 294 |
| OpenSearch | 2,54 | 15,17 | 1 365 799 600 |
| VictoriaMetrics | 0,72 | 8,98 | 50 405 048 |
| LDAP | 1,67 | 2,85 | 25 060 966 |

Les statistiques détaillent aussi les ressources par palier :

| Utilisateurs | Service | CPU p95 (%) | CPU maximum (%) | Mémoire maximale (octets) |
| --- | --- | ---: | ---: | ---: |
| 1 | PDL | 11,68 | 13,14 | 104 448 655 |
| 1 | Incidents | 11,86 | 13,32 | 106 220 749 |
| 1 | OpenSearch | 1,82 | 2,19 | 1 253 056 709 |
| 1 | VictoriaMetrics | 0,73 | 0,94 | 43 023 073 |
| 1 | LDAP | 1,08 | 1,43 | 19 272 827 |
| 5 | PDL | 15,66 | 146,91 | 113 875 354 |
| 5 | Incidents | 14,27 | 192,41 | 133 064 294 |
| 5 | OpenSearch | 3,26 | 15,17 | 1 285 268 963 |
| 5 | VictoriaMetrics | 0,70 | 8,98 | 47 469 036 |
| 5 | LDAP | 1,13 | 1,98 | 19 755 172 |
| 10 | PDL | 38,99 | 47,49 | 112 721 920 |
| 10 | Incidents | 37,46 | 50,27 | 129 499 136 |
| 10 | OpenSearch | 2,57 | 13,51 | 1 365 799 600 |
| 10 | VictoriaMetrics | 0,76 | 1,33 | 50 405 048 |
| 10 | LDAP | 1,76 | 2,85 | 25 060 966 |

Le disque utilisé de la VM passe de 7 229 861 888 à 7 537 311 744 octets ;
l'espace libre minimal observé est de 74 532 622 336 octets.

Le volume applicatif logique, rapports et audits compris, passe de
3 141 939 à 321 765 360 octets.

| Rapports | Classés avant campagne | Classés pendant campagne |
| --- | --- | --- |
| PDL | 44 fichiers / 570 278 octets | 4 230 fichiers / 30 073 890 octets |
| Incidents | 33 fichiers / 2 151 946 octets | 4 230 fichiers / 281 722 295 octets |

Ce classement est reconstruit après campagne à partir des dates de modification.
Il ne remplace pas une photographie initiale des rapports. Aucun rapport n'est
classé après la fin de campagne.

#### Conservation, expiration et nettoyage

La comparaison finale en lecture seule a réussi pour les 100 000 événements
PDL, 100 000 logs Incidents, dix documents, vingt séries et 4 200 points
métriques. Aucune donnée n'a été réparée. La politique reste à sa révision 1,
identique au fichier initial, et les quatre mappings restent actifs.

La vérification native après une heure a observé la disparition des 41 états
expirés depuis au moins dix secondes et la conservation des 1 002 états encore
valides contrôlés, sans modifier le TTL ni les données.

À la collecte finale, PDL conserve 3 150 états : 3 132 encore valides et
18 expirés restant sur disque. Aucun fichier invalide n'est trouvé ; Incidents
ne conserve aucun état dans cette campagne. L'expiration est contrôlée à la
lecture ; le nettoyage physique est déclenché par les émissions suivantes.
La capacité demeure bornée à 8 192 états.

Les vingt jetons de charge des dix identités sont tous révoqués ; aucun ne
reste actif.

Le dossier `endurance-fixed-v6/` conserve le rapport natif, la comparaison du
corpus, les statistiques et le reçu de collecte. Les onze fichiers de l'archive
de preuve ont passé leurs empreintes ; son SHA-256 est
`204164e46592d05d584ac7f1ccd574715e70d1e0d5163a9c33ded0cad8f12ebd`.

### Résultat final des pannes IA et de la limitation

**Réussi.** Huit pannes et huit contrôles transverses ont passé la campagne
`provider-faults-v2`, le 6 novembre 2026 de 15:47:27 à 15:50:00 UTC sur
Ubuntu 24.04. Les délais 30/60 secondes ont été réellement attendus. Le fournisseur
fictif était isolé dans les namespaces réseau des backends ; aucun port hôte,
volume privé ou socket Docker ne lui a été fourni. Ces résultats sont séparés
des conversations Albert réelles et de l'endurance.

La campagne a ensuite été rejouée sur la référence Ubuntu 24.04 avec le code
v6 : huit cas réussis, délais réels de 30,035 et 60,088 secondes, puis `check`
et les douze contrôles techniques réussis.

Les six secrets et les quinze jetons antérieurs sont conservés octet pour
octet. Les quinze nouveaux jetons de recette ont été révoqués.

## Limites de validité

Une VM Ubuntu ARM64 sur MacBook ne qualifie pas les performances d'un serveur
AMD64, d'un réseau SI réel ou d'un modèle interne. Les tickets simulés ne
qualifient pas l'API ITSM-NG réelle. Le wiki local de référence ne garantit pas
la configuration exacte du Wiki.js du SI tant que sa version et son éditeur
restent inconnus. Ces réserves sont des validations distinctes, pas des succès
à déduire des tests du paquet.


## Administration guidée des mappings — 16 novembre 2026

Le portail facultatif ajoute **Mappings PDL** : sélection des champs découverts,
types détectés, valeurs métier, relations d’identité, import/export YAML,
proposition inactive et approbation explicite avec différences et justification.
Il réutilise les validations du moteur portable et les revues existantes. Aucun
contrat MCP n’a été ajouté ou modifié.

Les régressions déterministes couvrent les refus avant lecture, la perte du
rôle, les changements de session, de politique, de schéma, de revue et de version
active, l’absence d’activation à l’enregistrement, les imports bornés et la saisie
sans YAML. Le runner Ubuntu fournit JUnit et Allure, indépendamment du navigateur.
**1 329 tests réussis et un ignoré** sont enregistrés sous
`.runtime/ci/mapping-ui-20261116-release`. Les résultats finaux sont consignés dans le [journal d’installation](11-installation-log.md#administration-des-mappings-pdl--16-novembre-2026).

La recette Playwright sur une vraie VM **Ubuntu 24.04 ARM64** a réussi
**13 contrôles**, avec OpenLDAP/OpenSearch réels et données fictives du SI
sandbox : refus lecteur/externe, reprise des quatre revues, découverte,
champs/types/relations, enregistrement sans doublon ni activation, export,
import invalide puis réimport valide, différences lisibles, refus sans
confirmation, approbation d’un index et effacement de l’écran à la déconnexion.
L’instance de qualification finale est `rie-9e165bdc6e4b`, distincte de
`rie-5ccf2ce0d183` où les quatre revues humaines restent en attente.
Les captures datées sont accessibles dans le [guide PDL](04-pdl.md#définir-le-mapping-dans-le-si-réel).

Les premières tentatives ont révélé un doublon de revue lié à la provenance du
YAML, un emplacement d’import restant occupé après une erreur et un compteur
non actualisé après approbation. Ces défauts sont corrigés et les tentatives
en échec sont conservées avec les reprises sous
`.runtime/sandbox-si-20261116/mapping-browser-candidate1` à `candidate5`.
Le dernier répertoire contient le bilan réussi, captures, diagnostics et
résultats Allure. Une attente du test a aussi été corrigée pour attendre la fin
d’une découverte avant d’ouvrir une liste.

**Limites :** pas de nouvel appel Albert ni de campagne de charge pour cette
administration ; pas de nouvelle qualification Ubuntu 26.04. Les résultats
conversationnels, endurance et compatibilité antérieurs gardent leur date et
leur version. La revue sémantique des vrais champs, valeurs et relations reste
à faire avec les responsables du SI-RIE. Les quatre revues de la recette humaine
ne sont pas approuvées par ces essais.
