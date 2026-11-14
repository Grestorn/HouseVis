# Playbacks for the user's house

A playback is a scripted clip the app plays by itself: menu **Vorführungen …**, section "Dein Haus"
(the demo house's are in the same panel). Values change on a timeline, the camera moves, pages or
views switch. Typical requests: "mach eine Vorführung wie ‚Ein Tag in zwei Minuten' für mein Haus",
"ein Rundflug um mein Haus, so wie es gerade ist", "zeig, wie abends die Lichter angehen".

## Where it goes

`<model folder>/playbacks/<id>.json`; `<id>` is lower case, digits and hyphens (`ein-tag`,
`rundflug`). The app offers it as soon as the file is there (it reloads on its own); an export to
the tablet takes it along if it plays on that device's views. In the house assistant the file is
created whole: `apply_patch` with one `add` at path `""`.

## Two kinds: ask which one the user wants

| | `"base"` | What shows | For |
|---|---|---|---|
| **Made up** | `"script"` (default) | only what the script sets; everything else has no value ("–") | "a day in my life": a story with made-up values and events |
| **As it is now** | `"live"` | the Miniserver's current values, live; the script overrides only what it sets | "my house right now": a fly-around, a camera tour, at most a different sky |

For a made-up playback, **set every value that shows** in `initial` at time 0: each room's
temperature and target temperature, each light (on or off), presence if shown, the outside values
(brightness, temperature, weather type, wind…). A room without its temperature shows "–" through
the whole playback. For a live playback set as little as possible: usually only the camera.

## The file

```json
{
  "title": { "de": "Ein Tag in zwei Minuten", "en": "A day in two minutes" },
  "base": "script",
  "page": "Haus",
  "lightMode": "real",
  "duration": 120,
  "initial": { "<datapoint>": <value>, … },
  "events": [
    { "t": 10, "set": { "aussen.helligkeit": 20000 }, "ramp": 15, "curve": "log" },
    { "t": 80, "set": { "light.wohnen.decke": "temp(90,2700)" }, "ramp": 3 },
    { "t": 30, "page": "Obergeschoss" }
  ],
  "camera": [
    { "t": 0, "azimuth": 225, "elevation": 30, "zoom": 1 },
    { "t": 120, "azimuth": 585, "elevation": 30, "zoom": 1, "ease": "inOut" }
  ]
}
```

| Field | Meaning |
|---|---|
| `title` | `{ "de", "en" }` (or one text): the name in the panel. Always give both. |
| `base` | `"script"` or `"live"`, see above |
| `page` / `view` | what shows at the start: a **page title** of the Windows device (landscape) from `devices.json`, or a **view id** from `views/` (one view filling the window; `"weather": true` adds its sky) |
| `duration` | seconds; default the last event or keyframe |
| `loop` | `true`: starts again until stopped (an unattended display); the panel also has a switch |
| `lightMode` | `real` (default) or `simple` |
| `initial` | datapoint id → value at time 0 |
| `events` | `{ "t", "set": { id: value }, "ramp": <seconds>, "curve": "log" }` and/or `{ "t", "page" \| "view" }` |
| `camera` | keyframes `{ "t", "azimuth", "elevation", "zoom", "target", "ease" }`; each field is interpolated between the keyframes that set it |

- **Values** are in the terms the display uses: a number as shown (°C, lux, km/h; shade and
  window positions 0 = open/up to 1 = closed/down; a dimmer in %), **1/0** for on/open/present
  (a window contact, presence, a switched light, rain), a light colour as Loxone text
  `"temp(<brightness %>,<kelvin>)"` (`"temp(0,2700)"` is off) or `"hsv(<h>,<s>,<v>)"`, or
  `"#rrggbb"`. The datapoint's own conversion (`invert`, scale) is not applied: write what should
  show.
- **Ramps** move numbers and light colours over `ramp` seconds from their value at `t`.
  Brightness outside spans decades: `"curve": "log"` (20 000 lx → 1 lx linearly stays bright until
  the last second).
- **Weather type** (`weatherType`): 1 clear, 2 fair, 3 cloudy, 4 heavily cloudy, 5 overcast,
  6 fog, 10 light rain, 11 rain, 12 heavy rain, 16 showers, 18 thunderstorm, 20 light snow,
  21 snow, 26 sleet. Rain and snow show as precipitation; `rain` (1/0) is the rain sensor.
- **Camera**: `azimuth` is the compass direction the house is seen from (0 north, 90 east, 180
  south, 225 south-west), unwrapped: 225 → 585 is one full turn. `elevation` 0–89 degrees,
  `zoom` relative to the view's own framing (1 = as the view frames itself, 1.5 closer),
  `target` `[x, y, height]` or `"room:<roomId>"`, `ease` `linear`/`inOut`/`cut`. Without keyframes
  the view keeps its own camera. On a page with several views the camera moves the largest one.
- Fields like `window`, `start`, `end`, `language`, `chrome`, `seed`, `scenario` are for screen
  recordings of the demo house; leave them out.

## Which datapoints

Everything comes from the user's configuration, never guessed:

- `datapoints.json` lists every id; `house.json` says which one shows what: `rooms[].data`
  (`temperature`, `targetTemperature`, `lightColor`, `light`, `presence`, …), `openings[].data`
  (`open`, `tilted`, `position`) and their `shades[].data.position`, `lights[].data`, indicators,
  and the house-wide `data` (`outsideBrightness`, `outsideTemperature`, `weatherType`, `rain`,
  `windSpeed`, `sunshine`, …).
- A room's light: its `lightColor` datapoint(s); with a list, set each output.
- Pages: `devices.json`, the Windows device's `landscape.pages[].title`; views: `views/<id>.json`.
- Ask the user what should happen where it is not obvious ("which lights are on in the evening",
  "is anyone home", "which windows open in the morning").

## Pattern: a day in two minutes (like the demo's "Vorführung")

`duration` 120, `base` `script`. Morning to night, a few things at a time:

- 0–20 s dawn: `outsideBrightness` from 1 to 20 000 lx (`ramp` 20, `curve` log); outside
  temperature up a few degrees; shades go up (position 0) one after another; a bathroom or
  kitchen light on early, off again by 25 s.
- 20–40 s morning: a window opens (1) for a while, then closes; presence moves between rooms.
- 40–75 s day: weather may change (clouds, a shower: `weatherType` 16 and `rain` 1 for 10 s);
  lights off; a room temperature rises a little in the sun.
- 75–95 s dusk: brightness down to 50 lx (`curve` log); lights come on room by room, warm white
  (`temp(80,2700)`); outside lights on; shades down (1).
- 95–120 s night: brightness 0.5 lx; lights go off one after another, bedrooms last.
- Pages or views every 15–20 s (`{ "t": 20, "page": "…" }`), or one view with a slow camera turn.

For a live playback, "as it is right now": `base` `live`, one view or page, a camera turn over
20–40 s, nothing else.

## Check

- `"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --house-check <model folder>` checks every playback against the house and lists what
  is wrong as `playbacks/<id>.json: …` (an unknown datapoint, page or view, a value that does not
  fit). A broken playback is not offered in the app.
- `"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --house-check --render <model folder> --playback <id> --at 0,30,60,90,120` renders those
  moments (`playback-<id>-<t>s.png`; a `live` playback shows only its own values there, add `--live`
  for the Miniserver's underneath). Look at them:
  is it dark when it should be, are the right lights on, does the camera show what it should.
- Tell the user where to find it: menu **Vorführungen …** → "Dein Haus"; Esc or a click ends it,
  "Dauerschleife" loops it.

## Examples

The demo house's playbacks in `${CLAUDE_PLUGIN_ROOT}/skills/house-model/example/playbacks/` show the camera, events, ramps and page changes
(`vorfuehrung.json`, `licht.json`, `wetter-zeitraffer.json`, `rundflug.json`). They are written
for the demo house in raw Loxone values over its own scenario; for the user's house write values
in display terms as described above.
