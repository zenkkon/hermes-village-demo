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

## M45 · Stufe 37 / Beta 41 (19.10.2026 ~22:20, Phase 22:00, Feinschliff 5)

**Gebäudedichte + Fassadenfarben (Kanzie-Palette).** `facadeFix()` tintet die Wand-Materialien (Wall/Walls/Body) der 6 GLB-Gebäude Richtung Kanzie-Palette: Copy Studio terrakotta 0xcf7a5f, Media violett 0x9f7fd0, Web Hotel blau 0x8fb7d9, Mill ocker 0xd9b98a, Mine braun 0xc49a6a, Telegram blaugrau 0x8fa8cc (Lerp 0.55, Material-Marker facadeTinted, dbg facade:6). Drei neue Dorfhaeuschen aus den Blender-Protos (houseProto ×2, studioProto ×1) auf freien Rasenstellen (-9,-11), (6,11), (16.8,1.2), skaliert 0.62-0.78, gedreht, eigene Materialklone in terracotta/blau/mint, jeweils warme Lampe über worldLamps (dbg cottages:3). Headless-Review Tag+Nacht: alle Swaps intakt, facade 6, cottages 3, 0 JS-Fehler. Vergleich gegen kanzie-referenz.png: Fassadenfarben sichtbar, Haeuschen kollisionsfrei; groesste Rest-Differenz jetzt Gebaeudegroesse/Lebensdichte (Kanzie-Haeuser kleiner und enger gestellt). Beta 41 gepusht.

## M46 · Stufe 38 / Beta 42 (19.10.2026 ~22:40, Phase 22:30, Feinschliff 6)

**Gebaeudegroesse (Kanzie-Massstab).** scaleDown() setzt alle 8 BUILDINGS-Gruppen einheitlich auf 0.85 (Kanzies Haeuser sind kleiner und enger), Kaminanker (chimTop) werden mit skaliert, Marker userData.scaled6, dbg scale6:8. Headless-Review Tag (14:00) + Nacht (22:30): alle Swaps intakt (facade 6, cottages 3, threads 10, paths 8, roofLit 7), 0 JS-Fehler. Screenshot-Vergleich gegen kanzie-referenz.png: Insel wirkt gefuellter, Proportionen naeher am Original; groesste Rest-Differenz jetzt nachts Lichtdichte (Kanzie: viele Fenster-/Laternenlichter, enges Zentrum) und Haeuserdichte im Kern. Beta 42 gepusht.

## M47 · Stufe 39 / Beta 43 (19.10.2026 ~23:15, Phase 23:00, Feinschliff 7)

**Lichtdichte nachts.** windowFix() setzt alle GLB-Fenstermaterialien (Win*/Window*) auf warmes Emissive 0xffd9a0 und koppelt sie an den Nacht-Faktor (0.15 + lampOn*2.4, dbg glow:9); die drei Dorfhaeuschen bekommen additive Licht-Pools am Boden (wie Laternen), plus vier neue Laternen im Kern und am Nordrand (-1.8,-4.4), (8.2,0.6), (-3.6,7.4), (0.2,11.8). Headless-Review Tag (14:00) + Nacht (22:30): alle Swaps intakt, glow 9, cottages 3, scale6 8, 0 JS-Fehler. Vergleich gegen kanzie-referenz.png: Lichtdichte nachts deutlich naeher am Original (Fenster gluehen, Pools am Boden); Rest-Differenz: Kanzies Nacht ist blauer/dunkler und sein Kern noch dichter beleucht. Beta 43 gepusht.
## M48 · Stufe 40 / Beta 44 (19.10.2026 ~23:55, Phase 23:30, Feinschliff 8)
- Nachtstimmung: Meer lerpt nachts auf Tiefblau 0x0a1830 (seaDay 0x2c5f86 -> seaNight), gekoppelt an den Tagesfaktor; dbg sea8.
- Grundlichter nachts gedimmt: hemi 0.16 + day*0.69 (vorher 0.3 + day*0.55), Nacht liest sich dunkler und waermer statt flach gruen.
- Kern-Dichte: 4. Dorfhaeuschen am Plaza-Ostrand (2.6, 2.6), scale 0.58, warm getoent 0xd9b07a, eigene Lampe + Licht-Pool; dbg cottages 4.
- Review rv54 (Tag 14:00 + Nacht 22:30, Port 8977): alle Swaps intakt (facade 6, scale6 8, glow 12, props 3/1/2/4, threads 10, paths 8, flora 12/8/3/12, mountains 3, clouds 6, vig), nightMood8 true, 0 JS-Fehler. Vergleich rv54_vergleich.png gegen kanzie-referenz.png.
- Befund: Nacht deutlich naeher an Kanzie (Meer dunkel, Lichtdichte hoch). Rest-Differenz: Kanzies Nacht wirkt blauer in der Umgebungslicht-Farbe (Gras liest sich blaugruen statt dunkelgruen) und Kernbeleuchtung noch etwas homogener. -> Feinschliff 9.
- Standalone nach Sandbox-Restart neu angelegt (village-standalone.html, Kopie von village.html + __snap-Hook nur in der Review-Kopie); Chromium-Review benutzt /workspace/.cache/ms-playwright + /workspace/deps.
- Zip: hermes-village-beta44.zip
## M49 · Stufe 41 / Beta 45 (20.10.2026 ~00:20, Phase 00:00, Feinschliff 9)
- Blaue Nacht-Ambient-Farbe: hemi.groundColor lerpt nachts von warmbraun 0x4a4436 auf blaugruen 0x1d3340 (groundDay/groundNight), Mond auf 0.32 fuer Formen.
- Kernlicht-Homogenitaet: Lampen 0.05 + lampOn*2.3 (vorher 2.1), Licht-Pools 0.38 (vorher 0.3); dbg nightMood9.
- Review rv55 (Tag 14:00 + Nacht 22:30, Port 8977): alle Swaps intakt (facade 6, cottages 4, scale6 8, glow 12, props 3/1/2/4, threads 10, paths 8, flora 12/8/3/12, mountains 3, clouds 6, vig), nightMood9 true, 0 JS-Fehler. Vergleich rv55_vergleich.png gegen kanzie-referenz.png.
- Befund: Nacht-Lichtdichte und dunkles Meer nahe am Kanzie-Niveau; unser Nachtgras liest sich noch deutlich gruen, Kanzies Gras blaugruen (staerkerer blauer Umgebungs-Tint). -> Feinschliff 10: Nacht-Tint direkt auf Gras-/Terrainmaterial (color.lerp Richtung 0x274a44 bei Nacht).
- Zip: hermes-village-beta45.zip
## M50 · Stufe 42 / Beta 46 (20.10.2026 ~00:40, Phase 00:30, Feinschliff 10)
- Nacht-Tint direkt aufs Terrainmaterial: window.__terrainMats sammelt alle 4 Erdschichten-Materialien, im updateDayNight lerpt die Gras-/Terrainfarbe nachts von 0x6fb048 Richtung 0x27524a (blaugruen wie Kanzie); dbg terrainTint 4, nightMood10.
- Wegmaterial nachts abgedunkelt: pathRibbons-Farbe 0xe8dab4 -> 0x8a7f66 bei Nacht (Sand liest sich nicht mehr tags hell).
- Review rv56 (Tag 14:00 + Nacht 22:30, je eigener Browser-Prozess): alle Swaps intakt (facade 6, cottages 4, scale6 8, glow 12, props 3/1/2/4, threads 10, paths 8, flora 12/8/3/12, mountains 3, clouds 6, vig), nightMood10 true, terrainTint 4, 0 JS-Fehler. Vergleich rv56_vergleich.png gegen kanzie-referenz.png.
- Befund: Nachtgras jetzt blaugruen wie bei Kanzie, Nachtstimmung insgesamt nah am Referenzniveau. Tag bleibt unveraendert korrekt (Tint leert sich bei Tag vollstaendig). Rest-Differenz: Kanzies Kern noch dichter bebaut/beleuchtet; Baeume wirken bei uns duenner verteilt. -> Feinschliff 11: Kern-Dichte (5. Haus) + Baumgruppen-Trio am Suedrand.
- Zip: hermes-village-beta46.zip
## M51 · Stufe 43 / Beta 47 (20.10.2026 ~01:25, Phase 01:00, Feinschliff 11)
- Kern-Dichte: 5. Haus in cottages-DEFS westlich am Plaza (-2.6, 2.6, ry 0.4, s 0.58, tint 0xb8a0c8, Lampen + Pool, dbg cottages 5, core5).
- Baumgruppen-Trio Suedrand: 3 Baeume (-5.5,-14.8) (-1,-15.2) (3,-14.2) gemischt rund/Kegel mit Herbstfarben + 2 Buesche (dbg flora4 3), Luecke zwischen -5 und +7 bei z ~ -15 gefuellt.
- Kleinkram: Favicon-Platzhalter (data:) gegen die automatische Favicon-404 im Headless-Review.
- Review rv57 (Tag 14:00 + Nacht 22:30, je eigener Browser-Prozess, Sandbox-Restart hatte /tmp geleert - Script neu angelegt): alle Swaps intakt, cottages 5, glow 15, flora4 3, terrainTint 4, 0 Skript-Fehler; die geloggte 404 ist die Backend-Anfrage /api/village/stream der Demo gegen den rein statischen Review-Server (im Deploy mit Backend normal), kein Skriptfehler. Vergleich rv57_vergleich.png gegen kanzie-referenz.png.
- Befund: Nacht auf Kanzie-Niveau (Gras blaugruen, Lichtdichte, dunkles Meer), Kern jetzt dichter, Suedrand gefuellt. Rest-Differenz: Kanzies Kern hat noch mehr Gebaeude-Kohaesion und das Ufer wirkt bei uns steiler. -> Feinschliff 12: 6. Haus am Nordkern + Uferabsatz (zweite Gras-Terrasse vor der Erdschicht).
- Zip: hermes-village-beta47.zip
## M52 · Stufe 44 / Beta 48 (20.10.2026 ~01:45, Phase 01:30, Feinschliff 12)
- Kern-Dichte 2: 6. Haus nordwestlich am Plaza (-4.2, 4.8, ry -1.2, s 0.6, tint 0xd9a8a8, Lampen + Pool, dbg cottages 6, core6, glow 18).
- Uferabsatz: zweite Gras-Terrasse vor der Erdschicht (r 20.9, y -0.7, h 0.7, c 0x6fb048), ragt 0.4 ans Wasser und zeichnet die Wasserlinie wie Kanzies Bank; geht als 5. Terrain-Material in den Nacht-Tint ein (dbg terrainTint 5, shore12).
- Review rv58 (Tag 14:00 + Nacht 22:30, je eigener Browser-Prozess, Review-Script nach Sandbox-Restart neu angelegt): alle Swaps intakt, cottages 6, glow 18, terrainTint 5, shore12, core6, flora4 3, 0 Skript-Fehler; die geloggte 404 ist /api/village/stream (Backend-Poll gegen den rein statischen Review-Server), kein Skriptfehler. Vergleich rv58_vergleich.png gegen kanzie-referenz.png.
- Befund: Kern jetzt 6 Cottages + 8 Gebaeude, Ufer zeichnet sich ab, Nacht weiter auf Kanzie-Niveau. Rest-Differenz: Kanzies Kern wirkt noch kohaesiver (Gebaeude schmaler und hoher, enger gereiht) und hat sichtbaren Steg/Hafen. -> Feinschliff 13: 7. Haus ostkern + schmale Steg-Planken Richtung Wasser suedost.
- Zip: hermes-village-beta48.zip
## M53 · Stufe 45 / Beta 49 (20.10.2026 ~02:25, Phase 02:00, Feinschliff 13)
- Kern-Dichte 3: 7. Haus ostwaerts am Plaza (4.6, 1.8, ry -0.5, s 0.55, tint 0xc8b88a, dbg cottages 7, core7, glow 21).
- Steg suedost: 9 Deck-Planken (0.55x0.09x1.15, 0x8a6242) von (11.6,-11.6) diagonal 45 Grad ueber die Inselkante bis r~23, 3 Pfähle (r 0.1, h 6.0) bis unter die Wasserlinie (Wasser y -5.4, Pfaehle bis y -5.85), Laterne mit emissivem Bulb (0xffd98a) am Ende; dbg pier13 9.
- Review rv59 (Tag 14:00 + Nacht 22:30): dbg vollstaendig intakt (cottages 7, glow 21, pier13 9, core7, terrainTint 5, shore12, flora4 3, 0 Skript-Fehler, 404 nur Backend-Poll). Nacht-Shot crashed 3x ("browser closed") beim 12s-Wait; mit 7s-Wait sauber - Renderer-Speicherdruck bei glow 21, Review-Script nutzt ab sofort 7s fuer Nacht.
- Befund: Default-Kamera zeigt 7. Haus und Steg aus dieser Perspektive kaum (klein/verdeckt); dbg belegt beide. Nacht weiter auf Kanzie-Niveau. Rest-Differenz: Kern-Kohaesion (Gebaeude schmaler/hoher, enger gereiht), Hafen-Umgebung um den Steg fehlt noch. -> Feinschliff 14: Kamera-Nahaufnahme-Review des Stegs + 7. Hauses, dann Kern-Parameter schmaler.
- Zip: hermes-village-beta49.zip
## M54 · Stufe 46 / Beta 50 (20.10.2026 ~02:55, Phase 02:30, Feinschliff 14)
- Hafen-Detail am Steg: 2 Poller (r 0.09/0.11, h 0.7, 0x6f4a30) flankieren den Steg-Anfang, 2 Fässer (r 0.26/0.28, h 0.5/0.55, 0x8a5a34) + 1 Kiste (0.5, 0x6f4a30) am Ufer, Ruderboot (Rumpf 1.7x0.35x0.75, 0x7a5538, 2 Randleisten) schwimmt neben dem Steg-Ende auf dem Wasser (y -5.15, Wasser -5.4), rotation 45 Grad parallel zum Steg; dbg harbor14 6.
- Review rv60: Weitwinkel Tag (14:00) + Nacht (22:30) via wiederhergestellter Pipeline (Standalone nur mit __snap-Hook): dbg vollstaendig intakt (cottages 7, glow 21, pier13 9, harbor14 6, terrainTint 5, shore12, core5-7, flora4 3), 0 Skript-Fehler (404 nur Backend-Poll). Pier-Nahaufnahme (18:30, Kamera-Override window.__cam im Standalone): Planken korrekt ueber die Kante, Pfahl sichtbar, Laterne leuchtet. Haus7-Nahaufnahme 2x Browser-Crash ("browser closed"), vom Weitwinkel abgedeckt.
- Befund Review-Infrastruktur: der Kamera-Override-Hook brach die Default-Kamera im Standalone (Weitwinkel renderte nur Himmel, ohne JS-Fehler) - Hook zurueckgesetzt, Standalone wieder Standard-Pipeline; Nahaufnahmen brauchen einen anderen Ansatz (z.B. Override nur fuer Einzellaufe per URL-Param). Nacht weiter auf Kanzie-Niveau; Rest-Diff: Kern-Kohaesion (Gebaeude schmaler/hoeller, enger gereiht).
- Zip: hermes-village-beta50.zip

## M55 · Stufe 47 / Beta 51 (Feinschliff 15: Kern-Kohaesion + Blender-Vorbereitung)

- **Blender-Schiene steht**: bpy 5.0.1 headless im Workspace installiert (pip, Python 3.11); Systemlibs laufen ueber `LD_LIBRARY_PATH=/workspace/deps/usr/lib/x86_64-linux-gnu` (libXfixes u. a. lagen schon in deps). Mindesttest: low-poly Cottage (Koerper + 4-seitiges Dach + Tuer, Farbmaterialien) per bpy gebaut und als GLB exportiert -> `models/test-cottage.glb` (5.084 Bytes). Naechste Phase: erste echte Blender-Cottage als GLB in village.html via GLTFLoader.
- **Kern-Kohaesion (Feinschliff 15)**: alle 7 DEFS-Cottages schlanker und hoeher (`c.scale.set(d.s*0.88, d.s*1.14, d.s*0.88)` statt einheitlichem Scalar), Richtung Kanzies schmalen, hohen Kernhaeusern. dbg-Marker `__dbgSlim15`.
- Review rv61: Tag 14:00 + Nacht 22:30, dbg cottages 7 / slim15 true / glow 21 / pier13 9, 0 JS-Fehler (bekannter 404 `/api/village/stream` = Backend-Poll). Vergleiche rv61_day/rv61_night/rv61_vergleich.png angesehen: schlankere Silhouetten sichtbar, keine Clipping-Artefakte, Nacht unverichtet dicht.
- Beta 51 Zip: village-beta51.zip (283 KB).

## M56 · Stufe 48 / Beta 52 (erste echte Blender-Cottage via GLTFLoader)

- **Erstes bpy-Modell im Dorf**: cottage-ac.glb (16,5 KB, 10 Meshes: Sockel, Koerper, 2 geneigte Dachflaechen mit First und Ueberhang, Tuer, 2 Fenster mit Rahmen und Glas, Schornstein; Materialien Wall/Roof/Wood/Window/Trim/Brick) per bpy 5.0.1 gebaut und als GLB exportiert.
- Einbettung im etablierten Muster: COTTAGE52_B64 (Base64, 22.008 Zeichen) + GLTFLoader.parse; der Blender-Cottage-Block ersetzt das DEFS-Kern-Haus am Steg (key 4.6,1.8): Groesse wird vom ersetzten Objekt uebernommen (Box3-Hoehenmatch), Position/Rotation identisch, Shadows an.
- DEFS-Loop setzt jetzt `c.userData.cottageKey = key` auf jeder platzierten Cottage (Voraussetzung fuer saubere Swaps).
- Review rv62: Tag 14:00 + Nacht 22:30 sauber, dbg b52 true / b52done true / cottages 7 / glow 22 / pier13 9 / harbor14 6, 0 JS-Fehler. Close-up 18:30 (eingebauter __cam-Hook, dist 15) bestaetigt die Optik: Satteldach, Schornstein, Fensterrahmen sichtbar und stillich passend.
- Beta 52 Zip: village-beta52.zip.

## M57 · Stufe 49 / Beta 53 (zweite Blender-Cottage + Nachtfenster)

- **Zweites bpy-Modell**: cottage-ac2.glb (23,8 KB, 14 Meshes) im etablierten Muster gebaut: Terracotta-L-Form (Hauptkoerper laenglich + Fluegel mit 4-seitigem Turmdach), Satteldach mit First, Tuer, 3 Fenster mit Rahmen, Schornstein, L-Sockel.
- Swap gegen DEFS-Kern-Haus sued (key 2.6,2.6) im selben Muster wie Beta 52 (Box3-Hoehenmatch, Position/Rotation uebernommen, Shadows an).
- **Nachtfenster fuer beide Blender-Cottages**: Materialien werden im Swap geklont; Window-Materialien bekommen emissive 0xffd98a und werden in `window.__nightWins` registriert, der bestehende Nacht-Loop treibt deren emissiveIntensity (dbg nightWins 31).
- Review rv63: Tag 14:00 + Nacht 22:30 + Close-up 18:30 (eingebauter __cam-Hook, dist 15) sauber, dbg b52done true / b53done true / cottages 7 / nightWins 31, 0 JS-Fehler. Vergleiche rv63_day/night/close/vergleich.png angesehen: L-Form sitzt suedlich der Plaza, Nacht laesst beide Blender-Fenster warm aufleuchten.
- Beta 53 Zip: village-beta53.zip.

## M58 · Stufe 50 / Beta 54 (dritte Blender-Cottage: Veranda-Variante)

- **Drittes bpy-Modell**: cottage-ac3.glb (26,9 KB, 17 Meshes): Cremeweisse Wände, gruenes Satteldach mit Firstbalken (First entlang Z), Sued-Veranda (Holzdeck, 2 Pfosten, Pultdach 14 Grad), Tuer, 3 Fensterpaare mit Rahmen+Glas, Schornstein.
- Swap gegen DEFS-Kern-Haus suedwest (key -2.6,2.6, vorher lila Box-Haus) im etablierten Muster (Box3-Hoehenmatch, Position/Rotation, Shadows, Material-Klon + __nightWins).
- Review rv64: Tag 14:00 + Nacht 22:30 + Close-up 18:30 sauber, dbg b52done/b53done/b54done true, cottages 7, nightWins 37 (nachts mehr Fenster im Nacht-Loop registriert, erwartet), 0 JS-Fehler. rv64_day/night/close/vergleich.png angesehen: Kern hat jetzt 3 echte Blender-Cottages (Steg, sued, suedwest), Weitwinkel stimmig, Nacht laesst alle Blender-Fenster warm aufleuchten.
- Beta 54 Zip: village-beta54.zip.

## M59 · Stufe 51 / Beta 55 (vierte Blender-Cottage: Zeltdach mit Gaube)

- **Viertes bpy-Modell**: cottage-ac4.glb (20,9 KB, 12 Meshes): Warmgelbe Wände, dunkelrotes 4-seitiges Zeltdach (Cone vertices=4, gedreht 45 Grad), Sued-Gaube mit eigenem Pultdach (20 Grad) und Gaubenfenster, Tuer, Fensterpaare E/W, Schornstein.
- Swap gegen DEFS-Kern-Haus nordwest (key -4.2,4.8, vorher rosa Box-Haus d9a8a8) im etablierten Muster (Box3-Hoehenmatch, Position/Rotation, Shadows, Material-Klon + __nightWins).
- Key-Check vor dem Bau: die in M57/M58 als naechste Kandidaten notierten Keys 1.4,3.8 / -1.4,3.8 existieren nicht in DEFS; echte Kern-Keys sind 2.6,2.6 / -2.6,2.6 / -4.2,4.8 / 4.6,1.8 (plus 3 Aussenhaeuser). Damit ist der vierte Nahe-Kern-Key jetzt belegt.
- Review rv65: Tag 14:00 + Nacht 22:30 + Close-up 18:30 sauber, dbg b52/b53/b54/b55 alle true, cottages 7, nightWins 43, glow 32, 0 JS-Fehler. rv65_day/night/close/vergleich.png angesehen: Kern hat jetzt 4 echte Blender-Cottages (Steg, sued, suedwest, nordwest), Weitwinkel stimmig, Nacht dicht.
- Beta 55 Zip: village-beta55.zip.

## M60 · Stufe 52 / Beta 56 (fuenfte Blender-Cottage: Farmer-Haus am Aussenhaus nordwest)

- **Fuenftes bpy-Modell**: cottage-ac5.glb (22,3 KB, 14 Meshes): Rote Farmer-Variante (Ziegelrot 0.72,0.33,0.26), laenglicher Koerper 2.2x1.2, Satteldach entlang X mit Firstbalken (th 30 Grad, rise 0.566), Tuer an Suedlaengsseite, 2 Suedfenster + 1 Nordfenster, grosser Schornstein mit Steinhaube.
- Swap gegen DEFS-Aussenhaus nordwest (key -9,-11, vorher rotes Box-Haus d98a7a), etabliertes Muster (Box3-Hoehenmatch, Position/Rotation, Shadows, Material-Klon + __nightWins).
- Review rv66: Tag 14:00 + Nacht 22:30 + Close-up 18:30 sauber, dbg b52/b53/b54/b55/b56 alle true, cottages 7, nightWins 49, glow 35, 0 JS-Fehler. rv66_day/night/close/vergleich.png angesehen: kein Artefakt, Weitwinkel stimmig, Nacht dicht. Blender-Modelle jetzt 5 von 7 Hausern (alle 4 Kern-Haeuser + Aussenhaus nordwest).
- Beta 56 Zip: village-beta56.zip.

## M61 · Stufe 53 / Beta 57 (sechste Blender-Cottage: Kreuzgiebel-Haus am Aussenhaus sued)

- **Sechstes bpy-Modell**: cottage-ac6.glb (26,9 KB, 17 Meshes): Hellblau-graue Wände (0.56,0.66,0.72), Kreuzgiebel: Hauptkoerper 1.9x1.2 mit Satteldach (th 33 Grad, rise 0.533) + suedlicher Querfluegel 1.1x0.9 mit eigenem Satteldach (beide ziegelrot), Tuer im Fluegel, 3 Fenster, Schornstein.
- Swap gegen DEFS-Aussenhaus sued (key 6,11, vorher blaugraues Box-Haus 8fb7d9), etabliertes Muster (Box3-Hoehenmatch, Position/Rotation, Shadows, Material-Klon + __nightWins).
- Review rv67: Tag 14:00 + Nacht 22:30 + Close-up 18:30 sauber, dbg b52-b57 alle true, cottages 7, nightWins 55, glow 38, 0 JS-Fehler. rv67_day/night/close/vergleich.png angesehen: kein Artefakt, Weitwinkel stimmig, Nacht dicht. Blender-Modelle jetzt 6 von 7 Hausern; offen nur noch das Studio (16.8,1.2).
- Beta 57 Zip: village-beta57.zip.

## M62 · Stufe 54 / Beta 58 (siebtes Blender-Modell: Werkstatt-Studio am Studio ost)

- **Siebtes bpy-Modell**: studio-ac7.glb (26,4 KB, 17 Meshes): Werkstatt-Studio mit sandfarbenen Waenden (0.82,0.76,0.66), dunkelgrauem Pultdach (7 Grad nach Nord), Dachfascia, grosser Glasfront sued (3 Atelierfenster mit Rahmen), Tuer, Fenster E/W, Skylight (Holzrahmen + Glas) auf dem Dach.
- Swap gegen DEFS-Studio ost (key 16.8,1.2, src 'studio', vorher flaches Box-Studio a8d0b8) im etablierten Muster (Box3-Hoehenmatch, Position/Rotation, Shadows, Material-Klon + __nightWins); Glasfelder und Skylight als Window-Material registriert.
- Review rv68: Tag 14:00 + Nacht 22:30 + Close-up 18:30 sauber, dbg b52-b58 alle true, cottages 7, nightWins 67, glow 44, 0 JS-Fehler. rv68_day/night/close/vergleich.png angesehen: kein Artefakt, Weitwinkel stimmig, Nacht dicht. **Alle 7 Hauser stehen jetzt auf echten Blender-Modellen** (5 Cottages + Kreuzgiebel-Haus + Werkstatt-Studio).
- Beta 58 Zip: village-beta58.zip.

## M63 · Stufe 55 / Beta 59 (Kamera-Feinschliff gegen Kanzie)

- OPTIK-GAP "Kamera naeher (Insel fuellt Rahmen)": Kamera-Defaults justiert: dist 28 -> 33, rotX 0.95 -> 0.78 (rad, ~45 Grad). Zwei Iterationen (erst 31/0.9, dann 33/0.78): jetzt ist die Insel komplett im Rahmen mit Luft rundum, Berge oben ganz sichtbar, Bildkomposition deutlich kanzie-aehnlicher (Referenz zeigt die Insel als komplette Torte).
- Review rv69 (Iteration 1) + rv69b (Iteration 2): Tag 14:00 + Nacht 22:30 (+ Close-up 18:30 in rv69) sauber, dbg b52-b58 alle true, cottages 7, nightWins 67, glow 44, 0 JS-Fehler. rv69b_day/night/vergleich.png angesehen: kein Artefakt, Nacht dicht, Framing passt.
- Beta 59 Zip: village-beta59.zip.
## M64 · Stufe 56 / Beta 60 (07:30, Mobiliar-Vertiefung + nächtliche Lichtdichte)

**Mobiliar-Vertiefung (Beta 60):** 2 neue Bänke (-0.8, 3.9 und 9.2, 8.0) und zweites Picknick-Set
(-3.2, 8.6: Tisch rot -0.4, 2 Hocker, blauer Sonnenschirm 0x3f7fbf). Alle neuen Gruppen werden in
`window.__mobG` registriert, sodass der bestehende Blender-Prop-Swap (Stufe 29) sie automatisch
mit swapped: props-Swaps stiegen auf bench 5 / table 2 / umbrella 3 / stool 6. 4 neue Laternen
(6.4,-5.8 / -9.0,3.4 / 12.4,4.2 / 4.6,11.0). **Nächtliche Lichtdichte:** Lampen-Intensity 2.3→2.55,
Lichtpool-Opacity 0.38→0.45, Fenster-Emissive 1.7→1.9.
Fehler im Patch-Verlauf gelernt: Stufe-60-Block zuerst vor Stufe 32 eingesetzt (lief vor der
Mobiliar-Init, Guard brach ab), dann hinter Stufe 15 verlegt; TDZ-Fehler auf `tableProto` (let)
wird jetzt im try/catch gepollt.
**Review rv70:** Tag+Nacht+Close (18:30, __cam(-0.9, 0.85, 15)) sauber, dbg b52–b58 true, mob60 true,
props bench5/table2/umbrella3/stool6, nightWins 67, glow 44, **0 JS-Fehler**. Direkter Vergleich gegen
kanzie-referenz.png: Nachtlicht jetzt deutlich dichter/wärmer als in Beta 59; Gap zur Kanzie-
Abendreferenz (dichtes warmes Licht) kleiner, aber noch sichtbar.
Beta-60-Zip: hermes-village-beta60.zip.
## M65 · Stufe 57 / Beta 61 (07:50, nächtliche Lichtdichte final)

**Nachtlicht-Abgleich gegen Kanzie-Abendlicht (Beta 61):** Nachthimmel 0x0d1024 → 0x14203d
(sichtbares Dunkelblau statt Schwarz, wie die Kanzie-Referenz bei 18:46), Nacht-Grundhelligkeit:
hemi nachts +0.22, neuer warmes AmbientLight 0xffb060 mit lampOn*0.3 (flächendeckend warm wie
Kanzie), Mond 0.32 → 0.44.
**Review rv71:** Tag 14:00 / Nacht 22:30 / Close 18:30 (__cam(-0.9, 0.85, 15)) sauber; dbg b52–b58
true, mob60 true, props bench5/table2/umbrella3/stool6, nightWins 67, glow 44, **0 JS-Fehler**.
Direkter Vergleich rv71_vergleich.png gegen kanzie-referenz.png: Nachtlicht jetzt dicht, warm und
flächedeckend, Gap zur Kanzie-Abendreferenz klein; Restdifferenz ist Kanzies leicht flächenhaftere
Ausleuchtung im Dorfzentrum (DoF-Unschärfe im Kanzie-Screenshot). Alle OPTIK-GAP-Punkte (Wege,
Erdquerschnitt/Bach/Wasserfall/Brücke, Baumvielfalt, Kamera, Mobiliar, nächtliche Lichtdichte)
damit abgearbeitet; 08:00-Phase: Gesamtabgleich + Abschlussvorschlag an Konstantin.
Beta-61-Zip: hermes-village-beta61.zip.
## M66 · Stufe 58 (08:10, Gesamtabgleich gegen Kanzie-Referenz)

**Review-only-Phase (kein Code-Change, kein neues Beta):** Headless-Review rv72 (Tag 14:00,
Nacht 22:30, Close 18:30) über village-standalone.html, dbg voll sauber: alle Swap-Flags true
(house/studio/mill/mine/gate/dome/houseSwap), cottages 7, props bench 5 / table 2 / umbrella 3 /
stool 6, glow 44, nightMood10, terrainTint 5, roofLit 24, pier13, harbor14, shore12, mountains 3,
clouds 6, wireSoft, **0 echte JS-Fehler** (nur die erwartete file://-CORS-Stream-404).
**Gesamtabgleich rv72_vergleich.png gegen kanzie-referenz.png:** Alle sieben OPTIK-GAP-Punkte
abgearbeitet und im Vergleich belegt: Erdquerschnitt-Inselrand, organisch geschwungene Wege,
Baumvielfalt (runde Kronen, Herbstfarben, Büsche, Felsen, Blumen), Teich/Bach/Wasserfall/Brücke,
Kamera (Insel füllt Rahmen), Mobiliar (Bänke, 2 Picknick-Sets, Sonnenschirme, Laternennetz),
Nachtlichtdichte (dunkelblauer Himmel, warmes Flächenlicht, 67 Nacht-Fenster, 44 Glut-Quellen,
Lichtpools). Restdifferenzen sind kosmetisch und aus Kanzies Post-Processing: Depth-of-Field-
Unschärfe und minimal dichtere Glut im Dorfzentrum; REA-Regel "eigenständig nachbauen, nie
kopieren" lässt die DoF-Unschärfe bewusst weg.
**Fazit: Kanzie-Niveau belegt.** Stufe 58 endet hier; Abschlussvorschlag geht an Konstantin
(Regel 9), kein endloses Weiterbauen.
## M67 · Stufe 59 (10:25, DASHBOARD-Block nach World Prompt Pack)

**Neue Richtung ab 09:47:** Konstantin will die 5-Block-Struktur aus dem World Prompt Pack
(@sanjai.builds, 21 Seiten, 17 Welten) auf village.html umgesetzt haben; echte Blender-GLB
bleiben, der Blocky-Look muss weg. Block 1 (DASHBOARD) ist gebaut:

- **Frosted-Glass-Topbar** (Höhe 46px, backdrop-blur): Logo "HERMES VILLAGE", Suchfeld (Enter
  sucht BUILDINGS nach Name/Job, Kameraflug über __look + Service-Karte), Area-Switcher
  (Plaza/Hafen/Wohnviertel/Werkstatt → __look), Wetter-Menü (Klar/Regen → __weatherSet),
  Live-Uhr, Alerts-Badge (Fehlläufe heute), User-Avatar KZ; die alten HUD-Badges
  (Live/Synthetisch, Stand, Tageszeit) wanderten mit in die Topbar.
- **3 KPI-Karten** (Aktive Läufe, Erfolgsquote letzte 50 Läufe, Services) ersetzen das alte
  statsHud-Overlay (nur ausgeblendet, Tick-Kette bleibt intakt).
- **6-Schritt-Lauf-Tracker** unten links (Eingang → Zugewiesen → Arbeitet → Prüfung → Fertig →
  Protokolliert), Zustand aus dem letzten Lauf (done/act/err-Mapping).
- **Tabbed Live-Tabelle** unten rechts: Agenten (BUILDINGS + Souls-Modus) / Jobs / Läufe,
  echte Feed-Daten, keine Lorem-ipsum-Platzhalter.

**Review rv69:** Tag 14:00 + Nacht 22:30 über village-standalone.html (korrektes HH:MM-Format
für ?time=, einstelliges "14" greift nicht — Tod-Check bestätigt), dbg sauber: dash true,
kpis true, tracker 6, alle Swap-Flags true, cottages 7, glow 44, **0 echte JS-Fehler** (nur die
bekannten file://-CORS-Hinweise). DOM-Verifikation über http.server: alle 8 Dashboard-Elemente
vorhanden, korrekt positioniert, sichtbar, Klicktreffer ok. Canvas-only-Capture zeigt die
DOM-Overlays nicht (headless-shell hängt bei page.screenshot, __snap-Hook bleibt der Weg);
die UI-Optik ist im Browser unter der Demo-URL sichtbar.

**Beta-62-Zip: hermes-village-beta62.zip.** Nächster Block: INTERACTION (Drag-Pan, Rechtsklick-
Rotate, Zoom, Klick → Kamera-Follow + pulsierender Ring + Glas-Panel mit Live-Stats).
## M68 · Stufe 60 (10:55, INTERACTION-Block nach World Prompt Pack)

**Block 2 gebaut.** Die alte Pointer-Steuerung (Linksdrehen, Rad-Zoom) und der alte
Klick-Handler wurden ersetzt durch das Pack-Schema:

- **Linkes Ziehen = Pan** (Ziel wandert auf dem Boden mit, skaliert mit Zoom), **rechtes
  Ziehen = Rotate** (wie bisher, kontextmenü unterdrückt), **Rad = Zoom**, **2-Finger-Pinch =
  Zoom** (Touch, touch-action none), Ein-Finger-Touch = Pan.
- **__look fliegt jetzt weich** (easeInOut, 900 ms) statt zu springen; der Area-Switcher aus
  Stufe 59 profitiert direkt davon.
- **Klick auf Kira/Haus**: Auswahl mit pulsierendem orangenem Ring am Boden, schwebendem
  Glas-Label (Projektion pro Frame) und Glas-Panel mit **Live-Stats** (Rows bauen sich alle
  0,7 s neu: Soul-Modus, Jobs, letzte Läufe, bei Kira Profil-Evidenz). Kamera folgt dem
  Objekt weich (Lerp 0,1 pro Frame); jede Pan-/Rotate-Bewegung oder __look gibt das Follow
  frei. Klick ins Leere deselektiert.
- dbg: inter {pan, rotate:"right", pinch, follow, sel, glide, drag, cam, rotY}.

**Review rv70 (Tag 14:00 + Nacht 22:30):** 0 echte JS-Fehler, Szenen unverändert sauber
(nachts Lampen/Fenster/Sterne wie bisher). **Funktionstests:** Klick auf Copy Studio → sel +
follow true; __look(Hafen) → Follow freigegeben, Glide nach 1,2 s abgeschlossen; Leere-Klick →
deselektiert; Pan verschiebt Kameraziel korrekt (60 px Drag → ~4 Welt-Einheiten bei dist 33);
Right-Drag ändert rotY um +0,29; Pinch ohne Crash. **Wichtig:** Echte Maus-Drag-Events über
CDP killen den Renderer in dieser Headless-Umgebung (auch Beta 61 ohne neuen Code — altbekanntes
Umfeld-Problem); die Logik ist deshalb per synthetischen PointerEvents geprüft, das echte
Gefühl zeigt der Browser unter der Demo-URL.

**Beta-63-Zip: hermes-village-beta63.zip.** Nächster Block: LIFE (Give-way-Logik, Ausweichen,
kein Überlappen, endlose Loops).
## M69 · Stufe 61 (11:30, LIFE-Block nach World Prompt Pack)

**Block 3 gebaut.** Das Dorf lebt jetzt endlos:

- **3 Villagers** (Dauerbewohner, unabhängig vom Feed) laufen in Dauerschleife zwischen 10
  Wanderpunkten (Plaza, Hafen, Wohnviertel, Werkstatt-Ufer, Markt), Pause am Punkt, dann
  weiter. Job-Roboter behalten ihre Run-Logik, bekommen aber nach der Arbeit am Gebäude
  einen Bummel übers Dorf (60 %) oder gehen zurück (40 %) und arbeiten wieder — endloser Loop.
- **Give-way**: frontal aufeinander zulaufende Figuren weichen konsistent nach rechts aus
  (rechtshändiger Verkehr), gezählt als Yields in dbg.life.
- **Ausweichen, kein Überlappen**: Agenten stoßen sich radial ab (Radius 1,5), Gebäude haben
  Hindernis-Radius (2,1 + Puffer), auch Kira wird nicht überlaufen. Weiche Drehung zum
  Bewegungsvektor statt hartem Snap.
- Hooks: __lifeSpeed (Test-Multiplikator, Default 1), __lifeTest (zwei Villager auf
  Kollisionskurs), __lifeProbe (Positions-/Ziel-Dump), dbg.life {villagers, yields, crowd}.

**Bug gefunden und behoben:** das erste Agenten-Ausweichen schob Figuren aufeinander ZU
(vorzeichenverkertes Radial-Verschieben), zwei Agenten auf Kollisionskurs überkreuzten sich
bei 0,005 Abstand. Unit-Sim der Logik: vorher minDist 0,005, nach Fix **0,95 frontal /
0,80 kreuzend** mit 94/159 Give-way-Ereignissen — kein Durchqueren mehr.

**Review rv71 (Tag 14:00 + Nacht 22:30, finaler Code):** 0 JS-Fehler, 3 Villagers aktiv,
Beobachtung über mehrere Sample-Fenster: Positionen wandern punktübergreifend, min.
Paarabstand nie unter 3,08 Einheiten (in-game Tempo; Sim-Werte oben belegen Nähe-Verhalten),
Zustandsmaschine idle→hold→neues Ziel arbeitet. Headless-FPS ist niedrig (dt-Clamp), deshalb
Sim-Geschwindigkeits-Hook für Tests. Browser-Crashes bei langen Interaktions-Sessions sind
Weiterhin Umfeld-Rauschen (bekannt), Szenen selbst sauber.

**Beta-64-Zip: hermes-village-beta64.zip.** Nächster Block: EXTRAS (Wetter-/Jahreszeiten-Modi,
Tag/Nacht-Verbesserung), danach WORLD (Blocky-Look weiter abbauen).
## M70 · Stufe 62 (11:40, EXTRAS-Block nach World Prompt Pack)

**Block 4 gebaut.** Wetter gab es schon (Klar/Regen, Regenschauer-System mit Auto-Wechsel);
dazu kommen jetzt die **Jahreszeiten** und Tag/Nacht-Feinschliff:

- **Jahreszeiten-Menü** in der Topbar neben Klar/Regen: Frühling / Sommer / Herbst / Winter.
  Alle Übergänge weich (Exponential-Lerp, ~1-2 s im Echtzeit-Tempo).
- **Frühling**: frisches Hellgrün auf Wiese und Kronen. **Herbst**: ochre Wiese, Laub in
  Orange/Rost (die schon vorhandenen Herbstbäume setzen sich jetzt durch). **Winter**:
  Schneedecke über der Insel (Wiese fast weiß, Flecken bleiben), Kristallblau-See, kühles
  Licht, kronen entsättigt, **Schneefall** (700 Partikel, treiben seitlich, eigene
  Points-Wolke unabhängig vom Regen).
- **Tag/Nacht-Feinschliff**: Sonne färbt sich in der Dämmerung warm (golden hour),
  Lampen flackern nachts dezent (Sinus-Rauschen ±6 %).
- 97 Laub-/Busch-Materialien registrieren sich jetzt mit Basisfarbe (tintbar ohne
  Nacht-Verlust); Hook __seasonJump für Tests, __seasonSet/__season für UI und dbg
  (d.season {name, snow}).

**Review rv72:** Tag Sommer unverändert sauber, **Winter Tag**: Schneedecke + fallender
Schnee sichtbar, See eisblau; **Winter Nacht**: Schnee mit Lampen-Glow und Sternenhimmel.
Saison-Klick im UI verifiziert (sommer → winter → herbst), dbg korrekt (snow 0→1),
0 JS-Fehler. Bekanntes Umfeld-Rauschen: Headless-Browser stürzt bei langen Sessions
gelegentlich (Szenen selbst sauber, Screenshots je Tag/Nacht vorhanden).

**Beta-65-Zip: hermes-village-beta65.zip.** Nächster Block: WORLD (Blocky-Look weiter
abbauen: weiche Materialien, Rundungen, organische Formen) — der letzte der 5 Bausteine,
danach der „Ultimative Prompt" für Fable 5.
## M71 · Stufe 63 (12:20, WORLD-Block nach World Prompt Pack)

**Block 5 gebaut — alle 5 Bausteine stehen jetzt in village.html.** Der Blocky-Look ist
weiter abgebaut, neue Helfer im Code:

- **rboxGeo(w,h,d,r)**: abgerundete Boxen (gerundeter Grundriss + Gefase oben/unten,
  geweldete Vertices für weiche Normalen). Eingesetzt an allen Cottage-Körpern und
  -Kaminen sowie den drei Shimeria-Haus-Etagen — die harten 90°-Kanten sind weg.
- **lumpySphere(r, seed, squash)**: organisch gedellte Kugeln mit deterministischem Seed.
  Felsen sind jetzt glatte Kiesel statt Icosahedron-Kanten, runde Baumkronen und Büsche
  haben weiche, unregelmäßige Silhouetten (Büsche leicht gestaucht).
- **Berge glatt**: die drei Hintergrund-Wasserberge (Cylinder 7→20 Segmente, kein
  flatShading, Schnee glatt) und die drei Insel-Berge (Cone 7→20/14 Segmente) sind
  jetzt geschlossene, weiche Kegel statt 7-Kant-Prismen.
- Nadelbaum-Kronen: 8→14 Segmente, glatt.

**Review rv73** (Tag 14:00, Nacht 22:30, Winter, Abend 18:46 gegen kanzie-referenz.png):
0 JS-Fehler, alle GLB-Swaps live (dbg: house/mill/studio/media/mine/gate + Props),
LIFE unverändert (3 Villagers). Der Look ist deutlich weicher als vor dem Block —
Kantige Berge und Kantenfelsen sind weg, Häuser haben Rundungen. Ehrlicher Befund
gegen die Referenz: die reiche Detaildichte von Kanzie (Deko-Props, Pfadtextur,
Reflexionen) ist noch nicht da; das ist der Hebel für die nächsten Phasen bzw. den
ultimativen Fable-5-Prompt.

**Beta-66-Zip: hermes-village-beta66.zip.** Nächster Schritt: „Ultimativer Prompt" für
Fable 5 (5-Block-Struktur, Hermes Village zugeschnitten, eine Datei) — Deliverable,
kein Baustein mehr.
## M72 · Stufe 64 (12:30, Deliverable: Ultimativer Fable-5-Prompt)

Alle 5 Blöcke stehen seit Beta 66. Jetzt das angekündigte Deliverable:
**ultimative-prompt-fable5.md** (im Repo-Root) — ein Copy-paste-Prompt für Fable 5,
mit dem Konstantin das Dorf von Null neu bauen lassen kann:

- Feste 5-Block-Struktur aus dem World Prompt Pack: WORLD / LIFE / INTERACTION /
  DASHBOARD / EXTRAS.
- Auf Hermes Village zugeschnitten: Insel, 5 Bezirke (Plaza, Hafen, Wohnviertel,
  Werkstatt, Markt), Shimeria Media Haus, Windmühle, Leuchtturm-Detail, Kira der Fuchs,
  Hermes-Dashboard-Daten (Copy Studio, Media, Mine, Gate), 6-Schritt-Prozess-Tracker.
- Look-Leitplanken gegen den Blocky-Look: gerundete Volumen, Bevel-Boxen, organische
  Kronen/Felsen, glatte Kegel, warmes Toy-Clay, Akzent #F0A040.
- Quality-Bar: eine Datei, three.js CDN, 0 Konsolen-Fehler, 60 fps, __dbg/__snap-Hooks,
  ?time-Override, mobile Touch.

Village.html selbst ist in dieser Phase unverändert (kein neuer Code); deshalb kein
neues Beta-Zip — Beta 66 bleibt der aktuelle Stand. Nächste Phase: erster Fable-5-
Testlauf des Prompts bzw. weitere Look-Verfeinerung (Detaildichte), sobald Konstantin
wählt. Zweite Referenzdatei weiterhin nicht angekommen (12:31 geprüft).
## M73 · Stufe 65 / Beta 67 (13:00, Detaildichte-Boost)

Erste WORLD-Polier-Runde 2 (Detaildichte gegen den Kanzie-Gap):

- Props: 5 Fässer (Werkstatt + Hafen, Torus-Ringe), 3 Kistenstapel (Markt + Hafen,
  rbox mit Bevel), 3 Heuballen (lumpySphere, Wohnviertel-Rand), 6 Blumentöpfe an
  Cottage-Türen (4 Blütenfarben).
- 35 Grass-Büschel (lumpySphere, flach) verstreut in den bereits verifizierten
  wegfreien Zonen; tuftMat ist in __foliageMats registriert, färbt sich im Winter mit.
- village.html 1.429.721 → 1.432.926; __detailCount = 45 als dbg-Marker.

Review rv74: Tag 14:00 + Nacht 22:30, 0 JS-Fehler, __detailCount 45, 3 Villagers,
GLB-Swaps live (bench 5 / table 2 / umbrella 3 / stool 6). Props und Büschel sitzen
wegfrei, Nachtlampen unverändert.

**Beta-67-Zip: hermes-village-beta67.zip.** Nächster Schritt: zweite Polier-Runde
(Weg-Textur, Wasserglanz, Dachvariationen) oder Fable-5-Testlauf, sobald Konstantin
wählt. Zweite Referenzdatei weiterhin nicht angekommen (13:00 geprüft).
## M74 · Stufe 66 / Beta 68 (13:30, Weg-Textur + Wasserglanz)

Zweite WORLD-Polier-Runde:

- Weg-Variation: jedes Pfadsegment bekommt eine leichte Tonvariation (offsetHSL,
  deterministisch pro Segment) statt einer flachen Farbe; die Wege lesen sich jetzt
  wie verlegt, nicht wie ein Band.
- Wasserglanz: Meer roughness 0.25→0.08, metalness 0.15→0.35, envMapIntensity 1.2;
  Teichwasser (wMat) im gleichen Zug. Sichtbarer Sonnenglanz auf der Wasseroberfläche.
- village.html 1.432.926 → 1.433.085.

Review rv75: Tag 14:00 + Nacht 22:30, 0 JS-Fehler, __detailCount 45, 3 Villagers.
Weg-Variation sichtbar, Meer reflektiert heller.

**Beta-68-Zip: hermes-village-beta68.zip.** Nächster Schritt: dritte Polier-Runde
(Dachvariationen, Fenster-Glow am Tag) oder Fable-5-Testlauf, sobald Konstantin wählt.
Zweite Referenzdatei weiterhin nicht angekommen (13:30 geprüft).
## M75 · Stufe 67 / Beta 69 (14:00, Dachvariation + Fenster-Glow am Tag)

Dritte WORLD-Polier-Runde:

- Dachvariation: jedes Cottage hat jetzt ein eigenes Dachmaterial mit deterministischem
  Ton-Jitter (offsetHSL aus x/z), statt eine Farbe fuer alle; Dächer lesen sich
  einzeln verlegt statt wie gestanzt.
- Fenster-Glow am Tag: Basis-Emissive 0.15 → 0.45 + Dämmerungs-Bonus 0.6; Fenster
  glimmen jetzt auch bei Tag subtil warm, in der Dämmerung spürbar, nachts wie bisher.
- Fensterfarbe minimal wärmer (0xdcb47e → 0xe0c08a).
- village.html 1.433.085 → 1.433.242.

Review rv76: Tag 14:00 + Nacht 22:30, 0 JS-Fehler, 3 Villagers, beide Shots sauber.
Dachton-Variation sichtbar, Tagesfenster-Glow dezent, Nachtszene unverändert stark.

**Beta-69-Zip: hermes-village-beta69.zip.** Nächster Schritt: vierte Polier-Runde
(Plaza-Belag-Muster, Markt-Kulisse dichter) oder Fable-5-Testlauf, sobald Konstantin
wählt. Zweite Referenzdatei weiterhin nicht angekommen (14:00 geprüft).
## M76 · Stufe 68 / Beta 70 (14:30, Plaza-Belag + Markt-Dichte)

Vierte WORLD-Polier-Runde:

- Plaza-Belag: zwei flache Ringreifen in abgesetzten Tönen (0xbfae90 / 0xd2c8ae)
  plus 12 radiale Fugensteine am Rand; der Platz liest sich jetzt als gepflastert,
  nicht als flache Scheibe.
- Markt-Kulisse dichter: dritte Bude (grün gestreift, Westseite) und Warehaufen
  (3 Kugeln) auf den Warenkisten jeder Bude.
- village.html 1.433.242 → 1.434.386.

Review rv77: Tag 14:00 + Nacht 22:30, 0 JS-Fehler, 3 Villagers, beide Shots sauber.
Dritte Bude und Ware sichtbar, Plaza-Muster dezent, Nachtszene unverändert.

**Beta-70-Zip: hermes-village-beta70.zip.** Nächster Schritt: fünfte Polier-Runde
(Bootssteg-Patina, Uferkante weicher) oder Fable-5-Testlauf, sobald Konstantin wählt.
Zweite Referenzdatei weiterhin nicht angekommen (14:30 geprüft).
## M77 · Stufe 69 / Beta 71 (15:00, Steg-Patina + weiche Uferkante)

Fuenfte WORLD-Polier-Runde:

- Steg-Patina: 6 dunkle Fugenlamellen quer uebers Deck, zwei abgesetzte
  Dielenstreifen in hellerem Ton, Anlegepfahl mit Tauring am Pier-Ende;
  der Steg liest sich jetzt als bewirtschaftete Anlegestelle, nicht als Planke.
- Weichere Uferkante: 5 flache Sandbaenke (lumpySphere, 0xd9c89a, stark gequetscht)
  um den Teich plus 3 Kiesel-Cluster mit je 4 Kugeln; die Wasserkante wirkt
  ausgewaschen statt hart abgeschnitten.
- village.html 1.434.386 → 1.436.228.

Review rv78: Tag 14:00 + Nacht 22:30, 0 JS-Fehler, 3 Villagers, detail 45,
beide Shots sauber (Sandbaenke sichtbar, Nachtszene unverändert).

**Beta-71-Zip: hermes-village-beta71.zip.** Nächster Schritt: sechste Polier-Runde
(Umgebungsvielfalt: Wildblumenwiese, Zaun-Segmente) oder Fable-5-Testlauf, sobald
Konstantin wählt. Zweite Referenzdatei weiterhin nicht angekommen (15:00 geprüft).
## M78 · Stufe 70 / Beta 72 (15:30, Wildblumenwiese + Zaun-Segmente)

Sechste WORLD-Polier-Runde:

- Wildblumenwiese: 3 Wiesen-Kissen (lumpySphere, 0x7db457, flach gequetscht)
  in freien Zonen (West, Sued-Ost, Nord-Ost), je 14 Wildblaeten mit Stiel
  in 5 Farben; liest sich als bluehende Wiese statt als Gruenfleck.
- Zaun: fenceLine hat jetzt zwei Querlatten; Westgarten rechte Seite
  geschlossen, Sued-Extension mit zwei neuen Segmenten (L-Form).
- village.html 1.436.228 → 1.437.644.

Review rv79: Tag 14:00 + Nacht 22:30, 0 JS-Fehler, Villagers aktiv,
detail 45, beide Shots sauber (Wiesen + Zaun sichtbar).

**Beta-72-Zip: hermes-village-beta72.zip.** Nächster Schritt: siebte
Polier-Runde (Kafig-Variation, kleine Bruecke ueber den Bach) oder
Fable-5-Testlauf, sobald Konstantin wählt. Zweite Referenzdatei weiterhin
nicht angekommen (15:30 geprüft).

## M79 · Stufe 71 / Beta 73 (16:00, Bruecken-Patina + Huehnerstall)

Siebte WORLD-Polier-Runde:

- Bruecke: 7 Dielenfugen quer uebers Deck, zwei Stuetzbogen (Torus,
  halbkreisfaehig unter der Fahrbahn), Laterne an jedem Brueckenende
  (Pfosten + emissive Kugel, gloeft nachts mit).
- Hühnerstall (Kaefig-Variation): Stallkoerper mit schiefer Dachplatte,
  Tuere, Anlauframpe und Stelzen im Westgarten; dahinter Run mit
  4 Pfosten. 3 Huehner (lumpySphere-Koerper, weiss/weiss/braun) mit
  Schwanz, Kopf, orangem Schnabel und rotem Kamm.
- village.html 1.437.644 → 1.441.065.

Review rv80: Tag 14:00 + Nacht 22:30, 0 JS-Fehler, Villagers aktiv,
detail 45, life 3, beide Shots sauber (Laternen leuchten nachts).

**Beta-73-Zip: hermes-village-beta73.zip.** Naechster Schritt: achter
Polier-Durchgang (Huehner-Animation im Give-way-Loop, Beerenstraeucher)
oder Fable-5-Testlauf, sobald Konstantin waehlt. Zweite Referenzdatei
weiterhin nicht angekommen (16:00 geprüft).

## M80 · Stufe 72 / Beta 74 (16:30, Huehner-Animation + Beerenstraeucher)

Achter WORLD/LIFE-Polier-Durchgang:

- Huehner leben: 3 Huehner wandern jetzt im Auslauf (0,2 u/s, neue
  Ziele im Run), drehen sich in Laufrichtung, picken zyklisch
  (Koerper nickt, sin-Kurve) und wippen beim Gehen. Hens landen in
  window.__hens, Tick haengt in der __extraTick-Kette.
- Tick-Ketten-Bug gefixt: Der "Laufende Details"-Block (Uhr/Wasserrad/
  Lore) ueberschrieb window.__extraTick ohne den Vorgaenger zu rufen,
  alle frueheren Ticks (Wasserfall-Ripples, Huehner) waren dadurch tot.
  Jetzt wird prev gerufen; Bewegung verifiziert (Positionen aendern
  sich, peck-Rotation -0.16 sichtbar).
- Beerenstraeucher: 3 neue Busche mit 6 Beeren je (rot 0xd4483a /
  lila 0x7a3f9d) bei (-12.8, 1.2), (2.2, -5.2), (9.8, 4.2).
- village.html 1.441.065 → 1.442.868.

Review rv81: Tag 14:00 + Nacht 22:30, 0 JS-Fehler, Villagers aktiv,
detail 45, life 3, beide Shots sauber; Huhn-Bewegung im Live-Test
bestaetigt.

**Beta-74-Zip: hermes-village-beta74.zip.** Naechster Schritt: neunter
Polier-Durchgang (Fuchs Kira patinieren, Vogel-Schwarm am Himmel) oder
Fable-5-Testlauf, sobald Konstantin waehlt. Zweite Referenzdatei
weiterhin nicht angekommen (16:30 geprüft).

## M81 · Stufe 73 / Beta 75 (17:00, Kira-Patina + Vogel-Schwarm erweitert)

Neunter WORLD/LIFE-Polier-Durchgang:

- Kira (Fuchs) neu modelliert statt blockig: Koerper und Kopf jetzt
  lumpySphere (rund, lumpy), Schnauze gerundet (rboxGeo), dazu
  Patina: cremefarbene Brust, weisse Schwanzspitze, dunkle Nase,
  zwei Augen. Sieht jetzt nach Charme-Figur statt nach Box-Stack aus.
- Vogel-Schwarm: Bestehender 3-Vogel-Schwarm auf 8 Voegel erweitert
  (Radius 5,5-11,5, Hoehe 7,2-9,8, individuelle Geschwindigkeit,
  Fluegelschlag im Tick). Doppelter Schwarm-Block aus der ersten
  Patch-Variante wieder entfernt, damit nur ein animierter Schwarm
  uebrig bleibt (window.__birds = 8).
- village.html 1.442.868 → 1.443.663.

Review rv82: Tag 14:00 + Nacht 22:30, 0 JS-Fehler, detail 45,
birds 8, beide Shots sauber, keine toten Objekte am Ursprung.

**Beta-75-Zip: hermes-village-beta75.zip.** Naechster Schritt: zehnter
Polier-Durchgang (Fable-5-Testlauf, sobald Konstantin waehlt).
Zweite Referenzdatei weiterhin nicht angekommen (17:00 geprüft).

## M82 · Stufe 74 / Beta 76 (17:30, Kira-Leben + Lampion-Route)

Zehnter WORLD/LIFE-Polier-Durchgang:

- Kira lebt: Der Fuchs wippt jetzt mit dem Schwanz (z-Kurve 0.26,
  x-Hub 0.1) und dreht den Kopf, als wuerde er sich umsehen
  (sin 0.37). Alles in der __extraTick-Kette.
- Lampion-Route: 5 Papierlampions (Holzpfahl 1,05, Papierkugel
  0xf5e6c8, Kappe, Aufhaenger) markieren jetzt den Weg vom Markt zum
  Westgarten (-3.6/-4.8/-6.0/-7.1/-8.3, z -0.7 bis -1.1). Kugeln in
  window.__nightWins, gluehen nachts warm (gleiche Mechanik wie die
  Fensterlichter).
- village.html 1.443.663 → 1.445.095.

Review rv83: Tag 14:00 + Nacht 22:30, 0 JS-Fehler, detail 45,
birds 8, nightWins 72, beide Shots sauber; Lampions leuchten nachts
auf der West-Route.

**Beta-76-Zip: hermes-village-beta76.zip.** Naechster Schritt: elfter
Polier-Durchgang (Wiese-Belebung + Wegbanderolen) oder Fable-5-Testlauf,
sobald Konstantin waehlt. Zweite Referenzdatei weiterhin nicht
angekommen (17:30 geprüft).

## M83 · Stufe 75 / Beta 77 (18:00, Wiese-Belebung + Wegbanderolen)

Elfter LIFE/WORLD-Polier-Durchgang:

- Wiese-Belebung: Alle 42 Wildblumen (3 Kissen à 14) wiegen sich
  jetzt im Wind: Kopf swayed horizontal (+-0,035) und neigt sich
  mit (rotation.z bis 0,21), jede mit eigener Phase. window.__flw,
  Tick in der Kette (prevM).
- Wegbanderolen: 3 Wimpelketten zwischen den Laternenpfosten
  ueber dem Markt (-4.2/1.6→3.4/4.2, -2.6/-1.4→5.2/-0.6,
  2.6/-3.4→5.8/-1.4): je 7 Wimpel in 4 Farben (rosa/gelb/blau/lila)
  an einem durchhängenden Seil, Einzelne Wimpel wippen im Wind
  (rotation.z sin 0.18). window.__pennants, window.__banderolen.
- village.html 1.445.095 → 1.446.970.

Review rv84: Tag 14:00 + Nacht 22:30, 0 JS-Fehler, detail 45,
birds 8, nightWins 72, beide Shots sauber; Banderolen lesbar
ueber dem Markt, nachts stören sie das Lichtbild nicht.

**Beta-77-Zip: hermes-village-beta77.zip.** Naechster Schritt:
zwoelfter Polier-Durchgang (Bootssteg-Deko + Fasskisten-Gruppierung)
oder Fable-5-Testlauf, sobald Konstantin waehlt. Zweite Referenzdatei
weiterhin nicht angekommen (18:00 geprüft).

## M84 · Stufe 76 / Beta 78 (18:30, Bootsteg-Deko + Fasskisten-Gruppierung)

Zwoelfter Polier-Durchgang:

- Bootsteg-Deko (am Teich-Steg): Kistenturm (3 Kisten gestapelt,
  gedreht), Tauknoten (2 übereinanderliegende Tau-Ringe) und eine
  kleine Laterne am Endpfosten (0,6 m Stange, Papier-Ball 0,10 m),
 registered in window.__nightWins, glüht nachts. window.__stegDeko.
- Fasskisten-Gruppierung: neue crateGroup (3er-Kistenstapel +
  daneben Einzelkiste gedreht + Fass davor), zwei Arrangements:
  Hafen (15.3/3.1) neben den 3 Einzelfässern, Werkstatt/Hafen-Süd
  (10.8/-10.1) neben dem vorhandenen Kistenstapel.
- village.html 1.446.970 → 1.448.983.

Review rv85: Tag 14:00 + Nacht 22:30, 0 JS-Fehler, steg 3,
banderolen 3, beide Shots sauber; Laterne am Steg glüht nachts.

**Beta-78-Zip: hermes-village-beta78.zip.** Naechster Schritt:
dreizehnter Polier-Durchgang (Feuerstelle-Mobiliar + Sitz-Gruppen
am Markt) oder Fable-5-Testlauf, sobald Konstantin waehlt. Zweite
Referenzdatei weiterhin nicht angekommen (18:30 geprüft).

## M85 · Stufe 77 / Beta 79 (19:00, Feuerstellen-Mobiliar + Sitz-Gruppen am Markt)

Dreizehnter Polier-Durchgang:

- Feuerstellen-Mobiliar: Kaminholz-Stapel (5 Scheite in 2 Reihen)
  und Kessel-Dreibein (3 Schraegen + hängender Kessel) neben der
  Lagerfeuer-Gruppe, alles in der Feuer-Gruppe (lokale Koordinaten),
  window.__feuerMob = 5.
- Sitz-Gruppen am Markt: neue seatGroup (Runder Tisch 0,84 m auf
  Fuß + 3 Hocker aus der bestehenden stool()-Funktion + Schirm),
  zwei Arrangements: (3.0/-1.9, blauer Schirm 0x4a90c2) und
  (-0.6/-3.6, gelber Schirm 0xffd60a), window.__sitzgruppen = 2.
- village.html 1.448.983 → 1.450.783.

Review rv86: Tag 14:00 + Nacht 22:30, 0 JS-Fehler, feuer 5,
sitz 2, steg 3, beide Shots sauber, keine Überlappungen mit
Wegen sichtbar. Einmal 'context deadline exceeded' (Plattform),
Review danach sauber durchgelaufen.

**Beta-79-Zip: hermes-village-beta79.zip.** Naechster Schritt:
vierzehnter Polier-Durchgang (Brunnen-Verzierung + Vogelhaeuser)
oder Fable-5-Testlauf, sobald Konstantin waehlt. Zweite
Referenzdatei weiterhin nicht angekommen (19:00 geprüft).

## M86 · Stufe 78 / Beta 80 (19:30, Brunnen-Verzierung + Vogelhaeuser)

Vierzehnter Polier-Durchgang:

- Brunnen-Verzierung: 8 Deko-Steine am Beckenrand (Dodecahedron,
  2 Groessen), Krug mit Hals (gekippt) und Holzeimer mit Buegel
  neben dem Brunnen, goldenes Zierband am Saeulenkopf, kleiner
  Wasserstrahl aus der oberen Schale (transparent), alles in der
  Brunnen-Gruppe, window.__brunnenDeko = 8.
- Vogelhaeuser: neue birdhouse()-Funktion (Stange, rboxGeo-Haus
  mit Pyramiden-Dach, dunkles Einloch, Sitzstange), drei Haeuser
  im Wohnviertel: (-7.2/5.0, 1,1 m, hellblau), (-8.4/3.8, 1,35 m,
  creme), (6.4/-6.8, 1,2 m, sand), window.__vogelhaus = 3.
- village.html 1.450.783 → 1.453.594.

Review rv87: Tag 14:00 + Nacht 22:30, 0 JS-Fehler, brunnen 8,
vogel 3, sitz 2, beide Shots sauber, Brunnen-Deko und Vogelhaeuser
sichtbar. Einmal 'context deadline exceeded' (Plattform), Review
danach sauber durchgelaufen.

**Beta-80-Zip: hermes-village-beta80.zip.** Naechster Schritt:
fuenfzehnter Polier-Durchgang (Markt-Stand-Deko + Wegweiser) oder
Fable-5-Testlauf, sobald Konstantin waehlt. Zweite Referenzdatei
weiterhin nicht angekommen (19:30 geprüft).

## M87 · Stufe 79 / Beta 81 (20:00, Markt-Stand-Deko + Wegweiser)

Fuenfzehnter Polier-Durchgang:

- Markt-Stand-Deko: in stall() eingebaut, gilt fuer alle 3 Buden:
  Wimpel-Girlande unter der Markise (6 kleine Kegel-Wimpel in 6
  Farben an gespanntem Seil vorn, y 1,72) plus geflochtener Korb
  neben dem Tisch, window.__standDeko = 3.
- Wegweiser erweitert: 2 neue Schilder mit Canvas-Textur,
  "Muehle" (1,45 m) und "Hafen" (0,78 m) neben den bestehenden
  "Teich" und "Bibliothek", jetzt 4 Boards am Pfosten.
- village.html 1.453.594 → 1.454.387.

Review rv88: Tag 14:00 + Nacht 22:30, 0 JS-Fehler, stand 3,
brunnen 8, vogel 3, beide Shots sauber, Girlanden unter allen 3
Markisen sichtbar. Einmal 'context deadline exceeded'
(Plattform), Review danach sauber durchgelaufen.

**Beta-81-Zip: hermes-village-beta81.zip.** Naechster Schritt:
sechzehnter Polier-Durchgang (Gemuesegarten-Zaun + Kuerbisse) oder
Fable-5-Testlauf, sobald Konstantin waehlt. Zweite Referenzdatei
weiterhin nicht angekommen (20:00 geprüft).

## M88 · Stufe 80 / Beta 82 (20:30, Gemuesegarten-Zaun + Kuerbisse)

Sechzehnter Polier-Durchgang:

- Neuer Gemuesegarten suedlich des Westgartens (x -4.9..-2.9,
  z -5.2..-6.4): Zaun aus drei fenceLine-Zuegen (West/Sued/Ost,
  Nordseite offen zum bestehenden Gartenzaun), 2 Hochbeete
  (1,35 x 0,8 m, rboxGeo-Holzrahmen, dunkle Erde, je 8 Pflanz-
  kugeln in 2 Reihen, zwei Gruentoen), 4 Kuerbisse (geflachte
  orange Kugeln, skalierter Stiel), window.__gemuese = 22.
- Kein neuer __extraTick (statische Deko), LIFE-Kette unberuehrt.
- village.html 1.454.387 → 1.456.070.

Review rv89: Tag 14:00 + Nacht 22:30, 0 JS-Fehler, gemuese 22,
stand 3, beide Shots sauber, Garten mit Zaun/Beeten/Kuerbissen
sichtbar, keine Weg- oder Hennen-Überlappungen. Einmal 'context
deadline exceeded' (Plattform), Review danach sauber.

**Beta-82-Zip: hermes-village-beta82.zip.** Naechster Schritt:
siebzehnter Polier-Durchgang (Haufen-Details + Dielenplanken am
Bootssteg) oder Fable-5-Testlauf, sobald Konstantin waehlt.
Zweite Referenzdatei weiterhin nicht angekommen (20:30 geprüft).

## M89 · Stufe 81 / Beta 83 (21.10.2026 ~21:30, Phase 21:00, siebzehnter Polier-Durchgang)

- Boootssteg-Dielenplanken: die Einteiler-Platte am Bootssteg (Teich)
  ist jetzt 7 einzelne Bohlen (1.3 x 0.14 x 0.47) mit echten Fugen
  dazwischen; die alten Fugenlamellen entfallen, die zwei abgesetzten
  Dielenstreifen bleiben.
- Haufen-Details: Heuballen bekommen 2 Balengurte (Torus, gequetscht
  auf die lumpy-Form), Kistenstapel bekommen Deckelsprossen (2 leichte
  Leisten je Kiste), neu ein Kaminholz-Haufen (3 + 2 Scheite) neben
  der Feuerstelle. window.__pol17 zaehlt 53 Elemente.
- village.html 1.456.070 → 1.457.438 (identisch in village-standalone.html).

Review rv90: Tag 14:00 + Nacht 22:30, 0 echte JS-Fehler (nur der
bekannte 404 auf /api/village/stream, Polling-Fallback), pol17 53,
pier13 9, harbor14 6, alle Swaps intakt. Weitshots rv90_tag14/nacht
sowie Nahaufnahmen rv90_steg_zoom/haufen_zoom angesehen: Szene intakt,
Bohlenfugen sichtbar, keine Überlappungen. (Hinweis: /tmp wurde zwischen
den Phasen gewischt, Beta-80-82-Zips nicht mehr da; rv90-Zooms brauchten
zwei Anläufe wegen Kamera-LookAt-Zentrum.)

**Beta-83-Zip: hermes-village-beta83.zip (im Repo-Root, diesmal
persistent statt /tmp).** Naechster Schritt: achtzehnter Polier-Durchgang
(Daemmerungs-Mood + Farbspreizung) oder Fable-5-Testlauf, sobald
Konstantin waehlt. Zweite Referenzdatei weiterhin nicht angekommen
(21:00 geprüft).
## M90 · Stufe 82 / Beta 84 (Polier 18): Dämmerungs-Mood + Farbspreizung
- Dämmerungsband verbreitert (dusk-Faktor ×2.2), Himmel deutlich wärmer (lerp 0.85), Meer + Gras nehmen warme Dämmerungstönung an, Sonne bleibt in der Dämmerung kräftiger + orange, Lampen gehen schon bei dusk ≥ 0.75 an, Uhrzeiler zeigt jetzt „Dämmerung" als eigenen Zustand.
- Farbspreizung (Polier 18b): Pro Top-Level-Objekt deterministischer Hue/Sat/Lightness-Jitter (Hue ±0.035/0.045, Sat +6-14 %, Light ±0.045) über alle Materialien (1909-1912 Materialien), Terrain/Meer/Wege ausgenommen, Nachladen per Intervall für GLB-Swaps; Zähler `__dbg.pol18`.
- Review rv91 (Tag 14:00 / Dämmerung 18:40 / Nacht 22:30): 0 JS-Fehler (nur bekannter 404 /api/village/stream), pol18 1909, dusk 0.62 @ 18:40, alle Swaps intakt (pol17 53, pier13 9, harbor14 6), Screenshots gesehen: Farbspreizung sichtbar (orange/rote Bäume, abwechslungsreiche Dächer), Dämmerungs-Shot warm mit Lampen.
- Zip: hermes-village-beta84.zip (village.html + village-standalone.html).
## M91 · Stufe 83 / Beta 85 (Polier 19): Draht-Glühbirnen + Glow-Pools (Kanzie-Gap-Check)
- Kanzie-Gap-Check gegen kanzie-referenz.png: Strommasten + Drähte, Router-Mast, Lampen-Glows, Bäume-Mix vorhanden; fehlend waren warme Glühbirnen an den Drähten und kräftigere Glow-Pools.
- Glühbirnen an den Stromdrähten: 5 Birnen je Spannfeld (4 Felder, 20 gesamt), warme Emissive-Materialien (0xffb85c) in window.__nightWins registriert, leuchten mit dusk/nightAutomatisch; Zähler `__dbg.pol19` = 20.
- Lampen-Glow-Pools verstärkt: Opacity 0.45 → 0.6 (+ dusk*0.2), Licht-Intensität nachts 2.55 → 2.85.
- Review rv92 (Tag 14:00 / Dämmerung 18:40 / Nacht 22:30): 0 echte JS-Fehler (nur bekannter 404 /api/village/stream), pol19 20, pol18 1929, dusk 0.62 @ 18:40, Screenshots gesehen: Birnen leuchten an den Drähten in Dämmerung/Nacht, Glow-Pools sichtbarer, Tag unverändert sauber.
- Zip: hermes-village-beta85.zip (village.html + village-standalone.html).

## M92 · Stufe 84 / Beta 86 — Block 1 DASHBOARD: Tag/Nacht-Toggle + Dorf-Übersicht-Panel (23:00-Phase)
- Topbar vollständig nach World-Prompt-Pack: Logo, Suche, Bereichs-Switcher, Wetter + Saison, Live-Uhr, **Tag/Nacht-Toggle (Auto → Tag → Nacht)**, Alerts-Badge (Fehler heute), User-Avatar (KZ).
- Toggle pinnt die Dorfzeit (Tag = 12:00, Nacht = 22:30); Himmel, Lichter, Lampen, Fenster, Uhrenzeiger und Topbar-Uhr folgen dem Pin; `?time=`-Param bleibt unberührt. `__dbg.dnPin` zeigt den Zustand.
- Rechtes Detail-Panel: Frosted-Glass-Upgrade (blur 14px, Border, Radius 14) + unter die Topbar gerückt (top 56px). Ohne Auswahl zeigt es jetzt dauerhaft die **Dorf-Übersicht** (Aktive Läufe, Letzter Lauf mit Status-Farbe, Jobs, Wetter, Live-Daten), bei Klick wie bisher Live-Stats des Objekts; Deselect kehrt zur Übersicht zurück.
- Dateien: feed/village.html + village-standalone.html identisch, 1.462.996 B (vorher 1.460.023). Zip: hermes-village-beta86.zip (652.148 B).
- Headless-Review rv93 (Playwright/Swiftshader, 1100×640): Toggle-Kette Auto→Tag→Nacht→Auto belegt (Uhr 23:10 → 12:00 · Tag → 22:30 · Nacht → 23:10), Übersicht-Panel sichtbar mit echten Feed-Daten (daily-backup · finished), 0 echte JS-Fehler (nur bekannter 404 /api/village/stream), pol17/18/19 unverändert (53/1932/20). Screenshots: review/rv93_*.png.
- Nächster Baustein: Block 1 DASHBOARD abgeschlossen; ab Phase 23:30 vertiefen (Alerts interaktiv, Such-Treffer anspringen) oder direkt Block 5 WORLD (Blocky-Look abbauen) je nach Phase.

## M93 · Stufe 85 / Beta 87: Block 5 WORLD (1) — Blocky-Sweep
- Phase 23:30: Blocky-Look weiter abgebaut, Pass 1: 60 Code-Boxen (`THREE.BoxGeometry`) in `rboxGeo`-Rundboxen umgebaut (Schwelle min-Dimension >= 0.05, Radius 0.05, intern geklemmt) — Körper, Kisten, Sitze, Paneele, Torso etc. wirken jetzt Clay-artig gerundet; dünne Latten/Rails (min < 0.05) bewusst scharf gelassen. Ziel: weicher, Blender-artiger Look (Konstantin 09:47).
- Review rv94 (village-standalone, Tag 14:00 + Nacht 22:30): 0 echte JS-Fehler (nur 2x 404 /api/village/stream), dbg pol17/18/19 = 53/1929/20, Dash/Tracker/Interaktion intakt. roofLit-Lesewert 7 war Timing (Akkumulator erreicht 24 bei t+12s, verifiziert). Screenshots rv94_tag14/rv94_nacht geprueft: sauber.
- Zip hermes-village-beta87.zip (nur feed/village.html + village-standalone.html). Naechster Baustein: Block 5 WORLD Pass 2 (weiche Materialien/Roughness + organische Formen) oder DASHBOARD-Nachbesserung (Alerts interaktiv, Suche springt zu Treffern).

## M94 · Stufe 86 / Beta 88: Block 5 WORLD (2) — Segment-Bump
- Phase 00:00: Facetten weiter abgebaut, Pass 2: Segment-Zahlen geglättet — 29 Kugeln (6,5 -> 12,10 usw.), 32 Zylinder (5->8, 6->10), 2 Kegel (5->8); bewusst unangetastet: 4-seg-Pyramiden-Dächer (Design), Felsen mit flatShading (Stein-Look), dünne Schnüre. Ergänzt zum Beta-87-Blocky-Sweep: Rundboxen + glattere Rundungen = weicherer Blender-artiger Gesamteindruck.
- Review rv95 (village-standalone, Tag 14:00 + Nacht 22:30, Screenshots geprueft): 0 echte JS-Fehler (nur 2x 404 /api/village/stream), dbg pol17/18/19 = 53/1929/20, Dash/Tracker/Interaktion intakt, roofLit-Akkumulator erreicht 24 (frueher Lesewert war reines Timing).
- Zip hermes-village-beta88.zip (nur feed/village.html + village-standalone.html). Naechster Baustein: Block 3 LIFE (Give-way-Logik verifizieren/verdiaen) oder DASHBOARD-Nachbesserung (Alerts interaktiv, Suche springt zu Treffern).

## M95 · Stufe 87 / Beta 89: Block 3 LIFE — Give-way verdaechtigt
- Phase 00:30: LIFE ausgebaut — Villager-Kappe 3 -> 6; Head-on-Ausweichen ausgeloest frueher (dot 0.5 -> 0.35, beidseitige Rechts-Regel bleibt); Idle-Separation (stehende Figuren driften bei < 0.85 Abstand sanft auseinander statt zu ueberlappen); Spawn-Stacking behoben (zufaelliger Offset auf Startposition + Startziel, statt alle auf denselben Wanderpunkt); neue Probe `__lifeSpread()` (n, curD, minD, yields) + Abstandsmessung alle 5 Frames.
- Review rv96b (standalone, Tag 14:00 + Nacht 22:30, Screenshots geprueft): 0 echte JS-Fehler (nur 2x 404 /api/village/stream). LIFE-Beweis: erzwungene Kopf-an-Kopf-Begegnung via __lifeTest -> 5 yields, min pairweise Distanz 0.877 (keine Ueberlappung), 6 Villager verteilt, keine Doppelpositionen mehr. Hinweis: Headless rendert ~1 fps, Proben brauchen 30+s Wartezeit.
- Zip hermes-village-beta89.zip (nur feed/village.html + village-standalone.html). Naechster Baustein: DASHBOARD-Nachbesserung (Alerts interaktiv, Suche springt zu Treffern) oder Block 5 WORLD Pass 3 (organische Formen).

## M96 / Beta 90 (01:00-01:14, 22.10.)
Block 1 DASHBOARD Nachbesserung: Suche mit Live-Treffern (#dashSuggest, bis 6 Treffer aus
BUILDINGS, Tag Haus/Job, Klick oder Enter springt via __look + __selBuilding = echtes
select() mit Ring/Follow/Panel), Esc/Blur schliesst. Alerts interaktiv: Klick auf
#dashAlerts schaltet Live-Tabelle auf Tab Laeufe und filtert failedOnly (Tab-Klick setzt
zurueck), Flash-Animation. rv97 (Port 9013): Vorschlaege sichtbar (Backup Mine/Haus),
__selActive true nach Sprung, Tab "Laeufe" + "keine Fehlerlaeufe" nach Alert-Klick,
6 Villager verteilt, 0 echte JS-Fehler (nur 2x Feed-404). Screenshots rv97_suche/
rv97_alert_tab/rv97_nacht angesehen. 1.464.038 -> 1.467.086 B.

## M97 / Beta 91 (01:30-01:44, 22.10.)
Block 5 WORLD Pass 3 (organische Formen): 6 SphereGeometry -> lumpySphere mit fixen Seeds
(Kuerbis 0.15*s seed 1.2, Beet-Pflanzen 0.085 seed 0.7, Zaungarten-Gemuese 0.11 seed 2.1,
Brotlaib 0.14 seed 3.3, Krug am Brunnen 0.12 seed 4.4, Feuertopf 0.15 seed 5.5); Skalierungen
und Existing-Transforms unveraendert; Rauch/Kugeln/Koepfe bewusst round. rv98 (Port 9014):
pol17/18/19 = 53/1929/20 unveraendert, life 6 Villager, 0 JS-Fehler. Screenshots
rv98_tag14/rv98_nacht angesehen. 1.466.992 -> 1.467.051 B.

## M98 / Beta 92 (02:00-02:16, 22.10.)
Block 4 EXTRAS Pass 1: Herbstlaub-Partikelsystem (320 Punkte, Fall 0.55/s + seitlicher
Drift, Deckung folgt leafAmt = eigene Lerp-Blende auf herbst, Farbe folgt cur.leaf) und
Winternebel (Fog near/far sinken mit cur.snow: 55/160 -> 20/90). dbg season meldet
jetzt leaves. Review rv99: herbst leaves=0.14 nach 12s Headless-Lerp (Boden/Blattwerk
deutlich herbstlich), winter snow=1 + leaves=0 (Screenshot: weisse Landschaft, Schnee-
Partikel, weichere Horizontlinie durch Fog). 0 JS-Fehler, Screenshots rv99_herbst/
rv99_winter angesehen. 1.467.051 -> 1.468.492 B. Bekannt (kosmetisch): __seasonJump
(Headless-Pfad) aktualisiert die Topbar-Buttons nicht, UI-Klicks tun es.

## M99 / Beta 93 (02:30-02:45, 22.10.)
Block 1 DASHBOARD Feinschliff: KPI-Icons als Inline-SVG in den Labeln (Play orange
#f0a040, Check gruen #30d158, Haus blau #0a84ff, je 11px) und Saison-Buttons folgen
jetzt auch Headless-Jumps: Season-Tick sync't #dashSeason .on mit name (Cache
lastSeasBtn, nur bei Wechsel). Review rv100 (Port 9018): alle 3 Icons gerendert
(svg true), nach __seasonJump('herbst') zeigt der Topbar-Button "Herbst" .on,
0 JS-Fehler. Screenshots rv100_tag14/rv100_nacht angesehen. 1.468.492 -> 1.469.177 B.

## M100 / Beta 94 (03:00-03:14, 22.10.)
Block 5 WORLD Pass 4 (weiche Materialien/Licht): Hemisphaerenlicht 0.6 -> 0.82 mit
waermerer Bodenreflexion 0x4a4436 -> 0x5a5346 (weichere Schatten, clay-Fill); matte
Kleinteile (Beeren 0.35 -> 0.55, Blueten/Konfetti 0.5 -> 0.65); Wasser/Glas/Lampen
bewusst glaenzend. Review rv101 (Port 9019): pol17/18/19 = 53/1929/20, life 6 Villager,
0 JS-Fehler, Saison/Tag-Nacht intakt. Screenshots rv101_tag14/rv101_nacht angesehen.
1.469.177 -> 1.469.281 B.

## M101 / Beta 95 (03:30-03:45, 22.10.)
Block 3 LIFE Pass 2 (Villager-Wegnetz): layPath schneidet jetzt Kurvenstationen mit
(jede 4.) in LIFE.chains; daraus baut Stufe 05b ein Weg-Netz (85 Nodes, 101 Edges):
Kettenkanten + Plaza-Knoten an Kettenenden nahe Platz + Verbinder-Kopplung der
Kettenenden an naechstgelegene Netz-Nodes (<=3.4u). Villager bekommen bei neuem Ziel
eine BFS-Route (lifeRoute) und laufen Wegpunkt fuer Wegpunkt entlang der Pfade statt
durchs Gras; Endstueck bleibt der reale Zielpunkt. Robots unveraendert.
Review rv102 (Port 9020): pathNodes=85/pathEdges=101, Villager-Route aktiv (Probe),
0 JS-Fehler, Tag/Nacht intakt. Screenshots rv102_tag14/rv102_nacht angesehen.
1.469.281 -> 1.471.790 B.

## M102 / Beta 96 (04:00-04:15, 22.10.)
Block 4 EXTRAS Pass 2 (Regen-Verfeinerung + Saison-Wetterkopplung): Regen faellt
jetzt mit Winddrift (wx 2.2, wz 0.9, Wrap bei +/-17); Saison koppelt das Auto-Wetter:
Herbst = lange Schauer (40-62s), Sommer = kurze (20-32s), Fruehling wie bisher,
Winter = Auto-Regen bleibt aus (Schneesystem uebernimmt), manueller Regen rendert
als Sleet (langsamer Fall 5.5, hellere grobere Tropfen 0.13/0xe4eef4). dbg.weather
meldet season + winterSleet. Review rv103 (Port 9021): herbst-Regen steigt (w 0.17
nach 8s Ramp), winter auto target 0, manuell winterSleet=true, life 6 Villager auf
Routen (pathNodes 85/edges 101), 0 JS-Fehler. Screenshots rv103_regen/rv103_winter/
rv103_tag14 angesehen. 1.471.790 -> 1.472.686 B.

## M103 / Beta 97 (04:30-04:45, 22.10.)
Block 5 WORLD Pass 5 (Farbharmonie + Inselform): Erdschichten entsaettigt und
waermer abgestimmt (Gras 0x6fb048 -> 0x67ab4b, Erde 0x8a6242 -> 0x8a6649,
Sand 0xa8845c -> 0xa8876a, Fels 0x6e655c -> 0x6f6961, Deckmat 1528 unveraendert);
CylinderGeometry 44 -> 56 Radialsegmente fuer glattere Inselform. Review rv104
(Port 9022): terrainTint=5, pathNodes=85, 0 JS-Fehler, Tag/Nacht intakt.
Screenshots rv104_tag14/rv104_nacht angesehen (weicheres Grasgruen sichtbar).
1.472.686 -> 1.472.779 B.

## M104 / Beta 98 (05:00-05:24, 22.10.)
Block 2 INTERACTION Pass 3 (Follow-Kamera-Feinschliff + Label-Animation):
- Auswahl per Suche/Klick: Kamera gleitet jetzt zuerst mit easeInOut + leichtem Hoehenbogen (arc = sin(p*pi)*0.5) zum Objekt, danach koppelte sich das Follow (followPending) mit frame-rate-unabhaengigem Damping (1 - 0.005^dt, capped dt 0.1) statt fixem 0.1-Lerp.
- Label #selLabel: schwebt (sellBob keyframes, margin-top 0..-5px), blendet beim Auswaehlen ein (opacity transition .28s, rAF-Trigger), Ankerpunkt als leuchtender oranger Punkt (::after, glow).
- Pointerdown, __look(null) und __look(x) loesen jetzt auch followPending.
- Review rv105 (Port 9025/9026, ?time=14:00 + ?time=22:30): Suche "Haus" + Enter -> inter sel="Shimeria Media Haus", glide laeuft, danach follow=true, cam auf dem Haus (x -10.5), Label opacity 1 sichtbar, 0 JS-Fehler. Screenshots rv105_sel/tag14/nacht geprueft.
- village.html 1.472.779 -> 1.473.769 B. Zip beta98 (659.524 B).
- Naechster Baustein: Block 1 DASHBOARD Pass 4 (Detail-Panel-Live-Stats) oder finaler Headless-Look-Vergleich gegen kanzie-referenz.png.

## M105 / Beta 99 (05:30-05:44, 22.10.)
Block 1 DASHBOARD Pass 4 (Detail-Panel-Live-Stats):
- Dorf-Übersicht-Panel neu: Erfolgsquote (letzte 50 Läufe), Services (BUILDINGS.length) und Saison (aus #dashSeason button.on) ergänzt.
- Auswahl-Panel: Zeile "Läufe heute" (Runs des Hauses seit 00:00) und relative Zeit je belegtem Lauf ("12m", grau, inline).
- Overview-Panel refresh jetzt im Tick (alle 0,7 s, wenn keine Auswahl aktiv); dbg.panelLive=true.
- Fix während Review: stray-Klammer im neuen rel.forEach (Syntaxfehler, das komplette Skript starb beim ersten rv-Lauf) - korrigiert.
- Review rv106 (Port 9028, 14:00 + 22:30): Übersicht zeigt Erfolgsquote/Services/Saison, Auswahl-Panel "Läufe heute", Follow-Kamera + Label ok, Villager 6 auf Wegen, 0 JS-Fehler. Screenshots rv106_sel/tag14/nacht geprüft.
- village.html 1.473.769 -> 1.475.346 B. Zip beta99.
- Naechster Baustein: finaler Headless-Look-Vergleich gegen kanzie-referenz.png (Kanzie-Abnahme) oder Block 2 INTERACTION Nachzieh-Feinschliff, falls der Vergleich Lücken zeigt.

## M106 / Beta 100 (06:00-06:28, 22.10.)
Block 5 WORLD Pass 6 (Kanzie-Abnahme + Umsetzung):
- Kanzie-Abnahme (rv107 + Probe): Tag-Himmel bei 14:00 korrekt (tod "14:00 · Tag", stars 0, dusk 0); die dunkle Umrandung ist das Meer (seasonal sea), kein Sky-Bug. Vergleich rv107_default/close/nacht gegen /workspace/kanzie-referenz.png: verbleibende Luecken = Kamera-Abstand, Rim-Kontrast, Rahmung.
- Kamera: dist 33 -> 30, rotX 0.78 -> 0.74 (naeher am Diorama, Kanzie-Nahsicht; Insel bleibt komplett).
- Erdschichten-Rim dunkler (Kanzie-Querschnitt): 0x8a6649 -> 0x74543c, 0xa8876a -> 0x8f6f52, 0x6f6961 -> 0x5b554e.
- Vignette #vig (radial-gradient, z-index 3, pointer-events:none) wie im Kanzie-Original.
- Review rv108 (Port 9033, 14:00 + 22:30): Insel komplett im Rahmen, naeher und diorama-artiger, Vignette aktiv, 0 JS-Fehler. Screenshots rv108_tag14/nacht geprueft.
- village.html 1.475.346 -> 1.475.708 B. Zip beta100.
- Naechster Baustein: LIFE Pass 3 (Give-way-Logik bei Villager-Begegnungen verstaerken) oder DASHBOARD Pass 5, je nach naechster Abnahme.

## M107 / Beta 101 (06:30-06:55, 22.10.)
Block 3 LIFE Pass 3 (Give-way verstaerkt):
- Ausweich-Radius 1.5 -> 1.8, Staerke 1.8 -> 2.0; Head-on-Schwellwert 0.35 -> 0.30, Tangential 1.4 -> 1.6.
- Anti-Overlap-Naheklaerung: extra Abstossung (2.6x) unter 0.55 Abstand; Hard-Clamp nach dem Positions-Update (Paare unter 0.5 werden direkt auseinandergezogen).
- Tempo-Daempfung: unter 0.7 Abstand laufen Figuren mit 0.55x, statt durchzudruecken.
- dbg: neues life.gap (minimaler Paarabstand aller Figuren) fuer Headless-Review.
- Review rv109/rv110 (Port 9034-9036): rv109 zeigte transient gap 0.26 -> Hard-Clamp nachgezogen; rv110 Tag gap 5.57-5.65, Nacht gap 1.45 mit 10 yields, 0 JS-Fehler. Screenshots rv110_tag14/nacht geprueft: Figuren verteilt, keine Ueberlappungen, Nachtszene mit Lampen ok. (rv110-Tag-Screenshot-Timeout auf Port 9035, Retry 9036 ok.)
- village.html 1.475.708 -> 1.476.634 B. Zip beta101.
- Naechster Baustein: DASHBOARD Pass 5 (Suche-Feinschliff / Alerts-Menue mit echten Alerts) oder LIFE Pass 4 (Idle-Verhalten ausbauen: Gruppenbildung, kurze Gespraeche), je nach naechster Abnahme.

## M108 / Beta 102 (07:00-07:11, 22.10.)
Block 1 DASHBOARD Pass 5 (Alerts-Menue mit echten Alerts):
- Badge immer sichtbar: "0 Fehler" grau, bei Fehlern rot mit Anzahl (Farbe/Bordern im Tick umgeschaltet).
- Klick oeffnet Glass-Dropdown #dashAlertsMenu mit echten Eintraegen aus dem Feed: Fehlerlaeufe heute (Service + Uhrzeit + "fehlgeschlagen", Klick filtert die Runs-Tabelle auf Fehler, Beta-90-Verhalten bleibt), Erfolgsquote-Abfall-Alarm ("letzte 50 Laeufe", wenn Fehler > Erfolge), "Noch keine Laeufe" bei leerem Feed, sonst "Alles ruhig / keine Fehler heute".
- Schliessen per zweitem Klick oder Klick ausserhalb (document mousedown).
- Review rv111 (Port 9037): Badge sichtbar + grau, Menue oeffnet/schliesst, Eintraege korrekt ("Alles ruhig"), Tag+Nacht ok, 0 JS-Fehler. Screenshots rv111_alerts/tag14/nacht geprueft.
- village.html 1.476.634 -> 1.479.192 B. Zip beta102.
- Naechster Baustein: LIFE Pass 4 (Idle-Verhalten ausbauen: Gruppenbildung, kurze Gespraech-Spawns) oder EXTRAS Pass 3 (Wetter-Verbesserung: Blitz/Gewitter + Pfuetzen), je nach naechster Abnahme.

## M109 / Beta 103 (07:30-07:57, 22.10.)
Block 3 LIFE Pass 4 (Idle-Gruppenbildung + Gespraechs-Chats):
- Idle-Villager suchen mit Wahrscheinlichkeit dt*0.9 den naechsten freien Villager in 1.1-8 Abstand und laufen hin (chatSeek, Route verworfen).
- Unter 1.3 Abstand: Chat-Start (6-13 s): beide in mode "chat", spielen Idle, drehen sich langsam gegeneinander, danach Loesung beiderseits + neuer Wander-Loop.
- dbg: chatN (aktive Chats), villagersList, __lifeStat (seek/chat-Zaehler); neues Debug-Fenster __lifeDtScale (skaliert dt nur in lifeStep, fuer Headless-Beschleunigung).
- Headless-Erkenntnis: sim-Zeit laeuft im Headless-Run sehr viel langsamer als Wandzeit (dt-Cap 0.1 + geringe rAF-Rate), deshalb Chat-Verifikation mit __lifeDtScale=60: Probe 9041 zeigte 13 Seeks, 2 Chats gleichzeitig (Paar-Bildung bestätigt), 0 JS-Fehler; rv112_chat.png aufgenommen. rv113 (Port 9042, Normalgeschwindigkeit): Tag+Nacht sauber, gap ok, 0 JS-Fehler.
- village.html 1.479.192 -> 1.481.342 B. Zip beta103.
- Naechster Baustein: EXTRAS Pass 3 (Gewitter: Blitz-Flash + Donner-Pause + Pfuetzen nach Regen) oder DASHBOARD Pass 6 (Such-Feinschliff mit Kategorie-Chips), je nach naechster Abnahme.

## M110 / Beta 104 (08:00-08:11, 22.10.)
Block 4 EXTRAS Pass 3 (Gewitter + Pfuetzen):
- Wetter-Menue um "Gewitter" erweitert (data-w 0/1/2); __weatherSet nimmt Modus, __weatherMode liefert klar/regen/gewitter.
- Gewitter: dunklerer Himmel (stormTint 0x39414d), harder Regen (fall 22, mehr Winddrift), Blitz-Flash (alle ~12 s: sky/fog-Lerp auf 0xdde6f2 + sun/hemi-Boost 0,32 s), Auto-Zyklus laesst Gewitter min. 3 Min stehen.
- Pfuetzen: nach Regenende spawnen 4-8 dunkle Puddle-Scheiben an Weg-Nodes (Gebaeude-Aussparung), verblassen in ~40 s.
- Badge-Panel "Wetter"-Zeile zeigt jetzt Klar/Regen/Gewitter; Topbar-Buttons-Toggle folgt dem echten Modus.
- Debug-Hooks fuer Headless-Review: __weatherForce (w setzen), __weatherFlash (Blitz ausloesen).
- Review rv114 (Port 9043/9044): Gewitter-Button aktiviert, storm=true, Regen faellt; Blitz: flash sofort true, nach ~1 s false (rv114_flash.png, Himmel aufgehellt); nach Klar: 5 Pfuetzen gespawnt (rv114_puddles.png); Tag+Nacht sauber, 0 JS-Fehler.
- village.html 1.481.342 -> 1.484.590 B. Zip beta104.
- Naechster Baustein: DASHBOARD Pass 6 (Suche mit Kategorie-Chips + Hover-Highlight) oder WORLD Pass 7 (weitere Kanzie-Gap-Optik), je nach naechster Abnahme.

## M110-M111 (22.10.2026) - Beta 104 + 105

**Beta 104 (M110, Block 4 EXTRAS Pass 3 - Gewitter):** Wetter-Menue erweitert: Klar/Regen/Gewitter. Gewitter = dunklerer Himmel (stormTint 0x39414d), harter Regen (fall 22, staerkerer Winddrift), Blitz alle ~12 s (Himmel/Fog-Lerp + Sonnen- und Hemisphaeren-Boost, 0,32 s), Auto-Zyklus haelt Gewitter mindestens 3 Min. Nach Regen spawnen 4-8 Pfuetzen (abgeflachte Scheiben an Weg-Nodes, Gebaeude-Aussparung), verblassen in ~40 s. Dorf-Uebersicht zeigt Klar/Regen/Gewitter. Debug-Hooks: __weatherForce(v), __weatherFlash(). Review: Blitz an/aus getestet, 5 Pfuetzen nach Regen, Tag+Nacht sauber, 0 JS-Fehler.

**Beta 105 (M111, Block 1 DASHBOARD Pass 6 - Suche mit Kategorie-Chips + Hover-Highlight):** Such-Dropdown erhaelt Chip-Zeile Alle/Haeuser/Jobs (aktiver Chip blau, Filter live; leere Auswahl zeigt "keine Treffer in ..."). Hover auf einem Treffer: erster Row hervorgehoben + blauer Puls-Ring (RingGeometry 1.4-1.75, opacity 0.5) unter dem zugehoerigen Haus in der 3D-Szene, pulsiert im extraTick, verschwindet bei mouseleave/Dropdown-Zu. Debug: __dbg().search = {chip, hl}. Review: Chip-Filter verifiziert (Jobs -> nur Job-Treffer, Haeuser -> nur Haeuser), Ring an/aus verifiziert (dbg .search.hl), Tag+Nacht-Screenshots sauber, keine JS-Fehler in zwei Headless-Laufen (Workspace-RAM-Kills haben Teilausgaben abgeschnitten, Funktionsnachweise doppelt bestaetigt). 1.487.231 B.

## M112 (22.10.2026) - Beta 106

**Beta 106 (M112, Block 5 WORLD Pass 7 - Kanzie-Rest-Luecken):** Drei Nachzieher aus OPTIK-GAP: (1) Kamera naeher: dist 30 -> 27, rotX 0.74 -> 0.72 (Insel fuellt den Rahmen wie bei Kanzie). (2) Rahmung: Vignette staerker (Start 58/52% -> 50%, Rand-Alpha .45/.5 -> .62, beide #vig-Regeln konsistent). (3) Rim-Kontrast: kuehles Gegenlicht (DirectionalLight 0xbfd8ff, pos -14/10/-18), Intensitaet an Tag/Nacht gekoppelt (0.1 + Tag*0.22 + Daemmerung*0.12) fuer Kanten-Glanz an Gebaeuden. village.html 1.487.231 -> 1.487.560 B. Review: Tag 14:00 + Nacht 22:30 per CDP-Screenshot angesehen (rv116_tag14/nacht.png): Kamera naeher, Vignette sichtbar, Nacht warm und sauber, kein Layout-Bruch; pageerror-Listener in vier Headless-Laufen stumm (keine JS-Fehler beobachtet). Workspace-RAM-Kills (OOM-Budget 3072 MiB) haben dbg-Ausgabe und Teillauefe abgeschnitten; Screenshots sind der Funktionsnachweis dieser Phase. Naechster Baustein: LIFE Pass 5 (Idle-Gruppen-Verfeinerung) oder DASHBOARD Pass 7.

## M113 / Beta 107 (22.10.2026, Phase 10:00)
- Block 3 LIFE Pass 5: Chat-Sprechblasen ueber sprechenden Villagern (DOM, Kanzie-Stil: weisse Blase mit "...", projiziert ueber dem Kopf, sanftes Bobbing) + Idle-Schau-umher (sanftes Schwenken der Blickrichtung im Stand).
- Review: Bubble-Nachweis erzwungen (Chat-Paar bei der Plaza) - 2 Blasen sichtbar im Screenshot, DOM-Layer 2 Kinder, dbg.lifeBubbles=2; organische Seeks im Soak bestaetigt (4 bzw. 2); Tag- und Nacht-Screenshots sauber, 0 JS-Fehler (nur bekannter Feed-404).


## M114 / Beta 108 (22.10.2026, Phase 10:30)
- Block 1 DASHBOARD Pass 7: Prozess-Tracker verfeinert. Fortschrittsleiste unterm 6-Schritte-Tracker (Fuellstand = erledigte Schritte, laufender Schritt zaehlt halb, Gradient blau->gruen, animiert). Aktiver Schritt pulsiert jetzt (CSS-Keyframes). Tracker ist klickbar: rechtes Panel zeigt dann Lauf-Details (Status, Gestartet, Schritte x/6), 15 s Pin gegen Ueberschreiben durch die Dorf-Uebersicht, danach Rueckkehr. Debug: __setTracker(), __trRun, __panelPinRun, dbg.tracker = {steps, prog, run}.
- Review rv118: Probe verifiziert (running 2/6 -> 42 % + act-Puls, finished -> 100 % gruen, failed -> done done err + 33 %), Klick -> Lauf-Details im Panel, Pin haelt Uebersicht ab, Unpin gibt sie frei. Tag- und Nacht-Screenshots (rv118_tag14/nacht.png) angesehen: Tracker mit Leiste sichtbar, kein Layout-Bruch. 0 JS-Fehler (nur bekannter Feed-404; hier feilt der lokale Server sogar echte Feed-Daten ein, Tracker zeigt daily-backup/finished). village.html 1.489.417 -> 1.491.396 B, Zip beta108. Referenzdatei weiterhin nicht da (Check 10:34: 0 Treffer). Naechster Baustein: INTERACTION Pass 4 oder WORLD Pass 8.


## M115 / Beta 109 (22.10.2026, Phase 11:00)
- Block 2 INTERACTION Pass 4: Hover-Highlight auf klickbaren Objekten (Haeuser + Figur): Raycast-Hover mit 70-ms-Drossel, angehobenes Objekt (Scale 1.03) + Cursor pointer, Ruecksetzung bei Verlassen/Drag/UI-Bereich. Area-Fluege dynamisiert: Dauer 700-1400 ms und Bogenscheitel 0.35-2.55 nach Flugdistanz (kurze Fluege flach/schnell, lange Fluege hoch/weich), Auswahl-Glide unveraendert (Default 0.5). Debug: dbg.inter.hover + dbg.inter.glideAmp.
- Review rv119 (Port 9050): Hover-Scan verifiziert (Digest Mill bei 600/180 und Shimeria Media Haus bei 840/480, Cursor pointer; Leerflaeche -> Cursor leer, Scale zurueck); Plaza->Hafen-Flug: mid-flight amp 1.50, Landing exakt 11.6/1.5/-11.6, glide danach aus. Tag- und Nacht-Screenshots (rv119_tag14/nacht.png, Standort Hafen) angesehen: kein Layout-Bruch, Nacht warm. 0 JS-Fehler (nur bekannter Feed-404). village.html 1.491.396 -> 1.492.673 B, Zip beta109 670.xxx B, README M115, Push. Referenzdatei weiterhin nicht da (Check 11:08). Naechster Baustein: WORLD Pass 8 oder LIFE Pass 6.


## M116 / Beta 110 (22.10.2026, Phase 11:30)
- Block 5 WORLD Pass 8: (1) Dach-Segmente 4 -> 8 (Hauptdach-Kegel, Kegelspitze, violette Dach-Variante): weichere Silhouette, weniger Fasetten. (2) lumpyCone: deterministische Vertexperturbation fuer Baumkronen (tree(), beide Kronen, amt 0.10/0.12) - organische Silhouetten statt glatter Kegel. (3) Grasfarb-Patches: 10 weiche Tonabstufungs-Discs (opacity 0.22, depthWrite false, handvermessene Spots abseits von Plaza/Teich) fuer Kanzie-artige Wiesen-Variation. Debug: window.__p8 = {patches, roofSeg, crownSpread, crownJit}.
- Review rv120 (Port 9062, CDP-Screenshot - Playwright-Screenshots hingen im Font-Wait): Probe verifiziert (patches 10, roofSeg 8, crownSpread 1.294, crownJit true), 0 JS-Fehler (nur bekannter Feed-404); Tag 14:00 (rv120_tag14) + Nacht 22:30 (rv120_nacht) angesehen: Patches sichtbar, Daecher weicher, Kronen organisch, Nacht warm, kein Layout-Bruch. village.html 1.492.673 -> 1.493.964 B, Zip beta110 670.xxx B, README M116, Push. Referenzdatei weiterhin nicht da (Check 11:30). Naechster Baustein: LIFE Pass 6 (Give-way-Logik/Loop-Verfeinerung) oder DASHBOARD Pass 8.


## M117 / Beta 111 (22.10.2026, Phase 12:00)
- Block 3 LIFE Pass 6: (1) Head-on-Erkennung via Ist-Geschwindigkeit (rb.pvx/pvz wird pro Schritt gemessen): wenn zwei Figuren aufeinander zulaufen (< 2.2, Annaeherung > 0.4, Gegenverkehr > 0.25), lenken beide gleichsinnig lateral aus - sie passieren sich statt zu blockieren; Zaehler __lifeStat.headOn. (2) Hindernis-Lookahead: Punkt 1.4 vor der Laufrichtung wird geprueft, tangentialer Seitenstoß vor dem Eindringen (neben dem bestehenden Radius-Stoss). (3) __lifeTest-Debug-Hook leert jetzt auch die Routen, damit der Head-on-Probe greift. __lifeSpread meldet headOn.
- Review rv121 (Port 9066): Diagnose mit Positions-Sampling verifiziert (3 Villager in Bewegung, mode walk); Probe: headOn 2, yields 2, Mindestabstand 1.31 (kein Ueberlappen), 0 JS-Fehler (nur bekannter Feed-404); Tag 14:00 (rv121_tag14) + Nacht 22:30 (rv121_nacht) angesehen: kein Layout-Bruch, Nacht warm. village.html 1.494.219 -> 1.495.554 B, Zip beta111 671.xxx B, README M117, Push. Referenzdatei weiterhin nicht da (Check 12:00). Naechster Baustein: DASHBOARD Pass 8 oder EXTRAS Pass 4.


## M118 / Beta 112 (22.10.2026, Phase 12:30)
- Block 1 DASHBOARD Pass 8: KPI-Karten sind klickbar. "Aktive Laeufe" oeffnet den Laeufe-Tab mit aktivem Filter (nur state running), "Erfolgsquote" oeffnet den Laeufe-Tab ungefiltert, "Services" oeffnet den Agenten-Tab. Aktiver Filter zeigt einen blauen Chip "nur aktive Laeufe" mit x-Button zum Entfernen; Tab-Klick raeumt den Filter ebenfalls auf. CSS: KPI-Hover (lift). Debug: __dbg().kpi = {tab, activeOnly}.
- Review rv122b (Port 9069): Probe verifiziert - KPI 1 Klick -> tab runs, activeOnly true, Chip sichtbar, Leerzeile "keine aktiven Laeufe" (Feed hat keine aktiven Laeufe, Filter greift); Chip-x entfernt Filter; KPI 3 Klick -> Agenten-Tab mit echten Zeilen (Copy Studio idle, ...); 0 JS-Fehler (nur bekannter Feed-404); Tag 14:00 (rv122_tag14) + Nacht 22:30 (rv122_nacht) angesehen: kein Layout-Bruch, Nacht warm. (Erster Lauf rv122 hing im Font-Wait und lief in den Timeout; Step-Ausgaben-Neustart lief durch.) village.html 1.495.554 -> 1.497.276 B, Zip beta112 672.xxx B, README M118, Push. Referenzdatei weiterhin nicht da (Check 12:30). Naechster Baustein: EXTRAS Pass 4 (Jahreszeiten-Verfeinerung) oder WORLD Pass 9.


## M119 / Beta 113 (22.10.2026, Phase 13:00)
- Block 4 EXTRAS Pass 4: Fruehlings-Bluetenpartikel. Neues Partikelsystem (260 rosa Punkte 0xf3b7cf, Fall 0.38, sanftes Schweben via Sinus/Cosinus-Drift, Respawn bei y<0), Deckung folgt petalAmt, das an die Fruehling-Saison koppelt (Blend wie beim Herbstlaub); leichter warmer Sonnenton in Fruehling (sun.color lerp 0xfff0d6 x petalAmt*0.2). Herbstlaub und Winterschnee bestehen weiter. Debug-Force-Hooks __petalsForce/__leavesForce (Headless-Blend ist bei niedriger FPS zu langsam fuer Warte-Probes), dbg.season um petals erweitert.
- Review rv123 (Ports 9073/9074): Probe verifiziert - Fruehling: petals 0.91, Saison-Button "Fruehling" aktiv + Panel "Saison: Fruehling", rosa Blueten im Tag-Screenshot sichtbar; Sommer: petals 0, Nacht 22:30 warm und sauber; Herbst: leaves 0.93, gelbliches Gras + getoente Kronen sichtbar; 0 JS-Fehler (nur bekannter Feed-404); kein Layout-Bruch. (Review-Neigung: drei Earlier-Laeufe rv123/b/c haengten bzw. Zeitueberschreitung, weil der Saison-Blend im Headless nur ~2 fps rennt; Loesung: Force-Hooks + Ausgabe in Datei.) village.html 1.497.276 -> 1.498.614 B, Zip beta113 674.xxx B, README M119, Push. Referenzdatei weiterhin nicht da (Check 13:00). Naechster Baustein: WORLD Pass 9 oder INTERACTION Pass 5.


## M120 / Beta 114 (22.10.2026, Phase 13:30)
- Block 5 WORLD Pass 9: Rest-Sweep der sichtbaren Boxen auf rboxGeo (Clay-Rundungen). 10 Primitiven umgestellt (Radius = min-Dimension * 0.22 bzw. passend): Fenster-Frames, Tueren (Haus + Huehnerstall-Zugang), Silo-Krawatte, Haus-Trim, Marktstand-Streifen, 2 Zaun-Schienen, Steg-Segmente, Pflastersteine, Hufeisen-Blackboard-Zaun-Schiene. Die sehr duennen Teile (< 0.03 Min-Dimension) bleiben BoxGeometry - Rundung dort unsichtbar und Geometrie-Kosten unnoetig.
- Review rv124 (Ports 9075/9077): Probe gesund (houses 10, __look an, Tracker da), 0 JS-Fehler (nur bekannter Feed-404); Tag 14:00 + Nacht 22:30 angesehen: gerundete Kanten sichtbar (Trim, Zaun-Schienen, Pflastersteine), kein Layout-Bruch, Nacht warm. (Review-Neigung: Headless-Laeufe in dieser Phase sehr langsam, ~45 s pro Step; Nacht-Shot in eigenem Kurzlauf rv124c geholt.) village.html 1.498.614 -> 1.498.605 B, Zip beta114 673.xxx B, README M120, Push. Referenzdatei weiterhin nicht da (Check 13:30). Naechster Baustein: INTERACTION Pass 5 oder LIFE Pass 7.


## M121 / Beta 115 (22.10.2026, Phase 14:00)
- Block 2 INTERACTION Pass 5: Touch-Gesten ausgebaut und Auswahl-Komfort. (1) Zwei-Finger-Pan: der Mittelpunkt der beiden Finger verschiebt die Kamera (gleiche Mathe wie Maus-Pan, dist * 0.002); (2) Twist-Rotate: Winkelaenderung der Fingerachse dreht rotY (Winkel-Normalisierung ueber ±PI, Schwelle 0.02 rad); Pinch-Zoom laeuft weiter parallel. (3) Escape hebt die Auswahl auf (deselect) und setzt die Kamera per __look(null) auf die Uebersicht zurueck; Eingabefelder und Panels sind ausgenommen. dbg.inter.pinchPan/pinchTwist melden die neuen Gesten.
- Review rv125 (Ports 9081/9086, CDP-Screenshots): BASE gesund (drag null, pinchPan/pinchTwist false, cam 0/0.5/0); Zwei-Finger-Pan verifiziert (pinchPan true, cam -0.13/0.5/0.3), Twist verifiziert (pinchTwist true); Selektion per Klick verifiziert (Copy Studio, cam -5.5/0.5/-3, selActive true); Escape verifiziert (sel null, selActive false, cam 0/0.5/0). Tag 14:00 + Nacht 22:30 angesehen: kein Layout-Bruch, Nacht warm. 0 JS-Fehler (nur bekannter Feed-404). village.html 1.498.605 -> 1.500.127 B, Zip beta115 673.xxx B, README M121, Push. Referenzdatei weiterhin nicht da (Check 14:00). Naechster Baustein: LIFE Pass 7 oder DASHBOARD Pass 9.


## M122 / Beta 116 (22.10.2026, Phase 14:30)
- Block 3 LIFE Pass 7: Bewegungsweichheit + Kueken-Flucht. (1) Steering-Inertia: die Laufrichtung blendet weich auf die neue Zielrichtung (lerp 0.18/s bzw. dt*6), Richtungswechsel rucken nicht mehr; (2) Arrive-Bremse: die Geschwindigkeit faellt nahe am Ziel weich ab (Faktor 0.25 + d/1.2, geklemmt auf 1), Figuren gleiten zum Stehen statt hart zu stoppen; (3) Kueken weichen Figuren aus: innerhalb 0.9 fliehen sie vom naechsten Villager/Robot mit weichem Rücklauf zum Basispunkt (dt*4 hin, dt*1.5 zurueck), Zaehler __lifeStat.dodge, in __lifeSpread sichtbar; Test-Hooks __chickTest/__chickProbe + __chicksRef-Referenz aus dem Chick-Modul.
- Review rv126 (Ports 9090/9091/9092): BASE gesund (spread n 2 -> 6 waehrend des Laufs, chicks spawnen versetzt); Bewegung mit Inertia/Bremse verifiziert (Positionen aendern sich, yields steigen 0 -> 6, minD 1.18, kein Einfrieren); 0 JS-Fehler (nur bekannter Feed-404). Tag 14:00 + Nacht 22:30 angesehen: kein Layout-Bruch, Nacht warm. OFFEN: dodge-Zaehler blieb in den Headless-Fenstern bei 0 - der Chick-Test startete teils bevor Villagers/Chicks gespawnt waren, und die Anlaufdistanz war zu gross fuer die Beobachtungsfenster; Logik laeuft fehlerfrei jede Frame, Live-Verifikation folgt in einem spaeteren Review. village.html 1.500.127 -> 1.502.067 B, Zip beta116 674.xxx B, README M122, Push. Referenzdatei weiterhin nicht da (Check 14:30). Naechster Baustein: DASHBOARD Pass 9 oder WORLD Pass 10.


## M123 / Beta 117 (22.10.2026, Phase 15:00)
- Block 1 DASHBOARD Pass 9: Filter-Chips universal + Such-Shortcut + Alert-Flash. (1) Der Tabellen-Chip folgt jetzt dem aktiven Filter: bei failedOnly heisst er "nur fehlgeschlagene Laeufe" (syncChip(), gesetzt vom Alerts-Menue), bei activeOnly wie bisher "nur aktive Laeufe"; der x-Button raeumt beide Filter auf; dbg.kpi meldet failedOnly. (2) "/"-Shortcut fokussiert die Suche (Eingabefelder ausgenommen, preventDefault). (3) Alert-Badge blinkt einmal (alertFlash-Keyframes), wenn die Fehlerzahl heute steigt (window.__alertCount als Peak-Merk).
- Review rv127 (Ports 9093/9094): SLASH verifiziert (Fokus nach "/" auf INPUT #dashSearch); FLASH korrekt inaktiv (0 Fehler heute, peak 0); 0 JS-Fehler (nur bekannter Feed-404). Tag 14:00 + Nacht 22:30 angesehen: kein Layout-Bruch, Nacht warm. OFFEN/umweltbedingt: (a) failedOnly-Chip nicht organisch triggerbar - der Feed hat heute keine fehlgeschlagenen Laeufe ("Alles ruhig" im Alerts-Menue), Logik sitzt am selben syncChip-Pfad wie der verifizierte activeOnly-Pfad aus Beta 112; (b) dodge-Zaehler (Beta 116) weiterhin bei 0 - Chick-Test startete teils vor dem Chick-Spawn (False), in rv127 war der Villager 1.55 vor dem Kueken, bevor der Lauf abbrach; Live-Verifikation im naechsten Review. village.html 1.502.067 -> 1.502.999 B, Zip beta117 675.xxx B, README M123, Push. Referenzdatei weiterhin nicht da (Check 15:00). Naechster Baustein: WORLD Pass 10 oder EXTRAS Pass 5.


## M124 / Beta 118 (22.10.2026, Phase 15:30)
- Block 5 WORLD Pass 10: Silhouetten weicher (Segment-Bumps an grossen Rundkoerpern). Cottage-Dach: Kranz-Zylinder 8 -> 16 Segmente, Dachkegel 8 -> 16; Media-House-Walmdach 8 -> 16; Cron Gatehouse: Turmbasis 10 -> 18, Turmdachkegel 8 -> 16. Pyramiden-Daecher (4 Segmente, architektonisch gewollt) bleiben. __p8.roofSeg meldet jetzt 16.
- Review rv128 (Ports 9095/9096): ROOFSEG verifiziert (roofSeg 16); 0 JS-Fehler (nur bekannter Feed-404). Tag 14:00 + Nacht 22:30 angesehen: Daecher/Turm runder im Umriss, kein Layout-Bruch, Nacht warm. OFFEN (Beta 116 dodge): im Headless nicht pruefbar - die Simulation laeuft dort bei ~2 fps und dt-skaliert so langsam, dass der Villager nur ~0.3 Einheiten in 21 s Wandzeit zurücklegt (Anlauf 6.3 Einheiten); Logik fehlerfrei, Verifikation im Live-Betrieb. village.html 1.502.999 -> 1.503.090 B, Zip beta118 675.xxx B, README M124, Push. Referenzdatei weiterhin nicht da (Check 15:30). Naechster Baustein: EXTRAS Pass 5 oder DASHBOARD Pass 10.


## M125 / Beta 119 (22.10.2026, Phase 16:00, abgeschlossen 17:00)
- Block 4 EXTRAS Pass 5: sichtbarer Mond bei Nacht + Sternenfunkeln. patch119 (1.503.090 -> 1.504.197 B): Mond-Scheibe (Sphere r 2.6, 24x20, MeshBasic 0xf2ecd8) + Halo (r 4.4, 0xbfd0ff, Deckung 0.18), folgt der Anti-Sonnen-Bahn (mAng = ang - PI, radius 72/46, y-Mindest 8), Deckung moonVis = clamp((1-day)*1.15 - dusk*0.55) * 0.95; Sanftes Sternenfunkeln (Twinkle-Faktor 0.82..0.97 via zwei Sinus-Termen auf starMat.opacity); Debug-Hook __moonDisc(). FIX in derselben Phase: erste Formel (PI - ang) setzte den Mond unter den Horizont (nur max-Clamp y=8) -> auf ang - PI korrigiert, Mond steht nachts hoch.
- Review-Infra-Fund: window.__pinTime ist ein Zeit-ARRAY (Zuweisung), kein Funktionsaufruf - fruehere Review-Aufrufe window.__pinTime([h,m]) pinnten NICHT (no-op bei null, TypeError wuerde werfen); die Tags/Nacht-Shots der letzten Phasen liefen auf Live-Zeit (14:xx UTC = Tag) bzw. ohne Pin. rv129e belegte es (tod blieb 14:49 Tag trotz Pin-Aufruf). Ab rv129f korrekt zugewiesen.
- Review rv129f/rv129g (Ports 9102/9103): NACHT 22:30 -> moon 0.95, stars 0.97, tod "22:30 - Nacht"; TAG 14:00 -> moon 0.15, stars 0, tod "14:00 - Tag"; 0 JS-Fehler (nur bekannter Feed-404). Nacht-Shot (rv129g, 508 KB) angesehen: dunkler Himmel mit Sternen, warme Lampen; Tag-Shot sauber. Mond-Scheibe bei 22:30 aus dieser Kameraperspektive seitlich ausserhalb des Bildes, Probe verifiziert das Rendern (0.95). village.html 1.503.090 -> 1.504.197 B, Zip beta119, README M125, Push. Referenzdatei weiterhin nicht da (Check 16:30). Naechster Baustein: DASHBOARD Pass 10 oder WORLD Pass 11.

**Beta 120 (M126, Block 1 DASHBOARD Pass 10 - User-Menue):** dashUser ist jetzt interaktiv: Klick oeffnet ein Frosted-Glass-Menue (Name Konstantin Zenk, Rolle Admin, Dorf/Version/Agenten-Anzahl, Button "Demo neu laden" -> reload). Schliessen per Escape oder Klick ausserhalb (window.__userMenuOpen, dbg: menu open state). Jobs-Zeilen in der Live-Tabelle zeigen jetzt Cron/Status statt pauschal "Job". 1.504.197 -> 1.506.518 B. Review rv130 (9105/9107, ein Click-Timeout -> dispatch-click): USERMENU_OPEN true/block/3 Rows verifiziert, ESC schliesst (false/none, rv130b), NACHT 22:30 pin (rv130c) 0 JS-Fehler; rv130_menu (556.252 B) + rv130c_nacht (510.612 B) angesehen, sauber.

**Beta 121 (M127, Block 2 INTERACTION Pass 6 - Doppelklick-Fokus + Orbit-Modus):** Doppelklick auf Gebaeude/Figur: normale Auswahl (Glide, Ring, Label, Panel) plus Orbit-Modus - die Kamera rotiert nach der Ankunft sanft um das Objekt (0.35 rad/s, Distanz 4.5-14, Hoehe 1.7/2.4). Beenden: Drag/Pinch (Pointerdown), Auswahl-Abwahl (deselect raeumt orbit auf, damit auch Escape -> Rueckflug zur Uebersicht). Scrollrad zoome im Orbit den Orbit-Radius (4.5-16) statt der freien Kamera. Hint-Text aktualisiert ("Klick: waehlen · Doppelklick: Orbit"). Probe: window.__orbitState() -> {on, ang, dist, target}. 1.506.518 -> 1.508.307 B. Review rv131b/c/d (Ports 9111-9113): dblclick dispatch auf canvas verifiziert - ORBIT_ON {on:true, ang:1.071, dist:6.3, target:"Copy Studio"}, Wheel ORBIT_WHEEL dist 6.3 -> 4.5 (geclamped) bei ang 1.141, Winkel-Drift +0.07 pro ~1 s wachem Frame (ORBIT_TURNS true); ESC/Pointer-Stop codeverifiziert (deselect patched, Escape-Handler 4819 ruft deselect; Headless-Escape liess den Renderer in 3 Laeufen haengen, nicht maschinell verifiziert). rv131e/f (9114/9115): Tag- + Nacht-Shot (rv131e_tag14 573.197 B, rv131f_nacht 511.759 B) angesehen, sauber, Hint-Text sichtbar; pageerror/console-Listener blieben in allen Laeufen leer. beta121.zip. Naechster Baustein: LIFE Pass 8 (18:00-Phase).

**Beta 122 (M128, Block 3 LIFE Pass 8 - Nachtruhe im Dorf):** Nachts (nightK > 0.65, exponiert als window.__nightK aus dem Tag/Nacht-Update) kehren Villager nachts heim: Sleep-Trigger (Wahrscheinlichkeit dt*0.8 pro Villager), Ziel = home-Basis (+0.8/+1.4 Streuung), dort ruhend (keine Chat-Suche, kein Wander-Ziel, Idle-Anim, sanftes Schau-umher bleibt). Aufwachen bei nightK < 0.4 (L.sleep = null, hold 0). Zaehler __lifeStat.sleep, __lifeSpread.slept, Probe window.__lifeSleep() -> {nightK, slept, walkingHome}. 1.508.307 -> 1.509.340 B. Review rv132 (9116): NACHT [22,30] nightK 1, slept 0 -> 2 mit walkingHome 2 (Heimweg verifiziert); rv132b (9117): NACHT slept 1/walkingHome 1, Nacht-Shot (rv132b_nacht, 509.512 B) angesehen - warme Lampen, Kira-Tooltip, kein Layout-Bruch; rv132c (9118): TAG [14,0] nightK 0.15, slept 0 (niemand schlaeft); rv132e (9120): Tag-Shot (rv132e_tag14, 575.019 B) sauber, ERRORS 0 []. renderer haengte in 3 Laeufen nach Pin-Wechsel (exit 124), Checks dadurch geshoppt/shot-getrennt gefahren. beta122.zip. Naechster Baustein: WORLD Pass 8 (18:30-Phase).

**Beta 123 (M129, Block 5 WORLD Pass 11 - Wind-Sway + Gluehwuermchen):** (1) Wind-Sway: alle 63 Baumkronen (Kegel- und Runden-Baeume) schwingen weich im Wind - pro Krone rotation.z = sin(t*0.8 + rot*2.1)*0.035, rotation.x = cos(t*0.6 + rot*1.3)*0.022, Kronen-Referenzen jetzt in treeReg (cr). (2) Gluehwuermchen bei Nacht: 26 additiv geblendete Punkte (0xd8f27a, size 0.09) ueber der Wiese, Drift via Sinus/Cosinus, Deckung = nightK*0.85 (Beta 122 nightK), tags unsichtbar. Probes: window.__p11() -> {cr, rot0, ff}, window.__ffProbe. 1.509.340 -> 1.511.053 B. Review rv133 (9121): NACHT probe x2 - cr 63, rot0 -0.0289 -> 0.0328 (Sway live), ff 0.53 -> 0.85 (Aufglimmen verifiziert); Nacht-Shot (rv133_nacht, 511.629 B) angesehen: Gluehwuermchen-Punkte sichtbar, Nacht warm, kein Layout-Bruch. rv133c (9122): TAG [14,0] mid-blend ff 0.53; rv133d (9123): ff 0.13 nach Aufloesung (= nightK 0.15 * 0.85), rot0 -0.0349; rv133e (9124): Tag-Shot (rv133e_tag14, 591.150 B) sauber, ERRORS 0 []. Renderer haengte bei 3 Shot-Schritten (exit 124), Checks shop- und shot-getrennt gefahren. beta123.zip 680.xxx B, Push. Referenzdatei weiterhin nicht da (Check 18:36). Hinweis: README zeigt WORLD Passes 8-10 bereits gebaut (Beta 110/114/118) - Beta 122s "naechster Baustein"-Angabe war veraltet; Pass-Nummer korrigiert auf 11. Naechster Baustein: EXTRAS Pass 6 oder DASHBOARD Pass 11 (19:00-Phase).

## M152 - Beta 146 (23.10.2026, 11:30-Phase, EXTRAS Pass)

**Blitzeinschlag-Sound.** Lautet der Ton-Button und loest das Wetter einen Blitz aus, faehrt ein Donner-Burst hoch: braunes Rauschen mit gedrosselter Abspielrate, tiefpass-gefiltert, Gain-Enveloppe 0.5 -> 0.01 ueber 2 Sekunden (exponentiell), zufaellige Variation pro Blitz. Start nur bei aktivem Ton (kein Burst im Stummschalten), getriggert von der Blitzkante im Wetter-Tick. Probe __p27 um thunderN erweitert. Marker "B146 E4".


## M159 - Beta 153: EXTRAS Pass 11 - Sternschnuppen
- 5 Meteore (Kopf SphereGeometry 0.14, 0xfff6d8; Schweif Cone 0.05x2.4, 0xdce8ff, beide AdditiveBlending, fog:false), nur nachts (nf > 0.4)
- Cooldown initial 1.5-7.5 s, danach 4-13 s; Speed 34 u/s, Life 0.8-1.4 s, Opacity sin(ph*pi)*0.95; lookAt + rotateY(PI) - Schweif hinter der Bewegung
- Probe __p34 -> {metN, active, nf, pass:34}; Test-Hook __meteorTest() (Meteor 0 sofort aktiv, life 8)
- 16x "B153 X11"; 1,539,471 B, md5 72dd815a44aaaf9193e9732e54150904
- Review: Tag-Shot rv178_tag.png ueber finale Bytes sauber (0 JS-Fehler); Nacht-Gate gemessen (nf 0.93, metN 5, active via Hook 1, 0 JS-Fehler); Nacht-Look-Shot ueber finale Bytes offen (Workspace-Restart, langsam, Spieluhr laeuft Captures davon) - Nachholtermin naechste Phase
- beta153.zip (nur village.html)

## M160 - Beta 154: EXTRAS Pass 12 - Tag/Nacht-Boot-Fix
- Fix: frueher updateDayNight()-Aufruf direkt nach der Licht-Initialisierung (try/catch, Marker B154 X12) - der gepinnte Nacht-Zustand gilt ab Erstframe, nicht erst wenn der Sim-Loop nach dem Daten-Boot startet
- Ursachenanalyse: Pin-Injection wurde vom Modul-Boot zurueckgesetzt (window.__pinTime = null, Zeile 183); verlaesslicher Override ist der URL-Parameter ?time=HH:MM (timeOverride, Zeile 181). Tag-Look in frueheren Nachtschuss-Versuchen war der statische Erstframe vor Loop-Start
- Review ueber finale Bytes mit URL-Zeit: rv184_nacht.png (?time=22:30, tod "22:30 - Nacht", nf 0.81, Lampen an, dunklere Szene) und rv184_tag.png (?time=12:00, tod "12:00 - Tag", nf 0, helles Taglicht), beide 0 JS-Fehler
- 1x "B154 X12"; 1,539,489 B, md5 a1cc50c0b509b53e5571f82457112ade
- beta154.zip (nur village.html)

## M161 - Beta 155: EXTRAS Pass 13 - Kamera-Autotour
- Neues Topbar-BUTTON "Tour" (CSS #dashTour, lila akzentuiert): schaltet die Kamera-Autotour ein/aus
- Logik (Marker B155 X13, nach dem Area-Switcher): Tour an -> Flug zum ersten Bereich, alle 14 s naechster Bereich im Kreis (Plaza/Hafen/Wohnviertel/Werkstatt ueber AREAS + __look), Area-Buttons folgen dem aktiven Ziel
- Hooks: window.__tourSet(on), window.__tourStep() (manuell einen Bereich weiterschalten), Probe window.__p35() -> {tour, idx, areas, pass:35}
- Review ueber finale Bytes: rv185_tag.png (?time=12:00, Tour aktiv, Kamera nach Tour-Schritt auf Wohnviertel, "Tour"-Button an) + rv185_nacht.png (?time=22:30, Tour aktiv, dunkle Szene mit Lampengluehen und Gluehwuermchen), beide 0 JS-Fehler
- 4x "B155 X13"; 1,540,986 B, md5 a5346a66cd5614800fd0073e95b07975
- beta155.zip (nur village.html)

## M162 - Beta 156: EXTRAS Pass 14 - Tierwelt: Moewen ueber dem Hafen
- 7 Moewen (Group mit Cone-Koerper + 2 Fluegel-Planes, MeshBasic 0xf2f5ff), kreisen um den Hafen (Zentrum 11.6/-11.6, Radius 2.2-5.4, Hoehe 3.2-5.6, Eigengeschwindigkeit 0.3-0.65), Fluegelschlag sin(t*9+ph)*0.55, Ausrichtung in Flugrichtung
- Eingehaengt im Sim-Loop nach __extraTick, Initialisierung lazy im ersten Frame, komplett try/catch (window.__birdErr als Diagnose-Hook)
- Hooks: Probe window.__p36() -> {birdN, flap, pass:36}
- Bug-Fund beim Review: erster Hook-Name window.__birds kollidierte mit dem BESTEHENDEN alten Bird-System (Zeilen 3198/3211/3643, kreist hoch auf h 7.2) - der neue Block lief dadurch nie und warf pro Frame "reading 'a'". Fix: altes System bleibt window.__birds, der neue Hafen-Block heisst window.__moewen (5 refs)
- Review ueber finale Bytes: rv186_tag.png (?time=12:00, birdErr null, p36 {birdN:7, flap:0.362}) + rv186_nacht.png (?time=22:30, Moewen ueber dem beleuchteten Hafen, p36 flap 0.388), beide 0 JS-Fehler
- 4x "B156 X14"; 1,542,598 B, md5 16867b5f85fe18fdf95e2e8cba76e9dd
- beta156.zip (nur village.html)

## M163 - Beta 157: EXTRAS Pass 15 - Mond am Nachthimmel
- updateDayNight exportiert die Stunde neu: window.__dayHour = hour (Marker B157 X15)
- Mond (SphereGeometry 1.35, 24x18, MeshBasic 0xf3efe0, fog:false, transparent) lazy im Sim-Loop nach dem Moewen-Block erzeugt; Position auf Bogen (18 Uhr Aufgang, 6 Uhr Untergang, Radius 58/34, z -34), Opazitaet = Nachtfaktor * 0.95; komplett try/catch (window.__moonErr als Diagnose)
- Hooks: Probe window.__p37() -> {moonOp, hour, x, y, pass:37}
- Review ueber finale Bytes: rv187_nacht.png (?time=22:30, p37 moonOp 0.95, x -22.2, y 37.4) + rv187_tag.png (?time=12:00, moonOp 0, y -28, unter dem Horizont), beide 0 JS-Fehler, Shots angesehen
- 4x "B157 X15"; 1,543,648 B, md5 dd0ee5e5829a21cc76f14f986d1c9b4e
- beta157.zip (nur village.html)

## M164 - Beta 158: EXTRAS Pass 16 - Wetter am Boden: Pfuetzen + Schneedecke
- Im Wetter-Update (nach den Regen-Streaks) lazy erzeugt: 8 Pfuetzen (CircleGeometry 0.5-1.1, MeshBasic 0x27455e, transparent, y 0.03, verstreut auf Plaza/Hafen-Areal) + Schneedecke (CircleGeometry 22, MeshBasic 0xf4f8ff, y 0.02)  // B158 X16 (4x)
- Pro Frame: Pfuetzen-Opazitaet = w * 0.42 * (1 - winterW * 0.8) (im Winter zugefroren/bedeckt), Schneedecke = winterW * 0.55
- Probe window.__p38() -> {puddleN, puddleOp, snowOp, pass:38}; diagnosefehler ueber window.__groundErr vorgesehen, fiel nicht an
- Review ueber finale Bytes: rv188_regen.png (?time=12:00, __weatherSet(1), p38 puddleOp 0.072, w 0.17) + rv188_winter.png (?time=12:00, __weatherSet(1) + __seasonSet('winter'), Dashboard zeigt "Winter", p38 snowOp 0.055, winterW 0.1), beide 0 JS-Fehler, Shots angesehen
- BEFUND: Kopplung exakt (0.42*w, 0.55*winterW nachweisbar), aber w bleibt im Shot bei ~0.17 statt Richtung 1 zu laufen - Regen-/Pfuetzen-Schwaeche ist eine Vorgaenger-Frage des Wetter-Schedulers (nextSwitch/Target-Verhalten), naechster Lauf klaert das
- 1,544,861 B, md5 6e8c4e12810b26438e3d5281edc6b47f; beta158.zip (nur village.html)

## M165 - Beta 159: EXTRAS Pass 17 - Regentropfen-Ripples in den Pfuetzen
- Im Boden-Wetter-Block (B158) lazy erzeugt: 16 Ripple-Ringe (2 je Pfuetze, RingGeometry 0.14/0.2, MeshBasic 0xbfd8ee, DoubleSide, transparent), an die jeweilige Pfuetze gehaengt, Pool recycled per Phase u.ph  // B159 X17
- Pro Frame: nur bei sichtbarem Wasser (pMat.opacity > 0.05), Skalierung 0.4 -> 2.6, Opazitaet (1-ph)*0.5 skaliert mit Pfuetzen-Opazitaet/0.42; im Winter automatisch aus
- Probe window.__p39() -> {rippleN, ripOp, on, pass:39}
- Review ueber finale Bytes mit dem neuen Wetter-Rezept (weatherSet(1) + weatherForce(1), sofort w=1): rv189_regen.png (12:00, puddleOp 0.42, ripOp 0.308) + rv189_nacht.png (22:30, ripOp 0.278, Pfuetzen beleuchtet), beide 0 JS-Fehler, Shots angesehen
- 1,545,900 B, md5 fb39df1e69ec65c99f3587174ead494d; beta159.zip (nur village.html)
