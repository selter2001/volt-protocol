# Requirements: VoltProtocol — Luxury Redesign

**Defined:** 2026-02-24
**Core Value:** Użytkownik otwiera aplikację i natychmiast czuje, że pracuje z narzędziem klasy premium — precyzja techniczna w luksusowym opakowaniu.

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### Visual Foundation

- [x] **VIS-01**: Body ma Void Black (#0a0a0f) tło z inline SVG noise texture (feTurbulence, 3-5% opacity)
- [x] **VIS-02**: Tailwind v4 `@theme` block definiuje pełny system design tokenów (kolory, border-radius, spacing, shadows)
- [x] **VIS-03**: Color system: Void Black (#0a0a0f) base, Electric Cyan (#00f0ff) akcenty interaktywne, Soft Platinum (#e0e0e8) tekst/bordery
- [x] **VIS-04**: Premium geometric sans font (np. Space Grotesk) na labele i nagłówki
- [x] **VIS-05**: Monospaced font (np. JetBrains Mono) na wartości numeryczne w tabelach i inputach
- [x] **VIS-06**: Ambient gradient orbs (rozmyte kolorowe koła) za glassmorphism panelami dla głębi wizualnej

### Glassmorphism Components

- [x] **GLASS-01**: Wszystkie panele/karty uzywaja frosted glass effect (backdrop-blur 12-20px, rgba tlo, ultra-thin 0.5px Soft Platinum border)
- [x] **GLASS-02**: Bento Card layout z 24px rounded corners na sekcje formularza i zalaczniki
- [x] **GLASS-03**: Input fields z glassmorphism tlem i glowing Electric Cyan focus ring
- [x] **GLASS-04**: Hero Result Card na dole z grain-gradient tlem i duza bold typografia na koncowa ocene protokolu
- [x] **GLASS-05**: Tabele danych w Bento Cards z dark glass rows zamiast standardowych bialych wierszy

### Navigation & Layout

- [x] **NAV-01**: Zakladki (tabs) maja dark luxury styling z animowanym active indicator (cyan underline/glow)
- [x] **NAV-02**: Top bar z przyciskami eksportu ma glass background i spojny dark theme
- [x] **NAV-03**: Footer z dark theme i subtlnym Soft Platinum stylem

### Animation & Interaction

- [x] **ANIM-01**: Smooth hover/focus CSS transitions na wszystkich interaktywnych elementach (200-300ms ease)
- [x] **ANIM-02**: Spring physics count-up animacja na wynikach obliczeń (Id, Zsmax, Ia) przy zmianie wartości
- [x] **ANIM-03**: Neomorphic pressed-in depth effect na przyciskach (box-shadow inset przy click)
- [x] **ANIM-04**: ~~Flow Effect~~ — Skipped (tabbed layout: sekcje nie widoczne jednocześnie, SVG linie bez sensu wizualnego)
- [x] **ANIM-05**: Glowing focus states na inputach (Electric Cyan glow ring via pseudo-element)
- [x] **ANIM-06**: Verdict cells (POZYTYWNA/NEGATYWNA) z subtlnym glow effect odpowiednim do wyniku (green/red)
- [x] **ANIM-07**: `prefers-reduced-motion` media query wyłącza wszystkie animacje keyframe, zachowuje instant state changes

### Functional Preservation

- [x] **PRES-01**: PDF export generuje identyczny dokument jak przed redesignem (jasne tło, polskie znaki, landscape Zał.2)
- [x] **PRES-02**: Word export (.docx) generuje identyczny dokument jak przed redesignem
- [x] **PRES-03**: JSON export/import działa identycznie — dane zachowane w tym samym formacie
- [x] **PRES-04**: localStorage save/load działa identycznie
- [x] **PRES-05**: Wszystkie obliczenia (SWZ, izolacja, RCD, uziemienie) zwracają identyczne wyniki
- [x] **PRES-06**: Dynamiczne dodawanie/usuwanie sekcji i wierszy działa identycznie
- [x] **PRES-07**: JS selektory (.verdict-cell, .id-cell, .zsmax-cell, .izol-verdict-cell) zachowane na wszystkich zredesignowanych elementach
- [x] **PRES-08**: Aplikacja pozostaje single HTML file bez build step

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
| VIS-01 | Phase 1 | Done (01-01) |
| VIS-02 | Phase 1 | Done (01-01) |
| VIS-03 | Phase 1 | Done (01-01) |
| VIS-04 | Phase 1 | Done (01-01) |
| VIS-05 | Phase 1 | Done (01-01) |
| VIS-06 | Phase 1 | Done (01-01) |
| GLASS-01 | Phase 1 | Done (01-02) |
| GLASS-02 | Phase 1 | Done (01-02) |
| GLASS-03 | Phase 1 | Done (01-02) |
| GLASS-04 | Phase 1 | Done (01-02) |
| GLASS-05 | Phase 1 | Done (01-02) |
| NAV-01 | Phase 1 | Done (01-02) |
| NAV-02 | Phase 1 | Done (01-02) |
| NAV-03 | Phase 1 | Done (01-02) |
| ANIM-01 | Phase 1 | Done (01-02) |
| ANIM-05 | Phase 1 | Done (01-02) |
| ANIM-07 | Phase 1 | Done (01-01) |
| PRES-07 | Phase 1 | Done (01-02) |
| PRES-08 | Phase 1 | Done (01-02) |
| ANIM-02 | Phase 2 | Done (02-01) |
| ANIM-03 | Phase 2 | Done (02-01) |
| ANIM-04 | Phase 3 | Skipped (03-01) |
| ANIM-06 | Phase 2 | Done (02-01) |
| PRES-01 | Phase 3 | Done (03-01) |
| PRES-02 | Phase 3 | Done (03-01) |
| PRES-03 | Phase 3 | Done (03-01) |
| PRES-04 | Phase 3 | Done (03-01) |
| PRES-05 | Phase 3 | Done (03-01) |
| PRES-06 | Phase 3 | Done (03-01) |

**Coverage:**
- v1 requirements: 29 total
- Mapped to phases: 29
- Unmapped: 0 ✓

---
*Requirements defined: 2026-02-24*
*Last updated: 2026-02-24 after roadmap creation*
