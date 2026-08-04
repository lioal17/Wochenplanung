# README_RESTORE.md – Vollständige Wiederherstellung der Wochenplanung-App

**Snapshot-Datum:** 2026-08-04
**Stand (Commit):** `48dcb40` vom 2026-07-16 (Branch `main`)
**Zweck:** Diese Datei beschreibt, wie das Projekt aus dieser ZIP-Datei **vollständig**
wiederhergestellt wird – auch noch in mehreren Jahren, ohne Internet und ohne
Zugriff auf GitHub.

---

## 0. Das Wichtigste in 30 Sekunden

Die Wochenplanung ist eine **reine Offline-Webapp aus einer einzigen Datei**.

> **Wiederherstellung = `index.html` aus dieser ZIP entpacken und im Browser
> per Doppelklick öffnen. Fertig.**

Es gibt **keinen Server, keine Datenbank, keinen Build-Schritt und keine zu
installierenden Pakete**. Alles Nötige steckt in `index.html`.

---

## 1. Voraussetzungen für die Wiederherstellung

| Voraussetzung | Details |
|---|---|
| **Betriebssystem** | Beliebig (Windows, macOS, Linux) – irrelevant |
| **Browser** | Ein moderner Browser: Chrome/Edge (empfohlen), Firefox oder Safari |
| **Programm zum Entpacken** | Standard-ZIP-Entpacker des Betriebssystems |
| **Internet** | **Nicht erforderlich** für den Betrieb (siehe Abschnitt 8) |
| **Node.js / npm / Python** | **Nicht erforderlich** |
| **Webserver (Apache/nginx/IIS)** | **Nicht erforderlich** |
| **Datenbankserver** | **Nicht erforderlich** |

Mindestanforderung an den Browser: Unterstützung für `localStorage`, ES6-JavaScript
und die File-API. Jeder Browser ab ca. Baujahr 2018 erfüllt das.

---

## 2. Schritt-für-Schritt-Anleitung zur vollständigen Wiederherstellung

1. **ZIP entpacken** – in einen beliebigen Ordner, z. B. `C:\Wochenplanung\`
   oder `~/Wochenplanung/`.
2. **Ordnerinhalt prüfen** – es muss mindestens `index.html` vorhanden sein
   (ca. 1,5 MB). Siehe Abschnitt 9 für die vollständige Dateiliste.
3. **`index.html` per Doppelklick öffnen** – die App startet sofort im
   Standardbrowser.
   *Alternative:* Rechtsklick → „Öffnen mit" → gewünschter Browser.
4. **Funktion prüfen** – die Oberfläche der Wochenplanung erscheint. Die App ist
   damit vollständig wiederhergestellt.
5. **Eigene Daten einspielen** – siehe Abschnitt 5 („Datenwiederherstellung").
6. *(Optional)* **Lesezeichen setzen**, damit die Datei schnell wiederzufinden ist.

> **Hinweis:** Die App muss **nicht** „installiert" werden. Sie darf an jedem
> beliebigen Ort liegen – auch auf einem USB-Stick oder Netzlaufwerk.

---

## 3. Installation aller Abhängigkeiten

**Es sind keine Abhängigkeiten zu installieren.** Alle Bibliotheken sind
vollständig **in `index.html` eingebettet** (kein `npm install`, keine
`package.json`, kein `node_modules`).

### Enthaltene Bibliotheken (offline, fest eingebettet)

| Bibliothek | Version | Zweck | Status |
|---|---|---|---|
| **jsPDF** | 2.5.1 | PDF-Erzeugung | eingebettet – offline |
| **jsPDF-AutoTable** | 3.8.2 | Tabellen in PDFs | eingebettet – offline |
| **SheetJS (xlsx)** | 0.18.5 | Excel-Import | eingebettet – offline |

### Extern nachgeladene Bibliotheken (nur für Dokument-Import)

Diese beiden werden **nur bei Bedarf** aus dem CDN geladen und sind für den
Kernbetrieb **nicht** notwendig:

| Bibliothek | Version | Quelle | Zweck |
|---|---|---|---|
| **pdf.js** | 3.11.174 | `https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/` | Import aus PDF-Dateien |
| **mammoth** | 1.6.0 | `https://cdn.jsdelivr.net/npm/mammoth@1.6.0` | Import aus Word-Dateien |

**Falls diese CDNs in Zukunft nicht mehr erreichbar sind:** Die App funktioniert
weiterhin vollständig – lediglich der *Import aus PDF-/Word-Dateien* entfällt.
Alle anderen Funktionen (Planung, Excel-Import, sämtliche PDF-Exporte,
Datensicherung) laufen unverändert offline weiter.

> **Wichtig:** Diese CDNs dienen **ausschließlich dem Laden von Programmcode**.
> Es werden **niemals Daten dorthin gesendet.**

---

## 4. Einrichtung der Datenbank

**Entfällt – das Projekt verwendet keine Datenbank.**

Es gibt keinen Datenbankserver, kein Schema, keine Migrationen und keine
Verbindungszeichenfolgen. Die Datenhaltung erfolgt ausschließlich im
**`localStorage` des Browsers** auf dem lokalen Gerät.

Konsequenzen, die man kennen muss:

- Die Daten hängen an der **Kombination aus Browser + Gerät + Benutzerprofil**.
- Ein anderer Browser oder ein anderer PC sieht **eigene, leere** Daten.
- Das Löschen von „Browserdaten/Cookies/Websitedaten" **löscht auch die
  Planungsdaten**.
- Der private Modus / Inkognito-Modus speichert **nicht dauerhaft**.

---

## 5. Einspielen des Datenbank-Backups (Datenwiederherstellung)

### 5.1 Warum in dieser ZIP keine Daten enthalten sind

Diese ZIP enthält **bewusst keine Teilnehmerdaten** – das ist kein Versehen,
sondern die oberste Projektregel (Datenschutz, siehe `SECURITY.md` und
`CLAUDE.md`). Teilnehmerdaten umfassen personenbezogene und besonders
schützenswerte Angaben (Namen, Absenzen inkl. Krankheit/Militär, Notizen) und
dürfen das lokale Gerät **niemals** verlassen – auch nicht in einem Backup-Archiv,
in einem Repository oder in einem Chat.

Die ZIP stellt daher die **Anwendung** wieder her. Die **Daten** sichern und
spielen Sie selbst lokal ein – über die eingebaute Export-/Import-Funktion:

### 5.2 Datensicherung erstellen (regelmäßig durchführen!)

1. App öffnen.
2. Funktion **„Datensicherung" / „Export"** aufrufen.
3. Die App erzeugt eine **`.json`-Datei** mit dem kompletten Datenbestand.
4. Diese Datei **lokal und sicher** ablegen (z. B. verschlüsselter Ordner,
   dienstlicher Datenträger) – **niemals** in Cloud-Dienste hochladen,
   niemals ins Git-Repository legen, niemals per Chat/E-Mail versenden.

> Die `.gitignore` blockiert `*.json`, `*.csv`, `*.xlsx` bereits automatisch,
> damit solche Dateien nicht versehentlich committet werden.

### 5.3 Datensicherung wieder einspielen

1. Wiederhergestellte `index.html` im Browser öffnen.
2. Funktion **„Datensicherung einlesen" / „Import"** aufrufen.
3. Die zuvor gesicherte `.json`-Datei auswählen.
4. Der Datenbestand ist wiederhergestellt.

> **Für eine echte vollständige Wiederherstellung brauchen Sie also zwei Teile:**
> **(A)** diese ZIP (die App) **+ (B)** Ihre eigene aktuelle `.json`-Datensicherung
> (die Daten). Bewahren Sie beide getrennt, aber parallel aktuell auf.

---

## 6. Konfiguration der Umgebungsvariablen

**Entfällt – das Projekt verwendet keine Umgebungsvariablen.**

Es gibt **keine** `.env`, **keine** `.env.example`, keine API-Schlüssel, keine
Tokens, keine Zugangsdaten und keine Endpunkt-URLs. Das ist Absicht: Da die App
ohne Server und ohne Netzwerkzugriff auf Daten arbeitet, existiert nichts, was
konfiguriert werden müsste.

**Es sind entsprechend auch keinerlei sensible Zugangsdaten in dieser ZIP
enthalten** – weder im Klartext noch verschlüsselt.

Die einzige „Konfiguration" sind Einstellungen, die direkt in der App vorgenommen
und im `localStorage` gespeichert werden (und damit Teil Ihrer `.json`-Sicherung
aus Abschnitt 5 sind).

---

## 7. Starten des Projekts

### Variante A – Normalbetrieb (empfohlen)

`index.html` doppelklicken. Mehr ist nicht nötig.

### Variante B – lokaler Webserver (optional)

Nur nötig, falls ein Browser den Datei-Zugriff (`file://`) stark einschränkt:

```bash
# Mit Python 3
python3 -m http.server 8080
# danach im Browser öffnen: http://localhost:8080/index.html
```

```bash
# Mit Node.js
npx serve .
```

> Achtung: Ein Wechsel zwischen `file://` und `http://localhost` bedeutet einen
> **anderen localStorage-Bereich** – die Daten erscheinen dann leer. In dem Fall
> die `.json`-Sicherung erneut importieren (Abschnitt 5.3).

### Variante C – Veröffentlichung über GitHub Pages (optional)

Das Repository enthält den fertigen Workflow
`.github/workflows/deploy-pages.yml`. Bei einem Push auf `main` wird
**ausschließlich** `index.html` (plus `.nojekyll`) veröffentlicht – die interne
Dokumentation bleibt bewusst außen vor. Für die reine Wiederherstellung ist
dieser Weg **nicht** erforderlich.

---

## 8. Bekannte Voraussetzungen und Besonderheiten

### 8.1 Datenschutz (oberstes Gebot)

- Die App ist eine **reine Offline-/Local-First-Anwendung**: kein Backend, kein
  Tracking, keine Telemetrie, kein Cloud-Sync.
- Eine strikte **Content-Security-Policy** verhindert jeden Datenabfluss:

  ```
  default-src 'none'; script-src 'unsafe-inline' https://cdnjs.cloudflare.com https://cdn.jsdelivr.net;
  style-src 'unsafe-inline'; img-src data: blob:; worker-src blob: https://cdnjs.cloudflare.com;
  connect-src https://cdnjs.cloudflare.com https://cdn.jsdelivr.net; base-uri 'none'; form-action 'none'
  ```

  Diese CSP darf bei künftigen Änderungen **nicht aufgeweicht** werden.
- Teilnehmerdaten gehören **niemals** in Commits, PRs, ZIPs, Screenshots, Logs
  oder in einen Chat mit einer KI.

### 8.2 Datenhaltung – die wichtigste Stolperfalle

Daten liegen **nur** im `localStorage` des jeweiligen Browsers. Deshalb:

- **Regelmäßig** die `.json`-Datensicherung erstellen (Abschnitt 5.2).
- **Vor** einem Browserwechsel, einem Geräteneustart-Setup oder dem Löschen von
  Browserdaten **unbedingt** exportieren.
- Der Inkognito-/Privatmodus verliert die Daten beim Schließen.

### 8.3 Betrieb ohne Internet

Voll funktionsfähig ohne Internet sind: Planung, Excel-Import, **alle**
PDF-Exporte, Export/Import der Datensicherung. Nur der Import aus **PDF- und
Word-Dateien** benötigt eine Internetverbindung (CDN-Nachladung, Abschnitt 3).

### 8.4 Langzeit-Archivierung (Öffnen in mehreren Jahren)

Da alles in einer einzigen HTML-Datei mit eingebettetem Code steckt und keine
Laufzeitumgebung, kein Paketmanager und kein Dienst benötigt wird, ist die
Langzeitfähigkeit hoch: Solange ein Browser existiert, der HTML und JavaScript
ausführt, lässt sich die App öffnen. Es gibt keine Pakete, die „verschwinden"
können, und keine Server, die abgeschaltet werden könnten.

Empfehlung: diese ZIP zusammen mit der aktuellen `.json`-Datensicherung an
**mindestens zwei** getrennten, lokal kontrollierten Orten aufbewahren.

### 8.5 Weiterentwicklung

- Die gesamte App liegt in **einer** `index.html` (HTML + CSS + JS inline).
  Es gibt bewusst keinen Build-Schritt.
- Ausgaben, die Nutzerdaten in HTML rendern, müssen **immer** über die
  Hilfsfunktion `esc()` laufen (XSS-Härtung).
- Änderungen laufen über Branch + Pull Request nach `main`, nie per Direkt-Push.

---

## 9. Inhalt dieser ZIP-Datei (Prüfliste)

| Datei / Ordner | Inhalt |
|---|---|
| `index.html` | **Die komplette Anwendung** – Quellcode, Styles, Templates, Komponenten, eingebettete Bibliotheken (ca. 1,5 MB, ~4.700 Zeilen) |
| `README_RESTORE.md` | Diese Wiederherstellungsanleitung |
| `README.md` | Kurzübersicht des Projekts |
| `CLAUDE.md` | Verbindliche Projektregeln (Datenschutz, Arbeitsweise) |
| `SECURITY.md` | Sicherheits- und Datenschutzkonzept |
| `PROJEKT.md` | Ausführliche Projektdokumentation |
| `UEBERGABE.md` | Übergabedokumentation |
| `.github/workflows/deploy-pages.yml` | Deployment-Konfiguration (GitHub Pages) |
| `.gitignore` | Schutzregeln gegen versehentliches Committen von Daten |
| `.nojekyll` | Pages-Einstellung (verhindert Jekyll-Verarbeitung) |
| `RECOVERY-2026-07-10/` (8 Dateien) | Detaillierte Wiederherstellungs-/Analyse-Dokumentation: Projektzusammenfassung, Layout- & Designsystem, Features, Dead-Code-Analyse, GitHub-Analyse, Versions-Snapshot, Finalisierung |
| `docs/archiv/` (3 Dateien) | Archivierte Analyseberichte und Projektzusammenfassungen |

**Nicht enthalten (bewusst):** Teilnehmerdaten, Export-`*.json`, `*.csv`,
`*.xlsx` – aus Datenschutzgründen (siehe Abschnitt 5.1).

**Nicht vorhanden (existiert im Projekt nicht):** `package.json`/`node_modules`
(keine Abhängigkeiten), `.env`/`.env.example` (keine Umgebungsvariablen),
Datenbank-Dumps (keine Datenbank), Build-Skripte (kein Build), separate
Asset-/Font-/CSS-Ordner (alles in `index.html` eingebettet).

---

## 10. Schnellprüfung nach der Wiederherstellung

- [ ] `index.html` vorhanden und ca. 1,5 MB groß
- [ ] App öffnet sich im Browser und zeigt die Oberfläche
- [ ] Eigene `.json`-Datensicherung erfolgreich importiert
- [ ] Ein PDF-Export wird korrekt erzeugt (bestätigt: Offline-Bibliotheken intakt)
- [ ] Eine neue `.json`-Datensicherung erstellt und sicher lokal abgelegt
