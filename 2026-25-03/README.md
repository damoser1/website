---
author: David Moser
label: 25. März 2026
date: 25. März 2026
order: 94
---

# Bugfixes & Verbesserungen

## Behobene Fehler

**Gutschein-Erstellung (View-Problem)**  
Die Seite war nicht erreichbar, weil die Datei im falschen Ordner lag.

**Template-Auswahl**  
Die Option „Template auswählen“ konnte fälschlich abgeschickt werden. Jetzt ist sie deaktiviert und nur als Platzhalter sichtbar.

**Custom Benachrichtigugnen statt Browser-Benachrichtigungen**  
Alle `confirm()`-Dialoge wurden durch einheitliche Benachrichtigungen ersetzt (z. B. beim Löschen oder erneuten Einladen).  
Mit Overlay, klarer Beschriftung und ESC schließbar.

**Quick-Add für Schüler**  
Schüler können jetzt direkt beim Gutschein-Erstellen hinzugefügt werden (ohne Seitenwechsel).  

---

## Platzhalter-System

**Neue Platzhalter:**  
{Name}, {Klasse}, {Datum}, {Ausstellungsdatum}, {Ablaufdatum}, {Schuljahr}, {Begründung}, {Zusatztext}

**Verbesserte Anzeige**  
Alle Platzhalter werden jetzt übersichtlich in einer Box mit Beschreibung angezeigt.

---

## UI-Verbesserungen

- Rollen jetzt auf Deutsch („Administrator“, „Lehrer“)  
- Hover-Effekt für alle Tabellen  
- Hinweis + Link, wenn Daten fehlen (z. B. keine Templates/Schüler)  
- Neues Feld: „Gültigkeit in Tagen“ bei Templates

---

## Wichtige Änderungen

- `VoucherPdfService`: neue Platzhalter  
- Views: Quick-Add, Modals, bessere UI & Struktur  

---

## Offene Punkte

- Archivierung & automatische Löschung  
- Fehlerhandling bei E-Mails  
- Suche & Filter  
- PDF-Vorschau  
- Storage-Link & TCPDF einrichten  