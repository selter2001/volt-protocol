# Technology Stack

**Project:** VoltProtocol — Luxury Redesign
**Domain:** Glassmorphism / Dark Luxury UI — single-file vanilla web app
**Researched:** 2026-02-24
**Overall confidence:** HIGH (core techniques) / MEDIUM (animation library choice)

---

## Constraint Baseline

The existing app ships as a single `index.html` with no build step. The following is already locked and must NOT change:

| Already in use | Version | Role |
|---------------|---------|------|
| Tailwind CSS Browser (Play CDN) | `@tailwindcss/browser@4` (v4.x) | Utility-first styles |
| pdfmake | `0.3.5` | PDF export |
| Vanilla JS | ES2020+ | All logic |

Everything new must load via CDN `<script>` or `<link>` tag, with zero build tooling.

---

## Recommended Stack — New Additions

### Animation Engine

| Technology | Version | CDN | Purpose | Why |
|-----------|---------|-----|---------|-----|
| anime.js | 4.3.6 | See below | Spring physics, count-up numbers, stagger reveals | Lighter than GSAP (no plugin system needed), ESM + UMD CDN available, native spring() easing, excellent number-tweening API. GSAP is heavier and overkill for this scope. |

**CDN (UMD — works with plain `<script>` tag, no modules needed):**
```html
<script src="https://cdn.jsdelivr.net/npm/animejs@4.3.6/dist/bundles/anime.umd.min.js"></script>
```
After loading, destructure from the global: `const { animate, spring, stagger } = anime;`

**ESM alternative (use inside `<script type="module">`):**
```html
<script type="module">
  import { animate, spring, stagger } from 'https://cdn.jsdelivr.net/npm/animejs@4.3.6/+esm';
</script>
```

> Confidence: MEDIUM — anime.js v4 is current (last release Feb 2025), but its CDN path was verified against jsDelivr and official docs. UMD build confirmed at `dist/bundles/anime.umd.min.js`.

---

### Typography

| Role | Font | Weights | Why |
|------|------|---------|-----|
| Primary UI labels, headings | **Space Grotesk** | 400, 500, 600, 700 | Proportional sans-serif descended from the Space Mono family — shares DNA with the monospace used for numerics. Geometric, technical, premium feel without being "another Inter". Available free on Google Fonts. |
| Numeric values, measurements, code | **JetBrains Mono** | 400, 500 | Purpose-built for developer/technical contexts. Excellent digit legibility, even spacing for numeric alignment, available on Google Fonts. Works with the "precision instrument" narrative. |

**Combined Google Fonts CDN link:**
```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
```

**Usage mapping:**
```css
/* Labels, UI text, headings */
font-family: 'Space Grotesk', system-ui, sans-serif;

/* Numeric values, measurements, calculated results */
font-family: 'JetBrains Mono', 'Courier New', monospace;
```

> Confidence: HIGH — both fonts verified on Google Fonts, CDN format from official Google Fonts API v2 docs.

---

### CSS Techniques (Zero external dependencies)

All techniques below are pure CSS, using features already supported in Tailwind v4 utility classes or implementable via `<style type="text/tailwindcss">`.

#### 1. Glassmorphism 2.0 (Frosted Glass)

The `backdrop-filter: blur()` property is the core mechanism (~95% browser support as of 2025). Combine with semi-transparent backgrounds and ultra-thin borders.

```css
/* Glassmorphism card base */
.glass-card {
  background: rgba(255, 255, 255, 0.03);
  backdrop-filter: blur(20px) saturate(150%);
  -webkit-backdrop-filter: blur(20px) saturate(150%);
  border: 0.5px solid rgba(224, 224, 232, 0.12);  /* Soft Platinum @ 12% */
  border-radius: 24px;
}

/* Input with Electric Cyan border focus */
.glass-input {
  background: rgba(255, 255, 255, 0.04);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border: 0.5px solid rgba(0, 240, 255, 0.25);  /* Cyan @ 25% default */
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
}
.glass-input:focus {
  border-color: rgba(0, 240, 255, 0.8);
  box-shadow: 0 0 0 1px rgba(0, 240, 255, 0.3),
              0 0 20px rgba(0, 240, 255, 0.15);
}
```

Tailwind v4 utility equivalents: `backdrop-blur-xl`, `bg-white/[.03]`, `border border-white/[.12]`

> Confidence: HIGH — `backdrop-filter` support and technique verified across multiple authoritative CSS sources.

#### 2. Noise / Grain Texture (Void Black Background)

Use an inline SVG `feTurbulence` filter applied via a pseudo-element. No external files, no base64, no separate HTTP request.

```html
<!-- Inline SVG noise filter (once in the document) -->
<svg style="position:absolute;width:0;height:0">
  <defs>
    <filter id="noise-filter">
      <feTurbulence
        type="fractalNoise"
        baseFrequency="0.65"
        numOctaves="3"
        stitchTiles="stitch"
        result="noise"
      />
      <feColorMatrix type="saturate" values="0" in="noise" result="greyNoise"/>
      <feBlend in="SourceGraphic" in2="greyNoise" mode="overlay" result="blended"/>
      <feComponentTransfer>
        <feFuncA type="linear" slope="0.08"/>
      </feComponentTransfer>
    </filter>
  </defs>
</svg>
```

```css
/* Apply to body or hero sections */
body::before {
  content: '';
  position: fixed;
  inset: 0;
  background-image: url("data:image/svg+xml,..."); /* or use filter below */
  filter: url(#noise-filter);
  opacity: 0.04;
  pointer-events: none;
  z-index: 0;
}

/* Simpler alternative — CSS-only noise via feTurbulence in background-image */
.noise-bg {
  background-image: url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' width='200' height='200'><filter id='n'><feTurbulence type='fractalNoise' baseFrequency='0.65' numOctaves='3' stitchTiles='stitch'/></filter><rect width='200' height='200' filter='url(%23n)' opacity='0.08'/></svg>");
}
```

> Confidence: HIGH — technique from CSS-Tricks "Grainy Gradients" (authoritative source), supported in all modern browsers.

#### 3. OLED Glow Effects (Glowing Borders, Neon Accents)

Stack multiple `box-shadow` values at increasing blur radii. Use `::after` pseudo-element for animated glow to avoid repaints on the main element.

```css
/* Static cyan glow on result cards */
.glow-cyan {
  box-shadow:
    0 0 2px #00f0ff,
    0 0 10px rgba(0, 240, 255, 0.4),
    0 0 40px rgba(0, 240, 255, 0.15),
    inset 0 0 20px rgba(0, 240, 255, 0.05);
}

/* Text glow on the final verdict */
.glow-text {
  text-shadow:
    0 0 10px rgba(0, 240, 255, 0.8),
    0 0 30px rgba(0, 240, 255, 0.4);
}

/* Animated glow — cheaper via opacity animation on pseudo-element */
.glow-pulse::after {
  content: '';
  position: absolute;
  inset: -1px;
  border-radius: inherit;
  box-shadow: 0 0 20px rgba(0, 240, 255, 0.6);
  animation: glow-breathe 3s ease-in-out infinite;
  pointer-events: none;
}
@keyframes glow-breathe {
  0%, 100% { opacity: 0.3; }
  50%       { opacity: 1; }
}
```

> Confidence: HIGH — standard multi-layer box-shadow pattern, pseudo-element opacity trick confirmed from CSS performance guides.

#### 4. Neomorphic-Depth Buttons (Pressed-In Effect)

Works on the dark `#0a0a0f` background by using inverted dual shadows (no light shadow on near-black — use a slightly lighter shade instead).

```css
.btn-neo {
  background: #0f0f16;
  box-shadow:
    4px 4px 10px rgba(0, 0, 0, 0.6),
    -2px -2px 8px rgba(255, 255, 255, 0.04);
  transition: box-shadow 0.15s ease, transform 0.1s ease;
}
.btn-neo:active {
  box-shadow:
    inset 3px 3px 8px rgba(0, 0, 0, 0.5),
    inset -1px -1px 5px rgba(255, 255, 255, 0.03);
  transform: scale(0.98);
}
```

> Confidence: HIGH — inset shadow technique is standard neumorphism, verified across multiple CSS tutorials.

#### 5. "Flow Effect" — Animated Lines (Electricity Between Sections)

Pure SVG with CSS `stroke-dashoffset` animation. No library needed.

```html
<svg class="flow-connector" viewBox="0 0 2 100" preserveAspectRatio="none">
  <path
    d="M1,0 L1,100"
    stroke="url(#flow-gradient)"
    stroke-width="1"
    fill="none"
    stroke-dasharray="8 12"
    class="flow-path"
  />
  <defs>
    <linearGradient id="flow-gradient" x1="0" y1="0" x2="0" y2="1">
      <stop offset="0%" stop-color="transparent"/>
      <stop offset="50%" stop-color="#00f0ff" stop-opacity="0.8"/>
      <stop offset="100%" stop-color="transparent"/>
    </linearGradient>
  </defs>
</svg>
```

```css
.flow-path {
  stroke-dashoffset: 0;
  animation: flow-down 2s linear infinite;
}
@keyframes flow-down {
  from { stroke-dashoffset: 0; }
  to   { stroke-dashoffset: -40; }
}
```

> Confidence: HIGH — `stroke-dashoffset` animation is well-documented in CSS-Tricks SVG Line Animation guide.

---

### Tailwind v4 Theme Configuration

Define custom design tokens inside `<style type="text/tailwindcss">` in the HTML head. This is the correct v4 CDN approach — no `tailwind.config.js` needed.

```html
<style type="text/tailwindcss">
  @theme {
    --color-void:     #0a0a0f;
    --color-void-2:   #0f0f16;
    --color-cyan:     #00f0ff;
    --color-cyan-dim: rgba(0, 240, 255, 0.25);
    --color-platinum: #e0e0e8;
    --color-platinum-dim: rgba(224, 224, 232, 0.5);

    --font-sans: 'Space Grotesk', system-ui, sans-serif;
    --font-mono: 'JetBrains Mono', 'Courier New', monospace;

    --radius-bento: 24px;
  }
</style>
```

Usage in markup: `bg-void`, `text-cyan`, `text-platinum`, `font-mono`, `rounded-bento`

> Confidence: HIGH — `@theme` directive with `<style type="text/tailwindcss">` confirmed as correct v4 CDN configuration pattern from official Tailwind v4 docs.

---

## Alternatives Considered

| Category | Recommended | Alternative | Why Not |
|----------|-------------|-------------|---------|
| Animation | anime.js 4.x | GSAP 3.14 | GSAP is 71KB min vs anime.js ~17KB; GSAP's plugin system is overkill for spring + count-up; anime.js v4 is a complete rewrite with better ESM/UMD CDN story |
| Animation | anime.js 4.x | CSS-only keyframes | Spring physics require JS; count-up numbers require JS; CSS-only can't sequence chained animations reactively |
| Primary font | Space Grotesk | Inter | Inter is ubiquitous and generic; Space Grotesk shares lineage with Space Mono giving thematic unity with the monospace numerics |
| Primary font | Space Grotesk | Syne / Urbanist | Syne is too editorial; Urbanist lacks the technical character needed for "Industrial Elegance" |
| Mono font | JetBrains Mono | Space Mono | Space Mono is more stylized/retro; JetBrains Mono is cleaner at small sizes — important for measurement values in tight table cells |
| Mono font | JetBrains Mono | Fira Code | Fira Code has ligatures that may confuse electrical values (e.g., `->`, `!=`); JetBrains Mono ligatures are controllable |
| Noise texture | Inline SVG feTurbulence | External PNG texture | External PNG = extra HTTP request; base64 PNG = bloat in HTML; inline SVG filter = zero requests, scales infinitely |
| Noise texture | Inline SVG feTurbulence | CSS `background-image: url("data:...")` SVG string | Both work; the `<defs>` approach is reusable across multiple elements via `filter: url(#id)` |

---

## What NOT to Use

| Technology | Why to Avoid |
|-----------|--------------|
| GSAP ScrollTrigger | App is a form tool — no scroll-based animation needed; added weight without benefit |
| Three.js / WebGL | Massively disproportionate for this use case; would double load time |
| Lottie / rive.app | Requires external JSON animation files; breaks single-file constraint |
| Framer Motion | React-only; incompatible with vanilla JS |
| Motion One (`motion/mini`) | ESM-only, requires module bundler for practical use — single `<script>` tag won't work reliably |
| CSS-only count-up | CSS `@property` counter animations have poor browser support for numeric tweening; JS required |
| backdrop-filter: blur() > 30px | GPU-intensive; on forms with many inputs it causes jank on mid-range devices |
| Paid fonts (Aeonik, GT Alpina) | Explicitly out of scope per PROJECT.md |
| Bootstrap / Material UI | Would conflict with Tailwind v4 utility system |

---

## Full CDN Load Order (Recommended `<head>` Structure)

```html
<!-- 1. Google Fonts — preconnect first, then load -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">

<!-- 2. Tailwind CSS v4 Play CDN (already present) -->
<script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>

<!-- 3. Custom theme tokens — after Tailwind loads -->
<style type="text/tailwindcss">
  @theme {
    /* ... design tokens ... */
  }
</style>

<!-- 4. pdfmake (already present — keep below Tailwind) -->
<script src="https://cdn.jsdelivr.net/npm/pdfmake@0.3.5/build/pdfmake.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/pdfmake@0.3.5/build/vfs_fonts.min.js"></script>

<!-- 5. anime.js v4 UMD — last, before app script -->
<script src="https://cdn.jsdelivr.net/npm/animejs@4.3.6/dist/bundles/anime.umd.min.js"></script>
```

**Load order rationale:**
- Fonts load first (async, doesn't block render, `display=swap` prevents FOIT)
- Tailwind parses DOM classes before content renders
- `@theme` block in `<style type="text/tailwindcss">` is processed by Tailwind, must come after the CDN script
- anime.js last — only needed on interaction, not for initial paint

---

## Sources

- anime.js v4 installation: https://animejs.com/documentation/getting-started/installation/
- anime.js v4 spring easing: https://animejs.com/documentation/easings/spring/
- anime.js releases (v4.3.6 confirmed): https://github.com/juliangarnier/anime/releases
- anime.js CDN UMD path confirmed: https://cdn.jsdelivr.net/npm/animejs@4.3.6/dist/bundles/anime.umd.min.js
- GSAP v3.14.1 CDN files: https://cdn.jsdelivr.net/npm/gsap@3.14.1/dist/
- GSAP installation docs: https://gsap.com/docs/v3/Installation/
- Glassmorphism 2025 techniques: https://playground.halfaccessible.com/blog/glassmorphism-design-trend-implementation-guide
- Dark glassmorphism UI trend: https://medium.com/@developer_89726/dark-glassmorphism-the-aesthetic-that-will-define-ui-in-2026-93aa4153088f
- CSS grain/noise texture (feTurbulence): https://css-tricks.com/grainy-gradients/
- CSS grain pseudo-element approach: https://ibelick.com/blog/create-grainy-backgrounds-with-css
- SVG line animation (stroke-dashoffset): https://css-tricks.com/svg-line-animation-works/
- Neumorphism CSS pressed effect: https://css-tricks.com/neumorphism-and-css/
- Space Grotesk on Google Fonts: https://fonts.google.com/specimen/Space+Grotesk
- JetBrains Mono on Google Fonts: https://fonts.google.com/specimen/JetBrains+Mono
- Google Fonts API v2 multi-font syntax: https://developers.google.com/fonts/docs/css2
- Tailwind v4 @theme configuration: https://tailwindcss.com/docs/theme
- Tailwind v4 CDN setup: https://tailkits.com/blog/tailwind-css-v4-cdn-setup/
