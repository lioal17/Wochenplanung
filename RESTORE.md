# RESTORE.md – Vollständige Wiederherstellungsanleitung

**Referenzstand:** Commit `48dcb40` (`main`) vom 2026-07-16
**Paket erstellt:** 2026-08-04
**Live-Adresse:** https://lioal17.github.io/Wochenplanung/

Diese Anleitung stellt die Anwendung **exakt im Zustand vom 2026-08-04** wieder her –
auch dann, wenn das GitHub-Repository beschädigt oder gelöscht wurde.

---

## 0. Kurzfassung (60 Sekunden)

```
ZIP entpacken  →  index.html im Browser doppelklicken  →  fertig.
```

Es gibt **keinen Server, keine Datenbank, keinen Build-Schritt, keine Installation**.
Die gesamte Anwendung – HTML, CSS, JavaScript, Design, Icons, PDF-Engine – steckt
in der **einen Datei `index.html`** (1,5 MB).

---

## 1. Welche Dateien werden benötigt?

### 1.1 Zwingend erforderlich (ohne diese läuft nichts)

| Datei | Größe | Ohne sie … |
|---|---|---|
| **`index.html`** | 1'500'156 Bytes | … existiert die Anwendung nicht. **Das ist die komplette App.** |

> **Das ist die vollständige Liste.** `index.html` hat **keine** externen
> Abhängigkeiten zu Dateien im Paket: keine separaten `.css`, `.js`, Bild-,
> Icon- oder Schriftdateien. Alle relativen Pfade innerhalb der Datei sind
> `data:`-URIs oder blob-URLs, funktionieren also standortunabhängig.

### 1.2 Für die erneute Veröffentlichung nötig

| Datei | Zweck |
|---|---|
| `.github/workflows/deploy-pages.yml` | Automatisches GitHub-Pages-Deployment |
| `.nojekyll` | Verhindert Jekyll-Verarbeitung durch GitHub Pages |

### 1.3 Für Verständnis und Weiterentwicklung

`README.md`, `PROJECT_STRUCTURE.md`, `CHANGELOG_BASELINE.md`, `CLAUDE.md`,
`SECURITY.md`, `PROJEKT.md`, `UEBERGABE.md`, `RECOVERY-2026-07-10/`,
`docs/archiv/`, `.gitignore`

### 1.4 Komplettes Repository-Backup

| Datei | Zweck |
|---|---|
| `_git-repo/Wochenplanung-repo.bundle` | **Gesamte Git-Historie** (vollständige Historie ab 05.06.2026, rund 400 Commits, alle Branches inkl. aller `sicherung-*`-Stände) – siehe Abschnitt 6 |

---

## 2. Wiederherstellung – Schritt für Schritt

### Schritt 1: ZIP entpacken
In einen beliebigen Ordner, z. B. `C:\Wochenplanung\` oder `~/Wochenplanung/`.
Der Speicherort ist frei wählbar – auch USB-Stick oder Netzlaufwerk.

### Schritt 2: Vollständigkeit prüfen
`index.html` muss vorhanden und **ca. 1,5 MB** groß sein. Ist sie deutlich
kleiner, wurde sie beim Kopieren beschädigt.

*Optionale Prüfsumme (SHA-256 der Referenzversion):*
```
4ed904b36a1843b77210a6279339450501a6fe8148791728b3f2799e3d3b47df
```
```bash
sha256sum index.html          # Linux / macOS
certutil -hashfile index.html SHA256   # Windows
```

### Schritt 3: Anwendung starten
`index.html` **doppelklicken**. Die App öffnet sich im Standardbrowser und ist
sofort einsatzbereit.

### Schritt 4: Funktionsprüfung
Siehe Checkliste in Abschnitt 9.

### Schritt 5: Eigene Daten einspielen
Über **Import / Datensicherung einlesen** die eigene `.json`-Sicherung laden.
(Teilnehmerdaten sind aus Datenschutzgründen **nicht** Teil dieses Pakets.)

---

## 3. Wie wird das Projekt gestartet?

### Variante A – Doppelklick (Normalfall, empfohlen)
`index.html` öffnen. Funktioniert über das `file://`-Protokoll vollständig.

### Variante B – lokaler Webserver (nur falls nötig)
Manche Browser schränken `file://` stark ein. Dann:

```bash
python3 -m http.server 8080     # → http://localhost:8080/index.html
```
```bash
npx serve .                     # Node.js-Alternative
```

> ⚠️ **Wichtig:** `file://` und `http://localhost` haben **getrennte
> localStorage-Bereiche**. Nach einem Wechsel wirkt die App leer – dann einfach
> die `.json`-Sicherung erneut importieren.

### Variante C – GitHub Pages (öffentliche Veröffentlichung)
Siehe Abschnitt 4.

---

## 4. GitHub Pages neu einrichten und veröffentlichen

Falls das Repository gelöscht wurde und die App wieder online soll:

### 4.1 Repository neu anlegen
1. Auf GitHub ein neues Repository erstellen (z. B. `Wochenplanung`).
2. **Sichtbarkeit:** Die App enthält **keine** Daten und keine Geheimnisse, ein
   öffentliches Repository ist für GitHub Pages am einfachsten. Bei privatem
   Repository ist Pages nur mit kostenpflichtigen Plänen verfügbar.

### 4.2 Dateien hochladen
```bash
git init
git add index.html .nojekyll .github/workflows/deploy-pages.yml \
        README.md RESTORE.md PROJECT_STRUCTURE.md CHANGELOG_BASELINE.md \
        CLAUDE.md SECURITY.md PROJEKT.md UEBERGABE.md .gitignore
git commit -m "Wiederherstellung Referenzstand 2026-08-04"
git branch -M main
git remote add origin https://github.com/<BENUTZER>/<REPO>.git
git push -u origin main
```

*Oder – deutlich besser – aus dem Git-Bundle (Abschnitt 6), damit die komplette
Historie erhalten bleibt.*

### 4.3 Pages aktivieren
1. Repository → **Settings** → **Pages**
2. **Source:** `GitHub Actions` auswählen (**nicht** „Deploy from a branch")
3. Der mitgelieferte Workflow übernimmt den Rest automatisch.

### 4.4 Notwendige Einstellungen im Überblick

| Einstellung | Wert |
|---|---|
| Pages Source | **GitHub Actions** |
| Workflow-Datei | `.github/workflows/deploy-pages.yml` |
| Auslöser | Push auf `main` oder manuell (`workflow_dispatch`) |
| Berechtigungen | `contents: read`, `pages: write`, `id-token: write` (bereits im Workflow gesetzt) |
| Environment | `github-pages` (wird automatisch angelegt) |
| Jekyll | deaktiviert über `.nojekyll` |

### 4.5 Was wird veröffentlicht?
Der Workflow kopiert **bewusst nur** `index.html` und `.nojekyll` nach `_site/`.
Die interne Dokumentation (`PROJEKT.md`, `UEBERGABE.md`, Analyseberichte) bleibt
im Repository, wird aber **nicht** öffentlich ausgeliefert.

### 4.6 Ergebnis
Nach 1–2 Minuten ist die App erreichbar unter:
`https://<BENUTZER>.github.io/<REPO>/`

---

## 5. Welche Abhängigkeiten existieren?

### 5.1 Fest eingebettet – funktionieren vollständig offline

| Bibliothek | Version | Zweck |
|---|---|---|
| **jsPDF** | 2.5.1 | Erzeugung sämtlicher PDF-Dokumente |
| **jsPDF-AutoTable** | 3.8.2 | Tabellen-Layout in den PDFs |
| **SheetJS (xlsx)** | 0.18.5 | Excel-Import (`.xlsx`, `.xls`) |

Diese sind **im Quelltext von `index.html` enthalten** – kein Download, kein
Paketmanager, keine Internetverbindung nötig. Sie können nicht „verschwinden".

### 5.2 Extern nachgeladen – nur für den Dokument-Import

| Bibliothek | Version | Quelle |
|---|---|---|
| **pdf.js** | 3.11.174 | `https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js` (+ `pdf.worker.min.js`) |
| **mammoth** | 1.6.0 | `https://cdn.jsdelivr.net/npm/mammoth@1.6.0/mammoth.browser.min.js` |

**Falls diese CDNs eines Tages nicht mehr existieren:** Die Anwendung läuft
**vollständig weiter**. Es entfällt ausschließlich der *Import aus PDF- und
Word-Dateien*. Planung, Excel-Import, **alle** PDF-Exporte, Druck sowie
Export/Import der Datensicherung funktionieren unverändert.

*Wer das absichern will:* die beiden Dateien einmalig herunterladen, neben
`index.html` legen und die zwei `<script src="…">`-Zeilen auf die lokalen Pfade
umbiegen. Die CSP muss dafür **nicht** aufgeweicht werden (lokale Pfade sind
`self`, das Entfernen der CDN-Einträge ist eine Verschärfung, keine Lockerung).

### 5.3 Keine weiteren Abhängigkeiten
Es gibt **kein** `package.json`, **kein** `node_modules`, **keine** `.env`,
**keine** Datenbank, **kein** Framework (React/Vue/Angular), **kein** TypeScript,
**kein** Build-Tool (Webpack/Vite), **keinen** CSS-Präprozessor (Sass/Tailwind).
Reines Vanilla-HTML/CSS/JavaScript.

---

## 6. Komplettes Repository aus dem Git-Bundle wiederherstellen

Im Paket liegt `_git-repo/Wochenplanung-repo.bundle` – die **vollständige
Git-Historie** ab dem 05.06.2026 (rund 400 Commits) und alle Branches (inklusive der
`sicherung-*`-Wiederherstellungspunkte).

```bash
# Repository inkl. kompletter Historie wiederherstellen
git clone _git-repo/Wochenplanung-repo.bundle Wochenplanung
cd Wochenplanung
git branch -a          # alle Branches, auch alle sicherung-* Stände
```

Danach kann jeder frühere Stand wiederhergestellt werden:
```bash
git checkout sicherung-2026-07-13-1703   # Beispiel: älterer Sicherungsstand
```

Auf ein neues GitHub-Repository hochladen:
```bash
git remote set-url origin https://github.com/<BENUTZER>/<REPO>.git
git push -u origin --all
```

> Damit ist selbst der Totalverlust von GitHub abgedeckt: Historie, alle
> Branches und alle bisherigen Sicherungspunkte sind lokal reproduzierbar.

---

## 7. Welche Browser werden unterstützt?

| Browser | Mindestversion | Status |
|---|---|---|
| **Chrome / Edge** (Chromium) | 80+ (2020) | ✅ empfohlen, primär getestet |
| **Firefox** | 78+ (2020) | ✅ unterstützt |
| **Safari** (macOS/iOS) | 14+ (2020) | ✅ unterstützt |
| Internet Explorer 11 | – | ❌ **nicht** unterstützt |

**Technische Mindestanforderungen:** ES6-JavaScript (Arrow Functions, `const`/`let`,
Template Literals), `localStorage`, File API / `FileReader`, `Blob` und
`URL.createObjectURL`, CSS Custom Properties (`--variablen`), CSS Grid & Flexbox.

**Bedienung:** Für Drag-and-Drop-Zuweisungen ist ein Desktop-Browser mit Maus
klar am komfortabelsten. Das Layout besitzt einen Breakpoint bei
`max-width: 640px` für kleine Bildschirme; die Anwendung ist konzeptionell aber
auf Desktop-Nutzung ausgelegt.

---

## 8. Wichtige Besonderheiten

### 8.1 Datenhaltung – die zentrale Stolperfalle
Alle Daten liegen ausschließlich im **`localStorage` des Browsers**
(Schlüssel: `lw_db_v3`, Zeitstempel: `lw_lastsave`). Daraus folgt:

- Daten hängen an **Browser + Gerät + Benutzerprofil**. Ein anderer Browser oder
  PC zeigt einen leeren Stand.
- **Browserdaten/Websitedaten löschen ⇒ Planungsdaten sind weg.**
- Der Inkognito-/Privatmodus speichert **nicht** dauerhaft.
- Deshalb: **regelmäßig** über die Export-Funktion eine `.json`-Sicherung
  erstellen und lokal sicher ablegen.

### 8.2 Datenschutz (oberstes Projektgebot)
- Reine Offline-/Local-First-App: kein Backend, kein Tracking, keine Telemetrie,
  kein Cloud-Sync.
- Strikte Content-Security-Policy verhindert Datenabfluss:
  ```
  default-src 'none'; script-src 'unsafe-inline' https://cdnjs.cloudflare.com https://cdn.jsdelivr.net;
  style-src 'unsafe-inline'; img-src data: blob:; worker-src blob: https://cdnjs.cloudflare.com;
  connect-src https://cdnjs.cloudflare.com https://cdn.jsdelivr.net; base-uri 'none'; form-action 'none'
  ```
  Diese CSP darf bei künftigen Änderungen **nicht** aufgeweicht werden.
- Die beiden CDNs liefern **ausschließlich Programmcode**; es werden **niemals
  Daten dorthin gesendet** (`form-action 'none'` schließt auch Formular-POSTs aus).
- Teilnehmerdaten gehören niemals in Commits, Pull Requests, ZIP-Pakete,
  Screenshots, Logs – und nie in einen Chat mit einer KI.
- Auf geteilten Rechnern nach Nutzung die Browserdaten löschen.

### 8.3 Langzeitfähigkeit
Da alles in einer einzigen HTML-Datei liegt und weder Laufzeitumgebung noch
Paketmanager oder Dienst benötigt wird, ist die Langzeitfähigkeit hoch: Solange
ein Browser HTML und JavaScript ausführt, läuft die Anwendung. Es gibt keine
Pakete, die aus einer Registry verschwinden könnten, und keine Server, die
abgeschaltet werden könnten.

### 8.4 Weiterentwicklung
- Alles in **einer** `index.html` (HTML + CSS + JS inline), bewusst ohne Build.
- Nutzerdaten, die in HTML gerendert werden, **immer** über `esc()` (XSS-Schutz).
- Änderungen über Branch + Pull Request nach `main`, nie per Direkt-Push.

---

## 9. Prüfliste nach der Wiederherstellung

- [ ] `index.html` vorhanden, ca. 1,5 MB
- [ ] App öffnet sich im Browser, Oberfläche erscheint
- [ ] Alle sechs Bereiche erreichbar: Plan, Personen, WB-TN, Monat, Rapport, Import
- [ ] Design korrekt: Farben, Icons, Layout, Favicon im Browser-Tab
- [ ] Eine Teilnehmerzeile lässt sich per Drag-and-Drop zuweisen
- [ ] **Wochenplan-PDF** wird erzeugt (bestätigt: jsPDF + AutoTable intakt)
- [ ] **Monatsrapport-PDF** wird erzeugt
- [ ] Druckfunktion liefert korrektes Layout
- [ ] Excel-Import funktioniert (bestätigt: SheetJS intakt, offline)
- [ ] `.json`-Datensicherung lässt sich exportieren **und** wieder importieren
- [ ] Nach Neuladen der Seite sind die Daten noch vorhanden (localStorage)
- [ ] *(nur mit Internet)* PDF-/Word-Import funktioniert
