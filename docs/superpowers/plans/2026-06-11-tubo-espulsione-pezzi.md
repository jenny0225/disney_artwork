# Tubo Espulsione Pezzi Topolino — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Aggiungere un'animazione al gruppo `Tubo` del poster Disney in modo che "sputi" ciclicamente 4 pezzi vettoriali di Topolino, controllabile tramite 2 slider.

**Architecture:** Macchina a stati per il ciclo del tubo (carica → sparo → cooldown) + fisica semplice per i pezzi lanciati, tutto dentro il loop `requestAnimationFrame` esistente. Pezzi SVG embeddati come stringhe JS per funzionare offline su `file://`.

**Tech Stack:** HTML5 inline, CSS3 inline, JavaScript nativo, SVG inline nel DOM.

---

## File Structure

| File | Azione | Responsabilità |
|------|--------|--------------|
| `index 2.html` | **Modifica** | File principale: markup SVG, CSS slider, JavaScript animazione |

---

## Task 1: Aggiungere il contenitore pezzi nell'SVG

**Files:**
- Modify: `index 2.html` (nella sezione SVG)

- [ ] **Step 1: Inserire `<g id="pezzi-container">` dopo `bg-static`**

Dopo la chiusura di `</g>` del gruppo `bg-static` (riga 227), aggiungere:

```xml
  <!-- Contenitore pezzi caduti (davanti a tutto) -->
  <g id="pezzi-container"></g>
```

Verifica: Aprire il file in un editor e controllare che `pezzi-container` sia l'ultimo figlio dell'SVG, dopo `bg-static`.

---

## Task 2: Aggiungere gli slider del tubo al pannello di controllo

**Files:**
- Modify: `index 2.html` (nella sezione `.control-panel`, dopo il 4° slider)

- [ ] **Step 2: Aggiungere markup degli slider**

Dopo il 4° slider (dopo `</div>` del gruppo `velocita-bandiere`), aggiungere:

```html
  <div class="slider-group">
    <label for="intensita-tubo" class="slider-label">Intensita espulsione</label>
    <input type="range" class="modern-slider" id="intensita-tubo" min="20" max="100" value="50">
  </div>
  <div class="slider-group">
    <label for="velocita-tubo" class="slider-label">Velocita ciclo tubo</label>
    <input type="range" class="modern-slider" id="velocita-tubo" min="10" max="100" value="40">
  </div>
```

Verifica: Aprire il file in browser, i 6 slider devono apparire nel pannello a destra.

---

## Task 3: Embed SVG dei pezzi come stringhe JavaScript

**Files:**
- Modify: `index 2.html` (nella sezione `<script>`, prima delle variabili esistenti)

- [ ] **Step 3: Aggiungere array `PEZZI_SVG` con i path dei 4 file**

Subito all'inizio del tag `<script>`, aggiungere:

```javascript
const PEZZI_SVG = [
  {
    name: 'faccia1',
    viewBox: '0 0 625.5 434.1',
    paths: `
      <path d="M163.1,231.9c-27.3,16.2-59.7,21.2-90.1,10.5-35.9-12.6-61.4-42.6-69.9-79.7C-9.5,107.4,16.7,49.5,66.8,23.2c38-20,83.2-16.2,115.9,10.9,51.4,42.7,53.8,120,11.6,170.5-11.7,8.5-21.2,16.9-31.1,27.2Z" fill="#fff"/>
      <path d="M469.9,225.2c-13.1-15.8-27.6-26.6-44-37.7-26.5-36.8-32.1-85.2-12.6-126.2,26.2-55,90.1-77.1,144.2-49.2,56,28.9,82.2,96.4,60.1,155.3-22.6,60.4-90.8,87.9-147.8,57.8Z" fill="#fff"/>
      <path d="M482.3,405.5c-10.5-6.9-20-11.3-33.2-10,23.9-76.8,11.7-142.6-45.8-201.8,89.1,44.3,130.6,147.4,100.7,240.4-6.8-11.7-12.5-20.8-21.7-28.7Z" fill="#fff"/>
      <path d="M178.3,395.7c-22.6-1-40.3,12.3-51.6,33.9-25.8-87.1,29-198.3,110-244.5-34.5,38.2-73.1,99.3-69.4,151.6.9,20.2,4.9,38.5,11,59Z" fill="#fff"/>
      <path d="M308.1,186.9c-8.2-7.6-12.3-11.6-22.8-18,17.4-4,36-5.7,54.2-1.7-10.8,6,19,21.8-31.5,19.7Z" fill="#fff"/>
      <path fill="none" stroke="#fcfcfc" stroke-miterlimit="10" stroke-width="14" d="M172.8,395.2s-24.8-218.3,135.2-218.3"/>
      <path fill="none" stroke="#fcfcfc" stroke-miterlimit="10" stroke-width="14" d="M465.3,395.2s24.8-218.3-135.2-218.3"/>
    `
  },
  {
    name: 'mano',
    viewBox: '0 0 252.3 280.2',
    paths: `
      <path d="M121.6,200.8c-1.4-4.3-3.9-6.9-6.2-9.9s-4.5-4.7-8.7-3.2c-20.3,7.4-41.3,12.3-62.8,12.2-15.3,0-35.6-4.8-41.7-19-6-14,.7-29.2,13.4-37.1,21.5-13.3,53-12,78.4-8.4-.4-2.1-2.3-3.7-4.1-4.4-9.7-3.7-19-7.3-28.3-12l-10.7-6.7c-16.2-10.1-32.6-30.2-25.7-48.9,3.3-9.1,9.9-17.1,19.2-20.1,8.4-2.6,17.3-1.4,25.5,1.8,15.8,6.3,29.5,15.9,41.8,27.5l10.8,10.1c.3-2.2.1-4.1-.5-5.3l-8.6-17.5c-7.8-15.9-12.9-44,3.2-55.1,12.3-8.4,29-6.1,39.3,4.8,12.9,13.6,19.8,38.8,22.7,57.8.5,3.3,1.3,5.8,4.6,7.2,9.6,4.1,18.9,8.8,27.3,15.2l5.4,5c2.8,2.5,14-7.8,24,4.5,14.1,17.3,14.2,50.2,9.3,72.2s-5.2,15.7-10.6,21.6-13.8,6.9-20.7,2.9-4.3-1.3-6.2.3c-8.5,7.4-18,11.7-28.8,14.8s-14.5,15.2-17.2,22.9l-7.8,22.2c-2.7,7.7-7.5,14.9-14.5,19.5-8.1,5.3-18.4,5.7-27.7,1.4s-13.7-10.2-16.4-19.4c-6-21.1,5.8-41.8,22.1-57.2ZM132,193.6c2.1,1.8,4.6,4.9,4,7.1-13.8,11.5-26.5,25.7-27.7,43.7s2.6,18,9.9,23.2c8.1,5.8,19.1,4.9,25.6-2.6,11.3-13.1,8.8-32.7,24.2-52.3,3-3.8,5.9-8.7,10.5-10l8.6-2.5c9.6-4.2,17.7-10.1,25.1-17.5.7-.4,2.2-1.4,2.8-1.2,2.9,1.2,7.6,9.4,16.8,4.9,1.3-1.2,3.6-4,4.4-5.5,5.3-11.1,7.1-22.2,7.1-34.3,0-13.8-2.1-37.1-11.5-44.4s-9.7,2.8-16.8,6c-4.9-7.9-10.2-14.1-18.3-17.8l-17.8-8.1c-2.9-1.3-6.2-2.8-8.3-5.2-2-17-5.8-33.2-12.9-48.5s-8.7-14.9-17.1-18.6c-6.5-3-14.7-2-20.3,2.7-12.2,10.1-3.4,36.6,4.5,50.4l10.9,18.9c.5.9,1.3,3.4.6,3.9l-2.5,1.8c-4.1,3-8.6,5.6-13.3,6.9l-14.9-14.8c-9.1-9-19.4-16-30.9-21.9-7.4-3.8-15.3-6.2-23.9-4.9s-10.1,3.3-13.1,7.1c-7,8.8-5.9,20.3.1,29.3,12.7,19,37.8,29.3,59.8,34.8,2.6.7,6,2.4,7.4,4.4l-.3,9.4c-.1,3-.7,6-1.9,8.9-20.9-3.6-41.4-4.5-62.1-.9s-18.9,5.1-26,12c-5.2,5-6.6,13.4-4,20,4.5,11.7,23.6,14.9,36.9,14.3,21.3-.8,41.3-5.8,60.7-14.2,2.2-1,5.3-1.5,7.3-1.3,5.1,6.2,10.1,11.4,16.3,16.7Z" fill="#fff"/>
      <path d="M188.4,127.2c-1.3-4.6-4.2-5.9-6.9-8-3.6-3.9-8.3-6.6-14.6-6.6,0,2.8,1.7,6.4,4.4,8,5.4,3.1,10.5,6.1,17,6.6ZM177.9,146.1l1.8-3.5c-8.2-4.6-22.2-9.8-27.7-4.2,1,8.1,15.2,7.3,25.9,7.8ZM181,166.1c-1.6-5.2-10.9-4.6-18.1-3.7-3.7.5-6.7,1.5-9.2,4.8,5.3,5.9,19.3,2.4,27.3-1.1Z" fill="#fff"/>
      <path d="M177.9,146.1c-10.7-.5-24.9.3-25.9-7.8,5.6-5.5,19.5-.4,27.7,4.2l-1.8,3.5Z" fill="#fff"/>
      <path d="M181,166.1c-8.1,3.5-22,7-27.3,1.1,2.4-3.3,5.4-4.3,9.2-4.8,7.3-.9,16.5-1.6,18.1,3.7Z" fill="#fff"/>
      <path d="M188.4,127.2c-6.6-.6-11.6-3.5-17-6.6s-4.4-5.2-4.4-8c6.2,0,11,2.6,14.6,6.6,2.7,2.1,5.6,3.4,6.9,8Z" fill="#fff"/>
    `
  },
  {
    name: 'scarpa',
    viewBox: '0 0 373.7 226.7',
    paths: `
      <path d="M47.5,176.3l-22-22.1c-12-12-18.7-28.9-22.7-45.4C-9.4,58.7,20.2,15.4,68.8,7.1c30.9-5.3,62.1,6.1,86.6,25.2,2.9,2.3,5.8,7.4,9.9,7.1,8.6-.5,16.5,1.8,23.4,6.4,2.2,1.5,8.9,1.1,10.7-.8,5.2-5.5,1.1-22.7,11.5-32C220.1,4.8,232.2.1,244.7.1h43.6c17.7-.1,46.9,12.1,60.8,26.8s15,31.5,4.3,46.9c1,3.6,2.6,7.3,4.8,10.2,6.4,8.4,11.8,17.4,14,28.1,7.3,35.1-14.2,66.3-43.7,84.9s-25.9,14.7-40.6,18.8c-14.7,4.1-29,7.5-44.4,9.2-28.5,3-56.3,1.8-84.9-2-40.7-5.5-79.3-20.2-111.3-46.7ZM6.1,78.1c-.6,4.8-.9,9.4,0,14.1l3.7,20.6c4.3,17.4,14.1,31.7,26,44.8,7,7.7,14.2,14.8,22.7,20.8,37.2,26.4,69.8,35.4,114.7,39.6l10.6,1,21.6,1.1,31.4-.7c34.7-3.3,71.3-11.7,98.8-33.7,10.3-8.2,19.5-16.8,25.3-28.5,12.4-25.1,8.3-53.6-12.1-72.7-1.8-1.7-4-7.4-2.4-9.2,6.2-6.9,10.8-16.8,8-25.9s-5.6-14.4-10.9-18.7c-7.9-6.5-15.5-11.3-24.7-14.5l-8.5-3c-10.1-3.5-19.9-4.1-30.2-5.1-16.7-1.6-33.7-1.5-49.4,3.7-12.5,4.1-19.7,13.2-19.9,26.7l-.3,15c-.3,1.7-5.5,2.1-7.7,2.1,3.2,10.9,12.4,20.8,12.3,34.5l-10.7-17.7-4.6-6.6-14.2-14.6c-2.1-2.2-7.8-4.8-10-3.2-3.4,2.5.3,6.5,1.1,8.6l5.6,15.2c1.7,4.7,1.3,11.5,1.2,16.9l-11.2-27.4c-10.2-24.9-53.8-49.5-80.9-49.3h-18.4c-14.3.2-35.4,12.1-46.8,23-11.7,11.3-18,26.6-20.1,43.1Z" fill="#fff"/>
      <path d="M283.7,61.2h12.9c13.8,0,33.6-3.8,32-14.2-1.6-10.6-27.9-15.8-42.9-17l-26-2.1-10.2,2.4c-5.4,1.3-11.9,3.6-15.2,8.4s-.4,7.1,1.1,8.4c9.7,7.6,22.8,11.5,34.9,12.7l13.4,1.4Z" fill="#fff"/>
    `
  },
  {
    name: 'facciaTopolino',
    viewBox: '0 0 322.4 371.8',
    paths: `
      <path d="M113.9,344.5l-12.3-3.6c-35.8-13.5-75.7-40.2-94-73.5-13.4-24.3-8.4-52.3,11.7-71.1,9.1-8.5,20.7-12.4,33.4-7.7-21.5-60.7-14.7-121.2,31.1-166.4,11.3-11.1,15-15.1,32.9-20,17-6.3,30,5.2,44.6,15.1C186.4-.7,194.4-5.7,222.2,7.2c4.5,2.1,8.1,6.3,11.8,9.6,49,44.8,58.6,109.2,35.6,171.8,13-4.9,25.1-.5,34.2,8.3,19.2,18.8,24.4,45.9,11.9,70-19.6,38.1-66.5,66.8-107.6,79.5l-13.5,12.3c-19,17.3-46.7,17.6-65.4,0l-15.2-14.3ZM196.4,78.8c-5.2-5.6-13.8-4.8-17.8,1.4-14,22-10.2,74.6-5,103.2-9.3-.4-17-.3-26.3.3,5.5-28.2,9.2-85.3-5.7-104.2-4.6-5.9-13.3-5.8-17.7,0-17.5,23.3-20.7,85.1.8,108.3l-14.5,7.2c28.6-9.4,59.7-10.4,88.3-3.8,7.7,1.8,13.4,5.9,20.3,10-4.8-7.4-13-10.3-21.9-13.5,21.4-24.5,17-90.5-.4-109.1ZM202,260.9c9-6.8,12.9-16.5,12.4-26.1-.5-8.9-5.2-17.3-13.5-23.3-23.7-17.1-57.9-16.7-81,1.3-8.1,6.3-12.3,15.3-12,24.6.3,8.3,4.3,17.2,12,23.2,23.4,18.1,58.1,18.5,82.2.4ZM291.7,253.2c-9.7-15-28.5-16.5-42.4-8.7l22.1-.3c-15.3,38.4-72.1,65.8-111,65.5-37.4-.3-89.9-27.5-105-65l20.9-.5c-16-9-36.2-4.1-43,13.7,6.2-5.8,9.9-9.2,18.3-12.3,6.6,21.9,24.7,32.7,39.6,47.1,11.5,13,18.5,29.7,28.7,43.7,18.4,25.1,51.2,30.9,75.1,9.5,7.2-6.4,13.2-15.1,18.7-23.5l20.1-31c18.5-11.5,33.2-25.9,42.4-45.8l15.5,7.5Z" fill="#fff"/>
    `
  }
];
```

Verifica: Nessun errore di sintassi JavaScript. I path devono corrispondere ai contenuti originali dei file SVG in `pezzi topolino/`.

---

## Task 4: Aggiungere variabili globali e listener slider

**Files:**
- Modify: `index 2.html` (nella sezione `<script>`, dopo le variabili esistenti)

- [ ] **Step 4: Aggiungere variabili e listener per il tubo**

Dopo la riga `let tFlags = 0;`, aggiungere:

```javascript
  let intensitaTubo = 50;
  let velocitaTubo = 40;
  let tTubo = 0;
  let tuboState = 0; // 0=IDLE, 1=CHARGING, 2=FIRING, 3=COOLDOWN
  let pezzoIndex = 0;
  let pezzi = [];
  const MAX_PEZZI = 200;
  const TUBO_ORIGIN = { x: 540, y: 1485 };
  const TUBO_SPAWN = { x: 539, y: 1452 };
  const pezziContainer = document.getElementById('pezzi-container');
  const tuboGroup = document.getElementById('Tubo');
```

Dopo l'ultimo listener esistente (`velocita-bandiere`), aggiungere:

```javascript
  document.getElementById('intensita-tubo').addEventListener('input', (e) => {
    intensitaTubo = parseFloat(e.target.value);
  });
  document.getElementById('velocita-tubo').addEventListener('input', (e) => {
    velocitaTubo = parseFloat(e.target.value);
  });
```

Verifica: Aprire in browser, muovere i 2 nuovi slider, controllare in console che `intensitaTubo` e `velocitaTubo` cambino.

---

## Task 5: Implementare la macchina a stati del tubo e la fisica dei pezzi

**Files:**
- Modify: `index 2.html` (nella funzione `update()`, prima di `requestAnimationFrame`)

- [ ] **Step 5: Aggiungere logica tubo e pezzi nel loop `update()`**

Prima della riga `requestAnimationFrame(update);` alla fine della funzione `update()`, aggiungere:

```javascript
    // --- INIZIO LOGICA TUBO ---
    if (tuboGroup && pezziContainer) {
      let cycleDuration = 2000 - (velocitaTubo - 10) * 20; // 10→2000ms, 100→400ms
      let dt = 16.7; // approssimazione deltaTime in ms (60fps)
      tTubo += dt;

      let chargeDur = cycleDuration * 0.40;
      let fireDur = cycleDuration * 0.10;
      let cooldownDur = cycleDuration * 0.50;

      if (tuboState === 0 && tTubo >= 0) {
        tuboState = 1; // IDLE → CHARGING
        tTubo = 0;
      } else if (tuboState === 1 && tTubo >= chargeDur) {
        tuboState = 2; // CHARGING → FIRING
        tTubo = 0;

        // SPAWN PEZZO
        let svgData = PEZZI_SVG[pezzoIndex];
        pezzoIndex = (pezzoIndex + 1) % PEZZI_SVG.length;

        let g = document.createElementNS('http://www.w3.org/2000/svg', 'g');
        g.innerHTML = svgData.paths;
        g.setAttribute('style', 'pointer-events: none;');
        pezziContainer.appendChild(g);

        let vb = svgData.viewBox.split(' ').map(Number);
        let vbW = vb[2], vbH = vb[3];
        let scale = 0.25;

        let pezzo = {
          el: g,
          x: TUBO_SPAWN.x + (Math.random() - 0.5) * 10,
          y: TUBO_SPAWN.y,
          vx: (Math.random() - 0.5) * 6,
          vy: -6 - (intensitaTubo * 0.14),
          rotation: Math.random() * 30 - 15,
          vRot: Math.random() * 10 - 5,
          landed: false,
          vbW: vbW,
          vbH: vbH,
          scale: scale
        };
        pezzi.push(pezzo);

        // Rimuovi vecchi pezzi se oltre MAX_PEZZI
        if (pezzi.length > MAX_PEZZI) {
          let old = pezzi.shift();
          if (old.el.parentNode) old.el.parentNode.removeChild(old.el);
        }

      } else if (tuboState === 2 && tTubo >= fireDur) {
        tuboState = 3; // FIRING → COOLDOWN
        tTubo = 0;
      } else if (tuboState === 3 && tTubo >= cooldownDur) {
        tuboState = 1; // COOLDOWN → CHARGING (loop)
        tTubo = 0;
      }

      // Animazione transform tubo
      let scaleX = 1, scaleY = 1;
      if (tuboState === 1) {
        let p = Math.min(1, tTubo / chargeDur);
        scaleX = 1 - p * 0.05;   // 1 → 0.95
        scaleY = 1 + p * 0.15;   // 1 → 1.15
      } else if (tuboState === 2) {
        let p = Math.min(1, tTubo / fireDur);
        scaleX = 0.95 + p * 0.10; // 0.95 → 1.05
        scaleY = 1.15 - p * 0.30; // 1.15 → 0.85
      }
      tuboGroup.setAttribute('transform',
        `translate(${TUBO_ORIGIN.x},${TUBO_ORIGIN.y}) scale(${scaleX},${scaleY}) translate(${-TUBO_ORIGIN.x},${-TUBO_ORIGIN.y})`
      );
    }
    // --- FINE LOGICA TUBO ---

    // --- INIZIO FISICA PEZZI ---
    for (let i = 0; i < pezzi.length; i++) {
      let p = pezzi[i];
      if (p.landed) continue;

      p.vy += 0.4; // gravità
      p.x += p.vx;
      p.y += p.vy;
      p.rotation += p.vRot;
      p.vx *= 0.995; // attrito aria

      let yTarget = 1550 + Math.random() * 370; // 1550–1920
      if (p.y >= yTarget) {
        p.y = yTarget;
        p.landed = true;
        p.vx = 0;
        p.vy = 0;
        p.vRot = 0;
        p.rotation += Math.random() * 30 - 15;
      }

      let cx = p.vbW / 2;
      let cy = p.vbH / 2;
      p.el.setAttribute('transform',
        `translate(${p.x},${p.y}) rotate(${p.rotation} ${cx} ${cy}) scale(${p.scale}) translate(${-cx},${-cy})`
      );
    }
    // --- FINE FISICA PEZZI ---
```

Verifica: Aprire in browser. Il tubo deve gonfiarsi e sgonfiarsi ciclicamente. Dopo qualche secondo i pezzi devono apparire e cadere verso il basso accumulandosi. I 2 nuovi slider devono modulare velocità e intensità.

---

## Task 6: Aggiungere CSS per il transform-origin del tubo

**Files:**
- Modify: `index 2.html` (nella sezione `<style>`, dopo `#forno-pulse`)

- [ ] **Step 6: Aggiungere regola CSS per il tubo**

Dopo la regola `#forno-pulse { transform-origin: 540px 1480px; }`, aggiungere:

```css
  #Tubo {
    transform-origin: 540px 1485px;
  }
```

Verifica: Aprire in browser, ispezionare il gruppo `Tubo` — deve avere `transform-origin: 540px 1485px` nelle computed styles.

---

## Task 7: Commit finale e test visivo

**Files:**
- Modify: `index 2.html`

- [ ] **Step 7: Testare in browser**

Aprire `index 2.html` direttamente nel browser (doppio click). Verificare:
1. I 6 slider appaiono nel pannello a destra
2. I 4 slider vecchi continuano a funzionare (castello pulsa, bandiere sventolano)
3. Il tubo si gonfia e sgonfia ciclicamente
4. I pezzi escono dal tubo uno dopo l'altro in sequenza
5. I pezzi volano e si accumulano nella parte bassa del poster
6. Dopo ~80 cicli (circa 2 minuti a velocità 40), i pezzi più vecchi vengono rimossi per evitare overflow DOM
7. Lo slider "Intensità espulsione" aumenta/diminuisce l'altezza del lancio
8. Lo slider "Velocità ciclo tubo" aumenta/diminuisce la frequenza degli spari

- [ ] **Step 8: Commit**

```bash
git add "index 2.html"
git commit -m "feat: tubo espulsione pezzi topolino con 2 slider"
```

---

## Spec Coverage Checklist

| Requisito Spec | Task che lo implementa |
|----------------|------------------------|
| Gruppo `pezzi-container` dopo `bg-static` | Task 1, Step 1 |
| 2 slider nuovi nel pannello | Task 2, Step 2 |
| Embed SVG pezzi come stringhe JS | Task 3, Step 3 |
| Macchina a stati tubo (4 stati) | Task 5, Step 5 |
| Ciclo CHARGING→FIRING→COOLDOWN | Task 5, Step 5 |
| Spawn pezzi in sequenza ciclica | Task 5, Step 5 |
| Fisica: gravità, attrito, rotazione | Task 5, Step 5 |
| Atterraggio con yTarget casuale | Task 5, Step 5 |
| Limite MAX_PEZZI = 200 (FIFO) | Task 5, Step 5 |
| Transform-origin tubo | Task 6, Step 6 |
| Integrazione loop rAF esistente | Task 5, Step 5 |
| Listener slider tubo | Task 4, Step 4 |
| Stile CSS per tubo | Task 6, Step 6 |
| Nessun framework / nessuna CDN | Tutti i task |
| Funzionamento su `file://` | Task 3, Step 3 (embed stringhe) |

## Placeholder Scan

- Nessun TBD/TODO trovato.
- Tutti i task contengono codice completo o markup esatto.
- I nomi delle variabili sono consistenti: `intensitaTubo`, `velocitaTubo`, `tuboState`, `pezzi`, `MAX_PEZZI`, ecc.

## Type Consistency

- `tuboState` è un numero 0–3, non una stringa.
- `tTubo` accumula ms (float).
- `intensitaTubo` e `velocitaTubo` sono float (da parseFloat).
- I transform SVG usano stringhe interpolate.

---

**Piano completo e salvato in `docs/superpowers/plans/2026-06-11-tubo-espulsione-pezzi.md`.**

**Due opzioni di esecuzione:**

**1. Subagent-Driven (raccomandato)** — Dispaccio un subagent fresco per ogni task, revisione tra i task, iterazione rapida

**2. Inline Execution** — Eseguo i task in questa sessione, esecuzione a batch con checkpoint per revisione

**Quale preferisci?**
