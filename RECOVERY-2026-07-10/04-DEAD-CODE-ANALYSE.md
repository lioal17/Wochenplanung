# Dead-Code- & Bereinigungsanalyse – Stand 10.07.2026

> ⚠️ **Nur Empfehlungen.** Es wurde **nichts** gelöscht, geändert oder refactored.
> Diese Liste ist bewusst konservativ; jede Umsetzung sollte einzeln geprüft und über
> einen Branch + PR erfolgen. **Datenschutz-/Funktionsverhalten hat Vorrang vor Aufräumen.**

---

## 1. Methodik

- App-Code isoliert (Haupt-`<script>`, Zeilen 1465–4660, 3194 Zeilen) und von den
  **eingebetteten Minified-Libs** (SheetJS, jsPDF, AutoTable) getrennt analysiert.
- Für jede im App-Block definierte Funktion (142 `function`-Definitionen + 38 benannte
  Arrow-Funktionen = **180 Namen**) die Gesamt-Referenzen in `index.html` gezählt
  (inkl. `onclick="…"`-Strings im Markup).
- Zusätzlich Scan auf `TODO/FIXME/HACK/debugger/console.*` und offensichtliche Altlasten.

---

## 2. Ergebnis: Der App-Code ist sauber

| Prüfung | Ergebnis |
|---|---|
| Ungenutzte App-Funktionen (0 Referenzen) | **0** – jede der 180 Funktionen wird verwendet |
| `TODO` / `FIXME` / `HACK` / `XXX` | **0** im App-Block |
| `debugger`-Statements | **0** |
| `console.log` / `console.debug` | **0** |
| `console.warn` | **4** – alle **legitim** (Fehlerbehandlung in `loadDB`, `saveDB`, Migrationen) → **behalten** |

**Fazit:** Es gibt **keinen** eindeutig toten Funktions-/App-Code. Eine „Dead-Code-Löschung"
im engeren Sinn ist derzeit **nicht** notwendig.

> Hinweis: Die naive Zählung über die **gesamte** Datei meldet 13 scheinbar „ungenutzte"
> Namen (`Jw`, `Kw`, `Qw`, `Yw`, `Zw`, `ak`, `fk`, `hk`, `ik`, `nk`, `qw`, `rk`, `xg`).
> Diese stammen **ausnahmslos aus den minifizierten Fremd-Bibliotheken** und sind **KEIN**
> App-Code. **Nicht anfassen** – Teil der eingebetteten jsPDF/SheetJS-Builds.

---

## 3. Empfehlungen (optional, niedrige Priorität)

### E1 — Repo-Dokumentation konsolidieren *(Empfehlung, unkritisch)*
Es existieren mehrere historische Doku-Stände parallel:
- `PROJEKT-ZUSAMMENFASSUNG-2026-06-19.md`
- `PROJEKT-ZUSAMMENFASSUNG-2026-07-03.md`
- `ANALYSE-BERICHT-2026-07-05.md`

Diese sind **veraltet**, aber als Historie evtl. gewollt. **Empfehlung:** entweder belassen
(Archiv) oder in einen Ordner `docs/archiv/` verschieben, damit das Repo-Wurzelverzeichnis
übersichtlicher wird. **Nicht** ersatzlos löschen, solange `PROJEKT.md` nicht alle relevanten
Punkte übernommen hat. *(Kein Einfluss auf die App – werden nicht deployt.)*

### E2 — `PROJEKT.md` auf Stand 10.07.2026 nachziehen *(Empfehlung)*
`PROJEKT.md` trägt Stand **2026-07-03** und kennt die heutigen Änderungen noch nicht
(Input-AG-Montag, eCase-Global-Badges, Rapport-eCase-Kachel, WB-Reset-Button,
WB-Badge = nur rot, Feld `wbAgInformed`). **Empfehlung:** bei Gelegenheit ergänzen
(dieses Recovery-Paket dokumentiert die Deltas bereits vollständig).

### E3 — SheetJS-Version *(Sicherheits-Empfehlung, aus `SECURITY.md`)*
SheetJS 0.18.5 hat bekannte CVEs (CVE-2023-30533 / CVE-2024-22363). Upgrade nur über
`cdn.sheetjs.com` (nicht npm). **Als langfristige Empfehlung geführt** – im aktuellen
Offline-Einsatz (keine fremden Uploads von Dritten) geringes Restrisiko.

### E4 — Keine ungenutzten Assets/Konfigurationen gefunden
- **Assets:** Es gibt **keine** separaten Bild-/Font-/JS-/CSS-Dateien (alles inline, Icons =
  Emoji). → nichts zu bereinigen.
- **Konfigurationen:** `.gitignore`, `.nojekyll`, `deploy-pages.yml` sind aktiv und korrekt.
  Keine verwaisten Konfig-Dateien.
- **Testdateien:** Es gibt **keine** Testdateien/Testframeworks im Repo → nichts zu entfernen.

---

## 4. Ausdrücklich NICHT empfohlen

- ❌ **Keine** Umstrukturierung der Single-File-Architektur (die Ein-Datei-Bauweise ist ein
  bewusstes Feature: Portabilität, Offline, einfache Weitergabe).
- ❌ **Keine** Auslagerung/Minifizierung der eingebetteten Libs (würde Offline-Fähigkeit und
  Weitergabe verkomplizieren).
- ❌ **Keine** automatische CSS-„Purge"-Bereinigung ohne manuelle Prüfung – viele Klassen
  werden dynamisch in JS-Templates (`innerHTML`) gesetzt und sind für statische Tools
  schwer erkennbar (Gefahr von Fehl-Löschungen).
