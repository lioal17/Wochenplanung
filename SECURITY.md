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
> **unverschlüsselt** im Browserprofil. Auf **geteilten Rechnern** die Funktion
> **„🗑 Daten löschen"** verwenden – sie entfernt `localStorage` **und** die
> IndexedDB vollständig.

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

## Offene, manuell umzusetzende Härtung: SRI für die CDN-Skripte

Zwei Bibliotheken werden weiterhin per CDN geladen (nur für den
PDF-/Word-Import nötig): `pdf.js` (cdnjs) und `mammoth` (jsDelivr). Zur
Absicherung gegen eine CDN-Kompromittierung sollten sie **Subresource-Integrity
(SRI)**-Hashes erhalten. Die Hashes lokal berechnen (nicht ratbar) und ergänzen:

```bash
curl -s https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js \
  | openssl dgst -sha384 -binary | openssl base64 -A
curl -s https://cdn.jsdelivr.net/npm/mammoth@1.6.0/mammoth.browser.min.js \
  | openssl dgst -sha384 -binary | openssl base64 -A
```

Dann in `index.html` an den beiden `<script src=…>`-Tags ergänzen:

```html
<script src="…pdf.min.js" integrity="sha384-<HASH>" crossorigin="anonymous"></script>
<script src="…mammoth.browser.min.js" integrity="sha384-<HASH>" crossorigin="anonymous"></script>
```

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
