# HYROX Logboek — projectcontext

## Wat dit is
Eén self-contained `index.html` bestand (geen build-stap, geen dependencies behalve
Google Fonts via CDN) dat een interactief trainingslogboek toont voor een HYROX-schema.
Gehost via GitHub Pages. Er bestaan **twee aparte repo's/varianten** van dit zelfde idee:

1. **hyrox-logboek-Sascha** — Sascha's eigen, gepersonaliseerde trainingsschema.
2. **joachimprovement-logboek** — het originele 12-weekse "Open-Level" HYROX-plan
   (Base/Pace/Accelerate/Prime/Race), bedoeld om door te sturen naar een bevriende
   personal trainer (Joachim, joachimprovement.be). Visuele stijl is gebaseerd op
   diens website: donker/industrieel, crème tekst, dun goud accent, Anton-lettertype
   voor titels (zware hoofdletter-wordmark "JOACHIMPROVEMENT" bovenaan), scherpe hoeken
   (geen border-radius), subtiele diagonale streeptextuur op de achtergrond.

**Belangrijke afspraak**: functionele/structurele verbeteringen (bv. een bugfix, een UX-
verbetering aan hoe velden werken) horen in **beide** varianten toegepast te worden.
Inhoud of stijl die specifiek is voor één variant (Sascha's eigen schema-inhoud, of de
Joachimprovement-huisstijl) blijft apart. Bij twijfel welke variant bedoeld is: vragen.

## Technische opbouw (beide varianten, zelfde patroon)
- Alles in één `.html` bestand: `<style>` inline, data + logica in één `<script>` blok.
- Data zit als een JS-array `const WEEKS = [...]` bovenaan het script, hardcoded.
  - Sascha's versie: elke week heeft `trainings: [{num, intensity, type, workout}]`
    (geen vaste dagen — "Training 1" t/m "Training 5", bewust herordend zodat nooit
    twee ZWARE sessies na elkaar staan; `intensity` is 'licht'/'gemiddeld'/'zwaar').
  - Joachim-versie: elke week heeft `days: [{day, type, workout, notes}]` met echte
    weekdagen (Maandag–Vrijdag), plus een `block`-veld (BASE/PACE/ACCELERATE/PRIME/RACE).
- **Opslag**: `localStorage` (key-prefix `hyrox_log_...`), want dit draait als gewone
  statische site zonder backend/account. Geen window.storage of Claude-specifieke API's
  meer — dat was een eerdere iteratie die afhankelijk was van de Claude-omgeving en is
  bewust losgekoppeld.
- **Per-beweging invoer**: workouts worden geparsed in regels; elke regel die een
  "beweging" is (niet een rust-regel of rondestructuur-header) krijgt een eigen
  invulveld. De functie `metricForMovement(text)` kiest het juiste label/placeholder:
  - bevat kg/lbs/KB/DB/sandbag/barbell/dumbbell/kettlebell/plate/sled/carry/wall ball
    → "Gewicht"
  - bevat row/ski/bike/erg/assault → "Niveau/weerstand"
  - bevat run/jog/sprint/strides → "Tempo/snelheid"
  - fallback → ook "Gewicht" (veiligste aanname voor HYROX-bewegingen)
- **Rondes**: `detectRounds(workoutLines)` herkent "X RFT" / "X Rondes" / "X Rounds" in
  de eerste regels van een workout. Bij een duidelijk rondegetal (2–10) krijg je per
  ronde een apart tijdveld (géén gewicht per ronde — gewicht hoort bij de beweging,
  niet bij de ronde, want dat verandert normaal niet tussen rondes binnen één sessie).
  Geen herkend rondegetal (AMRAP/EMOM/ladder) → één "Eindresultaat (tijd/score)"-veld.
- **Aerobic/rust-dagen** (type bevat "aerobic" of "rust") krijgen altijd de simpele
  indeling: Tijd + Tempo/snelheid + Notities — geen per-beweging gewichtvelden, want
  dat is niet relevant voor een loopsessie.
- Notities-veld staat altijd onderaan elke sessie, ongeacht het type.

## Bekende losse eindjes / mogelijke volgende stappen
- Geen automatische GitHub-sync vanuit de Claude-chat-omgeving; dat is net de reden om
  dit project nu in Claude Code te zetten.
- Bij het uploaden/hernoemen op GitHub moet het bestand exact `index.html` heten in de
  root van de repo, anders geeft GitHub Pages een 404 ("File not found").
- Beide varianten delen dezelfde kernlogica (parsing, opslag, rendering) maar hebben nu
  losse, volledig gedupliceerde bestanden — geen gedeelde codebase. Dat zou op termijn
  opgeschoond kunnen worden (bv. gedeelde JS-module), maar is bewust simpel gehouden
  omdat het via GitHub Pages zonder build-proces moet blijven werken.
