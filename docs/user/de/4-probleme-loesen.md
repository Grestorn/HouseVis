# 4 – Probleme lösen

[← Übersicht](README.md) · [English](../en/4-troubleshooting.md)

> [!IMPORTANT]
> **Dein Problem steht nicht hier?** Frag im Forum unter [Hilfe](https://forum.housevis.de/c/hilfe). Einen Fehler
> meldest du unter [Fehlerberichte](https://forum.housevis.de/c/fehlerberichte), am besten mit der Version (Menü →
> **Über HouseVis …**), den Schritten, die zum Fehler führen, und einem Bildschirmfoto.

## In das Haus hineinsehen

| Problem | Was tun |
|---|---|
| Eine Wand, das Dach oder ein anderes Geschoss verdeckt den Blick in einen Raum | Menü → **Ansichten bearbeiten …**: den Raum unter **Aktive Räume** anhaken (seine Wände zur Kamera werden durchsichtig). Immer noch verdeckt: die Wand im Bild anklicken (**Angezeigte Elemente**) → durchsichtig oder ausgeblendet. Dach oder oberes Geschoss: unter **Angezeigte Elemente** den Haken entfernen oder mit einem **Schalter** versehen. |
| Ein Raum zeigt keine Temperatur und kein Licht | Er muss in dieser Ansicht ein **aktiver Raum** sein und Loxone zugeordnet (Claude bitten, ihn zuzuordnen). |
| Beschriftungen überlappen oder verdecken das Haus | Die Kamera etwas drehen oder weniger Räume in dieser Ansicht aktiv machen; unter **Werte und Beschriftungen** lassen sich die Werte ausblenden. |
| Ein Raum fehlt in der Liste der aktiven Räume | Ein Stockwerk oder eine Gruppe, die unter **Angezeigte Elemente** nicht angehakt ist, verbirgt ihn; erst wieder anhaken. |
| Die Seite ist leer oder sagt „Dieses Gerät zeigt keine Ansicht" | Menü → **Geräte / Seiten bearbeiten …**: eine Ansicht auf die Seite legen. |

## Das Modell

| Problem | Was tun |
|---|---|
| Ein Fenster, eine Wand oder das Dach sitzt falsch | Claude genau sagen, was falsch ist („das Küchenfenster liegt 30 cm weiter links", „der First verläuft Ost–West") – in Claude Code oder im **Haus-Assistenten**. |
| Ein Giebel ist offen, man sieht unters Dach | Die Außenkante des Dachs passt nicht auf die Außenwände (die Prüfung meldet „gable stays open"). Claude bitten, die Dachkante auf die Außenwände zu setzen. |
| Ein Fenster zeigt offen, obwohl es zu ist (oder umgekehrt) | Der Loxone-Kontakt meldet andersherum: Claude sagen („Fenster Bad zeigt offen, wenn es zu ist"). |
| Etwas wird falsch gezeichnet, obwohl das Modell stimmt (weiße Streifen, fehlende Flächen) | Das ist ein Fehler in HouseVis: bitte mit Bildschirmfoto [im Forum melden](https://forum.housevis.de/c/fehlerberichte). |
| HouseVis zeigt das falsche Haus oder „Noch kein Hausmodell" | Menü → **Konfigurationsordner …**: den Ordner wählen, in dem deine `house.json` liegt. |

## Miniserver

| Problem | Was tun |
|---|---|
| Verbunden, aber Räume zeigen keine Werte | Der Loxone-Benutzer darf diese Räume nicht sehen. In Loxone Config Rechte für alle Räume geben oder mit einem Benutzer anmelden, der sie hat. |
| „Miniserver getrennt" | Ist der Miniserver erreichbar (Adresse, Netz)? HouseVis versucht es selbst erneut. Neue Adresse: Menü → **Mit Miniserver verbinden**. |
| Anmeldung abgelehnt oder Zertifikatsfrage | Benutzername und Passwort prüfen; **Verschlüsselte Verbindung (TLS)** nur bei einem Miniserver 2. Ein neues Zertifikat nur im eigenen Netz übernehmen. |

## Kameras

| Problem | Was tun |
|---|---|
| „Keine oder eine falsche Anmeldung“ | Menü → **Kameras …**: Benutzer und Kennwort neu eintragen und **Speichern**. Nach einer geänderten Adresse ist das immer nötig, und an jedem Gerät eigens. |
| „Die Adresse der Kamera ist unbekannt“ oder „Keine Verbindung zur Kamera“ | Adresse und Port prüfen; ist die Kamera im selben Netz? Bei Reolink: **HTTP** an der Kamera eingeschaltet und gespeichert (sonst leitet sie auf HTTPS um)? |
| „Die Kamera antwortet nicht rechtzeitig“ | Kameras im WLAN schlafen manchmal ein: Das erste Bild nach einer Pause dauert einige Sekunden, HouseVis versucht es zweimal. Bleibt es dabei, den WLAN-Empfang der Kamera prüfen. |
| Reolink: Standbild ja, Livebild nein | An der Kamera **RTMP** einschalten (*Netzwerk → Erweitert → Servereinstellungen*) und speichern; unter **Kameras …** den Stream **Flüssig (Fluent)** wählen, *Ausgewogen* hat nicht jedes Modell. |
| Das Livebild ruckelt oder hängt hinterher | Stream **Flüssig (Fluent)** in H.264 mit niedriger Bildrate; auf einem schwachen Tablet besser ein Standbild statt des Livebilds. |
| Intercom: „Die Intercom geht nur mit einer verschlüsselten Verbindung zum Miniserver“ | Menü → **Mit Miniserver verbinden**, mit **Verschlüsselte Verbindung (TLS)**. |
| Intercom: „Die Intercom lehnt die Anmeldung von HouseVis ab“ | Der Miniserver-Benutzer, mit dem HouseVis angemeldet ist (Menü → **Mit Miniserver verbinden**), braucht in Loxone Config das Recht für die Intercom. Hat er es, neu mit dem Miniserver verbinden. |

## Geschwindigkeit

| Problem | Was tun |
|---|---|
| Langsam oder ruckelig, besonders auf dem Tablet | Am PC unter **Geräte / Seiten bearbeiten …** beim Gerät **Einfache Beleuchtung** und **Reduzierte Bildrate** einschalten, dann neu exportieren und einlesen. Weniger Ansichten pro Seite helfen auch. |
| Der Bildschirm wird dunkel | **Automatische Präsenz-Erkennung** ist für dieses Gerät an, oder **Sleepmodus erlauben** (Menü → **Geräte / Seiten bearbeiten …**). |

## Das Modell mit Claude erstellen

| Problem | Was tun |
|---|---|
| Claude Code kennt den HouseVis-Skill nicht | Beide Befehle sind nötig: `/plugin marketplace add …` **und** `/plugin install house-model@housevis`; danach eine neue Sitzung. |
| Claude hält an: Nutzungslimit erreicht | Normal mit einem Pro-Abo. Bis zu der Uhrzeit warten, die Claude nennt, dann „weiter" schreiben. |
| Claude findet HouseVis nicht | HouseVis auf demselben PC installieren ([Kapitel 1](1-installation.md)); liegt es woanders, Claude den Pfad sagen. |
| Windows warnt bei der Installation | SmartScreen kennt eine neue Version noch nicht. Zeigt **Weitere Informationen** den Herausgeber „Martin Korndörfer": **Trotzdem ausführen**. Ein anderer oder unbekannter Herausgeber: nicht ausführen, erneut von der Download-Seite laden. |
