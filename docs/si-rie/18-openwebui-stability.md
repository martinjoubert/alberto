# Stabilité du PoC dans Open WebUI avec Albert API — 11 novembre 2026

La campagne demandée est terminée : **96 messages réellement envoyés par
navigateur**, dont **neuf conversations de dix messages** et six demandes
adressées au mauvais profil. Chaque jeu a été exécuté **trois fois**, sur la
même version métier et sans relance automatique d’un échec.

**La stabilité et la généricité attendues ne sont pas acquises.**
52 messages sur 96 satisfont tous les contrôles
fonctionnels ; 50 satisfont aussi les invariants ciblés de
restitution, soit **46 messages en écart**. **Aucune des neuf conversations
ne satisfait les dix tours sans écart.** Ces scores comprennent les
six essais de mauvais profil, dont l’attente est un refus compréhensible sans
lecture métier hors sujet. Un `partial` explicite sur les données manquantes
peut réussir ; un `ok` portant sur la mauvaise période échoue.

| Jeu, sur les trois passages | Échanges | Fonctionnel | Restitution comprise |
| --- | ---: | ---: | ---: |
| PDL, questions de référence | 30 | 24 | 22 |
| Incidents, questions de référence | 30 | 19 | 19 |
| Fil mixte et reformulations | 30 | 9 | 9 |
| Demandes au mauvais profil | 6 | 0 | 0 |

Les **90 échanges principaux** comptent donc 52 réussites fonctionnelles,
dont 50 avec les invariants de restitution. Les six contre-exemples de profil
échouent à leur critère de guidage sans lecture hors sujet.

La campagne utilise de vraies requêtes Albert sur des données fictives. Elle
ne valide ni le raccordement au SI-RIE réel, ni les autres profils ACL, ni la
capacité sous charge. Les défauts ci-dessous doivent être corrigés puis cette
grille conversationnelle doit être rejouée avant de déclarer ce parcours
stable pour la remise. Les corrections AUD-01 à AUD-03 et leurs preuves
historiques restent distinctes.

## Environnement et version testés

| Élément | Valeur observée |
| --- | --- |
| Serveur métier | VM Lima Ubuntu 24.04 ARM64, 8 vCPU, 16 Go, disque 80 Go |
| Instance | `dinum-rie-restauree-20261109` ; projet `rie-60b04880a219` |
| Image des deux moteurs | `dinum/rie-delivery:runtime-93f90467b650ea8578a6` |
| Client | Open WebUI 0.11.3 local sur le Mac ; deux profils de recette existants |
| Relais, dialogue et planification PDL | Albert API, `gpt-oss-120b` |
| Rédaction Incidents | Albert API, `gemma-4-31b-it` |
| Relecture Incidents | Albert API, `gpt-oss-120b` |
| Sources | OpenSearch, VictoriaMetrics et LDAP réels du laboratoire ; corpus fictifs |
| Tickets | Précédents sur API native ITSM-NG 1.6.17 de laboratoire ; chronologie courante sur snapshot fictif |
| Identité | Alice, compte LDAP interne fictif ; jeton métier individuel |
| Début / fin UTC | 2026-11-11T11:06:12.217916+00:00 / 2026-11-11T12:28:00.936730+00:00 |
| Durée totale | 1.36 h, incluant les attentes de quota |
| Matrice source SHA-256 | `490a1ee99c4680917a54841212af16672f786d7127fe4e9ecaefe554a29ef68e` |

Les paramètres, filtres, outils et connexion officielle Albert sont comparés
par empreintes avant/après. Le contexte de construction métier reste
`93f90467b650ea8578a6` malgré les nouveaux outils de qualification. Aucun prompt,
modèle, mapping, règle d’accès ou corpus n’est corrigé entre deux passages.
Le harnais ne renvoie pas un message échoué ; les corrections de rédaction
bornées prévues par le moteur restent actives et sont comptées dans les usages.
Le portail documentaire peut recevoir le bilan après la fin des échanges ;
ce rafraîchissement ne modifie pas les moteurs testés.

## Comment les essais ont été jugés

Les messages passent par le champ de saisie et le bouton d’envoi Open WebUI,
avec Playwright et Chrome. Les lectures API récupèrent uniquement les messages
synthétiques créés par cette campagne. Elles ne rejouent pas la génération et
ne lisent pas les conversations humaines existantes. Les captures masquent la
barre latérale. Les continuations et sessions sont masquées dans les exports ;
le raisonnement interne éventuellement conservé par le client est exclu.

Chaque fil conserve le même identifiant pendant ses dix tours. Le fil mixte
fait PDL trois fois, Incidents trois fois, PDL deux fois puis Incidents deux
fois, avec changement explicite du profil dans le même fil. L’essai ne prétend
pas qu’un profil unique sait router automatiquement les deux UC. Les six
contre-exemples vérifient justement la réaction au mauvais choix de profil.

Les références sont celles des tutoriels : quatre familles PDL, six documents
croisés pour edge-a dans demo-alpha, puis changements de source, d’IP, de
périmètre et fenêtre vide. Incidents vérifie R1/R2, Rennes, R3 sans mesures,
R4 exclu, les précédents ITSM et les statuts à midi. Les contrôles comparent
les périodes réellement exécutées, les équipements et documents retournés,
les filtres, la complétude, les sources citées et les modèles effectivement
retournés. Ils ne se contentent pas de comparer deux textes d’IA.

Les demandes de diagnostic normal attendent une rédaction **et** une relecture
réelles validées. Les demandes de droits, les clarifications et les suivis de
statut ont leur propre contrat : l’absence de seconde synthèse métier n’y est
pas assimilée à une panne, ni présentée comme une nouvelle génération backend.
Les usages chiffrés dans le reçu sont conservés ; aucune réponse fictive de
fournisseur n’a remplacé Albert.

Trois ajustements du harnais sont séparés du PoC : accepter le statut natif
`clarification` d’Incidents ; traiter un plan PDL nul comme une clarification
inattendue plutôt qu’une exception Python ; reconnaître aussi une limitation
formulée « ne permettent pas ». La règle interdisant une lecture hors sujet
reste appliquée. Les reçus du même message sont appariés par identifiants,
les assertions originales sont jointes et aucun échange n’est relancé pour
corriger ces assertions. Les trois erreurs du harnais initial sur un plan nul
ont ainsi été réévaluées depuis les reçus existants ; elles restent des échecs
métier de clarification inattendue.

Une lecture ciblée par Codex rapproche également les hypothèses R2 de leurs
sources, contrôle le refus R4 et les divergences de période et de périmètre.
Ce contrôle complémentaire ne constitue pas une validation métier humaine
exhaustive.

## État technique et portée des mesures

Au point de contrôle final, les huit services Ubuntu sont démarrés, les six
services principaux sont sains, aucun OOM n’est signalé et les compteurs de
redémarrage valent zéro, comme au début. L’origine configurée des moteurs
est `https://albert.api.etalab.gouv.fr/v1`. Les neuf comparaisons de paramètres,
filtres et connexions Open WebUI réussissent. Ces constats ne remplacent pas
une campagne de panne ou de charge.

Les durées mesurées sont de **21,9 s au p50**, **65,2 s au p95** et **73,5 s au
p99**, hors attente imposée entre les demandes Incidents. Les reçus documentent
69 appels métier : 42 rédactions et 27 relectures, pour 309 750 tokens connus.
Quatorze réponses ont nécessité une reprise bornée. Les appels auxiliaires du
client, de dialogue et de planification non chiffrés dans ces reçus ne sont pas
compris dans ce total.

## Défauts et limites mis en évidence

### OW-01 — Une période horaire peut devenir une journée entière

**Priorité : bloquant pour la fidélité des recherches, dans les deux UC.**
La demande « le 12 octobre 2026 entre 08:00 et 09:00 UTC » devient parfois
`00:00 → 00:00` le lendemain. En Incidents, « entre 10 h et 12 h, heure de
Paris » peut devenir `22:00Z la veille → 22:00Z`, soit la journée civile entière.
15 lectures effectivement exécutées ont été
signalées pour mauvaise période ; une clarification sans lecture n’entre pas
dans ce compteur.
Les cinq parcours concernés reproduisent cette divergence lors des trois
passages.

Le compte de documents peut rester juste par coïncidence. Pour R3, la fenêtre
élargie récupère même des points en dehors de la plage demandée : une absence
de mesures sur 10–12 h peut alors être remplacée par une moyenne d’un autre
créneau. Une relecture IA valide peut approuver un texte cohérent avec ces
preuves mal cadrées ; ce verdict ne valide pas la fidélité à la question.

La reproduction locale confirme la cause : `relative_window` et
`french_window` sont dupliquées à l’identique dans
`rie_agent/natural_dates.py` et `obs_poc/assistant.py`. Les heures sont reconnues
après « de … à … », mais certaines queues « entre … et … » retombent sur une
journée. Le défaut se produit aussi au premier message d’un fil neuf.

**Clôture attendue :** résoudre correctement les formulations horaires usuelles
ou demander une précision avant toute lecture ; interdire un élargissement
silencieux et rejouer des fenêtres contenant des données différentes à
l’extérieur. Corriger la logique dans les deux UC, avec une source commune si
sa séparation de dépendances est maintenue.

### OW-02 — Clarifications superflues et retours de contexte incomplets

**Priorité : parcours conversationnel à fiabiliser.**
8 réponses de clarification
inattendues apparaissent sur des demandes censées être complètes ou déjà
contextualisées. Le retour implicite à un ancien sujet PDL après plusieurs
messages Incidents n’est pas garanti. Une demande de précédents ITSM peut
également déclencher une question sur une période de comparaison.

Le refus prudent de deviner protège les données, mais ne satisfait pas le
parcours demandé. Une demande explicite peut rétablir le bon résultat ; le
tableau des répétitions distingue ces récupérations du succès du retour
implicite. Il faut préserver l’état de chaque UC et séparer recherche de
précédents, comparaison temporelle et changement de sujet.

**Clôture attendue :** répéter les dix tours, conserver le bon équipement,
la bonne fenêtre et les sources après un changement de sujet, puis reprendre
sans réinterpréter un contexte refusé ou remplacer une donnée absente.

### OW-03 — La comparaison de deux équipements peut dépasser le budget de relecture

**Priorité : comparaison Rennes à terminer.**
Le diagnostic comparé dispose des mesures et journaux, mais le contexte fourni
au relecteur peut dépasser la limite locale de 24 000 caractères. Le reçu porte
`connector / review_context_budget_exceeded`, observé
**3 fois**. Il ne s’agit pas d’une
preuve d’indisponibilité Albert : le relecteur n’est pas appelé à cette étape.

**Clôture attendue :** réduire et borner les éléments transmis en conservant
les preuves nécessaires, ou annoncer une portée réellement supportée ; vérifier
la comparaison à deux équipements sans enlever la relecture ni augmenter les
limites silencieusement.

### OW-04 — Certains refus ou appels répétés aboutissent à un message générique

**Priorité : intégration Open WebUI/MCP.**
Un essai PDL sur fenêtre vide a déclenché deux appels d’outil et le client a
signalé `Tool-call limit reached (1 iterations)`. Le filtre protège alors contre
la publication d’une réponse sans traitement vérifiable.

Pour R4 exclu, le serveur renvoie « Équipement ou périmètre inconnu ou non
autorisé », sans preuve métier. Le client marque l’appel comme échoué et affiche
un message générique, en perdant cette explication utile. Cette observation ne
constitue pas une fuite ACL : le refus côté serveur et le défaut de restitution
sont deux constats séparés.

**Clôture attendue :** un seul appel métier pour une demande simple ; rendre les
refus explicites et les fenêtres vides sans inventer de résultat ni dépasser
le contrat d’outillage.

### OW-05 — Mauvais profil : le périmètre par défaut peut être consulté hors sujet

**Priorité : cadrage des demandes mixtes.**
Le profil PDL peut demander un `network_scope` pour une question Incidents au
lieu d’indiquer le changement de profil nécessaire. Inversement, le profil
Incidents peut collecter les mesures de Rennes pour une demande de comptage
PFS, puis annoncer qu’il ne peut pas y répondre. Le compte demandé n’est pas
inventé, mais la lecture hors sujet reste inutile et trompeuse.

Les sources sont autorisées pour Alice ; cette campagne n’établit pas un
contournement des droits. L’attente est un arrêt avant la collecte et un guidage
vers le bon profil. Le développement d’un routeur automatique constitue une
décision distincte, pas une capacité démontrée par ces essais.

### OW-06 — Une explication PDL invente une limite de comptage

**Priorité : fidélité de la restitution.**
Un texte visible prétend que le comptage est limité à 100 000 documents par
`max_rows`. Le reçu indique `max_rows=100` et ce paramètre borne les lignes de
recherche affichées, pas le comptage exact effectué sans liste de documents.
Les contrôles de restitution identifient
deux occurrences de cette famille.
Un nombre final correct ne suffit donc pas à valider toutes les explications.

**Clôture attendue :** ancrer les explications de limites sur le contrat et le
reçu effectifs ; vérifier aussi les demandes à résultat nul et les corpus plus
volumineux.

### OW-07 — La génération reste variable, et les garde-fous refusent certaines rédactions

**Priorité : disponibilité d’une synthèse qualifiée.**
Le code retourne `partial` lorsque les contraintes ou la relecture ne sont
pas validées. Six diagnostics dont le scénario exige une synthèse qualifiée
n’ont pas obtenu de validation complète : trois refus de contrainte et trois
dépassements du budget de relecture. Les clarifications préalables sont comptées
séparément.

Une septième synthèse est refusée lors du constat de mesures absentes sur R3 :
le relecteur rejette un conseil d’analyse des journaux du collecteur non présent
dans les documents cités. Les faits attendus — absence de points et moyenne
inconnue — restent corrects, donc le contrôle factuel réussit avec un résultat
`partial`. Cela ne qualifie pas la synthèse IA rejetée ; son erreur
`factual_review_failed` reste visible dans le cas et les pièces jointes.

Un brouillon rejeté consulté pour cette campagne affirme que la congestion
« entraînant des pertes et une augmentation de la latence ». Les preuves
permettent une corrélation, pas cette causalité certaine ; le contrôle a une
raison valable de refuser ce texte. Les précédents et observations restent
lisibles, avec une confiance abaissée. Il faut améliorer la rédaction et les
reprises bornées, puis requalifier, plutôt que rendre la relecture facultative.

Le mélange Gemma pour la rédaction et GPT-OSS pour le dialogue/relecteur est
bien celui testé. Les seuls résultats présents ne suffisent pas à classer
universellement les modèles ; le parseur de dates et les budgets locaux
expliquent aussi des échecs indépendants du fournisseur.

### OW-08 — Le périmètre précédent peut remplacer le périmètre demandé

**Priorité : bloquant pour la fidélité PDL et la restitution.**
Après une demande sur `demo-beta`, une nouvelle question complète demande un
comptage dans `demo-alpha` sur une autre période. Un essai exécute la bonne
période mais conserve le filtre `network_scope = demo-beta`. Le résultat vaut
zéro, comme le résultat attendu dans l’autre périmètre. Le texte final affirme
pourtant que `demo-alpha` a été utilisé, avec une correction du périmètre qui
n’existe pas dans le plan exécuté.

Le défaut porte sur la priorité du dernier message et sur la fidélité du texte
au reçu natif. Les deux périmètres sont autorisés pour Alice : il ne démontre
pas un contournement des ACL. Le résultat nul ne permet pas de l’écarter.

**Clôture attendue :** valider les filtres exigés par la dernière demande avant
lecture, puis empêcher le dernier relais de présenter un filtre différent de
celui exécuté. Ajouter des données contrastées hors du périmètre demandé dans
les régressions, sans remplacer les reçus déjà en échec.

## Lire et remettre les preuves

Dans NiceGUI, ouvrir **Qualification**, puis la campagne
`openwebui-baseline-redacted-20261111`. Les cas rouges restent rouges ; chaque message
présente l’attendu/obtenu, le texte affiché, les modèles et usages connus, les
codes d’erreur et les assertions originales. Les captures de cette première
campagne restent archivées en privé ; les captures susceptibles de montrer un
identifiant de reprise ont été retirées du paquet de remise.
Voir le [fonctionnement des rapports](13-test-reports.md) pour l’accès LDAP,
les téléchargements et le mécanisme d’intégrité.

Le dossier de remise contient le présent bilan, les résultats détaillés
Markdown/JSON/CSV, la matrice figée, les reçus métier sélectionnés, les captures,
les états des services et les empreintes de configuration. Un manifeste SHA-256
contrôle les fichiers ; il n’atteste pas à lui seul l’authenticité de leur origine.
Les sessions, clés, identifiants de reprise et raisonnements internes ne
font pas partie de l’export.

**Correction de l’export, le 11 novembre :** le masquage initial reconnaissait
les reprises Incidents, mais pas la forme plus longue des reprises PDL dans
les arguments JSON sérialisés. Certains de ces identifiants figuraient dans
les premières pièces jointes. Ils étaient liés à l’utilisateur et déjà expirés
lors du constat ; il ne s’agissait pas de clés API ou de mots de passe.
Le masquage et ses régressions couvrent désormais les deux formats. Les deux
anciens rendus ont été retirés du catalogue actif et conservés dans une archive
privée. Le nouveau rendu masqué conserve **50 réussites et 46 échecs** ; les
preuves textuelles et leurs manifestes ont été régénérés, sans rejouer ni
requalifier les réponses historiques. Ne pas additionner ces présentations
d’une seule exécution.

Le [suivi des corrections et de leur revalidation](19-openwebui-corrections.md)
reste distinct de ce constat initial.

Les suites déterministes exécutées comptent 1 174 réussites et un cas Linux
ignoré sur Mac, dont les treize régressions de l’oracle. Elles se distinguent
des conversations réelles et ne clôturent pas leurs échecs.

Les 13 tests unitaires de l’oracle et les contrôles de lecture du rapport sont
consignés séparément des 96 échanges réels. Ni leur somme ni une ancienne
endurance ne sont présentées comme de nouvelles conversations réussies.

## Résultats détaillés des 96 échanges

Les tableaux et fiches ci-dessous sont calculés à partir des reçus de la
campagne complète. « Fonctionnel » couvre le contrat et les données attendues ;
« restitution comprise » ajoute les invariants ciblés du texte et des limites.
Les catégories peuvent se recouvrir et plusieurs tours en écart peuvent avoir
une même cause initiale.


Campagne du 11 novembre 2026 sur le SI-RIE simulé Ubuntu ARM64. Données fictives, fournisseur Albert réel.

État : **completed**. Messages envoyés : **96/96**.
Contrôles fonctionnels satisfaits : **52/96** ; avec les invariants de restitution : **50/96**.

## Méthode et périmètre

Trois jeux de dix messages, chacun répété trois fois à l’identique : PDL, Incidents, puis fil mixte avec reformulations. Deux demandes au mauvais preset complètent chaque répétition. Les messages passent par le navigateur Open WebUI ; les réponses natives MCP et les textes affichés sont conservés. Le harnais ne renvoie aucun message en échec ; les reprises de rédaction bornées du moteur restent actives et tracées.

Les changements de modèle PDL/Incidents se font explicitement dans le même fil. Le test du retour implicite à un ancien sujet reste une attente métier exigeante : une clarification prudente protège les données mais ne satisfait pas cette attente de continuité.

Le code, les modèles et les paramètres métier restent figés pendant les répétitions. Les assertions sont séparées des réponses du système. Le statut de clarification Incidents est `clarification`, celui de PDL `needs_clarification` ; les assertions initiales et leur normalisation sont traçables.

Une erreur de l’oracle initial sur un plan PDL nul est réévaluée à partir du reçu du même message, sans nouvel appel au modèle. Le reçu est apparié par identifiants de conversation et de message. L’erreur initiale reste conservée.

## Résultats par jeu

| Jeu | Messages | Fonctionnel | Restitution comprise |
| --- | ---: | ---: | ---: |
| pdl_reference | 30 | 24 | 22 |
| incidents_reference | 30 | 19 | 19 |
| mixed_variants | 30 | 9 | 9 |
| wrong_incident_request_in_pdl | 3 | 0 | 0 |
| wrong_pdl_request_in_incidents | 3 | 0 | 0 |

Les trois premiers jeux correspondent aux 90 échanges principaux. Les deux lignes wrong correspondent aux six contrôles du mauvais choix de profil.

## Résultats par conversation

| Répétition | Jeu | Messages | Fonctionnel | Restitution comprise | Changements d’UC dans le fil |
| ---: | --- | ---: | ---: | ---: | ---: |
| 1 | pdl_reference | 10 | 8 | 7 | 0 |
| 1 | incidents_reference | 10 | 6 | 6 | 0 |
| 1 | mixed_variants | 10 | 3 | 3 | 3 |
| 1 | wrong_incident_request_in_pdl | 1 | 0 | 0 | 0 |
| 1 | wrong_pdl_request_in_incidents | 1 | 0 | 0 | 0 |
| 2 | pdl_reference | 10 | 8 | 8 | 0 |
| 2 | incidents_reference | 10 | 7 | 7 | 0 |
| 2 | mixed_variants | 10 | 3 | 3 | 3 |
| 2 | wrong_incident_request_in_pdl | 1 | 0 | 0 | 0 |
| 2 | wrong_pdl_request_in_incidents | 1 | 0 | 0 | 0 |
| 3 | pdl_reference | 10 | 8 | 7 | 0 |
| 3 | incidents_reference | 10 | 6 | 6 | 0 |
| 3 | mixed_variants | 10 | 3 | 3 | 3 |
| 3 | wrong_incident_request_in_pdl | 1 | 0 | 0 | 0 |
| 3 | wrong_pdl_request_in_incidents | 1 | 0 | 0 | 0 |

## Résultats par question

| Jeu / question | Essais | Fonctionnel | Restitution comprise | Statuts observés |
| --- | ---: | ---: | ---: | --- |
| pdl_reference / rights | 3 | 3 | 3 | success, success, success |
| pdl_reference / missing_period | 3 | 3 | 3 | needs_clarification, needs_clarification, needs_clarification |
| pdl_reference / period_reply | 3 | 3 | 1 | success, success, success |
| pdl_reference / equipment | 3 | 3 | 3 | success, success, success |
| pdl_reference / tacacs_followup | 3 | 3 | 3 | success, success, success |
| pdl_reference / exclude_tacacs | 3 | 3 | 3 | success, success, success |
| pdl_reference / client_ip_new_subject | 3 | 3 | 3 | success, success, success |
| pdl_reference / scope_change | 3 | 3 | 3 | success, success, success |
| pdl_reference / empty_period | 3 | 0 | 0 | None, success, success |
| pdl_reference / return_equipment | 3 | 0 | 0 | success, success, success |
| incidents_reference / missing_period | 3 | 3 | 3 | clarification, clarification, clarification |
| incidents_reference / period_reply | 3 | 0 | 0 | partial, partial, partial |
| incidents_reference / ticket_at_noon | 3 | 3 | 3 | ok, ok, ok |
| incidents_reference / r2_followup | 3 | 3 | 3 | ok, ok, ok |
| incidents_reference / compare_site | 3 | 0 | 0 | partial, partial, partial |
| incidents_reference / r3_missing_latency | 3 | 3 | 3 | partial, partial, partial |
| incidents_reference / r3_cpu_followup | 3 | 3 | 3 | partial, partial, partial |
| incidents_reference / excluded_equipment | 3 | 0 | 0 | None, None, None |
| incidents_reference / return_r1_precedents | 3 | 1 | 1 | clarification, ok, clarification |
| incidents_reference / r1_status_after_changes | 3 | 3 | 3 | ok, ok, ok |
| mixed_variants / pdl_reword_count | 3 | 0 | 0 | success, success, success |
| mixed_variants / pdl_reword_equipment | 3 | 3 | 3 | success, success, success |
| mixed_variants / pdl_reword_tacacs | 3 | 3 | 3 | success, success, success |
| mixed_variants / switch_to_incidents | 3 | 0 | 0 | partial, partial, partial |
| mixed_variants / incidents_reword_r2 | 3 | 0 | 0 | partial, partial, partial |
| mixed_variants / incidents_r2_noon | 3 | 0 | 0 | clarification, clarification, clarification |
| mixed_variants / implicit_return_to_pdl | 3 | 0 | 0 | needs_clarification, needs_clarification, needs_clarification |
| mixed_variants / explicit_recovery_pdl | 3 | 3 | 3 | success, success, success |
| mixed_variants / switch_r3_missing | 3 | 0 | 0 | partial, partial, partial |
| mixed_variants / missing_data_followup | 3 | 0 | 0 | None, None, None |
| wrong_incident_request_in_pdl / incident_request_in_pdl | 3 | 0 | 0 | needs_clarification, needs_clarification, needs_clarification |
| wrong_pdl_request_in_incidents / pdl_request_in_incidents | 3 | 0 | 0 | partial, partial, partial |

## Écarts observés

| Famille d’écart | Occurrences |
| --- | ---: |
| Client, transport ou protocole d’outil (`client_or_transport`) | 7 |
| Retour implicite à une autre UC non satisfait (`cross_uc_context_return`) | 3 |
| Génération exigée par le scénario non validée (`generation_not_validated`) | 6 |
| Clarification superflue (`unexpected_clarification`) | 8 |
| Explication visible non étayée (`visible_answer_fidelity`) | 2 |
| Mauvais profil : absence de guidage ou lecture hors sujet (`wrong_domain_handling`) | 6 |
| Période exécutée différente de la demande (`wrong_period`) | 15 |
| Source, périmètre ou filtre différent de la demande (`wrong_scope_or_filter`) | 2 |

Les catégories peuvent se recouvrir. Un échec au début d’un fil peut expliquer plusieurs suites de questions ; ces occurrences ne représentent pas autant de bugs indépendants.

## Durées et consommation observées

Durées de bout en bout par message : p50 21.886 s, p95 65.206 s, p99 73.542 s. L’attente imposée entre deux demandes Incidents est exclue de ces durées, mais reste comprise dans la durée totale de campagne.
Générations métier documentées : 69 appels de rédaction/relecture, 309750 tokens totaux connus. Les appels de relais, dialogue et planification non chiffrés dans les reçus ne sont pas inclus.
Appels par étape : {'draft': 42, 'review': 27}. Réponses ayant nécessité une reprise bornée de génération : 14.
Synthèses non validées au total : 7. Ce compteur inclut les demandes factuelles où les observations peuvent être correctes malgré une synthèse refusée ; la catégorie des générations exigées par le scénario les distingue.

| UC | p50 (s) | p95 (s) | p99 (s) |
| --- | ---: | ---: | ---: |
| pdl | 36.718 | 70.186 | 73.542 |
| incidents | 15.652 | 27.083 | 31.024 |

Ces durées mélangent des types de demandes différents et leurs échecs. Elles ne constituent ni un débit sous charge ni un engagement de temps de réponse du SI-RIE.

## Détail des échanges

### Répétition 1 — pdl_reference — message 1

**Question :** Quels sont mes index accessibles ?

**Résultat :** success ; fonctionnel réussi ; durée 8.285 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "rights"}`
**Conversation de preuve :** `cc79c6e5-1ebc-4243-a991-a1c6191e2bf6`

### Répétition 1 — pdl_reference — message 2

**Question :** Compte les refus PFS dans demo-alpha.

**Résultat :** needs_clarification ; fonctionnel réussi ; durée 11.246 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "clarification"}`
**Conversation de preuve :** `cc79c6e5-1ebc-4243-a991-a1c6191e2bf6`

**Clarification retournée :** Sur quelle période souhaitez-vous consulter le PDL ? Indiquez une date complète ou une période relative explicite.

### Répétition 1 — pdl_reference — message 3

**Question :** Le 12 octobre 2026 de 08:00 à 09:00 UTC.

**Résultat :** success ; fonctionnel réussi ; durée 28.414 s.
**Contrôles non satisfaits :** no_invented_count_cap_in_visible_answer
**Attendu :** `{"kind": "pdl_count", "total": 2, "scope": "demo-alpha"}`
**Conversation de preuve :** `cc79c6e5-1ebc-4243-a991-a1c6191e2bf6`

### Répétition 1 — pdl_reference — message 4

**Question :** Montre les logs de l'équipement edge-a PFS BB PFAING TACACS dans demo-alpha du 12 octobre 2026 de 08:00 à 09:00 UTC.

**Résultat :** success ; fonctionnel réussi ; durée 57.812 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_equipment", "families": "all"}`
**Conversation de preuve :** `cc79c6e5-1ebc-4243-a991-a1c6191e2bf6`

### Répétition 1 — pdl_reference — message 5

**Question :** Uniquement TACACS pour ce même équipement, sur la même période.

**Résultat :** success ; fonctionnel réussi ; durée 33.594 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_equipment", "families": "tacacs"}`
**Conversation de preuve :** `cc79c6e5-1ebc-4243-a991-a1c6191e2bf6`

### Répétition 1 — pdl_reference — message 6

**Question :** Pour edge-a dans demo-alpha, sur la même période, montre PFS, BB et PFAING, sans TACACS.

**Résultat :** success ; fonctionnel réussi ; durée 70.186 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_equipment", "families": "without_tacacs"}`
**Conversation de preuve :** `cc79c6e5-1ebc-4243-a991-a1c6191e2bf6`

### Répétition 1 — pdl_reference — message 7

**Question :** Recherche les accès PFAING de l'adresse IP source 192.0.2.25 dans demo-alpha le 12 octobre 2026 de 08:00 à 09:00 UTC.

**Résultat :** success ; fonctionnel réussi ; durée 36.718 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_pfaing", "total": 2}`
**Conversation de preuve :** `cc79c6e5-1ebc-4243-a991-a1c6191e2bf6`

### Répétition 1 — pdl_reference — message 8

**Question :** Compte les refus PFS dans demo-beta le 12 octobre 2026 de 08:00 à 09:00 UTC.

**Résultat :** success ; fonctionnel réussi ; durée 25.575 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_count", "total": 1, "scope": "demo-beta"}`
**Conversation de preuve :** `cc79c6e5-1ebc-4243-a991-a1c6191e2bf6`

### Répétition 1 — pdl_reference — message 9

**Question :** Compte les refus PFS dans demo-alpha le 12 octobre 2026 de 07:00 à 07:30 UTC.

**Résultat :** None ; fonctionnel échoué ; durée 37.735 s.
**Contrôles non satisfaits :** client_finished, exactly_one_completed_native_tool, native_result_present
**Attendu :** `{"kind": "pdl_count", "total": 0, "scope": "demo-alpha", "start": "2026-10-12T07:00:00Z", "end": "2026-10-12T07:30:00Z"}`
**Conversation de preuve :** `cc79c6e5-1ebc-4243-a991-a1c6191e2bf6`

**Erreur client :** `{"content": "Tool-call limit reached (1 iterations)."}`

### Répétition 1 — pdl_reference — message 10

**Question :** Reprenons edge-a : retrouve son activité dans les quatre familles PFS, BB, PFAING et TACACS, périmètre demo-alpha, le 12 octobre 2026 entre 08:00 et 09:00 UTC.

**Résultat :** success ; fonctionnel échoué ; durée 48.95 s.
**Contrôles non satisfaits :** correct_pdl_period
**Attendu :** `{"kind": "pdl_equipment", "families": "all"}`
**Conversation de preuve :** `cc79c6e5-1ebc-4243-a991-a1c6191e2bf6`

### Répétition 1 — incidents_reference — message 1

**Question :** Diagnostique R1 sur le site de Rennes.

**Résultat :** clarification ; fonctionnel réussi ; durée 6.265 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "clarification"}`
**Conversation de preuve :** `f31cdad4-6094-40c9-8f96-17adef97bcbc`

### Répétition 1 — incidents_reference — message 2

**Question :** Le 28 septembre 2026 de 10:00 à 12:00, heure de Paris.

**Résultat :** partial ; fonctionnel échoué ; durée 18.442 s.
**Contrôles non satisfaits :** successful_incident_result, real_generated_and_reviewed_diagnostic, actual_draft_and_review_models_with_tokens
**Attendu :** `{"kind": "incident_diagnostic", "equipment": ["OBS-RTR-01"], "generation": true}`
**Conversation de preuve :** `f31cdad4-6094-40c9-8f96-17adef97bcbc`

### Répétition 1 — incidents_reference — message 3

**Question :** À 12 h, heure de Paris, l'alerte était-elle résolue et le ticket déjà clôturé ?

**Résultat :** ok ; fonctionnel réussi ; durée 6.262 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "incident_resolution", "equipment": ["OBS-RTR-01"]}`
**Conversation de preuve :** `f31cdad4-6094-40c9-8f96-17adef97bcbc`

### Répétition 1 — incidents_reference — message 4

**Question :** Fais la même analyse pour R2, sur la même période et sur le site de Rennes.

**Résultat :** ok ; fonctionnel réussi ; durée 23.648 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "incident_diagnostic", "equipment": ["OBS-RTR-02"], "generation": true}`
**Conversation de preuve :** `f31cdad4-6094-40c9-8f96-17adef97bcbc`

### Répétition 1 — incidents_reference — message 5

**Question :** Compare les équipements du site de Rennes le 28 septembre 2026 de 10:00 à 12:00, heure de Paris. Distingue les mesures, les alertes et les logs de chaque équipement.

**Résultat :** partial ; fonctionnel échoué ; durée 19.613 s.
**Contrôles non satisfaits :** successful_incident_result, real_generated_and_reviewed_diagnostic, actual_draft_and_review_models_with_tokens
**Attendu :** `{"kind": "incident_diagnostic", "equipment": ["OBS-RTR-01", "OBS-RTR-02"], "generation": true}`
**Conversation de preuve :** `f31cdad4-6094-40c9-8f96-17adef97bcbc`

### Répétition 1 — incidents_reference — message 6

**Question :** Pour R3 le 28 septembre 2026 de 10:00 à 12:00, heure de Paris, combien de points de latence sont disponibles et quelle est la latence moyenne ?

**Résultat :** partial ; fonctionnel réussi ; durée 24.72 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "incident_missing", "equipment": ["OBS-RTR-03"], "metric": "latency"}`
**Conversation de preuve :** `f31cdad4-6094-40c9-8f96-17adef97bcbc`

### Répétition 1 — incidents_reference — message 7

**Question :** Et pour son CPU, sur la même période : combien de points et quelle moyenne ?

**Résultat :** partial ; fonctionnel réussi ; durée 18.575 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "incident_missing", "equipment": ["OBS-RTR-03"], "metric": "cpu"}`
**Conversation de preuve :** `f31cdad4-6094-40c9-8f96-17adef97bcbc`

### Répétition 1 — incidents_reference — message 8

**Question :** Diagnostique R4 le 28 septembre 2026 de 10:00 à 12:00, heure de Paris.

**Résultat :** None ; fonctionnel échoué ; durée 10.452 s.
**Contrôles non satisfaits :** exactly_one_completed_native_tool, native_result_present
**Attendu :** `{"kind": "blocked_equipment"}`
**Conversation de preuve :** `f31cdad4-6094-40c9-8f96-17adef97bcbc`

### Répétition 1 — incidents_reference — message 9

**Question :** Diagnostique R1 le 28 septembre 2026 de 10:00 à 12:00, heure de Paris, sur le site de Rennes. Recherche les précédents ITSM et propose la solution la mieux étayée, à valider par un opérateur, avec son niveau de confiance.

**Résultat :** clarification ; fonctionnel échoué ; durée 15.652 s.
**Contrôles non satisfaits :** correct_incident_equipment, correct_incident_period, successful_incident_result, five_metrics_per_equipment, exact_log_count, real_generated_and_reviewed_diagnostic, actual_draft_and_review_models_with_tokens, native_itsm_historical_precedents, conditional_solution_and_separate_confidence, displayed_citations_known
**Attendu :** `{"kind": "incident_diagnostic", "equipment": ["OBS-RTR-01"], "generation": true, "precedents": true}`
**Conversation de preuve :** `f31cdad4-6094-40c9-8f96-17adef97bcbc`

### Répétition 1 — incidents_reference — message 10

**Question :** À midi, heure de Paris, l'alerte était-elle déjà résolue et le ticket déjà clôturé ?

**Résultat :** ok ; fonctionnel réussi ; durée 12.53 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "incident_resolution", "equipment": ["OBS-RTR-01"]}`
**Conversation de preuve :** `f31cdad4-6094-40c9-8f96-17adef97bcbc`

### Répétition 1 — mixed_variants — message 1

**Question :** Combien de décisions de refus ont été enregistrées par PFS dans demo-alpha, le 12 octobre 2026 entre 8 heures et 9 heures UTC ?

**Résultat :** success ; fonctionnel échoué ; durée 46.793 s.
**Contrôles non satisfaits :** correct_pdl_period
**Attendu :** `{"kind": "pdl_count", "total": 2, "scope": "demo-alpha"}`
**Conversation de preuve :** `8b8d65ea-da50-475b-aea5-19f463032e24`

### Répétition 1 — mixed_variants — message 2

**Question :** Retrouve l'activité de edge-a dans PFS, Backbone, PFAING et TACACS, pour demo-alpha, le 12 octobre 2026 de 08:00 à 09:00 UTC. Appuie-toi sur les relations entre les sources.

**Résultat :** success ; fonctionnel réussi ; durée 60.937 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_equipment", "families": "all"}`
**Conversation de preuve :** `8b8d65ea-da50-475b-aea5-19f463032e24`

### Répétition 1 — mixed_variants — message 3

**Question :** Garde ce créneau et cet équipement, mais limite le résultat aux événements TACACS.

**Résultat :** success ; fonctionnel réussi ; durée 37.74 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_equipment", "families": "tacacs"}`
**Conversation de preuve :** `8b8d65ea-da50-475b-aea5-19f463032e24`

### Répétition 1 — mixed_variants — message 4

**Question :** Changeons de sujet : analyse les mesures, les journaux et les tickets de R1 à Rennes le 28 septembre 2026 entre 10 h et 12 h, heure de Paris. Distingue les observations et les causes possibles.

**Résultat :** partial ; fonctionnel échoué ; durée 23.892 s.
**Contrôles non satisfaits :** correct_incident_period, successful_incident_result, exact_log_count
**Attendu :** `{"kind": "incident_diagnostic", "equipment": ["OBS-RTR-01"], "generation": true}`
**Conversation de preuve :** `8b8d65ea-da50-475b-aea5-19f463032e24`

### Répétition 1 — mixed_variants — message 5

**Question :** Sur ce même créneau, fais maintenant le diagnostic de R2 à Rennes.

**Résultat :** partial ; fonctionnel échoué ; durée 18.542 s.
**Contrôles non satisfaits :** correct_incident_period, successful_incident_result, exact_log_count
**Attendu :** `{"kind": "incident_diagnostic", "equipment": ["OBS-RTR-02"], "generation": true}`
**Conversation de preuve :** `8b8d65ea-da50-475b-aea5-19f463032e24`

### Répétition 1 — mixed_variants — message 6

**Question :** À 12 h, heure de Paris, l'alerte était-elle résolue et le ticket déjà clôturé ?

**Résultat :** clarification ; fonctionnel échoué ; durée 9.438 s.
**Contrôles non satisfaits :** correct_incident_equipment, correct_incident_period, successful_incident_result, five_metrics_per_equipment, exact_log_count, technical_resolution_and_ticket_close_remain_distinct, noon_answer_explicit, displayed_citations_known
**Attendu :** `{"kind": "incident_resolution", "equipment": ["OBS-RTR-02"]}`
**Conversation de preuve :** `8b8d65ea-da50-475b-aea5-19f463032e24`

### Répétition 1 — mixed_variants — message 7

**Question :** Reviens aux logs de l'équipement du début de cette conversation, avec le même périmètre et la même période, dans les quatre familles.

**Résultat :** needs_clarification ; fonctionnel échoué ; durée 14.798 s.
**Contrôles non satisfaits :** business_success, correct_pdl_period, exact_expected_documents, exact_total, equipment_context_preserved, no_scope_contamination, real_planner_model_returned
**Attendu :** `{"kind": "pdl_equipment", "families": "all", "cross_uc_memory": true}`
**Conversation de preuve :** `8b8d65ea-da50-475b-aea5-19f463032e24`

**Clarification retournée :** Sur quelle période souhaitez-vous consulter le PDL ? Indiquez une date complète ou une période relative explicite.

### Répétition 1 — mixed_variants — message 8

**Question :** Montre les logs de l'équipement edge-a PFS BB PFAING TACACS dans demo-alpha du 12 octobre 2026 de 08:00 à 09:00 UTC.

**Résultat :** success ; fonctionnel réussi ; durée 59.546 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_equipment", "families": "all"}`
**Conversation de preuve :** `8b8d65ea-da50-475b-aea5-19f463032e24`

### Répétition 1 — mixed_variants — message 9

**Question :** Autre incident : pour R3, le 28 septembre 2026 entre 10 h et midi en heure de Paris, quelle est la latence moyenne et combien de points ont été collectés ?

**Résultat :** partial ; fonctionnel échoué ; durée 21.052 s.
**Contrôles non satisfaits :** correct_incident_period, missing_points_explicit, missing_mean_is_unknown_not_zero
**Attendu :** `{"kind": "incident_missing", "equipment": ["OBS-RTR-03"], "metric": "latency"}`
**Conversation de preuve :** `8b8d65ea-da50-475b-aea5-19f463032e24`

### Répétition 1 — mixed_variants — message 10

**Question :** Pour ce même équipement et cette même période, s'agit-il d'une latence nulle ou d'une absence de collecte ? Explique la limite avant de conclure.

**Résultat :** None ; fonctionnel échoué ; durée 12.535 s.
**Contrôles non satisfaits :** exactly_one_completed_native_tool, native_result_present
**Attendu :** `{"kind": "incident_missing", "equipment": ["OBS-RTR-03"], "metric": "latency"}`
**Conversation de preuve :** `8b8d65ea-da50-475b-aea5-19f463032e24`

### Répétition 1 — wrong_incident_request_in_pdl — message 1

**Question :** Diagnostique R1 à Rennes le 28 septembre 2026 de 10:00 à 12:00, heure de Paris, avec ses métriques, alertes et tickets.

**Résultat :** needs_clarification ; fonctionnel échoué ; durée 43.834 s.
**Contrôles non satisfaits :** wrong_domain_explicitly_explained
**Attendu :** `{"kind": "wrong_uc"}`
**Conversation de preuve :** `ef41d285-0f21-4f03-989d-7c0c19de321a`

**Clarification retournée :** Quel est le network_scope (demo-alpha ou demo-beta) correspondant à Rennes pour l'équipement R1 ?

### Répétition 1 — wrong_pdl_request_in_incidents — message 1

**Question :** Compte les refus PFS dans demo-alpha le 12 octobre 2026 de 08:00 à 09:00 UTC.

**Résultat :** partial ; fonctionnel échoué ; durée 12.454 s.
**Contrôles non satisfaits :** wrong_domain_request_has_no_business_evidence
**Attendu :** `{"kind": "wrong_uc"}`
**Conversation de preuve :** `e676af41-92d5-433f-b01d-d8f171d5aae3`

### Répétition 2 — pdl_reference — message 1

**Question :** Quels sont mes index accessibles ?

**Résultat :** success ; fonctionnel réussi ; durée 6.365 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "rights"}`
**Conversation de preuve :** `4055b7ce-973d-477f-aff2-ded4eb2e7b74`

### Répétition 2 — pdl_reference — message 2

**Question :** Compte les refus PFS dans demo-alpha.

**Résultat :** needs_clarification ; fonctionnel réussi ; durée 12.385 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "clarification"}`
**Conversation de preuve :** `4055b7ce-973d-477f-aff2-ded4eb2e7b74`

**Clarification retournée :** Sur quelle période souhaitez-vous consulter le PDL ? Indiquez une date complète ou une période relative explicite.

### Répétition 2 — pdl_reference — message 3

**Question :** Le 12 octobre 2026 de 08:00 à 09:00 UTC.

**Résultat :** success ; fonctionnel réussi ; durée 28.588 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_count", "total": 2, "scope": "demo-alpha"}`
**Conversation de preuve :** `4055b7ce-973d-477f-aff2-ded4eb2e7b74`

### Répétition 2 — pdl_reference — message 4

**Question :** Montre les logs de l'équipement edge-a PFS BB PFAING TACACS dans demo-alpha du 12 octobre 2026 de 08:00 à 09:00 UTC.

**Résultat :** success ; fonctionnel réussi ; durée 47.854 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_equipment", "families": "all"}`
**Conversation de preuve :** `4055b7ce-973d-477f-aff2-ded4eb2e7b74`

### Répétition 2 — pdl_reference — message 5

**Question :** Uniquement TACACS pour ce même équipement, sur la même période.

**Résultat :** success ; fonctionnel réussi ; durée 56.206 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_equipment", "families": "tacacs"}`
**Conversation de preuve :** `4055b7ce-973d-477f-aff2-ded4eb2e7b74`

### Répétition 2 — pdl_reference — message 6

**Question :** Pour edge-a dans demo-alpha, sur la même période, montre PFS, BB et PFAING, sans TACACS.

**Résultat :** success ; fonctionnel réussi ; durée 71.457 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_equipment", "families": "without_tacacs"}`
**Conversation de preuve :** `4055b7ce-973d-477f-aff2-ded4eb2e7b74`

### Répétition 2 — pdl_reference — message 7

**Question :** Recherche les accès PFAING de l'adresse IP source 192.0.2.25 dans demo-alpha le 12 octobre 2026 de 08:00 à 09:00 UTC.

**Résultat :** success ; fonctionnel réussi ; durée 46.149 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_pfaing", "total": 2}`
**Conversation de preuve :** `4055b7ce-973d-477f-aff2-ded4eb2e7b74`

### Répétition 2 — pdl_reference — message 8

**Question :** Compte les refus PFS dans demo-beta le 12 octobre 2026 de 08:00 à 09:00 UTC.

**Résultat :** success ; fonctionnel réussi ; durée 30.859 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_count", "total": 1, "scope": "demo-beta"}`
**Conversation de preuve :** `4055b7ce-973d-477f-aff2-ded4eb2e7b74`

### Répétition 2 — pdl_reference — message 9

**Question :** Compte les refus PFS dans demo-alpha le 12 octobre 2026 de 07:00 à 07:30 UTC.

**Résultat :** success ; fonctionnel échoué ; durée 35.993 s.
**Contrôles non satisfaits :** requested_scope_retained, scope_filter_matches_the_question
**Attendu :** `{"kind": "pdl_count", "total": 0, "scope": "demo-alpha", "start": "2026-10-12T07:00:00Z", "end": "2026-10-12T07:30:00Z"}`
**Conversation de preuve :** `4055b7ce-973d-477f-aff2-ded4eb2e7b74`

### Répétition 2 — pdl_reference — message 10

**Question :** Reprenons edge-a : retrouve son activité dans les quatre familles PFS, BB, PFAING et TACACS, périmètre demo-alpha, le 12 octobre 2026 entre 08:00 et 09:00 UTC.

**Résultat :** success ; fonctionnel échoué ; durée 63.502 s.
**Contrôles non satisfaits :** correct_pdl_period
**Attendu :** `{"kind": "pdl_equipment", "families": "all"}`
**Conversation de preuve :** `4055b7ce-973d-477f-aff2-ded4eb2e7b74`

### Répétition 2 — incidents_reference — message 1

**Question :** Diagnostique R1 sur le site de Rennes.

**Résultat :** clarification ; fonctionnel réussi ; durée 7.393 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "clarification"}`
**Conversation de preuve :** `caee74c5-bb52-477b-9cfe-3535940b2899`

### Répétition 2 — incidents_reference — message 2

**Question :** Le 28 septembre 2026 de 10:00 à 12:00, heure de Paris.

**Résultat :** partial ; fonctionnel échoué ; durée 18.501 s.
**Contrôles non satisfaits :** successful_incident_result, real_generated_and_reviewed_diagnostic, actual_draft_and_review_models_with_tokens
**Attendu :** `{"kind": "incident_diagnostic", "equipment": ["OBS-RTR-01"], "generation": true}`
**Conversation de preuve :** `caee74c5-bb52-477b-9cfe-3535940b2899`

### Répétition 2 — incidents_reference — message 3

**Question :** À 12 h, heure de Paris, l'alerte était-elle résolue et le ticket déjà clôturé ?

**Résultat :** ok ; fonctionnel réussi ; durée 6.386 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "incident_resolution", "equipment": ["OBS-RTR-01"]}`
**Conversation de preuve :** `caee74c5-bb52-477b-9cfe-3535940b2899`

### Répétition 2 — incidents_reference — message 4

**Question :** Fais la même analyse pour R2, sur la même période et sur le site de Rennes.

**Résultat :** ok ; fonctionnel réussi ; durée 22.664 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "incident_diagnostic", "equipment": ["OBS-RTR-02"], "generation": true}`
**Conversation de preuve :** `caee74c5-bb52-477b-9cfe-3535940b2899`

### Répétition 2 — incidents_reference — message 5

**Question :** Compare les équipements du site de Rennes le 28 septembre 2026 de 10:00 à 12:00, heure de Paris. Distingue les mesures, les alertes et les logs de chaque équipement.

**Résultat :** partial ; fonctionnel échoué ; durée 19.736 s.
**Contrôles non satisfaits :** successful_incident_result, real_generated_and_reviewed_diagnostic, actual_draft_and_review_models_with_tokens
**Attendu :** `{"kind": "incident_diagnostic", "equipment": ["OBS-RTR-01", "OBS-RTR-02"], "generation": true}`
**Conversation de preuve :** `caee74c5-bb52-477b-9cfe-3535940b2899`

### Répétition 2 — incidents_reference — message 6

**Question :** Pour R3 le 28 septembre 2026 de 10:00 à 12:00, heure de Paris, combien de points de latence sont disponibles et quelle est la latence moyenne ?

**Résultat :** partial ; fonctionnel réussi ; durée 16.669 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "incident_missing", "equipment": ["OBS-RTR-03"], "metric": "latency"}`
**Conversation de preuve :** `caee74c5-bb52-477b-9cfe-3535940b2899`

### Répétition 2 — incidents_reference — message 7

**Question :** Et pour son CPU, sur la même période : combien de points et quelle moyenne ?

**Résultat :** partial ; fonctionnel réussi ; durée 16.698 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "incident_missing", "equipment": ["OBS-RTR-03"], "metric": "cpu"}`
**Conversation de preuve :** `caee74c5-bb52-477b-9cfe-3535940b2899`

### Répétition 2 — incidents_reference — message 8

**Question :** Diagnostique R4 le 28 septembre 2026 de 10:00 à 12:00, heure de Paris.

**Résultat :** None ; fonctionnel échoué ; durée 12.635 s.
**Contrôles non satisfaits :** exactly_one_completed_native_tool, native_result_present
**Attendu :** `{"kind": "blocked_equipment"}`
**Conversation de preuve :** `caee74c5-bb52-477b-9cfe-3535940b2899`

### Répétition 2 — incidents_reference — message 9

**Question :** Diagnostique R1 le 28 septembre 2026 de 10:00 à 12:00, heure de Paris, sur le site de Rennes. Recherche les précédents ITSM et propose la solution la mieux étayée, à valider par un opérateur, avec son niveau de confiance.

**Résultat :** ok ; fonctionnel réussi ; durée 31.024 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "incident_diagnostic", "equipment": ["OBS-RTR-01"], "generation": true, "precedents": true}`
**Conversation de preuve :** `caee74c5-bb52-477b-9cfe-3535940b2899`

### Répétition 2 — incidents_reference — message 10

**Question :** À midi, heure de Paris, l'alerte était-elle déjà résolue et le ticket déjà clôturé ?

**Résultat :** ok ; fonctionnel réussi ; durée 6.54 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "incident_resolution", "equipment": ["OBS-RTR-01"]}`
**Conversation de preuve :** `caee74c5-bb52-477b-9cfe-3535940b2899`

### Répétition 2 — mixed_variants — message 1

**Question :** Combien de décisions de refus ont été enregistrées par PFS dans demo-alpha, le 12 octobre 2026 entre 8 heures et 9 heures UTC ?

**Résultat :** success ; fonctionnel échoué ; durée 29.753 s.
**Contrôles non satisfaits :** correct_pdl_period
**Attendu :** `{"kind": "pdl_count", "total": 2, "scope": "demo-alpha"}`
**Conversation de preuve :** `e8e0e158-79ec-4e1d-9c20-ce6915056e14`

### Répétition 2 — mixed_variants — message 2

**Question :** Retrouve l'activité de edge-a dans PFS, Backbone, PFAING et TACACS, pour demo-alpha, le 12 octobre 2026 de 08:00 à 09:00 UTC. Appuie-toi sur les relations entre les sources.

**Résultat :** success ; fonctionnel réussi ; durée 65.206 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_equipment", "families": "all"}`
**Conversation de preuve :** `e8e0e158-79ec-4e1d-9c20-ce6915056e14`

### Répétition 2 — mixed_variants — message 3

**Question :** Garde ce créneau et cet équipement, mais limite le résultat aux événements TACACS.

**Résultat :** success ; fonctionnel réussi ; durée 29.884 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_equipment", "families": "tacacs"}`
**Conversation de preuve :** `e8e0e158-79ec-4e1d-9c20-ce6915056e14`

### Répétition 2 — mixed_variants — message 4

**Question :** Changeons de sujet : analyse les mesures, les journaux et les tickets de R1 à Rennes le 28 septembre 2026 entre 10 h et 12 h, heure de Paris. Distingue les observations et les causes possibles.

**Résultat :** partial ; fonctionnel échoué ; durée 27.08 s.
**Contrôles non satisfaits :** correct_incident_period, successful_incident_result, exact_log_count
**Attendu :** `{"kind": "incident_diagnostic", "equipment": ["OBS-RTR-01"], "generation": true}`
**Conversation de preuve :** `e8e0e158-79ec-4e1d-9c20-ce6915056e14`

### Répétition 2 — mixed_variants — message 5

**Question :** Sur ce même créneau, fais maintenant le diagnostic de R2 à Rennes.

**Résultat :** partial ; fonctionnel échoué ; durée 17.692 s.
**Contrôles non satisfaits :** correct_incident_period, successful_incident_result, exact_log_count
**Attendu :** `{"kind": "incident_diagnostic", "equipment": ["OBS-RTR-02"], "generation": true}`
**Conversation de preuve :** `e8e0e158-79ec-4e1d-9c20-ce6915056e14`

### Répétition 2 — mixed_variants — message 6

**Question :** À 12 h, heure de Paris, l'alerte était-elle résolue et le ticket déjà clôturé ?

**Résultat :** clarification ; fonctionnel échoué ; durée 8.559 s.
**Contrôles non satisfaits :** correct_incident_equipment, correct_incident_period, successful_incident_result, five_metrics_per_equipment, exact_log_count, technical_resolution_and_ticket_close_remain_distinct, noon_answer_explicit, displayed_citations_known
**Attendu :** `{"kind": "incident_resolution", "equipment": ["OBS-RTR-02"]}`
**Conversation de preuve :** `e8e0e158-79ec-4e1d-9c20-ce6915056e14`

### Répétition 2 — mixed_variants — message 7

**Question :** Reviens aux logs de l'équipement du début de cette conversation, avec le même périmètre et la même période, dans les quatre familles.

**Résultat :** needs_clarification ; fonctionnel échoué ; durée 13.892 s.
**Contrôles non satisfaits :** business_success, correct_pdl_period, exact_expected_documents, exact_total, equipment_context_preserved, no_scope_contamination, real_planner_model_returned
**Attendu :** `{"kind": "pdl_equipment", "families": "all", "cross_uc_memory": true}`
**Conversation de preuve :** `e8e0e158-79ec-4e1d-9c20-ce6915056e14`

**Clarification retournée :** Sur quelle période souhaitez-vous consulter le PDL ? Indiquez une date complète ou une période relative explicite.

### Répétition 2 — mixed_variants — message 8

**Question :** Montre les logs de l'équipement edge-a PFS BB PFAING TACACS dans demo-alpha du 12 octobre 2026 de 08:00 à 09:00 UTC.

**Résultat :** success ; fonctionnel réussi ; durée 67.672 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_equipment", "families": "all"}`
**Conversation de preuve :** `e8e0e158-79ec-4e1d-9c20-ce6915056e14`

### Répétition 2 — mixed_variants — message 9

**Question :** Autre incident : pour R3, le 28 septembre 2026 entre 10 h et midi en heure de Paris, quelle est la latence moyenne et combien de points ont été collectés ?

**Résultat :** partial ; fonctionnel échoué ; durée 19.077 s.
**Contrôles non satisfaits :** correct_incident_period, missing_points_explicit, missing_mean_is_unknown_not_zero
**Attendu :** `{"kind": "incident_missing", "equipment": ["OBS-RTR-03"], "metric": "latency"}`
**Conversation de preuve :** `e8e0e158-79ec-4e1d-9c20-ce6915056e14`

### Répétition 2 — mixed_variants — message 10

**Question :** Pour ce même équipement et cette même période, s'agit-il d'une latence nulle ou d'une absence de collecte ? Explique la limite avant de conclure.

**Résultat :** None ; fonctionnel échoué ; durée 14.764 s.
**Contrôles non satisfaits :** exactly_one_completed_native_tool, native_result_present
**Attendu :** `{"kind": "incident_missing", "equipment": ["OBS-RTR-03"], "metric": "latency"}`
**Conversation de preuve :** `e8e0e158-79ec-4e1d-9c20-ce6915056e14`

### Répétition 2 — wrong_incident_request_in_pdl — message 1

**Question :** Diagnostique R1 à Rennes le 28 septembre 2026 de 10:00 à 12:00, heure de Paris, avec ses métriques, alertes et tickets.

**Résultat :** needs_clarification ; fonctionnel échoué ; durée 35.891 s.
**Contrôles non satisfaits :** wrong_domain_explicitly_explained
**Attendu :** `{"kind": "wrong_uc"}`
**Conversation de preuve :** `3c681d3a-f123-4c16-b1a1-d79c22246d8a`

**Clarification retournée :** Pour le diagnostic de l'équipement R1, quels types de logs souhaitez‑vous interroger (métriques, alertes, tickets) et dans quelles familles d'index (par ex. rie‑pfs‑fixture, rie‑bb‑fixture, rie‑pfaing‑fixture, rie‑tacacs‑fixture) ? De plus, quel réseau (network_scope) doit être appliqué pour cette requête ?

### Répétition 2 — wrong_pdl_request_in_incidents — message 1

**Question :** Compte les refus PFS dans demo-alpha le 12 octobre 2026 de 08:00 à 09:00 UTC.

**Résultat :** partial ; fonctionnel échoué ; durée 11.575 s.
**Contrôles non satisfaits :** wrong_domain_request_has_no_business_evidence
**Attendu :** `{"kind": "wrong_uc"}`
**Conversation de preuve :** `2e29e70a-e81d-4fe3-92e6-79c366afc639`

### Répétition 3 — pdl_reference — message 1

**Question :** Quels sont mes index accessibles ?

**Résultat :** success ; fonctionnel réussi ; durée 6.505 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "rights"}`
**Conversation de preuve :** `f9212d36-c579-4d96-8b60-520507e5834b`

### Répétition 3 — pdl_reference — message 2

**Question :** Compte les refus PFS dans demo-alpha.

**Résultat :** needs_clarification ; fonctionnel réussi ; durée 7.491 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "clarification"}`
**Conversation de preuve :** `f9212d36-c579-4d96-8b60-520507e5834b`

**Clarification retournée :** Sur quelle période souhaitez-vous consulter le PDL ? Indiquez une date complète ou une période relative explicite.

### Répétition 3 — pdl_reference — message 3

**Question :** Le 12 octobre 2026 de 08:00 à 09:00 UTC.

**Résultat :** success ; fonctionnel réussi ; durée 29.751 s.
**Contrôles non satisfaits :** no_invented_count_cap_in_visible_answer
**Attendu :** `{"kind": "pdl_count", "total": 2, "scope": "demo-alpha"}`
**Conversation de preuve :** `f9212d36-c579-4d96-8b60-520507e5834b`

### Répétition 3 — pdl_reference — message 4

**Question :** Montre les logs de l'équipement edge-a PFS BB PFAING TACACS dans demo-alpha du 12 octobre 2026 de 08:00 à 09:00 UTC.

**Résultat :** success ; fonctionnel réussi ; durée 64.237 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_equipment", "families": "all"}`
**Conversation de preuve :** `f9212d36-c579-4d96-8b60-520507e5834b`

### Répétition 3 — pdl_reference — message 5

**Question :** Uniquement TACACS pour ce même équipement, sur la même période.

**Résultat :** success ; fonctionnel réussi ; durée 48.084 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_equipment", "families": "tacacs"}`
**Conversation de preuve :** `f9212d36-c579-4d96-8b60-520507e5834b`

### Répétition 3 — pdl_reference — message 6

**Question :** Pour edge-a dans demo-alpha, sur la même période, montre PFS, BB et PFAING, sans TACACS.

**Résultat :** success ; fonctionnel réussi ; durée 73.542 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_equipment", "families": "without_tacacs"}`
**Conversation de preuve :** `f9212d36-c579-4d96-8b60-520507e5834b`

### Répétition 3 — pdl_reference — message 7

**Question :** Recherche les accès PFAING de l'adresse IP source 192.0.2.25 dans demo-alpha le 12 octobre 2026 de 08:00 à 09:00 UTC.

**Résultat :** success ; fonctionnel réussi ; durée 43.136 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_pfaing", "total": 2}`
**Conversation de preuve :** `f9212d36-c579-4d96-8b60-520507e5834b`

### Répétition 3 — pdl_reference — message 8

**Question :** Compte les refus PFS dans demo-beta le 12 octobre 2026 de 08:00 à 09:00 UTC.

**Résultat :** success ; fonctionnel réussi ; durée 28.881 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_count", "total": 1, "scope": "demo-beta"}`
**Conversation de preuve :** `f9212d36-c579-4d96-8b60-520507e5834b`

### Répétition 3 — pdl_reference — message 9

**Question :** Compte les refus PFS dans demo-alpha le 12 octobre 2026 de 07:00 à 07:30 UTC.

**Résultat :** success ; fonctionnel échoué ; durée 33.996 s.
**Contrôles non satisfaits :** requested_scope_retained, scope_filter_matches_the_question
**Attendu :** `{"kind": "pdl_count", "total": 0, "scope": "demo-alpha", "start": "2026-10-12T07:00:00Z", "end": "2026-10-12T07:30:00Z"}`
**Conversation de preuve :** `f9212d36-c579-4d96-8b60-520507e5834b`

### Répétition 3 — pdl_reference — message 10

**Question :** Reprenons edge-a : retrouve son activité dans les quatre familles PFS, BB, PFAING et TACACS, périmètre demo-alpha, le 12 octobre 2026 entre 08:00 et 09:00 UTC.

**Résultat :** success ; fonctionnel échoué ; durée 53.411 s.
**Contrôles non satisfaits :** correct_pdl_period
**Attendu :** `{"kind": "pdl_equipment", "families": "all"}`
**Conversation de preuve :** `f9212d36-c579-4d96-8b60-520507e5834b`

### Répétition 3 — incidents_reference — message 1

**Question :** Diagnostique R1 sur le site de Rennes.

**Résultat :** clarification ; fonctionnel réussi ; durée 7.552 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "clarification"}`
**Conversation de preuve :** `e951b051-a53c-4417-bd1a-f0d12b55f74a`

### Répétition 3 — incidents_reference — message 2

**Question :** Le 28 septembre 2026 de 10:00 à 12:00, heure de Paris.

**Résultat :** partial ; fonctionnel échoué ; durée 27.759 s.
**Contrôles non satisfaits :** successful_incident_result, real_generated_and_reviewed_diagnostic, actual_draft_and_review_models_with_tokens
**Attendu :** `{"kind": "incident_diagnostic", "equipment": ["OBS-RTR-01"], "generation": true}`
**Conversation de preuve :** `e951b051-a53c-4417-bd1a-f0d12b55f74a`

### Répétition 3 — incidents_reference — message 3

**Question :** À 12 h, heure de Paris, l'alerte était-elle résolue et le ticket déjà clôturé ?

**Résultat :** ok ; fonctionnel réussi ; durée 9.628 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "incident_resolution", "equipment": ["OBS-RTR-01"]}`
**Conversation de preuve :** `e951b051-a53c-4417-bd1a-f0d12b55f74a`

### Répétition 3 — incidents_reference — message 4

**Question :** Fais la même analyse pour R2, sur la même période et sur le site de Rennes.

**Résultat :** ok ; fonctionnel réussi ; durée 21.886 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "incident_diagnostic", "equipment": ["OBS-RTR-02"], "generation": true}`
**Conversation de preuve :** `e951b051-a53c-4417-bd1a-f0d12b55f74a`

### Répétition 3 — incidents_reference — message 5

**Question :** Compare les équipements du site de Rennes le 28 septembre 2026 de 10:00 à 12:00, heure de Paris. Distingue les mesures, les alertes et les logs de chaque équipement.

**Résultat :** partial ; fonctionnel échoué ; durée 24.035 s.
**Contrôles non satisfaits :** successful_incident_result, real_generated_and_reviewed_diagnostic, actual_draft_and_review_models_with_tokens
**Attendu :** `{"kind": "incident_diagnostic", "equipment": ["OBS-RTR-01", "OBS-RTR-02"], "generation": true}`
**Conversation de preuve :** `e951b051-a53c-4417-bd1a-f0d12b55f74a`

### Répétition 3 — incidents_reference — message 6

**Question :** Pour R3 le 28 septembre 2026 de 10:00 à 12:00, heure de Paris, combien de points de latence sont disponibles et quelle est la latence moyenne ?

**Résultat :** partial ; fonctionnel réussi ; durée 27.083 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "incident_missing", "equipment": ["OBS-RTR-03"], "metric": "latency"}`
**Conversation de preuve :** `e951b051-a53c-4417-bd1a-f0d12b55f74a`

### Répétition 3 — incidents_reference — message 7

**Question :** Et pour son CPU, sur la même période : combien de points et quelle moyenne ?

**Résultat :** partial ; fonctionnel réussi ; durée 20.045 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "incident_missing", "equipment": ["OBS-RTR-03"], "metric": "cpu"}`
**Conversation de preuve :** `e951b051-a53c-4417-bd1a-f0d12b55f74a`

### Répétition 3 — incidents_reference — message 8

**Question :** Diagnostique R4 le 28 septembre 2026 de 10:00 à 12:00, heure de Paris.

**Résultat :** None ; fonctionnel échoué ; durée 5.647 s.
**Contrôles non satisfaits :** exactly_one_completed_native_tool, native_result_present
**Attendu :** `{"kind": "blocked_equipment"}`
**Conversation de preuve :** `e951b051-a53c-4417-bd1a-f0d12b55f74a`

### Répétition 3 — incidents_reference — message 9

**Question :** Diagnostique R1 le 28 septembre 2026 de 10:00 à 12:00, heure de Paris, sur le site de Rennes. Recherche les précédents ITSM et propose la solution la mieux étayée, à valider par un opérateur, avec son niveau de confiance.

**Résultat :** clarification ; fonctionnel échoué ; durée 13.876 s.
**Contrôles non satisfaits :** correct_incident_equipment, correct_incident_period, successful_incident_result, five_metrics_per_equipment, exact_log_count, real_generated_and_reviewed_diagnostic, actual_draft_and_review_models_with_tokens, native_itsm_historical_precedents, conditional_solution_and_separate_confidence, displayed_citations_known
**Attendu :** `{"kind": "incident_diagnostic", "equipment": ["OBS-RTR-01"], "generation": true, "precedents": true}`
**Conversation de preuve :** `e951b051-a53c-4417-bd1a-f0d12b55f74a`

### Répétition 3 — incidents_reference — message 10

**Question :** À midi, heure de Paris, l'alerte était-elle déjà résolue et le ticket déjà clôturé ?

**Résultat :** ok ; fonctionnel réussi ; durée 8.738 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "incident_resolution", "equipment": ["OBS-RTR-01"]}`
**Conversation de preuve :** `e951b051-a53c-4417-bd1a-f0d12b55f74a`

### Répétition 3 — mixed_variants — message 1

**Question :** Combien de décisions de refus ont été enregistrées par PFS dans demo-alpha, le 12 octobre 2026 entre 8 heures et 9 heures UTC ?

**Résultat :** success ; fonctionnel échoué ; durée 40.16 s.
**Contrôles non satisfaits :** correct_pdl_period
**Attendu :** `{"kind": "pdl_count", "total": 2, "scope": "demo-alpha"}`
**Conversation de preuve :** `a15ee39b-984b-4b05-a9e0-c5cb4811ead4`

### Répétition 3 — mixed_variants — message 2

**Question :** Retrouve l'activité de edge-a dans PFS, Backbone, PFAING et TACACS, pour demo-alpha, le 12 octobre 2026 de 08:00 à 09:00 UTC. Appuie-toi sur les relations entre les sources.

**Résultat :** success ; fonctionnel réussi ; durée 48.222 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_equipment", "families": "all"}`
**Conversation de preuve :** `a15ee39b-984b-4b05-a9e0-c5cb4811ead4`

### Répétition 3 — mixed_variants — message 3

**Question :** Garde ce créneau et cet équipement, mais limite le résultat aux événements TACACS.

**Résultat :** success ; fonctionnel réussi ; durée 31.003 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_equipment", "families": "tacacs"}`
**Conversation de preuve :** `a15ee39b-984b-4b05-a9e0-c5cb4811ead4`

### Répétition 3 — mixed_variants — message 4

**Question :** Changeons de sujet : analyse les mesures, les journaux et les tickets de R1 à Rennes le 28 septembre 2026 entre 10 h et 12 h, heure de Paris. Distingue les observations et les causes possibles.

**Résultat :** partial ; fonctionnel échoué ; durée 20.158 s.
**Contrôles non satisfaits :** correct_incident_period, successful_incident_result, exact_log_count
**Attendu :** `{"kind": "incident_diagnostic", "equipment": ["OBS-RTR-01"], "generation": true}`
**Conversation de preuve :** `a15ee39b-984b-4b05-a9e0-c5cb4811ead4`

### Répétition 3 — mixed_variants — message 5

**Question :** Sur ce même créneau, fais maintenant le diagnostic de R2 à Rennes.

**Résultat :** partial ; fonctionnel échoué ; durée 17.87 s.
**Contrôles non satisfaits :** correct_incident_period, successful_incident_result, exact_log_count
**Attendu :** `{"kind": "incident_diagnostic", "equipment": ["OBS-RTR-02"], "generation": true}`
**Conversation de preuve :** `a15ee39b-984b-4b05-a9e0-c5cb4811ead4`

### Répétition 3 — mixed_variants — message 6

**Question :** À 12 h, heure de Paris, l'alerte était-elle résolue et le ticket déjà clôturé ?

**Résultat :** clarification ; fonctionnel échoué ; durée 8.698 s.
**Contrôles non satisfaits :** correct_incident_equipment, correct_incident_period, successful_incident_result, five_metrics_per_equipment, exact_log_count, technical_resolution_and_ticket_close_remain_distinct, noon_answer_explicit, displayed_citations_known
**Attendu :** `{"kind": "incident_resolution", "equipment": ["OBS-RTR-02"]}`
**Conversation de preuve :** `a15ee39b-984b-4b05-a9e0-c5cb4811ead4`

### Répétition 3 — mixed_variants — message 7

**Question :** Reviens aux logs de l'équipement du début de cette conversation, avec le même périmètre et la même période, dans les quatre familles.

**Résultat :** needs_clarification ; fonctionnel échoué ; durée 9.003 s.
**Contrôles non satisfaits :** business_success, correct_pdl_period, exact_expected_documents, exact_total, equipment_context_preserved, no_scope_contamination, real_planner_model_returned
**Attendu :** `{"kind": "pdl_equipment", "families": "all", "cross_uc_memory": true}`
**Conversation de preuve :** `a15ee39b-984b-4b05-a9e0-c5cb4811ead4`

**Clarification retournée :** Sur quelle période souhaitez-vous consulter le PDL ? Indiquez une date complète ou une période relative explicite.

### Répétition 3 — mixed_variants — message 8

**Question :** Montre les logs de l'équipement edge-a PFS BB PFAING TACACS dans demo-alpha du 12 octobre 2026 de 08:00 à 09:00 UTC.

**Résultat :** success ; fonctionnel réussi ; durée 49.866 s.
**Contrôles non satisfaits :** aucun
**Attendu :** `{"kind": "pdl_equipment", "families": "all"}`
**Conversation de preuve :** `a15ee39b-984b-4b05-a9e0-c5cb4811ead4`

### Répétition 3 — mixed_variants — message 9

**Question :** Autre incident : pour R3, le 28 septembre 2026 entre 10 h et midi en heure de Paris, quelle est la latence moyenne et combien de points ont été collectés ?

**Résultat :** partial ; fonctionnel échoué ; durée 13.251 s.
**Contrôles non satisfaits :** correct_incident_period, missing_points_explicit, missing_mean_is_unknown_not_zero
**Attendu :** `{"kind": "incident_missing", "equipment": ["OBS-RTR-03"], "metric": "latency"}`
**Conversation de preuve :** `a15ee39b-984b-4b05-a9e0-c5cb4811ead4`

### Répétition 3 — mixed_variants — message 10

**Question :** Pour ce même équipement et cette même période, s'agit-il d'une latence nulle ou d'une absence de collecte ? Explique la limite avant de conclure.

**Résultat :** None ; fonctionnel échoué ; durée 7.742 s.
**Contrôles non satisfaits :** exactly_one_completed_native_tool, native_result_present
**Attendu :** `{"kind": "incident_missing", "equipment": ["OBS-RTR-03"], "metric": "latency"}`
**Conversation de preuve :** `a15ee39b-984b-4b05-a9e0-c5cb4811ead4`

### Répétition 3 — wrong_incident_request_in_pdl — message 1

**Question :** Diagnostique R1 à Rennes le 28 septembre 2026 de 10:00 à 12:00, heure de Paris, avec ses métriques, alertes et tickets.

**Résultat :** needs_clarification ; fonctionnel échoué ; durée 26.944 s.
**Contrôles non satisfaits :** wrong_domain_explicitly_explained
**Attendu :** `{"kind": "wrong_uc"}`
**Conversation de preuve :** `5d5703a3-a990-4885-82e9-308dc9e87d33`

**Clarification retournée :** Veuillez préciser quels types de logs (métriques, alertes, tickets) vous souhaitez récupérer et dans quels index (rie-pfs-fixture, rie-bb-fixture, rie-pfaing-fixture, rie-tacacs-fixture) ils se trouvent.

### Répétition 3 — wrong_pdl_request_in_incidents — message 1

**Question :** Compte les refus PFS dans demo-alpha le 12 octobre 2026 de 08:00 à 09:00 UTC.

**Résultat :** partial ; fonctionnel échoué ; durée 10.731 s.
**Contrôles non satisfaits :** wrong_domain_request_has_no_business_evidence
**Attendu :** `{"kind": "wrong_uc"}`
**Conversation de preuve :** `f950aa09-9b7a-4514-b66b-5e78346bb59c`

## Limites de la qualification

- Trois répétitions par question ; aucune garantie statistique de fiabilité.
- Une seule identité interne fictive ; ce n’est pas une nouvelle recette des profils ACL externes ni une réception du SI-RIE réel.
- Le fil mixte change explicitement de preset ; aucun routage automatique entre UC n’est revendiqué.
- Les tokens comptés couvrent les usages métier de rédaction/relecture retournés, pas tous les appels de relais, dialogue ou planification.
- Le contrôle initial de clarification Incidents est aligné sur son statut déclaré clarification ; les assertions initiales sont conservées.
- Les invariants de restitution sont ciblés ; ils ne prouvent pas exhaustivement chaque affirmation ni une causalité.

Les états des services, les versions et les empreintes des configurations sont joints au dossier de preuves. Les mesures de cette VM ne constituent pas un dimensionnement du SI-RIE ni une qualification de ses données réelles.
