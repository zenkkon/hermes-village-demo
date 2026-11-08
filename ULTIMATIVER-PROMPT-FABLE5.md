# ULTIMATIVER PROMPT: HERMES VILLAGE (Fable 5)

Dies ist der komplette Bauauftrag. Als EINEN Prompt in Fable 5 eingeben (oder Block für Block nachschieben, Reihenfolge beibehalten). Ziel: eine einzige HTML-Datei, die das Dorf als interaktive 3D-Szene mit Live-Dashboard zeigt. Kein Lorem ipsum, keine Platzhalterdaten, nichts Blocky.

---

## ROLLE UND ZIEL

Du bist ein 3D-Web-Entwickler. Baue eine einzelne, eigenständige HTML-Datei (Three.js über CDN erlaubt) namens Hermes Village: eine kleine, lebhafte Dorf-Insel, die als visuelles Cockpit für einen KI-Agenten-Server (Hermes) dient. Die Optik ist weich und toy-artig (Blender-Clay-Render-Look), NICHT Minecraft/Blocky. Alle Zahlen im Dashboard kommen aus echten Zustandsdaten der Szene (Agenten, Services, Jobs), niemals aus erfundenen Statik-Strings.

## GLOBALER LOOK (gilt für alles)

- Weiche Materialien: MeshStandardMaterial mit roughness 0.85 bis 1, keine harten Facetten, flatShading off. Rundungen statt Ecken: Kegel/Kugeln/Kapseln, lumpyCone-Deformation (organische Stauchung der Silhouette, kein Siebeneck).
- Farbpalette: sattes Grün für die Insel (0x3f8f52-Bereich), warme Wandtöne (Creme, Terracotta, Salbei) mit drei Akzent-Häusern in Violett (0x9278cc), Teal (0x5fa8b5) und Magenta-Rosa (0xd184a8). Dächer rot/ocker/salbei-grün (0xc46a4a, 0xa8593f, 0x6b8f7a).
- Licht: warmes Key-Light + Hemisphere, weiche Schatten (PCFSoft), Dunst/Fog am Horizont. Nacht: dunkelblauer Himmel (0x0d1b3d-Bereich), warme Fenster- und Laternenpunkte, Objekte nachleuchtend statt schwarz.
- Kamera: 3/4-View, Start rotY -2.0, Pitch 0.66, Distanz 52, Insel füllt den Frame, große weiche Berg-Silhouette hinter dem Dorf am oberen Bildrand.
- UI: Frosted-Glass (backdrop-filter blur, rgba-Weiß/Nacht-Dunkel, 1px helle Border, große Radien), Apple-Feeling: Press-Feedback (scale 0.97 bei :active), Spring-Übergänge, Reduced-Motion-Fallback.

## BLOCK 1: WORLD

- Insel: abgerundeter Erd-Disk (Torte) mit Grasschicht oben, Erdschichtrand sichtbar, weiche Kante. Durchmesser füllt die Kamera.
- Gebäude (jeweils Cottage-Baukasten: quaderähnlicher, aber gerundeter Korpus + Satteldach + Details): Copy Studio (violett), Skill Forge (teal), Ledger Haus, Queue Haus (magenta-rosa), Relay Haus, Webhotel, Mine mit Stollen und Gleisen, Stilt House am Wasser, Windmühle mit drehenden Flügeln, Sternwarte als Glaskuppel, Bibliothek, Teich mit Steg, Hafenecke mit Leuchtturm (rot-weiß gestreift, blinkend) und Boot.
- Gebäudedetails: Fenster mit Fenstersims, Fensterläden, Dachgauben, Kamine mit Rauch bei Aktivität, Wege als helle, weich gebogene Pfade zwischen den Gebäuden, Wegweiser-Schilder ("Werkstatt", "Observatory", "Hafen", "Teich", "Bibliothek", "Mühle").
- Bäume: Mischwald, ~50% Rundkronen, 40% Kegel, ein paar Birken, deterministisch gestreut, keine Kollision mit Haus-Footprints.
- Berge: drei Rim-Berge am Inselrand (weiches Blau-Grau mit Schneekappen) plus drei größere Backdrop-Berge hinter der Insel, vom Dunst weich gezeichnet.
- Leben im Detail: Vögel am Himmel, Möwen am Hafen, Laternen, kleine Zäune, Blumenbeete.

## BLOCK 2: DASHBOARD

- Topbar (Frosted-Glass, volle Breite): Logo "Hermes Village", Suche (filtert Gebäude), Bereichs-Switcher, Wetter-Menü (Sonne/Regen/Nebel/Gewitter), Live-Uhr (echte Uhrzeit, updatebar), Alert-Badge, Tag/Nacht-Toggle, User-Avatar.
- 3 KPI-Karten (Glass, klickbar, mit Mini-Sparkline): 1) Aktive Läufe ("Agenten arbeiten"), 2) Services ("Häuser im Dorf"), 3) Erfolgsrate (Anteil abgeschlossener Jobs). Werte live aus der Szene berechnet, bei Klick zoomt die Kamera auf die zugehörigen Gebäude.
- Rechtes Detail-Panel: erscheint bei Objekt-Klick, Glass, zeigt Name, Rolle, Jobs, Live-Stats des Objekts.
- Prozess-Tracker unten links: 6 Schritte mit Fortschrittsmarkierung (z. B. Aufträge annehmen, Planen, Bauen, Testen, Liefen, Berichten), der aktuelle Schritt pulsiert.
- Tabbed Live-Tabelle unten: Tabs "Agenten", "Runs", "Jobs". Zeilen kommen aus der Simulationslogik (Agenten-Objekte, Job-Queue, Run-Historie mit echten Job-Namen), Status-Farbpunkte, nichts Statisches.

## BLOCK 3: INTERACTION

- Steuerung: Linksdrag = Pan, Rechtsdrag (oder 2-Finger) = Orbit/Rotate, Scroll/Pinch = Zoom mit Grenzen. Weiches Lag (Kamera-Easing), kein Ruckeln.
- Klick auf ein Objekt (Gebäude, Agent, Boot): Kamera gleitet sanft hin und folgt dem Objekt, pulsierender Ring am Boden unter dem Objekt, schwebendes Label über dem Objekt, Glass-Panel mit Live-Stats öffnet rechts. Klick ins Leere räumt die Auswahl auf.
- Area-Switcher (Topbar oder unten): 5 Bereiche - Plaza, Hafen, Wohnviertel, Werkstatt, Wasserfall. Klick fliegt die Kamera in einem Bogen (arc) zum Bereich, Dauer ~5-6 s, Landung mit weichem Ease.
- Doppelklick = Orbit-Fokus auf das Objekt, WASD = Kamera bewegen, Taste 0 = Home-Reset (Pitch 0.66, Startwinkel).

## BLOCK 4: LIFE

- Alles bewegt sich endlos in Loops: Agenten (kleine Kapsel-Figuren mit Kopf, weich, ~10 Stück) laufen auf den Wegen zwischen Gebäuden, besuchen "ihre" Gebäude, pausieren, laufen weiter.
- Give-Way-Logik: Begegnen sich zwei Figuren, weichen beide deterministisch aus (Seitenversatz), NIEMALS Überlappung oder Durchdringung; auch keine Kollision mit Gebäuden, Bäumen, Zäunen.
- Dauerbewegung: Windmühlenflügel drehen, Boot schaukelt am Hafen, Rauch steigt aus Kaminen mit aktiven Agenten, Vögel kreisen, Wasserfunkeln, Flaggen wehen.
- Loop-Sicherheit: jede Bewegung ist periodisch definiert (Phasenwinkel + Zeit), keine Anhäufung, keine Drift, auch nach Stunden stabil.

## BLOCK 5: EXTRAS

- Wetter: 4 Modi mit weichen Übergängen (Easing ~1 s): Sonne, Regen (Partikel + nasse, dunklere Böden), Nebel (Fog-Dichte 0.07 bis 0.12, Berg-Silhouette weicher), Gewitter (dunklerer Himmel, Blitzlicht, Vögel schneller/hektischer).
- Jahreszeiten: Sommer, Herbst (warmes Laub, getönte Kronen), Winter (Schneekappen auf Dächern ~27%, entsättigte Wiese, dünnerer Nebel).
- Tag/Nacht: Auto-Zyklus + manueller Toggle. Nacht = warme Fensterlichter, Laternen, Sternenhimmel, Mondlicht-Farbe; alles bleibt lesbar, nichts schwarz.
- Sound-Toggle (stumm default), Screenshot-/Clean-Shot-Modus (UI ausblenden), Impressum-lose Versionsanzeige (v1.2-Stil) unten rechts.

## DATEN UND REALISMUS

- Die Welt ist die Datenquelle: ein Agent = ein Bewohner mit Haus und Jobliste; Services = Gebäude mit Rollen; Jobs = Queue mit Namen, Cron/Status. KPIs, Tracker und Tabelle lesen genau diese Objekte, keine erfundenen Konstanten.
- Sprach- und Textstil: kurze, reale Bezeichnungen (Job-Namen, Status-Wörter), keine Blindtexte, keine leeren Tabs.

## ABNAHMEKRITERIEN (Definition of Done)

1. Eine Datei, lädt ohne Build-Schritt, 0 JavaScript-Fehler in der Konsole.
2. Erster Eindruck im 3/4-View wie ein Blender-Clay-Render: weiche Formen, keine Blockwürfel, Insel füllt den Frame, Berg-Silhouette oben.
3. Alle 5 Blöcke sichtbar umgesetzt (World, Dashboard, Interaction, Life, Extras) - jeder Block einzeln demonstrierbar.
4. Dashboard zeigt ausschließlich echte Zustandsdaten; Tab-Wechsel, KPI-Klick und Area-Wechsel funktionieren.
5. Tag- UND Nachtszene je ein Screenshot: nachts lesbar und warm, tags satt und weich.
6. 2 Minuten laufen lassen: keine Überlappungen, keine Figuren in Häusern, Loops stabil.
