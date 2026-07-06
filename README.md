# Lernwerkstatt – Einteilungsplan

Wochenplanungs-Tool für das BASISJOB Motivationssemester.

## Starten

Direkt im Browser öffnen → GitHub Pages URL nach Aktivierung verfügbar.

Oder lokal: `index.html` einfach im Browser öffnen – kein Server nötig.

## Features

- Wochenplan mit VM/NM-Einteilung, Abwesenheitscodes, eCase-Tracking
- Automatische Sperrung von Schultagen und Sportterminen
- Fixpräferenzen für wiederkehrende Werkstatt-Zuweisung
- Wirkungsbericht-Erinnerungen (8W / 10W) mit Mailto-Link
- Monatsübersicht und Monatsrapport als PDF (BASISJOB-Format)
- Export / Import als JSON-Datensicherung

## Daten

Alle Daten werden lokal im Browser (localStorage) gespeichert.
Regelmässig über den Export-Button sichern.

> **Datenschutz:** Die Daten (inkl. personenbezogener Angaben) liegen
> **unverschlüsselt** im Browser. Auf geteilten Rechnern die Funktion
> **„🗑 Daten löschen"** nutzen. Exportierte `.json`-Sicherungen enthalten
> Personendaten und dürfen **nicht** ins Repository gelangen (siehe `.gitignore`).
> Details in [`SECURITY.md`](SECURITY.md).