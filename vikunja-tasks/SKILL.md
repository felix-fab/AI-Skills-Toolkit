---
name: vikunja-tasks
description: >-
  Erstellen von Vikunja-Tasks über den Vikunja-MCP-Server (Tools vikunja__*).
  Immer verwenden, wenn der Nutzer eine Aufgabe/Task in Vikunja anlegen, abändern
  oder planen will – egal ob er "Vikunja", "Task anlegen", "ToDo", "Erinnerung an mich"
  oder ein konkretes Vorhaben nennt. Konventionen: Titel beginnt mit
  "[Modul, System|App] TODO", immer Fälligkeit klären, automatisch Benachrichtigung
  4 Tage vorher setzen, Beschreibung mit Kontext/Details füllen und das Projekt
  (Bereich) erfragen bzw. aus dem Kontext ableiten.
metadata:
  opencode/autoinvoke: true
---

# Vikunja-Tasks

Erstellt saubere, handlungsfähige Tasks im Vikunja-Instanz des Nutzers über den
**Vikunja MCP-Server**. Der Server ist als `streamableHttp`-Endpoint angebunden
(`https://felix-home-server.tail436c23.ts.net:450/api/v2/mcp`) und stellt seine
Tools unter dem Namensraum `vikunja__*` bereit. In dieser Session sind das u.a.:

- `vikunja__tasks_create` / `tasks_update` / `tasks_list` / `tasks_read`
- `vikunja__projects_list` / `projects_read` / `projects_create`
- `vikunja__labels_list` / `labels_create`
- `vikunja__users_search`
- `vikunja__find_action` / `do_action` / `filter_syntax`

Bevor du einen Tool-Aufruf machst, prüfe die **live verfügbaren Tool-Schemas** in
dieser Session – die konkreten Parameternamen und Pflichtfelder können abweichen.

## Zweck und warum diese Regeln existieren

Ein Vikunja-Task ist nur nützlich, wenn er drei Dinge erfüllt: Er ist **sofort
wiedererkennbar** (Titelkonvention), er hat einen **klaren Zeitpunkt** (Fälligkeit)
und er ist **ohne Rückfrage abarbeitbar** (Kontext in der Beschreibung). Die
Regeln unten sorgen genau dafür – sie verhindern vage Titel, Tasks ohne Datum und
leere Beschreibungen, die den Nutzer später zum Nachfragen zwingen.

## Ablauf bei "Task anlegen"

Arbeite die folgenden Schritte der Reihe nach ab. Frage nur das wirklich
Notwendige, und nur wenn du es nicht aus dem Kontext ableiten kannst.

### 1. Kontext und Projekt (Bereich) ermitteln

Der Task muss in einem **Projekt (Bereich)** angelegt werden. Gehe so vor:

1. Rufe `vikunja__projects_list` mit `format: "markdown"` und
   `fields: ["id", "title"]` auf, um die aktuell verfügbaren Projekte zu sehen.
2. **Erkennst du eindeutig aus dem Nutzerkontext**, welches Projekt gemeint ist,
   wähle es direkt und nenne es kurz in deiner Antwort (z.B. "Ich lege das in
   ›Home Server‹ an").
3. **Bist du dir unsicher** oder nennst der Nutzer keinen Bereich, frag einmalig
   nach, in welchem Projekt der Task landen soll. Gib die vorhandenen Projekte als
   Auswahl an (z.B. Privat, Studium, Humera, Home Server, Penetration Testing).
   Wähle **nie stillschweigend** ein Projekt, wenn mehrere plausibel sind.

> [!tip]
> Nutzer sagen selten explizit "Projekt". Erkenne es aus dem Inhalt: Home-Server-
> und Netzwerk-Themen → „Home Server", Prüfungs-/Vorlesungsthemen → „Studium",
> Pentest-/CTF-Themen → „Penetration Testing", Alltag/Familie → „Privat".

### 2. Titel mit Konvention

Der Tasktitel **beginnt, wann immer möglich**, mit dem Muster
`[Modul, System|App] TODO <Kurzfassung>`:

```
[Home Server, Vikunja] TODO Backup der Datenbank einrichten
[Studium, Mathe] TODO Übungsblatt 5 bearbeiten
```

- `Modul`/`System|App` = der Bereich bzw. das System, das der Task betrifft
  (z.B. Home Server, Vikunja, Nextcloud, Mathe, Netzwerk).
- `TODO` = wörtlich so übernehmen – es macht den Taskstatus auf einen Blick klar.
- Danach eine **prägnante, handlungsorientierte** Kurzbeschreibung (Verb + Objekt).

Wenn sich das Muster nicht sinnvoll anwenden lässt (z.B. ein rein privater, nicht
systembezogener Task), beginne trotzdem mit einem aussagekräftigen Titel und
erkläre dem Nutzer kurz, warum du die Konvention weggelassen hast.

### 3. Fälligkeit immer klären

Ein Vikunja-Task bekommt **immer** eine Fälligkeit (`due_date`). Frage nicht "ob",
sondern nach dem konkreten Zeitpunkt:

- Ist das Datum (und ggf. die Uhrzeit) aus dem Nutzerkontext ableitbar, übernimm es.
- Sonst frage einmalig nach dem Fälligkeitsdatum. Fehlt eine Uhrzeit, verwende das
  Tagesende (23:59) als sinnvollen Standard und erwähne das.

Formatiere `due_date` als ISO-8601 Datetime (z.B. `2026-01-15T23:59:00`). Ohne
Fälligkeit ist ein Task wertlos – halte hier nicht ein, bis ein Datum steht.

### 4. Benachrichtigung 4 Tage vor Fälligkeit

Setze **automatisch** einen Vikunja-Reminder **4 Tage vor der Fälligkeit**, damit
der Nutzer genug Vorlauf hat. Nutze das `reminders`-Feld von `tasks_create`:

```json
"reminders": [
  {
    "relative_to": "due_date",
    "relative_period": -345600
  }
]
```

- `relative_period` ist in **Sekunden**: `-4 * 24 * 3600 = -345600` (negativ = vor
  dem Bezugspunkt).
- `relative_to: "due_date"` koppelt den Reminder an die Fälligkeit, sodass er bei
  einer verschobenen Fälligkeit automatisch mitwandert – das ist robust.
- Alternativ geht ein absoluter `reminder`-Timestamp (Fälligkeit minus 4 Tage),
  falls das Schema nur das zulässt.

> [!note]
> Vikunja wertet `relative_period` relativ zum `relative_to`-Zeitpunkt aus. Für
> "4 Tage vorher" ist der negative Sekundenwert die sauberste, selbsterklärende
> Lösung. Falls das Schema des Tasks `reminders` nicht unterstützt, weise den
> Nutzer darauf hin und vermerke die Erinnerung stattdessen als ersten Punkt in
> der Beschreibung.

### 5. Beschreibung mit Details füllen

Die Beschreibung ist **kein leeres Feld**. Fülle sie mit allem, was nötig ist,
damit der Task ohne Rückfrage abarbeitbar ist. Nutze `format: "markdown"` und
strukturiere mit Überschriften/Listen, wo sinnvoll:

- **Ziel / Was** – das konkrete Ergebnis in einem Satz.
- **Schritte** – eine nummerierte Checkliste der Teilaufgaben, wenn du sie kennst.
- **Kontext** – warum der Task existiert, betroffenes System/Modul, Abhängigkeiten.
- **Definition of Done** – woran man erkennt, dass der Task fertig ist.

Hebe bekannte Fakten aus dem Gespräch auf (Server, Pfad, Versionen, Personen).
Erfinde **nie** Details; wenn dir Kontext fehlt, frage gezielt nach oder lasse den
Abschnitt weg, statt zu raten.

### 6. Task anlegen und bestätigen

Rufe `vikunja__tasks_create` auf mit `project` (numerische ID des gewählten
Projekts), `title`, `description` (`format: "markdown"`), `due_date` und
`reminders`. Nach erfolgreichem Anlegen bestätigst du dem Nutzer kurz und
strukturiert:

- Titel des Tasks
- Projekt (Bereich)
- Fälligkeit
- Bestätigung der Erinnerung ("Erinnerung 4 Tage vorher gesetzt")

Beispiel-Antwort:

> ✅ Task angelegt in ›Home Server‹
> **[Home Server, Vikunja] TODO Backup der Datenbank einrichten**
> Fällig: 15.01.2026, 23:59 · Erinnerung: 4 Tage vorher (11.01.2026)

## Umgang mit bestehenden Tasks

- **Task suchen**: `vikunja__tasks_list` mit `format: "markdown"` und ggf. einem
  Filter (siehe `vikunja__filter_syntax`).
- **Task aktualisieren** (Fälligkeit, Beschreibung, Projekt): `vikunja__tasks_update`
  mit `PATCH`-Semantik – nur die Felder ändern, die angegeben sind. Vergiss nicht,
  den Reminder anzupassen, wenn sich die Fälligkeit ändert (relativ bleibt er
  automatisch korrekt).
- **Task löschen**: `vikunja__tasks_delete` (verschiebt in den Papierkorb).

## Qualitätssicherung vor dem Anlegen

Kurzprüfung, bevor du `tasks_create` aufrufst:

- Beginnt der Titel (wenn sinnvoll) mit `[Modul, System|App] TODO`?
- Ist eine Fälligkeit gesetzt?
- Ist der Reminder 4 Tage vor Fälligkeit konfiguriert?
- Ist die Beschreibung mit Kontext/Schritten gefüllt (nicht leer)?
- Ist das Projekt (Bereich) eindeutig gewählt bzw. bestätigt?
