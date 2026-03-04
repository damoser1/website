---
author: David Moser
label: Doku 25. Februar 2026
date: 25. Februar 2026
---

# Projektdokumentation – Gutscheinverwaltung erweitert

## Grundkonzept

Das Projekt ist ein schulinternes Gutscheinsystem, mit dem Gutscheine für Schüler erstellt, verwaltet und eingelöst werden können. Die Gutscheine werden auf Basis konfigurierbarer Templates generiert, per E-Mail als PDF mit QR-Code an die Schüler versendet und können anschließend vom Klassenvorstand digital eingelöst werden.

Zentrale Eigenschaften des Systems:

- Vollständige CRUD-Operationen für Gutscheine (Erstellen, Einsehen, Bearbeiten, Löschen)
- Rollenbasiertes Berechtigungssystem (Administrator, Lehrer, Schüler)
- Konfigurierbares Template-System für verschiedene Gutscheintypen
- Automatischer Versand des Gutscheins per E-Mail als PDF-Dokument mit individueller Gutschein-ID und QR-Code zur eindeutigen Identifikation.
- Stapel-Erstellung und Mehrfachversand: Ein Gutschein kann in einem Vorgang für mehrere Schüler erstellt werden (z. B. Teilnehmer einer Veranstaltung). Pro Schüler wird ein eigener Gutschein mit eigener Gutschein-ID und eigenem QR-Code generiert und versendet.
- Bestätigungsworkflow für Erstellung und Einlösung
- Archivierungssystem mit Aufbewahrungsfrist

## Rollenmodell

Das System unterscheidet drei Benutzerrollen mit klar abgegrenzten Berechtigungen.

### Administrator (Direktion / Sekretariat)

Der Administrator verfügt über Vollzugriff auf alle Funktionen des Systems. Im Einzelnen umfasst dies:

- Vollzugriff auf alle Gutscheine (Erstellen, Bearbeiten,  Einsehen, Löschen (auch abgelehnte oder fehlerhafte Gutscheine))
- Verwaltung und Anpassung von Gutschein-Templates 
- Vergabe und Entzug von Lehrerberechtigungen zur Gutscheinerstellung
- Bestätigung von Gutscheinen nach Erstellung durch Lehrpersonen


### Lehrer

Lehrpersonen haben eingeschränkte Rechte, die vom Administrator individuell vergeben werden:

- Erstellung von Gutscheinen (nur mit erteilter Berechtigung durch den Administrator)
- Nicht jede Lehrperson kann Gutscheine erstellen – die Berechtigung wird gezielt vergeben
- Der Klassenvorstand kann Gutscheine von Schülern seiner eigenen Klasse einlösen
- Einlösung erfolgt durch Scannen des QR-Codes oder Eingabe der Gutscheinnummer

### Schüler

Schüler haben keinen direkten Zugriff auf das System. Sie sind ausschließlich Empfänger von Gutscheinen, die per E-Mail als PDF zugestellt werden. Ein Schüler wird erst in der Datenbank angelegt, wenn er seinen ersten Gutschein erhält bzw der Lehrer oder Administrator den Gutschein erstellt.

### Berechtigungsmatrix

| Funktion                 | Administrator | Lehrer                 | Schüler        |
|--------------------------|---------------|------------------------|----------------|
| Gutschein erstellen      | ✓             | Mit Berechtigung       | –              |
| Gutschein bestätigen     | ✓             | –                      | –              |
| Gutschein einlösen       | ✓             | Nur KV                 | Übergabe an KV |
| Gutschein einsehen       | Alle          | Selbst erstellte       | –              |
| Templates erstellen      | ✓             | –                            | –      |
| Templates verwalten      | ✓             | –                            | –      |
| Templates erweitern      | ✓             | ✓ (individuelle Anpassungen) | –      |
| Gutschein-Inhalt ergänzen| ✓             | nur bei Erstellung    | –              |
| Berechtigungen vergeben  | ✓             | –                      | –              |
| Archiv & Löschrechte     | ✓             | –                      | –              |
| Gutschein empfangen      | –             | –                      | ✓              |

### Archiv
Die Archivierung und Löschung von Gutscheinen erfolgt automatisiert. Abgelaufene Gutscheine werden in ein Archiv überführt und nach Ablauf einer definierten Aufbewahrungsfrist endgültig gelöscht. Existieren danach keine archivierten Gutscheine mehr, wird auch der zugehörige Schülereintrag automatisch aus der Datenbank entfernt.


## Workflow

### Gutschein-Erstellung

Der Erstellungsprozess durchläuft mehrere Schritte mit einem Bestätigungsmechanismus:

1. Eine berechtigte Lehrperson oder ein Administrator erstellt einen neuen Gutschein auf Basis eines vordefinierten Templates (Bearbeitung möglich).
2. Der Gutschein erhält einen Status und wird dem Sekretariat bzw. der Direktion zur Bestätigung vorgelegt. Abgelehnte Gutscheine können vom Administrator dauerhaft gelöscht werden, ansonsten wird er archiviert
3. Das Sekretariat oder die Direktion prüft den Gutschein und bestätigt oder lehnt ihn ab.
4. Nach der Bestätigung wird der Gutschein automatisch per E-Mail als PDF an den Schüler versendet. Der Status wechselt nach Versand.

### Stapel-Erstellung (Mehrere Empfänger)

Neben der Erstellung für einzelne Schüler unterstützt das System eine Stapel-Erstellung für mehrere Empfänger (z. B. eine Gruppe von Teilnehmern einer Veranstaltung).

1. Der Lehrer/Administrator wählt ein Template und erfasst die gemeinsamen Gutschein-Daten (Begründung, Gültigkeit, optional Zusatztext).
2. Es wird eine Empfängerliste gewählt (z. B. Klasse, mehrere ausgewählte Schüler oder Import aus einer Liste).
3. Das System erzeugt pro Empfänger einen eigenen Gutschein mit eindeutiger Gutschein-ID und QR-Code.
4. Die Gutscheine werden gesammelt zur Bestätigung vorgelegt (Batch).
5. Nach Bestätigung werden pro Empfänger automatisiert PDF und E-Mail versendet. Versandfehler werden pro Gutschein protokolliert, damit einzelne Empfänger erneut versendet werden können.

### Gutschein-Einlösung

Die Einlösung eines Gutscheins erfolgt ausschließlich durch den Klassenvorstand des jeweiligen Schülers:

1. Der Schüler leitet die erhaltene E-Mail mit Gutschein-Id an den Lehrer weiter oder übergibt den ausgedruckten Gutschein mit QR-Code.
2. Das Sekretariat oder die Direktion prüft den Gutschein und bestätigt oder lehnt ihn ab.
3. Der Klassenvorstand bestätigt die Einlösung im System. Der Gutschein-Status wieder und Gutschein gelangt in das Archiv.


## Template

Ein Gutschein-Template definiert den festen Aufbau und das Erscheinungsbild eines Gutscheins und ermöglicht eine standardisierte Erstellung.

Eigenschaften:
- Fester Textaufbau
- Platzhalter (Name, Klasse, Datum, Schuljahr, Begründung)
- Layout-Vorlage für das Design des Gutscheins
- Optional Logos oder Unterschriften
- Aktiv/Inaktiv-Status ob Template verwendbar ist

Jeder Gutschein-Typ basiert auf einem vom Administrator konfigurierbaren Template. Templates werden zentral verwaltet und sorgen für eine einheitliche Gestaltung sowie konsistente Inhalte.

| Template          | Beschreibung                                  |
|-------------------|-----------------------------------------------|
| Nachmittag frei   | Freistellung für einen Nachmittag             |
| Vormittag frei    | Freistellung für einen Vormittag              |
| Ganzer Tag frei   | Freistellung für einen gesamten Schultag      |
| 2 Tage frei       | Freistellung für zwei gesamte Schultage       |

## Gutschein

Ein Gutschein nutzt ein konkretes Template und enthält individuelle, schülerbezogene Informationen.

- Nutzung eines Templates als strukturelle Grundlage
- Eindeutige Gutscheinnummer zur Identifikation
- Zuordnung zu Aussteller (Administrator, Lehrperson) und Empfänger (Schüler)
- Individuelle Begründung (z. B. „Ausgezeichneter Erfolg", „Teilnahme Wettbewerb")
- Ausstellungsdatum und optionales Ablaufdatum
- QR-Code zur Einlösung

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

## Technische Umsetzung

### Datenbankmodell


## Allgemeines UI-Konzept

Die Benutzeroberfläche ist rollenbasiert aufgebaut, wobei nur die für die jeweilige Rolle relevanten Funktionen angezeigt werden. 

### Layout und Navigation

#### Benutzeroberfläche Administratoren

**Navigation:**

- Dashboard
- Gutscheine (Suchfunktion Gutscheine, alle Gutscheine einsehen, selbst erstelle Gutscheine einsehen, Gutscheine erstellen)
- Templates
- Benutzer/Berechtigungen
- Archiv

**Funktionen im Detail:**

**Dashboard**
- Gesamtübersicht über Gutscheine nach Status, Anzahl ausstehender Bestätigungen
- Schnellzugriff auf zuletzt erstellte Gutscheine
- Hinweise auf ablaufende oder abgelaufene Gutscheine



#### Benutzeroberfläche Lehrer

**Navigation:**

- Dashboard
- Gutscheine (Suchfunktion Gutscheine, selbst erstelle Gutscheine, Gutschein erstellen)
- Templates personalisieren
- Gutschein einlösen

**Funktionen im Detail:**


Todo:
UI Konzept erweitern

Zurücksetzen des Gutscheins bzw Umänderung im Nachinein
Kv bekommt nur das Recht Gutscheine bestätigen zu können, 

### Bisherige Umsetzung

Das Laravel-Projekt wurde initialisiert und die Pakete Fortify (Authentifizierung) sowie Sanctum (API-Token-Verwaltung) eingebunden. Anschließend wurden drei Datenbankmigrationen erstellt: Die `users`-Tabelle wurde um Felder für Rolle, Klassenzugehörigkeit, Matrikelnummer und Berechtigungen erweitert. Die Tabelle `voucher_rules` speichert die konfigurierbaren Gutschein-Vorlagen (Typ, Menge, Einheit, Gültigkeitsdauer). Die Tabelle `vouchers` bildet die einzelnen ausgestellten Gutscheine ab – inklusive Status-Verwaltung (aktiv, eingelöst, abgelaufen, ungültig), Archivierung mit Aufbewahrungsfrist und Nachverfolgung der Einlösung.

### Vorbereitung


- Projektstruktur angelegt und Konzept inkl. Datenbankmodell erarbeitet (angefangen)
