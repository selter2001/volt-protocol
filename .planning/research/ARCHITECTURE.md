# Architecture Research

**Domain:** Single-file HTML web app — dynamic forms, real-time calculations, PDF export
**Researched:** 2026-02-23
**Confidence:** HIGH (core patterns), MEDIUM (PDF export specifics)

## Standard Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        UI LAYER (DOM)                            │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐   │
│  │  FormHeader  │  │  Assessment  │  │  AttachmentTabs      │   │
│  │  (dane ogól.)│  │  (orzeczenie)│  │  (Zał. 1-4)          │   │
│  └──────┬───────┘  └──────┬───────┘  └──────────┬───────────┘   │
│         │                 │                     │               │
├─────────┴─────────────────┴─────────────────────┴───────────────┤
│                      EVENT BUS (Pub/Sub)                         │
│         field:change → calc:run → state:update → dom:render      │
├─────────────────────────────────────────────────────────────────┤
│                     STATE LAYER (Single Source)                  │
│                                                                  │
│  ┌───────────────┐  ┌──────────────────┐  ┌──────────────────┐  │
│  │  AppState     │  │  ProtectionDB    │  │  UIState         │  │
│  │  (form data)  │  │  (B/C/D + Ia)    │  │  (active tab...) │  │
│  └───────────────┘  └──────────────────┘  └──────────────────┘  │
├─────────────────────────────────────────────────────────────────┤
│                    SERVICES LAYER                                │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐   │
│  │  Calculator  │  │  Persistence │  │  PDFExporter         │   │
│  │  (formulas)  │  │  (localStorage│  │  (jsPDF+AutoTable)   │   │
│  └──────────────┘  └──────────────┘  └──────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Typical Implementation |
|-----------|----------------|------------------------|
| `AppState` | Centralny obiekt przechowujący wszystkie dane formularza | Plain JS object, odczytywany/modyfikowany przez wszystkie moduły |
| `ProtectionDB` | Stała baza zabezpieczeń B/C/D z wartościami Ia; lookup po type+rating | Zagnieżdżony obiekt const, funkcja `getIa(type, rating)` |
| `UIState` | Stan interfejsu (aktywna zakładka, zwinięte sekcje) | Osobny obiekt, niezrzucany do localStorage |
| `EventBus` | Luźne powiązanie między modułami przez Pub/Sub | Minimalne 30 linii: `on(event, fn)`, `emit(event, data)` |
| `Calculator` | Obliczenia Załącznika 1 (Id, Zsmax, ocena) i Załącznika 4 (Rpo, ocena) | Czyste funkcje: dane wejściowe → wynik, zero side-effects |
| `FormRenderer` | Renderowanie dynamicznych wierszy/sekcji/podsekcji | Funkcje `renderRow(data)`, `renderSection(data)` zwracające HTML string |
| `FormController` | Obsługa zdarzeń DOM (dodaj wiersz, usuń wiersz, zmień wartość) | Event delegation na `document` lub kontenerach sekcji |
| `Persistence` | `save()` / `load()` do/z localStorage; serializacja AppState | JSON.stringify/parse, klucz `voltprotokol_v1` |
| `PDFExporter` | Budowa dokumentu PDF z jsPDF + AutoTable; paginacja | Iteruje AppState → buduje tabele → `doc.save()` |

## Recommended Project Structure

Projekt to jeden plik HTML. Kod JS jest wewnątrz `<script>` na końcu `<body>`, podzielony na sekcje komentarzami. Struktura logiczna (nie plikowa):

```
index.html
├── <head>
│   └── CDN links (Tailwind, jsPDF, jsPDF-AutoTable)
├── <body>
│   ├── #app                         — główny kontener widoku formularza
│   │   ├── #form-header             — sekcja danych ogólnych
│   │   ├── #assessment              — sekcja orzeczenia
│   │   └── #attachments             — zakładki załączników 1-4
│   │       ├── #attachment-1        — SWZ (uziemienie)
│   │       ├── #attachment-2        — Rezystancja izolacji
│   │       ├── #attachment-3        — RCD
│   │       └── #attachment-4        — Uziemienie
│   └── <script>
│       │
│       ├── // === CONSTANTS ===
│       │   └── PROTECTION_DB        — baza B/C/D + Ia
│       │
│       ├── // === STATE ===
│       │   ├── AppState             — wszystkie dane formularza
│       │   └── UIState              — stan UI
│       │
│       ├── // === EVENT BUS ===
│       │   └── EventBus             — pub/sub (emit, on)
│       │
│       ├── // === CALCULATOR ===
│       │   ├── calcAttachment1Row() — Id = Usk/Zs; Zsmax = Usk/Ia; ocena
│       │   └── calcAttachment4Row() — Rpo = Rp × Wk; ocena
│       │
│       ├── // === FORM RENDERER ===
│       │   ├── renderHeaderForm()
│       │   ├── renderAssessmentForm()
│       │   ├── renderAttachment1()
│       │   ├── renderAttachment2()
│       │   ├── renderAttachment3()
│       │   └── renderAttachment4()
│       │
│       ├── // === FORM CONTROLLER ===
│       │   ├── bindHeaderEvents()
│       │   ├── bindAttachmentEvents()  — event delegation, add/remove rows
│       │   └── bindGlobalButtons()     — save, load, export PDF
│       │
│       ├── // === PERSISTENCE ===
│       │   ├── saveToLocalStorage()
│       │   └── loadFromLocalStorage()
│       │
│       ├── // === PDF EXPORTER ===
│       │   ├── buildPDFHeader()
│       │   ├── buildPDFAssessment()
│       │   ├── buildAttachment1Table()
│       │   ├── buildAttachment2Table()
│       │   ├── buildAttachment3Table()
│       │   ├── buildAttachment4Table()
│       │   └── exportPDF()             — entry point
│       │
│       └── // === INIT ===
│           └── init()               — bootstrap, bind events, render
```

### Structure Rationale

- **Sekcje komentarzowe `// === NAME ===`:** Zastępują pliki — każda sekcja jest samowystarczalna, łatwa do zwinięcia w edytorze (code folding). Jest to standardowa praktyka dla dużych single-file aplikacji.
- **CONSTANTS na początku:** Baza zabezpieczeń jest niemodyfikowalna i używana przez Calculator oraz FormRenderer — musi być dostępna od początku.
- **STATE przed EventBus:** EventBus subskrybuje zmiany stanu — stan musi istnieć pierwszy.
- **CALCULATOR bez zależności DOM:** Czyste funkcje numeryczne łatwo testować w konsoli. Nie dotyką DOM. Nie wywołują EventBus.
- **PDF EXPORTER na końcu:** Zależy od wszystkich pozostałych modułów (odczytuje AppState, wywołuje buildX funkcje) — buduje jako ostatni, wywołany przez guzik.

## Architectural Patterns

### Pattern 1: Centralny AppState + Event-Driven Updates

**What:** Jeden obiekt `AppState` jest jedynym źródłem prawdy. Każda zmiana w formularzu zapisuje wartość do AppState, emituje zdarzenie przez EventBus, a dopiero potem Calculator i DOM-updater reagują.

**When to use:** Zawsze, gdy wiele sekcji (obliczenia, podsumowanie, ocena) musi reagować na zmiany w innych sekcjach.

**Trade-offs:** Przewidywalny przepływ danych, łatwy debug. Minusem jest nieznaczny narzut przy każdym keystroke — dla formularzy z <500 polami nieistotny.

**Example:**
```javascript
// W FormController (event delegation):
document.getElementById('attachments').addEventListener('input', (e) => {
  if (!e.target.dataset.field) return;
  const { rowId, field } = e.target.dataset;
  AppState.attachment1.rows[rowId][field] = e.target.value;
  EventBus.emit('attachment1:row:changed', { rowId });
});

// W Calculator (subscriber):
EventBus.on('attachment1:row:changed', ({ rowId }) => {
  const row = AppState.attachment1.rows[rowId];
  const result = calcAttachment1Row(row);
  AppState.attachment1.rows[rowId].calculated = result;
  EventBus.emit('attachment1:row:calculated', { rowId, result });
});

// W FormRenderer (subscriber):
EventBus.on('attachment1:row:calculated', ({ rowId, result }) => {
  updateRowDOM(rowId, result); // aktualizuj tylko komórki wynikowe
});
```

### Pattern 2: Event Delegation dla Dynamicznych Wierszy

**What:** Zamiast bindować listenery na każdy wiersz (którego jeszcze może nie być), jeden listener siedzi na rodzicielskim kontenerze tabeli i przechwytuje zdarzenia przez `e.target.closest('[data-action]')`.

**When to use:** Zawsze przy dynamicznie dodawanych/usuwanych wierszach. Wiersze mogą być dodawane i usuwane bez rebindowania eventów.

**Trade-offs:** Jeden listener zamiast N listenerów — lepsza wydajność, prostszy kod. Wymaga konsekwentnego używania `data-action` i `data-row-id` atrybutów w HTML.

**Example:**
```javascript
document.getElementById('attachment-1').addEventListener('click', (e) => {
  const btn = e.target.closest('[data-action]');
  if (!btn) return;
  const { action, sectionId } = btn.dataset;
  if (action === 'add-row') addRowToSection(sectionId);
  if (action === 'remove-row') removeRow(btn.dataset.rowId);
});
```

### Pattern 3: Revealing Module Pattern (IIFE) dla Enkapsulacji

**What:** Każda logiczna sekcja kodu jest owinięta w IIFE zwracające publiczne API. Zmienne wewnętrzne są prywatne. Do reszty kodu trafia tylko to, co potrzebne.

**When to use:** Dla Persistence, Calculator, PDFExporter — modułów z wewnętrznym stanem lub pomocniczymi funkcjami, które nie powinny zaśmiecać globalnej przestrzeni.

**Trade-offs:** Enkapsulacja bez bundlera, czytelna granica modułu. Minusem jest verbose boilerplate IIFE — ale w single-file to najlepsza dostępna opcja.

**Example:**
```javascript
const Calculator = (() => {
  // private
  function _getIa(type, rating) {
    return PROTECTION_DB[type][rating]?.Ia ?? null;
  }

  // public API
  function calcAttachment1Row(row) {
    const Ia = _getIa(row.protectionType, row.protectionRating);
    if (!Ia || !row.Usk || !row.Zs) return { Id: null, Zsmax: null, verdict: null };
    const Id = row.Usk / row.Zs;
    const Zsmax = row.Usk / Ia;
    return { Id, Zsmax, verdict: row.Zs <= Zsmax ? 'POZYTYWNA' : 'NEGATYWNA' };
  }

  function calcAttachment4Row(row) {
    const Rpo = row.Rp * row.Wk;
    return { Rpo, verdict: Rpo <= row.Rw ? 'POZYTYWNA' : 'NEGATYWNA' };
  }

  return { calcAttachment1Row, calcAttachment4Row };
})();
```

### Pattern 4: PDF-only DOM dla Eksportu (Hidden Print Container)

**What:** Zamiast eksportować interaktywny formularz HTML do PDF (html2canvas na całym body — problematyczne ze stylami Tailwind, inputami, itp.), PDFExporter buduje dokument jsPDF programatycznie z danych AppState, używając jsPDF-AutoTable do tabel.

**When to use:** Zawsze dla tego projektu. Podejście html2canvas działa słabo z Tailwind (klasy utility to nie print CSS), z inputami (wartości w `value`, nie w DOM text), i z paginacją (wymaga precyzyjnych `page-break-*` CSS).

**Trade-offs:** Pełna kontrola nad wyglądem PDF (czcionki, marginesy, kolory, nagłówki na każdej stronie), niezawodna paginacja. Wymaga napisania logiki budowania PDF osobno od renderowania formularza — około 100-200 linii kodu buildPDF.

## Data Flow

### Przepływ Danych: Zmiana Pola w Wierszu

```
[User types in input field]
          ↓
[FormController: input event listener (delegation)]
          ↓
[AppState update: rows[rowId][field] = value]
          ↓
[EventBus.emit('row:changed', { attachmentId, sectionId, rowId })]
          ↓
    ┌─────┴──────┐
    ↓            ↓
[Calculator]  [UIState updates (dirty flag)]
calcRowResult()
    ↓
[AppState update: rows[rowId].calculated = result]
    ↓
[EventBus.emit('row:calculated', result)]
    ↓
[FormRenderer: updateRowCells(rowId, result)]
[Minimal DOM update — tylko komórki wynikowe]
```

### Przepływ Danych: Eksport PDF

```
[User clicks "Eksportuj PDF"]
          ↓
[PDFExporter.exportPDF()]
          ↓
[Reads AppState (all data, already calculated)]
          ↓
[jsPDF: new doc, set fonts, margins]
          ↓
[buildPDFHeader(doc, AppState.header)]    — strona 1: dane ogólne
          ↓
[buildPDFAssessment(doc, AppState.assessment)]
          ↓
[buildAttachment1Tables(doc, AppState.attachment1)]
  └─ forEach section → autoTable(doc, { head, body, ... })
  └─ AutoTable handles page breaks automatically
  └─ showHead: 'everyPage' for headers on each page
          ↓
[buildAttachment2Tables, 3, 4...]
          ↓
[doc.save('protokol.pdf')]
```

### Przepływ Danych: Zapis/Odczyt localStorage

```
ZAPIS (na życzenie, przycisk "Zapisz"):
[User clicks "Zapisz"]
    ↓
[Persistence.save()]
    ↓
[JSON.stringify(AppState)] → localStorage['voltprotokol_v1']
    ↓
[UIState.showSaveNotification()]

ODCZYT (na życzenie, przycisk "Wczytaj"):
[User clicks "Wczytaj"]
    ↓
[Persistence.load()]
    ↓
[JSON.parse(localStorage['voltprotokol_v1'])]
    ↓
[Merge into AppState (głęboka kopia)]
    ↓
[EventBus.emit('state:loaded')]
    ↓
[FormRenderer.rerenderAll() — pełny re-render]
```

### Stan Zarządzany (co trafia do AppState)

```javascript
AppState = {
  header: {
    date, temperature, protocolNumber, executor,
    instruments: [...],
    location, client
  },
  assessment: {
    verdict,          // 'POZYTYWNA' | 'NEGATYWNA'
    nextInspectionDate,
    comments
  },
  attachment1: {      // SWZ
    sections: [
      {
        id, title,    // np. "Parter"
        subsections: [
          {
            id, title,  // np. "Tablica rozdzielcza"
            rows: [
              {
                id, circuit, protectionType, protectionRating,
                Ia,           // auto z DB lub ręczna korekta
                tw, Usk, Zs,
                calculated: { Id, Zsmax, verdict }  // computed
              }
            ]
          }
        ]
      }
    ]
  },
  attachment2: { ... },
  attachment3: { ... },
  attachment4: { ... }
}
```

## Scaling Considerations

Projekt to narzędzie dla jednego użytkownika w przeglądarce — skalowanie do wielu użytkowników nie dotyczy. Relevant są tylko limity przeglądarki:

| Concern | Realny limit | Podejście |
|---------|--------------|-----------|
| Rozmiar localStorage | 5-10 MB per origin | AppState serialized to JSON — protokół z 200 wierszami to ~50 KB, bezpieczne |
| DOM performance (dużo wierszy) | >500 wierszy tabeli zaczyna lagować | Limit 15 wierszy na sekcję z auto-podziałem na nowy załącznik (1.1, 1.2) — problem nie wystąpi |
| jsPDF canvas limit | Max ~32767px wysokość canvas dla html2canvas | Nie dotyczy — używamy jsPDF programatycznie, nie html2canvas |
| AutoTable paginacja | Automatyczna, sprawdzona przy setkach wierszy | showHead: 'everyPage', pageBreak: 'auto' |

### Build Order (kolejność implementacji, zależności)

Komponenty muszą być budowane od dołu stosu — każdy zależy od niższych warstw:

```
1. CONSTANTS (PROTECTION_DB)       — zero dependencies
        ↓
2. AppState + UIState              — depends on: nothing
        ↓
3. EventBus                        — depends on: nothing
        ↓
4. Calculator                      — depends on: PROTECTION_DB, AppState
        ↓
5. FormRenderer (szkielet HTML)    — depends on: AppState, UIState
        ↓
6. FormController (event binding)  — depends on: AppState, EventBus, Calculator, FormRenderer
        ↓
7. Persistence                     — depends on: AppState, FormRenderer (rerenderAll)
        ↓
8. PDFExporter                     — depends on: AppState (read-only), jsPDF, AutoTable CDN
        ↓
9. init()                          — bootstraps all above, renders initial state
```

**Implication dla roadmapy:** Fazy powinny odpowiadać poziomom tego stosu — np. Faza 1 kończy się na działającym FormRenderer + FormController (widoczny formularz), Faza 2 dodaje Calculator + PDFExporter, Faza 3 dodaje Persistence + polish.

## Anti-Patterns

### Anti-Pattern 1: html2canvas na Całym Dokumencie

**What people do:** `html2pdf().from(document.body).save()` — renderują cały formularz HTML jako obraz PDF.

**Why it's wrong:** (1) Tailwind utility classes to nie print CSS — układ sypie się w różnych rozmiarach. (2) Wartości `<input>` nie pojawiają się w html2canvas (to DOM attribute `value`, nie text content). (3) Paginacja jest zawodna dla długich tabel — znany bug w html2pdf.js issue #200 i #227. (4) PDF jest rastrowy — nie można zaznaczać tekstu.

**Do this instead:** Buduj PDF programatycznie z jsPDF + AutoTable, czytając dane z AppState (nie z DOM). Pełna kontrola, niezawodna paginacja, selektywny tekst.

### Anti-Pattern 2: Inline Event Handlers w Dynamicznie Generowanym HTML

**What people do:** `<button onclick="removeRow(${rowId})">` wewnątrz template string przy renderowaniu wiersza.

**Why it's wrong:** (1) Przy każdym ponownym renderze wierszy (np. po load z localStorage) handler wskazuje na row ID z zamkniętego closure — prowadzi do błędów. (2) Trudny do testowania i debugowania. (3) Narusza separation of concerns — logika JS w HTML stringu.

**Do this instead:** Event delegation na kontenerze sekcji + `data-action="remove-row"` i `data-row-id="${rowId}"` jako atrybuty HTML. Jeden listener obsługuje wszystkie wiersze, przeszłe i przyszłe.

### Anti-Pattern 3: Bezpośrednia Modyfikacja DOM bez AppState

**What people do:** `document.getElementById('row-3-result').textContent = calcResult` — aktualizują DOM bez zapisywania do AppState.

**Why it's wrong:** Po załadowaniu danych z localStorage i re-renderze, te wartości przepadają. PDF Exporter czyta AppState, nie DOM — PDF będzie pusty lub nieaktualny.

**Do this instead:** Zawsze: zmiana wartości → AppState → EventBus → DOM update. DOM jest projekcją stanu, nigdy jego źródłem.

### Anti-Pattern 4: Globalne Zmienne na Poziomie Script

**What people do:** `var currentRow = null; var sectionCount = 0;` na poziomie globalnym skryptu — setki mutable globals.

**Why it's wrong:** W 2000+ linii pliku JS kolizje nazw stają się nieuniknione. Debugowanie nieoczekiwanych zmian stanu jest koszmarem.

**Do this instead:** Revealing Module Pattern (IIFE) dla każdego modułu. Wszystkie wewnętrzne zmienne są prywatne. Tylko eksportowane API widoczne zewnętrznie.

### Anti-Pattern 5: Auto-Save do localStorage na Każdym Keystroke

**What people do:** `AppState.on('change', () => localStorage.setItem(...))` — zapis przy każdej zmianie pola.

**Why it's wrong:** (1) JSON.stringify całego AppState przy każdym naciśnięciu klawisza to 50-200 KB operacji na sekundę. (2) Użytkownik traci kontrolę — nie może "cofnąć" i wczytać poprzedniej wersji protokołu. (3) Wymagania projektu mówią wprost: "ręczny, na życzenie użytkownika".

**Do this instead:** Przycisk "Zapisz" + ewentualne `beforeunload` ostrzeżenie jeśli są niezapisane zmiany.

## Integration Points

### External Services (CDN)

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| Tailwind CSS CDN | `<script src="https://cdn.tailwindcss.com">` w `<head>` | Tylko do UI — nie używany w PDF export |
| jsPDF CDN | `<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.x.x/jspdf.umd.min.js">` | Globalny `window.jsPDF` |
| jsPDF-AutoTable CDN | `<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.x.x/jspdf.plugin.autotable.min.js">` | Rozszerza `jsPDF.prototype` — musi być załadowany po jsPDF |

**Ważne:** jsPDF-AutoTable musi być załadowany PO jsPDF (rozszerza jego prototyp). Kolejność `<script>` tagów ma znaczenie.

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| FormController ↔ AppState | Bezpośrednia mutacja + EventBus emit | FormController jest jedynym miejscem, które mutuje AppState |
| Calculator ↔ AppState | Read AppState, update `calculated` subkeys, emit event | Calculator nie modyfikuje pól wejściowych — tylko pola wynikowe |
| PDFExporter ↔ AppState | Read-only | PDF Exporter nigdy nie modyfikuje AppState |
| Persistence ↔ AppState | Save: read-only serialize. Load: full replace + rerenderAll | Po load: EventBus.emit('state:loaded') wyzwala pełny re-render |
| FormRenderer ↔ UIState | Read UIState do renderowania (np. aktywna zakładka) | UIState nie jest persystowany do localStorage |

## Sources

- [jsPDF-AutoTable — GitHub, simonbengtsson](https://github.com/simonbengtsson/jsPDF-AutoTable) — HIGH confidence (official repo)
- [html2pdf.js — Client-side HTML-to-PDF](https://ekoopmans.github.io/html2pdf.js/) — HIGH confidence (official docs)
- [Build a state management system with vanilla JavaScript — CSS-Tricks](https://css-tricks.com/build-a-state-management-system-with-vanilla-javascript/) — MEDIUM confidence (community)
- [How to Build Single-File JavaScript Applications Without Build Tools — BSWEN](https://docs.bswen.com/blog/2026-02-21-single-file-javascript-apps/) — MEDIUM confidence (2026-02-21)
- [Mastering Modules and Modular Design Patterns in Vanilla JavaScript — ProCodeBase](https://procodebase.com/article/mastering-modules-and-modular-design-patterns-in-vanilla-javascript) — MEDIUM confidence (community)
- [Module Pattern — patterns.dev](https://www.patterns.dev/vanilla/module-pattern/) — HIGH confidence (authoritative patterns reference)
- [Page Break Issue #200 — html2pdf.js GitHub](https://github.com/eKoopmans/html2pdf.js/issues/200) — HIGH confidence (known limitation, issue tracker)
- [Event-Based Architectures in JavaScript — freeCodeCamp](https://www.freecodecamp.org/news/event-based-architectures-in-javascript-a-handbook-for-devs/) — MEDIUM confidence (community)

---
*Architecture research for: single-file HTML app z dynamicznymi formularzami i eksportem PDF*
*Researched: 2026-02-23*
