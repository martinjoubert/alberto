# Sources complémentaires des diagrammes

Les douze figures sont des modèles draw.io natifs dans `diagrams/`,
avec des objets et connecteurs éditables. Leurs SVG embarquent aussi le modèle
XML ; les PNG sont des rendus locaux destinés à la lecture.

Les figures 09 à 12 sont les conversions des quatre diagrammes Mermaid des
guides `docs/demonstrations/pdl-architecture.md` et `incidents-architecture.md`,
affichés par `render_architecture()` dans `demos/pdl-demo/pdl_technical_flow.py`.
Leurs sources Mermaid exactes sont conservées ici :

- [PDL : composants](09-pdl-demo-architecture.mmd).
- [PDL : déroulement](10-pdl-demo-flow.mmd).
- [Incidents : composants](11-incidents-demo-architecture.mmd).
- [Incidents : déroulement](12-incidents-demo-flow.mmd).

Le contenu et les connexions de ces figures restent ceux des démos ; les
différences de la livraison Ubuntu sont précisées dans
[Architecture des deux UC et des ACL](../reference/03-architecture.md).
La conversion utilise le graphe Mermaid et sa géométrie pour produire des
blocs, losanges, groupes et connecteurs natifs, y compris les flèches doubles
et pointillées. Le SVG conserve le même modèle draw.io dans son attribut
`content`, pour l'export Wiki.js.

Pour régénérer ces quatre figures depuis le checkout avec Playwright, un
navigateur Chromium et le bundle Mermaid local de NiceGUI :

```bash
python scripts/export_demo_architecture.py \
  --mermaid-dist /path/to/nicegui/elements/mermaid/dist \
  --browser-executable /path/to/chromium
```

Le script ne contacte aucun service externe. Il met à jour les sources `.mmd`,
les modèles `.drawio` et les rendus `.svg` et `.png`. Les contrôles d'export
vérifient que le modèle embarqué dans chaque SVG correspond au fichier draw.io.

Ce répertoire conserve deux sources PlantUML du tutoriel de mapping portable :
[mapping-relations.puml](mapping-relations.puml) et
[mapping-workflow.puml](mapping-workflow.puml). Leur sens métier reste valable
pour la livraison Ubuntu ; leurs commandes sont expliquées dans la page PDL.
Ces deux fichiers sont des copies publiques des sources conservées dans
`docs/code-tutorial/assets/uml/`. Elles permettent au paquet SI-RIE de rester
autonome. Lors d'une modification, synchroniser les deux exemplaires avant
l'export ; une suppression ici rendrait les liens du paquet incomplets.

La [séquence Mermaid d'identité](local-client-identity.mmd) décrit les jetons
individuels des clients Open WebUI locaux. Elle remplace pour cette livraison
la représentation historique d'une signature commune confiée à Open WebUI.

La lecture des pages ne dépend pas d'un serveur PlantUML, Kroki ou Mermaid.
Toute régénération doit se faire avec des outils locaux approuvés, puis être
vérifiée visuellement. Un export de texte seul ne prouve pas que le dessin
final est lisible ou qu'il reste éditable.
