---
phase: 03-eksport-i-dane
verified: 2026-02-24T08:08:09Z
status: human_needed
score: 8/8 must-haves verified
re_verification: false
gaps: []
human_verification:
  - test: "Kliknij Eksportuj PDF — sprawdź czy polskie znaki (ą, ę, ś, ź, ć, ó, ł, ń, ż) wyświetlają się poprawnie w pobranym PDF"
    expected: "Polskie litery renderują się bez zastępowania znakami zastępczymi ani kwadratami. Roboto z vfs_fonts.min.js pokrywa Latin Extended-A — wymaga potwierdzenia przez CDN dostępny online."
    why_human: "Nie można zweryfikować renderowania fontu bez uruchomienia aplikacji w przeglądarce z dostępem do CDN pdfmake."
  - test: "Kliknij Eksportuj PDF — sprawdź czy Załącznik 2 otwiera się w orientacji landscape, a pozostałe strony są portrait"
    expected: "Strony Zał. 2 są poziome (landscape). Zał. 1, 3, 4 i formularz główny są pionowe (portrait)."
    why_human: "Orientacja strony wymaga weryfikacji w faktycznie wygenerowanym pliku PDF — kod jest poprawny (stack z pageOrientation: 'landscape'), ale efekt visual wymaga oka człowieka."
  - test: "Otwórz stronę, wypełnij dane, kliknij Zapisz. Odśwież stronę (Ctrl+R). Kliknij Wczytaj. Sprawdź czy dane są przywrócone we wszystkich zakładkach."
    expected: "Toast 'Dane zapisane.' po zapisie. Po odświeżeniu formularz pusty. Po Wczytaj: toast 'Dane wczytane.', formularz z danymi, ocena końcowa przeliczona."
    why_human: "Interakcja z localStorage wymaga działającej przeglądarki — nie można symulować sesji bez uruchomienia aplikacji."
  - test: "Kliknij Eksport JSON — pobierz plik. Odśwież stronę. Kliknij Import JSON — wybierz pobrany plik. Sprawdź czy dane przywróciły się poprawnie."
    expected: "Plik voltprotokol-*.json pobiera się poprawnie. Po imporcie toast 'Wczytano z pliku.' i pełne przywrócenie stanu we wszystkich zakładkach."
    why_human: "Import przez FileReader + dialog wyboru pliku wymaga interakcji z przeglądarką."
---

# Phase 3: Eksport i Dane — Raport Weryfikacji

**Cel fazy:** Elektryk może wygenerować profesjonalny plik PDF do pobrania, zapisać i wczytać stan formularza
**Zweryfikowano:** 2026-02-24T08:08:09Z
**Status:** human_needed
**Re-weryfikacja:** Nie — weryfikacja inicjalna

## Osiągnięcie celu

### Prawdy obserwowalne

| #  | Prawda                                                                                  | Status      | Dowód                                                                                   |
|----|-----------------------------------------------------------------------------------------|-------------|-----------------------------------------------------------------------------------------|
| 1  | Kliknięcie "Eksportuj PDF" pobiera plik .pdf na dysk                                    | ZWERYFIKOWANO | `exportPDF()` linia 2441: `pdfMake.createPdf(dd).download(filename)`. Przycisk podpięty w init() linia 2497. |
| 2  | PDF zawiera formularz główny (numer, obiekt, wykonawca, przyrządy, ocena, orzeczenie)   | ZWERYFIKOWANO | `buildFormMainContent()` linie 1934-2066: tytuł, obiekt, wykonawca, tabela przyrządów (headerRows:1), ocena końcowa (calcAttachmentVerdict), orzeczenie (calcOrzeczenie), linie podpisu. |
| 3  | PDF zawiera Zał. 1 (SWZ) z 4-wierszowym nagłówkiem i auto-paginacją                    | ZWERYFIKOWANO | `buildAttachment1Content()` linie 2069-2189: 4 wiersze nagłówka (colSpan/rowSpan), `headerRows: 4`, 13 kolumn. |
| 4  | PDF zawiera Zał. 2 (izolacja) w orientacji landscape                                    | ZWERYFIKOWANO (technicznie) | `buildDocDefinition()` linia 2434: `{ stack: buildAttachment2Content(), pageOrientation: 'landscape', pageBreak: 'before' }`. Wymaga weryfikacji ludzkiej. |
| 5  | PDF zawiera Zał. 3 (RCD) i Zał. 4 (uziemienie) jako tabele portrait                   | ZWERYFIKOWANO | Zał. 3: stack z `pageOrientation: 'portrait'` linia 2435. Zał. 4: `pageBreak: 'before'` na tytule linia 2369. |
| 6  | Polskie znaki wyświetlają się poprawnie w PDF                                           | WYMAGA HUMAN | `defaultStyle: { font: 'Roboto', fontSize: 9 }` linia 2427. vfs_fonts.min.js z CDN załadowany linia 13. Samo renderowanie wymaga weryfikacji w przeglądarce. |
| 7  | PDF jest czarno-biały z numeracją stron w stopce                                       | ZWERYFIKOWANO | Footer linia 2428-2430: `currentPage + ' / ' + pageCount`. Jedyny kolor tekstu to `#9ca3af` dla placeholdera "(uzupełnij dane pomiarowe)" — szary, akceptowalny. fillColor sekcji to szarości (#d1d5db, #f3f4f6). |
| 8  | Kliknięcie Zapisz zachowuje dane w localStorage                                         | ZWERYFIKOWANO | `saveToLocalStorage()` linia 1855-1866: `localStorage.setItem(STORAGE_KEY, JSON.stringify(AppState))`. Klucz: 'voltprotokol-state'. Obsługa błędów QuotaExceededError. |
| 9  | Kliknięcie Wczytaj odtwarza dane z localStorage — formularze wypełnione, obliczenia przeliczone | ZWERYFIKOWANO | `loadFromLocalStorage()` linia 1868-1878: `JSON.parse → restoreAppState()`. `restoreAppState()` linia 1842-1853: deep copy, `recalculateAll()`, pełny re-render wszystkich 5 funkcji render*(), `updateFinalAssessmentDisplay()`. |
| 10 | Kliknięcie Eksport JSON pobiera plik .json                                              | ZWERYFIKOWANO | `exportToJSON()` linia 1880-1892: JSON.stringify → Blob → URL.createObjectURL → anchor.click() → revokeObjectURL. |
| 11 | Kliknięcie Import JSON wczytuje plik .json i odtwarza stan                              | ZWERYFIKOWANO | `importFromJSON()` linia 1894-1914: FileReader → JSON.parse → restoreAppState(). |
| 12 | Toast notification informuje o wyniku operacji                                          | ZWERYFIKOWANO | `showToast()` linia 1815-1821: div Tailwind z auto-dismiss 2500ms, guard na parentNode. Użyty we wszystkich 4 operacjach persistence. |

**Wynik:** 12/12 prawd zweryfikowanych (4 wymagają potwierdzenia ludzkiego dla efektów visual/runtime)

### Wymagane artefakty

| Artefakt      | Oczekiwane                                              | Status     | Szczegóły                                                                      |
|---------------|---------------------------------------------------------|------------|--------------------------------------------------------------------------------|
| `index.html`  | Sekcja `// === PDF EXPORTER ===` z funkcjami build*    | ZWERYFIKOWANO | Linia 1916. 10 funkcji: pdfVal, makeSectionRow, makeSubsectionRow, buildFormMainContent, buildAttachment1-4Content, buildDocDefinition, exportPDF. Łącznie ~530 linii (1916-2445). |
| `index.html`  | Sekcja `// === PERSISTENCE ===` z funkcjami save/load  | ZWERYFIKOWANO | Linia 1811. 7 funkcji: showToast, recalculateAll, restoreAppState, saveToLocalStorage, loadFromLocalStorage, exportToJSON, importFromJSON. ~105 linii. |
| `index.html`  | 5 przycisków w nagłówku (Zapisz, Wczytaj, Eksport JSON, Import JSON, Eksportuj PDF) | ZWERYFIKOWANO | Linie 21-25. Wszystkie przyciski z id i klasami Tailwind. |
| `index.html`  | Event listenery podpięte w init()                       | ZWERYFIKOWANO | Linie 2497-2501. Wszystkie 5 przycisków podpięte do odpowiednich funkcji. |

### Weryfikacja kluczowych powiązań

| Od                        | Do                                          | Via                          | Status       | Szczegóły                                     |
|---------------------------|---------------------------------------------|------------------------------|--------------|-----------------------------------------------|
| `exportPDF()`             | `pdfMake.createPdf(dd).download()`          | `buildDocDefinition()`       | PODPIĘTE     | Linia 2441-2444. pdfmake załadowany z CDN linii 12-13. |
| `buildAttachment1Content()` | `AppState.swzFixedRow`, `AppState.sections` | DDO table body               | PODPIĘTE     | Linie 2123-2174. Stały wiersz + dynamiczne sekcje. |
| `buildAttachment2Content()` | `AppState.attachment2`, `pageOrientation: 'landscape'` | stack wrapper          | PODPIĘTE     | Linia 2434 w buildDocDefinition. |
| `saveToLocalStorage()`    | `localStorage.setItem(STORAGE_KEY, ...)`   | `JSON.stringify(AppState)`   | PODPIĘTE     | Linia 1857. |
| `loadFromLocalStorage()`  | `restoreAppState(saved)`                    | `JSON.parse → restoreAppState` | PODPIĘTE  | Linia 1873. |
| `restoreAppState(saved)`  | `recalculateAll() + render*() + updateFinalAssessmentDisplay()` | deep copy + pełny re-render | PODPIĘTE | Linie 1846-1852. |
| `exportToJSON()`          | `Blob + URL.createObjectURL + a.click()`   | `JSON.stringify → Blob`      | PODPIĘTE     | Linie 1881-1891. |
| `importFromJSON()`        | `FileReader + JSON.parse + restoreAppState()` | `input[type=file]`        | PODPIĘTE     | Linie 1895-1913. |
| `btn-export-pdf`          | `exportPDF()`                               | `addEventListener('click')`  | PODPIĘTE     | Linia 2497. |
| `btn-save`                | `saveToLocalStorage()`                      | `addEventListener('click')`  | PODPIĘTE     | Linia 2498. |
| `btn-load`                | `loadFromLocalStorage()`                    | `addEventListener('click')`  | PODPIĘTE     | Linia 2499. |
| `btn-export-json`         | `exportToJSON()`                            | `addEventListener('click')`  | PODPIĘTE     | Linia 2500. |
| `btn-import-json`         | `importFromJSON()`                          | `addEventListener('click')`  | PODPIĘTE     | Linia 2501. |

### Pokrycie wymagań

| Wymaganie | Status       | Szczegóły                                                                              |
|-----------|--------------|----------------------------------------------------------------------------------------|
| EXP-01    | SPELNIONE    | Przycisk "Eksportuj PDF" → exportPDF() → pdfMake.createPdf().download(). Plik .pdf pobierany. |
| EXP-02    | SPELNIONE    | Footer z numeracją stron (currentPage / pageCount). Brak kolorowych fontów (jedyny wyjątek: `#9ca3af` szary placeholder). fillColor sekcji to szarości drukowalne B&W. |
| EXP-03    | SPELNIONE    | headerRows: 4 (Zał.1), headerRows: 2 (Zał.2), headerRows: 1 (Zał.3, 4). pdfmake automatycznie powtarza nagłówki przy paginacji. |
| EXP-04    | WYMAGA HUMAN | Kod: font: 'Roboto', vfs_fonts.min.js z CDN. Poprawność renderowania polskich znaków w wygenerowanym PDF wymaga weryfikacji manualnej. |
| EXP-05    | SPELNIONE    | saveToLocalStorage() → localStorage.setItem(STORAGE_KEY, JSON.stringify(AppState)). Obsługa QuotaExceededError. Toast potwierdzenie. |
| EXP-06    | SPELNIONE    | loadFromLocalStorage() → JSON.parse → restoreAppState() → recalculateAll() + pełny re-render. Toast potwierdzenie. |
| EXP-07    | SPELNIONE    | exportToJSON() → Blob download z nazwą voltprotokol-*.json. revokeObjectURL po pobraniu. |
| EXP-08    | SPELNIONE    | importFromJSON() → input[type=file] → FileReader → JSON.parse → restoreAppState(). Obsługa błędów parsowania. |

### Wykryte antywzorce

| Plik        | Linia | Wzorzec           | Poziom   | Wpływ                                                                                       |
|-------------|-------|-------------------|----------|---------------------------------------------------------------------------------------------|
| index.html  | 2036  | `color: '#9ca3af'` | INFO    | Szary tekst placeholdera "(uzupełnij dane pomiarowe)" w PDF gdy brak orzeczenia. Nie narusza EXP-02 — szarość widoczna w druku B&W, to komunikat informacyjny. |

Brak blokerów ani ostrzeżeń krytycznych.

### Weryfikacja wymagana przez człowieka

#### 1. Polskie znaki w PDF (EXP-04)

**Test:** Otwórz index.html w przeglądarce (wymaga internetu do CDN pdfmake/vfs_fonts). Wypełnij dane z polskimi znakami (np. obiekt "Budynek mieszkalny, dz. 12/3 ul. Ząbkowska"). Kliknij "Eksportuj PDF". Otwórz pobrany plik PDF.
**Oczekiwane:** Znaki ą, ę, ś, ź, ć, ó, ł, ń, ż wyświetlają się poprawnie — nie ma kwadratów ani znaków zastępczych.
**Dlaczego człowiek:** Renderowanie fontu Roboto z vfs_fonts.min.js (CDN) wymaga aktywnego środowiska przeglądarki.

#### 2. Orientacja landscape Zał. 2

**Test:** Po wygenerowaniu PDF sprawdź strony z Załącznikiem 2 (Rezystancja izolacji).
**Oczekiwane:** Strony Zał. 2 są w orientacji poziomej (A4 landscape). Formularz główny, Zał. 1, 3, 4 są pionowe (portrait).
**Dlaczego człowiek:** Stack z pageOrientation: 'landscape' w buildDocDefinition wymaga weryfikacji w wygenerowanym PDF.

#### 3. Cykl Zapisz/Wczytaj z localStorage

**Test:** Wypełnij formularz, dodaj sekcje i wiersze pomiarowe. Kliknij "Zapisz". Odśwież stronę (Ctrl+R). Kliknij "Wczytaj".
**Oczekiwane:** (1) Toast "Dane zapisane." po zapisie. (2) Po odświeżeniu dane znikają. (3) Po Wczytaj: toast "Dane wczytane.", wszystkie zakładki z przywróconymi danymi, ocena końcowa prawidłowo wyliczona.
**Dlaczego człowiek:** Weryfikacja persystencji między sesjami wymaga działającej przeglądarki z obsługą localStorage.

#### 4. Cykl Eksport/Import JSON

**Test:** Kliknij "Eksport JSON" — pobierz plik. Odśwież stronę. Kliknij "Import JSON" — wybierz pobrany plik.
**Oczekiwane:** Plik voltprotokol-*.json pobiera się. Po imporcie toast "Wczytano z pliku." i pełne przywrócenie stanu (identycznie jak Wczytaj).
**Dlaczego człowiek:** Dialog wyboru pliku i FileReader wymagają interakcji z przeglądarką.

### Podsumowanie

Wszystkie 8 wymagań (EXP-01 do EXP-08) ma kompletną implementację w kodzie:

- **PDFExporter** (linie 1916-2445): 10 funkcji, ~530 linii. Formularz główny z ocenami i orzeczeniem, Zał. 1 (13 kolumn, 4-wierszowy nagłówek), Zał. 2 (15 kolumn, landscape), Zał. 3, Zał. 4. pdfmake DDO zbudowany poprawnie z headerRows, fillColor sekcji, stopką z numeracją stron.

- **Persistence** (linie 1811-1915): 7 funkcji, ~105 linii. localStorage z obsługą błędów, deep copy przez JSON roundtrip, restoreAppState wywołuje recalculateAll() i pełny re-render (renderFormTab, renderAttachment1-4, updateFinalAssessmentDisplay), eksport/import JSON z Blob i FileReader, toast notifications.

- **Podpięcie**: 5 przycisków w nagłówku (linie 21-25), 5 event listenerów w init() (linie 2497-2501). Wszystkie powiązania są obecne i kompletne.

Pozostają 4 pozycje wymagające weryfikacji manualnej dotyczące efektów visual i runtime (polskie znaki, landscape, persystencja przeglądarki).

---

_Zweryfikowano: 2026-02-24T08:08:09Z_
_Weryfikator: Claude (gsd-verifier)_
