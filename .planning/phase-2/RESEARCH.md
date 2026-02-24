# Phase 2 Research: Interactive Layer

## Animation Hook Points

### Count-Up Targets (ANIM-02)

Targeted DOM updates (NOT full re-renders) — ideal for element-level animation:

| Cell | Selector | Update Function | Line | Value |
|------|----------|-----------------|------|-------|
| Id | `.id-cell[data-row-id]` | `updateSWZComputedCells()` | ~1746 | `calc.Id.toFixed(2)` |
| Zsmax | `.zsmax-cell[data-row-id]` | `updateSWZComputedCells()` | ~1749 | `calc.Zsmax.toFixed(2)` |
| Rpo | `.uziem-rpo-cell[data-row-id]` | `updateUZIEMComputedCells()` | ~1929 | `calc.Rpo.toFixed(2)` |

Update chain: user input → handleXXXFieldChange() → calcXXXRow() → updateXXXComputedCells() → textContent assignment

### Verdict Glow Targets (ANIM-06)

All verdict cells use classList.add/remove with `text-green-700`/`text-red-700`:

| Selector | Update Function | Lines |
|----------|-----------------|-------|
| `.verdict-cell` | `updateSWZComputedCells()` | ~1753-1755 |
| `.izol-verdict-cell` | `updateIZOLVerdict()` | ~1831-1833 |
| `.rcd-verdict-cell` | `updateRCDVerdict()` | ~1883-1885 |
| `.uziem-verdict-cell` | `updateUZIEMComputedCells()` | ~1936-1938 |
| `#assess-swz/izol/rcd/uziem` | `updateAssessmentCell()` | ~1948-1955 |
| `#orzeczenie-text` | `updateOrzeczenieDisplay()` | ~1977-1986 |

### Button Press Targets (ANIM-03)

CSS-only solution via `:active` pseudo-class — no JS changes needed:
- `.btn-glass:active` → `--shadow-neomorph-pressed` token (already defined in @theme)
- `.btn-primary-glass:active` → same with cyan tint

### Existing Infrastructure

- **@theme tokens**: `--shadow-neomorph-pressed` already defined
- **No animation library** — use native `requestAnimationFrame`
- **prefers-reduced-motion** already handled in CSS (ANIM-07)
- **EventBus** available for coordination but not needed for targeted updates
- **200ms base transition** standard already set

## Approach Decision

**Count-up**: Pure JS with `requestAnimationFrame` — no library needed. ~300ms duration, ease-out curve.
Intercept `textContent` assignments in update functions, animate from old value to new value.

**Verdict glow**: Pure CSS — add `box-shadow` to verdict classes. No JS changes needed.

**Button press**: Pure CSS `:active` state — no JS changes needed.

**prefers-reduced-motion**: Already handled globally — animations will be disabled automatically.
