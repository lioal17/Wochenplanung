# PROJECT_STRUCTURE.md – Komplette Projektstruktur

**Stand:** 2026-08-04 · Referenz-Commit `48dcb40` (`main`)

Dokumentiert jede Datei und jeden Ordner des Projekts mit **Zweck**, **Funktion**
und **Abhängigkeiten**.

---

## 1. Übersicht

```
Wochenplanung/
├── index.html                          ★ DIE KOMPLETTE ANWENDUNG (1,5 MB)
├── README.md                             Projektbeschreibung & Einstieg
├── RESTORE.md                            Wiederherstellungsanleitung
├── PROJECT_STRUCTURE.md                  Diese Datei
├── CHANGELOG_BASELINE.md                 Referenzstand 2026-08-04
├── CLAUDE.md                             Verbindliche Projektregeln
├── SECURITY.md                           Sicherheits-/Datenschutzkonzept
├── PROJEKT.md                            Ausführliche Fachdokumentation
├── UEBERGABE.md                          Übergabedokumentation
├── .gitignore                            Schutz vor Daten-Commits
├── .nojekyll                             GitHub-Pages-Schalter
├── .github/
│   └── workflows/
│       └── deploy-pages.yml              Deployment-Automatik
├── RECOVERY-2026-07-10/                  Analyse-/Recovery-Doku (8 Dateien)
│   ├── 00-README-WIEDERHERSTELLUNG.md
│   ├── 01-PROJEKTZUSAMMENFASSUNG.md
│   ├── 02-LAYOUT-DESIGNSYSTEM.md
│   ├── 03-FEATURES.md
│   ├── 04-DEAD-CODE-ANALYSE.md
│   ├── 05-GITHUB-ANALYSE.md
│   ├── 06-VERSIONS-SNAPSHOT.md
│   └── 07-FINALISIERUNG.md
├── docs/
│   └── archiv/                           Historische Berichte (3 Dateien)
│       ├── ANALYSE-BERICHT-2026-07-05.md
│       ├── PROJEKT-ZUSAMMENFASSUNG-2026-06-19.md
│       └── PROJEKT-ZUSAMMENFASSUNG-2026-07-03.md
└── _git-repo/                            (nur im ZIP-Paket)
    └── Wochenplanung-repo.bundle         Komplette Git-Historie
```

**Bemerkenswert:** Es gibt bewusst **keine** Ordner `src/`, `assets/`, `css/`,
`js/`, `img/`, `fonts/`, `node_modules/`, `dist/` oder `build/`. Alles liegt in
`index.html`.

---

## 2. Die Anwendung

### `index.html` ★

| | |
|---|---|
| **Zweck** | Die vollständige Anwendung – Struktur, Design, Logik, Bibliotheken |
| **Größe** | 1'500'156 Bytes · 4'718 Zeilen |
| **SHA-256** | `4ed904b36a1843b77210a6279339450501a6fe8148791728b3f2799e3d3b47df` |
| **Abhängigkeiten** | **Keine** lokalen Dateien. Optional zwei CDNs (nur Dokument-Import) |

#### Innerer Aufbau

| Bereich | Umfang | Inhalt |
|---|---|---|
| `<head>` / Meta | – | CSP, Viewport, Favicon + Apple-Touch-Icon als `data:`-URI |
| `<style>`-Blöcke | 4 Blöcke, ~129'800 Zeichen | Komplettes Design-System |
| HTML-Struktur | – | 6 Views + Overlays/Modals |
| `<script>`-Blöcke | 6 Blöcke, ~1'246'500 Zeichen | Bibliotheken + Anwendungslogik (177 Funktionen) |

#### Die 6 Ansichten (Views)

| View-ID | Bereich | Funktion |
|---|---|---|
| `view-plan` | **Wochenplan** | Kernansicht: VM/NM-Einteilung, Drag-and-Drop-Zuweisung, Abwesenheitscodes, eCase-Tracking, automatische Sperrung von Schul-/Sportterminen |
| `view-person` | **Personen** | Teilnehmerverwaltung, Stammdaten, Fixpräferenzen für wiederkehrende Werkstatt-Zuweisungen |
| `view-wbtn` | **WB-TN** | Wirkungsberichte: Übersicht, Erinnerungen, fortlaufender Runden-Zyklus ab Eintrittsdatum |
| `view-monthly` | **Monat** | Monatsübersicht als Aggregation der Wochenpläne |
| `view-rapport` | **Rapport** | Monatsrapport im BASISJOB-Format, Archivierung |
| `view-import` | **Import** | Datenübernahme aus Excel, PDF, Word sowie JSON-Datensicherung |

Umschaltung über die Funktion `show('<view>')`.

#### Wichtige UI-Elemente

| Element-ID | Funktion |
|---|---|
| `planArea`, `dayBar`, `weekLabel` | Wochenplan-Raster, Tagesleiste, Wochennavigation |
| `pList`, `dz` | Teilnehmerliste, Drop-Zone für Drag-and-Drop |
| `legendBox`, `planEcBadge` | Legende der Codes, eCase-Kennzeichnung |
| `kursTags`, `schTags`, `sptTags` | Kurs-, Schul-, Sport-Marker |
| `moArea`, `moLabel`, `moHint` | Monatsansicht |
| `rptArea`, `rapportOverlay` | Rapport-Ansicht und -Dialog |
| `wbtnList` | Wirkungsbericht-Liste |
| `impPreviewOverlay`, `impPreviewBody`, `impProgress`, `impResults` | Import-Vorschau mit Fortschritt und Ergebnisbericht |
| `pmOverlay`, `pmTitle`, `pmExtra` | Personen-Detaildialog |
| `neoOverlay` | Neophyt-/Ersteintritts-Dialog |
| `toastDock` | Benachrichtigungen (mit `@keyframes toast-in`) |
| `pageDragOverlay` | Datei-Drag-and-Drop über die ganze Seite |
| `saveFileLabel`, `saveInfo` | Anzeige des letzten Sicherungszeitpunkts |

---

## 3. Design-System (in `index.html`)

| Aspekt | Umsetzung |
|---|---|
| **Farbsystem** | 32 CSS-Custom-Properties in `:root` |
| **Graustufen** | `--g0` (#ffffff) bis `--g9` (#141414) – 10 Stufen |
| **Akzentfarbe** | `--accent: #ffed00` (Gelb), `--accent-bg: #fffdf0` |
| **Markenfarben** | `--navy: #141414`, `--navy-dark: #000000`, `--navy-light: #333333` |
| **Statusfarben** | grün `#059669`, rot `#c0392b`, amber `#d97706`, blau `#2e6db4`, violett `#7c3aed`, teal `#0d9488` – je mit passender `-bg`-Variante |
| **Tabellen-Zebra** | `--zebra: #daeef3` (Bildschirm); PDF-Zebra `#fff9d6` (seit Commit `47c8f00`) |
| **Radius** | `--r: 8px` |
| **Layout** | CSS Grid & Flexbox |
| **Responsive** | `<meta viewport>` + Breakpoint `@media(max-width:640px)`; Drag-and-Drop-Bedienung ist auf Desktop ausgelegt |
| **Animation** | `@keyframes toast-in` + 7 `transition`-Regeln |
| **Icons/Logo** | 3 `data:image`-URIs (Favicon als SVG, Apple-Touch-Icon als PNG) – **keine** externen Bilddateien |
| **Schriftarten** | Systemschriften (font-stack) – **keine** Webfont-Dateien, keine Google Fonts |

> Wichtig für die Wiederherstellung: Da Icons als `data:`-URIs eingebettet und
> Schriften Systemschriften sind, gibt es **keine fehlenden Assets** und **keine
> zerbrechlichen relativen Pfade**.

---

## 4. Mechanik – Funktionsgruppen

Insgesamt **177 Funktionen**. Die für die Wiederherstellung wichtigsten:

### 4.1 PDF-Erzeugung (jsPDF 2.5.1 + AutoTable 3.8.2, offline)

| Funktion | Zweck |
|---|---|
| `genPDF` | Zentrale PDF-Erzeugung |
| `genWeekPDF` | Wochenplan als PDF |
| `genTNPDF` | Teilnehmerbezogenes PDF |
| `genRapportPDF` | Monatsrapport (BASISJOB-Format) |
| `genAllRapportsPDF` | Sammel-PDF aller Rapporte |
| `genNeophytPDF` | PDF für Neueintritte |
| `drawRapportPage` | Seitenaufbau des Rapports |
| `pdfHead`, `pdfFoot` | Kopf-/Fußzeilen-Layout |

### 4.2 Import

| Funktion | Zweck | Abhängigkeit |
|---|---|---|
| `doImport` | Import-Einstieg | – |
| `doImportFSA` | Excel-Import | SheetJS 0.18.5 (eingebettet, offline) |
| `doImportDoc` | Word-Import | mammoth 1.6.0 (**CDN**) |
| `_impParsePDF` | PDF-Import | pdf.js 3.11.174 (**CDN**) |
| `processImportJSON` | JSON-Datensicherung einlesen | – |
| `_jsonImportDiff` | Abgleich Bestand ↔ Import | – |
| `openImportPreview`, `confirmImportPreview`, `closeImportPreview` | Vorschau vor Übernahme | – |

### 4.3 Rapport / Wirkungsberichte

`openRapportModal`, `closeRapportModal`, `renderRapport`, `isRapportArchived`

### 4.4 Datenhaltung

| Element | Wert |
|---|---|
| Datenbank-Schlüssel | `lw_db_v3` (Konstante `DBKEY`) |
| Zeitstempel letzte Sicherung | `lw_lastsave` (Konstante `SAVEKEY`) |
| Speicherort | `localStorage` des Browsers – **kein Server, keine Datenbank** |

### 4.5 Sicherheit

`esc()` – Pflicht-Escaping für alle in HTML gerenderten Nutzerdaten (XSS-Schutz).

---

## 5. Konfigurations- und Infrastrukturdateien

| Datei | Zweck | Funktion | Abhängigkeiten |
|---|---|---|---|
| `.github/workflows/deploy-pages.yml` | Deployment | Bei Push auf `main` (oder manuell): kopiert **nur** `index.html` + `.nojekyll` nach `_site/` und veröffentlicht auf GitHub Pages. Interne Doku bleibt bewusst außen vor. | GitHub Actions: `checkout@v4`, `configure-pages@v4`, `upload-pages-artifact@v3`, `deploy-pages@v4`; Pages-Source muss auf „GitHub Actions" stehen |
| `.nojekyll` | Pages-Schalter | Leere Datei; verhindert Jekyll-Verarbeitung | wird vom Workflow mitkopiert |
| `.gitignore` | Datenschutz | Blockiert `*.json`, `*.csv`, `*.xlsx`, `*.xls` (Export-Dateien mit Personendaten) sowie `_site/`, OS-/Editor-Reste. Ausnahme: `/.github/**/*.json` | – |

---

## 6. Dokumentationsdateien

| Datei | Zweck |
|---|---|
| `README.md` | Projektbeschreibung, Aufbau, Technologien, Start, Hauptkomponenten |
| `RESTORE.md` | Vollständige Wiederherstellung inkl. GitHub-Pages-Neueinrichtung |
| `PROJECT_STRUCTURE.md` | Diese Datei – Struktur mit Zweck/Funktion/Abhängigkeiten |
| `CHANGELOG_BASELINE.md` | Referenzstand 2026-08-04 als Ausgangspunkt für spätere Änderungen |
| `CLAUDE.md` | Verbindliche Projektregeln – Datenschutz als oberstes Gebot, Arbeitsweise |
| `SECURITY.md` | Sicherheits- und Datenschutzkonzept |
| `PROJEKT.md` | Ausführliche fachliche Dokumentation (~20 KB) |
| `UEBERGABE.md` | Übergabedokumentation (~12 KB) |

### `RECOVERY-2026-07-10/` – Analysepaket vom 10.07.2026

| Datei | Inhalt |
|---|---|
| `00-README-WIEDERHERSTELLUNG.md` | Einstieg ins Analysepaket |
| `01-PROJEKTZUSAMMENFASSUNG.md` | Fachliche Gesamtübersicht |
| `02-LAYOUT-DESIGNSYSTEM.md` | **Layout- und Designsystem im Detail** – zentrale Referenz zur Rekonstruktion des UI-Designs |
| `03-FEATURES.md` | Featureliste |
| `04-DEAD-CODE-ANALYSE.md` | Analyse ungenutzten Codes |
| `05-GITHUB-ANALYSE.md` | Repository-/Workflow-Analyse |
| `06-VERSIONS-SNAPSHOT.md` | Versionsstand der Bibliotheken |
| `07-FINALISIERUNG.md` | Abschlussbericht |

### `docs/archiv/` – historische Berichte

`ANALYSE-BERICHT-2026-07-05.md`, `PROJEKT-ZUSAMMENFASSUNG-2026-06-19.md`,
`PROJEKT-ZUSAMMENFASSUNG-2026-07-03.md`

> Diese drei Dateien lagen bis Juli 2026 im Wurzelverzeichnis und wurden später
> nach `docs/archiv/` verschoben. In alten `sicherung-*`-Branches erscheinen sie
> deshalb noch am alten Ort – inhaltlich fehlt nichts.

---

## 7. Was es im Projekt bewusst NICHT gibt

| Nicht vorhanden | Grund |
|---|---|
| Datenbank, Dumps, Migrationen | Daten liegen ausschließlich im `localStorage` |
| `.env` / `.env.example` | Keine Umgebungsvariablen, keine Schlüssel, keine Endpunkte |
| `package.json` / `node_modules` | Keine Paketabhängigkeiten – Bibliotheken sind eingebettet |
| Build-Tools, `dist/`, `build/` | Kein Build-Schritt – die Quelle **ist** das Auslieferungsartefakt |
| Framework (React/Vue/Angular), TypeScript | Bewusst Vanilla-JavaScript |
| Sass/SCSS/Tailwind | Reines CSS mit Custom Properties |
| Separate Asset-/Font-/Icon-Ordner | Icons als `data:`-URIs, Systemschriften |
| Backend, API, Serverkonfiguration | Reine Offline-/Local-First-Anwendung |
| Tests / CI-Testpipeline | Nur der Pages-Deployment-Workflow |

---

## 8. Abhängigkeitsdiagramm

```
                        index.html
                            │
        ┌───────────────────┼────────────────────┐
        │                   │                    │
   EINGEBETTET          BROWSER-APIs         OPTIONAL (CDN)
   (offline)                │                nur Dokument-Import
        │                   │                    │
  jsPDF 2.5.1          localStorage         pdf.js 3.11.174
  AutoTable 3.8.2      File / FileReader    mammoth 1.6.0
  SheetJS 0.18.5       Blob / ObjectURL          │
        │              Drag & Drop API           │
        ▼                   ▼                    ▼
  Alle PDF-Exporte,   Datenhaltung &        PDF-/Word-Import
  Excel-Import        JSON-Sicherung        (entfällt ohne Internet –
  → laufen offline    → lokal, gerätegebunden  App läuft weiter)
```

**Deployment-Kette:**
`Push auf main` → `deploy-pages.yml` → kopiert `index.html` + `.nojekyll` nach
`_site/` → `upload-pages-artifact` → `deploy-pages` → `https://lioal17.github.io/Wochenplanung/`
