# Animazione Castello e Bandiere — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) o superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Aggiungere al file `index 2.html` la logica JavaScript nativa (requestAnimationFrame) che anima il castello con pulsazione sinusoidale e le tre bandiere con sventolio a curve di Bezier, controllabili tramite 4 slider già stilizzati nel CSS.

**Architecture:** Singolo loop rAF condiviso; stato in 6 variabili globali let; DOM query una sola volta all'avvio. Le bandiere vengono inizializzate da coordinate hardcoded (il file HTML contiene già `<path>` vuoti). Nessuna libreria esterna.

**Tech Stack:** HTML5, CSS3 inline, JavaScript nativo (ES5-compatibile), SVG inline.

---

## File Structure

| File | Azione | Responsabilità |
|------|--------|----------------|
| `index 2.html` | **Modifica** | Aggiungere 4 slider nel pannello, regola CSS `.slider-group`, blocco `<script>` con logica di animazione completa |

---

## Task 1: Aggiungere 4 slider nel pannello di controllo

**Files:**
- Modifica: `index 2.html` — dentro `.control-panel` (linea ~50) e nel blocco `<style>`

- [ ] **Step 1: Aggiungere CSS `.slider-group`**

Inserire all'interno del blocco `<style>` (dopo la regola `.modern-slider::-webkit-slider-thumb:hover`):

```css
  .slider-group {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
    gap: 6px;
  }
```

- [ ] **Step 2: Inserire gli slider nel pannello**

All'interno del `<div class="control-panel">` (prima del `</div>` di chiusura), aggiungere:

```html
  <div class="slider-group">
    <span class="slider-label">Intensità pulsazione</span>
    <input type="range" class="modern-slider" id="intensita-castello" min="0" max="100" value="40">
  </div>
  <div class="slider-group">
    <span class="slider-label">Velocità pulsazione</span>
    <input type="range" class="modern-slider" id="velocita-castello" min="10" max="100" value="30">
  </div>
  <div class="slider-group">
    <span class="slider-label">Intensità sventolio</span>
    <input type="range" class="modern-slider" id="intensita-bandiere" min="0" max="100" value="60">
  </div>
  <div class="slider-group">
    <span class="slider-label">Velocità sventolio</span>
    <input type="range" class="modern-slider" id="velocita-bandiere" min="10" max="100" value="50">
  </div>
```

- [ ] **Step 3: Commit parziale**

```bash
git add "index 2.html"
git commit -m "feat: aggiunti 4 slider intensità/velocità castello e bandiere"
```

---

## Task 2: Implementare il motore di animazione JavaScript

**Files:**
- Modifica: `index 2.html` — aggiungere blocco `<script>` alla fine del `<body>`, prima di `</body>`

- [ ] **Step 1: Inizializzare stato e riferimenti DOM**

Aggiungere in fondo al `<body>`:

```html
<script>
(function() {
  'use strict';

  // Stato globale
  var tPulse = 0;
  var tFlags = 0;
  var intensitaCastello = 40;
  var velocitaCastello = 0.03;
  var intensitaBandiere = 60;
  var velocitaBandiere = 0.06;

  // Riferimenti DOM
  var castlePulse = document.getElementById('castle-pulse');
  var sliderIntCast = document.getElementById('intensita-castello');
  var sliderVelCast = document.getElementById('velocita-castello');
  var sliderIntFlag = document.getElementById('intensita-bandiere');
  var sliderVelFlag = document.getElementById('velocita-bandiere');

  // Coordinate originali delle bandiere (dal sorgente SVG)
  // bandiera_1, bandiera_2, bandiera_3 — ordine: tip-left, base-top-right, base-bottom-right, tip-left (chiusura)
  var flagOrigins = {
    bandiera_1: { ax: 274.51, ay: 291.52, bx: 343.90, by: 313.72, cx: 343.90, cy: 269.25 },
    bandiera_2: { ax: 567.61, ay: 81.07,  bx: 637.01, by: 103.27, cx: 637.01, cy: 58.87 },
    bandiera_3: { ax: 404.66, ay: 151.15, bx: 474.05, by: 173.34, cx: 474.05, cy: 128.87 }
  };

  // Struttura dati runtime per ogni bandiera
  var flags = [];
```

- [ ] **Step 2: Inizializzare le bandiere (popolare `<path>` vuoti)**

Continuare nello stesso blocco `<script>`, subito dopo la dichiarazione di `flags`:

```javascript
  function initFlags() {
    var ids = ['bandiera_1', 'bandiera_2', 'bandiera_3'];
    for (var i = 0; i < ids.length; i++) {
      var id = ids[i];
      var el = document.getElementById(id);
      if (!el) continue;

      var o = flagOrigins[id];
      if (!o) continue;

      // Se è ancora un polygon, convertilo in path (defensive)
      if (el.tagName.toLowerCase() === 'polygon') {
        var path = document.createElementNS('http://www.w3.org/2000/svg', 'path');
        path.setAttribute('id', id);
        path.setAttribute('fill', el.getAttribute('fill') || '#010101');
        el.parentNode.replaceChild(path, el);
        el = path;
      }

      flags.push({
        el: el,
        ax: o.ax, ay: o.ay,
        bx: o.bx, by: o.by,
        cx: o.cx, cy: o.cy,
        phase: i * 0.9 // sfasamento per bandiera
      });
    }
  }

  initFlags();
```

- [ ] **Step 3: Collegare slider allo stato**

Continuare nello stesso blocco `<script>`:

```javascript
  function updateState() {
    intensitaCastello = parseInt(sliderIntCast.value, 10);
    // Mappa 10→0.01, 100→0.10
    velocitaCastello = parseInt(sliderVelCast.value, 10) * 0.001;

    intensitaBandiere = parseInt(sliderIntFlag.value, 10);
    // Mappa 10→0.02, 100→0.12
    var vFlag = parseInt(sliderVelFlag.value, 10);
    velocitaBandiere = 0.02 + (vFlag - 10) * ((0.12 - 0.02) / 90);
  }

  // Event listeners
  if (sliderIntCast) sliderIntCast.addEventListener('input', updateState);
  if (sliderVelCast) sliderVelCast.addEventListener('input', updateState);
  if (sliderIntFlag) sliderIntFlag.addEventListener('input', updateState);
  if (sliderVelFlag) sliderVelFlag.addEventListener('input', updateState);

  updateState(); // sincronizza allo stato iniziale
```

- [ ] **Step 4: Scrivere il loop requestAnimationFrame**

Continuare nello stesso blocco `<script>`:

```javascript
  function animate() {
    tPulse += velocitaCastello;
    tFlags += velocitaBandiere;

    // Pulsazione castello
    if (castlePulse) {
      var scale = 1 + Math.sin(tPulse) * (intensitaCastello * 0.002);
      castlePulse.style.transform = 'scale(' + scale.toFixed(4) + ')';
    }

    // Sventolio bandiere
    for (var i = 0; i < flags.length; i++) {
      var f = flags[i];
      var wave = Math.sin(tFlags + f.phase);
      var tipOffset = wave * (intensitaBandiere * 0.18);
      var belly = Math.cos(tFlags + f.phase) * (intensitaBandiere * 0.10);

      var ax1 = f.ax;
      var ay1 = f.ay + tipOffset;
      var ax2 = f.ax;
      var ay2 = f.ay + tipOffset;

      var c1x = (f.ax + f.bx) / 2 + belly;
      var c1y = (f.ay + f.by) / 2 + tipOffset * 0.5;
      var c2x = (f.ax + f.cx) / 2 + belly;
      var c2y = (f.ay + f.cy) / 2 + tipOffset * 0.5;

      var d = 'M' + ax1.toFixed(2) + ' ' + ay1.toFixed(2) +
              ' Q' + c1x.toFixed(2) + ' ' + c1y.toFixed(2) +
              ' ' + f.bx.toFixed(2) + ' ' + f.by.toFixed(2) +
              ' L' + f.cx.toFixed(2) + ' ' + f.cy.toFixed(2) +
              ' Q' + c2x.toFixed(2) + ' ' + c2y.toFixed(2) +
              ' ' + ax2.toFixed(2) + ' ' + ay2.toFixed(2) + ' Z';

      f.el.setAttribute('d', d);
    }

    requestAnimationFrame(animate);
  }

  requestAnimationFrame(animate);
})();
</script>
```

- [ ] **Step 5: Commit parziale**

```bash
git add "index 2.html"
git commit -m "feat: implementato motore rAF per pulsazione castello e sventolio bandiere"
```

---

## Task 3: Verifica visiva e collaudo manuale

**Files:**
- Nessuna modifica file; verifica manuale del file `index 2.html`

- [ ] **Step 1: Aprire il file nel browser**

Comando (Windows):
```powershell
Start-Process "index 2.html"
```

Oppure aprire manualmente il file `C:\Users\25jen\Documents\GitHub\disney_artwork\index 2.html` nel browser.

- [ ] **Step 2: Controlli funzionali**

| Controllo | Azione | Risultato atteso |
|-----------|--------|------------------|
| Castello pulsa | Osservare il castello | Leggera espansione/contrazione ritmica |
| Bandiere sventolano | Osservare le 3 bandiere in alto | Ondulazione fluida con effetto pancia |
| Slider intensità castello | Portare a 0 | Pulsazione si ferma (scale fisso a 1) |
| Slider velocità castello | Portare a 10 poi 100 | Lenta vs rapida pulsazione |
| Slider intensità bandiere | Portare a 0 | Sventolio si ferma (bandiere statiche) |
| Slider velocità bandiere | Portare a 10 poi 100 | Lento vs rapido sventolio |
| Ordine z | Verificare bandiere sopra sfondo | Bandiere visibili, non coperte |

- [ ] **Step 3: Commit finale**

```bash
git add "index 2.html"
git commit -m "chore: verifica visiva completata, animazioni responsive agli slider"
```

---

## Checklist di copertura dello spec

| Requisito spec | Task che lo implementa |
|----------------|------------------------|
| Singolo loop rAF | Task 2, Step 4 |
| 6 variabili state globali | Task 2, Step 1 |
| Query DOM una sola volta | Task 2, Step 1 & 2 |
| 4 slider con default corretti | Task 1, Step 2 |
| Pulsazione castello sinusoidale | Task 2, Step 4 (formula `scale = 1 + sin(...)`) |
| Sventolio bandiere con Bezier | Task 2, Step 4 (curve Q con punti di controllo) |
| Conversione polygon→path | Task 2, Step 2 (defensive + hardcoded origins) |
| Resilienza (elementi mancanti) | Task 2, Step 2 e 4 (check `if (!el)` e `if (castlePulse)`) |
| Nessuna dipendenza esterna | Intero piano (solo JS nativo) |
| Apertura via `file://` | Task 3 (verifica manuale) |

---

*Piano completato. Nessun placeholder rilevato. Tutti i riferimenti a variabili e ID sono coerenti tra i task.*
