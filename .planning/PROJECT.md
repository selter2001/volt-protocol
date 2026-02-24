# VoltProtocol — Luxury Redesign

## What This Is

Redesign wizualny aplikacji VoltProtocol — generatora protokołów kontrolno-pomiarowych PN-HD 60364-6. Przekształcenie technicznego narzędzia elektryka w premium interfejs w stylu "Industrial Elegance" (Tesla App × Apple Health), zachowując 100% obecnej funkcjonalności w jednym pliku HTML.

## Core Value

Użytkownik otwiera aplikację i natychmiast czuje, że pracuje z narzędziem klasy premium — precyzja techniczna w luksusowym opakowaniu.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Void Black tło z subtlną noise texture jako baza wizualna
- [ ] Glassmorphism 2.0 (frosted glass) na polach input z ultra-thin (0.5px) Electric Cyan borderami
- [ ] Bento Cards z 24px rounded corners zamiast standardowych tabel
- [ ] Monospaced font na wartości numeryczne, premium sans/serif na labele
- [ ] Spring physics animacje na zmianie wyników (count-up effect)
- [ ] "Flow Effect" — świecące animowane linie łączące sekcje (symulacja prądu)
- [ ] Neomorphic-depth efekt na buttonach (haptic pressed-in feel)
- [ ] Glowing focus states na inputach
- [ ] Hero Result Card z grain-gradient tłem i dużą typografią na końcową ocenę
- [ ] Kolorystyka: Void Black (#0a0a0f) tło, Electric Cyan (#00f0ff) akcenty interaktywne, Soft Platinum (#e0e0e8) tekst/bordery
- [ ] 100% zachowanie obecnych funkcji (PDF, Word, JSON export/import, localStorage, obliczenia)
- [ ] Cały kod w jednym pliku HTML (obecna architektura)
- [ ] Subtlne OLED-style glow na wykresach/wynikach

### Out of Scope

- Zmiana architektury kodu (rozbicie na pliki, framework) — zostajemy przy single HTML
- Nowa funkcjonalność (nowe formularze, nowe obliczenia) — to jest redesign, nie feature add
- Backend/server — aplikacja pozostaje 100% klient-side
- Płatne fonty (Aeonik, etc.) — używamy darmowych odpowiedników premium quality

## Context

- Obecna aplikacja: ~119KB single HTML file z Vanilla JS + Tailwind CSS v4 (Play CDN) + pdfmake
- 5 zakładek: Protokół + 4 załączniki pomiarowe (SWZ, Izolacja, RCD, Uziemienie)
- Automatyczne obliczenia: Id, Zsmax, Ia, Rpo z verdyktem POZYTYWNA/NEGATYWNA
- Baza zabezpieczeń B/C/D z prądami 6-125A
- Dynamiczne sekcje (dodawanie/usuwanie wierszy)
- Export: PDF (z polskimi znakami, landscape dla Zał.2), Word (.docx), JSON
- Import: JSON, Word, localStorage
- Grupa docelowa: Polscy elektrycy wykonujący pomiary — potrzebują narzędzia, które wygląda profesjonalnie i wzbudza zaufanie klientów

## Constraints

- **Architektura**: Single HTML file — bez build step, bez bundlera, bez frameworka
- **Funkcjonalność**: Wszystkie obecne funkcje muszą działać identycznie po redesignie
- **Fonty**: Tylko darmowe (Google Fonts/CDN) — dobrane do estetyki premium
- **Kompatybilność**: Tailwind CSS v4 Play CDN + pdfmake 0.3.5 (obecny stack)
- **Performance**: Animacje nie mogą spowalniać formularzy na słabszych urządzeniach
- **PDF Export**: Redesign nie wpływa na wygenerowany PDF (PDF ma osobny styl)

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Tylko redesign wizualny, bez zmiany architektury | Użytkownik chce szybki efekt wow, nie przebudowę | — Pending |
| Void Black + Cyan + Platinum mix kolorystyka | Łączy oba kierunki — czerń jako baza, cyan na interakcje, platinum na treść | — Pending |
| Darmowe fonty dobrane przez Claude | Elastyczność wyboru najlepszego darmowego dopasowania do "Industrial Elegance" | — Pending |
| Pełne animacje z briefu (spring, glow, flow) | Użytkownik chce pełny efekt premium, nie kompromis | — Pending |

---
*Last updated: 2026-02-24 after initialization*
