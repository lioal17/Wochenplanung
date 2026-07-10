# 🔒 Wiederherstellungspunkt (Recovery-Paket) – Stand 10.07.2026

> **Zweck:** Dieses Paket ist der **verbindliche Referenzstand (Snapshot)** des Projekts
> *Lernwerkstatt – Einteilungsplan (Wochenplanung)* zum **10.07.2026**.
> Wird dieses Paket zusammen mit dem Projekt später erneut übergeben, lässt sich der
> Zustand von heute **exakt und ohne Interpretationsspielraum** wiederherstellen.

---

## 0. Verbindlicher Snapshot – die harten Fakten

| Merkmal | Wert |
|---|---|
| Datum | **2026-07-10** (Commit-Zeit `2026-07-10 17:20:30 +0000`) |
| Kanonischer Stand | **`origin/main`** = Commit `2fca922d3cd7645438cda888f26020edef94721b` |
| Inhaltlich identischer Arbeitskopie-Commit | `a4939be3a575cfb976aed4d5cd3678a107075887` (in `main` gemerged) |
| **`index.html` SHA-256** | `6b6f6439b983bfa091ecb49ed1b86863ec788bb906fd7c8bab5fbe091cc56110` |
| `index.html` git-blob | `32fc494c871b69029aa5f7491ad2fbef2d1211dd` (1 496 228 Bytes, 4662 Zeilen) |
| Repository | `github.com/lioal17/Wochenplanung` |
| Live (GitHub Pages) | Deploy aus `main` via `.github/workflows/deploy-pages.yml` |

> **Wichtigster Satz des Pakets:** Die Datei **`index.html` IST die App** (HTML+CSS+JS inline,
> kein Build, kein Server). Wer eine `index.html` mit obigem SHA-256 besitzt, hat den
> heutigen Stand **1:1** – unabhängig von Git. Die restlichen Dokumente beschreiben diesen
> Stand so, dass er notfalls auch **ohne** die Datei rekonstruiert werden könnte.

---

## 1. Wiederherstellung – wie zurücksetzen?

### Weg A (empfohlen, 100 % exakt): über Git
```bash
git fetch origin
git checkout main
git reset --hard 2fca922d3cd7645438cda888f26020edef94721b
# Verifikation:
sha256sum index.html
# muss ergeben: 6b6f6439b983bfa091ecb49ed1b86863ec788bb906fd7c8bab5fbe091cc56110
```

### Weg B (ohne Git): Datei zurückspielen
Die im Projekt vorhandene `index.html` durch die Version mit dem oben genannten
**SHA-256** ersetzen. Prüfen mit `sha256sum index.html` (bzw. unter Windows
`certutil -hashfile index.html SHA256`). Stimmt der Hash, ist der Stand exakt hergestellt.

### Weg C (Rekonstruktion aus Dokumentation)
Falls nur dieses Paket vorliegt: Die App lässt sich anhand von
`01-PROJEKTZUSAMMENFASSUNG.md`, `02-LAYOUT-DESIGNSYSTEM.md` und `03-FEATURES.md`
(zusammen mit dem vorhandenen `PROJEKT.md` im Repo) als einzelne `index.html` neu aufbauen.
Externe Libs siehe Abhängigkeitsliste.

---

## 2. Was gehört NICHT zum Code-Snapshot: die Teilnehmer-Daten

> ⚠️ **Datenschutz (oberstes Gebot):** Teilnehmer-(TN-)Daten sind **absichtlich nicht**
> im Repository und **nicht** in diesem Paket. Sie liegen ausschließlich **lokal im
> Browser** (`localStorage`).

- Die App-Datei enthält **keine** Personendaten.
- Der Datenbestand wird **separat** gesichert: in der App **Export → JSON**.
- Wiederherstellung der Daten: in der App **Import** der JSON-Sicherung.
- localStorage-Schlüssel: siehe `01-PROJEKTZUSAMMENFASSUNG.md` §Persistenz.

**Ein vollständiger „Stand heute" besteht also aus zwei Teilen:**
1. **Code** = diese `index.html` (per SHA-256 verifizierbar) → dieses Paket.
2. **Daten** = deine private JSON-Export-Sicherung (bleibt bei dir, nie ins Repo).

---

## 3. Inhalt dieses Recovery-Ordners

| Datei | Inhalt |
|---|---|
| `00-README-WIEDERHERSTELLUNG.md` | **Dieses Dokument** – Snapshot-Fakten & Restore-Anleitung |
| `01-PROJEKTZUSAMMENFASSUNG.md` | Architektur, Technologien, Struktur, Datenmodell, Build, Deployment, Abhängigkeiten, Konfiguration |
| `02-LAYOUT-DESIGNSYSTEM.md` | Vollständige Layout-/Design-Dokumentation (Aufbau, Navigation, Farben, Komponenten, Responsiveness, UX) |
| `03-FEATURES.md` | Feature-für-Feature-Beschreibung inkl. der Änderungen vom 10.07.2026 |
| `04-DEAD-CODE-ANALYSE.md` | **Empfehlungen** zu ungenutztem Code / Bereinigung (nichts gelöscht) |
| `05-GITHUB-ANALYSE.md` | **Empfehlungen** zu Branches/PRs/Repo-Hygiene |
| `06-VERSIONS-SNAPSHOT.md` | Verifizierbarer Datei-/Hash-/Branch-Snapshot |

---

## 4. Wichtiger Hinweis zur Unveränderlichkeit

Dieses Paket wurde rein **analytisch** erstellt:
- **Keine** Code-Änderung, **kein** Refactoring, **keine** Löschung am Projekt.
- Alle Optimierungs-/Bereinigungs-Ideen sind ausdrücklich als **Empfehlung** markiert
  (`04-…` und `05-…`) und **nicht** umgesetzt.
- Das Hinzufügen dieses Dokumentations-Ordners ist der einzige Zusatz und berührt die
  App (`index.html`) nicht.
