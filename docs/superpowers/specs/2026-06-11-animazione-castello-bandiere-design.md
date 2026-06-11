# Design: Poster Disney Interattivo — Animazione Castello e Bandiere

**Data:** 2026-06-11  
**Progetto:** poster disney / `index 2.html`  
**Argomento:** Aggiunta logica JavaScript di animazione (pulsazione castello + sventolio bandiere) con pannello slider

---

## 1. Panoramica

Il poster digitale basato sull'artwork vettoriale Disney deve diventare interattivo: il castello "respira" con una pulsazione sinusoidale e le tre bandiere sventolano con curve di Bezier animate. L'utente controlla intensità e velocità di entrambe le animazioni tramite slider minimali a destra del poster.

---

## 2. Stack & Vincoli

| Livello | Tecnologia | Vincoli |
|---------|-----------|---------|
| Markup | HTML5 inline | Nessun framework |
| Stili | CSS3 inline in `<head>` | Nessun preprocessor |
| Animazione | JavaScript nativo | `requestAnimationFrame`, nessuna libreria esterna |
| Grafica | SVG inline nel DOM | Nessuna CDN, elementi manipolati via JS |
| Ambiente | Browser moderno | Supporto SVG + rAF, funziona aperto come file `file://` |

---

## 3. Architettura

### 3.1 Struttura del loop

Un **singolo** `requestAnimationFrame` aggiorna tutte le animazioni ogni frame (~60 Hz). Lo stato è mantenuto in 6 variabili globali `let`:

- `tPulse` — fase temporale per il castello
- `tFlags` — fase temporale per le bandiere
- `intensitaCastello` — scala massima della pulsazione
- `velocitaCastello` — velocità angolare della sinusoide castello
- `intensitaBandiere` — ampiezza massima del sventolio
- `velocitaBandiere` — frequenza del sventolio

Il DOM viene interrogato **una sola volta** all'avvio (`DOMContentLoaded`):
- `castlePulse` → elemento `<g id="castle-pulse">`
- `bandiera_1`, `bandiera_2`, `bandiera_3` → elementi `<path>` (già convertiti da `<polygon>`)

### 3.2 Inizializzazione bandiere

Al caricamento, per ogni bandiera:
1. Leggere i 4 punti del `<polygon>` originale (top-left, top-right, bottom-right, bottom-left)
2. Generare un `<path>` con una curva quadratica di Bezier per la punta sinistra mobile
3. Sostituire l'elemento `<polygon>` con il nuovo `<path>` o aggiornare il `d` di quello esistente
4. Salvare i punti originali come riferimento per l'animazione

### 3.3 Aggiornamento frame

```
requestAnimationFrame(loop)
  ├── tPulse += velocitaCastello
  ├── tFlags += velocitaBandiere
  ├── scale = 1 + sin(tPulse) * (intensitaCastello * 0.002)
  │   └── castlePulse.style.transform = `scale(${scale})`
  └── per ogni bandiera:
        ├── calcola offset punta = sin(tFlags + offsetFase) * intensitaBandiere
        ├── calcola punti di controllo in controfase
        └── aggiorna attributo 'd' del <path>
```

---

## 4. Componenti

### 4.1 Pannello Slider

Aggiunti 4 `<input type="range">` dentro `.control-panel`:

| # | Label | Default | Range | Mappato su |
|---|-------|---------|-------|------------|
| 1 | Intensità pulsazione | 40 | 0 – 100 | `intensitaCastello` |
| 2 | Velocità pulsazione | 30 | 10 – 100 | `velocitaCastello` (0.01 – 0.10) |
| 3 | Intensità sventolio | 60 | 0 – 100 | `intensitaBandiere` |
| 4 | Velocità sventolio | 50 | 10 – 100 | `velocitaBandiere` (0.02 – 0.12) |

**Stile** ereditato da `.control-panel`, `.slider-label`, `.modern-slider` già presenti nel file.

**Comportamento**: `input` event aggiorna immediatamente la variabile globale; nessun debounce necessario con rAF.

### 4.2 Pulsazione castello

- Formula: `scale = 1 + Math.sin(tPulse) * (intensitaCastello * 0.002)`
- `transform-origin: 540px 960px` (centro del viewBox 1080×1920)
- Applicazione: `castlePulse.style.transform = \`scale(${scale})\``
- Quando `intensitaCastello = 0` → scale fisso a 1.0, animazione statica

### 4.3 Sventolio bandiere

Ogni bandiera è un quadrilatero con:
- **Base destra fissa** (punti b e c)
- **Punta sinistra mobile** (punti a1 e a2) — oscillano verticalmente
- **Curve quadratiche** — i punti di controllo oscillano in controfase rispetto alla punta per creare l'effetto "pancia d'onda"

**Path generato**:
```
M ax,ay
Q cx1,cy1 bx,by
L cx,cy
Q cx2,cy2 ax2,ay2
Z
```

Dove `cx1,cy1` e `cx2,cy2` sono i punti di controllo calcolati in controfase.

---

## 5. Data Flow

```
┌─────────────┐     oninput      ┌─────────────────┐
│   Slider    │ ───────────────> │ Variabili state │
│  (4 input)  │                  │  (6 let global) │
└─────────────┘                  └────────┬────────┘
                                          │
                                          │ legge
                                          ▼
                                    ┌─────────────┐
                                    │  rAF loop   │
                                    │  (1x/frame) │
                                    └──────┬──────┘
                                           │ scrive
                              ┌────────────┼────────────┐
                              ▼            ▼            ▼
                         ┌────────┐   ┌─────────┐  ┌─────────┐
                         │castle  │   │bandiera │  │bandiera │
                         │pulse   │   │   1     │  │   2     │ ...
                         └────────┘   └─────────┘  └─────────┘
```

---

## 6. Error Handling & Resilienza

- **Elemento mancante**: se `castlePulse` o una bandiera non esistono, l'animazione per quell'elemento viene saltata silenziosamente. Nessun crash, nessun alert.
- **Browser legacy**: se `requestAnimationFrame` non è disponibile, il poster rimane completamente statico. Il file è comunque visibile.
- **Stili CSS**: se `.control-panel` non esiste, gli slider vengono comunque aggiunti al body.

---

## 7. Performance

- **Un solo rAF**: evitare più loop indipendenti
- **Nessun reflow forzato**: si aggiornano solo `transform` e attributo `d`
- **Query DOM una sola volta**: tutti gli elementi salvati in variabili all'avvio
- **Nessun garbage**: riutilizzo delle stesse stringhe per `d` dove possibile

---

## 8. Verifica Visiva

| Controllo | Criterio |
|-----------|----------|
| Fluidità | 60 fps senza jitter |
| Slider a 0 | Animazione completamente ferma |
| Slider al massimo | Movimento evidente ma non caotico |
| Fisica | Sventolio naturale, nessun "smerdamento" |
| Apertura file | Funziona via `file://` senza server |

---

## 9. Note di Implementazione

- **Conversione polygon → path**: le bandiere originali nel file SVG sono `<polygon>`. Il codice di inizializzazione deve convertirle in `<path>` con attributo `d` vuoto all'avvio, poi popolarlo ogni frame.
- **Ordine z**: `castle-pulse` deve restare **primo** nel SVG (dietro), `bg-static` dopo (davanti). Non modificarlo.
- **Transform-origin**: già impostato nel CSS a `540px 960px`. Non toccare.
- **Colori**: castello e bandiere nere (`#010101`); sfondo bianco. Non toccare.

---

## 10. Approccio Scelto

**Approccio A — Singolo loop rAF** (raccomandato e approvato).  
Codice compatto, performance ottimale, adatto a un single-file HTML autonomo.

---

## 11. Riferimenti

- AGENTS.md — convenzioni progetto, colori, stack
- `index 2.html` — file principale da modificare
- `Artwork Disney Tavola Disegno 1.svg` — sorgente SVG di riferimento
