# Roadmap: VoltProtocol — Luxury Redesign

## Overview

Redesign wizualny aplikacji VoltProtocol w 3 fazach narzuconych przez zależności techniczne. Faza 1 buduje fundament wizualny (tokeny, tło, glassmorphism, typografia, nawigacja) bez żadnych animacji — każdy krok kończy się testem funkcjonalnym. Faza 2 nakłada warstwę interaktywną (count-up, neomorphic buttons, glow na verdyktach) dopiero po potwierdzeniu, że statyczny redesign nic nie zepsuł. Faza 3 to stress test i Flow Effect — opcjonalny element najwyższego ryzyka performance, wdrażany tylko po pełnej weryfikacji.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

- [x] **Phase 1: Visual Foundation** - Fundament wizualny: tokeny, tlo, glassmorphism, typografia, nawigacja, statyczne stany -- bez animacji
- [ ] **Phase 2: Interactive Layer** - Warstwa animacji: count-up na wynikach, neomorphic buttons, glow na verdyktach, spring transitions
- [ ] **Phase 3: Flow Effect & Stress Test** - Flow Effect (decyzja UX) + pełna weryfikacja pod obciążeniem 20+ wierszy i wszystkich eksportów

## Phase Details

### Phase 1: Visual Foundation
**Goal**: Aplikacja wygląda jak narzędzie klasy premium — Void Black tło, glassmorphism karty, dual-font system, zakładki z dark luxury stylem — i wszystkie funkcje działają identycznie jak przed redesignem.
**Depends on**: Nothing (first phase)
**Requirements**: VIS-01, VIS-02, VIS-03, VIS-04, VIS-05, VIS-06, GLASS-01, GLASS-02, GLASS-03, GLASS-04, GLASS-05, NAV-01, NAV-02, NAV-03, ANIM-01, ANIM-05, ANIM-07, PRES-07, PRES-08
**Success Criteria** (what must be TRUE):
  1. Otwarcie aplikacji w przeglądarce pokazuje Void Black tło z subtlną noise texture i rozmytymi ambient orbs — bez białych/szarych elementów z poprzedniego designu
  2. Wszystkie sekcje formularza i załączniki wyświetlają frosted glass karty z 24px zaokrągleniami, ultra-thin bordarami i glassmorphism tłem
  3. Pola input mają Electric Cyan focus ring po kliknięciu, wartości numeryczne renderują się w monospace, labele w geometric sans
  4. Zakładki (Protokół, Załączniki 1-4) mają dark luxury styling z animowanym cyan wskaźnikiem aktywnej zakładki
  5. Po wpisaniu danych i wygenerowaniu PDF — dokument PDF jest identyczny z wersją sprzed redesignu (jasne tło, polskie znaki)
**Plans**: 2 plans

Plans:
- [x] 01-01-PLAN.md -- Design tokens + body styling (Tailwind @theme, Void Black tlo, noise texture, ambient orbs, prefers-reduced-motion)
- [x] 01-02-PLAN.md -- Glassmorphism components (karty, inputy, tabele, Hero Result Card) + nawigacja (top bar, zakladki, footer)

### Phase 2: Interactive Layer
**Goal**: Aplikacja daje pełne odczucie premium "Industrial Elegance" — wyniki obliczeń animują się count-up'em, przyciski mają haptic pressed-in feel, verdykty świecą odpowiednim kolorem.
**Depends on**: Phase 1
**Requirements**: ANIM-02, ANIM-03, ANIM-06
**Success Criteria** (what must be TRUE):
  1. Po zmianie wartości wejściowych (np. prąd zabezpieczenia) liczby wyników (Id, Zsmax, Ia, Rpo) animują się count-up'em do nowej wartości zamiast przeskakiwać
  2. Kliknięcie przycisku eksportu (PDF, Word, JSON) pokazuje wizualny pressed-in efekt (inset shadow) przez czas kliknięcia
  3. Komórki z verdyktem POZYTYWNA świecą zielonym glow, NEGATYWNA czerwonym — efekt widoczny bez hover
**Plans**: TBD

Plans:
- [ ] 02-01: Animacje wyników (count-up via rAF lub anime.js) + neomorphic button states + verdict glow

### Phase 3: Flow Effect & Stress Test
**Goal**: Opcjonalny Flow Effect wdrożony (lub świadomie odrzucony), aplikacja zweryfikowana pod pełnym obciążeniem danych — 20+ wierszy we wszystkich zakładkach, wszystkie eksporty, prefers-reduced-motion.
**Depends on**: Phase 2
**Requirements**: ANIM-04, PRES-01, PRES-02, PRES-03, PRES-04, PRES-05, PRES-06
**Success Criteria** (what must be TRUE):
  1. Wypełniona zakładka z 20+ wierszami danych działa płynnie — brak janku przy animacjach, przewijaniu i edycji pól
  2. PDF export, Word export i JSON export/import zwracają identyczne wyniki jak przed redesignem przy pełnym zestawie danych
  3. localStorage poprawnie zapisuje i wczytuje stan z pełnym zestawem danych po redesignie
  4. W przeglądarce z prefers-reduced-motion: enabled wszystkie animacje keyframe są wyłączone, aplikacja działa normalnie
**Plans**: TBD

Plans:
- [ ] 03-01: Stress test wszystkich zakładek i eksportów + Flow Effect (decyzja i implementacja lub świadome pominięcie)

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Visual Foundation | 2/2 | Complete | 2026-02-24 |
| 2. Interactive Layer | 0/1 | Not started | - |
| 3. Flow Effect & Stress Test | 0/1 | Not started | - |
