# Roadmap: VoltProtokół (SEP-Calc)

## Overview

Aplikacja jest budowana jako kompletny generator protokołów PN-HD 60364-6 w jednym pliku HTML, bez instalacji i rejestracji. Trzy fazy dostarczają kompletny produkt: najpierw dynamiczne formularze z obliczeniami elektrycznymi, potem meta-dane protokołu wymagane normą, na końcu eksport PDF i persystencja danych.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [x] **Phase 1: Formularze i Obliczenia** — Wszystkie 4 załączniki z dynamicznymi sekcjami, obliczeniami i bazą zabezpieczeń
- [x] **Phase 2: Kompletność Normowa** — Formularz główny z danymi ogólnymi, ocena końcowa i legendy
- [ ] **Phase 3: Eksport i Dane** — PDF do pobrania, localStorage, JSON import/eksport

## Phase Details

### Phase 1: Formularze i Obliczenia
**Goal**: Elektryk może wypełnić kompletny formularz pomiarowy z automatycznymi obliczeniami i oceną dla wszystkich 4 badań
**Depends on**: Nothing (first phase)
**Requirements**: DYN-01, DYN-02, DYN-03, DYN-04, DYN-05, DYN-06, SWZ-01, SWZ-02, SWZ-03, SWZ-04, SWZ-05, SWZ-06, SWZ-07, IZOL-01, IZOL-02, IZOL-03, IZOL-04, IZOL-05, IZOL-06, IZOL-07, RCD-01, RCD-02, RCD-03, RCD-04, RCD-05, UZIEM-01, UZIEM-02, UZIEM-03, UZIEM-04, UZIEM-05, UZIEM-06, PROT-01, PROT-02, PROT-03
**Success Criteria** (what must be TRUE):
  1. Użytkownik może dodawać i usuwać sekcje (Parter, Piętro), podtytuły (Pokój 1, Obwód oświetlenia) i wiersze pomiarowe we wszystkich załącznikach — numeracja Lp. aktualizuje się automatycznie
  2. Załącznik 1 (SWZ): po wpisaniu Zs i wyborze zabezpieczenia (np. B16) wartości Id i Zsmax obliczają się automatycznie, ocena POZYTYWNA/NEGATYWNA pojawia się natychmiast
  3. Załącznik 2 (izolacja): dla obwodów 1-fazowych aktywne są tylko pola L-N, L-PE, N-PE; dla 3-fazowych wszystkie 10 kombinacji; ocena generuje się na podstawie Rw=1MΩ
  4. Załącznik 3 (RCD) i Załącznik 4 (uziemienie): pomiary i obliczenia Rpo=Rp×Wk z automatyczną oceną działają poprawnie
  5. Sekcje i podtytuły z Zał. 1 są wspólne dla Zał. 2 — zmiana w jednym automatycznie odzwierciedla się w drugim
**Plans:** 3 plans

Plans:
- [x] 01-01-PLAN.md — Architektura (AppState, EventBus, PROTECTION_DB, Calculator) + dynamiczne sekcje/podtytuły/wiersze z Lp.
- [x] 01-02-PLAN.md — Załącznik 1 (SWZ) — pełna tabela 13 kolumn, obliczenia Id/Zsmax, baza zabezpieczeń B/C/D, ocena
- [x] 01-03-PLAN.md — Załącznik 2 (izolacja 1-faz/3-faz), Załącznik 3 (RCD), Załącznik 4 (uziemienie) + weryfikacja wizualna

### Phase 2: Kompletność Normowa
**Goal**: Protokół jest kompletny zgodnie z PN-HD 60364-6 — zawiera dane ogólne, orzeczenie o zdatności i legendy oznaczeń
**Depends on**: Phase 1
**Requirements**: FORM-01, FORM-02, FORM-03, FORM-04, FORM-05, FORM-06, FORM-07
**Success Criteria** (what must be TRUE):
  1. Użytkownik może wypełnić numer protokołu (auto-generowany z datą), dane obiektu, dane wykonawcy z numerami SEP, datę badania i temperaturę
  2. Użytkownik może wpisać przyrządy pomiarowe (nazwa, nr seryjny, nr świadectwa wzorcowania) dla 4 typów pomiarów
  3. Tabela oceny końcowej generuje się automatycznie na podstawie wyników z 4 załączników, orzeczenie zawiera datę następnego badania (+5 lat)
**Plans:** 1 plan

Plans:
- [ ] 02-01-PLAN.md — Zakładka Protokół (tab 0): AppState.form, pola metadanych FORM-01–05, tabela przyrządów, ocena końcowa FORM-06, orzeczenie FORM-07

### Phase 3: Eksport i Dane
**Goal**: Elektryk może wygenerować profesjonalny plik PDF do pobrania, zapisać i wczytać stan formularza
**Depends on**: Phase 2
**Requirements**: EXP-01, EXP-02, EXP-03, EXP-04, EXP-05, EXP-06, EXP-07, EXP-08
**Success Criteria** (what must be TRUE):
  1. Kliknięcie "Eksportuj PDF" pobiera plik .pdf z pełną treścią protokołu — polskie znaki (ą, ę, ś, ź, ć, ó, ł, ń, ż) wyświetlają się poprawnie, dokument jest czarno-biały z numeracją stron
  2. Przyciski "Zapisz" i "Wczytaj" przechowują i odtwarzają dane z localStorage — żaden wpis nie ginie między sesjami przeglądarki
  3. Użytkownik może wyeksportować dane do pliku .json i wczytać je z powrotem — działa jako przenoszenie protokołu między komputerami
**Plans**: TBD

Plans:
- [ ] 03-01: PDFExporter (pdfmake DDO, polskie znaki, paginacja, orientacja landscape Zał. 2)
- [ ] 03-02: Persistence (localStorage + JSON export/import)

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Formularze i Obliczenia | 3/3 | Complete | 2026-02-24 |
| 2. Kompletność Normowa | 1/1 | Complete | 2026-02-24 |
| 3. Eksport i Dane | 0/2 | Not started | - |
