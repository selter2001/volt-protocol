# VoltProtokół (SEP-Calc)

## What This Is

Darmowa aplikacja webowa (SPA) typu Open Source, hostowana na GitHub Pages, służąca do generowania protokołów kontrolno-pomiarowych instalacji elektrycznych zgodnie z normą PN-HD 60364-6. Przeznaczona dla elektryków wykonujących pomiary odbiorcze i okresowe, którzy potrzebują szybkiego, profesjonalnego narzędzia do tworzenia dokumentacji pomiarowej.

## Core Value

Elektryk może w przeglądarce wypełnić dane pomiarowe i jednym kliknięciem wygenerować kompletny, normatywny protokół kontrolno-pomiarowy w formacie PDF — bez instalacji, bez rejestracji, za darmo.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Formularz główny z danymi ogólnymi (data, temperatura, numer protokołu, dane wykonawcy, przyrządy pomiarowe)
- [ ] Sekcja "Ocena wykonanych badań odbiorczych" z orzeczeniem i datą następnego badania (+5 lat)
- [ ] Załącznik 1 (SWZ) — ochrona przez samoczynne wyłączenie zasilania z obliczeniami Id, Zsmax i automatyczną oceną
- [ ] Załącznik 2 — rezystancja izolacji z polami Rp dla obwodów 1-fazowych i 3-fazowych
- [ ] Załącznik 3 (RCD) — badanie wyłączników różnicowoprądowych
- [ ] Załącznik 4 — uziemienie z obliczeniem Rpo = Rp × Wk i oceną
- [ ] Dynamiczne dodawanie sekcji (Tytuły: Parter, Piętro; Podtytuły: obwody) we wszystkich załącznikach
- [ ] Baza zabezpieczeń (B, C, D) z automatycznym Ia + możliwość ręcznej korekty
- [ ] Legendy oznaczeń pod każdą tabelą
- [ ] Eksport do PDF (przycisk generujący plik .pdf do pobrania)
- [ ] Zapis/Wczytaj dane z localStorage (ręczny, na życzenie użytkownika)
- [ ] Interfejs w całości po polsku

### Out of Scope

- Backend / baza danych — aplikacja działa w 100% w przeglądarce
- Rejestracja użytkowników — brak kont, brak logowania
- Aplikacja mobilna natywna — web-first, responsywność wystarczy
- Inne rodzaje protokołów (np. odgromowe, gazowe) — tylko elektryczne wg PN-HD 60364-6

## Context

- **Technologia:** Jeden plik HTML z Tailwind CSS (CDN) + czysty JavaScript. Biblioteka do PDF (np. jsPDF + html2canvas lub podobna) ładowana z CDN.
- **Hosting:** GitHub Pages — statyczny deploy, zero kosztów.
- **Norma:** PN-HD 60364-6 definiuje wymagania dla badań odbiorczych instalacji elektrycznych.
- **Wzór dokumentu:** Użytkownik posiada referencyjny wzór protokołu (plik zostanie dodany do repo) — układ wydruku ma się na nim wzorować.
- **Grupa docelowa:** Elektrycy z uprawnieniami SEP wykonujący pomiary odbiorcze i okresowe w Polsce.
- **Obliczenia kluczowe:**
  - Załącznik 1: Id = Usk / Zs; Zsmax = Usk / Ia; ocena: Zs ≤ Zsmax → POZYTYWNA
  - Załącznik 4: Rpo = Rp × Wk; ocena: Rpo ≤ Rw → POZYTYWNA
- **Struktura załączników:** Każdy ma stały pierwszy wiersz + wiersz "Tablica rozdzielcza" + dynamiczne sekcje/podsekcje z możliwością dodawania wierszy.
- **Czasy tw w Zał. 1:** Lista wyboru: 0.2s, 0.4s, 5s.
- **Rezystancja izolacji (Zał. 2):** Rw domyślnie 1 MΩ. Obwody 1-faz: L-N, L-PE, N-PE. Obwody 3-faz: L1-N, L2-N, L3-N, L1-L2, L1-L3, L2-L3, L1-PE, L2-PE, L3-PE, N-PE.
- **RCD (Zał. 3):** tz domyślnie 300ms.
- **Uziemienie (Zał. 4):** Rw domyślnie 10 Ω.

## Constraints

- **Tech stack**: Jeden plik HTML + Tailwind CDN + vanilla JS + biblioteka PDF z CDN — brak bundlerów, brak frameworków
- **Hosting**: GitHub Pages — tylko statyczne pliki
- **Licencja**: Open Source
- **Język UI**: Polski w całości

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Jeden plik HTML zamiast frameworka | Prostota, zero konfiguracji, łatwy deploy na GH Pages | — Pending |
| jsPDF/html2canvas do PDF zamiast window.print() | Użytkownik chce przycisk "Eksportuj PDF" z pobraniem pliku | — Pending |
| localStorage z ręcznym zapisem/odczytem | Kompromis — dane nie giną, ale użytkownik kontroluje kiedy zapisać | — Pending |
| Baza zabezpieczeń + ręczna korekta Ia | Wygoda automatyzacji z elastycznością dla nietypowych przypadków | — Pending |

---
*Last updated: 2026-02-23 after initialization*
