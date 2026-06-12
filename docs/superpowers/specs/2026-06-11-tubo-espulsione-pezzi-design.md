# Design Spec — Tubo Espulsione Pezzi Topolino

## 1. Panoramica

Aggiungere un'animazione al gruppo `Tubo` del poster Disney in modo che "sputi" (espella) ciclicamente 4 pezzi vettoriali di Topolino. I pezzi vengono lanciati in sequenza, volano con fisica semplice e si accumulano davanti al poster fino a coprirlo. L'utente controlla intensitÃ  e velocitÃ  dell'espulsione tramite 2 slider aggiuntivi nel pannello di controllo esistente.

## 2. Stack e Vincoli

| Livello | Tecnologia | Vincoli |
|---------|-----------|---------|
| Markup | HTML5 inline | Nessun framework |
| Stili | CSS3 inline nel `<head>` | Nessun preprocessor |
| Animazione | JavaScript nativo | `requestAnimationFrame`, nessuna libreria esterna |
| Grafica | SVG inline nel DOM | Nessuna CDN, elementi manipolati via JS |
| Dati pezzi | SVG path embeddati come stringhe JS | Funziona su `file://` senza fetch |
| Ambiente | Browser moderno | Supporto SVG + rAF |

## 3. Struttura SVG Modificata

Il DOM dell'SVG nel file `index 2.html` viene modificato come segue:

```xml
<svg ...>
  <defs>...</defs>

  <!-- Gruppo animato castello + bandiere -->
  <g id="castle-pulse">
    <path id="castello" ... />
    <path id="bandiera_1" d="" fill="#010101" />
    <path id="bandiera_2" d="" fill="#010101" />
    <path id="bandiera_3" d="" fill="#010101" />
  </g>

  <!-- Gruppo statico sfondo -->
  <g id="bg-static">
    <g id="Tubo">...</g>
    <!-- tutti gli altri elementi statici -->
  </g>

  <!-- NUOVO: Contenitore pezzi caduti (davanti a tutto) -->
  <g id="pezzi-container"></g>
</svg>
```

**Ordine z**: `castle-pulse` â†’ `bg-static` â†’ `pezzi-container` (primo = dietro, ultimo = davanti).

## 4. Architettura

### 4.1 Macchina a stati del tubo

```javascript
const TUBO_STATES = {
  IDLE: 0,      // riposo, scale(1)
  CHARGING: 1,  // gonfiamento pre-sparo
  FIRING: 2,    // contrazione + spawn pezzo
  COOLDOWN: 3   // attesa prima del prossimo ciclo
};
```

Stato in variabili globali:
- `tTubo` â€” timestamp accumulato per il ciclo tubo
- `tuboState` â€” stato corrente della macchina a stati
- `intensitaTubo` â€” spinta espulsione (20â€“100, default 50)
- `velocitaTubo` â€” velocitÃ  ciclo (10â€“100, default 40)

### 4.2 Ciclo del tubo

| Fase | Durata (base) | Transform | Azione |
|------|---------------|-----------|--------|
| CHARGING | 40% del ciclo | `scaleX(0.95) scaleY(1.15)` | Gonfiamento verso l'alto, "carica" |
| FIRING | 10% del ciclo | `scaleX(1.05) scaleY(0.85)` → `scale(1)` | Snap di espulsione all'**inizio** della fase; spawn pezzo istantaneo; ritorno a `scale(1)` entro fine fase |
| COOLDOWN | 50% del ciclo | `scale(1)` | Pausa prima del prossimo pezzo |

Durata totale ciclo mappata da `velocitaTubo`: 10 â†’ 2000ms, 100 â†’ 400ms.

Transform-origin del tubo: centro geometrico approssimato del gruppo `Tubo`, **540px 1485px**.

### 4.3 Sequenza pezzi

Ordine ciclico (loop infinito):
1. `faccia 1.svg`
2. `mano topolino.svg`
3. `scarpa topolino.svg`
4. `faccia topolino.svg`

Indice corrente salvato in `pezzoIndex` (0â€“3), incrementato a ogni FIRING.

### 4.4 Embed pezzi SVG

I path dei 4 file SVG vengono embeddati come stringhe JavaScript in un array `PEZZI_SVG`. Ogni elemento contiene i path estratti (solo tag `<path>`, niente `<svg>` wrapper) cosÃ¬ da poter essere inserito direttamente in un `<g>`.

Esempio struttura:
```javascript
const PEZZI_SVG = [
  { name: 'faccia1', paths: '<path d="..." fill="#fff"/>' },
  { name: 'mano', paths: '<path d="..." fill="#fff"/>' },
  { name: 'scarpa', paths: '<path d="..." fill="#fff"/>' },
  { name: 'facciaTopolino', paths: '<path d="..." fill="#fff"/>' }
];
```

I path originali dei file `.svg` vengono letti una volta e trasformati in stringhe statiche nel codice sorgente. Non c'Ã¨ dipendenza dal filesystem a runtime.

## 5. Fisica dei pezzi

### 5.1 Spawn

Quando il tubo entra in stato `FIRING`:

```javascript
let pezzo = {
  x: 539 + (Math.random() - 0.5) * 10,   // centro tubo + jitter
  y: 1452,
  vx: (Math.random() - 0.5) * 6,          // -3 a +3
  vy: -6 - (intensitaTubo * 0.14),        // -8.8 a -20 (intensitÃ  20â€“100)
  rotation: Math.random() * 30 - 15,      // -15Â° a +15Â°
  vRot: Math.random() * 10 - 5,          // -5 a +5 gradi/frame
  landed: false,
  el: document.createElementNS(...)
};
```

Il `<g>` del pezzo viene appendato a `pezzi-container`.

### 5.2 Simulazione per frame

```javascript
if (!pezzo.landed) {
  pezzo.vy += 0.4;        // gravitÃ
  pezzo.x += pezzo.vx;
  pezzo.y += pezzo.vy;
  pezzo.rotation += pezzo.vRot;

  // attrito aria leggero
  pezzo.vx *= 0.995;

  // Atterraggio
  let yTarget = 1550 + Math.random() * 370;  // 1550â€“1920
  if (pezzo.y >= yTarget) {
    pezzo.y = yTarget;
    pezzo.landed = true;
    pezzo.vx = 0;
    pezzo.vy = 0;
    pezzo.vRot = 0;
    // offset rotazione finale casuale
    pezzo.rotation += Math.random() * 30 - 15;
  }

  pezzo.el.setAttribute('transform',
    `translate(${pezzo.x},${pezzo.y}) rotate(${pezzo.rotation})`
  );
}
```

### 5.3 Scala dei pezzi

Ogni pezzo viene scalato per adattarsi al poster. Fattore di scala fisso: **0.25** (regolabile). Il transform diventa:
```javascript
`translate(${x},${y}) rotate(${rotation}) scale(0.25)`
```

Il `transform-origin` di ogni pezzo Ã¨ il proprio centro (calcolato dal viewBox originale).

### 5.4 Limitazione numero pezzi

- Massimo **200 pezzi** nel DOM.
- Quando si supera il limite, il pezzo piÃ¹ vecchio viene rimosso (`pezzi-container.removeChild(firstChild)`).
- I pezzi atterrati sono `<g>` con `pointer-events: none` per non interferire con la UI.

## 6. Slider di controllo

Aggiunta di 2 slider al pannello esistente (totale 6 slider):

| # | Label | ID | Default | Range | Mappato su |
|---|-------|-----|---------|-------|------------|
| 5 | IntensitÃ  espulsione | `intensita-tubo` | 50 | 20â€“100 | `intensitaTubo` (spinta verticale e gonfiamento max) |
| 6 | VelocitÃ  ciclo tubo | `velocita-tubo` | 40 | 10â€“100 | `velocitaTubo` (durata ciclo in ms) |

Markup pattern identico agli slider esistenti:
```html
<div class="slider-group">
  <span class="slider-label">IntensitÃ  espulsione</span>
  <input type="range" id="intensita-tubo" class="modern-slider"
         min="20" max="100" value="50">
</div>
```

Event `input` aggiorna immediatamente le variabili globali.

## 7. Colori e stili

- Pezzi: ereditano i `fill` originali dai file SVG (principalmente `#fff` con stroke `#1d1d1b`)
- UI: nero su bianco trasparente, stile coerente con gli slider esistenti
- Nessun nuovo colore aggiunto

## 8. Requisiti e robustezza

- Se il gruppo `Tubo` non esiste, l'animazione tubo viene saltata silenziosamente.
- Se `pezzi-container` non esiste, i pezzi non vengono spawnati.
- Se `requestAnimationFrame` non Ã¨ disponibile, il poster resta statico.
- I pezzi SVG embeddati come stringhe garantiscono funzionamento su `file://` senza CORS o fetch.
- Performance: loop fisico O(n) con n â‰¤ 200. nessun calcolo pesante per frame.

## 9. Variabili globali aggiuntive

```javascript
let tTubo = 0;
let tuboState = TUBO_STATES.IDLE;
let intensitaTubo = 50;
let velocitaTubo = 40;
let pezzoIndex = 0;
let pezzi = [];        // array di oggetti stato pezzo
const PEZZI_SVG = [...];
const MAX_PEZZI = 200;
const TUBO_ORIGIN = { x: 540, y: 1485 };
const TUBO_SPAWN = { x: 539, y: 1452 };
```

## 10. Riferimenti

- File principale: `index 2.html`
- Asset pezzi: `pezzi topolino/faccia 1.svg`, `pezzi topolino/mano topolino.svg`, `pezzi topolino/scarpa topolino.svg`, `pezzi topolino/faccia topolino.svg`
- Spec precedente: `docs/superpowers/specs/2026-06-11-animazione-castello-bandiere-design.md`
