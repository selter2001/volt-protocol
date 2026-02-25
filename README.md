# VoltProtocol

**Generator protokołów kontrolno-pomiarowych PN-HD 60364-6**

Electrical inspection protocol generator per PN-HD 60364-6 — runs entirely in the browser, no installation required.

> Created by **Wojciech Olszak**

<h3 align="center">
  <a href="https://selter2001.github.io/volt-protocol/">Otwórz aplikację / Open App</a>
</h3>

---

## Screenshots

### Formularz główny / Main Form
![Protokół - dane ogólne](screenshots/01-protokol-form.png)
![Protokół - przyrządy i ocena](screenshots/02-protokol-instruments.png)
![Protokół - ocena końcowa i orzeczenie](screenshots/03-protokol-assessment.png)

### Załączniki pomiarowe / Measurement Appendices
![Zał. 1 - Skuteczność zerowania (SWZ)](screenshots/04-swz-table.png)
![Zał. 2 - Rezystancja izolacji](screenshots/05-izolacja-table.png)
![Zał. 3 - Wyłączniki różnicowoprądowe (RCD)](screenshots/06-rcd-table.png)
![Zał. 4 - Rezystancja uziemienia](screenshots/07-uziemienie-table.png)

---

## PL — Opis

Darmowa aplikacja webowa do generowania protokołów z pomiarów elektrycznych zgodnie z normą **PN-HD 60364-6**.

### Funkcje

- **4 załączniki pomiarowe** — skuteczność zerowania (SWZ), rezystancja izolacji, wyłączniki różnicowoprądowe (RCD), uziemienia
- **Automatyczne obliczenia** — Id, Zsmax, Ia, Rpo, ocena POZYTYWNA/NEGATYWNA
- **Baza zabezpieczeń** — charakterystyki B/C/D z prądami znamionowymi 6–125A
- **Dynamiczne sekcje** — dodawaj/usuwaj sekcje, podtytuły i wiersze pomiarowe
- **Formularz główny** — dane obiektu, wykonawca, przyrządy pomiarowe, ocena końcowa z orzeczeniem
- **Eksport PDF** — profesjonalny protokół do pobrania, polskie znaki, paginacja, landscape dla Zał. 2
- **Eksport Word** — eksport/import .doc z round-trip danych
- **Zapis/odczyt** — localStorage + eksport/import JSON między komputerami
- **Zero instalacji** — jeden plik HTML, działa w przeglądarce

### Design

Aplikacja wykorzystuje dark luxury theme "Industrial Elegance":
- **Void Black** (#0a0a0f) tło z noise texture
- **Electric Cyan** (#00f0ff) akcenty interaktywne
- **Soft Platinum** (#e0e0e8) tekst i bordery
- Glassmorphism karty z backdrop-blur i ultra-thin borders
- Animacje count-up na wynikach obliczeń
- Neomorphic pressed-in efekt na przyciskach
- Glow effect na verdyktach (zielony/czerwony)
- Space Grotesk + JetBrains Mono (Google Fonts)

### Jak używać

1. Otwórz stronę w przeglądarce
2. Wypełnij dane protokołu i pomiary
3. Kliknij **Eksportuj PDF** aby pobrać gotowy protokół
4. Opcjonalnie: **Zapisz** do localStorage lub **Eksport JSON** aby przenieść dane

---

## EN — Description

Free web application for generating electrical measurement protocols according to **PN-HD 60364-6** (Polish implementation of IEC 60364-6).

### Features

- **4 measurement appendices** — loop impedance (SWZ), insulation resistance, RCD testing, earthing
- **Automatic calculations** — fault current, max impedance, protective current, assessed results
- **Protection device database** — B/C/D characteristics, rated currents 6–125A
- **Dynamic sections** — add/remove sections, subsections, and measurement rows
- **Main form** — facility data, inspector details, instruments, final assessment with verdict
- **PDF export** — professional protocol download with Polish characters, pagination, landscape for Appendix 2
- **Word export** — .doc export/import with full data round-trip
- **Save/load** — localStorage persistence + JSON export/import for portability
- **Zero installation** — single HTML file, runs in any modern browser

### Design

The app features a dark luxury "Industrial Elegance" theme:
- **Void Black** (#0a0a0f) background with noise texture
- **Electric Cyan** (#00f0ff) interactive accents
- **Soft Platinum** (#e0e0e8) text and borders
- Glassmorphism cards with backdrop-blur and ultra-thin borders
- Count-up animations on calculation results
- Neomorphic pressed-in button effects
- Verdict glow effects (green/red)
- Space Grotesk + JetBrains Mono typography (Google Fonts)

### Usage

1. Open the page in your browser
2. Fill in protocol data and measurements
3. Click **Eksportuj PDF** to download the completed protocol
4. Optionally: **Zapisz** to save locally or **Eksport JSON** to transfer data

---

## Tech Stack

- Vanilla HTML/CSS/JS — single file, no build step
- [Tailwind CSS v4](https://tailwindcss.com/) (Play CDN) with custom `@theme` design tokens
- [pdfmake 0.3.5](https://pdfmake.github.io/docs/) for PDF generation
- [Google Fonts](https://fonts.google.com/) — Space Grotesk, JetBrains Mono, Roboto
- CSS glassmorphism (`backdrop-filter`, `@layer components`)
- `requestAnimationFrame` count-up animations
- `prefers-reduced-motion` accessibility support

## Author

**Wojciech Olszak** — [GitHub](https://github.com/selter2001)

## License

[MIT](LICENSE) © 2026 Wojciech Olszak
