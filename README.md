# panoramax-360-accuracy

Local web tool to re-position 360° street-level photo sequences (GoPro Max…) using OSM/IGN landmarks as anchors, with a reversible EXIF trace, before or after upload to [Panoramax](https://panoramax.fr).

Outil web local pour recaler des séquences de photos 360° de rue (GoPro Max…) sur des repères OSM/IGN, avec une trace EXIF réversible, avant ou après leur téléversement dans Panoramax.

> Statut : prototype utilisable, testé sur une séquence de ~2 600 photos (RD938) et sur l'instance `panoramax.openstreetmap.fr`.

## Utilisation

Un seul fichier, aucune installation : ouvrir `index.html` dans le navigateur (Chrome ou Edge pour l'écriture directe dans les fichiers). Le fond de carte et les repères OSM nécessitent un accès réseau ; si les fonds restent vides, servir le dossier en local (`python3 -m http.server`) plutôt que d'ouvrir le fichier depuis un aperçu intégré.

1. **Importer** : dossier de photos (position lue dans l'EXIF), CSV ExifTool, ou séquence Panoramax (URL + jeton).
2. **Repérer** : vigilances (pas, vitesse, densité, écarts), repères OSM (bornes, passages piétons, îlots…), vues aériennes (IGN, Bing).
3. **Examiner** : mode examen avec visionneuse 360° à côté du marqueur, cône de vue, lecture de la séquence.
4. **Caler** : poser des ancres sur des repères sûrs ; redistribution des photos intermédiaires « en chaise musicale » le long de la trace d'origine (décalage borné en nombre de photos, correction latérale optionnelle).
5. **Arbitrer les doublons** : clusters de photos trop proches (stop, embouteillage), choix de la photo à garder.
6. **Écrire** : positions et cap dans l'EXIF (Chrome/Edge), ou CSV ExifTool ; ou envoi vers Panoramax (position, cap, visibilité, tags).
7. **Reprendre** : la session (ancres, exclusions, photos examinées) est enregistrée automatiquement.

## Traçabilité et réversibilité

Chaque photo recalée porte l'état de son calage (`reposition/1` : position d'origine, ancre et thématique, méthode, changeset, date, historique), et les photos déjà en ligne reçoivent des tags `reposition:*`. Voir [docs/reposition-protocol.md](docs/reposition-protocol.md).

## Limites connues

- Écriture directe dans les JPEG : seul le bloc GPS de l'EXIF est modifié ; le XMP 360° (GPano) est conservé.
- Non testé : lecture du champ `reposition/1` par ExifTool, PATCH Panoramax avec `semantics`, exposition du champ par Panoramax.
- Certains tags OSM d'ancres (`road_marking=*`, `junction=yes`) ne sont pas des usages OSM établis (voir le protocole).
- Resynchronisation temporelle sur une trace GPX : pas encore implémentée.

## Licence

MIT, voir [LICENSE](LICENSE).
