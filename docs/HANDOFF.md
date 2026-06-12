# Handoff — Poster Disney Interattivo

**Date:** 2026-06-12  
**Branch:** `feat/fill-with-things` (synced with `origin/feat/fill-with-things`)  
**Main file:** `index.html`  
**Audience:** Designer or developer picking up animation work

---

## What this is

A single-page interactive Disney poster (1080×1920 SVG). The castle breathes, three flags wave, and a bottom pipe (`#Tubo`) elastically stretches. On each stretch peak, the pipe is meant to eject cloned Mickey body-part shapes upward.

Open `index.html` directly in a browser — no build step.

---

## Repository layout

```
disney_artwork/
├── index.html                          # ← Main deliverable (all-in-one)
├── index 2.html.backup                 # Older snapshot (forno slider, extra controls)
├── castello disney.svg                 # SVG with renamed groups (source reference)
├── Artwork Disney Tavola Disegno 1.svg # Full original artwork
├── pezzi topolino/                     # Detached Mickey part SVGs (not yet wired in)
│   ├── faccia topolino.svg
│   ├── mano topolino.svg
│   ├── scarpa topolino.svg
│   └── faccia 1.svg
├── AGENTS.md                           # Project conventions (partially outdated — see below)
└── docs/
    ├── HANDOFF.md                      # This file
    └── superpowers/
        ├── specs/2026-06-11-animazione-castello-bandiere-design.md
        └── plans/2026-06-11-animazione-castello-bandiere.md
```

---

## Git history (feature branch)

| Commit     | Summary |
|------------|---------|
| `72b2411`  | feat: add pulsing pipe (anime.js elastic `#Tubo`) |
| `65cc41a`  | Update index.html — pipe spit system scaffold (`#spit-layer`, `spitRandomShape`) |

Base branch: `main` (castle + flags only).

---

## What's working

### 1. Castle pulse + flag wave (native `requestAnimationFrame`)

- **Group:** `#castle-pulse` contains `#castello` + `#bandiera_1/2/3`
- **Loop:** Single `requestAnimationFrame` in inline `<script>`
- **Controls:** 4 sliders (intensity/speed for castle and flags)
- **Transform origin:** `540px 960px` on `#castle-pulse`
- Flags start with empty `d=""`; JS rebuilds Bezier paths every frame

### 2. Pipe stretch (`#Tubo`, anime.js v4)

- **Element:** `<g id="Tubo">` in `#bg-static` — funnel at bottom of poster (~y 1297–1517)
- **CSS:** `transform-box: fill-box; transform-origin: center top`
- **Animation:** Looped timeline — `scaleY` 1 → 1.35 (900ms elastic) → back to 1 (800ms elastic)
- **Import:** `import { createTimeline } from 'https://esm.sh/animejs@4'` (requires network)

### 3. Pipe spit (partial — logic exists, assets missing)

On the stretch **down** phase, when `scaleY` passes its peak and starts shrinking, `spitRandomShape()` runs once per cycle:

1. Picks a random hidden source element from `SPIT_IDS`
2. Clones it into `#spit-layer`
3. Positions clone at the pipe mouth (`getTubeMouth()` via `tubo.getBBox()`)
4. Animates `translateY` upward by `SPIT_RISE` (280px) over 1400ms

**Constants** (in module script):

```javascript
const SPIT_IDS = ['Piede', 'orecchie', 'Faccia', 'Mano'];
const SPIT_RISE = 280;
const SPIT_DURATION = 1400;
```

---

## What's NOT working / incomplete

### Critical: spit source elements are missing from the DOM

The JS and CSS reference four SVG elements that **do not exist** in `index.html`:

| Expected ID | Referenced in | Present in SVG? |
|-------------|---------------|-----------------|
| `Piede`     | CSS + `SPIT_IDS` | **No** |
| `orecchie`  | CSS + `SPIT_IDS` | **No** |
| `Faccia`    | CSS + `SPIT_IDS` | **No** |
| `Mano`      | CSS + `SPIT_IDS` | **No** |

`spitSources` filters to an empty array → `spitRandomShape()` returns immediately. The pipe stretches but nothing is ejected.

**Likely fix:** Embed SVG paths from `pezzi topolino/` as hidden `<g>` elements with those IDs (off-screen or `visibility: hidden`), or inline the paths directly in `index.html`.

### Orphan CSS: `#forno-pulse`

```css
#forno-pulse { transform-origin: 540px 1480px; }
```

No `#forno-pulse` group exists in the current SVG. The backup file had a forno pulse slider — never ported to `index.html`.

### AGENTS.md drift

| AGENTS.md says | Current reality |
|----------------|-----------------|
| Main file is `index 2.html` | Renamed to `index.html` |
| No external libraries | anime.js v4 loaded from esm.sh CDN |
| Single rAF loop for all animations | Pipe uses separate anime.js module script |
| 6 global state variables | Still true for castle/flags only |

Update `AGENTS.md` when the pipe/spit work stabilizes.

### Offline use

Pipe animation **requires internet** for `https://esm.sh/animejs@4`. Castle/flags work offline. To support `file://` fully, vendor anime.js locally (a `libs/` folder was tried and removed in favor of CDN).

### Spit cleanup

Cloned shapes append to `#spit-layer` but are never removed after animation — will accumulate in memory/DOM over long sessions.

### Direction semantics

Objects animate **upward** from the pipe mouth (`translateY` decreases). Visually this reads as things shooting *out* of the pipe, not gravity-falling *down*. Confirm with design intent.

---

## SVG structure (z-order)

```
<svg viewBox="0 0 1080 1920">
  <g id="castle-pulse">     ← behind (animated: scale)
    castello, bandiera_1/2/3
  </g>
  <g id="bg-static">        ← in front (static artwork)
    Tubo, tubi_fermi, stars, Mickey, clouds, gears, ...
  </g>
  <g id="spit-layer"></g>   ← empty container for cloned spit shapes (on top)
</svg>
```

Pipe-related static elements in `#bg-static`:

- `#Tubo` — animated output funnel
- `#tubi_fermi` — factory pipe network (includes central vertical shaft at x≈525, y≈895)

---

## How to preview

```bash
open index.html
# or serve locally if testing module/CORS edge cases:
python3 -m http.server 8080
# then http://localhost:8080/index.html
```

**Checklist:**

- [ ] Castle pulses smoothly; sliders respond
- [ ] Three flags wave with phase offset
- [ ] `#Tubo` stretches elastically on loop
- [ ] Shapes eject from pipe on peak (blocked until source elements exist)
- [ ] Works without network for castle/flags; pipe needs CDN

---

## Suggested next steps (priority order)

1. **Add spit source shapes** — Import `pezzi topolino/*.svg` paths into hidden `<g id="Piede">` etc. inside the SVG (or a `<defs>` + `<use>` pattern). Verify `getBBox()` returns sensible values.

2. **Verify spit timing** — Tune `SPIT_RISE`, `SPIT_DURATION`, and peak detection threshold (`peak.maxScale > 1.05`) against the artwork.

3. **Remove DOM clones** — On anime timeline `onComplete`, remove shape from `#spit-layer`.

4. **Decide offline strategy** — Vendor anime.js under `libs/` or accept CDN-only for pipe.

5. **Reconcile AGENTS.md** — Update file names, document dual animation engines (rAF + anime.js), pipe/spit conventions.

6. **Optional: `#forno-pulse`** — Either wire the oven pulse group from backup or delete orphan CSS.

7. **Merge `feat/fill-with-things` → `main`** — After spit sources are in place and visually approved.

---

## Key code locations in `index.html`

| Lines (approx.) | What |
|-----------------|------|
| 14–31 | CSS: transform origins, hidden spit sources, spit-layer transforms |
| 76–81 | `#castle-pulse` group |
| 83–89 | `#Tubo` |
| 240 | `#spit-layer` |
| 252–345 | Castle/flag rAF loop + slider bindings |
| 346–434 | anime.js module: pipe stretch + spit system |

---

## Design references

- Original spec (castle + flags only): `docs/superpowers/specs/2026-06-11-animazione-castello-bandiere-design.md`
- Implementation plan: `docs/superpowers/plans/2026-06-11-animazione-castello-bandiere.md`

Pipe/spit work is **not** documented in those files — it was added ad hoc on `feat/fill-with-things`.

---

## Contact / context

Branch name `feat/fill-with-things` reflects the goal: populate the factory pipe with animated objects (Mickey parts) ejected from `#Tubo`. Castle and flag animation from the original milestone are stable; pipe spit is the active workstream.
