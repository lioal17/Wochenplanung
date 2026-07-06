# Übergabe- & Notfall-Dokumentation – Wochenplanung Lernwerkstatt

> **Zweck:** Dieses Dokument ermöglicht es einer Nachfolge-Person, das Programm
> **ab dem aktuellen Stand vollständig weiterzuführen und weiterzuentwickeln** –
> auch wenn die ursprüngliche Entwicklerin/der Entwickler nicht mehr verfügbar ist.
> Es ist bewusst so geschrieben, dass auch jemand ohne Vorwissen über dieses
> konkrete Projekt einsteigen kann.
>
> **Stand der Sicherung:** 2026-06-25
> **Repository:** https://github.com/lioal17/Wochenplanung
> **Letzter Commit (HEAD):** `b029ca1` – „Merge pull request #97"
> **Branch dieser Sicherung:** `claude/amazing-bohr-crbowh`

---

## 0. Das Wichtigste in 60 Sekunden

- Das Programm ist **eine einzige Datei**: `index.html`. Mehr braucht es nicht.
- **Zum Benutzen:** Datei doppelklicken → öffnet im Browser. Kein Server, keine
  Installation, kein Internet nötig (Ausnahme: nur der Dokument-Import braucht Internet).
- **Zum Sichern der Teilnehmer-Daten:** in der App den Export-/Speichern-Knopf nutzen
  (lädt eine `.json`-Datei herunter). Diese Datei aufbewahren. Die Daten liegen **nicht**
  im Programm, sondern im Browser des jeweiligen Computers.
- **Zum Weiterentwickeln:** `index.html` mit einem Texteditor (oder dem GitHub-Repo)
  bearbeiten. Der Code ist HTML + CSS + JavaScript, alles in dieser einen Datei.
- **Diese Sicherung enthält alles** (Code, komplette Versionsgeschichte, Doku) –
  siehe Abschnitt 8.

---

## 1. Was ist dieses Programm?

Ein **Wochenplanungs- und Rapport-Tool für das BASISJOB Motivationssemester** der
Lernwerkstatt (Kanton Thurgau). Es verwaltet Teilnehmende, plant deren Wochen
(Vormittag/Nachmittag-Einteilung in Werkstätten / Schule / Sport), erfasst
Abwesenheiten und eCase-Status und erzeugt verschiedene PDF-Auswertungen
(Wochenplan, Teilnehmer-Rapport, Monatsrapport, Neophyt-Liste).

Die **vollständige fachliche und technische Beschreibung** (Datenmodell, alle
Codes, Berechnungsregeln, Funktionen) steht in **`PROJEKT.md`** – dieses Dokument
hier ist die *Übergabe-Sicht*, `PROJEKT.md` ist das *technische Nachschlagewerk*.

---

## 2. Welche Dateien gehören zum Projekt?

| Datei | Inhalt | Wichtigkeit |
|-------|--------|-------------|
| **`index.html`** | Das **gesamte Programm** (HTML + CSS + JavaScript, ~1,4 MB). | ⭐ kritisch |
| `PROJEKT.md` | Vollständige technische Doku (Datenmodell, Codes, Regeln, Funktionen). | ⭐ kritisch |
| `README.md` | Kurzbeschreibung & Schnellstart. | wichtig |
| `UEBERGABE.md` | **Dieses Dokument** (Übergabe/Notfall). | ⭐ kritisch |
| `PROJEKT-ZUSAMMENFASSUNG-2026-06-19.md` | Historischer Stands-Bericht (Snapshot). | Referenz |
| `PROJEKT-ZUSAMMENFASSUNG-2026-07-03.md` | Historischer Stands-Bericht (Snapshot). | Referenz |
| `.github/workflows/deploy-pages.yml` | Automatisches Veröffentlichen auf GitHub Pages. | wichtig |
| `.nojekyll` | Verhindert, dass GitHub Pages die Datei umschreibt. | technisch |

> **Es gibt keine versteckten Bausteine.** Es gibt keinen Build-Schritt, keine
> `node_modules`, keine Datenbank, keinen Server. Was im Repo liegt, ist alles.

---

## 3. Wo liegt der Code, wo komme ich dran?

- **GitHub-Repository:** https://github.com/lioal17/Wochenplanung
  → Eigentümer-Konto: **`lioal17`**. Die Nachfolge-Person braucht **Schreibrechte**
  auf dieses Repo (als Mitarbeiter/Collaborator hinzufügen oder Repo-Eigentum übertragen).
- **Versionsverwaltung:** Git. Die **komplette Geschichte** (seit 2026-06-09) ist
  in dieser Sicherung als Git-Bundle enthalten (siehe Abschnitt 8) –
  damit ist das Projekt selbst dann vollständig wiederherstellbar, wenn das
  GitHub-Konto verloren geht.
- **Veröffentlichte Version (GitHub Pages):** Bei jedem Push auf den `main`-Branch
  baut der Workflow `deploy-pages.yml` automatisch die öffentliche Web-Version.
  Die URL erscheint in den GitHub-Pages-Einstellungen des Repos.

### Mindest-Zugänge, die die Nachfolge-Person braucht
1. **GitHub-Konto** mit Zugriff auf `lioal17/Wochenplanung` (Schreibrechte).
2. Die **JSON-Datensicherungen** der echten Teilnehmer-Daten (liegen lokal auf dem
   Arbeitscomputer, **nicht** auf GitHub – siehe Abschnitt 6). Diese müssen physisch
   übergeben werden (USB, geschütztes Laufwerk o. ä.).

---

## 4. Wie benutze ich das Programm? (für den Arbeitsalltag)

1. `index.html` im Browser öffnen (Doppelklick oder GitHub-Pages-URL).
2. Teilnehmende anlegen/bearbeiten (Ansicht **👥 Teilnehmer**).
3. Wochen planen (Ansicht **📅 Wochenplan**).
4. Auswertungen erzeugen: **📄 Monatsrapport**, Teilnehmer-Rapport, Wochenplan-PDF.
5. **Regelmässig sichern** über den Export-/Speichern-Knopf (JSON-Datei) – siehe Abschnitt 6.

Eine ausführliche Erklärung aller Ansichten, Codes und Regeln steht in `PROJEKT.md`
(Abschnitte 4–7).

---

## 5. Wie entwickle ich das Programm weiter?

### Voraussetzungen
- Ein **Texteditor** (empfohlen: VS Code, kostenlos). Damit lässt sich `index.html`
  direkt öffnen und bearbeiten.
- Grundkenntnisse in **HTML, CSS und JavaScript**. Die Datei ist in „Vanilla
  JavaScript" geschrieben – **kein Framework**, kein React/Vue, keine Hilfsbibliothek
  zum Bauen. Wer JavaScript kann, kann diese Datei lesen und ändern.
- **Git** (zum Versionieren) – optional, aber empfohlen. Alternativ kann man im
  GitHub-Web-Editor direkt arbeiten.

### Aufbau von `index.html`
Die Datei ist klar gegliedert (von oben nach unten):
1. `<style>`-Block ganz oben: **Design-Tokens** (Farben unter `:root`) und alles CSS.
2. `<body>`: das HTML-Gerüst der Oberfläche (Navigation, Ansichten, Modals).
3. Mehrere `<script>`-Blöcke am Ende:
   - eingebettete Bibliotheken (jsPDF, jsPDF-AutoTable, SheetJS) – **nicht ändern**;
   - der eigentliche **App-Code** (Datenmodell `DB`, alle Funktionen, `init()` am Ende).

### Typischer Arbeitsablauf (mit Git)
```bash
# 1. Repo holen (einmalig)
git clone https://github.com/lioal17/Wochenplanung.git
cd Wochenplanung

# 2. Eigenen Branch für die Änderung anlegen
git checkout -b meine-aenderung

# 3. index.html im Editor ändern, im Browser testen (Datei öffnen, F5)

# 4. Änderung sichern
git add index.html
git commit -m "Kurz beschreiben, was geändert wurde"

# 5. Hochladen
git push -u origin meine-aenderung
# danach auf GitHub einen Pull Request stellen und auf main mergen
```

### Testen
Es gibt **keine automatisierten Tests**. Getestet wird **manuell im Browser**:
Datei öffnen, Funktion durchklicken, Ergebnis prüfen (z. B. ein Test-PDF erzeugen).
Wichtigste Prüfpunkte nach einer Änderung:
- App startet ohne Fehler (Browser-Entwicklerkonsole, F12 → „Console" → keine roten Fehler).
- Teilnehmer anlegen, Woche planen, ein PDF erzeugen funktioniert noch.
- JSON-Export/Import funktioniert noch.

### Veröffentlichen
Sobald die Änderung im `main`-Branch ist (per Merge), veröffentlicht GitHub Pages
sie automatisch (Workflow `deploy-pages.yml`). Nichts weiter zu tun.

---

## 6. Die echten Teilnehmer-Daten sichern & wiederherstellen ⚠️

**Sehr wichtig – hier liegt das grösste Risiko:**

- Die eingegebenen **Teilnehmer-Daten liegen NICHT im Code/auf GitHub**, sondern
  lokal im Browser des Arbeitscomputers (`localStorage`, technischer Schlüssel
  `lw_db_v3`). Sie sind an **diesen einen Browser auf diesem einen Gerät** gebunden.
- **Sicherung:** in der App den **Export-/Speichern-Knopf** benutzen → lädt eine
  `.json`-Datei mit Datum im Namen herunter. Diese Datei **ausserhalb des Browsers**
  aufbewahren (Netzlaufwerk, USB-Stick, Cloud-Ordner). Empfehlung: **mindestens
  wöchentlich**, und immer vor grösseren Änderungen.
- **Wiederherstellen / Umzug auf neuen Computer:** `index.html` öffnen → Import-Knopf →
  die zuletzt gesicherte `.json`-Datei einlesen. Damit sind alle Daten wieder da.
- **Wichtig:** Es gibt **keinen** internen Auto-Backup-Snapshot in der App. Die
  einzige Sicherung ist der **manuelle JSON-Export** – deshalb regelmäßig sichern.
- **Geteilte Rechner:** Nach der Nutzung die **Browserdaten** für die Seite löschen
  (Browser-Einstellungen → Websitedaten). Damit sind localStorage und IndexedDB weg.
  Vorher per Export sichern!

> **Datenschutz:** Diese JSON-Dateien enthalten **Personendaten** von Teilnehmenden.
> Sie gehören auf ein geschütztes Laufwerk, nicht in ein öffentliches Repository
> oder einen unverschlüsselten USB-Stick. **Niemals** eine JSON-Datensicherung nach
> GitHub hochladen.

---

## 7. Eingesetzte Bibliotheken (Lizenzen / Abhängigkeiten)

In `index.html` eingebettet (funktionieren offline):
- **jsPDF 2.5.1** + **jsPDF-AutoTable 3.8.2** – PDF-Erzeugung.
- **SheetJS / xlsx 0.18.5** – Excel-Import.

Nur per Internet/CDN (nur für Dokument-Import nötig):
- **PDF.js 3.11.174** – PDF-Import (Teilnehmerlisten).
- **Mammoth.js 1.6.0** – Word-(docx)-Import.

Alle sind quelloffene Bibliotheken. Solange `index.html` unverändert bleibt, müssen
diese **nicht** aktualisiert werden, damit die App läuft.

---

## 8. Inhalt dieser Sicherung (was wurde gesichert?)

Diese „vollumfängliche Datensicherung" besteht aus einem Archiv
**`Wochenplanung-Vollbackup-2026-06-25.zip`**, das Folgendes enthält:

1. **`index.html`** – das vollständige, lauffähige Programm (aktueller Stand).
2. **Alle Dokumentations-Dateien**: `UEBERGABE.md` (dieses Dokument), `PROJEKT.md`,
   `README.md`, `PROJEKT-ZUSAMMENFASSUNG-2026-06-19.md`.
3. **`.github/`** + **`.nojekyll`** – die Veröffentlichungs-Konfiguration.
4. **`Wochenplanung-git-vollbackup-2026-06-25.bundle`** – die **komplette
   Git-Versionsgeschichte** (alle Commits, alle Branches) in einer einzigen
   Datei. Damit ist das gesamte Projekt **auch ohne GitHub** wiederherstellbar:

```bash
# Projekt aus dem Bundle vollständig wiederherstellen:
git clone Wochenplanung-git-vollbackup-2026-06-25.bundle Wochenplanung
cd Wochenplanung
# -> komplettes Repo inkl. aller Branches und der ganzen Historie ist wieder da
```

> **Was diese Sicherung NICHT enthält:** die echten Teilnehmer-Daten (JSON, siehe
> Abschnitt 6) – diese müssen aus Datenschutzgründen separat und geschützt übergeben
> werden. Und die GitHub-**Zugangsrechte** – diese muss das `lioal17`-Konto manuell
> an die Nachfolge-Person erteilen.

---

## 9. Notfall-Checkliste für die Nachfolge-Person

Wenn du dieses Projekt übernimmst, arbeite diese Liste ab:

- [ ] **Archiv entpacken** (`Wochenplanung-Vollbackup-2026-06-25.zip`) und an einem
      sicheren Ort ablegen.
- [ ] `index.html` testweise im Browser öffnen – startet die App?
- [ ] **GitHub-Zugang sichern:** Zugriff auf `lioal17/Wochenplanung` erhalten
      (Schreibrechte) – oder, falls das Konto verloren ist, aus dem Git-Bundle ein
      neues eigenes Repository aufbauen (Abschnitt 8).
- [ ] **Teilnehmer-Daten besorgen:** die letzte JSON-Datensicherung beschaffen und
      über den Import-Knopf einlesen (Abschnitt 6).
- [ ] `PROJEKT.md` lesen – das ist das vollständige technische Handbuch.
- [ ] Eigenen kleinen Test-Change machen (z. B. einen Tippfehler korrigieren) und
      den Ablauf aus Abschnitt 5 einmal durchspielen, um die Werkzeugkette zu prüfen.
- [ ] Eine **regelmässige Daten-Sicherungs-Routine** etablieren (Abschnitt 6).

---

## 10. Kontakt / Verantwortlichkeiten

- **GitHub-Eigentümer:** `lioal17`
- **Repository:** https://github.com/lioal17/Wochenplanung
- Bei Fragen zur Fachlogik (Codes, Rapport-Regeln, Berechnungen): siehe `PROJEKT.md`.

> Diese Datensicherung wurde am **2026-06-25** erstellt, um die lückenlose
> Weiterführung des Projekts unabhängig von einer einzelnen Person sicherzustellen.
</content>
</invoke>
