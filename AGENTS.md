# AGENTS.md — Poster Disney Interattivo

## Panoramica

Poster digitale animato basato sull'artwork vettoriale Disney.

- **Castello** — "respira" con pulsazione sinusoidale
- **Bandiere** — sventolano con curve di Bezier animate
- **Tubo** — si gonfia e sgonfia ciclicamente per 5 secondi massimo, sputando pezzi vettoriali di Topolino
- **Pezzi** — volano con fisica semplice e si accumulano dal basso verso l'alto fino a coprire l'intero poster
- **Controllo** — 6 slider minimali a destra del poster regolano tutte le animazioni

---

## Stack

| Livello | Tecnologia | Vincoli |
|---------|-----------|---------|
| Markup | HTML5 inline | Nessun framework |
| Stili | CSS3 inline nel `<head>` | Nessun preprocessor |
| Animazione | JavaScript nativo | `requestAnimationFrame`, nessuna libreria esterna |
| Grafica | SVG inline nel DOM | Nessuna CDN, elementi manipolati via JS |
| Dati pezzi | SVG path embeddati come stringhe JS | Funziona su `file://` senza fetch |
| Ambiente | Browser moderno | Supporto SVG + rAF |

---

## Struttura file

```
poster disney/
├── index 2.html                # Poster Artwork completo (file principale)
├── Artwork Disney Tavola Disegno 1.svg   # Sorgente vettoriale completo (castello disney)
├── castello disney.svg         # Asset aggiornato con gruppi rinominati
├── AGENTS.md                   # Questo file
├── pezzi topolino/             # Asset SVG dei pezzi sparati dal tubo
│   ├── faccia 1.svg
│   ├── faccia topolino.svg
│   ├── mano topolino.svg
│   └── scarpa topolino.svg
└── docs/
    └── superpowers/
        ├── specs/
        │   ├── 2026-06-11-animazione-castello-bandiere-design.md
        │   └── 2026-06-11-tubo-espulsione-pezzi-design.md
        └── plans/
            ├── 2026-06-11-animazione-castello-bandiere.md
            └── 2026-06-11-tubo-espulsione-pezzi.md
```

---

## Convenzioni

### SVG
- Gli elementi animati devono avere un `id` univoco: `castello`, `bandiera_1`, `bandiera_2`, `bandiera_3`, `Tubo`
- Le bandiere sono `<path>` vuoti all'avvio; il JS popola l'attributo `d` ogni frame
- Il castello è un singolo `<path>` monolitico
- Gruppo animato castello: `<g id="castle-pulse">` contiene castello + 3 bandiere
- Gruppo statico: `<g id="bg-static">` contiene tutti gli altri elementi (incluso `Tubo`)
- Contenitore pezzi: `<g id="pezzi-container"></g>` posizionato **dopo** `bg-static` (davanti a tutto)
- Ordine z: `castle-pulse` → `bg-static` → `pezzi-container`

### CSS pezzi embeddati
I pezzi SVG usano classi `.st0` e `.st1` definite nell'HTML:
```css
.st0 { fill: #fff; }
.st1 { fill: none; stroke: #fcfcfc; stroke-miterlimit: 10; stroke-width: 14px; }
```

### Animazioni
- **Singolo loop `requestAnimationFrame`** per tutte le animazioni
- Nessuna libreria esterna (no Anime.js, no GSAP)
- Stato in variabili globali (vedi sotto)
- DOM query una sola volta all'avvio

### Slider
- Pannello fluttuante a **destra del poster** (`right: 40px`, centrato verticalmente)
- 6 slider:
  1. **Intensità pulsazione** — default `40`, range `0–100`
  2. **Velocità pulsazione** — default `30`, range `10–100`
  3. **Intensità sventolio** — default `60`, range `0–100`
  4. **Velocità sventolio** — default `50`, range `10–100`
  5. **Intensità espulsione** — default `50`, range `20–100`
  6. **Velocità ciclo tubo** — default `40`, range `10–100`
- Stile minimal: sfondo bianco semi-trasparente con `backdrop-filter: blur(12px)`, bordo sottile, border-radius 40px
- Label in maiuscolo, letter-spacing 1.5px, font-family di sistema
- Event `input` aggiorna immediatamente le variabili globali

### Colori
- Castello e bandiere: **nero** (`#010101` o `#000`)
- Pezzi Topolino: **bianchi** (`#fff`) con stroke nero/grigio
- Sfondo pagina: **bianco** (`#fff`)
- UI: nero su bianco trasparente

---

## Logica animazione

### Pulsazione castello
```javascript
tPulse += velocitaCastello;
let scale = 1 + Math.sin(tPulse) * (intensitaCastello * 0.002);
castlePulse.style.transform = `scale(${scale})`;
// transform-origin: centro del viewBox (540px 960px per artwork 1080x1920)
```

### Sventolio bandiere
```javascript
tFlags += velocitaBandiere;
let wave = Math.sin(tFlags + phase); // phase: 0, 0.9, 1.8 per le 3 bandiere
let tipOffset = wave * (intensitaBandiere * 0.18);
let belly = Math.cos(tFlags + phase) * (intensitaBandiere * 0.10);
// Path: M ax,ay Q c1x,c1y bx,by L cx,cy Q c2x,c2y ax,ay Z
```

### Tubo espulsione pezzi

**Stati del tubo:**
- `0 = IDLE` → avvio automatico
- `1 = CHARGING` (40% del ciclo) — gonfiamento verso l'alto `scaleY(1.15)`
- `2 = FIRING` (10% del ciclo) — snap di espulsione + spawn pezzo
- `3 = COOLDOWN` (50% del ciclo) — pausa prima del prossimo pezzo

**Ciclo di vita:**
- Il tubo parte automaticamente all'avvio (`tuboRunning = true`, `tuboStartTime = Date.now()`)
- Dopo `TUBO_MAX_DURATION = 5000` ms (5 secondi), il tubo si ferma (`tuboRunning = false`)
- I pezzi continuano a esistere e restano visibili

**Spawn pezzi:**
- Sequenza ciclica: `faccia 1` → `mano` → `scarpa` → `faccia topolino`
- Posizione di spawn: centro bocca del tubo (`TUBO_SPAWN: 539, 1452`)
- Velocità iniziale: `vx` casuale ±3, `vy` verso l'alto proporzionale a `intensitaTubo`

**Fisica pezzi:**
```javascript
p.vy += 0.4;        // gravità
p.x += p.vx;
p.y += p.vy;
p.rotation += p.vRot;
p.vx *= 0.995;      // attrito aria
```

**Accumulo dal basso verso l'alto:**
- `accumuloY` inizia a 1920 (fondo del poster)
- Ogni pezzo atterra su `accumuloY`, poi `accumuloY` scende dell'altezza del pezzo × 0.6
- L'accumulo continua fino a `accumuloY < -200`, coprendo tutta la tavola
- I pezzi atterrati restano fermi (`landed = true`)

---

## Variabili globali

| Variabile | Ruolo |
|-----------|-------|
| `tPulse`, `tFlags` | Timestamp animazioni castello e bandiere |
| `intensitaCastello`, `velocitaCastello` | Controllo pulsazione |
| `intensitaBandiere`, `velocitaBandiere` | Controllo sventolio |
| `intensitaTubo`, `velocitaTubo` | Controllo espulsione |
| `tTubo` | Timer interno ciclo tubo |
| `tuboState` | Stato macchina a stati (0–3) |
| `tuboRunning` | Booleano: il tubo sta ancora sparando? |
| `tuboStartTime` | `Date.now()` all'avvio del ciclo |
| `TUBO_MAX_DURATION` | 5000 ms — durata massima ciclo |
| `pezzoIndex` | Indice pezzo corrente (0–3, ciclico) |
| `pezzi` | Array di oggetti pezzo in volo |
| `accumuloY` | Coordinata Y corrente di accumulo (scende dal basso verso l'alto) |
| `PEZZI_SVG` | Array embeddato con i path SVG dei 4 pezzi |

---

## Mappatura slider → variabili

| Slider | ID | Default | Range | Mappato su |
|--------|-----|---------|-------|------------|
| Intensità pulsazione | `intensita-castello` | 40 | 0 – 100 | `intensitaCastello` |
| Velocità pulsazione | `velocita-castello` | 30 | 10 – 100 | `velocitaCastello` (×0.001) |
| Intensità sventolio | `intensita-bandiere` | 60 | 0 – 100 | `intensitaBandiere` |
| Velocità sventolio | `velocita-bandiere` | 50 | 10 – 100 | `velocitaBandiere` (0.02–0.12) |
| Intensità espulsione | `intensita-tubo` | 50 | 20 – 100 | `intensitaTubo` (spinta verticale) |
| Velocità ciclo tubo | `velocita-tubo` | 40 | 10 – 100 | `velocitaTubo` (durata ciclo) |

---

## Requisiti

- Deve funzionare **senza server** (aprire direttamente il file `.html` nel browser)
- Nessuna dipendenza esterna da CDN o npm
- Compatibilità: browser moderni con supporto SVG e `requestAnimationFrame`
- Se un elemento animato non esiste, l'animazione viene saltata silenziosamente (no crash)
- Se `requestAnimationFrame` non è disponibile, il poster resta statico
- Il ciclo di espulsione del tubo dura massimo **5 secondi**
- I pezzi si accumulano dal basso verso l'alto fino a coprire tutta la tavola

---

## Preferenze utente

- L'utente è un **designer**, non fare domande tecniche
- Preferisce soluzioni pulite, minimaliste, con attenzione al dettaglio visivo
- Vuole controllo diretto tramite slider intuitivi
- Le animazioni devono essere fluide e fisicamente credibili (no effetti "smerdati")

---

## Modifiche comuni

- **Cambiare intensità default**: modificare `value` negli `<input>`
- **Cambiare velocità default**: modificare `value` negli `<input>`
- **Cambiare range velocità**: modificare `min`/`max` e il fattore di mappatura nel JS
- **Cambiare durata ciclo tubo**: modificare `TUBO_MAX_DURATION` nel JS (default 5000 ms)
- **Cambiare colori**: agire sui `fill` nel CSS e sugli attributi SVG originali
- **Aggiungere elementi all'artwork**: modificarli nel file `.svg` sorgente, poi rigenerare l'HTML mantenendo `castle-pulse`, `bg-static` e `pezzi-container`
- **Aggiungere nuovi slider**: seguire il pattern esistente (`.slider-group`, `.slider-label`, `.modern-slider`) e collegare `addEventListener('input', ...)`

---

## Riferimenti

- `docs/superpowers/specs/2026-06-11-animazione-castello-bandiere-design.md` — Design spec castello + bandiere
- `docs/superpowers/specs/2026-06-11-tubo-espulsione-pezzi-design.md` — Design spec tubo espulsione
- `docs/superpowers/plans/2026-06-11-animazione-castello-bandiere.md` — Implementation plan castello + bandiere
- `docs/superpowers/plans/2026-06-11-tubo-espulsione-pezzi.md` — Implementation plan tubo espulsione
