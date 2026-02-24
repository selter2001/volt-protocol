---
phase: 02-kompletnosc-normowa
verified: 2026-02-24T00:39:12Z
status: gaps_found
score: 8/9 must-haves verified
gaps:
  - truth: "Użytkownik może wpisać przyrządy pomiarowe (nazwa, nr seryjny, nr świadectwa wzorcowania) dla 4 typów pomiarów"
    status: partial
    reason: "Tabela FORM-05 ma tylko 2 edytowalne kolumny (nazwa, nr świadectwa wzorcowania) — brakuje kolumny 'Nr seryjny'. Pole serial istnieje w AppState.form.instruments[] ale nie jest renderowane ani obsługiwane przez handler."
    artifacts:
      - path: "index.html"
        issue: "renderFormTab() generuje tabelę FORM-05 z 3 kolumnami nagłówkowymi (linie 582-584), ale wiersze mają tylko 2 inputy: form-instr-{TYPE}-name i form-instr-{TYPE}-calibration. Brak inputa data-field='form-instr-{TYPE}-serial'."
    missing:
      - "Dodaj 4. kolumnę nagłówkową 'Nr seryjny' do tabeli FORM-05 w renderFormTab()"
      - "Dodaj input type=text data-field='form-instr-{TYPE}-serial' w każdym wierszu tabeli FORM-05"
      - "Rozszerz regex w handleInstrumentFieldChange() z /(name|calibration)/ na /(name|serial|calibration)/ (linia 1224)"
human_verification:
  - test: "Sprawdź czy wpisywanie danych w polach zakładki Protokół nie gubi fokusa (focus loss)"
    expected: "Po wpisaniu tekstu w pole 'Nazwa obiektu', kursor pozostaje w polu i można kontynuować pisanie bez klikania ponownie"
    why_human: "Anti-focus-loss wymaga interakcji użytkownika — nie można zweryfikować statyczną analizą kodu"
  - test: "Sprawdź reaktywność oceny końcowej: przejdź do Zał. 1, wpisz Zs=0.5 i Usk=230, wróć na Protokół"
    expected: "Komórka 'Badanie skuteczności ochrony SWZ' (Zał. 1) pokazuje POZYTYWNY w kolorze zielonym"
    why_human: "Cross-tab reactive update wymaga ręcznej interakcji z przeglądarką"
  - test: "Wpisz datę badania np. 2026-02-20, sprawdź orzeczenie"
    expected: "Sekcja 'Data następnego badania' wyświetla '20.02.2031 r.' lub po dokonaniu zmian"
    why_human: "Wymaga interakcji z input[type=date] i weryfikacji obliczenia +5 lat"
---

# Faza 02: Kompletność Normowa — Raport Weryfikacji

**Cel fazy:** Protokół jest kompletny zgodnie z PN-HD 60364-6 — zawiera dane ogólne, orzeczenie o zdatności i legendy oznaczeń
**Zweryfikowano:** 2026-02-24T00:39:12Z
**Status:** LUKI ZNALEZIONE
**Re-weryfikacja:** Nie — weryfikacja wstępna

---

## Osiągnięcie Celu

### Obserwowalne prawdy

| # | Prawda | Status | Dowód |
|---|--------|--------|-------|
| 1 | Zakładka "Protokół" jest widoczna jako pierwsza zakładka (tab 0) i domyślnie aktywna | ZWERYFIKOWANA | `data-tab="0" class="...border-blue-600 text-blue-600"` (linia 22), `UIState = { activeTab: 0 }` (linia 139), `renderTabs()` iteruje `i=0..4` (linia 374) |
| 2 | Użytkownik może wpisać numer protokołu (auto-generowany RRRR/MM/001, edytowalny) | ZWERYFIKOWANA | `generateProtocolNumber()` (linia 69), `AppState.form.protocolNumber = generateProtocolNumber()` w `init()` (linia 1821), input `data-field="form-protocolNumber"` (linia 515) |
| 3 | Użytkownik może wpisać dane obiektu (nazwa, adres, działka) | ZWERYFIKOWANA | 3 inputy z `data-field="form-objectName/Address/Parcel"` (linie 525, 529, 533), switch-case w `handleFormFieldChange()` (linie 1189-1197) |
| 4 | Użytkownik może wpisać dane wykonawcy (imię nazwisko, nr SEP eksploatacja, nr SEP dozór) | ZWERYFIKOWANA | 3 inputy z `data-field="form-executorName/SepE/SepD"` (linie 544, 548, 552), switch-case (linie 1198-1206) |
| 5 | Użytkownik może wpisać datę badania i temperaturę otoczenia | ZWERYFIKOWANA | `input[type=date]` z `data-field="form-testDate"` (linia 563), `input[type=number]` z `data-field="form-testTemp"` (linia 568), obsługa w handlerze (linie 1207-1214) |
| 6 | Użytkownik może wpisać przyrządy pomiarowe (nazwa, nr seryjny, nr świadectwa wzorcowania) dla 4 typów pomiarów | CZESCIOWO ZWERYFIKOWANA | Tabela FORM-05 ma 3 nagłówkowe kolumny (linie 582-584) ale tylko 2 edytowalne inputy per wiersz: `name` i `calibration`. Pole `serial` istnieje w `AppState.form.instruments[]` (linia 131) ale NIE jest renderowane w UI — brak inputa `data-field="form-instr-{TYPE}-serial"` |
| 7 | Tabela oceny końcowej generuje się automatycznie z wynikami per załącznik (POZYTYWNY/NEGATYWNY/—) | ZWERYFIKOWANA | `calcAttachmentVerdict()` skanuje attachment1-4 (linie 195-246), `calcFinalAssessment()` (linia 248), tabela z 6 wierszami i id `assess-swz/izol/rcd/uziem` (linie 619-663), `updateFinalAssessmentDisplay()` z guardem (linia 1758) |
| 8 | Orzeczenie o zdatności instalacji wyświetla się z automatyczną datą następnego badania (+5 lat) | ZWERYFIKOWANA | `calcNextTestDate()` dodaje 5 lat do `AppState.form.testDate` (linia 262), `calcOrzeczenie()` (linia 269), sekcja `#orzeczenie-text` i `#next-test-date` renderowana z warunkowym tekstem (linie 675-691) |
| 9 | Zmiana danych w załącznikach 1-4 automatycznie aktualizuje ocenę końcową i orzeczenie bez gubienia focusu | ZWERYFIKOWANA | EventBus.on('attachment1-4:changed', updateFinalAssessmentDisplay) (linie 1811-1815), targeted update z guardem `if (!document.getElementById('assess-swz')) return;` (linia 1758), `updateFinalAssessmentDisplay()` też wolany bezpośrednio z `handleSWZFieldChange/handleIZOLFieldChange/handleRCDFieldChange/handleUZIEMFieldChange` (linie 1507, 1535, 1611, 1661, 1713) |

**Wynik: 8/9 prawd zweryfikowanych (1 częściowo)**

---

### Wymagane artefakty

| Artefakt | Oczekiwane | Status | Szczegóły |
|----------|-----------|--------|-----------|
| `index.html` (AppState.form) | Obiekt form z polami: protocolNumber, objectName, objectAddress, objectParcel, executorName, executorSepE, executorSepD, testDate, testTemp, instruments[] | ZWERYFIKOWANY | Linie 120-136; `instruments[].serial` istnieje w stanie ale brakuje inputa w UI |
| `index.html` (renderFormTab) | Funkcja renderująca całą zawartość zakładki Protokół | ZWERYFIKOWANY | Linia 503-706, 203 linie kodu, substancjalny |
| `index.html` (handleFormFieldChange) | Handler zapisujący input użytkownika do AppState.form | ZWERYFIKOWANY | Linia 1184-1221, switch/case dla wszystkich 9 pól formularza |
| `index.html` (handleInstrumentFieldChange) | Handler dla pól instrumentów z regex pattern | CZĘŚCIOWO | Linia 1223-1232, regex `/(name|calibration)$/` — brak `serial` (linia 1224) |
| `index.html` (calcAttachmentVerdict + calcFinalAssessment) | Obliczanie oceny per załącznik | ZWERYFIKOWANY | Linie 195-257 |
| `index.html` (calcNextTestDate + calcOrzeczenie) | Data następnego badania i orzeczenie | ZWERYFIKOWANY | Linie 259-274 |
| `index.html` (updateFinalAssessmentDisplay + updateOrzeczenieDisplay) | Targeted DOM update dla oceny i orzeczenia | ZWERYFIKOWANY | Linie 1757-1795 |
| `index.html` (EventBus subscriptions) | Subskrypcje attachment:changed → updateFinalAssessmentDisplay | ZWERYFIKOWANY | Linie 1811-1815, 5 subskrypcji |

---

### Weryfikacja kluczowych połączeń

| Od | Do | Przez | Status | Szczegóły |
|----|-----|-------|--------|-----------|
| `renderFormTab()` | `AppState.form` | `const f = AppState.form` (linia 505) | PODŁĄCZONE | Każde pole renderowane przez `escapeHtml(f.field)` |
| `input[data-field^="form-"]` | `handleFormFieldChange()` | `app.addEventListener('input')` z `field.startsWith('form-')` (linia 1375) | PODŁĄCZONE | Delegacja na `#app`, obsługa też w `change` (linia 1434) |
| `handleFormFieldChange()` | `AppState.form` | switch/case | PODŁĄCZONE | Wszystkie 9 przypadków pokryte (linie 1186-1214) |
| `calcFinalAssessment()` | `AppState.attachment1/2/3/4` | `calcAttachmentVerdict()` | PODŁĄCZONE | Skanuje wszystkie wiersze, dynamiczne i stałe |
| `EventBus.on('attachment*:changed')` | `updateFinalAssessmentDisplay()` | EventBus.on | PODŁĄCZONE | 5 subskrypcji (linie 1811-1815) |
| `handleSWZ/IZOL/RCD/UZIEMFieldChange()` | `updateFinalAssessmentDisplay()` | bezpośrednie wywołanie | PODŁĄCZONE | 5 wywołań (linie 1507, 1535, 1611, 1661, 1713) |
| `renderFormTab()` przy tab switch | trigger | `if (UIState.activeTab === 0) renderFormTab()` (linia 1243) | PODŁĄCZONE | Pełny re-render gwarantuje świeżość tabeli oceny |

---

### Pokrycie wymagań

| Wymaganie | Status | Problem blokujący |
|----------|--------|-------------------|
| FORM-01: Numer protokołu auto-generowany RRRR/MM/NNN | SPEŁNIONE | — |
| FORM-02: Dane obiektu badanego (nazwa, adres, nr działki) | SPEŁNIONE | — |
| FORM-03: Dane wykonawcy (imię, SEP E, SEP D) | SPEŁNIONE | — |
| FORM-04: Warunki badań (data, temperatura) | SPEŁNIONE | — |
| FORM-05: Przyrządy pomiarowe (nazwa, **nr seryjny**, nr świadectwa) | ZABLOKOWANE | Brak inputa nr seryjny w UI — pole serial w AppState ale nie renderowane |
| FORM-06: Ocena końcowa z wynikami per załącznik | SPEŁNIONE | — |
| FORM-07: Orzeczenie z datą następnego badania (+5 lat) | SPEŁNIONE | — |

---

### Znalezione antypatterns

Brak znaczących antypatternów. Analiza `return null` (linie 48, 49, 77, 244, 261, 1463, 1465, 1569, 1571) — wszystkie są prawidłowymi wartownikami (`if (!input) return null`), nie stubami. Brak TODO/FIXME/placeholder. Brak pustych handlerów.

---

### Wymagana weryfikacja manualna

#### 1. Brak gubienia fokusa przy wpisywaniu

**Test:** Otwórz index.html, wejdź w zakładkę "Protokół", kliknij w pole "Nazwa obiektu" i zacznij wpisywać tekst.
**Oczekiwane:** Kursor pozostaje w polu przez cały czas wpisywania. Nie skacze na początek pola ani do innego elementu.
**Dlaczego manualnie:** Wymaga interakcji z przeglądarką — analiza statyczna potwierdza brak re-renderu przy input (delegacja + targeted update), ale zachowanie fokusa trzeba potwierdzić wizualnie.

#### 2. Reaktywność oceny końcowej cross-tab

**Test:** Przejdź do Zał. 1, wpisz Zs=0.5 i Usk=230 w wierszu, następnie wróć na zakładkę "Protokół".
**Oczekiwane:** Komórka "Badanie skuteczności ochrony SWZ" (Zał. 1) pokazuje "POZYTYWNY" w kolorze zielonym.
**Dlaczego manualnie:** Cross-tab update zależy od EventBus + targeted DOM update — weryfikacja końcowa wymaga przeglądarki.

#### 3. Obliczenie daty następnego badania

**Test:** Wpisz datę badania "2026-02-20", sprawdź sekcję "Orzeczenie".
**Oczekiwane:** "Data następnego badania: 20.02.2031 r. lub po dokonaniu zmian."
**Dlaczego manualnie:** `parseLocalDate()` i `calcNextTestDate()` są poprawne w kodzie, ale weryfikacja timezone-safe działania wymaga przeglądarki.

---

### Podsumowanie luk

**1 luka blokująca cel fazy:**

Tabela przyrządów pomiarowych (FORM-05) nie zawiera pola "Nr seryjny" w interfejsie użytkownika. REQUIREMENTS.md i cel fazy wyraźnie wymagają trzech kolumn edytowalnych: "nazwa, nr seryjny, nr świadectwa wzorcowania". Aktualnie tabela ma tylko dwie kolumny edytowalne (nazwa + nr świadectwa). Pole `serial` istnieje w `AppState.form.instruments[]` (linia 131), ale:
- Brak nagłówka kolumny "Nr seryjny" w tabeli (linie 582-584)
- Brak inputa `data-field="form-instr-{TYPE}-serial"` w wierszach (linie 592-595)
- Regex w `handleInstrumentFieldChange()` nie obsługuje `serial` (linia 1224: `/(name|calibration)$/`)

Naprawa jest minimalna: dodanie kolumny i inputa do `renderFormTab()` + rozszerzenie regex w handlerze.

---

*Zweryfikowano: 2026-02-24T00:39:12Z*
*Weryfikator: Claude (gsd-verifier)*
