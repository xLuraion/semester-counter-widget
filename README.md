# Semester Counter – Scriptable Widget

Ein minimalistisches iPhone-Widget für wichtige Semesterzeiträume, Termine und Countdowns.

Das Widget funktioniert automatisch in **Small**, **Medium** und **Large**.

## Einrichtung

Du brauchst nur die kostenlose App **Scriptable**.

### 1. Scriptable installieren

Installiere Scriptable aus dem App Store und öffne die App einmal.

https://apps.apple.com/app/scriptable/id1405459188

### 2. Widget-Script kopieren

1. Öffne die `.js`-Datei aus diesem Repository.
2. Kopiere den kompletten Inhalt.
3. Öffne Scriptable.
4. Tippe oben rechts auf **+**.
5. Füge den Code ein.
6. Gib dem Script z. B. den Namen **Semester Counter**.

Du musst im Script nichts ändern.

Standardmäßig lädt das Widget die Semester-Daten von:

```text
https://lukasedelmann.com/semester.json
```

### 3. Widget testen

Starte das Script in Scriptable mit dem **Play-Button**.

Danach kannst du direkt eine Vorschau auswählen:

- Small
- Medium
- Large

Wenn die Vorschau erscheint, ist alles eingerichtet.

### 4. Widget zum Homescreen hinzufügen

1. Halte den Homescreen gedrückt.
2. Wähle **Widget hinzufügen**.
3. Suche nach **Scriptable**.
4. Wähle die gewünschte Größe: Small, Medium oder Large.
5. Füge das Widget hinzu.
6. Halte das Widget gedrückt und wähle **Widget bearbeiten**.
7. Wähle bei **Script** dein eben angelegtes **Semester Counter** Script aus.

Fertig.

Du kannst dasselbe Script gleichzeitig für mehrere Widget-Größen verwenden. Die Darstellung passt sich automatisch an.

## Was wird angezeigt?

Je nach Widget-Größe werden unter anderem angezeigt:

- aktuell laufende wichtige Zeiträume
- der nächste wichtige Zeitraum oder Termin
- weitere kommende Termine
- ein Countdown in Tagen
- im Large-Widget zusätzlich der Semesterfortschritt

---

# Optional für Kenner

## Eigene Datenquelle verwenden

Wenn du nicht die Standard-Datenquelle verwenden möchtest, kannst du im Script diese Zeile ändern:

```javascript
const DATA_URL =
  "https://lukasedelmann.com/semester.json";
```

Ersetze die URL einfach durch deine eigene öffentlich erreichbare JSON-Datei.

Beispiel:

```javascript
const DATA_URL =
  "https://example.com/semester.json";
```

## Aufbau der `semester.json`

Beispiel:

```json
{
  "semester": "WiSe 2026/27",
  "timezone": "Europe/Berlin",
  "semesterStart": "2026-09-14",
  "semesterEnd": "2027-02-28",
  "items": [
    {
      "type": "period",
      "title": "Prüfungsanmeldung",
      "start": "2026-09-01",
      "end": "2026-09-09",
      "category": "important",
      "priority": 10
    },
    {
      "type": "period",
      "title": "Interdisziplinäre Woche",
      "start": "2026-09-28",
      "end": "2026-10-02",
      "category": "semester",
      "priority": 8
    },
    {
      "type": "event",
      "title": "Mathe-Klausur",
      "start": "2027-01-21",
      "time": "09:00",
      "category": "exam",
      "priority": 10
    }
  ]
}
```

### Eintragstypen

**Zeitraum**

```json
{
  "type": "period",
  "title": "Prüfungsanmeldung",
  "start": "2026-09-01",
  "end": "2026-09-09",
  "category": "important",
  "priority": 10
}
```

**Einzeltermin**

```json
{
  "type": "event",
  "title": "Klausur",
  "start": "2027-01-21",
  "time": "09:00",
  "category": "exam",
  "priority": 10
}
```

`time` ist bei Einzelterminen optional.

Eine höhere `priority` macht einen Eintrag wichtiger, falls mehrere Einträge gleichzeitig aktiv sind.

## Hinweis zur Aktualisierung

iOS entscheidet selbst, wann Homescreen-Widgets aktualisiert werden. Das Script fordert regelmäßig neue Daten an, ein exakter Aktualisierungszeitpunkt kann von Scriptable aber nicht garantiert werden.
