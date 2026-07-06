# Projektbeschreibung – Lernwerkstatt Einteilungsplan (Wochenplanung)

> **Zweck dieses Dokuments:** Vollständige fachliche und technische Beschreibung der App,
> sodass das Tool auch ohne den Originalcode rekonstruiert werden könnte. Es ergänzt
> (ersetzt aber nicht) die Datei `index.html`, welche selbst die zuverlässigste
> Sicherung ist (reiner Text/HTML, einfach im Browser öffnen).
>
> Stand: 2026-07-03 (Details zu Zwischenständen siehe die datierten
> `PROJEKT-ZUSAMMENFASSUNG-*.md`-Dateien)

---

## 1. Überblick

**Wochenplanungs-Tool für das BASISJOB Motivationssemester** der Lernwerkstatt.
Verwaltet Teilnehmende, plant deren Wochen (Vormittag/Nachmittag-Einteilung in
Werkstätten/Schule/Sport), erfasst Abwesenheiten und eCase-Status und erzeugt
diverse PDF-Auswertungen.

- **Eine einzige Datei:** `index.html` (HTML + CSS + JavaScript inline, kein Build, kein Server).
- **Start:** Datei im Browser öffnen oder via GitHub Pages.
- **Daten:** ausschließlich lokal im Browser (`localStorage`). Es gibt **keine** Personendaten
  im Code. Sicherung der Daten erfolgt manuell per JSON-Export.

### Bibliotheken
**Lokal in die Datei eingebettet (funktionieren ohne Internet):**
- **jsPDF 2.5.1** + **jsPDF-AutoTable 3.8.2** – PDF-Erzeugung (Wochenplan, Teilnehmer-Rapport, Monatsrapport, Neophyt).
- **SheetJS (xlsx) 0.18.5** – Excel-Import.

**Weiterhin per CDN (nur für Dokument-Import nötig, brauchen Internet):**
- **PDF.js 3.11.174** – PDF-Import (Teilnehmerlisten).
- **Mammoth.js 1.6.0** – Word-(docx)-Import.

> **Offline-Betrieb:** App öffnen, planen, speichern, JSON Export/Import und **alle
> PDF-Exporte** laufen ohne Internet. Nur der PDF-/Word-Import (pdf.js/mammoth) benötigt
> eine Internetverbindung. Damit ist die Datei für die Weitergabe als eigenständige
> Offline-Anwendung geeignet.

---

## 2. Datenmodell

### 2.1 Globaler State
```js
let DB = { participants:[], plans:{}, rptNotes:{} };
```
- `participants` – Array von Teilnehmer-Objekten.
- `plans` – verschachteltes Objekt: `plans["YYYY-MM-DD"][participantId] = planEintrag`.
- `rptNotes` – freie Notizen pro Teilnehmer/Monat (für den **Monatsrapport**).

### 2.2 Teilnehmer-Objekt (`participant`)
| Feld | Typ | Bedeutung |
|------|-----|-----------|
| `id` | string | Eindeutige ID (`uid()`) |
| `vorname`, `nachname` | string | Name |
| `status` | string | `aktiv` (Lernwerkstatt), `schnupper`, `praktikum`, `lehrstelle`, `prklehr` (Praktikum+Lehrstelle) |
| `jobCoach` | `lio`\|`tom`\|`yvi` | zuständiger Jobcoach |
| `bewerbenJC` | string | Jobcoach für „Bewerben" (Fallback: `jobCoach`) |
| `stammwerkstatt` | string | Stammwerkstatt |
| `einsatzVon` | `YYYY-MM-DD` | **Eintritt** (= Einsatz von) |
| `einsatzBis` | `YYYY-MM-DD` | **Ende ZV** (= Einsatz bis) |
| `iiz` | string | IIZ-Feld |
| `aktiviertAm` | `YYYY-MM-DD` | Aktivierungsdatum (für Wirkungsbericht-Fristen) |
| `bemerkung` | string | Freitext |
| `schultage` | array | `[{typ, wochentag}]` – fixe wöchentliche Schultage (unverändert) |
| `kurse` | array | `[{typ, erfasstAm, zeit}]` – manuell erfasste Kurse mit Zeiträumen (kein Wochentag) |
| `sporttermine` | array | `[wochentag,…]` – fixe Sport-NM-Termine (max. 2/Woche) |

`schultage[].typ` ∈ `S`, `DAZ`, `DAZ-VM`, `DAZ-NM`, `FöA`, `FöA-VM`, `FöA-NM` (manuell wählbar).
Einheitliches Schema: **Basis-Code = ganzer Tag**, `-VM` = Vormittag, `-NM` = Nachmittag.
Importe schreiben zusätzlich `DK-VM`/`IKPC-VM` (bzw. `-NM`) in `schultage` (½ Tag, unverändertes
Verhalten). `wochentag`: 1=Mo … 5=Fr.

**Schultage** verhalten sich wie bisher: wöchentlich wiederkehrend, ohne Datumsgrenze
(außer `einsatzVon`/`einsatzBis`), inkl. rückwirkend.

**Kurse** (`kurse`) sind ein **eigener Bereich** im Fenster „Teilnehmer bearbeiten"
(analog Sporttermine), ausschließlich **manuell** erfasst. `kurse[].typ` ∈ `DK`, `DK-VM`, `DK-NM`,
`IKPC`, `IKPC-VM`, `IKPC-NM` (Basis = ganzer Tag, `-VM`/`-NM` = halber Tag). **Kein Wochentag** –
der Kurs gilt an jedem Werktag (Mo–Fr) innerhalb seiner
aktiven Zeiträume; der Rapport zählt die DK-Lektionen entsprechend pro Werktag. Jeder Eintrag trägt:
- `erfasstAm: 'YYYY-MM-DD'` – Erfassungsdatum (= Standard-Startdatum).
- `zeit: [{von, bis}, …]` – bis zu **3** unabhängige Zeiträume (Von/Bis je `YYYY-MM-DD`).

Gültigkeit eines Kurses an einem Datum (`schultagAktiv`):
- **Kein Zeitraum gesetzt** → gilt ab `erfasstAm` unbefristet (Vergangenheit bleibt unberührt).
- **Nur „Von"** → ab diesem Datum unbefristet. **„Von"+„Bis"** → nur in diesem Zeitraum.
- Aktiv, sobald das Datum in **einen** der Zeiträume fällt; sonst ist der Slot frei planbar.

> **Importe** schreiben nie in `kurse`. Damit bleibt das gesamte Import-Verhalten
> (inkl. importiertem `DK-VM`/`IKPC-VM` in `schultage`) unverändert; die Zeitraum-Logik gilt
> ausschließlich für manuell erfasste Kurse.

### 2.3 Plan-Eintrag (`plans[date][pid]`)
| Feld | Bedeutung |
|------|-----------|
| `vm` | Werkstatt-Einteilung Vormittag (z. B. `Metall`, `Bewerben`, `Sport`, …) |
| `nm` | Werkstatt-Einteilung Nachmittag |
| `abwesenheit` | Absenz-Code (siehe unten) oder `''` |
| `abwesenheitText` | Freitext-Bemerkung zur Absenz |
| `ecaseEntered` | bool – wurde Absenz in eCase erfasst? |
| `sw` | Schnupper-Flag: `S.LW` (Schnupperwoche Lernwerkstatt) oder `S.EX` (Schnupperlehre extern, ganze Woche) |
| `bemerkung` | Freitext |

### 2.4 Persistenz (localStorage-Keys)
- `lw_db_v3` – Hauptdatenbank (`DB`). Beim Laden über `sanitizeDB()` strukturell
  validiert (Feld-Whitelist, Typkoersion, ID-Format).
- `lw_lastsave` – Zeitstempel der letzten manuellen Sicherung.
- Optionaler File-System-Access-Handle (IndexedDB `lw_fh`) für „Speichern ohne Dialog".
- Vollständiges Löschen der lokalen Daten erfolgt über die **Browser-Einstellungen**
  (Websitedaten für die Seite entfernen) – es gibt bewusst keinen In-App-Löschknopf.

---

## 3. Konstanten / Codes

### 3.1 Werkstätten (`WS`)
`Holzwerkstatt, Bewerben, Metall, Metallverpacken, Crea, Lernstück, Loli, Neophyten, Feldeinsatz, Textil, Reinigung, Lernen` (+ `Sport` nur im NM-Slot, + „Anderes…" Freitext).

**Kürzel (`WS_ABBR`):** Holzwerkstatt=H, Bewerben=BW, Metall=M, Metallverpacken=MV,
Crea=Crea, Lernstück/Lernstücke=LS, Loli=Loli, Neophyten=NEO, Feldeinsatz=FL,
Textil=Text, Reinigung=Rein, Lernen=Lernen, Sport=Sport.

### 3.2 Schul-Codes (Bereich „Schultage")
Einheitliches Schema (wie Absenz-Codes): **Basis = ganzer Tag**, `-VM` = Vormittag, `-NM` = Nachmittag.
- `S` = Schule (ganzer Tag).
- `DAZ` / `DAZ-VM` / `DAZ-NM` = Deutsch als Zweitsprache (ganzer Tag / Vormittag / Nachmittag).
- `FöA` / `FöA-VM` / `FöA-NM` = Förderatelier (ganzer Tag / Vormittag / Nachmittag).

### 3.2a Kurs-Codes (Bereich „Kurse", manuell, mit Zeiträumen)
- `DK` = Deutschkurs Migros **ganzer Tag** (blockiert VM **und** NM), **8 Lektionen**.
- `DK-VM` = ½ Tag VM, blockiert VM, **4 Lektionen**. `DK-NM` = ½ Tag NM, blockiert NM, **4 Lektionen**.
- `IKPC` / `IKPC-VM` / `IKPC-NM` = Informatik/PC (ganzer Tag / Vormittag / Nachmittag),
  keine eigene Rapport-/Lektionen-Zeile.

> Beim **Import** entstehen Schul-Codes weiterhin als Halbtag im Feld `schultage`
> (Standard `-VM`, „Nachmittag" → `-NM`; `DMA` → `S` Ganztag). Verhalten unverändert.
> Die Zeitraum-Logik gilt **nur** für den manuell befüllten Bereich „Kurse" (`kurse`).
>
> **Migration (einmalig, automatisch beim Laden):** Bisher bedeutete der Basis-Code
> `DAZ`/`FöA`/`DK`/`IKPC` „½ Tag VM". Alte Einträge werden auf `…-VM` umgeschrieben und
> behalten so exakt ihre bisherige Wirkung (`DK` in `kurse` war bereits Ganztag → bleibt).
> Neue Einträge nutzen das einheitliche Schema (Basis = ganzer Tag).

### 3.3 Absenz-Codes (`ABS`)
`BA`, `BA-NM`, `BA-VM`, `BA-W` (Bezahlte Absenz / -Nachmittag / -Vormittag / ganze Woche),
`FE`, `FE-W` (Ferien / Ferien ganze Woche),
`KR`, `KR-NM`, `KR-VM`, `KR-W` (Krank / halbtags / ganze Woche),
`MI` (Militär/Zivilschutz), `PA` (Praktikum), `SL` (Schnupperlehre),
`UA`, `UA-NM`, `UA-VM`, `UA-W` (Unbezahlte Absenz / halbtags / ganze Woche),
`UN`, `UN-NM`, `UN-VM`, `UN-W` (Unfall / halbtags / ganze Woche), `ZS` (Zu spät).

- **Kein eCase-Auftrag:** `SL` (Schnupperlehre/-tag) und `PA` (Praktikum) erzeugen
  **keinen** offenen eCase-Eintrag (weder im Tages-/Wochenplan, im Badge noch im Monatsplan).

- **Halbtags-Absenzen** (`SOFT_ABS`): ZS, UA-VM/NM, KR-VM/NM, BA-VM/NM, UN-VM/NM –
  Werkstatt-Einteilung des nicht betroffenen Slots bleibt möglich.
- **Wochencodes** BA-W / FE-W / KR-W / UA-W / UN-W: nur am **Montag** einsteuerbar, füllen
  automatisch Mo–Fr (5 Tage); Entfernen löscht die ganze Woche. Nur `UA`/`UA-W` gelten im
  Monatsrapport als **unbezahlt** – `UA-VM`/`UA-NM` (Halbtag) sowie alle übrigen Absenz-Codes
  (inkl. `UN`-Familie) zählen als **bezahlt**.

### 3.4 Jobcoaches
`lio`→Lio, `tom`→Tom, `yvi`→Yvi (jeweils eigene Farbe).

### 3.5 Feiertage
`tgHolidays(jahr)` berechnet Feiertage/Brückentage (inkl. Ostern via `easterSunday`).
An diesen Tagen ist die LW geschlossen → fallen **nicht** in Auswertungen/Rapporte.

---

## 4. Navigation / Views

Top-Navigation (`show(view)`):
1. **📅 Wochenplan** (`plan`) – Tages-/Wocheneinteilung VM/NM pro Teilnehmer.
2. **📊 Monatsübersicht** (`monthly`) – Matrix aller TN × Tage des Monats (VM/NM-Zellen).
3. **📄 Monatsrapport** (`rapport`) – pro Monat, mit Freitext-Notizen, PDF-Export.
4. **👥 Teilnehmer** (`person`) – Teilnehmer-Verwaltung (anlegen/bearbeiten/löschen).
5. **📥 Dokument-Import** (`import`) – Teilnehmerlisten aus PDF/Excel/Word einlesen.
6. **🌱 Neophyt** – direkter PDF-Export (Anzahl Teilnehmende pro Tag mit KW & Datum).
7. **👥 TN-Statistik** – direkter PDF-Export mit Teilnehmer-Kennzahlen (siehe §7).

Zusätzlich: **Teilnehmer-Rapport** (Modal, siehe §6) und diverse Speicher-/Import-Knöpfe.

---

## 5. Kernlogik

### 5.1 `getBlocks(p, date)` → `{vmB, nmB, fullB, vmL, nmL}`
Bestimmt „gesperrte" Slots (Schule/Sport/Praktikum), die nicht frei einteilbar sind:
- Status `praktikum` → ganztags `PA`; `prklehr` → ganztags `PA+LS`.
- `sw==='S.EX'` (Schnupperlehre) → ganztags `SL`.
- Einheitliche Auflösung über Helfer `applyBlk(typ)` (für Schultage **und** Kurse):
  - `typ==='S'` → ganzer Tag (`vmB=nmB=fullB=true`, Label `S`).
  - Suffix `-VM` → nur VM-Slot · Suffix `-NM` → nur NM-Slot (Label = Basis-Code).
  - **Basis-Code ohne Suffix** (z. B. `DAZ`, `DK`, `IKPC`) → VM **und** NM (ganzer Tag,
    Label = Basis-Code; `fullB` bleibt `false`, nur `S` setzt `fullB`).
- Aus `p.schultage` (gefiltert nach Wochentag, ohne Datums-/Zeitraum-Filter): `applyBlk(s.typ)`.
- Aus `p.kurse` (gefiltert nur über `schultagAktiv(k,date)` – kein Wochentag; gilt an jedem
  Werktag im Zeitraum; außerhalb gültiger Zeiträume zählt der Kurs **nicht**): `applyBlk(k.typ)`.
- Aus `p.sporttermine` → NM-Slot Label `Sport`.

### 5.2 `isActiveOnDay(p, date)`
`false`, wenn `date` vor `einsatzVon` oder nach `einsatzBis` liegt.

### 5.3 Slot-Priorität bei der Darstellung
**Absenz (je Halbtag) > manueller Tages-Eintrag (`plan.vm`/`plan.nm`, Schule- oder
Werkstatt-Code) > geplanter Block (Schule/Sport) > Lernwerkstatt (Default).**
Ganztags-Absenzen sperren beide Slots. D. h. ein fixer Sporttermin (oder Schultag) wird
angezeigt, **nur solange** für diesen Halbtag keine Absenz und keine manuelle Einteilung
vorliegt – die Tagesplanung hat immer Vorrang vor dem hinterlegten Sporttermin.

### 5.4 Weitere Helfer
- `neoCountForDay` – zählt Neophyten-Einteilung max. 1× pro TN/Tag.
- `copyPrev` – Vorwoche kopieren. `clearCurrentDay` – Tag leeren.
- `renderWBAlerts` / `sendWB` – **Wirkungsbericht-Erinnerungen** (8/10 Wochen ab `aktiviertAm`)
  mit Mailto-Link an den Jobcoach.

---

## 6. Teilnehmer-Rapport (zentrale Auswertung)

**Modal** „Teilnehmer-Rapport": Teilnehmer wählen → `genRapportPDF()` erzeugt ein PDF.

- **Ein einziger Gesamtrapport** pro Teilnehmer (keine Monats-/Zeitraumfilter).
- **Zeitraum:** automatisch von `einsatzVon` (bzw. erstem Plan-Eintrag) **bis und mit heute**
  (Generierungstag). Zukünftige, nur geplante Tage werden **nicht** mitgezählt.
- Wochenenden und Feiertage werden ausgeschlossen; Absenzen sperren die betroffenen Slots.
- Schul-Codes werden sowohl aus **fixen Schultagen** (Blocks) als auch aus **Tages-Einteilungen**
  (`plan.vm`/`plan.nm`) gezählt – damit stimmt der Rapport mit der Monatsübersicht überein.

### 6.1 Berechnungsregeln (Faktoren)

**SCHULE (Ausgabe in Lektionen):**
| Position | Regel |
|----------|-------|
| Schultage (S) | 1 ganzer Schultag (VM+NM) = **8** Lektionen (VM 4h + NM 4h) |
| DAZ / DAZ-VM / DAZ-NM | je ½-Tag (VM oder NM) = **4** Lektionen; ganzer Tag = 2× = **8** |
| FöA / FöA-VM / FöA-NM | je ½-Tag (VM oder NM) = **4** Lektionen; ganzer Tag = 2× = **8** |
| DK-VM / DK-NM | je ½-Tag = **4** Lektionen |
| DK (ganzer Tag, manuell) | = DK-VM + DK-NM = **8** Lektionen |
| **Total Schul-Lektionen** | Summe S + DAZ + FöA (+ DK) |

> Die Lektionen-Zählung läuft über das **Label** (`vmL`/`nmL` = Basis-Code) je Slot: ein
> Ganztags-Code zählt automatisch beide Halbtage (2× 4 = 8). `IKPC` hat keine eigene Lektionen-Zeile.

**LERNWERKSTATT (Ausgabe in Lektionen):**
| Position | Regel |
|----------|-------|
| Sport | je Einsatz = **2** Lektionen |
| Bewerben (BW) | je Einsatz = **4** Lektionen |

**ABWESENHEITEN (Ausgabe in Tagen bzw. Vorkommnissen):**
| Code | Regel |
|------|-------|
| Unbezahlte Absenz (UA) | ganzer Tag = 1; VM/NM = 0.5 |
| Bezahlte Absenz (BA) | ganzer Tag = 1; VM/NM = 0.5 |
| Krankheit (KR) | ganzer Tag = 1; VM/NM = 0.5; KR-W = 5 (je Wochentag 1) |
| Zu spät (ZS) | Anzahl Vorkommnisse (×) |

**SCHNUPPERLEHRE (Ausgabe in Tagen):**
| Position | Regel |
|----------|-------|
| Schnuppertag | je `SL`-Tag = 1 Tag |
| Schnupperlehre (ganze Woche) | je `S.EX`-Tag = 1 (volle Woche = 5 Tage) |

> **Ferien/Militär/Unfall im Teilnehmer-Rapport:** `FE` (Ferien), `MI` (Militär/
> Zivilschutz) und `UN` (Unfall) erscheinen im Abwesenheiten-Block **als eigene
> Zeile** – `FE`/`MI` jedoch **nur, wenn ihr Total > 0 ist** (sonst ausgeblendet, um
> die Tabelle kurz zu halten); `UN` wird immer angezeigt, auch bei 0. `FE-W` zählt
> dabei wie die anderen Wochencodes 5 Tage (Mo–Fr), `SL` (Schnupperlehre) läuft
> separat im Abschnitt SCHNUPPERLEHRE (siehe unten), nicht unter ABWESENHEITEN.

### 6.2 Darstellung
Zwei-Spalten-Tabelle **Bereich · Total**; pro Abschnitt nur die **Endzahl** mit Einheit
(z. B. „Schule 168 Lektionen", „Sport 14 Lektionen", „Bewerben 68 Lektionen").
Halbe Tage als `.5`, ganze Zahlen ohne Dezimalstelle. Kopf: Name, Jobcoach, Zeitraum.
Dateiname: `Rapport_<Nachname>_<Vorname>_Laufzeit.pdf`.

---

## 7. Weitere PDF-Exporte
- **`genWeekPDF`** – Wochenplan: Seite 1 Wochenübersicht, Seite 2 Tagesübersicht,
  Seite 3 Monatsübersicht (Querformat, Monat der berichteten Woche), Seite 4 Legende.
- **`genPDF` / `buildRpt`** – Monatsrapport (BASISJOB-Format, inkl. Freitext-Notizen).
- **`genNeophytPDF`** – Liste „Anzahl Teilnehmende pro Tag" (KW, Datum) übers ganze Jahr.
- **`genTNPDF`** (Kennzahlen aus `tnStats`) – Teilnehmer-Statistik als PDF: aktive
  Teilnehmende (exkl. Schnuppern), Anzahl mit aktivem IIZ, Aufschlüsselung nach Status
  (Lehrstelle/Praktikum/Praktikum+Lehrstelle), Schnuppern separat ausgewiesen.
  Bezugsmenge = aktive TN (`isActiveOnDay`) am Erstellungstag.

---

## 8. Speichern, Sicherung, Import

- **Auto-Save** in `localStorage` bei jeder Änderung; `dirty`-Flag warnt beim Schließen.
- **Manuelles Speichern** (`manualSave`) als JSON-Datei mit Datum/Zeitstempel im Namen.
  Via **File System Access API** (`showDirectoryPicker`) wird **einmalig ein Ordner**
  gewählt (`changeSaveLocation`, 📁-Knopf); das Ordner-Handle wird in IndexedDB
  abgelegt (`fhStore`/`fhLoad`). Danach legt jede Sicherung **ohne Fenster** eine neue
  Zeitstempel-Datei in diesem Ordner ab. Der Browser fragt aus Sicherheitsgründen nach
  jedem Neuladen der Seite **einmalig** nach der Schreib-Erlaubnis (nicht abschaltbar);
  innerhalb der Sitzung danach stumm. **Wochenplan generieren** (`genWeekPDF`) erzeugt
  das PDF und löst zugleich diese Sicherung aus.
- **JSON-Import** (`doImportFSA`/`processImportJSON`): mit **Vorschau der Änderungen**
  vor dem Überschreiben; schützt bestehende Daten (Abweichungen nur auf Bestätigung).
  Importierte Daten laufen durch `sanitizeDB()` (Whitelist/Typprüfung), bevor sie
  in der Vorschau erscheinen und übernommen werden.
- **Dokument-Import** (`handleDocUpload` → `parseDocText`): liest Teilnehmerlisten aus
  PDF/Excel/Word, erkennt Klassen-Codes (z. B. `FöA-Do-…`, `DMA-Mo-…`), Jobcoach,
  Eintritt/Ende-ZV; legt fixe Schultage & Sporttermine an. Eintritt/Ende-ZV landen
  in `einsatzVon`/`einsatzBis` (nicht im Namen).

---

## 9. Wiederherstellung

**Schnellster Weg:** Datei `index.html` zurückspielen (sie ist vollständige Klartext-App).
**Daten:** separat über den JSON-Export sichern und via „Import" wieder einspielen –
die App-Datei enthält selbst keine Teilnehmerdaten.

Falls nur dieses Dokument vorliegt: Die App lässt sich anhand der Abschnitte §2–§8
als einzelne `index.html` (HTML+CSS+JS, CDN-Libs aus §1) neu aufbauen.

---

## 10. Änderungshistorie dieser Session (Teilnehmer-Rapport)
1. Filter (Zeitraum/Monat) entfernt → Gesamtrapport; Modal-Titel „Teilnehmer-Rapport";
   BA-Zeile ergänzt; Halbtags-Logik (VM/NM = 0.5); Tabelle auf 2 Spalten vereinfacht.
2. Schule (FöA/DAZ/DK) auch aus Tages-Einteilung zählen (Abgleich mit Monatsübersicht).
3. Stichtag „heute": zukünftige Tage ausgenommen; `S` auch als Tages-Einteilung.
4. BW-Faktor von 4.5 → **4** Lektionen.
