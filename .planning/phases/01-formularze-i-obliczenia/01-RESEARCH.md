# Phase 1: Formularze i Obliczenia - Research

**Researched:** 2026-02-24
**Domain:** Vanilla JS single-file SPA — dynamiczne formularze pomiarowe, obliczenia elektryczne, stan aplikacji
**Confidence:** HIGH

---

## Summary

Faza 1 buduje kompletny fundament aplikacji: AppState jako jedyne źródło prawdy, cztery dynamiczne tabelaryczne formularze pomiarowe z automatycznymi obliczeniami i oceną, oraz bazę zabezpieczeń PROTECTION_DB. Jest to faza zerowych zewnętrznych zależności ponad już zdecydowany stack (HTML + Tailwind CDN + vanilla JS) — żadna nowa biblioteka nie jest potrzebna. Cała logika tej fazy to czysty JavaScript operujący na danych.

Kluczowy wzorzec techniczny to hierarchia danych: Section → Subsection → Row. DYN-06 wymaga, żeby sekcje i podtytuły były współdzielone między Zał. 1 i Zał. 2 — oznacza to, że AppState.sections jest jedną listą referencji, a Zał. 1 i Zał. 2 mają tylko swoje dane wierszy, powiązane przez `sectionId`/`subsectionId`. Jeśli ta struktura zostanie zaprojektowana błędnie na początku, refaktor będzie kosztowny.

Referencyjny wzór protokołu (reference-protokol-extracted.md) jest dostępny w repo i precyzyjnie definiuje strukturę każdego załącznika: Zał. 1 ma wielopoziomowy header tabeli (4 wiersze nagłówka z colSpan/rowSpan w pdfmake), Zał. 2 ma 15 kolumn (landscape), Zał. 3 jest prosty (7 kolumn), Zał. 4 jest prosty (7 kolumn). Wzorzec referencyjny musi być bazą dla każdej decyzji o układzie tabeli.

**Primary recommendation:** Zbuduj AppState ze współdzieloną listą `sections` najpierw — to uniezależnia kolejność implementacji Zał. 1, 2, 3, 4. Każdy załącznik to osobna tablica `rows` per `subsectionId`. Wszystkie obliczenia to czyste funkcje w module `Calculator` bez side-effectów.

---

## Standard Stack

### Core (bez zmian — zdecydowane przed fazą)

| Technologia | Wersja | Cel | Dlaczego |
|-------------|--------|-----|----------|
| HTML5 + Vanilla JS | ES2022+ | Shell + logika | Zero konfiguracji, zero frameworka |
| Tailwind CSS CDN | `@tailwindcss/browser@4` | Styling formularzy | Jedyny CSS framework z browser CDN |
| pdfmake | 0.3.5 | PDF (Faza 3) | Zdecydowane — tu tylko AppState + formularze |

### Faza 1 nie wymaga dodatkowych bibliotek

Cała faza operuje na czystym JS. Nie dodawać żadnych CDN dependencies. pdfmake zostanie załadowany w Fazie 3.

### CDN load order (w `<head>` pliku index.html)

```html
<!-- Tailwind CSS v4 Play CDN — jedyne CDN w Fazie 1 -->
<script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>

<!-- pdfmake — ładuj tutaj od początku, używany w Fazie 3 -->
<script src="https://cdn.jsdelivr.net/npm/pdfmake@0.3.5/build/pdfmake.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/pdfmake@0.3.5/build/vfs_fonts.min.js"></script>
```

---

## Architecture Patterns

### Recommended Structure (single-file `<script>`)

```
index.html
└── <script>  (na końcu <body>)
    │
    ├── // === CONSTANTS ===
    │   └── PROTECTION_DB        const, freeze()'d
    │
    ├── // === STATE ===
    │   ├── AppState             jedyne źródło prawdy, persystowane
    │   └── UIState              stan UI, NIE persystowane
    │
    ├── // === EVENT BUS ===
    │   └── EventBus             emit(event, data), on(event, fn)
    │
    ├── // === CALCULATOR ===
    │   ├── roundTo3(val)        pomocnicza, używana wszędzie
    │   ├── calcSWZRow(row)      Id, Zsmax, verdict dla Zał. 1
    │   ├── calcIZOLRow(row)     verdict dla Zał. 2
    │   ├── calcRCDRow(row)      verdict dla Zał. 3
    │   └── calcUZIEMRow(row)    Rpo, verdict dla Zał. 4
    │
    ├── // === FORM RENDERER ===
    │   ├── renderSections()     wspólna hierarchia sekcji/podsekcji
    │   ├── renderAttachment1()  SWZ tabela
    │   ├── renderAttachment2()  IZOL tabela
    │   ├── renderAttachment3()  RCD tabela
    │   └── renderAttachment4()  UZIEM tabela
    │
    ├── // === FORM CONTROLLER ===
    │   └── bindAllEvents()      event delegation na kontenerach
    │
    ├── // === PERSISTENCE ===
    │   ├── saveToLocalStorage()
    │   └── loadFromLocalStorage()
    │
    └── // === INIT ===
        └── init()               bootstrap
```

### Pattern 1: AppState — Struktura danych (kluczowa decyzja)

**Kluczowe:** DYN-06 wymaga wspólnych sekcji między Zał. 1 i Zał. 2. Sekcje i podsekcje są jedną listą w AppState — nie duplikowane per załącznik.

```javascript
// Źródło: analiza wymagań DYN-06 + wzorzec referencyjny protokołu
const AppState = {
  // Wspólna hierarchia — używana przez Zał. 1 i Zał. 2
  sections: [
    {
      id: 'sec-1',
      title: 'Parter',
      subsections: [
        {
          id: 'sub-1-1',
          title: 'Pokój 1 - obwód gniazd wtykowych'
        }
      ]
    }
  ],

  // Stały pierwszy wiersz Zał. 1 (SWZ-01)
  swzFixedRow: {
    circuit: 'Zabezpieczenie główne',
    protType: '',
    protCurrent: null,
    baseType: 'C',
    baseCurrent: 25,
    Ia: null,  // z PROTECTION_DB lub ręczna korekta
    Usk: null,
    tw: '5',
    Zs: null,
    calculated: { Id: null, Zsmax: null, verdict: null }
  },

  // Dane wierszy Zał. 1 — per subsectionId
  attachment1: {
    rowsBySubsection: {
      'sub-1-1': [
        {
          id: 'row-uuid',
          circuit: '',
          protType: '',        // np. '40/4P/0,03' (RCD dodatkowe)
          protCurrent: null,
          baseType: 'B',       // B/C/D
          baseCurrent: 16,
          Ia: null,            // null = z PROTECTION_DB; wartość = ręczna korekta
          Usk: null,
          tw: '0.2',           // '0.2' | '0.4' | '5'
          Zs: null,
          calculated: { Id: null, Zsmax: null, verdict: null }
        }
      ]
    }
  },

  // Dane wierszy Zał. 2 — per subsectionId (te same klucze co Zał. 1)
  attachment2: {
    fixedRow: {
      id: 'izol-fixed',
      name: 'WLZ',
      wireType: '',
      phaseType: '3',   // '1' | '3'
      rp: { L1N: null, L2N: null, L3N: null, L1L2: null, L1L3: null,
            L2L3: null, L1PE: null, L2PE: null, L3PE: null, NPE: null },
      Rw: 1,
      calculated: { verdict: null }
    },
    rowsBySubsection: {
      'sub-1-1': [
        {
          id: 'row-uuid',
          name: '',
          wireType: '',
          phaseType: '1',   // '1' = 1-fazowy: aktywne L1N, L1PE, NPE
          rp: { L1N: null, L2N: null, L3N: null, L1L2: null, L1L3: null,
                L2L3: null, L1PE: null, L2PE: null, L3PE: null, NPE: null },
          Rw: 1,
          calculated: { verdict: null }
        }
      ]
    }
  },

  // Zał. 3 — własna lista wierszy (brak sekcji/podsekcji)
  attachment3: {
    rows: [
      {
        id: 'row-uuid',
        deviceType: '',
        testResult: 'TAK',  // 'TAK' | 'NIE'
        In: null,
        Iw: null,
        tw: null,
        tz: 300,
        calculated: { verdict: null }
      }
    ]
  },

  // Zał. 4 — własna lista wierszy (brak sekcji/podsekcji)
  attachment4: {
    rows: [
      {
        id: 'row-uuid',
        deviceName: '',
        Rp: null,
        Wk: null,
        Rw: 10,
        calculated: { Rpo: null, verdict: null }
      }
    ]
  }
};
```

### Pattern 2: PROTECTION_DB — Baza Zabezpieczeń (PROT-01, PROT-02)

**Wartości Ia potwierdzone normą PN-EN/IEC 60898-1:**
- Charakterystyka B: Ia = 5 × In (wyzwalanie natychmiastowe w zakresie 3–5×In; konwencjonalna wartość obliczeniowa = górna granica = 5×In)
- Charakterystyka C: Ia = 10 × In (zakres 5–10×In; wartość obliczeniowa = 10×In)
- Charakterystyka D: Ia = 20 × In (zakres 10–20×In; wartość obliczeniowa = 20×In)

**Uwaga:** Norma podaje zakresy (B: 3–5×In, C: 5–10×In, D: 10–20×In). W obliczeniach protokołu do Zsmax = Usk/Ia używa się górnej granicy zakresu (najbardziej niekorzystny przypadek = konserwatywna ocena). Ta wartość jest zgodna z wzorcem referencyjnym: B16 → Ia=80A, C25 → Ia=250A.

**Weryfikacja z wzorcem referencyjnym:**
- Zał. 1.1 linia 2: B16 → Ia=80A ✓ (5×16=80)
- Zał. 1.1 linia 1: C25 → Ia=250A ✓ (10×25=250)
- Zał. 1.2 linia 25: B10 → Ia=50A ✓ (5×10=50)

```javascript
// Źródło: norma PN-EN/IEC 60898-1 + weryfikacja z reference-protokol-extracted.md
const PROTECTION_DB = Object.freeze({
  B: { multiplier: 5,  ratings: [6, 8, 10, 13, 16, 20, 25, 32, 40, 50, 63] },
  C: { multiplier: 10, ratings: [6, 8, 10, 13, 16, 20, 25, 32, 40, 50, 63] },
  D: { multiplier: 20, ratings: [6, 8, 10, 13, 16, 20, 25, 32, 40, 50, 63] }
});

function getIa(type, rating) {
  const spec = PROTECTION_DB[type];
  if (!spec) return null;
  if (!spec.ratings.includes(Number(rating))) return null;
  return spec.multiplier * Number(rating);
}

// Użycie: getIa('B', 16) === 80, getIa('C', 25) === 250, getIa('D', 20) === 400
```

### Pattern 3: Calculator — Czyste Funkcje

```javascript
// Źródło: wymagania SWZ-03, SWZ-04, SWZ-05 + wzorzec referencyjny protokołu
function roundTo3(val) {
  return Math.round(val * 1000) / 1000;
}

function calcSWZRow(row) {
  const Ia = (row.Ia != null) ? row.Ia : getIa(row.baseType, row.baseCurrent);
  if (!Ia || !row.Usk || !row.Zs) return { Id: null, Zsmax: null, verdict: null };
  const Id     = roundTo3(row.Usk / row.Zs);
  const Zsmax  = roundTo3(row.Usk / Ia);
  const verdict = roundTo3(row.Zs) <= Zsmax ? 'POZYTYWNA' : 'NEGATYWNA';
  return { Id, Zsmax, verdict };
}

// IZOL-06: Ocena POZYTYWNA gdy WSZYSTKIE aktywne pola Rp >= Rw
function calcIZOLRow(row) {
  const activeFields = getActiveIZOLFields(row.phaseType);  // zwraca ['L1N','L1PE','NPE'] dla 1-faz
  const allMeasured = activeFields.every(f => row.rp[f] != null);
  if (!allMeasured) return { verdict: null };
  const allPass = activeFields.every(f => roundTo3(row.rp[f]) >= roundTo3(row.Rw));
  return { verdict: allPass ? 'POZYTYWNA' : 'NEGATYWNA' };
}

// RCD-03: Iw <= In AND tw <= tz
function calcRCDRow(row) {
  if (row.Iw == null || row.In == null || row.tw == null || row.tz == null) {
    return { verdict: null };
  }
  const pass = roundTo3(row.Iw) <= roundTo3(row.In) && roundTo3(row.tw) <= roundTo3(row.tz);
  return { verdict: pass ? 'POZYTYWNA' : 'NEGATYWNA' };
}

// UZIEM-02, UZIEM-04: Rpo = Rp × Wk; ocena Rpo <= Rw
function calcUZIEMRow(row) {
  if (row.Rp == null || row.Wk == null) return { Rpo: null, verdict: null };
  const Rpo = roundTo3(row.Rp * row.Wk);
  const verdict = Rpo <= roundTo3(row.Rw) ? 'POZYTYWNA' : 'NEGATYWNA';
  return { Rpo, verdict };
}
```

### Pattern 4: Aktywne pola dla Zał. 2 (IZOL-03, IZOL-04)

```javascript
// Źródło: wymagania IZOL-03, IZOL-04 + wzorzec referencyjny Zał. 2
const IZOL_FIELDS_1PHASE = ['L1N', 'L1PE', 'NPE'];  // dla 1-fazowych
const IZOL_FIELDS_3PHASE = ['L1N','L2N','L3N','L1L2','L1L3','L2L3','L1PE','L2PE','L3PE','NPE'];

function getActiveIZOLFields(phaseType) {
  return phaseType === '3' ? IZOL_FIELDS_3PHASE : IZOL_FIELDS_1PHASE;
}

// W renderze: pola nieaktywne dostają atrybut disabled + klasy CSS szarego tła
// W obliczeniach: pola nieaktywne są ignorowane
```

### Pattern 5: EventBus — Minimalna implementacja

```javascript
// Źródło: wzorzec z ARCHITECTURE.md — pełna implementacja ~30 linii
const EventBus = (() => {
  const listeners = {};
  return {
    on(event, fn) {
      (listeners[event] = listeners[event] || []).push(fn);
    },
    emit(event, data) {
      (listeners[event] || []).forEach(fn => fn(data));
    }
  };
})();
```

### Pattern 6: Event Delegation + data-atrybuty

```javascript
// Źródło: ARCHITECTURE.md Pattern 2
// Jeden listener na cały kontener załącznika — obsługuje wszystkie akcje
document.getElementById('attachment-1').addEventListener('input', (e) => {
  const field = e.target.dataset.field;
  if (!field) return;
  const rowId = e.target.dataset.rowId;
  const subId = e.target.dataset.subId;
  AppState.attachment1.rowsBySubsection[subId] =
    AppState.attachment1.rowsBySubsection[subId].map(row =>
      row.id === rowId ? { ...row, [field]: e.target.value } : row
    );
  EventBus.emit('swz:row:changed', { subId, rowId });
});

// HTML atrybuty: data-field="Zs" data-row-id="row-uuid" data-sub-id="sub-1-1"
```

### Pattern 7: Automatyczna renumeracja Lp. (DYN-05)

```javascript
// Źródło: wymagania DYN-05
// Lp. NIGDY nie jest przechowywane w AppState — zawsze obliczane w renderze
function getLpNumber(sectionList, subId, rowId) {
  let counter = 1;
  for (const section of sectionList) {
    for (const sub of section.subsections) {
      const rows = AppState.attachment1.rowsBySubsection[sub.id] || [];
      for (const row of rows) {
        if (sub.id === subId && row.id === rowId) return counter;
        counter++;
      }
    }
  }
  return counter;
}
// Lp. jest wyliczane przy każdym renderze — nie przy każdej zmianie danych.
```

### Anti-Patterns to Avoid

- **Stan per-załącznik z duplikowanymi sekcjami:** Nie przechowuj sekcji/podsekcji osobno dla Zał. 1 i Zał. 2 — DYN-06 wymaga wspólnego źródła. Zmiana nazwy sekcji w Zał. 1 musi natychmiast odzwierciedlić się w Zał. 2.
- **Lp. w AppState:** Nie przechowuj numerów porządkowych w danych — zawsze obliczaj w renderze. Przy usuniętych wierszach trzeba by przeliczyć wszystkie kolejne, co jest error-prone.
- **Inline event handlers w HTML:** Nie używaj `onclick="deleteRow('${id}')"` — używaj `data-action` + event delegation.
- **Obliczenia w DOM event handlerze:** Nie licz `Id = Usk / Zs` inline w event listener — deleguj do `Calculator.calcSWZRow()`.
- **`parseFloat` bez fallback:** Zawsze `parseFloat(val) || null` — nie `parseFloat(val) || 0`, bo 0 jest prawidłową wartością (Rp=0 to błąd izolacji, nie pusty input).

---

## Don't Hand-Roll

| Problem | Nie buduj | Użyj zamiast | Dlaczego |
|---------|-----------|--------------|----------|
| Precyzja float | własna logika zaokrąglania | `roundTo3()` — jedna funkcja, wszędzie | 0.1+0.2=0.30000000000000004; edge case przy Zs=Zsmax |
| UUID dla wierszy | własny licznik `rowCounter++` | `crypto.randomUUID()` (Web Crypto API) | Wbudowane w przeglądarki Chrome 92+, Firefox 95+, Safari 15.4+ — zero zależności, zerowe kolizje |
| Pub/Sub | bezpośrednie wywołania funkcji między modułami | EventBus (30 linii) | Eliminuje spaghetti zależności; Calculator nie musi znać FormRenderer |
| Walidacja zakresu Ia | własne reguły walidacji | PROTECTION_DB + ręczna korekta (PROT-03) | Wartości niestandardowe muszą być możliwe; walidacja to UI responsibility |

---

## Common Pitfalls

### Pitfall 1: Niepoprawny porządek ładowania sekcji w AppState

**What goes wrong:** Zał. 2 renderuje sekcje/podsekcje z innego miejsca niż Zał. 1 — zmiana nazwy sekcji w Zał. 1 nie odzwierciedla się w Zał. 2.

**Why it happens:** Deweloper implementuje Zał. 1 z `AppState.attachment1.sections` i Zał. 2 z `AppState.attachment2.sections` jako osobne tablice.

**How to avoid:** `AppState.sections` to JEDNA tablica — jedyne źródło sekcji. Oba załączniki iterują `AppState.sections` żeby zbudować hierarchię, a wiersz per sekcja czerpie z `AppState.attachment1.rowsBySubsection[subId]` lub `AppState.attachment2.rowsBySubsection[subId]`.

**Warning signs:** Zmiana nazwy sekcji w Zał. 1 nie aktualizuje widoku Zał. 2.

### Pitfall 2: `parseFloat` zwraca `NaN` zamiast `null` dla pustych pól

**What goes wrong:** Obliczenie `Id = Usk / Zs` produkuje `NaN` gdy `Zs` jest puste. `NaN` propaguje się przez wszystkie dalsze obliczenia bez błędu.

**Why it happens:** `parseFloat('')` zwraca `NaN`, nie `null`. `NaN <= 2.5` to `false` — ocena NEGATYWNA dla pustego wiersza.

**How to avoid:** Normalizuj przy wpisie: `const val = e.target.value.trim(); AppState...field = val === '' ? null : parseFloat(val);`. W Calculator: `if (!row.Usk || !row.Zs) return { ...nullResult }`.

**Warning signs:** Puste wiersze pokazują NEGATYWNA zamiast braku oceny.

### Pitfall 3: Float precision przy porównaniu Zs <= Zsmax

**What goes wrong:** Zs = 0.69 Ω, Zsmax = 0.690 Ω — wynik powinien być POZYTYWNA, ale JavaScript liczy `0.69 <= 0.6900000000000001` → może być `false`.

**Why it happens:** IEEE 754 binary float. Zdecydowane w STATE.md.

**How to avoid:** `roundTo3(row.Zs) <= roundTo3(calculated.Zsmax)`. Konsekwentne we wszystkich porównaniach.

**Warning signs:** Graniczny przypadek Zs=Zsmax daje NEGATYWNA.

### Pitfall 4: Nieprawidłowe mapowanie aktywnych pól IZOL dla obwodów 1-fazowych

**What goes wrong:** Dla obwodu 1-fazowego, aktywne są L-N, L-PE, N-PE. Ale w AppState przechowujemy L1-N, L1-PE, N-PE (nazwy kolumn 3-fazowe). Kod renderuje L2-N jako aktywne (błąd logiczny).

**Why it happens:** Wzorzec referencyjny Zał. 2 ma 10 kolumn z nazwami L1-N...N-PE. Dla 1-fazowych "L-N" to faktycznie "L1-N".

**How to avoid:** Dla obwodów 1-fazowych: `phaseType === '1'` → aktywne pola to `['L1N', 'L1PE', 'NPE']`. Pola L2N, L3N, L1L2, L1L3, L2L3, L2PE, L3PE są `disabled` i traktowane jako wartości null przy ocenie.

**Warning signs:** Ocena IZOL dla 1-fazowego bierze pod uwagę pustą kolumnę L2-N.

### Pitfall 5: Dodawanie wiersza do Zał. 2 bez sprawdzenia czy subsection istnieje w `rowsBySubsection`

**What goes wrong:** Użytkownik dodaje nową sekcję i od razu dodaje wiersz do Zał. 2. `AppState.attachment2.rowsBySubsection['sub-new']` jest `undefined` → TypeError.

**Why it happens:** `rowsBySubsection` jest inicjalizowane tylko dla istniejących subsections — nowe subsections nie mają automatycznie klucza.

**How to avoid:** Przy dodawaniu wiersza: `if (!AppState.attachment2.rowsBySubsection[subId]) AppState.attachment2.rowsBySubsection[subId] = [];`. Lub inicjalizuj przy tworzeniu subsection.

**Warning signs:** Błąd `Cannot read property 'push' of undefined` w konsoli.

### Pitfall 6: Ręczna korekta Ia nie jest persystowana osobno od `baseType`/`baseCurrent`

**What goes wrong:** Użytkownik wybiera B16 (Ia=80 auto), ręcznie koryguje na 70, później zmienia baseType na C — aplikacja powinna zapytać czy zachować ręczną wartość.

**Why it happens:** Brak rozróżnienia między "Ia z bazy" i "Ia ręcznie ustawiona".

**How to avoid:** Pole `Ia: null` oznacza "użyj z PROTECTION_DB". Pole `Ia: 70` oznacza "ręcznie ustawione — nie nadpisuj". Przy zmianie `baseType`/`baseCurrent`: jeśli `row.Ia !== null` (ręczne), zapytaj użytkownika (lub po prostu zachowaj — PROT-03 mówi "użytkownik może ręcznie skorygować", nie "automatycznie resetuj").

**Warning signs:** Po zmianie prądu znamionowego Ia zmienia się ignorując ręczną korektę.

---

## Code Examples

### Struktura sekcji w pdfmake (Zał. 1 — nagłówek wielorzędowy)

Na podstawie wzorca referencyjnego protokołu, nagłówek tabeli Zał. 1 ma 4 wiersze:
- Wiersz 1: "Lp.", "Obwód...", "Zabezpieczenia" (colSpan 4), "Usk", "Ia", "Id", "Zs", "Zs max", "tw", "Ocena"
- Wiersz 2: "dodatkowe" (colSpan 2), "podstawowe" (colSpan 2)
- Wiersz 3: "typ", "prąd", "typ", "prąd IΔn"
- Wiersz 4: "-", "[A]", "-", "[A]", "[V]", "[A]", "[A]", "[Ω]", "[Ω]", "[s]", "-"

Wiersz sekcji ("Parter") to pełny colSpan przez wszystkie kolumny.

```javascript
// Źródło: pdfmake docs + wzorzec reference-protokol-extracted.md
// Wiersz sekcji-nagłówka w body tabeli (np. "Parter")
function makeSectionRow(title, numCols) {
  const cells = Array(numCols).fill('');
  cells[0] = {
    text: title,
    bold: true,
    colSpan: numCols,
    fillColor: '#e5e7eb',  // Tailwind gray-200
    alignment: 'left'
  };
  return cells;
}

// Wiersz podsekcji ("Pokój 1 - obwód gniazd wtykowych")
function makeSubsectionRow(title, numCols) {
  const cells = Array(numCols).fill('');
  cells[0] = {
    text: title,
    italics: true,
    colSpan: numCols,
    alignment: 'left'
  };
  return cells;
}

// Budowanie body tabeli Zał. 1
function buildAttachment1Body(sections, rowsBySubsection, fixedRow) {
  const body = [];

  // Stały wiersz "Tablica rozdzielcza" (SWZ-01)
  body.push(makeTableRowSWZ(1, 'Tablica rozdzielcza - zabezpieczenie główne', fixedRow));

  let lpCounter = 2;
  for (const section of sections) {
    body.push(makeSectionRow(section.title, 12));  // 12 = liczba kolumn Zał. 1
    for (const sub of section.subsections) {
      body.push(makeSubsectionRow(sub.title, 12));
      const rows = rowsBySubsection[sub.id] || [];
      for (const row of rows) {
        body.push(makeTableRowSWZ(lpCounter++, sub.title, row));
      }
    }
  }
  return body;
}
```

### Formularz HTML z data-atrybutami (Zał. 1 — jeden wiersz)

```html
<!-- Szablon wiersza Zał. 1 (klonowany przez <template>) -->
<template id="swz-row-template">
  <tr data-row-id="" data-sub-id="">
    <td class="lp-cell text-center"><!-- wypełniane przez renderer --></td>
    <td><input type="text" data-field="circuit" class="w-full" placeholder="Nazwa obwodu"></td>
    <td><input type="text" data-field="protType" class="w-20" placeholder="typ"></td>
    <td><input type="number" data-field="protCurrent" class="w-16" step="0.01"></td>
    <td>
      <select data-field="baseType" class="w-16">
        <option value="B">B</option>
        <option value="C">C</option>
        <option value="D">D</option>
      </select>
    </td>
    <td>
      <select data-field="baseCurrent" class="w-16">
        <!-- Opcje generowane z PROTECTION_DB.B.ratings -->
      </select>
    </td>
    <td><input type="number" data-field="Usk" class="w-20" step="0.1" placeholder="230"></td>
    <td class="ia-cell text-right"><!-- auto z DB lub edytowalne --></td>
    <td class="id-cell text-right"><!-- wyliczone --></td>
    <td><input type="number" data-field="Zs" class="w-20" step="0.001"></td>
    <td class="zsmax-cell text-right"><!-- wyliczone --></td>
    <td>
      <select data-field="tw">
        <option value="0.2">0,2</option>
        <option value="0.4">0,4</option>
        <option value="5">5</option>
      </select>
    </td>
    <td class="verdict-cell text-center font-bold"><!-- POZYTYWNA/NEGATYWNA --></td>
    <td><button data-action="remove-row" class="text-red-500">Usuń</button></td>
  </tr>
</template>
```

### Inicjalizacja z crypto.randomUUID()

```javascript
// Źródło: MDN Web Crypto API — wbudowane w Chrome 92+, Firefox 95+, Safari 15.4+
function createNewSWZRow(subId) {
  return {
    id: crypto.randomUUID(),
    circuit: '',
    protType: '',
    protCurrent: null,
    baseType: 'B',
    baseCurrent: 16,
    Ia: null,      // null = auto z PROTECTION_DB
    Usk: 230,      // domyślne napięcie sieciowe
    tw: '0.2',
    Zs: null,
    calculated: { Id: null, Zsmax: null, verdict: null }
  };
}

function createNewSection() {
  return {
    id: crypto.randomUUID(),
    title: 'Nowa sekcja',
    subsections: []
  };
}
```

### Renderowanie aktywnych/nieaktywnych pól IZOL (IZOL-03)

```html
<!-- Wiersz Zał. 2 z dynamicznie aktywowanymi kolumnami -->
<template id="izol-row-template">
  <tr data-row-id="" data-sub-id="">
    <td class="lp-cell"></td>
    <td><input type="text" data-field="name" class="w-full"></td>
    <td><input type="text" data-field="wireType" class="w-24"></td>
    <td>
      <select data-field="phaseType" data-action="phase-change">
        <option value="1">1-fazowy</option>
        <option value="3">3-fazowy</option>
      </select>
    </td>
    <!-- Pola Rp — aktywowane/dezaktywowane zależnie od phaseType -->
    <td><input type="number" data-field="rp.L1N"  class="izol-field w-16" step="0.01"></td>
    <td><input type="number" data-field="rp.L2N"  class="izol-field w-16" step="0.01"></td>
    <td><input type="number" data-field="rp.L3N"  class="izol-field w-16" step="0.01"></td>
    <td><input type="number" data-field="rp.L1L2" class="izol-field w-16" step="0.01"></td>
    <td><input type="number" data-field="rp.L1L3" class="izol-field w-16" step="0.01"></td>
    <td><input type="number" data-field="rp.L2L3" class="izol-field w-16" step="0.01"></td>
    <td><input type="number" data-field="rp.L1PE" class="izol-field w-16" step="0.01"></td>
    <td><input type="number" data-field="rp.L2PE" class="izol-field w-16" step="0.01"></td>
    <td><input type="number" data-field="rp.L3PE" class="izol-field w-16" step="0.01"></td>
    <td><input type="number" data-field="rp.NPE"  class="izol-field w-16" step="0.01"></td>
    <td class="text-center">1</td><!-- Rw stałe -->
    <td class="verdict-cell font-bold"></td>
    <td><button data-action="remove-row">Usuń</button></td>
  </tr>
</template>
```

```javascript
// Aktywacja/dezaktywacja pól IZOL przy zmianie phaseType
function applyPhaseType(row, phaseType) {
  const activeFields = getActiveIZOLFields(phaseType);
  const allFields = IZOL_FIELDS_3PHASE;
  allFields.forEach(field => {
    const input = row.querySelector(`[data-field="rp.${field}"]`);
    const isActive = activeFields.includes(field);
    input.disabled = !isActive;
    input.classList.toggle('bg-gray-100', !isActive);
    input.classList.toggle('cursor-not-allowed', !isActive);
    if (!isActive) input.value = '';
  });
}
```

---

## State of the Art

| Stare podejście | Aktualne | Zmienione | Wpływ |
|-----------------|----------|-----------|-------|
| Globalny `var` na poziomie skryptu | Revealing Module Pattern (IIFE) | Zawsze — ES2022 | Enkapsulacja, brak kolizji nazw |
| `onclick=""` inline w HTML | `data-action` + event delegation | Zawsze | Zero re-bindowania przy dynamicznych wierszach |
| Losowy counter `rowId++` | `crypto.randomUUID()` | Chrome 92+ (2021) | Bez kolizji, bez globalnego stanu licznika |
| `parseInt`/`parseFloat` bez normalizacji | `parseFloat(val) || null` | Dobra praktyka | Brak `NaN` propagacji |
| Duplikowane sekcje per załącznik | Wspólne `AppState.sections` + `rowsBySubsection` map | Design decyzja | DYN-06 bez duplikacji |

**Deprecated/outdated:**
- `document.getElementById` per każde pole — zamiast tego `data-field` + event delegation
- Auto-save przy każdym keystroke — zamiast tego ręczny przycisk "Zapisz"

---

## Open Questions

1. **Ograniczenie liczby prądów znamionowych w PROTECTION_DB**
   - Co wiemy: Norma PN-EN 60898-1 obejmuje zakresy do 125A. Typowe zastosowania domowe: 6A–63A.
   - Co jest niejasne: Czy elektrycy potrzebują wartości 80A, 100A, 125A w bazie?
   - Rekomendacja: Zaimplementuj zakres 6A–63A (zdefiniowany w PROTECTION_DB.ratings). Pole Ia jest edytowalne (PROT-03) — wartości niestandardowe można wpisać ręcznie. Lista 6–63 pokrywa >95% przypadków z wzorca referencyjnego.

2. **Zachowanie po zmianie prądu znamionowego gdy Ia jest ręcznie ustawione**
   - Co wiemy: PROT-03 mówi że użytkownik może ręcznie skorygować Ia.
   - Co jest niejasne: Czy zmiana `baseCurrent` powinna resetować ręczną korektę?
   - Rekomendacja: NIE resetuj automatycznie — zachowaj ręczną wartość. Użytkownik musi jawnie wyczyścić pole Ia żeby wrócić do wartości z bazy. Prościej i bezpieczniej.

3. **Orientacja tabeli w formularzu ekranowym dla Zał. 2**
   - Co wiemy: Zał. 2 ma 15 kolumn — zbyt dużo dla portrait na małym ekranie.
   - Co jest niejasne: Jak renderować tabelę Zał. 2 na ekranie (nie w PDF — to Faza 3)?
   - Rekomendacja: Na ekranie użyj `overflow-x-auto` z `min-w-max` — tabela scrolluje się poziomo. Szerokość pól IZOL można zmniejszyć do `w-14` (56px). Orientacja landscape jest tylko kwestią Fazy 3 (pdfmake DDO).

4. **Klawisz Enter dodający nowy wiersz**
   - Co wiemy: Nie ma wymagania w REQ.
   - Co jest niejasne: Czy elektryk oczekuje że Enter w ostatnim polu wiersza dodaje nowy wiersz?
   - Rekomendacja: NIE implementować w Fazie 1 — dodaj przycisk "Dodaj wiersz". Enter-to-add to v2 UX.

---

## Struktura Wzorca Referencyjnego — Tabele

Wzorzec referencyjny (reference-protokol-extracted.md) potwierdza następujące struktury:

### Załącznik 1 — SWZ
- **Stały wiersz 1:** "Tablica rozdzielcza - zabezpieczenie główne" (full-width merged row)
- **Sekcje:** Parter, Piętro, Piwnica (full-width merged rows z pogrubioną czcionką)
- **Podsekcje:** "Pokój 1 - obwód gniazd wtykowych" (full-width merged rows kursywą)
- **Kolumny danych:** Lp, Obwód, typ-prot-dod, prąd-prot-dod, typ-prot-podst, prąd-IΔn, Usk, Ia, Id, Zs, Zsmax, tw, Ocena = **13 kolumn**
- **Header tabeli:** 4-wierszowy z colSpan (patrz opis powyżej)
- **Legenda:** pod tabelą, dwukolumnowa

### Załącznik 2 — Izolacja
- **Stały wiersz 1 — sekcja:** "Zasilanie" + wiersz WLZ (wszystkie 10 pól Rp aktywne — WLZ to 3-fazowe)
- **Kolumny:** Lp, Nazwa obwodu, Typ przewodu, Rp-L1N, Rp-L2N, Rp-L3N, Rp-L1L2, Rp-L1L3, Rp-L2L3, Rp-L1PE, Rp-L2PE, Rp-L3PE, Rp-NPE, Rw, Ocena = **15 kolumn**
- **Sekcje i podsekcje:** Współdzielone z Zał. 1 (DYN-06)
- **Dane 1-fazowe:** Tylko 3 z 10 pól Rp wypełnione — reszta pusta (z wzorca referencyjnego widać że komórki są po prostu puste, nie zaznaczone inaczej)

### Załącznik 3 — RCD
- **Bez sekcji/podsekcji** — tylko plain list wierszy
- **Kolumny:** Lp, Typ urządzenia, TEST, In [mA], Iw [mA], tw [ms], tz [ms], Ocena = **8 kolumn**
- **Domyślne:** tz = 300 ms

### Załącznik 4 — Uziemienie
- **Bez sekcji/podsekcji** — tylko plain list wierszy
- **Kolumny:** Lp, Nazwa urządzenia, Rp [Ω], Wk, Rpo [Ω], Rw [Ω], Ocena = **7 kolumn**
- **Domyślne:** Rw = 10 Ω
- **Legenda:** RE (nie Rp!) → Wartość pomierzona rezystancji uziemienia (UZIEM-06 mówi RE, ale AppState używa Rp jako klucza — w legendzie używaj RE)

---

## Sources

### Primary (HIGH confidence)
- `/Users/wojciecholszak/Desktop/VoltProtokół/reference-protokol-extracted.md` — wzorzec referencyjny protokołu; dokładna struktura wszystkich 4 załączników, wartości Ia zweryfikowane (B16=80, C25=250, B10=50)
- `/Users/wojciecholszak/Desktop/VoltProtokół/.planning/research/ARCHITECTURE.md` — architektura AppState, EventBus, Calculator, wzorce event delegation (badania 2026-02-23)
- `/Users/wojciecholszak/Desktop/VoltProtokół/.planning/research/STACK.md` — stack: pdfmake 0.3.5, Tailwind v4, vanilla JS (badania 2026-02-23)
- `/Users/wojciecholszak/Desktop/VoltProtokół/.planning/research/PITFALLS.md` — pułapki: float precision, html2canvas, localStorage (badania 2026-02-23)
- pdfmake docs — colSpan, rowSpan, headerRows, fillColor, layout: https://pdfmake.github.io/docs/0.1/document-definition-object/tables/

### Secondary (MEDIUM confidence)
- kanałelektryczny.pl — wartości Ia dla charakterystyk B/C/D: 5×In, 10×In, 20×In (potwierdzone normą IEC/EN 60898-1); https://kanalelektryczny.pl/budowa-i-zasada-dzialania-wylacznikow-nadpradowych/
- Wartości PROTECTION_DB zweryfikowane krzyżowo przez obliczenia z wzorca referencyjnego (B16→80, C25→250, B10→50) — HIGH confidence

### Tertiary (LOW confidence)
- Brak — żadne twierdzenia w tym dokumencie nie opierają się wyłącznie na niezweryfikowanych źródłach

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — zdecydowany w poprzednich badaniach, zweryfikowany z CDN
- AppState struktura: HIGH — wyprowadzona z wymagań DYN-06 + wzorca referencyjnego
- PROTECTION_DB wartości: HIGH — potwierdzone normą IEC 60898-1 + krzyżowa weryfikacja z wzorcem referencyjnym
- Calculator formulas: HIGH — bezpośrednio z wymagań SWZ-03/04/05, UZIEM-02/04, RCD-03, IZOL-06
- Struktura tabel: HIGH — bezpośrednio z reference-protokol-extracted.md
- pdfmake DDO (Faza 3): MEDIUM — to Faza 3, tu tylko informacyjnie; weryfikacja z oficjalnych docs

**Research date:** 2026-02-24
**Valid until:** 2026-03-24 (30 dni — stabilna domena)
