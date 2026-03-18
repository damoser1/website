---
author: David Moser
label: 18. März 2026
date: 18. März 2026
order: 95
---

# Technische Umsetzung

## Middleware

Es wurden mehrere Middlewareklassen erstellt

- **EnsureSetupIsNeeded** – Erlaubt die Ersteinrichtung nur solange noch kein Benutzer existiert.
- **EnsureIsActive** – Loggt deaktivierte Benutzer automatisch aus.
- **EnsureIsAdmin** – Beschränkt Admin-Bereiche auf Administratoren.
- **EnsureCanCreateVouchers** – Prüft ob der Benutzer Gutscheine erstellen darf.
- **EnsureCanRedeemVouchers** – Prüft ob der Benutzer Gutscheine einlösen darf.

Admins haben bei den Berechtigungsprüfungen immer Zugriff.

## Mail

Es gibt zwei E-Mail-Typen:

- **InvitationMail** – Wird beim Anlegen eines neuen Benutzers verschickt und enthält einen Einladungslink zur Passwortvergabe.
- **VoucherMail** – Verschickt einen bestätigten Gutschein als PDF-Anhang an den Empfänger.

Beide sind bereits für Queue-Verarbeitung vorbereitet, laufen aktuell aber synchron.

## Service

Der **VoucherPdfService** erzeugt Gutscheine als PDF. Er nimmt den Template-Text, ersetzt die Platzhalter mit den konkreten Daten und baut daraus ein fertiges PDF mit Logo, Inhalt, Infos, Unterschrift und QR-Code zum Einlösen.

## Einladungs-Flow

Neue Benutzer werden vom Admin angelegt und bekommen eine Einladungs-E-Mail. Über den Link darin setzen sie ihr Passwort und werden direkt eingeloggt. Die Einladung läuft nach 7 Tagen ab und kann bei Bedarf erneut versendet werden.

## UI-Redesign

Alle Views wurden optisch überarbeitet. Ziel war ein einheitliches, ruhiges Erscheinungsbild mit durchgängigem Farbschema: graue Eingabefelder ohne Rahmen, dezente Labels, Indigo als Akzentfarbe für Buttons und Links.

Die Navigation wurde zweizeilig aufgebaut — Logo und Benutzername oben, gruppierte Navigationslinks unten mit Hervorhebung der aktiven Seite.

