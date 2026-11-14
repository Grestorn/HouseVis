# 2 – Das Hausmodell erstellen

[← Übersicht](README.md) · [English](../en/2-house-model.md)

> [!TIP]
> **Hängst du fest, oder ist das Modell fertig?** Im Forum hilft man dir unter
> [Hausmodell & Pläne](https://forum.housevis.de/c/hausmodell), und unter
> [Haus-Assistent & Skill-Feedback](https://forum.housevis.de/c/skill-feedback) erzählst du, wie es gelaufen ist
> ([mehr dazu](#hilfe-und-feedback)).

HouseVis braucht ein Modell deines Hauses: Geschosse, Räume, Wände, Fenster und Türen, Dach,
Terrasse, Garage – und welche Loxone-Bausteine zu welchem Raum und Fenster gehören. Du zeichnest
es nicht selbst: **Claude baut es mit dir aus deinen Bauplänen und Fotos**, fragt nach, was die
Pläne offenlassen, und zeigt dir das Ergebnis Schritt für Schritt.

## Welcher Weg?

| | **A: der Haus-Assistent in der App** (empfohlen) | **B: Claude Code mit dem HouseVis-Plugin** |
|---|---|---|
| Du brauchst | einen API-Schlüssel von Anthropic, nach Verbrauch abgerechnet | ein **Claude-Pro**-Abo (oder Max) |
| Kosten | ein ganzes Haus aus seinen Plänen **unter 20 $** (gemessen an einem großen, alles andere als einfachen Haus); eine kleine Änderung wenige Cent | dein Abo, nichts extra |
| Warten | nicht nötig: das Haus in einem Rutsch bauen | mit Pro stößt ein Haus mehrmals an das **5-Stunden-Limit**; du wartest und machst dann weiter |
| Du siehst | das Haus **wächst in der 3D-Ansicht**, während Claude es baut | Grundrisszeichnungen und Bilder von Claude; HouseVis zeigt jeden gespeicherten Schritt |
| Pläne kommen | als Anhang in den Chat | in einen Ordner auf deinem PC |

Beide nutzen dasselbe Wissen und erzeugen dieselben Dateien; du kannst jederzeit wechseln.
**Nimm Weg A** – außer du zahlst schon für Claude Pro oder Max und wartest lieber, als für die API
zu bezahlen.

## 1. Material sammeln

Leg alles in einen Ordner pro Haus, z. B. `C:\Users\<du>\HouseVis-Projekte\plans\mein-haus\`.

| Material | Nötig? |
|---|---|
| Grundriss jedes Geschosses (auch Keller) | **ja** |
| Mindestens ein Schnitt | **ja** |
| Nordrichtung (Nordpfeil, Lageplan oder einfach Claude sagen) | **ja** |
| Ansichten (eine pro Seite) | sehr empfohlen |
| Fotos jeder Fassade | sehr empfohlen |
| Was anders gebaut wurde als geplant, als Textdatei `plan-changes.txt` („Garage 70 cm nach Norden versetzt") | sehr empfohlen |
| Lageplan, eigene Skizze von Terrasse und Wegen | empfohlen |
| Elektroplan, Fotos in der Dämmerung (für Außen- und besondere Leuchten) | optional |

- **Der PDF-Plansatz des Architekten kommt unverändert hinein**, unter beliebigem Namen. Claude
  findet selbst heraus, welche Seite was zeigt.
- **Fotos nach dem benennen, was sie zeigen**: `photo-north-1.jpg` ist die Nordfassade (du stehst
  nördlich vom Haus). Innen: `photo-inside-wohnzimmer.jpg`.
- Formate: PDF, JPG, PNG; Notizen als Textdatei (TXT, MD). Pläne und Fotos bleiben auf deinem PC (siehe [Datenschutz](#datenschutz)).

## 2. Weg A: der Haus-Assistent in der App (empfohlen)

### Einmal einrichten

1. Erstelle einen API-Schlüssel auf [platform.claude.com](https://platform.claude.com)
   (**API Keys**; vorher unter **Billing** ein Zahlungsmittel hinterlegen). Er beginnt mit
   `sk-ant-`.
2. HouseVis: Menü → **KI-Assistent …**, Schlüssel einfügen. Er wird nur auf diesem PC
   verschlüsselt gespeichert.

### Das Haus bauen

1. **Anhängen …** für Pläne, Fotos und Notizen (z. B. `plan-changes.txt`), oder sie auf das
   Fenster ziehen; das Haus in ein paar Worten beschreiben, senden.
2. Claude liest die Pläne, sagt, was da ist und was fehlt, und stellt seine Fragen. „Weiß ich
   nicht" ist eine gute Antwort, dann schätzt Claude.
3. Claude baut das Modell Schritt für Schritt; die 3D-Ansicht folgt jeder Änderung. **Speichern**
   schreibt es; bis dahin nehmen **Rückgängig** und **Verwerfen** Änderungen zurück.

Oben im Fenster stehen die bisherigen Kosten des Gesprächs. Eine kleine Änderung kostet Cent; ein
großes, verwinkeltes Haus aus seinem kompletten Plansatz hat unter 20 $ gekostet, ein einfacheres
kostet weniger. Wer eine feste Obergrenze will, setzt in der Console ein monatliches
Ausgabenlimit.

### Mit Loxone verbinden

1. HouseVis muss mit deinem Miniserver verbunden sein ([Kapitel 1](1-installation.md)).
2. Bitte: *„Ordne Räume, Fenster und Lichter Loxone zu."* Claude liest die Bausteine deines
   Miniservers (nur lesend) und schlägt vor, was wohin gehört; die Live-Werte erscheinen schon vor
   dem Speichern in der 3D-Ansicht. Bestätige oder korrigiere. Fenster können eigene Kontakte
   haben oder alle aus einer **Fenster- und Türüberwachung** kommen.
3. Auf Wunsch setzt Claude auch **Leuchten** (Wegeleuchten, Wandleuchten, eine Pendelleuchte über
   dem Esstisch), **Möbel** als einfache Kästen (Küchenzeile, Oberschränke, Kleiderschrank;
   Lichtleisten sitzen an ihren Kanten, z. B. *„eine Lichtleiste unter den Oberschränken"*) und deine
   **Wetterstation**.
4. **Anzeigen** zeigen einen Zustand als Symbol: Musik läuft oder pausiert, die Waschmaschine
   läuft, Post im Briefkasten. Ein Gerät bekommt eine Anzeige mit einem Symbol je Zustand (*„zeig
   den Briefkasten: ein Brief für Post, ein Paket für ein Paket"*). Meist sitzt sie in der Reihe
   ihres Raums; eine, die zu einem eigenen Ort gehört, schwebt dort (*„zeig den Briefkasten an der
   Haustür"*).

### Spätere Änderungen

Öffne den Haus-Assistenten und sag, was falsch ist, z. B. *„Das Badfenster liegt 20 cm zu weit
rechts"* oder *„Die Garage hat ein Flachdach."* Claude ändert nur das. Links neben dem Assistenten
siehst du das Haus mit jeder Änderung: oben wählst du eine deiner Ansichten oder **Ganzes Haus**
(alle Räume offen, auch neue); ziehen dreht, das Mausrad zoomt, **Zurück zur Ansicht** stellt die
Kamera zurück. Gespeichert wird dabei nichts an den Ansichten.

## 3. Weg B: Claude Code

Für dich, wenn du schon ein Claude-Pro- oder Max-Abo hast und nicht zusätzlich für die API zahlen
willst.

> [!IMPORTANT]
> **Ein Pro-Abo ist das Minimum.** Mit dem kostenlosen Plan reicht das Nutzungslimit für ein Haus
> bei Weitem nicht. Auch mit Pro stößt der Hausbau mehrmals an das 5-Stunden-Limit: Claude hält an
> und sagt, wann es weitergeht. **Einfach warten** und dann fortsetzen – nichts geht verloren,
> Claude führt Notizen und macht dort weiter, wo es aufgehört hat. Wer nicht warten will, nimmt
> Weg A.

### Einmal einrichten

1. Installiere **Claude Code**: am einfachsten die **Claude-Desktop-App**, Reiter **Code**. (Auch
   möglich: im Terminal, siehe [Claude-Code-Schnellstart](https://code.claude.com/docs/en/quickstart),
   oder in VS Code.) Melde dich mit dem Konto an, das das Abo hat.
2. Installiere HouseVis auf demselben PC ([Kapitel 1](1-installation.md)). Claude prüft und
   zeichnet das Modell damit.
3. Installiere das **HouseVis-Plugin** in Claude Code. Das sind **zwei** Befehle, beide sind nötig:

   ```
   /plugin marketplace add Grestorn/HouseVis
   /plugin install house-model@housevis
   ```

   Der erste sagt Claude Code nur, wo das Plugin liegt; erst der zweite installiert es. Prüfe mit
   `/plugin`, dass **house-model** als installiert erscheint, und starte dann eine neue Sitzung.

   **Updates:** Claude Code aktualisiert das Plugin nicht von selbst. Nach einer neuen
   HouseVis-Version also auch `/plugin marketplace update housevis` ausführen und eine neue Sitzung
   starten. Vergisst du es, merkt Claude, dass Plugin und HouseVis verschiedene Versionen haben, und
   erinnert dich daran.

### Das Haus bauen

1. Öffne Claude Code auf dem Ordner, in dem `plans\` liegt (z. B. `HouseVis-Projekte`).
2. Schreib zum Beispiel: *„Erstelle das Hausmodell aus plans/mein-haus."*
3. Claude fragt zuerst, **wohin das Modell kommt**. Normalerweise in den Ordner, den HouseVis zeigt
   (`Dokumente\HouseVis`), dann siehst du das Haus sofort in der App. Liegt dort schon ein Haus,
   überschreibt Claude es nie und schlägt einen anderen Ordner vor; den öffnest du dann in HouseVis
   (Menü → **Konfigurationsordner …**).
4. Claude liest die Pläne, sagt, was da ist und was fehlt, und stellt seine Fragen – alle auf
   einmal. Beantworte sie; „weiß ich nicht" ist eine gute Antwort, dann schätzt Claude und notiert
   es.
5. Claude baut das Haus **Block für Block**: Geschosse und Räume, Wände, Fenster und Türen, Dach,
   Umgebung. Nach jedem Block zeigt es dir Grundrisszeichnungen oder 3D-Bilder und **wartet auf
   dein OK**. Schau genau hin – Fehler in den Räumen ziehen sich durch alles danach.
6. HouseVis zeigt jede gespeicherte Änderung sofort. Lass es neben Claude offen.

Pause gemacht oder ans Limit gestoßen? Schreib einfach *„Mach mit plans/mein-haus weiter."* Claude
führt seine Notizen in `plans/mein-haus/model-notes.md`.

### Mit Loxone verbinden

1. HouseVis muss mit deinem Miniserver verbunden sein ([Kapitel 1](1-installation.md)).
2. Bitte Claude: *„Ordne Räume, Fenster und Lichter Loxone zu."* Claude liest alle Bausteine deines
   Miniservers (nur lesend) und schlägt vor, was wohin gehört. Bestätige oder korrigiere.
3. Auf Wunsch setzt Claude auch **Leuchten** und deine **Wetterstation**.

### Spätere Änderungen

Sag, was falsch ist, z. B. *„Das Badfenster in plans/mein-haus liegt 20 cm zu weit rechts."* Claude
ändert nur das und zeigt dir das Ergebnis. *„Zeig mir die Seiten mit den Live-Werten"* liefert
Bilder deiner Seiten, wie die App sie gerade zeigt.

## 4. Nach dem Modell: Ansichten und Seiten

Das Modell allein zeigt noch nichts – eine **Ansicht** legt fest, von wo du schaust und welche Räume
offen sind. Klick auf dem Startbildschirm auf **Erste Ansicht anlegen**, dann weiter in
[Kapitel 3](3-app-benutzen.md).

## Hilfe und Feedback

- **Du kommst nicht weiter?** Frag im Forum unter [Hausmodell & Pläne](https://forum.housevis.de/c/hausmodell).
  Poste dort **keine kompletten Baupläne**, nur den Ausschnitt, um den es geht, und schwärze im
  Plankopf Adresse und Namen.
- **Erzähl, wie es gelaufen ist:** Unter [Haus-Assistent & Skill-Feedback](https://forum.housevis.de/c/skill-feedback)
  sammeln wir, was auf Anhieb stimmte, wo Claude danebenlag, was es gekostet und wie lange es
  gedauert hat. Daraus wird der Skill besser, für alle.
- **Zeig dein Haus** unter [Eure Häuser](https://forum.housevis.de/c/eure-haeuser), ohne Adresse und Namen auf den
  Bildern.

## Datenschutz

- Weg A schickt das Modell, deine Nachrichten und die Teile von Plänen und Fotos, die Claude ansieht,
  an Anthropic; Weg B schickt, was Claude Code liest (Pläne, Fotos, Dateien deines Arbeitsordners),
  unter deinem Abo. HouseVis selbst hat keinen Server und schickt nichts woandershin.
- **Weg A (API):** Anthropic trainiert seine Modelle nicht mit Daten, die über die API kommen, und
  löscht sie nach 30 Tagen. Dafür musst du nichts einstellen.
- **Weg B (Pro- oder Max-Abo):** Ob deine Sitzungen ins Training gehen, entscheidest du selbst in
  den [Datenschutz-Einstellungen von claude.ai](https://claude.ai/settings/data-privacy-controls)
  (Schalter zur Modellverbesserung). Ausgeschaltet bleiben sie 30 Tage gespeichert, eingeschaltet
  bis zu 5 Jahre. Prüf die Einstellung, bevor du Pläne hineingibst.
- **Bewertungen können immer ins Training gehen:** Daumen hoch/runter oder `/feedback` in Claude
  Code schicken das ganze Gespräch mit, samt Plänen; Anthropic hebt es bis zu 5 Jahre auf und darf
  damit trainieren – egal, wie die Einstellung steht. Erzähl lieber im [Forum](#hilfe-und-feedback), wie es gelaufen ist.
- Pläne sind persönliche Daten und meist urheberrechtlich geschützt (Architekt): nicht in
  öffentliche Repositorys oder geteilte Ordner legen.
- Stand November 2026; maßgeblich sind Anthropics
  [Datenschutzseiten](https://privacy.claude.com).

Weiter: [3 – Die App benutzen](3-app-benutzen.md)
