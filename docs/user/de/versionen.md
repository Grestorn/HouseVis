# Versionen

[← Übersicht](README.md) · [English](../en/versions.md)

Was sich in jeder Version für dich geändert hat. Das Plugin mit der App zusammen aktualisieren
(`/plugin marketplace update housevis`, [Kapitel 2](2-hausmodell.md)).

## 0.9.5

**Wichtig fürs Tablet:** Eine Konfiguration mit einer Kamera, PV-Modulen, einem Dach mit L-, T-
oder U-Umriss, Raumfeuchte oder einem Wert an einem Indikator lädt nicht in einer App vor 0.9.5.
Die Android-App zusammen mit der Windows-App aktualisieren, bevor du so eine Konfiguration
exportierst.

**Kameras**
- **Kamerabilder auf einer Seite:** neben dem Haus ein Standbild, das alle paar Sekunden erneuert
  wird, auf einer Seite mit Kameras das Livebild. Reolink-Kameras (Stand- und Livebild), die
  **Loxone Intercom** (2. Generation) über den Miniserver, andere Kameras mit Standbild.
  Eingerichtet unter Menü → **Kameras …**, mit dem Geräte-Editor auf eine Seite gelegt
  ([Kapitel 3](3-app-benutzen.md#kameras)). Das Demohaus hat eine Kamera zum Ausprobieren.

**Hausmodell (aus dem Beta-Feedback)**
- **PV-Module** auf dem Dach: Claude zeichnet sie nach Dachfotos oder Drohnenaufnahmen; ein
  Indikator über jedem Feld kann zeigen, was es gerade liefert ([Kapitel 2](2-hausmodell.md)).
- **Dächer mit L-, T- oder U-Umriss** sind jetzt ein Dachteil, mit Graten und Kehlen, wo sie
  hingehören; die Dachstücke eines älteren Modells fasst Claude auf Wunsch zu einem zusammen.
- **Raumfeuchte** in der kleinen Zeile unter der Temperatur, neben der Solltemperatur, beide mit
  einem Symbol, das du unter **Globale Einstellungen …** ändern oder abschalten kannst.
- **Werte an Indikatoren:** Ein Indikator kann einen Wert aus Loxone mit Einheit zeigen, z. B. die
  Temperatur des Teichs oder die Leistung der Wärmepumpe.
- **Alle 2.030 Lucide-Symbole** in der Symbolauswahl: die Hausgruppen zuerst, der Rest unter
  **Weitere Gruppen …**.
- Der KI-Assistent und der Skill platzieren Beschriftungen und Indikatoren in einer Ansicht nach
  Augenschein der gerenderten Seite.
- Die Modellprüfung warnt nicht mehr beim inneren Ende eines Zwerchgiebels, das ins Hauptdach
  läuft.

**Korrekturen**
- Windows: **Alt** blendet keine Menüleiste mehr ein (Alt+Klick im Ansichten-Editor geht wieder).
- Ansichten-Editor: Wände, die ein inaktiver Raum hinter einer Öffnung zeichnet (die Seitenwand
  hinter dem offenen Garagentor), lassen sich anklicken.
- Geräte-Editor: Alle Seiten auf ein Gerät mit nur einer Ausrichtung kopieren behält die Seiten
  dieser Ausrichtung.
- Formate mit Breite (`%5.1f`, `%02d`) werden formatiert angezeigt.
- Android: Textfelder in Dialogen bleiben über der Bildschirmtastatur.

## 0.9.4

**Wichtig fürs Tablet:** Eine Konfiguration mit einem Datenpanel oder einem Bodeneinbaustrahler
lädt nicht in einer App vor 0.9.4. Die Android-App zusammen mit der Windows-App aktualisieren,
bevor du so eine Konfiguration exportierst.

**Seiten und Geräte**
- **Datenpanels:** beliebige Werte aus Loxone auf einer Seite (PV-Leistung, Netzbezug, Batterie
  …), je Wert mit Symbol, Name und dem Format, das Loxone angibt; im Geräte-Editor aus der Liste
  der Werte des Miniservers gewählt, nach Raum und Kategorie gefiltert und durchsuchbar
  ([Kapitel 3](3-app-benutzen.md#datenpanels)).
- **Geräte-Editor aufgeräumt:** Geräte, Seiten und Panele in drei Spalten; das **+** öffnet eine
  Auswahl (HouseVis- oder Loxone-Seite; Ansicht, Uhr, Datum oder Datenpanel); Panele klappen auf
  und zu; die Panel-Liste ist die Reihenfolge von vorn nach hinten, Pfeile schieben eins nach vorn
  oder hinten. **Quer/Hoch getrennt** ist jetzt eine Einstellung des Geräts.

**Hausmodell mit anderen KI-Modellen**
- **KI-Assistent mit DeepSeek:** Neben einem Anthropic-Schlüssel nimmt der Assistent auch einen
  anderen Dienst mit Anthropics API, z. B. DeepSeek (`deepseek-flash`, sein Modell, das Pläne sehen
  kann): im Schlüssel-Formular das Häkchen setzen, Adresse, Modell und Schlüssel eintragen. Von uns
  nicht getestet; Kosten werden dann nicht angezeigt.
- **Der Skill für Codex, Gemini CLI und Mistral Vibe:** Den Hausmodell-Skill gibt es auch als
  einfachen Skill-Ordner für diese Agenten ([Kapitel 2](2-hausmodell.md#andere-ki-agenten-codex-gemini-cli-mistral-vibe));
  von uns nicht getestet.
- Seine Befehle stehen in PowerShell-Schreibweise, der Shell dieser Agenten unter Windows; ein
  Ordner `plans\` in einer Kopie dieses Repositorys bleibt aus Git heraus.

**Aus den ersten Rückmeldungen**
- **Bodeneinbaustrahler** als eigene Leuchtenart: in Boden oder Terrasse eingelassen, strahlt nach
  oben (Fassade, Säule, Baum).
- **Räume ohne Datenpunkte** nennt die Prüfung des Modells; ist der Assistent mit dem Miniserver
  verbunden (oder liegt beim Skill die `loxone-inventory.json` dabei), warnt sie bei jedem, dessen
  Loxone-Raum Licht, Raumregler oder Präsenz hat: ein Raum, der dunkel bleibt, weil seine
  Verknüpfung verloren ging, fällt so sofort auf.
- **KI-Assistent:** Fehlermeldungen sagen, wie es richtig geht (z. B. die kleingeschriebene Id,
  `lightColor` als Liste für mehrere Lichtsteuerungen eines Raums); eine Änderung, die Datenpunkte
  entfernt, meldet das sofort. Die Testbilder brechen beim Start nicht mehr so leicht ab
  („Starting the capture window did not finish"): mehr Zeit und ein zweiter Versuch.
- Der Skill fragt bei Fotos ohne Seitenangabe gleich nach Seite und Standort, prüft nach einer
  abgelehnten Änderung, ob alles angekommen ist, und nimmt mehrere Lichtsteuerungen eines Raums
  zusammen.
- **Mähroboter, Saugroboter und Kameras** bietet der Skill als Symbol an ihrem Platz an (Ladestation,
  Tor), mit ihrem Zustand: mäht, lädt, Störung, Klingel.

## 0.9.3

**Wichtig fürs Tablet:** Eine Konfiguration mit den neuen Symbolen (aus einem anderen Satz, mit
Farbe, oder eine Seite ohne Symbol) lädt nicht in einer App vor 0.9.3. Die Android-App zusammen mit
der Windows-App aktualisieren, bevor du so eine Konfiguration exportierst. Unveränderte
Konfigurationen laden weiter auch in älteren Apps.

**Das Haus ansehen**
- **Vorführungen …** (Menü): die Filme des Demo-Hauses aus dem Forum laufen in der App, dazu
  eigene für dein Haus, die der KI-Assistent anlegt, mit ausgedachten oder den aktuellen Werten;
  „Dauerschleife" lässt sie endlos laufen. Ersetzt Demo, Wetter-Demo und Vorführung im Menü.
- **Raumnamen auf dem Boden:** Ansichten, die von oben auf ein Stockwerk schauen, legen Namen von
  selbst auf den Boden, wo keine Wand sie gut zeigt; im Ansichten-Editor lässt sich ein Name auf
  den Boden ziehen und zurück.
- **Dachschrägen** erscheinen nur noch, wo man sie von innen sieht; von oben springt die Schräge
  nicht mehr von einer Seite auf die andere.
- Fenster und Türen in massiven Außenwänden haben keinen verschobenen zweiten Rahmen mehr.
- Fenster und Rollläden bewegen sich bei jeder Bildrate gleich schnell (auf dem Tablet waren sie
  zu langsam).

**Seiten und Einstellungen**
- **Symbole aus drei Sätzen** für Seiten und Anzeigen: Fluent Emoji farbig oder einfarbig, Lucide;
  einfarbige in einer Farbe deiner Wahl; eine Seite kann ohne Symbol auskommen. Ein Auswahlfenster
  mit Gruppen und Suche für beides.
- Geräte-Editor: **Seiten auf ein anderes Gerät kopieren** (Rechtsklick auf eine Seite, oder auf
  ein Gerät für alle seine Seiten); je Ansicht auf einer Seite **Temperaturen und Anzeigen
  ausblenden** oder ihre **Größe** (30–150 %) ändern; die Vorschau passt auch für Hochformat-Seiten.
- **Globale Einstellungen** in zwei Seiten (Optionen, Symbole); neu: die **Größe der
  Loxone-Seite** (Standard 90 %), damit z. B. der Ambient-Modus der Loxone-App Platz hat.
- Menü in Gruppen; „Haus-Assistent …" heißt jetzt **KI-Assistent …**.
- **Gerät exportieren …** schlägt den Ordner des letzten Exports vor.

**Android-Tablet**
- **Vollbild:** Statusleiste und Navigationstasten sind ausgeblendet (ein Wisch vom Rand zeigt sie
  kurz).
- **Die geladene Konfigurationsdatei wird beobachtet:** ändert sie sich (z. B. durch ein
  Sync-Programm wie QFile Pro), liest die App sie von selbst neu ein. „Konfiguration wird
  geladen …" deckt das Laden ab.
- **Einstellungen …** (Menü): Sprache und Größe der Loxone-Seite nur für dieses Tablet.
- Loxone-UI-Anmeldung: nach dem Speichern meldet sich die Loxone-Seite gleich an; das Kennwort
  lässt sich vor dem Senden anzeigen.

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
