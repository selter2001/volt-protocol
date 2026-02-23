# Stack Research

**Domain:** Single-file HTML SPA — electrical measurement protocol generator (PDF export)
**Researched:** 2026-02-23
**Confidence:** HIGH (core stack), MEDIUM (PDF library recommendation)

---

## Recommended Stack

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| HTML5 | — | Single-file app shell | Zero config, zero deploy friction, GitHub Pages serves one file — no bundler, no build step |
| Tailwind CSS (CDN) | v4.x (browser build) | Utility-first styling | Only CSS framework with a working browser-script CDN in 2025/2026; no PostCSS needed |
| Vanilla JavaScript | ES2022+ | App logic, dynamic forms, calculations | No framework overhead; `<template>` cloning + event delegation covers all dynamic-row requirements |
| pdfmake | 0.3.5 | PDF export with searchable text, proper tables | Declarative document definition, real vector text (not screenshot), automatic pagination, table headers repeat across pages — critical for protocol tables spanning multiple pages |

### Supporting Libraries

| Library | Version | CDN URL | Purpose | When to Use |
|---------|---------|---------|---------|-------------|
| pdfmake | 0.3.5 | `https://cdn.jsdelivr.net/npm/pdfmake@0.3.5/build/pdfmake.min.js` | PDF engine | Always — core PDF generation |
| pdfmake vfs_fonts | 0.3.5 | `https://cdn.jsdelivr.net/npm/pdfmake@0.3.5/build/vfs_fonts.min.js` | Embedded Roboto font | Always — load after pdfmake.min.js |
| Tailwind Browser CDN | 4.x | `https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4` | CSS utilities | Always — single `<script>` in `<head>` |

### Development Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| GitHub Pages | Static hosting | Push to `main`, enable Pages — single HTML file served directly |
| Browser DevTools | Debug, print preview | Use `Ctrl+Shift+P` → "Show print preview" to check layout before PDF export |
| VS Code + Live Server | Local dev | Open the single HTML file; no build step |

---

## PDF Library Comparison (Critical Decision)

This is the most important tech decision in the project. Four candidates evaluated:

### Quick Decision Matrix

| Criterion | pdfmake 0.3.x | jsPDF 4.x + AutoTable | html2pdf.js 0.14.x | window.print() |
|-----------|---------------|----------------------|-------------------|----------------|
| Text selectable in PDF | YES | YES (native) | NO (image-based) | YES |
| Table support | Excellent, built-in | Plugin required | Limited | Print CSS required |
| Page headers repeat | YES, automatic | YES via AutoTable | NO | Via CSS `thead` |
| CDN-only (no bundler) | YES | YES | YES | N/A |
| File size (CDN) | ~2.5 MB (incl. fonts) | ~350 KB + plugin | ~1.5 MB (wraps html2canvas) | 0 |
| Declarative API | YES — JSON DDO | NO — imperative | NO — DOM-based | NO |
| Maintenance (2026) | Active (0.3.5 Feb 2026) | Active (4.2.0 Feb 2025) | Slow (last release 2024) | Browser built-in |
| Polish characters (UTF-8) | YES | YES | YES | YES |
| Multi-page auto-pagination | YES, automatic | YES | Limited | Browser handles |
| Confidence | MEDIUM-HIGH | MEDIUM | LOW | MEDIUM |

### pdfmake — RECOMMENDED

**Why:** pdfmake uses a declarative "Document Definition Object" (DDO) — you describe what the document contains, not where to place it in pixels. For VoltProtokol, which has multi-section measurement tables that will span many pages, pdfmake's automatic pagination with repeating table headers is essential. You never manually calculate Y coordinates. Tables, colspans, borders, fill colors — all built in with no extra plugin.

**Key capabilities for this project:**
- `headerRows: 1` — header row repeats on every new page automatically
- Column widths: `'*'` (flex), `'auto'`, or fixed pixel values
- Table cell `fillColor`, borders via `layout` (`noBorders`, `lightHorizontalLines`, custom)
- `colSpan` and `rowSpan` for merged cells (protocol headers)
- Snaking columns (added 0.3.5) for complex layouts
- Vertical cell alignment (added 0.3.4) — needed for measurement tables

**CDN setup (load order matters):**
```html
<script src="https://cdn.jsdelivr.net/npm/pdfmake@0.3.5/build/pdfmake.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/pdfmake@0.3.5/build/vfs_fonts.min.js"></script>
```

**Limitation:** API is not HTML-based — you build the document programmatically from JavaScript data, not by cloning the DOM. This is actually an advantage for VoltProtokol: the PDF layout is defined explicitly and won't break when screen styles change.

### jsPDF 4.x + AutoTable 5.x — Alternative

**Why not primary:** Requires two CDN scripts (jsPDF + AutoTable plugin). API is imperative — manual coordinate management for non-table content. AutoTable covers table needs well, but anything outside a table requires manual `doc.text(x, y, ...)` calls with pixel coordinates. Managing a full-page protocol with headers, footers, and multiple table sections becomes error-prone.

**When to use instead:** If the PDF output needs to closely mirror the screen HTML (copy-paste DOM into PDF). jsPDF has a `.html()` method that renders HTML to PDF via html2canvas, but this produces image-based output — not suitable.

**CDN:**
```html
<script src="https://cdn.jsdelivr.net/npm/jspdf@4.2.0/dist/jspdf.umd.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/jspdf-autotable@5.0.7/dist/jspdf.plugin.autotable.min.js"></script>
```

**Note on AutoTable v5 breaking change:** In non-browser environments, the plugin no longer auto-applies to jsPDF. In-browser CDN usage is unaffected — `doc.autoTable({...})` still works.

### html2pdf.js — NOT RECOMMENDED

**Why not:** html2pdf.js wraps html2canvas (DOM screenshot) + jsPDF. The PDF output is a **rasterized image** — text is not selectable, not searchable, and the file is large. Protocol documents must have selectable text (electricians copy data, inspectors search PDFs). Additionally, maintenance is slow — last meaningful release was 2024, CDN version is behind npm. Do not use.

### window.print() — Viable for print layout only, not PDF download

**Why not as primary:** `window.print()` opens the browser print dialog — the user must choose "Save as PDF" manually, pick the filename, configure margins. There is no way to trigger a direct `.pdf` file download via `window.print()`. The PROJECT.md requirement is an "Eksportuj PDF" button that downloads a file — `window.print()` cannot satisfy this.

**When to use alongside pdfmake:** Add `@media print` CSS so the app also prints well from the browser. This is a secondary enhancement, not a replacement for pdfmake.

---

## Tailwind CSS CDN — Version Decision

### v4 Play CDN — RECOMMENDED for this project

The v4 Play CDN (`@tailwindcss/browser@4`) is the appropriate choice **given the single-file constraint**. Key facts:

- Script: `https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4`
- Supports `<style type="text/tailwindcss">` for `@apply` and `@theme` custom tokens
- Runs JIT engine in the browser — scans the document for class names at runtime
- Tailwind officially labels it "development only / not for production" due to runtime overhead
- For VoltProtokol: this warning is irrelevant — the app has no build step by design, and the runtime overhead is a one-time parse on load (not perceptible for an electrical protocol tool)

**Browser support (v4):** Safari 16.4+, Chrome 111+, Firefox 128+. Target users are Polish electricians — they use modern browser versions, this is not a concern.

**Custom configuration in single-file:**
```html
<script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
<style type="text/tailwindcss">
  @theme {
    --color-positive: #16a34a;
    --color-negative: #dc2626;
  }
  @layer utilities {
    .print-hide { @apply print:hidden; }
  }
</style>
```

### v3 CDN — Alternative

`https://cdn.tailwindcss.com` — v3 CDN is also JIT-in-browser, similar limitations. Use v3 only if targeting browsers older than Chrome 111 (no evidence this is needed for SEP electricians). For new projects in 2026, v4 is the standard.

---

## Vanilla JS Patterns for This Project

### Dynamic Table Rows

**Pattern: `<template>` element + event delegation**

Use `<template>` tags in HTML to define row blueprints. Clone and insert. Attach a single event listener to the parent `<tbody>` (not to each row) to handle add/remove/change events.

```html
<template id="row-tmpl">
  <tr>
    <td><input type="text" name="circuit-name"></td>
    <td><input type="number" name="zs-measured"></td>
    <td class="result-cell">—</td>
    <td><button class="remove-row">Usuń</button></td>
  </tr>
</template>
```

```javascript
// Clone row
const tmpl = document.getElementById('row-tmpl');
const row = tmpl.content.cloneNode(true);
tbody.appendChild(row);

// Event delegation for the whole section
tbody.addEventListener('input', (e) => {
  if (e.target.matches('[name="zs-measured"]')) recalculate(e.target.closest('tr'));
});
tbody.addEventListener('click', (e) => {
  if (e.target.matches('.remove-row')) e.target.closest('tr').remove();
});
```

**Why this pattern:** No jQuery, no framework. `closest()` walks up the DOM for the row context. Works for dynamically added rows without re-binding listeners.

### localStorage Save/Load

**Pattern: Serialize entire form state to JSON**

```javascript
function saveState() {
  const state = collectFormData(); // walk form, build object
  localStorage.setItem('voltprotokol-v1', JSON.stringify(state));
}
function loadState() {
  const raw = localStorage.getItem('voltprotokol-v1');
  if (!raw) return;
  applyFormData(JSON.parse(raw));
}
```

Manual save/load (button-triggered) as required by PROJECT.md. Do not use auto-save on every `input` event — risks corrupting a partially-filled form if the user closes midway.

### Automatic Calculations

**Pattern: Computed fields on `input` event**

```javascript
document.addEventListener('input', (e) => {
  if (e.target.matches('.zs-measured, .ia-field')) {
    const row = e.target.closest('tr');
    const zs = parseFloat(row.querySelector('.zs-measured').value) || 0;
    const ia = parseFloat(row.querySelector('.ia-field').value) || 0;
    const zsMax = ia > 0 ? (230 / ia).toFixed(3) : '—';
    row.querySelector('.zsmax-result').textContent = zsMax;
    row.querySelector('.result-cell').textContent = zs <= zsMax ? 'POZYTYWNA' : 'NEGATYWNA';
  }
});
```

Use `parseFloat` + `|| 0` guard against empty inputs. Display `—` not `0` for empty fields.

---

## Installation (CDN — No npm Required)

This project uses no npm. All dependencies are loaded via `<script>` tags in the HTML `<head>`:

```html
<!-- Tailwind CSS v4 Play CDN -->
<script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>

<!-- pdfmake + Roboto fonts (load order: pdfmake first, then vfs_fonts) -->
<script src="https://cdn.jsdelivr.net/npm/pdfmake@0.3.5/build/pdfmake.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/pdfmake@0.3.5/build/vfs_fonts.min.js"></script>
```

**Total external JS weight:** ~2.8 MB (Tailwind browser engine ~300 KB + pdfmake ~2.5 MB including fonts). Loaded once; cached by browser on subsequent visits.

---

## Alternatives Considered

| Recommended | Alternative | Why Not |
|-------------|-------------|---------|
| pdfmake | jsPDF + AutoTable | Requires two scripts; coordinate-based API harder to maintain for multi-section document; table headers repeat automatically in pdfmake without plugin |
| pdfmake | html2pdf.js | Image-based output — text not selectable; library maintenance slow since 2024 |
| pdfmake | window.print() | Cannot trigger .pdf file download programmatically; user must manually navigate print dialog |
| pdfmake | Puppeteer / headless Chrome | Requires server — app must be 100% client-side per constraints |
| pdfmake | React-PDF | Requires bundler and React — out of scope for single-file no-bundler constraint |
| Tailwind v4 CDN | Tailwind v3 CDN | v4 is current standard; v3 only needed for older browsers (not relevant for SEP target users) |
| Tailwind v4 CDN | Bootstrap CSS CDN | Tailwind's utility model works better with dynamic content; already decided in PROJECT.md |
| Vanilla JS | Alpine.js | Adds another CDN dependency; event delegation + template cloning covers all requirements without a framework |
| `<template>` cloning | innerHTML string building | `<template>` is safer (no XSS from string interpolation), cleaner separation of markup from logic |

---

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| html2pdf.js | Image-based PDF — text not selectable, not searchable; blurry output on high-DPI; file sizes 5-10x larger than vector PDF | pdfmake |
| jsPDF `.html()` method | Internally uses html2canvas — same problems as html2pdf.js: rasterized image, not text | pdfmake declarative DDO |
| React / Vue / Svelte | Requires bundler (Vite/Webpack); violates single-file constraint; adds significant complexity for a form-heavy tool | Vanilla JS with `<template>` pattern |
| Bootstrap CSS CDN | Class-name conflicts with Tailwind; Bootstrap adds ~200 KB of CSS with many unused styles | Tailwind v4 CDN |
| Auto-save to localStorage on every keystroke | Risk of persisting partial/invalid state; localStorage size limit (5 MB) can be hit with many dynamic rows | Manual save/load buttons |
| pdfmake with custom fonts (beyond Roboto) | Custom font embedding requires base64 encoding and custom vfs build — not doable in single-file CDN setup without a build step | Use included Roboto font; it supports Polish characters (ą, ę, ó, etc.) |
| npm / bundlers (Vite, Webpack, Rollup) | Violates the explicit single-file, no-bundler constraint in PROJECT.md | CDN script tags |

---

## Stack Patterns by Variant

**If the PDF needs custom fonts (e.g., company logo font):**
- Use base64-encoded font embedded directly in a `<script>` block as a vfs override
- Example: `pdfMake.fonts = { CustomFont: { normal: 'data:font/truetype;base64,...' } }`
- This is complex; only do if strictly required — Roboto is professional and readable

**If print-from-browser must also work well:**
- Add `@media print` CSS inside `<style type="text/tailwindcss">` to hide UI controls and set page margins
- Use `page-break-inside: avoid` on `<tr>` elements to prevent mid-row page breaks
- This is a complementary feature, not a replacement for pdfmake export

**If localStorage is insufficient for large protocols:**
- Add a JSON export/import feature: `Blob` + `URL.createObjectURL()` for download; `FileReader` for load
- No server needed; works in single-file CDN setup

---

## Version Compatibility

| Package | Version | Compatible With | Notes |
|---------|---------|-----------------|-------|
| pdfmake | 0.3.5 | All modern browsers (Chrome 60+, Firefox 60+, Safari 12+) | vfs_fonts.min.js must match pdfmake version exactly |
| pdfmake | 0.3.5 | Polish UTF-8 characters | Roboto includes full Latin Extended (ą ę ó ś ź ż ć ń ł) |
| @tailwindcss/browser | @4 | Chrome 111+, Firefox 128+, Safari 16.4+ | v4 requires modern CSS (cascade layers, color-mix) |
| jsPDF (if used) | 4.2.0 | jspdf-autotable 5.0.7 | AutoTable 5.x changed API for non-browser (Node); browser CDN usage unchanged |

---

## Sources

- jsDelivr package page for pdfmake — version 0.3.5 confirmed (Feb 22, 2026): https://www.jsdelivr.com/package/npm/pdfmake
- jsDelivr package page for jsPDF — version 4.2.0 confirmed (Feb 19, 2025): https://www.jsdelivr.com/package/npm/jspdf
- jsDelivr package page for jspdf-autotable — version 5.0.7 confirmed: https://www.jsdelivr.com/package/npm/jspdf-autotable
- Tailwind CSS v4 Play CDN official docs — CDN script and limitations: https://tailwindcss.com/docs/installation/play-cdn
- Tailwind CSS v4.0 release blog: https://tailwindcss.com/blog/tailwindcss-v4
- pdfmake GitHub releases — active maintenance verified Feb 2026: https://github.com/bpampuch/pdfmake/releases
- jsPDF GitHub releases — v4.0 breaking changes (FS access only, no API breaks): https://github.com/parallax/jsPDF/releases
- jsPDF AutoTable v5 breaking change (non-browser only): https://github.com/simonbengtsson/jsPDF-AutoTable/releases
- PDF library comparison (html2pdf.js image output confirmed): https://dmitriiboikov.com/posts/2025/01/pdf-generation-comarison/
- pdfmake table documentation: https://pdfmake.github.io/docs/0.1/document-definition-object/tables/
- npm trends — download counts (jsPDF 4.5M/wk, pdfmake 1.1M/wk, html2pdf.js 578K/wk): https://npmtrends.com/html-pdf-vs-html2pdf.js-vs-jspdf-vs-pdf-vs-pdfkit-vs-pdfmake
- html2pdf.js maintenance issues (CDN not updated): https://github.com/eKoopmans/html2pdf.js/issues/893

---

*Stack research for: VoltProtokół (SEP-Calc) — single-file HTML electrical protocol generator*
*Researched: 2026-02-23*
