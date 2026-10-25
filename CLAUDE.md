# Kurbel – Projektnotizen für Claude

Kurbel ist eine Indoor-Radtraining-App (Zwift-Ersatz) als **eine einzige HTML-Datei** (`index.html`, ca. 1,1 MB).
Sie läuft in Chrome auf dem MacBook und wird über GitHub Pages veröffentlicht. Nutzer: Max (spricht Deutsch).

## Arbeitsweise

- **Sprache:** UI-Texte und Antworten auf Deutsch, kurz und nicht technisch.
- **Datei nie komplett neu schreiben.** In `index.html` sind Leaflet 1.9.4 und three.js r158 eingebettet (sehr lange Zeilen).
  Änderungen immer gezielt per Suchen/Ersetzen an eindeutigen Stellen.
- In der three.js-Zeile wurde das erste `console.warn` durch `void 0,` ersetzt – nicht „reparieren“.
- Nach jeder Änderung: Seite in Playwright laden und auf `pageerror` prüfen (siehe Tests), dann committen und pushen.
- Design: Apple-artig weiß, Akzent ruhiges Blau `#2563eb` (Hover `#1d4ed8`, im dunklen Fahrt-Design `#4c8dff`), flache Buttons ohne 3D-Kante, Systemschrift (SF) wie auf der Startseite, Gewichte max. 600–700, dünne 1-px-Rahmen statt Schatten (Block „Ruhiger Apple-Look“ am Ende des Menü-CSS).
  **Keine Emojis** (auch nicht in Knöpfen oder Meldungen), klare Text-Buttons, farbige Punkte statt Icons; Werte-Fenster ist ein kleiner Symbol-Knopf (SVG) neben „Video“.
- Oben in der Leiste ersetzt ein neues Fenster (Dashboard, Plan, Fahrten, Menü) das offene (Klick-Listener auf `header` vor `#btnSetup`).

## Aufbau von index.html (Suchbegriffe)

| Bereich | Wichtige Namen |
|---|---|
| Einstellungen | `settings` (localStorage `kurbel.settings`), `saveSettings`, `LS` |
| Strecken | `KA_DATA`/`KA_NAMES` (4 komoot-Touren Karlsruhe), `PRO_DATA`/`PRO_META`/`PRO_COBBLES` (Paris–Roubaix mit 30 Pavé-Sektoren, Alpe d'Huez, Ventoux, Tourmalet, Galibier), `routeFromPoints`, `parseGPX`, `eleAt`, `gradeAt`, `latLonAt`, `cobbleAt` |
| Workouts | `W(dur,p0,p1)`, `reps`, `FAV_WORKOUTS`, `BUILTIN_WORKOUTS` (Kategorien `WO_CATS`), `parseZWO`/`parseERG`, Ordner-Sync `scanFolder` (Unterordner = Kategorie), Editor `openEditor` |
| Trainingspläne | `PG` (Generatoren), `PLANS` (ftp12, base8, short6; `weeksDef` mit Slots thr/z2/vo2/long/test/rec), `PLAN_WORKOUTS`, `PLAN_SCIENCE` (Quellen), `plan` (localStorage `kurbel.plan`, v2), `planNext`, `planMarkFromRide`, `openPlan` |
| Tagesform | `DF`, `dayBias()`, `renderDayForm`, Vorschlag aus TSB `formSuggest` |
| Bluetooth | `BleTrainer` (FTMS 0x1826; Fallback Tacx FE-C `6e40fec1…`, Wahoo `a026e005…`), `PowerMeter` (CPS 0x1818, Nullabgleich 0x2A66 Opcode 0x0C), `connectHR`, `DemoTrainer`, `DemoPowerMeter` |
| Kalibrieren | `trainer.spinDown` (FTMS 0x13 + Status 0x2ADA; FE-C Seite 1/2), `calTrainer`, `calPM`, Dialog `#calDlg` |
| Fahrt | `ride`, `startRide` (Zeit startet erst bei ersten Watt), `tick`, `physics`, `controlTrainer` (ERG/SIM), PowerMatch (Regelkreis Ziel vs. Powermeter, `live.pmOffset`, Plausibilität `checkPmDev`) |
| Export | `rideRecord` (inkl. `routeId`, `woId`, `tgt`), `buildFIT` (mit GPS-Koordinaten, sub_sport 58 = virtuell), `buildTCX`, Garmin-Trick (manufacturer 1, product 3121) |
| Strava | OAuth `#sAuth` (gehostet: gleicher Tab, `approval_prompt=force`; Datei: neuer Tab + Adresse einfügen), Rückkehr-Handler setzt `window.__stravaReturn`, `stravaUpload` (Duplikat → neue Kennung), immer VirtualRide wenn GPS |
| Verlauf | IndexedDB `kurbel` v2 (`kv`, `rides`), `saveRideToHistory`, `openHistory`, Backup/Restore |
| Dashboard | `openDash`, `ftpEstimate` (Test < 6 Wochen → hoch; sonst CP-Modell aus frei gefahrenen Abschnitten, ERG zählt nicht), `pmc` (CTL 42 / ATL 7), `lineChart`, `barChart`; Palette `#2563eb` / `#e8400f` (validiert) |
| Setup-Menü | `fillSelects`, `renderRouteCards`, `renderWoCards`, `woCats`, `pickWorkout`, `renderPlanBanner` |
| Karte | Leaflet, `TILES` (Esri Straße/Satellit, CyclOSM, OpenTopoMap – OSM und CARTO blockieren lokal geöffnete Dateien) |
| Startseite | `KurbelLanding` (three.js-Kurbel), Glue mit `enter(intent)` |

## Testen

Playwright (Chromium) gegen die lokale Datei oder `python3 -m http.server`:
- Startseite ausblenden: `KurbelLanding.running=false; document.getElementById('landing').style.display='none'`
- Demo: `#btnConn` → `#btnDemo`, dann `#btnStart`; Fahrt simulieren mit Schleife über `tick()` (live.power setzen, `lastTick=performance.now()-250`).
- Bluetooth mocken über `navigator.bluetooth` per `add_init_script` (FTMS-, FE-C- und Wahoo-Varianten).
- Strava mocken über `page.route` auf `oauth/authorize` und `oauth/token`.
- FIT prüfen mit `@garmin/fitsdk` (Decoder, `checkIntegrity`).

## Offen / Ideen

- Powermeter vs. Trainer: Max sah 350 W getreten, 130 W angezeigt. Diagnose-Anzeige („Trainer misst … W“) und Plausibilitätsschutz sind drin – echte Werte von Max stehen noch aus.
- Kalibrierung und FE-C/Wahoo-Steuerung nur mit Mocks getestet.
- Eventuell eigene Domain für GitHub Pages (dann Strava-Callback-Domain anpassen und Backup neu einspielen).
- Mögliche weitere Strecken: Flandern-Rundfahrt, Champs-Élysées.
