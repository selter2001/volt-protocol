# Architecture Patterns

**Domain:** Luxury dark UI redesign — single-file HTML application (VoltProtocol)
**Researched:** 2026-02-24
**Confidence:** HIGH (based on direct code inspection + verified CSS/Tailwind docs)

---

## Recommended Architecture

### The Single-File Constraint

The entire redesign lives inside one `index.html`. There is no build step, no bundler, no separate CSS file. This shapes every decision.

The file already has two distinct "zones":

```
<head>
  <!-- External CDN scripts -->
  <style type="text/tailwindcss">   ← THE CSS ZONE (add here)
    @theme { ... }
    @layer base { ... }
    @layer components { ... }
    @keyframes { ... }
  </style>
</head>

<body>
  <!-- HTML markup -->
  <script>
    // === ALL JS ===           ← THE JS ZONE (already organized)
  </script>
</body>
```

**The CSS zone does not yet exist.** The current file has no `<style>` block at all. It relies purely on Tailwind utility classes applied as strings inside JS template literals. The redesign introduces a `<style type="text/tailwindcss">` block — the Tailwind Play CDN processes this tag type natively.

---

## CSS Organization Approach

### Layer Stack (inside the single `<style type="text/tailwindcss">` tag)

```css
/* 1. DESIGN TOKENS — :root custom properties */
@theme {
  --color-void: #0a0a0f;
  --color-cyan: #00f0ff;
  --color-platinum: #e0e0e8;
  --color-glass-bg: rgba(255, 255, 255, 0.04);
  --color-glass-border: rgba(0, 240, 255, 0.15);
  --color-surface-1: rgba(255, 255, 255, 0.06);
  --color-surface-2: rgba(255, 255, 255, 0.10);
  --color-positive: #22c55e;
  --color-negative: #ef4444;
  --font-mono: 'JetBrains Mono', 'Fira Code', monospace;
  --font-sans: 'Inter', 'Plus Jakarta Sans', system-ui, sans-serif;
  --radius-card: 24px;
  --radius-input: 8px;
  --blur-glass: 12px;
  --shadow-glow-cyan: 0 0 20px rgba(0, 240, 255, 0.15);
  --shadow-neomorph-rest: 4px 4px 8px rgba(0,0,0,0.5), -2px -2px 6px rgba(255,255,255,0.03);
  --shadow-neomorph-pressed: inset 2px 2px 6px rgba(0,0,0,0.6), inset -1px -1px 4px rgba(255,255,255,0.02);
  --transition-base: 200ms cubic-bezier(0.4, 0, 0.2, 1);
}

/* 2. BASE RESET — body, root element, scrollbar */
@layer base {
  body { ... }
  ::-webkit-scrollbar { ... }
  @media (prefers-reduced-motion: reduce) { ... }
}

/* 3. COMPONENT STYLES — reusable named patterns */
@layer components {
  .card-glass { ... }
  .input-glass { ... }
  .btn-primary { ... }
  .btn-secondary { ... }
  .tab-item { ... }
  .tab-item--active { ... }
  .verdict-positive { ... }
  .verdict-negative { ... }
  .table-dark { ... }
  .section-header { ... }
  .hero-result-card { ... }
}

/* 4. ANIMATIONS — keyframes + animation utilities */
@keyframes glow-pulse { ... }
@keyframes count-up { ... }
@keyframes flow-line { ... }
@keyframes fade-in-up { ... }

@layer utilities {
  .animate-glow-pulse { animation: glow-pulse 2s ease-in-out infinite; }
  .animate-flow { animation: flow-line 3s linear infinite; }
}
```

**Rationale:** This order guarantees correct cascade precedence. `@theme` tokens are available to all subsequent layers. Base styles override browser defaults first. Component styles compose tokens. Animations are last because they reference tokens and components.

---

## Component Boundaries

The existing app has 6 distinct render contexts. Each maps to a visual component group:

| Component Group | What It Covers | Rendered By | Visual Treatment |
|----------------|----------------|-------------|-----------------|
| **App Shell** | Body, app container, noise texture | HTML `<body>` + `#app` | Void Black bg, grain texture via SVG filter |
| **Top Bar** | Title + action buttons (Zapisz, Eksport, PDF) | `renderFormTab()` header | Glassmorphic strip, cyan-accented buttons |
| **Tab Navigation** | 5 tab buttons | `renderTabs()` | Glowing active indicator, smooth border transition |
| **Bento Cards** | Each `<div class="bg-white border rounded p-4 mb-4">` in the form | `renderFormTab()` sections | 24px radius cards, glass surface-1 |
| **Data Tables** | SWZ / IZOL / RCD / UZIEM measurement tables | `renderAttachment1-4()` | Dark table headers, glass rows, monospace values |
| **Hero Result Card** | Orzeczenie section (final verdict) | `renderFormTab()` last section | Grain-gradient bg, large typography, glowing verdict text |

---

## Data Flow Direction

```
AppState (JS object)
  ↓ mutated by event handlers (handleSWZFieldChange etc.)
  ↓ recalculated by calc* functions
  ↓ written to DOM by render* functions
  ↓ CSS classes/custom properties style the DOM

Animations hook in via:
  - CSS transitions on class changes (verdict color flip)
  - JS count-up on textContent writes to computed cells
  - CSS keyframes on elements with animation utility classes
  - EventBus.on('attachment*:changed') → trigger enter animations
```

The JS event system does NOT need modification for the redesign. The existing pattern of:
1. User input → event listener reads `el.dataset.field`
2. Mutates `AppState`
3. Calls `calc*()` → updates `row.calculated`
4. Calls targeted DOM update (`updateSWZComputedCells`, `updateIZOLVerdict`, etc.)
5. Calls `updateFinalAssessmentDisplay()`

This pattern is **animation-integration-ready**: step 4 (targeted DOM update) is the injection point for count-up animations. Step 5 is the injection point for hero card glow triggers.

---

## Patterns to Follow

### Pattern 1: Token-Based Theming via @theme

**What:** Define all colors, spacing, blur values, shadows as `@theme` CSS custom properties. Never hardcode `#00f0ff` in a component rule — always use `var(--color-cyan)`.

**When:** Every CSS rule in `@layer components`. Every animation keyframe that references a color.

**Example:**
```css
@theme {
  --color-cyan: #00f0ff;
  --blur-glass: 12px;
}

@layer components {
  .input-glass {
    background: rgba(255, 255, 255, 0.04);
    backdrop-filter: blur(var(--blur-glass));
    border: 0.5px solid var(--color-glass-border);
  }
}
```

**Why:** When tuning values (e.g., reducing blur on mobile), you change one token, not 20 rules.

---

### Pattern 2: Glassmorphism Formula for Dark Background

**What:** The correct glassmorphism recipe on a dark (#0a0a0f) background. Standard tutorials assume light backgrounds — dark glassmorphism needs different ratios.

**Recipe:**
```css
.card-glass {
  background: rgba(255, 255, 255, 0.04);   /* very low alpha — dark bg */
  backdrop-filter: blur(12px) saturate(1.5);
  border: 0.5px solid rgba(0, 240, 255, 0.15); /* electric cyan tint, ultra-thin */
  border-radius: var(--radius-card);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.4), var(--shadow-glow-cyan);
}
```

**Why NOT `rgba(255,255,255,0.1)`:** On a near-black background, 10% white looks washed out. 4% is visually correct for "frosted dark glass."

**Fallback (no backdrop-filter support):**
```css
.card-glass {
  background: rgba(20, 20, 30, 0.85); /* fallback: opaque dark */
}
@supports (backdrop-filter: blur(1px)) {
  .card-glass {
    background: rgba(255, 255, 255, 0.04);
    backdrop-filter: blur(12px) saturate(1.5);
  }
}
```

---

### Pattern 3: JS-Injected HTML Classes — Additive Strategy

**What:** The existing `renderSWZRowCells()`, `renderIZOLRowCells()` etc. build HTML strings with hardcoded Tailwind class strings like `'border border-gray-300 px-2 py-1 text-center'`. The redesign replaces these string constants with new semantic class names.

**Strategy:** Replace the existing class string constants at the top of each render function:

```javascript
// BEFORE (current)
const inputCls = 'w-full border border-gray-300 rounded px-1 py-0.5 text-sm text-center';
const selectCls = 'w-full border border-gray-300 rounded px-1 py-0.5 text-sm';

// AFTER (redesigned)
const inputCls = 'input-glass w-full text-sm text-center font-mono';
const selectCls = 'input-glass w-full text-sm';
```

The new semantic classes (`input-glass`, `table-dark`, etc.) are defined in `@layer components`. This approach:
- Changes only the class string constants (2-4 lines per renderer)
- Leaves all logic, data flow, event binding untouched
- Allows granular rollback per renderer if something breaks

---

### Pattern 4: Animation Integration Points

**What:** The existing JS has precise update hooks. Animations attach here without changing the logic.

| Hook | Location | Animation to Attach |
|------|----------|-------------------|
| `updateSWZComputedCells(rowId, result)` | Lines 1567-1585 | Count-up effect on Id and Zsmax cells |
| `updateIZOLVerdict()` / `updateRCDVerdict()` / `updateUZIEMComputedCells()` | Lines 1655-1780 | Color flash + brief glow on verdict change |
| `updateFinalAssessmentDisplay()` | Lines ~1795-1825 | Hero card pulse when verdict changes |
| `EventBus.on('attachment*:changed')` | Lines 2861-2876 | Section fade-in on row add/remove |
| `showToast(message)` | Line 1831 | Toast redesign with glass style + slide-in animation |

**Count-up implementation (no library needed):**
```javascript
function animateCountUp(element, targetValue, decimals = 2) {
  if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) {
    element.textContent = targetValue.toFixed(decimals);
    return;
  }
  const start = 0;
  const duration = 600;
  const startTime = performance.now();
  function update(currentTime) {
    const elapsed = currentTime - startTime;
    const progress = Math.min(elapsed / duration, 1);
    const eased = 1 - Math.pow(1 - progress, 3); // cubic ease-out
    element.textContent = (start + (targetValue - start) * eased).toFixed(decimals);
    if (progress < 1) requestAnimationFrame(update);
  }
  requestAnimationFrame(update);
}
```

**Spring-like transitions via CSS `linear()` — for verdict color changes:**
```css
.verdict-cell {
  transition: color 400ms linear(0, 0.009, 0.035, 0.077, 0.141, 0.225, 0.326, 0.44, 0.557, 0.666, 0.758, 0.829, 0.879, 0.913, 0.935, 0.949, 0.959, 0.966, 0.977, 0.988, 1);
}
```
This is a native CSS spring approximation (no JS library). Confidence: HIGH — documented by Josh Comeau and MDN.

---

### Pattern 5: Flow Effect Architecture

**What:** The "flowing electric lines" connecting sections. This is a decorative overlay, not structural HTML.

**Implementation:** CSS-only using `::before`/`::after` pseudo-elements on section separators with animated linear-gradient backgrounds:

```css
.section-divider::after {
  content: '';
  display: block;
  height: 1px;
  background: linear-gradient(90deg, transparent, var(--color-cyan), transparent);
  background-size: 200% 100%;
  animation: flow-line 2s linear infinite;
}

@keyframes flow-line {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}
```

**Why pseudo-elements:** Zero new DOM nodes, zero JS changes. The JS renders section header `<tr>` rows — adding a CSS class `section-header` to those rows triggers the pseudo-element.

---

### Pattern 6: Noise Texture as CSS (no external image)

**What:** The Void Black background should have a subtle grain texture for depth. Premium apps use noise overlays.

**Implementation:** Inline SVG data URI — no external file, no HTTP request:

```css
body::before {
  content: '';
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 0;
  opacity: 0.03;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)' opacity='1'/%3E%3C/svg%3E");
  background-repeat: repeat;
  background-size: 128px 128px;
}
```

**Why inline SVG:** Single-file constraint. No external request. Scales to any resolution.

---

## Anti-Patterns to Avoid

### Anti-Pattern 1: Re-rendering for Style Changes

**What:** Calling `renderAttachment1()` or `renderFormTab()` when only a visual style needs to change.

**Why bad:** These functions re-render the entire innerHTML of their containers. Any focused input loses focus. Scroll position resets. Causes flicker. The existing codebase already avoids this for field updates — it uses targeted cell updates (`updateSWZComputedCells`). The redesign must follow the same pattern.

**Instead:** Style changes (class additions, CSS variable tweaks) should use `el.classList.add/remove()` or `el.style.setProperty()` on the specific element, never a full re-render.

---

### Anti-Pattern 2: Applying Glassmorphism to Table Cells

**What:** Adding `backdrop-filter: blur()` to every `<td>` or `<tr>`.

**Why bad:** Each element with `backdrop-filter` creates a new stacking context and compositing layer. Hundreds of table cells with blur = catastrophic GPU overdraw on mid-range hardware. Electricians may use older Windows laptops.

**Instead:** Apply glassmorphism only to card containers (`div.card-glass`). Table cells inside those cards inherit the glass appearance from the parent background. The table itself uses semi-transparent dark backgrounds only, no `backdrop-filter`.

```
.card-glass (backdrop-filter: blur — ONE per section)
  └── table.table-dark (background: transparent, no backdrop-filter)
       └── td (background: transparent — inherits parent glass look)
```

---

### Anti-Pattern 3: Replacing All Tailwind Classes with Custom Classes

**What:** Removing Tailwind utility classes entirely from JS-generated HTML and replacing with custom component classes.

**Why bad:** Tailwind utility classes handle layout (`grid`, `flex`, `gap-4`, `max-w-xs`). The redesign is visual only. Layout classes must remain. Replacing them risks breaking responsive behavior.

**Instead:** Keep all layout-related Tailwind utilities. Only replace appearance-related classes:

| Keep (layout) | Replace (appearance) |
|---------------|---------------------|
| `grid grid-cols-3 gap-4` | `bg-white border rounded` → `.card-glass` |
| `overflow-x-auto` | `bg-gray-50` → `bg-void` (token) |
| `flex items-center` | `text-gray-700` → `text-platinum` |
| `max-w-xs` | `border-gray-300` → `border-glass` |

---

### Anti-Pattern 4: `@layer base` Overriding Input Appearance Without `appearance-none`

**What:** Dark styling for `<input>` and `<select>` without resetting browser default appearance.

**Why bad:** Chrome/Safari apply browser-native styling to date inputs, number inputs, and `<select>` dropdowns. CSS rules like `background` and `border` may be overridden or ignored without `appearance: none`.

**Prevention:**
```css
@layer base {
  input, select, textarea {
    appearance: none;
    -webkit-appearance: none;
    /* then apply custom styles */
  }
}
```

For `<input type="date">`: additional webkit pseudo-element targeting needed (`input::-webkit-calendar-picker-indicator`).

---

### Anti-Pattern 5: Animating Non-Composited Properties in Tables

**What:** Using CSS transitions on `width`, `height`, `top`, `left`, `margin`, `padding` for table row animations.

**Why bad:** These properties trigger layout recalculation. A table with 30+ rows doing layout reflow on every input change is unacceptable performance.

**Instead:** Only animate composited properties: `opacity`, `transform`, `filter`, `box-shadow`. For row entry animations:

```css
@keyframes fade-in-up {
  from { opacity: 0; transform: translateY(8px); }
  to   { opacity: 1; transform: translateY(0); }
}

tr[data-row-id] {
  animation: fade-in-up 200ms ease-out both;
}
```

---

## Build Order (Recommended Sequence)

This order is critical. Each step must not break the previous.

### Step 1: Foundation Layer (no visual breakage risk)

Establish tokens and global styles. Nothing is visible yet — this is pure setup.

1. Add `<style type="text/tailwindcss">` block to `<head>`
2. Write `@theme` block with all color/spacing tokens
3. Add `@layer base` body styling: `background: var(--color-void)`, font imports
4. Add noise texture via `body::before`
5. **Verification:** App still functions. Only background color changed.

---

### Step 2: Top Bar + Tab Navigation

Highest visual impact, lowest functional risk. These elements have simple, non-dynamic HTML in the static `<body>` (not JS-generated):

```html
<div id="app" class="max-w-7xl mx-auto p-4">
  <div class="flex items-center justify-between ...">   ← TOP BAR
  <div id="tabs" class="flex border-b ...">             ← TABS
```

1. Style the top bar container: glass strip, refined typography
2. Style action buttons: neomorphic depth for primary (PDF), ghost variants for secondary
3. Style tab navigation: glowing active state via CSS custom property
4. **Verification:** Tab switching still works. All buttons trigger their actions.

---

### Step 3: Form Tab Cards (Protokol — Tab 0)

The main protocol form. This tab's HTML is generated by `renderFormTab()`. The render function uses two string constants:

```javascript
const inputCls = 'w-full border border-gray-300 rounded px-2 py-1.5 text-sm';
const labelCls = 'block text-sm font-medium text-gray-700 mb-1';
```

Replacement approach:
1. Define `.card-glass`, `.input-glass`, `.label-dark` in `@layer components`
2. Update `inputCls` and `labelCls` constants in `renderFormTab()`
3. Update the card wrapper `'bg-white border rounded p-4 mb-4'` → `'card-glass p-6 mb-4'`
4. Style the Hero Result Card (Orzeczenie section) with grain-gradient and large verdict typography
5. **Verification:** All form inputs still accept data. Local storage save/load works.

---

### Step 4: Data Tables (Attachments 1-4)

The most complex step. Each attachment renderer has its own `inputCls`/`selectCls` constants. Four renderers, similar patterns.

Order within this step:
1. **Attachment 1 (SWZ)** — most complex, most rows. Do first to establish the pattern.
2. **Attachment 3 (RCD)** — simpler table, validates the pattern
3. **Attachment 4 (Uziemienie)** — simpler table
4. **Attachment 2 (Izolacja)** — widest table (10 Rp columns), needs separate mobile handling

For each attachment:
1. Update `inputCls`, `selectCls`, cell class strings in the render function
2. Wrap tables in `.card-glass` containers
3. Style verdict cells: `.verdict-positive`, `.verdict-negative` with glow
4. Style section/subsection header rows: `.section-header`, `.subsection-header`
5. **Verification:** Calculations still trigger. Verdicts still update. Add/remove row still works.

---

### Step 5: Animations

Add last, after all static styles are confirmed working. Animations should never be a dependency for functionality.

Order:
1. CSS transitions on tab switching (opacity + transform)
2. Input focus glow states
3. Verdict color transitions (spring-like `linear()`)
4. Count-up animations for Id, Zsmax, Rpo computed cells
5. Flow Effect on section dividers
6. Toast animation redesign
7. Hero Result Card pulse glow
8. **Final verification with `prefers-reduced-motion: reduce`:** All animations suppressed. App still fully usable.

---

## Avoiding Breakage During Redesign

### The PDF Export Firewall

The PDF export (`buildDocDefinition()`, `buildFormMainContent()`, etc.) has its own internal styling — it generates pdfmake JSON, not HTML. CSS changes cannot affect it. It is immune to the visual redesign. However:

- The `verdict` value comes from `row.calculated.verdict` which is always `'POZYTYWNA'`/`'NEGATYWNA'`/`null` — this is not affected by class changes.
- The PDF uses its own color system (pdfmake `color` property). Not affected.
- **Risk: ZERO.** Confirm by running PDF export after Step 3.

### The Word Export Firewall

`buildWordHTML()` generates its own `<style>` block inside the Word document. It uses inline CSS strings. Not affected by the redesign's `<style type="text/tailwindcss">` block.

### The Class-Removal Verification Pattern

After each step, test these invariants:

```
1. Can I enter data in inputs? → Checks input styling didn't break pointer-events
2. Do calculations update? → Checks targeted DOM updates still find cells by selector
3. Can I add/remove rows? → Checks event delegation still works
4. Does tab switching hide/show panels? → Checks 'hidden' class toggle still works
5. Does localStorage save/load? → Checks AppState serialization unaffected
```

The most fragile point: targeted DOM queries like:
```javascript
document.querySelector('.verdict-cell[data-row-id="' + rowId + '"]')
document.querySelector('.id-cell[data-row-id="' + rowId + '"]')
```

These rely on class names `verdict-cell`, `id-cell`, `zsmax-cell`, `izol-verdict-cell`, `rcd-verdict-cell`. **These class names must be preserved exactly** in the redesigned render functions, even when adding new classes alongside them.

Safe pattern:
```javascript
// PRESERVE functional class selectors, ADD visual classes
html += '<td class="verdict-cell verdict-positive" data-row-id="...">POZYTYWNA</td>';
//              ^^^^^^^^^^^^ keep        ^^^^^^^^^^^^^^^^ add new
```

---

## Scalability Considerations

| Concern | Current (single file) | Future (if ever extracted) |
|---------|----------------------|--------------------------|
| Token changes | Edit `@theme` block once | Extract to `tokens.css` |
| Component changes | Edit `@layer components` | Extract to `components.css` |
| New attachment type | Add new render function + classes | Same pattern |
| Performance on 100-row tables | Use `backdrop-filter` on card wrapper only, not cells | No change needed |
| Mobile (Tailwind `sm:` variants) | Reduce blur on mobile via `@media` inside `@theme` override | Same approach |

---

## Sources

- [Tailwind CSS v4 Play CDN — Official Docs](https://tailwindcss.com/docs/installation/play-cdn) — HIGH confidence. `type="text/tailwindcss"` confirmed.
- [Tailwind CSS v4 @theme Directive](https://tailwindcss.com/blog/tailwindcss-v4) — HIGH confidence. CSS custom properties via `@theme` confirmed.
- [backdrop-filter — MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/backdrop-filter) — HIGH confidence. `@supports` fallback pattern confirmed.
- [Dark Glassmorphism: 2026 — Medium](https://medium.com/@developer_89726/dark-glassmorphism-the-aesthetic-that-will-define-ui-in-2026-93aa4153088f) — MEDIUM confidence. Dark-specific alpha ratios (low opacity on dark bg).
- [Springs and Bounces in Native CSS — Josh W. Comeau](https://www.joshwcomeau.com/animation/linear-timing-function/) — HIGH confidence. `linear()` timing function for spring physics confirmed.
- [Animated Counter with Intersection Observer — getButterfly](https://getbutterfly.com/animated-javascript-counter-up-with-the-intersection-observer-api/) — MEDIUM confidence. Count-up pattern via rAF confirmed viable.
- [Neumorphism and CSS — CSS-Tricks](https://css-tricks.com/neumorphism-and-css/) — HIGH confidence. `inset` box-shadow for pressed state confirmed.
- [Glassmorphism UI Best Practices — UX Pilot](https://uxpilot.ai/blogs/glassmorphism-ui) — MEDIUM confidence. Performance + accessibility guidelines.
- Direct code inspection of `index.html` (2923 lines) — HIGH confidence. All JS hook locations, class selectors, render function patterns verified firsthand.
