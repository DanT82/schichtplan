# Schichtplan Getränke – Projektübersicht

Übergabe-Dokument für weitere Arbeit mit Claude. **Im neuen Chat immer die aktuelle `index.html` (und bei Bedarf `sw.js`) mitgeben.**

Stand: 31.10.2026 · Version 1.0

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
- `teams/getraenke/weeks/{JJJJ-Www}`: `{shifts:[{id, empId, date, from, to, task, tasks[], sub?, auto?, autoRot?, autoLead?}]}`
- `teams/getraenke/members/{uid}`: `{name, username, role: admin|bearbeiten|lesen}`
- `teams/getraenke/meta/setup`: existiert nach der Ersteinrichtung
- Lokal: `localStorage["schichtplan-getraenke-v1"]` = `{cfg, weeks}` als Kopie; Firestore-Offline-Persistenz aktiv.

## Wichtige Logik
- Besetzungsregeln (`rules` + `areaHours`), Leitung zählt als Fläche, Früh/Spät-Wechsel der Leitung (wöchentlich), freie Tage mit automatischer Vertretung, Joshua Wechselschicht + Schule/Seminar-Schalter, Pausen (gesetzlich bzw. `breakMin`), Einsätze innerhalb einer Schicht (`sub`), mehrere Aufgaben je Schicht, Tagesaufgaben/Bestellungen mit Fähigkeiten, Ferien-Verfügbarkeit, „Leergut besetzt“-Hinweis, Automatik „Woche automatisch planen“, PDF A4 quer (eigener PDF-Generator, ohne Bibliothek).
- Zugänge anlegen: zweite Firebase-App-Instanz „zugang“ legt den Nutzer an, dann Admin schreibt `members/{uid}`.
