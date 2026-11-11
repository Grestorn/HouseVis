# Versionen

[← Übersicht](README.md) · [English](../en/versions.md)

Was sich in jeder Version für dich geändert hat. Das Plugin mit der App zusammen aktualisieren
(`/plugin marketplace update housevis`, [Kapitel 2](2-hausmodell.md)).

## 0.9.2

**Hausmodell**
- **Satteldach aus zwei Pultdächern** (der Behelf für ungleiche Überstände vor 0.9.1): die Giebel
  sind wieder geschlossen, die Meldung „gable stays open" kommt nicht mehr; der Skill bietet an,
  sie zu einem Satteldach zusammenzufassen. Wo die Meldung noch erscheint, sagt sie das: den Skill
  bitten, die beiden zusammenzufassen.

## 0.9.1

**Hausmodell**
- **Dächer so, wie die Pläne sie bemaßen:** je Seite eigener Überstand, eigene Traufhöhe und
  Neigung. Deine `house.json` wird beim ersten Start umgewandelt; die alte Datei bleibt daneben
  (`house.v6.json`), ältere Export-Dateien werden weiter gelesen.
- **Möbel** als einfache Kästen (Küchenzeile, Schränke, Kleiderschrank), Lichtleisten an ihren
  Kanten.
- **Lichtleisten**, die ansteigen oder senkrecht stehen, auch schräg entlang einer Treppe;
  **Röhren-Pendelleuchten**.
- **Anzeigen an einem eigenen Ort**, etwa der Briefkasten an der Haustür; je Ansicht im
  Ansichten-Editor verschiebbar.
- **Fensterkontakte aus einer „Fenster- und Türüberwachung"** (WindowMonitor), ein Baustein für
  viele Fenster.

**Das Haus ansehen**
- **Das Dach öffnet sich von selbst über aktiven Räumen**, wo es ihre Decke ist (Räume unter der
  Schräge), bis durch die Traufe; ein Klick aufs Dach macht es durchsichtig, ausgeblendet oder ganz
  geschlossen. Im Spitzboden stehen keine Wände mehr; Giebel unter einem durchsichtigen Dach sind
  ebenfalls durchsichtig.
- **Uhr und Datum** auf einer Seite, platziert wie eine Ansicht.
- Das **Statusfeld** oben links erscheint nur noch, wenn die Werte **nicht** live sind.
- Lichtleisten blenden im realistischen Modus weniger; Wolken springen nicht mehr, wenn der Wind
  wechselt.
- Gruben lassen sich in einer Ansicht weglassen; das Gelände schließt sich dann darüber.

**Windows-App**
- **Loxone-UI-Anmeldung …** (Menü): die Loxone-Seite kann sich ihre Anmeldung merken (freiwillig,
  verschlüsselt). Auf Android kann der Passwort-Manager sie ausfüllen.
- Geräte-Editor: die Vorschau zeigt die echten Ansichten; eine kopierte Ansicht kann auf jedem
  Gerät eine Seite bekommen.
- Haus-Assistent: das Haus neben dem Panel, mit Auswahl der Ansicht und frei drehbar.

**Claude-Code-Plugin**
- Bilder aller Seiten (`--render`), eines Geräts oder einer Seite, auch mit den **Live-Werten**
  deines Miniservers.
- Das Plugin prüft, ob es zum installierten HouseVis passt, und sagt, was zu aktualisieren ist.
- Der Skill hat aus den ersten damit gebauten Häusern gelernt.

## 0.9.0

Die erste öffentliche Beta: Windows-App mit den Editoren für Ansichten, Geräte und globale
Einstellungen, dem Haus-Assistenten, die Android-App fürs Wandtablet, das Demo-Haus und das
Hausmodell-Plugin für Claude Code.
