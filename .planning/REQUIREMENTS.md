# Requirements: VoltProtokół (SEP-Calc)

**Defined:** 2026-02-23
**Core Value:** Elektryk może w przeglądarce wypełnić dane pomiarowe i jednym kliknięciem wygenerować kompletny protokół kontrolno-pomiarowy w formacie PDF.

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### Formularz Główny

- [ ] **FORM-01**: Użytkownik może wpisać numer protokołu (auto-generowany z datą, np. 2026/02/001)
- [ ] **FORM-02**: Użytkownik może wpisać dane obiektu badanego (nazwa, adres, działka)
- [ ] **FORM-03**: Użytkownik może wpisać dane wykonawcy (imię, nazwisko, nr SEP eksploatacja, nr SEP dozór)
- [ ] **FORM-04**: Użytkownik może wpisać datę badania i temperaturę otoczenia
- [ ] **FORM-05**: Użytkownik może wpisać przyrządy pomiarowe (nazwa, nr seryjny, nr świadectwa wzorcowania) dla 4 typów pomiarów
- [ ] **FORM-06**: Tabela "Ocena wykonanych badań odbiorczych" generuje się automatycznie z wynikami per załącznik
- [ ] **FORM-07**: Orzeczenie o zdatności instalacji z automatyczną datą następnego badania (+5 lat od daty badania)

### Dynamiczne Sekcje

- [ ] **DYN-01**: Użytkownik może dodawać Tytuły Sekcji (np. Parter, Piętro, Piwnica) we wszystkich załącznikach
- [ ] **DYN-02**: Użytkownik może dodawać Podtytuły (np. Pokój 1 - obwód gniazd, Obwód oświetlenia) pod sekcjami
- [ ] **DYN-03**: Użytkownik może dodawać wiersze pomiarowe pod podtytułami
- [ ] **DYN-04**: Użytkownik może usuwać sekcje, podtytuły i wiersze
- [ ] **DYN-05**: Numeracja Lp. aktualizuje się automatycznie po dodaniu/usunięciu wierszy
- [ ] **DYN-06**: Sekcje i podtytuły są współdzielone między Zał. 1 i Zał. 2 (te same obwody)

### Załącznik 1 — SWZ

- [ ] **SWZ-01**: Stały wiersz "Tablica rozdzielcza - zabezpieczenie główne" na górze tabeli
- [ ] **SWZ-02**: Kolumny tabeli: Lp, Obwód, Zabezpieczenia dodatkowe (typ, prąd), podstawowe (typ, prąd IΔn), Usk, Ia, Id, Zs, Zsmax, tw, Ocena
- [ ] **SWZ-03**: Obliczenie Id = Usk / Zs (automatyczne po wpisaniu wartości)
- [ ] **SWZ-04**: Obliczenie Zsmax = Usk / Ia (automatyczne)
- [ ] **SWZ-05**: Automatyczna ocena: jeśli Zs ≤ Zsmax → POZYTYWNA, inaczej NEGATYWNA
- [ ] **SWZ-06**: Wybór tw z listy: 0.2s, 0.4s, 5s
- [ ] **SWZ-07**: Legenda oznaczeń pod tabelą (In, Ia, Id, tw, Zs, Zsmax, Usk)

### Załącznik 2 — Rezystancja Izolacji

- [ ] **IZOL-01**: Stały pierwszy wiersz (WLZ/Zasilanie)
- [ ] **IZOL-02**: Kolumny: Lp, Nazwa obwodu, Typ przewodu, Rp dla 10 kombinacji żył (L1-N, L2-N, L3-N, L1-L2, L1-L3, L2-L3, L1-PE, L2-PE, L3-PE, N-PE), Rw, Ocena
- [ ] **IZOL-03**: Obwody 1-fazowe: aktywne tylko pola L-N, L-PE, N-PE (reszta szara/nieaktywna)
- [ ] **IZOL-04**: Obwody 3-fazowe: wszystkie 10 pól aktywne
- [ ] **IZOL-05**: Rw domyślnie 1 MΩ
- [ ] **IZOL-06**: Automatyczna ocena: wszystkie Rp ≥ Rw → POZYTYWNA
- [ ] **IZOL-07**: Legenda oznaczeń pod tabelą (Rp, Rw)

### Załącznik 3 — RCD

- [ ] **RCD-01**: Kolumny: Lp, Typ urządzenia, TEST (TAK/NIE), In [mA], Iw [mA], tw [ms], tz [ms], Ocena
- [ ] **RCD-02**: tz domyślnie 300ms
- [ ] **RCD-03**: Automatyczna ocena: Iw ≤ In i tw ≤ tz → POZYTYWNA
- [ ] **RCD-04**: Możliwość dodawania wierszy dla kolejnych wyłączników RCD
- [ ] **RCD-05**: Legenda oznaczeń pod tabelą (In, Iw, tw, tz)

### Załącznik 4 — Uziemienie

- [ ] **UZIEM-01**: Kolumny: Lp, Nazwa urządzenia, Rp [Ω], Wk, Rpo [Ω], Rw [Ω], Ocena
- [ ] **UZIEM-02**: Obliczenie Rpo = Rp × Wk (automatyczne)
- [ ] **UZIEM-03**: Rw domyślnie 10 Ω
- [ ] **UZIEM-04**: Automatyczna ocena: Rpo ≤ Rw → POZYTYWNA
- [ ] **UZIEM-05**: Możliwość dodawania wierszy
- [ ] **UZIEM-06**: Legenda oznaczeń pod tabelą (RE, Wk, Rpo, Rw)

### Baza Zabezpieczeń

- [ ] **PROT-01**: Wbudowana lista charakterystyk B, C, D z automatycznym Ia na podstawie In
- [ ] **PROT-02**: Użytkownik może wybrać zabezpieczenie z listy rozwijanej (np. B16, C25)
- [ ] **PROT-03**: Użytkownik może ręcznie skorygować wartość Ia po wyborze z listy

### Eksport i Dane

- [ ] **EXP-01**: Przycisk "Eksportuj PDF" generuje plik .pdf do pobrania
- [ ] **EXP-02**: PDF jest czarno-biały, profesjonalny, z numeracją stron
- [ ] **EXP-03**: PDF zawiera auto-paginację załączników (1.1, 1.2, ...) gdy tabela przekracza jedną stronę
- [ ] **EXP-04**: PDF poprawnie renderuje polskie znaki (ą, ę, ś, ź, ć, ó, ł, ń, ż)
- [ ] **EXP-05**: Ręczny zapis danych do localStorage (przycisk "Zapisz")
- [ ] **EXP-06**: Ręczny odczyt danych z localStorage (przycisk "Wczytaj")
- [ ] **EXP-07**: Eksport danych do pliku .json
- [ ] **EXP-08**: Import danych z pliku .json

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### UX Enhancements

- **UX-01**: Kopiowanie sekcji/wierszy (duplikowanie z danymi)
- **UX-02**: Drag & drop do zmiany kolejności sekcji
- **UX-03**: Szablony typowych konfiguracji (dom jednorodzinny, mieszkanie)
- **UX-04**: Walidacja inline z komunikatami o błędach

### Rozszerzone Dane

- **DATA-01**: Historia protokołów (lista zapisanych w localStorage)
- **DATA-02**: Dane wykonawcy zapamiętywane między protokołami
- **DATA-03**: Auto-uzupełnianie typów przewodów (YDYp 3x2.5, YDY 5x4, etc.)

## Out of Scope

| Feature | Reason |
|---------|--------|
| Backend / baza danych | Aplikacja 100% client-side, zero kosztów hostingu |
| Rejestracja użytkowników | Brak potrzeby kont — narzędzie lokalne |
| Import z miernika | Wymaga integracji sprzętowej, scope creep |
| Zdjęcia w protokole | Rozmiar PDF, złożoność, poza normą PN-HD 60364-6 |
| Inne typy protokołów (odgromowe, gazowe) | Inna norma, inna struktura — osobny projekt |
| Aplikacja mobilna natywna | Web-first, responsywność wystarczy |
| Tryb wieloużytkownikowy | Wymaga backendu, sprzeczne z architekturą |
| Real-time collaboration | Wymaga WebSocket/backendu |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| FORM-01 | — | Pending |
| FORM-02 | — | Pending |
| FORM-03 | — | Pending |
| FORM-04 | — | Pending |
| FORM-05 | — | Pending |
| FORM-06 | — | Pending |
| FORM-07 | — | Pending |
| DYN-01 | — | Pending |
| DYN-02 | — | Pending |
| DYN-03 | — | Pending |
| DYN-04 | — | Pending |
| DYN-05 | — | Pending |
| DYN-06 | — | Pending |
| SWZ-01 | — | Pending |
| SWZ-02 | — | Pending |
| SWZ-03 | — | Pending |
| SWZ-04 | — | Pending |
| SWZ-05 | — | Pending |
| SWZ-06 | — | Pending |
| SWZ-07 | — | Pending |
| IZOL-01 | — | Pending |
| IZOL-02 | — | Pending |
| IZOL-03 | — | Pending |
| IZOL-04 | — | Pending |
| IZOL-05 | — | Pending |
| IZOL-06 | — | Pending |
| IZOL-07 | — | Pending |
| RCD-01 | — | Pending |
| RCD-02 | — | Pending |
| RCD-03 | — | Pending |
| RCD-04 | — | Pending |
| RCD-05 | — | Pending |
| UZIEM-01 | — | Pending |
| UZIEM-02 | — | Pending |
| UZIEM-03 | — | Pending |
| UZIEM-04 | — | Pending |
| UZIEM-05 | — | Pending |
| UZIEM-06 | — | Pending |
| PROT-01 | — | Pending |
| PROT-02 | — | Pending |
| PROT-03 | — | Pending |
| EXP-01 | — | Pending |
| EXP-02 | — | Pending |
| EXP-03 | — | Pending |
| EXP-04 | — | Pending |
| EXP-05 | — | Pending |
| EXP-06 | — | Pending |
| EXP-07 | — | Pending |
| EXP-08 | — | Pending |

**Coverage:**
- v1 requirements: 48 total
- Mapped to phases: 0
- Unmapped: 48 ⚠️

---
*Requirements defined: 2026-02-23*
*Last updated: 2026-02-23 after initial definition*
