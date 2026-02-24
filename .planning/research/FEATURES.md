# Feature Landscape — Luxury Dark UI (Industrial Elegance)

**Domain:** Premium dark web interface — technical tool (electrical measurement protocol generator)
**Reference apps:** Tesla App, Apple iOS 26 Liquid Glass, Revolut Ultra, fintech dashboards
**Researched:** 2026-02-24
**Research confidence:** MEDIUM-HIGH (WebSearch + official sources; no Context7 applicable for visual design patterns)

---

## Kontekst aplikacji

VoltProtocol to narzedzie formularzowe dla elektrykow — 5 zakladek, formularze z polami numerycznymi, automatyczne obliczenia, tabele z dynamicznymi wierszami, eksport PDF/Word/JSON. Redesign = warstwa wizualna "Industrial Elegance" bez zmiany funkcjonalnosci. Caly kod pozostaje w jednym pliku HTML z Tailwind v4 CDN.

**Kluczowe ograniczenie badawcze:** Wszystkie efekty wizualne musza byc implementowalne w vanilla CSS/JS bez build step, bez zewnetrznych bibliotek animacji (poza tym co juz jest — Tailwind CDN + pdfmake).

---

## Table Stakes

Cechy, ktorych brak sprawia ze interfejs wyglada tanio. Uzytkownik moze tego nie nazwac, ale natychmiast czuje "cos jest nie tak".

| Feature | Dlaczego oczekiwane | Zlozonosc | Uwagi |
|---------|---------------------|-----------|-------|
| **Void Black background z noise texture** | Flat czern (#000) wyglada jak placeholder; noise/grain dodaje glebi i organicznosci, stosowane od Apple do premium fintech | Niska | SVG noise lub CSS `filter: url(#noise)`; subtlny (2-5% opacity) |
| **Glassmorphism na cardach/panelach** | Standard 2025-2026 dla premium dark UI; `backdrop-filter: blur(10-15px)` z `rgba` tlem | Srednia | Wymaga warstw tla (gradient orbs za UI); bez nich "collapse into transparent box" |
| **Monospace font na liczbach** | IBM Plex Mono / JetBrains Mono — standard dla danych technicznych; premium fintech i Tesla uzywaja rozdzielonej typografii: sans na labele, mono na wartosci | Niska | Obydwa dostepne przez Google Fonts CDN — darmowe |
| **Glowing focus states na inputach** | Uzytkownik musi wiedziec gdzie jest. `box-shadow: 0 0 0 2px cyan` na `:focus` to minimum; bez tego formularz wyglada jak z lat 2015 | Niska | `outline: none` + niestandardowy `box-shadow` glow |
| **Hover states z micro-feedbackiem** | Kazdy interaktywny element musi reagowac subtelna zmiana — opacity, jasnosc bordera, lekki scale. Brak hover = UI wyglada jak statyczny dokument | Niska | CSS transitions 150-200ms `ease-out` |
| **Kolorystyczne verdykty (POZYTYWNA/NEGATYWNA)** | Wynik obliczen musi byc wizualnie natychmiastowy — kolor, nie tylko tekst. Zielony glow = dobrze, czerwony = uwaga | Niska | `text-shadow` / `box-shadow` glow na span z wynikiem |
| **Thin borders (0.5-1px) w accent color** | Ultra-thin cyan/platinum bordery na cardach i inputach — charakterystyczny detal premium UI. Grube (2px+) bordery = "tanio" | Niska | `border: 1px solid rgba(0, 240, 255, 0.3)` |
| **Semi-transparent tlo pod szklami** | Bez 10-20% solid overlay za tekstem na glass panelach — nieczytelnosc. WCAG wymaga 4.5:1 kontrastu | Niska | `background: rgba(10, 10, 15, 0.6)` + `backdrop-filter` |
| **Smooth tab transitions** | Przejscia miedzy zakladkami bez "migniecia" — fade lub slide, 200ms. Brak = aplikacja czuje sie "zepsuta" | Niska | CSS opacity/transform transition |
| **Consistent spacing system** | Bento-style layout wymaga spojnego gridu — 24px rounding na cardach, jednolite odstepy | Niska | Tailwind spacing utilities z custom values |

---

## Differentiators

Elementy "wow factor" — nieoczekiwane, ale wzmacniajace percepcje premium. Tu jest roznica miedzy "ladna apka" a "co to jest, musze to pokazac znajomym".

### Animacje i fizyka

| Feature | Wartosc | Zlozonosc | Uwagi |
|---------|---------|-----------|-------|
| **Count-up effect na wynikach obliczen** | Gdy Id/Zsmax/Ia sie zmienia, liczba "nacieka" od 0 do wartosci docelowej w ~400ms — efekt znany z Tesla dashboardu i Apple Health. Robi OGROMNE wrazenie dla narzedzia obliczeniowego | Wysoka | Vanilla JS `requestAnimationFrame` + easing function; wymaga debounce na inputach |
| **Spring physics na kartach/modalach** | Karty "wskazykuja" przy pojawieniu sie z lekkim overshoot (spring, nie linear). `linear()` CSS lub JS spring. Wsparcie przegladarek: 88%+ (Chrome/Safari/Firefox od 2023) | Srednia | CSS `linear()` timing function; graceful degradation na `ease-out` |
| **Glowing focus ring z "pulse"** | Na aktywnym inpucie — cyan border glow z subtelnym pulsem (keyframe opacity 0.6→1→0.6). Sygnalizuje "tu jestem, tu pracuje" | Srednia | CSS `@keyframes` + `animation: pulse 2s infinite` |
| **Hero Result Card z duzymi liczbami** | Wynik POZYTYWNA/NEGATYWNA jako duzy card na dole formularza — duza typografia, grain-gradient tlo, wyrazny glow. Jak wynik testu w Apple Fitness | Wysoka | Wymaga znajomosci layoutu konkretnej zakladki |

### Efekty wizualne

| Feature | Wartosc | Zlozonosc | Uwagi |
|---------|---------|-----------|-------|
| **Flow Effect — swiecace linie laczace sekcje** | Animowane linie symulujace "przeplyw pradu" miedzy sekcjami formularza. Bezposrednie nawiazanie do domeny (elektrycznosc). Unikatowy motyw "Industrial Elegance" | Bardzo wysoka | SVG `stroke-dashoffset` animacja lub Canvas; UWAGA: moze byc distraction na duzych formularzach — rozwazyc tylko na stronie glownej/wynikach |
| **Neomorphic depth na buttonach** | Przyciski z dual box-shadow (jasny gore-lewo, ciemny dol-prawo) dajacy efekt "wcisniety". Na `:active` invert cieni — haptic pressed-in feel. Stosowany w Tesla App Figma community designs | Niska | CSS `box-shadow: -3px -3px 6px rgba(255,255,255,0.05), 3px 3px 6px rgba(0,0,0,0.5)` |
| **OLED-style ambient glow na wynikach** | Wartosci numeryczne i statusy oplywajace delikatnym swiatlem w kolorze akcentu (cyan dla OK, amber/red dla bledow). Efekt "swiatlo od srodka" | Srednia | `text-shadow: 0 0 20px rgba(0,240,255,0.4)` + `box-shadow` |
| **Bento Cards layout** | Zamiast standardowych tabel — karty z 24px border-radius grupujace powiazane pola. Organizacja danych jak w Apple Health dashboard | Srednia | Tailwind `rounded-3xl` + glassmorphism + shadow |
| **Gradient ambient orbs w tle** | Za UI plywaja zamglone gradienty (cyan, platinum) — tworza "srodowisko" dla glassmorphism bez ktorego glass wyglada plasko. Subtelne, 3-8% opacity | Niska | CSS `radial-gradient` + `filter: blur(80px)` na `position: fixed` pseudo-elementach |
| **Noise/grain texture na hero cardach** | SVG-based noise overlay na cardach z wynikami — dodaje "druk" i materialnosc, odroznia premium od flat | Niska | SVG `feTurbulence` filter jako base64 data URI |

### Typografia i dane

| Feature | Wartosc | Zlozonosc | Uwagi |
|---------|---------|-----------|-------|
| **Dual-font system: sans labele + mono wartosci** | Labele w premium sans (Space Grotesk / DM Sans), wartosci numeryczne w IBM Plex Mono / JetBrains Mono. Estetyka techniczna premium — jak Bloomberg Terminal spotyka Apple | Niska | 2 Google Fonts CDN calls; warianty: Regular + Medium |
| **Tabular numbers (tnum)** | `font-variant-numeric: tabular-nums` na wszystkich polach numerycznych — zapobiega "skakaniu" cyfr przy zmianach wartosci | Niska | Jedna linia CSS; wymaga fontu z tnum support (IBM Plex Mono ma) |
| **Uppercase tracking na labelach** | `letter-spacing: 0.08em` + `text-transform: uppercase` na sekcji headerach — estetyka premium industrial (Tesla UI, fintech dashboards) | Niska | CSS klasa `.section-label` |

---

## Anti-Features

Elementy ktore wyglada efektownie w showreel, ale aktywnie szkodza UX w narzedziu technicznym uzytkowynym przez elektrykow wykonujacych pomiary.

| Anti-Feature | Dlaczego unikac | Co zamiast |
|-------------|-----------------|------------|
| **Heavy backdrop-filter na kazdym elemencie** | `backdrop-filter: blur(20px+)` na wielu elementach jednoczesnie = crash GPU na slabszych laptopach/tabletach. Elektricy uzywaja roznych urzadzen | Glassmorphism tylko na kartach wynikow i glownych sekcjach; inputy moga byc semi-transparent bez blur |
| **Animowanie blur property** | `blur` w CSS transitions = jank nawet na dobrych GPU. Jedyna animacja ktora jest zawsze problematyczna | Animowac `opacity`, `transform`, `box-shadow` — nigdy `backdrop-filter` value |
| **Flow Effect w srodku formularzy** | Animowane linie "pradu" przeplywajace przez pola inputow = cognitive distraction przy wpisywaniu danych pomiarowych. Uzytkownik traci fokus | Flow Effect tylko w strefach "bez akcji" — miedzy sekcjami, na stronie startowej, przy wynikach |
| **Auto-playing particles w tle** | Particles.js z setkami punktow = CPU drain; przy PDF export moze spowalniab caly plik HTML | Statyczne gradienty + subtelne pulsujace orby (CSS only) |
| **Ciagla animacja na danych wejsciowych** | "Flashy" animacje triggerowane kazdym keystroke = rozproszenie + CPU spike przy szybkim pisaniu | Animacje tylko przy `blur` (opuszczeniu pola) lub zmianie wyniku obliczen |
| **Pure white (#fff) tekst na void black** | Halation efekt — tekst "krwawi" na ciemnym tle, powoduje eye strain. Szczegolnie szkodliwy dla uzytkownikow z astygmatyzmem | Soft Platinum (#e0e0e8) — 87% white, eliminuje halation |
| **Neon colors >100% brightness** | Saturowany neon (pure #00ffff) na ciemnym tle tworzy halation i zmeczenie wzroku po 10+ minutach pracy | Electric Cyan (#00f0ff) w 60-80% opacity jako bordery; pelna nasycenie tylko na kluczowych akcjach |
| **Zbyt wiele "wow" animacji naraz** | Jesli kazdy element sie porusza/swieci jednoczesnie, nic nie jest specjalne. Hierarchia uwagi sie rozpadla | Max 2-3 animowane elementy na raz; reszta statyczna |
| **Animacje bez `prefers-reduced-motion`** | 35% uzytkownikow korzysta z reduced-motion (accessibility, slabosc sprzetu, preferencja). Brak respektowania = accessibility fail + performance problemy | `@media (prefers-reduced-motion: reduce)` wrapper na wszystkich keyframe animations |
| **Neomorphism na inputach** | Neumorphic inputs maja BARDZO slaby kontrast — WCAG fail. Szczegolnie widoczne na mniejszych ekranach i przy jasnym swietle otoczenia | Neumorphism tylko na buttonach (gdzie kontrast nie jest problemem), inputy = glassmorphism z wyrازnym borderem |
| **Skomplikowane scroll-based animations** | Elektricy niekoniecznie scrolluja strony — moga uzywac Tab do nawigacji. Scroll-trigger = animations never fire | Spring animations na `:focus`/data load, nie na scroll |

---

## Feature Dependencies

```
Ambient gradient orbs (tlo) → Glassmorphism na cardach
  (bez warstw tla glass "kolapsuje" do przezroczystego boxa)

Dual-font system (Google Fonts CDN) → Premium labele + mono values
  (bez fontu fallback na system-ui — degraduje elegancko)

Tabular numbers (tnum) → Count-up animation wyglada dobrze
  (bez tnum liczby "skacza" szerokoscia przy count-up)

Neomorphic buttons + Glowing focus → spojny "haptic" system
  (one bez drugich tworza niespojny mix stylistic)

Void Black tlo + noise texture → caly look "Industrial Elegance"
  (noise texture jest fundamentem — bez niej tlo wyglada cyfrowo, nie materialnie)

Hero Result Card → Count-up effect
  (Result Card jest "stage" dla count-up animacji)

prefers-reduced-motion guard → wszystkie keyframe animations
  (bez tego accessibility fail; musi byc na wszystkich animacjach)
```

---

## MVP Recommendation

### Faza 1 — Fundament wizualny (Table Stakes)
Priorytet:
1. **Void Black + noise texture tlo** — baza wszystkiego
2. **Ambient gradient orbs** — umozliwia glassmorphism
3. **Glassmorphism na glownych sekcjach** — natychmiastowy "wow"
4. **Dual-font system** — IBM Plex Mono na liczby + Space Grotesk na labele
5. **Glowing focus states + hover states** — interaktywnosc czuciowa
6. **Thin cyan bordery na inputach i cardach** — precyzja premium
7. **Bento Cards layout** — organizacja bez tabel
8. **Tabular numbers** — 1 linia CSS, duzy efekt

### Faza 2 — Differentiators
Po fazie 1, kiedy fundament stoi:
1. **Count-up na wynikach** — najwyzszy "wow" dla narzedzia obliczeniowego
2. **Neomorphic depth na buttonach** — haptic feel
3. **OLED glow na verdyktach** — wynik POZYTYWNA/NEGATYWNA z aura
4. **Spring animations na kartach** — polished transitions
5. **Hero Result Card** — grand finale kazdej zakladki

### Odroczono (do decyzji)
- **Flow Effect**: Potencjalnie najmocniejszy efekt "brand identity" ale tez najwyzsze ryzyko distraction i performance. Rozwazyc tylko na stronie wynikow/glownej, nie w srodku formularza.
- **Particles / Canvas animacje**: Out of scope dla single HTML bez build step.

---

## Complexity Assessment

| Feature | Implementacja | Ryzyko | Priorytet |
|---------|--------------|--------|-----------|
| Noise texture SVG | Niska — 10 linii CSS | Brak | P1 |
| Ambient orbs | Niska — CSS pseudo-elements | Brak | P1 |
| Glassmorphism panels | Srednia — z-index management | Performance na mobile | P1 |
| Dual-font system | Niska — Google Fonts CDN | Latency CDN (marginalny) | P1 |
| Glowing focus | Niska — CSS box-shadow | Brak | P1 |
| Thin borders | Niska — CSS | Brak | P1 |
| Bento Cards | Niska-Srednia — Tailwind refactor | Duzo HTML zmian | P1 |
| Tabular numbers | Niska — 1 linia CSS | Brak | P1 |
| Count-up animation | Wysoka — JS `requestAnimationFrame` | Debounce; rapid input | P2 |
| Neomorphic buttons | Niska — CSS box-shadow | Brak | P2 |
| OLED glow na verdyktach | Niska — CSS text/box-shadow | Niska | P2 |
| Spring animations | Srednia — CSS `linear()` | Browser support 88% | P2 |
| Hero Result Card | Srednia-Wysoka — layout redesign | Wymaga refaktoru HTML | P2 |
| Flow Effect | Bardzo wysoka — SVG/Canvas | Performance + UX distraction | P3 |
| prefers-reduced-motion | Niska — CSS media query | Wymagane dla wszystkich animacji | P1 (cross-cutting) |

---

## Sources

- [Dark Glassmorphism: The Aesthetic That Will Define UI in 2026 — Medium](https://medium.com/@developer_89726/dark-glassmorphism-the-aesthetic-that-will-define-ui-in-2026-93aa4153088f) — MEDIUM confidence (WebSearch)
- [12 Glassmorphism UI Features, Best Practices, and Examples — UXPilot](https://uxpilot.ai/blogs/glassmorphism-ui) — HIGH confidence (official resource, verified with WebFetch)
- [Neon Mode: Building a new Dark UI — Codista](https://www.codista.com/de/blog/neon-mode-building-new-dark-ui/) — HIGH confidence (verified with WebFetch)
- [Apple introduces Liquid Glass — Apple Newsroom](https://www.apple.com/newsroom/2025/06/apple-introduces-a-delightful-and-elegant-new-software-design/) — HIGH confidence (official Apple source)
- [Springs and Bounces in Native CSS — Josh W. Comeau](https://www.joshwcomeau.com/animation/linear-timing-function/) — HIGH confidence (authoritative CSS author)
- [Dark Neumorphism UI Tesla App — Figma Community](https://www.figma.com/community/file/1075333697693168660) — MEDIUM confidence (community design)
- [prefers-reduced-motion — CSS-Tricks](https://css-tricks.com/almanac/rules/m/media/prefers-reduced-motion/) — HIGH confidence
- [Design accessible animation and movement — Pope Tech](https://blog.pope.tech/2025/12/08/design-accessible-animation-and-movement/) — HIGH confidence (2025 publication)
- [IBM Plex Mono — Google Fonts](https://fonts.google.com/specimen/IBM+Plex+Mono) — HIGH confidence (official)
- [Top UI Design Trends 2026 — Figma Resource Library](https://www.figma.com/resource-library/web-design-trends/) — MEDIUM confidence
- [Neumorphism 2.0 — ecommercewebdesign.agency](https://ecommercewebdesign.agency/the-rise-of-neumorphism-2-0-soft-shadows-and-skeuomorphism-in-2025-designs/) — LOW-MEDIUM confidence (WebSearch only)
- [7 Mobile UX/UI Design Patterns Dominating 2026 — Sanjay Dey](https://www.sanjaydey.com/mobile-ux-ui-design-patterns-2026-data-backed/) — MEDIUM confidence
