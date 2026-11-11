# 4 – Troubleshooting

[← Overview](README.md) · [Deutsch](../de/4-probleme-loesen.md)

> [!IMPORTANT]
> **Your problem is not listed?** Ask in the forum under [Hilfe](https://forum.housevis.de/c/hilfe) (help, in German).
> Report bugs under [Fehlerberichte](https://forum.housevis.de/c/fehlerberichte), ideally with the version (menu →
> **About HouseVis …**), the steps that lead to the bug, and a screenshot.

## Seeing into the house

| Problem | What to do |
|---|---|
| A wall, the roof or another storey blocks the view into a room | Menu → **Edit views …**: tick the room under **Active rooms** (its walls facing you become transparent). Still blocked: click the wall in the picture (**Shown elements**) → transparent or hidden. Roof or upper storey: untick them under **Shown elements**, or give them a **Switch**. |
| A room shows no temperature or light | It must be an **active room** in this view, and mapped to Loxone (ask Claude to map it). |
| Labels overlap or cover the house | Turn the camera a little, or make fewer rooms active in this view; **Values and labels** can hide the values. |
| I can't find a room in the list of active rooms | A storey or group unticked under **Shown elements** hides it; tick it again first. |
| The page is empty or says "This device shows no view" | Menu → **Edit devices / pages …**: put a view on the page. |

## The model

| Problem | What to do |
|---|---|
| A window, wall or roof is in the wrong place | Tell Claude exactly what is wrong ("the kitchen window is 30 cm further left", "the roof ridge runs east–west") – in Claude Code or in the **House assistant**. |
| A gable is open, you can see under the roof | The roof's wall line does not match the outer walls (the check reports "gable stays open"). Ask Claude to put the roof's outline on the outer walls. |
| A window shows open when it is closed (or the other way round) | Loxone's contact reports the opposite: tell Claude ("window bathroom shows open when closed"). |
| Something is drawn wrongly although the model is right (white stripes, missing faces) | That is a HouseVis fault: please [report it in the forum](https://forum.housevis.de/c/fehlerberichte) with a screenshot. |
| HouseVis shows the wrong house or "No house model yet" | Menu → **Configuration folder …**: choose the folder that holds your `house.json`. |

## Miniserver

| Problem | What to do |
|---|---|
| Connected, but rooms show no values | The Loxone user may not see those rooms. Give it permission for all rooms in Loxone Config, or sign in with a user that has it. |
| "Miniserver disconnected" | Is the Miniserver reachable (address, network)? HouseVis tries again by itself. For a new address: menu → **Connect to Miniserver**. |
| Sign-in refused, or a certificate question | User name and password; **Encrypted connection (TLS)** only for a Miniserver 2. Accept a new certificate only in your own network. |

## Performance

| Problem | What to do |
|---|---|
| Slow or jerky, especially on a tablet | On the PC, under **Edit devices / pages …**, set the device's **Light sources: Simple** and **Reduced frame rate**, then export and read it in again. Fewer views per page helps too. |
| The screen goes dark | **Automatic presence detection** is on for this device, or **Allow sleep mode** (menu → **Edit devices / pages …**). |

## Creating the model with Claude

| Problem | What to do |
|---|---|
| Claude Code does not know the HouseVis skill | Both commands are needed: `/plugin marketplace add …` **and** `/plugin install house-model@housevis`; then a new session. |
| Claude stops: usage limit reached | Normal with a Pro subscription. Wait until the time Claude names, then say "continue". |
| Claude cannot find HouseVis | Install HouseVis on the same PC ([chapter 1](1-install.md)); if it is installed elsewhere, tell Claude the path. |
| Windows warns when installing | The installer is not signed yet: **More info** → **Run anyway**. |
