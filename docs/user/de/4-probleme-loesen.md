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
| Ein Fenster zeigt offen, obwohl es zu ist (oder umgekehrt) | Der Loxone-Kontakt meldet andersherum: Claude sagen („Fenster Bad zeigt offen, wenn es zu ist"). |
| Etwas wird falsch gezeichnet, obwohl das Modell stimmt (weiße Streifen, fehlende Flächen) | Das ist ein Fehler in HouseVis: bitte mit Bildschirmfoto [im Forum melden](https://forum.housevis.de/c/fehlerberichte). |
| HouseVis zeigt das falsche Haus oder „Noch kein Hausmodell" | Menü → **Konfigurationsordner …**: den Ordner wählen, in dem deine `house.json` liegt. |

## Miniserver

| Problem | Was tun |
|---|---|
| Verbunden, aber Räume zeigen keine Werte | Der Loxone-Benutzer darf diese Räume nicht sehen. In Loxone Config Rechte für alle Räume geben oder mit einem Benutzer anmelden, der sie hat. |
| „Miniserver getrennt" | Ist der Miniserver erreichbar (Adresse, Netz)? HouseVis versucht es selbst erneut. Neue Adresse: Menü → **Mit Miniserver verbinden**. |
| Anmeldung abgelehnt oder Zertifikatsfrage | Benutzername und Passwort prüfen; **Verschlüsselte Verbindung (TLS)** nur bei einem Miniserver 2. Ein neues Zertifikat nur im eigenen Netz übernehmen. |

## Geschwindigkeit

| Problem | Was tun |
|---|---|
| Langsam oder ruckelig, besonders auf dem Tablet | Am PC unter **Geräte / Seiten bearbeiten …** beim Gerät **Lichtquellen: Einfach** und **Reduzierte Bildrate** wählen, dann neu exportieren und einlesen. Weniger Ansichten pro Seite helfen auch. |
| Der Bildschirm wird dunkel | **Automatische Präsenz-Erkennung** ist für dieses Gerät an, oder **Sleepmodus erlauben** (Menü → **Geräte / Seiten bearbeiten …**). |

## Das Modell mit Claude erstellen

| Problem | Was tun |
|---|---|
| Claude Code kennt den HouseVis-Skill nicht | Beide Befehle sind nötig: `/plugin marketplace add …` **und** `/plugin install house-model@housevis`; danach eine neue Sitzung. |
| Claude hält an: Nutzungslimit erreicht | Normal mit einem Pro-Abo. Bis zu der Uhrzeit warten, die Claude nennt, dann „weiter" schreiben. |
| Claude findet HouseVis nicht | HouseVis auf demselben PC installieren ([Kapitel 1](1-installation.md)); liegt es woanders, Claude den Pfad sagen. |
| Windows warnt bei der Installation | Der Installer ist noch nicht signiert: **Weitere Informationen** → **Trotzdem ausführen**. |
