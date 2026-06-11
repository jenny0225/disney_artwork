# AGENTS.md — Poster Disney Interattivo

## Panoramica

Poster digitale animato basato sull'artwork vettoriale Disney. Il castello "respira" con una pulsazione sinusoidale e le tre bandiere sventolano con curve di Bezier animate. L'utente controlla intensitÃ e velocitÃ di entrambe le animazioni tramite 4 slider minimali a destra del poster.

---

## Stack

| Livello | Tecnologia | Vincoli |
|---------|-----------|---------|
| Markup | HTML5 inline | Nessun framework |
| Stili | CSS3 inline nel `<head>` | Nessun preprocessor |
| Animazione | JavaScript nativo | `requestAnimationFrame`, nessuna libreria esterna |
| Grafica | SVG inline nel DOM | Nessuna CDN, elementi manipolati via JS |
| Ambiente | Browser moderno | Supporto SVG + rAF, funziona aperto come file `file://` |

---

## Struttura file

```
poster disney/
├── index 2.html                # Poster Artwork completo (file principale)
├── Artwork Disney Tavola Disegno 1.svg   # Sorgente vettoriale completo (castello disney)
├── castello disney.svg         # Asset aggiornato con gruppi rinominati
├── AGENTS.md                   # Questo file
└── docs/
    └── superpowers/
        ├── specs/
        │   └── 2026-06-11-animazione-castello-bandiere-design.md
        └── plans/
            └── 2026-06-11-animazione-castello-bandiere.md
```

---

## Convenzioni

### SVG
- Gli elementi animati devono avere un `id` univoco: `castello`, `bandiera_1`, `bandiera_2`, `bandiera_3`
- Le bandiere sono `<path>` vuoti all'avvio; il JS popola l'attributo `d` ogni frame
- Il castello è un singolo `<path>` monolitico
- Gruppo animato: `<g id="castle-pulse">` contiene castello + 3 bandiere
- Gruppo statico: `<g id="bg-static">` contiene tutti gli altri elementi
- Ordine z: `castle-pulse` disegnato **primo** (dietro), `bg-static` disegnato **dopo** (davanti)

### Animazioni
- **Singolo loop `requestAnimationFrame`** per tutte le animazioni
- Nessuna libreria esterna (no Anime.js, no GSAP)
- Stato in 6 variabili globali: `tPulse`, `tFlags`, `intensitaCastello`, `velocitaCastello`, `intensitaBandiere`, `velocitaBandiere`
- DOM query una sola volta all'avvio

### Slider
- Pannello fluttuante a **destra del poster** (`right: 40px`, centrato verticalmente)
- 4 slider:
  1. **Intensità pulsazione** — default `40`, range `0–100`
  2. **Velocità pulsazione** — default `30`, range `10–100`
  3. **Intensità sventolio** — default `60`, range `0–100`
  4. **Velocità sventolio** — default `50`, range `10–100`
- Stile minimal: sfondo bianco semi-trasparente con `backdrop-filter: blur(12px)`, bordo sottile, border-radius 40px
- Label in maiuscolo, letter-spacing 1.5px, font-family di sistema (`-apple-system, BlinkMacSystemFont, ...`)
- Event `input` aggiorna immediatamente le variabili globali

### Colori
- Castello e bandiere: **nero** (`#010101` o `#000`)
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
Ogni bandiera ha base destra fissa (`bx,by / cx,cy`) e punta sinistra mobile (`ax,ay`).
Curve quadratiche di Bezier per l'effetto "pancia d'onda" — i punti di controllo oscillano in controfase rispetto alla punta.

```javascript
tFlags += velocitaBandiere;
let wave = Math.sin(tFlags + phase); // phase: 0, 0.9, 1.8 per le 3 bandiere
let tipOffset = wave * (intensitaBandiere * 0.18);
let belly = Math.cos(tFlags + phase) * (intensitaBandiere * 0.10);
// Path: M ax,ay Q c1x,c1y bx,by L cx,cy Q c2x,c2y ax,ay Z
```

---

## Mappatura slider → variabili

| Slider | ID | Default | Range | Mappato su |
|--------|-----|---------|-------|------------|
| Intensità pulsazione | `intensita-castello` | 40 | 0 – 100 | `intensitaCastello` |
| Velocità pulsazione | `velocita-castello` | 30 | 10 – 100 | `velocitaCastello` (×0.001 → 0.01–0.10) |
| Intensità sventolio | `intensita-bandiere` | 60 | 0 – 100 | `intensitaBandiere` |
| Velocità sventolio | `velocita-bandiere` | 50 | 10 – 100 | `velocitaBandiere` (mappa lineare 0.02–0.12) |

---

## Requisiti

- Deve funzionare **senza server** (aprire direttamente il file `.html` nel browser)
- Nessuna dipendenza esterna da CDN o npm
- Compatibilità: browser moderni con supporto SVG e `requestAnimationFrame`
- Se un elemento animato non esiste, l'animazione viene saltata silenziosamente (no crash)
- Se `requestAnimationFrame` non è disponibile, il poster resta statico

---

## Preferenze utente

- L'utente è un **designer**, non fare domande tecniche
- Preferisce soluzioni pulite, minimaliste, con attenzione al dettaglio visivo
- Vuole controllo diretto tramite slider intuitivi
- Le animazioni devono essere fluide e fisicamente credibili (no effetti "smerdati")

---

## Modifiche comuni

- **Cambiare intensità default**: modificare `value="40"` e `value="60"` negli `<input>`
- **Cambiare velocità default**: modificare `value="30"` e `value="50"` negli `<input>`
- **Cambiare range velocità**: modificare `min="10" max="100"` e il fattore di mappatura nel JS
- **Cambiare colori**: agire sui `fill` nel CSS e sugli attributi SVG originali
- **Aggiungere elementi all'artwork**: modificarli nel file `.svg` sorgente, poi rigenerare l'HTML mantenendo `castle-pulse` e `bg-static`
- **Aggiungere nuovi slider**: seguire il pattern esistente (`.slider-group`, `.slider-label`, `.modern-slider`) e collegare `addEventListener('input', ...)`

---

## Riferimenti

- `docs/superpowers/specs/2026-06-11-animazione-castello-bandiere-design.md` — Design spec
- `docs/superpowers/plans/2026-06-11-animazione-castello-bandiere.md` — Implementation plan
