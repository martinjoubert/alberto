# Consulter et remettre les rapports de tests

Le point d'entrée principal est **Qualification** dans NiceGUI, à l'adresse
`/qualification` du portail de l'instance. Se connecter avec un compte LDAP
interne membre d’un groupe déclaré dans `ldap.administrator_group_dns` ; un
interne lecteur ne peut pas ouvrir ces rapports. Chaque campagne présente la date de génération du rapport, sa
version source, son environnement et ses compteurs ; ouvrir ensuite le rapport
Allure pour examiner les cas, durées, étapes et pièces jointes disponibles.
Le ZIP de remise se télécharge depuis la même fiche.

Les tests et la génération Allure s'exécutent sur le **poste de qualification
ou le runner CI**. Le portail consulte des rapports importés en lecture seule ;
il ne lance ni tests, ni charge, ni panne. Node.js et Allure ne sont pas
installés dans les images des moteurs ou du portail.

## Lire correctement un résultat

| État | Interprétation |
| --- | --- |
| Réussi — `passed` | L'assertion a été exécutée et satisfaite dans l'environnement indiqué |
| Échoué — `failed` | Une assertion attendue n'est pas satisfaite |
| Erreur — `broken` | L'exécution a rencontré une erreur technique ; examiner son détail |
| Sauté — `skipped` | Un cas a été ignoré avec sa raison ; ce n'est pas une réussite |
| Incomplet — `unknown` | Le résultat n'est pas établi ; la campagne ne doit pas être déclarée entièrement réussie |
| Non qualifié | Une portée annoncée reste sans validation, par exemple Albert absent ou endurance non rejouée |

Une campagne peut avoir tous ses tests exécutés réussis et conserver des
portées non qualifiées. Lire ces portées avant d'en conclure qu'un déploiement
est accepté. Comparer uniquement les versions et environnements identifiés :
un cas Linux sauté sur le Mac et rejoué dans une VM garde deux preuves distinctes.
Dans Allure, les portées déclarées non qualifiées portent le titre
« Non qualifié — … » et sont affichées parmi les cas sautés ; NiceGUI les compte
séparément des cas sautés pour un motif technique. Elles n'ont pas de durée de
test inventée.
Le pourcentage Allure peut afficher 100 % des exécutions réussies tout en
conservant des cas sautés : lire aussi les compteurs et l'état de NiceGUI.
Les [résultats des campagnes antérieures](08-qualification.md) ne deviennent
pas des tests nouvellement exécutés parce qu'un rapport est remis en forme.
La date de génération du HTML se distingue des dates d'exécution mesurées et
de la date d'origine des reçus importés, conservées dans les résultats.

Pour la remise du 13 novembre, `ubuntu-final-operations-c2-20261113` regroupe
231 contrôles d'installation, reprise, pannes, identité native, ACL et navigateur.
`ubuntu-provider-first-failed-20261113` conserve les trois assertions échouées
du premier pilote fournisseur ; sa correction ne réécrit pas cet essai.
`ubuntu-final-endurance-20261113` présente les treize critères finaux et les
trois paliers réellement exécutés : 8 459 demandes en 7 200,087 s. Sa portée
non qualifiée mentionne le fournisseur IA de charge fictif, le SI réel et
la perte d'environ 31 minutes du journal d'accès ITSM par rotation.
Ces projections importent les verdicts, dates et empreintes des reçus sans
rejouer les tests ni inventer une durée individuelle. Le
[bilan de remise](20-final-ubuntu-qualification.md) distingue ces rapports de
l'artifact Allure GitLab, dont l'import nécessite un téléchargement authentifié.

Pour le lot ITSM du 10 novembre, `itsm-relecture-20261110` décrit le profil
final Gemma pour la rédaction et GPT-OSS pour la relecture via Albert.
`itsm-relecture-essais-20261110` conserve séparément le refus humain initial et
les essais intermédiaires, avec leurs échecs. Les contrôles à brouillons injectés
sont signalés comme tels ; ils ne valent ni collecte fraîche ni génération d'un
diagnostic complet. La validation humaine du dernier rendu reste non qualifiée.

## Comprendre un test rouge ou orange

Dans Allure, ouvrir **Suites de tests**, sélectionner le cas en échec puis
examiner le message, l'attendu/obtenu et la trace Python avec fichier et ligne.
Les pièces jointes disponibles comprennent `stdout`, `stderr` et les logs
Python ; les parcours Playwright ajoutent les erreurs console/JavaScript,
les échecs réseau et les statuts HTTP en erreur, ainsi que les captures possibles.
Une capture ou une fermeture du navigateur en échec ne remplace pas la cause
initiale. Si Chromium ne démarre pas, le reçu conserve aussi cette erreur.

Chaque canal texte est limité à **64 Kio**, avec un marqueur de troncature.
Les secrets reconnus et les mots de passe explicitement fournis au navigateur
sont masqués. Les deux formats d’identifiants de reprise conversationnelle
sont aussi masqués dans les textes, y compris les arguments JSON sérialisés.
Une image n’est pas masquée par ce traitement texte : examiner les captures
avant remise et écarter celles qui montrent un identifiant. Les événements HTTP ne collectent ni corps, ni en-têtes,
ni paramètres de requête. Le masquage ne remplace pas la sélection des pièces
autorisées à être remises. Les fichiers `*-process.log` conservés dans les
artefacts CI expliquent aussi les échecs avant la découverte des tests.

Une campagne coupée conserve un résultat **« Campagne non terminée »**, avec
son dernier point de contrôle. Elle ne devient pas verte parce que ses
premiers tests avaient réussi. Les suites qui suivent un arrêt opérateur sont
signalées comme non exécutées.

Les anciens reçus natifs importés conservent leurs contrôles, date et empreinte ;
ils ne recréent pas des logs historiques absents. Pour un nouveau reçu en échec,
l'opérateur peut joindre un journal texte sélectionné, par exemple :

```bash
python deploy/ubuntu/test_reporting.py --import-type operations --input /PRIVATE/PATH/report.json --allure-results /PRIVATE/PATH/results/allure-results --log-file /PRIVATE/PATH/selected-diagnosis.log
```

Ce journal est joint aux contrôles en échec après limitation et masquage.
Les logs Docker des moteurs et des sources ne sont pas aspirés automatiquement :
les consulter séparément avec la procédure d'[exploitation](07-operations.md).

## Conserver les journaux pendant une endurance

Sur **l'hôte Ubuntu de qualification**, avec les droits Docker de l'opérateur,
utiliser l'instance de charge distincte, explicitement préparée en mode simulé.
Le wrapper suivant démarre la capture avant les demandes et la termine après
la dernière réponse. Les cinq services de charge sont inclus ; ajouter les
noms exacts des conteneurs sources supplémentaires du laboratoire :

```bash
sudo python3 deploy/ubuntu/qualification_logs.py "$RIE_LOAD_INSTANCE" \
  --output /PRIVATE/PATH/new-campaign \
  --duration 7200 --period 5 \
  --log-container ITSM_API_CONTAINER_NAME \
  --log-container ITSM_DATABASE_CONTAINER_NAME
```

Le répertoire doit être neuf et extérieur aux sources. `endurance/` contient
les mesures habituelles ; `logs/` conserve les flux bruts privés, leurs tailles
et empreintes. Le reçu global n'est réussi que si la charge et la capture
réussissent. Un flux terminé prématurément, même avec un code zéro, ou le
dépassement de la limite globale de 512 Mio interdit un verdict réussi.
Une interruption conserve un reçu incomplet. Seuls les clients de lecture
des journaux sont arrêtés ; les conteneurs sources et leur rotation ne changent pas.

Le test Docker facultatif suivant crée un seul conteneur jetable sans réseau
ni volume métier, depuis une image Python déjà présente. Il émet 500 lignes,
force une rotation de deux fichiers de 1 Kio, puis coupe volontairement un
client de capture. Il ne démarre pas Albert et ne modifie aucun service métier :

```bash
sudo python3 deploy/ubuntu/tests/qualification_log_rotation.py \
  --image IMAGE_LOCALE_CONTENANT_PYTHON \
  --output /PRIVATE/PATH/new-log-rotation-test
```

Attendu : `passed: true`, 500 lignes capturées une seule fois et dans l'ordre,
moins de 500 lignes encore retenues par Docker et interruption du client
correctement refusée. Les flux bruts restent privés ; exporter seulement les
reçus, compteurs et empreintes explicitement sélectionnés. La capture est un
outil hôte facultatif et n'ajoute aucune dépendance aux images des moteurs.

## Rejouer les conversations longues avec Albert réel

La matrice `deploy/ubuntu/examples/openwebui-stability.json` couvre trois jeux
de dix messages, répétés trois fois : questions PDL du tutoriel, questions
Incidents du tutoriel, puis reformulations avec changements d’UC dans un même
fil. Six demandes au mauvais preset complètent les 90 échanges. Le profil
change explicitement dans Open WebUI ; ce parcours ne suppose pas de routeur
automatique entre les UC.

Ce harnais est destiné au laboratoire déjà configuré : profils
`recette-20261109-pdl` et `recette-20261109-incidents`, identité interne fictive
Alice et corpus des tutoriels. Il ne constitue pas une recette prête à utiliser
avec les noms, périodes ou identités du SI réel.

Sur le **poste de qualification**, dans l’environnement Python avec Playwright,
utiliser une session Open WebUI de qualification valide, conservée dans un
fichier privé de forme `{"token": "SESSION_PRIVÉE"}`. Ce fichier contient le
jeton de session Open WebUI ; aucune clé Albert ne figure dans la commande.
Le fournisseur est celui des profils existants, à contrôler avant l’exécution.

Compléter l’environnement de qualification décrit plus bas avec les dépendances
de navigateur verrouillées. Chrome doit être installé sur ce poste ; le harnais
utilise son exécutable et ne télécharge pas de navigateur :

```bash
python -m pip install --only-binary=:all: --require-hashes -r deploy/ubuntu/requirements-browser.lock
```

```bash
python -B deploy/ubuntu/tests/browser_stability.py \
  --session /PRIVATE/PATH/session-alice.json \
  --output /PRIVATE/PATH/albert-conversations
```

L’adresse Open WebUI est `http://127.0.0.1:8080` par défaut ; `--url` et
`--browser` permettent de désigner le client et le navigateur de qualification.
Chaque réponse conserve le texte affiché, le résultat natif, les contrôles et
une capture du seul fil synthétique. Aucun échec métier n’est relancé. Un délai
de 61 secondes après chaque demande Incidents conserve la limitation du profil.
Le code de sortie zéro indique que la campagne est terminée, pas que tous les
échanges ont réussi : examiner les résultats et leurs assertions.

Après la campagne, produire l’évaluation et les résultats Allure dans des
destinations neuves :

```bash
python -B deploy/ubuntu/tests/report_stability.py \
  --state /PRIVATE/PATH/albert-conversations \
  --output /PRIVATE/PATH/conversation-evaluation \
  --allure-results /PRIVATE/PATH/conversation-evaluation/allure-results
```

Attendu : `report.md`, `assessment.json`, `exchanges.csv` et les 96 résultats
Allure. Les contrôles comparent les périodes réellement exécutées, équipements,
sources, résultats et modèles retournés aux références du corpus ; un même
compte de documents ne masque pas une mauvaise période. Les assertions initiales
restent jointes lorsqu’un contrôle du harnais a été corrigé. Une campagne
incomplète ne peut pas produire le rapport Allure final. Le mode
`--allow-incomplete`, sans export Allure final, sert seulement au suivi.

Le HTML et le ZIP se construisent puis s’importent avec les commandes de cette
page. Cette campagne sollicite le fournisseur et conserve des rapports : elle
reste une action explicite de qualification, distincte des tests déterministes
de CI. Trois répétitions ne prouvent ni l’absence de défaut ni le comportement
sous charge ; les autres profils ACL et les sources du SI gardent leur recette.

## Éprouver le reporting avec des fautes volontaires

Après préparation de l'environnement de qualification ci-dessous, le harnais
suivant n'appelle aucun SI, fournisseur IA ou backend. Il provoque
des assertions, erreurs, délais dépassés, échecs de préparation/nettoyage et
interruptions pour vérifier leur conservation :

```bash
python -B deploy/ubuntu/tests/qualification_reporting_faults.py --output /PRIVATE/PATH/reporting-faults
```

Attendu : code zéro **du harnais** si les processus fautifs ont échoué comme
prévu et si les statuts, traces et logs sont conservés. Les cas injectés restent
en échec dans leur rapport Allure séparé, intitulé
**« Épreuves négatives du reporting — échecs attendus »**. Ils ne sont jamais
fusionnés avec les résultats métier. Une option `--node /PATH/node` génère
aussi directement son HTML et son ZIP, après installation des outils ci-dessous.
La CI rejoue ce contrôle et conserve son rapport séparément.

## Produire un rapport sur le poste de qualification

Depuis la racine du **dépôt complet**, avec Python 3.12, Node.js **24.22.2 LTS**
et npm disponibles. Ces outils concernent le poste de qualification, pas les
prérequis d'exploitation du serveur. Préparer un environnement Python dédié :

```bash
python3.12 -m venv /PRIVATE/PATH/venv-qualification
. /PRIVATE/PATH/venv-qualification/bin/activate
python -m pip install --only-binary=:all: --require-hashes -r deploy/ubuntu/requirements-runtime.lock -r ci/locks/requirements-dialogue.lock
npm ci --prefix deploy/ubuntu/reporting --ignore-scripts --no-audit --no-fund
```

Attendu : dépendances Python contrôlées par empreintes et installation d'Allure
**3.33.0**, avec les versions transitives de `package-lock.json`. La préparation
peut nécessiter l'accès aux dépôts autorisés ; les suites déterministes suivantes
bloquent leurs connexions réseau et neutralisent les clés IA ambiantes.

Choisir un **nouveau** dossier privé de campagne, puis exécuter les suites :

```bash
export RIE_REPORTS_DIR=/PRIVATE/PATH/qualification-20261107
python -B ci/runners/run_ubuntu_tests.py --reports-dir "$RIE_REPORTS_DIR"
```

Attendu : `source.xml`, `pdl.xml`, `incidents.xml` et `allure-results/` dans ce
dossier. Consigner le code de retour : zéro signifie réussite des suites
exécutées, un code non nul interdit de valider la remise. Produire le rapport
également après un échec pour pouvoir l'examiner ; ne pas transformer l'échec
du test en réussite de pipeline.

```bash
python deploy/ubuntu/build_report.py --results "$RIE_REPORTS_DIR/allure-results" --output /PRIVATE/PATH/report-20261107 --id qualification-20261107 --title "Qualification PDL, Incidents et ACL" --environment "Ubuntu 24.04 ARM64" --not-qualified "Conversations Albert non exécutées" --not-qualified "Endurance de deux heures non exécutée" --not-qualified "Recettes natives et navigateur non exécutées par les suites hors réseau"
```

Adapter l'environnement au système **réellement testé** : écrire « macOS ARM64 »
pour une exécution sur le Mac. Le dossier de sortie doit être nouveau. Attendu :
`index.html`, `report.json`, `qualification-report.zip` et son fichier `.sha256`.
Le ZIP contient
exactement `report.json` et `index.html`, avec l'empreinte et la taille du HTML.
Conserver ensemble le ZIP, son empreinte et les résultats bruts privés de la
campagne. La génération HTML ne relance pas les tests.
Répéter `--not-qualified` pour chaque portée non jouée et adapter les motifs à
la campagne réelle ; retirer un motif uniquement lorsqu'une preuve correspondante
a effectivement été produite et intégrée.

La recette navigateur utilise déjà **Playwright Python et Chromium**. Son option
`--allure-results DOSSIER` ajoute les résultats et captures à la campagne choisie ;
elle conserve aussi son JSON de recette. Elle requiert une instance de test
disponible et le fichier privé des comptes fictifs. Les suites hors réseau
précédentes ne démarrent pas implicitement cette recette. Vitest n'est pas utilisé
dans ce périmètre. La préparation du navigateur et le contrôle du nouveau
parcours Allure sont décrits [plus bas](#vérifier-le-parcours-navigateur).

## Importer puis consulter dans NiceGUI

Sur le **serveur Ubuntu**, à la racine de la livraison courante, avec les droits
de l'opérateur et le même chemin d'instance que lors de l'installation. Remettre
le ZIP généré par cette chaîne de qualification par le canal privé prévu,
vérifier sa provenance et son empreinte, puis :

```bash
sudo python3 deploy/ubuntu/manage.py import-report "$RIE_INSTANCE" --input /PRIVATE/PATH/qualification-report.zip
```

Attendu : rapport importé sous son identifiant, sans arrêt des services ni
appel Docker. L'import vérifie la structure et les empreintes avant publication ;
un identifiant déjà présent ne peut pas être remplacé. Une nouvelle exécution
utilise un nouvel identifiant, même si elle reprend les mêmes scénarios.

Sur le **poste utilisateur**, ouvrir le portail par le tunnel du
[manuel d'installation](01-installation.md#raccorder-le-poste-utilisateur--après-a-ou-b),
puis **Qualification**. Après connexion LDAP autorisée, retrouver l'identifiant
importé, vérifier date/version/environnement, ouvrir Allure et télécharger le ZIP.
Un catalogue vide signifie qu'aucun rapport n'a été importé. Un refus d'accès
requiert un compte habilité ; il ne justifie pas de partager le compte d'un autre
utilisateur.

Le HTML autonome se consulte sans CDN ni analytique externe. L'accès NiceGUI reste celui du portail
optionnel ; Open WebUI demeure installé localement sur les postes utilisateurs.

## Vérifier le parcours navigateur

Après import d'un rapport, sur le **poste de qualification** relié au portail
de test, préparer un environnement séparé de celui des moteurs. Le lock dédié
installe Playwright **1.63.0** et ses trois dépendances, avec empreintes :

```bash
python3.12 -m venv /PRIVATE/PATH/browser-venv
. /PRIVATE/PATH/browser-venv/bin/activate
python -m pip install --only-binary=:all: --require-hashes -r deploy/ubuntu/requirements-browser.lock
python -m playwright install chromium
```

Attendu : Playwright est importable et son Chromium est installé pour le compte
de qualification. Sur un poste Ubuntu neuf, si les bibliothèques système du
navigateur manquent, leur installation requiert les droits administrateur :

```bash
sudo /PRIVATE/PATH/browser-venv/bin/python -m playwright install-deps chromium
```

Cette préparation peut télécharger des paquets ; elle n'est pas une commande
à lancer dans les images ou conteneurs applicatifs. Un navigateur Chrome déjà
installé peut aussi être désigné par `--browser-executable CHEMIN_ABSOLU`.

Préparer le JSON privé des deux comptes **fictifs** `alice.martin` et
`eric.durand.ext`, avec le format suivant et les mots de passe récupérés selon
la [procédure des credentials de démonstration](06-acl.md). Le dossier reste
`0700` et le fichier `0600` ; ne pas y placer de secret LDAP administrateur :

```json
{"users":[{"uid":"alice.martin","password":"MOT_DE_PASSE_FICTIF_ALICE"},{"uid":"eric.durand.ext","password":"MOT_DE_PASSE_FICTIF_ERIC"}]}
```

Depuis la racine du dépôt, adapter l'URL au tunnel et l'identifiant au rapport
effectivement importé :

```bash
python -B deploy/ubuntu/tests/browser_qualification.py --url http://127.0.0.1:18090 --accounts-file /PRIVATE/PATH/synthetic-accounts.json --output /PRIVATE/PATH/browser-acceptance-20261107 --report-id qualification-20261107
```

Attendu : code zéro et `browser-results.json` réussi, avec captures et ZIP
réexporté. La recette vérifie les refus anonymes et externes, les compteurs,
le rendu Allure réel, l'isolation du cadre, les empreintes de l'export et la
révocation des liens après déconnexion. Elle n'appelle pas Albert et ne modifie
pas les politiques. Ajouter `--expected-case ACTUAL_CASE_NAME` pour vérifier
aussi sa présence dans la recherche de la vue Suites Allure. Ce nom doit être
celui d'un résultat de cette campagne, pas un texte arbitraire.

## Rapports dans la CI et conservation

Les pipelines de livraison produisent les résultats et le rapport comme artefacts,
y compris lorsqu'un test échoue. Le statut du job conserve l'échec ; un rapport
HTML produit n'est pas une validation. Télécharger le ZIP du job correspondant
à la version attendue, puis suivre le même import opérateur. Aucun déploiement
vers NiceGUI ni publication publique n'est déclenché par la génération du rapport.

Les rapports consultables par le portail sont distincts des dossiers bruts
`.runtime/ci/`, qui peuvent contenir des données privées. Ne pas importer ces
dossiers en bloc. Les traces et captures jointes restent des pièces à remettre
par le canal autorisé pour la campagne.

Conserver les **ZIP de rapports et leurs empreintes séparément** de la sauvegarde
froide applicative. Le dossier des rapports importés n'est pas présenté comme
inclus dans cette sauvegarde ; sur une instance restaurée, réimporter les ZIP
nécessaires. Consulter l'occupation avec `sudo du -sh "$RIE_INSTANCE/qualification"`.
Le portail affiche les vingt publications les plus récentes. Archiver les
anciens ZIP avant de déplacer explicitement leurs dossiers hors de
`qualification/` ; les liens vers une publication déplacée deviennent indisponibles.
Aucune purge automatique n'est mise en place. Les résultats Albert
et les deux heures d'endurance historiques gardent leur version et leurs limites.

Le générateur utilise le [format de résultat Allure](https://allurereport.org/docs/how-it-works-test-result-file/)
et le [rendu Allure Report 3](https://allurereport.org/docs/v3/generate-report/).
La préparation de Chromium suit la [documentation Playwright Python](https://playwright.dev/python/docs/browsers).


## Génération réelle avec Albert — lot ITSM du 10 novembre

Les suites hors réseau contrôlent les contrats et les erreurs avec des doubles
de test ; elles ne démontrent pas un appel Albert. La recette de génération
exige séparément, pour PDL, les modèles retournés par l'interpréteur et le planner,
et pour Incidents, une synthèse demandée, une analyse présente, une relecture
valide et les modèles/tokens effectivement retournés pour rédaction et relecture.
Le contrôle technique NiceGUI sans génération ne coche aucun de ces critères.

La fonction `generation_checks` de `deploy/ubuntu/qualification_openwebui.py`
applique ces conditions aux reçus de qualification. Ses régressions sont dans
la suite source exécutée par la CI. Les conversations réelles demandent un accès
Albert disponible et une identité individuelle ; elles restent une campagne
séparée, dont les preuves sont importées dans Allure avec leur date et leur version.
Aucun secret Albert ni jeton utilisateur n'entre dans le rapport public.

Les erreurs réelles de format, de contraintes ou de relecture sont conservées
avec leur statut. Un nouvel essai réussi ne transforme pas l'essai précédent
en réussite. Une synthèse rejetée laisse un rapport partiel explicite et retire
la solution de la conclusion validée. L'état « non qualifié » reste requis
lorsque la génération réelle n'a pas pu être exécutée.


Le correctif d'attribution du 10 novembre publie la campagne
`itsm-citations-20261110`. Elle distingue les régressions avec doubles hors
réseau, les cinq épreuves du **relecteur Albert réel avec affirmations injectées**,
et les deux diagnostics entièrement générés par Albert dans les interfaces.
Les preuves n'ont pas la même portée : les cinq épreuves ne qualifient pas la
rédaction. Le constat humain d'une mauvaise attribution dans l'ancienne version
reste conservé, et l'acceptation métier du nouveau rendu reste non qualifiée.


La phase `itsm-relecture-20261110` vérifie ensuite la correction bornée d'une
synthèse réellement rejetée pendant la recette humaine. Son rejeu reprend le
premier brouillon ; seuls le refus, la correction et la seconde relecture sont
réellement demandés au fournisseur. La campagne distingue ce contrôle des
générations fraîches des interfaces et conserve le résultat partiel initial.
Le numéro d'essai accompagne les usages fournisseur, sans masquer un rejet
antérieur ni transformer une correction non validée en réussite.


## Recette du formulaire de mappings PDL

Les régressions serveur et les contrôles NiceGUI en mémoire sont collectés par
le runner Ubuntu de la CI : rôles, interdiction d’index, révocation, schéma
modifié, concurrence, import invalide, proposition inactive et approbation
explicite. Ils n’emploient ni réseau ni Albert. Le contrôle navigateur séparé
`deploy/ubuntu/tests/browser_mappings.py` utilise Playwright et de vrais services.
Il produit captures, diagnostics filtrés, `summary.json` et résultats Allure.

Le lancer sur une **instance de qualification distincte**, avec ses propres
volumes, ports et clés. Préparer quatre revues fictives en attente, sans les
approuver. Fournir le JSON privé des comptes fictifs (`users`, entrées `uid` et
`password`) ; ne pas le joindre au rapport. `--expected-instance-id` doit être
l’identifiant exact de cette instance : le navigateur le compare avant toute
écriture. L’option explicite `--exercise-approval` permet d’approuver un seul
index de cet environnement d’essai ; l’omettre conserve les revues en attente.
La recette ne doit pas être dirigée vers une instance de production.

Sur le poste de qualification, avec l’environnement navigateur déjà préparé :

```bash
# Joue les contrôles sur le tunnel de la seule instance d’essai ; remplacer les chemins et l’identifiant.
sudo /PRIVATE/PATH/browser-venv/bin/python -B deploy/ubuntu/tests/browser_mappings.py --url http://127.0.0.1:28091 --expected-instance-id IDENTIFIANT_32_HEXA --accounts-file /PRIVATE/PATH/synthetic-accounts.json --exercise-approval --browser-executable /ABSOLUTE/PATH/chromium --output /PRIVATE/PATH/mapping-browser --allure-results /PRIVATE/PATH/mapping-browser/allure-results
```

**Attendu :** code zéro et `passed: true`. Une erreur conserve sa trace filtrée
et ses captures avant la reprise ; utiliser un nouveau dossier de rapport. Une
approbation de test ne remplace pas la revue métier du SI réel et ne qualifie
pas une conversation Albert. Pour une nouvelle exécution complète, préparer une
nouvelle instance ou de nouvelles revues fictives, en conservant les anciennes
preuves. La [présentation Allure](13-test-reports.md) utilise ces résultats comme
ceux des autres recettes navigateur.
