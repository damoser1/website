---
author: David Moser
label: 11. März 2026
date: 11. März 2026
order: 96
---

# Authentifizierung, Benutzerlogik und Datenmodell

In diesem Entwicklungsschritt wurden die ersten Laravel-Modelle sowie die grundlegenden Controller für Authentifizierung, Ersteinrichtung und Einladungslogik umgesetzt.  
Da das Datenbankmodell bereits in der vorherigen Dokumentation beschrieben wurde, liegt der Fokus hier auf der Anwendungslogik im Backend.

## Umgesetzte Modelle

Für die bereits definierten Tabellen wurden passende Modelle angelegt. Diese übernehmen die Zuweisung von Feldern, Typumwandlungen sowie die Beziehungen zwischen den Entitäten.

### `User`

Das Modell `User` bildet Administratoren und Lehrpersonen ab.

Verwendete Felder:

- `name`
- `email`
- `password`
- `role`
- `can_create_vouchers`
- `can_redeem_vouchers`
- `status`
- `invitation_token`
- `invitation_expires_at`

Zusätzlich wurden folgende Eigenschaften definiert:

- `password` wird automatisch gehasht gespeichert
- Berechtigungsfelder werden als Boolean gecastet
- `invitation_expires_at` wird als Datum behandelt
- sensible Felder wie `password`, `remember_token` und `invitation_token` sind verborgen

Beziehungen:

- Ein Benutzer kann viele ausgestellte Gutscheine besitzen
- Ein Benutzer kann viele eingelöste Gutscheine besitzen
- Ein Benutzer kann viele Schüler angelegt haben
- Ein Benutzer kann viele personalisierte Templates besitzen

Damit bildet das Modell sowohl Administratoren als auch Lehrpersonen vollständig ab.

---

### `Student`

Das Modell `Student` repräsentiert die Schüler als Empfänger von Gutscheinen.

Verwendete Felder:

- `name`
- `email`
- `created_by`

Beziehungen:

- Ein Schüler gehört zu dem Benutzer, der ihn angelegt hat
- Ein Schüler kann mehrere Gutscheine besitzen

Dieses Modell dient ausschließlich der Verwaltung der Empfänger und besitzt keine eigene Login-Funktion.

---

### `Voucher`

Das Modell `Voucher` bildet die zentrale Entität des Systems.

Verwendete Felder:

- `voucher_number`
- `template_id`
- `personalized_template_id`
- `issuer_id`
- `student_id`
- `class_name`
- `reason`
- `additional_text`
- `qr_token`
- `status`
- `rejection_reason`
- `batch_id`
- `issued_at`
- `expires_at`
- `sent_at`
- `redeemed_at`
- `redeemed_by`
- `archived_at`
- `retention_until`

Zusätzlich wurden mehrere Zeitfelder als Datumswerte gecastet.

Beziehungen:

- Ein Gutschein gehört zu einem Basis-Template
- Ein Gutschein kann optional ein personalisiertes Template verwenden
- Ein Gutschein gehört zu einem Aussteller
- Ein Gutschein gehört zu einem Schüler
- Ein Gutschein kann optional einem einlösenden Benutzer zugeordnet sein

Damit ist die fachliche Struktur eines Gutscheins bereits vollständig im Modell vorbereitet.

---

### `VoucherTemplate`

Das Modell `VoucherTemplate` dient zur Verwaltung der Basis-Vorlagen für Gutscheine.

Verwendete Felder:

- `name`
- `description`
- `content`
- `validity_days`
- `logo_path`
- `signature_path`
- `is_active`

Beziehungen:

- Ein Template kann für vielen Gutscheine verwendet werden
- Ein Template kann viele personalisierte Varianten besitzen

Dieses Modell bildet die Grundlage für die zentral verwalteten Gutschein-Vorlagen durch den Administrator.

---

### `PersonalizedTemplate`

Das Modell `PersonalizedTemplate` speichert individuelle Anpassungen von Basis-Templates durch Lehrpersonen.

Verwendete Felder:

- `base_template_id`
- `teacher_id`
- `content`

Beziehungen:

- Eine Personalisierung gehört zu einem Basis-Template
- Eine Personalisierung gehört zu einer Lehrperson
- Eine Personalisierung kann in mehreren Gutscheinen verwendet werden

Damit kann ein Lehrer ein bestehendes Template individuell anpassen, ohne das Original des Administrators zu verändern.

---

## Bisher umgesetzte Controller

Neben den Modellen wurden bereits mehrere Controller für die Benutzer- und Authentifizierungslogik implementiert.

---

### `SetupController`

Der `SetupController` dient der einmaligen Ersteinrichtung des Systems.

Funktionen:

- Anzeige der Setup-Seite beim ersten Systemstart
- Validierung der Eingabedaten für den ersten Benutzer
- Erstellung des ersten Benutzerkontos als Administrator
- automatischer Login nach erfolgreicher Registrierung
- Weiterleitung auf das Dashboard

Dabei wird der erste Benutzer mit folgenden Standardwerten erstellt:

- Rolle: `admin`
- Status: `active`
- Berechtigung zur Gutscheinerstellung: aktiviert
- Berechtigung zur Gutscheineinlösung: aktiviert

Dieser Controller stellt sicher, dass die öffentliche Registrierung nur zur initialen Einrichtung des Systems verwendet wird.

---

### `LoginController`

Der `LoginController` verwaltet den Login und Logout der Benutzer.

Funktionen:

- Anzeige der Login-Seite
- automatische Weiterleitung zur Setup-Seite, wenn noch kein Benutzer existiert
- Validierung von E-Mail und Passwort
- Prüfung, ob der Benutzer existiert
- Prüfung des Passwort-Hashs
- Prüfung, ob der Benutzerstatus auf `active` gesetzt ist
- Anmeldung des Benutzers
- Abmeldung des Benutzers

Dadurch wird verhindert, dass eingeladene oder deaktivierte Benutzer bereits auf das System zugreifen können.

---

### `InvitationController`

Der `InvitationController` bildet die Grundlage für den Einladungsprozess neuer Lehrpersonen.

Funktionen:

- Aufruf eines Einladungslinks über Token
- Suche des zugehörigen Benutzers anhand des Einladungstokens
- Prüfung, ob der Benutzer noch den Status `invited` besitzt
- Prüfung, ob der Einladungslink noch gültig ist
- Anzeige der Passwort-Vergabe-Seite
- Validierung des neuen Passworts
- Speichern des Passworts
- Aktivierung des Benutzerkontos
- Entfernen von Token und Ablaufdatum
- automatischer Login nach erfolgreicher Aktivierung

Damit wurde der im Konzept vorgesehene Einladungs-Workflow technisch vorbereitet.

---

### `PasswordResetController`

Der `PasswordResetController` setzt die Passwort-Zurücksetzung um.

Funktionen:

- Anzeige der Seite „Passwort vergessen“
- Validierung der eingegebenen E-Mail-Adresse
- Versand eines Reset-Links
- Anzeige der Reset-Seite über Token
- Validierung von Token, E-Mail und neuem Passwort
- Speichern des neuen Passworts
- Weiterleitung zurück zur Login-Seite nach erfolgreicher Änderung

Damit können Benutzer ihr Passwort selbstständig zurücksetzen, ohne dass ein Administrator eingreifen muss.

---
