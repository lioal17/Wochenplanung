# GitHub-Repository-Analyse – Stand 10.07.2026

> ⚠️ **Nur Empfehlungen.** Es wurden **keine** Branches gelöscht, keine PRs geschlossen,
> keine Repo-Einstellungen geändert. Umsetzung liegt bei dir.

Repository: **`github.com/lioal17/Wochenplanung`**
Standard-Branch: **`main`** (Tip `2fca922d3cd7645438cda888f26020edef94721b`)

---

## 1. Branch-Übersicht

| Branch | Tip-Commit | In `main` gemerged? | Empfehlung |
|---|---|---|---|
| `main` | `2fca922` | — (Standard) | **behalten** |
| `sicherung-2026-07-09-1935` | `0b7794f` | nein (älterer Stand) | **behalten** – bewusster Wiederherstellungspunkt (Projekt-Konvention laut `CLAUDE.md`) |
| `claude/ecase-button-sync-zazxy6` | `a4939be` | **ja** (PRs #142–#147) | **löschbar** (Arbeit ist in `main`) |
| `claude/wb-tn-layout-5pcdbu` | `d24a1e7` | **ja** | **löschbar** |
| `claude/serene-fermi-qrc682` | `007d15a` | **ja** | **löschbar** |
| `claude/layout-redesign-ui-only-p9j1lb` | `980d5f6` | **ja** | **löschbar** |

**Verifikation der Merge-Zugehörigkeit:** Für alle drei `claude/*`-Layout-/Feature-Branches
wurde `git merge-base --is-ancestor <branch> origin/main` bestätigt (= vollständig in `main`).
`claude/ecase-button-sync-zazxy6` ist über die Merge-Commits #142–#147 in `main` enthalten;
die Arbeitskopie ist **inhaltlich identisch** zu `origin/main` (Datei-Diff leer).

### Empfehlung Branch-Bereinigung *(optional)*
Die vier `claude/*`-Branches sind **abgeschlossen und gemerged** → können gefahrlos gelöscht
werden (nur Aufräumen der Branch-Liste; keine Auswirkung auf `main`). Beispiel:
```bash
git push origin --delete claude/ecase-button-sync-zazxy6
git push origin --delete claude/wb-tn-layout-5pcdbu
git push origin --delete claude/serene-fermi-qrc682
git push origin --delete claude/layout-redesign-ui-only-p9j1lb
```
> **`sicherung-2026-07-09-1935` NICHT löschen** – das ist ein absichtlicher Restore-Punkt.
> Tipp: Analog wäre ein Branch `sicherung-2026-07-10-…` als heutiger Restore-Punkt sinnvoll
> (siehe `00-README-WIEDERHERSTELLUNG.md`).

---

## 2. Pull Requests

- **Offene PRs:** **keine** (Stand der Analyse leer).
- Die zuletzt bearbeiteten PRs #142–#147 (eCase/WB-Änderungen) sind **gemerged**.
- **Empfehlung:** keine Aktion nötig.

---

## 3. Repo-Hygiene / technische Schulden

| Thema | Befund | Empfehlung |
|---|---|---|
| Build-Artefakte im Repo | **keine** (`_site/` ist in `.gitignore`) | ok |
| Versehentlich committete TN-Daten (`*.json`) | **keine** gefunden; `.gitignore` schützt | vor jedem Commit weiter prüfen |
| Alte Skripte/Legacy-Dateien | **keine** eigenständigen Skripte (App ist Single-File) | ok |
| Doku-Dubletten | mehrere `*ZUSAMMENFASSUNG*`/`ANALYSE-BERICHT` | optional archivieren (siehe `04-…` E1) |
| CI-Workflow | schlank, deployt nur `index.html`+`.nojekyll`; Action-Versionen aktuell (v3/v4) | ok |
| Branch-Schutz | `main` ist **nicht** protected | *(Empfehlung)* optional Branch-Protection für `main` aktivieren |
| Tags/Releases | **keine** Git-Tags (laut `CLAUDE.md` vom Environment blockiert → Restore per Branch) | Konvention beibehalten |

---

## 4. Zusammenfassung der GitHub-Empfehlungen

1. **Löschbar:** 4 gemergte `claude/*`-Branches.
2. **Behalten:** `main`, `sicherung-2026-07-09-1935`.
3. **Optional:** heutigen Restore-Branch `sicherung-2026-07-10-…` anlegen; Doku archivieren;
   Branch-Protection für `main` erwägen.
4. **Keine** offenen PRs, **keine** Build-Artefakte, **keine** TN-Daten im Repo → sauber.
