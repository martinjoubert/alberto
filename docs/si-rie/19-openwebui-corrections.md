# Corrections des conversations Open WebUI — 11 et 12 novembre 2026

Cette campagne reprend les défauts OW-01 à OW-08 de la
[première campagne](18-openwebui-stability.md). Ses 50 réussites sur 96 et ses
échecs restent conservés. Une correction déterministe ne vaut pas validation
d’une nouvelle conversation avec Albert.

## Résultat courant — 12 novembre 2026

**Conforme sur la matrice exécutée : 96 échanges conformes sur 96 ; 9 fils longs conformes sur neuf.**
Version `runtime-5afa381bc374efc4c4c7`, Ubuntu 24.04 ARM64, corpus fictifs,
Albert réel et Open WebUI local sur le Mac. Les trois répétitions utilisent
la même matrice, la même image et les mêmes modèles ; aucun échange en échec
n'est remplacé par un réessai masqué. Les essais antérieurs sont conservés.

L'exclusion explicite des sources PDL reconnaît le point de fin de phrase,
les alias approuvés et les négations coordonnées. Seules les cibles de logs
sont retirées ; la résolution d'identité et les ACL restent distinctes.
Un alias exclu ambigu ou un ensemble de cibles vide impose une clarification.

Le pilote v19 a révélé un nom d'outil abrégé avant lecture. Le relais v25
indique le nom exact et emploie une description de transport neutre. La copie
envoyée au modèle ne contient ni question métier ni reprise opaque. Le callable
protégé transmet la vraie demande au MCP ; Albert reste chargé de l'interprétation
et, pour les diagnostics, de la rédaction et de la relecture dans les moteurs.
Les signatures MCP sont conservées. Les six sondes API du relais restent historiques en v20 (code du filtre inchangé).
Le pilote mixte de dix échanges passe sur v25, séparément des 96 questions. PDL utilise maintenant Gemma pour son dialogue et sa planification ; le dialogue et la relecture Incidents conservent GPT-OSS. La réparation des remplacements de sources partage le budget borné des propositions.
Les douze échanges PDL v22, les cinq échanges PDL/Incidents et les trois
réparations contrôlées v21 restent attachés à leur version. Le dernier texte
utilisateur est conservé en relance ; les nouvelles recherches complètes ne
reprennent pas les anciens critères. Les formulations référentielles gardent
leur contexte. Une seule réparation du format est permise dans un délai partagé
de soixante secondes, avec 4 096 tokens par essai au maximum ;
les erreurs conservent phase, code fermé et identifiant de corrélation. La cause
exacte de l’échec générique v20 reste indéterminée : les anciennes réponses ne
sont pas reclassées comme réussies.

Les **1 279 non-régressions** passent, avec un contrôle ignoré sur Mac ; les
**149 contrôles ACL/LDAP** et les **douze contrôles techniques** passent dans Ubuntu.
La recette technique utilise un planificateur déterministe : elle ne constitue
pas une validation Albert. Les neuf contrôles du lecteur Allure passent pour
chacun des rapports de non-régression et de contrôles opérationnels.
Les preuves nouvelles sont dans `.runtime/ci/openwebui-fixes-20261112/`.
Les services restent démarrés à la fin de la validation.

## État historique à l'arrêt — 11 novembre, 23 h 45 à Paris

La campagne v18 est interrompue après **48 échanges, dont 47 conformes**.
La deuxième répétition PDL conserve TACACS malgré son exclusion explicite :
six documents sont rendus au lieu de quatre. Le prototype de correction v19
reste local et **ses quatre nouvelles régressions échouent** ; il n'est ni
paqueté ni déployé. La dernière suite entièrement verte est celle de v18 :
1 251 réussites et un contrôle ignoré sur Mac. La qualification complète et
les paquets finaux de remise restent à terminer après correction.

Les dix conteneurs du PoC et les VM Ubuntu/Colima sont arrêtés, avec données
et preuves conservées. La note détaillée de reprise est
`.runtime/ci/openwebui-fixes-20261111/REPRISE-20261111.md` dans le checkout local.

## Lire les résultats

Dans NiceGUI, ouvrir **Qualification**. Les trois rapports courants sont :

- `openwebui-fixes-units-v25-20261112` : 1 279 réussites et un contrôle ignoré sur Mac.
- `openwebui-controls-v25-20261112` : 173 contrôles, dont 149 ACL/LDAP,
  douze techniques sans génération, dix échanges du pilote mixte et deux réparations contrôlées de sélection des sources. Les
  réparations contrôlées et le pilote Incidents v21 restent historiques. Les portées sont précisées dans chaque cas.
- `openwebui-corrections-v25-20261112` : les 96 échanges réels et leur verdict strict,
  avec questions, attentes, résultats natifs et journaux de diagnostic.

Les portées non qualifiées s'affichent séparément des réussites. Les relectures
Albert de brouillons volontairement injectés et les réparations contrôlées des
versions précédentes restent des preuves historiques, pas de nouvelles
conversations de v25. Les captures originales restent privées ; les rapports
exportables omettent les images sans changer leurs verdicts.

| Passage | Type et résultat strict | Portée |
| --- | --- | --- |
| Initial | 50 / 96 ; zéro fil long entièrement conforme sur neuf | Référence historique, conservée |
| v7 | 95 / 96 ; huit fils longs conformes sur neuf | Matrice complète ; une attribution de source rejetée |
| v9 | 93 / 96 ; sept fils longs conformes sur neuf | Matrice complète ; trois clarifications superflues |
| v11 | 15 / 16 | Campagne interrompue après un défaut reproduit |
| v12 | 13 / 14 | Campagne interrompue après un défaut reproduit |
| v13 | 47 / 49 | Campagne interrompue après un défaut reproduit |
| v14 | 9 / 9 indépendants, puis 3 / 4 dans le pilote mixte | Matrice complète non lancée |
| v15 | 8 / 9 indépendants | Pilote mixte et matrice complète non lancés |
| v16 | 37 / 38 | Campagne interrompue ; JSON incomplet du relais avant le MCP |
| v17 | 17 / 18 | Campagne interrompue ; refus direct du relais avant le MCP |
| v18 | 47 / 48 | Interrompue à la demande de l’utilisateur ; exclusion TACACS ignorée |
| v19 | 1 / 2 dans le pilote | Nom d'outil abrégé par le relais ; matrice complète non lancée |
| v20 | 49 / 50 | Interrompue après une erreur Incidents générique ; cause initiale exacte non établie |
| v21 | 14 / 15 | Interrompue après omission de trois sources PDL explicitement demandées |
| v22 | 38 / 43 | Interrompue ; deux clarifications techniques, une erreur client Albert et deux sélections PDL incorrectes |
| v23 | 2 / 3 dans le pilote mixte | Ancien critère de refus conservé dans une recherche d'équipement ; matrice complète non lancée |
| v24 | 94 / 96 ; sept fils longs conformes sur neuf | Matrice complète ; un timeout de planification Albert et une clarification de sources injustifiée |
| v25 | 96 / 96 ; 9 fils longs conformes sur neuf | Matrice complète sur une image figée |

Les versions intermédiaires ne sont pas additionnées pour annoncer trois
répétitions réussies d'une même image. Les cas injectés ne comptent jamais
comme conversations normales.

## Progression par lots

Chaque lot est suivi de ses régressions ciblées, puis des trois suites Ubuntu :
source/opérateur/ACL/client Albert/CI, PDL et Incidents. Les appels externes sont
interdits dans ces suites. Un cas propre à Linux est ignoré sur le Mac et reste
compté séparément ; les imports des véritables images sont contrôlés sur Ubuntu.

| Lot | Défauts traités et comportement attendu | Validation déterministe |
| --- | --- | --- |
| 1 | OW-01 : parseur de dates commun, heures chiffrées ou écrites, midi/minuit ; demander une précision au lieu d’élargir silencieusement une fenêtre ambiguë | 1 180 réussites, un saut |
| 2 | OW-08 et OW-06 : priorité au dernier périmètre explicite issu du catalogue actif ; restitution PDL calculée depuis le plan et les preuves réellement exécutés | 1 182 réussites, un saut |
| 3 | OW-02 et OW-05 : reprise du dernier tour du même UC après un changement de profil ; conservation des paramètres vérifiés ; limites du domaine avant lecture | 1 189 réussites, un saut |
| 4 | OW-04 : un appel d’outil ; restitution des refus structurés, liés à cet appel et dépourvus de preuves ; aucun succès fabriqué depuis une erreur technique | 1 195 réussites, un saut |
| 5 | OW-03 : catalogue complet des sources pour la relecture, sans recopier les documents et précédents ; maintien du rapport visible et de la limite de 24 000 caractères | 1 197 réussites, un saut |
| 6 | OW-07 : consignes de causalité prudente, aide de correction précise et bornée ; nouvelle relecture obligatoire ; rejet conservé après épuisement des essais | 1 199 réussites, un saut |
| 7 | Qualification sur les nouvelles images Ubuntu et conversations longues réelles ; distinction entre observations et analogies historiques | 1 279 non-régressions réussies, un saut ; 96 / 96 échanges conformes |

Les deux UC conservent leurs contrats MCP. Les règles ACL et les contrôles
de causalité/citations restent applicables. Le dialogue porte au plus douze
réponses courtes sous une limite totale de 2 000 caractères ; ce n’est pas une
mémoire illimitée. Une reprise invalide ou un échec du dernier tour de cet UC
ne doit pas ressusciter une ancienne investigation.

Le rapport PDL reste détaillé : opération, période, filtres, comptage, documents,
limites et requêtes exécutées. Le rapport Incidents conserve mesures, événements,
chronologie, documents, précédents ITSM, proposition conditionnelle et ses
niveaux de confiance. Seules les répétitions dans l’entrée du relecteur sont
retirées. Un contexte réellement trop grand reste signalé explicitement.

## Environnement et premiers contrôles Ubuntu

L’instance restaurée de recette est conservée sur Ubuntu 24.04 ARM64. Le paquet
candidat contient un manifeste de 265 fichiers ; archive et manifeste ont été
vérifiés avant construction. Les points d’entrée des moteurs s’importent dans
leur véritable image sans NiceGUI et sans accès réseau.

Le candidat v18 décrit dans ces premiers contrôles utilise `dinum/rie-delivery:runtime-0bf502e4cf55a3edbd3f`.
Son paquet source contient
265 fichiers. Les deux filtres des profils
de recette Open WebUI ont été actualisés depuis les sources canoniques ; les
modèles, connexions individuelles et conversations antérieures sont conservés.
La disponibilité métier et les douze contrôles techniques passent.

Les 126 contrôles d’accès aux sources Incidents passent à nouveau sur ce candidat, avec restauration du
contenu des règles et nettoyage des credentials techniques. Le cycle de vie
LDAP passe 23 contrôles : droits administrateur explicites, changements de
groupes sans reconnexion, groupes ambigus, remplacement d’identité, expiration
et usurpation par en-tête. Le compte jetable est supprimé en fin de recette.

Un premier passage du harnais LDAP s’était arrêté : il exigeait un diagnostic
complet pour le compte jetable dépourvu de compte ITSM individuel. Le harnais
accepte désormais uniquement le résultat partiel explicite
`itsm-ng / individual_credential_missing`, avec les cinq mesures attendues et
sans génération. Les autres erreurs restent des échecs. Aucun compte partagé
ne remplace ce compte, et l’essai initial reste conservé.

## Essais intermédiaires conservés

Un premier fil de dix tours a réussi ses neuf premiers échanges PDL, puis a
demandé un champ physique au retour vers l’activité de l’équipement. L’essai
a été interrompu avec douze soumissions et onze réponses terminées dans le
harnais ; il ne constitue pas une campagne complète.

La planification expose désormais un résumé des rôles et espaces de noms
d’identité, dérivé uniquement des mappings actifs et à jour. Elle distingue
explicitement les sources d’identité et les cibles de logs, même lorsque ces
dernières ne portent pas le nom de l’équipement. Les trois formulations réelles
du pilote suivant passent, sans nom ni réponse de fixture dans le correctif.

Deux pilotes ont ensuite montré des confusions de domaine : une comparaison
Incidents redirigée vers PDL, puis une recherche de logs PDL traitée comme un
diagnostic incomplet. Les capacités décrites au classifieur ont été précisées.
Une recherche isolée, explicitement exprimée comme une opération sur les logs
PFS, BB, PFAING ou TACACS, est aussi refusée dans le profil Incidents avant
classification et lecture. La présence d’un objectif de mesures, d’alertes ou
de diagnostic conserve le parcours normal ; les formulations non reconnues
restent confiées à l’interpréteur. Ce garde-fou ne choisit ni équipement,
ni période, ni droits et ne remplace pas les appels Albert des diagnostics.

Le dernier pilote de comparaison Incidents a passé une rédaction Gemma et une
relecture GPT-OSS réelles : contexte de relecture de 21 908 caractères pour
une limite de 24 000. Le refus du mauvais profil passe ensuite dans le
navigateur. Ces pilotes et leurs échecs restent distincts des 96 échanges
de la campagne finale.

La répétition a encore révélé une demande superflue de champ physique PDL.
Une correction bornée de la planification complète les consignes : lorsque
la proposition demande un champ d’identité alors que des relations d’identité
actives sont approuvées, une seule nouvelle proposition est autorisée depuis
la même demande et le même catalogue. Les périodes et périmètres explicites
restent imposés par Python ; la validation habituelle des sources, mappings et
ACL s’applique ensuite. Aucune lecture n’a lieu pour cette première proposition
en clarification, aucune erreur de transport n’est relancée et une seconde
ambiguïté reste une clarification. Les deux tentatives sont tracées dans les
métadonnées de planification. Sans relations approuvées, cette correction
n’est pas déclenchée.

Le pilote mixte suivant a réussi six tours du même fil : quatre familles PDL,
TACACS seul, exclusion de TACACS, diagnostic Incidents avec Albert, retour
implicite au sujet PDL, puis données manquantes de R3. Les anciennes suites
de dialogue passent également : 206 tests du dialogue PDL/client et 504 tests
Incidents, avec quatre cas HTTP délégués à la suite Ubuntu complète. Ces
compteurs se recouvrent avec la suite Ubuntu ; ils ne s’additionnent pas.

Un contrôle supplémentaire a reproduit le décalage de deux heures d’une date
écrite « UTC+2 », interprétée comme UTC. Cette notation naturelle est désormais
refusée au lieu de déclencher une lecture sur la mauvaise fenêtre. Employer
deux bornes ISO 8601 avec leur décalage, ou indiquer l’heure de Paris/UTC.
GMT sans décalage est reconnu comme UTC. Une indication contradictoire de
fuseaux est refusée. Les dates françaises et les bornes ISO déjà prises en
charge conservent leurs contrôles d’heure d’été/hiver.

La reprise est aussi testée : une correction complète de période remplace
une date invalide antérieure ; une dernière période invalide ne réutilise
jamais les anciennes bornes. Le pilote navigateur a passé quatre tours sur
les deux UC : notation refusée sans preuves, puis correction ISO et résultat
sur la bonne période. Le diagnostic Incidents corrigé a appelé Albert pour
la rédaction et la relecture.

## Campagne conversationnelle réelle

La matrice initiale est conservée : trois jeux de dix tours, répétés trois
fois, plus six demandes au mauvais profil, soit 96 envois navigateur. Les
changements entre PDL et Incidents sont explicites dans le même fil. Les
attendus restent les références des corpus, avec des données contrastées en
dehors des périodes et périmètres demandés.

Albert réel assure les appels : GPT-OSS pour le dialogue et la planification
PDL, Gemma 4 pour la rédaction Incidents, GPT-OSS pour sa relecture. Les limites
d’appels restent actives. Une réponse factuelle calculée sur une question
d’état ne doit pas être confondue avec une synthèse IA ; les diagnostics qui
exigent une rédaction doivent présenter ses appels et sa relecture validée.

Le passage intermédiaire v7, sur `runtime-ebb840ac1fbbc66e6e52`, a terminé
ses 96 envois : **95 conformes, un échec de génération ; huit fils longs
entièrement conformes sur neuf**. Les 96 textes affichés correspondent aux
réponses natives, sans reprise opaque visible ; les empreintes de configuration
avant et après sont identiques. Les périodes, périmètres, changements de sujet
et refus passent tous les contrôles ciblés. L’échec reste conservé dans
Allure `openwebui-corrections-v7-20261111`.

L’échec restant concerne une hypothèse de défaut physique : une réparation
historique était citée sans expliquer suffisamment son rôle d’analogie.
Le candidat v8 indique au rédacteur et au relecteur le rôle de chaque source :
observation de la fenêtre, contexte technique ou précédent. Une hypothèse qui
cite un précédent doit citer aussi une observation ; son rendu commence par
« Piste par analogie, à confirmer ». Ce contrôle ne prouve pas la cause.
Les faits absents du ticket, la fausse acceptation d’une solution refusée et
les citations inconnues restent rejetés. Le rapport conserve toute sa densité.

Les contrôles sémantiques utilisent des brouillons injectés et un relecteur
Albert réel : ils vérifient ses décisions, sans qualifier une rédaction complète
ni une nouvelle collecte. Les pilotes navigateur suivants et la matrice complète
qualifient séparément le parcours de rédaction et relecture réelles.

Les sept contrôles sémantiques v8 passent : quatre mauvaises attributions sont
rejetées et trois usages prudents des sources sont acceptés. Les reçus indiquent
les décisions attendues, obtenues et les usages du modèle réellement retourné.

Le pilote v8 a ensuite réussi deux formulations sur trois. La demande
« compare les pistes aux précédents disponibles » était prise à tort pour
une comparaison de périodes. La matrice complète v8 n’a donc pas été lancée.
Le candidat v9 distingue ces deux objectifs avant de demander une seconde
fenêtre. Une comparaison aux tickets ou aux retours d’expérience n’autorise
aucune lecture de mesures sur une autre période. Une comparaison temporelle
demandée en plus conserve l’exigence de bornes explicites ; des paramètres de
référence antérieurs ou proposés par un modèle ne suffisent pas.

Les 215 tests ciblés passent, puis les trois suites complètes : 1 223 réussites
et un saut. Les trois formulations du pilote v9 passent avec rédaction et
relecture Albert. Les 149 contrôles ACL et les douze contrôles techniques sont
rejoués sur cette image avant la nouvelle campagne. Entre v8 et v9, seul
`obs_poc/domain.py` change dans les entrées de l’image métier ; les sources du
rédacteur et du relecteur des sept contrôles sémantiques sont identiques.

La campagne complète v9 termine avec **93 réussites sur 96 et sept fils longs
entièrement conformes sur neuf**. Les 96 textes affichés correspondent aux
rapports natifs ; les configurations avant/après sont identiques. Aucune
synthèse n'est rejetée. Trois clarifications superflues restent des échecs :
une perte de période et sa conséquence au tour suivant, puis un comptage où
le planificateur prend le nom de source pour une identité d'équipement.

Deux protections supplémentaires sont ensuite ajoutées :

- Si le modèle classe une demande comme nouvelle malgré « ce même créneau »,
  seules les dates confirmées de l'état signé peuvent être conservées.
  L'équipement et le périmètre viennent de la nouvelle question. Une nouvelle
  date explicite remplace ces bornes ; une date invalide ne réutilise pas
  l'ancienne période. Les sources sont relues avec les droits actuels.
- Pour un comptage, un nom ou alias de source approuvée employé comme identité
  déclenche une seule correction du plan avant lecture. Les libellés viennent
  du catalogue courant, sans liste de fixtures dans la protection. Un équipement
  explicitement nommé comme une source reste une demande d'identité. La seconde
  confusion est arrêtée ; aucun filtre ni critère ne peut être supprimé pour
  obtenir un succès. La correction partage son plafond avec celle des métadonnées.

Le candidat v11 passe 1 238 non-régressions et le pilote de quatre questions,
mais sa campagne est **interrompue après 16 échanges, dont 15 conformes** :
le planificateur demande encore une référence intermédiaire après sa seconde
proposition. Ces seize reçus sont conservés ; ils ne constituent pas trois
passages. La configuration est inchangée et les seize textes affichés restent
identiques aux réponses natives.

Le candidat v12 utilise un contrat de réparation dédié à la résolution
d'identité. Le modèle doit relever l'identité initiale, les cibles, le périmètre
et la période. Le moteur retrouve les références intermédiaires pendant ses
lectures, sous les relations déjà approuvées. La seconde réponse passe toujours
la validation habituelle des identités, relations, schémas et ACL. Il n'y a
aucun nouvel essai de transport, aucune approbation automatique et aucune
lecture de tous les logs pour contourner une identité absente. Ce contrat
ne remplace pas une demande visant une autre source ni une question de champ
sans rapport avec l'identité.

Sur ce candidat, **1 239 tests déterministes réussissent, un est ignoré**.
Trois contrôles provoquent la clarification initiale puis appellent Albert
réellement pour la réparation : les trois entrées sont validées sur le graphe
approuvé, sans lecture métier. Un pilote distinct de **sept échanges navigateur
réussit intégralement**, dont trois formulations d'activité retrouvant les six
documents réels du corpus. Les 149 contrôles ACL et les douze contrôles
techniques passent aussi. Les sept décisions du relecteur restent associées
à v8 ; ses sources sont identiques dans v12, sans transférer une qualification
plus large entre images.

La campagne v12 est également **interrompue après 14 échanges, dont 13
conformes** : le même refus de métadonnées persiste après la seconde proposition.
Les textes natifs, les empreintes avant/après et le reçu d'interruption sont
conservés. Ces quatorze envois ne constituent pas une matrice complète.

La réparation v13 repart uniquement des entrées validées : question originale,
catalogue courant, inventaire, motif d'erreur établi localement et contrat
d'exécution. La première affirmation erronée du modèle n'est plus réinjectée,
même sous une étiquette « non fiable ». Une régression vérifie que deux fausses
affirmations différentes produisent le même contexte de correction.

Les **1 240 non-régressions réussissent, avec un saut**. Trois contrôles injectent
la clarification exacte observée, puis appellent réellement Albert et exécutent
les lectures OpenSearch sous l'identité LDAP d'Alice : les six documents attendus
sont retrouvés dans chaque cas. La première clarification est une faute injectée,
pas une réponse Albert ; la réparation et les données sont réelles.

Une série indépendante de **neuf conversations PDL réussit** : trois formulations
d'activité d'équipement, répétées trois fois. Elle ne remplace pas la vérification
du contexte dans les fils de dix tours. Le pilote mixte puis la matrice complète
utilisent la même image et les mêmes modèles, sans changement de configuration.

La campagne v13 est **interrompue après 49 échanges, dont 47 conformes**.
La seconde proposition PDL réclame encore une référence intermédiaire ;
la clarification empêche aussi le suivi attendu. Les textes natifs et la
configuration inchangée sont vérifiés sur ces 49 échanges. Les neuf essais
indépendants précédents restent des réussites, sans rendre ce fil stable.

### Séparer l'intention des choix de schéma

Le candidat v14 remplace la réparation de métadonnées par une extraction
d'arguments via une fonction fournisseur imposée : identité initiale,
sources de logs et éventuels filtres supplémentaires. Albert ne reçoit pas
le schéma physique pour cette étape. Python choisit ensuite un unique ensemble
de relations déjà approuvées et valide le plan ; une ambiguïté réelle reste
un arrêt avant lecture. Les filtres supplémentaires ne sont pas supprimés.
Les sources d'identité ajoutées n'élargissent jamais les cibles de logs.
Cette structuration interne n'ajoute aucun outil MCP public ni nouvel accès.

Trois contrôles injectant la première clarification fautive passent avec
une réparation Albert réelle, des droits LDAP réels et les six documents
OpenSearch attendus. Neuf nouvelles conversations PDL indépendantes passent
également. Le pilote mixte s'arrête ensuite à son quatrième tour : après
trois recherches d'équipement réussies, un comptage autonome hérite à tort
de l'ancien équipement et produit une structure invalide. Le moteur refuse
la lecture. La matrice complète v14 n'est donc pas lancée.

### Éviter l'héritage d'un équipement dans un comptage autonome

Le candidat v15 conserve l'extraction structurée, puis ajoute un contrôle
de transition : un comptage entièrement fourni, avec période, périmètre
approuvé et source courante unique, repart du message actuel, sans reprendre
l'ancien équipement. Les références explicites et les demandes incomplètes
conservent leur contexte. Les noms de sources et valeurs de périmètre viennent
du catalogue courant, sans noms de fixtures dans la protection.

Les **1 248 non-régressions passent, avec un saut**. Une régression force
la mauvaise décision `follow_up`, vérifie que la question autonome est exécutée
seule, puis contrôle que « ce même équipement » conserve le contexte.
Les alias administrés, schémas modifiés et sources ambiguës sont aussi testés.
Le pilote navigateur contrôle en plus l'absence du filtre d'ancien équipement
dans le plan du comptage ; le seul total du corpus ne suffisait pas à le voir.

La série indépendante v15 termine avec **huit réussites sur neuf** : la dernière
demande réclame un identifiant d'ancre déjà défini dans le catalogue. Cette
formulation ne correspondait pas encore au détecteur de clarification technique.
Le pilote mixte et la matrice complète v15 ne sont pas lancés.

Le candidat v16 reconnaît aussi les noms de concepts d'identité approuvés,
au même titre que les champs physiques, ainsi que le vocabulaire des ancres,
du catalogue et des relations. Il utilise la même extraction structurée bornée ;
aucune relation ni autorisation n'est créée. La régression vérifie ces variantes
avec deux appels maximum. **1 249 non-régressions passent, avec un saut**.

La campagne v16 est **interrompue après 38 échanges, dont 37 conformes**.
Le relais Open WebUI produit une fois des arguments JSON incomplets pour
une question d'état à midi ; le MCP n'est pas appelé. Le message visible
signale l'absence de diagnostic vérifiable et ne publie aucune preuve.
Les 38 reçus et la configuration inchangée sont conservés ; l'audit d'affichage
identifie les 37 réponses natives et ce résultat natif absent.

### Transmettre les arguments protégés sans recopie par le relais

La question exacte et le jeton sont déjà liés au callable natif par le filtre.
Le candidat v17 expose donc au seul modèle de relais une fonction sans arguments
à recopier. Le filtre transmet lui-même les deux valeurs protégées au MCP,
dont la signature demeure inchangée. Le jeton opaque n'est plus envoyé au
modèle de relais ; les anciennes preuves ne sont pas placées dans son contexte.
Le serveur continue d'interpréter la demande avec Albert et de lire les sources
sous l'identité courante. Aucun diagnostic ni contrôle factuel n'est supprimé.

La campagne v16 conserve l'échec réel de parsing. Les régressions déterministes
échouent avant correction, puis vérifient l'appel sans paramètres métier fournis
par le relais, la question exacte transmise, l'absence du jeton côté fournisseur
et la conservation des métadonnées MCP. La variante PDL historique reste
distincte du contrat portable. **1 251 non-régressions passent, avec un saut**.

Le premier contrôle réel constate un écart de sérialisation du fournisseur :
`{"": {}}` au lieu de `{}`. Open WebUI 0.11.3 écarte les propriétés absentes
de la signature MCP avant d'invoquer le callable protégé : aucun paramètre
métier du fournisseur n'est transmis et les six documents attendus sont retrouvés.
L'attendu strict `{}` de ce contrôle supplémentaire n'est donc pas satisfait ;
cet écart est conservé comme observation de protocole, avec le filtrage vérifié
dans le code installé. Il ne modifie ni la matrice ni ses critères métier.
Une autre version d'Open WebUI doit être requalifiée sur ce contrat de filtre.

La campagne v17 est **interrompue après 18 échanges, dont 17 conformes**.
Sur une comparaison bénigne des mesures, alertes et logs de Rennes, le relais
renvoie un refus direct et n'appelle pas le MCP. Le budget configuré est de
4 096 tokens ; il n'est pas épuisé. Le message visible signale l'absence de
diagnostic vérifié. Les 18 reçus, le refus final et les usages sont conservés,
sans exporter le raisonnement interne. La configuration reste inchangée.

### Limiter le relais à une commande de transport neutre

Dans v18, le modèle de relais reçoit uniquement une commande constante pour
appeler la fonction sélectionnée avec le marqueur `question: ""`.
Ce schéma comporte un champ connu, au lieu d'un objet sans propriétés.
La vraie question, le jeton et les anciennes preuves ne lui sont pas envoyés.
Le callable protégé transmet au MCP le message utilisateur exact et le jeton
vérifié. Le backend interprète toujours la vraie demande avec Albert, relit
les sources sous l'identité courante et conserve ses contrôles factuels.
Le relais ne remplit aucune fonction d'autorisation ni de diagnostic.

Le contrat neutre passe **six appels Albert réels, trois par UC**, avec exactement
un appel et le marqueur attendu. Ce contrôle de protocole ne lit aucune source
et ne remplace pas les pilotes ou conversations navigateur. Les signatures
MCP originales et les objets de métadonnées restent inchangés. Les tests vérifient
que le texte métier et le jeton ne partent pas au relais, tandis que le callable
reçoit bien la demande exacte. **1 251 non-régressions passent, avec un saut**.

Le premier reçu Open WebUI contient la commande constante dans l'argument
`question`, plutôt que le marqueur vide demandé dans le schéma de relais.
Cette différence est conservée : le callable ignore cette valeur et injecte
la vraie demande protégée, dont les six documents attendus sont vérifiés.
La conformité littérale au marqueur vide n'est pas revendiquée dans Open WebUI ;
les six essais API directs, qui ne passent pas par ce client, l'ont bien produit.
Les critères de la matrice métier restent inchangés.

Lors de l'arrêt v18, la campagne comptait **47 échanges conformes sur 48** ; les trois répétitions restaient à rejouer. Leur résultat v25 figure séparément ci-dessous.

## Résultats de la version v25

**Conforme sur la matrice exécutée : 96 échanges conformes sur 96 ; 9 fils longs conformes sur neuf.**
Campagne du 12/11/2026, de 17:53:04 à 18:57:23 (Europe/Paris) ; durée totale 64.3 minutes, pauses de cadence incluses.

| Jeu | Échanges | Conformes |
| --- | ---: | ---: |
| `pdl_reference` | 30 | 30 |
| `incidents_reference` | 30 | 30 |
| `mixed_variants` | 30 | 30 |
| `wrong_incident_request_in_pdl` | 3 | 3 |
| `wrong_pdl_request_in_incidents` | 3 | 3 |

| Répétition | Fil | Tours conformes | Changements explicites d’UC |
| --- | --- | ---: | ---: |
| 1 | `pdl_reference` | 10 / 10 | 0 |
| 1 | `incidents_reference` | 10 / 10 | 0 |
| 1 | `mixed_variants` | 10 / 10 | 3 |
| 2 | `pdl_reference` | 10 / 10 | 0 |
| 2 | `incidents_reference` | 10 / 10 | 0 |
| 2 | `mixed_variants` | 10 / 10 | 3 |
| 3 | `pdl_reference` | 10 / 10 | 0 |
| 3 | `incidents_reference` | 10 / 10 | 0 |
| 3 | `mixed_variants` | 10 / 10 | 3 |

### Contrôles de restitution et génération

Les 96 messages réellement sauvegardés dans Open WebUI ont été relus sans génération. Leurs textes correspondent aux réponses natives et aucune reprise opaque n’est affichée.
Les douze contrôles supplémentaires des filtres de comptage sont réussis : un total correct ne masque pas un ancien filtre d’équipement.
Open WebUI 0.11.3 est installé sur le Mac ; les moteurs tournent sur Ubuntu 24.04 ARM64.
Les configurations avant/après sont identiques. L’empreinte de référence est `9170c48e42ed2d62299bcb9cd74696e49f8c9a04bd56441250086efc2ee0fdf2`.

0 réponse(s) avec synthèse non validée ; 1 réponse(s) ayant nécessité une réparation bornée. Les refus légitimes, données manquantes et états factuels attendus restent distingués des diagnostics exigeant une rédaction.
Les usages retournés comptent 62 appels de rédaction/relecture et 291,769 tokens métier. Ils ne couvrent pas tous les appels de dialogue, de planification et de relais Open WebUI.

| Étape | Modèles effectivement retournés |
| --- | --- |
| draft | `gemma-4-31b-it` |
| review | `gpt-oss-120b` |

### Choix des modèles et réparation des sources

PDL utilise `gemma-4-31b-it` pour le dialogue et la planification. Incidents conserve `gemma-4-31b-it` pour la rédaction, `gpt-oss-120b` pour son dialogue et `gpt-oss-120b` pour la relecture.
Les sondes ci-dessous vérifient la production de plans et leur validation locale, sans lecture métier ni recette ACL. Elles sont distinctes des conversations et leur effectif est limité.

| Modèle demandé | Plans conformes | Première proposition injectée, réparation réelle |
| --- | ---: | ---: |
| `gemma-4-31b-it` | 9 / 9 | 1 / 1 |
| `gpt-oss-120b` | 2 / 3 | 1 / 1 |

Gemma est retenu pour la recette PDL de cette version. Le comparatif conserve la clarification non attendue de GPT-OSS ; les essais ne sont pas additionnés pour fabriquer un taux de fiabilité. Les durées des sondes sont dans les reçus JSON ; aucune chronologie par assertion n’a été enregistrée pour leur import Allure.
La réparation des sources utilise la même limite de deux propositions que la réparation des métadonnées. La dernière sélection reconnue dans le catalogue est validée avant le choix du pool d’identité. Une addition ou une contradiction courante n’est pas transformée arbitrairement en remplacement. Les cibles inconnues et les critères non pris en charge restent opposables.
Le rapport `openwebui-models-v25-20261112` présente les douze sondes normales et les deux réparations contrôlées, avec leur provenance. Les premières propositions injectées sont signalées et ne comptent jamais dans les 96 échanges.
Le timeout de planification et la clarification de sources de v24 restent des échecs. Le classement du timeout a été précisé sans modifier les assertions ni les verdicts ; l’audit compare les mêmes 96 échanges avant/après.

### Interprétation du dialogue

0 échange(s) ont nécessité la réparation bornée du format de décision. Les métadonnées comptent 81 appels d’interprétation connus et 107258 tokens chiffrés. Ces compteurs restent distincts de la rédaction/relecture et n’incluent pas le relais du client. Aucune lecture métier n’est lancée avant validation de la décision. La cause exacte du retour générique v20 reste indéterminée ; ses 49 réussites sur 50 ne sont pas reclassées.

### Temps de réponse mesurés

Les temps ci-dessous vont de l’envoi navigateur à la réponse terminée, avec les services du laboratoire et Albert à cette heure. Ils incluent le fournisseur et le client ; les pauses entre questions ne sont pas incluses dans ces percentiles.

| Parcours | p50 | p95 | p99 |
| --- | ---: | ---: | ---: |
| Tous | 11.84 s | 27.21 s | 39.42 s |
| pdl | 11.37 s | 14.45 s | 17.89 s |
| incidents | 16.48 s | 29.99 s | 39.42 s |

### État du laboratoire pendant cette campagne

121 relevés système, avec 0 erreur(s) de mesure. Aucun OOM ni redémarrage spontané constaté.
Occupation disque : variation mesurée de 28.19 Mo. La disponibilité métier et les douze contrôles techniques sont revérifiés après la campagne.

Il s’agit d’échanges séquentiels : aucun dimensionnement multiutilisateur, débit maximal ou nouvelle endurance de deux heures n’est déduit de ces mesures.

| Conteneur | Pic CPU échantillonné | Pic mémoire échantillonné |
| --- | ---: | ---: |
| `rie-60b04880a219-incidents-1` | 43.33 % | 97.9 Mio |
| `rie-60b04880a219-ldap-1` | 2.16 % | 6.1 Mio |
| `rie-60b04880a219-opensearch-1` | 2.60 % | 1201.2 Mio |
| `rie-60b04880a219-pdl-1` | 13.60 % | 73.8 Mio |
| `rie-60b04880a219-portal-1` | 13.66 % | 135.4 Mio |
| `rie-60b04880a219-victoriametrics-1` | 1.52 % | 34.6 Mio |
| `rie-itsm-u24-20261110-api` | 7.14 % | 86.9 Mio |
| `rie-itsm-u24-20261110-db` | 6.56 % | 157.2 Mio |

Mémoire disponible minimale de la VM : 13582 Mio. Les pics sont échantillonnés, pas une mesure continue des maxima.

### État des défauts et limites

OW-01 à OW-08 sont corrigés et leurs scénarios rejoués sur cette matrice. Cette clôture porte sur les contrôles et données décrits ; elle ne garantit pas toutes les formulations futures, toutes les affirmations d’un modèle ni la causalité d’un incident.

Les rapports NiceGUI `openwebui-corrections-v25-20261112`, `openwebui-fixes-units-v25-20261112` et `openwebui-controls-v25-20261112` séparent conversations, non-régressions et contrôles opérationnels. Le lecteur du rapport conversationnel passe 10 contrôles navigateur, dont l’ouverture du reçu de génération réel.

Le paquet de preuves contient le rapport détaillé des 96 échanges, un CSV par question, les réponses natives, les contrôles initiaux et supplémentaires, les empreintes, les relevés système et les rapports Allure autonomes. Les images des conversations restent privées ; les rapports exportés conservent leurs verdicts sans ces captures.

Les sources explicitement nommées sont vérifiées avec le catalogue actif avant compilation ; une omission du modèle peut être complétée uniquement pour ces cibles nommées. Une exclusion seule ne complète pas une sélection erronée. Les sources inconnues, signatures mal formées, ancres absentes, incompatibilités et refus ACL restent opposables. Ce contrôle ne crée ni mapping ni relation.

## Limites et preuves

Les reçus de chaque lot, les essais échoués et les nouvelles conversations
sont distincts sous `.runtime/ci/openwebui-fixes-20261111/`. La correction du
masquage des reprises PDL est documentée dans le
[bilan initial](18-openwebui-stability.md#lire-et-remettre-les-preuves).

Cette revalidation ne réattribue ni les anciennes deux heures d’endurance,
ni la compatibilité 26.04, ni une qualification AMD64 aux nouvelles images.
La réception avec annuaire, corpus, droits ITSM, réseau, registry et machine du
SI réel reste à effectuer selon la [matrice SI-RIE](14-si-rie-acceptance.md).
L’observabilité, Dify et n8n restent hors périmètre.
