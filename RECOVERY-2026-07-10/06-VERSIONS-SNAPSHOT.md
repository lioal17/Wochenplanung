# Versions-Snapshot (verifizierbar) – Stand 10.07.2026

> Dieser Snapshot macht den heutigen Stand **prüfbar**: Wer eine Datei besitzt, kann per
> Hash abgleichen, ob sie exakt dem 10.07.2026 entspricht.

---

## 1. Git-Referenzen

| Ref | Commit |
|---|---|
| `origin/main` (kanonisch) | `2fca922d3cd7645438cda888f26020edef94721b` |
| Arbeitskopie / gemergter Feature-Commit | `a4939be3a575cfb976aed4d5cd3678a107075887` |
| Commit-Zeit (HEAD) | `2026-07-10 17:20:30 +0000` |

`origin/main` und die Arbeitskopie sind **inhaltlich identisch** (`git diff HEAD origin/main` = leer).

---

## 2. Datei-Snapshot (getrackte Dateien, git-blob-Hashes)

| git-blob (SHA-1) | Bytes | Datei |
|---|---|---|
| `368436f8655d8abb90ba101b49656f571a642386` | 1080 | `.github/workflows/deploy-pages.yml` |
| `f22debce1c87970c3d9e3ef6bb01324a96d0c0e1` | 877 | `.gitignore` |
| `e69de29bb2d1d6434b8b29ae775ad8c2e48c5391` | 0 | `.nojekyll` |
| `3b47669615a7af4edc1b72802a7ecd7ad50d9e98` | 11523 | `ANALYSE-BERICHT-2026-07-05.md` |
| `df51afe635bb1c511424c6a47445fa456351d9ea` | 3254 | `CLAUDE.md` |
| `0de04e5142b1abcadbb197ba6bdbe5eda5a824b9` | 7242 | `PROJEKT-ZUSAMMENFASSUNG-2026-06-19.md` |
| `0af475eded7a7716a5a4204f2541007ee378f870` | 7190 | `PROJEKT-ZUSAMMENFASSUNG-2026-07-03.md` |
| `3fad8782ef908e82745f698a032e34273b9b04d5` | 18318 | `PROJEKT.md` |
| `2f86a843b50689312543472992c17bddf94fa01c` | 1191 | `README.md` |
| `3c42bdc7b3282e07c0d035716d1105d215f34c4e` | 3701 | `SECURITY.md` |
| `b9f15bf143931f82bc38e17f3e44aa7d231e0a8d` | 11650 | `UEBERGABE.md` |
| `32fc494c871b69029aa5f7491ad2fbef2d1211dd` | 1496228 | **`index.html`** |

---

## 3. Kern-Hash der App

**`index.html` SHA-256:**
```
6b6f6439b983bfa091ecb49ed1b86863ec788bb906fd7c8bab5fbe091cc56110
```

Prüfen:
```bash
# Linux/macOS
sha256sum index.html
# Windows (PowerShell)
Get-FileHash index.html -Algorithm SHA256
# Windows (cmd)
certutil -hashfile index.html SHA256
```
Stimmt der Hash → die App ist **bit-genau** der Stand vom 10.07.2026.

---

## 4. Kennzahlen der App-Datei

| Kennzahl | Wert |
|---|---|
| Zeilen gesamt | 4662 |
| Bytes | 1 496 228 |
| Inline-`<script>`-Blöcke | 4 (SheetJS · jsPDF · AutoTable · **App**) |
| App-Script-Block | Zeilen 1465–4660 (~3194 Zeilen) |
| Benannte App-Funktionen | 180 (142 `function` + 38 Arrow), **0 ungenutzt** |
| Eingebettete Libs | jsPDF 2.5.1 · jsPDF-AutoTable 3.8.2 · SheetJS 0.18.5 |
| CDN-Libs (Import) | pdf.js 3.11.174 · mammoth 1.6.0 (mit SRI) |

---

## 5. Branch-Zustand (GitHub) zum Snapshot-Zeitpunkt

| Branch | Tip | Status |
|---|---|---|
| `main` | `2fca922` | Standard, kanonisch |
| `sicherung-2026-07-09-1935` | `0b7794f` | Restore-Punkt (älter) – behalten |
| `claude/ecase-button-sync-zazxy6` | `a4939be` | gemerged – löschbar |
| `claude/wb-tn-layout-5pcdbu` | `d24a1e7` | gemerged – löschbar |
| `claude/serene-fermi-qrc682` | `007d15a` | gemerged – löschbar |
| `claude/layout-redesign-ui-only-p9j1lb` | `980d5f6` | gemerged – löschbar |

Offene PRs: **keine**.

---

## 6. Wiederherstellungs-Kurzform

```bash
git fetch origin
git checkout -B main 2fca922d3cd7645438cda888f26020edef94721b
sha256sum index.html   # == 6b6f6439…6110  → Stand exakt hergestellt
```
Daten (TN) separat über den JSON-Export/-Import der App – **nicht** Teil dieses Snapshots
(Datenschutz).
