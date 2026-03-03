# T1D Spatial Atlas — Development Log

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
