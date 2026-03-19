# CLAUDE CODE OPDRACHT: VerspaanMeester

## Wie ben ik
Boris, student aan de Leidse Instrumentmakers School (LiS). Junior verspaaner.
ADHD/ADD/autisme — baat bij kort, puntig, visueel, geen muren tekst.
Machines: conventionele draaibank, CNC draaibank (Fanuc + Mazak/Mazatrol), CNC bewerkingscentrum (Fanuc + Mazak).
Meetmiddelen: schuifmaat, micrometer, klokje + statief.
Grootste frustratie: theorie snappen maar niet naar de machine kunnen vertalen.

## Wat is dit project
Een werkplaats-companion webapp genaamd VerspaanMeester. Het is een gereedschap dat je meeneemt naar de machine — geen quiz-app maar een labschrift + rekenmachine + naslagwerk + diagnose-tool.

## Technische eisen
- Single-page app: één `index.html` bestand
- React 18 via CDN (geen build tooling nodig)
- Babel standalone voor JSX transpilatie in de browser
- localStorage voor persistent logboek
- Geen API calls, geen externe dependencies behalve React CDN + Google Fonts
- Mobile-friendly (gebruiken aan de machine op telefoon)
- Dark theme, industrieel-functioneel design (denk Mitutoyo/Heidenhain interface)
- Deploy op GitHub Pages

## Architectuur
```
index.html
├── <head> met meta tags, fonts (IBM Plex Sans + JetBrains Mono), basis CSS
├── <body>
│   ├── <div id="root">
│   └── <script type="text/babel">
│       ├── Storage helper (localStorage wrapper)
│       ├── Design tokens (kleuren, fonts, border-radius)
│       ├── Knowledge base (materialen, diagnose-data, opdrachten)
│       └── App component
│           ├── State management (useState voor tabs, logboek, calc, etc.)
│           ├── Tab: Opdrachten (werkplaatsopdrachten met logboek per stap)
│           ├── Tab: Rekenmachine (draaien + frezen calculator)
│           ├── Tab: Diagnose (symptoom → oorzaak flowcharts)
│           ├── Tab: Naslagwerk (formules, params, materialen, toleranties, G-code)
│           └── Tab: Logboek (overzicht van alle notities)
```

## Design tokens
```javascript
const T = {
  bg0: "#0a0c0f",      // deepest background
  bg1: "#12151a",       // card background dark
  bg2: "#1a1e26",       // card background
  bg3: "#222832",       // button background
  border: "#2a303c",
  accent: "#c8965a",    // warm brass/bronze — hoofdkleur
  accentLight: "#daa872",
  accentDim: "#8a6838",
  green: "#5a9e6f",     // succes, meten
  red: "#b85c5c",       // fouten, urgentie
  blue: "#5a7fa8",      // draaien, theorie
  blueLight: "#7a9ec8",
  text: "#d0d4dc",
  textMuted: "#8891a0",
  textDim: "#5a6272",
  white: "#eaedf2",
  mono: "'JetBrains Mono', monospace",
  body: "'IBM Plex Sans', system-ui, sans-serif",
};
```

## Storage
```javascript
const store = {
  get(key) {
    try { const v = localStorage.getItem(key); return v ? JSON.parse(v) : null; }
    catch { return null; }
  },
  set(key, val) {
    try { localStorage.setItem(key, JSON.stringify(val)); } catch {}
  }
};
```
Storage key: `"vm3-logboek"` — bevat object met per opdracht-ID een object met stap-index → tekst + lastUpdate timestamp.

## De 5 tabbladen

### 1. OPDRACHTEN
Het hart van de app. 15 werkplaatsopdrachten die je aan de echte machine uitvoert.

Elke opdracht heeft:
- `id`: uniek (W01-W15)
- `proces`: draaien | frezen | cnc | meten | opspanning | project
- `titel`: korte naam
- `niveau`: 1-3
- `duur`: geschatte tijd
- `machines`: ["conventioneel", "cnc", "elk"]
- `doel`: één zin wat je leert
- `theorie`: compact theorieblok (pre-formatted text)
- `stappen`: array van { type, tekst } objecten

Stap-types en hun betekenis:
- `bereken` (⊕, blauw): reken iets uit met de rekenmachine
- `hypothese` (?, accent): schrijf je voorspelling op VOORDAT je iets doet
- `uitvoeren` (▶, wit): voer iets uit aan de machine
- `observeer` (◉, licht): noteer wat je ziet/hoort/voelt
- `meet` (△, groen): meet en noteer getallen
- `conclusie` (=, lichtblauw): trek je conclusie, vergelijk met hypothese

Elke stap heeft een textarea waar de gebruiker notities invult. Die worden persistent opgeslagen.

Filter-knoppen: Alle | Draaien | Frezen | CNC | Meten | Opspanning | Project

De 15 opdrachten:

#### W01: Je eerste bewuste snede (draaien, niv.1, 30min)
Leer verband toerental/diameter/snijsnelheid.
Theorie: n = (1000 × vc) / (π × Dm), CSS op CNC.
Stappen: bereken → hypothese → uitvoeren → observeer → meet → conclusie

#### W02: Neusradius en oppervlak (draaien, niv.2, 45min)
Experimenteel: Rmax ≈ 125 × fn² / rε
Twee zones met verschillende fn, meet het verschil.

#### W03: De stijfheidsproef (draaien, niv.2, 30min)
Doorbuiging ~ L³. Draai op twee uitsteeklengtes, meet het verschil.

#### W04: Materiaal leren lezen (draaien, niv.2, 20min/materiaal)
ISO P vs M vs N herkennen aan spanen, geluid, krachten.

#### W05: CSS vs. vaste RPM (draaien/cnc, niv.2, 30min)
G96 vs G97 bij vlakken. Meet het finish-verschil centrum vs. rand.

#### W06: Meelopend vs. tegenlopend (frezen, niv.2, 40min)
Climb vs conventional. Voel/meet het verschil in kracht en finish.

#### W07: Instelhoek experiment (frezen/cnc, niv.3, 45min)
kr = 90° vs 45° vs ronde plaat. Krachtenrichting ontdekken.

#### W08: Snijdata optimaliseren (frezen, niv.2, 30min)
Wetenschappelijke methode: één variabele tegelijk veranderen.

#### W09: Meetonzekerheid ontdekken (meten, niv.1, 20min)
Meet 5× hetzelfde met micrometer, dan 5× met schuifmaat. Vergelijk spreiding.

#### W10: Tolerantie vertalen naar proces (meten, niv.2, 30min)
H7 = 21 µm band. Meetonzekerheid < ¼ tolerantie. Kan dat met je schuifmaat?

#### W11: G-code anatomie: Fanuc (cnc, niv.2, 45min)
Lees een Fanuc-programma regel voor regel, teken op papier, vergelijk met single block.
Bevat compleet voorbeeldprogramma:
```
O0001 (PROGRAMMANAAM)
N10 G21 G40 G99
N20 T0101
N30 G96 S200 M03
N40 G50 S3000
N50 G00 X52.0 Z2.0
N60 G01 X50.0 F0.15
N70 G01 Z-30.0
N80 G00 X55.0
N90 G00 Z2.0
N100 M30
```

#### W12: Gereedschapsoffsets begrijpen (cnc, niv.2, 30min)
Geometry vs wear offset. Meet-corrigeer-meet cyclus. Radius vs diameter denken.

#### W13: Mazatrol vs. G-code (cnc, niv.3, 45min)
Programmeer hetzelfde werkstuk in beide. Ontdek grenzen van conversational programming.

#### W14: Klokken en uitlijnen (opspanning, niv.2, 30min)
TIR meten met meetklok. 3-klauw vs 4-klauw vs spantang.

#### W15: Complete as: tekening → product (project, niv.3, 2-3 uur)
Integratieproef: getrapte as met Ø25 h6, Ø30 vrij, Ø20 Ra 1.6, M16×2 draad.

### 2. REKENMACHINE
Twee modi: draaien en frezen.

**Draaien invoer:** vc (m/min), Dm (mm), fn (mm/omw), ap (mm), rε (mm), kc (N/mm²)
**Draaien resultaat:** n (omw/min), vf (mm/min), Q (cm³/min), Rmax (µm), Pc (kW)

Formules:
```
n = (1000 × vc) / (π × Dm)
vf = fn × n
Q = ap × fn × vc
Rmax = 125 × fn² / rε
Pc = (vc × ap × fn × kc) / 60000
```

**Frezen invoer:** vc (m/min), Dc (mm), fz (mm/tand), zc (tanden), ap (mm), ae (mm), kc (N/mm²)
**Frezen resultaat:** n (omw/min), vf (mm/min), Q (cm³/min), Pc (kW), Mc (Nm)

Formules:
```
n = (1000 × vc) / (π × Dc)
vf = fz × n × zc
Q = (ap × ae × vf) / 1000
Pc = (ap × ae × vf × kc) / 60000000
Mc = (Pc × 30000) / (π × n)
```

**Materiaal snelstart:** klikbare lijst met ISO-materiaalgroepen die kc invult:
| ISO | Naam | vc carbide (m/min) | kc (N/mm²) |
|-----|------|--------------------|------------|
| P | Staal | 200–400 | 1500–2500 |
| M | Roestvast | 120–250 | 2000–3000 |
| K | Gietijzer | 150–400 | 800–1600 |
| N | Non-ferro | 300–3000 | 400–900 |
| S | HRSA/Ti | 20–60 | 2500–4000 |
| H | Gehard | 80–200 | 3000–6000 |

### 3. DIAGNOSE
Twee modi: draaien en frezen. Klik op symptoom → klapt open met oorzaken, acties en verificatie.

**Draaien diagnose:**

| Symptoom | Urgentie | Oorzaken | Acties | Verificatie |
|----------|----------|----------|--------|-------------|
| Lange spanen wikkelen | 2 | fn te laag, ap < ⅔ rε, verkeerde chipbreaker | fn +0.05 stapsgewijs, ap verhogen, andere insert | Meet spaanvorm na wijziging |
| Ruw/harig oppervlak | 2 | rε te klein, kerfslijtage, BUE, wrijven | Loep/microscoop check, grotere rε/wiper, ap/fn herzien | Meet Ra/Rz |
| Trillingen/chatter | 3 | Uitsteek te lang, baar te klein, radiale kracht te hoog, opspanning | EERST kort/dik, kr→90°, kleinere rε, DAN data | Luister + meet maat |
| BUE | 2 | vc te laag, verkeerde coating, ISO M/N | vc omhoog, scherper, andere grade | Inspecteer snijkant |
| Maatafwijking/taper | 3 | Doorbuiging (L³), thermisch, slijtage, opspanning | Meet na 1e snede, controleer uitsteek + klauwplaat | Meet 2 posities met micrometer |

**Frezen diagnose:**

| Symptoom | Urgentie | Oorzaken | Acties | Verificatie |
|----------|----------|----------|--------|-------------|
| Chatter vlakfrezen | 3 | Tool te lang, te grote frees, slappe houder | 1.Korter 2.Stijvere houder 3.Kleinere frees 4.Wijdvertand 5.Minder ae 6.Demping >4×D | Luister + voel tafel |
| Slechte finish | 2 | fn>80% bs, axiale uitloop, trillingen | fn omlaag, uitloop meten (<0.01mm), wiper check | Meet Ra + visueel |
| Werkstuk tilt | 3 | Tegenlopend, opspankracht laag, te zware snede | Meelopend, opspanning check, ap/ae omlaag | Check opspanning voor+na |
| Hitte in sleuf | 2 | ae=volle diameter, geen koeling, vc te hoog | Trochoidaal, hellend insteken, koeling | Spaankleur: blauw=te heet |

Urgentie 3 = rode "STOP & FIX" badge, urgentie 2 = accent "AANDACHT" badge.

### 4. NASLAGWERK
5 sub-tabs: Formules | Parameters | Materialen | Toleranties | G-code

**Formules:** Draaien, frezen, boren formules in monospace + vuistregels:
- ap ≥ ⅔ × rε
- fn < 80% van wiper-breedte bs
- Doorbuiging ~ L³
- Stijfheid ~ D⁴
- Demping bij >4×D
- Meetonzekerheid < ¼ × tolerantie

**Parameters:** 10 parameters (vc, n, fn, fz, ap, ae, rε, kr, l/D, kc) elk met:
- symbool, naam, eenheid
- ↑ effect (groen)
- ↓ effect (rood)

**Materialen:** 6 ISO-groepen (P/M/K/N/S/H) met kleurcodering, vc-range, kc-range, kenmerken, risico.

**Toleranties:**
- IT-klassen tabel (IT6 t/m IT11 bij Ø25)
- Passingen tabel (H7/g6, H7/h6, H7/k6, H7/p6, H7/s6 met type en toepassing)
- Oppervlakteruwheid (Ra, Rz, Rt uitleg + waarschuwing dat dezelfde Ra anders kan functioneren)

**G-code:** Fanuc draaien (15 codes), Fanuc frezen (13 codes), M-codes (9 codes), Mazatrol tips, veiligheidsregels.

Fanuc draaien codes:
G00 Snelgang, G01 Lineair, G02/G03 Cirkelboog, G28 Referentiepunt, G40 TNRC uit, G41/G42 TNRC links/rechts, G50 Max rpm, G70 Finish-cyclus, G71 Ruw-cyclus, G76 Draadcyclus, G96 CSS, G97 Vast toerental, G98 Voeding/min, G99 Voeding/omw

Fanuc frezen codes:
G00, G01, G02/G03, G17 XY-vlak, G40 Compensatie uit, G41/G42 Freescomp, G43 Lengtecomp, G54-G59 Nulpunten, G73 Snelle peck, G81 Standaard boor, G83 Diepgat boor, G94 Voeding/min, G95 Voeding/omw

M-codes:
M00 Stop, M01 Optioneel stop, M03/M04 Spil CW/CCW, M05 Spil stop, M06 Toolwissel, M08/M09 Koeling aan/uit, M30 Einde

Veiligheidsregels:
- Dry-run / single block bij nieuw programma
- G50 Smax bij CSS
- Offsets controleren voor eerste snede
- Nooit G00 naar werkstuk zonder positiezekerheid
- M00 bij twijfel
- Deuren dicht, bril op

### 5. LOGBOEK
Overzicht van alle opdrachten waar notities in staan. Gesorteerd op laatst bewerkt. Klik om naar de opdracht te gaan. Toont voortgangsbalk (filled/total stappen) en datum.

## Deploy instructies voor Claude Code

```bash
# 1. Maak project directory
mkdir verspaanmeester && cd verspaanmeester

# 2. Initialiseer git
git init

# 3. Maak index.html (de hele app)
# [genereer het bestand op basis van bovenstaande specificatie]

# 4. Maak README.md
# [genereer op basis van bovenstaande]

# 5. Git commit
git add .
git commit -m "VerspaanMeester v3 — werkplaats-companion voor CNC verspanen"

# 6. Maak GitHub repository en push
gh repo create verspaanmeester --public --source=. --push

# 7. Enable GitHub Pages
gh api repos/{owner}/verspaanmeester/pages -X POST -f source.branch=main -f source.path=/

# Of handmatig: Settings → Pages → Deploy from branch → main → / (root)
```

## Toekomstige uitbreidingen (nice to have)
- PWA manifest voor offline gebruik + installeren als app
- Export logboek als PDF/markdown
- Meer opdrachten (boren, kotteren, draadsnijden)
- Spaanvorm-fotogalerij ter referentie
- Interactieve diagrammen (snijkrachtenrichting, spaandikteprofielen)
- Donker/licht thema toggle
- Taal toggle NL/EN

## Belangrijk
- GEEN externe API calls
- GEEN build tooling (geen npm, geen webpack, geen vite)
- ALLES in één index.html
- React + Babel via CDN
- localStorage voor data
- Mobile-first design
- Werkt offline na eerste load
