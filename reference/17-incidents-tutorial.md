# Tutoriel Incidents — du diagnostic aux questions de suivi

> Complément de référence — le [guide principal](../README.md) reste le seul parcours de déploiement à exécuter.

**R1, R3, Rennes, les tickets et les comptes ci-dessous appartiennent à un jeu
fictif de démonstration.** Leur présence dans le SI-RIE n'est pas établie par ce
tutoriel. Ils sont disponibles dans une instance isolée en mode `demo` après
préparation de l'annuaire et chargement réussi du corpus ; le mode `external`
ne les crée pas dans les sources ou l'annuaire du SI.

| Votre instance | Parcours à suivre |
| --- | --- |
| Simulation `demo` initialisée | [Démonstration reproductible](#avant-de-commencer-la-démonstration) avec les noms, dates et valeurs du jeu fictif |
| Raccordement aux sources SI-RIE en `external` | [Utiliser vos sources SI-RIE](#utiliser-vos-sources-si-rie), avec un incident connu et les identités préparées par l'équipe du SI |
| Sandbox SI simulé préparée en `external` | Même parcours fictif ci-dessous, seulement si le préparateur confirme les sources et comptes de la [sandbox](22-external-sandbox.md) ; aucun `seed` en `external` |
| Mode ou données disponibles inconnus | Faire confirmer l'instance et ses sources par l'installateur avant d'envoyer les questions |

## Utiliser vos sources SI-RIE

### Choisir un incident connu et des sources vérifiables

Préparer avec l'équipe réseau et les responsables des sources un exemple
réel à comparer à des observations déjà connues. Les éléments suivants
doivent être renseignés pour votre instance :

| Élément nécessaire | Valeur à obtenir |
| --- | --- |
| Identité et client | Votre compte LDAP, son autorisation OAuth personnelle et le modèle Open WebUI connecté au MCP Incidents de l'instance |
| Équipement et site | Identifiants réels déclarés dans l'inventaire, interfaces et périmètre autorisés |
| Période | Date, bornes horaires et fuseau d'un incident ou d'une situation connue |
| Mesures | Séries réellement raccordées, unités, cadence et couverture attendue |
| Logs et documents | Noms réels des index configurés et exemples connus sur cette période |
| Tickets | Source de tickets réellement configurée, fraîcheur, identifiant d'un ticket connu et ses horodatages |
| Référence métier | Faits et chronologie relus par l'expert réseau ; hypothèses et causes confirmées distinguées |

La présence du PoC dans le SI-RIE ne signifie pas qu'un raccordement
VictoriaMetrics, OpenSearch ou ITSM est déjà opérationnel. Le diagnostic complet
nécessite ses quatre droits de source. L'historique ITSM est une extension
distincte, à préparer selon [son guide](15-itsm-precedents.md).
Voir aussi [Technique Incidents](05-incidents.md) et la
[matrice de raccordement SI-RIE](14-si-rie-acceptance.md).

### Adapter les questions à votre incident

Les blocs suivants sont des **modèles à compléter** : remplacer chaque champ
entre crochets avant de l'envoyer dans le modèle Incidents de votre client.

**Nouveau fil :**

```text
Diagnostique [EQUIPEMENT] le [DATE] de [HEURE_DEBUT] à [HEURE_FIN], [FUSEAU], sur le site [SITE_AUTORISE]. Cite les mesures, les logs et les tickets disponibles et indique les données manquantes.
```

**À vérifier :** les faits et les citations correspondent aux sources et à la
période préparées. La cadence réelle détermine le nombre de points attendu ;
aucun total de 120 logs ou de 120 points, ni maximum de 80 ms ou de 3 %, n'est
présumé sur les données du SI.

**Même fil**, pour séparer les deux horodatages :

```text
À [HEURE_DE_REFERENCE], [FUSEAU], l'alerte de cet équipement était-elle résolue et le ticket déjà clôturé ? Cite les horodatages disponibles.
```

**À vérifier :** les horaires proviennent du ticket réel. Si la résolution
technique ou la clôture n'est pas renseignée, la réponse doit l'indiquer.
Les valeurs 11:30 et 12:15 du jeu fictif ne sont pas des résultats attendus
sur cet incident.

Pour une comparaison de sites, choisir un périmètre réellement déclaré dans
l'inventaire. Pour l'exercice de données manquantes, utiliser une lacune de
collecte connue ou une simulation dédiée. Ne pas supposer que R3 ou un équipement
nommé de la même manière représente ce cas dans le SI.

Les boutons de démonstration NiceGUI décrits plus bas utilisent le scénario
fixe R1 et ses dates fictives. Pour cet exemple SI-RIE, jouer la question adaptée
dans Open WebUI ; ces boutons fixes ne valident pas l'incident choisi.

### Vérifier les ACL avec la politique réellement configurée

Faire identifier les deux index configurés pour les logs et documents, ainsi
que les droits `incidents.metrics` et `incidents.tickets`. Les identifiants de
ces deux ressources sont ceux du logiciel ; les noms d'index et les règles
initiales du jeu fictif ne décrivent pas nécessairement votre installation.

Préparer avec l'administrateur un **lecteur disposant des quatre droits**, un
**compte de recette privé d'au moins un de ces droits** et un **administrateur
des accès**. Chaque lecteur utilise sa propre autorisation OAuth, selon l’[étape 10](../README.md#10-connecter-nicegui-et-open-webui-local). Rejouer le diagnostic
complet préparé : accès selon la politique pour le premier, refus sans preuves
des sources pour le second. Un externe n'est pas systématiquement interdit :
le résultat dépend des règles effectivement configurées et de son identité vérifiée.

Pour éprouver un retrait de droit dans une session ouverte, réserver une instance
ou des sources de recette avec l'administrateur. Relever la règle initiale,
contrôler le refus, puis restaurer cette règle et vérifier une nouvelle lecture.
Les clics de l'[étape ACL de démonstration](#9-vérifier-les-acl-incidents)
illustrent le mécanisme sur les sources fictives ; ils ne désignent pas une
modification à effectuer sur une source partagée du SI.

Consigner les observations dans la [fiche de recette](12-manual-acceptance.md).
Un rapport sur le corpus fictif prouve le parcours pédagogique, pas la qualité
du diagnostic ou du raccordement sur ces sources réelles.

## Avant de commencer la démonstration

**La suite utilise le corpus fictif initialisé en `demo`, ou celui de la sandbox `external` préparée à l’avance.**
Ce parcours apprend à diagnostiquer un équipement, distinguer résolution
technique et clôture d'un ticket, comparer deux équipements, reconnaître
des mesures manquantes et vérifier les droits sur les sources.
Prévoir **15 à 20 minutes**, selon les temps de réponse, puis environ cinq minutes
pour l'[exercice ACL](#9-vérifier-les-acl-incidents).
Pour une présentation courte, jouer les étapes 1, 2 et 4, puis montrer le refus
de l'externe à l'étape 9.

Dans votre **Open WebUI local**, sélectionner **SI-RIE — INCIDENTS**, nom affiché par la configuration livrée ; l’identifiant technique reste `albert-observabilite-gptoss`. Un ancien client peut avoir un autre libellé. Utiliser son autorisation OAuth personnelle et
un compte interne disposant des quatre droits : logs, documents, mesures et
tickets. Voir la [configuration du client](02-configuration.md#configurer-le-client-open-webui-local-qualifi%C3%A9).

Le lecteur de référence est `bruno.lecteur`. L'exercice ACL utilise aussi
`eric.durand.ext` (externe) et `alice.martin` (administratrice), dans des sessions
distinctes. Ce sont les identités fictives de la simulation ; être lecteur
interne suffit pour diagnostiquer les sources autorisées.

Le corpus fictif Ubuntu est fixé au **28 septembre 2026**. Les exemples portent
sur **10:00–12:00, heure de Paris** : R1 et R2 sont sur le **site de Rennes**,
R3 sur **Beta**, R4 est exclu. Ces dates sont différentes de celles du tutoriel PDL.
Les résultats indiqués sont les **attendus du corpus de référence**, à comparer
à la réponse reçue ; ils ne remplacent pas une exécution réelle.

Faire confirmer par l'installateur que l'[installation `demo`](01-installation.md#choisir-une-variante-avant-de-lexécuter)
a initialisé l'annuaire et le corpus Incidents. La présence de cette page dans
une instance `external` ne fournit ni R1, ni les tickets, ni les comptes de démo.
Leurs mots de passe sont générés par instance, selon le
[guide de connexion fictive](06-acl.md#préparer-une-connexion-fictive).

Dans NiceGUI, **Copier la question** copie le texte ; le coller et l'envoyer
manuellement dans Open WebUI. Commencer un nouveau fil lorsque l'étape le demande
et conserver le même fil pour les relances. Attendre au moins **60 secondes après
la fin de la réponse** avant de demander une nouvelle synthèse Incidents.
En cas de limitation, respecter le délai annoncé avant une relance manuelle.

## 1. Diagnostiquer R1 et lire les sources

**Nouveau fil.** Donner l'équipement, le lieu, la date et les heures.

```text
Diagnostique R1 le 28 septembre 2026 de 10:00 à 12:00, heure de Paris, sur le site de Rennes. Cite les sources et distingue résolution technique et clôture du ticket.
```

**À observer :** cinq mesures (CPU, latence, pertes, utilisations entrante et
sortante), leurs unités et leur couverture, les logs, le ticket et les fiches
documentaires citées. La référence R1 comporte **120 logs**, une latence
maximale de **80 ms** et des pertes maximales de **3 %** sur cette fenêtre.

Python calcule les faits ; le modèle rédige et relit la synthèse à partir des
preuves. Des signaux compatibles avec une congestion ne démontrent pas à eux
seuls une cause. Les [diagrammes Incidents](03-architecture.md#uc-incidents--composants)
montrent la place des différentes sources.

## 2. Distinguer la résolution de l'alerte et la clôture du ticket

**Même fil que l'étape 1.** Poser une question ciblée sur le diagnostic précédent.

```text
À 12 h, heure de Paris, l'alerte était-elle résolue et le ticket déjà clôturé ?
```

**Attendu pour R1 :** alerte techniquement résolue à **11:30**, ticket clôturé
à **12:15**. À 12:00, l'alerte est donc résolue mais le ticket n'est pas encore
clôturé. Retrouver les horodatages dans la source ticket ; le statut final
`closed` du snapshot ne doit pas être appliqué rétroactivement à midi.

## 3. Comparer les équipements du site de Rennes

**Nouveau fil.** Passer d'un équipement à un périmètre.

```text
Compare les équipements du site de Rennes le 28 septembre 2026 de 10:00 à 12:00, heure de Paris. Distingue les mesures, les alertes et les logs de chaque équipement.
```

**À observer :** R1 et R2 sont traités séparément. R1 présente des signaux
compatibles avec une congestion ; R2 présente une instabilité de liaison.
Les mesures et chronologies ne doivent pas être fusionnées en un équipement
fictif. R3, sur Beta, et R4, exclu, ne font pas partie de ce périmètre.

## 4. Reconnaître les données manquantes de R3

**Nouveau fil.** Demander une mesure précise plutôt qu'un diagnostic général.

```text
Pour R3 le 28 septembre 2026 de 10:00 à 12:00, heure de Paris, combien de points de latence sont disponibles et quelle est la latence moyenne ?
```

**Attendu : 0 point sur 120 attendus**, avec une latence moyenne **inconnue**,
pas égale à zéro. Le corpus simule un trou de collecte. Les logs et tickets
éventuellement disponibles restent utiles, mais ne remplacent pas les mesures
manquantes. La réponse doit expliquer cette limite.

## 5. Compléter une demande dans le même fil

**Nouveau fil.** Donner l'équipement mais omettre volontairement la période.

```text
Diagnostique R1 sur le site de Rennes.
```

**Attendu :** une demande de précision sur la période, sans produire un diagnostic
fondé sur des dates choisies implicitement. Répondre dans **le même fil** :

```text
Le 28 septembre 2026 de 10:00 à 12:00, heure de Paris.
```

**À observer :** le serveur conserve R1 et Rennes, complète les dates et effectue
une nouvelle lecture. La clarification est liée à votre identité et à ce fil.
Si elle a expiré, envoyer une question complète dans une nouvelle conversation.

## 6. Comprendre un refus de périmètre

**Nouveau fil.** Tester l'équipement explicitement exclu de la simulation.

```text
Diagnostique R4 le 28 septembre 2026 de 10:00 à 12:00, heure de Paris.
```

**Attendu : un refus**, sans diagnostic, mesures, logs ou ticket de R4. Ce refus
est une limite de périmètre ; il ne décrit pas l'état de santé de l'équipement.

Le refus de R4 vient de l'inventaire des équipements autorisés de cette
simulation. Il ne démontre pas à lui seul les droits d'un utilisateur.
L'[étape 9](#9-vérifier-les-acl-incidents) compare les identités et les droits
des sources sur **R1**, un équipement pourtant autorisé dans l'inventaire.

## 7. Générer le diagnostic dans NiceGUI avec Albert API

Sur la page NiceGUI Incidents, se connecter puis cliquer sur
**Diagnostiquer avec Albert API**. Ce bouton lit les sources et demande la
rédaction puis la relecture par Albert. Contrôler le modèle effectivement
retourné, les étapes et les consommations de tokens dans le rapport structuré.
Une erreur ou une synthèse non validée ne compte pas comme un succès IA.

Le volet **Contrôle technique des sources, sans génération IA** conserve le
bouton **Lire les preuves maintenant, sans génération IA** pour isoler une
panne de source. Il ne valide pas le parcours complet avant déploiement.
Ces boutons utilisent les sources
autorisées de **R1, 10:00–12:00, le 28 septembre 2026**, avec votre identité.
Comparer les faits au rapport de l'étape 1.

Le contrôle technique permet d'examiner les sources lorsque la synthèse est
indisponible. Les questions libres et les relances de ce tutoriel se jouent
dans Open WebUI ; le bouton NiceGUI garde son scénario R1 fixe.

## 8. Examiner les précédents et une solution proposée

**Extension facultative :** cette étape suppose l’historique ITSM activé par
l’opérateur et les droits individuels du lecteur préparés. Attendre le délai
de synthèse, puis **nouveau fil, modèle SI-RIE — INCIDENTS** :

```text
Diagnostique R1 le 28 septembre 2026 de 10:00 à 12:00, heure de Paris, sur le site de Rennes. Recherche les précédents ITSM-NG disponibles et présente la solution la mieux étayée, ses conditions et les niveaux de confiance du diagnostic et de la solution. Cite les sources et distingue résolution technique et clôture du ticket.
```

Comparer au diagnostic de l’étape 1. Dans NiceGUI, le bouton **Diagnostiquer
avec Albert API** rejoue ce scénario fixe avec génération. La lecture technique
« sans génération IA » ne remplace pas ce test du rapport généré.

Vérifier les tickets cités, les points de similitude et les différences avec
l'incident courant. La **confiance dans la solution proposée** est distincte
de la confiance dans le diagnostic et doit être expliquée. Une solution reste
à valider par un opérateur ; le tutoriel n'exécute aucune remédiation.

Si cette extension n'est pas activée, ne pas attendre de précédents ITSM.
Le [guide ITSM-NG](15-itsm-precedents.md) décrit sa configuration, son jeu de
démonstration et sa recette séparée.

## 9. Vérifier les ACL Incidents

**Objectif :** comprendre pourquoi un lecteur peut diagnostiquer R1 tandis
qu'un autre est refusé, puis constater le retrait et le rétablissement d'un
droit dans une session déjà ouverte. L'exercice utilise le bouton de lecture
des preuves de NiceGUI et ne nécessite pas de synthèse IA.

### Identifier les quatre droits nécessaires

Dans **ACL**, se connecter comme Bruno et cliquer sur **Afficher mes droits
actuels**. Le diagnostic complet exige les quatre autorisations ci-dessous.
Les noms d'index sont ceux de la simulation fournie.

| Source | Règle vérifiée | Politique initiale |
| --- | --- | --- |
| Logs réseau | Index `pdl-zabbix-observability-demo` | `internal` |
| Fiches documentaires | Index `obs-network-docs` | `internal` |
| Mesures VictoriaMetrics | Ressource `incidents.metrics` | `internal` |
| Tickets et suivi ITSM, y compris l'historique facultatif | Ressource `incidents.tickets` | `internal` |

Les deux dernières règles sont des **sources Incidents**, pas des index
OpenSearch. Donner accès aux seuls logs ne suffit donc pas à autoriser le
diagnostic complet.

### Comparer Bruno et Eric sur le même diagnostic

1. Dans une session NiceGUI de Bruno (`bruno.lecteur`, interne lecteur), ouvrir
   **Incidents**, puis cliquer sur **Lire les preuves maintenant, sans génération IA**.
   Attendu : les preuves R1 de l'étape 7, dont cinq mesures et 120 logs.
2. Dans une autre session, se connecter comme Eric (`eric.durand.ext`, externe)
   et cliquer sur le même bouton. Attendu avec la politique initiale : **refus**,
   sans mesures, logs, tickets ou documents dans le résultat.
3. Vérifier que Bruno et Eric ne peuvent ni ouvrir l'administration des règles
   ni consulter les rapports de qualification réservés aux administrateurs.

Pour jouer le même contrôle depuis Open WebUI, utiliser deux sessions utilisateur avec chacune **son autorisation OAuth personnelle**.
Une seconde fenêtre qui conserve la même autorisation conserve aussi la même identité.
Dans un **nouveau fil Incidents pour chacun**, envoyer :

```text
Diagnostique R1 le 28 septembre 2026 de 10:00 à 12:00, heure de Paris, sur le site de Rennes.
```

**Attendu :** diagnostic autorisé pour Bruno, refus pour Eric. Une demande
formulée autrement ou une déclaration de rôle dans le texte ne donne aucun droit.
Respecter l'intervalle entre synthèses indiqué au début du tutoriel.

### Retirer puis rétablir les mesures dans une session ouverte

1. Garder la session Incidents de Bruno ouverte après une lecture réussie.
2. Dans une troisième session, se connecter comme Alice (`alice.martin`) et
   ouvrir **Administration ACL**. Cliquer sur **Actualiser les règles**, puis
   noter la règle de **Mesures VictoriaMetrics** et la révision active.
3. Choisir **Source Incidents**, puis **Mesures VictoriaMetrics** dans **Source**.
   Choisir **Interdit à tous** (`blocked`) dans **Accès**, puis **Enregistrer la règle**.
   Vérifier la nouvelle règle et la révision ; actualiser en cas de conflit.
4. Dans la session de Bruno, cliquer à nouveau sur **Lire les preuves maintenant,
   sans génération IA**, sans se déconnecter. Attendu : **refus du diagnostic
   complet**, sans preuve des quatre sources. Alice serait également refusée
   pour cette lecture : son rôle administratif ne contourne pas `blocked`.
5. Dans la session Alice, actualiser puis rétablir **la valeur initialement notée**,
   `internal` dans la simulation initiale. Vérifier l'enregistrement et la révision.
6. Dans la même session Bruno, relancer la lecture. Attendu : nouvelles preuves
   autorisées. Les anciennes valeurs affichées avant le blocage ne doivent pas
   être présentées comme une nouvelle lecture.

**Équivalent dans Open WebUI :** modèle **SI-RIE — INCIDENTS**, conserver
le fil du compte testé. Après chaque changement de règle enregistré
(blocage, puis rétablissement), envoyer à nouveau la **demande complète** :

```text
Diagnostique R1 le 28 septembre 2026 de 10:00 à 12:00, heure de Paris, sur le site de Rennes.
```

Pendant le blocage : refus sans preuve des sources. Après rétablissement :
nouvelles preuves pour le seul compte autorisé, après le délai d’admission.
Le même texte sert au cycle Tickets et suivi ITSM ci-dessous.

Un droit retiré refuse le diagnostic ; une source autorisée mais en panne peut
produire un rapport partiel signalant l'erreur. Ces deux situations ne doivent
pas être confondues. Les anciens échanges Open WebUI peuvent rester dans
l'historique, mais le nouveau résultat ne doit pas republier des preuves refusées.

### Vérifier aussi la protection des précédents ITSM

Si l'historique de l'étape 8 est activé, refaire l'exercice précédent en
sélectionnant **Tickets et suivi ITSM** (`incidents.tickets`). Noter et restaurer
sa propre règle initiale avant de terminer.

Avec cette source à `blocked`, le nouveau diagnostic doit être refusé et ne
pas republier les précédents, les solutions ou les suivis de l'ancien rapport.
Passer cette seule règle à `authenticated` ne suffit pas à ouvrir le diagnostic
à Eric : les trois autres autorisations restent nécessaires. Sur un raccordement
ITSM réel, les droits natifs sur les profils, entités et suivis privés s'appliquent
également ; la règle du PoC ne les remplace pas.

**À retenir :** l'identité LDAP et la politique sont relues à chaque appel.
Les ACL standard protègent les index et les sources, pas des équipements ou des
sites au cas par cas pour chaque utilisateur. Le refus de R4 à l'étape 6 et le
refus d'Eric sur R1 vérifient donc deux contrôles différents.
Le [schéma des ACL communes](03-architecture.md#frontières-de-confiance) et le
[guide d'administration](06-acl.md#lire-et-modifier-la-politique-active) donnent
les détails du mécanisme.

## Si la réponse n'est pas celle attendue

| Situation | Prochaine action |
| --- | --- |
| Une précision manque | Répondre dans le même fil en donnant l'équipement ou la période demandés |
| Aucun appel à l'outil Incidents | Vérifier le modèle dédié et sa connexion MCP dans Open WebUI |
| Une source est indisponible | Lire les erreurs et la couverture ; ne pas assimiler une absence de données à un réseau sain |
| Synthèse non validée | Examiner les faits et la limite affichée ; ne pas présenter la synthèse comme confirmée |
| Limitation du fournisseur IA | Attendre le délai annoncé, puis relancer manuellement |
| Pas de lien Grafana | Utiliser les preuves du rapport ; le lecteur Grafana est facultatif dans cette livraison |

Poursuivre avec le [tutoriel PDL](16-pdl-tutorial.md), la
[technique Incidents](05-incidents.md) ou les [règles d'accès](06-acl.md).
