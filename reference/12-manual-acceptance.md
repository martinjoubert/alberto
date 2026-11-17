# Fiche de validation manuelle — PDL, Incidents et ACL

> Complément de référence — le [guide principal](../README.md) reste le seul parcours de déploiement à exécuter.

Cette fiche accompagne **la validation humaine du déploiement à venir**. Elle
ne reprend pas les résultats automatiques. Utiliser cette grille avec le
[guide principal](../README.md) pour consigner les contrôles des étapes 11 et 12.
Elle ne constitue pas une seconde installation. Pour qualifier un déploiement
neuf, partir d'une instance neuve sur le serveur prévu.
Pour éprouver aussi les prérequis, employer un Ubuntu neuf. Cocher ensuite les
résultats ici, sans répéter l'installation. Une recette d'utilisation seule
peut reprendre une instance existante, en le notant ; elle ne valide pas une
nouvelle installation. Sur un raccordement externe, faire autoriser les scénarios
et la fenêtre d'interruption par le responsable du SI.

Pour une réception du **SI-RIE réel**, compléter aussi la
[matrice d'intégration](14-si-rie-acceptance.md). La présente fiche courte couvre
le parcours humain ; elle ne remplace pas les contrats de données, la gouvernance
des droits, la charge et les écarts connus à traiter avant cette réception.

Renseigner : date `________` ; opérateur `________` ; version/manifeste
`________` ; machine/architecture `________` ; mode/instance `________` ;
fournisseur IA réellement accessible `________`.

Les textes exacts à envoyer sont dans l’[étape 11](../README.md#11-jouer-les-deux-uc-et-leurs-acl-avec-albert) :
**PDL-1/2** pour la clarification et les deux refus PFS de la sandbox,
**PDL-3/4/5** pour six documents puis deux TACACS puis quatre sans TACACS,
**INC-1/2/3** pour diagnostic, horodatages et clarification,
**ACL-1/2/3** pour les listes de droits et les demandes par identité.
Sur le SI réel, utiliser les [textes préparés de la fiche](26-si-connection-sheet.md#textes-open-webui-à-préparer-pour-le-si-réel)
et leurs attendus réels. Les valeurs chiffrées ci-dessus concernent seulement la sandbox.
Pour chaque essai, conserver le texte envoyé, le modèle, « nouveau fil » ou
« même fil », le titulaire et son résultat ; le mot « fait » seul ne décrit pas l’essai.

Pour chaque ligne, écrire **réussi**, **échoué** ou **non fait**, avec le reçu
ou la raison. Les cellules restent volontairement vides avant la séance.
Ne joindre aucun mot de passe, jeton ou preuve réelle non autorisée.

| Action humaine | Résultat attendu et procédure de référence | Résultat | Reçu ou raison |
| --- | --- | --- | --- |
| 1. Identifier la remise et le parcours choisi | Instance neuve, commit ou manifeste et mode identifiés ; installation terminée selon le [guide principal](../README.md) | | |
| 2. Lire l'état, les logs et le stockage | `ready` ou cause explicite ; logs ciblés et occupation consultables sans arrêter ni purger, selon [Exploitation](07-operations.md#lire-létat-sans-interrompre-le-service) | | |
| 3. Vérifier les trois rôles dans des sessions distinctes | Administrateur, interne lecteur et externe utilisent chacun leur identité ; Alice/Bruno/Eric dans une nouvelle démo. Le lecteur interne conserve ses lectures métier et refuse administration/rapports ; l'externe reste soumis à ses droits. Credentials fictifs récupérables compte par compte ; [ACL](06-acl.md) | | |
| 3 bis. Revoir les mappings dans NiceGUI | Dans **Mappings PDL**, l’administrateur retrouve les revues existantes, choisit les champs découverts, vérifie la proposition et lit les différences ; aucun mapping n’est activé avant confirmation et justification. Un lecteur/externe ne reçoit pas de schéma. Export YAML récupérable ; [parcours](../README.md#8-revoir-et-approuver-les-mappings-pdl) | | |
| 4. Jouer PDL et un suivi dans le même fil | Quatre familles visibles, période fixe et rapprochement de référence retrouvés, clarification et reprise liées au titulaire ; [Demandes exactes PDL-1 à PDL-5](../README.md#pdl--clarification-puis-reprise-du-même-fil) | | |
| 5. Examiner les quatre droits Incidents | Deux index physiques plus ressources métriques/tickets visibles et distincts ; l'interne autorisé lit les sources attendues, l'externe refusé ne voit aucune preuve interdite ; [ACL](06-acl.md#ce-qui-est-protégé) | | |
| 6. Refuser puis rétablir un droit UC2 | Dans la simulation, modifier successivement un droit de source avec les révisions correctes ; refus fermé, puis nouvelle demande réussie après rétablissement, sans changement d'identité ni preuve ancienne ; [Demande ACL-3 et cycle exact](../README.md#acl--lecteurs-refus-et-rétablissement) | | |
| 7. Arrêter puis reprendre explicitement | Même corpus, politique, mappings et jetons ; aucune réinitialisation nécessaire, selon [Arrêt/reprise](07-operations.md#arrêter-et-reprendre-volontairement) | | |
| 8. Restaurer une sauvegarde froide neuve | Nouvelle destination, empreintes vérifiées, données/droits conservés ; archive corrompue refusée si ce cas est joué ; [Sauvegarde/restauration](07-operations.md#sauvegarder-à-froid) | | |
| 9. Qualifier séparément le client IA et les exports | Si Albert disponible : conversations/citations/refus dans Open WebUI, sans boucle automatique ; sinon « génération non qualifiée ». Pages et draw.io exportables, sources éditables, selon [Client](02-configuration.md#configurer-le-client-open-webui-local-qualifi%C3%A9) et [Wiki.js](10-wikijs.md) | | |

Joindre les rapports de la campagne consultés depuis **Qualification** dans
NiceGUI, en identifiant leur version et leur environnement. Le
[parcours Allure](13-test-reports.md) détaille l'import et la consultation ;
les résultats automatiques joints ne remplissent pas cette fiche à la place
de l'opérateur.

Après la séance, rétablir les règles temporaires, révoquer les seuls jetons de
recette créés pour cette séance et arrêter les services si prévu avec
l'exploitant. Ne pas détruire les volumes. Consigner les écarts avant de déclarer
le déploiement accepté ; un cas non fait ne devient pas une réussite implicite.
