# Wochenplanung – Projekt-Zusammenfassung & Datensicherung

**Datum:** 2026-06-19
**Repository:** https://github.com/lioal17/Wochenplanung
**Arbeits-Branch:** `claude/amazing-archimedes-4d0agw`
**Stand (HEAD):** `c233bc5`

---

## 1. Was ist das Projekt?

Eine **Wochenplanungs- und Rapport-App für ein Motivationssemester** (Kanton Thurgau),
umgesetzt als **einzelne `index.html`** (Vanilla JavaScript, kein Build, kein Server).
Alle Daten werden lokal im Browser gespeichert (`localStorage`). Eingebettete Libraries:
jsPDF (PDF-Erzeugung), SheetJS/xlsx, pdf.js, mammoth (Word-Import).

**Kernfunktionen:**
- Teilnehmer (TN) verwalten (Status, Jobcoach, Einsatzdaten, Schultage, Kurse, Sport).
- Wochen-/Tagesplanung mit Halbtags-Slots (VM/NM): Schule, Werkstatt, Absenzen.
- Monatsübersicht (Raster) und Monatsrapport (Stunden).
- Teilnehmer-Rapport als PDF (Abwesenheiten, Schule, Lernwerkstatt, Schnupperlehre).

---

## 2. Heute umgesetzte Änderungen (Überblick)

Alle Änderungen betreffen ausschliesslich `index.html`.

### A. Schulferien automatisch berücksichtigen
- **Regelbasierte Berechnung** der TG-Schulferien (`computeFerien` / `isSchulferien`) –
  Sport-, Frühlings-, Pfingst-, Sommer-, Herbst- und Weihnachtsferien.
- **Zukunftssicher**: kein jährliches Einpflegen, **keine feste Tabelle, keine Zusatzdaten**
  (validiert gegen Schuljahre 2024–2032).
- In `getBlocks()` werden Schul-/Sport-Blöcke während der Ferien automatisch unterdrückt →
  wirkt zentral auf Rapport, Monatsübersicht, Raster und Export. In den Ferien zählt
  Schule nie – der TN ist in der Lernwerkstatt.

### B. Effektiv gültiger Code je Halbtag (Priorität)
1. **Manuelle Absenz** (z. B. KR-NM) – zählt nie als Schule/Werkstatt.
2. **Offizielle Schulferien** – geplanter Schul-/Sport-Code wird ignoriert → Lernwerkstatt.
3. **Slot-Eintrag** (`plan.vm/nm`, Werkstatt **oder** Schul-Code) übersteuert den geplanten Block.
4. **Geplante Schule / Sport.**
5. sonst **Lernwerkstatt** (Default).
- **Schule halbtagsgenau** (je Halbtag 4 Lektionen): `S`=8, `S+KR-NM`=4, `S+KR-VM`=4,
  `S+Ganztags-KR`=0.

### C. Tagesplan: geplante Schul-/Sport-Halbtage editierbar
- Geplante Schul- (S/DAZ/FöA/DK/IKPC) und Sport-Halbtage sind im Tagesplan als Dropdown
  editierbar (vorbelegt mit dem geplanten Code) und können auf Werkstatt geändert werden.
- **Praktikum/Schnupperlehre (PA/PA+LS/SL) bleiben gesperrt.**
- Schul-Codes sind **zuunterst** in der VM/NM-Einteilungsliste wählbar.

### D. Absenzen im Rapport vervollständigt
- Bisher zählte der Rapport nur UA/BA/KR/ZS. Neu auch **UN (Unfall), MI (Militär/Zivilschutz),
  FE/FE-W (Ferien)** – vorher wurden ganze Unfall-Wochen nicht gezählt.
- **Unfall als volle Code-Familie** (analog Krankheit): `UN-VM` (4h), `UN-NM` (4h),
  `UN` (8h), `UN-W` (ganze Woche, Mo–Fr, nur am Montag setzbar).
- **Wochencodes BA-W und UA-W** ergänzt – `BA-W` (Bezahlte Absenz ganze Woche) und
  `UA-W` (Unentschuldigte Absenz ganze Woche). Damit gibt es alle Wochencodes:
  BA-W, FE-W, KR-W, UA-W, UN-W (alle nur am Montag setzbar, füllen Mo–Fr).
- Absenzen im Teilnehmer-Rapport **alphabetisch**: BA, FE, KR, MI, UA, UN, ZS.
- **UN/BA(-W) gelten als bezahlte Absenz** (Monatsrapport); nur `UA`/`UA-W` sind unbezahlt.

### E. Layout
- Teilnehmer-Rapport **ohne Jobcoach-Kürzel** – nur noch der Name des TN.

### F. Review-Fixes (Qualität)
- Behoben: Ein ganzer, beidseitig auf Werkstatt übersteuerter Schultag zeigte im
  Monatsrapport 0 statt 8 Std.
- Behoben: Übersteuerter Schul-/Sport-Halbtag wird im Monatsraster jetzt korrekt als
  Werkstatt angezeigt (vorher weiter als Schul-Code).

---

## 3. Commits (heute)

| SHA | Beschreibung |
|-----|--------------|
| `0214c76` | Schulferien, Halbtags-Logik & Slot-Übersteuerung korrigieren |
| `113190e` | Tagesplan: geplante Schul-/Sport-Halbtage übersteuerbar machen |
| `081c785` | Rapport: Unfall/Militär/Ferien als Absenzen zählen |
| `afbfd19` | Absenzen: UN-Familie (UN-VM/UN-NM/UN/UN-W) wie KR umsetzen |
| `ff4ff7d` | Rapport: Absenzen alphabetisch, UN bezahlt, JC-Zeile entfernen |
| `935223e` | Review-Fixes: 0h-Bug + Raster-Anzeige |
| `3beedcd` | Doku: Projekt-Zusammenfassung & Datensicherung |
| `c233bc5` | Absenzen: Wochencodes BA-W und UA-W ergänzen |

**Umfang gesamt:** im Wesentlichen `index.html` (Logik) + diese Doku-Datei.

---

## 4. Links

- **Repository:** https://github.com/lioal17/Wochenplanung
- **PR #86 (gemmerged):** https://github.com/lioal17/Wochenplanung/pull/86
  → Schulferien, Halbtags-Logik, Slot-Übersteuerung, Tagesplan-Editierbarkeit
- **PR #87 (offen):** https://github.com/lioal17/Wochenplanung/pull/87
  → UN-Familie, Absenz-Zählung (UN/MI/FE), alphabetisch, UN=bezahlt, ohne JC, Review-Fixes
- **Branch:** `claude/amazing-archimedes-4d0agw`

> Hinweis: Die Review-Fixes (`935223e`) und alle Punkte aus Abschnitt 2.D/E sind in **PR #87**.
> Dieser ist noch **offen** – nach dem Merge sind alle heutigen Änderungen in `main`.

---

## 5. Qualitäts-Check (heute durchgeführt)

- ✅ JS-Syntax aller 6 Script-Blöcke fehlerfrei.
- ✅ Keine toten/„Shadow"-Variablen (keine Reste von `s_days`, `TG_FERIEN`, `present` etc.).
- ✅ Keine Debug-Reste (`console.log`/`debugger`/`TODO`) im App-Code.
- ✅ Keine doppelten Funktionsdefinitionen; alle neuen Funktionen definiert & genutzt.
- ✅ Ferien-Berechnung gegen bekannte TG-Termine geprüft (2025–2027 plausibel).
- ✅ Script-Tag-Struktur geschlossen/balanciert (6 echte Blöcke).

---

## 6. Wie sichere ich die echten Teilnehmer-Daten?

Der **Code** ist über Git/GitHub gesichert (dieser Branch + PRs). Die **eingegebenen
Teilnehmerdaten** liegen im Browser (`localStorage`), nicht im Code. Zur Sicherung:

- In der App die **Speichern-/Backup-Funktion** nutzen (lädt die Daten als **JSON-Datei**
  herunter, `JSON.stringify(DB)`). Diese JSON-Datei separat aufbewahren.
- Wiederherstellen über die **Restore-Backup-Funktion** in der App.
- Die App hält zusätzlich einen automatischen Snapshot (`lw_db_backup` im localStorage).

> Wichtig: `localStorage` ist an Browser/Gerät gebunden. Für eine echte Sicherung die
> JSON-Datei exportieren und ausserhalb des Browsers ablegen.

---

## 7. Offene Punkte / Rückfragen

1. **PR #87** ist noch offen – mergen, damit alles in `main` ist.
2. **UA-VM / UA-NM** (halbtägige unbezahlte Absenz) werden im Monatsrapport aktuell als
   *bezahlt* gezählt (nur ganztägiges `UA` ist unbezahlt). Unverändert gelassen –
   bei Bedarf auf „unbezahlt" umstellbar.
3. Auf Wunsch: PR #87 beobachten (CI/Review-Kommentare automatisch nachziehen).
