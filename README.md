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
