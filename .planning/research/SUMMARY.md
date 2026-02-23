# Project Research Summary

**Project:** VoltProtokół (SEP-Calc) — generator protokołów pomiarowych instalacji elektrycznych
**Domain:** Single-file HTML SPA — formularze dynamiczne, obliczenia elektryczne, eksport PDF
**Researched:** 2026-02-23
**Confidence:** HIGH (stack, architektura, pułapki), MEDIUM (funkcje — brak bezpośrednich polskich webowych odpowiedników)

## Executive Summary

VoltProtokół to klient-only single-file HTML SPA do generowania protokołów pomiarowych instalacji elektrycznych według normy PN-HD 60364-6. Narzędzie wypełnia realną lukę rynkową — wszystkie istniejące polskie rozwiązania (Sonel PE6, PROTON+, iiione) są aplikacjami desktopowymi wymagającymi instalacji i są płatne (500-900 PLN). Kluczowa przewaga pozycjonowania VoltProtokołu to zero barier wejścia: brak instalacji, brak rejestracji, darmowy dostęp. Produkt musi dostarczać kompletny protokół zgodny z normą (formularz danych ogólnych + 4 załączniki: SWZ, rezystancja izolacji, RCD, uziemienie) z automatycznymi obliczeniami i eksportem PDF.

Rekomendowany stack to HTML5 + Tailwind CSS v4 CDN + Vanilla JS (ES2022+) + pdfmake 0.3.5. Decyzja o PDF jest kluczowa: pdfmake z deklaratywnym DDO jest jedynym kandydatem spełniającym wymóg eksportu pliku .pdf z selektywnym tekstem i automatyczną paginacją tabel na wielu stronach. Alternatywy (html2pdf.js, window.print()) są nieakceptowalne: html2pdf.js generuje rasteryzowany obraz (tekst nieselektywny), a window.print() nie może wywołać bezpośredniego pobierania pliku .pdf. Architektura to centralny AppState jako jedyne źródło prawdy, warstwy Calculator/FormRenderer/PDFExporter komunikujące się przez minimalne EventBus, z event delegation dla dynamicznych wierszy.

Najważniejsze ryzyko techniczne to obsługa polskich znaków w PDF — jsPDF z domyślnymi fontami produkuje puste prostokąty zamiast ą/ę/ś/ź bez żadnego błędu w konsoli. pdfmake z wbudowanym fontem Roboto eliminuje ten problem od razu, co jest dodatkowym argumentem za wyborem pdfmake. Pozostałe ryzyka (float precision w obliczeniach elektrycznych, localStorage w Safari Private) są łatwe do obsłużenia przez standardowe wzorce defensywne i muszą być zaadresowane w Fazie 1, przed pisaniem logiki biznesowej.

## Key Findings

### Recommended Stack

Projekt jest budowany jako jeden plik HTML bez bundlera, bez npm, bez serwera. Wszystkie zależności ładowane przez CDN `<script>` tagi. Stack nie ma realnych alternatyw w tym constraincie: Tailwind v4 CDN to jedyny CSS framework z działającym browser CDN w 2026, a pdfmake to jedyne rozwiązanie spełniające wymagania PDF (deklaratywny DDO, auto-paginacja, repeating headers, selektywny tekst). Hosting przez GitHub Pages.

**Podstawowe technologie:**
- **HTML5 + Vanilla JS (ES2022+):** shell aplikacji i cała logika — `<template>` cloning + event delegation zamiast frameworka; zero overhead, pełna kontrola
- **Tailwind CSS v4 CDN (`@tailwindcss/browser@4`):** jedyny CSS framework z browser CDN w 2026; JIT engine skanuje klasy runtime; obsługuje `<style type="text/tailwindcss">` dla `@apply` i custom tokenów
- **pdfmake 0.3.5:** ZALECANY (nie jsPDF); deklaratywny DDO zamiast imperatywnego kodu; automatyczna paginacja tabel z `headerRows: 1`; Roboto z pełnym Latin Extended (polskie znaki); aktywny maintenance (release Feb 2026)
- **pdfmake vfs_fonts 0.3.5:** musi być załadowany PO pdfmake; wbudowany font Roboto eliminuje problem polskich znaków z góry

**Krytyczna kolejność ładowania CDN:**
```html
<script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
<script src="https://cdn.jsdelivr.net/npm/pdfmake@0.3.5/build/pdfmake.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/pdfmake@0.3.5/build/vfs_fonts.min.js"></script>
```

### Expected Features

Niszą jest generator protokołów zgodnych z PN-HD 60364-6. Wszystkie 4 załączniki (SWZ, izolacja, RCD, uziemienie) to table stakes — brak któregokolwiek dyskwalifikuje narzędzie. Konkurenci desktopowi mają je wszystkie. Unique selling point VoltProtokołu to nie funkcje zaawansowane, lecz zero barier wejścia.

**Must have — table stakes (v1):**
- Formularz danych ogólnych (data, temperatura, nr protokołu, dane wykonawcy, przyrządy pomiarowe) — wymagane przez PN-HD 60364-6
- Załącznik 1 (SWZ) z obliczeniami Id=Usk/Zs, Zsmax=Usk/Ia i oceną (POZYTYWNA/NEGATYWNA) — najważniejsze badanie
- Załącznik 2 (rezystancja izolacji, Rw=1MΩ) — obowiązkowe badanie odbiorcze
- Załącznik 3 (RCD, tz=300ms) — obowiązkowe badanie odbiorcze
- Załącznik 4 (uziemienie z obliczeniem Rpo=Rp×Wk) — obowiązkowe
- Baza zabezpieczeń B/C/D + automatyczne Ia + ręczna korekta — bez bazy Załącznik 1 jest manualny (dealbreaker)
- Dynamiczne sekcje/wiersze we wszystkich załącznikach — fundamentalne; statyczne tabele są nieakceptowalne
- Eksport PDF (plik .pdf do pobrania) — core value proposition
- Zapis/odczyt localStorage (ręczny, na życzenie) — bez persystencji traci dane
- Ocena końcowa + legendy oznaczeń — wymagane normą

**Should have — differentiators (v1.x):**
- Eksport/import JSON stanu — przenoszalność danych między komputerami
- Walidacja inline z podświetlaniem pól — real-time feedback (UK tools to mają, polskie nie)
- Automatyczne obliczanie daty następnego badania (+5 lat) — trywialna implementacja
- Kopiowanie sekcji (duplikowanie kondygnacji) — dla wielopiętrowych budynków

**Defer — v2+:**
- Wiele protokołów w localStorage (archiwum dokumentów) — wymaga redesignu UI i zarządzania limitami
- Podgląd print-preview — wartościowe UX ale złożona implementacja
- Zdjęcia/załączniki — wymaga IndexedDB lub backend, poza scope v1

### Architecture Approach

Architektura to layered single-file: UI Layer (DOM) — Event Bus (Pub/Sub) — State Layer (AppState) — Services Layer (Calculator, Persistence, PDFExporter). Kod w jednym `<script>` bloku na końcu `<body>`, podzielony na sekcje komentarzami `// === NAME ===` (zastępują pliki modułów). AppState jest jedynym źródłem prawdy — PDFExporter czyta dane z AppState, nigdy z DOM.

**Główne komponenty:**
1. **PROTECTION_DB (const)** — baza zabezpieczeń B/C/D z wartościami Ia; lookup `getIa(type, rating)`; zero dependencies; inicjalizowana jako pierwsza
2. **AppState + UIState** — centralny obiekt stanu formularza (persystowany) i stan UI (niepersystowany); AppState mutowany tylko przez FormController
3. **EventBus (Pub/Sub)** — luźne powiązanie Calculator/FormRenderer przez `emit`/`on`; ~30 linii; eliminuje bezpośrednie zależności między modułami
4. **Calculator** — czyste funkcje `calcAttachment1Row()`, `calcAttachment4Row()`; zero side-effects; zero DOM; łatwe do testowania w konsoli
5. **FormRenderer + FormController** — renderowanie dynamicznych wierszy przez `<template>` cloning; event delegation na kontenerach (nie na każdym wierszu)
6. **Persistence** — `save()`/`load()` do localStorage; klucz `voltprotokol_v1`; owinięte try/catch
7. **PDFExporter** — buduje dokument pdfmake DDO z AppState; nigdy nie modyfikuje stanu; ostatni w kolejności inicjalizacji

**Kolejność budowania komponentów:** CONSTANTS → AppState → EventBus → Calculator → FormRenderer → FormController → Persistence → PDFExporter → init()

### Critical Pitfalls

Wszystkie 5 krytycznych pułapek musi być zaadresowanych w Fazie 1 — wszystkie mają wysoki koszt naprawy po zbudowaniu logiki biznesowej.

1. **Polskie znaki znikają z PDF** — jsPDF z domyślnymi fontami produkuje puste prostokąty zamiast ą/ę/ś/ź bez żadnego błędu w konsoli. Unikanie: użyć pdfmake z wbudowanym Roboto (eliminuje problem z góry) — to główny argument za pdfmake zamiast jsPDF.

2. **Tabela z 15+ kolumnami (Załącznik 2) wychodzi poza stronę PDF** — jsPDF-AutoTable (i pdfmake) milcząco ucina kolumny po prawej. Unikanie: użyć orientacji landscape dla Załącznika 2; jawnie ustawić `columnStyles` z przeliczonymi szerokościami.

3. **html2canvas + duże tabele = freeze przeglądarki 10-60 sekund** — 883 węzły DOM = 8s, 2660 węzłów = 66s. Unikanie: NIE używać html2canvas; budować PDF programatycznie z pdfmake DDO czytając dane z AppState.

4. **localStorage rzuca wyjątek (Safari Private, file://)** — `SecurityError` bez fallbacku crashuje aplikację; nie ma błędu JavaScript tylko exception. Unikanie: owinąć try/catch; wyświetlić czytelny komunikat "Zapis danych niedostępny w trybie prywatnym".

5. **Precyzja float w obliczeniach elektrycznych** — `0.1 + 0.2 === 0.30000000000000004`; ocena `Zs ≤ Zsmax` może dać błędny wynik przy wartościach granicznych. Unikanie: konsekwentne `roundTo3()` przy wszystkich wyjściach i porównaniach; napisać testy graniczne w konsoli przed implementacją oceny.

## Implications for Roadmap

Architektura dyktuje klarowny order implementacji od dołu stosu ku górze. Badania wskazują na 3 fazy: fundament architektury + bezpieczeństwo, logika biznesowa + PDF, polish + differentiatory.

### Phase 1: Fundament — Architektura, Formularze, PDF

**Rationale:** Wszystkie 5 krytycznych pułapek musi być zaadresowanych przed napisaniem logiki biznesowej. AppState i event delegation to prerequisit dla wszystkich dynamicznych sekcji. Wybór strategii PDF i fontu to decyzja architektoniczna niemożliwa do zmiany bez refaktoru.
**Delivers:** Działający szkielet aplikacji — dynamiczne formularze wszystkich 4 załączników, obliczenia automatyczne, podstawowy eksport PDF z polskimi znakami, zapis/odczyt localStorage.
**Addresses:** Formularz danych ogólnych, Załącznik 1 (SWZ z obliczeniami), Załącznik 2, 3, 4, baza zabezpieczeń B/C/D, dynamiczne wiersze, eksport PDF, localStorage.
**Avoids:** Wszystkie 5 krytycznych pułapek (polska czcionka w PDF, tabela 15 kolumn, html2canvas performance, localStorage crash, float precision).
**Build order within phase:** PROTECTION_DB → AppState → EventBus → Calculator → FormRenderer → FormController → Persistence → PDFExporter → init().

### Phase 2: Ocena Końcowa, Kompletność Normy, UX

**Rationale:** Po działającym fundamencie — uzupełnienie elementów wymaganych przez PN-HD 60364-6 (ocena końcowa, legendy) i UX differentiators o niskim koszcie. Te elementy zależą od poprawnie działającego Phase 1.
**Delivers:** Protokół kompletny normowo — ocena końcowa z datą następnego badania, legendy oznaczeń pod tabelami, walidacja inline (czerwone pola przy Zs > Zsmax), auto-obliczanie daty +5 lat.
**Implements:** Assessment section AppState, walidacja CSS `.invalid`, domyślna data następnego badania.
**Avoids:** UX pułapka "brak wskazania postępu generowania PDF" (spinner + blokada przycisku); "błędy walidacji nie wskazują na konkretne pole".

### Phase 3: Differentiators i Export/Import

**Rationale:** Po walidacji core product — dodanie funkcji podnoszących wartość dla użytkownika bez ryzyka zmiany architektury fundament. Export JSON i kopiowanie sekcji bazują na już działającym AppState + Persistence.
**Delivers:** Eksport/import JSON stanu (przenoszalność danych), kopiowanie sekcji (duplikowanie kondygnacji dla budynków wielopiętrowych), polish PDF (orientacja landscape Załącznik 2, właściwe style tabel).
**Implements:** `Blob` + `URL.createObjectURL()` dla JSON download; `FileReader` dla import; deep-clone sekcji + renumeracja.
**Defers:** Wiele protokołów w localStorage (archiwum), podgląd print-preview, zdjęcia — v2+ po ustaleniu product-market fit.

### Phase Ordering Rationale

- **Zależności stosu wymuszają order Phase 1:** PROTECTION_DB i AppState muszą istnieć zanim Calculator może liczyć, zanim FormRenderer może renderować, zanim PDFExporter może eksportować. Nie można pominąć żadnego kroku.
- **Pułapki są skoncentrowane w Phase 1:** Wszystkie krytyczne decyzje (font PDF, orientacja tabeli, brak html2canvas, try/catch localStorage, roundTo3) muszą być podjęte jako pierwsze — koszt naprawy po zbudowaniu logiki biznesowej jest wysoki (szczególnie zamiana html2canvas na programatyczny PDF to przepisanie 200+ linii).
- **Phase 2 i 3 są addytywne:** Każda z nich dodaje funkcje na gotowym fundamencie, bez ryzyka regresji w logice obliczeniowej.
- **Anti-features identyfikowane w badaniach:** Backend, konta użytkowników, import z miernika, wielodostęp — nigdy nie budować. Scope creep w tę stronę zniszczyłby architekturę single-file.

### Research Flags

**Fazy potrzebujące głębszego researchu przy planowaniu:**
- **Phase 1 (PDFExporter):** Konkretna struktura DDO pdfmake dla każdego z 4 załączników (colSpan/rowSpan w nagłówkach tabel protokołu, orientacja landscape dla Załącznika 2, właściwe rozmiary kolumn). Warto zbadać przykłady pdfmake z kompleksowymi tabelami przed implementacją.
- **Phase 1 (PROTECTION_DB):** Kompletne wartości Ia dla zabezpieczeń charakterystyk B/C/D w zakresie 6A-63A — muszą być zgodne z aktualnymi normami. Źródło: tabele producenckie (np. Moeller, ABB) lub norma PN-EN 60898.

**Fazy ze standardowymi wzorcami (można pominąć research-phase):**
- **Phase 2 (ocena końcowa, legendy, walidacja inline):** Standardowe DOM manipulation i CSS — wzorce dobrze udokumentowane, zero ryzyka.
- **Phase 3 (JSON export/import):** `Blob` + `URL.createObjectURL()` + `FileReader` — standardowe Web API, MDN docs wystarczą.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | Wszystkie kluczowe technologie zweryfikowane przez oficjalne źródła (jsDelivr, Tailwind docs, pdfmake GitHub releases). Wersje potwierdzone Feb 2026. |
| Features | MEDIUM | Brak bezpośrednich polskich webowych odpowiedników — wnioski z analizy konkurencji desktopowej (Sonel PE6, iiione) i dyskusji forum ISE. Polskie prawo elektryczne (PN-HD 60364-6) stabilne. |
| Architecture | HIGH | Wzorce (centralny state, event delegation, revealing module, pdfmake DDO) opierają się na authoritative sources (patterns.dev, MDN, jsPDF-AutoTable GitHub). |
| Pitfalls | HIGH | Większość pułapek zweryfikowana przez oficjalne GitHub issues (html2canvas, jsPDF, jsPDF-AutoTable) z konkretnymi numerami issues i metrykami. |

**Overall confidence:** HIGH

### Gaps to Address

- **Dokładne wartości Ia dla PROTECTION_DB:** Research nie zebrał pełnej tabeli wartości Ia dla wszystkich charakterystyk B/C/D × prądy znamionowe. Wymaga weryfikacji z normą PN-EN 60898 lub katalogami producentów podczas Phase 1.
- **Struktura tabel referencyjnego wzoru protokołu:** FEATURES.md wspomina "wzór referencyjny" bez szczegółów struktury każdej tabeli. Należy skonsultować z klientem lub znaleźć oficjalny wzór PN-HD 60364-6 przed projektowaniem tabel PDF.
- **Limit kolumn Załącznika 2:** Orientacja landscape potwierdzona jako rozwiązanie, ale dokładna liczba i nazwy kolumn dla Załącznika 2 wymaga weryfikacji z referencyjnym dokumentem przed implementacją.

## Sources

### Primary (HIGH confidence)
- jsDelivr package pages (pdfmake 0.3.5, jsPDF 4.2.0, jspdf-autotable 5.0.7) — wersje CDN
- Tailwind CSS v4 Play CDN official docs — CDN script i ograniczenia
- pdfmake GitHub releases — aktywny maintenance zweryfikowany Feb 2026
- html2canvas GitHub issue #263 — metryki wydajności (883 węzły = 8s, 2660 węzłów = 66s)
- MDN Web Storage API — localStorage behavior w trybie prywatnym
- patterns.dev — Module Pattern (Revealing Module/IIFE)
- IEEE 754 — floating point precision w JavaScript (JavaCodeGeeks 2024)

### Secondary (MEDIUM confidence)
- Sonel PE6, iiione Pomiary Elektryczne, PROTON+, CERTY Megger, Arkusz ZPE24 — feature analysis
- Forum ISE — dyskusje elektryków o narzędziach do protokołów
- UK electrical cert software comparison (iCertifi, Tradecert features)
- pdf generation comparison (Dmitrii Boikov 2025) — html2canvas image output confirmed
- BSWEN blog (2026-02-21) — single-file JavaScript apps bez build tools
- jsPDF-AutoTable issues #282, #306 — wide table behavior

### Tertiary (LOW confidence)
- PROTON+ features — strona marketingowa bez szczegółów technicznych
- Norma PN-HD 60364-6 zakres — portal elektryka (nie oficjalny tekst normy)

---
*Research completed: 2026-02-23*
*Ready for roadmap: yes*
