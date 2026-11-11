# Loxone mapping (block 7)

Goal: `<model folder>/datapoints.json` plus the `data` references in house.json, as
**suggestions the user confirms**. Schema: `${CLAUDE_PLUGIN_ROOT}/skills/house-model/schema/datapoints.schema.json`. The Loxone facts the
mapping needs are in *Datapoints* below.

## Input

`loxone-inventory.json` in the working folder, written by `"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --inventory`. It lists
the Loxone rooms with their controls: `uuid`, `name`, `type`, `category`, `states`, and
`subControls` with their own UUIDs (`<uuid>/AI1`). If it is missing or old, run the command
yourself: it uses the sign-in the app has stored (read only). If it reports that HouseVis is not
connected, ask the user to connect it once in the app (menu → "Mit Miniserver verbinden"); you
cannot sign in yourself, the password lives only in the app's dialog.

The structure file only contains what the signed-in user may see. Rooms without controls usually
mean missing permissions, not missing hardware.

## Datapoints

```json
{
  "sources": { "miniserver": { "type": "loxone" } },
  "datapoints": {
    "kueche.temperature": { "source": "miniserver", "label": "Küche", "loxone": { "control": "<uuid>", "state": "tempActual" }, "unit": "°C" }
  }
}
```

Ids: `<roomId>.<role>` for rooms, `window.<openingId>`, `shade.<openingId>.position` — or follow
the convention of an existing datapoints.json of the user. `label` = the name in Loxone Config.

| HouseVis role | Typical Loxone control → state | `as` |
|---|---|---|
| room `temperature` | `IRoomControllerV2` → `tempActual`; or a sensor `InfoOnlyAnalog` → `value` | (number) |
| room `targetTemperature` | `IRoomControllerV2` → `tempTarget` | (number) |
| room `heating` | the climate controller's `controls` (`ClimateController`/`ClimateControllerUS`), picking the room controller | `{ "type": "climateDemand", "control": "<IRoomControllerV2 uuid>", "demand": "heating" }` |
| room `cooling` | same as `heating` | same, with `"demand": "cooling"` |
| room `presence` | `PresenceDetector` → `active` | `"boolean"` |
| room `climateMode` | `IRoomControllerV2` → `activeMode` (0 eco, 1 comfort, 2 building protection) | (number) |
| room `climateOverride` | `IRoomControllerV2` → `overrideReason` (2 window open, 15 pause timer → "paused") | (number) |
| room `light` | `LightControllerV2` → `activeMoods` | `{ "type": "boolean", "notEquals": "[778]" }` (778 = mood "Off") |
| room `lightColor` | the light controller's **output** subcontrol `…/AIn` that has a `color` state | `"color"`; a list for several outputs |
| opening `open` | window contact, often `InfoOnlyDigital` → `active` | `"boolean"`, with `"invert": true` if it reports 1 when closed |
| opening `open` / `tilted` from a WindowMonitor | "Fenster- und Türüberwachung" (`WindowMonitor`) → `windowStates` | `{ "type": "windowMonitor", "window": <index>, "state": "open" }` (or `"tilted"`), with `"name"`: the window's name in the block |
| opening/roof window `position` | `Window` → `position` (motorised windows), `Gate` → `position` (garage gates) | (number 0–1) |
| shade `position` | `Jalousie` → `position` | (number 0–1) |
| shade `slats` | `Jalousie` → `shadePosition` (blinds only) | (number 0–1) |
| indicator condition | e.g. `AudioZoneV2` → `playState` | used in `indicators[].rules[].when` |
| light source `lightColor` | a light controller output `…/AIn` (`ColorPickerV2`) → `color` | `"color"`; a list for several outputs |
| light source `light` | a switch output (`Switch` → `active`), or a lamp's output state (`InfoOnlyDigital` → `active`, e.g. "Wandstrahler Haustür") | `"boolean"` |
| light source `lightLevel` | a dimmer (`Dimmer` → `position`, 0–100) | (number) |
| house `outsideBrightness` | the weather station's brightness (`InfoOnlyAnalog` → `value`, format "…Lx") | (number, lux) |
| house `outsideTemperature` | an outside temperature (`InfoOnlyAnalog` → `value`, "…°"): ask which family | (number, °C) |
| house `feltTemperature` | "Gefühlte Temperatur" (weather service) | (number, °C) |
| house `humidity` | "Relative Luftfeuchte" / "Außenluftfeuchtigkeit" | (number, %) |
| house `windSpeed` | "Windgeschwindigkeit (…)" | (number, km/h) |
| house `windDirection` | "Windrichtung" (weather service; degrees where the wind comes from) | (number) |
| house `windWarning` | "Windwarnung" (weather station, `InfoOnlyDigital` → `active`) | `"boolean"` |
| house `rain` | "Regen (…)" (`InfoOnlyDigital` → `active`) | `"boolean"` |
| house `sunshine` | "Sonnenschein (…)" (`InfoOnlyDigital` → `active`) | `"boolean"` |
| house `weatherType` | "Wettertyp" from the weather service, the **number** (`InfoOnlyAnalog` → `value`), not a `TextState` made from it | (number, Loxone's codes 1–29) |

Quirks that matter here:

- Only the light controller **outputs** (subcontrols `AI1…AIn`, state `color`) report the real
  color and brightness; `masterColor`/`masterValue` are inputs and useless as feedback. Plain
  switch outputs (subcontrol type `Switch`, state `active`) have no color: use only `light`.
- Which output belongs to which room is not always obvious (one controller may serve several
  areas, e.g. bedroom and walk-in closet). Ask.
- Contacts: write the draft first, then read the live values (see step 4). If (almost) every
  contact reads `true` while the windows are surely closed (night, winter), they report 1 when
  closed → `invert` on all of them. Contacts named "… offen"/"… open" usually report 1 when open.
  Ask the user to open one window if in doubt.
- Left/right in Loxone names ("Fenster Küche links", "Jalousie rechts") is usually **seen from
  inside**, garage gates usually **seen from the street**. Cross-check with names that carry a
  compass direction ("Jalousie Norden"), then let the user confirm.
- `heating` and `cooling` come from the room controller's **demand**, which the climate
  controller lists per room controller in its state `controls` (-1 cooling, 0 none, 1 heating).
  The room controller is the one whose `tempActual` the room's `temperature` uses. Map both roles
  even without active cooling: a valve opens for cooling as well, so a valve-based flame would
  show while cooling, and the snowflake (`cooling`) shows whatever the controller's cooling
  output drives (for example roof windows opened for cooling).
- Only for rooms without a room controller on the climate controller (e.g. a sauna controlled by
  its own block), fall back to the valve: actuator output (`InfoOnlyAnalog` → `value`, "Stellantrieb
  …") with `{ "type": "boolean", "threshold": 0 }`. Several actuators in one room: `heating`
  takes one datapoint; ask, usually the valves are driven together and the first stands for all.
- A control's `presence` state (e.g. on a sauna or audio control) may be wired from another
  room's detector: compare it live with the room's PresenceDetector before showing it.
- Window contacts that appear only inside a "Fenster- und Türüberwachung" (`WindowMonitor`) and
  not as controls of their own are common: map them through the block (row *opening `open` from a
  WindowMonitor* above). The inventory lists the block's `windows` with `index`, `name`,
  `installPlace` and room; match them to the openings by name and place, and always write `name`,
  so a window added to the block later (which shifts the positions) is reported instead of
  silently showing the wrong window. The block reports open and tilted itself; no `invert`. Contacts
  that exist as controls of their own as well (`InfoOnlyDigital`) may be used instead; prefer the
  block when only it has the tilt.
- Sensors the user names but the inventory lacks (a motion detector "Decke") are not enabled for
  the user interface in Loxone Config, or the user lacks permission: record it, map it later.
- `lightColor` alone decides whether a room is lit (black = off); `light` is then optional.
- `CentralJalousie`, `CentralWindow`, `CentralLightController` are group controls: not per
  opening, skip them.

## Beyond the table

- **Keep every light output**: one datapoint per output, `light.<roomId>.<output-name>`
  (ColorPickerV2 → `color` as `"color"`, Dimmer → `position` with unit `%`, Switch → `active` as
  `"boolean"`), outdoor ones as `light.aussen.<output-name>` or the equivalent. The room's
  `lightColor` refers to the primary output's datapoint instead of a duplicate. Block 8 (light
  sources) refers to the others: an extra output ("Essplatz") becomes a pendant's `lightColor`,
  an outside output a lantern's or wall light's. A room that mixes several outputs keeps them in
  its list even when one of them also drives a light source.
- **Weather**: Loxone offers several sources, often side by side: the physical
  weather station ("… (Wetterstation)": brightness, wind, temperature, rain, sunshine, wind
  warning), the Loxone Weather Service ("… (Server)": the location's weather, weather type, felt
  temperature, humidity, wind direction, pressure, forecasts) and merged values ("… (System)").
  Map them to the house-wide roles of the table: the station's brightness as `outsideBrightness`;
  for the **outside temperature ask which one the user trusts**: a station on the roof reads warm
  in the sun, the System value may include a shaded sensor (e.g. the heat pump's) and is then the
  best ambient temperature, so suggest it. Wind speed: the station's measures at the house,
  suggest it. `rain` from the station's rain sensor (it notices a shower before the service).
  `weatherType` must be the service's **number**: an installation may have a status block
  turning it into its own texts (a `TextState` "Wettertyp"), which the app cannot read; it knows
  the codes itself. Values without a role (pressure, radiation, forecasts, fine dust) stay
  unreferenced datapoints. Offer block 9 (the weather station's place) once these are mapped.
- **Audio**: every `AudioZoneV2` → `<roomId>.audio` on `playState` (0 stopped, 1 paused, 2
  playing), and per room an indicator `{ "kind": "music", "room": …, "rules": [{ "when":
  { "datapoint": …, "equals": 2 } }] }`; a room with several zones uses `any`. Paused (1) may
  get a second rule with `"kind": "music-paused"`; that kind is not shown by default, the user
  can give it an icon (menu "Globale Einstellungen …"), so say so. Any kind can be set to "Nicht
  anzeigen" there (`"hidden"` in global.json). Any other kind you invent gets a placeholder icon
  until the user picks one, so say so too. Other indicators (mailbox, washing machine): one
  indicator per thing with a rule per state, see `format.md` *Indicators*.
- **Areas with controls of their own** (a sauna with its control's `tempActual`/`tempTarget`/
  `power`, its own light output and audio zone) can become a room of their own even if Loxone
  files their controls under another room; suggest it.
- Values without a place in the model yet (interior door contacts, a sauna's "active" state) are
  kept as unreferenced datapoints, so they need no second search later.

## Procedure

1. Match house.json rooms with inventory rooms by name; present the table (HouseVis room ↔ Loxone
   room ↔ controls found) and ask about areas without a match or with several candidates.
2. Per room propose the datapoints of the table above; per opening the contact, motor and shade
   whose names point to it (Loxone names like "Fenster Küche West 1", "Jalousie Küche links").
   Present ambiguous cases as a short list of options instead of guessing.
3. Write datapoints.json and the `data` references; run `"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --house-check <folder>`, which
   checks that every reference exists.
4. If HouseVis has been connected before (the stored token is used, read-only):
   `HOUSEVIS_CONFIG=<folder> "$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --values` prints the live value of every datapoint of
   that folder. Without `HOUSEVIS_CONFIG` it prints the app's configuration folder instead. Filter the
   output with grep; the `loxone:values=<filter>` form printed nothing in practice. Check
   plausibility (temperatures, which rooms are lit or playing, contact polarity) and report it.
