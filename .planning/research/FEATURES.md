# Feature Research

**Domain:** Generator protokołów pomiarowych instalacji elektrycznych (SPA, klient-side, PN-HD 60364-6)
**Researched:** 2026-02-23
**Confidence:** MEDIUM (brak bezpośrednich polskich webowych odpowiedników — wnioski oparte na analizie narzędzi desktopowych + analogów zagranicznych + dyskusji użytkowników)

---

## Feature Landscape

### Table Stakes (Users Expect These)

Funkcje, których brak sprawia, że narzędzie jest "niepełne" w oczach elektryka. Konkurencja desktopowa (Sonel PE6, PROTON+, iiione, CERTY) i szablony Excel wszystkie je mają.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Formularz danych ogólnych (data, temperatura, nr protokołu, dane wykonawcy, przyrządy pomiarowe) | Każdy protokół wg PN-HD 60364-6 musi zawierać te dane. Elektryka sprawdzają to kontrolerzy. | LOW | Statyczne pola, brak obliczeń. |
| Załącznik SWZ — impedancja pętli zwarcia z polami Zs, Zsmax, Id | Ochrona przez samoczynne wyłączenie zasilania to obowiązkowe badanie. Wszystkie programy to mają. | MEDIUM | Wymaga obliczeń Id=Usk/Zs i Zsmax=Usk/Ia. |
| Załącznik rezystancja izolacji (Rp dla obwodów 1- i 3-fazowych) | Standardowy pomiar odbiorczy. Musi być w każdym protokole. | LOW | Pola pomiarowe, brak obliczeń poza porównaniem z Rw=1MΩ. |
| Załącznik RCD — badanie wyłączników różnicowoprądowych | PN-HD 60364-6 wymaga dokumentowania badania RCD. Wszystkie programy to mają. | LOW | Głównie pola danych, tz=300ms domyślnie. |
| Załącznik uziemienie — z obliczeniem Rpo | Pomiar wymagany normą. Wszystkie programy to mają. | LOW | Obliczenie Rpo=Rp×Wk. |
| Automatyczne obliczenia z oceną (POZYTYWNA/NEGATYWNA) | Sonel PE6, iiione i PROTON+ wszystkie to robą. Elektrycy oczekują że program sam oceni wyniki. | MEDIUM | Zs≤Zsmax → POZYTYWNA; Rpo≤Rw → POZYTYWNA. |
| Eksport do PDF | Cel istnienia narzędzia. Każda konkurencja to ma. | MEDIUM | Kluczowa decyzja: jsPDF+html2canvas vs window.print(). |
| Baza zabezpieczeń (B/C/D, automatyczne Ia) | Sonel PE6 i iiione mają "rozbudowane bazy zabezpieczeń". Bez bazy użytkownik musi ręcznie szukać wartości Ia — czasochłonne i podatne na błędy. | MEDIUM | Charakterystyki B/C/D × prądy znamionowe (6A–63A typowe). |
| Możliwość ręcznej korekty Ia | Elektrycy używają zabezpieczeń niestandardowych (np. starsze typy). Każde narzędzie profesjonalne musi to umożliwiać. | LOW | Pole edytowalne nadpisujące wartość z bazy. |
| Interfejs w całości po polsku | Grupa docelowa: polscy elektrycy. Każde polskie narzędzie jest po polsku. | LOW | Czysta kopia — brak tłumaczeń. |
| Dynamiczne sekcje/wiersze (dodawanie obwodów, kondygnacji) | Każda instalacja ma inną liczbę obwodów. Statyczne tabele są nieakceptowalne — brak sekcji dla "Piętra 2" jest dealbreaker. | MEDIUM | Wzorzec: stały nagłówek + dynamiczne wiersze + "Dodaj wiersz". |
| Zapis/odczyt stanu (localStorage) | Elektryk pracuje nad protokołem w kilku sesjach. Bez persystencji traci dane. Excel i programy desktopowe oczywiście zapisują. | LOW | localStorage wystarczy — brak backendu to założenie projektu. |
| Ocena końcowa (orzeczenie) i data następnego badania | Wymagane przez normę PN-HD 60364-6. W każdym wzorze protokołu jest sekcja "Ocena wykonanych badań odbiorczych". | LOW | Data +5 lat od daty badania. |

### Differentiators (Competitive Advantage)

Funkcje, których brak nie dyskwalifikuje, ale których obecność buduje przewagę nad istniejącymi narzędziami.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Zero instalacji — działa w przeglądarce | Sonel PE6, PROTON+, iiione — wszystkie wymagają instalacji Windows. Web-first eliminuje barierę wejścia. To główna różnica pozycjonowania. | LOW | Wynika z single-file HTML + CDN. |
| Bezpłatny, open-source | Sonel PE6 kosztuje ~500–900 PLN, PROTON+ jest płatny, iiione ma trial 30 dni. Darmowe narzędzie to silny argument dla samodzielnych elektryków. | LOW | Koszt utrzymania GitHub Pages = 0 PLN. |
| Natychmiastowa dostępność (brak rejestracji) | Każdy program wymaga instalacji lub rejestracji. Brak "konta" to bezpośrednia przewaga UX dla jednorazowego użytkownika. | LOW | Wynika z architektury. |
| Eksport JSON stanu (backup/przeniesienie) | Programy desktopowe trzymają dane w nieprzenośnych formatach. JSON pozwala na backup, przeniesienie między komputerami, wersjonowanie. Forum ISE pokazuje potrzebę "archiwizacji". | LOW | Jeden przycisk "Eksportuj dane (JSON)" + drag-and-drop import. |
| Legendy oznaczeń pod tabelami | Wzór referencyjny zawiera legendy. Programy desktopowe często tego nie mają inline — elektryk musi pamiętać skróty. Legenda inline = protokół edukacyjny i samowytłumaczający się. | LOW | Statyczny HTML pod każdą tabelą. |
| Automatyczne wyliczanie daty następnego badania | Drobna wygoda: data badania +5 lat automatycznie. Iiione ma "kontrolę terminów" ale w formie osobnego modułu. Tutaj inline. | LOW | `new Date().setFullYear(+5)` — trywialna implementacja. |
| Podgląd print-preview przed eksportem | UK tools (iCertifi, Tradecert) to mają. Polskie narzędzia nie — użytkownik drukuje i widzi problem za późno. | MEDIUM | Dedykowana sekcja @media print lub modal z iframe. |
| Kopiowanie sekcji (duplikowanie kondygnacji/obwodów) | Sonel PE6 ma "automatyczne wypełnianie protokołów serią danych". Dla budynku wielopiętrowego kopiowanie gotowej sekcji zaoszczędza czas. | MEDIUM | Deep-clone sekcji JS + renumeracja. |
| Walidacja inline (czerwone pole gdy Zs > Zsmax) | UK tools (iCertifi v12.12.30) mają "real-time validation". Polskie narzędzia desktopowe oceniają dopiero po druku. Natychmiastowy feedback = mniej błędów. | LOW | CSS klasa `.invalid` + JS event listener. |
| Wiele protokołów w localStorage (lista dokumentów) | PROTON+ i Sonel PE6 mają archiwizację. Dla SPA: zamiast jednego klucza w localStorage — lista z nazwami/datami. | MEDIUM | Wymaga zarządzania stanem + UI listy. Ryzyko: localStorage limit ~5MB. |

### Anti-Features (Commonly Requested, Often Problematic)

Funkcje, które wydają się wartościowe, ale tworzą problemy przekraczające korzyści — szczególnie w kontekście single-file HTML + vanilla JS.

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| Backend / baza danych w chmurze | "Chcę dostęp z każdego komputera" | Wymaga serwera, autentykacji, utrzymania, RODO. Niszczy propozycję "zero kosztów". Forum ISE wskazuje na złożoność danych wielodostępowych. | Export/import JSON — użytkownik sam zarządza plikami. |
| Konta użytkowników / logowanie | "Chcę mieć swoje protokoły zawsze dostępne" | Autentykacja to projekt sam w sobie. Przy single-file HTML jest po prostu niemożliwa sensownie. | localStorage + export JSON to wystarczający poziom persystencji dla v1. |
| Biblioteka zdjęć / załączniki do protokołu | Sonel PE6 ma "wstawianie zdjęć i rysunków". Elektrycy dokumentują usterki zdjęciami. | Binary data w localStorage = szybkie wyczerpanie 5MB. Implementacja złożona. Nie jest wymagana przez PN-HD 60364-6 dla podstawowego protokołu. | Defer do v2 gdy przeniesienie na IndexedDB lub backend będzie rozważone. |
| Inne rodzaje protokołów (odgromowe, fotowoltaika, oświetlenie) | Arkusz ZPE24 ma protokoły PV i oświetlenia. Użytkownicy mogą prosić o rozszerzenie. | Scope creep. Każdy nowy typ to osobna logika, nowe obliczenia, nowe bazy danych. | Osobne projekty/pliki. VoltProtokół = tylko PN-HD 60364-6 elektryka. |
| Import danych z miernika (USB/Bluetooth) | CERTY (Megger) i Sonel PE6 mają integrację z miernikami. Elektrycy chcą "kliknij przenieś dane". | Web Serial API ma ograniczone wsparcie. Protokoły komunikacyjne mierników są własnościowe. Złożoność nie do udźwignięcia w vanilla JS. | Ręczne wpisanie jest standardem dla narzędzi webowych. |
| Drukowanie bezpośrednie (window.print bez PDF) | Prościej technicznie niż jsPDF. | @media print CSS generuje różne wyniki w różnych przeglądarkach. Brak kontroli nad nazwą pliku. Nie tworzy pliku .pdf do pobrania — a to jest explicit requirement projektu. | jsPDF + html2canvas lub html2pdf.js — bardziej deterministyczny output. |
| Tryb wieloużytkownikowy / "Collaborate Live" | UK tool Tradecert ma tę funkcję. "Kilku elektryków wypełnia razem". | Wymaga backendu (WebSockets). Niszczy architekturę SPA. Poza scope projektu. | Nie budować. Export JSON + ręczne scalanie to wystarczające dla małych firm. |
| Automatyczne powiadomienia email o terminach badań | PROTON+ ma "kontrolę terminów". Użytkownicy forum ISE chcieli SMS/email. | Wymaga backendu + bazy danych + schedulera. Niemożliwe w client-only SPA. | Kalendarz dat następnego badania w PDF jest wystarczający dla v1. |

---

## Feature Dependencies

```
[Formularz danych ogólnych]
    └──wymagane przez──> [Eksport PDF]
                             └──wymagane przez──> [Baza zabezpieczeń]

[Baza zabezpieczeń (B/C/D + Ia)]
    └──wymagane przez──> [Obliczenia Zsmax = Usk / Ia]
                             └──wymagane przez──> [Ocena Zs ≤ Zsmax]
                                                      └──wymagane przez──> [Eksport PDF]

[Dynamiczne sekcje/wiersze]
    └──wymagane przez──> [Załącznik SWZ]
    └──wymagane przez──> [Załącznik izolacja]
    └──wymagane przez──> [Załącznik RCD]
    └──wymagane przez──> [Załącznik uziemienie]

[localStorage zapis/odczyt]
    └──enhances──> [Eksport JSON]  (JSON = przenośna kopia localStorage)

[Kopiowanie sekcji] ──requires──> [Dynamiczne sekcje/wiersze]

[Wiele protokołów w localStorage] ──requires──> [localStorage zapis/odczyt]

[Walidacja inline] ──requires──> [Obliczenia automatyczne]

[Podgląd print-preview] ──requires──> [Eksport PDF] (lub przynajmniej @media print CSS)

[Ręczna korekta Ia] ──enhances──> [Baza zabezpieczeń]

[Ocena końcowa + data następnego badania] ──requires──> [Formularz danych ogólnych] (data badania)
```

### Dependency Notes

- **Baza zabezpieczeń wymaga Obliczeń Zsmax:** Bez wartości Ia z bazy obliczenie Zsmax=Usk/Ia nie może być automatyczne. Baza jest precondition dla automatycznej oceny w SWZ.
- **Dynamiczne sekcje wymagane przez wszystkie załączniki:** To jest fundamentalna architektura — musi być zaprojektowana przed implementacją jakiegokolwiek załącznika.
- **Eksport JSON enhances localStorage:** Gdy mamy localStorage, JSON export to jeden przycisk. Tanie wzmocnienie wartości.
- **Walidacja inline conflicts with "pokaż tylko na wydruku":** Jeśli użytkownik zobaczy czerwone pola podczas wypełniania, nie oczekuje że pojawią się w PDF. Trzeba zadbać o rozdzielenie stylów screen vs print.

---

## MVP Definition

### Launch With (v1)

Minimum viable product — kompletny protokół PN-HD 60364-6 możliwy do wygenerowania i zapisania.

- [ ] Formularz danych ogólnych — bez tego nie ma "nagłówka" protokołu
- [ ] Załącznik 1 (SWZ) z obliczeniami Id, Zsmax i oceną — najważniejsze badanie, highest stakes
- [ ] Załącznik 2 (rezystancja izolacji) — obowiązkowe badanie odbiorcze
- [ ] Załącznik 3 (RCD) — obowiązkowe badanie odbiorcze
- [ ] Załącznik 4 (uziemienie) z obliczeniem Rpo — obowiązkowe
- [ ] Baza zabezpieczeń B/C/D + ręczna korekta Ia — bez tego Załącznik 1 jest manualny
- [ ] Dynamiczne sekcje/wiersze we wszystkich załącznikach — bez tego narzędzie jest nieużyteczne dla realnych obiektów
- [ ] Eksport PDF — to jest core value całego produktu
- [ ] Zapis/odczyt localStorage — bez tego użytkownik traci dane przy odświeżeniu
- [ ] Ocena końcowa + data następnego badania — wymagana przez normę
- [ ] Legendy oznaczeń — zawarte we wzorze referencyjnym, niski koszt

### Add After Validation (v1.x)

Funkcje do dodania gdy core działa poprawnie.

- [ ] Eksport/import JSON stanu — gdy użytkownicy zgłoszą potrzebę przenoszenia danych między komputerami
- [ ] Walidacja inline (czerwone pola dla błędnych wartości) — gdy użytkownicy popełniają błędy podczas wypełniania
- [ ] Kopiowanie sekcji (duplikowanie kondygnacji) — gdy użytkownicy zgłoszą że ręczne dodawanie kolejnych sekcji jest uciążliwe
- [ ] Automatyczne wyliczanie daty następnego badania — trywialne, warto dodać w v1.1

### Future Consideration (v2+)

Funkcje do rozważenia po ustaleniu product-market fit.

- [ ] Wiele protokołów w localStorage (lista dokumentów z archiwum) — wymaga redesignu UI, ryzyko limitów localStorage
- [ ] Podgląd print-preview — wartościowe UX, ale złożona implementacja bez degradacji jakości PDF
- [ ] Zdjęcia/załączniki — wymaga IndexedDB lub backend, poza scope v1

---

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| Eksport PDF | HIGH | MEDIUM | P1 |
| Formularz danych ogólnych | HIGH | LOW | P1 |
| Załącznik SWZ z obliczeniami | HIGH | MEDIUM | P1 |
| Załącznik rezystancja izolacji | HIGH | LOW | P1 |
| Załącznik RCD | HIGH | LOW | P1 |
| Załącznik uziemienie | HIGH | LOW | P1 |
| Baza zabezpieczeń B/C/D | HIGH | MEDIUM | P1 |
| Dynamiczne sekcje/wiersze | HIGH | MEDIUM | P1 |
| localStorage zapis/odczyt | HIGH | LOW | P1 |
| Ocena końcowa + data następnego badania | HIGH | LOW | P1 |
| Legendy oznaczeń | MEDIUM | LOW | P1 |
| Ręczna korekta Ia | MEDIUM | LOW | P1 |
| Eksport/import JSON | MEDIUM | LOW | P2 |
| Walidacja inline | MEDIUM | LOW | P2 |
| Kopiowanie sekcji | MEDIUM | MEDIUM | P2 |
| Auto data następnego badania | LOW | LOW | P2 |
| Wiele protokołów w localStorage | MEDIUM | HIGH | P3 |
| Podgląd print-preview | MEDIUM | MEDIUM | P3 |
| Zdjęcia/załączniki | LOW | HIGH | P3 |

**Priority key:**
- P1: Must have dla v1 launch
- P2: Should have, add w v1.x
- P3: Nice to have, v2+ consideration

---

## Competitor Feature Analysis

Analiza istniejących narzędzi dostępnych dla polskich elektryków. Brak bezpośrednich webowych odpowiedników — VoltProtokół byłby pierwszy w tej niszy.

| Feature | Sonel PE6 (desktop, płatny) | PROTON+ (desktop, płatny) | Arkusz Excel ZPE24 (płatny) | VoltProtokół (plan) |
|---------|----------------------------|--------------------------|-----------------------------|---------------------|
| Instalacja | Wymagana (Windows) | Wymagana (Windows) | Excel 2010+ | Brak — przeglądarka |
| Koszt | ~500–900 PLN | Płatny | ~50–100 PLN | Darmowy |
| Protokół SWZ (impedancja pętli) | TAK | TAK | TAK | TAK |
| Protokół izolacji | TAK | TAK | TAK | TAK |
| Protokół RCD | TAK | TAK | TAK | TAK |
| Protokół uziemienia | TAK | TAK | TAK | TAK |
| Automatyczne obliczenia | TAK | TAK | TAK (Excel) | TAK |
| Baza zabezpieczeń | TAK (rozbudowana) | TAK | Ograniczona | TAK (B/C/D) |
| Eksport PDF | TAK | TAK | Przez Excel | TAK |
| Archiwizacja wielu protokołów | TAK | TAK | Pliki Excel | v1.x (localStorage) |
| Import z miernika | TAK (Sonel) | NIE | NIE | NIE (anti-feature) |
| Zdjęcia w protokole | TAK | TAK | NIE | NIE (anti-feature v1) |
| Terminy następnych badań | TAK | TAK | NIE | Inline (auto +5 lat) |
| Otwarto-źródłowy | NIE | NIE | NIE | TAK |
| Dostęp bez instalacji | NIE | NIE | NIE | TAK |
| Dodatkowe typy protokołów | PV, oświetlenie i inne | Termowizja, spawarki | PV, oświetlenie | NIE (focus) |

**Kluczowy wniosek:** VoltProtokół nie wygra funkcjami zaawansowanymi (import z miernika, zdjęcia, archiwum wielu protokołów). Wygrywa bezbarierowym dostępem (zero instalacji, zero kosztu, zero rejestracji) i otwartym kodem.

---

## Sources

- Sonel PE6 features: [Program SONEL Pomiary Elektryczne 6 - Metris](https://www.metris.pl/produkt/program-sonel-pomiary-elektryczne-6-pe6/) — MEDIUM confidence (producent)
- iiione Pomiary Elektryczne: [Program iiione Pomiary Elektryczne 3.2](https://www.iiione.com/software/program-pomiary-elektryczne.aspx) — MEDIUM confidence (producent)
- PROTON+: [PROTON+ Badania i Pomiary Elektryczne](https://proton.ise.pl) — LOW confidence (strona marketingowa, brak szczegółów technicznych)
- CERTY Megger: [CERTY Program do automatycznego tworzenia protokołów](https://centrummiernictwa.pl/produkt/certy-program-do-automatycznego-tworzenia-protokolow-megger/) — MEDIUM confidence
- Arkusz ZPE24: [Arkusz do pomiarów elektrycznych v1.1](https://zpe24.pl/shop/arkusz-do-pomiarow-elektrycznych/) — MEDIUM confidence
- UK electrical cert software comparison: [Best 18th Edition Certification Software For 2025](https://electrical-assistance.co.uk/18th-edition-certification-software/) — MEDIUM confidence
- iCertifi AI features: [iCertifi 12.12.30 Release](https://icertifi.co.uk/icertifi-12-12-30-release-ai-features-that-save-2-hours-per-certificate-board-vision-test-vision/) — HIGH confidence (official changelog)
- PDF generation approaches: [Creating PDFs from HTML + CSS in JavaScript: What actually works](https://joyfill.io/blog/creating-pdfs-from-html-css-in-javascript-what-actually-works) — MEDIUM confidence
- Dyskusje elektryków o narzędziach: [Forum ISE - Jaki program do protokołów](https://forum.ise.pl/viewtopic.php?t=4371) — MEDIUM confidence
- Norma PN-HD 60364-6 zakres: [Portal Elektryka - sprawdzanie instalacji](https://portalelektryka.pl/normy/pnhd-603646201607-sprawdzanie-instalacji-elektrycznych-niskiego-napiecia-3110.html) — HIGH confidence

---
*Feature research for: Generator protokołów pomiarowych instalacji elektrycznych (PN-HD 60364-6)*
*Researched: 2026-02-23*
