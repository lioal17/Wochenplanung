# Lernwerkstatt – Einteilungsplan

Wochenplanungs-Tool für das BASISJOB Motivationssemester.

**Live:** https://lioal17.github.io/Wochenplanung/

---

## Projektbeschreibung

Die Anwendung unterstützt die wöchentliche Einteilung von Teilnehmenden auf
Werkstätten und Einsätze. Sie erzeugt daraus Monatsübersichten, Monatsrapporte
im BASISJOB-Format und Wirkungsberichte – alles als PDF.

Das Projekt ist bewusst eine **reine Offline-/Local-First-Webapp**: kein Server,
kein Backend, keine Datenbank, kein Tracking. Alle Daten bleiben ausschließlich
lokal im Browser. Datenschutz hat in diesem Projekt Vorrang vor jedem Feature
(siehe [`SECURITY.md`](SECURITY.md) und [`CLAUDE.md`](CLAUDE.md)).

---

## Aufbau des Projekts

Die **gesamte Anwendung besteht aus einer einzigen Datei**: `index.html`
(ca. 1,5 MB, 4'718 Zeilen). Sie enthält HTML-Struktur, das komplette CSS, die
gesamte JavaScript-Logik sowie die eingebetteten Bibliotheken für PDF-Erzeugung
und Excel-Import.

Das ist Absicht: kein Build, keine Installation, keine Paketverwaltung – die
Quelldatei ist zugleich das Auslieferungsartefakt und lässt sich auch in vielen
Jahren noch mit einem beliebigen Browser öffnen.

### Ordnerstruktur

```
Wochenplanung/
├── index.html                    ★ Die komplette Anwendung
├── README.md                       Diese Datei
├── RESTORE.md                      Vollständige Wiederherstellungsanleitung
├── PROJECT_STRUCTURE.md            Struktur: Zweck/Funktion/Abhängigkeiten
├── CHANGELOG_BASELINE.md           Referenzstand als Ausgangspunkt
├── CLAUDE.md                       Verbindliche Projektregeln (Datenschutz)
├── SECURITY.md                     Sicherheits- und Datenschutzkonzept
├── PROJEKT.md                      Ausführliche Fachdokumentation
├── UEBERGABE.md                    Übergabedokumentation
├── .gitignore                      Schutz vor versehentlichen Daten-Commits
├── .nojekyll                       GitHub-Pages-Schalter
├── .github/workflows/
│   └── deploy-pages.yml            Automatisches Pages-Deployment
├── RECOVERY-2026-07-10/            Analyse- und Recovery-Doku (8 Dateien)
└── docs/archiv/                    Historische Berichte (3 Dateien)
```

Es gibt bewusst **keine** Ordner `src/`, `assets/`, `css/`, `js/`, `img/`,
`fonts/` oder `node_modules/` – alles steckt in `index.html`.

---

## Verwendete Technologien

| Bereich | Technologie |
|---|---|
| **Struktur** | HTML5 (Single-File) |
| **Design** | Reines CSS mit 32 Custom Properties, Grid & Flexbox – **kein** Sass/Tailwind |
| **Logik** | Vanilla JavaScript (ES6) – **kein** Framework, **kein** TypeScript |
| **Speicherung** | `localStorage` (Schlüssel `lw_db_v3`) – keine Datenbank |
| **PDF** | jsPDF 2.5.1 + jsPDF-AutoTable 3.8.2 *(eingebettet, offline)* |
| **Excel-Import** | SheetJS 0.18.5 *(eingebettet, offline)* |
| **PDF-Import** | pdf.js 3.11.174 *(CDN – nur hierfür Internet nötig)* |
| **Word-Import** | mammoth 1.6.0 *(CDN – nur hierfür Internet nötig)* |
| **Deployment** | GitHub Actions → GitHub Pages |

Ohne Internet läuft die App vollständig weiter – es entfällt lediglich der
Import aus PDF- und Word-Dateien.

---

## Start der Anwendung

### Lokal (Normalfall)
`index.html` im Browser öffnen – Doppelklick genügt. Kein Server nötig.

### Über GitHub Pages
https://lioal17.github.io/Wochenplanung/

### Optional über lokalen Webserver
```bash
python3 -m http.server 8080     # → http://localhost:8080/index.html
```
> Achtung: `file://` und `http://localhost` nutzen **getrennte** localStorage-
> Bereiche. Nach einem Wechsel wirkt die App leer – dann die `.json`-Sicherung
> erneut importieren.

**Browser:** Chrome/Edge 80+, Firefox 78+, Safari 14+. Internet Explorer wird
nicht unterstützt. Für Drag-and-Drop ist ein Desktop-Browser am komfortabelsten.

---

## Die wichtigsten Komponenten

Die Anwendung gliedert sich in sechs Ansichten (Umschaltung über `show('<view>')`):

| Ansicht | ID | Aufgabe |
|---|---|---|
| **Wochenplan** | `view-plan` | Kernansicht: VM/NM-Einteilung per Drag-and-Drop, Abwesenheitscodes, eCase-Tracking, automatische Sperrung von Schul-/Sportterminen |
| **Personen** | `view-person` | Teilnehmerverwaltung, Stammdaten, Fixpräferenzen für wiederkehrende Werkstatt-Zuweisungen |
| **WB-TN** | `view-wbtn` | Wirkungsberichte: Übersicht, Erinnerungen, fortlaufender Runden-Zyklus ab Eintrittsdatum |
| **Monat** | `view-monthly` | Monatsübersicht als Aggregation der Wochenpläne |
| **Rapport** | `view-rapport` | Monatsrapport im BASISJOB-Format inkl. Archivierung |
| **Import** | `view-import` | Datenübernahme aus Excel, PDF, Word sowie JSON-Datensicherung |

### Zentrale Funktionsgruppen

- **PDF-Erzeugung** (offline): `genWeekPDF` (Wochenplan), `genRapportPDF`
  (Monatsrapport), `genAllRapportsPDF` (Sammel-PDF), `genTNPDF`
  (teilnehmerbezogen), `genNeophytPDF` (Neueintritte); einheitliche Kopf-/
  Fußzeilen über `pdfHead` / `pdfFoot`.
- **Import**: `doImportFSA` (Excel, offline), `_impParsePDF` (PDF, CDN),
  `doImportDoc` (Word, CDN), `processImportJSON` + `_jsonImportDiff`
  (Datensicherung mit Bestandsabgleich) und die Vorschau
  `openImportPreview` / `confirmImportPreview`.
- **Rapport**: `renderRapport`, `openRapportModal`, `isRapportArchived`.
- **Sicherheit**: `esc()` – Pflicht-Escaping für alle in HTML gerenderten
  Nutzerdaten (XSS-Schutz).

Eine vollständige Aufschlüsselung steht in
[`PROJECT_STRUCTURE.md`](PROJECT_STRUCTURE.md).

---

## Features

- Wochenplan mit VM/NM-Einteilung, Abwesenheitscodes, eCase-Tracking
- Automatische Sperrung von Schultagen und Sportterminen
- Fixpräferenzen für wiederkehrende Werkstatt-Zuweisung
- Wirkungsberichte (Bereich „WB-TN"): Übersicht & Erinnerung an Yvi,
  fortlaufender Runden-Zyklus ab Eintrittsdatum
- Monatsübersicht und Monatsrapport als PDF (BASISJOB-Format)
- Export / Import als JSON-Datensicherung

---

## Daten

Alle Daten werden lokal im Browser (localStorage) gespeichert.
Regelmässig über den Export-Button sichern.

> **Datenschutz:** Die Daten (inkl. personenbezogener Angaben) liegen
> **unverschlüsselt** im Browser. Auf geteilten Rechnern nach der Nutzung die
> **Browserdaten löschen** (Browser-Einstellungen → Websitedaten). Exportierte
> `.json`-Sicherungen enthalten Personendaten und dürfen **nicht** ins Repository
> gelangen (siehe `.gitignore`). Details in [`SECURITY.md`](SECURITY.md).

**Wichtig:** Die Daten hängen an der Kombination Browser + Gerät + Benutzerprofil.
Ein anderer Browser oder PC zeigt einen leeren Stand; das Löschen der Browserdaten
löscht auch die Planung. Deshalb regelmäßig exportieren.

---

## Wiederherstellung

Wie das Projekt vollständig wiederhergestellt und GitHub Pages neu eingerichtet
wird, beschreibt [`RESTORE.md`](RESTORE.md). Der eingefrorene Referenzzustand ist
in [`CHANGELOG_BASELINE.md`](CHANGELOG_BASELINE.md) dokumentiert.

---

## Entwicklung

- Alles in **einer** `index.html` (HTML + CSS + JS inline), bewusst ohne Build.
- Nutzerdaten, die in HTML gerendert werden, **immer** über `esc()`.
- Die Content-Security-Policy **nicht** aufweichen.
- Änderungen über Branch + Pull Request nach `main`, kein Direkt-Push.
- Verbindliche Projektregeln: [`CLAUDE.md`](CLAUDE.md)
