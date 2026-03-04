---
author: David Moser
label: Doku 04. März 2026
date: 04. März 2026
---

# Technische Umsetzung

## Datenbankmodell

...

## Allgemeines UI-Konzept

Die Benutzeroberfläche ist rollenbasiert aufgebaut, wobei nur die für die jeweilige Rolle relevanten Funktionen angezeigt werden. 
Nach dem Login wird der Benutzer automatisch auf das rollenspezifische Dashboard weitergeleitet.

---

### Ersteinrichtung

Beim erstmaligen Aufruf des Systems (kein Benutzer vorhanden) wird automatisch die Registrierungsseite angezeigt:

- Nach erfolgreicher Registrierung wird der Account als Administrator gespeichert und der Benutzer auf das Admin-Dashboard weitergeleitet
- Die Registrierungsseite ist danach dauerhaft deaktiviert und nicht mehr erreichbar. Bei erneutem Aufruf der URL erfolgt eine Weiterleitung auf die Login-Seite.

---

### Benutzeroberfläche Administratoren

**Navigation:**

- Dashboard
- Gutscheine (Übersicht, Erstellen, Suche)
- Templates
- Schüler
- Benutzer/Berechtigungen
- Archiv

**Funktionen im Detail:**

**Dashboard**

- Gesamtübersicht über Gutscheine nach Status
- Anzahl ausstehender Bestätigungen mit Link zur Bestätigungsansicht
- Schnellzugriff auf zuletzt erstellte Gutscheine zum ändern der Gutscheine
- Hinweise auf ablaufende oder abgelaufene Gutscheine
- Schnellaktion-Buttons: „Neuen Gutschein erstellen", „Ausstehende bestätigen", etc.

**Gutscheine - Übersicht**

- Tabellarische Darstellung aller Gutscheine
- Filter- und Sortierfunktionen
- Suchfeld
- Klick auf Gutschein öffnet eine Detailansicht

**Gutscheine - Detailansicht**

- Alle Gutscheininformationen auf einem Blick und hervorheben des Status
- Aktionsbuttons je nach aktuellem Status
    - Ausstehend: „Bestätigen" / „Ablehnen"
    - Bestätigt: „Erneut senden" (bei Versandfehler)
    - Gesendet: „Einlösen" / „Als abgelaufen markieren"
    - Abgelehnt: „Dauerhaft löschen"
- Zurücksetzten einen bereits gesendeten Gutschein zurück (z. B. bei fehlerhaften Angaben). Möglichkeit zur nachträglichen Bearbeitung solange der Gutschein noch nicht  eingelöst wurde.

**Gutscheine - Erstellen**

- **Schritt 1**: Template auswählen
- **Schritt 2**: Empfänger wählen – drei Optionen:
    - Einzelnen Schüler auswählen (Suchfeld Name). Wird kein passender Schüler gefunden, kann ein neuer Schüler erstellt werden (Verlinkung).
    - Mehrere Schüler auswählen (Checkboxen zum auswählen).
- **Schritt 3**: Gutschein Daten erfassen (Pflicht- und Optionalfelder)
- **Schritt 4**: Vorschau des fertigen Gutscheins (PDF-Vorschau) oder Stapelansicht einer Zusammenfassung
- **Schritt 5** Absenden zur Bestätigung (Status wird gesetzt)
- Bei Stapel-Erstellung wird eine Zusammenfassung angezeigt vor dem Absenden

**Gutscheine - Bestätigungsansicht**

- Liste aller Gutscheine mit Status „Ausstehend", gruppiert nach Einzel- und Stapel-Erstellungen
- Bei Stapel-Gutscheinen: Anzeige mit Möglichkeit, den gesamten Batch oder einzelne Gutscheine zu bestätigen/abzulehnen
- Ablehnungsgrund als Pflichtfeld bei Ablehnung
- Nach Bestätigung: automatischer Versand wird ausgelöst, Statusanzeige aktualisiert sich

**Templates**

- Übersicht aller Templates bzw Vorschau in kompakter Ansicht
- Statusanzeige: aktiv / inaktiv bzw Gruppierung
- Template erstellen / bearbeiten:
    - Name und Beschreibung
    - Aufbau mit Platzhaltern
    - Layout-Einstellungen (Logo-Upload, Unterschrift-Upload)
    - Aktiv/Inaktiv-Schalter
- Vorschau Funktion
- Templates können deaktiviert, aber nicht gelöscht werden, solange Gutscheine darauf basieren

**Schüler**

- Übersicht aller im System erfassten Schüler als Tabelle
- Suchfeld
- Möglichkeit neuen Schüler anzulegen
- Schüler bearbeiten
- Schüler löschen: Nur möglich, wenn keine aktiven Gutscheine mehr vorhanden sind. Archivierte Gutscheine werden dabei mitgelöscht.
- Hinweis: Schüler werden auch automatisch angelegt, wenn sie bei der Gutschein-Erstellung als neuer Empfänger erfasst werden

**Benutzer / Berechtigungen Lehrer Übersicht**

- Übersicht aller Benutzer (Administor und Lehrpersonen) und ob Berechtigung vorhanden ist
- Berechtigungen zur Gutscheinerstellung und Gutschein-Einlösung vergeben/entziehen
- Möglichkeit, Accounts zu deaktivieren. Der Lehrer kann sich nicht mehr einloggen, seine Gutscheine bleiben erhalten. Ein deaktivierter Account kann jederzeit wieder aktiviert werden.
- Löschen eines Accounts auch möglich. Nur möglich, wenn keine aktiven Gutscheine mit diesem Lehrer als Aussteller verknüpft sind. Bestätigungsdialog vor dem Löschen.
- Hinweis: Schülerverwaltung erfolgt über den eigenen Menüpunkt

**Benutzer / Berechtigungen Lehrer anlegen**

- Anlegen eines Lehrers
- Optionale Zuweisung von Berechtigungen direkt beim Anlegen (Gutscheinerstellung, Gutscheineinlösung) – können auch nachträglich in der Übersicht vergeben werden
- Nach dem Absenden: System versendet automatisch eine Einladungs-E-Mail mit einmaligem Link zur Passwort-Vergabe
- Der neue Lehrer erscheint in der Übersicht mit Status „Eingeladen"
- Button „Erneut einladen": Für Lehrer mit Status „Eingeladen", deren Link abgelaufen ist – versendet eine neue Einladungs-E-Mail

**Archiv**

- Übersicht aller archivierten Gutscheine mit Filter- und Suchfunktionen
- Anzeige der verbleibenden Aufbewahrungsfrist pro Gutschein
- Hinweis auf automatische Löschung nach Fristablauf

---

#### Benutzeroberfläche Lehrer

**Navigation**

- Dashboard
- Meine Gutscheine (Übersicht, Erstellen)
- Schüler (Übersicht, Anlegen)
- Templates (Personalisieren)
- Gutschein einlösen (nur mit erteilter Berechtigung)

**Funktionen im Detail:**

**Dashboard**

- Übersicht der eigenen Gutscheine nach Status
- Schnellzugriff auf zuletzt erstellte eigene Gutscheine
- Hinweis auf abgelehnte Gutscheine mit Ablehnungsgrund und direktem Link zur Korrektur
- Für Lehrer mit Einlöse-Berechtigung: Anzahl offener Gutscheine

**Meine Gutscheine - Übersicht**

- Ansicht aller seblsterstellten Gutschein
- Filter und Suchfunktion
- Bei abgelehnten Gutscheinen: Anzeige des Ablehnungsgrunds und Button „Korrigieren und erneut einreichen". Öffnet den Gutschein im Bearbeitungsmodus . Nach dem Absenden wechselt der Status zurück auf „Ausstehend".

**Meine Gutscheine - Erstellen**

- Nur sichtbar, wenn die Berechtigung durch den Administrator erteilt wurde. Andernfalls wird ein Hinweis angezeigt.
- Ablauf identisch zur Administrator-Erstellung:
  - Template auswählen (nur aktive Templates sichtbar)
  - Empfänger wählen (Einzelschüler, Mehrfachauswahl). Auch hier kann über „Neuen Schüler anlegen" direkt ein neuer Schüler erfasst werden.
  - Daten erfassen (Begründung, Gültigkeit, Zusatztext)
  - Vorschau
  - Absenden zur Bestätigung
- Individuelle Anpassung des Templates möglich: Der Lehrer kann beim Erstellen den Zusatztext ergänzen und kleinere Änderungen am Textaufbau vornehmen (z. B. persönliche Formulierung), ohne das Basis-Template dauerhaft zu verändern.

**Schüler**

- Übersicht aller im System erfassten Schüler als Tabelle
- Suchfeld
- Möglichkeit neuen Schüler anzulegen
- Schüler bearbeiten
- Schüler löschen: Nur möglich, wenn keine aktiven Gutscheine mehr vorhanden sind. Archivierte Gutscheine werden dabei mitgelöscht.
- Hinweis: Schüler werden auch automatisch angelegt, wenn sie bei der Gutschein-Erstellung als neuer Empfänger erfasst werden

**Templates - Personalisieren**

- Übersicht aller aktiven Templates mit Vorschau
- Lehrer können eine persönliche Kopie eines Templates anlegen und dort individuelle Anpassungen vornehmen
- Das Basis-Template des Administrators bleibt dabei unverändert
- Personalisierte Templates werden beim Erstellen eines Gutscheins zusätzlich zu den Standard-Templates zur Auswahl angeboten
- Personalisierte Templates können jederzeit bearbeitet oder wieder gelöscht werden

**Gutschein einlösen**

- Nur sichtbar für Lehrpersonen, denen die Einlöse-Berechtigung vom Administrator erteilt wurde
- Zwei Einlösungs-Methoden:
    - Der Lehrer scannt den QR-Code mit der Kamera seines Smartphones – der Link öffnet die Webapp direkt auf der Einlöse-Seite des jeweiligen Gutscheins.
    - Gutschein-ID eingeben: Eingabefeld für manuelle Eingabe der Gutscheinnummer 
- Nach Öffnen des Links: Anzeige der Gutschein-Details zur Überprüfung
- Validierung: System prüft automatisch, ob der Gutschein gültig ist
- Bestätigungsbutton: „Einlösung bestätigen" → Status wechselt, Gutschein wird archiviert

---



Passwort Reset für Lehrer???
Sinnvoll ersten Admin account im seeder erstellen oder gleich in der Datenbank?
Wenn bestätigt, dann wird gutschein geschickt? wenn korrigiert, nicht mehr einlösbar, wie erfährt der Schüler das?