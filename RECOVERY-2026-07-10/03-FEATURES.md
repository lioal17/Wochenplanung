# Feature-Dokumentation – Stand 10.07.2026

> Beschreibt das **aktuelle Verhalten**. Ergänzt `PROJEKT.md` (im Repo) um die
> Änderungen vom 10.07.2026. Bei Abweichung gilt für diesen Snapshot **dieses** Dokument.

---

## 1. Wochenplan (`view-plan`)

- Tages-/Wocheneinteilung **VM (08:15–12:00)** und **NM (13:15–17:00)** pro TN.
- Kopf: KW + Datumsspanne; **Tagesleiste** Mo–Fr (`.day-btn`, „today" hervorgehoben).
- Spalten: Name · JC (Jobcoach) · IIZ · BW · VM · NM · Abwesenheit · **eCase** · Schnuppern.
- **Slot-Priorität:** Absenz (je Halbtag) > manueller Tages-Eintrag > geplanter Block
  (Schule/Sport) > Lernwerkstatt (Default). Ganztags-Absenzen sperren beide Slots.
- **Werkzeuge:** „Vorwoche kopieren" (`copyPrev`), „Alle Einträge löschen" (Tag leeren,
  mit `confirm`), VM/NM-Summen-Chips (`.plan-summary-split`).
- **eCase-Spalte:** Checkbox „In eCase eintragen und abhaken" → setzt `plan.ecaseEntered`;
  erledigt = „✓" (`--green`), offen = „!".

### eCase-Badge (Wochenplan)
- Kompaktes Warn-Badge „⚠️ N eCase-Einträge offen ▾" mit aufklappbarer Liste
  (Teilnehmer · Datum · Code · ✓).
- **Global über ALLE Monate** (nicht nur die aktuelle Woche) – gemeinsame Quelle
  `collectOpenEcase()`.

---

## 2. Monatsübersicht (`view-monthly`)

- Matrix **alle TN × alle Werktage des Monats**, je Tag zwei Zellen (VM/NM).
- Zelltypen: Absenz (`moc-abs`, rot), Werkstatt (`moc-work`), Schule (`moc-sch`),
  Praktikum (`moc-prk`), Individual/„Anders" (`moc-indiv`, violett), außerhalb Einsatz (`mo-x` = „X").
- Statistikspalten je TN: **UA** (unbezahlt) und **ZS** (zu spät).
- Volle **Legende** (Schule-/Werkstatt-/Absenz-Codes).
- **eCase-Badge** „⚠️ N eCase-Einträge offen ▾": **global**, identisch zum Wochenplan
  (gleiche Quelle `collectOpenEcase()`), aufklappbare Liste mit Abhak-Checkboxen
  (`setEcFromMo`).

---

## 3. Monatsrapport (`view-rapport`)

- Auswahl **TN + Monat** → Kennzahlen als **Stat-Kacheln** (`.rpt-stat`):
  - **Bezahlte Absenz** (BA-Familie, ohne UA/KR) · **Unbezahlte Absenz** (UA/UA-W) ·
    **Krank (KR)** · **Zu spät (ZS)** — jeweils **monatsbezogen**.
  - **eCase** — Kachel zeigt die **Gesamtzahl offener eCase-Einträge dieses TN über ALLE
    Monate** (global, wie der eCase-Button), Tooltip weist darauf hin. Farbe amber (>0)
    bzw. grün „✓" (0). Quelle: `collectOpenEcase().filter(pid)`.
- **Freitext-Bemerkungen** je TN/Monat (`rptNotes`), erscheinen im PDF unter dem Namen.
- PDF-Export im BASISJOB-Format.

> **Design-Entscheidung (10.07.2026):** Der frühere interaktive „eCase-Einträge offen"-
> **Dropdown/Badge** im Rapport wurde **entfernt**; Quittieren geschieht nur noch im
> Wochenplan / in der Monatsübersicht. Der Rapport zeigt eCase **nur als Kennzahl**.

---

## 4. Teilnehmer-Verwaltung (`view-person`)

- Liste aller TN, „+ Teilnehmer hinzufügen", Bearbeiten/Löschen (mit `confirm`).
- Bearbeiten-Fenster: Stammdaten, Status, Jobcoach, Eintritt/Ende-ZV, IIZ,
  **Schultage** (fix, wochentagbasiert), **Kurse** (manuell, mit bis zu 3 Zeiträumen),
  **Sporttermine** (fixe NM-Termine).
- Archiv-Logik: TN mit `einsatzBis` in der Vergangenheit gilt als archiviert (`isArchived`).

---

## 5. Dokument-Import (`view-import`)

- Teilnehmerlisten aus **PDF / Excel / Word** einlesen (`handleDocUpload` → `parseDocText`).
- Erkennt Klassen-Codes (`FöA-Do-…`, `DMA-Mo-…`), Jobcoach, Eintritt/Ende-ZV; legt fixe
  Schultage & Sporttermine an. Import schreibt **nie** in `kurse` (Zeitraum-Logik bleibt manuell).
- PDF-Import via **pdf.js** (`isEvalSupported:false`), Word via **mammoth** (CDN, Internet nötig).

---

## 6. WB-TN – Wirkungsberichte (`view-wbtn`)

Übersicht aller **aktiven** TN (`status` ∈ `aktiv`/`lehrstelle`, `einsatzVon` gesetzt, nicht
archiviert), sortiert nach nächstfälligem Wirkungsbericht.

### 6.1 Termin-Berechnung (deterministisch ab Eintritt)
Für Runde `r = wbRounds + 1`:
- **WB Job Coach:** `WB(r) = einsatzVon + 10 Wochen + (r−1)×12 Wochen`.
- **Input Arbeitsagoge:** `InputAG(r) = WB(r) − 2 Wochen`, **auf den Montag derselben Woche
  gelegt** (Helfer `toMonday`, Montag am/vor dem Datum). → Der Vorlauf bleibt ≥ 2 Wochen.
  *(Änderung 10.07.2026: Input AG fällt jetzt immer auf Montag – passend zur Montags-Teamsitzung.)*

### 6.2 Ampel (`wbStatus`)
| Ampel | Bedeutung | Bedingung |
|---|---|---|
| 🟢 grün | kein Handlungsbedarf | vor `InputAG`, Runde nicht fällig |
| 🔴 rot | **Arbeitsagoge informieren** (offen) | `heute ≥ InputAG` und `!wbAgInformed` |
| 🟡 gelb | Job Coach informieren | `wbAgInformed` gesetzt, Runde noch offen |

### 6.3 Aktionen
- **„Arbeitsagoge informiert"** (`wbInformAG`) – Toggle 🔴↔🟡 (verändert die Runde nicht).
- **„Job Coach erledigt"** (`wbMarkDone`) – schließt die Runde ab: `wbRounds++`,
  `wbAgInformed=false`, nächste Runde startet automatisch.
- **Runden-Button „Runde X."** (`wbResetRound`) – *(neu 10.07.2026)* setzt den Zyklus dieses
  TN auf **Runde 1** zurück (`wbRounds=0`, `wbAgInformed=false`), **mit Sicherheitsabfrage**.
  Danach rechnet der Zyklus wieder ab Eintritt; liegt der Eintritt zurück, ist die Runde
  sofort überfällig → rote Ampel + Badge.

### 6.4 WB-TN-Badge (Nav-Button)
- Zahl am „📅 WB-TN"-Button = **Anzahl offener Arbeitsagoge-WBs (🔴)** =
  wie viele Wirkungsberichte die Arbeitsagoge noch ausfüllen muss.
  *(Änderung 10.07.2026: zählt jetzt **nur rote**; gelbe/grüne zählen nicht mehr mit.)*

### 6.5 Start-Erinnerung
- `wbCheckReminders` zeigt beim App-Start ein Erinnerungs-Pop-up (Yvi), sobald ein Input
  Arbeitsagoge fällig ist (Buttons „Erledigt" / „Später erinnern"). **Kein Mailversand.**

---

## 7. PDF-Exporte

| Funktion | Ausgabe |
|---|---|
| `genWeekPDF` | Wochenplan (Wochenübersicht, Tagesübersicht, Monatsübersicht quer, Legende) – löst zugleich Sicherung aus |
| Monatsrapport (`buildRpt`/`genPDF`) | BASISJOB-Format inkl. Freitext-Notizen |
| `genNeophytPDF` | „Anzahl Teilnehmende pro Tag" (KW, Datum) übers Jahr |
| `genTNPDF` | Teilnehmer-Statistik (aktive TN, IIZ, Status-Aufschlüsselung, Schnuppern) |
| Teilnehmer-Rapport (`genRapportPDF`) | Gesamtrapport je TN (Schule-Lektionen, LW, Absenzen, Schnupperlehre) bis heute |

---

## 8. Speichern, Sicherung, Import

- **Auto-Save** in `localStorage` bei jeder Änderung.
- **Manuelles Speichern** (`manualSave`) als JSON mit Zeitstempel; optional per File System
  Access API **ohne Dialog** in einen gewählten Ordner (`changeSaveLocation`, Handle in IndexedDB).
- **JSON-Import** mit **Vorschau der Änderungen** vor dem Überschreiben; `sanitizeDB()`-Validierung.

---

## 9. Änderungen dieser Arbeitssession (bis 10.07.2026) – Kurzliste

1. eCase-Badges **Wochenplan + Monatsübersicht** über gemeinsame Quelle **global** synchron
   (identische Zahl, gemeinsames Feld `ecaseEntered`).
2. **Monatsrapport:** interaktives eCase-Dropdown **entfernt**; eCase-Kachel zeigt
   **Gesamtzahl pro TN** (global, wie der Button).
3. **WB-TN:** „Input Arbeitsagoge" immer auf **Montag** (`toMonday`).
4. **WB-TN-Badge:** zählt nur **offene Arbeitsagoge-WBs** (rote Ampel).
5. **WB-TN:** „Runde X."-Button **klickbar** → Zyklus auf Runde 1 zurücksetzen (mit Abfrage).

> Alle fünf Punkte sind in `origin/main` (Commit `2fca922…`) enthalten und im Browser verifiziert.
