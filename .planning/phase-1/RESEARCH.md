# Phase 1: Visual Foundation - Research

**Researched:** 2026-02-24
**Domain:** Analiza kodu index.html -- mapowanie punktow zmiany CSS dla luxury redesignu
**Confidence:** HIGH

## Summary

Plik `index.html` to single-file SPA (2922 linii) z Tailwind CSS v4 Play CDN. Caly UI jest renderowany dynamicznie przez JS -- HTML body zawiera jedynie szkielet (top bar, tabs, 5 kontenerow, footer), a cala zawartosc jest wstrzykiwana przez `innerHTML` w funkcjach render. Oznacza to, ze redesign wymaga zmian zarowno w statycznym HTML jak i w ~15 funkcjach JS ktore generuja stringi z klasami Tailwind.

Kluczowe odkrycie: JS uzywa 6 klas CSS jako selektorow querySelector (`.id-cell`, `.zsmax-cell`, `.verdict-cell`, `.izol-verdict-cell`, `.rcd-verdict-cell`, `.uziem-rpo-cell`, `.uziem-verdict-cell`) -- te klasy MUSZA zostac zachowane na elementach po redesignie. Dodatkowo JS manipuluje klasami `text-green-700`, `text-red-700`, `text-gray-400` i `hidden` przez classList.add/remove -- te rowniez musza byc zachowane lub zamapowane na nowe odpowiedniki.

**Primary recommendation:** Wszystkie zmiany wizualne realizowac przez: (1) Tailwind `@theme` block w `<style type="text/tailwindcss">` w head, (2) modyfikacje class stringow w JS render functions, (3) zachowanie wszystkich klas uywanych jako selektory JS.

---

## 1. HEAD Section -- Obecny stan i punkt wstawienia

### Linie 1-14: Obecna zawartosc `<head>`

```html
<!-- Linia 3-14 -->
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>VoltProtokol - Protokol kontrolno-pomiarowy</title>

  <!-- Tailwind CSS v4 Play CDN -->
  <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>

  <!-- pdfmake -->
  <script src="https://cdn.jsdelivr.net/npm/pdfmake@0.3.5/build/pdfmake.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/pdfmake@0.3.5/build/vfs_fonts.min.js"></script>
</head>
```

### CDN scripts (NIE RUSZAC)

| Script | Wersja | Linia | Cel |
|--------|--------|-------|-----|
| `@tailwindcss/browser@4` | v4 (latest) | 9 | Tailwind CSS v4 Play CDN |
| `pdfmake` | 0.3.5 | 12 | PDF export |
| `vfs_fonts` | 0.3.5 | 13 | Fonty dla pdfmake |

### Punkt wstawienia nowego bloku `<style>`

Nowy blok `<style type="text/tailwindcss">` powinien byc wstawiony **MIEDZY linia 9 (Tailwind CDN) a linia 11 (pdfmake)**. Tailwind v4 Play CDN automatycznie przetworzy `@theme` i `@layer` w tym bloku.

```html
<script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>

<!-- === TUTAJ WSTAWIC === -->
<style type="text/tailwindcss">
  @import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500;700&display=swap');

  @theme {
    --color-void-black: #0a0a0f;
    --color-electric-cyan: #00f0ff;
    --color-soft-platinum: #e0e0e8;
    --color-glass-bg: rgba(255, 255, 255, 0.04);
    --color-glass-border: rgba(224, 224, 232, 0.12);
    --color-glass-highlight: rgba(0, 240, 255, 0.06);
    --font-family-display: 'Space Grotesk', sans-serif;
    --font-family-mono: 'JetBrains Mono', monospace;
    --radius-card: 1.5rem;
    /* ... pelny system tokenow */
  }

  /* Body styling, noise texture SVG, ambient orbs */
</style>
<!-- === KONIEC WSTAWIENIA === -->

<script src="https://cdn.jsdelivr.net/npm/pdfmake@0.3.5/build/pdfmake.min.js"></script>
```

### Fonty Google do dodania

W bloku `<style type="text/tailwindcss">` na gorze dodac `@import`:
- **Space Grotesk** -- labele, naglowki (VIS-04)
- **JetBrains Mono** -- wartosci numeryczne (VIS-05)

---

## 2. Body Structure -- Statyczny HTML (linie 15-51)

### Linia 15: `<body>` tag

```html
<body class="bg-gray-50 text-gray-900">
```

**ZMIANA:** `bg-gray-50` -> `bg-void-black` + noise texture SVG inline, `text-gray-900` -> `text-soft-platinum`

### Linia 17: Glowny kontener `#app`

```html
<div id="app" class="max-w-7xl mx-auto p-4">
```

**ZACHOWAC layout:** `max-w-7xl mx-auto p-4`
**Opcjonalnie:** dodac `font-display` (Space Grotesk) jako bazowy font

### Linie 18-29: Top Bar (tytul + przyciski)

```html
<!-- Linia 18 -->
<div class="flex items-center justify-between mb-6 flex-wrap gap-2">
  <!-- Linia 19 -->
  <h1 class="text-2xl font-bold">Protokol kontrolno-pomiarowy PN-HD 60364-6</h1>
  <!-- Linia 20 -->
  <div class="flex gap-2 flex-wrap">
    <!-- 7 przyciskow, linie 21-27 -->
  </div>
</div>
```

**Layout do ZACHOWANIA:** `flex items-center justify-between mb-6 flex-wrap gap-2`, `flex gap-2 flex-wrap`
**Appearance do ZMIANY:**

| Element | Linia | Obecne klasy appearance | Zmiana na |
|---------|-------|------------------------|-----------|
| `<h1>` | 19 | `text-2xl font-bold` (kolor dziedziczy z body) | `text-soft-platinum font-display` |
| btn-save | 21 | `bg-green-600 text-white ... hover:bg-green-700` | glass button style |
| btn-load | 22 | `bg-green-600 text-white ... hover:bg-green-700` | glass button style |
| btn-export-json | 23 | `bg-gray-600 text-white ... hover:bg-gray-700` | glass button style |
| btn-import-json | 24 | `bg-gray-600 text-white ... hover:bg-gray-700` | glass button style |
| btn-export-word | 25 | `bg-purple-600 text-white ... hover:bg-purple-700` | glass button style |
| btn-import-word | 26 | `bg-purple-600 text-white ... hover:bg-purple-700` | glass button style |
| btn-export-pdf | 27 | `bg-blue-600 text-white ... hover:bg-blue-700` | glass button style |

Wspoolny wzorzec przyciskow: `bg-{color}-600 text-white px-3 py-2 rounded hover:bg-{color}-700 text-sm font-medium`
Nowy styl: glassmorphism buttons z Soft Platinum border, `backdrop-blur`, hover glow

### Linie 32-38: Zakladki `#tabs`

```html
<!-- Linia 32 -->
<div id="tabs" class="flex border-b border-gray-300 mb-4">
  <button data-tab="0" class="px-4 py-2 font-medium border-b-2 border-blue-600 text-blue-600">Protokol</button>
  <button data-tab="1" class="px-4 py-2 font-medium border-b-2 border-transparent text-gray-500 hover:text-gray-700">Zal. 1</button>
  <button data-tab="2" class="...">Zal. 2</button>
  <button data-tab="3" class="...">Zal. 3</button>
  <button data-tab="4" class="...">Zal. 4</button>
</div>
```

**Layout do ZACHOWANIA:** `flex`, `mb-4`, `px-4 py-2`
**Appearance do ZMIANY:**

| Element | Obecne | Nowe |
|---------|--------|------|
| Kontener `#tabs` | `border-b border-gray-300` | `border-b border-glass-border` (lub custom dark) |
| Tab aktywna (static HTML) | `border-b-2 border-blue-600 text-blue-600` | `border-b-2 border-electric-cyan text-electric-cyan` |
| Tab nieaktywna (static HTML) | `border-b-2 border-transparent text-gray-500 hover:text-gray-700` | `border-b-2 border-transparent text-soft-platinum/50 hover:text-soft-platinum` |

**UWAGA KRYTYCZNA:** Klasy zakladek sa ROWNIEZ ustawiane dynamicznie w `renderTabs()` (linia 384, 386) -- patrz sekcja 3.

### Linie 41-45: Kontenery zalacznikow

```html
<div id="attachment-0"></div>
<div id="attachment-1" class="hidden"></div>
<div id="attachment-2" class="hidden"></div>
<div id="attachment-3" class="hidden"></div>
<div id="attachment-4" class="hidden"></div>
```

**ZACHOWAC:** `hidden` -- JS dodaje/usuwa te klase (linia 393-395)
**BEZ ZMIAN:** Kontenery nie maja stylowania -- zawartosc jest generowana przez JS

### Linie 48-50: Footer

```html
<div class="mt-8 pt-4 border-t border-gray-200 text-center text-xs text-gray-400">
  VoltProtocol &copy; 2026 ...
  <a href="..." class="underline hover:text-gray-600">GitHub</a>
</div>
```

**Layout do ZACHOWANIA:** `mt-8 pt-4 text-center text-xs`
**Appearance do ZMIANY:**

| Element | Obecne | Nowe |
|---------|--------|------|
| Footer div | `border-t border-gray-200 text-gray-400` | `border-t border-glass-border text-soft-platinum/40` |
| Footer link | `underline hover:text-gray-600` | `underline hover:text-electric-cyan` |

---

## 3. JS Render Functions -- Class String Constants

### 3.1 `renderTabs()` (linia 379-398)

**Dynamiczne klasy (`.className =` na liniach 384, 386):**

```javascript
// Linia 384 -- aktywna zakladka
btn.className = 'px-4 py-2 font-medium border-b-2 border-blue-600 text-blue-600';

// Linia 386 -- nieaktywna zakladka
btn.className = 'px-4 py-2 font-medium border-b-2 border-transparent text-gray-500 hover:text-gray-700';
```

**ZMIANA:** Zamien `border-blue-600 text-blue-600` na `border-electric-cyan text-electric-cyan`, `text-gray-500 hover:text-gray-700` na `text-soft-platinum/50 hover:text-soft-platinum`.

**classList operacje (linie 393-395):**
```javascript
container.classList.remove('hidden');  // pokaz aktywny tab
container.classList.add('hidden');     // ukryj nieaktywny tab
```
**ZACHOWAC:** `hidden` musi dzialac -- jest to Tailwind utility

### 3.2 `renderFormTab()` (linia 519-726)

**Stale klasowe (linia 522-523):**

```javascript
const inputCls = 'w-full border border-gray-300 rounded px-2 py-1.5 text-sm';
const labelCls = 'block text-sm font-medium text-gray-700 mb-1';
```

**ZMIANA inputCls:** `border-gray-300 rounded` -> glassmorphism input (glass bg, Soft Platinum border, rounded-lg, focus:ring-electric-cyan)
**ZMIANA labelCls:** `text-gray-700` -> `text-soft-platinum/80 font-display`

**Card wrappers (powtarzajace sie na liniach 527, 536, 555, 574, 592, 626, 692):**

```javascript
'<div class="bg-white border rounded p-4 mb-4">'     // Linie 527, 536, 555, 574, 592
'<div id="assessment-section" class="bg-white border rounded p-4 mb-4">'  // Linia 626
'<div id="orzeczenie-section" class="bg-white border rounded p-4 mb-4">'  // Linia 692
```

**ZMIANA:** `bg-white border rounded` -> glassmorphism card (`bg-glass-bg backdrop-blur-xl border border-glass-border rounded-3xl`)

**Naglowki sekcji (powtarzajace sie):**

```javascript
'<h3 class="text-md font-semibold mb-3 text-gray-800">...'  // Linie 528, 537, 556, 575, 593, 627, 693
```

**ZMIANA:** `text-gray-800` -> `text-soft-platinum font-display`

**Tabele w formularzu:**

```javascript
'<thead class="bg-gray-50">'                                    // Linie 596, 630
'<th class="border border-gray-300 px-3 py-2 text-left font-medium">'  // Linie 598-601, 632-634
'<td class="border border-gray-300 px-3 py-2 ...">'           // Wielokrotnie
```

**ZMIANA:** `bg-gray-50` -> `bg-glass-highlight`, `border-gray-300` -> `border-glass-border`

**Instrumenty -- dodatkowe inline inputy (linie 609-616):**

```javascript
'class="w-full border border-gray-300 rounded px-2 py-1 text-sm"'
```

**ZMIANA:** jak inputCls powyzej

**Ocena assessment -- dynamiczne klasy (linie 654, 662, 670, 678):**

```javascript
const swzClass = assessment.swz === 'POZYTYWNY' ? 'text-green-700' : ... ? 'text-red-700' : 'text-gray-400';
```

**ZACHOWAC:** `text-green-700`, `text-red-700`, `text-gray-400` -- uzywane tez przez JS classList (patrz sekcja 4)

**Orzeczenie -- klasy (linie 696-703):**

```javascript
'<p id="orzeczenie-text" class="text-base font-bold text-green-700 mb-2">'   // NADAJE SIE
'<p id="orzeczenie-text" class="text-base font-bold text-red-700 mb-2">'     // NIE NADAJE
'<p id="orzeczenie-text" class="text-base text-gray-400 mb-2">'              // brak danych
'<p id="next-test-date" class="text-sm text-gray-700">'                      // data nastepnego
'<p id="next-test-date" class="text-sm text-gray-400">'                      // brak daty
```

**Podpisy (linie 715-721):**

```javascript
'<div class="border-b border-gray-400 w-48 mb-1"></div>'
'<div class="text-xs text-gray-500">wykonawca</div>'
```

**ZMIANA:** `border-gray-400` -> `border-soft-platinum/40`, `text-gray-500` -> `text-soft-platinum/50`

### 3.3 `renderSWZRowCells()` (linia 427-517) -- Attachment 1 komorki

**Stale klasowe (linia 434-435):**

```javascript
const inputCls = 'w-full border border-gray-300 rounded px-1 py-0.5 text-sm text-center';
const selectCls = 'w-full border border-gray-300 rounded px-1 py-0.5 text-sm';
```

**Komorki tabeli (powtarzajace sie):**

```javascript
'<td class="border border-gray-300 px-2 py-1 text-center font-medium">'      // Lp
'<td class="border border-gray-300 px-2 py-1 font-medium">'                   // Fixed row text
'<td class="border border-gray-300 px-1 py-0.5">'                             // Input cells
'<td class="border border-gray-300 px-2 py-1 text-right text-sm font-mono id-cell" ...>'  // Computed
'<td class="border border-gray-300 px-2 py-1 text-center font-bold verdict-cell ...">'    // Verdict
'<td class="border border-gray-300 px-2 py-1 text-center text-gray-400">'     // Fixed row action
'<td class="border border-gray-300 px-1 py-1 text-center">'                   // Delete btn cell
```

**Przyciski usuwania:**

```javascript
'class="text-red-500 hover:text-red-700 text-xs"'  // Linia 512
```

### 3.4 `renderAttachment1()` (linia 728-839)

**Naglowek i przycisk (linie 732-733):**

```javascript
'<h2 class="text-lg font-semibold mb-3">'
'<button ... class="mb-3 px-3 py-1 bg-blue-600 text-white rounded hover:bg-blue-700 text-sm">'
```

**Tabela (linia 736-778):**

```javascript
'<table class="w-full border-collapse border border-gray-300 text-sm">'
'<thead class="bg-gray-50 text-xs text-center">'
'<th ... class="border border-gray-300 px-2 py-1 ...">'   // Wielokrotnie
```

**Staly wiersz (linia 784):**

```javascript
'<tr class="bg-yellow-50" data-row-id="fixed">'
```

**ZMIANA:** `bg-yellow-50` -> dark glass accent (np. `bg-electric-cyan/5`)

**Wiersze sekcji/podsekcji (linie 791-811):**

```javascript
'<tr class="bg-gray-200">'                                    // Section header
'<tr class="bg-gray-100">'                                    // Subsection header
// Inputy w sekcjach:
'class="bg-transparent font-bold w-64 outline-none focus:bg-white focus:ring-1 focus:ring-blue-400 rounded px-1"'
// Przyciski sekcji:
'class="ml-3 px-2 py-0.5 bg-blue-500 text-white rounded text-xs hover:bg-blue-600"'    // Dodaj podtytul
'class="px-2 py-0.5 bg-red-500 text-white rounded text-xs hover:bg-red-600"'            // Usun sekcje
'class="ml-3 px-2 py-0.5 bg-green-500 text-white rounded text-xs hover:bg-green-600"'   // Dodaj wiersz
'class="px-2 py-0.5 bg-red-400 text-white rounded text-xs hover:bg-red-500"'            // Usun podtytul
```

**Legenda (linia 828):**

```javascript
'<div class="mt-4 text-xs text-gray-600 grid grid-cols-2 gap-x-6 gap-y-1">'
```

### 3.5 `renderIZOLRowCells()` (linia 841-910) -- Attachment 2 komorki

**Stala klasowa (linia 849):**

```javascript
const inputCls = 'w-full border border-gray-300 rounded px-1 py-0.5 text-sm text-center';
```

**Specjalny input Rp z szerokosia 14 (linia 889):**

```javascript
'class="w-14 border border-gray-300 rounded px-0.5 py-0.5 text-sm text-center font-mono' + disabledCls + '"'
// disabledCls = ' bg-gray-100 text-gray-400 cursor-not-allowed' (gdy pole nieaktywne)
```

**Verdykt komorka:**

```javascript
'<td class="border border-gray-300 px-2 py-1 text-center font-bold izol-verdict-cell ...">'
```

### 3.6 `renderAttachment2()` (linia 912-999)

**Analogiczna struktura do Attachment 1. Dodatkowe:**

```javascript
'<table class="min-w-max border-collapse text-sm">'           // min-w-max zamiast w-full (szersza tabela)
'<span class="text-sm text-gray-500">3-faz</span>'            // Linia 873 -- fixed row indicator
```

### 3.7 `renderAttachment3()` (linia 1002-1106) -- RCD

**Stale klasowe (linie 1004-1005):**

```javascript
const inputCls = 'w-full border border-gray-300 rounded px-1 py-0.5 text-sm text-center';
const selectCls = 'w-full border border-gray-300 rounded px-1 py-0.5 text-sm';
```

**Tabela:**

```javascript
'<tr class="bg-gray-50 text-center font-semibold">'           // Header
```

**Pusty stan (linia 1088):**

```javascript
'<td colspan="9" class="border border-gray-300 px-2 py-4 text-center text-gray-400 italic">'
```

**Verdict komorka:**

```javascript
'<td class="border border-gray-300 px-2 py-1 text-center font-bold rcd-verdict-cell ...">'
```

**Przycisk dodawania (linia 1095):**

```javascript
'<button data-action="add-row-rcd" class="mt-2 px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 text-sm">'
```

### 3.8 `renderAttachment4()` (linia 1108-1200) -- Uziemienie

**Stala klasowa (linia 1110):**

```javascript
const inputCls = 'w-full border border-gray-300 rounded px-1 py-0.5 text-sm text-center';
```

**Computed cell z wlasna klasa:**

```javascript
'<td class="border border-gray-300 px-2 py-1 text-right font-mono uziem-rpo-cell" ...>'  // Rpo
'<td class="border border-gray-300 px-2 py-1 text-center font-bold uziem-verdict-cell ...">'  // Verdict
```

**Pusty stan (linia 1182):**

```javascript
'<td colspan="8" class="border border-gray-300 px-2 py-4 text-center text-gray-400 italic">'
```

### 3.9 `updateOrzeczenieDisplay()` (linia 1796-1825)

**Dynamiczne `.className =` (pelne nadpisanie):**

```javascript
// Linia 1805 -- NADAJE SIE
orzText.className = 'text-base font-bold text-green-700 mb-2';
// Linia 1811 -- NIE NADAJE SIE
orzText.className = 'text-base font-bold text-red-700 mb-2';
// Linia 1814 -- brak danych
orzText.className = 'text-base text-gray-400 mb-2';
// Linia 1820 -- data nastepnego (jest)
nextDateEl.className = 'text-sm text-gray-700';
// Linia 1823 -- data nastepnego (brak)
nextDateEl.className = 'text-sm text-gray-400';
```

### 3.10 `showToast()` (linia 1831-1837)

```javascript
toast.className = 'fixed bottom-4 right-4 bg-gray-800 text-white px-4 py-2 rounded shadow-lg text-sm z-50';
```

**ZMIANA:** `bg-gray-800` -> glass toast (`bg-glass-bg backdrop-blur-xl border border-glass-border`)

---

## 4. JS Selektory -- MUSZA BYC ZACHOWANE

### 4.1 Selektory po ID (getElementById)

| ID | Linia | Funkcja | Kontekst |
|----|-------|---------|----------|
| `tabs` | 380, 1265 | renderTabs, bindTabEvents | Kontener zakladek |
| `attachment-0` | 520 | renderFormTab | Formularz |
| `attachment-1` | 729 | renderAttachment1 | SWZ |
| `attachment-2` | 913 | renderAttachment2 | Izolacja |
| `attachment-3` | 1003 | renderAttachment3 | RCD |
| `attachment-4` | 1109 | renderAttachment4 | Uziemienie |
| `app` | 1278 | bindAppEvents | Delegacja eventow |
| `assess-swz` | 1787, 1789 | updateFinalAssessmentDisplay | Ocena SWZ |
| `assess-izol` | 1790 | updateFinalAssessmentDisplay | Ocena izolacji |
| `assess-rcd` | 1791 | updateFinalAssessmentDisplay | Ocena RCD |
| `assess-uziem` | 1792 | updateFinalAssessmentDisplay | Ocena uziemienia |
| `orzeczenie-text` | 1797 | updateOrzeczenieDisplay | Tekst orzeczenia |
| `next-test-date` | 1798 | updateOrzeczenieDisplay | Data nastepnego |
| `assessment-section` | -- | Tylko w HTML (id na div) | Sekcja oceny |
| `orzeczenie-section` | -- | Tylko w HTML (id na div) | Sekcja orzeczenia |
| `btn-export-pdf` | 2909 | init | Przycisk PDF |
| `btn-save` | 2910 | init | Przycisk Zapisz |
| `btn-load` | 2911 | init | Przycisk Wczytaj |
| `btn-export-json` | 2912 | init | Przycisk JSON export |
| `btn-import-json` | 2913 | init | Przycisk JSON import |
| `btn-export-word` | 2914 | init | Przycisk Word export |
| `btn-import-word` | 2915 | init | Przycisk Word import |

### 4.2 Selektory po klasie CSS (querySelector) -- KRYTYCZNE

| Klasa CSS | Linia | Funkcja | Co robi |
|-----------|-------|---------|---------|
| `.id-cell` | 1569 | updateSWZComputedCells | Aktualizacja Id |
| `.zsmax-cell` | 1570 | updateSWZComputedCells | Aktualizacja Zsmax |
| `.verdict-cell` | 1571 | updateSWZComputedCells | Aktualizacja verdyktu SWZ |
| `.izol-verdict-cell` | 1656 | updateIZOLVerdict | Aktualizacja verdyktu izolacji |
| `.rcd-verdict-cell` | 1708 | updateRCDVerdict | Aktualizacja verdyktu RCD |
| `.uziem-rpo-cell` | 1755 | updateUZIEMComputedCells | Aktualizacja Rpo |
| `.uziem-verdict-cell` | 1761 | updateUZIEMComputedCells | Aktualizacja verdyktu uziemienia |

**REGULA:** Kazdy element z tymi klasami MUSI je zachowac po redesignie. Mozna DODAC klasy, ale NIE USUWAC tych.

### 4.3 Selektory po data-* atrybutach (querySelector)

```javascript
// Linia 1538-1540 -- restore focus po re-renderze
'[data-field="' + activeField + '"][data-row-id="' + activeRowId + '"]'

// Linia 1549 -- update Ia placeholder
'[data-field="Ia"][data-row-id="' + rowId + '"]'
```

**ZACHOWAC:** Atrybuty `data-field`, `data-row-id`, `data-sub-id`, `data-action`, `data-tab`, `data-attachment`, `data-section-id` -- caly system delegacji eventow na nich polega.

### 4.4 Klasy manipulowane przez classList.add/remove

| Klasa | Linie | Operacja |
|-------|-------|----------|
| `hidden` | 393, 395 | add/remove na kontenerach zakladek |
| `text-green-700` | 1582, 1660, 1712, 1765, 1778 | add na verdyktach POZYTYWNA/POZYTYWNY |
| `text-red-700` | 1583, 1661, 1713, 1766, 1780 | add na verdyktach NEGATYWNA/NEGATYWNY |
| `text-gray-400` | 1776, 1782 | add na brak danych (assessment cells) |

**REGULA:** Klasy `text-green-700`, `text-red-700`, `text-gray-400` sa dodawane/usuwane dynamicznie przez JS. Mozna:
- (a) Zachowac je i upewnic sie ze Tailwind je generuje (bezpiecznie)
- (b) Zamienic na custom klasy (np. `text-verdict-positive`) i zaktualizowac WSZYSTKIE miejsca w JS -- bardziej ryzykowne ale spjniejsze z designem

**REKOMENDACJA:** Opcja (a) -- zachowac `text-green-700`, `text-red-700`, `text-gray-400` bez zmian. Sa uzywane w ~20 miejscach w JS. Zamiana wymaga modyfikacji logiki JS w wielu funkcjach co zwieksza ryzyko regresji. Kolory green-700 i red-700 i tak pasuja do dark theme.

---

## 5. Appearance Classes -- DO ZAMIANY

### 5.1 Wspolne wzorce do zamiany globalnie

| Wzorzec | Wystapienia | Kontekst | Zamiana |
|---------|-------------|----------|--------|
| `bg-white` | 7x | Card wrappers w renderFormTab | `bg-glass-bg backdrop-blur-xl` |
| `bg-gray-50` | 6x | body, thead, header rows | `bg-void-black` (body), `bg-glass-highlight` (thead) |
| `bg-gray-100` | 3x | Subsection rows, disabled inputs | `bg-white/5` (rows), `bg-white/3` (disabled) |
| `bg-gray-200` | 2x | Section header rows | `bg-white/8` |
| `bg-yellow-50` | 2x | Fixed rows (SWZ, IZOL) | `bg-electric-cyan/5` |
| `border-gray-300` | ~80x | Wszystkie komorki tabel, inputy, selecty | `border-glass-border` |
| `border-gray-200` | 1x | Footer border-t | `border-glass-border` |
| `border-gray-400` | 2x | Podpisy linie | `border-soft-platinum/30` |
| `text-gray-900` | 1x | body | `text-soft-platinum` |
| `text-gray-800` | 7x | Naglowki h3 | `text-soft-platinum` |
| `text-gray-700` | 4x | Labele, texty | `text-soft-platinum/80` |
| `text-gray-600` | 3x | Legendy | `text-soft-platinum/60` |
| `text-gray-500` | 4x | Zakladki, helpy | `text-soft-platinum/50` |
| `text-gray-400` | 8x | Placeholdery, stale akcje, brak danych | `text-soft-platinum/30` (ZACHOWAC klase bo JS uzywa!) |
| `bg-blue-600` | 6x | Przyciski akcji, active tab border | `bg-electric-cyan/20` lub glass |
| `bg-green-600` | 2x | Przyciski save/load | glass green accent |
| `bg-gray-600` | 2x | Przyciski JSON | glass |
| `bg-purple-600` | 2x | Przyciski Word | glass purple accent |
| `bg-gray-800` | 1x | Toast | `bg-glass-bg backdrop-blur-xl` |
| `border-blue-600` | 1x | Active tab border (static HTML) | `border-electric-cyan` |
| `text-blue-600` | 1x | Active tab text (static HTML) | `text-electric-cyan` |
| `rounded` | ~30x | Inputy, selecty, przyciski | `rounded-lg` (inputy) lub `rounded-3xl` (karty) |
| `shadow-lg` | 1x | Toast | `shadow-[0_0_30px_rgba(0,240,255,0.1)]` |

### 5.2 Tabele -- Pattern do wyodrebnienia

Wspolny wzorzec komorki tabeli powtarza sie ~100 razy:
```
border border-gray-300 px-2 py-1
```

Rekomendacja: Uzyj `@layer components` w bloku `<style>`:
```css
@layer components {
  .cell { @apply border border-glass-border px-2 py-1; }
}
```
To pozwoli zamienic `border border-gray-300 px-2 py-1` na `cell` w calym kodzie, redukujac powtorzenia.

---

## 6. Layout Classes -- DO ZACHOWANIA (NIE ZMIENIAC)

### Uzywane w statycznym HTML

| Klasa(y) | Element | Cel |
|----------|---------|-----|
| `max-w-7xl mx-auto p-4` | #app | Centrowanie i padding |
| `flex items-center justify-between` | Top bar | Rozmieszczenie tytulu i przyciskow |
| `flex gap-2 flex-wrap` | Kontener przyciskow | Flow layout |
| `flex border-b ... mb-4` | #tabs | Zakladki w linii |
| `px-4 py-2` | Zakladki | Padding zakladek |
| `mb-6` | Top bar | Margin bottom |
| `mt-8 pt-4 text-center text-xs` | Footer | Spacing i wyrownanie |

### Uzywane w JS render functions

| Klasa(y) | Funkcja | Cel |
|----------|---------|-----|
| `p-4 mb-4` | renderFormTab cards | Card padding i margin |
| `grid grid-cols-1 md:grid-cols-3 gap-4` | renderFormTab | 3-kolumnowy grid |
| `grid grid-cols-1 md:grid-cols-2 gap-4` | renderFormTab (warunki) | 2-kolumnowy grid |
| `max-w-xs` | renderFormTab (numer) | Max szerokosc |
| `max-w-lg` | renderFormTab (warunki) | Max szerokosc |
| `overflow-x-auto` | Wszystkie tabele | Scroll horizontalny |
| `w-full border-collapse text-sm` | Tabele | Szerokosc i tekst |
| `min-w-max` | Tabela attachment-2 | Minimalna szerokosc |
| `min-w-[120px]`, `w-10`, `w-14`, `w-16`, `w-20`, `w-24`, `w-36`, `w-44`, `w-48` | Kolumny tabel | Szerokosc kolumn |
| `mt-4`, `mt-2`, `mt-8`, `mb-1`, `mb-3`, `ml-3` | Rne | Spacing |
| `flex items-center gap-2` | Temperatura input | Inline layout |
| `flex gap-16` | Podpisy | Spacing |
| `grid grid-cols-2 gap-x-6 gap-y-1` | Legendy | 2-kolumnowy grid |
| `colspan`, `rowspan` | Tabele HTML | Struktura tabeli |
| `px-1 py-0.5`, `px-2 py-1`, `px-3 py-2`, `px-0.5 py-0.5` | Komorki/inputy | Padding |

---

## 7. Kompletna mapa zmian -- Podsumowanie wg plan 01-01 i 01-02

### Plan 01-01: Design tokens + body styling

| Co | Gdzie (linia) | Zmiana |
|----|---------------|--------|
| `<style type="text/tailwindcss">` | Nowy blok po linii 9 | @import fonts, @theme block, body styles, noise SVG, ambient orbs |
| `<body>` tag | Linia 15 | `bg-gray-50 text-gray-900` -> `bg-void-black text-soft-platinum font-display` |
| Noise texture | Nowy element w body | Inline SVG z feTurbulence, position fixed, pointer-events-none |
| Ambient orbs | Nowe elementy w `#app` | 2-3 rozmyte div z gradient, position absolute, z-index -1 |

### Plan 01-02: Glassmorphism components + nawigacja

| Komponent | Funkcja JS | Stale/linie do zmiany |
|-----------|-----------|----------------------|
| Top bar | Statyczny HTML L18-29 | Wrapper + 7 przyciskow |
| Tab navigation | HTML L32-38 + `renderTabs()` L384, L386 | Active/inactive .className |
| Form cards | `renderFormTab()` L527,536,555,574,592,626,692 | 7x `bg-white border rounded` |
| Form inputs | `renderFormTab()` L522 | `inputCls` |
| Form labels | `renderFormTab()` L523 | `labelCls` |
| Form tables | `renderFormTab()` L596-634 | thead, th, td classes |
| Orzeczenie card | `renderFormTab()` L692-723 | Hero Result Card |
| SWZ inputs | `renderSWZRowCells()` L434-435 | `inputCls`, `selectCls` |
| SWZ table | `renderAttachment1()` L736-778 | Table, thead, th, section/subsection rows |
| SWZ section/sub buttons | `renderAttachment1()` L793-810 | 4 typy przyciskow |
| IZOL inputs | `renderIZOLRowCells()` L849 | `inputCls` |
| IZOL special Rp | `renderIZOLRowCells()` L889 | `w-14` input + disabledCls |
| IZOL table | `renderAttachment2()` L920-942 | Table, thead, th |
| RCD inputs | `renderAttachment3()` L1004-1005 | `inputCls`, `selectCls` |
| RCD table | `renderAttachment3()` L1014-1026 | thead, th |
| UZIEM inputs | `renderAttachment4()` L1110 | `inputCls` |
| UZIEM table | `renderAttachment4()` L1118-1130 | thead, th |
| Toast | `showToast()` L1833 | `.className` |
| Orzeczenie update | `updateOrzeczenieDisplay()` L1805-1823 | 5x `.className` |
| Footer | Statyczny HTML L48-50 | Border i tekst |
| Legendy | L828, L994, L1098, L1192 | `text-gray-600` |
| Przyciski dodawania | L733, L917, L1095, L1189 | `bg-blue-600` pattern |
| Empty state rows | L1088, L1182 | `text-gray-400 italic` |

---

## 8. Wzorzec bezpiecznej zamiany

### Krok 1: Definiuj centralne stale

Stworz obiekt `THEME_CLS` na poczatku `<script>`:
```javascript
const THEME_CLS = {
  card: 'bg-white/[0.04] backdrop-blur-xl border border-white/[0.12] rounded-3xl p-4 mb-4',
  input: 'w-full bg-white/[0.06] border border-white/[0.12] rounded-lg px-2 py-1.5 text-sm text-soft-platinum placeholder:text-soft-platinum/30 focus:outline-none focus:ring-2 focus:ring-electric-cyan/50 focus:border-electric-cyan/30',
  label: 'block text-sm font-medium text-soft-platinum/80 mb-1 font-display',
  select: 'w-full bg-white/[0.06] border border-white/[0.12] rounded-lg px-1 py-0.5 text-sm text-soft-platinum',
  cellInput: 'w-full bg-white/[0.06] border border-white/[0.12] rounded px-1 py-0.5 text-sm text-center text-soft-platinum',
  cellInputMono: 'w-14 bg-white/[0.06] border border-white/[0.12] rounded px-0.5 py-0.5 text-sm text-center font-mono text-soft-platinum',
  th: 'border border-white/[0.12] px-2 py-1',
  td: 'border border-white/[0.12] px-2 py-1',
  thead: 'bg-white/[0.06] text-xs text-center text-soft-platinum/70',
  sectionRow: 'bg-white/[0.08]',
  subsectionRow: 'bg-white/[0.04]',
  fixedRow: 'bg-electric-cyan/5',
  btnPrimary: 'bg-electric-cyan/20 text-electric-cyan border border-electric-cyan/30 px-3 py-2 rounded-lg hover:bg-electric-cyan/30 text-sm font-medium backdrop-blur-sm transition-all duration-200',
  btnDanger: 'text-red-400 hover:text-red-300 text-xs',
  btnSmall: 'px-2 py-0.5 rounded text-xs backdrop-blur-sm transition-all duration-200',
  tabActive: 'px-4 py-2 font-medium border-b-2 border-electric-cyan text-electric-cyan font-display',
  tabInactive: 'px-4 py-2 font-medium border-b-2 border-transparent text-soft-platinum/50 hover:text-soft-platinum font-display transition-colors duration-200',
};
```

### Krok 2: Zamien w render functions

```javascript
// PRZED:
const inputCls = 'w-full border border-gray-300 rounded px-2 py-1.5 text-sm';
// PO:
const inputCls = THEME_CLS.input;
```

To zapewnia centralna kontrole i latwe debugowanie.

---

## 9. Ryzyka i pitfalle

### Pitfall 1: Usunecie klasy-selektora JS
**Co idzie nie tak:** Zamiana `verdict-cell` na inna nazwe powoduje ze `updateSWZComputedCells()` nie znajduje komorki
**Jak unikac:** Nigdy nie usuwaj klas z sekcji 4.2. Dodawaj nowe obok.

### Pitfall 2: Zepsucie `hidden` utility
**Co idzie nie tak:** Tailwind v4 Play CDN musi wiedziec o `hidden` -- jesli customowy CSS nadpisze display, tabs przestana dzialac
**Jak unikac:** Nie definiuj `.hidden` w custom CSS. Tailwind automatycznie generuje `hidden { display: none; }`.

### Pitfall 3: Zepsucie `.className =` assignments
**Co idzie nie tak:** JS ustawia pelny className (np. linia 1805). Jesli dodasz klasy do elementu w renderze, a potem JS nadpisze className, dodane klasy znikna.
**Jak unikac:** Dla elementow z dynamicznym `.className =` (orzeczenie-text, next-test-date, toast), WSZYSTKIE klasy musza byc w stringu `.className =`. Nie polegaj na klasach z renderowania.

### Pitfall 4: Niewidoczny tekst na dark bg
**Co idzie nie tak:** Teksty wewnaz inputow dziedzicza kolor z body. Jesli input ma `bg-white/4` a tekst jest `text-soft-platinum`, ale placeholder jest zbyt jasny -- kontrast jest zly.
**Jak unikac:** Testuj kazdy typ inputu z danymi i bez. Placeholder powinien byc `text-soft-platinum/30`.

### Pitfall 5: PDF/Word export NIE POWINIEN byc zmieniony
**Co idzie nie tak:** Redesign przypadkowo zmienia stale w `buildWordHTML()` lub `buildDocDefinition()`
**Jak unikac:** Funkcje `buildWordHTML()`, `buildDocDefinition()`, `buildAttachment*Content()`, `buildFormMainContent()` -- NIE DOTYKAC. Maja wlasny system stylowania.

### Pitfall 6: Font loading failure
**Co idzie nie tak:** Google Fonts @import w `<style>` nie zaladuje sie (wolne polaczenie) -- caly UI wyglada zle
**Jak unikac:** Definiuj fallback w @theme: `'Space Grotesk', system-ui, sans-serif`

### Pitfall 7: backdrop-filter performance
**Co idzie nie tak:** Duzo elementow z `backdrop-blur` powoduje jank na slabszych maszynach
**Jak unikac:** Uzyj `backdrop-blur` tylko na card-level (7 kart formularza + top bar), nie na kazdej komorce tabeli. Komorki tabel uzyja prostego `bg-white/[0.04]` bez blur.

---

## 10. Elementy do dodania (nie istnieja w obecnym kodzie)

| Element | Cel | Gdzie dodac |
|---------|-----|-------------|
| Inline SVG noise texture | feTurbulence overlay na body (VIS-01) | Pierwszy element w `<body>`, position fixed, full screen, pointer-events-none |
| Ambient gradient orbs | 2-3 rozmyte kolorowe kola (VIS-06) | Elementy position absolute w `#app`, z-index -1 |
| Google Fonts @import | Space Grotesk + JetBrains Mono (VIS-04, VIS-05) | W `<style type="text/tailwindcss">` |
| @theme block | Design token system (VIS-02, VIS-03) | W `<style type="text/tailwindcss">` |
| Hero Result Card | Specjalny styl orzeczenia (GLASS-04) | Zmiana klas na `#orzeczenie-section` w renderFormTab() |
| CSS transitions | hover/focus 200-300ms (ANIM-01) | W @layer components lub @theme |
| Focus ring glow | Electric Cyan glow na inputach (ANIM-05) | W inputCls pattern |
| prefers-reduced-motion | Wylaczenie animacji (ANIM-07) | Media query w `<style>` |

---

## Sources

### Primary (HIGH confidence)
- Bezposrednia analiza pliku `/Users/wojciecholszak/Desktop/poprawkiiii/index.html` (2922 linii) -- pelny odczyt
- `.planning/REQUIREMENTS.md` -- wymagania VIS-*, GLASS-*, NAV-*, ANIM-*, PRES-*
- `.planning/ROADMAP.md` -- definicja Phase 1 i planow 01-01, 01-02

## Metadata

**Confidence breakdown:**
- Mapa HTML/JS: HIGH -- pelny odczyt pliku, kazda linia zweryfikowana
- Selektory JS: HIGH -- grep po querySelector/classList
- Wzorce zamiany: MEDIUM -- rekomendacje oparte na doswiadczeniu z Tailwind v4 + glassmorphism
- Performance pitfalls: MEDIUM -- backdrop-blur performance zalezna od hardware

**Research date:** 2026-02-24
**Valid until:** 2026-03-24 (stabilny kod, brak aktywnego rozwoju)
