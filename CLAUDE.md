# Kurbel – Projektnotizen für Claude

Kurbeln (früher „Kurbel“) ist eine Indoor-Radtraining-App (Zwift-Ersatz) als **eine einzige HTML-Datei** (`index.html`, ca. 1,1 MB).
Sie läuft in Chrome auf dem MacBook und wird über GitHub Pages veröffentlicht: https://nicmaxii.github.io/kurbeln/ (Repo `nicmaxii/kurbeln`, früher `Kurbel`). Strava-Titel: „Indoor kurbeln: <Strecke> · <Workout>“. Nutzer: Max (spricht Deutsch).

## Arbeitsweise

- **Sprache:** UI-Texte und Antworten auf Deutsch, kurz und nicht technisch.
- **Datei nie komplett neu schreiben.** In `index.html` sind Leaflet 1.9.4 und three.js r158 eingebettet (sehr lange Zeilen).
  Änderungen immer gezielt per Suchen/Ersetzen an eindeutigen Stellen.
- In der three.js-Zeile wurde das erste `console.warn` durch `void 0,` ersetzt – nicht „reparieren“.
- Nach jeder Änderung: Seite in Playwright laden und auf `pageerror` prüfen (siehe Tests), dann committen und pushen.
- Design: dunkel & Lime (seit Okt. 2026): Farbvariablen `--s0/--s1/--s2` (Flächen), `--tx/--tx2`, `--ln`, `--acc` #EDFF9A (Lime) mit `--onacc` #0C0C0D, `--sel/--onsel` für gewählte Chips; Block „Kurbeln: Dunkel & Lime“ am Ende des ersten `<style>`. Keine festen Hex-Farben in neuen Regeln – immer Variablen. Schrift Archivo schmal (font-stretch 88 %, `--uifont`), Zahlen in JetBrains Mono (`--numfont`), Logo „kurbeln“ klein in JetBrains Mono (`--logofont`). Startseite dunkel, 3D-Kurbel bleibt. Karte Standard „Kurbeln dunkel“ = Esri-Straßenkarte mit SVG-Filter `#kdark` (Helligkeits-Tabelle: Grund dunkel, Straßen hellgrau, Schrift hell; Klasse `tiles-dark`), Strecke immer in Lime-Tönen (`gradeCol`) – die Esri-Dark-Gray-Kacheln gingen nur bis Zoom 16 und waren auf Retina weiß. Achtung: auf Retina sind Kacheln 128 px (detectRetina) – nie feste Kachelgrößen per CSS setzen. Strava-Knöpfe im normalen Lime-Stil, kein Orange. Flache Buttons, dünne 1-px-Rahmen. Datenfarben nur Grau → Lime → Weiß (`ZC`, `gradeCol`, `CAT_STYLE`, `SLOT_COLOR`, `DC`), kein Rot/Blau/Grün. Startseite mit Körnung (`.lgrain`) und dezenten Speed-Linien hinter der Kurbel (`#ltrail`, 2D-Canvas halbe Auflösung hinter dem transparenten WebGL-Canvas, `drawFx`; Max hat sie aus Spur/Echo/Linien/Drehung gewählt). Fußleiste ohne Beschriftungen.
  **Keine Emojis** (auch nicht in Knöpfen oder Meldungen), klare Text-Buttons, farbige Punkte statt Icons; Werte-Fenster ist ein kleiner Symbol-Knopf (SVG) neben „Video“.
- Handy/Tablet: Block „Tablet & Handy“ (Media Queries 1180 px / 760 px) am Ende des Menü-CSS; Workout-Box dort unten über der Steuerleiste. iPhone/iPad: kein Web-Bluetooth → Hinweis auf App „Bluefy“.
- Einführung beim ersten Besuch: `TOUR`, `startTour` (localStorage `kurbel.tour`), erneut über „Einführung“ im Menü.
- Außerhalb der Fahrt (`syncRideUI`, Body-Klasse `riding`): keine Fußleiste (Höhenprofil/Workout), keine Umschaltung Karte/Werte/Video, kein Werte-Fenster-Knopf; Menü dafür größer. Menü-Knopf heißt „Kurbeln“. Street View startet immer aus.
- Oben in der Leiste ersetzt ein neues Fenster (Dashboard, Plan, Fahrten, Menü) das offene (Klick-Listener auf `header` vor `#btnSetup`).

## Aufbau von index.html (Suchbegriffe)

| Bereich | Wichtige Namen |
|---|---|
| Einstellungen | `settings` (localStorage `kurbel.settings`), `saveSettings`, `LS` |
| Strecken | `KA_DATA`/`KA_NAMES` (7 komoot-Touren Karlsruhe inkl. „Linksrheinisch mit Volldampf“, „Eis in Plittersdorf“, „Pfalz“; Hin-und-zurück-Stücke herausgeschnitten), `PRO_DATA`/`PRO_META`/`PRO_COBBLES` (Paris–Roubaix mit 30 Pavé-Sektoren, Alpe d'Huez, Ventoux, Tourmalet, Galibier), `routeFromPoints`, `parseGPX`, `eleAt`, `gradeAt`, `latLonAt`, `cobbleAt` |
| Workouts | `W(dur,p0,p1)`, `reps`, `FAV_WORKOUTS`, `BUILTIN_WORKOUTS` (Kategorien `WO_CATS`), `parseZWO`/`parseERG`, Ordner-Sync `scanFolder` (Unterordner = Kategorie), Editor `openEditor` |
| Trainingspläne | `PG` (Generatoren), `VB`/`vbFit` (Bausteine für Max’ Pläne, füllen auf Gesamtdauer auf), `KRAFT` + `VB.off` (Krafttraining/Rennen: `off: true`, nicht fahrbar, im Plan per Klick bzw. Banner „Erledigt“ abhaken), `PLANS` (ftp12, vo2b8 „VO2max-Booster“, base12 „Base“, gf12 „Peak Gran Fondo“, crit6 „Crit“, hit4 „Rad HIT“, base8, short6; `weeksDef` mit Einträgen `[slot, workout, tag]`, Slots thr/z2/vo2/long/test/rec/sit/kraft/race, `P.days`), `PLAN_WORKOUTS`, `PLAN_SCIENCE` (Quellen), `plan` (localStorage `kurbel.plan`, v2), `planNext`, `planMarkFromRide`, `PLAN_CATS` (Pläne haben `cat`: grundlage/leistung/rennen/zeit; Übersicht mit Reitern, `settings.planCat`), `openPlan(mode)` (ohne = aktiver Plan, `"list"` = Übersicht, Plan-ID = Vorschau „Plan ansehen“ mit Mini-Profilen + Detailkasten `#pvInfo`), `planWeeksHtml`, `startPlan` |
| Tagesform | `DF`, `dayBias()`, `renderDayForm`, Vorschlag aus TSB `formSuggest` |
| Bluetooth | `BleTrainer` (FTMS 0x1826; Fallback Tacx FE-C `6e40fec1…`, Wahoo `a026e005…`), `PowerMeter` (CPS 0x1818, Nullabgleich 0x2A66 Opcode 0x0C), `connectHR`, `DemoTrainer`, `DemoPowerMeter` |
| Kalibrieren | `trainer.spinDown` (FTMS 0x13 + Status 0x2ADA; FE-C Seite 1/2), `calTrainer`, `calPM`, Dialog `#calDlg` |
| Fahrt | `ride`, `startRide` (Zeit startet erst bei ersten Watt), `tick`, `physics`, `controlTrainer` (ERG/SIM), PowerMatch (optional, Standard aus seit `pmV2`: Regelkreis Ziel vs. Powermeter, `live.pmOffset`, `checkPmDev`). Standard: Trainer steuert, Powermeter läuft mit (`pmHist`, `pmFresh`, Sample-Index 7, `pmStats`/`pmNote` → Zusammenfassung + Strava-Beschreibung) |
| Fahrt-Anzeige | `updateHUD`; oben Mitte `#wbox` (links Ziel, Mitte aktuelle Watt groß `#wNow` + Powermeter `#wPm`, rechts Intervall-Restzeit `#wRem`; große Watt-Kachel links `#hPbox` dann ausgeblendet), Steigung `#grade` rechts; Vollbild `#btnFull`; Steuerleiste `#controls` ohne Schwierigkeit (`#grpDiff` versteckt, Wert kommt aus dem Menü), `#biasLbl` (`biasText`: „Intensität +6 % · +12 W“ bzw. „Fester Wert 150 W“, Handy ohne Wort) in einer Zeile |
| Export | `rideRecord` (inkl. `routeId`, `woId`, `tgt`), `buildFIT` (mit GPS-Koordinaten, sub_sport 58 = virtuell), `buildTCX`, Garmin-Trick (manufacturer 1, product 3121) |
| Strava | OAuth `#sAuth` (öffnet neuen Tab; gehostet kommt Strava auf die Seite zurück, Rückkehr-Handler „Rückkehr von Strava“ liest `?state=kurbel&code=`; Datei: Adresse einfügen), `stravaUpload` (Duplikat → neue Kennung), immer VirtualRide wenn GPS |
| Geist | `ghost`, `loadGhost` (schnellste frühere Fahrt mit gleicher `routeId`, Samples-Index = Sekunde), `ghostGap`, `updateGhostHUD`, Kachel `#ghostBox`, Schalter `#cGhost` (`settings.ghost`) |
| Verlauf | IndexedDB `kurbel` v2 (`kv`, `rides`), `saveRideToHistory`, `openHistory`, Backup/Restore |
| Dashboard | `openDash`, `ftpEstimate` (Test < 6 Wochen → hoch; sonst CP-Modell aus frei gefahrenen Abschnitten, ERG zählt nicht), `pmc` (CTL 42 / ATL 7), `lineChart`, `barChart`; Palette `#2563eb` / `#e8400f` (validiert) |
| Setup-Menü | `fillSelects`, `renderRouteCards`, `renderWoCards`, `woCats`, `pickWorkout`, `renderPlanBanner` |
| Karte | Leaflet in `#mapBox`; Fahrtrichtung oben + leicht schräg (`TILT`, perspective + rotateX) über `mapRot`/`sizeMapRot` (Container vergrößert, Fahrer bei 62 % Höhe, Steuerelemente aus dem Container herausgezogen), Knopf `#btnHeadUp`;  `TILES` (Esri Straße/Satellit, CyclOSM, OpenTopoMap – OSM und CARTO blockieren lokal geöffnete Dateien) |
| Startseite | `KurbelLanding` (three.js-Kurbel), Glue mit `enter(intent)` |

## Testen

- Versionsanzeige im Menü neben „Einführung“ (`.ver`, „Version 1.N“) bei jeder Veröffentlichung um 0.1 hochzählen – hilft bei Cache-Problemen.
- Kontrast: alle Texte/Knöpfe in allen Ansichten mindestens 4,5:1 (auch im Hover-Zustand). Text auf Lime immer `--onacc`.

Playwright (Chromium) gegen die lokale Datei oder `python3 -m http.server`:
- Startseite ausblenden: `KurbelLanding.running=false; document.getElementById('landing').style.display='none'`
- Demo: `#btnConn` → `#btnDemo`, dann `#btnStart`; Fahrt simulieren mit Schleife über `tick()` (live.power setzen, `lastTick=performance.now()-250`).
- Bluetooth mocken über `navigator.bluetooth` per `add_init_script` (FTMS-, FE-C- und Wahoo-Varianten).
- Strava mocken über `page.route` auf `oauth/authorize` und `oauth/token`.
- FIT prüfen mit `@garmin/fitsdk` (Decoder, `checkIntegrity`).

## Offen / Ideen

- Technische Kennungen bleiben `kurbel` (localStorage, IndexedDB, Strava-`state`, `KurbelLanding`).

- Powermeter vs. Trainer: große Abweichung bei Max, PowerMatch klappte nicht → jetzt Standard „Trainer steuert, Powermeter läuft mit“.
- Street View: bei Max noch nicht gelaufen – genaue Meldung steht aus (Schlüssel gilt pro Adresse: gehostet und lokale Datei getrennt eintragen).
- Kalibrierung und FE-C/Wahoo-Steuerung nur mit Mocks getestet.
- Eventuell eigene Domain für GitHub Pages (dann Strava-Callback-Domain anpassen und Backup neu einspielen).
- Mögliche weitere Strecken: Flandern-Rundfahrt, Champs-Élysées.
