# Précédents ITSM-NG et solution proposée — backlog et recette du PoC

> Complément de référence — le [guide principal](../README.md) reste le seul parcours de déploiement à exécuter.

Version cible confirmée : **ITSM-NG 1.6.17**. Le contrat de référence est celui
du [tag officiel v1.6.17](https://github.com/itsmng/itsm-ng/tree/v1.6.17), commit
`58a7a9fd7a5b2ef230c9c46ea2d846381c8e3f93`.

Ce lot ajoute la recherche historique et une aide à la décision dans le rapport
Incidents. Il ne remplace pas la source des alertes/tickets de la chronologie :
la lecture actuelle de ces événements reste explicitement un snapshot simulé.
Les deux disponibilités et leurs provenances doivent être distinguées.

## Les quatre US

| US | Besoin et critères d'acceptation | Validation humaine |
| --- | --- | --- |
| ITSM-01 | Lire l'historique par l'API 1.6.17 avec les credentials ITSM individuels, les restrictions natives et la règle `incidents.tickets`. Pagination, réponses incomplètes, erreurs, délais et changements de droits testés ; aucun token publié, aucune écriture de ticket | Le responsable ITSM confirme la version, le compte lecteur, ses entités/profils et un ticket connu autorisé/interdit dans le SI |
| ITSM-02 | Retenir au plus trois précédents antérieurs à la fenêtre étudiée ; montrer ressemblances, différences et sources. Distinguer solution acceptée, tentative refusée et simple clôture. L'absence de résultat n'est pas une preuve d'absence | Le référent incidents vérifie la pertinence des précédents et les limites du classement sur des cas représentatifs |
| ITSM-03 | Proposer une solution consignée et acceptée dans un précédent comparable, avec conditions, sources et confiance qualitative justifiée. S'abstenir si les solutions comparables divergent ou si aucune n'est étayée | Le référent métier valide l'utilité, les précautions d'application et le niveau de confiance affiché |
| ITSM-04 | Intégrer précédents et solution dans le rapport existant, avant les vérifications. Même réponse serveur dans NiceGUI, HTTP et Open WebUI ; sources et export conservés, refus sans fuite ; CI et Allure | L'utilisateur relit le rapport, ouvre une source autorisée et vérifie sa compréhension dans NiceGUI et Open WebUI |

**Bilan du laboratoire au 10 novembre : les quatre US sont implémentées et
vérifiées sur le corpus fictif.** Après comparaison guidée des deux rendus et
explication de l'hypothèse, l'utilisateur a donné son accord pour poursuivre.
Cet accord porte sur les scénarios discutés du PoC, sans constituer une
réception du SI-RIE réel. Les tickets sources ont ensuite été ouverts avec deux
identités natives dans un navigateur de qualification isolé ; les restrictions
ont été vérifiées. La confiance du navigateur habituel et les profils du SI
cible restent à réceptionner.

**Réserve ultérieure :** l'audit des deux UC et des ACL a ensuite confirmé trois
défauts hors des cas nominaux ci-dessus. Les corrections et leurs régressions
ont ensuite été réalisées, puis vérifiées sur Ubuntu et dans les parcours
Albert identifiés ci-dessous. Voir
[AUD-01 à AUD-03](14-si-rie-acceptance.md#points-ouverts-après-laudit-du-10-novembre-2026).
L'accord de poursuite et les essais réussis sont conservés avec leur portée ;
ils ne signifient pas que tous les scénarios sont exempts de défauts.

État du lot au 10 novembre : les 52 régressions Incidents spécifiques et les
cinq contrôles opérateur réussissent, intégrés aux suites CI/Allure. Les suites
complètes après les corrections AUD-01 à AUD-03 comptent **1 151 réussites et un
test Linux ignoré sur Mac** : 480 source/opérateur/ACL/client/CI, 163 PDL et
508 Incidents. Les résultats de chaque campagne antérieure restent attachés à sa version.

Le correctif `runtime-f94c456528fe0e67adbc` a passé sur Ubuntu 24.04 les douze
contrôles techniques, les lectures natives des deux identités ITSM et trois
parcours frais Albert : NiceGUI Incidents, Open WebUI Incidents, Open WebUI PDL.
Un essai de rédaction/relecture réel sur un contexte fictif adapté à
`GigabitEthernet0/1` est conservé séparément ; il ne vaut pas collecte sur une
autre interface du SI. Voir [les preuves et leur portée](14-si-rie-acceptance.md#points-ouverts-après-laudit-du-10-novembre-2026).

Un serveur issu de l'archive officielle 1.6.17 fonctionne dans la VM Ubuntu
24.04 ARM64, avec PHP 8.2, MariaDB 10.11 et des données fictives. Le connecteur
le consulte en HTTPS avec validation du certificat par une CA de laboratoire ;
le bundle conserve aussi les CA publiques nécessaires à Albert. Deux identités
ITSM distinctes sont liées aux sujets LDAP d'Alice et de Bruno. Le laboratoire
ne qualifie pas les certificats ni les profils du SI réel.

Les essais natifs antérieurs ont vérifié la restriction d'entité, les suivis
privés, les solutions acceptées et la désactivation/réactivation du lecteur.
Les quatre tickets, quatre solutions et huit suivis restent inchangés par
empreinte. L'essai initial PHP 7.4 a échoué sur une syntaxe de l'archive officielle :
son échec reste conservé, malgré le prérequis ancien indiqué dans composer.json.
Les reçus distinguent les versions et environnements de chaque essai.

**Une lecture sans génération ne valide pas le parcours IA.** Le premier retour
humain favorable portait uniquement sur le rapport calculé sans Albert ;
il ne validait pas le parcours avec génération et relecture. La revue ultérieure
et l'accord de poursuite sont consignés en fin de page.
Les nouvelles épreuves exigent les modèles effectivement retournés, les appels
réels et, pour Incidents, les tokens de rédaction/relecture et un verdict valide.
Un modèle seulement configuré, une réponse simulée ou un résultat partiel ne
suffit pas. Les résultats de ce lot restent distincts de la recette clôturée
les 9–10 novembre et de la réception du SI-RIE réel.

## Preuves de génération réelle du laboratoire

La campagne du 10 novembre utilise l'API Albert officielle, avec le modèle
**effectivement retourné** `gpt-oss-120b`. PDL a exécuté l'interprétation du
dialogue et le planner dans NiceGUI et Open WebUI. Incidents a rédigé puis
relu le diagnostic en tenant compte des précédents obtenus par l'API ITSM-NG
native d'Ubuntu, avec la solution publiée seulement après relecture valide.

Avant le correctif d’attribution, le reçu NiceGUI Incidents conserve les appels
de rédaction, correction et relecture, avec respectivement **3 886, 3 843 et 4 902 tokens totaux** retournés.
Celui d'Open WebUI conserve **3 838 et 4 978 tokens** pour rédaction et relecture.
Ces valeurs sont des usages fournisseur observés, pas une mesure de performance
ni une consommation garantie pour les prochaines demandes.

Les essais intermédiaires ayant échoué restent dans un rapport Allure séparé :
format fournisseur invalide, contraintes de synthèse et clarification PDL superflue.
Les corrections traitent les références vérifiées, la prudence grammaticale et
les rôles d'identité approuvés ; aucune réponse métier de démonstration n'est
insérée pour remplacer Albert. Les modèles et les règles d'autorisation restent
contrôlés au prochain appel. La campagne initiale reste accessible depuis
**Qualification** dans NiceGUI, sous `itsm-albert-reel-20261110`.

Le corpus reste fictif. Les alertes/tickets de la chronologie gardent leur
provenance snapshot ; seuls les précédents de ce laboratoire viennent du serveur
ITSM-NG natif. La pertinence sur les vrais tickets, les profils et données du SI,
la charge, l'endurance des nouvelles images et leur compatibilité 26.04 restent
à qualifier. Le premier retour humain sans IA ne vaut pas acceptation de cette
nouvelle chaîne de génération.

## Revue des attributions de sources — 10 novembre

La relecture humaine des deux rapports générés a relevé une mauvaise attribution :
une hypothèse de routage renvoyait à un ticket ne décrivant ni routage ni
configuration. La relecture IA précédente l'avait acceptée. Ce constat reste une
preuve d'écart ; il n'est pas transformé rétroactivement en réussite.

Le correctif transmet à Albert les descriptions des précédents et les solutions,
y compris celles refusées. La relecture reçoit chaque argument avec les contenus
de ses seules sources citées. Une information présente dans une autre fiche ne
justifie pas l'attribution au ticket. Les logs transmis à cette relecture restent
agrégés ; leurs lignes brutes ne sont pas ajoutées au contexte. Le budget de
contexte existant reste appliqué. Une clôture administrative doit désigner le
ticket ou le dossier, séparément de la résolution de l'alerte.

Dix régressions hors réseau couvrent ces liens, les refus de synthèse et la
formulation. Cinq contrôles avec le vrai relecteur Albert réussissent : rejet
de deux mauvaises attributions, dont la formulation prudente du rapport initial,
et d'une tentative refusée présentée comme une réussite ; acceptation d'une
hypothèse documentaire correctement citée et d'une solution historique conditionnelle.
**Les textes de ces cinq contrôles sont injectés pour tester le relecteur** :
ils ne valent pas génération d'un diagnostic ni nouvelle lecture des sources.
Les deux parcours complets NiceGUI et Open WebUI ont été rejoués séparément,
avec collecte fraîche, rédaction et relecture Albert réelles.
Les modèles retournés sont `gpt-oss-120b` : NiceGUI conserve 4 045 tokens de
rédaction et 6 416 de relecture ; Open WebUI conserve respectivement 4 091 et
6 790 tokens. Ces consommations datées ne constituent pas un engagement de capacité.

Les reçus du correctif se trouvent sous `.runtime/ci/itsm-citations-20261110/`.
La campagne Allure `itsm-citations-20261110` conserve les résultats de cette version ;
le rapport précédent reste attaché à sa version. Les nouveaux rapports générés
étaient alors encore à relire ; la suite de cette revue est consignée en fin
de page. La relecture IA ne garantit pas
l'absence de toute erreur sémantique sur d'autres cas.

Pour rejouer les sept contrôles du relecteur dans le laboratoire,
un opérateur choisit **son propre diagnostic d'essai synthétique** obtenu sur
l'API ITSM-NG native. Le nom du conteneur Incidents se lit dans `manage.py status`.
Sur Ubuntu, depuis les sources du correctif, remplacer les deux valeurs ci-dessous :

```bash
# Désigne le conteneur Incidents de recette à partir de son nom affiché par manage.py status.
export RIE_INCIDENTS_CONTAINER=INCIDENTS_CONTAINER_NAME
# Désigne dans le conteneur le diagnostic fictif déjà produit dont les citations seront testées.
export RIE_TEST_REPORT=/data/incidents/investigations/TEST_DIAGNOSIS_ID.json
# Teste les citations du diagnostic choisi avec au plus sept appels réels Albert ; ne refait pas la collecte des sources.
sudo docker exec -i "$RIE_INCIDENTS_CONTAINER" python - --source-report "$RIE_TEST_REPORT" --output /tmp/recette-citations.json --call-albert < deploy/ubuntu/tests/qualification_citations.py
```

Cette commande facultative fait au plus sept appels de relecture, aucun appel
réel de rédaction. Elle exige le corpus fictif natif, l'URL Albert officielle
et l'option explicite `--call-albert`. Elle ne modifie ni règle, ni ticket,
ni limite d'admission. Le résultat attendu est `passed: true` pour les sept cas,
avec refus des quatre affirmations mal étayées. Un échec reste un échec de
qualification. Choisir un nouveau nom de sortie pour chaque exécution.

## Nouvelle recette humaine : rejet puis correction bornée

La génération NiceGUI du 10 novembre à 12:21 UTC n'a pas été validée : elle
attribuait au ticket une « cause probable », alors que ce ticket disait « cause
à confirmer ». Le relecteur l'a rejetée et la solution a été retirée. Ce résultat
partiel est conservé ; il ne prouve pas une panne des sources.

Une première version partageait une seule correction entre les contraintes
locales et la relecture : trois générations sur quatre ont réussi, et la
quatrième a été refusée après deux défauts successifs. Ces résultats restent
conservés. Le correctif final sépare une correction locale et une factuelle,
avec motifs cumulés et au plus cinq appels pour la synthèse. Le motif du rejet accompagne une nouvelle rédaction
depuis les mêmes preuves, puis une nouvelle relecture. Un nouveau refus après consommation du budget correspondant maintient
le résultat partiel. Les budgets de contexte, de durée et d'admission restent appliqués.
L'affichage des appels précise l'essai ; le motif d'un refus est lisible dans
le rapport. Les routes de bascule explicites gardent leurs propres bornes ;
aucune bascule de modèle n'est activée par ce correctif.

Six cas de régression vérifient la récupération, deux refus successifs, les
budgets indépendants des contraintes et de la relecture, une correction locale
nécessaire après un refus et une panne IA pendant la correction. Une régression supplémentaire refuse des traces non vérifiées
dans un appel natif échoué. Dans le laboratoire, le brouillon effectivement
rejeté a aussi été rejoué : la première rédaction est reprise pour ce test,
mais le refus, la rédaction corrective et la seconde relecture sont demandés au
vrai Albert. Les trois appels réussissent et la seconde version est validée.
Ce rejeu contrôlé est distinct des diagnostics entièrement générés dans les
interfaces ; il ne vaut pas nouvelle collecte du SI.

Les preuves de cette phase sont sous `.runtime/ci/itsm-citations-20261110/`,
avec les préfixes `repair-`, `repair2-`, `gemma-`, `hybrid-` et `hybrid2-` qui
distinguent les versions successives. La version finale est publiée dans la campagne
`itsm-relecture-20261110`. La recette humaine du nouveau comportement est
consignée plus bas, séparément de la réception du SI réel qui reste à faire.

## Choix du modèle de rédaction Incidents

Avec GPT-OSS, la série à budget partagé puis la série à budgets séparés ont
chacune produit trois diagnostics validés sur quatre ; le résultat partiel
supplémentaire reste conservé. Le budget n'a pas été augmenté au-delà d'une
correction locale et d'une factuelle. Les refus ne sont pas transformés en
réussites par suppression d'un contrôle.

Une comparaison sur les mêmes preuves fictives et deux formulations a ensuite
réussi avec Gemma 4 et Mistral Medium 3.5, en rédaction et relecture réelles sur
Albert. Ce sont des échantillons de laboratoire, sans preuve de supériorité
générale. Gemma 4 est qualifié en priorité, conformément à la préférence de
l'utilisateur et à son usage local, en utilisant **l'API Albert**. Ses cinq
contrôles positifs/négatifs du relecteur ont réussi sans relâcher les critères.

Après les corrections et la [qualification du 12 novembre](19-openwebui-corrections.md),
les nouvelles instances préparent `ALBERT_MODEL=gemma-4-31b-it` pour le dialogue
et la planification PDL, ainsi que `OBS_MODEL_GPTOSS=gemma-4-31b-it` pour la
rédaction Incidents. La relecture reste sur `OBS_REVIEW_MODEL=gpt-oss-120b`, et le
dialogue Incidents sur `OBS_DIALOGUE_MODEL=gpt-oss-120b`. Les essais du 10 novembre
ci-dessus conservent leurs propres modèles et leurs résultats. Le nom technique
du preset et les URL MCP sont conservés ; le rapport affiche le modèle retourné.
Les anciennes instances exigent un changement explicite de configuration.
Il n'y a pas de bascule silencieuse vers Mistral ou vers une IA locale.

L'accès Gemma sur la clé de laboratoire est testé réellement ; cela ne garantit
ni les habilitations ni les quotas de la future clé SI-RIE. Les essais complets
des interfaces et leurs versions sont conservés dans les reçus de la campagne.

### Rédaction et relecture séparées

Gemma 4 seul a passé les quatre parcours frais des interfaces et les premiers
contrôles de citations, mais sa relecture a accepté à tort le brouillon de la
recette humaine initiale. Ce contre-exemple reste conservé. Le profil retient
donc Gemma 4 pour rédiger et GPT-OSS pour relire, tous deux via Albert.
L'interprétation et PDL conservent leur modèle précédent.

Un autre rejeu a montré un refus trop sévère du relecteur : une fiche historique
recommandait une vérification technique, mais son origine historique a été
interprétée comme une absence de recommandation. La règle précise désormais
que les sources citées peuvent se compléter et qu'un retour d'expérience peut
étayer une vérification en lecture seule si son texte la recommande. Cela ne
transforme pas son ancienne cause en cause actuelle. Un septième contrôle réel
couvre ce cas positif, en plus des affirmations incorrectes qui doivent rester
refusées. Les résultats de chaque version sont distingués dans les reçus.

Sur la version finale du moteur, les sept contrôles du relecteur réussissent.
Le rejeu contrôlé du brouillon humain est d'abord refusé par GPT-OSS, puis
corrigé par Gemma et validé lors d'une seconde relecture. Ce test ne recollecte
pas les sources. Deux diagnostics frais ont ensuite réussi : NiceGUI à
13:31 UTC et Open WebUI à 13:33 UTC le 10 novembre, avec collecte native,
rédaction Gemma et relecture GPT-OSS effectivement retournées par Albert.
NiceGUI a consommé les deux corrections autorisées ; Open WebUI a réussi au
premier essai. Les durées observées des diagnostics sont respectivement
28,571 s et 13,151 s, sans engagement de performance.

Ces résultats ne démontrent ni une supériorité générale de Gemma, ni l'absence
de futurs refus ou erreurs de relecture. Les essais intermédiaires en échec
et le refus humain de 12:21 UTC restent consultables dans la campagne distincte
`itsm-relecture-essais-20261110`. La revue des nouveaux rapports est consignée
plus bas ; la réception sur les tickets réels du SI-RIE reste à réaliser.

### Ouverture des tickets sources dans le laboratoire

Le relais SSH local du port 18469 donne accès au serveur ITSM-NG fictif d'Ubuntu.
Huit contrôles navigateur réussissent : vérification TLS stricte de la CA, du
nom d'hôte et de la clé attendue ; rejet d'une mauvaise empreinte ; connexion
native exigée pour un anonyme ; lecture des tickets 102 et 101 avec l'identité
liée à Alice ; isolation des sessions ; lecture du ticket commun 102 et refus
du ticket protégé 101 avec l'identité liée à Bruno. Aucun ticket, solution ou
suivi n'est modifié par cette recette.

Chrome utilise un profil isolé avec une exception limitée à l'empreinte de clé
publique du serveur de laboratoire, selon le
[mécanisme Chromium](https://chromium.googlesource.com/chromium/src/+/HEAD/services/network/ignore_errors_cert_verifier.h).
La vérification stricte de la chaîne et du nom est effectuée séparément avant
navigation. Aucun certificat n'est ajouté au trousseau du Mac, aucune clé
privée n'est transférée, et les credentials fictifs ne passent que par la
mémoire et les formulaires. Ce résultat ne qualifie pas la confiance du
navigateur habituel du poste ni celle du futur poste SI-RIE.

Le reçu final est `native-source-browser-c3/browser-results.json` sous
`.runtime/ci/itsm-citations-20261110/`. Les deux premiers pilotes restent en
échec dans leurs propres reçus : ils supposaient des noms de champs de connexion
fixes, puis un champ de titre éditable alors que le profil est lecteur. Le
pilote corrigé utilise les identifiants HTML stables et le titre effectivement
affiché en lecture seule ; aucune permission native n'a été élargie.

Sur le Mac de recette, le lanceur privé
`.runtime/recette-manuelle-20261109/ouvrir-itsm.command` vérifie d'abord TLS puis
ouvre un profil Chrome réservé au laboratoire. L'option `--check` ne lance pas
Chrome. Ces outils locaux et leurs certificats de laboratoire ne font pas
partie de l'installation du SI réel ; utiliser sur les postes cibles les URL,
certificats et identités approuvés par le SI.

## Fiche de raccordement ITSM à faire compléter

Avant de transposer les commandes de la sandbox, faire compléter cette fiche
par le responsable ITSM et les équipes réseau/sécurité. Les fichiers
`si-connection/private/` et les comptes Alice/Bruno sont propres à l’exercice ;
le déployeur n’a pas à les recréer dans le SI réel.

| Élément attendu | Fourniture et contrôle |
| --- | --- |
| Version et API | Version réellement déployée confirmée ; contrat ciblé ITSM-NG **1.6.17**, API activée. Une autre version exige une recette de compatibilité. |
| Adresses | `api_url` HTTPS terminant par `/apirest.php` et `public_url` HTTPS de l’interface native, sans secret dans l’URL. Les liens des tickets doivent être accessibles au navigateur de l’utilisateur. |
| Réseau et confiance | DNS, routage, port, filtres API et CA approuvés depuis les conteneurs Ubuntu, avec leur adresse source effective. Le bundle `SOURCE_CA_FILE` doit permettre la confiance ITSM et les autres sources, dont Albert s’il est joint par ce même bundle. |
| Authentification | App-Token du client si requis ; pour chaque lecteur, identifiant LDAP réel, `itsm_user_id` numérique et `user_token` individuel. Un mot de passe LDAP ou un compte Open WebUI ne remplace pas le token ITSM. |
| Droits natifs | Profil et entités actifs autorisant les tickets requis, leurs solutions et les suivis autorisés. L’administrateur des ACL du PoC n’obtient pas automatiquement des droits d’administration dans ITSM. |
| Dates et provenance | Fuseau de stockage des dates natives sans offset, période de référence et tickets connus. `synthetic: true` pour la sandbox ; `false` pour des données réelles confirmées. |
| Alertes courantes | Export compatible et responsable/fréquence d’actualisation du snapshot courant, distinct de la recherche des précédents par API. |
| Réception | Deux utilisateurs de droits natifs différents, ticket autorisé et ticket interdit connus, suivi privé, solution acceptée et cas sans précédent, avec résultats attendus fournis par le métier. |

Les appels de session `initSession`, `getFullSession`, `killSession` et les
lectures `listSearchOptions/Ticket`, `search/Ticket`, `Ticket/{id}`,
`Ticket/{id}/ITILSolution`, `Ticket/{id}/ITILFollowup`, puis les sous-objets
`ITILSolution/{id}` et `ITILFollowup/{id}` doivent être accessibles selon les
droits natifs. La recherche contrôle les champs natifs attendus ; un contrat
incompatible est refusé. Elle ne crée ni ne modifie de ticket.

Un ticket connu doit être résolu ou clos, antérieur au début de l’incident
analysé et appartenir à la fenêtre historique prise en charge, au plus 730 jours.
La lecture est bornée aux vingt premiers candidats ; les conditions et limites
du [classement](#classement-et-confiance) restent applicables. Fournir avec le
métier des mots-clés et symptômes représentatifs : une recherche lexicale ne
garantit pas de retrouver toutes les formulations ou tous les anciens tickets.

Si l’accès passe par un proxy obligatoire, un SSO ou du mTLS, faire qualifier
le mécanisme avant de déclarer le raccordement terminé. Plusieurs clients
désactivent l’héritage des variables proxy : définir `HTTPS_PROXY` seul ne
suffit pas. Voir les contrôles S03/S04 de la [matrice SI-RIE](14-si-rie-acceptance.md#matrice-de-vérification-dans-le-si-rie).

## Raccordement sur Ubuntu

Sur le serveur Ubuntu, opérateur avec `sudo`, depuis les nouvelles sources :

1. Sauvegarder l'instance et construire les nouvelles images avec `manage.py build`.
2. Faire confirmer les URL HTTPS, CA, fuseau de stockage des dates ITSM et
   l'activation de l'API par le responsable ITSM. Le connecteur ne déduit pas
   un nom d'hôte et ne désactive pas TLS. `SOURCE_CA_FILE` reste le bundle
   administré pour les sources, qui doit aussi inclure la CA ITSM si nécessaire.

   Les dates natives sans offset sont interprétées dans ce fuseau, transmis
   jusqu'au classement ; résolutions et dates des solutions y sont normalisées
   en UTC. Un offset explicite est respecté. Une heure locale ambiguë ou
   inexistante lors d'un changement d'heure est refusée ; le lecteur ne la
   devine pas. Le corpus de fixture conserve son fuseau connu Europe/Paris.
3. Copier le modèle public et remplacer les URL :

```bash
# Copie le modèle ITSM dans l'instance avec des droits 0600 ; son contenu reste à adapter avant activation.
sudo install -m 600 deploy/ubuntu/examples/itsm-history.json "$RIE_INSTANCE/config/itsm-history-input.json"
# Ouvre le modèle privé ITSM dans votre éditeur pour renseigner les URL HTTPS et le fuseau.
sudoedit "$RIE_INSTANCE/config/itsm-history-input.json"
```

4. Pour chaque utilisateur, faire fournir **son propre** `user_token` ITSM et
   son identifiant numérique ITSM dans un fichier privé extérieur au dépôt.
   Ce fichier d'entrée contient seulement `itsm_user_id` (entier) et
   `user_token` (chaîne). L'opérateur le provisionne sans afficher son contenu :

```bash
# Lie le sujet LDAP stable de cet utilisateur à ses seuls credentials ITSM fournis dans le fichier privé.
sudo python3 deploy/ubuntu/manage.py itsm-user "$RIE_INSTANCE" --username LDAP_USERNAME --input /PRIVATE/PATH/itsm-identity.json
```

La liaison utilise le sujet LDAP stable, pas un mail fourni par le client.
À chaque recherche, une nouvelle session ITSM est créée puis fermée ; son
`glpiID` doit correspondre à l'identifiant provisionné. Aucun compte ITSM
partagé ne remplace les utilisateurs. Si un App-Token est imposé par ITSM,
[l’ajouter depuis son fichier privé](#ajouter-lapp-token-fourni-par-itsm). Les tokens ne doivent
jamais apparaître dans les URL, commandes, logs, captures ou paquets publics.

5. Activer la configuration, valider puis appliquer :

```bash
# Enregistre le raccordement de l'historique ITSM ; l'étape up appliquera ensuite cette configuration.
sudo python3 deploy/ubuntu/manage.py configure-itsm-history "$RIE_INSTANCE" --input "$RIE_INSTANCE/config/itsm-history-input.json"
# Contrôle les fichiers et paramètres privés avant application ; ne démarre rien et n'appelle pas Albert.
sudo python3 deploy/ubuntu/manage.py validate "$RIE_INSTANCE"
# Démarre ou relance les services de cette instance et recharge sa configuration en conservant ses données.
sudo python3 deploy/ubuntu/manage.py up "$RIE_INSTANCE"
```

La configuration se trouve dans `config/itsm-history.json`, les secrets dans
`secrets/itsm-users.json`. La sauvegarde privée de l'instance les conserve.
Lorsque l'historique est activé, `/readiness` et `manage.py check` sondent aussi
cette source avec l'identité courante ; son indisponibilité ne devient pas un
succès silencieux. Cette sonde ne remplace pas le contrôle métier d'un ticket connu.
Elle peut réussir sur une recherche complète sans résultat : elle ne vérifie
alors ni le contenu d’un ticket ni ses solutions ou suivis. Conserver une preuve
positive sur un ticket réellement autorisé et les refus attendus d’un autre
profil ; un simple `ok` ne clôture pas cette réception.
Le compte LDAP de l'opérateur qui lance `check` doit donc posséder sa propre
liaison ITSM, comme chaque lecteur métier. La commande utilise par défaut
`alice.martin` dans la démonstration ; `--username LDAP_USERNAME` permet de
choisir l'opérateur du SI. Ne pas lui attribuer le token d'un autre utilisateur
pour faire réussir le contrôle. Une liaison manquante doit être provisionnée
par la même commande `itsm-user`, puis appliquée par `up` avant le nouveau contrôle.
Un changement ou retrait de token natif prend effet à la prochaine session ITSM.
Le retrait d'un utilisateur se fait dans le magasin privé puis par `up` pour
appliquer la copie ; révoquer aussi son token côté ITSM lorsqu'il doit cesser
d'être utilisable. Un jeton applicatif restauré ou un secret ITSM restauré
ne doit pas réactiver une habilitation révoquée après la sauvegarde.

Le mode de démonstration s'active avec une configuration privée minimale :
`{"version":1,"target_version":"1.6.17","provider":"fixture"}`.
Il est refusé hors `demo`. Les cinq tickets fictifs incluent deux solutions
concordantes, une autre famille de panne, une tentative refusée et un dossier
exclu. Ils ne constituent pas un test contre un serveur ITSM réel.

### Ajouter l’App-Token fourni par ITSM

Complément de l’[étape 9 du guide](../README.md#9-raccorder-les-précédents-itsm-ng),
**uniquement si le serveur ITSM exige un App-Token**. Sur Ubuntu, conserver les
variables `RIE_INSTANCE` et `RIE_SI` définies dans ce guide. Le fichier privé
`$RIE_SI/private/itsm-app.token` doit avoir été fourni par le responsable ITSM.

La commande ajoute ce secret au stockage privé existant sans afficher sa
valeur et préserve les liaisons individuelles enregistrées par `itsm-user` :

```bash
# Ajoute l'App-Token depuis le fichier privé, sans afficher sa valeur ni la passer en argument.
sudo python3 -c 'import json,pathlib,sys; p=pathlib.Path(sys.argv[1]); d=json.loads(p.read_text()); d["app_token"]=pathlib.Path(sys.argv[2]).read_text().strip(); p.write_text(json.dumps(d,indent=2)+"\n"); p.chmod(0o600)' "$RIE_INSTANCE/secrets/itsm-users.json" "$RIE_SI/private/itsm-app.token"
```

**Attendu :** commande terminée sans erreur ni valeur affichée. Reprendre
[Valider et appliquer le raccordement ITSM](../README.md#valider-et-appliquer-le-raccordement-itsm).

## Droits et absence de contournement

Les trois visibilités existantes sont conservées : interdit à tous,
internes uniquement, internes et externes authentifiés/classés. Une règle absente
refuse l'accès. Elles s'appliquent avant lecture de la configuration ou de l'API.
Un diagnostic Incidents complet exige toujours les quatre droits de sources
documentés dans [ACL](06-acl.md) ; ouvrir les tickets ne rend pas les autres
sources accessibles.

Les collections de solutions et suivis demandent des identifiants seulement.
Le contenu de chaque sous-objet est ensuite lu par son endpoint individuel,
afin de faire vérifier ses droits natifs par ITSM, notamment pour les suivis
privés. Un sous-objet refusé est exclu des preuves, du classement et du rapport.
Un refus du ticket ou une recherche incomplète ne produit aucune recommandation
fondée sur des données partielles cachées. Les liens ouvrent ITSM avec les
droits de la session du lecteur ; ils ne contiennent aucun token du connecteur.

## Classement et confiance

La recherche lexicale utilise les signaux déjà observés par le diagnostic,
puis un classement déterministe par mots communs et présence du même identifiant
d'équipement. Le nom d'un équipement dans le texte ne vaut pas une relation
CMDB vérifiée. Les tickets doivent être résolus/clôturés, avec une date de
résolution antérieure au début du diagnostic et vieille de moins de deux ans.
Au plus vingt candidats récents sont examinés et trois retenus ; la troncature
reste visible. Ce PoC ne prétend pas retrouver tous les synonymes ou toutes les
variantes d'une solution, ni faire une recherche sémantique générale.

Une solution doit être explicitement acceptée (`ITILSolution.status = 3`).
La clôture administrative seule ne suffit pas. Deux textes différents restent
différents : le moteur n'invente pas une équivalence sémantique.

- **Élevée** : plusieurs tickets concordants pour la même solution, équipement
  textuellement retrouvé, plusieurs signaux communs, diagnostic courant étayé
  et recherche non tronquée.
- **Modérée** : solution acceptée et comparable avec observations courantes
  étayées, mais preuves historiques limitées.
- **Faible** : observations insuffisantes ou recherche tronquée ; les limites
  sont expliquées. Des solutions comparables contradictoires entraînent une
  abstention, pas un choix arbitraire.

Ce niveau ne constitue ni une probabilité statistique de succès ni une garantie
de causalité. Il est distinct de la confiance dans le diagnostic. La solution
est le texte de la procédure consignée, à valider par un opérateur ; aucune
commande, modification d'équipement ou écriture ITSM n'est exécutée.
Les précédents et la proposition calculée sont transmis à la génération et à
la relecture Albert lorsqu'un diagnostic IA est demandé. Une relecture échouée
retire la solution proposée de la conclusion qualifiée. La sélection et son
niveau de confiance restent calculés par le serveur ; ils ne sont pas un
pourcentage inventé par le modèle.

## Recette et revue des rendus

Sur la démonstration, dans NiceGUI, ouvrir Incidents avec Alice et choisir
**Diagnostiquer avec Albert API**. Ce bouton utilise `OBS-RTR-01`, alias `R1`,
le 28 septembre 2026 de 10:00 à 12:00, heure de Paris, site de Rennes.
Attendre au moins 60 secondes après la réponse, puis, avec la même identité
autorisée, **nouveau fil Open WebUI, modèle SI-RIE — INCIDENTS** :

```text
Diagnostique OBS-RTR-01 le 28 septembre 2026 de 10:00 à 12:00, heure de Paris, sur le site de Rennes. Recherche les précédents ITSM-NG disponibles et présente la solution la mieux étayée, ses conditions et les niveaux de confiance du diagnostic et de la solution. Cite les sources et distingue résolution technique et clôture du ticket.
```

Comparer les faits et citations, pas une formulation littéralement identique.
Ce texte concerne le corpus fictif disponible dans `demo` ou la sandbox préparée.
Pour le SI réel, utiliser le [texte Incidents complété de la fiche](26-si-connection-sheet.md#incidents--diagnostic-précédents-et-suivi).
L’équipe de qualification vérifie les appels réels de rédaction/relecture,
les modèles retournés, les tokens et les erreurs. Le contrôle technique
« sans génération IA » ne valide pas cette chaîne.

Le validateur métier examine la pertinence de l'hypothèse, l'utilité de la
solution et ses conditions, ainsi que la compréhension des deux niveaux de
confiance. Le rapport détaillé reste destiné aux hyperviseurs : mesures,
chronologie, documents et précédents leur permettent de retrouver les
informations utiles à leur investigation.

### Comparaison guidée des deux rendus

Pour la recette, préparer une fiche comparative séparée du rapport
d'exploitation. Identifier les deux demandes, leur date de lecture, leur
périmètre et leur version. Comparer d'abord les données et les sources :
des observations différentes entre deux lectures ne constituent pas
automatiquement un défaut de génération. Présenter ensuite uniquement les
différences de formulation, de citations, de conclusion ou de précautions,
avec les passages et les preuves nécessaires à leur examen. Une égalité
technique ne démontre pas à elle seule une équivalence sémantique.

Les deux rapports fournis le 10 novembre (Open WebUI, lecture à 13:33 UTC ;
NiceGUI, lecture à 13:48 UTC) ont été comparés depuis leurs résultats structurés
conservés dans Ubuntu, sans nouvelle génération ni nouvelle collecte.
Les treize contrôles de comparaison réussissent. Les dix-sept recoupements
techniques précédents de ces deux rapports restent conservés séparément.

| Point comparé | Open WebUI | NiceGUI | Résultat de la revue ciblée |
| --- | --- | --- | --- |
| Données et contexte | OBS-RTR-01, 28 septembre 10 h–12 h, cinq séries complètes, 120 logs | Identiques, y compris valeurs et couverture | Aucune différence des données comparées |
| Chronologie, documents et précédents | Résolution à 11 h 30, clôture du ticket à 12 h 15 ; solutions acceptées des tickets 101 et 102, tentative 104 non retenue | Identiques | Aucun écart sur ces preuves |
| Situation | Pertes et « hausse de latence », événement résolu | Pertes et « augmentation de la latence », événement résolu | Formulations de même sens |
| Hypothèse | Le transfert « aurait pu » saturer la liaison ; S6 et S8 | Même hypothèse prudente ; S6, S8, S11 et S13 | Deux précédents ITSM supplémentaires cités ; aucune cause présentée comme établie |
| Solution, conditions et vérification conseillée | Limitation du débit après confirmation et accord ; comparaison de la configuration et validation opérateur ; vérification citée S9 | Identiques | Aucune différence de conseil ou de précaution |
| Génération | Un essai validé | Trois essais, soit deux corrections, dernier contrôle validé | Différence d'exécution conservée ; les deux diagnostics finaux sont `ok` |

Les reçus sont `human-reports-pair-comparison.json` et
`human-latest-report-crosschecks.json`, sous
`.runtime/ci/itsm-citations-20261110/`. Le premier conserve les résultats de
comparaison et les seuls passages générés utiles à la revue, sans export des
rapports bruts. Les deux modèles effectivement retournés sont Gemma 4 pour la
rédaction et GPT-OSS pour la relecture, via Albert.

La revue ciblée ne relève pas de divergence métier sur les points ci-dessus.
Elle ne vaut pas validation humaine ni preuve de pertinence sur les incidents
réels. Le retour utilisateur confirme l'utilité du détail pour les hyperviseurs ;
la difficulté signalée concerne la tâche de comparaison, sans défaut de densité
du rapport retenu. Après explication du mécanisme envisagé et de ses limites,
l'utilisateur a donné son accord pour poursuivre. La revue reste limitée aux
deux scénarios fictifs discutés : elle ne valide pas toutes les formulations
futures ni la pertinence du conseil sur les données réelles du SI.

### Contrôles d'accès et réception dans le SI

Dans Administration ACL, restreindre puis rétablir les tickets avec les trois
visibilités, avec Alice et un externe ; conserver les autres règles initiales.
Vérifier le refus sans preuve si une source requise est interdite. Les tickets
fictifs internes restent exclus pour l'externe même lorsque la règle générale
autorise les utilisateurs authentifiés. Restaurer les règles à la fin.

Dans le SI, faire en plus ouvrir un ticket et un suivi privé avec deux comptes
ITSM de droits différents, comparer aux écrans natifs et tester une révocation.
Un succès du double de protocole ou du mode fictif ne coche pas cette vérification.

| Date, version, environnement | US | Résultat humain et observations | Validateur |
| --- | --- | --- | --- |
| 10 novembre 2026 ; Ubuntu 24.04 ARM64 ; moteur `runtime-99870441b7e713e4c810` ; rédaction Gemma et relecture GPT-OSS via Albert | ITSM-02/03 et rendu ITSM-04 | Accord de poursuivre après comparaison des deux rapports, explication de l'hypothèse, du conseil conditionnel et des précautions ; détail du rapport conservé pour les hyperviseurs. Portée : deux cas fictifs discutés | Utilisateur, dans la conversation de recette |
| À planifier dans le SI-RIE réel | ITSM-01 à ITSM-04 | Version/API, entités et profils, tickets connus, certificats du poste, pertinence métier et charge réelles non réceptionnés | Responsables SI et métier à nommer |
