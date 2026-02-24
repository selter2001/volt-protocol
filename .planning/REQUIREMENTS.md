# Requirements: VoltProtocol — Luxury Redesign

**Defined:** 2026-02-24
**Core Value:** Użytkownik otwiera aplikację i natychmiast czuje, że pracuje z narzędziem klasy premium — precyzja techniczna w luksusowym opakowaniu.

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### Visual Foundation

- [ ] **VIS-01**: Body ma Void Black (#0a0a0f) tło z inline SVG noise texture (feTurbulence, 3-5% opacity)
- [ ] **VIS-02**: Tailwind v4 `@theme` block definiuje pełny system design tokenów (kolory, border-radius, spacing, shadows)
- [ ] **VIS-03**: Color system: Void Black (#0a0a0f) base, Electric Cyan (#00f0ff) akcenty interaktywne, Soft Platinum (#e0e0e8) tekst/bordery
- [ ] **VIS-04**: Premium geometric sans font (np. Space Grotesk) na labele i nagłówki
- [ ] **VIS-05**: Monospaced font (np. JetBrains Mono) na wartości numeryczne w tabelach i inputach
- [ ] **VIS-06**: Ambient gradient orbs (rozmyte kolorowe koła) za glassmorphism panelami dla głębi wizualnej

### Glassmorphism Components

- [ ] **GLASS-01**: Wszystkie panele/karty używają frosted glass effect (backdrop-blur 12-20px, rgba tło, ultra-thin 0.5px Soft Platinum border)
- [ ] **GLASS-02**: Bento Card layout z 24px rounded corners na sekcje formularza i załączniki
- [ ] **GLASS-03**: Input fields z glassmorphism tłem i glowing Electric Cyan focus ring
- [ ] **GLASS-04**: Hero Result Card na dole z grain-gradient tłem i dużą bold typografią na końcową ocenę protokołu
- [ ] **GLASS-05**: Tabele danych w Bento Cards z dark glass rows zamiast standardowych białych wierszy

### Navigation & Layout

- [ ] **NAV-01**: Zakładki (tabs) mają dark luxury styling z animowanym active indicator (cyan underline/glow)
- [ ] **NAV-02**: Top bar z przyciskami eksportu ma glass background i spójny dark theme
- [ ] **NAV-03**: Footer z dark theme i subtlnym Soft Platinum stylem

### Animation & Interaction

- [ ] **ANIM-01**: Smooth hover/focus CSS transitions na wszystkich interaktywnych elementach (200-300ms ease)
- [ ] **ANIM-02**: Spring physics count-up animacja na wynikach obliczeń (Id, Zsmax, Ia) przy zmianie wartości
- [ ] **ANIM-03**: Neomorphic pressed-in depth effect na przyciskach (box-shadow inset przy click)
- [ ] **ANIM-04**: Flow Effect — animowane SVG linie między sekcjami symulujące przepływ prądu, świecące po wypełnieniu sekcji
- [ ] **ANIM-05**: Glowing focus states na inputach (Electric Cyan glow ring via pseudo-element)
- [ ] **ANIM-06**: Verdict cells (POZYTYWNA/NEGATYWNA) z subtlnym glow effect odpowiednim do wyniku (green/red)
- [ ] **ANIM-07**: `prefers-reduced-motion` media query wyłącza wszystkie animacje keyframe, zachowuje instant state changes

### Functional Preservation

- [ ] **PRES-01**: PDF export generuje identyczny dokument jak przed redesignem (jasne tło, polskie znaki, landscape Zał.2)
- [ ] **PRES-02**: Word export (.docx) generuje identyczny dokument jak przed redesignem
- [ ] **PRES-03**: JSON export/import działa identycznie — dane zachowane w tym samym formacie
- [ ] **PRES-04**: localStorage save/load działa identycznie
- [ ] **PRES-05**: Wszystkie obliczenia (SWZ, izolacja, RCD, uziemienie) zwracają identyczne wyniki
- [ ] **PRES-06**: Dynamiczne dodawanie/usuwanie sekcji i wierszy działa identycznie
- [ ] **PRES-07**: JS selektory (.verdict-cell, .id-cell, .zsmax-cell, .izol-verdict-cell) zachowane na wszystkich zredesignowanych elementach
- [ ] **PRES-08**: Aplikacja pozostaje single HTML file bez build step

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Advanced Animation

- **ADV-01**: Particle effects na Hero Result Card przy verdykcie POZYTYWNA
- **ADV-02**: Parallax scroll effect między sekcjami
- **ADV-03**: Dark/light theme toggle

### Extended Polish

- **POL-01**: Custom scrollbar styling (thin, cyan)
- **POL-02**: Loading skeleton screens z glass effect
- **POL-03**: Toast notifications z glass styling

## Out of Scope

| Feature | Reason |
|---------|--------|
| Zmiana architektury (framework, rozbicie plików) | Użytkownik chce zachować single HTML file |
| Nowa funkcjonalność (nowe formularze, obliczenia) | To redesign wizualny, nie feature add |
| Płatne fonty | Budżet = darmowe Google Fonts |
| Backend/server | Aplikacja pozostaje 100% client-side |
| Redesign PDF output | PDF ma osobny styl, nie podlega redesignowi |
| Mobile-first responsive redesign | Obecny layout zachowany, tylko visual redesign |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| VIS-01 | Phase 1 | Pending |
| VIS-02 | Phase 1 | Pending |
| VIS-03 | Phase 1 | Pending |
| VIS-04 | Phase 1 | Pending |
| VIS-05 | Phase 1 | Pending |
| VIS-06 | Phase 1 | Pending |
| GLASS-01 | Phase 1 | Pending |
| GLASS-02 | Phase 1 | Pending |
| GLASS-03 | Phase 1 | Pending |
| GLASS-04 | Phase 1 | Pending |
| GLASS-05 | Phase 1 | Pending |
| NAV-01 | Phase 1 | Pending |
| NAV-02 | Phase 1 | Pending |
| NAV-03 | Phase 1 | Pending |
| ANIM-01 | Phase 1 | Pending |
| ANIM-05 | Phase 1 | Pending |
| ANIM-07 | Phase 1 | Pending |
| PRES-07 | Phase 1 | Pending |
| PRES-08 | Phase 1 | Pending |
| ANIM-02 | Phase 2 | Pending |
| ANIM-03 | Phase 2 | Pending |
| ANIM-04 | Phase 3 | Pending |
| ANIM-06 | Phase 2 | Pending |
| PRES-01 | Phase 3 | Pending |
| PRES-02 | Phase 3 | Pending |
| PRES-03 | Phase 3 | Pending |
| PRES-04 | Phase 3 | Pending |
| PRES-05 | Phase 3 | Pending |
| PRES-06 | Phase 3 | Pending |

**Coverage:**
- v1 requirements: 29 total
- Mapped to phases: 29
- Unmapped: 0 ✓

---
*Requirements defined: 2026-02-24*
*Last updated: 2026-02-24 after roadmap creation*
