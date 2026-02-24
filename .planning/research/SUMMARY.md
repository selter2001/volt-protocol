# Project Research Summary

**Project:** VoltProtocol — Luxury Redesign ("Industrial Elegance")
**Domain:** Glassmorphism / Dark Luxury UI — single-file vanilla web application
**Researched:** 2026-02-24
**Confidence:** HIGH (CSS techniques, architecture) / MEDIUM (animation library, visual design patterns)

## Executive Summary

VoltProtocol to narzedzie formularzowe dla elektrykow (5 zakladek, pomiary, obliczenia, eksport PDF/Word/JSON) wymagajace wylacznie redesignu warstwy wizualnej — bez zadnych zmian funkcjonalnych. Calot aplikacja dziala jako jeden plik `index.html` z Tailwind v4 CDN i pdfmake, bez buildu. Badania potwierdzaja, ze wszystkie wymagane efekty premium ("Industrial Elegance": glassmorphism, noise texture, glow, animacje) sa w pelni realizowalne w vanilla CSS i JS bez zadnych nowych zaleznosci poza anime.js 4.x (17KB, UMD CDN) i dwoma fontami z Google Fonts. Strategia jest zachowawcza: dodajemy `<style type="text/tailwindcss">` z `@theme` tokenami i `@layer components`, wymieniamy klasy Tailwind odpowiedzialne za wyglad, pozostawiajac wszystkie klasy layoutu i selektory JS bez zmian.

Glowne ryzyko to nie implementacja, lecz integralnosc — istniejacy JS uzywa klas CSS jako selektorow DOM (`verdict-cell`, `id-cell`, `zsmax-cell`) do targetowania obliczen. Te nazwy klas musza zostac zachowane dokladnie. Drugim kluczowym ryzykiem jest PDF export: pdfmake musi pozostac calkowicie izolowany od nowych tokenow CSS — generuje wlasny rendering, nie HTML/CSS. Oba ryzyka sa dobrze zidentyfikowane i maja jasne strategie mitygacji.

Podejscie fazowe jest narzucone przez architekture: najpierw fundament (tokeny + tlo + animacje off), potem komponenty statyczne (top bar, taby, karty, tabele), na koncu animacje jako warstwa opcjonalna. Taka kolejnosc pozwala weryfikowac funkcjonalnosc po kazdym kroku i wycofywac zmiany bez ryzyka utraty danych uzytkownika lub zepsutego eksportu.

## Key Findings

### Recommended Stack

Projekt jest jednoplikowy — zadnego buildu, zadnych zewnetrznych zasobow poza CDN. Tailwind v4 Play CDN jest juz obecny i wymaga tylko dodania bloku `<style type="text/tailwindcss">` z tokenami `@theme`. Jedyna nowa zaleznosc to anime.js 4.3.6 (UMD, ~17KB) do count-up i spring physics. Dwa fonty Google Fonts (Space Grotesk + JetBrains Mono) ladowane przez CDN. Wszystkie efekty wizualne — glassmorphism, noise texture, glow, flow lines — realizowane czystym CSS bez bibliotek.

**Core technologies:**
- **Tailwind CSS v4 Play CDN** (juz obecny): utility-first styling + `@theme` tokeny — jedyna zmiana to dodanie `<style type="text/tailwindcss">`
- **anime.js 4.3.6** (nowy, CDN UMD): count-up na wynikach obliczen, spring transitions — 17KB, lzejszy od GSAP, natywne spring() API
- **Space Grotesk** (Google Fonts CDN): primary UI font dla labelek i naglowkow — premium geometric sans, techiczny charakter
- **JetBrains Mono** (Google Fonts CDN): font monospace dla wartosci numerycznych — tabular nums, precyzja czytelnosci
- **CSS `backdrop-filter`** (natywny): glassmorphism na cardach — 95%+ browser support, zero JS
- **SVG `feTurbulence`** (inline): noise/grain texture — zero HTTP requests, single-file compatible
- **CSS `linear()` timing function**: spring physics bez JS na color transitions — 88%+ browser support

**Alternatywy odrzucone:** GSAP (zbyt ciezki, plugin system niepotrzebny), Three.js/WebGL (dysproporcjonalny do zakresu), Lottie (wymaga zewnetrznych plikow JSON), Framer Motion (React-only).

### Expected Features

Badania FEATURES.md definiuja dwupoziomowa hierarchie cech. Wszystkie sa realizowalne w ramach ograniczenia single-file.

**Must have (table stakes — Faza 1):**
- Void Black background (`#0a0a0f`) z noise texture — baza estetyki; bez noise tlo wyglada cyfrowo
- Ambient gradient orbs (CSS pseudo-elements) — warunek konieczny dla glassmorphism; bez nich glass "kolapsuje"
- Glassmorphism na glownych sekcjach i cardach — standard premium dark UI 2025-2026
- Dual-font system (Space Grotesk + JetBrains Mono) — oddzielenie UI labels od wartosci technicznych
- Glowing focus states na inputach + hover micro-feedback — podstawa interaktywnosci czuciowej
- Thin cyan borders (0.5-1px) — charakterystyczny detal premium
- Bento Cards layout (24px border-radius) — organizacja formularza bez tabel
- Tabular numbers (`font-variant-numeric: tabular-nums`) — 1 linia CSS, eliminuje "skakanie" cyfr
- `prefers-reduced-motion` wrapper na wszystkich animacjach — cross-cutting concern, P1

**Should have (differentiators — Faza 2):**
- Count-up effect na wynikach obliczen (Id, Zsmax, Ia, Rpo) — najwyzszy "wow" dla narzedzia obliczeniowego
- Neomorphic depth na buttonach (CSS dual shadow) — haptic pressed-in feel
- OLED glow na verdyktach POZYTYWNA/NEGATYWNA — natychmiastowa czytelnosc wyniku
- Spring animations na kartach i tab transitions — polished transitions via CSS `linear()`
- Hero Result Card z duzym verdyktem i grain-gradient tlem

**Defer (do decyzji / Faza 3+):**
- Flow Effect (animowane linie "elektrycznosci" miedzy sekcjami) — najwyzsze ryzyko performance i UX distraction; rozwazyc tylko na strefach bez inputow
- Particles/Canvas animacje — out of scope dla single HTML bez buildu

### Architecture Approach

Architektura to single-file bez build step. Caly redesign odbywa sie przez dodanie jednego bloku CSS (`<style type="text/tailwindcss">`) i minimalne zmiany klas w stalych JS (np. `const inputCls = ...`). Istniejacy event system i data flow pozostaja nienaruszone. Animacje sa dolaczane jako "hooks" do istniejacych punktow aktualizacji DOM (`updateSWZComputedCells`, `updateFinalAssessmentDisplay`) — nie wymagaja zmiany logiki.

**Major components:**
1. **App Shell** (`body`, `#app`) — Void Black background, noise texture overlay, ambient gradient orbs
2. **Top Bar + Tab Navigation** — glassmorphic strip, neomorphic buttons, glowing active tab indicator
3. **Bento Cards** (form sections) — `card-glass` klasa na wrapperach sekcji; `input-glass` na polach
4. **Data Tables** (Attachments 1-4) — `table-dark` na tabelach, `verdict-positive/negative` na wyrokach; glassmorphism tylko na card wrapperze, nie na komorkach
5. **Hero Result Card** (Orzeczenie) — grain-gradient, duza typografia, glow na verdykcie
6. **Animation Layer** — count-up via `requestAnimationFrame`, CSS spring via `linear()`, pseudo-element opacity dla glow

**Kluczowy constraint architektoniczny:** Selektory JS (`verdict-cell`, `id-cell`, `zsmax-cell`, etc.) musza byc zachowane dokladnie w zmodyfikowanych render functions. Strategia: dodawac nowe klasy, nie zastepowac istniejacych funkcjonalnych.

### Critical Pitfalls

1. **Glassmorphism contrast failure na dark background** — bez solid semi-opaque tint (6-8% white) wewnatrz paneli, tekst na `#0a0a0f` nie spelnia WCAG 4.5:1. Prewencja: zweryfikowac kazda pare tekst/tlo kontrastomierzem przed przejsciem do Fazy 2.

2. **Animowany `box-shadow` powoduje paint jank** — `box-shadow` nie jest GPU-composited; na formularzach z 15-20+ wierszami bezposrednia animacja glow wywoluje jank na mid-range hardware. Prewencja: pseudoelement `::after` z `opacity` animation (composited), nigdy bezposrednio animowac `box-shadow`.

3. **PDF export dziedziczacy dark theme** — pdfmake generuje wlasny rendering, nie HTML/CSS; ryzyko nastepuje jesli kod PDF zostanie zrefaktoryzowany zeby referencowac CSS custom properties. Prewencja: twarda izolacja kodu PDF (hardkodowane #000000/#FFFFFF), test PDF po kazdej fazie.

4. **Tailwind v4 CDN: `@theme` ignorowany bez `type="text/tailwindcss"`** — regularny `<style>` bez tego atrybutu jest pomijany przez Tailwind CDN. Prewencja: zawsze `<style type="text/tailwindcss">`, weryfikacja w DevTools (custom vars w `:root`).

5. **Browser autofill override** — Chrome/Safari aplikuja `!important` style na autofill (zolte/biale tlo) niszczac dark theme. Prewencja: `-webkit-autofill` box-shadow override jako czesc bazowych styli inputow (5 linii CSS).

6. **Halation na cyan text** — Electric Cyan (#00f0ff) na Void Black powoduje "krwawienie" tekstu, szczegolnie u uzytkownikow z astygmatyzmem. Prewencja: cyan tylko na bordery/ikony/wskazniki; dane (liczby, etykiety) w Soft Platinum (#e0e0e8).

## Implications for Roadmap

Badania wskazuja jednoznacznie na 3-fazowa strukture narzucona przez zaleznosci techniczne i ryzyko.

### Faza 1: Foundation — Visual Base

**Rationale:** Wszystkie efekty wizualne Fazy 2 i 3 zaleza od tego: glassmorphism wymaga ambient orbs w tle; count-up wymaga tabular numbers; wszelkie animacje wymagaja `prefers-reduced-motion` guardu. Tokeny musza byc poprawne zanim cokolwiek innego zostanie zbudowane. To rowniez etap, w ktorym nalezy zamknac wszystkie ryzyka systemowe (PDF isolation, Tailwind v4 setup, autofill override, CSS scope).

**Delivers:** W pelni dzialajaca apke z nowym tlem, typografia, cardami i inputami — bez animacji. Wyglad premium, brak efektow "wow".

**Addresses features:** Void Black + noise, ambient orbs, glassmorphism panels, dual-font system, thin borders, bento cards layout, tabular numbers, glowing focus states (statyczne), `prefers-reduced-motion` framework.

**Avoids pitfalls:** Kontrast (weryfikacja przed Faza 2), PDF isolation (ustawiana na poczatku), Tailwind v4 CDN setup (pierwsza weryfikacja), autofill override, CSS scope bleed, halation (regula: cyan = UI, platinum = dane).

**Build order (z ARCHITECTURE.md):**
1. `<style type="text/tailwindcss">` + `@theme` tokeny → weryfikacja DevTools
2. `@layer base` body styling + noise texture → weryfikacja: tylko tlo sie zmienilo
3. Ambient gradient orbs (CSS pseudo-elements)
4. Top Bar + Tab Navigation (glassmorphic styling)
5. Bento Cards + `card-glass` / `input-glass` komponenty (Tab 0 — Protokol)
6. Data Tables (Attachments 1-4, kolejno: SWZ, RCD, Uziemienie, Izolacja)
7. Hero Result Card (statyczny, bez animacji)
8. Autofill override + PDF isolation boundary + weryfikacja eksportow

### Faza 2: Interactive Layer — Animations and States

**Rationale:** Animacje musza byc dodawane po potwierdzeniu ze statyczny redesign nie psuje zadnej funkcjonalnosci. Kazda animacja jest "hookiem" do istniejacego data flow, nie zmiana logiki. Ta faza ma najwyzsze ryzyko performance (count-up debounce, pseudo-element glow pattern, animation budget) — stad osobna faza z wlasna weryfikacja.

**Delivers:** Pelne doswiadczenie "Industrial Elegance": count-up na wynikach, spring transitions, glowing focus rings, neomorphic button states, verdykty z glow efektem, Hero Result Card aktywny.

**Addresses features:** Count-up effect (Id, Zsmax, Ia, Rpo), neomorphic depth na buttonach, OLED glow na verdyktach, spring animations na kartach, tab transitions, Hero Result Card aktywacja.

**Uses:** anime.js 4.3.6 (count-up), CSS `linear()` (spring physics), `requestAnimationFrame` (custom count-up bez library), pseudo-element `::after` opacity pattern.

**Avoids pitfalls:** Paint jank (pseudo-element pattern, nie bezposredni box-shadow animation), neomorphic button affordance (primary actions = filled cyan, nie pure neomorph), dynamic row animation budget (count-up tylko na summary values, Intersection Observer dla flow lines).

### Faza 3: Stress Test and Flow Effect (optional)

**Rationale:** Flow Effect (animowane linie "pradu") jest potencjalnie najmocniejszym elementem brand identity ale rowniez najwyzszym ryzykiem UX distraction i performance. Sluszne jest wprowadzenie go dopiero po potwierdzeniu ze Fazy 1-2 dzialaja poprawnie pod pelnym obciazeniem danych (20 wierszy na zakladke). To tez etap finalnej weryfikacji wszystkich "looks done but isn't" checklist items.

**Delivers:** Opcjonalny Flow Effect (SVG stroke-dashoffset lub CSS gradient animation) na dividerach sekcji (NIE w srodku formularzy), peln weryfikacja 20+ wierszy / wszystkie zakladki / PDF / Word / JSON import, test `prefers-reduced-motion`.

**Addresses features:** Flow Effect (jesli decyzja pozytywna), stress test animation budget.

**Avoids pitfalls:** Off-screen animation waste (Intersection Observer), dynamic row budget (weryfikacja przy max danych), finalna weryfikacja PDF/Word/JSON eksportow.

### Phase Ordering Rationale

- **Foundation przed animations:** Glassmorphism, ambient orbs i tokeny sa warunkiem koniecznym dla wlasciwego wygladu animacji. Budowanie animacji bez finalnych tla/kolorow prowadzi do iteracji.
- **Static styling przed JS hooks:** Istniejacy kod JS uzywa klas jako selektorow — zmiana klas bez weryfikacji funkcji niszczy obliczenia. Kazdy krok statycznego restylu konczy sie testem funkcjonalnym.
- **Flow Effect na koncu:** Najwyzsze ryzyko performance i UX distraction; w obecnej konfiguracji jest "P3 — do decyzji", nie blokujace dla MVP.
- **PDF isolation w Fazie 1:** Nie mozna zakladac ze to prosta weryfikacja — nalezy ustawic granice (komentarz, hardkodowane kolory) zanim jakikolwiek refaktor JS zostanie dotkniety.

### Research Flags

Fazy z dobrze udokumentowanymi wzorcami (mozna pomin research-phase):
- **Faza 1:** Glassmorphism, noise texture, Tailwind v4 `@theme`, CSS typography — wszystko zweryfikowane na oficjalnych zrodlach. Wzorce gotowe do implementacji bezposrednio z STACK.md i ARCHITECTURE.md.
- **Faza 2 (CSS spring, pseudo-element glow):** Wzorce wysokiej pewnosci (Josh Comeau, CSS-Tricks, MDN).

Fazy mogace wymagac glebszego research podczas planowania:
- **Faza 2 (count-up debounce + anime.js integracja):** anime.js v4 jest nowa wersja (complete rewrite z Feb 2025); API count-up via `animate()` vs custom `requestAnimationFrame` wymaga weryfikacji podczas implementacji. Confidence: MEDIUM.
- **Faza 3 (Flow Effect):** Decyzja UX (czy wdrozac) wymaga kontekstu projektu. Techniczne wzorce sa jasne (SVG stroke-dashoffset).

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | Tailwind v4 CDN potwierdzone oficjalnymi docs; anime.js v4.3.6 CDN path zweryfikowany na jsDelivr; fonty Google Fonts oficjalne. Jedyna niepewnosc: anime.js v4 API jest nowe (rewrite Feb 2025) |
| Features | HIGH | Hierarchia table stakes / differentiators dobrze udokumentowana. Realizowalnosc w vanilla CSS/JS potwierdzona. Anti-features dobrze uzasadnione technicznie |
| Architecture | HIGH | Oparte na bezposredniej inspekcji kodu zrodlowego (2923 linii `index.html`). Wszystkie hook points, selektory JS, render functions zweryfikowane firsthand |
| Pitfalls | HIGH | Zrodla wysokiej pewnosci: NNGroup, Axess Lab, MDN, CSS-Tricks oficjalne. Pitfall-to-phase mapping konkretny i actionable |

**Overall confidence:** HIGH

### Gaps to Address

- **anime.js v4 count-up API:** Nalezy zweryfikowac podczas implementacji czy wbudowane `animate()` API jest wystarczajace dla count-up, czy lepszy jest custom `requestAnimationFrame`. ARCHITECTURE.md proponuje custom rAF — to rekomendacja do weryfikacji przy pierwszym podejsciu do Fazy 2.
- **0.5px borders na non-retina Windows:** Pitfalls wskazuje ze 0.5px moze renderowac sie jako 0px na 1x display. Nalezy zdecydowac podczas Fazy 1 czy uzywac `@media (max-resolution: 1dppx)` breakpointa z 1px fallback.
- **Flow Effect decyzja UX:** Badania klasyfikuja Flow Effect jako "P3 — do decyzji". Decyzja wymaga kontekstu od stakeholderow — czy narzedzie formularzowe dla elektrykow to odpowiednie miejsce dla dekoracyjnych animowanych linii.

## Sources

### Primary (HIGH confidence)
- Tailwind CSS v4 Official Docs (tailwindcss.com) — `@theme`, `@layer`, Play CDN, `type="text/tailwindcss"`
- MDN Web Docs — `backdrop-filter`, `linear()` timing function, animation performance
- anime.js Official Docs + GitHub Releases — v4.3.6 UMD CDN path, spring easing API
- Google Fonts Official — Space Grotesk, JetBrains Mono
- CSS-Tricks — SVG feTurbulence noise, stroke-dashoffset animation, neumorphism, SVG line animation
- NNGroup — Glassmorphism accessibility i best practices
- Axess Lab — Glassmorphism contrast failure patterns
- Josh W. Comeau — CSS `linear()` spring physics
- pdfmake Official Docs — styling isolation

### Secondary (MEDIUM confidence)
- Medium (@developer_89726) — Dark Glassmorphism 2026 design trends
- UXPilot — Glassmorphism performance i accessibility guidelines
- Figma Community (Dark Neumorphism Tesla App) — visual reference
- getButterfly — count-up animation via requestAnimationFrame pattern
- CSS-Tricks (autofill override) — webkit autofill pseudo-selector

### Tertiary (LOW-MEDIUM confidence)
- Sanjay Dey (UX Design Patterns 2026) — trend reference, bez weryfikacji technicznej
- ecommercewebdesign.agency (Neumorphism 2.0) — WebSearch only, trend context

---
*Research completed: 2026-02-24*
*Ready for roadmap: yes*
