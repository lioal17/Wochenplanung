# CHANGELOG_BASELINE.md – Referenzstand (Baseline)

> **Diese Datei beschreibt den eingefrorenen Referenzzustand vom 2026-08-04.**
> Sie ist der Ausgangspunkt für alle späteren Änderungen: Neue Anpassungen werden
> in Abschnitt 8 fortlaufend unterhalb dieser Baseline dokumentiert.

---

## 1. Baseline-Kennzahlen

| Merkmal | Wert |
|---|---|
| **Baseline-Datum** | 2026-08-04 |
| **Referenz-Commit** | `48dcb40` |
| **Commit-Datum** | 2026-07-16, 08:25 (+0200) |
| **Branch** | `main` |
| **Commit-Titel** | Merge pull request #164 (PDF-Zebrafarbe) |
| **Live-URL** | https://lioal17.github.io/Wochenplanung/ |
| **Letztes erfolgreiches Deployment** | 2026-07-16, 06:25 UTC – Status `success`, aus Commit `48dcb40` |
| **Commits in der Historie** | 400 (erster Commit 2026-06-05) |
| **Branches im Repository** | 8 (`main`, 5× `sicherung-*`, Arbeitsbranches) |

### Zustand der Anwendungsdatei

| Merkmal | Wert |
|---|---|
| Datei | `index.html` |
| Größe | 1'500'156 Bytes |
| Zeilen | 4'718 |
| **SHA-256** | `4ed904b36a1843b77210a6279339450501a6fe8148791728b3f2799e3d3b47df` |
| CSS | 4 `<style>`-Blöcke, ~129'800 Zeichen, 32 Custom Properties |
| JavaScript | 6 `<script>`-Blöcke, ~1'246'500 Zeichen, 177 Funktionen |
| Views | 6 (`plan`, `person`, `wbtn`, `monthly`, `rapport`, `import`) |

> Die SHA-256-Prüfsumme ist der maßgebliche Nachweis: Stimmt sie überein, ist die
> Anwendung **bitgenau** im Baseline-Zustand.

---

## 2. Bibliotheksstand (eingefroren)

| Bibliothek | Version | Einbindung |
|---|---|---|
| jsPDF | 2.5.1 | eingebettet – offline |
| jsPDF-AutoTable | 3.8.2 | eingebettet – offline |
| SheetJS (xlsx) | 0.18.5 | eingebettet – offline |
| pdf.js | 3.11.174 | CDN cdnjs – nur PDF-Import |
| mammoth | 1.6.0 | CDN jsDelivr – nur Word-Import |

**GitHub-Actions-Stand:** `checkout@v4`, `configure-pages@v4`,
`upload-pages-artifact@v3`, `deploy-pages@v4`

---

## 3. Funktionsumfang zum Baseline-Zeitpunkt

### Wochenplan
- VM/NM-Einteilung mit Drag-and-Drop-Zuweisung
- Abwesenheitscodes, eCase-Tracking
- Automatische Sperrung von Schultagen und Sportterminen
- Fixpräferenzen für wiederkehrende Werkstatt-Zuweisungen

### Personen
- Teilnehmerverwaltung und Stammdaten
- Schulfeld auch bei Praktikum + Lehre (`prklehr`) sichtbar *(Commit `6c7ec92`)*

### Wirkungsberichte (WB-TN)
- Übersicht und Erinnerungsfunktion
- Fortlaufender Runden-Zyklus ab Eintrittsdatum

### Monat & Rapport
- Monatsübersicht als Aggregation der Wochenpläne
- Monatsrapport im BASISJOB-Format, mit Archivierung
- VM/NM zählen je 4 Stunden Arbeitszeit für **alle** Einteilungscodes *(Commit `a7b72bd`)*
- Bildschirm-Monatsrapport: Tag-Spalte im Format «01. Mi», zentrierte Kopfzeile *(Commit `07e807e`)*
- Teilnehmer-Rapport zählt eingesteuerte (auch zukünftige) Planung; Datumsgrenzen lokal *(Commit `831aab1`)*
- Teilnehmer-Rapport zählt eingesteuerte Planung auch nach Ende-ZV/Austritt *(Commit `bc3aa33`)*
- Praktikum/Lehre: Schultag zählt weiter (4 Tage PA + 1× Schule) *(Commit `cf97820`)*

### PDF-Ausgaben (alle offline)
Wochenplan, Teilnehmer-PDF, Monatsrapport, Sammel-PDF aller Rapporte,
Neophyten-PDF – einheitliche Kopf-/Fußzeilen über `pdfHead` / `pdfFoot`.

### Import / Export
- Excel-Import (offline, SheetJS)
- PDF-Import (pdf.js, benötigt Internet)
- Word-Import (mammoth, benötigt Internet)
- JSON-Datensicherung: Export und Import mit Vorschau und Bestandsabgleich

---

## 4. Designstand

| Aspekt | Baseline-Wert |
|---|---|
| Akzentfarbe | `--accent: #ffed00` |
| Markenfarbe | `--navy: #141414` (dark `#000000`, light `#333333`) |
| Graustufen | `--g0` #ffffff … `--g9` #141414 |
| Statusfarben | grün `#059669`, rot `#c0392b`, amber `#d97706`, blau `#2e6db4`, violett `#7c3aed`, teal `#0d9488` |
| Zebra Bildschirm | `--zebra: #daeef3` |
| **Zebra PDF** | **`#fff9d6`** – zuletzt geändert von `#d9edf2` *(Commit `47c8f00`, 2026-07-16)* |
| Eckradius | `--r: 8px` |
| Breakpoint | `@media(max-width:640px)` |
| Animation | `@keyframes toast-in`, 7 `transition`-Regeln |
| Icons | 3 `data:image`-URIs (Favicon SVG, Apple-Touch-Icon PNG) |
| Schriften | Systemschriften – keine Webfonts |

---

## 5. Datenhaltung (unverändert seit Einführung von v3)

| Merkmal | Wert |
|---|---|
| Speicher | `localStorage` des Browsers |
| Datenschlüssel | `lw_db_v3` |
| Zeitstempel-Schlüssel | `lw_lastsave` |
| Datenbank | **keine** |
| Server / Backend | **keiner** |

---

## 6. Sicherheitsstand

Content-Security-Policy zum Baseline-Zeitpunkt (**darf nicht aufgeweicht werden**):

```
default-src 'none';
script-src 'unsafe-inline' https://cdnjs.cloudflare.com https://cdn.jsdelivr.net;
style-src 'unsafe-inline';
img-src data: blob:;
worker-src blob: https://cdnjs.cloudflare.com;
connect-src https://cdnjs.cloudflare.com https://cdn.jsdelivr.net;
base-uri 'none';
form-action 'none'
```

- XSS-Härtung: alle in HTML gerenderten Nutzerdaten laufen über `esc()`
- Kein Tracking, keine Telemetrie, kein Cloud-Sync, kein Backend
- **Datenschutzprüfung der Baseline:** In allen 400 Commits der Historie wurde
  **nie** eine `*.json`, `*.csv` oder `*.xlsx` committet – das Repository ist
  frei von Teilnehmerdaten. ✅

---

## 7. Letzte Änderungen vor der Baseline

| Commit | Datum | Änderung |
|---|---|---|
| `48dcb40` | 2026-07-16 | Merge PR #164 |
| `47c8f00` | 2026-07-16 | PDF-Zebrafarbe von `#d9edf2` auf `#fff9d6` umgestellt |
| `b2b9de0` | 2026-07-14 | Merge PR #163 |
| `6c7ec92` | 2026-07-13 | Teilnehmer-Formular: Schulfeld auch bei Praktikum+Lehre (`prklehr`) |
| `cf97820` | 2026-07-13 | Praktikum/Lehre: Schultag zählt weiter (4 Tage PA + 1× Schule) |
| `bc3aa33` | 2026-07-13 | Teilnehmer-Rapport: Planung auch nach Ende-ZV/Austritt zählen |
| `831aab1` | 2026-07-13 | Teilnehmer-Rapport: zukünftige Planung zählen, Datumsgrenzen lokal |
| `07e807e` | 2026-07-12 | Monatsrapport: Tag-Spalte «01. Mi», zentrierte Kopfzeile |
| `a7b72bd` | 2026-07-12 | Monatsrapport: VM/NM je 4h für alle Einteilungscodes |

---

## 8. Änderungen NACH der Baseline

> Ab hier fortlaufend eintragen. Empfohlenes Format:
>
> ```
> ## JJJJ-MM-TT – Kurztitel
> - **Commit:** <hash>
> - **Was:** <Beschreibung der Änderung>
> - **Warum:** <Anlass>
> - **Betroffen:** <Bereich/View/Funktion>
> - **Datenschutz geprüft:** offline ✅ · CSP unverändert ✅ · keine TN-Daten ✅
> ```

*(Noch keine Einträge – die Baseline ist der aktuelle Stand.)*

---

## 9. Wiederherstellung auf exakt diese Baseline

**Aus dem ZIP-Paket:**
`index.html` entpacken, im Browser öffnen. Prüfsumme gegen Abschnitt 1 abgleichen.

**Aus dem mitgelieferten Git-Bundle:**
```bash
git clone _git-repo/Wochenplanung-repo.bundle Wochenplanung
cd Wochenplanung
git checkout 48dcb40
```

Details in [`RESTORE.md`](RESTORE.md).
