---
phase: 01-visual-foundation
plan: 01
type: execute
wave: 1
depends_on: []
files_modified:
  - index.html
autonomous: true
requirements:
  - VIS-01
  - VIS-02
  - VIS-03
  - VIS-04
  - VIS-05
  - VIS-06
  - ANIM-07

must_haves:
  truths:
    - "Otwarcie index.html w przegladarce pokazuje Void Black (#0a0a0f) tlo zamiast jasnego szarego"
    - "Na ciemnym tle widoczna jest subtlna noise texture (grain) nie blokujaca interakcji z UI"
    - "W tle widac 2-3 rozmyte kolorowe orby (cyan, platinum) tworzace ambient glow"
    - "Tekst jest wyswietlany fontem Space Grotesk zamiast systemowego sans-serif"
    - "DevTools :root zawiera wszystkie custom properties z @theme (--color-void, --color-cyan, --color-platinum itd.)"
    - "Przy wlaczonym prefers-reduced-motion animacje/tranzycje sa wylaczone"
    - "Wszystkie istniejace funkcje (formularze, obliczenia, eksport PDF, localStorage) dzialaja identycznie jak przed zmiana"
  artifacts:
    - path: "index.html"
      provides: "Style block z @theme tokenami, @layer base, noise overlay, ambient orbs + body class change"
      contains: "@theme"
  key_links:
    - from: "style[type=text/tailwindcss] @theme"
      to: "Tailwind CSS v4 Play CDN"
      via: "CDN automatycznie przetwarza @theme w style block z type=text/tailwindcss"
      pattern: "type=\"text/tailwindcss\""
    - from: "body class bg-void"
      to: "@theme --color-void"
      via: "Tailwind utility bg-void mapuje na --color-void z @theme"
      pattern: "bg-void"
    - from: "@import url Google Fonts"
      to: "@theme --font-sans"
      via: "Google Fonts CDN laduje Space Grotesk i JetBrains Mono, @theme rejestruje je jako font-sans i font-mono"
      pattern: "font-sans.*Space Grotesk"
---

<objective>
Wstawienie bloku design tokenow (Tailwind @theme), zmiana body na Void Black, dodanie noise texture overlay, ambient gradient orbs i prefers-reduced-motion — tworzac wizualny fundament premium "Industrial Elegance" bez dotykania jakiegokolwiek kodu JS.

Purpose: Ten plan buduje baze wizualna (ciemne tlo, kolory, fonty, tekstury), na ktorej Plan 01-02 zbuduje glassmorphism komponenty i nawigacje.
Output: Zmodyfikowany index.html z pelnym systemem tokenow i ciemnym premium wyglada body.
</objective>

<execution_context>
@./.claude/get-shit-done/workflows/execute-plan.md
@./.claude/get-shit-done/templates/summary.md
</execution_context>

<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@.planning/STATE.md
@.planning/phase-1/RESEARCH.md
@index.html (linie 1-30 — HEAD + body tag + top bar)
</context>

<tasks>

<task type="auto">
  <name>Task 1: Google Fonts import + Tailwind @theme token system + @layer base body styling + noise texture + ambient orbs + prefers-reduced-motion</name>
  <files>index.html</files>
  <action>
W pliku index.html wykonaj dokladnie 2 zmiany:

**ZMIANA 1 — Wstaw blok style MIEDZY linia 9 (Tailwind CDN script) a linia 11 (komentarz pdfmake).**

Wstaw nastepujacy blok:

```html
<!-- Design System: Luxury Industrial Elegance -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<style type="text/tailwindcss">
  @import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500;700&display=swap');

  @theme {
    /* Kolory bazowe */
    --color-void: #0a0a0f;
    --color-void-2: #0f0f16;
    --color-cyan: #00f0ff;
    --color-cyan-dim: rgba(0, 240, 255, 0.25);
    --color-platinum: #e0e0e8;
    --color-platinum-dim: rgba(224, 224, 232, 0.5);

    /* Kolory semantyczne */
    --color-positive: #22c55e;
    --color-negative: #ef4444;

    /* Glass system */
    --color-glass-bg: rgba(255, 255, 255, 0.04);
    --color-glass-border: rgba(224, 224, 232, 0.12);
    --color-glass-highlight: rgba(0, 240, 255, 0.06);

    /* Surface levels */
    --color-surface-1: rgba(255, 255, 255, 0.06);
    --color-surface-2: rgba(255, 255, 255, 0.10);

    /* Typografia */
    --font-sans: 'Space Grotesk', system-ui, sans-serif;
    --font-mono: 'JetBrains Mono', 'Courier New', monospace;

    /* Zaokraglenia */
    --radius-bento: 24px;
    --radius-input: 8px;

    /* Cienie */
    --shadow-glow-cyan: 0 0 20px rgba(0, 240, 255, 0.15), 0 0 40px rgba(0, 240, 255, 0.05);
    --shadow-neomorph-rest: 4px 4px 12px rgba(0, 0, 0, 0.4), -2px -2px 8px rgba(255, 255, 255, 0.03);
    --shadow-neomorph-pressed: inset 3px 3px 8px rgba(0, 0, 0, 0.5), inset -1px -1px 4px rgba(255, 255, 255, 0.02);

    /* Tranzycje */
    --transition-base: 200ms ease;
  }

  @layer base {
    :root {
      color-scheme: dark;
    }

    body {
      background-color: var(--color-void);
      color: var(--color-platinum);
      font-family: var(--font-sans);
    }

    /* Autofill override dla ciemnego tla */
    input:-webkit-autofill,
    input:-webkit-autofill:hover,
    input:-webkit-autofill:focus {
      -webkit-box-shadow: 0 0 0 1000px #0f0f16 inset !important;
      -webkit-text-fill-color: #e0e0e8 !important;
    }
  }

  /* Ambient gradient orbs */
  #app::before {
    content: '';
    position: fixed;
    top: -20%;
    left: -10%;
    width: 600px;
    height: 600px;
    background: radial-gradient(circle, rgba(0, 240, 255, 0.08) 0%, transparent 70%);
    filter: blur(80px);
    pointer-events: none;
    z-index: -1;
  }

  #app::after {
    content: '';
    position: fixed;
    bottom: -20%;
    right: -10%;
    width: 500px;
    height: 500px;
    background: radial-gradient(circle, rgba(224, 224, 232, 0.05) 0%, transparent 70%);
    filter: blur(80px);
    pointer-events: none;
    z-index: -1;
  }

  /* prefers-reduced-motion — ANIM-07 */
  @media (prefers-reduced-motion: reduce) {
    *,
    *::before,
    *::after {
      animation-duration: 0.01ms !important;
      animation-iteration-count: 1 !important;
      transition-duration: 0.01ms !important;
    }
  }
</style>
```

Wazne szczegoly:
- Atrybut `type="text/tailwindcss"` jest OBOWIAZKOWY — bez niego Tailwind v4 CDN nie przetworzy @theme.
- `@import url(...)` dla fontow MUSI byc wewnatrz bloku `<style>`, nie jako osobny `<link>` tag (Google Fonts @import wewnatrz style block jest poprawny i dziala).
- Link preconnect PRZED style blockiem przyspiesza ladowanie fontow.
- Nie modyfikuj zadnych istniejacych skryptow (Tailwind CDN, pdfmake).

**ZMIANA 2 — Zmien body tag (linia 15).**

Zamien:
```html
<body class="bg-gray-50 text-gray-900">
```
na:
```html
<body class="bg-void text-platinum font-sans">
```

**ZMIANA 3 — Dodaj noise texture overlay jako pierwsze dziecko body (przed #app).**

Wstaw zaraz po otwierajacym tagu `<body>`:
```html
<!-- Noise texture overlay -->
<div class="fixed inset-0 pointer-events-none z-0 opacity-[0.03]" style="background-image: url(&quot;data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)' opacity='1'/%3E%3C/svg%3E&quot;); background-repeat: repeat; background-size: 128px 128px;"></div>
```

Wazne:
- `pointer-events: none` — UI pod spodem musi byc klikalny.
- `z-0` — pod elementami UI (ktore beda miec z-index wyzszy lub auto).
- `opacity-[0.03]` — subtlny efekt grain, nie dominujacy.

KRYTYCZNE OGRANICZENIA:
1. NIE dotykaj zadnych funkcji JS — ten plan to TYLKO CSS + 1 zmiana klasy body + 1 nowy div HTML.
2. NIE modyfikuj skryptow pdfmake ani zadnego kodu po tagu `</head>` (poza body class i noise div).
3. NIE usuwaj istniejacych komentarzy w head.
  </action>
  <verify>
    <automated>
# Sprawdz ze @theme zostal wstawiony
grep -c '@theme' index.html | grep -q '^1$' && echo "PASS: @theme block present" || echo "FAIL: @theme block missing"

# Sprawdz ze body ma nowe klasy
grep -q 'class="bg-void text-platinum font-sans"' index.html && echo "PASS: body classes updated" || echo "FAIL: body classes not updated"

# Sprawdz noise texture div
grep -q 'pointer-events-none z-0 opacity-\[0.03\]' index.html && echo "PASS: noise overlay present" || echo "FAIL: noise overlay missing"

# Sprawdz type=text/tailwindcss
grep -q 'type="text/tailwindcss"' index.html && echo "PASS: style type correct" || echo "FAIL: style type missing"

# Sprawdz Google Fonts import
grep -q 'Space+Grotesk' index.html && echo "PASS: Space Grotesk imported" || echo "FAIL: Space Grotesk missing"
grep -q 'JetBrains+Mono' index.html && echo "PASS: JetBrains Mono imported" || echo "FAIL: JetBrains Mono missing"

# Sprawdz ambient orbs
grep -q '#app::before' index.html && echo "PASS: ambient orb 1 present" || echo "FAIL: ambient orb 1 missing"
grep -q '#app::after' index.html && echo "PASS: ambient orb 2 present" || echo "FAIL: ambient orb 2 missing"

# Sprawdz prefers-reduced-motion
grep -q 'prefers-reduced-motion' index.html && echo "PASS: reduced motion present" || echo "FAIL: reduced motion missing"

# Sprawdz ze pdfmake nie zostal uszkodzony
grep -q 'pdfmake@0.3.5' index.html && echo "PASS: pdfmake intact" || echo "FAIL: pdfmake damaged"

# Sprawdz ze JS nie zostal zmodyfikowany (liczba linii JS nie zmienila sie drastycznie)
grep -c 'function ' index.html
    </automated>
    <manual>Otworz index.html w przegladarce:
1. Tlo powinno byc ciemne (prawie czarne) — nie jasne szare
2. Tekst powinien byc jasny platinum — nie ciemny
3. Na tle widoczna subtlna tekstura grain
4. W tle widac rozmyte kolorowe orby (subtelne, cyan i platinum)
5. Font tytulu powinien byc Space Grotesk (geometryczny sans-serif)
6. DevTools -> Elements -> sprawdz :root ma custom properties (--color-void itd.)
7. Wpisz dane w formularz — pola dzialaja normalnie
8. Kliknij Zapisz/Wczytaj — localStorage dziala
9. Kliknij Eksportuj PDF — PDF generuje sie poprawnie z polskimi znakami na jasnym tle</manual>
  </verify>
  <done>
1. index.html zawiera blok style type="text/tailwindcss" z pelnym systemem @theme tokenow (kolory, fonty, promienie, cienie, tranzycje)
2. Body tag ma klasy bg-void text-platinum font-sans
3. Noise texture overlay div jest pierwszym dzieckiem body z pointer-events-none i z-0
4. Ambient gradient orbs renderuja sie przez #app::before i #app::after
5. prefers-reduced-motion media query wylacza animacje i tranzycje
6. Google Fonts (Space Grotesk + JetBrains Mono) sa importowane i zarejestrowane w @theme
7. Zaden kod JS nie zostal zmodyfikowany — wszystkie funkcje (formularz, obliczenia, PDF, Word, JSON, localStorage) dzialaja identycznie
  </done>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <name>Task 2: Weryfikacja wizualna i funkcjonalna</name>
  <what-built>Design token system, Void Black tlo z noise texture, ambient gradient orbs, dual-font system (Space Grotesk + JetBrains Mono), prefers-reduced-motion. Caly wizualny fundament premium redesignu.</what-built>
  <how-to-verify>
1. Otworz index.html w przegladarce (Chrome/Edge)
2. Sprawdz wizualnie:
   - Tlo jest ciemne (#0a0a0f Void Black), NIE jasne szare
   - Tekst jest jasny platinum (#e0e0e8), czytelny na ciemnym tle
   - Na tle widoczna subtlna tekstura grain (zoom 200% zeby zobaczyc wyraznie)
   - W rogu lewym-gornym i prawym-dolnym widac rozmyte kolorowe swiatelka (ambient orbs)
   - Font naglowka to Space Grotesk (geometryczny, premium feel)
3. Sprawdz funkcjonalnosc:
   - Wpisz dane w formularzu — pola akceptuja tekst i liczby
   - Przelacz zakladki (Protokol, Zal. 1-4) — przelaczanie dziala
   - Kliknij "Zapisz" i "Wczytaj" — localStorage dziala
   - Kliknij "Eksportuj PDF" — PDF generuje sie (jasne tlo, polskie znaki)
   - Kliknij "Eksport JSON" / "Import JSON" — dziala
4. DevTools -> Elements -> sprawdz na :root:
   - Widoczne custom properties: --color-void, --color-cyan, --color-platinum itd.
   - Font-family na body to 'Space Grotesk'
5. (Opcjonalnie) W systemie wlacz "Reduce motion" i sprawdz, ze animacje sa wylaczone
  </how-to-verify>
  <resume-signal>Wpisz "approved" jesli wszystko wyglada dobrze, lub opisz problemy do poprawienia.</resume-signal>
</task>

</tasks>

<verification>
Weryfikacja calosciowa planu 01-01:
1. `grep '@theme' index.html` zwraca dokladnie 1 wynik
2. `grep 'bg-void text-platinum' index.html` potwierdza nowe klasy body
3. `grep 'pointer-events-none' index.html` potwierdza noise overlay
4. `grep 'prefers-reduced-motion' index.html` potwierdza ANIM-07
5. `grep 'Space+Grotesk' index.html` i `grep 'JetBrains+Mono' index.html` potwierdzaja fonty
6. Otwarcie w przegladarce pokazuje ciemne tlo z noise i orbami
7. Formularze, zakladki, eksporty dzialaja identycznie jak przed zmianami
</verification>

<success_criteria>
- Aplikacja renderuje sie na Void Black tle z subtlna noise texture i ambient orbami
- DevTools pokazuje pelny system custom properties z @theme
- Space Grotesk jest widoczny jako font bazowy, JetBrains Mono zarejestrowany do uzycia
- prefers-reduced-motion wylacza wszelkie animacje/tranzycje
- Zadna funkcja JS nie zostala naruszona — PDF, Word, JSON, localStorage, obliczenia dzialaja jak wczesniej
</success_criteria>

<output>
After completion, create `.planning/phase-1/01-01/01-01-SUMMARY.md`
</output>
