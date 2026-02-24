---
phase: 3
plan: 03-01
title: "Stress Test & Flow Effect Decision"
requirements: [PRES-01, PRES-02, PRES-03, PRES-04, PRES-05, PRES-06, ANIM-04]
depends_on: [02-01]
estimated_tasks: 9
---

# Plan 03-01: Stress Test + Flow Effect Decision

## Goal

Zweryfikować, że po pełnym redesignie wizualnym (Phase 1 + Phase 2) wszystkie funkcje aplikacji działają identycznie: eksporty (PDF, Word, JSON), localStorage, obliczenia, dynamiczne operacje. Podjąć świadomą decyzję o Flow Effect (ANIM-04).

## Context

Eksporty PDF i Word używają hardkodowanych kolorów — są CAŁKOWICIE izolowane od zmian CSS. JSON i localStorage serializują AppState bez odwołań do DOM. Obliczenia to czyste funkcje. Ryzyko regresji jest NISKIE, ale weryfikacja jest wymagana przez PRES-01 do PRES-06.

Flow Effect (ANIM-04: animowane SVG linie między sekcjami) nie ma sensu w tabbed layout — użytkownik widzi tylko jedną zakładkę naraz. Rekomendacja: świadome pominięcie.

## Files to Modify

Główny plik: `index.html` — TYLKO jeśli stress test ujawni bug. Plan jest przede wszystkim weryfikacyjny.

## Tasks

### TASK-01: Flow Effect — decyzja (ANIM-04)

**Decyzja**: Świadome pominięcie ANIM-04.

**Uzasadnienie**:
1. Tabbed layout — sekcje nie są widoczne jednocześnie, SVG linie między zakładkami nie mają sensu wizualnego
2. Zero istniejącej infrastruktury SVG — budowa od zera
3. Wysoki koszt performance (SVG overlay + resize observer + animacje)
4. Niski zysk UX — efekt dekoracyjny w narzędziu technicznym

**Akcja**: Oznaczyć ANIM-04 jako "Skipped (tabbed layout)" w REQUIREMENTS.md.

### TASK-02: Stress Test — dodanie 20+ wierszy danych

Wypełnić aplikację pełnym zestawem testowych danych:
- Formularz główny: wszystkie pola wypełnione (numer protokołu, dane zleceniodawcy, daty, itd.)
- Zał. 1 (SWZ): 2 sekcje × 2 podsekcje × 3 wiersze = 12+ wierszy + wiersz stały
- Zał. 2 (IZOL): 2 sekcje × 2 podsekcje × 3 wiersze = 12+ wierszy + wiersz WLZ
- Zał. 3 (RCD): 5+ wierszy
- Zał. 4 (UZIEM): 3+ wiersze

**Check**: Aplikacja nie janku je, edycja jest płynna, re-rendery nie blokują UI.

### TASK-03: Weryfikacja obliczeń (PRES-05)

Sprawdzić poprawność obliczeń na danych testowych:

| Obliczenie | Input | Oczekiwany output |
|------------|-------|-------------------|
| SWZ: Id | Usk=230, Zs=0.5 | Id=460.00 |
| SWZ: Zsmax | Usk=230, Ia=160 (C16) | Zsmax=1.44 |
| SWZ: verdict | Zs=0.5 ≤ Zsmax=1.44 | POZYTYWNA |
| IZOL: verdict | Rp L1-N=500, Rw=1 | POZYTYWNA |
| RCD: verdict | Iw=25, In=30, tw=20, tz=300 | POZYTYWNA |
| UZIEM: Rpo | Rp=5, Wk=1.5 | Rpo=7.50 |
| UZIEM: verdict | Rpo=7.50 ≤ Rw=10 | POZYTYWNA |

**Check**: `calcSWZRow()`, `calcIZOLRow()`, `calcRCDRow()`, `calcUZIEMRow()` zwracają poprawne wyniki. Animacje count-up wyświetlają finalne wartości.

### TASK-04: Weryfikacja dynamicznych operacji (PRES-06)

Przetestować cykl życia sekcji/wierszy:
1. Dodaj sekcję → dodaj podsekcję → dodaj wiersze (SWZ i IZOL)
2. Edytuj tytuły sekcji i podsekcji
3. Usuń wiersz → sprawdź renumerację Lp.
4. Usuń podsekcję → sprawdź usunięcie wierszy z obydwu attachmentów
5. Usuń sekcję → sprawdź kaskadowe usunięcie
6. Dodaj wiersz RCD → edytuj → usuń
7. Dodaj wiersz UZIEM → edytuj → usuń

**Check**: Wszystkie operacje CRUD działają identycznie. EventBus emituje poprawne eventy. Re-rendery nie zostawiają "duchów" w DOM.

### TASK-05: Weryfikacja JSON export/import (PRES-03)

1. Wypełnij aplikację danymi testowymi (TASK-02)
2. Eksportuj JSON → otwórz plik, sprawdź strukturę AppState
3. Odśwież stronę (wyczyść stan)
4. Importuj JSON → sprawdź czy wszystkie dane przywrócone
5. Porównaj: obliczone wartości, verdykty, stany selectów

**Check**: JSON round-trip zachowuje 100% danych. `restoreAppState()` przelicza wszystko poprawnie.

### TASK-06: Weryfikacja localStorage (PRES-04)

1. Wypełnij aplikację danymi testowymi
2. Kliknij "Zapisz" → sprawdź toast "Zapisano"
3. Odśwież stronę (F5)
4. Kliknij "Wczytaj" → sprawdź toast "Wczytano"
5. Porównaj: wszystkie pola, obliczone wartości, verdykty

**Check**: localStorage round-trip zachowuje 100% danych. QuotaExceededError handling działa.

### TASK-07: Weryfikacja PDF export (PRES-01)

1. Wypełnij aplikację pełnym zestawem danych
2. Kliknij "Eksportuj PDF"
3. Sprawdź w wygenerowanym PDF:
   - Jasne tło (NIE dark theme — PDF ma własne kolory)
   - Polskie znaki: ą, ć, ę, ł, ń, ó, ś, ź, ż
   - Zał. 2 w orientacji landscape
   - Verdykty POZYTYWNA/NEGATYWNA widoczne i poprawne
   - Wszystkie dane testowe obecne w PDF
   - Tabele z poprawną liczbą kolumn (SWZ=13, IZOL=15, RCD, UZIEM)

**Check**: PDF identyczny z wersją sprzed redesignu. Hardkodowane kolory niezmodyfikowane.

### TASK-08: Weryfikacja Word export/import (PRES-02)

1. Eksportuj Word → otwórz w przeglądarce/edytorze
2. Sprawdź strukturę dokumentu, polskie znaki, landscape Zał.2
3. Importuj ten sam plik z powrotem → sprawdź round-trip danych
4. Porównaj: AppState po imporcie vs przed eksportem

**Check**: Word round-trip działa. Base64 payload w komentarzu HTML poprawnie zakodowany i dekodowany.

### TASK-09: Weryfikacja prefers-reduced-motion (ANIM-07 re-check)

Sprawdzić, że z włączonym `prefers-reduced-motion: reduce` w przeglądarce:
1. Animacje count-up (animateCountUp) przeskakują od razu do wartości docelowej
2. Verdict glow (text-shadow) jest statyczny (brak pulsacji)
3. Przycisk :active efekt (neomorphic press) działa natychmiastowo
4. Wszystkie zakładki przełączają się normalnie
5. Obliczenia i eksporty działają identycznie

Sprawdzić w CSS:
- `@media (prefers-reduced-motion: reduce)` istnieje i wyłącza `animation`, `transition`
- `animateCountUp()` sprawdza `window.matchMedia('(prefers-reduced-motion: reduce)')` i ustawia wartość bez animacji

**Check**: Z pełnym zestawem danych (20+ wierszy) i prefers-reduced-motion: reduce — aplikacja działa normalnie, brak żadnych animacji keyframe.

## Automated Checks

```
CHECK-01: [PRES-05] calcSWZRow({Usk:230, Zs:0.5, baseType:'C', baseCurrent:16}).Id === 460
CHECK-02: [PRES-05] calcSWZRow({Usk:230, Zs:0.5, baseType:'C', baseCurrent:16}).Zsmax === 1.4375 (rounded)
CHECK-03: [PRES-05] calcSWZRow({Usk:230, Zs:0.5, baseType:'C', baseCurrent:16}).verdict === 'POZYTYWNA'
CHECK-04: [PRES-05] calcIZOLRow with all Rp >= Rw → 'POZYTYWNA'
CHECK-05: [PRES-05] calcRCDRow({Iw:25, In:30, tw:20, tz:300}).verdict === 'POZYTYWNA'
CHECK-06: [PRES-05] calcUZIEMRow({Rp:5, Wk:1.5, Rw:10}).Rpo === 7.5
CHECK-07: [PRES-05] calcUZIEMRow({Rp:5, Wk:1.5, Rw:10}).verdict === 'POZYTYWNA'
CHECK-08: [PRES-03] JSON.stringify(AppState) → JSON.parse() → restoreAppState() preserves all data
CHECK-09: [PRES-04] localStorage.setItem/getItem round-trip preserves all data
CHECK-10: [PRES-06] addSection → sections.length increases by 1
CHECK-11: [PRES-06] addSubsection → section.subsections.length increases by 1
CHECK-12: [PRES-06] addRow-swz → rowsBySubsection[subId].length increases by 1
CHECK-13: [PRES-06] removeRow → rowsBySubsection[subId].length decreases by 1
CHECK-14: [PRES-06] removeSection → cascades removal of all subsection rows
CHECK-15: [PRES-01] exportPDF() calls pdfMake.createPdf() without error
CHECK-16: [PRES-02] exportToWord() generates Blob without error
CHECK-17: [ANIM-04] ANIM-04 marked as Skipped in REQUIREMENTS.md
CHECK-18: [PRES-01] buildAttachment2Content() sets pageOrientation: 'landscape'
CHECK-19: [PRES-02] buildWordHTML() contains @page Section2 with landscape
CHECK-20: [PRES-08] index.html remains single file, no external dependencies added
CHECK-21: [ANIM-07] @media (prefers-reduced-motion: reduce) rule exists in CSS and disables animation/transition
CHECK-22: [ANIM-07] animateCountUp() checks matchMedia('(prefers-reduced-motion: reduce)') before animating
```

## Risk Assessment

- **Ryzyko regresji eksportów**: NISKIE — PDF/Word/JSON/localStorage są izolowane od CSS
- **Ryzyko performance**: NISKIE — obliczenia to O(n) na liczbę wierszy, 20+ wierszy to nic
- **Flow Effect**: ZERO RYZYKA — świadomie pomijamy
- **Jedyne realne ryzyko**: Jeśli `restoreAppState()` nie obsługuje nowych pól dodanych w Phase 2 (np. verdict glow classes) — ale te są generowane dynamicznie z obliczeń, nie zapisywane w AppState

## Success Criteria

1. Wszystkie PRES-01 do PRES-06 zweryfikowane pozytywnie
2. ANIM-04 świadomie odrzucony z udokumentowanym uzasadnieniem
3. Aplikacja z 20+ wierszami działa płynnie
4. Wszystkie 22 automated checks przechodzą
5. prefers-reduced-motion wyłącza animacje, aplikacja działa normalnie z 20+ wierszami
