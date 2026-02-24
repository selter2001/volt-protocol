---
phase: 02-interactive-layer
plan: 01
type: execute
wave: 1
depends_on: []
files_modified:
  - index.html
autonomous: true
requirements:
  - ANIM-02
  - ANIM-03
  - ANIM-06

must_haves:
  truths:
    - "Po zmianie wartosci wejsciowych (prad zabezpieczenia, Zs, Usk) liczby wynikow (Id, Zsmax) animuja sie count-up do nowej wartosci zamiast przeskakiwac"
    - "Po zmianie wartosci uziemienia (Rp, Wk) wartosc Rpo animuje sie count-up do nowej wartosci"
    - "Klikniecie dowolnego buttona (.btn-glass, .btn-primary-glass) pokazuje wizualny pressed-in efekt (inset shadow) przez czas klikniecia"
    - "Komorki z verdyktem POZYTYWNA maja subtlny zielony glow, NEGATYWNA czerwony glow — widoczny bez hover"
    - "Orzeczenie NADAJE SIE ma zielony glow, NIE NADAJE SIE czerwony glow"
    - "Przy prefers-reduced-motion count-up jest wylaczony — wartosc pojawia sie natychmiast"
    - "Wszystkie istniejace funkcje (obliczenia, eksporty, localStorage) dzialaja identycznie"
  artifacts:
    - path: "index.html"
      provides: "CSS verdict glow classes, button :active states, JS animateCountUp utility, modified update functions"
      contains: "animateCountUp"
  key_links:
    - from: "animateCountUp()"
      to: "updateSWZComputedCells(), updateUZIEMComputedCells()"
      via: "Replaces direct textContent assignment with animated transition"
      pattern: "animateCountUp"
    - from: ".btn-glass:active, .btn-primary-glass:active"
      to: "@theme --shadow-neomorph-pressed"
      via: "CSS :active pseudo-class uses existing shadow token"
      pattern: "shadow-neomorph-pressed"
    - from: ".verdict-glow-positive, .verdict-glow-negative"
      to: "@theme --color-positive, --color-negative"
      via: "CSS classes add box-shadow glow matching verdict color"
      pattern: "verdict-glow"
---

<objective>
Dodanie warstwy interaktywnej: count-up animacje na wynikach obliczen, neomorphic pressed-in efekt na buttonach, glow na verdyktach. Czyste CSS + lekki JS utility — bez zewnetrznych bibliotek animacji.

Purpose: Po Fazie 1 (statyczny dark luxury) ten plan dodaje "zycie" do interfejsu — liczby plynnie sie zmieniaja, przyciski maja haptic feel, verdykty swieca odpowiednim kolorem.
Output: Zmodyfikowany index.html z pelna warstwa interaktywna.
</objective>

<execution_context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@.planning/STATE.md
@.planning/phase-2/RESEARCH.md
</execution_context>

<context>
@index.html
</context>

<tasks>

<task type="auto">
  <name>Task 1: CSS — verdict glow classes + button :active neomorphic states</name>
  <files>index.html</files>
  <action>
W pliku index.html dodaj nowe reguly CSS. Wszystkie zmiany w istniejacym bloku `<style type="text/tailwindcss">`.

**ZMIANA 1 — Dodaj verdict glow i button :active WEWNATRZ @layer components (przed zamykajacym `}` bloku @layer components, czyli przed linia ~177).**

Dodaj dokladnie te reguly:

```css
    /* Verdict glow — ANIM-06 */
    .verdict-glow-positive {
      text-shadow: 0 0 8px rgba(34, 197, 94, 0.6), 0 0 20px rgba(34, 197, 94, 0.3);
    }
    .verdict-glow-negative {
      text-shadow: 0 0 8px rgba(239, 68, 68, 0.6), 0 0 20px rgba(239, 68, 68, 0.3);
    }

    /* Neomorphic button press — ANIM-03 */
    .btn-glass:active {
      transform: scale(0.97);
      box-shadow: inset 3px 3px 8px rgba(0, 0, 0, 0.5), inset -1px -1px 4px rgba(255, 255, 255, 0.02);
    }
    .btn-primary-glass:active {
      transform: scale(0.97);
      box-shadow: inset 3px 3px 8px rgba(0, 0, 0, 0.5), inset -1px -1px 4px rgba(0, 240, 255, 0.05);
    }
```

Wazne:
- Dodaj PRZED zamykajacym `}` bloku `@layer components` (linia ~177).
- `verdict-glow-positive` i `verdict-glow-negative` uzywaja text-shadow (nie box-shadow) — glow na samym tekscie verdyktu.
- Button :active uzywa `transform: scale(0.97)` + inset shadow dla haptic pressed-in feel.
- prefers-reduced-motion juz obsluguje `transition-duration: 0.01ms !important` globalnie — ale transform i text-shadow nie sa animowane, wiec dzialaja natychmiast (instant visual feedback, nie animacja).
  </action>
  <verify>
    <automated>
grep -q 'verdict-glow-positive' index.html && echo "PASS: verdict-glow-positive" || echo "FAIL"
grep -q 'verdict-glow-negative' index.html && echo "PASS: verdict-glow-negative" || echo "FAIL"
grep -q 'btn-glass:active' index.html && echo "PASS: btn-glass:active" || echo "FAIL"
grep -q 'btn-primary-glass:active' index.html && echo "PASS: btn-primary-glass:active" || echo "FAIL"
grep -q 'scale(0.97)' index.html && echo "PASS: scale transform" || echo "FAIL"
    </automated>
    <manual>Kliknij button (np. Eksportuj PDF) — powinien sie lekko "wcisnac" (zmniejszyc o 3% + inset shadow). Puszczenie wraca do normalnego stanu.</manual>
  </verify>
  <done>CSS verdict glow classes i button :active neomorphic states dodane w @layer components.</done>
</task>

<task type="auto">
  <name>Task 2: JS — animateCountUp utility + verdict glow logic + hook into update functions</name>
  <files>index.html</files>
  <action>
W sekcji `<script>` pliku index.html dodaj utility function i zmodyfikuj istniejace update functions.

**ZMIANA 1 — Dodaj animateCountUp() utility PRZED funkcja updateSWZComputedCells() (przed linia ~1739).**

Wstaw dokladnie:

```javascript
// === ANIMATION UTILITIES (Phase 2) ===

function animateCountUp(element, newValue, decimals) {
  if (!element) return;
  // Check prefers-reduced-motion
  if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) {
    element.textContent = newValue != null ? newValue.toFixed(decimals) : '';
    return;
  }
  if (newValue == null) {
    element.textContent = '';
    return;
  }
  const oldText = element.textContent.trim();
  const oldValue = oldText === '' ? 0 : parseFloat(oldText);
  const diff = newValue - oldValue;
  if (Math.abs(diff) < 0.001) {
    element.textContent = newValue.toFixed(decimals);
    return;
  }
  const duration = 300;
  const start = performance.now();
  function tick(now) {
    const elapsed = now - start;
    const progress = Math.min(elapsed / duration, 1);
    // ease-out cubic
    const eased = 1 - Math.pow(1 - progress, 3);
    const current = oldValue + diff * eased;
    element.textContent = current.toFixed(decimals);
    if (progress < 1) {
      requestAnimationFrame(tick);
    }
  }
  requestAnimationFrame(tick);
}

function applyVerdictGlow(cell, verdict) {
  if (!cell) return;
  cell.classList.remove('verdict-glow-positive', 'verdict-glow-negative');
  if (verdict === 'POZYTYWNA' || verdict === 'POZYTYWNY') {
    cell.classList.add('verdict-glow-positive');
  } else if (verdict === 'NEGATYWNA' || verdict === 'NEGATYWNY') {
    cell.classList.add('verdict-glow-negative');
  }
}
```

**ZMIANA 2 — Zmodyfikuj updateSWZComputedCells() (linie ~1739-1757).**

Zamien CALA funkcje updateSWZComputedCells na:

```javascript
function updateSWZComputedCells(rowId, result) {
  const idCell = document.querySelector('.id-cell[data-row-id="' + rowId + '"]');
  const zsmaxCell = document.querySelector('.zsmax-cell[data-row-id="' + rowId + '"]');
  const verdictCell = document.querySelector('.verdict-cell[data-row-id="' + rowId + '"]');

  animateCountUp(idCell, result.Id, 2);
  animateCountUp(zsmaxCell, result.Zsmax, 2);

  if (verdictCell) {
    verdictCell.textContent = result.verdict || '';
    verdictCell.classList.remove('text-green-700', 'text-red-700');
    if (result.verdict === 'POZYTYWNA') verdictCell.classList.add('text-green-700');
    if (result.verdict === 'NEGATYWNA') verdictCell.classList.add('text-red-700');
    applyVerdictGlow(verdictCell, result.verdict);
  }
}
```

**ZMIANA 3 — Zmodyfikuj updateUZIEMComputedCells() (linie ~1925-1940).**

Zamien CALA funkcje updateUZIEMComputedCells na:

```javascript
function updateUZIEMComputedCells(rowId, result) {
  const rpoCell = document.querySelector('.uziem-rpo-cell[data-row-id="' + rowId + '"]');
  animateCountUp(rpoCell, result.Rpo, 2);

  const verdictCell = document.querySelector('.uziem-verdict-cell[data-row-id="' + rowId + '"]');
  if (verdictCell) {
    verdictCell.textContent = result.verdict || '';
    verdictCell.classList.remove('text-green-700', 'text-red-700');
    if (result.verdict === 'POZYTYWNA') verdictCell.classList.add('text-green-700');
    if (result.verdict === 'NEGATYWNA') verdictCell.classList.add('text-red-700');
    applyVerdictGlow(verdictCell, result.verdict);
  }
}
```

**ZMIANA 4 — Dodaj applyVerdictGlow do updateIZOLVerdict() (linie ~1827-1835).**

Zamien CALA funkcje updateIZOLVerdict na:

```javascript
function updateIZOLVerdict(rowId, result) {
  const verdictCell = document.querySelector('.izol-verdict-cell[data-row-id="' + rowId + '"]');
  if (verdictCell) {
    verdictCell.textContent = result.verdict || '';
    verdictCell.classList.remove('text-green-700', 'text-red-700');
    if (result.verdict === 'POZYTYWNA') verdictCell.classList.add('text-green-700');
    if (result.verdict === 'NEGATYWNA') verdictCell.classList.add('text-red-700');
    applyVerdictGlow(verdictCell, result.verdict);
  }
}
```

**ZMIANA 5 — Dodaj applyVerdictGlow do updateRCDVerdict() (linie ~1879-1885).**

Zamien CALA funkcje updateRCDVerdict na:

```javascript
function updateRCDVerdict(rowId, result) {
  const verdictCell = document.querySelector('.rcd-verdict-cell[data-row-id="' + rowId + '"]');
  if (verdictCell) {
    verdictCell.textContent = result.verdict || '';
    verdictCell.classList.remove('text-green-700', 'text-red-700');
    if (result.verdict === 'POZYTYWNA') verdictCell.classList.add('text-green-700');
    if (result.verdict === 'NEGATYWNA') verdictCell.classList.add('text-red-700');
    applyVerdictGlow(verdictCell, result.verdict);
  }
}
```

**ZMIANA 6 — Dodaj applyVerdictGlow do updateAssessmentCell() (linie ~1944-1956).**

Zamien CALA funkcje updateAssessmentCell na:

```javascript
function updateAssessmentCell(id, verdict) {
  const cell = document.getElementById(id);
  if (!cell) return;
  cell.textContent = verdict || '\u2014';
  cell.classList.remove('text-green-700', 'text-red-700', 'text-gray-400');
  if (verdict === 'POZYTYWNY') {
    cell.classList.add('text-green-700');
  } else if (verdict === 'NEGATYWNY') {
    cell.classList.add('text-red-700');
  } else {
    cell.classList.add('text-gray-400');
  }
  applyVerdictGlow(cell, verdict);
}
```

**ZMIANA 7 — Dodaj verdict glow do updateOrzeczenieDisplay().**

W funkcji updateOrzeczenieDisplay() (linia ~1968), po kazdym `orzText.className = ...` dodaj applyVerdictGlow:

Po linii `orzText.className = 'text-base font-bold text-green-700 mb-2';` (POZYTYWNA):
```javascript
    applyVerdictGlow(orzText, 'POZYTYWNA');
```

Po linii `orzText.className = 'text-base font-bold text-red-700 mb-2';` (NEGATYWNA):
```javascript
    applyVerdictGlow(orzText, 'NEGATYWNA');
```

Po linii `orzText.className = 'text-base text-platinum/30 mb-2';` (puste):
```javascript
    applyVerdictGlow(orzText, null);
```

**ZMIANA 8 — Dodaj verdict glow do renderFormTab() — statyczne verdykty w Assessment Table.**

W renderFormTab() w liniach gdzie tworzone sa komorki z POZYTYWNY (stale wiersze "Sprawdzenie dokumentacji" i "Ogleedziny"):

Linia z `text-center font-bold text-green-700">POZYTYWNY` — dodaj klase `verdict-glow-positive`:
`text-center font-bold text-green-700 verdict-glow-positive">POZYTYWNY`

Tak samo dla dynamicznych assessment rows — klasy verdict-glow-positive/negative dodawane sa juz przez applyVerdictGlow() w updateAssessmentCell(), ale renderFormTab() tworzy je od nowa. Dodaj verdict glow do stringow klasy w renderFormTab():

Dla swzClass, izolClass, rcdClass, uziemClass — po przypisaniu klasy tekstowej, dodaj odpowiedni glow.

Zamien wzorzec:
```javascript
const swzClass = assessment.swz === 'POZYTYWNY' ? 'text-green-700' : assessment.swz === 'NEGATYWNY' ? 'text-red-700' : 'text-gray-400';
```
na:
```javascript
const swzClass = assessment.swz === 'POZYTYWNY' ? 'text-green-700 verdict-glow-positive' : assessment.swz === 'NEGATYWNY' ? 'text-red-700 verdict-glow-negative' : 'text-gray-400';
```

Tak samo dla izolClass, rcdClass, uziemClass — dodaj `verdict-glow-positive` po `text-green-700` i `verdict-glow-negative` po `text-red-700`.

Tak samo dla statycznych POZYTYWNY (Dokumentacja, Ogleedziny):
`text-center font-bold text-green-700">POZYTYWNY` → `text-center font-bold text-green-700 verdict-glow-positive">POZYTYWNY`

I w renderFormTab() linie orzeczenia:
- `text-base font-bold text-green-700 mb-2` → `text-base font-bold text-green-700 verdict-glow-positive mb-2`
- `text-base font-bold text-red-700 mb-2` → `text-base font-bold text-red-700 verdict-glow-negative mb-2`

Analogicznie dla renderSWZRowCells() — verdict cell tworzony z klasa:
Linia ~677: dodaj verdict glow do statycznie renderowanego verdyktu:
```javascript
const verdictClass = calc.verdict === 'POZYTYWNA' ? 'text-green-700 verdict-glow-positive' : calc.verdict === 'NEGATYWNA' ? 'text-red-700 verdict-glow-negative' : '';
```

Tak samo w renderIZOLRowCells(), renderAttachment3() (rcd), renderAttachment4() (uziem) — wszedzie gdzie jest:
```javascript
const verdictClass = calc.verdict === 'POZYTYWNA' ? 'text-green-700' : calc.verdict === 'NEGATYWNA' ? 'text-red-700' : '';
```
zamien na:
```javascript
const verdictClass = calc.verdict === 'POZYTYWNA' ? 'text-green-700 verdict-glow-positive' : calc.verdict === 'NEGATYWNA' ? 'text-red-700 verdict-glow-negative' : '';
```

KRYTYCZNE OGRANICZENIA:
1. NIE zmieniaj logiki obliczen — tylko dodaj animacje/glow do istniejacych update functions.
2. ZACHOWAJ text-green-700 / text-red-700 / text-gray-400 classList — glow jest DODATKOWA klasa.
3. NIE dodawaj zewnetrznych bibliotek — wszystko w czystym JS/CSS.
4. animateCountUp() MUSI sprawdzac prefers-reduced-motion i pomijac animacje jesli aktywne.
  </action>
  <verify>
    <automated>python3 -c "
html = open('index.html').read()
js = html[html.find('<script>'):]

checks = [
    ('function animateCountUp(' in js, 'animateCountUp utility exists'),
    ('function applyVerdictGlow(' in js, 'applyVerdictGlow utility exists'),
    ('prefers-reduced-motion' in js, 'reduced-motion check in JS'),
    ('requestAnimationFrame(tick)' in js, 'rAF used for count-up'),
    ('animateCountUp(idCell' in js, 'count-up on Id cell'),
    ('animateCountUp(zsmaxCell' in js, 'count-up on Zsmax cell'),
    ('animateCountUp(rpoCell' in js, 'count-up on Rpo cell'),
    ('applyVerdictGlow(verdictCell' in js, 'glow on verdict cells'),
    ('applyVerdictGlow(cell' in js, 'glow on assessment cells'),
    ('applyVerdictGlow(orzText' in js, 'glow on orzeczenie'),
    ('verdict-glow-positive' in js, 'verdict-glow-positive used in JS'),
    ('verdict-glow-negative' in js, 'verdict-glow-negative used in JS'),
    ('text-green-700' in js, 'text-green-700 preserved'),
    ('text-red-700' in js, 'text-red-700 preserved'),
    ('.verdict-cell' in js, 'verdict-cell selector preserved'),
    ('.id-cell' in js, 'id-cell selector preserved'),
    ('.zsmax-cell' in js, 'zsmax-cell selector preserved'),
    ('.uziem-rpo-cell' in js, 'uziem-rpo-cell selector preserved'),
]
passed = sum(1 for ok, _ in checks if ok)
for ok, msg in checks:
    print(f\"{'PASS' if ok else 'FAIL'}: {msg}\")
print(f\"\n{passed}/{len(checks)} checks passed\")
assert passed == len(checks), f'{len(checks)-passed} checks failed'
print('All Phase 2 JS checks passed.')
"
    </automated>
    <manual>
Otworz index.html w przegladarce:
1. Zakładka Protokol → dwa stale POZYTYWNY maja zielony glow
2. Zal.1 SWZ → wpisz Usk=230, zmien Zs → Id i Zsmax animuja sie plynnie (count-up ~300ms)
3. Zal.1 SWZ → verdykt POZYTYWNA swieci zielonym, NEGATYWNA czerwonym
4. Zal.4 Uziemienie → dodaj wiersz, wpisz Rp i Wk → Rpo animuje sie count-up
5. Kliknij dowolny button (np. Eksportuj PDF) → pressed-in efekt (lekkie wciecia + cien)
6. Eksportuj PDF → plik generuje sie poprawnie
7. Zapisz/Wczytaj → dane zachowane
    </manual>
  </verify>
  <done>
1. animateCountUp() z rAF i ease-out cubic, prefers-reduced-motion safe
2. applyVerdictGlow() dodaje text-shadow glow do verdyktow
3. updateSWZComputedCells() uzywa count-up na Id i Zsmax
4. updateUZIEMComputedCells() uzywa count-up na Rpo
5. Wszystkie verdict update functions (SWZ, IZOL, RCD, UZIEM, Assessment, Orzeczenie) maja glow
6. Button :active neomorphic press efekt na .btn-glass i .btn-primary-glass
7. Statyczne verdykty w renderFormTab() maja glow classes
  </done>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <name>Task 3: Weryfikacja wizualna i funkcjonalna</name>
  <what-built>Count-up animacje na wynikach obliczen (Id, Zsmax, Rpo), neomorphic pressed-in efekt na buttonach, glow na verdyktach (POZYTYWNA = zielony, NEGATYWNA = czerwony). Pelna warstwa interaktywna premium "Industrial Elegance".</what-built>
  <how-to-verify>
1. Otworz index.html w przegladarce (Chrome/Edge)
2. Sprawdz count-up (ANIM-02):
   - Zal.1 SWZ: wpisz Usk=230, Zs=0.5 → Id i Zsmax powinny sie plynnie zanimowac do nowych wartosci
   - Zmien Zs na 1.0 → wartosci znow plynnie sie zmienia (nie przeskakuja)
   - Zal.4: dodaj wiersz, wpisz Rp=5, Wk=1.2 → Rpo animuje sie do 6.00
3. Sprawdz verdict glow (ANIM-06):
   - Zal.1 SWZ: po wpisaniu pelnych danych verdykt POZYTYWNA swieci zielonym glow
   - Zmien dane zeby byl NEGATYWNA → swieci czerwonym glow
   - Zakladka Protokol: "POZYTYWNY" w tabeli ocen swieci zielonym
   - Orzeczenie: "NADAJE SIE" ma zielony glow, "NIE NADAJE SIE" czerwony
4. Sprawdz button press (ANIM-03):
   - Kliknij i przytrzymaj dowolny button — widac pressed-in efekt (wciecie + cien)
   - Puszczenie wraca do normalnego stanu
5. Sprawdz funkcjonalnosc:
   - Eksport PDF → generuje sie poprawnie
   - Zapisz/Wczytaj → localStorage dziala
   - Eksport/Import JSON → dziala
   - Obliczenia poprawne (Id, Zsmax, verdykt)
6. (Opcjonalnie) Wlacz "Reduce motion" w systemie → count-up wylaczony, wartosci pojawiaja sie natychmiast
  </how-to-verify>
  <resume-signal>Wpisz "approved" jesli wszystko wyglada dobrze, lub opisz problemy do poprawienia.</resume-signal>
</task>

</tasks>

<verification>
Weryfikacja calosciowa planu 02-01:
1. `grep 'verdict-glow-positive' index.html` potwierdza CSS class i uzycie w JS
2. `grep 'animateCountUp' index.html` potwierdza utility i hook w update functions
3. `grep 'btn-glass:active' index.html` potwierdza neomorphic press
4. `grep 'prefers-reduced-motion' index.html` potwierdza accessibility w JS i CSS
5. Count-up widoczny w przegladarce przy zmianie wartosci
6. Glow widoczny na verdyktach
7. Pressed-in efekt na buttonach
8. Eksporty i obliczenia dzialaja identycznie
</verification>

<success_criteria>
- Count-up animacja (300ms, ease-out cubic) na Id, Zsmax, Rpo przy kazdej zmianie wartosci wejsciowej
- prefers-reduced-motion wylacza count-up — wartosci pojawiaja sie natychmiast
- Verdict glow: text-shadow zielony na POZYTYWNA/POZYTYWNY, czerwony na NEGATYWNA/NEGATYWNY
- Button :active: scale(0.97) + inset shadow na .btn-glass i .btn-primary-glass
- Zero bledow w konsoli przegladarki
- PDF, Word, JSON eksporty dzialaja bez bledow
- localStorage save/load dziala poprawnie
- Wszystkie obliczenia zwracaja identyczne wyniki
</success_criteria>

<output>
After completion, create `.planning/phase-2/02-01/02-01-SUMMARY.md`
</output>
