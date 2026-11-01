# Schichtplan Getränke – Projektübersicht

Übergabe-Dokument für weitere Arbeit mit Claude. **Im neuen Chat immer die aktuelle `index.html` (und bei Bedarf `sw.js`) mitgeben.**

Stand: 01.11.2026 · Version 1.9.1

## Worum es geht
Wochen-Schichtplan für die Getränkeabteilung (Edeka Wehrmann): Leergut, Fläche, Einweg, Leitung. Nutzer: Dan (Abteilungsleiter, Admin), Marktleitung (nur ansehen). Gerät: vor allem iPad, als Web-App auf dem Home-Bildschirm, offline-fähig.

## Wo liegt was
| Was | Wo |
|---|---|
| App | https://dant82.github.io/schichtplan/ (GitHub Pages, Repo `DanT82/schichtplan`, Branch `main`, Ordner `/`) |
| Datenbank | Firebase-Projekt `schichtplan-44c7b`, Firestore europe-west3, Spark-Tarif |
| Anmeldung | Firebase Auth E-Mail/Passwort. Benutzername `x` wird intern zu `x@schichtplan.local` |
| Konfiguration | `firebase-config.js` (nicht geheim) |
| Regeln | `firestore.rules` (in Firebase veröffentlicht) |

## Dateien
`index.html` (komplette App, Vanilla JS), `sw.js` (Service Worker, `CACHE` bei jedem Update hochzählen), `manifest.webmanifest`, `icon-*.png`, `firebase-config.js`, `fb-app.js`/`fb-auth.js`/`fb-firestore.js` (Firebase SDK 10.14.1 compat, lokal für Offline), `firestore.rules`.

## Datenmodell (Firestore)
- `teams/getraenke/plan/config`: alles außer Schichten: `settings` (Zeiten, Bundesland, leadCover, leadRotation), `employees[]`, `rules[]`, `areaHours`, `dayOverrides`, `dayTasks[]`, `skills[]`, `vacations[]`
- `teams/getraenke/weeks/{JJJJ-Www}`: `{shifts:[{id, empId, date, from, to, task, tasks[], sub?, auto?, autoRot?, autoLead?}], done:{"JJJJ-MM-TT|aufgabenId":true}}`
- `dayTasks[]` in config: `{id, label, days[], time, skill, owner (empId), deps:[empId, empId]}` = Vertretungsmatrix
- `teams/getraenke/members/{uid}`: `{name, username, role: admin|bearbeiten|lesen}`
- `teams/getraenke/meta/setup`: existiert nach der Ersteinrichtung
- Lokal: `localStorage["schichtplan-getraenke-v1"]` = `{cfg, weeks}` als Kopie; Firestore-Offline-Persistenz aktiv.

## Wichtige Logik
- Besetzungsregeln (`rules` + `areaHours`), Leitung zählt als Fläche, Früh/Spät-Wechsel der Leitung (wöchentlich), freie Tage mit automatischer Vertretung, Joshua Wechselschicht + Schule/Seminar-Schalter, Pausen (gesetzlich bzw. `breakMin`), Einsätze innerhalb einer Schicht (`sub`), mehrere Aufgaben je Schicht, Tagesaufgaben/Bestellungen mit Fähigkeiten, Ferien-Verfügbarkeit, „Leergut besetzt“-Hinweis, Automatik „Woche automatisch planen“, PDF A4 quer (eigener PDF-Generator, ohne Bibliothek).
- Zugänge anlegen: zweite Firebase-App-Instanz „zugang“ legt den Nutzer an, dann Admin schreibt `members/{uid}`.

## Änderungen
- 1.1 (01.11.2026): Reiter „Vertretung“ (Vertretungsmatrix mit Verantwortlich, 1. und 2. Vertretung, Übersichtsraster), Panel „Aufgaben diese Woche“ mit Abhaken, überfällige Aufgaben als Hinweis, PDF mit [ ]/[x]. `sw.js` CACHE = schichtplan-v2.
- 1.2 (01.11.2026): Feste Schichten je Mitarbeiter (`fixed:[{days,from,to,task}]`, Aufgabe `auto_lf` = Leergut wenn Lücke, sonst Fläche); einmalige Übernahme Timon Mo/Mi/Fr 17–22 Einweg, Joshua Mi 16–21:15 (`migrFixed1`). Automatik füllt jetzt Bereich für Bereich über die ganze Woche (erst Leergut), Aushilfen ohne Tagesgrenze vor Schülern. CACHE schichtplan-v3.
- 1.3 (01.11.2026): Besondere Öffnungszeiten je Datum (`dayOverrides[datum].open/close`), Regeln und automatische Schichten werden auf Öffnung bis Schluss + 15 Min. begrenzt, Früh-Beginn verschiebt sich auf frühere Öffnung. Eingetragen: 24.12.2026 06–13, 31.12.2026 07–14 (`migrHours1`). CACHE schichtplan-v4.
- 1.4 (01.11.2026): Aufgeräumte Wochenansicht: Umschalter Plan/Aufgaben/Besetzung/Stunden, Konflikte & Hinweise als Seitenfenster (Knopf oben), „Mehr“-Menü (PDF, Früh/Spät, Woche leeren), Leitungszeile statt Panel, Feld antippen öffnet Aktionen (Schicht, Einsatz, Frei, Schule, Seminar, Urlaub), Stunden-Spalte rechts, Namensspalte fixiert. Konto & Sicherung über den Status-Knopf oben rechts, Zugänge dort. Abschnitte in Regeln und Profil auf- und zuklappbar. CACHE schichtplan-v5.
- 1.5 (01.11.2026): Ansicht „Besetzung“ zeigt unter jedem Tagesbalken die Personen als Zeitstrahl (Name + Uhrzeit, Einsätze berücksichtigt). CACHE schichtplan-v6.
- 1.6 (01.11.2026): Ansicht „Besetzung“ als einfache Tabelle (Tag × Leergut/Fläche) mit Status „besetzt“/„fehlt von–bis“ und Namensliste statt Zeitstrahl. CACHE schichtplan-v7.
- 1.7 (01.11.2026): Stundenkonto je Mitarbeiter (`konto:{hours,date}`; Saldo = Start + Σ(Ist − Grund-Soll) aller geplanten Wochen ab Startwoche). Überstundenabbau (`cfg.abbau[wid][empId]` = "all" oder Minuten): Fenster mit Probeplanung der nächsten 8 Wochen (mit/ohne Abbau), „passt“ = keine neue Lücke und niemand sonst mehr Stunden. Automatisch planen gleicht Konten in Folgewochen aus (`settings.autoAusgleich`, Standard an). Name im Plan antippen öffnet den Abbau. CACHE schichtplan-v8.
- 1.7.1: Abbau-Fix: Beim Eintragen werden alle (auch handeingetragenen, außer festen) Schichten des Mitarbeiters in der Woche entfernt und neu geplant; Lückenfüllen hält das reduzierte Wochensoll ein (neue Schicht nur bei ≥ 2 Std. Rest). CACHE schichtplan-v9.
- 1.8: Feste Schicht mit `coverLead`: Ist der Mitarbeiter abwesend, verlängert die Automatik die späteste Leitungsschicht bis zum Ende der festen Schicht und ergänzt die Aufgabe (`coverFix`). Timon Mo/Mi/Fr Einweg → Leitung bis 22:00 (`migrCover1`). Prüfung in Konflikten. CACHE schichtplan-v10.
- 1.9: Reiter „Automatik“ (`renderAuto`): Übersicht aller automatischen Abläufe, erzeugt aus den aktuellen Einstellungen, mit Sprung zur jeweiligen Einstellung. CACHE schichtplan-v11.
- 1.9.1: Uhrzeit bei Aufgaben bedeutet „erledigt bis“: Person zählt, wenn ihre Schicht vor dieser Uhrzeit beginnt (vorher: musste genau zu der Uhrzeit da sein). CACHE schichtplan-v12.
