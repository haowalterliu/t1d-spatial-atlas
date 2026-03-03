# T1D Spatial Atlas — Development Log

## Session: FOV Viewer — Canvas, Axes & Layout Polish
**Date:** 2026-03-03
**File Modified:** `fov-viewer.html`

---

### Summary of Changes

#### 1. JavaScript bug fixes — script was entirely non-functional

Two bugs caused the entire `<script>` block to crash silently:

| Bug | Root cause | Fix |
|-----|-----------|-----|
| Temporal Dead Zone (TDZ) | `viewerCanvas.classList.add('state-light')` called before `const viewerCanvas` was declared | Moved both `const viewerCanvas` and `const confirmOverlay` into the DOM refs block |
| Null element reference | `<div id="confirmOverlay">` was placed **after** `</script>`, so `getElementById` returned `null` | Moved the overlay HTML to before `<script>` |

Downstream effects that were also fixed as a result: Apply button now responds, empty/loading states render correctly, confirm dialog logic works.

---

#### 2. WCAG AA colour fix

`.loading-est` text colour changed from `#6b7280` (contrast 4.37:1, **fails** AA) to `#4b5563` (contrast 6.97:1, **passes** AA) on `#f7f8fa` background.

---

#### 3. Column divider

Added a visible separator between the left Controls column and the right Viewer column.

```css
.left-col {
  border-right: 1px solid var(--border);
  padding-right: 32px;
  margin-right: 32px;
}
```

On mobile (< 900 px) the divider switches to a horizontal `border-bottom`.

---

#### 4. Loading → canvas transition (30 s timer)

After Apply is clicked the canvas enters loading state. After 30 seconds (prototype duration) `showCanvas()` is called automatically:

```js
loadTimer = setTimeout(showCanvas, 30000);

function showCanvas() {
  canvasLoading.style.display = 'none';
  fovDotsEl.style.display = 'grid';
  fovAxesEl.style.display = 'block';
  viewerCrosshair.style.display = 'block';
}
```

If Apply is clicked again mid-loading, the previous timer is cancelled (`clearTimeout`) and a fresh 30 s run begins.

---

#### 5. X/Y axes on canvas

Added coordinate axes rendered in HTML/CSS inside `fov-axes`.

| Axis | Range | Interval |
|------|-------|---------|
| X | 19 000 – 23 000 | 1 000 |
| Y | 36 000 → 32 000 (top → bottom) | 1 000 |

```html
<div class="fov-axes" id="fovAxes">
  <div class="fov-axis-y">
    <span>36000</span> … <span>32000</span>
  </div>
  <div class="fov-axis-x">
    <span>19000</span> … <span>23000</span>
  </div>
</div>
```

`.fov-dots` inset adjusted to `top:10px; left:54px; right:10px; bottom:30px` to leave room for the axes.

---

#### 6. Hero — standardised to shared `page-hero` pattern

**Before:** Custom `ctrl-hero` block lived inside the left column, above the Controls panel. Used bespoke CSS classes (`.ctrl-hero`, `.ctrl-hero-eyebrow`, `.ctrl-hero-title`, `.ctrl-hero-desc`).

**After:** Hero extracted from the left column and placed above `viewer-layout` using the same classes as every other inner page.

```html
<div class="page-hero">
  <div class="page-eyebrow"><span class="eyebrow-line"></span>Spatial Explorer</div>
  <div class="page-h1">FOV Viewer</div>
  <p class="page-desc">…</p>
</div>
```

All custom `.ctrl-hero*` CSS removed. Left column now contains only `ctrl-panel`.

---

#### 7. Canvas — always light mode, WCAG-compliant

**Before:** Canvas defaulted to dark (`#0f1523`) and toggled to light (`#f7f8fa`) via `.state-light` class at runtime.

**After:** Canvas is permanently `#f7f8fa`. Dark default, `.state-light` rule, and all JS `classList` toggles removed.

Axis and crosshair colours updated for light background:

| Element | Before | After |
|---------|--------|-------|
| Axis labels | `rgba(255,255,255,0.45)` | `#4b5563` (6.97:1 — WCAG AA) |
| Tick marks | `rgba(255,255,255,0.25)` | `rgba(0,0,0,0.18)` |
| Crosshair | `rgba(255,255,255,0.6)` | `rgba(99,102,241,0.6)` (indigo) |

---

### New CSS Classes Added

| Class | Purpose |
|-------|---------|
| `.fov-axes` | Absolute overlay containing both axis rulers |
| `.fov-axis-x` | Bottom ruler with X tick labels |
| `.fov-axis-y` | Left ruler with Y tick labels |

---

### JS Changes

| Symbol | Change |
|--------|--------|
| `viewerCanvas` | Moved declaration to DOM refs block (was after first use — TDZ crash) |
| `confirmOverlay` | Moved declaration to DOM refs block (element now exists before script runs) |
| `loadTimer` | New `let` — holds the `setTimeout` handle for the 30 s canvas reveal |
| `showCanvas()` | New function — hides loading panel, shows dots + axes + crosshair |

---

## Session: FOV Viewer UI Refactor
**Date:** 2026-03-03
**File Modified:** `fov-viewer.html`

---

### Summary of Changes

#### 1. Viewer Toolbar — replaced static location badge with dynamic info badges

**Before:**
Single static badge showing `D001 · FOV 001 · x: 0–1000 μm · y: 0–1000 μm`

**After:**
Three dynamic badges (Condition / Donor / FOV) that sync with the left-panel selects in real time via JS event listeners. Removed x/y coordinate display.

```html
<span class="viewer-badge">
  <span class="viewer-badge-label">Condition</span>
  <span id="badge-condition">Control</span>
</span>
```

---

#### 2. Gene Display — added Apply button

Added a submit button below the Gene Display select inside `.ctrl-section`.

```html
<button class="gene-submit-btn">Apply</button>
```

Styled with `var(--accent)` background, full width, hover opacity transition.

---

#### 3. Cell Types — moved from ctrl-panel to canvas right side

**Before:** Cell Types was the last `ctrl-section` inside the left `ctrl-panel`.

**After:** Cell Types is now a standalone `cell-types-panel` placed to the right of the simulated canvas, inside a new `canvas-row` flex container.

```html
<div class="canvas-row">
  <div class="viewer-canvas">...</div>
  <div class="cell-types-panel">
    <div class="cell-types-head">Cell Types</div>
    <!-- legend rows -->
  </div>
</div>
```

---

#### 4. Canvas height — aligned to left panel height

**Before:** `viewer-canvas` used `aspect-ratio: 16/9` (fixed proportions).

**After:**
- `.viewer-layout` changed from `align-items: start` → `align-items: stretch`
- `.viewer-canvas` uses `flex: 1; min-height: 300px` instead of fixed aspect ratio
- Canvas now stretches to match the ctrl-panel height on the left

---

#### 5. Donor & FOV dropdowns — expanded options

| Select   | Before | After |
|----------|--------|-------|
| Donor    | 5 options (HPAP-001–005) | 20 options (HPAP-001–020) |
| FOV      | 5 options (1–5)           | 25 options (1–25)         |

---

### New CSS Classes Added

| Class | Purpose |
|-------|---------|
| `.canvas-row` | Flex row wrapping canvas + cell types panel |
| `.viewer-badge` | Info badge in toolbar |
| `.viewer-badge-label` | Small uppercase label prefix inside badge |
| `.cell-types-panel` | Standalone cell type legend panel |
| `.cell-types-head` | Section header for cell types panel |
| `.gene-submit-btn` | Apply button for Gene Display |

---

### JS Logic Added

Selects for Condition, Donor, and FOV are wired to `change` event listeners that update the toolbar badges and canvas overlay text in real time.

```js
conditionSel.addEventListener('change', updateBadges);
donorSel.addEventListener('change', updateBadges);
fovSel.addEventListener('change', updateBadges);
```
