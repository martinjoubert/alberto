# Compléments du guide de déploiement

**Le seul parcours à exécuter est le [guide principal](../README.md), étapes 1 à 12.**
Ces pages répondent à une question précise ; elles ne se lisent pas dans l'ordre
de leurs numéros et ne constituent pas une deuxième installation.

## Préparer le raccordement

- Assistant **Déploiement** dans NiceGUI : [préparation guidée du raccordement](../README.md#préparer-avec-nicegui).
- [Fiche de raccordement à compléter avec les responsables du SI](26-si-connection-sheet.md).
- [Détail des fichiers et contrats de configuration](02-configuration.md).
- [Glossaire : instance, mapping, LDAP, ACL, jeton et recette](23-glossary.md).

### Fiche de raccordement

Elle rassemble les paramètres fournis par les responsables des sources : URLs,
CA, comptes lecteurs, groupes, index, séries et cas métier. Les secrets sont
remis séparément. Elle se complète avant le déploiement ; le guide explique
ensuite où reporter ces valeurs dans les fichiers privés du PoC.

## Comprendre le code, les flux et les accès

- [Architecture et services](03-architecture.md).
- [Moteur et mappings PDL](04-pdl.md).
- [Diagnostic Incidents et contrats des sources](05-incidents.md).
- [Identités et ACL des deux UC](06-acl.md).
- [Précédents et solutions ITSM-NG](15-itsm-precedents.md).
- [Connexion personnelle Open WebUI, LDAP et OAuth](25-user-access.md).

## Exploiter et réceptionner

- [Exploitation, mise à jour, sauvegarde et restauration](07-operations.md).
- [Résultats de qualification et limites](08-qualification.md).
- [Grille de recette manuelle](12-manual-acceptance.md).
- [Rapports Allure et intégration continue](13-test-reports.md).
- [Réception dans le SI-RIE réel et responsabilités](14-si-rie-acceptance.md).

## Démontrer et s'exercer dans le laboratoire

- [Démonstrations NiceGUI et captures datées](09-demonstrations.md).
- [Tutoriel métier PDL](16-pdl-tutorial.md).
- [Tutoriel métier Incidents](17-incidents-tutorial.md).
- [Paramètres et état de la sandbox du 16 novembre](22-external-sandbox.md).

## Maintenir le code et publier les images

- [Espace VS Code, builds, publication dans la registry et cycle de correction](27-development-and-images.md).

## Consulter une variante ou exporter vers le wiki

- [Référence d'installation : démonstration autonome, script et construction des images](01-installation.md).
- [Exporter et importer Markdown, images et draw.io dans Wiki.js](10-wikijs.md).

## Retrouver une preuve historique

Ces pages conservent les dates, versions, échecs et limites réellement observés.
Elles ne décrivent pas des commandes supplémentaires à rejouer pour installer.

- [Journal des installations et corrections](11-installation-log.md).
- [Campagne initiale de stabilité Open WebUI](18-openwebui-stability.md).
- [Corrections et conversations requalifiées](19-openwebui-corrections.md).
- [Remise et qualification Ubuntu du 13 novembre](20-final-ubuntu-qualification.md).
- [Réserves et clôture des preuves du 14 novembre](21-evidence-reservations.md).
