# Finalisierung & Repo-Aufräumung – 10.07.2026

> Abschlussdokument. Hält den **finalen, aufgeräumten** Zustand fest, damit jederzeit
> eindeutig ist, wie „Stand 10.07.2026" wiederhergestellt wird.

---

## 1. Der eine verbindliche Wiederherstellungspunkt

| Was | Wert |
|---|---|
| **Restore-Branch (Snapshot)** | **`sicherung-2026-07-10-1755`** |
| Standard-Branch | `main` (enthält das komplette Recovery-Paket) |
| **`index.html` SHA-256 (immutabler Anker)** | `6b6f6439b983bfa091ecb49ed1b86863ec788bb906fd7c8bab5fbe091cc56110` |

> **Merke:** Der Restore-Branch `sicherung-2026-07-10-1755` ist der **eingefrorene**
> Referenzstand. Er wird nie weiterentwickelt und zeigt für immer auf den heutigen Zustand.

### Wiederherstellen (Code) in einem Schritt
```bash
git fetch origin
git checkout -B main origin/sicherung-2026-07-10-1755
sha256sum index.html   # muss 6b6f6439…6110 ergeben  → exakt der Stand vom 10.07.2026
```
Alternativ (ohne Git): eine `index.html` mit obigem SHA-256 verwenden → bit-genau identisch.

---

## 2. Aufgeräumter Branch-Zustand (nach Finalisierung)

| Branch | Zweck | Status |
|---|---|---|
| `main` | aktueller Hauptstand (inkl. Recovery-Paket) | aktiv |
| **`sicherung-2026-07-10-1755`** | **Wiederherstellungspunkt HEUTE** | eingefroren |
| `sicherung-2026-07-09-1935` | Wiederherstellungspunkt Vortag | eingefroren |

**Entfernt** (waren vollständig in `main` gemerged, daher gefahrlos gelöscht):
- `claude/ecase-button-sync-zazxy6`
- `claude/wb-tn-layout-5pcdbu`
- `claude/serene-fermi-qrc682`
- `claude/layout-redesign-ui-only-p9j1lb`

Offene Pull Requests: **keine**.

---

## 3. Was zum vollständigen „Stand heute" gehört

Ein kompletter Rückspiel-Stand besteht aus **zwei** Teilen:

1. **Code** = dieser Restore-Branch / `index.html` (per SHA-256 verifizierbar). ✅ im Repo gesichert.
2. **Teilnehmer-Daten** = deine private **JSON-Datensicherung** aus der App
   (Button **`💾 Speichern`**). Diese liegt **ausschließlich bei dir lokal**, niemals im
   Repository (Datenschutz). Rückspielen über **Import** in der App.

> Ohne Teil 2 wird die App auf den heutigen **Code**-Stand zurückgesetzt, startet aber mit
> den Daten, die dann im Browser vorliegen. Für „alles wie jetzt" brauchst du **beide** Teile.

---

## 4. Bestätigung: rein additive Analyse

- **Keine** Änderung an `index.html`, **kein** Refactoring, **keine** Löschung von App-Code.
- Ergänzt wurde ausschließlich der Ordner `RECOVERY-2026-07-10/` (Dokumentation).
- Branch-Löschungen betreffen nur **gemergte** Feature-Branches (Inhalt bleibt in `main`).
- Dead-Code-Analyse: **0** ungenutzte App-Funktionen (siehe `04-DEAD-CODE-ANALYSE.md`).
