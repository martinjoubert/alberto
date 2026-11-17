# Exporter vers Wiki.js et maintenir les diagrammes

Le dossier canonique est `docs/si-rie/`. Le portail facultatif inclut ces pages
dans son image dérivée ; le runtime ordinaire ne dépend pas de NiceGUI.
NiceGUI et le paquet documentaire
présentent les mêmes pages et figures. L'export ne contacte aucun wiki et ne
publie pas automatiquement dans le SI.

## Export complet ou fichier individuel

Depuis la racine du code, choisir un nouveau fichier ZIP de sortie :

```bash
python3 deploy/ubuntu/wiki_export.py --output /EXPORT/PATH/dinum-rie-documentation.zip --base-path /dinum-rie
```

Le ZIP contient :

| Entrée | Utilisation |
| --- | --- |
| `README.md` | Point d'entrée de la remise |
| `portable/` | Markdown et pièces jointes, avec liens relatifs utilisables hors wiki |
| `wikijs/` | Même contenu, liens adaptés au préfixe choisi |
| `wikijs-native/` | Variante pour l'éditeur Markdown Wiki.js 2.x, avec ses blocs de diagrammes éditables |
| `DOCUMENTATION-MANIFEST.json` | Liste des fichiers, tailles, empreintes SHA-256 et identité source disponible |

La commande refuse d'écraser un ZIP existant. Le paramètre `--source-root`
permet de tester un dossier extrait ; `--source-base-url` peut enregistrer l'URL
HTTPS de la version du code remise. Un checkout modifié est identifié comme tel,
pas présenté comme une publication déjà réalisée.

Pour une page seule, copier son Markdown et les images qu'elle référence, ou
utiliser le téléchargement individuel dans NiceGUI. Les `.drawio`, SVG et PNG
sont téléchargeables indépendamment. Le navigateur ne reçoit pas de chemin
libre vers les secrets du serveur : les fichiers publics suivent une liste de
formats et de répertoires autorisés.

## Importer dans Wiki.js

La version et la configuration du wiki cible sont encore à confirmer. Le
parcours portable reste valable sans plugin ni accès Internet :

1. Choisir le préfixe de pages, par exemple `dinum-rie`, lors de l'export.
2. Dans Wiki.js, créer une page Markdown par fichier du dossier `wikijs/`.
   `README.md` devient la page d'entrée du préfixe ; les autres noms deviennent
   les chemins correspondants, sans suffixe `.md`.
3. Charger les images et sources de `assets/`, `diagrams/` et `sources/` du
   paquet sous le même préfixe ou ajuster les liens selon les règles du wiki.
4. Vérifier l'index, les tableaux, les blocs de commande, les accents, les
   figures et les liens entre pages avant publication aux lecteurs.
5. Faire relire la page de qualification et les scénarios par leurs responsables.

Avec l'éditeur Markdown et l'intégration draw.io de Wiki.js 2.x, choisir à la
place les pages de `wikijs-native/`. Elles contiennent le format natif du plugin :
un bloc de code `diagram` dont la ligne centrale encode le SVG et son modèle
en base64. Le bouton **Edit Diagram** de l'éditeur ouvre ce modèle. Copier le
bloc entier ; ne pas modifier sa ligne encodée à la main. Les liens vers les
fichiers `.drawio` restent disponibles comme sources indépendantes du plugin.

Après modification du dessin, utiliser **Enregistrer et quitter** ou le
raccourci indiqué par draw.io, puis **Save** dans l'éditeur de page Wiki.js.
Ces deux sauvegardes sont distinctes : le retour du dessin met à jour le texte
Markdown, et la sauvegarde de la page conserve ce texte dans le wiki.

Ce format est celui du [code officiel de l'éditeur Markdown Wiki.js 2.5.315](https://github.com/requarks/wiki/blob/v2.5.315/client/components/editor/editor-markdown.vue).
La variante `wikijs/` et les PNG/SVG restent préférables lorsque le module ou
la version du wiki cible ne sont pas encore identifiés. Importer une seule
variante de chaque page à un même chemin.

Si le stockage Git de Wiki.js est retenu, utiliser un dépôt documentaire dédié,
conformément à sa [documentation Git](https://docs.requarks.io/en/storage/git).
Ne pas supposer que la synchronisation d'un sous-dossier du dépôt applicatif
suffit. L'export ZIP constitue une autre remise, qui ne nécessite pas ce module.

## Modifier un vrai diagramme draw.io

Chaque figure principale possède une source native `.drawio`, faite de boîtes
et de connecteurs modifiables, et des rendus SVG/PNG. Ouvrir le `.drawio` dans
l'éditeur approuvé ou draw.io Desktop, modifier un libellé ou une liaison,
enregistrer puis régénérer les images locales. Vérifier la lisibilité après
l'export ; ne pas remplacer la source par une capture du dessin.

Le SVG peut embarquer le modèle natif pour faciliter un réimport. Un SVG
ordinaire importé comme image unique ne devient pas un diagramme éditable.
Références : [formats d'import/export draw.io](https://github.com/jgraph/drawio/wiki/Import-and-Export)
et [export local Desktop](https://github.com/jgraph/drawio/wiki/Desktop-App).

Dans le Wiki.js 2.5.315 de référence, l'import d'un SVG comme pièce jointe
assainit son XML et retire l'attribut `content` qui porte le modèle draw.io.
Le dessin reste lisible, mais cette image jointe ne suffit plus à rouvrir son
modèle. Les PNG et les `.drawio` joints séparément conservent leurs octets.
La variante `wikijs-native/` conserve le modèle dans le texte Markdown utilisé
par le plugin, avant l'assainissement de l'image affichée.

Les huit sources couvrent déploiement, simulation, flux PDL, flux Incidents,
identité/ACL, relations entre index, approbation des mappings et reprise.
Le répertoire `sources/` conserve les compléments Mermaid/PlantUML utiles,
avec des rendus locaux pour les lecteurs sans moteur de diagrammes.

## Vérifier le plugin du wiki cible

Wiki.js comporte une intégration draw.io, mais son composant de référence
charge un éditeur embarqué à `embed.diagrams.net`. L'édition dans un SI sans
sortie Internet dépend donc de la version et de sa configuration ; elle n'est
pas déduite de la seule présence d'un bouton. Le paquet `.drawio` et l'édition
Desktop locale permettent de préparer les figures sans cette dépendance.
Voir le [composant officiel Wiki.js 2.5.315](https://github.com/requarks/wiki/blob/v2.5.315/client/components/editor/editor-modal-drawio.vue).

Conserver l'éditeur Markdown si l'édition intégrée du diagramme est attendue.
La conversion Markdown vers un éditeur HTML peut conserver seulement l'image
rendue et perdre cette édition ; la [documentation des éditeurs Wiki.js](https://docs.requarks.io/s/editors)
décrit cette limite. L'essai sur un Wiki.js local de référence doit vérifier
lecture, ouverture dans draw.io, modification, sauvegarde et réexport. Il ne
remplace pas l'essai sur la version exacte du SI.

## Maintenir une documentation cohérente

Modifier d'abord la page canonique et la source du diagramme, puis régénérer
les rendus et le ZIP. Les liens du mode portable doivent rester fonctionnels
après extraction sans réseau. Le manifeste permet de comparer le fichier
servi par NiceGUI, le fichier exporté et celui remis pour le wiki.

Les résultats de recette, dates et captures sont liés à leur version. Une
ancienne preuve reste historique ; elle ne doit pas prendre la date de la
nouvelle livraison sans nouvelle exécution. Les journaux privés, sauvegardes,
clés et jetons ne sont jamais des pièces jointes du paquet documentaire.
