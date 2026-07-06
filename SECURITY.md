# Sicherheit & Datenschutz

Dieses Tool ist eine reine **Offline-/Local-First-Webapp** (eine einzige
`index.html`). Es gibt keinen Server, keine Benutzerkonten, kein Tracking und
keine Telemetrie. Alle Daten bleiben lokal im Browser.

## Wo liegen die Daten?

- **`localStorage['lw_db_v3']`** – die gesamte Datenbank (Teilnehmende,
  Einteilungen, Notizen) als **unverschlüsseltes** JSON.
- **`localStorage['lw_lastsave']`** – Zeitstempel der letzten Sicherung.
- **IndexedDB `lw_fh`** – optionaler Handle auf den gewählten Speicherordner
  (File System Access API).

> **Wichtig:** Die gespeicherten Daten enthalten personenbezogene und teils
> besondere Personendaten (z. B. Krankheits-/Militärabsenzen). Sie liegen
> **unverschlüsselt** im Browserprofil. Auf **geteilten Rechnern** nach der
> Nutzung die **Browserdaten löschen** (Browser-Einstellungen → Websitedaten für
> die Seite entfernen) – das löscht `localStorage` und die IndexedDB vollständig.

## Datensicherungen niemals committen

Die per Export erzeugten `*.json`-Dateien enthalten Personendaten und dürfen
**nicht** ins Repository gelangen. Die `.gitignore` schützt davor; bitte
trotzdem vor jedem Commit prüfen.

## Getroffene Schutzmaßnahmen (Stand dieser Analyse)

- **XSS-Härtung:** Alle Namen/Freitexte/IDs werden vor der Ausgabe HTML-escaped
  (`esc()`), inkl. einfacher Anführungszeichen in `onclick`-Handlern.
- **Import-Validierung:** `sanitizeDB()` bereinigt geladene/importierte Daten
  strukturell (Feld-Whitelist, Typkoersion, ID-Format) – auch beim Laden aus
  `localStorage`.
- **PDF-Import:** `pdf.js` läuft mit `isEvalSupported:false` (entschärft
  CVE-2024-4367 beim Öffnen fremder PDFs).
- **Content-Security-Policy:** restriktive CSP (`default-src 'none'`), Skripte
  nur von den zwei genutzten CDNs; verhindert Nachladen fremder Skripte und
  Daten-Exfiltration.

## Subresource-Integrity (SRI) für die CDN-Skripte — umgesetzt

Die zwei per CDN geladenen Bibliotheken (`pdf.js` via cdnjs, `mammoth` via
jsDelivr; nur für den PDF-/Word-Import nötig) tragen jetzt **SRI-Hashes**
(`integrity="sha384-…" crossorigin="anonymous"`). Damit lädt der Browser die
Skripte nur, wenn ihr Inhalt exakt dem erwarteten Stand entspricht — Schutz
gegen eine CDN-Kompromittierung oder MITM.

Die Hashes wurden aus den **identischen npm-Artefakten** berechnet
(`pdfjs-dist@3.11.174/build/pdf.min.js`, `mammoth@1.6.0/mammoth.browser.min.js`),
die beide CDNs unverändert ausliefern.

> Bei einem künftigen **Versionswechsel** einer der Libs müssen die Hashes neu
> berechnet werden (sonst blockiert der Browser das Skript):
> ```bash
> curl -s <CDN-URL> | openssl dgst -sha384 -binary | openssl base64 -A
> # oder aus npm:  npm pack <paket>@<version> && tar -xzf … && openssl dgst -sha384 -binary <datei> | openssl base64 -A
> ```

> Hinweis: Der PDF.js-**Worker** (`pdf.worker.min.js`) wird intern per
> `importScripts` geladen und lässt sich nicht per SRI absichern; hier greifen
> die CSP-`worker-src`-Beschränkung und `isEvalSupported:false`.

## Eingebettete Bibliotheken (Lizenzen & bekannte CVEs)

| Bibliothek | Version | Lizenz | Hinweis |
|-----------|---------|--------|---------|
| jsPDF + AutoTable | 2.5.1 / 3.8.2 | MIT | CVE-2025-29907 (ReDoS via `addImage`) – hier nicht ausnutzbar, da keine fremden Bild-URLs. |
| SheetJS (xlsx) | 0.18.5 | Apache-2.0 | CVE-2023-30533 / CVE-2024-22363 – Upgrade nur über `cdn.sheetjs.com` (nicht npm); als langfristige Empfehlung geführt. |

## Eine Schwachstelle melden

Da es sich um ein internes Tool ohne öffentliche Angriffsfläche (kein Server)
handelt: Auffälligkeiten bitte direkt der/dem Projektverantwortlichen melden.
