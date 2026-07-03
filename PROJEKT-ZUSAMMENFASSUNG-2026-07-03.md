# Wochenplanung – Projekt-Zusammenfassung & Datensicherung

**Datum:** 2026-07-03
**Repository:** https://github.com/lioal17/Wochenplanung
**Arbeits-Branch:** `claude/daily-planning-sports-priority-2nbsrc`
**Stand (HEAD, `main`):** `e71d4e3` – „Merge pull request #106"

---

## 1. Was ist das Projekt?

Eine **Wochenplanungs- und Rapport-App für ein Motivationssemester** (Kanton Thurgau),
umgesetzt als **einzelne `index.html`** (Vanilla JavaScript, kein Build, kein Server).
Alle Daten werden lokal im Browser gespeichert (`localStorage`). Eingebettete Libraries:
jsPDF (PDF-Erzeugung), SheetJS/xlsx, pdf.js, mammoth (Word-Import).

Vollständige technische Doku: `PROJEKT.md`. Übergabe-/Notfall-Doku: `UEBERGABE.md`.
Vorheriger Stands-Bericht: `PROJEKT-ZUSAMMENFASSUNG-2026-06-19.md`.

---

## 2. Heute umgesetzte Änderung: Punkt 3 – Priorität der Tagesplanung vor Sportterminen

### Gemeldetes Problem
Teilnehmer mit fixem Sporttermin (Teilnehmer → Sporttermine → NM-Slot, max. 2×/Woche)
zeigten im **Wochenplan-PDF** weiterhin „Sport" für einen Nachmittag, obwohl für diesen
Tag bereits manuell etwas anderes eingeteilt war (z. B. Werkstatt-Code „Reinigung").
Reproduzierbar bei zwei Teilnehmern.

### Ursache
`getBlocks()` liefert für einen Sporttermin nur den *möglichen* Block (`nmB=true`,
`nmL='Sport'`) – unabhängig davon, ob für den Tag bereits manuell etwas anderes
geplant wurde. Die Zusammenführung mit der Tagesplanung (`plan.vm`/`plan.nm`) ist an
mehreren Stellen im Code separat implementiert. An drei Stellen (Teilnehmer-Rapport,
TN-Statistik, Monatsübersicht) war die Priorität bereits korrekt gelöst
(„manueller Eintrag übersteuert den Block"). Im **Wochenplan-PDF** (`genWeekPDF`,
Seite 1 „Wochenübersicht" und Seite 2 „Tagesübersicht") fehlte diese Übersteuerung –
dort gewann immer der Block, unabhängig von der Tagesplanung.

### Fix (`index.html`, Funktion `genWeekPDF`)
- **Seite 1 (Wochenübersicht):** Zell-Berechnung so geändert, dass ein manueller
  Plan-Eintrag (`plan.vm`/`plan.nm`) immer den geplanten Block (Sport/Schule) übersteuert.
- **Seite 2 (Tagesübersicht):**
  - Neue Hilfsfunktion `slotEff(plan, block)` (lokal in `genWeekPDF`), die pro Halbtag
    den effektiven Wert nach der Priorität **Absenz (Halbtag) > manueller Plan-Eintrag >
    geplanter Block** berechnet – analog zur bereits korrekten Logik im Tagesplan-Bildschirm.
  - Der Teilnehmer-Filter für die Tagesliste nutzte bisher den *rohen* Block-Status
    (`!bl.nmB`), wodurch Teilnehmer mit überschriebenem Sporttermin teils gar nicht erst
    auf der Liste erschienen. Der Filter arbeitet jetzt mit dem effektiven Wert.
  - Halbtägige Absenzen (z. B. „KR-NM") wurden bisher gar nicht berücksichtigt und
    hätten „Sport" weiterhin angezeigt – jetzt hat die Absenz für den betroffenen
    Halbtag ebenfalls Vorrang.
- **`PROJEKT.md` (Abschnitt 5.3)** korrigiert: die dort dokumentierte Priorität
  („Absenz > Block > Werkstatt-Einteilung") war widersprüchlich zur tatsächlichen
  (und jetzt überall konsistenten) Regel. Neu dokumentiert:
  **Absenz (Halbtag) > manueller Tages-Eintrag > geplanter Block > Lernwerkstatt (Default).**

### Nicht verändert (bereits korrekt, bewusst nicht angefasst)
- `getBlocks()` selbst – liefert weiterhin nur den rohen Block, das ist korrekt so.
- Teilnehmer-Rapport (`getRptRows`), TN-Statistik (`countSlot`), Monatsübersicht
  (`renderMonthly`/`moCell`) – hatten die Priorität bereits richtig umgesetzt.
- Tagesplan-Bildschirm (`renderDayPlan`) – war die Referenz-Implementierung für den Fix.

---

## 3. Commits / PR (heute)

| SHA | Beschreibung |
|-----|--------------|
| `3250570` | Fix: Tagesplanung übersteuert Sporttermin im Wochenplan-PDF |
| `e71d4e3` | Merge pull request #106 (in `main`) |

**PR:** [#106 – „Fix: Tagesplanung hat Priorität vor Sporttermin im Wochenplan-PDF"](https://github.com/lioal17/Wochenplanung/pull/106) – **gemergt**.

**Umfang:** `index.html` (Logik-Fix) + `PROJEKT.md` (Doku-Korrektur, siehe unten) + diese Zusammenfassung.

> Hinweis: `PROJEKT.md` wurde **nach** dem Merge von PR #106 lokal korrigiert und ist
> noch nicht gepusht – siehe Abschnitt 5 „Nächste Schritte".

---

## 4. Qualitäts-Check (heute durchgeführt)

- ✅ Alle 4 Inline-Script-Blöcke in `index.html` syntaktisch fehlerfrei geparst (Node `Function`-Konstruktor-Check).
- ✅ Diff auf die zwei betroffenen Stellen in `genWeekPDF` begrenzt (keine Seiteneffekte auf Rapport/Statistik/Monatsübersicht).
- ✅ Styling-Regeln der PDF-Tabellen (`didParseCell`, fette/farbige Codes wie „Sport", „PA", Schul-Codes) gegen die neuen effektiven Werte geprüft – funktionieren unverändert korrekt.
- ⚠️ **Kein Browser-Test durchgeführt** (Umgebung ohne UI) – siehe Testplan im PR und Empfehlung unten.

---

## 5. Nächste Schritte / offene Punkte

1. **Manueller Test empfohlen** (im Browser, mit den zwei ursprünglich betroffenen
   Teilnehmern): Sporttermin am gleichen Wochentag hinterlegen, Nachmittag manuell auf
   „Reinigung" setzen, Wochenplan-PDF erzeugen → Seite 1 + 2 müssen „Reinigung" statt
   „Sport" zeigen; ohne manuelle Einteilung muss weiterhin „Sport" erscheinen.
2. **`PROJEKT.md`-Korrektur pushen:** Die Doku-Korrektur in Abschnitt 5.3 liegt aktuell
   nur lokal vor und muss noch committet/gepusht werden (Branch wurde nach dem Merge
   von PR #106 neu von `main` gestartet, siehe unten).
3. Bekannter, **nicht** behobener Randfall (bewusst ausserhalb des Scopes belassen):
   Auf Seite 1 des Wochenplan-PDFs wird bei einer Absenz weiterhin **derselbe** Code in
   VM **und** NM eingetragen, auch wenn die Absenz nur einen Halbtag betrifft
   (z. B. `KR-VM`). Das ist ein bereits vor diesem Fix bestehendes, separates
   Detail-Verhalten – bei Bedarf als eigener Punkt nachreichen.
4. Aus der Juni-Zusammenfassung weiterhin unverändert: `UA-VM`/`UA-NM` (halbtägige
   unbezahlte Absenz) zählen im Monatsrapport aktuell als *bezahlt* (nur ganztägiges
   `UA` ist unbezahlt) – bewusst unverändert gelassen.

---

## 6. Wie sichere ich die echten Teilnehmer-Daten?

Der **Code** ist über Git/GitHub gesichert (dieser Branch + PR #106, gemergt in `main`).
Die **eingegebenen Teilnehmerdaten** liegen im Browser (`localStorage`), nicht im Code.
Zur Sicherung:

- In der App die **Speichern-/Backup-Funktion** nutzen (lädt die Daten als **JSON-Datei**
  herunter, `JSON.stringify(DB)`). Diese JSON-Datei separat aufbewahren.
- Wiederherstellen über die **Restore-Backup-Funktion** in der App.
- Die App hält zusätzlich einen automatischen Snapshot (`lw_db_backup` im localStorage).

> Wichtig: `localStorage` ist an Browser/Gerät gebunden. Für eine echte Sicherung die
> JSON-Datei exportieren und ausserhalb des Browsers ablegen. Diese Datei enthält
> Personendaten und gehört **nicht** ins GitHub-Repository.

---

## 7. Links

- **Repository:** https://github.com/lioal17/Wochenplanung
- **PR #106 (gemergt):** https://github.com/lioal17/Wochenplanung/pull/106
- **Branch:** `claude/daily-planning-sports-priority-2nbsrc`
