---
author: David Moser
label: 25. Februar 2026
date: 25. Februar 2026
order: 3
---

# Projektdokumentation – Gutscheinverwaltung erweitert

## Grundkonzept

Das Projekt ist ein schulinternes Gutscheinsystem, mit dem Gutscheine für Schüler erstellt, verwaltet und eingelöst werden können. Die Gutscheine werden auf Basis konfigurierbarer Templates generiert, per E-Mail als PDF mit QR-Code an die Schüler versendet und können anschließend von berechtigten Lehrpersonen digital eingelöst werden.

Zentrale Eigenschaften des Systems:

- Vollständige CRUD-Operationen für Gutscheine (Erstellen, Einsehen, Bearbeiten, Löschen)
- Rollenbasiertes Berechtigungssystem (Administrator, Lehrer, Schüler)
- Konfigurierbares Template-System für verschiedene Gutscheintypen
- Automatischer Versand des Gutscheins per E-Mail als PDF-Dokument mit individueller Gutschein-ID und QR-Code zur eindeutigen Identifikation.
- Stapel-Erstellung und Mehrfachversand: Ein Gutschein kann in einem Vorgang für mehrere Schüler erstellt werden (z. B. Teilnehmer einer Veranstaltung). Pro Schüler wird ein eigener Gutschein mit eigener Gutschein-ID und eigenem QR-Code generiert und versendet.
- Bestätigungsworkflow für Erstellung und Einlösung
- Archivierungssystem mit Aufbewahrungsfrist

---

## Rollenmodell

Das System unterscheidet drei Benutzerrollen mit klar abgegrenzten Berechtigungen.

### Administrator (Direktion / Sekretariat)

Der erste Administrator-Account wird über eine einmalige Registrierung angelegt. Beim erstmaligen Aufruf des Systems wird automatisch die Registrierungsseite angezeigt. Nach erfolgreicher Registrierung wird dieser Account als Administrator gespeichert und die öffentliche Registrierung dauerhaft deaktiviert. Alle weiteren Benutzer (Administratoren und Lehrpersonen) werden ausschließlich über den Einladungs-Workflow durch einen bestehenden Administrator erstellt.

Der Administrator verfügt über Vollzugriff auf alle Funktionen des Systems. Im Einzelnen umfasst dies:

- Vollzugriff auf alle Gutscheine (Erstellen, Bearbeiten,  Einsehen, Löschen sowie Zurücksetzen oder Korrigieren von Gutscheinen bei fehlerhaften Einträgen)
- Verwaltung und Anpassung von Gutschein-Templates 
- Vergabe und Entzug von Lehrerberechtigungen zur Gutscheinerstellung und Gutscheineinlösung
- Erstellung von Lehrer-Accounts: Der Administrator legt einen neuen Lehrer mit Name und E-Mail-Adresse an. Das System versendet automatisch eine Einladungs-E-Mail mit einem einmaligen Link zur Passwort-Vergabe. Erst nach Abschluss der Passwort-Vergabe ist der Account aktiv.
- Bestätigung von Gutscheinen nach Erstellung durch Lehrpersonen
- Verwaltung von Schülern (Anlegen, Bearbeiten, Löschen)

Sollten nachträglich Fehler festgestellt werden (z. B. falscher Empfänger oder falsches Ablaufdatum), kann ein Administrator den Gutschein zurücksetzen oder korrigieren. Dabei wird der Status angepasst und der Gutschein gegebenenfalls erneut zur Bestätigung vorgelegt.

### Lehrer

Lehrpersonen haben eingeschränkte Rechte, die vom Administrator individuell vergeben werden:

- Erstellung von Gutscheinen (nur mit erteilter Berechtigung durch den Administrator)
- Nicht jede Lehrperson kann Gutscheine erstellen – die Berechtigung wird gezielt vergeben
- Einlösung von Gutscheinen (nur mit erteilter Berechtigung durch den Administrator)
- Einlösung erfolgt durch Scannen des QR-Codes oder Eingabe der Gutscheinnummer
- Verwaltung von Schülern (Anlegen, Bearbeiten, Löschen)

Der Lehrer scannt den QR-Code mit der Kamera seines Smartphones – der Link öffnet die Webapp direkt auf der Einlöse-Seite des jeweiligen Gutscheins.

### Schüler

Schüler haben keinen direkten Zugriff auf das System. Sie sind ausschließlich Empfänger von Gutscheinen, die per E-Mail als PDF zugestellt werden. Ein Schüler wird in der Datenbank angelegt, wenn er vom Administrator oder einer Lehrperson manuell erfasst wird oder bei der Gutschein-Erstellung als neuer Empfänger hinzugefügt wird.

Eine separate Speicherung der Klassenzugehörigkeit ist für die Funktionsweise des Systems nicht erforderlich, da Gutscheine ausschließlich über die eindeutige Schüler-ID verwaltet werden. Die Klasse kann jedoch beim Erstellen eines Gutscheins über ein Dropdown-Menü ausgewählt und als Platzhalter im Template verwendet werden, ohne dass sie dauerhaft am Schüler gespeichert wird.

### Berechtigungsmatrix

| Funktion                 | Administrator | Lehrer                 | Schüler        |
|--------------------------|---------------|------------------------|----------------|
| Gutschein erstellen      | ✓             | Mit Berechtigung       | –              |
| Gutschein bestätigen     | ✓             | –                      | –              |
| Gutschein einlösen       | ✓             | Mit Berechtigung       | Übergabe an Lehrer |
| Gutschein einsehen       | Alle          | Selbst erstellte       | –              |
| Templates erstellen      | ✓             | –                            | –      |
| Templates verwalten      | ✓             | –                            | –      |
| Templates erweitern      | ✓             | ✓ (individuelle Anpassungen) | –      |
| Gutschein-Inhalt ergänzen| ✓             | nur bei Erstellung    | –              |
| Berechtigungen vergeben  | ✓             | –                      | –              |
| Archiv & Löschrechte     | ✓             | –                      | –              |
| Gutschein empfangen      | –             | –                      | ✓              |
| Schüler verwalten        | ✓             | ✓                      | –              |
| Lehrer-Accounts anlegen  | ✓             | –                            | –              |
| Lehrer deaktivieren/löschen | ✓          | –                            | –              |

---

### Archiv
Die Archivierung und Löschung von Gutscheinen erfolgt automatisiert. Abgelaufene Gutscheine werden in ein Archiv überführt und nach Ablauf einer definierten Aufbewahrungsfrist endgültig gelöscht. Existieren danach keine archivierten Gutscheine mehr, wird auch der zugehörige Schülereintrag automatisch aus der Datenbank entfernt.

---

## Workflow

### Lehrer-Account-Erstellung

Die Erstellung von Lehrer-Accounts erfolgt ausschließlich durch den Administrator:

1. Der Administrator legt einen neuen Lehrer an und erfasst Name und E-Mail-Adresse.
2. Das System versendet automatisch eine Einladungs-E-Mail an die angegebene Adresse. Die E-Mail enthält einen einmaligen Link zur Passwort-Vergabe.
3. Der Lehrer öffnet den Link und vergibt sein persönliches Passwort.
4. Nach erfolgreicher Passwort-Vergabe ist der Account aktiv und der Lehrer kann sich einloggen.
5. Der Einladungslink ist nach einmaliger Nutzung oder nach Ablauf einer definierten Frist ungültig. Bei Bedarf kann der Administrator eine neue Einladung versenden.

### Gutschein-Erstellung

Der Erstellungsprozess durchläuft mehrere Schritte mit einem Bestätigungsmechanismus:

1. Eine berechtigte Lehrperson oder ein Administrator erstellt einen neuen Gutschein auf Basis eines vordefinierten Templates (Bearbeitung möglich).
2. Der Gutschein erhält einen Status und wird dem Sekretariat bzw. der Direktion zur Bestätigung vorgelegt. Abgelehnte Gutscheine können vom Administrator dauerhaft gelöscht werden, ansonsten wird er archiviert
3. Das Sekretariat oder die Direktion prüft den Gutschein und bestätigt oder lehnt ihn ab.
4. Bei Ablehnung: Der Gutschein erhält erneut einen Status mit einem Ablehnungsgrund. Der erstellende Lehrer kann den Gutschein korrigieren (z. B. Begründung anpassen, Empfänger ändern) und erneut zur Bestätigung einreichen. Der Status wechselt dann zurück auf „Ausstehend". Alternativ kann der Administrator den abgelehnten Gutschein dauerhaft löschen.
5. Nach der Bestätigung wird der Gutschein automatisch per E-Mail als PDF an den Schüler versendet. Der Status wechselt nach Versand.

### Stapel-Erstellung (Mehrere Empfänger)

Neben der Erstellung für einzelne Schüler unterstützt das System eine Stapel-Erstellung für mehrere Empfänger (z. B. eine Gruppe von Teilnehmern einer Veranstaltung).

1. Der Lehrer/Administrator wählt ein Template und erfasst die gemeinsamen Gutschein-Daten (Begründung, Gültigkeit, optional Zusatztext).
2. Es wird eine Empfängerliste gewählt (z. B. mehrere ausgewählte Schüler oder Import aus einer Liste).
3. Das System erzeugt pro Empfänger einen eigenen Gutschein mit eindeutiger Gutschein-ID und QR-Code.
4. Die Gutscheine werden gesammelt zur Bestätigung vorgelegt (Batch).
5. Nach Bestätigung werden pro Empfänger automatisiert PDF und E-Mail versendet. Versandfehler werden pro Gutschein protokolliert, damit einzelne Empfänger erneut versendet werden können.

### Gutschein-Einlösung

Die Einlösung eines Gutscheins erfolgt ausschließlich durch den Klassenvorstand des jeweiligen Schülers:

1. Der Schüler leitet die erhaltene E-Mail mit Gutschein-Id an den Lehrer weiter oder übergibt den ausgedruckten Gutschein mit QR-Code.
2. Das Sekretariat oder die Direktion prüft den Gutschein und bestätigt oder lehnt ihn ab.
3. Der Klassenvorstand bestätigt die Einlösung im System. Der Gutschein-Status wieder und Gutschein gelangt in das Archiv.


### Passwort-Zurücksetzung

Lehrpersonen und auch der Admin können ihr Passwort über die „Passwort vergessen“-Funktion zurücksetzen.

1. Das System versendet automatisch eine E-Mail mit einem einmaligen Reset-Link.
2. Über diesen Link kann ein neues Passwort vergeben werden.
3. Der Link ist zeitlich begrenzt und kann nur einmal verwendet werden.



---

## Template

Ein Gutschein-Template definiert den festen Aufbau und das Erscheinungsbild eines Gutscheins und ermöglicht eine standardisierte Erstellung.

Eigenschaften:
- Fester Textaufbau
- Platzhalter (Name, Klasse, Datum, Schuljahr, Begründung)
- Layout-Vorlage für das Design des Gutscheins
- Optional Logos oder Unterschriften
- Aktiv/Inaktiv-Status ob Template verwendbar ist

Die Klasse wird nicht am Schüler gespeichert, sondern beim Erstellen eines Gutscheins über ein Dropdown-Menü ausgewählt und als Platzhalter in das Template eingefügt.

Jeder Gutschein-Typ basiert auf einem vom Administrator konfigurierbaren Template. Templates werden zentral verwaltet und sorgen für eine einheitliche Gestaltung sowie konsistente Inhalte.

| Template          | Beschreibung                                  |
|-------------------|-----------------------------------------------|
| Nachmittag frei   | Freistellung für einen Nachmittag             |
| Vormittag frei    | Freistellung für einen Vormittag              |
| Ganzer Tag frei   | Freistellung für einen gesamten Schultag      |
| 2 Tage frei       | Freistellung für zwei gesamte Schultage       |

---

## Gutschein

Ein Gutschein nutzt ein konkretes Template und enthält individuelle, schülerbezogene Informationen.

- Nutzung eines Templates als strukturelle Grundlage
- Eindeutige Gutscheinnummer zur Identifikation
- Zuordnung zu Aussteller (Administrator, Lehrperson) und Empfänger (Schüler)
- Individuelle Begründung (z. B. „Ausgezeichneter Erfolg", „Teilnahme Wettbewerb")
- Klasse (wird beim Erstellen über Dropdown-Menü ausgewählt, nicht am Schüler gespeichert)
- Ausstellungsdatum und optionales Ablaufdatum
- QR-Code zur Einlösung

---

## Statusübergänge

Ein Gutschein durchläuft während seines Lebenszyklus mehrere definierte Statusübergänge. Nach der Erstellung ist der Gutschein ausstehend und muss von einem Administrator bestätigt und aktiviert werden. Ein aktiver Gutschein kann anschließend eingelöst werden. Bei Überschreiten des Ablaufdatums wird der Gutschein automatisch archiviert und nach Ablauf der Aufbewahrungsfrist endgültig gelöscht.

| Status       | Beschreibung                     | Verantwortlich | Nächster Status            |
|--------------|----------------------------------|---------------|-----------------------------|
| Entwurf      | Gutschein wurde angelegt         | Lehrer/Admin  | Ausstehend                  |
| Ausstehend   | Wartet auf Bestätigung           | Admin         | Bestätigt / Abgelehnt       |
| Bestätigt    | Freigegeben                      | System/Admin  | Gesendet                    |
| Gesendet     | Mail/PDF wurde verschickt        | System        | Eingelöst / Abgelaufen      |
| Eingelöst    | KV hat eingelöst                 | KV/Admin      | Archiviert                  |
| Abgelaufen   | Datum überschritten              | System        | Archiviert                  |
| Archiviert   | Im Archiv                        | System/Admin  | Gelöscht                    |
| Gelöscht     | endgültig entfernt               | System/Admin  | –                           |

---

### Vorbereitung

- Projektstruktur angelegt und Konzept inkl. Datenbankmodell erarbeitet (angefangen)