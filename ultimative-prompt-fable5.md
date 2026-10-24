# Ultimativer Fable-5-Prompt: Hermes Village

Der komplette Copy-Paste-Prompt für Fable 5, um Hermes Village neu zu bauen.
Struktur folgt dem World Prompt Pack (@sanjai.builds): WORLD / LIFE / INTERACTION /
DASHBOARD / EXTRAS. Auf Hermes Village zugeschnitten: Agentengebäude, echte
Dashboard-Daten, deutsches UI. Alles in einer Datei, ein Paste, ein Build.

---

**COPY THIS PROMPT — PASTE INTO FABLE 5**

Build a single-file interactive web app (HTML, CSS and JS) using Three.js from a CDN:
a live 3D village operations dashboard called "Hermes Village" for a personal AI-agent
system ("Hermes") that feels like a strategy game, rendered as a living toy-like clay
diorama. UI language is German. Accent color: #FF7A29 (Hermes orange) on deep
dark teal-green base with warm off-white text.

WORLD: a circular island village built in code with soft, rounded, Blender-style
materials — no hard-edged boxes, no blocky primitives: use spherified, noise-warped
geometry everywhere (trees, rocks, terrain, building bases melt softly into the grass
as earthen skirts). The island edge shows a dark earth cross-section like a slice of
cake, with a small stream that tips over the edge as a tiny waterfall. Paths are
organically curved, sand-colored ribbons with soft blended edges, small pebbles and
short side-paths — never symmetrical spokes. A central plaza with a round fountain
and market stalls, a bell tower; a harbor quarter at the shore with a wooden pier,
two fishing boats and a lighthouse; a residential quarter with pitched-roof cottages;
a workshop district; a wooden bridge over the stream and an irregularly shaped pond.
Each building is an agent: Copy Studio, Research Observatory, Backup Mine, Shimeria
Media Haus, Cron Gatehouse, Web Hotel, Digest Mill, Telegram Zentrale — each with a
small warm glowing sign and soft window pools. Vegetation mixes pines and round-crown
trees (some autumn-orange), bushes, soft rocks, flower patches. Benches, picnic
tables, a red parasol, lanterns with warm light pools. One accent color #FF7A29 on
warm greens and sand. Soft shadows, gentle vignette, camera close enough that the
island fills the frame.

LIFE: agent-villagers walk the paths forever in endless loops: short strolls between
nearby waypoints with a 45% pull toward the plaza, give-way logic everywhere —
everything yields, nothing overlaps: agents slow and sidestep near obstacles, brake
and keep right at narrow choke points (bridge, gate), overtake slower walkers
sideways, and never clip each other. When two agents come within 2.5 units they
naturally stop and chat (floating chat bubble with one-line quotes like "Hooks
schreiben. Der dritte ist immer der beste."), then stroll on together as a small
group and part again. Give them names and roles (KIRA, LIRA, BRUNO, ...) and vary
speeds and routes so the scene is never static.

INTERACTION: drag to pan, right-drag to rotate, scroll or pinch to zoom. Click any
building, villager or boat: the camera glides to it and follows it as it moves, a
pulsing ring and a floating soft-shadowed label mark it, and a frosted glass detail
panel shows live stats (status, last run, success rate, current activity). An area
switcher flies the camera between Plaza, Hafen, Wohnviertel and Werkstatt.

DASHBOARD: frosted glass panels over the 3D scene. Top bar: logo "HERMES VILLAGE",
a search box ("Suche: Haus, Agent, Job ..."), area switcher (Plaza, Hafen,
Wohnviertel, Werkstatt), a weather menu, a live clock, an alerts bell, a day/night
toggle and a user chip. Three KPI cards: Aktive Läufe, Erfolgsquote (letzte 50 Läufe),
Services (Häuser im Dorf). A right detail panel "Dorf-Übersicht" with realistic live
data: active runs, last run "daily-backup", status "finished", jobs, success rate 100%,
services 10, season and weather. Bottom-left a run tracker with six steps: Eingang,
Zugewiesen, Arbeitet, Prüfung, Fertig, Protokolliert — each step lights up in real
time. A tabbed live table with three tabs (Agenten, Jobs, Läufe) listing real
Hermes-agent rows: Copy Studio, Research Observatory, Backup Mine, Shimeria Media
Haus, Cron Gatehouse, Web Hotel, Digest Mill, Telegram Zentrale with status idle or
finished, job names like "daily-backup" and "Morning Digest", run counts and
percentages. Occasionally KIRA pops a chat bubble with a witty one-liner. Realistic
data only, no lorem ipsum.

EXTRAS: a season system (Sommer, Herbst, Winter, Frühling) that recolors terrain,
trees and particles — falling autumn leaves, winter snow with frost edges on the
frozen pond and snow skirts at building feet, spring blossoms. A weather menu
(Klar, Regen, Gewitter): rain with shiny puddles and streak lines, a thunderstorm
with a visible lightning bolt that strikes and thunder that arrives with a realistic
delay after the flash. A soft morning fog bell-curve peaking around 6:40 and an
evening dusk haze. Wind gusts sway tree crowns and drive seasonal particles
(leaves, petals, pollen). Animated day/night with warm lantern pools, glowing
windows and a star field. Works on mobile with a bottom-sheet panel.

After the build, self-check: everything rounded and soft (no blocky look), all six
tracker steps present, all tabs populated with realistic data, and at least one
natural agent chat triggered within 90 seconds of load.

---

**REMIX-FOLLOW-UPS (nach dem ersten Build):**
- "Mehr Leben: 12 Agenten, mehr Chat-Zufälle, ein Marktschau-Spiel am Plaza."
- "Neues Quart: Bienenhof + Felder mit Traktor-Runden im give-way-System."
- "Kamera: Auto-Tour alle 14 Sekunden durch die vier Quartiere."
- "Nacht-Feinschliff: mehr Laternen pro Fläche, stärkere Fenster-Pools."
