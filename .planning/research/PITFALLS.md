# Pitfalls Research

**Domain:** Single-file HTML app — electrical protocol generator z PDF
**Researched:** 2026-02-23
**Confidence:** HIGH (większość zweryfikowana przez oficjalne GitHub issues, MDN, npm docs)

---

## Critical Pitfalls

### Pitfall 1: Polskie znaki znikają z wygenerowanego PDF

**What goes wrong:**
jsPDF z domyślnymi fontami produkuje PDF z pustymi kwadratami lub znakami zapytania zamiast ą, ę, ś, ź, ó, ć, ń, ł. Dzieje się to cicho — żaden błąd nie jest rzucany w konsoli, protokół "wygenerował się", ale jest nieczytelny.

**Why it happens:**
14 standardowych fontów PDF (Helvetica, Times, Courier itd.) jest ograniczonych do strony kodowej ASCII/Latin-1. Glify polskich znaków diakrytycznych nie istnieją w tych fontach. jsPDF nie ostrzega o brakujących glyfach — po prostu renderuje pusty prostokąt.

**How to avoid:**
1. Użyj niestandardowego fontu TTF z pełną obsługą Unicode (np. DejaVu Sans, Open Sans, Roboto) — font musi zawierać glify polskie.
2. Przekonwertuj font TTF do formatu Base64 JS za pomocą narzędzia fontconverter z repozytorium jsPDF.
3. Załaduj font przez `doc.addFileToVFS()` + `doc.addFont()` + `doc.setFont()` PRZED jakimkolwiek wywołaniem `doc.text()`.
4. Alternatywa: użyj html2canvas jako warstwy renderowania — canvas rasteryzuje tekst z fontów systemowych, co omija problem glyfów. Polskie znaki będą poprawne, ale tekst będzie obrazkiem (nie do zaznaczenia).

**Warning signs:**
- Testuj z pierwszym protokołem zawierającym "Sprawdzono", "Sieć", "Odzielna" — frazy z polskimi znakami
- Jeżeli test PDF pokazuje "?" lub puste miejsca zamiast ą/ę/ś — problem gotowy
- Sprawdzaj w Chrome, Firefox i Safari — każdy może zachowywać się inaczej

**Phase to address:**
Faza 1 — przed napisaniem jakiegokolwiek kodu generowania PDF. Wybór strategii fontu to decyzja architektoniczna, która wpływa na wszystko inne.

---

### Pitfall 2: Tabela z 15+ kolumnami wychodzi poza szerokość strony PDF

**What goes wrong:**
Tabela rezystancji izolacji (Załącznik 2) z kolumnami L-N, L-PE, N-PE, L1-N, L2-N, L3-N itd. wymaga 15+ kolumn. jsPDF-AutoTable nie obsługuje automatycznie przepełnienia poziomego — tabela jest po prostu ucinana przy prawej krawędzi strony bez ostrzeżenia.

**Why it happens:**
jsPDF-AutoTable zarządza pionowymi podziałami stron (wiersze → nowe strony), ale poziome przepełnienie (za dużo kolumn) jest milcząco obcinane. Domyślna szerokość strony A4 to 210mm, a przy 15+ kolumnach każda kolumna ma ~12mm co jest zbyt wąskie dla czytelnych liczb.

**How to avoid:**
Strategia 1 — Orientacja pozioma (landscape): `jsPDF('l', 'mm', 'a4')` daje 297mm szerokości zamiast 210mm. Dla 15 kolumn = ~18mm na kolumnę — wystarczające.
Strategia 2 — `horizontalPageBreak: true` w AutoTable: rozkłada kolumny na wiele stron (nieczytelne dla protokołu — odrzucić).
Strategia 3 — Zredukuj rozmiar tekstu w kolumnach (`fontSize: 7`) i jawnie ustaw `columnStyles` z wymaganą szerokością minimalną.
**Rekomendacja:** Orientacja landscape dla Załącznika 2. Sprawdź z referencyjnym dokumentem Word czy tak jest w oryginale.

**Warning signs:**
- Kolumny po prawej stronie tabeli brakuje lub są ucięte w podglądzie PDF
- `doc.internal.pageSize.getWidth()` minus marginesy < suma szerokości kolumn

**Phase to address:**
Faza 1 — przy tworzeniu prototypu tabel. Przed napisaniem pełnej logiki formularza ustal orientację każdego załącznika.

---

### Pitfall 3: html2canvas + duże tabele = timeout lub freeze przeglądarki

**What goes wrong:**
Przy podejściu html2canvas (renderowanie DOM → obraz → PDF) czas renderowania rośnie nieliniowo z rozmiarem DOM. Pomiar: 883 węzły DOM = 8 sekund, 2660 węzłów = 66 sekund. Protokół z 4 załącznikami i dziesiątkami wierszy może mieć 2000+ węzłów DOM, co powoduje kilkudziesięciosekundowe zawieszenie przeglądarki.

**Why it happens:**
html2canvas musi przeparsować i narysować każdy element CSS. Złożone tabelki z Tailwind CSS (każda komórka z dziesiątkami klas utility) dramatycznie spowalniają rendering. Firefox jest szczególnie wolny (raportowane 2+ minuty vs Chrome).

**How to avoid:**
Rekomendowane podejście: **NIE używaj html2canvas do generowania PDF**. Zamiast tego:
1. Użyj jsPDF-AutoTable bezpośrednio (budowanie tabel programatycznie z danych stanu JS)
2. Alternatywnie użyj `window.print()` z `@media print` CSS — to natywne rozwiązanie przeglądarki bez opóźnień
Jeżeli html2canvas jest niezbędny: renderuj sekcję po sekcji, nie cały dokument naraz; użyj `scale: 1` zamiast `scale: 2`.

**Warning signs:**
- Strona "freezuje" po kliknięciu "Generuj PDF"
- Konsola pokazuje długie bloki bez aktywności
- `performance.now()` przed i po `html2canvas()` → >5000ms to sygnał alarmowy

**Phase to address:**
Faza 1 — decyzja architektury PDF musi być świadoma tego limitu. Prototyp "proof of concept" z pełną tabelą musi być testem wydajności.

---

### Pitfall 4: localStorage niedostępne lub rzuca wyjątek w niektórych kontekstach

**What goes wrong:**
Wywołanie `localStorage.setItem()` rzuca `SecurityError: Access is denied` w trybie incognito Safari, przy blokadzie third-party cookies, lub gdy plik jest otwierany bezpośrednio przez `file://` (nie przez serwer HTTP). Aplikacja crashuje bez fallbacku.

**Why it happens:**
- Safari Private Browsing blokuje localStorage całkowicie (rzuca wyjątek, nie zwraca null)
- Plik otwierany przez `file://` ma niezdefiniowane zachowanie localStorage w różnych przeglądarkach
- Chrome v80+ blokuje third-party cookies domyślnie, co wpływa na localStorage w iframes

**How to avoid:**
```javascript
function safeLocalStorage() {
  try {
    const test = '__test__';
    localStorage.setItem(test, '1');
    localStorage.removeItem(test);
    return localStorage;
  } catch (e) {
    return null; // graceful degradation
  }
}
```
Zawsze owijaj dostęp do localStorage w try/catch. Wyświetl czytelny komunikat gdy storage jest niedostępny: "Zapis danych niedostępny w trybie prywatnym — dane zostaną utracone po zamknięciu karty".

**Warning signs:**
- Aplikacja testowana tylko w Chrome — brak testów w Safari Private
- Brak try/catch wokół operacji localStorage
- Plik HTML otwierany bezpośrednio z dysku (file://) zamiast przez lokalny serwer

**Phase to address:**
Faza 1 — przy implementacji zapisu/odczytu danych. Wrapper bezpieczeństwa musi być pierwszą rzeczą napisaną przed logiką zapisu.

---

### Pitfall 5: Precyzja zmiennoprzecinkowa w obliczeniach elektrycznych

**What goes wrong:**
`0.1 + 0.2 === 0.30000000000000004` w JavaScript. W obliczeniach: `Zs = 230 / (0.1 + 0.05)` może dać `1.5333333333333335` zamiast `1.533`. Wyświetlane na protokole wartości wyglądają nieprofesjonalnie. Gorzej: ocena `Zs ≤ Zsmax` może dać błędny wynik przy wartościach granicznych: `1.53333... ≤ 1.533` to `false` gdy oba powinny być równe.

**Why it happens:**
JavaScript używa IEEE 754 binary floating-point. Nie można dokładnie reprezentować niektórych dziesiętnych ułamków binarnie.

**How to avoid:**
1. Wszystkie wyświetlane wartości zaokrąglaj do sensownej precyzji: `(Math.round(val * 1000) / 1000).toFixed(3)`
2. Porównania zawsze na zaokrąglonych wartościach: `roundTo3(Zs) <= roundTo3(Zsmax)`
3. Dla obliczeń przemysłowych definiuj stałą precyzji na początku kodu i używaj konsekwentnie
4. Unikaj bibliotek (decimal.js) — to overhead dla prostych elektrycznych obliczeń

**Warning signs:**
- Wyniki obliczeń wyświetlają się z 12+ cyframi po przecinku
- Protokół pokazuje "NEGATYWNA" gdy ręczny kalkulator daje wynik graniczny "POZYTYWNA"
- Testy z wartościami 0.1, 0.2, 0.3 nie przechodzą

**Phase to address:**
Faza 1 — przy pisaniu funkcji obliczeniowych. Napisz testy jednostkowe (nawet inline w konsoli) dla przypadków granicznych przed implementacją oceny.

---

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Globalny stan w zmiennych `let data = {}` na górze pliku | Prosto, szybko | Spaghetti przy >500 linii; debug niemożliwy gdy stan się korumpuje | Nigdy — użyj struktury od razu |
| Event listenery na każdym dynamicznym wierszu zamiast delegacji | Łatwiejsze do napisania | Memory leak przy dodawaniu/usuwaniu setek wierszy; wolne | Nigdy przy dynamicznych wierszach |
| Inline style do print layoutu zamiast `@media print` | Szybka iteracja | Tailwind nadpisuje przy zmianie, nie skaluje | Tylko w pierwszym prototypie |
| Hardcoded wartości (`Ia` dla B-10A = 50A) zamiast bazy danych | Brak konfiguracji | Zmiana normy wymaga edycji kodu wszędzie | Tylko jeśli dane są absolutnie stabilne |
| `document.getElementById` dla każdego pola zamiast referencji | Brak struktury | O(n) przy każdym odczycie stanu formularza; błędy literówek w ID | Tylko przy <10 polach |
| html2canvas zamiast jsPDF-AutoTable | Łatwa implementacja | 30-60s renderowania przy pełnym protokole | Nigdy dla głównej ścieżki — tylko dla nagłówka/logo |

---

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| jsPDF z CDN | Ładowanie synchroniczne — skrypt blokuje stronę | Użyj `defer` lub dynamicznego importu; sprawdź czy `window.jspdf` jest dostępne przed użyciem |
| html2canvas z CDN | `html2canvas is not defined` gdy wywoływany zbyt wcześnie | Sprawdź w DevTools Network że CDN się załadował; rozważ `document.readyState === 'complete'` |
| Tailwind CDN (Play CDN) | Klasy `print:hidden` i `print:block` mogą nie działać z CDN w trybie JIT | Zweryfikuj czy Play CDN obsługuje wszystkie użyte warianty print |
| jsPDF + polskie fonty | Font załadowany po wywołaniu `doc.text()` — znaki wychodzą domyślnym fontem | Ładuj font jako pierwsza operacja w funkcji generowania PDF, przed jakimikolwiek operacjami rysowania |
| localStorage + GitHub Pages | Dane zapisane na `file://` vs `https://` są w osobnych "storages" — użytkownik traci dane po przejściu na GH Pages | Zawsze testuj ze ścieżki HTTP, nie file:// |

---

## Performance Traps

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| html2canvas na całym dokumencie | "Freezing" przeglądarki 10-60s po kliknięciu "Generuj PDF" | Buduj PDF bezpośrednio z danych (jsPDF-AutoTable) zamiast renderować DOM | Już przy 500 węzłach DOM |
| Nasłuchiwanie `input` na każdym polu formularza | Każdy keystroke przelicza wszystko — lag przy szybkim wpisywaniu | Debounce obliczenia z 300ms; obliczaj tylko gdy zmienią się pola wejściowe danego obliczenia | Przy 50+ polach z żywymi obliczeniami |
| Odczyt `localStorage` przy każdym evencie | Operacje I/O blokują główny wątek | Cache stanu w pamięci JS; zapisuj do localStorage tylko przy jawnym "Zapisz" | Już przy 100+ zapisów/min |
| Duże JSON w localStorage | Wolny zapis/odczyt, ryzyko przekroczenia limitu 5MB | Waliduj rozmiar przed zapisem; ostrzegaj użytkownika gdy zbliża się do limitu | Przy 50+ wierszach w każdym załączniku |

---

## UX Pitfalls

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| Nie ma "Zapisz" ani "Wczytaj" — dane giną po odświeżeniu | Elektryk traci godzinę pracy przez przypadkowe odświeżenie | Autosave co 30s do localStorage z ikoną stanu; osobny przycisk "Eksportuj JSON" do pliku |
| PDF generuje się ze stylami ekranowymi (kolorowe nagłówki, ciemne tła) | Protokół wygląda jak screenshot strony www, nie jak dokument | Dedykowany "print mode" CSS (`@media print`) lub osobny kontener budowany tylko dla PDF |
| Błędy walidacji nie wskazują na konkretne pole | "Wprowadź wartość" bez wskazania gdzie — użytkownik szuka błędu | Scrolluj do pierwszego błędnego pola; podświetlaj pole czerwoną ramką |
| Brak wskazania postępu generowania PDF | Użytkownik klika wielokrotnie myśląc że nic się nie dzieje → wiele plików PDF | Zablokuj przycisk + pokaż spinner podczas generowania |
| Formularz nie waliduje zakresu wartości pomiarowych | Wartość `-999` MΩ akceptowana — protokół z nonsensem | Dla każdego pola pomiarowego: min/max, wymagalność, format numeryczny |

---

## "Looks Done But Isn't" Checklist

- [ ] **PDF z polskimi znakami:** Wygeneruj PDF i otwórz w zewnętrznym readerze (nie podglądzie Chrome) — sprawdź czy ą/ę/ś/ź są poprawne
- [ ] **Podział stron PDF:** Wygeneruj protokół z 20+ wierszami w Załączniku 1 — sprawdź czy nagłówek tabeli powtarza się na kolejnej stronie
- [ ] **Tabela Załącznik 2:** Sprawdź czy wszystkie 15 kolumn zmieszczą się na stronie — wygeneruj PDF z danymi we wszystkich komórkach
- [ ] **localStorage po zamknięciu:** Zapisz, zamknij kartę, otwórz ponowo — sprawdź czy dane są odtworzone kompletnie
- [ ] **localStorage w Safari Private:** Otwórz stronę w trybie prywatnym Safari — aplikacja nie może crashować
- [ ] **Obliczenia graniczne:** Wpisz Zs równe dokładnie Zsmax — ocena musi być POZYTYWNA (nie negatywna z powodu float)
- [ ] **Dodawanie/usuwanie wierszy:** Dodaj 10 wierszy, usuń 5, dodaj 3 — numery porządkowe muszą być ciągłe, obliczenia poprawne
- [ ] **Drukowanie z przeglądarki:** `Ctrl+P` powinno dawać czysty layout bez elementów UI (navbar, przyciski)
- [ ] **Plik z CDN offline:** Otwórz stronę bez internetu — jasny komunikat zamiast broken UI

---

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Polskie znaki nie działają w PDF | MEDIUM | Zamień html2canvas na jsPDF-AutoTable z custom fontem; lub zmień podejście na `window.print()` |
| Tabela ucięta na PDF | LOW | Zmień orientację na landscape; jawnie ustaw `columnStyles` z przeliczonymi szerokościami |
| html2canvas zbyt wolne | HIGH | Przepisz generowanie PDF z html2canvas na jsPDF-AutoTable; zmiana architekturalna |
| localStorage crashuje | LOW | Dodaj try/catch wrapper; 1-2h pracy |
| Błędy float w obliczeniach | LOW | Dodaj funkcję `roundTo(val, decimals)` i stosuj przy wszystkich wyjściach; 2-4h |
| Brak podziału stron tabeli | LOW | Dodaj `thead { display: table-header-group }` w `@media print`; lub przełącz na jsPDF-AutoTable z `repeatHeaders: true` |

---

## Pitfall-to-Phase Mapping

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| Polskie znaki w PDF | Faza 1 — Prototyp PDF | Generuj testowy PDF z pełnym alfabetem polskim i otwórz w Adobe Reader |
| Tabela 15 kolumn ucięta | Faza 1 — Struktura tabel | Prototyp Załącznika 2 z wszystkimi kolumnami w PDF |
| html2canvas timeout | Faza 1 — Decyzja architektury | Benchmark czas generowania przy 30 wierszach |
| localStorage rzuca wyjątek | Faza 1 — Zapis/odczyt | Test w Safari Private Mode + file:// |
| Precyzja float | Faza 1 — Obliczenia | Test jednostkowy z wartościami granicznymi (Zs = Zsmax) |
| Brak nagłówka na kolejnych stronach | Faza 1 — Layout PDF | Protokół z min. 3 stronami — nagłówek tabeli powtarzany |
| Memory leak z event listenerami | Faza 1 — Architektura formularza | DevTools Memory → Heap Snapshot przed/po dodaniu 50 wierszy |
| Stan globalny jako spaghetti | Faza 1 — Architektura stanu | Code review czy `appState` jest jedynym źródłem prawdy |

---

## Sources

- [jsPDF GitHub — UTF-8 support issue #2968](https://github.com/parallax/jsPDF/issues/2968) — MEDIUM confidence (oficjalne repo, długa dyskusja)
- [jsPDF-AutoTable issue #306 — columnWidth wrap cuts table off](https://github.com/simonbengtsson/jsPDF-AutoTable/issues/306) — MEDIUM confidence
- [jsPDF-AutoTable issue #282 — Wide Table goes beyond page width](https://github.com/simonbengtsson/jsPDF-AutoTable/issues/282) — MEDIUM confidence
- [html2canvas issue #263 — Bad performance](https://github.com/niklasvh/html2canvas/issues/263) — HIGH confidence (oficjalne repo, metryki podane)
- [html2canvas issue #2863 — 30s+ with console open](https://github.com/niklasvh/html2canvas/issues/2863) — MEDIUM confidence
- [MDN — Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API) — HIGH confidence
- [LogRocket — localStorage complete guide](https://blog.logrocket.com/localstorage-javascript-complete-guide/) — MEDIUM confidence
- [Devlin Peck — How to Use Custom Fonts with jsPDF](https://www.devlinpeck.com/content/jspdf-custom-font) — MEDIUM confidence
- [Handling Floating Point Precision in JavaScript — JavaCodeGeeks 2024](https://www.javacodegeeks.com/2024/11/handling-floating-point-precision-in-javascript.html) — HIGH confidence
- [CSS Print — Repeat table headers @media print](https://www.codegenes.net/blog/repeat-table-headers-in-print-mode/) — MEDIUM confidence
- [Tailwind CSS print styles guide](https://www.jacobparis.com/content/css-print-styles) — MEDIUM confidence
- [html2canvas CORS issues — html2canvas repo issue #1544](https://github.com/niklasvh/html2canvas/issues/1544) — HIGH confidence (oficjalne repo)
- [Memory Leaks in JavaScript — ditdot.hr](https://www.ditdot.hr/en/causes-of-memory-leaks-in-javascript-and-how-to-avoid-them) — MEDIUM confidence

---
*Pitfalls research for: VoltProtokół (SEP-Calc) — single-file HTML electrical protocol generator*
*Researched: 2026-02-23*
