# Clôture des réserves de preuve — novembre 2026

Cette page suit les deux réserves locales de la
[remise du 13 novembre](20-final-ubuntu-qualification.md). Les résultats initiaux
et les quelque 31 minutes de journal ITSM perdues restent conservés ; une
nouvelle preuve ne recrée pas ces lignes historiques.

## Génération Albert réellement testée

La campagne Open WebUI du 12 novembre a exécuté trois répétitions, 96 messages
conformes et neuf fils longs conformes, sur les deux UC. Les reçus conservent
les modèles et usages effectivement retournés : Gemma 4 pour le dialogue et la
planification PDL ainsi que la rédaction Incidents ; GPT-OSS pour le dialogue
et la relecture Incidents. Ils comptent notamment 45 interprétations PDL,
39 planifications, 36 interprétations Incidents, 31 rédactions et 31 relectures.
Les demandes de droits et les refus restent déterministes lorsqu'aucune
génération n'est nécessaire. Voir [la campagne et ses limites](19-openwebui-corrections.md).

Un contrôle ciblé a également été rejoué le **13 novembre, à 22 h 30 (Paris)**,
dans deux nouvelles conversations Chrome/Open WebUI, avec l'identité individuelle
d'Alice. Les **24 critères réussissent**, sans nouvelle tentative métier :
six documents PDL attendus, équipement et période exacts, puis diagnostic
Incidents avec cinq mesures, 120 logs, rédaction et relecture validées.
Albert retourne Gemma pour PDL et la rédaction Incidents, GPT-OSS pour le
dialogue et la relecture Incidents. Les reçus Incidents enregistrent respectivement
4 289 et 5 634 tokens pour la rédaction et la relecture.

Le rapport `albert-two-uc-smoke-20261113` est disponible dans le
[catalogue NiceGUI](http://127.0.0.1:18090/qualification), avec 24 réussites
et deux périmètres non qualifiés : charge simultanée Albert et SI-RIE réel.
Ses huit contrôles navigateur passent, dont le rendu Allure, l'export et les
refus d'accès. Ce contrôle ciblé ne remplace pas les trois répétitions et les
conversations longues du 12 novembre.

L'endurance utilise un fournisseur PDL fictif et Incidents sans synthèse.
Elle mesure les moteurs, leurs sources et leurs ACL sous charge, et ne compte
jamais comme une validation de capacité simultanée d'Albert.

## Capture continue malgré la rotation

Le nouveau wrapper hôte `qualification_logs.py` lance `docker logs --follow`
avant les demandes. Les flux vont directement dans des fichiers privés, sans
être accumulés en mémoire Python. La rotation des services reste activée.
Un arrêt inattendu du client, une interruption ou un dépassement de la limite
de capture ne peut pas devenir un succès de qualification.

Les neuf régressions locales passent. Le test réel sur Docker Ubuntu capture
les **500 lignes une seule fois et dans l'ordre**, alors que Docker n'en retient
plus que **cinq** après rotation. La coupure volontaire du client est détectée,
et le conteneur source reste démarré. Le conteneur canari est supprimé ensuite.

La campagne complète utilise les contextes métier et de charge
`5afa381bc374efc4c4c7` et `d7f64e439f44ffecb17e`. Le nouvel outil est facultatif,
exécuté sur l'hôte de qualification et absent de ces deux images. Les commandes,
permissions, résultats attendus et limites sont dans
[le parcours de capture](13-test-reports.md#conserver-les-journaux-pendant-une-endurance).

**Nouvelle endurance interrompue à la demande de l'utilisateur le 13 novembre
à 22 h 46 (Paris), avant extinction du Mac.** Les 263 demandes terminées
ne présentent aucune erreur inattendue ; les sept captures privées sont
conservées. Ce résultat partiel **ne valide pas les deux heures**. Les dix
jetons temporaires de la campagne ont été révoqués ; les vingt anciens
enregistrements de jetons restent inchangés. Les données et disques sont conservés.

### Campagne complète du 14 novembre

**Réserve de capture levée.** La nouvelle fenêtre va de **9 h 34 à 11 h 34
(Paris)**, sans addition de la veille : **7 200,098 secondes, 8 460 demandes,
zéro erreur métier inattendue et 810 refus ACL attendus sans preuve divulguée**.
Aucun OOM, arrêt ou redémarrage spontané n'est observé. Les dix identités LDAP
et leurs dix comptes ITSM restent distincts. La cadence minimale mesurée est
de 5,000017 secondes entre deux départs d'un même utilisateur ; aucune violation.
Le maximum observé est de neuf requêtes simultanées pour dix utilisateurs.

| Utilisateurs | Durée demandée | Demandes | PDL p95 (ms) | Incidents autorisés p95 (ms) | Refus ACL attendus |
| --- | --- | ---: | ---: | ---: | ---: |
| 1 | 30 min | 360 | 28,175 | 620,801 | 0 |
| 5 | 45 min | 2 700 | 36,798 | 1 206,020 | 270 |
| 10 | 45 min | 5 400 | 100,790 | 2 025,913 | 540 |

Les refus rapides restent séparés des diagnostics autorisés dans ces latences.
À dix utilisateurs, le pic CPU échantillonné est de 20,739 % des huit vCPU,
le pic de mémoire utilisée de 2,47 Gio et aucun swap utilisé. Les mesures
p50/p95/p99 et les ressources de chaque palier restent dans les reçus et
les pièces jointes Allure. Les maxima sont échantillonnés, sans recherche de
saturation ; ces valeurs ne dimensionnent pas le SI-RIE ni un serveur AMD64.

Les **sept flux continus totalisent 47 103 101 octets**. Leurs clients restent
actifs sur toute la fenêtre ; tailles et empreintes sont vérifiées. Les
3 059 lignes de fin encore retenues par Docker sont toutes présentes dans les
captures. Aucun trou supérieur à trente secondes n'est observé sur les trois
flux continuellement actifs, et aucun marqueur d'erreur, exception ou alerte
mémoire n'est relevé. Les flux silencieux sont conservés aussi. Les lignes brutes
restent privées ; les compteurs et empreintes sont exportables.

La comparaison complète confirme les **100 000 événements PDL, 100 000 logs
Incidents, dix documents et 4 200 points sur vingt séries métriques**, sans
réparation. Les 8 036 fichiers antérieurs, la politique, les mappings, les
anciens jetons et la configuration restent identiques. Les tickets, solutions
et suivis ITSM sont préservés ; les dix jetons temporaires sont révoqués.
La fenêtre a produit 4 230 rapports PDL et 3 420 rapports Incidents, soit
362 988 824 octets de rapports immuables supplémentaires, sans suppression automatique.

### Anomalie du collecteur et correctif vérifié séparément

**Le verdict global initial reste en échec.** À 11 h 31, un cache de conversation
expire entre son énumération et sa lecture par le collecteur. Celui-ci lève
`FileNotFoundError` : un comptage de fichiers/octets manque parmi les 420 relevés.
Le CPU, la mémoire, l'espace disque et l'état des conteneurs de ce relevé sont
bien enregistrés. Aucun rapport ni document métier n'est perdu.

Cette seule cause produit trois indicateurs d'échec par propagation dans le
pilote et son wrapper. Le rapport `ubuntu-endurance-storage-c2-20261114`
conserve **33 contrôles réussis et trois échoués** ; ses deux périmètres non
qualifiés concernent Albert sous charge et le SI réel. Un libellé de diagnostic
a été clarifié dans la publication C2, sans modifier les verdicts ; la première
projection reste dans l'historique privé.

Le correctif compte explicitement les entrées disparues pendant le relevé et
conserve les autres erreurs, notamment les refus d'accès au système de fichiers.
Il ne désactive pas le cache. Trois régressions déterministes passent, ainsi que
**250 relevés dans Docker avec 119 expirations concurrentes**. L'image de
qualification reconstruite `ae2921050bb626af86af` passe ensuite un **smoke de
60,092 secondes, 73 demandes, sans erreur de collecte ni modification du corpus**.
Le contexte des moteurs reste `5afa381bc374efc4c4c7`.

Le rapport distinct `ubuntu-storage-sampler-fix-20261114` compte **onze réussites**.
Les non-régressions locales passent : 1 293 réussites et un test Linux ignoré sur
Mac. Ces contrôles ne transforment pas rétroactivement la campagne initiale en
succès global. **Une nouvelle endurance de deux heures après ce correctif n'est
pas rejouée, conformément à la décision de l'utilisateur.** Les deux rapports
restent accessibles dans le [catalogue NiceGUI](http://127.0.0.1:18090/qualification).

## Import du rapport CI privé dans NiceGUI

**Réserve levée le 14 novembre.** Le téléchargement authentifié utilise un
jeton HTTPS de lecture d'API enregistré dans le trousseau macOS. La clé SSH
continue de servir aux opérations Git. Le jeton n'entre ni dans la livraison,
ni dans un fichier `.env`, ni dans la configuration du portail.

Le véritable artifact de
[`report:ubuntu`](https://gitlab.com/clients_neovelion_tech/dinum_rie_demo_albert_api/-/jobs/18021645247),
issu de la [CI du commit `ad0898d`](https://gitlab.com/clients_neovelion_tech/dinum_rie_demo_albert_api/-/pipelines/3066660031),
a été téléchargé puis importé dans NiceGUI sous l'identifiant
`gitlab-3066660031-18021645247`. Les sept jobs de cette CI réussissent ; le
rapport compte **1 291 réussites, zéro échec et trois périmètres non qualifiés**.
Il conserve sa date de génération et son commit source propre ; ce n'est pas
une reconstruction locale présentée comme un résultat CI.

L'empreinte du ZIP sélectionné correspond au journal CI ; les empreintes du
HTML et ses ressources autonomes ont été vérifiées. Les **huit contrôles
navigateur passent**, dont le rendu Allure, les métadonnées, l'export et les
refus d'accès. L'import n'a redémarré aucun service.

Le correctif du collecteur dispose ensuite de sa propre
[CI `4405317`](https://gitlab.com/clients_neovelion_tech/dinum_rie_demo_albert_api/-/pipelines/3068108416),
avec **sept jobs réussis et 1 294 tests réussis sans saut Linux**. Son véritable
artifact du [job `18031174685`](https://gitlab.com/clients_neovelion_tech/dinum_rie_demo_albert_api/-/jobs/18031174685)
est également vérifié puis importé sous `gitlab-3068108416-18031174685`.
Ses huit contrôles navigateur passent. Les deux publications CI conservent
chacune leur commit, leur date et leurs empreintes.

Les trois périmètres non qualifiés concernent cette seule suite de CI :
déploiement/exploitation sur VM, navigateur/Albert réel, puis endurance et
performances SI-RIE. Les campagnes VM et Albert gardent leurs rapports
distincts ; leur exécution ne modifie pas rétroactivement les verdicts CI.

La réception du SI-RIE, ses identités et données réelles, ses accès Albert et
le dimensionnement de sa machine restent dans la
[matrice de réception](14-si-rie-acceptance.md). Ces validations ne sont pas
clôturées par les deux réserves locales.
