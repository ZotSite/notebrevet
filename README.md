# notebrevet

Ticket à gratter interactif pour annoncer les résultats du brevet des collèges.

Le ticket est rendu en WebGL : une couche d'or métallique procédurale (relief, reflets, éclats) recouvre les 4 cases. On la gratte à la pièce, avec les copeaux qui tombent, le son de grattage et la vibration sur mobile. Chaque case se révèle indépendamment une fois suffisamment grattée ; quand les 4 sont découvertes, les confettis partent.

## Les notes

Elles se règlent dans `notes.json` :

```json
{
  "francais": 10,
  "histoire": 12,
  "sciences": 14,
  "maths": 16
}
```

## Lancer

Un serveur local est nécessaire (le navigateur refuse de charger l'image en texture WebGL depuis `file://`) :

```bash
npx serve .
```

Puis ouvrir l'adresse affichée. Aucune dépendance, aucun build : tout tient dans `index.html`.

Astuce : ajouter `?reveal` à l'URL révèle les 4 cases immédiatement.
