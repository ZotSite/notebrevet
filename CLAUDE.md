# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Projet

Ticket à gratter "Résultats du Brevet" en WebGL, entièrement contenu dans `index.html` (HTML + CSS + GLSL + JS vanilla, aucune dépendance, aucun build). `gratte.png` (1049×1499) est l'image du ticket, chargée comme texture WebGL. Les notes affichées sous le grattage viennent de `notes.json` (`francais`, `histoire`, `sciences`, `maths` — l'ordre des clés dans `SUBJECTS` doit correspondre à celui de `ZONES`).

## Lancer

Servir localement (obligatoire : en `file://`, Chrome bloque le chargement de `gratte.png` comme texture WebGL) :

```
npx serve .
```

Pas de tests, pas de lint, pas de bundler. Vérification visuelle possible en headless :

```
chrome --headless=new --disable-gpu --enable-unsafe-swiftshader --window-size=800,900 --screenshot=out.png --virtual-time-budget=4000 http://localhost:3000
```

Paramètre d'URL `?reveal` : révèle instantanément toutes les cases (`?reveal=0,3` seulement certaines). Sert à vérifier le calage des `ZONES` sur l'image sans avoir à gratter.

## Architecture (tout dans index.html)

Le rendu repose sur deux passes WebGL partageant un quad plein écran :

1. **Passe brush** (`FRAG_BRUSH`) — dessinée dans un framebuffer offscreen (`maskFBO` / `maskTex`, résolution `MW×MH` = 384×~233). Chaque mouvement de pointeur trace un segment (uMouse→uPrev) avec du bruit procédural, accumulé en blend additif. Cette texture est le **masque de grattage** (rouge > seuil = gratté).
2. **Passe composite** (`FRAG_COMP`) — rend le ticket final : texture du ticket + couche d'or métallique procédurale (height field dérivé du masque, éclairage spéculaire, envMap procédurale, glints). Les 4 zones grattables (Français, Histoire-Géo, Sciences, Maths) sont définies par `ZONES` [x, y, w, h] en UV (origine en bas à gauche, convention GL), mesurées sur les cases grises de `gratte.png`, et passées au shader via `uZones[4]`.

Autour de ça :

- **Texture du ticket** : `composeTicket()` dessine `gratte.png` en canvas 2D puis peint dans chaque case un panneau papier portant la note lue dans `notes.json` (ce panneau recouvre la mention imprimée sur l'image). Le résultat est uploadé via `setTicketTexture`. L'upload et le démarrage de la boucle rAF attendent le `Promise.all` image + fetch JSON.
- **Révélation par case** : `scratchedPctPerZone()` fait un `readPixels` par zone toutes les 12 frames. Dès qu'une case dépasse `ZONE_THRESHOLD` (55 %), `revealZone(i)` la dissout seule — chaque zone a son propre avancement dans `reveal[4]` (uniforme `uReveal[4]`, indexé par la zone trouvée dans le fragment shader). `finish()` (bannière + confettis) n'est appelé qu'une fois les 4 cases révélées.
- **Particules** (copeaux + confettis) : second canvas 2D superposé (`#flakes`), tableau `particles` partagé, distingué par le flag `confetti`.
- **Audio** : Web Audio procédural (bruit blanc filtré pour le grattage, arpège d'oscillateurs pour le résultat). Initialisé au premier `pointerdown` (contrainte autoplay).
- **Interactions** : la pièce (`#coin`) est un div qui suit le pointeur ; tilt 3D du ticket via CSS transform sur `#wrap` ; la lumière du shader suit la souris ou le gyroscope (`deviceorientation`) ; vibration haptique sur mobile.

Constantes clés en tête de script : `TW/TH` (taille ticket 434×620, même ratio que gratte.png), `DPR` (capé à 2), `ZONES`, `MW/MH` (résolution du masque).
