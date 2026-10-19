# hermes-nexus (Stufe 04)

Datenruecken fuer das Hermes-Village / Dashboard: ein read-only Collector,
der die Hermes-Zustandsflaechen in den Ereignisvertrag aus
HERMES-VILLAGE-STUFE-03.md (§10) uebersetzt. Die Demo-Oberflaeche von
Stufe 03 bleibt unberuehrt; sie kann spaeter denselben Feed konsumieren.

## Inhalt
- `collector/collector.py` - Collector, nur Python-Stdlib, nur lesend
- `contract/events.schema.json` - Datenvertrag (JSON Schema)
- `feed/events.json` - Beispiel-Feed
- `systemd-launchagent/ai.fo.hermes-nexus-collector.plist` - Auto-Start alle 60 s (optional)

## Erster Lauf (auf dem Mac, Terminal)
    python3 collector/collector.py       --hermes-home ~/.hermes/profiles/ad-copywriter       --out feed/events.json

- Der Feed enthaelt: Profil-Liveness (aus gateway_heartbeats), Cron-Runs
  (aus cron/executions.db, pseudonymisiert), Ticker-Zustand.
- Wird ein Pfad nicht gefunden oder fehlt eine Spalte, bleibt das Feld
  `unknown`. Der Feed erfindet nichts.
- Der Feed enthaelt nun auch `jobs`: Jobnamen, Zeitplan, enabled aus
  cron/jobs.json (toleranter Parser: Liste oder {"jobs": [...]}, fehlende
  Felder bleiben unknown). Das exakte Schema von jobs.json ist aus Stufe-03
  §15 unbelegt; der Parser markiert, was er nicht sicher liest.
- write_state bleibt bewusst `unknown`: Arbeitssemantik aus
  session_turn_leases / async_delegations erst nach Schema-Pruefung
  anschliessen (Stufe 03 §15).
- Cross-Profil (jarvis, shimeria-*) ist unbestaetigt: erst je Profil einen
  Collector starten und die Feeds mergen, sobald der Zugriff belegt ist.

## Stand
- M1 Collector: gebaut und auf dem Mac live gegen `~/.hermes/profiles/ad-copywriter`
  abgenommen (Echtlauf 19:07: 50 runs, 23 jobs; v3-Echtlauf 19:5x bestätigt).
- M2 jobs.json: gebaut, Schema durch Echtlauf belegt (id, name, schedule, enabled,
  state, failure_streak, last_status, provider).
- M4 Oberflaeche: `feed/dashboard.html` liest denselben Feed (events.json im
  selben Ordner, Refresh alle 30 s), zeigt Quelle dauerhaft (Live/Synthetisch),
  Unbekanntes als unbekannt, Listenansicht auf kleinen Displays. Rein lesend.
- M5 Village (Beta): `feed/village.html` - 3D-Diorama auf demselben Feed.
  Geprüft mit headless Chromium + SwiftShader (4 Tageszeiten gerendert, keine
  Konsolen-/Seitenfehler, Klick-Karten belegt):
  - Tag-Nacht-Zyklus nach echter lokaler Uhrzeit; Debug: `village.html?time=23:10`
  - Insel mit Erdschichten-Querschnitt, Wasserfall, Bäume/Büsche, Wege, Laternen
  - Fenster und Laternen leuchten nachts, tagsüber gedimmt
  - Kaminrauch; dichter, wenn Läufe laufen
  - Roboter spawnen pro laufendem Run und zerplatzen mit Konfetti beim Ende
  - Kira (Fuchs): geht bei work_state=working ins Copy Studio, Sprechblase,
    Panel mit Profil-Zustand; Warnlinie, wenn heute Läufe failed sind
  - Klickbare Service-Karten: Copy Studio / Research Observatory / Backup Mine,
    matchen echte Jobnamen, zeigen Zustand der letzten Läufe
  - Weiche Schatten an/aus (Button unten rechts)
  Befund vom Echtlauf-Umfeld: 8 Bots failen mit HTTP 402 (leeres opencode-go-
  Guthaben); 2 mit fehlendem `mcp`-Paket; fixes sind Mac-seitig.

## Abnahme (M1)
1. Selbsttest ohne Quellen: `python3 collector/collector.py` -> synthetic feed.
2. Echtlauf gegen `~/.hermes/profiles/ad-copywriter` liefert >= 1 Profil (ok oder stale).
3. Zwei Laufe hintereinander liefern identische Feldstruktur (Regression).
4. Optional als LaunchAgent installieren (Vorlage liegt bei, alle 60 s).

## Grenzen (bewusst)
- Kein Schreiben, keine Steuerung: Eingriffe bleiben in Hermes approvals.
- jobs.json wird bewusst noch nicht geparst; Zeitplaene kommen in M2.

## M6 · Nachtlauf 2: echte 3D-Figuren (2026-10-11)

- `village.html` lädt jetzt ein echtes, gerigget 3D-Modell (RobotExpressive.glb, CC-BY, as base64 eingebettet) für die Arbeits-Roboter, mit Fallback auf den Prozedural-Roboter, falls das Laden fehlschlägt.
- Roboter-Zyklus end-to-end im Headless-Render verifiziert: Spawn (Wave-Animation) -> Walk zum passenden Gebäude -> Idle; wenn der Run im Feed auf `finished` geht, despawnt der Roboter mit Konfetti-Burst (Peak 14 Partikel belegt, danach sauber aufgeräumt), keine JS-Fehler.
- Animationen: Idle, Walking, Wave (Wave nur beim Spawn, LoopOnce). Roboter laufen zum Gebäude, das per Job-Keyword gematcht wird (Studio/Observatorium/Mine).
- Ungetestet bis hierhin: Performance auf echter GPU im Browser, mehrere gleichzeitige Roboter bei vielen parallelen Runs (Code-Cap: max 6).

## M7 · Nachtlauf 3: Dorf-Ausbau (2026-10-12)

- Von 3 auf 8 Service-Gebäude, jedes mit eigener Silhouette: Copy Studio (Cottage, blaues Dach), Research Observatory (Kuppel mit Schlitz), Backup Mine (Stollen, Gleise, fahrende Lore), Shimeria Media Haus (3 Stockwerke, Leuchtschild), Cron Gatehouse (Uhrturm mit gehender Uhr nach echter Ortszeit), Web Hotel (Pfahlhaus), Digest Mill (Wasserrad am Teich, dreht), Telegram Zentrale (Bootshaus am Teich).
- Neues Umgebungsdetail: Marktplatz mit Tafel und Bänken, zwei Marktbuden mit Streifenmarkisen, Kopfsteinpflaster-Wege (Kopfsteine statt Kacheln), 6 Laternen, 5 Strommasten mit hängenden Drähten (Bezier-Sag), Gartenzaun am Copy Studio, 3 Berge mit Schneekappen am Inselrand, 46 Blumen, Steine am Rand, mehr Bäume/Sträucher.
- Kamera: Standard-Distanz 24, Startwinkel tiefer (0.88). Inselradius 17 -> 20.5.
- Fensterfarbe tagsüber entschärft (weniger Neon), Dachfarben diversifiziert (blau, purpur, petrol, rostrot, orange).
- Verifiziert im Headless-Render bei 14:00 und 23:10, 0 JS-Fehler; Robot-Spawn (Wave) mit neuem Layout belegt, Uhr und Wasserrad laufen über __extraTick.
- Ungetestet: GPU-Performance (deutlich mehr Objekte/Lichter), Klick-Karten der 5 neuen Gebäude im Detail.

## M8 · Nachtlauf 4: Klick-Karten und Roboter verifiziert (2026-10-12)

- Alle 8 Klick-Karten im Headless-Render durchgeklickt und belegt: Copy Studio, Research Observatory, Backup Mine, Shimeria Media Haus, Cron Gatehouse, Web Hotel, Digest Mill, Telegram Zentrale. Job-Zuordnung per Stichwort geprueft (shimeria-store-daily -> Media Haus, telegram-digest-send -> Gatehouse + Telegram Zentrale + Mill, insider-flow-radar -> Gatehouse, daily-backup -> Mine).
- Neue Debug-Hilfe window.__bpos(): projiziert die Gebaeude-Mittelpunkte in Bildschirmkoordinaten (fuer Tests und Fehlersuche neben window.__dbg).
- Roboter-Layout-Test: 5 synchrone Jobs -> 5 Roboter (Cap 6 unveraendert), Wave beim Spawn, danach Walking Richtung Zielgebaeude belegt; Kaminrauch steigt am Media Haus.
- Hinweis fuer schnelle Maschinen: im Headless-Software-Rendering laeuft die Szene stark verlangsamt (SwiftShader), deshalb wirkt die Wave-Phase dort lang. Auf echter GPU ist das nicht zu erwarten; trotzdem der einzige Punkt, der nur auf dem Mac final geprueft werden kann.
- Ungetestet bleibt: GPU-Performance auf dem Mac, Reihenfolge der Zielwahl bei Stichwort-Ueberschneidungen (Store/Landingpage matchen Media Haus vor Web Hotel, gewollt).

## M9 · Nachtlauf 5: Live-Anschluss per SSE (2026-10-12)

- Neu: `collector/village_stream.py` (nur Python-Stdlib, localhost-only). Dient das `feed/`-Verzeichnis statisch aus und bietet `/api/village/stream` als Server-Sent-Events-Kanal: bei jeder Feed-Aenderung geht eine `state`-Message mit dem kompletten Feed raus, dazwischen Ping-Kommentare. Start: `python3 collector/village_stream.py [port]` (Standard 8788), dann http://127.0.0.1:8788/village.html.
- `village.html` hat jetzt das Kanzie-Prinzip eingebaut: Beim Oeffnen versucht das Frontend zuerst den SSE-Stream (`EventSource`), bei 3 Fehlern faellt es automatisch aufs Polling von `events.json` (30 s) zurueck und versucht nach 60 s wieder den Stream. Der Feed-Badge zeigt weiterhin Live/Synthetisch, `window.__dbg()` meldet zusaetzlich `stream: live|poll|off`.
- End-to-end im Headless-Render verifiziert: Szenario A (Stream an) -> `stream: "live"`, 5 Roboter gespawnt, 0 JS-Fehler. Szenario B (Stream blockiert) -> Fallback `stream: "poll"`, Roboter spawnen weiter, 0 JS-Fehler. Push-Test: Feed-Datei waehrend laufender Seite geaendert -> State-Message kam an (Stand-Zeitstempel sprang, Roboterzahl 5 -> 4 nach Run-Ende), Stream blieb live.
- Ein Prozess bedient jetzt beides: statisches Hosting und Live-Feed. Der Collector (60-s-Zyklus) bleibt unveraendert und schreibt nur events.json; der Stream-Server liest nur.
- Ungetestet: Langzeit-Stabilitaet des Streams (mehrere Stunden), Verhalten bei Browser-Ruhemodus/Tabs im Hintergrund.

## M10 · Stufe 05: Pulse, Notice Board, Kueken, Feuerwerk (2026-10-14)

Neu in `village.html` (Beta 6):

- **Draehte mit Pulsen**: Router-Mast am Platz; jedes der jetzt **10** Gebaeude haengt per
  Draht daran. Läufe feuern Pulse auf dem Draht: blau = gestartet, gruen = erfolgreich,
  rot = fehlgeschlagen (Diff ueber `runs` im Feed-Beobachter).
- **Notice Board** (Schwarzes Brett nahe dem Platz): Jobs mit fehlgeschlagenem Lauf
  bekommen eine Papier-Karte (Canvas-Textur, Jobname + FEHLER), max. 6 Karten.
  Steht eine Karte, wartet ein Roboter am Brett.
- **Kueken**: `load.delegations.by_state.pending` > 0 laesst bis zu drei Kueken am
  Platz huepfen (Delegationen im Gange).
- **Feuerwerk**: Backup-/Deploy-Job laeuft erfolgreich durch -> Funkenregen ueber dem
  Dorf. Manuell testbar via `window.__fireworks()`.
- **Neue Gebaeude**: Zweitgehirn-Bibliothek (Jobs: Brain/Zweitgehirn/Second) und
  Handels-Kiosk (Trading/VTX/Kraken/Macro). Vorher landeten diese Jobs defaultmaessig
  im Copy Studio.
- **Details**: Brunnen auf dem Platz, Namensschilder an jedem Gebaeude, Ente auf dem
  Teich, Fässer/Kisten, zusaetzliche Baeume, Sterne bei Nacht (Opazitaet folgt der
  Himmelfarbe), Router-Antenne blinkt.
- Kira-Alert-Zeile entfern den veralteten OpenCode-Bezug ("Da liegt Arbeit auf dem
  Brett. Schau auf die Tafel.").
- Fix: `updateDayNight` guardt `worldLamps`-Eintraege ohne Fenster (neue Gebaeude).

Verifiziert (Headless, SwiftShader): 10 Gebaeude in `__bpos`, 5 Roboter, Pulse laufen
und despawnen, 1 FEHLER-Karte, 1 Kueken, Sterne-Opazitaet 1 bei `?time=23:10`,
Feuerwerk 40 Funken, 0 JS-Fehler. Demo: `village-standalone.html` mit eingebettetem
synthetischen Feed (1 Run als `failed`, damit die Tafel im Demo sichtbar ist).

## M11 · Stufe 05b: Wege, Platz, Blumen + Hoehen-Fix (2026-10-15)

- **Wege**: Erdwege vom Dorfplatz zu jedem der 10 Gebaeude, Platzbelag (Kreis) um
  den Brunnen, 26 Blumen auf der Wiese. Neuer Kamera-Hook `window.__cam(rotY, rotX,
  dist)` und `window.__snap()` fuer Headless-Review-Screenshots aus beliebigen
  Winkeln.
- **Hoehen-Fix**: Die Grasoberkante der Insel liegt bei y=+0.6 (Layer-0-Zylinder
  reicht bis +0.6). Teich, Teichrand, Brunnen, Ente, Wege, Platz und Blumen lagen
  auf y<=0.05 und waren unter dem Gras begraben (Teich war in keiner Ansicht
  sichtbar, der Brunnen zeigte nur seine Schale). Alle auf ~0.61 gehoben.
- Verifiziert per Headless-Screenshots aus 6 Winkeln (Tag + Nacht): Wege, Platz,
  Brunnen, Teich mit Wasserrad und Ente, Namensschilder, FEHLER-Karte auf der Tafel
  sichtbar; 0 JS-Fehler.

## M12 · Stufe 05c: Windmuehle, Teleskop, Garten, Voegel, Gluehwuermchen (2026-10-15)

- **Windmuehle** am Westrand mit vier drehenden Fluegeln (Landmarke).
- **Teleskop** auf Stativ vor der Research Observatory, dreht langsam.
- **Gemuesegarten** mit Zaun, Beeten und bunten Reihen westlich des Copy Studio.
- **Voegel**: drei Voegel kreisen hoch ueber dem Dorf mit Fluegelschlag.
- **Gluehwuermchen**: 16 leuchtende Punkte wandern naechtlich ueber die Wiese
  (Opazitaet folgt der Nacht). Sterne blieben bestehen.
- **Wegweiser** am Platz ("Teich", "Bibliothek") und Buecherstapel an der
  Zweitgehirn-Bibliothek.
- **Fix**: Der Stufe-05-Block hatte den Basis-Animations-Hook ueberschrieben; die
  Uhrturm-Zeiger und das Muehlrad standen still. Beide laufen wieder (Hook-Kette).
  Voegel verkleinert und hoeher gekreist, damit sie die Kamera nicht vollflaechig
  kreuzen; Teleskop steiler aufgestellt.
- Verifiziert Headless (Tag + Nacht, 6 Winkel): 0 JS-Fehler, dbg korrekt
  (Roboter 5, Pulse, 1 FEHLER-Karte, 1 Kueken), Teleskop/Windmuehle/Garten/Voegel/
  Gluehwuermchen in den Screenshots sichtbar.

## M13 · Stufe 05d: Leben im Dorf (2026-10-15)

- **Wolken**: sechs weiße Cluster ziehen langsam übers Dorf, nachts abgedunkelt.
- **Wasser-Wellen**: drei pulsierende Ringe auf dem Teich.
- **Lagerfeuer** am Platzrand: Steinkreis, Feuerholz, flackernde Flammen,
  nachts Lichtkegel und eigener Rauch.
- **Kamera-Autotour**: nach 20 Sekunden ohne Eingabe dreht die Kamera langsam
  weiter; jede Eingabe stoppt sie.
- **REA-Analyse** des Kanzie-Dorfs (REA 6.5.0, analyze-web-bundle über lokalen
  CDP-Chromium): Funde und Stufe-06-Kandidaten in REA-KANZIE-NOTIZEN.md
  (Souls-API /api/village/souls/bench + /graph, demo-config.json, changelog.json).
- Verifiziert Headless (Tag + Nacht, 7 Winkel): 0 JS-Fehler, dbg korrekt.

## M14 · Stufe 06: Souls-Schicht (2026-10-15, ~22:55)

- **Souls-Leuchten**: ueber jedem der 10 Gebaeude schwebt ein Status-Orb mit Ring
  (REA-Kanzie: /api/village/souls/bench + /graph als Vorbild, eigene visuelle
  Umsetzung, Demo weiter nur synthetischer Feed). Farben: blau pulsierend = Run
  laeuft, gruen = kuerzlich fertig (22 s), rot = fehlgeschlagen (45 s), warmes
  Beige = idle. Nachts leuchten die Orbs staerker (Opazitaet folgt __night()).
- **Wartebank (Souls-Bank)**: Holzbank am Platzrand beim Lagerfeuer; bei
  fehlgeschlagenen Runs warten bis zu 2 Roboterfiguren auf der Bank (Demo-Feed
  hat 1 failed Run -> 1 Roboter). Verschwinden, wenn keine Fails mehr da sind.
- **Versions-Chip + Changelog**: Chip "v1.0" unten rechts, Klick oeffnet ein
  Changelog-Panel (M10-M14, Kanzie-Muster changelog.json als eingebettete Daten).
- **Fix im eigenen Block**: V3 ist IIFE-lokal, Bank-Sitze nutzen jetzt
  new THREE.Vector3 (erster Review-Lauf fing "V3 is not defined" ab).
- Verifiziert Headless (SwiftShader, Tag ?time=10:00 + Nacht ?time=22:30, je
  4 Winkel, Dorf + Standalone): 0 JS-Fehler, dbg souls = [running x4, failed x1,
  idle x5] im Demo, benchRobots = 1, Changelog-Panel oeffnet per Klick.
  17 Screenshots unter /workspace/shots/rv10_*.
- Packets: hermes-nexus-stufe04-beta10.zip (README M14 + REA-KANZIE-NOTIZEN.md),
  village-standalone.html mit eingebettetem __DEMO_FEED + fetch-Stub.

## M15 · Stufe 07: Souls-Graph, Beziehungsfäden (2026-10-16, ~00:40)

- **Beziehungsfäden** (REA-Kanzie: /api/village/souls/graph als Beziehungs-Netz,
  eigene visuelle Umsetzung, Demo weiter nur synthetischer Feed): 10 leichte
  Bögen zwischen logisch verbundenen Gebäuden (Copy Studio <-> Media Haus,
  Research <-> Bibliothek, Cron <-> Digest/Web/Backup, Digest <-> Media,
  Media <-> Telegram/Kiosk, Bibliothek <-> Copy Studio). Tagsüber dezent,
  nachts heller (Opazitaet folgt __night()).
- **Aktiv-Puls**: Wenn ein Gebäude in einen aktiven Soul-Zustand wechselt
  (running/finished/failed), leuchten seine Fäden 9 s heller und hellerblau.
- **Klick-Karte erweitert**: Die Gebäude-Karte zeigt jetzt zusätzlich die Zeile
  "Soul" mit dem Zustand (ruht/arbeitet/fertig/Fehler) aus der Stufe-06-Schicht.
- **Ketten-Disziplin**: Eigene __feedObserver- und __extraTick-Ketten (prevObs/
  prevTick), bestehende Stufen unangetastet.
- Verifiziert Headless (SwiftShader, Tag ?time=10:00 + Nacht ?time=22:30, je
  3 Winkel, Dorf + Standalone): 0 JS-Fehler, dbg threads=10, threadsActive im
  Demo-Feed > 0 durch laufende Runs, Klick auf Cron Gatehouse zeigt Soul-Zeile
  ("ruht"), Screenshots unter /workspace/shots/rv11_*.
- Packets: hermes-nexus-stufe04-beta11.zip, village-standalone.html mit
  __DEMO_FEED + fetch-Stub neu gebaut.

## M16 · Stufe 08: Dorf-Statistik-HUD (2026-10-16, ~02:50)

- **Live-Statistik-Karte links oben** (REA-Kanzie: Live-Stats-Dashboard): aktive
  Läufe (blaue Zahl), Erfolgsquote ueber die letzten 50 Runs, belegte Haeuser
  (Fehler-Rot / sonst Gruen), letzte Aktivitaet mit relativer Zeit.
- Updates ueber die __extraTick-Kette alle 0,5 s, Daten aus dem Feed (runs)
  und der Souls-Schicht (window.__souls). Reines Anzeige-Element, kein
  Eingriff in Feed oder Collector.
- Verifiziert Headless (SwiftShader, Tag ?time=10:00 + Nacht ?time=22:30, Dorf
  + Standalone): 0 JS-Fehler, HUD rendert und zaehlt live (Standalone-Demo:
  4 aktive Laeufe, Erfolgsquote 0%, 1 Fehler-Haus, letzter Lauf vor 101 h),
  Screenshots unter /workspace/shots/rv12_*.
- Packets: hermes-nexus-stufe04-beta12.zip, village-standalone.html neu gebaut.

## M17 · Stufe 09: Wetter-System, Regenschauer (2026-10-16, ~04:40)

- **Regenschauer**: 1.500 Regentropfen (THREE.Points) fallen uebers Dorf,
  Schauer kommen und gehen (Zufallsphasen: 28-42 s Regen, 55-105 s trocken,
  sanfte Uebergaenge ~30 s). Bei Regen: Sonne -35%, Hemisphaerenlicht -25%,
  Himmel/Nebel Richtung Regengrau gelerpt.
- HUD (Stufe 08) zeigt jetzt eine Wetter-Zeile (Regen/Klar).
- Ketten-Disziplin: eigener __extraTick-Wrapper (prevTick), Lichter/Himmel
  werden pro Frame von der Tageszeit neu gesetzt, kein Aufstauen.
- Verifiziert Headless (SwiftShader): Regen erzwungen (window.__weatherSet),
  Tag + Nacht, Dorf + Standalone: 0 JS-Fehler, HUD zeigt "Regen",
  Rueckschalter trocknet wieder ab, Screenshots /workspace/shots/rv13_*
  (inkl. rain_v2, Regen sichtbar).
- Packets: hermes-nexus-stufe04-beta13.zip, village-standalone.html neu gebaut.

## M18 · Stufe 10: Minimap (2026-10-16, ~06:42)

- **Minimap unten links** (REA-Kanzie: Uebersicht mit Live-Zustaenden):
  Canvas-Karte der Dorf-Insel mit allen 10 Gebaeuden als Dots, Farbe folgt
  dem Soul-Zustand (blau laufend + pulsierender Ring, gruen fertig, rot
  Fehler, beige ruhend), Kira als gelber Punkt.
- Update ueber die __extraTick-Kette alle 0,4 s. Fehler im ersten Review
  gefunden und gefixt: Skalierung war Division statt Multiplikation, Dots
  sammelten sich in der Mitte.
- Verifiziert Headless (SwiftShader): 0 JS-Fehler, Minimap rendert
  (Gebaeude-Dots sichtbar, Live-Zustaende),Screenshots /workspace/shots/rv14_*
  inkl. fullui_v2 mit sichtbarem HUD + Minimap (echter Seiten-Screenshot, da
  __snap nur die 3D-Szene aufnimmt).
- Packets: hermes-nexus-stufe04-beta14.zip, village-standalone.html neu gebaut.
## M19 · Stufe 11 / Beta 15 (16.10.2026 ~21:45, Phase 21:30)
Optik-Gap-Punkt 1: Wege organisch geschwungen. Die sechs geraden Speichen sind
ersetzt durch Quadratic-Bezier-Wege (pro Gebäude deterministischer Versatz,
sandfarben 0xc9ae7e, Breite 0.95), Segment für Segment entlang der Kurve gelegt,
mit kleinen Steinchen (0x9a988f) am Rand und fünf Querwegen zwischen
Nachbarwegen. Kopfsteinpflaster-Wege und Platz unverändert. Review rv17: Tag +
Nacht Headless, 0 JS-Fehler, dbg normal (robots 5, proto true).
## M20 · Stufe 12 / Beta 16 (16.10.2026 ~22:18, Phase 22:00)
Optik-Gap-Punkt 2: Bach + Wasserfall + Bruecke. Der bisher losgeloeste
Wasserfall-Box ist an die Suedish-Ost-Kante umgesetzt (17.9, -2.4, 10.55,
radial ausgerichtet) und wird jetzt vom Bach gespeist: geschwungener
Bachlauf vom Teich (12.6, 7.4) zur Kante, 14 Segmente Wasser (0x4a90c2)
mit dunklen Uferstreifen (0x6b4f33), Weisswasser-Lippe, Gischt-Ellipse
und 3 animierte Ripples im Meer (eigener __extraTick-Chain). Holzbruecke
quer zum Bach (Deck, zwei Rails, vier Pfosten, zwei Traeger, matWall-Holz).
Neuer Hook __wasser (stream/fall/bridge/segs). Changelog-Chip auf v1.2,
Eintraege Stufe 09/10 und 11 ergaenzt. Review rv18 (7 Winkel): Tag, Nacht
und 17:00-Nachmittagslicht, 0 JS-Fehler, dbg normal; Bach + Bruecke in
rv18_wasser_f/g klar sichtbar, Wasserfall-Sturz aus Standardwinkeln dezent
(Feinschliff-Kandidat: prominenter machen).

## M21 · Stufe 13 / Beta 17 (16.10.2026 ~23:12, Phase 23:00, Nachtruhe)
Optik-Gap-Punkt 3: Baumvielfalt. Herbstpalette (0xd9822b, 0xc2542a,
0xe8a33d, 0xb57a2e): jede dritte Kiefer herbstlich gefaerbt, runde
Kronenbaeume abwechselnd gruen/herbstlich, 8 neue runde Kronenbaeume
(Mix), 8 Herbstbuesche, 12 Felsen (IcosahedronGeometry, flat shading,
Grautoene 0x8d8d85/0x7a7a72/0x9a968c, verstreut + am Inselrand, keiner
auf Wegen), 3 Blumenflecken (Cluster aus 8 Blueten, rosa/gelb/lila) nahe
der Plaza-Wegflaechen. Neuer Hook __flora (autumn/rocks=12/bushes=8/
roundExtra=8/patches=3). village.html + standalone gepatcht, node --check
rc 0 beide. Review rv19 (Tag 10:00, Nacht 22:30, Abend 17:00 + Nahkamera):
0 JS-Fehler, dbg normal, __flora true; Herbstfarben, Felsen, Buesche und
Blumenflecken sichtbar, Vergleich gegen kanzie-referenz.png bestaetigt
Baumvielfalt auf Kanzie-Niveau. ZIP beta17 (398.6 kB, 6 Dateien), Push
e9bd522..? (Beta 17). Naechste Phase 00:00: Optik-Gap-Punkt 4 (Kamera
naeher, Insel fuellt den Rahmen).

## M22 · Stufe 14 / Beta 18 (16.10.2026 ~23:37, Phase 23:30, Nachtruhe)
Optik-Gap-Punkt 4: Kamera naeher. Standardkamera dist 24 -> 18.5 und
rotX 0.88 -> 0.8, die Insel fuellt jetzt den Rahmen wie in der
Kanzie-Referenz (Gebaeude deutlich groesster Bildanteil); Zoom-Minimum
bleibt 9, Nutzer kann per Mausrad weiterhin frei zoomen. village.html +
standalone gepatcht, node --check rc 0 beide. Review rv20 (17:00 +
22:30): 0 JS-Fehler, dbg normal (robots 5, proto true), Framing geprueft
gegen kanzie-referenz.png, Inselrahmen vergleichbar. ZIP beta18
(401 kB, 6 Dateien), Push 2447d37..? (Beta 18). Naechste Phase 00:00:
Optik-Gap-Punkt 5 (Mobiliar: Baenke, Picknick, Sonnenschirm, Laternen).

## M23 · Stufe 15 / Beta 19 (17.10.2026 ~00:06, Phase 00:00, Nachtruhe)
Optik-Gap-Punkt 5: Mobiliar. 3 neue Holzbaenke (Rueckenlehne, entlang der
Weg-Speichen), Picknick-Set am Teich-Weg (Tisch mit Balken-Gestell, 2
Hocker, Brotlaib), roter Sonnenschirm darueber, blauer Sonnenschirm am
Plaza-Rand mit 2 Hockern (Kegelform, DoubleSide), 6 neue Laternen
entlang der Speichen (insgesamt 12). Neuer Hook __mobiliar (benches=3/
picnic=true/parasols=2/lampsNew=6/stools=3). village.html + standalone
gepatcht, node --check rc 0 beide. Review rv21 (Tag 10:00, Nacht 22:30,
Nahaufnahme Plaza): 0 JS-Fehler, dbg normal (robots 5, proto true),
Mobiliar sichtbar und im Stil passend (Holz 0x8a6a48/0x6e5238), Vergleich
gegen kanzie-referenz.png bestaetigt Mobiliar-Dichte. ZIP beta19
(403 kB, 6 Dateien), Push 3303d54..? (Beta 19). Naechste Phase 00:30:
Optik-Gap-Punkt 6 (nachts Lichtdichte).


## M42 · Stufe 34 / Beta 38 (19.10.2026 ~20:40, Phase 20:30, Feinschliff 2)

**Vignette + Kamera-Nahaufnahme-Check.** Kanzie hat eine deutliche dunkle
Vignette an den Raendern; wir hatten einen flachen hellen Hintergrund. Eingriff:
#vig als fixed Overlay (radial-gradient, transparent 58% -> rgba(8,10,22,.45) 100%,
pointer-events none, z-index 4) direkt nach #bubble; dbg um vig erweitert.

**Kamera-Nahaufnahme:** rv44_fall (Nacht, __cam(1.04, 0.55, 15)) und
rv44_fall_day (Tag, __cam(1.15, 0.6, 13)): Der Dorf-Kern haelt Nahaufnahme -
Platz, Uhr-Turm, Marktstaende, Wege sauber, keine erkennbaren Z-Fighting- oder
Texturfehler. Wasserfall/Teich im Vollbild sichtbar; Bach, Holzbruecke und
Wasserfall existieren bereits (M36) und wirken im Detail unauffaellig.

**Review rv44 (Port 8955, Tag 14:00 / Nacht 22:30):** vig true Tag+Nacht, alle
Swaps intakt (props 3/1/2/4, domeSwap, paths 8, roofLit 7, flora3 12/8/3/12),
threads 10, keine echten JS-Fehler (7x 404 stream, Polling-Fallback).
rv44_day: Randvignette rauscht die Szene Richtung Kanzie-Mood, Inselmitte bleibt
hell. rv44_night: Vignette verstaerkt die warme Lichtdichte. rv44_vergleich.png
aktualisiert.

**Artefakte:** /workspace/hermes-village-beta38.zip. Standalone aktualisiert.
Naechste Phase 21:00: Feinschliff 3 (Rest-Differenzen aus rv44_vergleich), dann
Abschlussvorschlag, sobald der Vergleich Kanzie-Niveau belegt.


## M41 · Stufe 33 / Beta 37 (19.10.2026 ~20:15, Phase 20:00, Feinschliff 1)

**Gras-Mottling + Flora-Dichte 3.** Direkter Vergleich rv42_vergleich.png gegen
kanzie-referenz.png: groesste Rest-Differenzen sind die untexturierte, flache
Wiese (Kanzie hat Flecken und Dichte) und Luecken im Baumbestand. Zwei Eingriffe:
(1) flora3(): 12 unregelmaessige Gras-Ton-Patches (CircleGeometry, 4 Gruentoene,
skaliert/rotiert, y 0.612, receiveShadow), (2) dritte Flora-Welle: 12 Randring-
Baeume (Mix Kegel/rund, Gruen/Herbst, Wasserfall-Seite um (17,10) freigehalten),
8 Buesche im Innenbereich, 3 Blumenflecken. dbg um flora3 erweitert.

**Review rv43 (Port 8953, Tag 14:00 / Nacht 22:30):** flora3 {trees 12, bushes 8,
patches 3, motes 12} Tag+Nacht, alle Swaps intakt (props 3/1/2/4, domeSwap,
paths 8, roofLit 7), threads 10, keine echten JS-Fehler (8x 404 stream,
Polling-Fallback). Erster Screenshot-Lauf Timeout bei Fonts/Render, zweiter Lauf
mit erhoehtem Timeout sauber. rv43_day: Wiese wirkt gemustert statt flach,
Randring dichter, keine Kollisionen mit Wegen/Gebaeuden sichtbar. rv43_night:
Lichtdichte gehalten. rv43_vergleich.png aktualisiert.

**Artefakte:** /workspace/hermes-village-beta37.zip. Standalone aktualisiert.
Naechste Phasen: Feinschliff 2 (Nahaufnahme-/Kamera-Check gegen Referenz,
Wasserfall-/Bruecke-Detail), dann Abschlussvorschlag, sobald der Vergleich
Kanzie-Niveau belegt.


## M40 · Stufe 32 / Beta 36 (19.10.2026 ~19:45, Phase 19:30)

**GLB-Dachfarben etwas heller.** roofFix() traversiert die Szene und hellt
Materialien mit Name "Roof" oder "Cap" auf (offsetHSL 0, -0.02, +0.09, einmalig
per userData.roofLit markiert), per setInterval alle 500 ms bis 40 Versuche, damit
asynchrone GLB-Swaps sicher abgedeckt sind; window.__roofFix und window.__roofLit
als Zähler. Getroffen: 7 Materialien (House, Media, Mill, Mine, Studio, Tele,
Webhotel-Cap); Gate (Stein) und Dome (Kuppel) bleiben bewusst unberuehrt. dbg um
roofLit erweitert.

**Review rv42 (Port 8951, Tag 14:00 / Nacht 22:30):** roofLit 7 Tag+Nacht, alle
Swaps intakt (props 3/1/2/4, domeSwap, paths 8, alle Gebaeude), threads 10, keine
echten JS-Fehler (6x 404 /api/village/stream, Polling-Fallback im Review).
rv42_day: Daecher wirken etwas heller und lesslicher gegen das Gruen, Silhouetten
bleiben lesbar. rv42_night: Lichtdichte gehalten. rv42_vergleich.png aktualisiert.

**Artefakte:** /workspace/hermes-village-beta36.zip. Standalone aktualisiert.
Damit sind alle OPTIK-GAP-Punkte (1-7) mindestens einmal angegangen; nächste
Phasen: Feinschliff (Details aus dem Vergleich), dann Abschlussvorschlag an
Konstantin, sobald der Vergleich Kanzie-Niveau belegt.


## M39 · Stufe 31 / Beta 35 (19.10.2026 ~19:25, Phase 19:00)

**Weg-Textur organisch (Kopfsteinpflaster auf Sandbaendern).** Die 8 Weg-Speichen
bekommen unter die Kopfsteine geschwungene Sandbaender: pathRibbon() baut pro Weg
einen 2n-Streifen aus Quads mit Sand-Canvas-Textur (4 Farbtupfer, 900 Speckles),
leichter S-Kurve (curv = sin(t*1.6*pi + phase)*0.7), Wellen-Kante (wob =
sin(t*2.2*pi + phase)*0.55), laufender Breite 0.9-1.4, Normalen manuell auf
(0,1,0), receiveShadow. Farbton 0xe8dab4 auf 0xd8c8a4-Basis, UV-Wiederholung
entlang des Wegs. Baender liegen bei y 0.015 unter den Kopfsteinen (0.035), sodass
beide Schichten sichtbar bleiben. dbg um paths (8) erweitert.

**Review rv41 (Port 8949, Tag 14:00 / Nacht 22:30):** paths 8 Tag+Nacht, alle
Swaps intakt (props 3/1/2/4, domeSwap, alle Gebaeude), threads 10, keine echten
JS-Fehler (nur 7x 404 /api/village/stream, Polling-Fallback im Review ohne
Backend). rv41_day: Wege lesen sich von oben jetzt als sandige Pfade mit
unregelmaessiger Kante statt Beton-Speichen. rv41_night: Lichtdichte gehalten,
Baender dunkel aber sichtbar. rv41_vergleich.png aktualisiert.

**Artefakte:** /workspace/hermes-village-beta35.zip. Standalone aktualisiert.
Rest-Gaps: GLB-Dachfarben etwas heller, dann Feinschliff. (Bemerkung: /tmp wurde
zwischen den Phasen gewischt, Review-Script neu aufgebaut, /tmp/hvd-Klon neu.)


## M38 · Stufe 30 / Beta 34 (19.10.2026 ~18:45, Phase 18:30)

**Research Observatory als echtes Blender-Modell + mehr Laternen.** assets/build_dome.py (bpy 5.0.1): 10-seitiger Turmkorpus (Stein #d8d3c8), Gesimsring creme, Halbkugel-Kuppel (Skalierung 0.78, blaugrau #8b93a8-Anmutung), Kuppel-Basisring, Holztür mit Rahmen, 2 Seitenfenster, Teleskop-Ansatz schräg aus der Kuppel, Plinthe. ac_dome.glb 57936 B. Zwei Build-Fehler gefixt: box() hatte Höhe/Tiefe vertauscht (Bleche statt Wände), Render-Kamera nutzte falsches Track-Achsen-Paar ('-Y','Z' → '-Z','Y'), deshalb zunächst nur Boden im Preview.

**Einbau:** DOME_B64 + GLTFLoader.parse → window.__domeGL; applyDome() versteckt do_base/do_dome/do_door und hängt blenderdome in die dome-Gruppe; der Code-Slit (emissive) und die Lampe bleiben für die Nachtleucht-Logik erhalten (worldLamps.win unangetastet). applyDome wird sowohl aus dem Loader-Callback als auch aus dome() gerufen (Reihenfolge-unabhängig, kein TDZ). Erster Versuch scheiterte: window.__domeGroup-Zuweisung landete hinter return (toter Code) → Probe-Script zeigte group:false, Fix vor return.

**Laternen:** 6 neue lampPost-Aufrufe im Mobiliar-Block (Gesamt 12, lampsNew 6→12).

**Review rv40 (Port 8946/8948, Tag 14:00 / Nacht 22:30):** domeSwap true + blenderdome-Knoten vorhanden, alle Swaps intakt (props 3/1/2/4, alle Gebäude), chicks 1, threads 10, 0 echte JS-Fehler. Zoom rv40_dome_day/night: Kuppel sitzt sauber auf dem Turm, nachts Lampen + Kuppelspalt sichtbar. rv40_vergleich.png aktualisiert.

**Artefakte:** /workspace/hermes-village-beta34.zip. Standalone aktualisiert. Rest-Gaps: Weg-Textur organisch, GLB-Dachfarben etwas heller; danach Feinschliff.


## M37 · Stufe 29 / Beta 33 (19.10.2026 ~18:25, Phase 18:00)

**Mobiliar als echte Blender-Modelle (Bank, Picknick-Tisch, Sonnenschirm, Hocker).** assets/build_props.py (bpy 5.0.1, LD_LIBRARY_PATH-Debian-Libs) baut vier Sitz-/Möbel-Modelle mit M_WOOD (#8a6a47, 0.54 roughness), M_DARKWOOD, M_CANOPY (#d94f3d) und M_MAST: Bank mit Sitzfläche, Rückenlehne, 4 Beinen und Armlehnen; Picknick-Tisch mit Deckel und 2 Seitenbänken; Sonnenschirm mit Mast, 8-Segment-Kegel-Kanopy und Knauf; Hocker als Zylinder-Stumpf. Export: ac_bench.glb 9648 B, ac_table.glb 8004 B, ac_umbrella.glb 13032 B, ac_stool.glb 2972 B; Render-Preview prev_props.png geprüft.

**Einbau in village.html:** PROPS_B64-Block (4 Base64-GLBs) + GLTFLoader.parse → Prototypen benchProto/tableProto/umbProto/stoolProto; trySwapProps() ersetzt in der mobiliar-IIFE die Code-Primitiven: 3 Bänke (woodBench), Picknick-Tisch (tbl), beide Sonnenschirme (Canopy-Farbe bleibt rot/blau via Materialname "Canopy"), 3 Hocker. Die Gruppen behalten Position/Rotation/col; Ursprung der GLBs ist Objektmitte am Boden. window.__mobG hält die Gruppenreferenzen; dbg erweitert um props {bench,table,umbrella,stool}.

**Fixes:** Patch-Anker beim ersten Lauf zu früh gesetzt (Props-Block mitten im robot-Load) → SyntaxError "Unexpected token 'const'"; village.html aus Beta-32-Stand (/tmp/hvd) wiederhergestellt und mit vollem Statement-Anker neu gepatcht. build_props.py: primitive_cylinder_add radius1 in bpy 5.0.1 unzulässig → cone-Hack (radius1=radius2).

**Review rv39 (Port 8945, Tag 14:00 / Nacht 22:30):** props {bench:3, table:1, umbrella:2, stool:4} je Struktur, alle früheren Swaps intakt (acHouse/acMill/acStudio/acMedia/acMine/acGate je 1, houseSwap/gateSwap true), chicks 1, threads 10, 0 echte JS-Fehler (nur 2× erwartete 404 /api/village/stream, SSE-frei). Zoom-Shots rv39_picnic_day/night: roter Schirm über Picknick-Tisch, Nachts Lampen an. rv39_vergleich.png (Referenz 1024×640 vs rv39_day) angelegt.

**Artefakte:** /workspace/hermes-village-beta33.zip (village.html 1.07 MB + README + OPTIK-GAP + REA-Notizen + GLBs + Preview). Standalone aktualisiert. Nächste Phasen (Rest-Gaps): Research-Observatory-Kuppel als GLB, Weg-Textur organisch, GLB-Dachfarben etwas heller, mehr Laternen im Dorfverteiler; danach Feinschliff.
## M36 · Stufe 28 / Beta 32 (19.10.2026 ~17:55, Phase 17:30)

**Telegram-Zentrale als echtes Blender-Modell.** Der prozedurale Boathouse (Kisten-Haus am Teich) ist ersetzt durch `ac_tele.glb` (57.7 kB, Blender 5.0.1 headless): Sockel-Plinthe 2.3×2.1, blaugrauer Korpus 2.0×1.8×1.4 (0x7a94b8 ≙ Original-Farbwelt), Creme-Trim-Kranz, 4-seitiges Kegeldach 0x3f6fb5 mit Überhang, Ziegel-Schornstein mit Kappe, dunkle Fenster-Schlitze, Holz-Türrahmen an der +Z-Seite. Bevels 0.04/2 (Dach 0.03/2).

**Bau-Fehler gefunden und behoben:** erster Export stand kopfüber im Bach-Rahmen (Höhe/Tiefe im Build-Skript vertauscht, Blender Z=oben) und das Kegeldach war um die falsche Achse gedreht (Y statt Z). Beides via Headless-Preview + Szenen-Zoom-Review sichtbar gemacht und korrigiert.

**Einbau:** HOUSE_B64 in `feed/village.html` durch das neue GLB ersetzt (Patch-Skript), prozedurale Tür + Lampe (0x8ec8ff) bleiben erhalten, `hm.rotation.y = PI` richtet den Türrahmen zur Szene. Syntax-Check OK, Standalone aktualisiert.

**Neuer Debug-Hook:** `window.__look(x,y,z)` verschiebt den Orbit-Mittelpunkt (neben `__cam`), damit Gebäude ab jetzt gezielt im Headless-Review anzoomt werden können.

**Review rv38 (Headless, Tag + Nacht, Port 8937):** alle Swaps true (houseSwap/studioSwap/mhSwap/mineSwap/gateSwap), acHouse/acMill/acStudio/acMedia/acMine/acGate je 1, 0 echte JS-Fehler (die zwei 404s sind nur `/api/village/stream`, SSE-frei im Demo-Modus). Zoom-Shots Tag + Nacht an der Teichstelle: Haus steht korrekt, Türseite zeigt zum Steg, nachts leuchtet die Lampe auf die Wand.

**Artefakte:** `rv38_vergleich.png` (Kanzie-Referenz links, rv38 Tag rechts), `rv38_day_zoom.png` / `rv38_night_zoom.png`. ZIP Beta 32 (village-standalone.html, README.md, REA-KANZIE-NOTIZEN.md, OPTIK-GAP-KANZIE.md, feed/village.html, feed/events-synthetic.json).

**Nächste Phasen:** Research-Observatory-Kuppel als Blender-Modell (sieht schon gut aus, aber fehlt in der GLB-Familie), Weg-Textur, GLB-Dachfarben leicht hellen, Mobiliar (Bänke, Picknick-Tisch, Sonnenschirm, mehr Laternen). Danach Feinschliff, dann Abschlussvorschlag, sobald der Screenshot-Vergleich Kanzie-Niveau belegt.

## M35 · Stufe 27 / Beta 31 (19.10.2026 ~17:12, Phase 17:00, Cron Gatehouse als Blender-Modell)

Cron Gatehouse (Uhrturm, 1.2 / -6.5) ist jetzt ein echtes Blender-GLB; Ziffernblatt, Zeiger, Leuchtkugel und Lampe bleiben prozedural (die Zeiger laufen animiert ueber window.__clock).

**Modell** (`assets/build_gate.py`): Turmschaft 10-seitig, unten 1.25 oben 1.05, Hoehe 5.6 (Stein #9aa0ac), Kranz-Gesims unter dem Dach, Sockel-Plinthe (1.6 unten), 8-seitiges Kegeldach #9aa0ac mit Bevel, Eingang an der Front (dunkle Tuer #14100c + Holzrahmen: 2 Pfosten + Sturz #6e5238), zwei dunkle Fenster-Schlitze an den Flanken. bpy 5.0.1: primitive_cylinder_add kennt kein radius1/radius2 mehr, Kegel-Primitiven nutzen. Export: `assets/ac_gate.glb` (20.3 kB), Preview `assets/prev_gate.png` (CYCLES).

**Einbau**: Prozedurale Primitive in clockTower() benannt (`gt_base` Koerper, `gt_roof` Kegeldach); `swapGate()` versteckt nur `gt_*`-Kinder und haellt Ziffernblatt (emissiv), Rand, Zeiger (animiert), Leuchtkugel und Lampe unberuehrt. `window.__gateSwap = gate`; `GATE_B64` + `GLTFLoader.parse` → `gateProto`; dbg um `acGate` + `gateSwap` erweitert.

**Review** (Headless, Port 8936, ?time=14:00 und 22:30): dbg Tag + Nacht: acGate 1, acMine 1, acMedia 1, acStudio 1, acHouse 1, acMill 1, alle Swaps true, 0 JS-Fehler. rv37_day/night + Vergleich rv37_vergleich.png gegen kanzie-referenz.png: Turm sitzt an der Plaza-Nordseite, nachts leuchtet das Ziffernblatt weiter an alter Stelle, Dorf intakt. Rest-Gaps: Telegram-Zentrale + Research-Observatory-Kuppel (Kuppel sieht schon gut aus), Weg-Textur von oben, GLB-Dachfarben etwas heller als Primitive-Vorgaenger, Mobiliar (Baenke, Picknick, Sonnenschirm).

## M34 · Stufe 26 / Beta 30 (19.10.2026 ~16:40, Phase 16:30, Backup Mine als Blender-Modell)

Backup Mine (2, 8) ist jetzt ein echtes Blender-GLB mit Stollen und Gleisen statt Code-Primitiven.

**Modell** (`assets/build_mine.py`): Huettenkoerper 3.6×3.2×2.3 in Sand #cbb99a, 4-seitiges Pyramidendach mit Ueberhang in Ziegel #8a4a3a, Stollen-Eingang an der Front (dunkler Kasten #14100c) mit Holzrahmen (zwei Stuetzpfosten + Sturz, #6e5238), Seitentuern-Holzblende #5a3c28, zwei Stuetzbalken unter dem Dach, Gleise Richtung Inselmitte (zwei Stahl-Schienen #777777, 4 Holz-Schwellen). Bevels 0.05/2 auf Struktur, Schienen/Schwellen ohne. Export: `assets/ac_mine.glb` (23.1 kB), Preview `assets/prev_mine.png` (CYCLES, Front-Ansicht mit Gleisen).

**Einbau**: `window.__mineSwap = mine` an `mineBuild(2, 8)`; `MINE_B64` + `GLTFLoader.parse` → `mineProto`; `swapMine()` versteckt alle nicht-emissiven Top-Level-Meshes der Gruppe (Koerper, Dach, Tuer, Kamin, Stollen-Loch, Schienen, Schwellen) und setzt das GLB ein. Die Lore (`window.__cart`, animiert, Fahrt 4.4 ± 1.6) bleibt unberuehrt und faehrt weiter auf den GLB-Gleisen; das emissive Fenster der Huette leuchtet nachts weiter. dbg um `acMine` + `mineSwap` erweitert.

**Review** (Headless, Port 8936, ?time=14:00 und 22:30): dbg Tag + Nacht: acMine 1, acMedia 1, acStudio 1, acHouse 1, acMill 1, mineSwap/mhSwap/studioSwap true, 0 JS-Fehler. rv36_day/night + Vergleich rv36_vergleich.png gegen kanzie-referenz.png: Dorf intakt, Mine sitzt an ihrem Platz am Kreuzungsweg; Close-up-Search durchs KIRA-Dialog-Overlay verdeckt, Befund stuetzt sich auf dbg + Ganzszenen. Rest-Gaps: Cron Gatehouse + Telegram-Zentrale als GLB, Weg-Textur von oben, Dachfarben der GLB-Modelle etwas heller als die Primitive-Vorgaenger.

## M33 · Stufe 25 / Beta 29 (19.10.2026 ~16:10, Phase 16:00, Shimeria Media Haus als Blender-Modell)

Shimeria Media Haus (lila 3-Etage, -10.5/3.5) ist jetzt ein echtes Blender-GLB statt Code-Klötze.

**Modell** (`assets/build_media.py`, 1 Durchlauf): drei Etagen 3.6×1.15×2.8 (Lila #b070a8) mit Creme-Trims (#f2e8dc) auf jeder Etagen-Kante, Pyramidendach 4-seitig (Radius 2.66, Höhe 1.0) in Violett #6a4fa3, Bevels (0.04, 2 Segmente). Export: `assets/ac_media.glb` (11.2 kB), Preview `assets/prev_media.png` (CYCLES).

**Einbau**: Primitiven in `hotel()` benannt (`mh_body`, `mh_trim`, `mh_roof`), `MEDIA_B64` + `GLTFLoader.parse` → `mediaProto`; `swapMedia()` versteckt die `mh_*`-Kinder und setzt das GLB ein. Das prozedurale Neon-Schild (Canvas-Textur "SHIMERIA HAUS", emissiveMap) und die emissiven Fenster bleiben sichtbar und leuchten nachts weiter; `chimTop` unverändert. `window.__mhSwap = media`; dbg um `acMedia` + `mhSwap` erweitert.

**Review** (Headless, Port 8936, ?time=14:00 und 22:30): dbg Tag + Nacht: acMedia 1, acStudio 1, acHouse 1, acMill 1, mhSwap true, studioSwap true, 0 JS-Fehler. rv35_day/night angesehen, Vergleich rv35_vergleich.png gegen kanzie-referenz.png: Media-Haus wirkt weicher (Trims + Bevel-Dach), Schild leuchtet nacht weiter an der gleichen Stelle. Rest-Gaps: verbleibende Primitive-Bauten (Observatorium ok als Kuppel, Backup Mine, Cron Gatehouse), Weg-Textur von oben, Farblicht der großen Dächer (GLB-Dach etwas heller als Primitive-Vorgänger).

## M32 · Stufe 24 / Beta 28 (19.10.2026 ~15:35, Phase 15:30, Copy Studio als Blender-Modell)

Copy Studio (blauer Pyramiden-Dach-Bau, -5.5/-3) ist jetzt ein echtes Blender-GLB statt Code-Klötze.

**Modell** (`assets/build_studio.py`, 1 Durchlauf): Körper 3.4×2.7×3.4 (Creme #e8d5ae, wie bisher), Pyramiden-Dach zweiteilig (Kranz + Spitze) mit Überhang in Blau #3f6fb5, Tür in Holz #5a3c28 an der Front (+Z), Kamin in Ziegel #9a4f3d auf der Rückseite mit Abdeckkappe, Bevels (0.05, 2 Segmente) für weiche AC-Kanten. Export: `assets/ac_studio.glb` (10.1 kB), Preview `assets/prev_studio.png` (CYCLES).

**Einbau**: `STUDIO_B64` + `GLTFLoader.parse` → `studioProto`; `swapStudio()` versteckt alle nicht-emissiven Primitiven der studio-Gruppe (Körper, Dach, Tür, Kamin) und setzt das GLB ein. Die emissiven Fenster (winA/B/C) bleiben prozedural und leuchten nachts weiter, `chimTop` unverändert (Rauch kommt weiter aus der Kaminposition). `window.__studioSwap = studio` nach der Erzeugung; dbg um `acStudio` + `studioSwap` erweitert.

**Review** (Headless, Port 8935, ?time=14:00 und 22:30): dbg Tag + Nacht: acStudio 1, acMill 1, acHouse 1, studioSwap true, houseSwap true, 0 JS-Fehler. Screenshots rv34_day.png / rv34_night.png angesehen, Vergleich rv34_vergleich.png gegen kanzie-referenz.png: Studio-Dach wirkt weicher, Bevel sichtbar; nachts Lichtdichte gehalten. Rest-Gaps: große Flachdächer (Media Haus, Bibliothek) und der Beton-Look der Wege von oben; Baum-/Buschdichte inzwischen nah am Original.

## M31 · Stufe 23 / Beta 27 (19.10.2026 ~15:20, Phase 15:00, Flora-Dichte + Kamera naeher)
Zweite Flora-Welle (flora2-IIFE nach Stufe 13): +12 Baeume (Mix
Kegel/rund, gruen + Herbstfarben, Rand- und Zwischenraeume), +8
Buesche an Gebaeude-Raendern, +4 Felsen, +3 Blumenflecken
(window.__flora2 als dbg). Kamera naeher: dist 38 -> 30, rotX
0.5 -> 0.46, Insel fuellt den Rahmen deutlich besser. Wege
schwingen staerker: bulge 0.16 -> 0.22 plus Sinus-Anteil. Checks:
node --check rc 0; Headless-Review rv33 Tag + Nacht: dbg acHouse 1,
acMill 1, houseSwap true, proto true, 0 JS-Fehler; Vergleichsbild
rv33_vergleich.png (kanzie-referenz + Tag + Nacht) angesehen:
Framing deutlich naeher an der Referenz, Rest-Gaps bleiben
Gebaeude (noch Primitive-Bloecke, naechste Blender-Phase) und
Grund-Textur. Korrektur zu M30: village-standalone.html wird seit
Beta 26 als Kopie von feed/village.html gefuehrt, der Baum-Swap
ist dort also vorhanden (M30-Notiz zur Luchtecke hinfaellig).
ZIP beta27 (488 kB-Klasse), Push auf main.

## M30 · Stufe 22 / Beta 26 (19.10.2026 ~14:45, Phase 14:30, Blender-Modelle Web Hotel + Digest Mill)
Zwei weitere Code-Primitiven durch echte Blender-Modelle ersetzt
(build_ac_buildings.py, 3 Iterationen: Dome-Roofs via cut_below
abgeflacht, Bevels reduziert, Schornstein auf Kaminseite +Y, keine
Front-Fenster). AC-Web Hotel: rosa Waende #e8b0b8, Rosen-Dach
#d17888, Stelzen #8a6a4a, Schornstein mit Kappe, Seitenfenster mit
Rahmen. Digest Mill: cremefarbene Steinbasis #e8d9b8, gruenes
Dach #3f7f5c, Kamin, Windrad-artige Flügel-Andeutung, Wasserrad-
Körper (Animation window.__wheel bleibt an der Cottage-Stelle,
Rad-Look jetzt aus dem GLB). Export: assets/ac_webhotel.glb
(20.9 kB), assets/ac_mill.glb (49.7 kB), Previews
prev_webhotel.png / prev_mill.png. Einbau: WEBHOUSE_B64 /
MILL_B64 als Base64, GLTFLoader.parse + swapWebhouse()/swapMill()
tauschen die Primitive (wh_body/wh_roof/wh_leg bzw. mm_*),
Primitiven benannt und entfernt, prozedurale Fenster bleiben für
Glow; window.__whSwap/__millSwap für Checks, dbg erweitert
(acHouse, acMill). village-standalone.html gleich gepatcht.
Pruefungen: node --check rc 0 beide Dateien; Headless-Review
rv32 (Tag + Nacht, 1280x800): dbg acHouse 1, acMill 1,
houseSwap true, proto true, 0 echte JS-Fehler; Close-ups
rv32_close_a/b/c bestaetigen beide Modelle im Dorf sichtbar
(Web Hotel links auf Stelzen mit rosa Dach, Mill am Teich mit
gruenem Dach). Vergleichsbild rv32_vergleich.png (Referenz +
Tag + Nacht): Richtung stimmt, Rest-Gap bleibt Dichte/Kamera
(Phase M31: Wege organisch + Baeume vielfaeltiger + Kamera).
Standalone-Lücke: Baum-Swap (AC_TREE_B64/upgradeTrees) fehlt
dort noch, nächste Phase nachholen. ZIP beta26.

## M29 · Stufe 21 / Beta 25 (19.10.2026 ~14:08, Phase 13:30, erster Blender-Build)
Konstantin hat entschieden (13:30): Weiterbau mit echten Blender-Modellen
statt Code-Primitiven. bpy 5.0.1 headless im Workspace installiert
(fe produziert libGL/libglx .debs, entpackt nach /workspace/deps,
LD_LIBRARY_PATH=/workspace/deps/usr/lib/x86_64-linux-gnu). Erstes
Modell: hermes-nexus/models/boathouse.glb (17 kB, handgebautes Haus:
Waende #7a94b8, Satteldach #34528a mit First und Ueberstand, Sockel,
Fenster beidseitig, Schornstein mit Kappe, Bevels) via bpy-Skript
/tmp/make_boathouse.py exportiert. Einbau: GLB als Base64 in
village-standalone.html + feed/village.html eingebettet (HOUSE_B64),
GLTFLoader.parse tauscht im Boathouse ("Telegram Zentrale") body+roof
gegen den GLB-Klon (prozedurale Tuer bleibt fuer Lampen-/Fenster-Glow),
rotation.y = PI (Tuer zur Steg-Seite), eigenes DoorFront im GLB
ausgeblendet. dbg erweitert (house, houseSwap). Pruefungen: node
--check rc 0 beide Dateien; dbg-Check: house true, houseSwap true,
robots 5, proto true, 0 echte JS-Fehler (nur erwarteter file://-CORS-
Stream-Hinweis); Review rv34: Tag + Nacht Ganzansicht, isolierte
Modellansicht (rv34_model_iso.png) sauber, Crop am Teich zeigt
Satteldach-Kante statt Pyramide. Naechste Phase 14:00: weitere
Gebaeude auf Blender-Modelle umstellen + Feincheck der Ausrichtung
im Einsatz.

## M28 · Stufe 20 / Beta 24 (17.10.2026 ~02:43, Phase 02:30, Nachtruhe)
Kamera-Feinjustierung auf Basis belegter Rest-Gaps aus M27
(Teich-Ausrichtung rechts oben statt links, Insel zu gross im
Rahmen). __pond-Hook-Sweep ueber yaw -0.8/-1.0/-1.2/-1.6/-2.0 bei
dist 38, rotX 0.5: Teich-NDC wandert von x 0.471 (yaw -0.8) nach
x 0.046 (yaw -2.0), pondVisible ueberall true. Screenshots
rv30_yaw* angesehen: yaw -2.0 beste Referenzkomposition (ganze
Insel, dunkler Abendhimmel, Wolken, Teich oben mitte). Finale
Standardkamera: rotY -2.0, rotX 0.5, dist 38. village.html +
standalone gepatcht, node --check rc 0 beide. Review rv31
(Daemmerung 18:45, Tag 10:00, Nacht 22:30): 0 JS-Fehler, dbg
normal (robots 5, proto true), pond ndc x 0.044 visible true.
vergleich_final.png neu (Referenz vs. finale Kamera). Rest-Detail:
Yaw der Referenz weicht ab (Referenz-Teich links mitte), optisch
klein. Alle Optik-Gap-Punkte + Feinschliff + Kamera abgeschlossen.

## M27 · Stufe 19 / Beta 23 (17.10.2026 ~02:13, Phase 02:00, Nachtruhe)
Kamera-Feinschliff auf Referenzkomposition (Phase 02:00, 4 iterierte
Vergleiche gegen kanzie-referenz.png): Standardkamera rotY 0.7 -> -1.2
(Teich-/Wasserfallseite nach vorn), rotX 0.8 -> 0.5 (niedriger,
cinematischer Winkel), dist 18.5 -> 36 (Insel mit Abendhimmel
drumrum, wie Referenz). Iterationen: rv26 (rotX 0.58/dist 20, noch
zu steil), rv27 (0.52/25, Rahmen laeuft voll), rv28 (0.5/30, besser,
Kante fehlt), rv29 (0.5/36, beste Uebereinstimmung: dunkler Himmel
um die Insel, warme Lichter, Teich + Berge sichtbar). village.html +
standalone gepatcht, node --check rc 0 beide. Review rv29 Tag+Nacht+
Daemmerung: 0 JS-Fehler, dbg normal (robots 5, proto true).
vergleich_final.png neu gebaut (Referenz vs. iter.4). Rest-Abweichungen,
dokumentiert: Insel noch etwas groesser im Rahmen als in der Referenz,
Erdquerschnitt-Kante unten teilweise angeschnitten, Teich-Ausrichtung
rechts oben statt links mitte (Yaw-Detail). Einstufung: Kamera-Gap
weitgehend geschlossen, Rest feinjustierbar.

## M26 · Stufe 18 / Beta 22 (17.10.2026 ~01:42, Phase 01:30, Nachtruhe)
Feinschliff-Runde nach allen 7 Optik-Gap-Punkten. Lagerfeuer-Umgebung:
3 Sitz-Stuempfe (Zylinder 0.15/0.34, Holz 0x8a6a48, Top 0xb08d5e)
+ 4 zusaetzliche Steine am Feuer-Ring (Radius 0.62). village.html +
standalone gepatcht, node --check rc 0 beide. Review rv24 (Feuer
Tag/Nacht, Wasserfall-Tiefwinkel rotY -1.2/-1.4, pitch 0.15-0.35,
dist 30): 0 JS-Fehler, dbg normal (robots 5, proto true). WICHTIG:
Wasserfall-Sturz jetzt sichtbar bestaetigt - aus tiefem Aussenwinkel
zeigt er den Sturz ueber die Kante mit Schaum-Ringen am Fuss
(1523 Fall-Pixel im Scan). Abschlussvergleich rv25/vergleich_final.png
(Kanzie-Referenz 18:46 vs. Build 18:45): Stimmung, Lichter, Wege,
Wasser, Baeume, Mobiliar auf Referenzniveau; Rest-Gap: cinematischere
Kamerafuehrung (niedrigerer Winkel, Bloom) und nochere
Gebaeudedichte in der Referenz. ALLE 7 Optik-Gap-Punkte + Feinschliff
abgearbeitet. Abschlussvorschlag an Konstantin vorbereitet, Auslieferung
zur Phase 07:00 (Nachtruhe-Regel).

## M25 · Stufe 17 / Beta 21 (17.10.2026 ~01:28, Phase 01:00, Nachtruhe)
Optik-Gap-Punkt 7: Wasser- und Teich-Feinschliff. Teich organisch
gestaucht (Ellipse 1.18 x 0.88) plus Bucht nach Suedwesten, 9
Ufersteine, 3 Schilf-Buschel, 4 Seerosenblaetter mit 2 Blueten;
Wasserfall prominenter (Breite x1.45, heller 0xd6f0ff, 4
Schaum-Streifen, Splash x1.35, zusaetzlicher Schaum-Ring an der
Basis). Neuer permanenter Debug-Hook __pond (Position, Projektion,
Verdecker-Check). village.html + standalone gepatcht, node --check rc 0
beide. Review rv23f/h (Tag rotY -1.2/-1.4 + Nacht 22:30): 0 JS-Fehler,
dbg normal (robots 5, proto true), __wasser polish true, Teich mit
Bucht, Ufersteinen, Seerosen und Schilf sichtbar, nachts warm
ausgeleuchtet. Fall-Sturz aus Ueberwasser-Winkeln unter der
Inselkante nicht einnehmbar (Geometrie, nicht Fehler) - Aenderung
code-verifiziert. Vergleich kanzie-referenz bestaetigt. ZIP beta21,
Push ba29a5a..? (Beta 21). Alle 7 Optik-Gap-Punkte abgearbeitet;
naechste Phase 01:30: Feinschliff-Runde (Wasserfall-Sturz-Sichtbarkeit
per Kamera-Tiefstand pruefen, Restdetails), dann Abschlussvorschlag
an Konstantin (nach 07:00).

## M24 · Stufe 16 / Beta 20 (17.10.2026 ~00:39, Phase 00:30, Nachtruhe)
Optik-Gap-Punkt 6: nachts Lichtdichte. Lichtpool unter jeder der 12
Laternen (CircleGeometry 1.7, AdditiveBlending, warm 0xffc860, Opacity
mit lampOn 0 bis 0.3), Lampen-PointLight 7 -> 8.5 Reichweite und
Intensitaet nachts 1.8 -> 2.1, Lagerfeuer-Licht flackert jetzt mit 2.1
statt 1.6. village.html + standalone gepatcht, node --check rc 0 beide.
Review rv22 (Nacht 22:30, Lagerfeuer-Nahaufnahme, Tag 10:00 als
Regression): 0 JS-Fehler, dbg normal (robots 5, proto true), Lichtpools
auf den Wegen sichtbar, Nacht deutlich waermer/dichter, Vergleich gegen
kanzie-referenz.png bestaetigt. ZIP beta20 (404 kB, 6 Dateien), Push
ba29a5a..? (Beta 20). Naechste Phase 01:00: Optik-Gap-Punkt 7
(Wasser/Teich-Form, Feinschliff), danach resterender Feinschliff +
Abschlussvorschlag.

## Stufe 04 - Datenvertrag als TypeScript (contract/)
- `contract/village-snapshot.ts` - VillageSnapshot, drei Ebenen als
  nominal getrennte Typen; Verstoese gegen die Trennung sind Compilerfehler
  (Beweisdatei `contract/layer-rules.test-d.ts`, @ts-expect-error).
  "Unbekannt bleibt unbekannt": fehlende Felder degradieren nach
  'unknown'/null, nur der Feed-Kopf ist strikt.
- `contract/snapshot-provider.ts` - Provider mit Live/Mock-Umschaltung;
  das Quellen-Badge liest `meta.source` aus dem Feed, nicht den Modus.
- `contract/dist/` - kompilierte ES-Module fuer village.html.
- `contract/smoke.test.mjs` - laeuft gegen die echten Feeds in `feed/`.
- `contract/REVIEW-LOOP.md` - Pruefungsreihenfolge + Performance-Budget.
- `contract/INTEGRATION.md` - Einbindung in village.html ohne Bundler.
Pruefungen: tsc --noEmit (0 Fehler, Pflicht) + `node contract/smoke.test.mjs`
(alle gruen, Pflicht) vor jeder Uebergabe.


## M43 · Stufe 35 / Beta 39 (19.10.2026 ~21:25, Phase 21:00, Feinschliff 3)

**Kamera auf Kanzie-Winkel + Berge im Hintergrund.** Der Vergleich rv44
gegen kanzie-referenz.png zeigte die groesste Rest-Differenz: Kanzie sieht
die Insel deutlich seitlicher (Insel fuellt den Rahmen, Horizont sichtbar),
wir von steil oben. Eingriffe in feed/village.html:
1. Kamera-Default rotX 0.46 -> 0.95, dist 30 -> 28 (Wheel-Clamps bleiben
   0.12..1.25 / 9..60). Jetzt seitenaehnliche Sicht wie Kanzie, Insel
   fuellt den Rahmen, Inselrand und Meer sichtbar.
2. Berge im Hintergrund neu (IIFE mountains(), dbg __mountains=3):
   3 Schneekegel hinter der Insel (Richtung +x/+z, hinter dem Dorf),
   h 12/10/9, Basis auf dem Meeresboden (y -3), Koerper 0x93a0a8 hell,
   Schnee 0xeef3f7. Erste Versuche (h 19-26) waren zu dominant bzw.
   unsichtbar (steile Kamera) -> iterativ verkleinert/versetzt.
3. Wolken hoeher gesetzt (y 14-16 -> 20-24), sonst haengen sie bei der
   flacheren Kamera vor dem Dorf.
Review rv49 (Port 8967): Tag 14:00 + Nacht 22:30, mountains 3, alle Swaps
intakt (props 3/1/2/4, domeSwap, paths 8, roofLit 7, flora3 12/8/3/12,
vig true), threads 10, 0 echte JS-Fehler (8x 404 stream). rv49_day/night
angesehen; rv49_vergleich.png (Kanzie | Tag | Nacht): Kamerablick jetzt
klar naeher an Kanzie, Berge + Horizont vorhanden. village-standalone.html
1156784 Zeichen. ZIP beta39, Push (Beta 39).

## M44 · Stufe 36 / Beta 40 (19.10.2026 ~21:45, Phase 21:30, Feinschliff 4: Himmel & Leitungen)

**Netz-Look weicher (groesster Rest-Differenz aus rv49_vergleich: die schwarzen
Leitungen machten das Dorf zum Diagramm).** Drei Eingriffe in feed/village.html:
1. Gebaeude-Router-Draehte: Farbe 0x22262e -> 0x39424c, Opacity 0.7 -> 0.26
   (Pulse laufen weiter auf denselben Kurven).
2. Masten-Draehte (sagWire): transparent, Opacity 0.6 statt deckend.
3. Wolken: Opacity 0.92 -> 0.5, Farbe 0xffffff -> 0xf4f8fb, Kugel-Radius
   0.9/0.35-Staffel -> 0.7/0.28-Staffel; Berge gedampft (body 0x93a0a8 ->
   0x89949c, Schnee 0xeef3f7 -> 0xe6edf3), Groesse/Position unveraendert.
Neue dbg-Felder: clouds 6, wireSoft true. node --check rc 0, Standalone sync.

**Review rv50 (Headless, Port 8971, Tag 14:00 + Nacht 22:30):** alle Swaps
intakt (house/studio/mh/mine/gate/dome/props 3/1/2/4), threads 10, paths 8,
roofLit 7, flora3 12/8/3/12, vig true, mountains 3, clouds 6, wireSoft true,
0 echte JS-Fehler (2x erwartete 404 /api/village/stream). rv50_day/night
angesehen, rv50_vergleich.png (Kanzie | Tag | Nacht): Leitungen lesen sich
jetzt als dezente Faeden statt Netzplan, Himmel ruhiger, Berge treten zurueck.

**Artefakte:** /workspace/hermes-village-beta40.zip. Push (Beta 40).
Naechste Phase 22:00: Feinschliff 5 (Rest-Differenzen aus rv50_vergleich:
Gebaeudedichte/Fassadenfarben), danach Abschlussvorschlag.
