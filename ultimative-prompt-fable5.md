# ULTIMATIVE PROMPT · Hermes Village (Fable 5, eine Datei)

> Copy-paste den Block unten komplett als Prompt. Eine Datei, kein Build-Step.

---

Build **Hermes Village**: a single-file, self-contained 3D miniature world in one
`village.html`. Load three.js from a CDN (`three@0.160` module + GLTFLoader), no build
step, no other files. The world is a small floating island seen from a soft orbit
camera: a cozy toy-like village of an AI-agent company called Hermes. The look must be
**soft Blender-style "toy clay"**, NOT blocky: rounded volumes, smooth shading, gentle
ambient occlusion feeling, warm stylized colors, everything slightly hand-made.
Accent color of the UI: **#F0A040** (warm amber). Style reference: polished cozy
low-poly dioramas (Zelda-adjacent warmth, Monument-Valley cleanliness).

## BLOCK 1 · WORLD (the scene)

- Floating island (radius ~22 units) over a calm dark-blue sea circle; a soft
  snow-capped mountain ridge sits at the back edge (3 smooth cones, no visible facets);
  3 more distant gray-blue water-mountains rise from the sea behind the island.
- Grass ground in warm green with subtle darker patches; sandy paths ribbon from the
  central Plaza (a round stone circle with a well) to the five districts:
  **Plaza** (center), **Hafen** (boathouse + wooden pier + water), **Wohnviertel**
  (cottages), **Werkstatt** (workshop + windmill), **Markt** (market stalls with
  striped awnings).
- Buildings: rounded-box cottages (beveled walls, no 90° hard edges) with pyramid
  roofs, chimneys (light smoke puffs), warm glowing windows at night; a taller 3-floor
  "Shimeria Media Haus" with a glowing neon sign; a round lookout tower; a water mill;
  a small observatory dome. If GLB/Blender models are available, use them instead of
  primitives; primitives must still look soft and toy-like.
- Nature: ~40 trees in two kinds (smooth conical conifers, lumpy round canopies with
  deterministic bumps), bushes, smooth pebble rocks (squashed, dented spheres, never
  faceted crystals), flower patches, a pond with floating leaves, reeds, a small beach
  with a waterfall from a floating rock, lanterns along all paths.
- Day/night cycle driven by a clock (override with `?time=HH:MM`): golden-hour warm sun
  at dusk, deep blue night with stars, moonlight, lamps and windows turn on, gentle lamp
  flicker at night.

## BLOCK 2 · LIFE (the simulation)

- 3+ villagers wander endlessly between 10 wander points across all districts; they
  pause, pick a new target, walk with soft bobbing animation. Yield rule: on head-on
  paths, both turn RIGHT (give-way); radial separation so nobody ever overlaps
  (agents radius 1.5, buildings radius 2.1, the fox character **Kira** is never
  overrun, radius 1.1). All loops are endless, nothing ever freezes.
- Job robots (small service bots) leave their buildings during "work" and stroll through
  the village after work, then return; endless day rhythm.
- Kira (small fox) idles near the Plaza, occasionally trots a short loop.

## BLOCK 3 · INTERACTION (the camera & selection)

- Left-drag pans, right/middle-drag orbits (clamped pitch), wheel + two-finger pinch
  zoom (distance 9–60), touch-friendly. Prevent context menu.
- Click any building, villager, robot or Kira: camera **glides** to it (~900 ms
  ease-in-out), then follows it with a soft lerp; the target gets a pulsing ground ring
  (amber), a floating name label, and a frosted-glass panel with **live stats** (name,
  role, status rows that refresh every 0.7 s). Click empty ground to deselect.
- Area switcher in the topbar flies the camera between Plaza, Hafen, Wohnviertel,
  Werkstatt, Markt.

## BLOCK 4 · DASHBOARD (the HUD, frosted glass)

- Topbar: logo "HERMES", search field, area switcher, weather menu, season menu, live
  clock, alert dot, day/night toggle, round user avatar.
- Three KPI cards (Agents online / Services running / Jobs completed) with big numbers
  that tick realistically every few seconds.
- Right detail panel appears when something is selected.
- Bottom-left: a 6-step process tracker (e.g. Inbox triage → Plan → Tools → Reply →
  QA → Done) with one active step, and a tabbed live table (Agents / Services / Jobs)
  with realistic rows (agent names like "Copy Studio", "Media", "Mine", "Gate"; no
  lorem ipsum), refreshed on a 500 ms tick.
- Everything is real DOM overlaying the canvas (CSS `backdrop-filter` frosted glass),
  not in-canvas UI.

## BLOCK 5 · EXTRAS (mood systems)

- Weather: Clear / Rain (rain particle streaks + darker tint + puddle sheen), gentle
  auto-switching optional.
- Seasons menu: Spring (fresh light green), Summer (default), Autumn (ochre grass,
  orange crowns), Winter (snow blanket, desaturated crowns, icy-blue sea, cool light,
  700 drifting snow particles). All transitions lerp softly over ~1–2 s.
- Chimney smoke puffs, waterfall splash, windmill blades turning, fireflies at night,
  confetti burst when a job completes in the table.

## QUALITY BAR (must all hold)

- One HTML file, works offline-ish (only three.js CDN), 0 console errors.
- Runs 60 fps on a laptop; all animations are time-based, not frame-based.
- `window.__dbg()` returns live state (agents, season, weather, selected object);
  `window.__snap()` returns a PNG data-URL of the canvas for automated review.
- Mobile: touch pan/rotate/pinch works, HUD collapses to essentials on small screens.
- The whole scene must feel like a warm, alive toy diorama, soft and Blender-like,
  never angular or blocky.
