# Wochenplanung – Projekt-Zusammenfassung & Datensicherung

**Datum:** 2026-06-29
**Repository:** https://github.com/lioal17/Wochenplanung
**Arbeits-Branch:** `claude/neophytes-annual-billing-pdf-4hf075`
**Stand (HEAD):** `f8b73c5`

---

## 1. Was ist das Projekt?

Eine **Wochenplanungs- und Rapport-App für ein Motivationssemester** (Kanton Thurgau),
umgesetzt als **einzelne `index.html`** (Vanilla JavaScript, kein Build, kein Server).
Alle Daten werden lokal im Browser gespeichert (`localStorage`). Eingebettete Libraries:
jsPDF (PDF-Erzeugung), SheetJS/xlsx, pdf.js, mammoth (Word-Import) – alle **offline-fähig
lokal eingebettet**.

**Kernfunktionen:**
- Teilnehmer (TN) verwalten (Status, Jobcoach, Einsatzdaten, Schultage, Kurse, Sport, Archiv).
- Wochen-/Tagesplanung mit Halbtags-Slots (VM/NM): Schule, Werkstatt, Absenzen.
- Monatsübersicht (Raster) und Monatsrapport (Stunden).
- Teilnehmer-Rapport als PDF (Abwesenheiten, Schule, Lernwerkstatt, Schnupperlehre).
- Wochenplan-PDF inkl. Monatsübersicht (Querformat) und Legende.
- TN-Statistik-/Kennzahlen-PDF.
- **Neophyten-Auswertung** als Jahres-PDF.

---

## 2. Heute umgesetzte Änderung (Überblick)

Alle Änderungen betreffen ausschliesslich `index.html`.

### Neophyten-Auswertung: Ganzjahres-PDF statt Sommer/Herbst

**Hintergrund:** Neophyten werden **1× jährlich** abgerechnet. Bisher erzeugte die App
zwei getrennte PDFs (Sommer = Januar–August, Herbst = September–Dezember), was nicht zum
jährlichen Abrechnungsrhythmus passt.

**Neu:**
- **Ein Ganzjahres-PDF** (Januar–Dezember) statt getrennter Sommer-/Herbst-Auswertungen –
  „immer alles, was zählbar war".
- Das Auswahl-Modal zeigt nur noch **einen** Button „🌱 Jahres-PDF" (vorher zwei).
- **Monats-Zwischensummen** (Jan–Dez) bleiben im PDF erhalten, darunter ein Jahres-Total.
- Dateiname jetzt `Neophyten_<Jahr>.pdf` (vorher `Neophyten_Sommer_<Jahr>.pdf` /
  `Neophyten_Herbst_<Jahr>.pdf`).

**Rückwirkende Funktion (bereits vorhanden, bleibt erhalten):**
- Alle Pläne aller Jahre bleiben dauerhaft in `localStorage` – es gibt kein Löschen
  alter Jahre. Das Jahr-Dropdown sammelt jedes in `DB.plans` gespeicherte Jahr.
- Damit lässt sich z. B. in 2027 das Jahr **2026** auswählen und korrekt auswerten.
  Der Hinweistext im Modal weist aktiv auf die rückwirkende Auswahl hin.

**Technisch:**
- `neoDaysForPeriod(year, period)` → `neoDaysForYear(year)`: Zeitraumfilter (`inP`) entfernt;
  sammelt alle zählbaren Tage des Jahres (sortiert nach Datum, Struktur `{d, cnt}`).
- `genNeophytPDF(period)` → `genNeophytPDF()`: `PERIODS`-Konfiguration entfernt; Titel
  „Neophyten-Einsatz", Untertitel „Ganzes Jahr · Januar–Dezember {Jahr}", Fuss-Total
  „Total Jahr {Jahr}".
- Zähllogik `neoCountForDay(p, d)` (max. 1 Einteilung pro TN/Tag) **unverändert** –
  bleibt die einzige Quelle der Zählung.

---

## 3. Commits (heute)

| SHA | Beschreibung |
|-----|--------------|
| `f8b73c5` | Neophyten: Ganzjahres-PDF statt Sommer/Herbst |

**Umfang gesamt:** `index.html` (Logik + Modal) + diese Doku-Datei.

---

## 4. Links

- **Repository:** https://github.com/lioal17/Wochenplanung
- **PR #103 (offen):** https://github.com/lioal17/Wochenplanung/pull/103
  → Neophyten: Ganzjahres-PDF statt Sommer/Herbst, Rückwirkung über Jahr-Auswahl
- **Branch:** `claude/neophytes-annual-billing-pdf-4hf075`

> Hinweis: PR #103 ist noch **offen** – nach dem Merge ist die Änderung in `main`.

---

## 5. Qualitäts-Check (heute durchgeführt)

- ✅ JS-Syntax des Neophyt-Blocks fehlerfrei (Function-Konstruktor-Prüfung).
- ✅ Keine Referenzen mehr auf `'sommer'`/`'herbst'`/`PERIODS`/`neoDaysForPeriod`.
- ✅ Monats-Zwischensummen-Logik (Gruppierung nach `d.getMonth()`) deckt jetzt Jan–Dez ab.
- ✅ Modal, Nav-Tooltip und Funktionssignaturen konsistent (ein Jahres-Button).
- ✅ Rückwirkung verifiziert: Jahr-Dropdown listet jedes in `DB.plans` vorhandene Jahr.

---

## 6. Wie sichere ich die echten Teilnehmer-Daten?

Der **Code** ist über Git/GitHub gesichert (dieser Branch + PR). Die **eingegebenen
Teilnehmerdaten** liegen im Browser (`localStorage`), nicht im Code. Zur Sicherung:

- In der App die **Speichern-/Backup-Funktion** nutzen (lädt die Daten als **JSON-Datei**
  herunter, `JSON.stringify(DB)`). Diese JSON-Datei separat aufbewahren.
- Die App speichert in einen festen Ordner mit zeitstempel-basierten Sicherungen.
- `localStorage` ist an Browser/Gerät gebunden – für eine echte Sicherung die JSON-Datei
  exportieren und ausserhalb des Browsers ablegen.

> Wichtig: Gerade weil die Neophyten-Auswertung jetzt **rückwirkend** über vergangene Jahre
> läuft, sollten die Teilnehmerdaten **nicht gelöscht** und regelmässig als JSON gesichert
> werden – nur so bleiben alte Jahre (z. B. 2026) auch 2027 noch auswertbar.

---

## 7. Offene Punkte / Rückfragen

1. **PR #103** ist noch offen – mergen, damit die Änderung in `main` ist.
2. Auf Wunsch: PR #103 beobachten (CI/Review-Kommentare automatisch nachziehen).
