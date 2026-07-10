# Layout- & Designsystem-Dokumentation – Stand 10.07.2026

> Ziel: das aktuelle Layout so beschreiben, dass es **exakt rekonstruiert** werden kann.
> Alle Werte sind direkt aus `index.html` (Commit `2fca922…`) extrahiert.

---

## 1. Seitenaufbau (Grundgerüst)

```
┌──────────────────────────────────────────────────────────────┐
│  HEADER / TOP-NAVIGATION (nav-btn Reihe)                       │
│  📅 Wochenplan  📊 Monatsübersicht  📄 Monatsrapport           │
│  👥 Teilnehmer  📥 Dokument-Import           …  📅 WB-TN (→)   │
├──────────────────────────────────────────────────────────────┤
│  MAIN (.main; bei Monatsübersicht .main-wide)                  │
│    genau EINE .view.active wird angezeigt (.view{display:none})│
│    → view-plan | view-monthly | view-rapport                  │
│      view-person | view-import | view-wbtn                    │
├──────────────────────────────────────────────────────────────┤
│  Overlays: Modals (.modal), Toaststapel (#toastDock)          │
└──────────────────────────────────────────────────────────────┘
```

- **View-Umschaltung:** `.view{display:none}` / `.view.active{display:block}`; `show(name)`
  entfernt `active` von allen Views/Nav-Buttons, setzt es beim Ziel und ruft den passenden
  Renderer (`renderPlan`, `renderMonthly`, `renderRapport`, `renderPersons`, `renderWBTN`).
- **Breiten-Sonderfall:** In der Monatsübersicht bekommt `.main` die Klasse `main-wide`
  (breitere Matrix-Darstellung).

---

## 2. Navigation & Menüs

| Reihenfolge | Label | `data-v` / Aktion | Besonderheit |
|---|---|---|---|
| 1 | 📅 Wochenplan | `plan` | Startansicht (`view-plan` ist initial `active`) |
| 2 | 📊 Monatsübersicht | `monthly` | schaltet `.main` auf `main-wide` |
| 3 | 📄 Monatsrapport | `rapport` | |
| 4 | 👥 Teilnehmer | `person` | enthält „+ Teilnehmer hinzufügen" |
| 5 | 📥 Dokument-Import | `import` | |
| 6 | 📅 WB-TN | `wbtn` | `nav-wbtn`: `margin-left:auto` (rechts abgesetzt), Farbe `#b45309`, fett; **Zahlen-Badge** (`.n-badge`) |

- Aktiver Button: `.nav-btn.active`.
- Zusätzliche Direkt-Exporte (Neophyt-PDF, TN-Statistik-PDF) als Buttons.

---

## 3. Designsystem – Farben (CSS `:root`-Tokens)

### Akzente / Semantik
| Token | Wert | Verwendung |
|---|---|---|
| `--navy` | `#141414` | dunkle Grundfarbe/Text |
| `--green` | `#059669` | Erfolg / erledigt / „✓" |
| `--red` | `#c0392b` | Fehler / Absenz / rote Ampel |
| `--amber` | `#d97706` | Warnung / offen / gelbe Ampel / eCase-Badge |
| `--blue` | `#2e6db4` | Info / bezahlte Absenz |
| `--purple` | `#7c3aed` | Individual-/„Anders"-Code (violett) |
| `--teal` | `#0d9488` | Akzent |
| `--accent` | `#ffed00` | Signalgelb (Highlight) |
| `--zebra` | `#daeef3` | Zebra-/Tabellen-Hintergrund |

### Graustufen-Skala
| Token | Wert | | Token | Wert |
|---|---|---|---|---|
| `--g0` | `#ffffff` | | `--g5` | `#94a3b8` |
| `--g1` | `#f7f7f7` | | `--g6` | `#666666` |
| `--g2` | `#f4f4f4` | | `--g7` | `#475569` |
| `--g3` | `#e2e2e2` | | `--g8` | `#333333` |
| `--g4` | `#cbd5e1` | | `--g9` | `#141414` |

### Form-Tokens
| Token | Wert |
|---|---|
| `--r` (Radius) | `8px` |
| `--sh1` (Schatten klein) | `0 1px 3px rgba(0,0,0,.1),0 1px 2px rgba(0,0,0,.06)` |
| `--sh2` (Schatten mittel) | `0 4px 6px rgba(0,0,0,.07),0 2px 4px rgba(0,0,0,.05)` |
| `--sh3` (Schatten groß) | `0 10px 25px rgba(0,0,0,.12),0 4px 6px rgba(0,0,0,.05)` |

> Ampel-Sonderfarben WB-TN: grün `#15803d`, gelb `#b45309`, rot `var(--red)`;
> Karten-Hintergründe `--red-bg` / `--amber-bg` (helle Tönungen).

---

## 4. Typografie

- **Schriftfamilie (global):** `'Segoe UI', system-ui, -apple-system, sans-serif`.
  (Systemschriften, **keine** Web-Fonts geladen → offline-konform, keine externen Requests.)
- Buttons/Inputs erben teils `font-family:inherit`.
- Größen: relative/px-Werte je Komponente (Karten-Titel ~15 px, Kennzahlen groß/fett,
  Tabellen 11–13 px, Hinweistexte 11–12.5 px).

---

## 5. Icons

- **Ausschließlich Emoji** als Icons (📅 📊 📄 👥 📥 🌱 ⚠️ ✓ ↩ 🟢 🟡 🔴 📁).
  Keine Icon-Font, kein SVG-Sprite → keine externen Assets, voll offline.
- Statussymbole: „✓" (erledigt, `--green`), „!" (eCase ausstehend), „X" (Tag außerhalb Einsatz).

---

## 6. Kernkomponenten (CSS-Klassen)

| Komponente | Klasse(n) | Beschreibung |
|---|---|---|
| Navigations-Button | `.nav-btn`, `.nav-btn.active`, `.nav-wbtn` | Top-Navigation |
| Zahlen-Badge | `.n-badge` | kleiner Zähler am Nav-Button (WB-TN) |
| Buttons | `.btn` + `.btn-primary`/`.btn-secondary`/`.btn-success`/`.btn-danger`/`.btn-blue`/`.btn-icon`/`.btn-sm` | Aktionsknöpfe |
| Karte | `.card` | Container mit Rahmen/Schatten |
| Wochenplan-Tabelle | `.plan-tbl`, `.plan-tbl-wrap`, `.tbl-wrap` | Haupttabelle Tagesplan |
| Monats-Matrix | `.mo-tbl`, `.mo-wrap`, Zellen `.moc-abs`/`.moc-work`/`.moc-sch`/`.moc-prk`/`.moc-indiv`/`.mo-x` | Monatsübersicht |
| Tagesleiste | `.day-btn`, `.day-btn.active`, `.day-btn.today` | Mo–Fr Auswahl im Wochenplan |
| eCase-Badge | `.warn-bar`, `.ec-badge-btn`, `.ec-badge-wrap`, `.ec-drop`, `.ec-wrap`, `.ec-ok`, `.ec-open`, `.ec-cb` | Badge-Button + aufklappbare Liste |
| Rapport-Kennzahl | `.rpt-stats`, `.rpt-stat` (`.val`/`.lbl`) | Stat-Kacheln (bez./unbez. Absenz, KR, ZS, eCase) |
| WB-Karte | `.wb-card` (+ `.wbr-red`/`.wbr-yellow`/`.wbr-green`), `.wb-card-head`, `.wb-card-grid`, `.wb-field`, `.wb-card-status`, `.wb-card-actions`, `.wb-round`, `.wb-pill` (`.wb-green`/`.wb-yellow`/`.wb-red`), `.wb-btn-ag` | Wirkungsbericht-Karten |
| Summen-Chips | `.sum-chip`, `.plan-summary-split`, `.sum-slot` | VM/NM-Auswertung unter Wochenplan |
| Chips/Badges | `.chip` | Status-Kennzeichnungen an Namen |
| Legende | `.legend-box`, `.mo-legend-full`, `.lg-grp`, `.lg-row`, `.lg-i` | Farb-/Code-Legende |
| Modal | `.modal` | Dialoge (z. B. Teilnehmer-Rapport) |
| Toast | `.toast`, `#toastDock` | kurze Rückmeldungen |

### 6.1 WB-Runden-Button (Stand 10.07.2026)
`.wb-round` ist ein **`<button>`** (nicht mehr nur ein Label):
```css
.wb-round{display:inline-block;text-align:center;font-weight:700;
  background:var(--g2);border-radius:10px;padding:2px 10px;font-size:12px;
  border:none;cursor:pointer;color:inherit;font-family:inherit;transition:background .12s}
.wb-round:hover{background:var(--amber-bg,#fde68a)}
```
Klick → `wbResetRound(id)` (Zyklus auf Runde 1 zurück, mit Sicherheitsabfrage).

---

## 7. Abstände & Layout-Prinzipien

- **Radius:** durchgängig `--r` (8 px), Chips/Badges teils 10 px.
- **Schatten:** dreistufig (`--sh1`/`--sh2`/`--sh3`) je nach Elevation.
- **Layout-Technik:** Flexbox (Nav, Karten-Kopf, Chip-Reihen) + CSS-Grid
  (`.wb-card-grid`, Kennzahl-Reihen). Tabellen für Plan-/Matrix-Ansichten.
- **Tabellen:** horizontal scrollbar in `.tbl-wrap`/`.mo-wrap`/`.plan-tbl-wrap` (Overflow),
  damit breite Matrizen die Seite nicht sprengen; Wochentrennung per `.mo-sep`.

---

## 8. Animationen

- **`@keyframes toast-in`** – einzige Keyframe-Animation: Einblenden der Toasts (`#toastDock`).
- **`transition`** – 7 Vorkommen (u. a. `.wb-round:hover` `background .12s`, Button-/Hover-States).
- Bewusst **sparsam** gehalten (kein aufwändiges Motion-Design) → ruhige, schnelle UI.

---

## 9. Responsiveness

- **Ein Breakpoint:** `@media (max-width:640px)`.
  - `.wb-card-grid` wird 1-spaltig (`grid-template-columns:1fr`).
  - `.wb-card-status` rückt in eigene Zeile (`grid-column:1`, `justify-self:start`).
- Grundlayout ist ansonsten fluid (Flex/Grid, `max-width`, Overflow-Scroll bei Tabellen).
- Primär für **Desktop/Laptop** ausgelegt (Planungswerkzeug), auf Schmalgeräten nutzbar.

---

## 10. UI-Verhalten & UX-Besonderheiten

- **Genau eine Ansicht sichtbar**; Wechsel rendert die Zielansicht frisch aus `DB`
  → Ansichten sind nach jedem Wechsel datenkonsistent.
- **eCase-Badges** (Wochenplan & Monatsübersicht) sind **global** (alle Monate) und teilen
  dieselbe Quelle → identische Zahl; aufklappbare Liste bleibt beim Abhaken offen
  (`_ecOpen`), schließt bei Klick außerhalb.
- **Farbcodierung als Leitsystem:** rot = Absenz/Handlungsbedarf, grün = erledigt,
  gelb/amber = offen/Warnung, violett = Individual-Code, blau = Info.
- **WB-TN-Ampel** (🟢/🟡/🔴) plus **Badge** = Zahl offener **roter** (Arbeitsagoge-)WBs.
- **Sofort-Feedback** via Toasts (z. B. „✓ Arbeitsagoge informiert", „↩ Zurück auf Runde 1").
- **Sicherheitsabfragen** (`confirm`) vor destruktiven Aktionen (Tag/TN löschen, Runde zurücksetzen).
- **Auto-Save** bei jeder Änderung; `dirty`-Flag warnt beim Schließen des Tabs.
- **Kein Dark-Mode** (bewusst; helle, kontraststarke Arbeitsoberfläche).
