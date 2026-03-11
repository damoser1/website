---
author: David Moser
label: 11. März 2026
date: 11. März 2026
order: 96
---

# Erweiterung der technischen Planung – Models, Controller und Systemlogik

## Überblick

Mit den neuen Änderungen wurde die bisherige technische Planung des Gutscheinsystems weiter konkretisiert. Während die vorherigen Dokumentationsbeiträge vor allem das Konzept, die Rollen, die Benutzeroberflächen und das Datenbankmodell beschrieben haben, wird in diesem Beitrag die Models und Controller dokumentiert.

Die Controller sind in vier Bereiche unterteilt:

- `Web\Auth` – Authentifizierung und Ersteinrichtung
- `Web\Shared` – Gemeinsame Funktionen für Admin und Lehrer
- `Web\Admin` – Administratorspezifische Verwaltung
- `Web\Teacher` – Lehrerspezifische Funktionen

---

## Umgesetzte Modellstruktur

Für die bereits definierten Tabellen wurden passende Modelle angelegt. Diese übernehmen die Zuweisung von Feldern, Typumwandlungen sowie die Beziehungen zwischen den Entitäten.

### User

Das Model `User` repräsentiert die internen Benutzer des Systems, also Administratoren und Lehrpersonen. Es bildet die Grundlage für Login, Rollensteuerung, Berechtigungen und Benutzerverwaltung.

Verwendete Felder:

- `name`, `email`, `password`
- `role` (admin, teacher)
- `can_create_vouchers`, `can_redeem_vouchers`
- `status` (invited, active, deactivated)
- `invitation_token`, `invitation_expires_at`

Casts und Hidden:

- `password` wird automatisch gehasht gespeichert
- Berechtigungsfelder werden als Boolean gecastet
- `invitation_expires_at` wird als Datum behandelt
- `password`, `remember_token` und `invitation_token` sind verborgen

Beziehungen:

- Ein Benutzer kann mehrere Gutscheine ausstellen (`issuedVouchers`)
- Ein Benutzer kann mehrere Gutscheine einlösen (`redeemedVouchers`)
- Ein Benutzer kann mehrere Schüler anlegen (`createdStudents`)
- Eine Lehrperson kann mehrere personalisierte Templates besitzen (`personalizedTemplates`)

---

### Student

Das Model `Student` repräsentiert die Empfänger der Gutscheine. Schüler besitzen keinen Login, müssen aber eindeutig als Personen verwaltet werden können.

Verwendete Felder:

- `name`, `email`, `created_by`

Beziehungen:

- Ein Schüler kann mehrere Gutscheine besitzen (`vouchers`)
- Ein Schüler gehört zu dem Benutzer, der ihn angelegt hat (`creator`)

Die Klasse eines Schülers wird nicht dauerhaft im Schülerprofil gespeichert, sondern nur beim Erstellen eines Gutscheins als Freitexteingabe übernommen.

---

### Voucher

Das Model `Voucher` stellt die zentrale Entität des Systems dar. Es verbindet Templates, Schüler, Aussteller, Statusverlauf und Einlösung.

Verwendete Felder:

- `voucher_number`, `qr_token`
- `template_id`, `personalized_template_id`
- `issuer_id`, `student_id`, `redeemed_by`
- `class_name`, `reason`, `additional_text`
- `status`, `rejection_reason`, `batch_id`
- `issued_at`, `expires_at`, `sent_at`, `redeemed_at`, `archived_at`, `retention_until`

Alle Zeitfelder werden als Datumswerte gecastet.

Beziehungen:

- Jeder Gutschein basiert auf einem Basis-Template (`template`)
- Ein Gutschein kann optional auf einer personalisierten Template-Version beruhen (`personalizedTemplate`)
- Jeder Gutschein gehört zu einem Aussteller (`issuer`)
- Jeder Gutschein gehört zu einem Schüler (`student`)
- Ein eingelöster Gutschein kann dem einlösenden Benutzer zugeordnet werden (`redeemer`)

---

### VoucherTemplate

Das Model `VoucherTemplate` speichert die vom Administrator verwalteten Standardvorlagen.

Verwendete Felder:

- `name`, `description`, `content`
- `validity_days`
- `logo_path`, `signature_path`
- `is_active` (als Boolean gecastet)

Beziehungen:

- Ein Template kann für viele Gutscheine verwendet werden (`vouchers`)
- Ein Template kann Grundlage mehrerer personalisierter Lehrer-Templates sein (`personalizedTemplates`)

---

### PersonalizedTemplate

Das Model `PersonalizedTemplate` erweitert das Basissystem um eine persönliche Anpassung der Gutscheininhalte durch Lehrpersonen. Beim Personalisieren wird der gesamte Textaufbau des Basis-Templates als vollständige Kopie übernommen und kann frei angepasst werden.

Verwendete Felder:

- `base_template_id`, `teacher_id`, `content`

Beziehungen:

- Jede Personalisierung gehört zu einem Basis-Template (`baseTemplate`)
- Jede Personalisierung gehört zu einer Lehrperson (`teacher`)
- Eine Personalisierung kann von mehreren Gutscheinen verwendet werden (`vouchers`)

---

## Umgesetzte Controller

Die Controller sind nach Rollen und Verantwortlichkeiten in vier Namespaces aufgeteilt:

### Authentifizierungsbereich (`Web\Auth`)

#### SetupController

Zuständig für die Ersteinrichtung des Systems.

- Anzeige der Setup-Seite
- Validierung von Name, E-Mail und Passwort
- Erstellen des ersten Benutzers mit Rolle `admin` und Status `active`
- Automatischer Login und Weiterleitung auf das Dashboard

#### LoginController

Zuständig für die Anmeldung bestehender Benutzer.

- Weiterleitung auf Setup, falls noch kein Benutzer existiert
- Manuelle Prüfung von E-Mail und Passwort
- Statusprüfung: nur aktive Accounts können sich einloggen
- Login mit optionaler Remember-Funktion
- Logout-Funktion

#### InvitationController

Zuständig für den Einladungsprozess neuer Benutzer.

- Suche des Benutzers anhand von Token, Status und Ablaufdatum
- Anzeige der Passwortvergabe-Seite mit E-Mail des eingeladenen Benutzers
- Setzen des Passworts, Aktivierung des Accounts und Löschen des Tokens
- Automatischer Login nach Abschluss

#### PasswordResetController

Zuständig für die Zurücksetzung vergessener Passwörter.

- Anzeige der Passwort-vergessen-Seite
- Versand des Reset-Links
- Anzeige der Reset-Seite mit Token
- Setzen des neuen Passworts
- Weiterleitung auf Login bei Erfolg

---

### Gemeinsame Bereiche (`Web\Shared`)

#### DashboardController

Das Dashboard liefert abhängig von der Rolle unterschiedliche Informationen.

Für Administratoren:

- Anzahl ausstehender Gutscheine
- Die letzten 10 erstellten Gutscheine

Für Lehrpersonen:

- Die letzten 10 eigenen Gutscheine
- Anzahl abgelehnter Gutscheine

#### StudentController

Die gemeinsame Schülerverwaltung für Admin und Lehrer.

- Übersicht aller Schüler mit Paginierung
- Anlegen neuer Schüler mit Validierung (Name, E-Mail eindeutig)
- Bearbeiten vorhandener Schüler
- Löschen nur bei fehlenden aktiven Gutscheinen (Prüfung über Status)
- Archivierte Gutscheine des Schülers werden beim Löschen mitentfernt

#### VoucherCreateController

Zuständig für die Erstellung von Gutscheinen (Einzel und Stapel).

- Laden aktiver Templates und vorhandener Schüler
- Validierung: Template muss existieren, mindestens ein Schüler, Begründung als Pflichtfeld
- Bei mehreren Schülern: automatische Vergabe einer Batch-ID
- Pro Schüler: eigene Gutscheinnummer und eigener QR-Token
- Alle Gutscheine werden gespeichert

#### RedeemController

Zuständig für die digitale Einlösung über QR-Code-Token.

- Laden des Gutscheins
- Anzeige der Gutschein-Details mit Student und Template
- Prüfung: Status muss `sent` sein und Ablaufdatum darf nicht überschritten sein
- Speichern der Einlösung: Status auf `redeemed`, Zeitpunkt und einlösender Benutzer

---

### Administrationsbereich (`Web\Admin`)

#### VoucherController

Zuständig für die administrative Verwaltung aller Gutscheine.

- Übersicht aller Gutscheine mit Eager Loading (Student, Aussteller, Template) und Paginierung
- Detailansicht mit zusätzlichem Laden des Einlösers
- Bearbeiten von Begründung, Zusatztext, Klasse und Ablaufdatum
- Löschen von Gutscheinen
- Bestätigungsansicht: alle Gutscheine mit Status `pending`
- Bestätigen: Status auf `confirmed`, Ausstellungsdatum wird gesetzt
- Ablehnen: Status auf `rejected` mit Pflichtfeld Ablehnungsgrund
- Zurücksetzen: Status zurück auf `pending`, Ablehnungsgrund und Versanddatum werden gelöscht

#### TemplateController

Zuständig für die Verwaltung der Gutschein-Vorlagen.

- Übersicht aller Templates sortiert nach Name
- Erstellen und Bearbeiten mit Validierung (Name, Inhalt als Pflichtfelder)
- Optionaler Upload von Logo und Unterschrift
- Löschen nur möglich, wenn keine Gutscheine auf dem Template basieren
- Deaktivieren als Alternative zum Löschen

#### UserController

Zuständig für die Benutzerverwaltung durch den Administrator.

- Übersicht aller Benutzer sortiert nach Name
- Anlegen neuer Benutzer mit Rolle (admin oder teacher) und optionalen Berechtigungen
- Generierung eines Einladungstokens
- Versand der Einladungs-E-Mail mit Link zur Passwortvergabe
- Erneutes Einladen: neuer Token wird generiert und versendet
- Berechtigungen (Erstellung, Einlösung) per Toggle aktualisieren
- Deaktivieren und Aktivieren von Accounts
- Löschen nur möglich, wenn keine aktiven Gutscheine mit dem Benutzer als Aussteller verknüpft sind

#### ArchiveController

Zuständig für die Anzeige archivierter Gutscheine.

- Laden aller Gutscheine mit Status `archived`
- Eager Loading von Student, Aussteller und Template
- Sortierung nach Archivierungszeitpunkt
- Paginierung mit 20 Einträgen pro Seite

---

### Lehrerbereich (`Web\Teacher`)

#### MyVoucherController

Zuständig für die Verwaltung der eigenen Gutscheine durch Lehrpersonen.

- Übersicht aller selbst erstellten Gutscheine
- Detailansicht: Zugriff nur auf eigene Gutscheine, Prüfung über
- Bearbeitung nur bei abgelehnten Gutscheinen
- Nach Korrektur: Status wechselt zurück auf `pending`, Ablehnungsgrund wird gelöscht

#### PersonalizedTemplateController

Zuständig für die Personalisierung von Templates durch Lehrpersonen.

- Übersicht aller aktiven Basis-Templates und eigener Personalisierungen
- Erstellen: Basis-Template wird als vollständige Kopie übernommen, Lehrer kann Inhalt frei anpassen
- Bearbeiten und Löschen eigener Personalisierungen
- Zugriffskontrolle

---

## Technische Abbildung der Kernprozesse

Die fachlich beschriebenen Abläufe werden durch die Backend-Struktur wie folgt abgebildet:

### 1. Ersteinrichtung

- `LoginController` prüft beim Aufruf, ob Benutzer existieren
- Falls nicht: Weiterleitung auf `SetupController`
- Erster registrierter Benutzer erhält Rolle `admin` und Status `active`

### 2. Benutzer-Einladung

- Administrator erstellt Benutzer`
- System erzeugt Token mit 7 Tagen Gültigkeit und versendet E-Mail
- Eingeladener Benutzer öffnet Link, `InvitationController` prüft Token
- Nach Passwortvergabe: Account wird auf `active` gesetzt, Token gelöscht

### 3. Gutschein-Erstellung

- Berechtigter Benutzer wählt Template und Schüler in `VoucherCreateController`
- Pro Schüler wird ein eigener Gutschein mit eindeutiger Nummer und QR-Token erstellt
- Bei Mehrfachauswahl wird eine gemeinsame Batch-ID vergeben
- Alle Gutscheine werden mit Status `pending` gespeichert

### 4. Gutschein-Prüfung

- Bestätigung: Status auf `confirmed`, Ausstellungsdatum wird gesetzt
- Ablehnung: Status auf `rejected` mit Pflichtfeld Ablehnungsgrund
- Lehrer kann Gutschein korrigieren und erneut einreichen

### 5. Gutschein-Einlösung

- QR-Code enthält Link mit Token
- System prüft Status (`sent`) und Ablaufdatum
- Berechtigter Benutzer bestätigt, Zeitpunkt und Einlöser werden gespeichert

### 6. Archivierung

- Eingelöste oder abgelaufene Gutscheine sollen später automatisch archiviert werden
- `ArchiveController` zeigt archivierte Gutscheine an
- Löschlogik nach Ablauf der Aufbewahrungsfrist ist geplant

---

## Nächste geplante Ausbauschritte

- PDF-Generierung für bestätigte Gutscheine
- Automatischer E-Mail-Versand an Schüler
- Statuswechsel auf `sent` nach erfolgreichem Versand
- Fehlerbehandlung bei fehlgeschlagenem Versand
- Automatische Archivierungslogik
- Automatische Löschlogik nach Aufbewahrungsfrist
- Such-, Filter- und Vorschaufunktionen in den Verwaltungsansichten