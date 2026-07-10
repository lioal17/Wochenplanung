# CLAUDE.md – Verbindliche Projektregeln (Wochenplanung)

## 🔒 OBERSTES GEBOT: Datenschutz (nicht verhandelbar)

**Teilnehmer-(TN-)Daten dürfen NIEMALS hochgeladen, übertragen oder aus dem
lokalen Browser herausgegeben werden. Sie bleiben ausschließlich lokal und
offline. Datenschutz hat oberste Priorität – vor jedem Feature.**

Immer einzuhalten:

- Die App bleibt eine **reine Offline-/Local-First-Webapp** (eine einzige
  `index.html`, Daten nur in `localStorage`, **kein** Server/Backend, kein
  Tracking, keine Telemetrie).
- **Keine** Funktion einbauen, die TN-Daten per Netzwerk sendet: kein `fetch`/
  XHR/WebSocket/`sendBeacon`/Formular-POST an externe Ziele, kein Cloud-Sync,
  kein Analytics.
- Die **Content-Security-Policy nicht aufweichen** (`default-src 'none'`,
  `connect-src` nur die zwei CDNs für den Dokument-*Import*, `form-action 'none'`),
  sodass keine Datenexfiltration möglich wird.
- **Nie** echte TN-Daten in Commits, Pull Requests, ZIP-Sicherungen, Screenshots,
  Logs, Artifacts oder Issue-/PR-Kommentaren – und niemals an Dritte oder externe
  Dienste – weitergeben. Export-`*.json` (Datensicherungen) gehören **nicht** ins
  Repository (siehe `.gitignore`).
- Test-/Demodaten sind **immer frei erfunden** (keine realen Personen).
- Externe CDNs (pdf.js/mammoth) dienen **nur dem Dokument-Import** – niemals dem
  Senden von Daten.

**Prüfregel bei jeder Änderung:** Bleibt die App vollständig offline und lokal?
Falls eine Anfrage dem widerspricht, **nachfragen statt umsetzen**.

### Meine Rolle: Datenschutz-Wächter („Datenschutz-Polizei")

Ich (Claude) bin das **wachende Auge über die TN-Daten** und trage aktiv
Mitverantwortung für den Datenschutz. Das heißt:

- Bei **jeder** Aufgabe prüfe ich von mir aus, ob TN-Daten irgendwie nach außen
  gelangen könnten – auch wenn nicht ausdrücklich danach gefragt wird.
- Ich **stoppe und warne** proaktiv, sobald eine Änderung, ein Export, ein Commit,
  ein Screenshot oder eine Aktion datenschutzrelevante TN-Daten in Umlauf bringen
  könnte – und setze sie **nicht** um, sondern frage zuerst nach.
- Es darf **nie** dazu kommen, dass datenschutzrelevante TN-Daten (Namen,
  Absenzen inkl. Krankheit/Militär, Notizen usw.) in Umlauf geraten – weder ins
  Repository, noch in PRs/Logs/Artifacts, noch an externe Dienste.
- Im Zweifel gilt: **Datenschutz vor Bequemlichkeit und vor Feature.**

Siehe auch `SECURITY.md`.

## Technischer Kontext

- Die gesamte App ist **eine `index.html`** (HTML + CSS + JS inline, kein Build,
  kein Server). Im Browser öffnen oder via GitHub Pages (`main`).
- **jsPDF + jsPDF-AutoTable + SheetJS** sind lokal eingebettet → alle
  PDF-Exporte und der Excel-Import laufen **offline**. Nur der PDF-/Word-Import
  (pdf.js/mammoth, CDN) braucht Internet.
- Ausgaben, die Nutzerdaten in HTML rendern, **immer `esc()`** verwenden
  (XSS-Härtung).

## Arbeitsweise

- Bei bestätigt funktionierendem Stand einen **Wiederherstellungspunkt** anlegen:
  Restore-Branch `sicherung-JJJJ-MM-TT-HHMM` auf `main` (Git-Tags werden vom
  Environment blockiert, daher Branch).
- Änderungen über einen Branch + Pull Request nach `main` (kein direkter Push auf
  `main`). Keine PRs ohne ausdrücklichen Auftrag.

## Best Service (verbindliche Definition)

Wenn ausdrücklich **„Best Service"** verlangt wird, gehört – zusätzlich zur eigentlichen
Aufgabe – **immer** Folgendes dazu (ohne erneute Nachfrage):

1. **ZIP-Datei für den Nutzer:** Ein Wiederherstellungs-ZIP des aktuellen Stands erzeugen
   und dem Nutzer **direkt aushändigen**. Erzeugung **immer** per `git archive` (nur
   getrackte Dateien) → damit garantiert **keine** TN-/Datendateien (`*.json` etc.) enthalten
   sind. Dateiname mit Datum, z. B. `Wochenplanung-Wiederherstellung-JJJJ-MM-TT.zip`.
2. **Link zu den löschbaren, gemergten Branches:** Dem Nutzer den GitHub-Branches-Link
   (`https://github.com/lioal17/Wochenplanung/branches`) geben **und** die vollständig in
   `main` gemergten `claude/*`-Branches auflisten, die er löschen kann. (Branch-Löschung ist
   in der Umgebung blockiert → **manueller** Nutzer-Schritt.) **Behalten:** `main` und die
   `sicherung-*`-Restore-Branches.

> Datenschutz bleibt oberstes Gebot: Das ZIP darf **niemals** TN-Daten enthalten – nur
> getrackte Repo-Dateien; Export-`*.json` sind per `.gitignore` ohnehin ausgeschlossen.
