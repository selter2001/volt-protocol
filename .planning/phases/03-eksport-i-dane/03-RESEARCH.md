# Phase 3: Eksport i Dane - Research

**Researched:** 2026-02-24
**Domain:** pdfmake 0.3.5 DDO (Document Definition Object), localStorage, JSON import/export, Blob download API
**Confidence:** HIGH (pdfmake API) / HIGH (localStorage/JSON)

---

## Summary

Faza 3 implementuje dwa niezależne moduły: `PDFExporter` (03-01) i `Persistence` (03-02). Oba są self-contained — nie wpływają na siebie nawzajem ani na istniejący AppState/EventBus/renderer. To faza integracyjna, nie budowania fundamentów.

**PDFExporter** odczytuje dane z `AppState` i produkuje Document Definition Object (DDO) dla pdfmake. Biblioteka pdfmake 0.3.5 jest już załadowana w `<head>` — wystarczy wywołać `pdfMake.createPdf(docDefinition).download('protokol.pdf')`. Krytyczna kwestia: **Roboto z vfs_fonts.min.js obsługuje polskie znaki** — Roboto pokrywa Latin Extended-A (128 znaków, w tym ą, ę, ś, ź, ć, ó, ł, ń, ż). Polskie znaki działają z domyślnym vfs_fonts bez żadnej dodatkowej konfiguracji fontów. Wymaganie EXP-04 jest spełnione przez Roboto out-of-the-box.

**Zał. 2 (izolacja)** ma 15 kolumn i musi być w orientacji landscape. pdfmake obsługuje mieszane orientacje w jednym dokumencie przez `pageOrientation: 'landscape'` + `pageBreak: 'before'` na konkretnym elemencie treści. Strony Zał. 1, 3, 4 i formularz główny są w portrait — tylko sekcja Zał. 2 jest landscape.

**Persistence** to dwie niezależne funkcje: localStorage save/load (EXP-05/06) i JSON export/import (EXP-07/08). localStorage używa `JSON.stringify(AppState)` → klucz `'voltprotokol-state'`. JSON export to Blob download z `URL.createObjectURL`. JSON import to `<input type="file">` + `FileReader` + `Object.assign`. Po wczytaniu — pełny re-render wszystkich zakładek.

**Primary recommendation:** Zacznij od Persistence (03-02) — prostsze, żadnych decyzji DDO. Potem PDFExporter (03-01) bottom-up: najpierw formularz główny jako tekst, potem każdy załącznik jako tabela. Testuj PDF po każdym załączniku.

---

## Standard Stack

### Core (zdecydowane)

| Biblioteka | Wersja | Cel | Dlaczego |
|------------|--------|-----|----------|
| pdfmake | 0.3.5 | Generowanie PDF | Zdecydowane przed Fazą 1 — auto-paginacja tabel, Roboto z polskimi znakami, CDN |
| Vanilla JS built-in | ES2022+ | localStorage, Blob, FileReader | Zero zależności — wbudowane w przeglądarki |

### Nie potrzeba żadnych nowych bibliotek CDN

Wszystko co potrzebne jest już w pliku:
- `pdfmake@0.3.5` + `vfs_fonts.min.js` — w `<head>` od początku
- `localStorage` API — wbudowane w przeglądarki
- `Blob` + `URL.createObjectURL` — wbudowane w przeglądarki
- `FileReader` — wbudowane w przeglądarki

### Minimalne wersje przeglądarek (dla używanych API)

| API | Chrome | Firefox | Safari |
|-----|--------|---------|--------|
| localStorage | 4+ | 3.5+ | 4+ |
| Blob + URL.createObjectURL | 23+ | 19+ | 7.1+ |
| FileReader | 6+ | 3.6+ | 6+ |
| `<a download>` attribute | 14+ | 20+ | 10.1+ |

---

## Architecture Patterns

### Recommended Structure (rozszerzenie istniejącego `<script>`)

```
index.html <script>
│
├── // === CONSTANTS ===
├── // === STATE ===       (AppState, UIState)
├── // === EVENT BUS ===
├── // === CALCULATOR ===
├── // === FORM RENDERER ===
├── // === FORM CONTROLLER ===
│
├── // === PDF EXPORTER ===          (NOWE - 03-01)
│   ├── buildFormMainContent()       formularz główny → DDO content[]
│   ├── buildAttachment1Content()    Zał. 1 (SWZ) → DDO table
│   ├── buildAttachment2Content()    Zał. 2 (IZOL) → DDO table, landscape
│   ├── buildAttachment3Content()    Zał. 3 (RCD) → DDO table
│   ├── buildAttachment4Content()    Zał. 4 (UZIEM) → DDO table
│   ├── buildDocDefinition()         łączy wszystkie sekcje w DDO
│   └── exportPDF()                  wywołuje pdfMake.createPdf().download()
│
├── // === PERSISTENCE ===           (NOWE - 03-02)
│   ├── saveToLocalStorage()
│   ├── loadFromLocalStorage()
│   ├── exportToJSON()
│   └── importFromJSON()
│
└── // === INIT ===
```

### Pattern 1: Minimalne DDO dla pdfmake

```javascript
// Źródło: https://pdfmake.github.io/docs/0.3/getting-started/client-side/
// Wywołanie: pdfMake.createPdf(docDefinition).download('protokol.pdf')

function buildDocDefinition() {
  return {
    pageSize: 'A4',
    pageOrientation: 'portrait',   // domyślna — Zał. 2 nadpisze per-element
    pageMargins: [40, 60, 40, 60], // [left, top, right, bottom] w pt

    // Numeracja stron w stopce (EXP-02)
    footer: function(currentPage, pageCount) {
      return {
        text: currentPage.toString() + ' / ' + pageCount,
        alignment: 'center',
        fontSize: 9,
        margin: [0, 10, 0, 0]
      };
    },

    defaultStyle: {
      font: 'Roboto',     // domyślny font z vfs_fonts.min.js
      fontSize: 9         // 9pt — protokoły mają dużo danych
    },

    content: [
      ...buildFormMainContent(),
      ...buildAttachment1Content(),
      // Zał. 2: landscape — pageOrientation per-element
      {
        table: buildAttachment2Table(),
        pageOrientation: 'landscape',
        pageBreak: 'before'
      },
      // Powrót do portrait dla Zał. 3
      {
        stack: buildAttachment3Content(),
        pageOrientation: 'portrait',
        pageBreak: 'before'
      },
      ...buildAttachment4Content()
    ]
  };
}
```

### Pattern 2: Tabela pdfmake z headerRows (auto-paginacja, EXP-03)

```javascript
// Źródło: https://pdfmake.github.io/docs/0.3/document-definition-object/tables/
// headerRows: N — N pierwszych wierszy powtarza się na każdej stronie

function buildTableWithHeaderRows(headers, body, widths) {
  return {
    table: {
      headerRows: headers.length,  // liczba wierszy nagłówka
      widths: widths,               // szerokości kolumn
      body: [
        ...headers,                 // wiersze nagłówka
        ...body                     // wiersze danych
      ]
    },
    layout: 'lightHorizontalLines'  // lub własny layout
  };
}

// Przykład Zał. 1 (SWZ) — nagłówek 4-wierszowy, 13 kolumn
const swzTable = buildTableWithHeaderRows(
  [
    // Wiersz 1: Lp, Obwód, Zabezpieczenia (colSpan=4), Usk, Ia, Id, Zs, Zsmax, tw, Ocena
    [
      {text: 'Lp.', rowSpan: 4, alignment: 'center'},
      {text: 'Obwód', rowSpan: 4, alignment: 'center'},
      {text: 'Zabezpieczenia', colSpan: 4, alignment: 'center'},
      {}, {}, {},
      {text: 'Usk', rowSpan: 3, alignment: 'center'},
      {text: 'Ia', rowSpan: 3, alignment: 'center'},
      {text: 'Id [A]', rowSpan: 3, alignment: 'center'},
      {text: 'Zs', rowSpan: 3, alignment: 'center'},
      {text: 'Zs max', rowSpan: 3, alignment: 'center'},
      {text: 'tw', rowSpan: 3, alignment: 'center'},
      {text: 'Ocena', rowSpan: 4, alignment: 'center'}
    ],
    // Wiersz 2: dodatkowe (colSpan=2), podstawowe (colSpan=2)
    [{}, {}, {text: 'dodatkowe', colSpan: 2}, {}, {text: 'podstawowe', colSpan: 2}, {}, {}, {}, {}, {}, {}, {}, {}],
    // Wiersz 3: typ, prąd, typ, prąd IΔn
    [{}, {}, {text: 'typ'}, {text: 'prąd'}, {text: 'typ'}, {text: 'prąd IΔn'}, {}, {}, {}, {}, {}, {}, {}],
    // Wiersz 4: jednostki
    [{}, {}, {text: '-'}, {text: '[A]'}, {text: '-'}, {text: '[A]'}, {text: '[V]'}, {text: '[A]'}, {text: '[A]'}, {text: '[Ω]'}, {text: '[Ω]'}, {text: '[s]'}, {}]
  ],
  body,        // wiersze danych z AppState
  widths       // np. [20, '*', 30, 30, 30, 35, 30, 30, 35, 30, 30, 25, 55]
);
```

**KRYTYCZNE:** Przy `colSpan` i `rowSpan` — każda "zajęta" komórka musi mieć placeholder `{}` w wierszu. Brak placeholder powoduje błąd `Cannot read property 'text' of undefined`.

### Pattern 3: Wiersz sekcji i podsekcji (full-width merged row)

```javascript
// Źródło: analiza reference-protokol-extracted.md + pdfmake docs
// Sekcja-nagłówek (np. "Parter") — ciemniejsze tło, bold
function makeSectionRow(title, numCols) {
  const cells = Array(numCols).fill('');
  cells[0] = {
    text: title,
    bold: true,
    colSpan: numCols,
    fillColor: '#d1d5db',    // szare tło — czytelne w B&W (EXP-02)
    alignment: 'left',
    fontSize: 9
  };
  return cells;
}

// Podsekcja (np. "Pokój 1 - obwód gniazd wtykowych") — jaśniejsze tło, kursywa
function makeSubsectionRow(title, numCols) {
  const cells = Array(numCols).fill('');
  cells[0] = {
    text: title,
    italics: true,
    colSpan: numCols,
    fillColor: '#f3f4f6',    // bardzo jasne szare
    alignment: 'left',
    fontSize: 9
  };
  return cells;
}
```

### Pattern 4: Orientacja landscape dla Zał. 2

```javascript
// Źródło: pdfmake docs + https://pdfmake.github.io/docs/0.3/document-definition-object/page/
// pageOrientation per-element zmienia orientację od tej strony
// pageBreak: 'before' zapewnia nową stronę przed zmianą orientacji
// pageBreak: 'after' + portrait przywraca portrait NA NASTĘPNEJ stronie

content: [
  // ... formularz główny i Zał. 1 (portrait) ...

  // Zał. 2 w landscape:
  {
    table: { /* attachment2 table */ },
    pageOrientation: 'landscape',
    pageBreak: 'before'
  },

  // Zał. 3 z powrotem do portrait:
  {
    text: 'ZAŁĄCZNIK NR 3...',
    pageOrientation: 'portrait',
    pageBreak: 'before'
  },
  // ... reszta Zał. 3 i 4 w portrait
]
```

**UWAGA:** `pageOrientation` na elemencie `table` nie jest oficjalnie udokumentowane — działa przez implementację pdfmake, ale bezpieczniej jest opakować tabelę w `{ stack: [...], pageOrientation: 'landscape', pageBreak: 'before' }`. Weryfikacja wymagana przy implementacji.

### Pattern 5: localStorage save/load (EXP-05/06)

```javascript
// Źródło: MDN Web API localStorage + JSON.stringify/parse
const STORAGE_KEY = 'voltprotokol-state';

function saveToLocalStorage() {
  try {
    // NIE zapisuj calculated — liczymy przy wczytaniu
    // Zapisuj cały AppState (z calculated to OK — mała aplikacja)
    localStorage.setItem(STORAGE_KEY, JSON.stringify(AppState));
    showToast('Zapisano.');
  } catch (e) {
    // Możliwy QuotaExceededError (5-10MB limit, ale AppState jest mały <100KB)
    showToast('Błąd zapisu: ' + e.message);
  }
}

function loadFromLocalStorage() {
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    if (!raw) { showToast('Brak zapisanych danych.'); return; }
    const saved = JSON.parse(raw);
    restoreAppState(saved);
    showToast('Wczytano.');
  } catch (e) {
    showToast('Błąd odczytu: ' + e.message);
  }
}
```

### Pattern 6: JSON export/import (EXP-07/08)

```javascript
// Źródło: MDN Blob API + anchor download attribute
function exportToJSON() {
  const json = JSON.stringify(AppState, null, 2);
  const blob = new Blob([json], { type: 'application/json' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  // Nazwa pliku z numerem protokołu i datą
  const proto = AppState.form.protocolNumber.replace(/\//g, '-') || 'protokol';
  a.download = `voltprotokol-${proto}.json`;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(url);   // WAŻNE: zwolnij pamięć
}

// JSON import — input[type=file] + FileReader
function initJSONImport() {
  const input = document.createElement('input');
  input.type = 'file';
  input.accept = '.json,application/json';
  input.onchange = (e) => {
    const file = e.target.files[0];
    if (!file) return;
    const reader = new FileReader();
    reader.onload = (ev) => {
      try {
        const saved = JSON.parse(ev.target.result);
        restoreAppState(saved);
        showToast('Wczytano z pliku.');
      } catch (err) {
        showToast('Błąd: nieprawidłowy plik JSON.');
      }
    };
    reader.readAsText(file);
  };
  input.click();
}
```

### Pattern 7: restoreAppState() — krytyczna funkcja po wczytaniu

```javascript
// Po wczytaniu danych (localStorage lub JSON) — pełny reset i re-render
function restoreAppState(saved) {
  // Głęboka kopia do AppState (bezpieczne podejście)
  Object.assign(AppState, JSON.parse(JSON.stringify(saved)));

  // Przelicz calculated dla wszystkich wierszy
  // (JSON nie przechowuje funkcji; calculated może być null po parse)
  recalculateAll();

  // Pełny re-render wszystkich zakładek
  renderFormTab();
  renderAttachment1();
  renderAttachment2();
  renderAttachment3();
  renderAttachment4();

  // Aktualizuj wyniki oceny końcowej
  updateFinalAssessmentDisplay();
}

function recalculateAll() {
  // SWZ fixed row
  AppState.swzFixedRow.calculated = calcSWZRow(AppState.swzFixedRow);

  // SWZ dynamic rows
  for (const sec of AppState.sections) {
    for (const sub of sec.subsections) {
      const rows1 = AppState.attachment1.rowsBySubsection[sub.id] || [];
      rows1.forEach(row => row.calculated = calcSWZRow(row));

      const rows2 = AppState.attachment2.rowsBySubsection[sub.id] || [];
      rows2.forEach(row => row.calculated = calcIZOLRow(row));
    }
  }

  // Attachment 2 fixed row
  AppState.attachment2.fixedRow.calculated = calcIZOLRow(AppState.attachment2.fixedRow);

  // RCD
  AppState.attachment3.rows.forEach(row => row.calculated = calcRCDRow(row));

  // UZIEM
  AppState.attachment4.rows.forEach(row => row.calculated = calcUZIEMRow(row));
}
```

### Pattern 8: Przyciski PDF/Zapisz/Wczytaj w UI

Przyciski dodane do istniejącego nagłówka lub na zakładce "Protokół". Prosta implementacja:

```javascript
// Dodaj przyciski do HTML (w <div id="app"> obok <h1>)
// Lub do contenera zakładki "Protokół"
// Bez EventBus — direct button click handlers

function bindExportButtons() {
  document.getElementById('btn-export-pdf').addEventListener('click', exportPDF);
  document.getElementById('btn-save').addEventListener('click', saveToLocalStorage);
  document.getElementById('btn-load').addEventListener('click', loadFromLocalStorage);
  document.getElementById('btn-export-json').addEventListener('click', exportToJSON);
  document.getElementById('btn-import-json').addEventListener('click', initJSONImport);
}
```

### Anti-Patterns to Avoid

- **Generowanie PDF z html2canvas:** html2canvas z 2660 węzłami DOM = 66s freeze. NIE używaj. Zdecydowane: pdfmake programatycznie z AppState.
- **Odczyt danych z DOM przy generowaniu PDF:** PDFExporter czyta z `AppState`, nigdy z `document.querySelector`. DOM może nie być aktualny.
- **Auto-save na każdy keystroke:** Blokuje synchronicznie wątek główny (localStorage.setItem jest synchroniczne). Ręczny przycisk "Zapisz" (EXP-05) jest wymagany i wystarczy.
- **Placeholder `''` zamiast `{}` w rowSpan/colSpan:** pdfmake oczekuje obiektów jako placeholderów, nie stringów. `''` może działać w niektórych wersjach, ale `{}` jest bezpieczne.
- **`calculated` jako źródło prawdy przy wczytaniu:** Po `JSON.parse` pole `calculated` może mieć stare wartości lub `null`. Zawsze wywołaj `recalculateAll()` po wczytaniu stanu.
- **Brak `URL.revokeObjectURL()` po download:** Powoduje wyciek pamięci. Zawsze revoke po kliknięciu linku.
- **`JSON.parse(JSON.stringify(state))` przy shallow assign:** Głęboka kopia chroni przed mutacją referencji. Użyj `Object.assign` z deep copy dla pewności.

---

## Don't Hand-Roll

| Problem | Nie buduj | Użyj zamiast | Dlaczego |
|---------|-----------|--------------|----------|
| Generowanie PDF | własny PDF renderer | pdfmake DDO | Paginacja, fonty, landscape/portrait — ogromna złożoność ręcznie |
| Polskie znaki w PDF | własna enkodowania | Roboto z vfs_fonts.min.js | Roboto pokrywa Latin Extended-A — ą,ę,ś,ź,ć,ó,ł,ń,ż out-of-the-box |
| Persystencja danych | własna baza danych | localStorage + JSON | 5-10MB wystarczy; AppState to <100KB |
| Download pliku | własny serwer | Blob + URL.createObjectURL + `<a download>` | Zero serwerów, zero zależności |
| Upload pliku | własny drag&drop | `<input type="file">` + FileReader | Wbudowane w HTML5, zero zależności |
| Animacja postępu PDF | własny spinner | blokujące `pdfMake.createPdf().download()` | PDF generuje się synchronicznie (<1s dla typowego protokołu) |

**Key insight:** pdfmake programatycznie z AppState to jedyna opcja bez 66s freeze html2canvas. Cała wartość tej fazy to prawidłowe mapowanie AppState → DDO.

---

## Common Pitfalls

### Pitfall 1: colSpan/rowSpan wymaga dokładnie N-1 placeholderów `{}`

**What goes wrong:** Tabela z `colSpan: 4` bez 3 placeholderów w tym samym wierszu powoduje błąd pdfmake: "Cannot read property 'text' of undefined" lub misalignment kolumn.

**Why it happens:** pdfmake liczy komórki per wiersz. Przy `colSpan: N` oczekuje N komórek (jedna główna + N-1 `{}`). Przy `rowSpan: N` oczekuje N wierszy z komórką główną w pierwszym i `{}` w kolejnych.

**How to avoid:** Nagłówek Zał. 1 ma 4 wiersze nagłówka × 13 kolumn. Każdy wiersz musi mieć dokładnie 13 elementów. Policz dokładnie. Użyj helpera:
```javascript
function makeHeaderRow(cells, totalCols) {
  console.assert(cells.reduce((n, c) => n + (c.colSpan || 1), 0) === totalCols,
    'Header row must have exactly ' + totalCols + ' columns');
  return cells;
}
```

**Warning signs:** "Error: Invalid table header" lub kolumny przesunięte w PDF.

### Pitfall 2: Polskie znaki — Roboto w vfs_fonts.min.js vs vfs_fonts.js

**What goes wrong:** Polskie znaki wyświetlają się jako `?` lub są puste.

**Why it happens:** pdfmake's Standard 14 fonts (Helvetica, Times, Courier) obsługują TYLKO ANSI (angielski). Roboto z vfs_fonts OBSŁUGUJE Latin Extended-A. Problem pojawia się gdy ktoś przypadkowo określi font jako 'Helvetica' zamiast 'Roboto'.

**How to avoid:** Upewnij się, że `defaultStyle.font: 'Roboto'` jest ustawione w DDO. Roboto z CDN `vfs_fonts.min.js` pokrywa Latin Extended-A (128 glyphy, w tym wszystkie polskie znaki).

**Confidence:** MEDIUM — Roboto pokrywa Latin Extended-A (potwierdzone przez Google Fonts i Wikipedia), ale faktyczne działanie z `vfs_fonts.min.js` z CDN wymaga weryfikacji przy implementacji.

**Warning signs:** Polskie znaki jako `?` lub whitespace w PDF.

**Fallback jeśli Roboto z CDN nie obsługuje polskich znaków:**
Option A: Użyj font URL z CDN Google Fonts z Latin Extended:
```javascript
pdfMake.fonts = {
  Roboto: {
    normal: 'https://fonts.gstatic.com/s/roboto/v32/KFOmCnqEu92Fr1Mu4mxK.woff2',
    // ... pozostałe warianty
  }
};
```
Option B: Base64-encode Roboto z Google Fonts i dołącz inline do HTML (brak CDN dependency, ale +1MB HTML).

**Rekomendacja:** Przetestuj z domyślnym CDN vfs_fonts.min.js jako pierwsze krok. Jeśli polskie znaki nie działają — Option A (URL font z Google Fonts).

### Pitfall 3: pageOrientation landscape dla jednej sekcji tabeli

**What goes wrong:** Cały dokument staje się landscape, zamiast tylko Zał. 2.

**Why it happens:** `pageOrientation` w root DDO ustawia orientację dla całego dokumentu. Aby zmienić per-strona, trzeba użyć `pageOrientation` per element z `pageBreak`.

**How to avoid:**
```javascript
// BŁĄD: cały dokument landscape
{ pageOrientation: 'landscape', content: [...] }

// OK: tylko element z pageBreak
{ stack: [attachment2Table], pageOrientation: 'landscape', pageBreak: 'before' }
// Następny element z powrotem do portrait:
{ text: 'Załącznik 3...', pageOrientation: 'portrait', pageBreak: 'before' }
```

**Warning signs:** Strony Zał. 1 i 3 są landscape w wygenerowanym PDF.

### Pitfall 4: localStorage QuotaExceededError

**What goes wrong:** `localStorage.setItem()` rzuca `QuotaExceededError` gdy dane przekroczą limit (typowo 5-10MB per origin).

**Why it happens:** AppState z dużą liczbą pomiarów może rosnąć. Typowy protokół (100 wierszy × 4 załączniki) to ~200KB JSON — bezpieczne. Problem pojawia się gdy użytkownik ma starą sesję + nową + dużo danych.

**How to avoid:** Zawsze otaczaj `localStorage.setItem` w `try/catch`. Informuj użytkownika o błędzie.

**Warning signs:** Cichy brak zapisu (bez try/catch błąd jest niezłapany).

### Pitfall 5: FileReader.onload — wynik jest string, nie object

**What goes wrong:** Po `reader.readAsText(file)` wynik `ev.target.result` to String, nie Object. Próba bezpośredniego `Object.assign(AppState, ev.target.result)` przypisuje string.

**How to avoid:** Zawsze `JSON.parse(ev.target.result)` przed użyciem.

**Warning signs:** AppState staje się stringiem — wszystkie renderery crashują.

### Pitfall 6: PDF blokuje UI podczas generowania

**What goes wrong:** pdfmake generuje PDF synchronicznie. Dla bardzo dużych dokumentów (500+ wierszy) może zablokować UI na 1-3 sekundy.

**Why it happens:** JavaScript jest single-threaded. pdfmake nie używa Web Workers.

**How to avoid:** Przed wywołaniem `exportPDF()` pokaż prostą informację "Generowanie...". Dla typowego protokołu (50-100 wierszy) czas generowania wynosi <500ms — nie jest problemem.

**Warning signs:** UI freezuje się na chwilę po kliknięciu "Eksportuj PDF".

### Pitfall 7: `JSON.parse` nie odtwarza funkcji w `calculated`

**What goes wrong:** Po wczytaniu stanu z localStorage, pole `row.calculated` może być `{Id: null, verdict: null}` (poprawne) lub mieć stare wartości bez przeliczenia. To nie jest błąd krytyczny, ale wiersze mogą pokazywać stare obliczenia.

**How to avoid:** Po każdym `restoreAppState()` wywołaj `recalculateAll()`. Opcjonalnie: nie zapisuj `calculated` w JSON — tylko dane wejściowe, a `calculated` zawsze przeliczaj przy wczytaniu (czystsze podejście).

---

## Code Examples

Zweryfikowane wzorce z oficjalnych źródeł:

### Minimalny działający PDF z polskim tekstem

```javascript
// Źródło: https://pdfmake.github.io/docs/0.3/getting-started/client-side/
function exportPDF() {
  const docDefinition = {
    pageSize: 'A4',
    defaultStyle: { font: 'Roboto', fontSize: 10 },
    footer: function(currentPage, pageCount) {
      return { text: currentPage + ' / ' + pageCount, alignment: 'center', fontSize: 8 };
    },
    content: [
      { text: 'Protokół kontrolno-pomiarowy PN-HD 60364-6', bold: true, fontSize: 14 },
      { text: 'Polskie znaki: ą ę ś ź ć ó ł ń ż Ą Ę Ś Ź Ć Ó Ł Ń Ż' },
      { text: 'Wynik: POZYTYWNA' }
    ]
  };
  pdfMake.createPdf(docDefinition).download('protokol.pdf');
}
```

### Tabela z nagłówkiem wielorzędowym (fragment Zał. 1)

```javascript
// Źródło: pdfmake docs tables + analiza reference-protokol-extracted.md
function buildSWZTableDDO() {
  // Nagłówek 4-wierszowy — dokładnie 13 elementów per wiersz
  const header = [
    // Wiersz 1 (row 0): Lp(r4), Obwód(r4), Zabezpieczenia(c4), Usk(r3), Ia(r3), Id(r3), Zs(r3), Zsmax(r3), tw(r3), Ocena(r4)
    [
      {text:'Lp.', rowSpan:4, alignment:'center'},
      {text:'Obwód', rowSpan:4, alignment:'center'},
      {text:'Zabezpieczenia', colSpan:4, alignment:'center'}, {}, {}, {},
      {text:'Usk [V]', rowSpan:3, alignment:'center'},
      {text:'Ia [A]', rowSpan:3, alignment:'center'},
      {text:'Id [A]', rowSpan:3, alignment:'center'},
      {text:'Zs [Ω]', rowSpan:3, alignment:'center'},
      {text:'Zs max [Ω]', rowSpan:3, alignment:'center'},
      {text:'tw [s]', rowSpan:3, alignment:'center'},
      {text:'Ocena', rowSpan:4, alignment:'center'}
    ],
    // Wiersz 2: puste(r), puste(r), dodatkowe(c2), podstawowe(c2), puste×6, puste(r)
    [{},{},{text:'dodatkowe',colSpan:2,alignment:'center'},{},{text:'podstawowe',colSpan:2,alignment:'center'},{},{},{},{},{},{},{},{}],
    // Wiersz 3: puste×2, typ, prąd, typ, prąd IΔn, puste×6, puste
    [{},{},{text:'typ'},{text:'prąd'},{text:'typ'},{text:'prąd IΔn'},{},{},{},{},{},{},{}],
    // Wiersz 4: puste×2, -, [A], -, [A], [V], [A], [A], [Ω], [Ω], [s], -
    [{},{},{text:'-'},{text:'[A]'},{text:'-'},{text:'[A]'},{text:'[V]'},{text:'[A]'},{text:'[A]'},{text:'[Ω]'},{text:'[Ω]'},{text:'[s]'},{text:'-'}]
  ];

  const body = buildSWZBody();  // wiersze danych z AppState

  return {
    table: {
      headerRows: 4,   // powtarza 4 wiersze na każdej stronie (EXP-03)
      widths: [20, '*', 25, 30, 25, 35, 30, 30, 30, 30, 35, 25, 55],
      body: [...header, ...body]
    },
    layout: 'lightHorizontalLines'
  };
}
```

### JSON save/load z try/catch

```javascript
// Źródło: MDN localStorage + JSON API
const STORAGE_KEY = 'voltprotokol-state';

function saveToLocalStorage() {
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(AppState));
    showToast('Dane zapisane.');
  } catch (e) {
    if (e.name === 'QuotaExceededError') {
      showToast('Błąd: brak miejsca w przeglądarce.');
    } else {
      showToast('Błąd zapisu: ' + e.message);
    }
  }
}

function loadFromLocalStorage() {
  const raw = localStorage.getItem(STORAGE_KEY);
  if (!raw) { showToast('Brak zapisanych danych.'); return; }
  try {
    const saved = JSON.parse(raw);
    restoreAppState(saved);
    showToast('Dane wczytane.');
  } catch (e) {
    showToast('Błąd: uszkodzone dane w localStorage.');
  }
}
```

### Blob download JSON

```javascript
// Źródło: MDN Blob API + anchor download attribute
function exportToJSON() {
  const json = JSON.stringify(AppState, null, 2);
  const blob = new Blob([json], { type: 'application/json' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  const proto = (AppState.form.protocolNumber || 'protokol').replace(/\//g, '-');
  a.download = 'voltprotokol-' + proto + '.json';
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(url);
}
```

### Prosty toast notification (bez biblioteki)

```javascript
// Toast dla feedbacku po Zapisz/Wczytaj — bez alert(), bez biblioteki
function showToast(message) {
  const toast = document.createElement('div');
  toast.textContent = message;
  toast.className = 'fixed bottom-4 right-4 bg-gray-800 text-white px-4 py-2 rounded shadow-lg text-sm z-50';
  document.body.appendChild(toast);
  setTimeout(() => document.body.removeChild(toast), 2500);
}
```

---

## Struktura DDO dla pełnego protokołu

Na podstawie `reference-protokol-extracted.md` i AppState z Fazy 1+2:

### Sekcja 1: Formularz główny (portrait, strona 1-2)

```
content: [
  // Tytuł
  { text: 'PROTOKÓŁ KONTROLNO POMIAROWY NR ' + form.protocolNumber, bold: true, fontSize: 14 },

  // Punkt 1: Obiekt badany
  { text: '1. Obiekt badany:', bold: true, margin: [0, 10, 0, 5] },
  { text: form.objectName + ', dz. ' + form.objectParcel + ', adres ' + form.objectAddress },

  // Punkt 2: Wykonawca
  { text: '2. Wykonawca:', bold: true, margin: [0, 10, 0, 5] },
  { text: 'Protokół oraz badania wykonał oraz zatwierdził ' + form.executorName + '. ...' },

  // Punkt 3: Warunki badań
  // ... przyrządy pomiarowe jako tabela 3-kolumnowa

  // Punkt 4: Ocena końcowa — tabela 3-kolumnowa, 6 wierszy
  // Punkt 5: Orzeczenie — bold text

  // Podpis: '___ ___' + 'wykonawca miejscowość i data'
]
```

### Sekcja 2: Zał. 1 (portrait) — auto-paginacja przez headerRows: 4

```
// Tytuł Zał. 1.X generowany automatycznie przez paginację (headerRows)
// Lub: każda "strona" Zał. 1 to osobna sekcja content[] z tytułem
// Prościej: jeden blok content z headerRows — pdfmake dzieli automatycznie
```

### Sekcja 3: Zał. 2 (landscape) — 15 kolumn

```javascript
// 15 kolumn: Lp, Nazwa, TyPrzewodu, L1N, L2N, L3N, L1L2, L1L3, L2L3, L1PE, L2PE, L3PE, NPE, Rw, Ocena
// Szerokości w landscape (841pt szerokości - 80pt margines = 761pt)
// np. [20, '*', 40, 30, 30, 30, 30, 30, 30, 30, 30, 30, 30, 25, 55]
```

### Sekcja 4 i 5: Zał. 3 i 4 (portrait) — proste tabele

```
// Zał. 3 (RCD): 8 kolumn — Lp, Typ, TEST, In, Iw, tw, tz, Ocena
// Zał. 4 (UZIEM): 7 kolumn — Lp, Nazwa, Rp, Wk, Rpo, Rw, Ocena
```

---

## State of the Art

| Stare podejście | Aktualne | Zmienione | Wpływ |
|-----------------|----------|-----------|-------|
| html2canvas → PDF | pdfmake DDO programatycznie | Zdecydowane przed Fazą 1 | 66s freeze → <500ms |
| auto-save na keystroke | ręczny przycisk "Zapisz" | Best practice | Bez blokowania UI |
| IndexedDB dla danych | localStorage | Wymagania mówią localStorage | Prostsze API |
| Server-side download | Blob + URL.createObjectURL | HTML5 File API | Zero serwerów |
| alert() dla feedbacku | Toast notification | UX best practice | Nie blokuje UI |

**Deprecated/outdated:**
- `saveAs` z FileSaver.js — niepotrzebne gdy używamy Blob + `<a download>` (to samo zachowanie, zero zależności)
- `pdfMake.open()` — otwiera PDF w nowej karcie zamiast pobierać; `.download()` jest właściwe dla EXP-01

---

## Open Questions

1. **Czy Roboto z CDN `vfs_fonts.min.js` faktycznie obsługuje polskie znaki bez konfiguracji?**
   - Co wiemy: Roboto pokrywa Latin Extended-A (ą, ę, ś, ź, ć, ó, ł, ń, ż są w Latin Extended-A). vfs_fonts.min.js z CDN zawiera Roboto. Pdfmake domyślnie używa Roboto.
   - Co jest niejasne: Czy konkretna kompilacja Roboto w vfs_fonts.min.js@0.3.5 zawiera pełny subset Latin Extended-A, czy tylko Basic Latin?
   - Rekomendacja: Przetestuj polskie znaki w pierwszym kroku 03-01 (przed budowaniem tabel). Jeśli nie działają, przełącz na font URL z Google Fonts CDN (Latin Extended subset).
   - Fallback URL: `https://fonts.gstatic.com/s/roboto/v32/KFOmCnqEu92Fr1Mu4mxK.woff2` (Roboto Regular z Latin Extended)
   - Confidence: MEDIUM — wymaga weryfikacji przy implementacji

2. **pageOrientation per-element na table vs stack wrapper**
   - Co wiemy: pdfmake obsługuje `pageOrientation` per-element. Działa dla `text` nodes.
   - Co jest niejasne: Czy `pageOrientation` bezpośrednio na obiekcie `{ table: {...} }` działa, czy trzeba opakować w `{ stack: [tableObj], pageOrientation: 'landscape' }`?
   - Rekomendacja: Użyj stack wrapper jako bezpieczniejszy pattern. Testuj przy implementacji.
   - Confidence: MEDIUM — docs 0.3 nie są jednoznaczne w tej kwestii

3. **Gdzie umieścić przyciski PDF/Zapisz/Wczytaj w UI?**
   - Co wiemy: Brak CONTEXT.md — brak decyzji użytkownika. Wymagania mówią "przycisk Eksportuj PDF", "przycisk Zapisz", "przycisk Wczytaj".
   - Co jest niejasne: Czy przyciski są na zakładce "Protokół", na górze strony (sticky), czy w każdej zakładce?
   - Rekomendacja: Przyciski na górze aplikacji (obok <h1>) widoczne z każdej zakładki — elektryk eksportuje PDF z Zał. 1, nie musi wracać do zakładki "Protokół". Sticky bar lub fixed header.
   - Confidence: MEDIUM — decyzja UX, obie opcje prawidłowe technicznie

4. **Czy zapisywać `calculated` w localStorage/JSON?**
   - Co wiemy: `calculated` to dane pochodne z `calcSWZRow(row)` itp. Można je zawsze przelicz.
   - Co jest niejasne: Czy bardziej czytelnie nie zapisywać `calculated` i liczyć przy wczytaniu?
   - Rekomendacja: Zapisuj cały AppState łącznie z `calculated` (prostsze). Po wczytaniu i tak wywołaj `recalculateAll()` dla spójności (bo wersja aplikacji mogła się zmienić).
   - Confidence: HIGH — obie opcje działają

5. **Numeracja Zał. 1.1, 1.2... w PDF (EXP-03)**
   - Co wiemy: Reference pokazuje "Zał. 1.1", "Zał. 1.2" — numery stron w ramach jednego załącznika.
   - Co jest niejasne: Czy numeracja 1.1, 1.2 jest w tytule każdej strony tabeli, czy to po prostu zwykła numeracja stron w stopce?
   - Rekomendacja: pdfmake z `headerRows: 4` automatycznie powtarza nagłówek tabeli na każdej stronie. Numeracja Zał. 1.1, 1.2 to po prostu numer strony w stopce (cały dokument). Osobna numeracja per-załącznik jest zbędną złożonością.
   - Confidence: MEDIUM — reference shows "Zał. 1.1, 1.2" w tabeli oceny, ale to może być po prostu numery stron w dokumencie

---

## Sources

### Primary (HIGH confidence)

- `https://pdfmake.github.io/docs/0.3/getting-started/client-side/` — pdfmake 0.3 CDN setup, createPdf().download() API
- `https://pdfmake.github.io/docs/0.3/document-definition-object/tables/` — table DDO: headerRows, widths, body, colSpan, rowSpan, layout
- `https://pdfmake.github.io/docs/0.3/document-definition-object/page/` — pageSize, pageOrientation, pageMargins, per-element orientation
- `https://pdfmake.github.io/docs/0.1/document-definition-object/headers-footers/` — footer function z currentPage/pageCount (0.1 docs, ale API kompatybilne z 0.3)
- `https://pdfmake.github.io/docs/0.3/fonts/standard-14-fonts/` — Standard 14 fonts obsługują TYLKO ANSI (nie polskie znaki) — dlatego używamy Roboto
- `/Users/wojciecholszak/Desktop/VoltProtokół/reference-protokol-extracted.md` — struktura protokołu: formularz główny, Zał. 1-4, nagłówki tabel, układ
- `/Users/wojciecholszak/Desktop/VoltProtokół/index.html` — aktualny AppState (lines 84-137), EventBus, Calculator, renderery — do czego PDFExporter się integruje
- MDN — localStorage API, Blob API, URL.createObjectURL, FileReader, `<a download>`

### Secondary (MEDIUM confidence)

- Wikipedia — Roboto covers "Latin Extended-A (128) and Latin Extended-B (18)" — potwierdzenie pokrycia polskich znaków
- Google Fonts — Roboto Latin Extended subset — alternatywny fallback dla fontów
- `https://pdfmake.github.io/docs/0.1/document-definition-object/page/` — per-page orientation example (0.1 docs)

### Tertiary (LOW confidence)

- WebSearch wyniki o pdfmake + Polish characters — niejednoznaczne, wymagają weryfikacji przy implementacji
- pdfmake-unicode npm package — alternatywa jeśli Roboto z vfs_fonts nie działa (nie używamy jeśli Roboto działa)

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — pdfmake 0.3.5 CDN + vanilla JS APIs, wszystko zweryfikowane z oficjalnych docs
- Architecture (PDFExporter): HIGH — DDO pattern zweryfikowany z docs; mapowanie AppState→DDO jest mechaniczne
- Architecture (Persistence): HIGH — localStorage/JSON/Blob to standardowe Web APIs, dobrze znane
- Polskie znaki w Roboto: MEDIUM — Roboto pokrywa Latin Extended-A (potwierdzone), ale CDN vfs_fonts.min.js wymaga weryfikacji przy implementacji
- pageOrientation per-element: MEDIUM — działa wg docs i przykładów, ale stack wrapper vs direct na table wymaga testów
- Pitfalls: HIGH — colSpan/rowSpan placeholders i QuotaExceededError to dobrze znane problemy pdfmake

**Research date:** 2026-02-24
**Valid until:** 2026-03-24 (30 dni — stabilna domena, pdfmake 0.3 stabilny)
