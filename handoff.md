# Handoff — ROLEX-SITE / variations-expo.html

Date : 2026-06-02
Branche de travail : `main` / `claude/rolex-webflow-site-FF8tB`
Sauvegarde stable : `stable-expo-entry-click-scroll-working` (commit `9c10334`)
Déployé sur : https://frpone.github.io/ROLEX-SITE/variations-expo.html

---

## Objectif général

Construire un site portfolio Rolex en trois parties :

1. **Couronne 3D** — animation Three.js pilotée par le scroll (ScrollTrigger + GSAP)
2. **Affiche d'entrée** — poster cliquable qui déclenche l'entrée dans l'expo
3. **Séquence d'entrée de la salle** — 143 frames JPEG scrubées au scroll sur un `<canvas>` fixed, se terminant face aux trois affiches avec lightbox interactive

---

## État actuel (stable)

### Ce qui fonctionne
- Séquence 143 frames chargées depuis `./FRAMES_INTRO2/frame_001.jpg` → `frame_143.jpg`
- Canvas responsive : `resizeCanvas()` recalcule `canvas.width/height` à chaque resize et redessine la frame courante
- `drawImageCover()` : object-fit cover, coordonnées entières (`Math.round`), `clearRect` avant chaque frame
- Aucun flash noir à l'entrée : `phase-expo` reste `display:none` jusqu'à ce que `frame_001` soit dessinée
- Clic sur n'importe quelle zone de l'image → avance automatiquement jusqu'à la dernière frame via `lenis.scrollTo(SCROLL_H, { duration: 2.4 })` (listener global en capture sur `document`)
- Scroll manuel fonctionne dans les deux sens
- Overlay des affiches (`#frame-overlay`) apparaît à `progress >= 0.985`, disparaît en scrollant en arrière
- Lightbox (prev/next, zoom ×2, swipe) opérationnelle sur les 3 affiches
- `SCROLL_H = 2500` px
- `html { overflow-y: scroll; scrollbar-gutter: stable }` — évite tout reflow lié à la scrollbar
- `#scroll-hint` positionné via `getCoverRect()` + coordonnées source normalisées (`HINT_SOURCE_X = 0.524`, `HINT_SOURCE_Y = 0.925`)

### Paramètre de réglage fin
```css
:root {
  --scroll-hint-offset-x: 25px; /* décalage horizontal du hint si besoin */
}
```
*(actuellement remplacé par le positionnement JS via coverRect — cette variable n'est plus active)*

---

## Fichiers modifiés

| Fichier | Rôle |
|---|---|
| `variations-expo.html` | Fichier unique — tout le site |
| `FRAMES_INTRO2/frame_001.jpg` → `frame_143.jpg` | Frames extraites de `INTRO4.mp4` à 2560×1440 |

Aucun autre fichier touché.

---

## Fonctions clés dans `initImageSequence()`

| Fonction | Rôle |
|---|---|
| `getDpr()` | Retourne `min(devicePixelRatio, 2)` |
| `resizeCanvas()` | Recalcule canvas + spacer + redessine + repositionne hint |
| `drawImageCover(img)` | Dessine une image en object-fit:cover sur le canvas |
| `renderFrame(idx)` | Point d'entrée unique — clamp idx, vérifie img.complete, appelle drawImageCover |
| `getCoverRect()` | Même math que drawImageCover, retourne `{x,y,w,h}` en px CSS |
| `positionScrollHint()` | Positionne #scroll-hint via coordonnées source normalisées |
| `showPosterArrows()` / `hidePosterArrows()` | Toggle opacity de `#frame-overlay` |
| `tick()` | rAF loop : progress → renderFrame + hint + arrows |
| `goToEnd()` | `lenis.scrollTo(SCROLL_H, { duration: 2.4, easing: cubic })` |
| `handleExpoClick(e)` | Guards overlay/lightbox, appelle goToEnd() |
| `isPhaseExpoActive()` | Vérifie `computed display !== 'none'` sur `#phase-expo` |
| `shouldIgnoreExpoClick(e)` | Ignore header, liens, lightbox ouverte, overlay visible |
| `startWhenFrame0Ready()` | Attend frame 0 → resizeCanvas → renderFrame(0) → cache loader → show phase |

---

## Ce qui a été tenté sans succès

### Décalage de la dernière frame (résolu)
- **Tentative 1** : `document.body.style.overflow = 'hidden'` supprimé → la scrollbar disparaissait quand lenis stoppait → viewport s'élargissait → canvas se redimensionnait → saut visuel
- **Tentative 2** : guard `if (_scrollStopped) return` dans `resizeCanvas()` → insuffisant car le CSS `width:100%` du canvas recalculait quand même le layout
- **Tentative 3** : verrouillage `canvas.style.width = window.innerWidth + 'px'` avant `lenis.stop()` → améliorait mais ne supprimait pas complètement le saut
- **Solution finale** : supprimer `lenis.stop()` entièrement + `html { scrollbar-gutter: stable }` → aucun reflow possible

### Centrage de #scroll-hint (résolu)
- Plusieurs tentatives avec `left: 50%` + offsets en px (+10, +50, +35, 0, +15, +25) — résultat incohérent selon la taille d'écran car centré sur le viewport et non sur le passage visible dans l'image
- **Solution finale** : `getCoverRect()` + coordonnées source normalisées `(0.524, 0.925)` → position ancrée sur le couloir réel de l'image

### Flash noir à l'entrée (résolu)
- `phase-expo.style.display = 'block'` appelé immédiatement dans `enterExpo()` → `#intro2-fixed { background:#000 }` + `#intro2-loader` couvraient le canvas
- **Solution** : garder `phase-expo` en `display:none`, passer un callback `onFirstFrameReady` à `initImageSequence()`, afficher seulement après que frame 0 soit dessinée et loader caché

### Clic sur canvas (résolu partiellement)
- `canvas.onclick` seul ne suffisait pas à cause de la hiérarchie z-index / pointer-events des couches fixed
- **Solution** : listener global `document.addEventListener('click', handler, true)` en capture — reçoit l'événement avant tout élément enfant

---

## Prochaines étapes possibles

- Vérifier le comportement mobile (touch scroll + tap pour avancer)
- Affiner `HINT_SOURCE_X` / `HINT_SOURCE_Y` si le hint n'est pas bien centré sur certaines résolutions
- Tester le chargement progressif des frames sur connexion lente (le loader est caché dès frame 0, les frames suivantes peuvent manquer pendant le scroll)
- Ajouter une transition de sortie de la phase expo (fondu vers la page suivante si applicable)
- Créer un `CLAUDE.md` avec les règles de contribution (ne pas toucher à drawImageCover, responsive, etc.)
