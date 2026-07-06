# Vollständige Projektanalyse & Optimierung — Abschlussbericht

**Projekt:** Lernwerkstatt Einteilungsplan (Wochenplanung)
**Datum:** 2026-07-05 · **Branch:** `claude/project-analysis-optimization-w4ayfl`
**Umfang:** Single-File-Webapp `index.html` (≈1,48 MB; App-Code ≈133 KB, Rest eingebettete Bibliotheken)

---

## 1. Zusammenfassung

Das Projekt ist ein durchdachtes, funktional vollständiges **Offline-First-Tool** ohne
Server, ohne Tracking, ohne Cookies. Die bewusste Single-File-Architektur (eine `index.html`,
lokal per `localStorage`) ist für den Einsatzzweck (Weitergabe, Offline-Betrieb) sinnvoll und
wurde **beibehalten**. Die Domänenlogik (Blocks, Feiertage, Rapporte) ist sorgfältig
kommentiert.

Die Analyse fand jedoch einen **realen Stored/DOM-XSS-Pfad**, veraltete CDN-Bibliotheken mit
bekannten CVEs, fehlende Eingabevalidierung beim Import, keine Möglichkeit zum vollständigen
Löschen personenbezogener Daten sowie mehrere Wartbarkeits-/Performance-Schwächen.

**Von den identifizierten Problemen wurden alle risikoarm behebbaren direkt umgesetzt**
(10 Commits), jede Änderung durch einen Browser-Test bzw. einen **byte-genauen
Golden-Master-Snapshot** (alle Ansichten + 5 PDF-Exporte) als verhaltensneutral abgesichert.

**Wichtiger Befund vorab — Teilnehmerdaten:** Die gesamte Git-Historie (alle Branches) enthält
**keine** Teilnehmer-Datendateien; Personendaten liegen ausschließlich lokal im Browser. Im
Quelltext stehen jedoch **drei echte Mitarbeiter-/Beraternamen** (Import-Erkennung, `index.html`)
und drei interne E-Mail-Adressen — öffentlich einsehbar (siehe §4).

---

## 2. Kritische Probleme

Keine „kritischen" (unmittelbar aus dem Netz ausnutzbaren) Lücken — die App hat keinen Server
und keine aktive Netzwerkkommunikation. Das schwerste Problem war **hoch** eingestuft (XSS,
siehe §3.1) und ist behoben.

---

## 3. Sicherheitsprobleme

| # | Befund | Schwere | Status |
|---|--------|---------|--------|
| S1 | **Stored/DOM-XSS** über Teilnehmer-Namen & -IDs | **Hoch** | ✅ behoben |
| S2 | **CVE-2024-4367** (pdf.js 3.11.174, JS-Ausführung via präpariertem PDF) | **Hoch** | ✅ entschärft |
| S3 | Import übernahm Daten **ohne Validierung** | **Hoch** | ✅ behoben |
| S4 | CDN-Skripte ohne CSP/SRI | **Mittel** | ✅ CSP / ⚠ SRI dokumentiert |
| S5 | veraltete eingebettete Libs (xlsx 0.18.5, jsPDF 2.5.1) | **Mittel** | 📋 Empfehlung |
| S6 | `saveDB` ohne Quota-Fehlerbehandlung | **Niedrig** | ✅ behoben |

**S1 — XSS (behoben, Commit `180ffff` + `749fbf0`-Snapshot).** `esc()` maskierte kein `'`;
Namen/IDs landeten unescaped in `innerHTML` und `onclick`-Handlern (u. a. Tagesplan,
Teilnehmerliste, Monatsübersicht, eCase-Listen). Ein Name wie `<img src=x onerror=…>` oder
eine ID `x'),alert(1),('` (z. B. via manipuliertem JSON-Import) wurde beim Rendern ausgeführt.
→ *Fix:* `esc()` maskiert nun auch `'`→`&#39;` und wird an **allen** Namen-/ID-Sinks angewandt.
Ein Browser-Regressionstest mit XSS-Payloads bestätigt: kein Skript wird ausgeführt, der Name
erscheint als Text. Der Test fand dabei eine in der statischen Analyse übersehene Sink-Stelle
(Namensspalte Monatsübersicht) — ohne Test wäre sie offen geblieben.

**S2 — pdf.js CVE-2024-4367 (entschärft, Commit `42e95ff`).** `getDocument()` läuft nun mit
`isEvalSupported:false`; damit kann ein präpariertes PDF beim Import keine JS-Ausführung mehr
über die Font-Matrix auslösen. Vollständiger Fix wäre ein Upgrade auf pdf.js ≥ 4.2.67 (siehe §11).

**S3 — Import-Validierung (behoben, Commit `671d86e`).** Neue Funktion `sanitizeDB()`
rekonstruiert geladene/importierte Daten auf eine **Feld-Whitelist** mit Typkoersion; ungültige
IDs werden auf `uid()`-Format normalisiert und in `plans`/`rptNotes` umgeschlüsselt (Altdaten
bleiben erhalten). Eingehängt in `loadDB` **und** `processImportJSON`. 15 Testfälle grün.

**S4 — CSP (behoben, Commit `663f0b1`).** Restriktive `Content-Security-Policy`
(`default-src 'none'`), Skripte nur von den zwei genutzten CDNs; verhindert Nachladen fremder
Skripte und Daten-Exfiltration (fetch/XHR/WebSocket/Bild-Beacon). Getestet: Rendering, Styles,
Favicon, jsPDF-Erzeugung, Blob-Download — **0 Verstöße**. **SRI** konnte in dieser Umgebung
nicht verifiziert berechnet werden (Egress-Policy blockiert die CDNs); ein blind gesetzter Hash
würde die App bei Fehler brechen. → als exakt dokumentierter Handschritt in `SECURITY.md`.

---

## 4. Datenschutzprobleme

- **Unverschlüsselte lokale Speicherung** besonderer Personendaten (Krankheits-/Militärabsenzen)
  in `localStorage`. Bisher **keine Löschfunktion**. → ✅ **behoben:** Funktion **„🗑 Daten löschen"**
  (`wipeAllData`, Commit `42e95ff`) entfernt `localStorage` **und** IndexedDB nach zwei
  Bestätigungen; Hinweis in `README.md`/`SECURITY.md`.
- **Öffentliche Pages-Auslieferung interner Doku.** Der Workflow veröffentlichte das *gesamte*
  Repo → `UEBERGABE.md` etc. waren öffentlich abrufbar. → ✅ **behoben** (Commit `7eb0009`): Deploy
  liefert nur noch `index.html` + `.nojekyll`.
- **Personendaten im Quelltext:** drei reale Mitarbeiter-/Beraternamen (Import-Erkennung) und drei
  interne E-Mail-Adressen (`…@basisjob.ch`). → 📋 **Empfehlung** (verhaltensändernd, nicht
  unaufgefordert umgesetzt): Berater-Erkennung konfigurierbar auslagern.
- **JSON-Exporte mit Personendaten** könnten versehentlich committet werden. → ✅ **behoben**:
  `.gitignore` (Commit `7eb0009`).
- **Positiv:** kein Tracking, keine Telemetrie, kein `fetch`/XHR im App-Code, keine Cookies.

---

## 5. Performanceprobleme

- **Feiertags-/Ferienberechnung ohne Cache** (Hauptlast): `computeFerien`/`easterSunday` wurden
  pro Teilnehmer×Tag-Zelle über `getBlocks`→`isSchulferien` neu berechnet. Messung: bei 25 TN
  **650 Aufrufe** von `computeFerien` pro Monats-Render. → ✅ **behoben** (Commit `62384fb`):
  Jahres-Memoisierung → nur noch **2 echte Berechnungen**. Verhaltensneutral (Golden-Master).
- **Voll-Serialisierung bei jeder Änderung** (`saveDB` schreibt die ganze DB): akzeptabel für die
  Datenmenge; Textfelder nutzen `onchange` (nicht pro Tastendruck). → 📋 optional: Debounce für
  Wochencode-Batches.
- **Voll-Neurender pro Änderung** via einmaligem `innerHTML`: für die Datenmenge in Ordnung, kein
  Layout-Thrashing. Nicht geändert.

---

## 6. Dead Code (entfernt, Commit `083de0c`)

- **12 ungenutzte CSS-Klassen** (`btn-xs`, `divider`, `leg-dot`, `mo-vm`, `mo-nm`, `mo-vm-sep`,
  `moc-empty`, `ra-ua`, `rpt-ua`, `row-absent`, `tag-fix`, `wknd`) + nie ausgelöste `.ws-sel:disabled`.
  Jede einzeln als „nur in Definition, nirgends genutzt" verifiziert.
- **`SCH_LABELS={}`** (leeres Objekt) samt drei Fallback-Lookups.
- **Toter Parameter `disabled`** in `buildWsCell` (alle 4 Aufrufe übergaben `false`).
- **Doppelter Kommentar** in `migrateSchnupper`.
- Kein auskommentierter Code, keine doppelten Funktionen, keine Debug-Reste gefunden.

---

## 7. Architektur- & Codequalitätsprobleme

- **Starke Duplikation** (behoben, Commits `749fbf0`, `b981503`): Halbtags-Absenz-Mengen 6× →
  3 zentrale Konstanten (`SOFT_ABS_SET`/`HALF_ABS_SET`/`HALF_ABS_BASE`); PDF-Fußzeilen 3× →
  ein `pdfPageFooters`-Helper; Sortier-Comparator 8× → `byName`. Die bewusst unterschiedlichen
  Bottom-Prädikate und das eigenständige `NO_ECASE`-Set blieben korrekt getrennt.
- **Sehr lange Funktionen** (`genWeekPDF` ~314 Z., `renderDayPlan` ~213 Z.): vermischen
  Datenaufbereitung und Layout. → 📋 Empfehlung: nicht ohne Testnetz zerlegen (jetzt vorhanden).
- **Globaler Namespace / Inline-`onclick`**: architekturbedingt (kein Build); vertretbar.
- **Fehlende Enums** für Status-/Absenz-Codes (Magic Strings verstreut). → 📋 Empfehlung.

---

## 8. Verbesserungsvorschläge (umgesetzt)

Alle in §3–§7 mit ✅ markierten Punkte wurden umgesetzt und einzeln getestet — 10 Commits,
jeweils verhaltensneutral belegt (Browser-Test + Golden-Master über alle Ansichten und 5 PDFs).

---

## 9. Sofort umsetzbare Änderungen (durch dich, ohne Code)

1. **Pull Request mergen** (GitHub → „Compare & pull request" für den Branch → Merge).
2. **SRI-Hashes ergänzen** (5 Minuten, Anleitung in `SECURITY.md`) — schließt die letzte
   CDN-Härtungslücke.
3. Prüfen, ob die drei realen Namen/E-Mails im Quelltext öffentlich stehen dürfen.

---

## 10. Langfristige Empfehlungen

- **Bibliotheken aktualisieren:** pdf.js → ≥ 4.2.67 (CVE-2024-4367 vollständig), xlsx → 0.20.x
  (via `cdn.sheetjs.com`; CVE-2023-30533/‑22363). Regressionsrisiko im PDF-Layout → mit dem
  jetzt vorhandenen Golden-Master-Testnetz gut absicherbar.
- **Automatisiertes Testnetz dauerhaft einchecken** (die in dieser Analyse gebauten
  Snapshot-/XSS-Tests) und in einen CI-Workflow hängen.
- **Berater-/Jobcoach-Daten** aus dem Quelltext in eine Konfiguration auslagern.
- Optional: lange PDF-Funktionen modular aufteilen; Debounce für Batch-Speichervorgänge.

---

## 11. Qualitätsbewertung (0–10, Stand nach Umsetzung)

| Kategorie | Score | Begründung |
|-----------|:----:|-----------|
| **Architektur** | 7 | Single-File bewusst & passend; klare Blockgliederung. Abzug: sehr lange Funktionen, globaler Scope. |
| **Sicherheit** | 8 | XSS geschlossen, Import validiert, CSP + pdf.js-Härtung. Abzug: SRI noch manuell, Libs veraltet. |
| **Datenschutz** | 8 | Kein Tracking, Löschfunktion neu, Pages-Deploy eingegrenzt, `.gitignore`. Abzug: unverschlüsselter localStorage, Namen im Code. |
| **Performance** | 8 | Feiertags-Memoisierung eliminiert die Hauptlast; Render/Save für die Datenmenge angemessen. |
| **Codequalität** | 8 | Nach Dedup deutlich DRY-er, Dead Code entfernt. Abzug: Magic Strings. |
| **Wartbarkeit** | 8 | Gute Kommentare, zentrale Helfer, jetzt Testnetz. Abzug: Funktionslänge. |
| **Skalierbarkeit** | 6 | Für die Zielgröße (Dutzende TN) gut; Voll-Serialisierung/-Render begrenzen sehr große Datenmengen. |
| **Testabdeckung** | 4 | Vor der Analyse **keine** Tests; jetzt Snapshot-/XSS-/Sanitize-Tests vorhanden, aber noch nicht als CI eingecheckt. |
| **Dokumentation** | 9 | Sehr ausführlich (PROJEKT/UEBERGABE); Drift korrigiert, neue Features + SECURITY.md ergänzt. |
| **Lesbarkeit** | 8 | Klare Namen & Kommentare; Abzug: dichte Ternary-Ketten in Monats-/PDF-Zellen. |
| **GitHub-Qualität** | 7 | Saubere Commit-/PR-Historie, minimale CI-Rechte. Neu: `.gitignore`, `SECURITY.md`, eingegrenzter Deploy. Abzug: kein LICENSE/Dependabot/CODEOWNERS. |

**Gesamteindruck:** solides, gepflegtes Fachtool; nach dieser Runde in Sicherheit, Datenschutz
und Wartbarkeit deutlich robuster.

---

## 12. Prioritätenliste

**🔴 Kritisch:** — (keine offen)

**🟠 Hoch:**
- ✅ XSS-Härtung · ✅ Import-Validierung · ✅ pdf.js-Härtung *(alle erledigt)*
- ⏳ SRI-Hashes ergänzen (manuell, `SECURITY.md`)

**🟡 Mittel:**
- ✅ CSP · ✅ Datenlöschfunktion · ✅ Pages-Deploy eingrenzen *(erledigt)*
- 📋 Libs aktualisieren (pdf.js/xlsx) · 📋 Namen/E-Mails aus Code auslagern

**🟢 Niedrig:**
- ✅ saveDB-Robustheit · ✅ Dedup · ✅ Dead Code · ✅ Memoisierung · ✅ Doku-Korrekturen *(erledigt)*
- 📋 Magic Strings → Enums · 📋 lange Funktionen aufteilen · 📋 Tests als CI einchecken

---

*Erstellt im Rahmen der beauftragten Vollanalyse. Jede umgesetzte Änderung ist als separater,
einzeln getesteter Commit nachvollziehbar; Verhaltensneutralität der Refactorings wurde per
byte-genauem Golden-Master-Vergleich (alle Ansichten + 5 PDF-Exporte) belegt.*
