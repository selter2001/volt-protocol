# Phase 3 Research: Flow Effect & Stress Test

## Export/Storage Architecture

### PDF Export (linia ~3106)
- **Biblioteka**: pdfmake v0.3.5 (CDN)
- **Budowanie**: `buildDocDefinition()` (~3087) → `exportPDF()` (~3106)
- **Izolacja od CSS**: PEŁNA — hardkodowane kolory (`#9ca3af`), brak odwołań do tokenów CSS
- **Polskie znaki**: Unicode escape sequences (`\u00d3`, `\u00f3`, `\u0142`)
- **Landscape Zał.2**: linia ~3099: `pageOrientation: 'landscape'`
- **Funkcje zawartości**:
  - `buildFormMainContent()` (~2599)
  - `buildAttachment1Content()` (~2734) — SWZ, 13 kolumn
  - `buildAttachment2Content()` (~2857) — IZOL, 15 kolumn, landscape
  - `buildAttachment3Content()` (~2969) — RCD
  - `buildAttachment4Content()` (~3029) — UZIEM

### Word Export (linia ~2537)
- **Metoda**: Raw HTML z MS Office XML namespaces (nie docx.js)
- **Izolacja od CSS**: PEŁNA — hardkodowane kolory (`#d1d5db`, `#f3f4f6`, `green`, `red`)
- **Round-trip**: AppState zakodowany base64 w komentarzu HTML dla importu
- **Landscape Zał.2**: `@page Section2 { size: 297mm 210mm; mso-page-orientation: landscape; }`
- **Import**: `importFromWord()` (~2551) — wyciąga dane z komentarza HTML

### JSON Export/Import (linie ~2149/~2163)
- **Serializacja**: `JSON.stringify(AppState, null, 2)` — pretty-print
- **Deseriializacja**: `JSON.parse()` → `restoreAppState()` (~2111)
- **Format pliku**: `voltprotokol-{nr}.json`
- **Izolacja od CSS**: PEŁNA — czysta serializacja obiektu

### localStorage (linie ~2124/~2137)
- **Klucz**: `voltprotokol-state`
- **Zapis**: `localStorage.setItem(key, JSON.stringify(AppState))`
- **Odczyt**: `localStorage.getItem(key)` → `JSON.parse()` → `restoreAppState()`
- **Obsługa błędów**: QuotaExceededError, uszkodzone dane
- **Autosave**: BRAK — tylko manualne (przycisk)

### restoreAppState() (~2111)
- Deep copy obiektu → podmiana AppState → przeliczenie → re-render wszystkiego
- Krytyczna funkcja dla JSON import, Word import, localStorage load

## Funkcje obliczeniowe

Wszystkie 4 funkcje są CZYSTO OBLICZENIOWE — zero zależności od DOM/CSS:

| Funkcja | Linia | Wejście | Wyjście |
|---------|-------|---------|---------|
| `calcSWZRow()` | ~386 | Ia, Usk, Zs | Id, Zsmax, verdict |
| `calcIZOLRow()` | ~395 | phaseType, rp[], Rw | verdict |
| `calcRCDRow()` | ~403 | Iw, In, tw, tz | verdict |
| `calcUZIEMRow()` | ~411 | Rp, Wk, Rw | Rpo, verdict |

## Zarządzanie dynamicznymi sekcjami/wierszami

- **Centralny delegator**: `bindAppEvents()` (~1490)
- **Click**: `[data-action]` — add-section, add-subsection, add-row-*, remove-*
- **Input**: `[data-field]` — pola tekstowe, wartości numeryczne
- **Change**: `[data-field]` — selecty (baseType, baseCurrent, phaseType, testResult)
- **EventBus pipeline**: sections:changed → render oba attachmenty; attachment{N}:changed → render attachment N

## Flow Effect (ANIM-04)

### Stan infrastruktury
- **SVG**: BRAK — jedyny SVG to noise texture w body (feTurbulence)
- **Path/line/animate**: ZERO istniejących elementów
- **Połączenia między sekcjami**: BRAK koncepcji w kodzie

### Ocena wykonalności
- Wymagałoby budowy od zera: SVG overlay, śledzenie pozycji sekcji, animacje linii
- Ryzyko performance: SVG overlay + resize observer + animacje na każdy re-render
- Single-file constraint: cały kod SVG + animacji w jednym HTML
- Tabbed layout: sekcje nie są widoczne jednocześnie (każda zakładka to osobny attachment)
- **WNIOSEK**: Flow Effect między sekcjami nie ma sensu wizualnego w tabbed layout — użytkownik widzi tylko jedną zakładkę naraz

### Rekomendacja
**Świadome pominięcie ANIM-04** — tabbed layout uniemożliwia wizualny przepływ między sekcjami. Wartość UX jest zerowa, ryzyko performance wysokie.

## Stress Test — kluczowe punkty weryfikacji

1. **20+ wierszy w SWZ i IZOL**: dodać sekcje/podsekcje/wiersze, sprawdzić płynność
2. **PDF z pełnymi danymi**: landscape Zał.2, polskie znaki, verdykty
3. **Word z pełnymi danymi**: landscape, round-trip (export → import)
4. **JSON round-trip**: export → import → porównanie stanu
5. **localStorage round-trip**: save → reload strony → load → porównanie
6. **Obliczenia**: SWZ (Id, Zsmax), IZOL (verdict), RCD (verdict), UZIEM (Rpo, verdict)
7. **prefers-reduced-motion**: animacje wyłączone, aplikacja działa normalnie
8. **Dynamiczne operacje**: add/remove sekcji, podsekcji, wierszy we wszystkich zakładkach
