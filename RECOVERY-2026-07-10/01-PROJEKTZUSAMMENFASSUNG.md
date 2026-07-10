# Projektzusammenfassung – Stand 10.07.2026

## 1. Überblick

**Lernwerkstatt – Einteilungsplan (Wochenplanung)** ist ein Wochenplanungs-Tool für das
**BASISJOB Motivationssemester** der Lernwerkstatt. Es verwaltet Teilnehmende (TN), plant
deren Wochen (Vormittag/Nachmittag-Einteilung in Werkstätten/Schule/Sport), erfasst
Abwesenheiten und den **eCase**-Status, verwaltet **Wirkungsberichte (WB-TN)** und erzeugt
diverse **PDF-Auswertungen**.

- **Zielgruppe/Nutzung:** intern, ein bis wenige Bearbeiter:innen (Jobcoaches Lio/Tom/Yvi).
- **Betriebsart:** **reine Offline-/Local-First-Webapp**, kein Server, kein Login, kein Tracking.

---

## 2. Architektur

| Aspekt | Umsetzung |
|---|---|
| Architekturtyp | **Single-File-Webapp** – die gesamte App ist **eine `index.html`** (HTML + CSS + JS inline) |
| Build-Prozess | **Keiner.** Kein Bundler, kein Transpiler, kein `npm`-Build, keine `package.json` |
| Server/Backend | **Keiner.** Kein API-Server, keine Datenbank, keine Cloud |
| Datenhaltung | **Ausschließlich lokal** im Browser (`localStorage`, optional File System Access API + IndexedDB-Handle) |
| Rendering | Vanilla JS, direkte DOM-Manipulation über `innerHTML`-Templates; Views werden per `show(name)` umgeschaltet |
| Zustandsmodell | Ein globales `DB`-Objekt (In-Memory) ↔ `localStorage` (Persistenz via `saveDB()`/`loadDB()`) |
| Sicherheit | Strikte **Content-Security-Policy** (`default-src 'none'`), `esc()`-XSS-Härtung, `sanitizeDB()`-Importvalidierung, SRI auf CDN-Skripten |

**Designphilosophie:** Maximale Portabilität & Datensparsamkeit. Die Datei ist alleinlauffähig
(Doppelklick im Browser). Alles, was TN-Daten nach außen tragen könnte, ist bewusst unterbunden.

---

## 3. Verwendete Technologien

- **HTML5 / CSS3 / Vanilla JavaScript** (ES2020+, keine Frameworks).
- **CSS Custom Properties** (Design-Tokens in `:root`), Flexbox & CSS-Grid.
- **Browser-APIs:** `localStorage`, **File System Access API** (`showDirectoryPicker`),
  **IndexedDB** (Ordner-Handle), `Blob`/`URL.createObjectURL` (Downloads).

### Eingebettete Bibliotheken (lokal in `index.html`, offline-fähig)
| Lib | Version | Zweck | Lizenz |
|---|---|---|---|
| **jsPDF** | 2.5.1 | PDF-Erzeugung | MIT |
| **jsPDF-AutoTable** | 3.8.2 | Tabellen in PDFs | MIT |
| **SheetJS (xlsx)** | 0.18.5 | Excel-Import | Apache-2.0 |

### Externe Bibliotheken (per CDN, **nur** für Dokument-Import, brauchen Internet)
| Lib | Version | CDN | SRI |
|---|---|---|---|
| **PDF.js** | 3.11.174 | cdnjs.cloudflare.com | `sha384-…` gesetzt, `crossorigin=anonymous`; `isEvalSupported:false` |
| **Mammoth.js** | 1.6.0 | cdn.jsdelivr.net | `sha384-…` gesetzt, `crossorigin=anonymous` |

> **Offline-Grenze:** Öffnen, Planen, Speichern, JSON Export/Import und **alle PDF-Exporte**
> laufen ohne Internet. **Nur** der PDF-/Word-Import (pdf.js/mammoth) benötigt Netz.

---

## 4. Projektstruktur (Repository)

```
Wochenplanung/
├── index.html                          # DIE APP (HTML+CSS+JS inline) – 1'496'228 B, 4662 Zeilen
├── .nojekyll                           # GitHub Pages: kein Jekyll-Processing
├── .gitignore                          # schützt *.json/*.xlsx/… (TN-Datensicherungen) + _site/
├── .github/workflows/deploy-pages.yml  # CI: deployt NUR index.html + .nojekyll nach GitHub Pages
├── README.md                           # Kurzbeschreibung / Start / Datenschutzhinweis
├── SECURITY.md                         # Sicherheits-/Datenschutzkonzept, CVEs, SRI
├── CLAUDE.md                           # Verbindliche Projektregeln (Datenschutz-Gebot!)
├── PROJEKT.md                          # Vollständige fachlich-technische Beschreibung
├── UEBERGABE.md                        # Übergabe-/Betriebsdokument
├── PROJEKT-ZUSAMMENFASSUNG-2026-06-19.md   # historischer Zwischenstand
├── PROJEKT-ZUSAMMENFASSUNG-2026-07-03.md   # historischer Zwischenstand
├── ANALYSE-BERICHT-2026-07-05.md           # historischer Analysebericht
└── RECOVERY-2026-07-10/                # DIESES Recovery-Paket (neu)
```

### Innerer Aufbau von `index.html`
| Zeilen (ca.) | Inhalt |
|---|---|
| 1–12 | `<head>`, **CSP-Meta**, `<title>Lernwerkstatt – Einteilungsplan</title>` |
| ~13–600 | `<style>` – komplettes CSS (Design-Tokens, Layout, Komponenten) |
| ~600–1020 | `<body>` – Markup aller Views (`view-plan`, `-monthly`, `-rapport`, `-person`, `-wbtn`, `-import`) + Modals |
| 1023–1047 | `<script>` **SheetJS/xlsx** (eingebettet) |
| 1051–1451 | `<script>` **jsPDF 2.5.1** (eingebettet) |
| 1453–1464 | `<script>` **jsPDF-AutoTable 3.8.2** (eingebettet) |
| 1465–4660 | `<script>` **App-Code** (~3194 Zeilen, 180 benannte Funktionen) |
| 1046–1047 | `<script src=…>` **pdf.js** + **mammoth** (CDN, mit SRI) |

---

## 5. Datenmodell

### 5.1 Globaler State
```js
let DB = { participants:[], plans:{}, rptNotes:{} };
```
- `participants` – Array der TN-Objekte.
- `plans` – `plans["YYYY-MM-DD"][participantId] = planEintrag`.
- `rptNotes` – Freitext-Notizen pro TN/Monat (Monatsrapport), Key `"<pid>_<YYYY-MM>"`.

### 5.2 Teilnehmer-Objekt (wichtigste Felder)
`id`, `vorname`, `nachname`, `status` (`aktiv`/`schnupper`/`praktikum`/`lehrstelle`/`prklehr`),
`jobCoach` (`lio`/`tom`/`yvi`), `bewerbenJC`, `stammwerkstatt`, `einsatzVon` (Eintritt),
`einsatzBis` (Ende ZV), `iiz`, `bemerkung`,
`schultage[]` (`{typ, wochentag}`), `kurse[]` (`{typ, erfasstAm, zeit[]}`), `sporttermine[]`,
sowie die **Wirkungsbericht-Felder**:
- `wbRounds: number` – Anzahl abgeschlossener WB-Runden (Default `0`).
- `wbAgInformed: bool` – „Arbeitsagoge informiert" der aktuellen Runde.

### 5.3 Plan-Eintrag (`plans[date][pid]`)
`vm`, `nm` (Werkstatt-Einteilung VM/NM), `abwesenheit` (Absenz-Code oder `''`),
`abwesenheitText`, **`ecaseEntered: bool`** (in eCase erfasst?), `sw` (`S.LW`/`S.EX`
Schnupper-Flag), `bemerkung`.

### 5.4 Persistenz (localStorage / IndexedDB)
| Schlüssel | Inhalt |
|---|---|
| `lw_db_v3` | **Gesamte Datenbank** (`DB`) als unverschlüsseltes JSON; beim Laden über `sanitizeDB()` validiert |
| `lw_lastsave` | Zeitstempel der letzten manuellen Sicherung |
| `lw_db_backup_*` | (optional) automatische Sicherungskopien, z. B. durch das WB-Reset-Snippet |
| IndexedDB `lw_fh` | optionaler Handle auf den gewählten Speicherordner (File System Access API) |

> Es gibt **bewusst keinen** In-App-Knopf zum vollständigen Löschen aller Daten → Löschung
> nur über Browser-Einstellungen (Websitedaten entfernen).

---

## 6. Hauptfunktionen (Kurzüberblick – Details in `03-FEATURES.md`)

1. **Wochenplan** – Tages-/Wocheneinteilung VM/NM je TN, Absenzen, eCase-Tracking.
2. **Monatsübersicht** – Matrix TN × Tage (VM/NM-Zellen), eCase-Badge (global).
3. **Monatsrapport** – Kennzahlen je TN/Monat + Freitext-Notizen, PDF-Export.
4. **Teilnehmer-Verwaltung** – anlegen/bearbeiten/löschen, Schultage/Kurse/Sport.
5. **Dokument-Import** – Teilnehmerlisten aus PDF/Excel/Word.
6. **WB-TN (Wirkungsberichte)** – Runden-Zyklus je TN, Ampel, Badge, Reset.
7. **PDF-Exporte** – Wochenplan, Monatsrapport, Neophyt-Liste, TN-Statistik, Teilnehmer-Rapport.
8. **Sicherung** – JSON-Export/-Import, optional „Speichern ohne Dialog" (Ordner-Handle).

---

## 7. Build, Start & Deployment

- **Installation:** keine. Keine Abhängigkeiten zu installieren, kein `npm install`.
- **Start lokal:** `index.html` im Browser öffnen (Doppelklick). Für den **Dokument-Import**
  wird Internet benötigt (CDN-Libs); alles andere läuft offline.
- **Build:** entfällt (kein Build-Schritt).
- **Deployment (GitHub Pages):** Workflow `.github/workflows/deploy-pages.yml`
  - Trigger: Push auf `main` (oder manuell `workflow_dispatch`).
  - Schritt „Prepare site": kopiert **nur** `index.html` + `.nojekyll` nach `_site/`
    → interne Doku (`PROJEKT.md`, `RECOVERY-*`, …) wird **nicht** veröffentlicht.
  - Upload-Artifact + `deploy-pages@v4`.
- **Umgebungsvariablen:** **keine.** Die App nutzt keine Env-Variablen; der Workflow nutzt nur
  Standard-GitHub-Pages-Permissions (`contents:read`, `pages:write`, `id-token:write`).

---

## 8. Konfigurationsübersicht

| Ort | Konfiguration |
|---|---|
| `index.html` `<head>` | **CSP** (`default-src 'none'`; script/connect nur cdnjs + jsDelivr; `form-action 'none'`; `base-uri 'none'`) |
| `.gitignore` | blockt `*.json`, `*.csv`, `*.xlsx`, `*.xls`, `_site/`, OS-/Editor-Reste; erlaubt `.github/**/*.json` |
| `.nojekyll` | schaltet Jekyll auf GitHub Pages ab |
| `deploy-pages.yml` | Pages-Deploy (siehe §7) |
| CSS `:root` | Design-Tokens (Farben, Radius, Schatten) – siehe `02-LAYOUT-DESIGNSYSTEM.md` |
| In-Code-Konstanten | `WS`, `WS_ABBR`, Schul-/Kurs-/Absenz-Codes, Jobcoaches, Feiertagslogik |

---

## 9. Externe Abhängigkeiten (Zusammenfassung)

- **Runtime, offline:** keine externen Laufzeit-Abhängigkeiten (Libs sind eingebettet).
- **Runtime, online (nur Import):** cdnjs (pdf.js), jsDelivr (mammoth) – per SRI abgesichert.
- **CI:** `actions/checkout@v4`, `actions/configure-pages@v4`,
  `actions/upload-pages-artifact@v3`, `actions/deploy-pages@v4`.

---

## 10. Bekannte Besonderheiten / Betriebshinweise

- **Datenschutz zuerst:** TN-Daten dürfen nie ins Repo/PR/Log/Screenshot (siehe `CLAUDE.md`, `SECURITY.md`).
- **Unverschlüsselter localStorage:** auf geteilten Rechnern nach Nutzung Browserdaten löschen.
- **File System Access API:** nur in Chromium-basierten Browsern voll verfügbar; der Browser
  fragt nach jedem Reload einmalig nach Schreibrecht (nicht abschaltbar).
- **Einmalige Migrationen** beim Laden: `migrateSchnupper`, `migrateSchulCodes` (idempotent).
- **CVE-Hinweise:** jsPDF CVE-2025-29907 (hier nicht ausnutzbar), SheetJS CVE-2023-30533/
  CVE-2024-22363 (Upgrade nur über cdn.sheetjs.com – als Empfehlung geführt).
