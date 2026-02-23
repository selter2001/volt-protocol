---
phase: 01-formularze-i-obliczenia
plan: 01
subsystem: ui
tags: [vanilla-js, tailwind, event-bus, appstate, single-file-app, dynamic-forms]

# Dependency graph
requires: []
provides:
  - "AppState jako jedyne zrodlo prawdy (sections, swzFixedRow, attachment1-4)"
  - "EventBus (pub/sub) do komunikacji miedzy modulami"
  - "PROTECTION_DB z charakterystykami B/C/D i getIa()"
  - "Calculator: calcSWZRow, calcIZOLRow, calcRCDRow, calcUZIEMRow z roundTo3()"
  - "System zakladek (4 tabs) z event delegation"
  - "Dynamiczne sekcje/podtytuly/wiersze z automatyczna numeracja Lp."
  - "DYN-06: wspoldzielone sekcje miedzy Zal.1 i Zal.2"
  - "Row factories: createNewSWZRow, createNewIZOLRow, createNewRCDRow, createNewUZIEMRow"
affects: [01-02-PLAN, 01-03-PLAN]

# Tech tracking
tech-stack:
  added: [tailwindcss-browser-v4, pdfmake-0.3.5]
  patterns: [single-file-spa, appstate-source-of-truth, eventbus-pubsub, data-action-delegation, computed-lp-numbering]

key-files:
  created: [index.html]
  modified: []

key-decisions:
  - "AppState.sections jako jedna wspoldzielona tablica dla Zal.1 i Zal.2 (DYN-06)"
  - "Lp. nigdy nie przechowywane w AppState - obliczane w renderze (DYN-05)"
  - "Event delegation na #app z data-action zamiast inline onclick"
  - "rowsBySubsection inicjalizowane dla OBU zalacznikow przy tworzeniu subsection (Pitfall 5)"
  - "Section title edit w Zal.1 re-renderuje tylko Zal.2 (nie Zal.1 - unikniecie utraty focusu)"

patterns-established:
  - "EventBus.on/emit do komunikacji miedzy modulami"
  - "data-action + event delegation zamiast inline handlers"
  - "Row factories z crypto.randomUUID()"
  - "roundTo3() przy wszystkich porownaniach float"
  - "parseFloat(val) || null (nie || 0)"

# Metrics
duration: 4min
completed: 2026-02-24
---

# Plan 01-01: Szkielet HTML + AppState + dynamiczne sekcje Summary

**Single-file SPA z AppState, EventBus, PROTECTION_DB, Calculator, systemem 4 zakladek i dynamicznymi sekcjami/podtytulami/wierszami ze wspoldzieleniem miedzy Zal.1 i Zal.2**

## Performance

- **Duration:** 4 min
- **Started:** 2026-02-23T23:30:35Z
- **Completed:** 2026-02-23T23:34:39Z
- **Tasks:** 2
- **Files modified:** 1

## Accomplishments
- AppState jako jedyne zrodlo prawdy z pelna struktura danych dla 4 zalacznikow
- EventBus (pub/sub IIFE) dla zdekuplowanej komunikacji miedzy modulami
- PROTECTION_DB (B/C/D, 6-63A) z getIa() i Calculator z 4 funkcjami obliczeniowymi + roundTo3()
- System zakladek z event delegation (data-tab)
- Dynamiczne sekcje/podtytuly/wiersze z cascadowym usuwaniem i automatyczna numeracja Lp.
- DYN-06: wspoldzielone sekcje miedzy Zal.1 i Zal.2 przez EventBus.on('sections:changed')
- Legendy pod kazdym zalacznikiem (SWZ-07, IZOL-07, RCD-05, UZIEM-06)
- Domyslna zawartosc: sekcja "Parter" z podtytulem i jednym wierszem SWZ

## Task Commits

Each task was committed atomically:

1. **Task 1: Szkielet HTML + AppState + EventBus + PROTECTION_DB + Calculator** - `99c633f` (feat)
2. **Task 2: Dynamiczne sekcje, podtytuly i wiersze + automatyczna numeracja Lp.** - `95bca5f` (feat)

## Files Created/Modified
- `index.html` - Cala aplikacja: HTML + Tailwind CSS + JS (812 linii) z AppState, EventBus, PROTECTION_DB, Calculator, FormRenderer, FormController, init

## Decisions Made
- AppState.sections jako jedna wspoldzielona tablica dla Zal.1 i Zal.2 - wymuszenie DYN-06 od poczatku
- Lp. nigdy nie przechowywane w AppState - obliczane w renderze przez getSWZLpNumber/getIZOLLpNumber
- Edycja nazwy sekcji w Zal.1 re-renderuje tylko Zal.2 (nie caly Zal.1) - zapobiega utracie focusu na input
- rowsBySubsection inicjalizowane dla OBU zalacznikow przy tworzeniu nowej subsection (Pitfall 5 z RESEARCH)
- Wiersz staly "Tablica rozdzielcza" ma Lp.1 i nie moze byc usuniety

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness
- Fundament gotowy: AppState + EventBus + dynamiczne sekcje dzialaja
- Plan 01-02 moze dodawac pelna edycje pol w Zal.1 (SWZ) - szkielety wierszy gotowe
- Plan 01-03 moze dodawac pelna edycje pol w Zal.2/3/4 - szkielety wierszy gotowe
- DYN-06 dziala od poczatku - nie bedzie potrzeby refaktoru

## Self-Check: PASSED

- [x] index.html exists (812 lines, >= 500 min)
- [x] Commit 99c633f found (Task 1)
- [x] Commit 95bca5f found (Task 2)
- [x] 2 commits matching "01-01" in git log

---
*Phase: 01-formularze-i-obliczenia*
*Completed: 2026-02-24*
