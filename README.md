# Alberto — livraison PDL, Incidents et ACL

**Déployeur SI-RIE : [commencer par le guide pas à pas, étapes 1 à 12](docs/si-rie/24-deploy-walkthrough.md).**

Faire compléter d'abord la [fiche de raccordement](docs/si-rie/26-si-connection-sheet.md)
par les responsables du SI. Elle précise les accès, certificats, identités,
images approuvées de la registry GitLab et données de recette.

## Fichiers à transmettre au serveur Ubuntu

- [Sources PDL + Incidents + ACL](rie-si-rie-ubuntu-2026-11-17-si-deploy.tar.gz)
  et [leur SHA-256](rie-si-rie-ubuntu-2026-11-17-si-deploy.tar.gz.sha256).
- [Documentation Wiki.js portable](rie-si-rie-wikijs-2026-11-17-si-deploy.zip)
  et [son SHA-256](rie-si-rie-wikijs-2026-11-17-si-deploy.zip.sha256) : Markdown,
  images locales et sources draw.io éditables.
- [Références de la livraison](DELIVERY.json) : commit source, empreintes et
  résultats déterministes. Les paramètres et secrets réels sont remis séparément.

Ce dépôt distribue des archives : un `git clone` de ce dépôt ne décompresse
pas les sources. Les commandes `deploy/ubuntu` se trouvent dans l’archive
source, après extraction.

Déposer l'archive source et son SHA-256 sous `$HOME/deploy` sur Ubuntu, puis
suivre le guide. Ne pas extraire par-dessus une ancienne installation. Le
README de l'archive renvoie au même guide ; les autres manuels sont des références.
Open WebUI demeure installé sur chaque poste utilisateur ; aucun frontend
Open WebUI central n'est créé.

Le code opérateur sélectionné est dans l'archive vérifiable, sans historique
du dépôt de développement. Le moteur est celui qualifié dans le laboratoire ;
la réception des sources, droits, PKI, quotas et performances du SI réel reste
à effectuer. Les anciens bundles PDL ci-dessous sont conservés historiquement,
ils ne remplacent pas cette livraison des deux UC.
