# Phase 2: Kompletność Normowa - Research

**Researched:** 2026-02-24
**Domain:** Vanilla JS single-file SPA — formularz główny protokołu (metadane, przyrządy pomiarowe, ocena końcowa, orzeczenie)
**Confidence:** HIGH

---

## Summary

Faza 2 dodaje do istniejącej aplikacji formularz główny protokołu: pola metadanych (numer, obiekt, wykonawca, data, temperatura), tabelę przyrządów pomiarowych, automatyczną tabelę oceny końcowej i orzeczenie. Jest to praca wyłącznie na AppState + DOM — bez nowych bibliotek, bez zmian w architekturze.

Kluczowa techniczna decyzja tej fazy: nowa zakładka "Protokół" (lub sekcja nad zakładkami załączników) musi być widoczna jako osobna jednostka nawigacyjna, ale jednocześnie dane z tej zakładki (data badania) muszą być dostępne przy obliczaniu orzeczenia (+5 lat). Tabela oceny końcowej (FORM-06) odczytuje wyniki z AppState attachment1–4 i generuje podsumowanie — to jedyny punkt gdzie formularz główny "dotyka" danych z załączników.

Auto-generacja numeru protokołu (FORM-01) według wzorca `RRRR/MM/NNN` (np. `2026/02/001`) wymaga tylko `new Date()` + padding. Numer jest edytowalny po auto-generacji — użytkownik musi móc go zmienić (bo może mieć więcej niż jeden protokół w miesiącu, licznik `001` nie jest globalny). W wzorcu referencyjnym numer to `2026/02/001` — format jest jasny.

**Primary recommendation:** Dodaj nową zakładkę "Protokół" jako zakładkę nr 0 (przed Zał. 1–4), rozszerz AppState.form o wszystkie pola FORM-01–07, zbuduj renderer `renderFormTab()` analogicznie do istniejących rendererów, i podpnij obsługę pod istniejący EventBus + event delegation.

---

## Standard Stack

### Core (bez zmian)

| Technologia | Wersja | Cel | Dlaczego |
|-------------|--------|-----|----------|
| HTML5 + Vanilla JS | ES2022+ | Shell + logika | Zero konfiguracji — ta sama co Faza 1 |
| Tailwind CSS CDN | `@tailwindcss/browser@4` | Styling formularza | Ta sama co Faza 1 |
| pdfmake | 0.3.5 | PDF (Faza 3) | Załadowany od początku — tu tylko AppState |

### Faza 2 nie wymaga dodatkowych bibliotek

Wszystko to czysty JavaScript + DOM. Nie dodawać żadnych CDN dependencies.

Jedyna "nowa" technika: `new Date()` do auto-generacji numeru protokołu i obliczenia daty następnego badania (+5 lat) — wbudowane w JavaScript, zero zależności.

---

## Architecture Patterns

### Rozszerzenie AppState o dane formularza głównego

**Kluczowe:** Wszystkie pola FORM-01–07 muszą trafić do `AppState.form`. Formularz główny to dane — nie UI. Tak samo jak `AppState.sections` jest źródłem prawdy dla sekcji, `AppState.form` jest źródłem prawdy dla metadanych protokołu.

```javascript
// Dodać do AppState (między sections a attachment1):
const AppState = {
  // ... istniejące pola ...

  form: {
    // FORM-01: Numer protokołu (auto-generowany, edytowalny)
    protocolNumber: '',      // np. '2026/02/001' — generowany w init()

    // FORM-02: Dane obiektu
    objectName: '',          // np. 'Budynek mieszkalny'
    objectAddress: '',       // np. 'ul. Kowalska 5, 00-001 Warszawa'
    objectParcel: '',        // nr działki

    // FORM-03: Dane wykonawcy
    executorName: '',        // imię i nazwisko
    executorSepE: '',        // nr SEP eksploatacja, np. '415/E1/634/25'
    executorSepD: '',        // nr SEP dozór, np. '416/D1/634/25'

    // FORM-04: Warunki badań
    testDate: '',            // data badania, np. '2026-02-20' (input[type=date])
    testTemp: null,          // temperatura °C, np. 2

    // FORM-05: Przyrządy pomiarowe (4 wiersze — jeden na typ pomiaru)
    instruments: [
      { type: 'SWZ',  name: '', serial: '', calibration: '' },
      { type: 'RCD',  name: '', serial: '', calibration: '' },
      { type: 'IZOL', name: '', serial: '', calibration: '' },
      { type: 'UZIEM',name: '', serial: '', calibration: '' }
    ]

    // FORM-06: Ocena końcowa — NIE przechowywana, zawsze obliczana z AppState attachment1-4
    // FORM-07: Orzeczenie — NIE przechowywane, generowane dynamicznie z testDate + wyników
  }
};
```

**Uwaga FORM-06:** Tabela oceny jest obliczana w locie z danych załączników — nie persystowana oddzielnie. Funkcja `calcFinalAssessment()` skanuje `AppState.attachment1/2/3/4` i zwraca wyniki per załącznik.

**Uwaga FORM-07:** Orzeczenie jest funkcją `AppState.form.testDate` + wyników `calcFinalAssessment()`. Nie jest przechowywane — generowane przy renderze.

### Nowa zakładka "Protokół" jako zakładka nr 0

**Podejście:** Dodaj zakładkę o `data-tab="0"` jako pierwszą zakładkę. Rozszerz `UIState.activeTab` o wartość `0`. Zakładka ta pokazuje formularz główny.

Istniejące zakładki (1–4) pozostają bez zmian. Nowy kontener `<div id="attachment-0"></div>` (lub `<div id="form-tab"></div>`) renderuje formularz główny.

```html
<!-- W HTML — przed istniejącymi zakładkami -->
<div id="tabs" class="flex border-b border-gray-300 mb-4">
  <button data-tab="0" class="px-4 py-2 font-medium ...">Protokół</button>
  <button data-tab="1" class="px-4 py-2 font-medium ...">Zał. 1 - SWZ</button>
  <!-- ... pozostałe zakładki ... -->
</div>

<div id="attachment-0"></div>  <!-- lub id="form-tab" -->
<div id="attachment-1" class="hidden"></div>
<!-- ... -->
```

**Alternatywa:** Formularz główny jako sekcja nad zakładkami (zawsze widoczna, nie zakładka). Wady: wizualnie tłoczy strony. Zakładka jest czystsza — spójne z istniejącym UX.

### Obliczanie oceny końcowej (FORM-06)

Tabela oceny to 6 wierszy (wg wzorca referencyjnego):
1. Sprawdzenie dokumentacji instalacji — zawsze POZYTYWNY (stałe)
2. Oględziny instalacji elektrycznej — zawsze POZYTYWNY (stałe)
3. Badanie SWZ (Zał. 1) — wynik z attachment1
4. Badanie izolacji (Zał. 2) — wynik z attachment2
5. Badanie RCD (Zał. 3) — wynik z attachment3
6. Badanie uziemienia (Zał. 4) — wynik z attachment4

```javascript
// Źródło: wymagania FORM-06 + wzorzec referencyjny protokołu
function calcAttachmentVerdict(attachmentKey) {
  // Zbiera wszystkie oceny wierszy danego załącznika
  // Zwraca 'POZYTYWNY' | 'NEGATYWNY' | null (brak danych)
  let allVerdicts = [];

  if (attachmentKey === 'attachment1') {
    // Stały wiersz
    if (AppState.swzFixedRow.calculated.verdict) {
      allVerdicts.push(AppState.swzFixedRow.calculated.verdict);
    }
    // Wiersze sekcji
    for (const sec of AppState.sections) {
      for (const sub of sec.subsections) {
        const rows = AppState.attachment1.rowsBySubsection[sub.id] || [];
        for (const row of rows) {
          if (row.calculated.verdict) allVerdicts.push(row.calculated.verdict);
        }
      }
    }
  } else if (attachmentKey === 'attachment2') {
    if (AppState.attachment2.fixedRow.calculated.verdict) {
      allVerdicts.push(AppState.attachment2.fixedRow.calculated.verdict);
    }
    for (const sec of AppState.sections) {
      for (const sub of sec.subsections) {
        const rows = AppState.attachment2.rowsBySubsection[sub.id] || [];
        for (const row of rows) {
          if (row.calculated.verdict) allVerdicts.push(row.calculated.verdict);
        }
      }
    }
  } else if (attachmentKey === 'attachment3') {
    for (const row of AppState.attachment3.rows) {
      if (row.calculated.verdict) allVerdicts.push(row.calculated.verdict);
    }
  } else if (attachmentKey === 'attachment4') {
    for (const row of AppState.attachment4.rows) {
      if (row.calculated.verdict) allVerdicts.push(row.calculated.verdict);
    }
  }

  if (allVerdicts.length === 0) return null;
  return allVerdicts.every(v => v === 'POZYTYWNA') ? 'POZYTYWNY' : 'NEGATYWNY';
}

function calcFinalAssessment() {
  return {
    documentation: 'POZYTYWNY',  // zawsze
    inspection:    'POZYTYWNY',  // zawsze
    swz:  calcAttachmentVerdict('attachment1'),
    izol: calcAttachmentVerdict('attachment2'),
    rcd:  calcAttachmentVerdict('attachment3'),
    uziem: calcAttachmentVerdict('attachment4')
  };
}
```

**Uwaga:** "Sprawdzenie dokumentacji" i "Oględziny" są zawsze POZYTYWNY w wzorcu referencyjnym — nie ma dla nich danych pomiarowych. Można je zrobić statyczne (zawsze wyświetla POZYTYWNY).

### Orzeczenie (FORM-07)

Orzeczenie zależy od:
1. Wszystkich wyników z `calcFinalAssessment()` — muszą być POZYTYWNY
2. Daty badania `AppState.form.testDate` — do obliczenia daty następnego badania

```javascript
// Wzorzec z reference-protokol-extracted.md:
// "NA PODSTAWIE POZYTYWNYCH WYNIKÓW BADAŃ I POMIARÓW STWIERDZAM,
//  ŻE INSTALACJA ELEKTRYCZNA W OBIEKCIE NADAJE SIĘ DO EKSPLOATACJI."
// "Data następnego badania: 20.02.2031r. lub po dokonaniu zmian."

function calcNextTestDate(testDateStr) {
  // testDateStr: 'YYYY-MM-DD' z input[type=date]
  if (!testDateStr) return null;
  const d = new Date(testDateStr);
  if (isNaN(d)) return null;
  d.setFullYear(d.getFullYear() + 5);
  // Formatuj: DD.MM.RRRR
  const dd = String(d.getDate()).padStart(2, '0');
  const mm = String(d.getMonth() + 1).padStart(2, '0');
  const yyyy = d.getFullYear();
  return `${dd}.${mm}.${yyyy}`;
}

function calcOrzeczenie() {
  const assessment = calcFinalAssessment();
  const allPositive = Object.values(assessment).every(v => v === 'POZYTYWNY');
  const nextDate = calcNextTestDate(AppState.form.testDate);
  return { allPositive, nextDate };
}
```

### Auto-generacja numeru protokołu (FORM-01)

```javascript
// Format: RRRR/MM/NNN — zgodnie z wzorcem referencyjnym '2026/02/001'
// NNN: sekwencyjny numer — domyślnie '001' (nie ma globalnego licznika)
function generateProtocolNumber() {
  const now = new Date();
  const yyyy = now.getFullYear();
  const mm = String(now.getMonth() + 1).padStart(2, '0');
  return `${yyyy}/${mm}/001`;
}

// Używane w init():
AppState.form.protocolNumber = generateProtocolNumber();
// Użytkownik może edytować pole — nie nadpisuj po init()
```

### Przyrządy pomiarowe (FORM-05)

Wzorzec referencyjny pokazuje tabelę 3-kolumnową (wykonane pomiary, przyrząd, nr świadectwa) z 4 wierszami. Kolumna "Wykonane pomiary" to stały tekst — nie edytowane przez użytkownika. Edytowane: nazwa+nr przyrządu, nr świadectwa.

```javascript
// Stałe etykiety dla 4 typów pomiarów (wg wzorca referencyjnego):
const INSTRUMENT_LABELS = {
  SWZ:   'Pomiar skuteczności ochrony przeciwporażeniowej przez samoczynne wyłączenie zasilania',
  RCD:   'Pomiar wyłączników różnicowo-prądowych',
  IZOL:  'Pomiar rezystancji izolacji',
  UZIEM: 'Pomiar rezystancji uziomu'
};

// AppState.form.instruments[i].name + serial = np. 'MPI-520 nr 728693'
// AppState.form.instruments[i].calibration = np. 'Metronika 128/2026'
```

### Pattern: renderFormTab()

Analogicznie do `renderAttachment1()` — funkcja buduje innerHTML kontenera zakładki protokołu.

```javascript
function renderFormTab() {
  const container = document.getElementById('attachment-0');
  const f = AppState.form;
  const assessment = calcFinalAssessment();
  const orzeczenie = calcOrzeczenie();

  container.innerHTML = `
    <div class="space-y-6 max-w-4xl">

      <!-- FORM-01: Numer protokołu -->
      <div class="bg-white border rounded p-4">
        <h2 class="text-lg font-bold mb-3">Numer protokołu</h2>
        <input type="text"
               data-field="form-protocolNumber"
               value="${escapeHtml(f.protocolNumber)}"
               class="border border-gray-300 rounded px-2 py-1 w-48 text-sm"
               placeholder="RRRR/MM/NNN">
      </div>

      <!-- FORM-02: Obiekt badany -->
      <!-- FORM-03: Wykonawca -->
      <!-- FORM-04: Data + temperatura -->
      <!-- FORM-05: Przyrządy -->
      <!-- FORM-06: Ocena końcowa (auto) -->
      <!-- FORM-07: Orzeczenie (auto) -->

    </div>
  `;
}
```

**Uwaga:** `escapeHtml()` musi być zdefiniowane dla bezpieczeństwa wstrzykiwania. Prosta implementacja:
```javascript
function escapeHtml(str) {
  if (str == null) return '';
  return String(str)
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;');
}
```

### Pattern: obsługa pól FORM w event delegation

Rozszerz istniejący `bindAppEvents()` o obsługę pól formularza głównego:

```javascript
// Dodaj do istniejącego handlera input na document lub na kontenerze #attachment-0
document.getElementById('attachment-0').addEventListener('input', (e) => {
  const field = e.target.dataset.field;
  if (!field) return;
  handleFormFieldChange(field, e.target.value);
});

function handleFormFieldChange(field, value) {
  switch (field) {
    case 'form-protocolNumber': AppState.form.protocolNumber = value; break;
    case 'form-objectName':     AppState.form.objectName = value; break;
    case 'form-objectAddress':  AppState.form.objectAddress = value; break;
    case 'form-objectParcel':   AppState.form.objectParcel = value; break;
    case 'form-executorName':   AppState.form.executorName = value; break;
    case 'form-executorSepE':   AppState.form.executorSepE = value; break;
    case 'form-executorSepD':   AppState.form.executorSepD = value; break;
    case 'form-testDate':
      AppState.form.testDate = value;
      // Odśwież orzeczenie (data następnego badania zmienia się)
      updateOrzeczenieDisplay();
      break;
    case 'form-testTemp':
      AppState.form.testTemp = value === '' ? null : parseFloat(value);
      break;
    default:
      // Sprawdź czy to instrument field: 'form-instr-SWZ-name' etc.
      handleInstrumentFieldChange(field, value);
  }
}

function handleInstrumentFieldChange(field, value) {
  // field format: 'form-instr-{TYPE}-{PROP}' np. 'form-instr-SWZ-name'
  const match = field.match(/^form-instr-(\w+)-(name|serial|calibration)$/);
  if (!match) return;
  const [, type, prop] = match;
  const instr = AppState.form.instruments.find(i => i.type === type);
  if (!instr) return;
  instr[prop] = value;
}
```

### Kiedy odświeżać tabelę oceny końcowej (FORM-06)?

Tabela oceny końcowej powinna aktualizować się gdy:
- Zmienia się jakikolwiek verdict w attachment1–4 (każda zmiana wiersza)
- Użytkownik przełącza się na zakładkę "Protokół"

**Podejście:** EventBus + re-render tylko komponentu oceny (nie całego formularza).

```javascript
// Subskrybuj zdarzenia z załączników
EventBus.on('attachment1:changed', updateFinalAssessmentDisplay);
EventBus.on('attachment2:changed', updateFinalAssessmentDisplay);
EventBus.on('attachment3:changed', updateFinalAssessmentDisplay);
EventBus.on('attachment4:changed', updateFinalAssessmentDisplay);
EventBus.on('sections:changed',    updateFinalAssessmentDisplay);

function updateFinalAssessmentDisplay() {
  // Aktualizuj tylko komórki oceny — nie re-renderuj całego formularza
  // (unika gubienia focusu w polach tekstowych)
  const assessment = calcFinalAssessment();
  updateAssessmentCell('assess-swz',  assessment.swz);
  updateAssessmentCell('assess-izol', assessment.izol);
  updateAssessmentCell('assess-rcd',  assessment.rcd);
  updateAssessmentCell('assess-uziem',assessment.uziem);
  updateOrzeczenieDisplay();
}
```

### Anti-Patterns to Avoid

- **Re-render całego formularza na każdą zmianę danych z załączników:** Gdy użytkownik edytuje pole "Nazwa obiektu" i jednocześnie tabela oceny się odświeża przez pełny `renderFormTab()`, pole traci focus i kursor. Zamiast tego: targeted DOM update dla komórek oceny + orzeczenia, NIE pełny re-render.
- **Przechowywanie wyników FORM-06 w AppState:** Redundancja z attachment1–4. Zawsze obliczaj w locie z `calcFinalAssessment()`.
- **Auto-reset numeru protokołu:** Po `init()` numer jest ustawiony — nie nadpisuj przy każdym re-renderze. Sprawdź `if (!AppState.form.protocolNumber)` w init.
- **Walidacja blokująca:** Nie blokuj użytkownika komunikatami "pole wymagane" — pola mogą być puste w trakcie wypełniania. Ocena końcowa po prostu pokazuje `—` gdy brak danych.

---

## Don't Hand-Roll

| Problem | Nie buduj | Użyj zamiast | Dlaczego |
|---------|-----------|--------------|----------|
| Formatowanie daty +5 lat | własna logika dat | `new Date()` + `setFullYear()` + `padStart()` | Wbudowane w JS; obsługuje przestępne, zaokrąglenia miesięcy automatycznie |
| Sanitizacja HTML | własny regex | `escapeHtml()` (4 replace) | XSS przy renderze innerHTML z danych użytkownika |
| Generacja numeru protokołu | własny globalny licznik | `new Date()` + stały suffix `001` | Numer jest edytowalny — licznik globalny niepotrzebny |
| Targeted DOM update | re-render całego kontenera | `document.getElementById('assess-swz').textContent = ...` | Eliminuje gubienie focusu |

---

## Common Pitfalls

### Pitfall 1: Pełny re-render formularza kasuje focus pól tekstowych

**What goes wrong:** `renderFormTab()` zastępuje `innerHTML` całego kontenera. Gdy użytkownik pisze w polu "Nazwa obiektu" i jednocześnie odpalane jest `EventBus.on('attachment1:changed', renderFormTab)`, pole traci focus i kursor jest resetowany.

**Why it happens:** `innerHTML` rebuild usuwa i tworzy nowe elementy DOM — przeglądarka traci focus.

**How to avoid:** Dwie oddzielne funkcje:
1. `renderFormTab()` — pełny render, wywoływany tylko RAZ przy przełączeniu na zakładkę lub przy init
2. `updateFinalAssessmentDisplay()` — targeted update, wywoływany przez EventBus — aktualizuje tylko komórki oceny przez `textContent`

**Warning signs:** Wpisywanie w pole tekstowe przerywa się po każdym znaku.

### Pitfall 2: `new Date(testDateStr)` z input[type=date] daje różne wyniki w różnych strefach czasowych

**What goes wrong:** `new Date('2026-02-20')` w przeglądarce z polską strefą czasową (UTC+1) zwraca `2026-02-19T23:00:00Z` — data off-by-one przy formatowaniu dnia.

**Why it happens:** ISO 8601 date strings bez czasu są interpretowane jako UTC. `new Date('2026-02-20')` to północ UTC, co w strefie UTC+1 to poprzedni dzień.

**How to avoid:** Użyj `new Date(year, month-1, day)` z ręcznym parsowaniem stringa:
```javascript
function parseLocalDate(dateStr) {
  // dateStr: 'YYYY-MM-DD'
  const [yyyy, mm, dd] = dateStr.split('-').map(Number);
  return new Date(yyyy, mm - 1, dd);  // lokalna strefa, bez UTC konwersji
}

function calcNextTestDate(testDateStr) {
  if (!testDateStr) return null;
  const d = parseLocalDate(testDateStr);
  if (isNaN(d)) return null;
  d.setFullYear(d.getFullYear() + 5);
  const day = String(d.getDate()).padStart(2, '0');
  const month = String(d.getMonth() + 1).padStart(2, '0');
  return `${day}.${month}.${d.getFullYear()}`;
}
```

**Warning signs:** Data następnego badania jest o 1 dzień wcześniej niż oczekiwana.

### Pitfall 3: Instrumenty — stały `type` vs edytowalny `name+serial`

**What goes wrong:** Deweloper tworzy 4 edytowalne wiersze bez stałych etykiet — użytkownik musi wpisać "Pomiar SWZ" i "Pomiar RCD" ręcznie.

**Why it happens:** Nieczytelne wymaganie FORM-05 — "dla 4 typów pomiarów" oznacza stałe etykiety w kolumnie "Wykonane pomiary", edytowalne tylko pola przyrządu i świadectwa.

**How to avoid:** `AppState.form.instruments[i].type` jest stały (SWZ/RCD/IZOL/UZIEM) i wyświetlany jako stały tekst z `INSTRUMENT_LABELS[type]`. Użytkownik edytuje tylko `name`, `serial`, `calibration`.

**Warning signs:** Kolumna "Wykonane pomiary" jest pustym edytowalnym polem.

### Pitfall 4: Brak `escapeHtml()` przy renderze innerHTML z AppState

**What goes wrong:** Użytkownik wpisuje `<script>alert(1)</script>` jako nazwę obiektu. Przy `innerHTML = '...' + f.objectName + '...'` — XSS (choć w lokalnej aplikacji ryzyko niskie, to zepsuje rendering).

**Why it happens:** `innerHTML` interpretuje tagi HTML.

**How to avoid:** Zawsze `escapeHtml(f.objectName)` przy budowaniu HTML stringa. Lub użyj `textContent` dla pól tylko-odczyt.

**Warning signs:** Wpisanie `<b>test</b>` w pole tekstowe pogrubia tekst w interfejsie.

### Pitfall 5: Zakładka "Protokół" pokazuje przestarzałe dane oceny

**What goes wrong:** Użytkownik wypełnia Zał. 1 (verdict POZYTYWNA), potem wchodzi na zakładkę Protokół — tabela oceny pokazuje `—` zamiast POZYTYWNY.

**Why it happens:** `renderFormTab()` jest wywoływane tylko raz (init). EventBus nie triggeruje re-renderu formularza.

**How to avoid:** Przy kliknięciu w zakładkę "Protokół" zawsze wywołaj `updateFinalAssessmentDisplay()`. Alternatywnie: przy każdym przełączeniu zakładek wykonaj `renderFormTab()` (pełny re-render jest OK jeśli to zakładka "Protokół" — bo nie ma aktywnych pól z focusem).

**Warning signs:** Ocena końcowa nie aktualizuje się bez wejścia i wyjścia z zakładki.

---

## Code Examples

### Struktura HTML tabeli przyrządów (wzorzec z reference-protokol-extracted.md)

```html
<!-- Tabela FORM-05: Przyrządy pomiarowe — 3 kolumny, 4 wiersze -->
<table class="w-full border-collapse text-sm">
  <thead>
    <tr class="bg-gray-100">
      <th class="border px-2 py-1 text-left">Wykonane pomiary</th>
      <th class="border px-2 py-1 text-left">Przyrządy pomiarowe</th>
      <th class="border px-2 py-1 text-left">Nr świadectwa wzorcowania</th>
    </tr>
  </thead>
  <tbody>
    <!-- Renderowane dla każdego AppState.form.instruments[i] -->
    <tr>
      <td class="border px-2 py-1 text-sm text-gray-700">
        Pomiar skuteczności ochrony przeciwporażeniowej...
      </td>
      <td class="border px-2 py-1">
        <input type="text"
               data-field="form-instr-SWZ-name"
               class="w-full border-0 text-sm"
               placeholder="np. MPI-520 nr 728693">
      </td>
      <td class="border px-2 py-1">
        <input type="text"
               data-field="form-instr-SWZ-calibration"
               class="w-full border-0 text-sm"
               placeholder="np. Metronika 128/2026">
      </td>
    </tr>
    <!-- ... kolejne 3 wiersze ... -->
  </tbody>
</table>
```

### Struktura HTML tabeli oceny końcowej (FORM-06)

```html
<!-- Tabela FORM-06: Ocena wykonanych badań — 3 kolumny, 6 wierszy -->
<table class="w-full border-collapse text-sm mt-4">
  <thead>
    <tr class="bg-gray-100">
      <th class="border px-2 py-1 text-left">Wykonane badania odbiorcze</th>
      <th class="border px-2 py-1 text-center">Załączniki</th>
      <th class="border px-2 py-1 text-center">Ogólny wynik</th>
    </tr>
  </thead>
  <tbody>
    <!-- Wiersze statyczne (zawsze POZYTYWNY) -->
    <tr>
      <td class="border px-2 py-1">Sprawdzenie dokumentacji instalacji</td>
      <td class="border px-2 py-1 text-center">-</td>
      <td class="border px-2 py-1 text-center font-bold text-green-700">POZYTYWNY</td>
    </tr>
    <tr>
      <td class="border px-2 py-1">Oględziny instalacji elektrycznej</td>
      <td class="border px-2 py-1 text-center">-</td>
      <td class="border px-2 py-1 text-center font-bold text-green-700">POZYTYWNY</td>
    </tr>
    <!-- Wiersze auto-obliczane (targeted DOM update przez id) -->
    <tr>
      <td class="border px-2 py-1">
        Badanie skuteczności ochrony przeciwporażeniowej...
      </td>
      <td class="border px-2 py-1 text-center text-xs text-gray-600">Zał. 1</td>
      <td id="assess-swz" class="border px-2 py-1 text-center font-bold">—</td>
    </tr>
    <tr>
      <td class="border px-2 py-1">Badanie stanu rezystancji izolacji...</td>
      <td class="border px-2 py-1 text-center text-xs text-gray-600">Zał. 2</td>
      <td id="assess-izol" class="border px-2 py-1 text-center font-bold">—</td>
    </tr>
    <tr>
      <td class="border px-2 py-1">Badanie skuteczności ochrony dodatkowej (RCD)...</td>
      <td class="border px-2 py-1 text-center text-xs text-gray-600">Zał. 3</td>
      <td id="assess-rcd" class="border px-2 py-1 text-center font-bold">—</td>
    </tr>
    <tr>
      <td class="border px-2 py-1">Badanie rezystancji uziemienia...</td>
      <td class="border px-2 py-1 text-center text-xs text-gray-600">Zał. 4</td>
      <td id="assess-uziem" class="border px-2 py-1 text-center font-bold">—</td>
    </tr>
  </tbody>
</table>
```

### Targeted update komórek oceny

```javascript
function updateAssessmentCell(id, verdict) {
  const cell = document.getElementById(id);
  if (!cell) return;
  cell.textContent = verdict || '—';
  cell.classList.remove('text-green-700', 'text-red-700');
  if (verdict === 'POZYTYWNY') cell.classList.add('text-green-700');
  if (verdict === 'NEGATYWNY') cell.classList.add('text-red-700');
}

function updateOrzeczenieDisplay() {
  const orz = calcOrzeczenie();
  const textEl = document.getElementById('orzeczenie-text');
  const dateEl = document.getElementById('next-test-date');
  if (!textEl || !dateEl) return;

  if (orz.allPositive) {
    textEl.textContent =
      'NA PODSTAWIE POZYTYWNYCH WYNIKÓW BADAŃ I POMIARÓW STWIERDZAM, ' +
      'ŻE INSTALACJA ELEKTRYCZNA W OBIEKCIE NADAJE SIĘ DO EKSPLOATACJI.';
    textEl.className = 'font-bold text-green-700 text-sm';
  } else {
    textEl.textContent =
      'NA PODSTAWIE NEGATYWNYCH WYNIKÓW BADAŃ I POMIARÓW STWIERDZAM, ' +
      'ŻE INSTALACJA ELEKTRYCZNA W OBIEKCIE NIE NADAJE SIĘ DO EKSPLOATACJI.';
    textEl.className = 'font-bold text-red-700 text-sm';
  }

  dateEl.textContent = orz.nextDate
    ? `Data następnego badania: ${orz.nextDate}r. lub po dokonaniu zmian.`
    : 'Data następnego badania: (uzupełnij datę badania)';
}
```

---

## Struktura Wzorca Referencyjnego — Formularz Główny

Z `reference-protokol-extracted.md` potwierdzone sekcje formularza głównego:

### Sekcja 1: Numer protokołu
- Tytuł: "PROTOKÓŁ KONTROLNO POMIAROWY NR 2026/02/001"
- Format numeru: `RRRR/MM/NNN` — rok/miesiąc/numer_sekwencyjny (zerowanie co miesiąc)

### Sekcja 2-4: Dane obiektu, wykonawcy, warunki badań
- "Obiekt badany: Budynek mieszkalny, dz. [działka], adres [adres]"
- "Wykonawca: Protokół oraz badania wykonał oraz zatwierdził [imię nazwisko]. Świadectwa Kwalifikacyjne SEP na stanowisku eksploatacji nr [nr_e] oraz na stanowisku dozoru [nr_d]."
- "Badania wykonano w dniu [data]"
- "Temperatura otoczenia w dniu badań: [T]°C."

### Sekcja 5 (sekcja 3 w docs): Przyrządy pomiarowe
Tabela 3-kolumnowa (bez jawnych nagłówków w oryginale — kolumny: opis pomiaru | nazwa+nr przyrządu | nr świadectwa):
- Pomiar SWZ → MPI-520 nr 728693 → Metronika 128/2026
- Pomiar RCD → MPI-520 nr 728693 → Metronika 128/2026
- Pomiar izolacji → MPI-520 nr 728693 → Metronika 128/2026
- Pomiar uziomu → MPI-520 nr 728693 → Metronika 128/2026

### Sekcja 6: Ocena wykonanych badań odbiorczych (6 wierszy)
| Badanie | Załączniki | Ogólny wynik |
|---------|-----------|-------------|
| Sprawdzenie dokumentacji instalacji | - | POZYTYWNY |
| Oględziny instalacji elektrycznej | - | POZYTYWNY |
| Badanie skuteczności SWZ zakończone na pomiarze nr 54 | Załączniki nr 1.1, 1.2, 1.3, 1.4 | POZYTYWNY |
| Badanie stanu rezystancji izolacji zakończone na pomiarze nr 54 | Załączniki nr 2.1, 2.2, 2.3, 2.4 | POZYTYWNY |
| Badanie skuteczności ochrony dodatkowej (RCD) zakończone na pomiarze nr 1 | Załącznik nr 3 | POZYTYWNY |
| Badanie rezystancji uziemienia zakończone na pomiarze nr 1 | Załącznik nr 4 | POZYTYWNY |

**Uwaga o "zakończone na pomiarze nr X":** W wzorcu referencyjnym jest "zakończone na pomiarze nr 54" — to liczba pomiarów w załączniku. Dla FORM-06 można tę część pominąć lub zastąpić automatyczną liczbą wierszy (opcja dodatkowa, niekrytyczna).

### Sekcja 7-8: Orzeczenie + data następnego badania
- Stały tekst orzeczenia (zależy od wyniku)
- "Data następnego badania: 20.02.2031r. lub po dokonaniu zmian."
- Miejsce na podpis: "\_\_\_\_\_\_ \_\_\_\_\_\_" + "wykonawca miejscowość i data"

---

## State of the Art

| Stare podejście | Aktualne | Wpływ |
|-----------------|----------|-------|
| Pełny re-render przy każdej zmianie | Targeted DOM update dla komórek auto-obliczanych | Brak gubienia focusu w polach tekstowych |
| `new Date(dateString)` bezpośrednio | `new Date(year, month-1, day)` z parsowaniem | Poprawna data w każdej strefie czasowej |
| `innerHTML += ...` w pętli | Budowanie stringa HTML, jeden `innerHTML =` na końcu | Jeden reflow zamiast wielu |

---

## Open Questions

1. **Zakładka "Protokół" jako tab 0 czy sekcja powyżej zakładek?**
   - Co wiemy: UI ma zakładki 1–4 dla załączników. Formularz główny to odrębna sekcja.
   - Co jest niejasne: Czy elektryk oczekuje zakładki "Protokół" na początku, czy stałej sekcji nad zakładkami?
   - Rekomendacja: Nowa zakładka "Protokół" jako pierwsza (tab 0) — spójne z istniejącym UX, łatwe do implementacji. Sekcja powyżej zakładek komplikuje scrollowanie na małych ekranach.

2. **Kolumna "zakończone na pomiarze nr X" w tabeli oceny**
   - Co wiemy: Wzorzec referencyjny zawiera "zakończone na pomiarze nr 54" — automatycznie policzone wiersze.
   - Co jest niejasne: Czy FORM-06 wymaga tej frazy? Wymaganie mówi tylko "generuje się automatycznie z wynikami per załącznik".
   - Rekomendacja: Pomiń "zakończone na pomiarze nr X" — to dodatkowa złożoność niewymagana przez FORM-06. Wystarczy wynik POZYTYWNY/NEGATYWNY.

3. **Podpis w orzeczeniu — pole tekstowe czy statyczny placeholder?**
   - Co wiemy: Wzorzec ma linię "wykonawca miejscowość i data" z podkreślnikiem.
   - Co jest niejasne: Czy aplikacja ma renderować pole podpisu (niemożliwe — to PDF), czy tylko placeholder w formularzu?
   - Rekomendacja: Wyświetl statyczny tekst "\_\_\_\_\_ \_\_\_\_\_\_" + "wykonawca miejscowość i data" — pdfmake obsłuży to prawidłowo jako element dokumentu w Fazie 3. W formularzu ekranowym to tylko informacja wizualna.

4. **Co pokazać gdy brak danych do oceny (żaden attachment nie ma wierszy z verdyktami)?**
   - Co wiemy: `calcFinalAssessment()` zwróci null dla wszystkich 4 badań.
   - Co jest niejasne: Czy orzeczenie powinno wtedy pokazać "BRAK DANYCH" czy po prostu być puste?
   - Rekomendacja: Gdy wszystkie verdykty są null — wyświetl `—` w komórkach oceny i tekst "(uzupełnij dane pomiarowe)" w miejscu orzeczenia. Nie blokuj, nie waliduj.

---

## Sources

### Primary (HIGH confidence)
- `/Users/wojciecholszak/Desktop/VoltProtokół/reference-protokol-extracted.md` — wzorzec referencyjny protokołu; struktura sekcji 1–8 formularza głównego, format numeru protokołu, tabela przyrządów, ocena końcowa, orzeczenie, data następnego badania (linie 1–67)
- `/Users/wojciecholszak/Desktop/VoltProtokół/index.html` — istniejąca implementacja (1383 linie); AppState, EventBus, renderery, event delegation — wzorce do rozszerzenia
- `/Users/wojciecholszak/Desktop/VoltProtokół/.planning/phases/01-formularze-i-obliczenia/01-RESEARCH.md` — architektura AppState, EventBus, wzorce renderera
- `/Users/wojciecholszak/Desktop/VoltProtokół/.planning/REQUIREMENTS.md` — definicje FORM-01–07

### Secondary (MEDIUM confidence)
- MDN Web Docs — `Date` API: `new Date(year, month-1, day)`, `setFullYear()`, `padStart()` — standardowe, dobrze znane
- Wzorzec timezone safe date parsing — `new Date('YYYY-MM-DD')` vs `new Date(y, m, d)` — znany problem, potwierdzony w MDN

### Tertiary (LOW confidence)
- Brak — żadne twierdzenia nie opierają się wyłącznie na niezweryfikowanych źródłach

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — ta sama co Faza 1, zweryfikowana
- AppState.form struktura: HIGH — bezpośrednio z wymagań FORM-01–07 + wzorca referencyjnego
- calcFinalAssessment logika: HIGH — bezpośrednio z wymagania FORM-06 + wzorca referencyjnego (6 wierszy, 4 auto-obliczane)
- calcNextTestDate (+5 lat): HIGH — z wzorca (badanie 20.02.2026, następne 20.02.2031)
- Timezone pitfall: HIGH — znany problem MDN
- UI: zakładka vs sekcja powyżej: MEDIUM — decyzja UX, obie opcje prawidłowe technicznie

**Research date:** 2026-02-24
**Valid until:** 2026-03-24 (30 dni — stabilna domena)
