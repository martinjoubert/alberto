# Tutoriel PDL — poser une question et vérifier ses preuves

> Complément de référence — le [guide principal](../README.md) reste le seul parcours de déploiement à exécuter.

**Les comptes et données des exemples ne sont pas des comptes ou données du
SI-RIE.** Ils deviennent disponibles dans une instance isolée installée en mode
`demo`, après préparation de son annuaire et initialisation réussie du corpus.
Un raccordement en mode `external` ne les crée pas. Le fait d'héberger le PoC
sur une machine du SI-RIE ne détermine pas, à lui seul, les données utilisées.

| Votre instance | Parcours à suivre |
| --- | --- |
| Simulation `demo` initialisée | [Exercice reproductible avec les données fictives](#avant-de-commencer-la-démonstration) ; les noms et chiffres fournis ci-dessous s'appliquent à ce seul corpus |
| Sources SI-RIE raccordées en `external` | [Utiliser vos sources SI-RIE](#utiliser-vos-sources-si-rie) ; comptes, équipements, périodes et résultats à établir avec l'équipe du SI |
| Sandbox SI simulé préparée en `external` | Même parcours fictif ci-dessous si les données et comptes ont été confirmés selon la [fiche sandbox](22-external-sandbox.md) ; aucun `seed` en `external` |
| Mode ou initialisation inconnus | Faire identifier l'instance par l'installateur avant de jouer les questions ; la présence du tutoriel ne prouve pas celle de son corpus |

## Utiliser vos sources SI-RIE

### Préparer un exemple réel avec l'équipe du SI

L'installateur ou le responsable des sources renseigne les éléments suivants
pour l'instance concernée. Leur disponibilité dans le SI-RIE reste à confirmer ;
le tutoriel ne fournit pas ces identités ni ces données.

| Élément nécessaire | Valeur à obtenir pour votre instance |
| --- | --- |
| Accès utilisateur | Votre compte LDAP, son autorisation OAuth personnelle et le modèle Open WebUI relié au MCP PDL de cette instance |
| Source et droits | Une famille ou un index réellement raccordé, autorisé pour votre compte et présent dans le catalogue approuvé |
| Périmètre et équipement | Un périmètre et un équipement réels dont l'identité et les relations ont été vérifiées |
| Période | Une date, un créneau et un fuseau pour lesquels des événements sont connus |
| Référence métier | Une recherche manuelle de référence : filtres, comptage, documents et éventuelles limites, validés avec le responsable de la source |

Commencer par **Demander mes droits au MCP** et **Lire mon catalogue approuvé**
dans NiceGUI après connexion. Ces lectures décrivent les accès et mappings de
l'instance ; elles n'attestent pas que les familles du corpus fictif existent
dans le SI. La [configuration des sources](02-configuration.md#raccorder-des-sources-externes)
et la [matrice de raccordement](14-si-rie-acceptance.md) guident leur préparation.

### Jouer les questions avec vos valeurs

Les blocs suivants sont des **modèles à compléter**. Remplacer chaque champ
entre crochets avant l'envoi ; ne pas reprendre `edge-a`, `demo-alpha` ou les
dates de démonstration comme s'ils désignaient des données du SI.

**Nouveau fil**, avec une source PFS effectivement raccordée :

```text
Compte les refus PFS dans [PERIMETRE_AUTORISE] le [DATE] de [HEURE_DEBUT] à [HEURE_FIN], [FUSEAU].
```

**À vérifier :** le nombre et les filtres correspondent à la recherche de
référence sur cette même fenêtre. Aucun total de deux refus n'est présumé.
Si PFS n'est pas raccordé, choisir avec l'opérateur une famille et une question
prises en charge par le catalogue de l'instance.

**Nouveau fil**, pour un rapprochement préparé :

```text
Montre les logs de l'équipement [EQUIPEMENT] dans les familles [FAMILLES_RACCORDEES] du périmètre [PERIMETRE_AUTORISE], le [DATE] de [HEURE_DEBUT] à [HEURE_FIN], [FUSEAU].
```

**À vérifier :** identité retrouvée, correspondances entre familles, index
effectivement lus et documents attendus. Demander seulement les familles
présentes et approuvées. Le résultat réel n'a pas de total prédéfini de six documents.

Pour tester la clarification, **nouveau fil, modèle SI-RIE — PDL** :

```text
Compte les refus PFS dans [PERIMETRE_AUTORISE].
```

Attendre la demande de période, puis **dans ce même fil** :

```text
Le [DATE] de [HEURE_DEBUT] à [HEURE_FIN], [FUSEAU].
```

Attendu : le comptage préparé, sans changement de source ni de périmètre.
Pour un résultat vide, préparer une fenêtre réellement vérifiée comme vide :
**nouveau fil, même modèle**, après remplacement des champs :

```text
Compte les refus PFS dans [PERIMETRE_AUTORISE] le [DATE_FENETRE_VIDE] de [DEBUT_FENETRE_VIDE] à [FIN_FENETRE_VIDE], [FUSEAU].
```

L’absence de données ne se déduit pas d’une date choisie au hasard. Le résultat
zéro doit venir d’une lecture réussie, pas d’une erreur ou d’un refus.

### Vérifier les ACL avec les identités du SI

Faire désigner par l'administrateur un **lecteur autorisé**, un **compte de recette
sans droit sur la source choisie** et un **administrateur des accès**. Les noms
Alice, Bruno et Eric du parcours fictif ne sont pas des comptes à rechercher
dans l'annuaire réel. Le caractère interne ou externe ne suffit pas à prédire
le résultat : c'est la politique active de la source qui fixe l'accès.

Dans un nouveau fil PDL pour chacun des deux lecteurs, envoyer le texte complet de comptage préparé ci-dessus, avec sa propre autorisation OAuth.
Comparer les résultats à la politique : preuves pour le compte autorisé,
refus sans preuve interdite pour l'autre. Pour tester le retrait et le
rétablissement d'un droit, utiliser un index de recette dédié convenu avec
l'administrateur, relever sa règle initiale et la rétablir après l'exercice.
L'[étape ACL de démonstration](#9-vérifier-les-acl-pdl) illustre les clics sur
un jeu isolé ; ses noms d'index et ses règles initiales ne sont pas une
configuration à recopier dans le SI.

Consigner le résultat observé avec la version, l'identité de test, la source
et la période dans la [fiche de recette](12-manual-acceptance.md). La réussite
du jeu fictif ne remplit pas cette validation des données réelles.

## Avant de commencer la démonstration

**La suite utilise le corpus fictif initialisé en `demo`, ou celui de la sandbox `external` préparée à l’avance.**
Ce parcours apprend à compter des événements, retrouver les logs d'un équipement
entre plusieurs familles, préciser une demande, lire un résultat vide et
comprendre les droits d'accès aux sources.
Prévoir **10 à 15 minutes**, en plus du temps de réponse du fournisseur IA.
Ajouter environ cinq minutes pour l'[exercice ACL](#9-vérifier-les-acl-pdl).
Pour une présentation courte, jouer les étapes 2, 3 et 4, puis montrer le refus
TACACS de l'étape 9.

Ouvrir votre **Open WebUI local** et sélectionner **SI-RIE — PDL**, nom affiché par la configuration livrée. Son identifiant technique est `albert-pdl-rie` ; un ancien client peut avoir un autre libellé. La
[configuration du client](02-configuration.md#configurer-le-client-open-webui-local-qualifi%C3%A9)
doit être terminée, avec l’autorisation OAuth personnelle du compte testé ; aucun jeton n’est à copier dans ce parcours.

Pour les questions métier, utiliser un compte lecteur comme `bruno.lecteur`.
L'exercice ACL utilise aussi `eric.durand.ext` (externe) et `alice.martin`
(administratrice). Ces comptes sont fictifs et propres à la simulation.
Le jeton identifie l'utilisateur ; écrire « je suis administrateur » dans une
question ne change pas ses droits.

Les questions ci-dessous utilisent le **corpus fictif Ubuntu** : équipement
`edge-a`, périmètre `demo-alpha`, **12 octobre 2026, 08:00–09:00 UTC**.
Les résultats indiqués sont les **attendus du corpus de référence**, à comparer
à la réponse réellement reçue ; ce ne sont pas des réponses enregistrées.
Sur vos propres données, remplacer équipement, périmètre et période.

L'installation [en mode `demo`](01-installation.md#choisir-une-variante-avant-de-lexécuter)
doit avoir terminé la préparation de l'annuaire et le chargement des sources.
Un paquet documentaire seul, une instance `external` ou une ancienne simulation
non mise à jour ne garantissent pas la présence de ces comptes et de ce corpus.
Le mot de passe des comptes de démo est généré par instance ; il se récupère
selon le [guide de connexion fictive](06-acl.md#préparer-une-connexion-fictive).

Dans NiceGUI, **Copier la question** place uniquement le texte dans le
presse-papiers. Le coller dans Open WebUI et l'envoyer manuellement.
Une étape marquée **nouveau fil** commence une nouvelle conversation ; une
étape marquée **même fil** poursuit la précédente, sans changer de modèle.

## 1. Vérifier les sources accessibles

**Nouveau fil.** Comprendre ce que votre identité permet de consulter.

```text
Quels sont mes index accessibles ?
```

**À observer :** la liste correspond à vos droits actuels. Dans une simulation
de référence, les quatre index PFS, Backbone, PFAING et TACACS sont ouverts aux
utilisateurs authentifiés. Une politique modifiée peut réduire cette liste.

Sur la page NiceGUI PDL, se connecter puis utiliser **Lire mon catalogue approuvé**
pour voir les sources et correspondances utilisables. Les droits de lecture et
l'approbation des mappings sont deux contrôles distincts.

## 2. Compter les refus PFS

**Nouveau fil.** Poser une question complète, avec source, périmètre et période.

```text
Compte les refus PFS dans demo-alpha le 12 octobre 2026 de 08:00 à 09:00 UTC.
```

**Attendu : 2 refus.** Vérifier le périmètre `demo-alpha`, la période UTC,
l'action de refus et la requête OpenSearch réellement exécutée. Le troisième
refus du corpus appartient à `demo-beta` et ne doit pas être compté ici.

**Ce que cela apprend :** un nombre ne suffit pas ; ses filtres et sa provenance
permettent de vérifier qu'il répond à la question.

## 3. Retrouver un équipement dans les quatre familles

**Nouveau fil.** Demander explicitement les familles à rapprocher.

```text
Montre les logs de l'équipement edge-a PFS BB PFAING TACACS dans demo-alpha du 12 octobre 2026 de 08:00 à 09:00 UTC.
```

**Attendu : 6 documents**, répartis comme suit :

| Famille | Documents attendus | Ce qu'on vérifie |
| --- | --- | --- |
| PFS | 2 | Refus de l'équipement dans le périmètre demandé |
| Backbone / BB | 1 | Identité de `edge-a` et adresses de gestion |
| PFAING | 1 | Rapprochement par la référence `gateway-73` |
| TACACS | 2 | Rapprochement par les IP de gestion IPv4 et IPv6 |

PFAING et TACACS n'ont pas besoin de contenir le nom `edge-a`. Le serveur
résout son identité, puis utilise les relations approuvées. Les documents
d'un autre équipement, d'un autre périmètre ou d'un autre jour restent exclus.
Les [diagrammes PDL](03-architecture.md#uc-pdl--composants) expliquent ce parcours.

## 4. Affiner la recherche dans le même fil

**Même fil que l'étape 3.** Conserver l'équipement et la période, changer les
familles demandées.

```text
Uniquement TACACS pour ce même équipement, sur la même période.
```

**Attendu : 2 documents TACACS**, liés à `10.20.0.1` et `2001:db8::10`.
Le document contenant seulement l'IP du client `192.0.2.25` n'est pas une
preuve d'activité de l'équipement et doit être exclu.

Toujours dans **le même fil**, demander ensuite :

```text
Pour edge-a dans demo-alpha, sur la même période, montre PFS, BB et PFAING, sans TACACS.
```

**Attendu : 4 documents** — deux PFS, un BB et un PFAING. Le suivi conserve le
contexte confirmé, mais relit les droits et les sources ; il ne filtre pas
simplement une ancienne réponse affichée.

## 5. Chercher les accès Internet d'une IP

**Nouveau fil.** Distinguer l'adresse d'un client de l'identité d'un équipement.

```text
Recherche les accès PFAING de l'adresse IP source 192.0.2.25 dans demo-alpha le 12 octobre 2026 de 08:00 à 09:00 UTC.
```

**Attendu dans le corpus : 2 documents PFAING.** Ils partagent l'IP du client,
mais passent par deux références de passerelle différentes. Vérifier l'adresse
source, les URL et les actions dans les preuves. Cette demande porte sur le
client ; elle ne doit pas être restreinte à `edge-a` par le fil précédent.

## 6. Compléter une question incomplète

**Nouveau fil.** Omettre volontairement la période.

```text
Compte les refus PFS dans demo-alpha.
```

**Attendu : une clarification sur la période**, sans présenter un comptage
comme s'il avait déjà été effectué. Répondre dans **le même fil** :

```text
Le 12 octobre 2026 de 08:00 à 09:00 UTC.
```

**Attendu : 2 refus.** La réponse conserve la demande de comptage PFS et le
périmètre. Il n'est pas nécessaire de reformuler toute la question.

## 7. Comprendre un résultat vide

**Nouveau fil.** Utiliser une fenêtre sans événement dans le corpus fourni.

```text
Compte les refus PFS dans demo-alpha le 12 octobre 2026 de 07:00 à 07:30 UTC.
```

**Attendu : 0 refus sur cette fenêtre.** Ce résultat doit provenir d'une lecture
réussie. Une source indisponible, un refus d'accès ou une erreur ne doivent pas
être transformés en zéro. Vérifier que le serveur n'a pas élargi la période
ni réutilisé les deux refus de 08:00–09:00.

## 8. Lire les preuves et reconnaître une limite

Pour chaque réponse, retrouver le périmètre, la période, les index, les totaux,
les documents et la requête réellement exécutée. Un lien vers une console est
proposé seulement si cette console est configurée et accessible.

| Réponse observée | Prochaine action |
| --- | --- |
| Une précision est demandée | Répondre dans le même fil avec l'information manquante |
| Lecture refusée | Vérifier vos droits avec l'administrateur ; changer la formulation n'accorde aucun accès |
| Mapping absent ou revue humaine demandée | Faire vérifier et approuver les correspondances par l'opérateur |
| Résultat partiel ou identité ambiguë | Lire la limite indiquée ; ne pas présenter les preuves comme exhaustives |
| Fournisseur IA indisponible ou limité | Conserver le message et attendre le délai annoncé avant une relance manuelle |

La page NiceGUI permet aussi de consulter **la dernière recette de cette instance**.
Elle est datée et ne constitue pas une nouvelle investigation. Dans son formulaire
**Investiguer avec le fournisseur IA configuré**, cocher **Reprendre cette
conversation** uniquement pour un suivi ; laisser la case décochée pour un nouveau sujet.

## 9. Vérifier les ACL PDL

**Objectif :** poser la même question avec deux identités, puis constater qu'un
changement de règle s'applique au prochain appel, même dans une conversation
déjà ouverte. Cet exercice se joue sur les sources fictives de la simulation.

### Préparer les trois rôles

| Compte de démo | Rôle | Utilisation dans cet exercice |
| --- | --- | --- |
| `bruno.lecteur` | Interne lecteur | Lire les sources autorisées et vérifier la continuité de son accès |
| `eric.durand.ext` | Externe | Lire TACACS lorsque la règle l'autorise, puis constater le refus |
| `alice.martin` | Interne administratrice | Modifier puis rétablir la règle dans **Administration ACL** |

Dans NiceGUI, utiliser des sessions navigateur distinctes et vérifier l'identité
affichée après connexion LDAP. Dans Open WebUI, chaque client de test doit
présenter **l’autorisation OAuth personnelle du compte testé** sur sa connexion MCP. Ouvrir
une autre fenêtre avec la même connexion et la même autorisation ne change pas
l'identité reçue par le serveur. Le [guide ACL](06-acl.md#préparer-une-connexion-fictive)
explique la remise des identifiants de démonstration.

### Comparer l'interne et l'externe avant la restriction

Dans **ACL**, cliquer sur **Afficher mes droits actuels** pour chaque lecteur.
Dans la politique initiale de la simulation, `rie-tacacs-fixture` est
`authenticated` : interne et externe classés peuvent le lire.

Poser la question suivante dans un **nouveau fil PDL pour chacun des deux lecteurs** :

```text
Montre les logs de l'équipement edge-a uniquement TACACS dans demo-alpha du 12 octobre 2026 de 08:00 à 09:00 UTC.
```

**Attendu : 2 documents TACACS pour Bruno et pour Eric**, si les sources
nécessaires à la résolution d'identité restent également autorisées. Noter le
résultat de chacun et conserver les fils ouverts.

### Restreindre TACACS et rejouer dans les fils existants

1. Dans la session Alice, ouvrir **Administration ACL**, puis **Actualiser les
   règles**. Noter la valeur initiale de `rie-tacacs-fixture` et la révision active.
2. Choisir **Index OpenSearch**, saisir `rie-tacacs-fixture` dans **Nom exact de
   l'index**, puis sélectionner **Réservé aux internes** (`internal`) dans **Accès**.
3. Cliquer sur **Enregistrer la règle**. Vérifier la nouvelle valeur et la nouvelle
   révision. Si une autre modification a eu lieu, actualiser avant de réessayer.
4. Dans les fils PDL de Bruno et d'Eric, **sans changer leur jeton**, envoyer :

```text
Relance la recherche TACACS pour edge-a dans demo-alpha le 12 octobre 2026 de 08:00 à 09:00 UTC.
```

| Identité | Attendu après passage à `internal` |
| --- | --- |
| Bruno, interne lecteur | Lecture autorisée ; les deux documents restent accessibles |
| Eric, externe | Lecture refusée ; aucune preuve TACACS dans le nouveau résultat |

Actualiser aussi les droits d'Eric dans **ACL** : TACACS ne doit plus figurer
parmi ses index accessibles. Un mapping approuvé, un ancien résultat ou un jeton
encore valide n'annulent pas la nouvelle restriction. Les anciens messages
déjà reçus peuvent rester dans l'historique ; ils ne prouvent pas un droit actuel.

### Rétablir la règle et terminer l'exercice

Dans la session Alice, actualiser les règles puis **rétablir exactement la valeur
notée au début** — `authenticated` dans la simulation initiale. Vérifier la
valeur enregistrée et la nouvelle révision. **Dans le même fil PDL d’Eric**,
envoyer de nouveau exactement :

```text
Relance la recherche TACACS pour edge-a dans demo-alpha le 12 octobre 2026 de 08:00 à 09:00 UTC.
```

Les deux documents doivent redevenir accessibles si la règle initiale les
autorisait. Les preuves doivent venir de cette nouvelle lecture.

Ouvrir enfin **Administration ACL** avec Bruno puis Eric : tous deux doivent
être refusés. Être interne lecteur ne donne pas le droit de modifier une règle.
Une règle `blocked` refuse la lecture à tous, y compris à Alice.

**À retenir :** les ACL PDL portent sur les index concrets utilisés par la
recherche. `edge-a`, `demo-alpha` et les dates sont des critères métier ; ils
ne constituent pas des ACL individuelles par équipement, site ou période.
Le [schéma des ACL communes](03-architecture.md#frontières-de-confiance) situe
la vérification du jeton, des groupes LDAP et de la politique côté serveur.

Poursuivre avec le [tutoriel Incidents](17-incidents-tutorial.md), les
[droits et refus](09-demonstrations.md#démo-acl--cinq-minutes) ou la
[technique PDL](04-pdl.md).
