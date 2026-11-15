# 3 – Die App benutzen

[← Übersicht](README.md) · [English](../en/3-using-the-app.md)

## Wo was ist

![Menü](../images/de/menu.jpg)

- **Seitenleiste** (oben): ein Knopf pro Seite. Eine Seite zeigt eine oder mehrere Ansichten oder
  die Loxone-Oberfläche.
- **Schalter** (oben im Bild): Teile des Hauses beim Anschauen ein- und ausblenden, z. B. das Dach
  oder alles Äußere.
- **Statusfeld** (oben links): nur wenn die Werte nicht live vom Miniserver kommen (Verbindung
  wird aufgebaut oder ist getrennt, simulierte Daten, Demo).
- **Menü ☰** (oben links):

| Menüpunkt | Wofür |
|---|---|
| Mit Miniserver verbinden | anmelden, Miniserver wechseln |
| Loxone-UI-Anmeldung … | Benutzer und Kennwort der Loxone-Oberfläche speichern, damit sie nach dem Start der App nicht jedes Mal fragt (freiwillig; das Kennwort liegt dann verschlüsselt auf dem Gerät, siehe Hinweis im Dialog; nur bei verschlüsselter Verbindung). Nach dem Speichern zeigt die App die Loxone-Seite und meldet sich gleich an; lehnt Loxone ab, öffnet sich der Dialog wieder mit einem Hinweis |
| KI-Assistent … | das Modell mit Claude ändern ([Kapitel 2](2-hausmodell.md)) und mehr: Vorführungen anlegen, Zuordnungen zu Loxone prüfen |
| Ansichten bearbeiten … | von wo du schaust, welche Räume offen sind, was ausgeblendet ist |
| Geräte / Seiten bearbeiten … | welche Seiten PC, Tablet oder Handy zeigen; pro Gerät auch einfache Beleuchtung und reduzierte Bildrate für langsamere Geräte; Export fürs Tablet |
| Globale Einstellungen … | zwei Seiten, links gewählt. **Optionen**: Sprache der App (sonst die des Geräts); Größe der Loxone-Seite (kleiner passt mehr hinein, z. B. für den Ambient-Modus der Loxone-App; Standard 90 %). **Symbole**: das Symbol für Heizung, Musik, … aus drei Sätzen: **Fluent Emoji** (farbig), **Fluent Emoji (High Contrast)** (dieselben einfarbig) und **Lucide** (schlichte Linien); zuerst Vorschläge, sonst nach Gruppe oder mit der Suche, z. B. „Lautsprecher“ oder „Waschmaschine“. Einfarbige Symbole bekommen eine **Farbe** (Standard, eine aus der Palette oder „Eigene …“; Farbe wählen, dann das Symbol). Ein Zustand lässt sich auch ganz ausblenden („Nicht anzeigen“, z. B. pausierte Musik) |
| Konfigurationsordner … | welches Hausmodell die App zeigt |
| Konfiguration laden … | nur Tablet: eine exportierte Datei laden ([Kapitel 1](1-installation.md)) |
| Einstellungen … | nur Tablet: Sprache der App und Größe der Loxone-Seite für dieses Gerät, abweichend von den Globalen Einstellungen |
| Gerät: … | ausprobieren, wie die Seiten eines anderen Geräts aussehen |
| Vorführungen … | was die App von selbst zeigt. **Demo-Haus**: das ausgedachte Haus ansehen; alle Wettertypen nacheinander (Wetter-Demo); ein ganzer Tag in zwei Minuten, die Seiten wechseln von selbst (Vorführung, für einen Bildschirm, der einfach läuft); dazu kurze Filme mit Kamerafahrt (Rundflug, Licht am Abend, …). **Dein Haus**: deine eigenen Vorführungen, die der Haus-Assistent dir anlegt, z. B. „Mach eine Vorführung wie ‚Ein Tag in zwei Minuten‘ für mein Haus“, mit ausgedachten oder mit den aktuellen Werten. Esc oder ein Klick beendet eine Vorführung; „Dauerschleife“ lässt sie endlos laufen. |
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
  Wand im Bild anklicken, um sie durchsichtig zu machen oder auszublenden. Das **Dach** ist von
  sich aus über den aktiven Räumen offen, soweit es dort ihre Decke bildet (Räume unter einem
  niedrigen Dach, Kniestock); ein Klick aufs Dach macht es durchsichtig, verborgen oder ganz fest.
- **Werte und Beschriftungen** – Temperaturen und Symbole im Bild an ihren Platz ziehen (auch ein
  Symbol an einem eigenen Ort, etwa der Briefkasten an der Haustür; nur für diese Ansicht). Mit
  Umschalt beim Ziehen hebt oder senkt man es; ein Doppelklick setzt es zurück. Den Raumnamen
  zieht man auf eine andere Wand des Raums oder auf seinen Boden – passend für eine Ansicht, die
  von oben auf ein Stockwerk schaut; solche Ansichten legen Namen von selbst auf den Boden, wo
  keine Wand sie gut zeigt.
- **Kopie anlegen** – eine neue Ansicht aus dieser beginnen. Beim ersten **Speichern** (Strg+S)
  fragt HouseVis, ob sie auf jedem Gerät eine eigene Seite bekommen soll. Strg+Z macht rückgängig.

## Seiten und Geräte

![Geräte und Seiten](../images/de/device-editor.jpg)

Menü → **Geräte / Seiten bearbeiten …**: jedes Gerät (PC, Tablet, Handy) hat seine Seiten. Das
**+** neben **Seiten** fügt eine **HouseVis-Seite** hinzu (Ansichten, Uhr, Datum und Datenpanels,
in der Vorschau platziert; die Vorschau zeigt die Ansichten selbst) oder eine **Loxone-UI**-Seite
mit der Loxone-Oberfläche. Pro Seite: der **Titel**, davor ihr Symbol in der Seitenleiste (ein
Klick darauf wählt es aus denselben Sätzen wie die Anzeigen, einfarbige mit Farbe, oder „Kein
Symbol zeigen“), und **Aktuelles Wetter anzeigen**. Das **+** neben **Panele** fügt hinzu, was auf
der Seite liegt: eine **Ansicht** (daneben die Ansichten, die noch nicht auf der Seite sind),
**Uhr**, **Datum** (Uhrzeit und langes Datum mit Wochentag und Monat, so wie die
Systemeinstellungen des Geräts sie schreiben, z. B. 12 oder 24 Stunden) und **Datenpanel**. Nur das
gewählte Panel ist aufgeklappt und in der Vorschau markiert; ein Klick auf seinen Namen klappt es
auf oder zu. Die Liste ist auch die Reihenfolge von vorn nach hinten: was oben steht, liegt vorn
und verdeckt alles darunter; die Pfeile rechts oben an jedem Panel schieben es eine Ebene nach vorn
oder hinten, der Papierkorb entfernt es.

![Hinzufügen auf der Seite](../images/de/device-add-menu.jpg)

Pro Ansicht auf der Seite: **Mit Rahmen und Titel**, **Temperaturen und Anzeigen
ausblenden** (nur das Haus, z. B. für eine kleine Nebenansicht) und **Größe der Temperaturen und
Anzeigen** (30 bis 150 %, Doppelklick auf den Regler: 100 %).

Pro Gerät (das **+** neben **Geräte** legt eins an, die Symbole in seiner Zeile duplizieren oder
löschen es): **Name**, **Plattform** und **Seitenverhältnis**. **Quer/Hoch getrennt** gibt dem
Gerät eigene Seiten für Quer- und Hochformat (sonst dieselben in beiden); dann wählen zwei kleine
Symbole neben **Seiten**, welche Liste du bearbeitest, und neben **Vorschau** siehst du die Seite
in beiden Formen. Unter **Einstellungen**: **Sleepmodus erlauben** (für Wandtablets aus),
**Automatische Präsenz-Erkennung** (dunkel, solange niemand im Raum ist; der Bildschirm bleibt dann
ohnehin an), **Einfache Beleuchtung** und **Reduzierte Bildrate** (beide für schwächere Geräte).
Ein Rechtsklick auf ein Gerät macht es zum **Standard** (das Gerät, das der PC ohne eigene Wahl
zeigt) oder kopiert alle seine Seiten auf ein anderes Gerät; ein Rechtsklick auf eine Seite kopiert
sie. Hat das Ziel schon eine Seite gleichen Namens, wird nach einer Rückfrage diese ersetzt.

![Rechtsklick auf ein Gerät](../images/de/device-menu.jpg)

**Gerät exportieren …** schreibt die Datei fürs Android-Tablet
([Kapitel 1](1-installation.md#android-tablet)).

### Datenpanels

![Ein Datenpanel im Geräte-Editor](../images/de/data-panel.jpg)

Ein **Datenpanel** zeigt beliebige Werte aus Loxone auf einer Seite, z. B. die Leistung der
PV-Anlage, den Netzbezug oder den Ladestand der Batterie: je Wert ein Symbol, ein Name und die Zahl
im Format, das Loxone für sie angibt (z. B. „%.2f kW“). **+** → **Datenpanel** legt eins an (ganz vorn); es wird
platziert und in der Größe geändert wie die Uhr, eine Seite kann mehrere haben. Pro Panel:
**Anordnung** (nebeneinander oder untereinander; jeder Wert bekommt gleich viel Platz, die Schrift
wird so groß, wie es passt) und **Mit Hintergrund**. **+ Datenpunkt** öffnet die Liste aller Werte
des verbundenen Miniservers, nach **Raum** und **Kategorie** gefiltert und nach Namen durchsuchbar,
jeder mit seinem aktuellen Wert:

![Datenpunkt wählen](../images/de/value-picker.jpg)

Ein Klick nimmt den Wert mit dem Namen des Bausteins, seinem Format und einem passenden Symbol.
Danach lässt sich alles ändern: das **Symbol** (oder „Kein Symbol zeigen“), der **Name** (leer: nur
Symbol und Wert) und das **Format** (`%.1f` eine Nachkommastelle, `%d` keine, `%%` ein
Prozentzeichen, dahinter die Einheit; **Aus Loxone** holt es neu). Die Liste braucht eine Verbindung
zum Miniserver; sie zeigt nur, was der angemeldete Benutzer in Loxone sehen darf.

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
