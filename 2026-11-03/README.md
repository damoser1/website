---
author: David Moser
label: 11. März 2026
date: 11. März 2026
order: 96
---
 
# Erweiterung der technischen Planung – Models, Controller und Systemlogik
 
## Überblick
 
Die Backend-Struktur wurde in vier Controller-Bereiche aufgeteilt:
 
- `Web\Auth` – Authentifizierung und Ersteinrichtung
- `Web\Shared` – Gemeinsame Funktionen für Admin und Lehrer
- `Web\Admin` – Administratorspezifische Verwaltung
- `Web\Teacher` – Lehrerspezifische Funktionen
 
---
 
## Models
 
### User
 
Repräsentiert Administratoren und Lehrpersonen.
 
Felder: `name`, `email`, `password`, `role`, `can_create_vouchers`, `can_redeem_vouchers`, `status`, `invitation_token`, `invitation_expires_at`
 
Beziehungen: `issuedVouchers`, `redeemedVouchers`, `createdStudents`, `personalizedTemplates`
 
### Student
 
Repräsentiert die Gutschein-Empfänger. Kein Login, Klasse wird nicht am Schüler gespeichert.
 
Felder: `name`, `email`, `created_by`
 
Beziehungen: `vouchers`, `creator`
 
### Voucher
 
Zentrale Entität – verbindet Templates, Schüler, Aussteller und Einlösung.
 
Felder: `voucher_number`, `qr_token`, `template_id`, `personalized_template_id`, `issuer_id`, `student_id`, `redeemed_by`, `class_name`, `reason`, `additional_text`, `status`, `rejection_reason`, `batch_id`, `issued_at`, `expires_at`, `sent_at`, `redeemed_at`, `archived_at`, `retention_until`
 
Beziehungen: `template`, `personalizedTemplate`, `issuer`, `student`, `redeemer`
 
### VoucherTemplate
 
Vom Administrator verwaltete Gutschein-Vorlagen.
 
Felder: `name`, `description`, `content`, `validity_days`, `logo_path`, `signature_path`, `is_active`
 
Beziehungen: `vouchers`, `personalizedTemplates`
 
### PersonalizedTemplate
 
Vollständige Kopie eines Basis-Templates, frei anpassbar durch Lehrer.
 
Felder: `base_template_id`, `teacher_id`, `content`
 
Beziehungen: `baseTemplate`, `teacher`, `vouchers`
 
---
 
## Controller
 
### Auth
 
| Controller | Funktion |
|---|---|
| SetupController | Ersteinrichtung – erster User wird Admin |
| LoginController | Login/Logout, Weiterleitung auf Setup wenn kein User existiert |
| InvitationController | Einladungslink annehmen, Passwort setzen, Account aktivieren |
| PasswordResetController | Passwort-vergessen und Reset |
 
### Shared
 
| Controller | Funktion |
|---|---|
| DashboardController | Rollenbasiertes Dashboard (Admin: ausstehende Gutscheine, Lehrer: eigene Gutscheine) |
| StudentController | CRUD Schüler, Lösch-Schutz bei aktiven Gutscheinen |
| VoucherCreateController | Gutschein-Erstellung (Einzel + Stapel mit Batch-ID) |
| RedeemController | Einlösung über QR-Token, Status- und Ablaufprüfung |
 
### Admin
 
| Controller | Funktion |
|---|---|
| VoucherController | Alle Gutscheine verwalten, bestätigen, ablehnen, zurücksetzen |
| TemplateController | CRUD Templates mit Logo/Unterschrift-Upload |
| UserController | Benutzer anlegen mit Einladungs-E-Mail, Berechtigungen, deaktivieren/löschen |
| ArchiveController | Archivierte Gutscheine anzeigen |
 
### Teacher
 
| Controller | Funktion |
|---|---|
| MyVoucherController | Eigene Gutscheine einsehen, abgelehnte korrigieren und erneut einreichen |
| PersonalizedTemplateController | Templates personalisieren, bearbeiten, löschen |
 
 
---
 
## Offene Punkte
 
- Automatische Archivierungslogik
- Automatische Löschlogik nach Aufbewahrungsfrist
- Fehlerbehandlung bei fehlgeschlagenem E-Mail-Versand
- Such- und Filterfunktionen in den Verwaltungsansichten