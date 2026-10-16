# REA-Kanzie-Notizen (Stufe 05d, 15.10.2026)

Analyse mit REA 6.5.0 (`rea-agents`), Kommando `analyze-web-bundle` über einen
lokalen CDP-Chromium gegen https://kanzie.com/nexus/village/index.html?v=1.19.4.

Evidence-ID: `ev_df394a765bd1c5b7a9e0cff199b0e47ade12c5fc127f61ad2d3401c03a93bb6a`
Umfang: 13 Skripte analysiert, 675.601 AST-Knoten, 0 Parse-Fehler.

## Kernfunde

- Kanzie-Dorf holt Agentenzustände über zwei API-Endpunkte:
  `/api/village/souls/bench` und `/api/village/souls/graph`
  (fetch in Skript `scr_8b22dc94…`). "Souls" = Agenten mit Aktivitätszustand;
  bench = wer gerade auf der Bank sitzt, graph = Beziehungen/Abhängigkeiten.
- Weitere Fetches: `./demo-config.json` und `./changelog.json`.
  Konfiguration und Versionshistorie liegen als eigene Dateien neben dem Dorf.
- 57 Endpoint-Kandidaten (überwiegend WebGL-Erweiterungen, niedrige Konfidenz),
  24 Chunk-Edges über 13 gebündelte Skripte.

## Ideen für unsere Stufen (Kanzie-Prinzip-kompatibel)

1. Stufe-06-Kandidat: eigene "Souls"-Schnittstelle im Collector (localhost-only),
   z. B. `/api/village/souls/bench` + `/graph`, mit Aktivitätszustand pro Agent
   (arbeitet / wartet / schläft). Demo weiter nur mit synthetischem Feed.
2. `demo-config.json` + `changelog.json` als Muster für die Village-Versionierung
   (Anzeige "v1.x" im HUD).
3. Beziehungs-Graph der Agenten als spätes Extra.

## Grenzen der Analyse

Source-Maps nicht angefordert; der Digest liefert nur Shas, keine URLs der
13 Skripte; keine WebMCP-Deklarationen gefunden.

## Stufe-06-Umsetzung (15.10.2026, ~22:55)

- Kanzie-"Souls" (bench/graph) als eigene visuelle Schicht nachgebaut:
  Status-Orb je Gebaeude (running/finished/failed/idle) aus dem Feed abgeleitet,
  Wartebank bei fehlgeschlagenen Runs, Changelog-Chip (changelog.json-Muster).
- Die Souls-Endpoint-UI selbst (Server-Graph der Beziehungen) bleibt Stufe-07-
  Kandidat fuer den Collector (localhost-only).

## Stufe-07-Umsetzung (16.10.2026, ~00:40)

- Kanzie-/souls/graph (Beziehungs-/Abhaengigkeits-Netz) als Beziehungsfäden
  nachgebaut: 10 Boegen zwischen logisch verbundenen Gebaeuden, Atmung im Tick,
  Aktiv-Puls 9 s bei Soul-Modewechsel, nachts heller.
- Klick-Karte um Souls-Zustand erweitert (REA-Idee: Zustand pro Agent sichtbar).

## Stufe-08-Umsetzung (16.10.2026, ~02:50)

- Kanzie-Live-Stats-Dashboard als HUD-Karte links oben nachgebaut: aktive
  Laeufe, Erfolgsquote (letzte 50 Runs), belegte Haeuser, letzte Aktivitaet.
  Daten rein aus Feed + Souls-Schicht, Update im extraTick (0,5 s Takt).

## Stufe-09-Umsetzung (16.10.2026, ~04:40)

- Wetter-System (Regenschauer mit Zufallsphasen, Licht-/Himmel-Daempfung,
  HUD-Wetterzeile) als atmosphaerische Erweiterung auf Kanzie-Niveau.

## Stufe-10-Umsetzung (16.10.2026, ~06:42)

- Minimap mit Live-Soul-Dots nachgebaut (Canvas, Gebaeudepositionen skaliert,
  Zustandsfarben wie die Orbs). Lernpunkt: __snap fotografiert nur das
  WebGL-Canvas, DOM-Overlays brauchen echte Playwright-Screenshots.
