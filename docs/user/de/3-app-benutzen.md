# 3 – Die App benutzen

[← Übersicht](README.md) · [English](../en/3-using-the-app.md)

## Wo was ist

![Menü](../images/de/menu.jpg)

- **Seitenleiste** (oben): ein Knopf pro Seite. Eine Seite zeigt eine oder mehrere Ansichten oder
  die Loxone-Oberfläche.
- **Schalter** (oben im Bild): Teile des Hauses beim Anschauen ein- und ausblenden, z. B. das Dach
  oder alles Äußere.
- **Statusfeld** (oben links): die Verbindung zum Miniserver.
- **Menü ☰** (oben links):

| Menüpunkt | Wofür |
|---|---|
| Mit Miniserver verbinden | anmelden, Miniserver wechseln |
| Ansichten bearbeiten … | von wo du schaust, welche Räume offen sind, was ausgeblendet ist |
| Geräte / Seiten bearbeiten … | welche Seiten PC, Tablet oder Handy zeigen; pro Gerät auch Lichtquellen (realistisch oder einfach) und reduzierte Bildrate für langsamere Geräte; Export fürs Tablet |
| Globale Einstellungen … | Sprache der App (sonst die des Geräts); Symbole für Heizung, Musik, … (rund tausend: zuerst Vorschläge, sonst nach Gruppe oder mit der Suche, z. B. „Lautsprecher“ oder „Waschmaschine“); ein Zustand lässt sich auch ganz ausblenden („Nicht anzeigen“, z. B. pausierte Musik) |
| Haus-Assistent … | das Modell mit Claude ändern ([Kapitel 2](2-hausmodell.md)) |
| Konfigurationsordner … | welches Hausmodell die App zeigt |
| Gerät: … | ausprobieren, wie die Seiten eines anderen Geräts aussehen |
| Demo, Wetter-Demo, Vorführung | das ausgedachte Demo-Haus; alle Wettertypen nacheinander; ein ganzer Tag in zwei Minuten mit wechselndem Wetter, die Seiten wechseln von selbst (für ein Video oder einen Bildschirm, der einfach läuft) |
| **Forum: Hilfe und Feedback** | öffnet [forum.housevis.de](https://forum.housevis.de) im Browser: Fragen, Fehler melden, Ideen (nur Windows) |

## Ansichten: was du siehst

Eine **Ansicht** ist eine Kameraposition plus die Wahl, was offen ist. **Aktive Räume** sind
aufgeschnitten: du siehst hinein, mit Temperatur, Lichtfarbe und Anzeigen. Alle anderen Räume sind
geschlossene Wände.

![Ansichten bearbeiten](../images/de/view-editor.jpg)

Menü → **Ansichten bearbeiten …**, dann:

- **Aktive Räume** – die Räume anhaken, in die du hineinsehen willst. Keiner angehakt: das
  geschlossene Haus von außen.
- **Kamera** – im Bild ziehen dreht, Mausrad zoomt, rechte Maustaste verschiebt.
- **Angezeigte Elemente** – nur einzelne Stockwerke zeigen; bei Gruppen wie Dach oder Garten den
  Haken unter **Anzeigen** entfernen; einer Gruppe einen **Schalter** auf der Seite geben. Eine
  Wand im Bild anklicken, um sie durchsichtig zu machen oder auszublenden.
- **Werte und Beschriftungen** – Temperaturen und Symbole im Bild an ihren Platz ziehen.
- **Kopie anlegen** – eine neue Ansicht aus dieser beginnen. **Speichern** (Strg+S); Strg+Z macht
  rückgängig.

## Seiten und Geräte

![Geräte und Seiten](../images/de/device-editor.jpg)

Menü → **Geräte / Seiten bearbeiten …**: jedes Gerät (PC, Tablet, Handy) hat seine Seiten. Füge eine
**Seite mit Ansichten** hinzu und platziere eine oder mehrere Ansichten darauf (in der Vorschau
ziehen), oder eine **Loxone-Seite** mit der Loxone-Oberfläche. Pro Seite: **Wetter als Himmel**. Pro
Gerät: **Sleepmodus erlauben** (für Wandtablets aus) und **Automatische Präsenz-Erkennung** (dunkel,
solange niemand im Raum ist; der Bildschirm bleibt dann ohnehin an).
**Gerät exportieren …** schreibt die Datei fürs Android-Tablet
([Kapitel 1](1-installation.md#android-tablet)).

## Ideen: so kannst du HouseVis nutzen

**Das ganze Haus in 3D** – eine Ansicht von Südwesten mit allen Wohnräumen aktiv, eine zweite
kleine von Nordosten für die Rückseite. Gut als Hauptseite am PC.

![Ganzes Haus](../images/de/house.jpg)

**Geschossmodelle** – eine Seite pro Geschoss: eine Ansicht mit nur diesem Stockwerk (**Angezeigte
Elemente**), Dach ausgeblendet, Kamera steil von oben. Liest sich wie ein lebender Grundriss:
welcher Raum warm ist, wo Licht brennt, welches Fenster offen ist.

![Obergeschoss von oben](../images/de/storey.jpg)

**Das Haus von außen** – kein aktiver Raum: Fassaden, Dach, Terrasse, Gartenbeleuchtung, das Wetter
über dem Haus. Schön fürs Handy oder als „Gute-Nacht"-Seite, die offene Fenster zeigt.

![Von außen](../images/de/outside.jpg)

**Wandtablet** – ein Gerät ohne **Sleepmodus erlauben** und mit **Automatische Präsenz-Erkennung**
(es wird dunkel, wenn der Präsenzmelder niemanden sieht), eine Geschossseite pro Etage und
eine Loxone-Seite zum Schalten.

**Schalter statt vieler Ansichten** – eine Ansicht des ganzen Hauses mit Schaltern für Dach und
Obergeschoss: antippen, und du schaust ins Erdgeschoss.

Weiter: [4 – Probleme lösen](4-probleme-loesen.md)
