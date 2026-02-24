---
phase: 01-visual-foundation
plan: 02
type: execute
wave: 2
depends_on: ["01-01"]
files_modified: [index.html]
autonomous: true
requirements: [GLASS-01, GLASS-02, GLASS-03, GLASS-04, GLASS-05, NAV-01, NAV-02, NAV-03, ANIM-01, ANIM-05, PRES-07, PRES-08]

must_haves:
  truths:
    - "Wszystkie sekcje formularza (Protokol, Zal.1-4) wyswietlaja frosted glass karty z 24px zaokragleniami zamiast bialych bg-white boxow"
    - "Inputy maja ciemne glassmorphism tlo i po kliknieciu pojawia sie Electric Cyan focus glow ring"
    - "Zakladki maja dark luxury styling — aktywna z cyan underline, nieaktywne z platinum/50 tekstem"
    - "Top bar buttons maja glass styling (btn-glass / btn-primary-glass) zamiast kolorowych bg-green/blue/purple"
    - "Hero Result Card (orzeczenie) ma gradient tlo z cyan akcentem i jest wizualnie wyrozniajacy"
    - "Tabele maja dark glass thead (bg-white/[0.06]) i cienkie bordery (border-white/[0.12]) zamiast border-gray-300"
    - "Footer ma dark theme (border-white/10, text-platinum/40)"
    - "Wszystkie JS selektory (.verdict-cell, .id-cell, .zsmax-cell, .izol-verdict-cell, .rcd-verdict-cell, .uziem-rpo-cell, .uziem-verdict-cell) sa obecne w DOM"
    - "Obliczenia Id, Zsmax, verdykt dzialaja identycznie — wpisanie danych daje poprawne wyniki"
    - "Eksporty (PDF, Word, JSON) i localStorage save/load dzialaja bez bledow"
  artifacts:
    - path: "index.html"
      provides: "Glassmorphism component classes in @layer components + restyled HTML/JS"
      contains: "@layer components"
  key_links:
    - from: "index.html (@layer components)"
      to: "index.html (JS render functions)"
      via: "CSS class names used in JS template strings"
      pattern: "card-glass|input-glass|select-glass|btn-glass|btn-primary-glass|hero-result-card"
    - from: "index.html (JS querySelector selectors)"
      to: "index.html (HTML/JS elements)"
      via: "CSS class selectors in JS"
      pattern: "\\.verdict-cell|\\.id-cell|\\.zsmax-cell|\\.izol-verdict-cell|\\.rcd-verdict-cell|\\.uziem-rpo-cell|\\.uziem-verdict-cell"
    - from: "index.html (classList.add/remove)"
      to: "index.html (verdict display)"
      via: "Tailwind utility classes toggled by JS"
      pattern: "text-green-700|text-red-700|text-gray-400|hidden"
---

<objective>
Dodanie warstwy glassmorphism components do index.html i restyling calego UI na dark luxury theme.

Purpose: Plan 01-01 ustawil tokeny, body, noise i orbs. Ten plan buduje na tym fundamencie — dodaje klasy komponentow (card-glass, input-glass, btn-glass, hero-result-card) w @layer components, a nastepnie podmienia klasy w statycznym HTML (top bar, tabs, footer) i we wszystkich JS render functions (renderFormTab, renderSWZRowCells, renderAttachment1-4, renderTabs, updateOrzeczenieDisplay, showToast). Po tym planie cala aplikacja wyglada jak premium dark tool z glassmorphism.

Output: Zmodyfikowany index.html z pelnym dark luxury UI — glassmorphism karty, inputy z cyan focus, dark tabele, Hero Result Card, dark nawigacja i footer.
</objective>

<execution_context>
@./.claude/get-shit-done/workflows/execute-plan.md
@./.claude/get-shit-done/templates/summary.md
</execution_context>

<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@.planning/STATE.md
@.planning/phase-1/RESEARCH.md
@.planning/phase-1/01-01/01-01-SUMMARY.md
@index.html
</context>

<tasks>

<task type="auto">
  <name>Task 1: Dodaj @layer components z klasami glassmorphism + restyling statycznego HTML (top bar, tabs, footer)</name>
  <files>index.html</files>
  <action>
KROK A — Dodaj @layer components do istniejacego bloku style type="text/tailwindcss" (dodanego w Plan 01-01), PO zakonczeniu istniejacych regul @layer base. Wstaw dokladnie ten blok:

```css
@layer components {
  .card-glass {
    background: rgba(255, 255, 255, 0.04);
    backdrop-filter: blur(12px) saturate(1.5);
    -webkit-backdrop-filter: blur(12px) saturate(1.5);
    border: 1px solid rgba(224, 224, 232, 0.12);
    border-radius: 24px;
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
  }
  .input-glass {
    background: rgba(255, 255, 255, 0.06);
    border: 1px solid rgba(224, 224, 232, 0.12);
    border-radius: 8px;
    color: #e0e0e8;
    transition: border-color 200ms ease, box-shadow 200ms ease;
  }
  .input-glass:focus {
    outline: none;
    border-color: rgba(0, 240, 255, 0.5);
    box-shadow: 0 0 0 2px rgba(0, 240, 255, 0.2), 0 0 20px rgba(0, 240, 255, 0.1);
  }
  .input-glass::placeholder {
    color: rgba(224, 224, 232, 0.3);
  }
  .select-glass {
    background: rgba(255, 255, 255, 0.06);
    border: 1px solid rgba(224, 224, 232, 0.12);
    border-radius: 8px;
    color: #e0e0e8;
  }
  .select-glass option {
    background: #0f0f16;
    color: #e0e0e8;
  }
  .btn-glass {
    background: rgba(255, 255, 255, 0.06);
    border: 1px solid rgba(224, 224, 232, 0.12);
    border-radius: 8px;
    color: #e0e0e8;
    transition: all 200ms ease;
  }
  .btn-glass:hover {
    background: rgba(255, 255, 255, 0.10);
    border-color: rgba(0, 240, 255, 0.3);
  }
  .btn-primary-glass {
    background: rgba(0, 240, 255, 0.15);
    border: 1px solid rgba(0, 240, 255, 0.3);
    border-radius: 8px;
    color: #00f0ff;
    transition: all 200ms ease;
  }
  .btn-primary-glass:hover {
    background: rgba(0, 240, 255, 0.25);
    box-shadow: 0 0 20px rgba(0, 240, 255, 0.15);
  }
  .hero-result-card {
    background: linear-gradient(135deg, rgba(255,255,255,0.06) 0%, rgba(0,240,255,0.04) 100%);
    backdrop-filter: blur(16px);
    -webkit-backdrop-filter: blur(16px);
    border: 1px solid rgba(0, 240, 255, 0.2);
    border-radius: 24px;
    box-shadow: 0 8px 32px rgba(0,0,0,0.4), 0 0 40px rgba(0,240,255,0.05);
  }
}
```

KROK B — Modyfikacja statycznego HTML. Zmien klasy w nastepujacych elementach:

1. TOP BAR (linia ~19): h1 — dodaj `text-platinum` do istniejacych klas:
   `<h1 class="text-2xl font-bold text-platinum">...`

2. TOP BAR BUTTONS (linie ~21-27) — podmien klasy wygladu zachowujac ID:
   - btn-save: `btn-glass px-3 py-2 text-sm font-medium text-green-400 border-green-500/30 hover:border-green-400/50`
   - btn-load: `btn-glass px-3 py-2 text-sm font-medium text-green-400 border-green-500/30 hover:border-green-400/50`
   - btn-export-json: `btn-glass px-3 py-2 text-sm font-medium`
   - btn-import-json: `btn-glass px-3 py-2 text-sm font-medium`
   - btn-export-word: `btn-glass px-3 py-2 text-sm font-medium text-purple-400 border-purple-500/30 hover:border-purple-400/50`
   - btn-import-word: `btn-glass px-3 py-2 text-sm font-medium text-purple-400 border-purple-500/30 hover:border-purple-400/50`
   - btn-export-pdf: `btn-primary-glass px-4 py-2 text-sm font-medium` (primary action = cyan)

3. TABS (linia ~32): kontener tabs:
   `<div id="tabs" class="flex border-b border-white/10 mb-4">`

4. Tab buttons (linie ~33-37):
   - Aktywna (data-tab="0"): `px-4 py-2 font-medium border-b-2 border-cyan text-cyan transition-colors duration-200`
   - Nieaktywne (data-tab="1"-"4"): `px-4 py-2 font-medium border-b-2 border-transparent text-platinum/50 hover:text-platinum transition-colors duration-200`

5. FOOTER (linia ~48): kontener:
   `border-t border-white/10 text-center text-xs text-platinum/40`
   Link: `underline hover:text-cyan`

WAZNE: Nie zmieniaj klasy `hidden` na kontenerach attachment-1 do attachment-4. Nie zmieniaj id atrybutow. Nie zmieniaj struktury HTML — tylko class atrybuty.
  </action>
  <verify>
    <automated>python3 -c "
import re
html = open('index.html').read()
checks = [
    ('@layer components' in html, '@layer components block exists'),
    ('.card-glass' in html, 'card-glass class defined'),
    ('.input-glass' in html, 'input-glass class defined'),
    ('.btn-glass' in html, 'btn-glass class defined'),
    ('.btn-primary-glass' in html, 'btn-primary-glass class defined'),
    ('.hero-result-card' in html, 'hero-result-card class defined'),
    ('border-white/10 mb-4' in html, 'tabs border updated'),
    ('text-platinum' in html, 'platinum text used'),
    ('border-cyan text-cyan' in html, 'active tab cyan styling'),
    ('btn-primary-glass px-4 py-2' in html, 'PDF button is primary-glass'),
    ('hover:text-cyan' in html, 'footer link hover cyan'),
    ('id=\"btn-save\"' in html, 'btn-save ID preserved'),
    ('id=\"btn-export-pdf\"' in html, 'btn-export-pdf ID preserved'),
    ('id=\"tabs\"' in html, 'tabs ID preserved'),
    ('class=\"hidden\"' in html or 'class=\\\"hidden\\\"' in html, 'hidden class preserved on attachments'),
]
passed = sum(1 for ok, _ in checks if ok)
for ok, msg in checks:
    print(f\"{'PASS' if ok else 'FAIL'}: {msg}\")
print(f\"\n{passed}/{len(checks)} checks passed\")
assert passed == len(checks), f'{len(checks)-passed} checks failed'
print('All static HTML checks passed.')
"
    </automated>
    <manual>Otworz index.html w przegladarce. Top bar powinien miec ciemne glass buttons, zakladki z cyan aktywnym indicatorem, footer w ciemnym temacie.</manual>
  </verify>
  <done>Blok @layer components z 7 klasami komponentow istnieje w style block. Top bar, tabs i footer maja dark luxury klasy. Wszystkie HTML ID i hidden klasy zachowane.</done>
</task>

<task type="auto">
  <name>Task 2: Restyling JS render functions — formularze, tabele, attachmenty, orzeczenie, toast</name>
  <files>index.html</files>
  <action>
Zmodyfikuj klasy CSS w stringach JS render functions. Kazda zmiana jest podmiana stringa klasy — NIE zmieniaj logiki JS, NIE zmieniaj struktury DOM, NIE usuwaj klas uzywanych jako selektory querySelector.

KRYTYCZNE SELEKTORY JS — MUSISZ ZACHOWAC na elementach:
- `.id-cell`, `.zsmax-cell`, `.verdict-cell`, `.izol-verdict-cell`, `.rcd-verdict-cell`, `.uziem-rpo-cell`, `.uziem-verdict-cell`

KRYTYCZNE classList.add/remove — MUSISZ ZACHOWAC w logice:
- `text-green-700`, `text-red-700`, `text-gray-400`, `hidden`

DYNAMICZNE .className = — MUSISZ UMIESCIC WSZYSTKIE klasy w stringu (bo nadpisuje caly className):

=== renderTabs() (linie ~384, 386) ===
Linia 384 (active tab): btn.className = 'px-4 py-2 font-medium border-b-2 border-cyan text-cyan transition-colors duration-200';
Linia 386 (inactive tab): btn.className = 'px-4 py-2 font-medium border-b-2 border-transparent text-platinum/50 hover:text-platinum transition-colors duration-200';

=== renderFormTab() (linie ~519-726) ===
Linia ~522 inputCls: 'w-full input-glass px-2 py-1.5 text-sm'
Linia ~523 labelCls: 'block text-sm font-medium text-platinum/80 mb-1'

Card wrappers (linie ~527, 536, 555, 574, 592): 'bg-white border rounded p-4 mb-4' -> 'card-glass p-6 mb-4'
Section headings (linie ~528, 537, 556, 575, 593, 627, 693): 'text-md font-semibold mb-3 text-gray-800' -> 'text-md font-semibold mb-3 text-platinum'

Assessment section (linia ~626): 'bg-white border rounded p-4 mb-4' -> 'card-glass p-6 mb-4' (zachowaj id="assessment-section")
Assessment thead (linia ~630): 'bg-gray-50' -> 'bg-white/[0.06]'
Assessment th cells (linie ~632-634): 'border border-gray-300' -> 'border border-white/[0.12]'
Assessment td cells (linie ~641-682): 'border border-gray-300' -> 'border border-white/[0.12]'

Orzeczenie section (linia ~692): 'bg-white border rounded p-4 mb-4' -> 'hero-result-card p-6 mb-4' (zachowaj id="orzeczenie-section")
Orzeczenie labels (linie ~696-703): ZACHOWAJ text-green-700, text-red-700, text-gray-400 (uzywane przez classList)
Next date labels (linie ~708, 710): ZACHOWAJ text-gray-700, text-gray-400 (nadpisywane przez updateOrzeczenieDisplay)

Signature lines (linie ~715, 719): 'border-gray-400' -> 'border-platinum/30', 'text-gray-500' -> 'text-platinum/50'
Instrument inputs w tabeli (linie ~609-616): 'border border-gray-300 rounded' -> 'input-glass'

=== renderSWZRowCells() (linie ~427-517) ===
Linia ~434 inputCls: 'w-full input-glass px-1 py-0.5 text-sm text-center'
Linia ~435 selectCls: 'w-full select-glass px-1 py-0.5 text-sm'

Wszystkie td cells: 'border border-gray-300' -> 'border border-white/[0.12]'
Verdict cell — ZACHOWAJ verdict-cell class: 'border border-gray-300 px-2 py-1 text-center font-bold verdict-cell' -> 'border border-white/[0.12] px-2 py-1 text-center font-bold verdict-cell'
Tak samo ZACHOWAJ .id-cell, .zsmax-cell na odpowiednich td
Delete button (~linia 512): 'text-red-500 hover:text-red-700' -> 'text-red-400 hover:text-red-300'

=== renderAttachment1() (linie ~728-839) ===
Heading (~732): dodaj 'text-platinum'
Add section button (~733): 'mb-3 px-3 py-1 bg-blue-600 text-white rounded hover:bg-blue-700 text-sm' -> 'mb-3 px-3 py-1 btn-primary-glass text-sm'
Table (~736): 'border border-gray-300' -> 'border border-white/[0.12]'
thead (~739): 'bg-gray-50 text-xs text-center' -> 'bg-white/[0.06] text-xs text-center text-platinum/70'
th cells (~743-776): 'border border-gray-300' -> 'border border-white/[0.12]'
Text colors (~744, 771): 'text-gray-500' -> 'text-platinum/50'
Fixed row (~784): 'bg-yellow-50' -> 'bg-cyan/5'
Section header row (~791): 'bg-gray-200' -> 'bg-white/[0.08]'
Subsection header row (~803): 'bg-gray-100' -> 'bg-white/[0.04]'
Focus styles (~793, 805): 'focus:bg-white focus:ring-1 focus:ring-blue-400' -> 'focus:bg-white/10 focus:ring-1 focus:ring-cyan/40'
Section buttons (~794, 797, 806, 809):
  - bg-blue-500 text-white -> btn-glass text-cyan
  - bg-red-500 text-white -> btn-glass text-red-400
  - bg-green-500 text-white -> btn-glass text-green-400
  - bg-red-400 text-white -> btn-glass text-red-400
Legend (~828): 'text-gray-600' -> 'text-platinum/60'

=== renderIZOLRowCells() (linie ~841-910) ===
Linia ~849 inputCls: 'w-full input-glass px-1 py-0.5 text-sm text-center'
Wszystkie td: 'border border-gray-300' -> 'border border-white/[0.12]'
Rp input (~889): 'border border-gray-300' -> 'input-glass', disabledCls: ' bg-gray-100 text-gray-400 cursor-not-allowed' -> ' bg-white/[0.03] text-platinum/30 cursor-not-allowed'
ZACHOWAJ .izol-verdict-cell

=== renderAttachment2() (linie ~912-999) ===
Ten sam wzorzec co Attachment1:
- Table: 'border border-gray-300' -> 'border border-white/[0.12]'
- thead: 'bg-gray-50' -> 'bg-white/[0.06]'
- Headings: dodaj text-platinum
- Buttons: bg-blue-600 -> btn-primary-glass
- Fixed row: bg-yellow-50 -> bg-cyan/5
- Section/subsection rows: bg-gray-200 -> bg-white/[0.08], bg-gray-100 -> bg-white/[0.04]
- Legend: text-gray-600 -> text-platinum/60

=== renderAttachment3() (linie ~1002-1106) ===
Linie ~1004-1005:
  inputCls: 'w-full input-glass px-1 py-0.5 text-sm text-center'
  selectCls: 'w-full select-glass px-1 py-0.5 text-sm'
Wszystkie borders: border-gray-300 -> border-white/[0.12]
thead: bg-gray-50 -> bg-white/[0.06]
Empty state (~1088): text-gray-400 -> text-platinum/30
Add button (~1095): bg-blue-600... -> btn-primary-glass
ZACHOWAJ .rcd-verdict-cell

=== renderAttachment4() (linie ~1108-1200) ===
Linia ~1110 inputCls: 'w-full input-glass px-1 py-0.5 text-sm text-center'
Ten sam wzorzec border/thead/button co inne attachmenty
ZACHOWAJ .uziem-rpo-cell, .uziem-verdict-cell

=== updateOrzeczenieDisplay() (linie ~1796-1825) ===
Linia ~1805: orzText.className = 'text-lg font-bold text-green-700 mb-2';
Linia ~1811: orzText.className = 'text-lg font-bold text-red-700 mb-2';
Linia ~1814: orzText.className = 'text-base text-platinum/30 mb-2';
Linia ~1820: nextDateEl.className = 'text-sm text-platinum/70';
Linia ~1823: nextDateEl.className = 'text-sm text-platinum/30';

=== showToast() (linia ~1833) ===
toast.className = 'fixed bottom-4 right-4 bg-void-2/90 backdrop-blur-xl border border-white/[0.12] text-platinum px-4 py-2 rounded-xl shadow-lg text-sm z-50';

Uwaga: "bg-void-2" to token z Plan 01-01 (@theme --color-void-2: #14141e). Jesli token nosi inna nazwe (np. void-dark, void-secondary), uzyj odpowiednika. Jesli nie istnieje, uzyj bg-[#14141e]/90.

=== font-mono na komorkach numerycznych ===
Dodaj font-mono do:
- .zsmax-cell td jesli nie ma jeszcze
- inputCls w tabelach (juz ma text-center, dodaj font-mono)
- Nie usuwaj font-mono tam gdzie juz jest (np. .id-cell)
  </action>
  <verify>
    <automated>python3 -c "
import re
html = open('index.html').read()
js_section = html[html.find('<script>'):]

checks = [
    # Component classes used in JS
    ('card-glass p-6 mb-4' in js_section, 'card-glass used in JS render'),
    ('input-glass px-2 py-1.5' in js_section or 'input-glass px-1 py-0.5' in js_section, 'input-glass used in JS render'),
    ('select-glass' in js_section, 'select-glass used in JS render'),
    ('btn-primary-glass' in js_section, 'btn-primary-glass used in JS render'),
    ('hero-result-card p-6 mb-4' in js_section, 'hero-result-card used in JS render'),

    # Preserved JS selectors
    ('verdict-cell' in js_section, 'verdict-cell class preserved'),
    ('id-cell' in js_section, 'id-cell class preserved'),
    ('zsmax-cell' in js_section, 'zsmax-cell class preserved'),
    ('izol-verdict-cell' in js_section, 'izol-verdict-cell class preserved'),
    ('rcd-verdict-cell' in js_section, 'rcd-verdict-cell class preserved'),
    ('uziem-rpo-cell' in js_section, 'uziem-rpo-cell class preserved'),
    ('uziem-verdict-cell' in js_section, 'uziem-verdict-cell class preserved'),

    # Preserved classList classes
    ('text-green-700' in js_section, 'text-green-700 preserved for verdicts'),
    ('text-red-700' in js_section, 'text-red-700 preserved for verdicts'),

    # Tab styling in renderTabs
    ('border-cyan text-cyan' in js_section, 'active tab cyan in renderTabs'),
    ('text-platinum/50' in js_section, 'inactive tab platinum in renderTabs'),

    # Dark table styling
    ('border-white/[0.12]' in js_section, 'dark borders in tables'),
    ('bg-white/[0.06]' in js_section, 'dark thead in tables'),

    # Toast styling
    ('backdrop-blur-xl' in js_section, 'glass toast styling'),

    # updateOrzeczenieDisplay
    ('text-platinum/30 mb-2' in js_section, 'orzeczenie no-data platinum styling'),
    ('text-platinum/70' in js_section, 'next date platinum styling'),

    # No leftover light theme patterns in render functions (spot check)
    ('bg-white border rounded p-4 mb-4' not in js_section, 'no leftover bg-white card pattern in JS'),
]
passed = sum(1 for ok, _ in checks if ok)
for ok, msg in checks:
    print(f\"{'PASS' if ok else 'FAIL'}: {msg}\")
print(f\"\n{passed}/{len(checks)} checks passed\")
assert passed == len(checks), f'{len(checks)-passed} checks failed'
print('All JS render function checks passed.')
"
    </automated>
    <manual>
Otworz index.html w przegladarce i przetestuj pelna funkcjonalnosc:
1. Zakladka Protokol — glassmorphism karty, ciemne inputy z cyan focus
2. Wypelnij dane formularza — inputy przyjmuja tekst/liczby
3. Zakladka Zal.1 SWZ — ciemna tabela, dodaj wiersz, wpisz dane, sprawdz obliczenia Id/Zsmax/verdykt
4. Zakladki Zal.2, 3, 4 — tabele renderuja sie, dodawanie wierszy dziala, obliczenia poprawne
5. Orzeczenie (Hero Result Card) — wyrozniajacy gradient, poprawny verdykt NADAJE SIE / NIE NADAJE SIE
6. Zapisz -> Wczytaj -> dane zachowane
7. Eksport PDF -> PDF generuje sie poprawnie (jasne tlo, polskie znaki)
8. Eksport/Import JSON -> dane round-trip
9. Eksport/Import Word -> dane round-trip
10. Toast pojawia sie w dark glass stylu
    </manual>
  </verify>
  <done>Wszystkie JS render functions uzywa nowych klas glassmorphism. Selektory JS (.verdict-cell, .id-cell, etc.) zachowane. Obliczenia dzialaja. Eksporty dzialaja. Cala aplikacja wyswietla dark luxury UI.</done>
</task>

</tasks>

<verification>
Po zakonczeniu obu taskow, pelna weryfikacja:

1. Wizualna: Otworz index.html — brak bialych/szarych elementow z poprzedniego designu, caly UI w dark luxury theme
2. Glassmorphism: Karty maja frosted glass effect z 24px zaokragleniami, inputy maja ciemne tlo i cyan focus
3. Nawigacja: Top bar z glass buttons, tabs z cyan active indicator, footer w dark theme
4. Hero Card: Orzeczenie ma gradient tlo z cyan akcentem
5. Tabele: Dark thead, cienkie bordery, font-mono na wartosciach numerycznych
6. Funkcjonalnosc: Wszystkie 5 zakladek dziala, obliczenia poprawne, eksporty generuja sie bez bledow
7. Selektory: querySelector('.verdict-cell') itd. zwracaja elementy w DOM
8. Single file: Caly kod w jednym index.html
</verification>

<success_criteria>
- Blok @layer components zawiera 7 klas komponentow (card-glass, input-glass, select-glass, btn-glass, btn-primary-glass, hero-result-card, oraz ich warianty hover/focus)
- Zero bialych bg-white/bg-gray-50 kart w JS render functions
- Wszystkie 7 klas selektorow JS obecne w DOM po renderowaniu
- Top bar buttons maja btn-glass / btn-primary-glass
- Zakladki: aktywna = cyan border + text, nieaktywne = platinum/50
- Footer: border-white/10, text-platinum/40, link hover:text-cyan
- Orzeczenie sekcja uzywa hero-result-card
- Tabele: bg-white/[0.06] thead, border-white/[0.12] cells
- Inputy: input-glass z cyan focus glow
- Toast: glass styling z backdrop-blur
- PDF/Word/JSON export dzialaja bez bledow
- Plik index.html jest jedynym plikiem (PRES-08)
</success_criteria>

<output>
After completion, create `.planning/phase-1/01-02/01-02-SUMMARY.md`
</output>
