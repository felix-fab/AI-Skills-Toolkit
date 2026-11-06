---
name: Markdown-Notizen
description: Regeln für Markdown-Notizen in diesem Vault. Verwenden beim Erstellen, bearbeiten oder umformatieren von Notizen - kein H1-Titel mit Dateinamen, keine nummerierten Überschriften, keine Icons, interne Links als Wikilinks, Callouts für Hinweise, Commands in shell-session Blöcken.
metadata:
  opencode/autoinvoke: true
---

# Markdown-Notizen

Regeln für alle Markdown-Notizen in diesem Vault.

## Überschriften

- **Kein H1-Titel mit dem Dateinamen.** Die Notiz beginnt direkt mit dem Inhalt oder mit `##`.
- Keine Überschrift, die nur den Dateinamen wiederholt (`Server Setup.md` beginnt **nicht** mit `# Server Setup`).
- **Keine Nummerierung** von Überschriften: kein `## 1. Setup`, kein `### 2.3 Konfiguration`.
- Überschriften beschreiben den Inhalt, nicht ihre Position.

## Icons

- **Keine Icons oder Emojis** in Überschriften, Listen, Tabellen oder Tabellenüberschriften.
- Kein `## 🛠️ Tools`, keine `✅` / `⚠️` / `➡️` in Aufzählungen.

## Interne Links

- Immer Wikilinks: `[[Notiz]]`, `[[Notiz|Anzeigetext]]`, `[[Notiz#Überschrift]]`.
- Link auf eine Überschrift derselben Notiz: `[[#Überschrift]]`.
- Keine Markdown-Links auf interne Dateien (`[Text](Notiz.md)`) – die sind externen URLs vorbehalten.
- **Kein Abschnitt „Verwandte Notizen"** (oder ähnlich benannte Abschnitte wie „Siehe auch") am Ende der Notiz. Interne Links zu verwandten Notizen werden stattdessen direkt im Fließtext gesetzt.

## Callouts

Hinweise werden als Obsidian-Callouts geschrieben, nie als einfaches Zitat `>`.

Verfügbare Typen:

- `[!warning]` – Risiko, Fehlerquelle, Warnung
- `[!tip]` – Tipp oder Best Practice
- `[!info]` – Zusatzinformation
- `[!note]` – neutrale Anmerkung
- `[!example]` – Beispiel
- `[!danger]` – Gefahr, destruktive Aktion
- `[!quote]` – Zitat

```markdown
> [!warning]
> Mit Proxychains `-sT` verwenden, weil nur der TCP-Connect-Scan funktioniert.
```

## Commands

- Jeder Befehl in einem Fenced Block mit der Sprache `shell-session`.
- Kein `bash`, kein `sh`, kein `powershell` für Shell-Befehle.
- Ein Befehl pro Block, mehrzeilige Befehlsfolgen in einem Block.

Beispiel (Fence mit `shell-session` öffnen, Befehl, Fence schließen):

```shell-session
sudo nmap -sS 10.10.0.1
```

- Inline-Befehle, Flags und Pfade in Backticks: `-sS`, `--script=banner`, `/etc/nut/nut.conf`.

## Kurzprüfung vor dem Speichern

- Beginnt die Notiz ohne H1-Dateinamentitel?
- Sind alle Überschriften unnumeriert?
- Keine Icons oder Emojis?
- Alle internen Links in `[[...]]`?
- Kein Abschnitt „Verwandte Notizen" am Ende?
- Alle Hinweise als `[!typ]`-Callout?
- Alle Commands in `shell-session`?
