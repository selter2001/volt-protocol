# Pitfalls Research

**Domain:** Luxury dark UI redesign of technical HTML tool (VoltProtocol)
**Researched:** 2026-02-24
**Confidence:** HIGH (verified against MDN, NNGroup, Axess Lab, Tailwind official docs, pdfmake GitHub)

---

## Critical Pitfalls

### Pitfall 1: Glassmorphism Contrast Failure on Dark Background

**What goes wrong:**
Semi-transparent `backdrop-filter: blur()` panels placed over `#0a0a0f` (Void Black) create illegible text. The frosted glass effect blends text into the dark surface at ratios well below the WCAG minimum of 4.5:1 for normal text. Input labels, field values, and error states all become unreadable — especially for Soft Platinum (#e0e0e8) text at lower opacity.

**Why it happens:**
Designers test glassmorphism on light or gradient backgrounds where it was originally designed. On very dark, near-solid backgrounds the "frosted" layer provides almost no contrast improvement — it looks correct in Figma where backgrounds are full-opacity illustrations, but fails on real near-black backgrounds.

**How to avoid:**
- Add a solid semi-opaque tint (10-20% white, e.g. `background: rgba(255,255,255,0.06)`) inside every glass panel — this lifts the effective background from #0a0a0f to roughly #1a1a20, giving enough delta for platinum text
- Measure all text/background combinations with a contrast checker before shipping each component (minimum 4.5:1 for normal text, 3:1 for large/bold)
- Never use Electric Cyan (#00f0ff) as text color on glass — it fails contrast on dark glass at any weight below 24px bold
- Use Soft Platinum (#e0e0e8) at full opacity for all data values, never reduce opacity to create "muted" variants without re-checking ratio

**Warning signs:**
- Squint test: if labels disappear when you squint at the screen, contrast is too low
- Text in input fields looks correct in bright office light but disappears on OLED/dim screens
- Electric Cyan labels on dark glass (this always fails WCAG)

**Phase to address:** Phase 1 — Foundation (color tokens, base components). Every glass panel component must have contrast verified before moving to Phase 2.

---

### Pitfall 2: Animated box-shadow/glow Causing Paint Jank on Form Interaction

**What goes wrong:**
Animating `box-shadow` directly (e.g., focus glow on inputs, glowing result cards, "flow effect" lines) triggers a full repaint on every animation frame. On mid-range laptops and older Windows machines used by Polish electricians in the field, this causes 15-20fps jank. The form feels "laggy" when typing, which destroys the premium feel immediately.

**Why it happens:**
`box-shadow` is not a GPU-composited property. Unlike `transform` and `opacity`, it forces the browser to recalculate paint for every affected element on every frame. When multiple inputs have simultaneous focus glows and the "flow effect" lines pulse, the composite cost stacks.

**How to avoid:**
- Use the pseudo-element opacity technique: place the glow state in `::after` (with full `box-shadow` value), set initial `opacity: 0`, then animate only `opacity` to `1` on focus/hover — opacity is GPU-composited, costs near zero
- Keep `backdrop-filter` blur radius at 12-16px maximum; each additional pixel increases compositing cost quadratically
- Limit simultaneous `backdrop-filter` elements to 5-6 (one per tab section, not per input)
- "Flow effect" animated lines must use `transform: translateX/Y` + `opacity` only — never `left`/`top` or `width` animation
- Add `will-change: transform, opacity` only to actively animated elements, remove after animation completes (overuse creates GPU memory pressure)
- Implement `prefers-reduced-motion` media query: all spring/glow/flow animations reduce to instant state changes when user has motion sensitivity enabled

**Warning signs:**
- Chrome DevTools Performance tab shows paint rectangles flashing across the form area during typing
- FPS drops below 50 when an input has focus glow active
- Scrolling through the form feels sticky while any animation is running

**Phase to address:** Phase 2 — Animations. Must establish the pseudo-element glow pattern as a standard before implementing any animated effect.

---

### Pitfall 3: PDF Export Inheriting Dark Theme Styles

**What goes wrong:**
pdfmake generates PDFs from a JavaScript document definition object — it does not render HTML/CSS. However, the risk is in the JavaScript that builds this definition: if the code reads computed CSS variables (e.g., `getComputedStyle`) or uses inline color values that were refactored to point to `#0a0a0f` or `#00f0ff`, the generated PDF becomes a dark-background PDF. A dark PDF with cyan text is unreadable when printed — which is the primary use case for Polish electricians submitting protocol reports.

**Why it happens:**
During redesign, developers sometimes standardize all color usage to CSS custom properties, then update the pdfmake definition to reference the same tokens for "consistency." The PDF definition must remain completely isolated from the UI theme layer.

**How to avoid:**
- Keep pdfmake document definition in a clearly demarcated code section (e.g., `// === PDF DEFINITION — NEVER IMPORT UI COLORS ===`)
- Hardcode all PDF colors as explicit hex values: black text (#000000), white background (#FFFFFF), gray borders (#999999) — never reference CSS variables or `getComputedStyle`
- After any redesign phase, run a PDF export test immediately and verify: white background, black text, printed landscape layout (Zał. 2)
- The Polish characters test (ą, ę, ó, ź, ż, ć, ń, ł) must pass after every change — pdfmake font encoding is sensitive to changes in the vfs_fonts setup

**Warning signs:**
- PDF preview in browser shows dark background (immediate red flag)
- Polish characters rendering as boxes or question marks in exported PDF
- Landscape orientation (Załącznik 2 — Izolacja) breaking to portrait

**Phase to address:** Phase 1 — Define PDF isolation boundary before touching any JS. Phase 3 — Validation: run full PDF export test suite after completing redesign.

---

### Pitfall 4: Tailwind CSS v4 Play CDN Custom Theme Not Applying

**What goes wrong:**
Custom color tokens defined in a `<style>` block without `type="text/tailwindcss"` are ignored entirely by the Play CDN. The utility classes `text-cyan-electric`, `bg-void-black`, etc. do not generate. The app silently falls back to Tailwind's default palette or renders no styles, which is invisible in development until you check the browser's computed styles.

**Why it happens:**
Tailwind v4 Play CDN only processes `<style type="text/tailwindcss">` blocks. A regular `<style>` block or an inline `<style>` without the type attribute is treated as plain CSS by the browser and bypasses the Tailwind compiler entirely. The `@theme` directive inside a regular style block does nothing.

**How to avoid:**
- Always use `<style type="text/tailwindcss">` for any Tailwind customization in the single HTML file
- Define custom colors using exact v4 convention: `--color-void-black: #0a0a0f;` inside `@theme {}`
- Verify utility classes generated by opening DevTools → Sources → inspecting the virtual CSS file Tailwind CDN generates at runtime
- Never use `@layer base` for color definitions — use `@theme` for design tokens, `@layer utilities` for custom utility classes only

**Warning signs:**
- Custom color utility classes applied in HTML but elements show default Tailwind colors
- DevTools Styles panel shows no custom color variables under `:root`
- `bg-void-black` doesn't appear in computed styles autocomplete

**Phase to address:** Phase 1 — First task before any visual work. Verify theme token system works end-to-end with a single dummy component.

---

### Pitfall 5: Neomorphic Buttons Losing Affordance (Users Can't Tell What's Clickable)

**What goes wrong:**
Neomorphic "pressed-in" buttons rely on subtle box-shadow direction reversal to communicate state. On a very dark background (#0a0a0f), the shadow delta between "raised" and "pressed" states becomes invisible — both states look flat. Users cannot identify interactive elements, hesitate before clicking, and lose confidence in the tool. This is particularly damaging for electricians who use the app under varied lighting (outdoors, in electrical cabinets).

**Why it happens:**
Neomorphism was designed for mid-gray backgrounds (#e0e0e8 range) where light and dark shadows have room to contrast. On near-black, the "dark shadow" is indistinguishable from the background and the "light shadow" creates a thin highlight that is too subtle under ambient light.

**How to avoid:**
- For primary action buttons (PDF Export, Save, Add Row): use a solid Electric Cyan fill with Void Black text instead of pure neomorphic style — clear affordance is non-negotiable for primary actions
- Reserve neomorphic style for secondary/tertiary controls only (tab switches, toggle checkboxes)
- Enforce visible `:hover` state: at minimum `opacity` shift or `transform: scale(1.02)` must be perceptible
- Active/pressed state must use `transform: scale(0.98)` as the primary signal, not just shadow reversal
- Test buttons specifically under F.lux/Night Shift color temperature shift — this is when subtlety disappears

**Warning signs:**
- During review, someone asks "is this button clickable?"
- Hover state is not visible without DevTools inspection
- Users miss the "Eksportuj PDF" button on first use in testing

**Phase to address:** Phase 2 — Buttons and interactive states. Document a clear rule: primary actions use filled style, secondary use neomorphic.

---

### Pitfall 6: Browser Autofill Overriding Dark Input Backgrounds

**What goes wrong:**
Chrome, Firefox, and Safari apply `!important` user-agent styles to autofill inputs with bright yellow or white backgrounds (`-webkit-autofill`). On a dark glassmorphism form, a single autofilled field snaps to a blinding white/yellow box that destroys the premium aesthetic and visually breaks the layout.

**Why it happens:**
Browsers enforce autofill visual indicators as a security/UX pattern and use `!important` styles that cannot be overridden by standard CSS. This is a known browser behavior that designers consistently forget to handle until they test with real form data.

**How to avoid:**
- Use the `-webkit-autofill` pseudo-selector with an inset box-shadow override:
  ```css
  input:-webkit-autofill {
    -webkit-box-shadow: 0 0 0px 1000px #1a1a24 inset !important;
    -webkit-text-fill-color: #e0e0e8 !important;
    transition: background-color 5000s ease-in-out 0s;
  }
  ```
- Set `color-scheme: dark` on `<html>` or `:root` to signal dark preference to the browser, which causes some browsers to apply dark autofill styling by default
- Test with real autofill data (not just typed text) in Chrome, Firefox, and Safari before each phase completion

**Warning signs:**
- Yellow/white flash on input fields when browser suggests saved data
- Autofilled input text becomes invisible (black text on glass-dark background)
- The form "breaks" visually when viewed with saved form data in Chrome

**Phase to address:** Phase 1 — Input component styling. Add autofill overrides as part of the base input style definition.

---

### Pitfall 7: Halation Effect Making Cyan Text Unreadable

**What goes wrong:**
Electric Cyan (#00f0ff) on Void Black (#0a0a0f) produces halation — bright text appears to "bleed" or glow into the background in a way that looks like blur, making the characters hard to read. This is especially problematic for users with astigmatism (estimated 30-40% of the population). Ironically, the "glow" effect that makes the UI look premium actively makes critical numerical values harder to read.

**Why it happens:**
The eye's lens diffracts very bright wavelengths (cyan/blue especially) against very dark backgrounds. This is the same reason white text on pure black is harder to read than on #121212 — the contrast is too extreme. Adding actual CSS glow (`text-shadow`) to cyan text compounds the problem significantly.

**How to avoid:**
- Never use cyan as the primary color for data values or numerical readouts — use Soft Platinum (#e0e0e8) for all data
- Reserve cyan exclusively for: borders, interactive indicators, icons, and small UI accents
- Do not add `text-shadow` glow to any text element carrying data (results, measurements, protocol fields)
- Keep background slightly above absolute black: `#0a0a0f` is acceptable, but `#000000` is not — add `--color-surface-base: #0f0f18` for glass card backgrounds to further reduce halation

**Warning signs:**
- Measurement values (Id, Zsmax, Ia, Rpo) in cyan feel "blurry" even at correct font size
- Text at 14px in Electric Cyan becomes unreadable at arm's length
- Hero Result Card with large cyan text causes eye fatigue after 10 seconds of viewing

**Phase to address:** Phase 1 — Typography and color system. Establish and enforce the rule: cyan is for interaction indicators, platinum is for data content.

---

### Pitfall 8: Dynamic Row Sections Breaking Animation Budget

**What goes wrong:**
VoltProtocol has dynamically added/removed rows in all 4 attachments (SWZ, Izolacja, RCD, Uziemienie). Each row contains multiple input fields. When spring physics count-up animations trigger on calculation results, AND multiple rows each have glow focus states, AND the "flow effect" SVG lines pulse — the browser hits its paint/composite budget simultaneously. On a form with 15-20 rows, this causes dropped frames.

**Why it happens:**
Row-based forms are the hardest case for animation: each new row adds n animated elements, but the animation budget is fixed. Single-page demos of premium UI never test with real data volumes.

**How to avoid:**
- Spring count-up animation: trigger only on the final calculated summary values (Zsmax, Ia, Rpo), not on per-row inputs — at most 4-6 animated values simultaneously
- Implement Intersection Observer for flow-effect lines: only animate lines that are in the current viewport
- Use `requestAnimationFrame` scheduling for count-up: update DOM max 60 times/second, not on every `input` event
- Debounce calculation updates: wait 150ms after last keypress before running calculations and triggering count-up
- Test with maximum realistic data: open all 4 tabs, fill 20 rows each, then trigger a recalculation

**Warning signs:**
- Chrome Performance tab shows "Long Task" blocks during typing in a populated form
- count-up animation runs but skips frames visibly on mid-range hardware
- Adding a new row causes a visible layout shift in the animated result section

**Phase to address:** Phase 2 — Animations. Must establish performance budget (max 60fps on 2019 mid-range laptop) before implementing count-up.

---

### Pitfall 9: Single HTML File Style Scope Bleed

**What goes wrong:**
All CSS lives in one file with no module scoping. Aggressive dark-theme resets (e.g., `* { color: #e0e0e8 }` or `body { background: #0a0a0f }`) accidentally override pdfmake's print styles, dialog overlays, or third-party CDN component styles. A global `input { background: transparent }` rule breaks the browser's native date picker UI, which is already painful on dark backgrounds.

**Why it happens:**
Single HTML file architecture has no CSS encapsulation. Broad selectors written for efficiency have unpredictable cascade effects across the entire document, including elements rendered by pdfmake's download trigger, Word export blobs, and file input dialogs.

**How to avoid:**
- Use Tailwind utility classes directly on elements instead of broad CSS selectors wherever possible
- Any global CSS resets go inside a wrapper class (`.app-shell` or `#app`) — never `* {}` or `body {}`
- Use `:where()` for lower-specificity resets that can be overridden by utilities
- Keep a "global reset exclusion list": date inputs, file inputs, and `<select>` elements need explicit dark styling rather than inheriting from broad resets
- Review cascade order: Tailwind utilities must appear after any custom CSS rules in the `<style>` block

**Warning signs:**
- pdfmake download button or export dialog has wrong colors/text
- Native `<input type="date">` calendar picker shows white text on white background
- `<select>` dropdown shows dark text on dark background (inherited from global reset)

**Phase to address:** Phase 1 — CSS architecture. Establish scoping convention before writing any dark-theme styles.

---

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Animate `box-shadow` directly | Simpler code, fewer elements | Paint jank on all animations; degrades premium feel | Never — use pseudo-element opacity instead |
| Global color reset `* { color: ... }` | Fast dark theme base | Breaks date pickers, selects, pdfmake overlays | Never in single-file app |
| Use `opacity: 0.7` for "muted" text | Easy hierarchy | Fails WCAG contrast on dark backgrounds | Never for data values; only decorative elements |
| Hardcode all animations inline (no `prefers-reduced-motion`) | Simpler implementation | Accessibility failure; motion sickness risk | Never — add after every animation |
| Add `backdrop-filter` to every card | Full glassmorphism aesthetic | GPU memory exhaustion on mobile; 20+ blur layers | Limit to max 6 simultaneously visible |
| Use cyan (#00f0ff) for data readouts | Visually striking results | Halation; WCAG contrast failure; eye fatigue | Never for data; only for UI indicators |

---

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| pdfmake 0.3.5 | Reading CSS computed variables for PDF color values | Hardcode all PDF colors as explicit hex; isolate PDF definition from UI theme completely |
| pdfmake 0.3.5 | Forgetting Polish character test after font/style changes | After every redesign phase, export PDF and verify: ą, ę, ó, ź, ż, ć, ń, ł — all must render |
| Tailwind v4 Play CDN | `<style>` block without `type="text/tailwindcss"` | Always use `<style type="text/tailwindcss">` for `@theme` definitions |
| localStorage | Assuming serialized data survives a redesign that changes field IDs | Keep all field `id` attributes identical to current; never rename during visual redesign |
| Browser autofill | CSS not targeting `-webkit-autofill` pseudo-selector | Add explicit autofill overrides for dark backgrounds as part of base input styles |
| Word export (.docx) | Inheriting HTML color styles in Blob generation | Verify Word export uses its own isolated style definitions, not CSS variables |

---

## Performance Traps

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| Direct `box-shadow` animation on 20+ inputs | Typing lag; janky focus transitions | Pseudo-element `::after` + `opacity` animation | Any device with integrated GPU |
| Multiple simultaneous `backdrop-filter` | Scroll stutter; tab-switch freeze | Limit to <=6 glass panels visible simultaneously | 5+ panels on mid-range mobile |
| `requestAnimationFrame` loop for count-up without debounce | Every keystroke triggers animation; form feels sticky | Debounce 150ms, cancel in-flight animations before new ones start | Forms with 20+ rows |
| Flow-effect SVG lines animating off-screen | Wasted GPU cycles even on hidden tabs | Intersection Observer to pause off-viewport animations | Tab 0 active, tabs 1-4 still animating |
| `will-change: transform` on all elements permanently | GPU memory exhaustion; browser crashes on memory-constrained devices | Apply `will-change` only during animation, remove after | Devices with <4GB RAM |

---

## UX Pitfalls

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| No visible difference between POZYTYWNA/NEGATYWNA verdict in dark theme | Electrician misreads verdict; submits incorrect protocol | Green (#00ff88) for POZYTYWNA, Red (#ff4444) for NEGATYWNA — must be high-contrast even on dark; add icon |
| Tabs with only subtle border-bottom difference on dark theme | Users lose track of active tab; navigate to wrong attachment | Active tab: solid Electric Cyan indicator + slight background lift; not just text color change |
| Glassmorphism input borders too thin (0.5px as per brief) | 0.5px renders as 0px on non-retina Windows displays | Use 1px on `@media (max-resolution: 1dppx)`, 0.5px only on hi-DPI |
| Premium animation plays on every form interaction | Tool feels slow for power users doing repetitive data entry | Animations on result calculation only; form input/tab switch: instant (no animation) |
| Dark result card with grain-gradient too visually dominant | Distraction from the verdict value itself | Hero card animation plays once on calculation, then settles — not looping |

---

## "Looks Done But Isn't" Checklist

- [ ] **Glassmorphism panels:** Contrast ratio verified with actual hex values, not Figma preview — check every text/background pair
- [ ] **PDF export:** Generated PDF has white background and black text — check after every JS change
- [ ] **Polish characters in PDF:** ą, ę, ó, ź, ż, ć, ń, ł all render correctly — check after any font or vfs change
- [ ] **Autofill override:** Form tested with Chrome's saved autofill data — not just manually typed text
- [ ] **prefers-reduced-motion:** All animations have a `@media (prefers-reduced-motion: reduce)` fallback
- [ ] **Tab persistence:** Switching tabs preserves all entered data — localStorage/state not reset on tab change
- [ ] **Dynamic rows:** "Dodaj wiersz" / "Usuń wiersz" still works and recalculates correctly after redesign
- [ ] **Word export:** .docx download produces readable output with correct field values
- [ ] **JSON import:** Previously exported JSON loads correctly into the redesigned form (field IDs unchanged)
- [ ] **Verdict legibility:** POZYTYWNA (green) vs NEGATYWNA (red) clearly distinguishable on dark background
- [ ] **Form at 20 rows:** Animation budget tested with maximum realistic data volume
- [ ] **Windows Chrome non-retina:** 0.5px borders tested on 1x display (may render as 0px)

---

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| PDF exports with dark background | LOW | Restore PDF definition to pre-redesign state; hardcode explicit colors; never reference CSS variables |
| Contrast failures discovered late | MEDIUM | Audit all text/background pairs with automated tool (axe-core or Contrast Finder); adjust surface colors system-wide |
| Animation jank across all inputs | MEDIUM | Replace all `box-shadow` transitions with pseudo-element `opacity` pattern; 1-2 day refactor |
| Autofill breaking dark theme | LOW | Add 5-line `-webkit-autofill` override block to base input styles |
| localStorage data loss after field ID rename | HIGH | If field IDs changed: write migration function to map old keys to new; test against real user exports |
| `prefers-reduced-motion` omitted everywhere | MEDIUM | Add `@media (prefers-reduced-motion: reduce)` wrapper to every animation definition; systematic but tedious |

---

## Pitfall-to-Phase Mapping

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| Glassmorphism contrast failure | Phase 1 — Color tokens and base components | Contrast checker on every surface/text pair before Phase 2 |
| PDF style isolation | Phase 1 — JS architecture boundary | Export PDF after Phase 1 completes; verify white bg + black text |
| Tailwind v4 CDN `@theme` setup | Phase 1 — First implementation task | Check DevTools that custom color variables appear in `:root` |
| Browser autofill override | Phase 1 — Input component | Test autofill in Chrome with saved form data |
| CSS scope bleed (global resets) | Phase 1 — CSS architecture | Test date inputs, selects, pdfmake download after any global style change |
| Neomorphic button affordance | Phase 2 — Interactive components | User test: ask someone unfamiliar with the app to "click Save" without guidance |
| Halation on cyan text | Phase 1 — Typography system | View results section under dim ambient light conditions |
| Animated box-shadow paint cost | Phase 2 — Animation implementation | Chrome Performance tab: verify <4ms paint during focus glow animation |
| Dynamic rows animation budget | Phase 2 — Animation + Phase 3 stress test | Fill 20 rows per tab, trigger recalculation, verify 60fps in DevTools |
| Window autofill CSS override | Phase 1 — Base input styles | Fill form with browser-saved data; verify no white/yellow flash |

---

## Sources

- [Glassmorphism Meets Accessibility — Axess Lab](https://axesslab.com/glassmorphism-meets-accessibility-can-frosted-glass-be-inclusive/) — HIGH confidence
- [Glassmorphism: Definition and Best Practices — NNGroup](https://www.nngroup.com/articles/glassmorphism/) — HIGH confidence
- [How to Animate CSS Box Shadows — Tobias Ahlin](https://tobiasahlin.com/blog/how-to-animate-box-shadow/) — HIGH confidence
- [CSS and JavaScript animation performance — MDN](https://developer.mozilla.org/en-US/docs/Web/Performance/Guides/CSS_JavaScript_animation_performance) — HIGH confidence
- [CSS backdrop-filter — Can I Use](https://caniuse.com/css-backdrop-filter) — HIGH confidence
- [Play CDN — Tailwind CSS Official Docs](https://tailwindcss.com/docs/installation/play-cdn) — HIGH confidence
- [Tailwind v4 dark mode issues — GitHub Discussion #16517](https://github.com/tailwindlabs/tailwindcss/discussions/16517) — HIGH confidence
- [Upgrading to Tailwind v4 dark mode — GitHub Issue #16516](https://github.com/tailwindlabs/tailwindcss/issues/16516) — HIGH confidence
- [Dark Mode Accessibility Guide — Smashing Magazine](https://www.smashingmagazine.com/2025/04/inclusive-dark-mode-designing-accessible-dark-themes/) — MEDIUM confidence
- [Legacy App UI Redesign Mistakes — XB Software](https://xbsoftware.com/blog/legacy-app-ui-redesign-mistakes/) — MEDIUM confidence
- [pdfmake styling — Official docs](https://pdfmake.github.io/docs/0.1/document-definition-object/styling/) — HIGH confidence
- [Neumorphism in UI — Interaction Design Foundation](https://www.interaction-design.org/literature/topics/neumorphism) — MEDIUM confidence
- [CSS autofill override — CSS-Tricks](https://css-tricks.com/snippets/css/change-autocomplete-styles-webkit-browsers/) — MEDIUM confidence

---

*Pitfalls research for: Luxury dark UI redesign — VoltProtocol (single-file HTML, Tailwind v4 CDN, pdfmake)*
*Researched: 2026-02-24*
