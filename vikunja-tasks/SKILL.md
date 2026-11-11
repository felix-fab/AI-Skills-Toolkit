---
name: vikunja-tasks
description: >-
  Erstellen von Vikunja-Tasks über den Vikunja-MCP-Server (Tools vikunja__*).
  Immer verwenden, wenn der Nutzer eine Aufgabe in Vikunja anlegen, abändern oder
  planen will. Konventionen: Titel beginnt mit "[Studium Modul oder Projekt]", immer
  Fälligkeit klären, automatisch Benachrichtigung 4 Tage vorher, Beschreibung mit
  Details füllen und das Projekt (Bereich) erfragen bzw. aus dem Kontext ableiten.
metadata:
  opencode/autoinvoke: true
---

# Vikunja-Tasks

Erstellt saubere, handlungsfähige Tasks im Vikunja-Instanz des Nutzers über den
**Vikunja MCP-Server** (Tools unter dem Namensraum `vikunja__*`, z.B.
`tasks_create`, `tasks_update`, `tasks_list`, `projects_list`, `labels_list`,
`users_search`). Prüfe vor jedem Aufruf die **live verfügbaren Tool-Schemas**
dieser Session – Parameternamen und Pflichtfelder können abweichen.

## Warum diese Regeln

Ein Task ist nur nützlich, wenn er sofort wiedererkennbar ist (Titel), einen klaren
Zeitpunkt hat (Fälligkeit) und ohne Rückfrage abarbeitbar ist (Beschreibung).

## Ablauf beim Anlegen

**1. Projekt (Bereich) ermitteln.** Rufe `projects_list` auf (mit `format:
"markdown"`, `fields: ["id", "title"]`). Erkennst du das Projekt eindeutig aus dem
Kontext, wähle es direkt und nenne es kurz. Bist du unsicher, frage einmalig nach
und gib die vorhandenen Projekte als Auswahl an. Wähle nie stillschweigend ein
Projekt, wenn mehrere plausibel sind.

**2. Titel.** Beginne, wann immer sinnvoll, mit `[Studium Modul oder Projekt]
<kurze Aufgabenbeschreibung>` – in der Klammer steht das betroffene Modul bzw.
Projekt (z.B. `[Mathe]`, `[Home Server]`, `[Netzwerk]`), danach folgt eine kurze,
handlungsorientierte Aufgabenbeschreibung (Verb + Objekt). Ist das Muster
unpassend, beginne trotzdem mit einem aussagekräftigen Titel und erkläre kurz
warum.

**3. Fälligkeit.** Setze immer ein `due_date` (ISO-8601 Datetime). Ist das Datum
aus dem Kontext ableitbar, übernimm es; sonst frage einmalig nach. Fehlt eine
Uhrzeit, verwende das Tagesende (23:59) als Standard und erwähne das.

**4. Benachrichtigung 4 Tage vorher.** Setze automatisch einen Reminder über das
`reminders`-Feld mit `relative_to: "due_date"` und `relative_period: -345600`
(=-4 Tage in Sekunden; negativ = vor dem Bezugspunkt). Der relative Modus wandert
bei verschobener Fälligkeit automatisch mit. Falls das Schema nur einen absoluten
`reminder`-Timestamp zulässt, verwende Fälligkeit minus 4 Tage. Unterstützt der
Task keine Reminder, weise darauf hin und vermerke die Erinnerung als ersten Punkt
in der Beschreibung.

**5. Beschreibung.** Fülle sie (nicht leer) mit `format: "markdown"`, sofern
sinnvoll strukturiert: Ziel/Was, Schritte, Kontext (betroffenes System, Abhängig-
keiten), Definition of Done. Nutze bekannte Fakten aus dem Gespräch; erfinde nie
Details – fehlt Kontext, frage gezielt nach oder lasse den Abschnitt weg.

**6. Anlegen & bestätigen.** Rufe `tasks_create` auf mit `project` (numerische
ID), `title`, `description`, `due_date`, `reminders`. Bestätige danach kurz:
Titel, Projekt, Fälligkeit und die Erinnerung („Erinnerung 4 Tage vorher").

## Umgang mit bestehenden Tasks

- **Suchen:** `tasks_list` mit `format: "markdown"` und ggf. Filter
  (siehe `filter_syntax`).
- **Ändern:** `tasks_update` (PATCH-Semantik – nur angegebene Felder). Passt der
  Reminder an, wenn sich die Fälligkeit ändert (relativ bleibt er korrekt).
- **Löschen:** `tasks_delete` (verschiebt in den Papierkorb).

## Kurzprüfung vor dem Anlegen

- Titel beginnt (wenn sinnvoll) mit `[Studium Modul oder Projekt]`?
- Fälligkeit gesetzt?
- Reminder 4 Tage vor Fälligkeit konfiguriert?
- Beschreibung mit Kontext/Schritten gefüllt?
- Projekt (Bereich) eindeutig gewählt bzw. bestätigt?
