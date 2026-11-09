---
name: house-model
description: Builds or updates the HouseVis 3D house model (house.json) from floor plans, sections, elevations and facade photos, and maps it to the Loxone Miniserver (datapoints.json). Use when the user wants to model a house for HouseVis, "create the house model from my plans", start a new house project, asks which plans or photos to provide, mentions plans/<project>/, or wants to correct or extend an existing model ("the bathroom window is wrong", "add the garage", "map the rooms to Loxone").
---

# House model from plans

Turns architectural plans and photos into a complete HouseVis model: `house.json` (storeys,
rooms, exterior walls, windows and doors, roof, surroundings, light sources) and
`datapoints.json` (the Loxone mapping). Works for any house, not only the HouseVis author's.

Talk to the user **in their language**; everything written to files (ids, notes headings,
comments) follows HouseVis' rules: English, except room and view names, which are written as
the user calls them.

## Ground rules

- **As built beats as planned.** `plan-changes.txt`/`.md` lists what was built differently; it
  overrides the drawings. Apply every change explicitly, including what moves with it (a moved
  wall takes its openings along), note each one in `model-notes.md` under *Decisions*, and ask if
  a change is ambiguous (which door, measured from where).
- **Material comes only from `plans/<project>/` and the user's answers.** Do not read the
  house.json or datapoints.json of the app's configuration folder (the one HouseVis shows) to fill
  gaps — unless the user asks to update that very model. (The owner's own house is the acceptance test: it must be rebuilt from
  the plans, not copied.)
- **Never invent precision.** Every value comes from a printed dimension, a level mark, a photo,
  the user — or is a typical value that is marked *estimated* in `model-notes.md` and mentioned to
  the user. When two sources disagree by more than 2 cm (or 1° for roof pitches), show both and
  ask.
- **Which source wins.** For the building itself: the architect's dimension chains, then
  `plan-changes`. For products added later (conservatory, carport, canopy): the manufacturer's
  drawing for sizes and heights, the owner's own drawings or photos for the position. For the
  surroundings (terrace, paths, drive, lawn): the owner's own site sketch, since it is as built,
  but only to ±10 cm (check its scale against the house width; owners' sketches are often a few
  percent off). Record in `model-notes.md` which source each value came from.
  Prefabricated parts (garages, carports) can be built lower or higher than their elevation
  shows (a prefab garage drawn 2.82 m and 2.60 m high stood 2.50 m): check such heights on a
  photo against a known dimension in the same wall plane, e.g. a door's height, and ask when they
  disagree.
- **Furniture is not building.** Things standing in front of a window or wall in a photo (stacked
  chairs, shelves, plants, a barbecue) are never a building part: no parapet, no fixed glazing,
  no wall. When a photo seems to contradict the plan, ask instead of modelling it.
- **Render faults are not always model faults.** If the format can express something correctly but
  the scene draws it wrong (white stripes on a coloured facade, missing faces), that is a HouseVis
  fault: in the HouseVis repository fix the scene (with a test, as a separate commit when the user
  asks); elsewhere tell the user to report it to the HouseVis author with the render. Do not bend
  the model around it. Workarounds in the model are fine only for what the format cannot express, and
  they go into `model-notes.md`.
- **Confirm block by block.** After each block below, show the result (summary + plan or render
  images) and wait for the user's confirmation or corrections before the next block. Errors in the
  rooms carry into everything after them.
- **Plans and photos stay local.** `plans/` is git-ignored; never commit, upload or copy its content
  anywhere else. The generated model in the `<model folder>` is personal data too.
- **Never overwrite a house model.** A new model goes only into a folder without a `house.json`
  (step 0). Changing an existing model is fine when the user asks to correct or extend that very
  model; anything else in the app's configuration folder stays untouched. For a complete example
  of every format feature see the demo house in `${CLAUDE_PLUGIN_ROOT}/skills/house-model/example/`.
- Ask, don't guess, for everything Loxone-specific (which area is which Loxone room, which output
  lights which area).

## Files

| Path | Who writes it | What |
|---|---|---|
| `plans/<project>/…` | user | the architect's PDF set as delivered, plus named single files and photos, see [reference/intake.md](reference/intake.md) |
| `plans/<project>/model-notes.md` | skill | sources per value, decisions, estimates, open questions, progress |
| `<model folder>/house.json` | skill | the model; the folder is chosen in step 0 |
| `<model folder>/datapoints.json` | skill | Loxone mapping (block 7) |
| `<model folder>/devices.json`, `views/` | `house:check --views` | eight check views |
| `test-output/house-check/` | `house:check` | plan drawings and renders for comparison |
| `test-output/plan-tiles/<file>[-p<page>]/` | `plans:tiles` | full-resolution tiles of plan sheets and scans |

Reference: [reference/format.md](reference/format.md) (from plans to coordinates; read it and
`${CLAUDE_PLUGIN_ROOT}/skills/house-model/schema/house.schema.json` before block 1), [reference/loxone-mapping.md](reference/loxone-mapping.md)
(block 7), `${CLAUDE_PLUGIN_ROOT}/skills/house-model/reference/model-format.md` (the format and its reasoning).

## Tools

The tools are the installed HouseVis app's command line. It is installed per user, by default
as `%LOCALAPPDATA%\Programs\HouseVis\HouseVis.exe`; the commands below call it that way from the
Bash tool. If that file does not exist, ask the user where HouseVis is installed (or to install it)
and use that path. In PowerShell: `& "$env:LOCALAPPDATA\Programs\HouseVis\HouseVis.exe" --house-check …`.
Run these commands yourself. Should the user ever have to run one, give it in the Bash form,
`"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" …`, as below: Claude Code's `!` prefix runs
Bash, where `%LOCALAPPDATA%` is not replaced.
Relative paths (`plans/<project>/…`, `test-output/…`) are relative to the folder Claude Code
works in.

```
"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --house-check <model folder>            # validate, summary, plan-<floor>.png
"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --house-check <model folder> --views    # (re)write devices.json + 8 check views
"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --house-check <model folder> --walls    # also list the wall ids a view can set
"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --house-check --render <model folder>           # 3D renders, one per view
"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --house-check --render <model folder> --lit=lights --lux 1
                                                         # the same at night, lamps on, rooms dark
"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --plan-tiles plans/<project>/<file> [--pages 2] [--rotate 270]
                                                         # PDF page or scan → full-resolution tiles
"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --config-folder                                    # the folder the app shows a house from
"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --inventory                                 # loxone-inventory.json (block 7)
HOUSEVIS_CONFIG=<model folder> "$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --values
                                                         # live value of every datapoint (block 7)
```

All options, with their defaults: `house:check` takes `--out <folder>` (default
`test-output/house-check/`), with `--render` also `--lit` (every room and lamp on), `--lit=lights`,
`--lux <n>` and `--weather <code>`; `plans:tiles` takes `--pages 1-3,5`, `--dpi 50..600` (300),
`--rotate 0|90|180|270`, `--tile <px>` (1400) and `--out <folder>` (default
`test-output/plan-tiles/`). `"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --house-check --help` prints the complete list. Exit codes:
0 done, 1 failed (errors listed at the end), 2 wrong arguments. Do not try to discover options by
starting the tools with guessed arguments: an argument the app does not know starts the app
itself.

`house:check` exits with errors listed at the end; fix them before showing anything to the user.
Open the PNGs it writes with the Read tool and compare them with the plans yourself first.

## Step 0 — project and intake

1. Ask for the project name if it is not clear (lower-case, digits, hyphens), e.g. `plans/korn-haus`.
   **Where the model goes** (the `<model folder>` of every command below; ask before writing
   anything into it): run `"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --config-folder`, which prints the folder HouseVis shows its
   house from and whether a house model is in it.
   - No house model there: suggest that folder, so the app shows the new house right away; the
     user confirms it or names another.
   - A house model is there: **never** write the new one into it. Say whose model it is (the one
     the app shows now), suggest `plans/<project>/config/` instead and ask.
   - For an existing project, take the folder `model-notes.md` names (*Model folder*); for a
     correction of the model the app shows, the folder `config:folder` prints.
   Note the choice in `model-notes.md` under *Model folder*. When it is not the app's folder,
   tell the user to open it in HouseVis (menu **Konfigurationsordner …**, the app restarts with
   it) to watch the house take shape.
2. If `plans/<project>/` does not exist or is empty: present the intake from
   [reference/intake.md](reference/intake.md) in the user's language — where to put the files,
   that the architect's multi-page PDF can go in unchanged, how to name material without a title
   (photos, untitled sheets), which plans and photos are needed — and stop until the files are
   there.
3. Read `plan-changes.*`, `room-names.*` and `notes.*` first if they exist, so every later step
   knows what was built differently and what the rooms are called. Then inventory the folder. Multi-page PDFs (usually the architect's set, any file name): go through
   the pages (Read tool, `pages`, at most 20 per call) and identify each sheet by its title block
   ("Grundriss EG", "Schnitt A-A", "Ansicht Süd", scale, plan date/revision). Single files: by
   their name prefix, and check that the content matches. Write the result as a page index into
   `model-notes.md` (*Sources*: file, page, what, scale, date) so later blocks open the right page
   directly. If two sheets show the same thing with different dates, ask which one is current.
   Report to the user: what is there, what is missing, what is unreadable, and what that means
   (e.g. "no elevations: window heights will be estimated"). Required are a floor plan per storey,
   at least one section, and the north direction; ask for missing required material before
   starting, offer to continue with estimates for the rest.
4. Plan sheets are far too large to read whole: the Read tool scales them down until the
   dimension figures are illegible, and many architects' PDFs contain no text layer (dimensions
   are drawn as vector outlines). Run `"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --plan-tiles <file> --pages <n>` (PDF pages are
   rendered at 300 dpi) and read the tiles; the output lists each tile's pixel range. Check the
   `overview.png` first: drawings are often placed sideways on the sheet, then rerun with
   `--rotate 90` or `--rotate 270` until the title block reads upright. Note the page, rotation
   and tile folder in `model-notes.md`.
5. Create `model-notes.md` with sections *Sources*, *Decisions*, *Estimated values*, *Open
   questions*, *Progress*. Record which file and page each block used. Keep it updated after every
   block; it lets a later session continue where this one stopped (read it first when resuming).
6. Ask the questions the plans cannot answer, all at once. **Room names:** unless
   `room-names.*` covers every area, list the plan's rooms per storey (name and printed area) and
   ask explicitly for the Loxone room name of each and which areas belong to another room
   (walk-in closet → bedroom, sauna → bathroom). Never derive Loxone names from the plan: plans
   often carry old or generic names ("Arbeiten" may now be "Musikzimmer"). Offer to write the
   answers into `room-names.txt` (format in [reference/intake.md](reference/intake.md)) so they
   survive the session. Also ask whether anything was built differently that `plan-changes` does
   not list yet (offer to write it there).

## The blocks

Each block: extract → write → `"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --house-check` → compare the output with the source yourself
→ show the user (summary table in their language + the relevant images) → wait for confirmation →
update `model-notes.md`.

1. **Frame.** Storeys (ids such as `ug`, `eg`, `og`, `dg`; names as the user calls them), floor
   heights and exterior wall thickness per storey from the section, origin (outer south-west corner
   of the main building), north direction and how the house is aligned. Show: a table of storeys
   with level, height, wall thickness, source.
2. **Rooms per storey.** Rooms (Loxone names) and spaces from the dimension chains; check that every
   chain adds up; every area becomes a space (hallways, stairs, storage). Show: `plan-<floor>.png`
   per storey next to the original plan, and the areas compared with the areas printed in the plan.
   Typical findings to look for: gaps wider than 0.7 m that should be walls (overlapping or missing
   rooms), rooms not aligned between storeys, an interior wall that came out too thick.
3. **Openings.** Windows and exterior doors per wall with width, sill, top, type, sashes, side
   panels, shades (elevations and photos). Show: per storey the list from `house:check` and the
   plan image; mark estimated heights.
4. **Roof.** Parts, footprint with overhangs, eave and ridge heights, pitch check, roof windows.
   Show: the pitch printed by `house:check` against the section's value; the top storey's plan with
   the roof faces.
5. **Surroundings.** Terrain level, terrace and gravel areas, pits and light wells, exterior stairs,
   steps and canopies, conservatory, garage as outbuilding. All of it against the outer faces.
   **A garage (or any closed outbuilding) is built complete:** its area (`outbuilding: true`, the
   polygon is its inside) with its own `exteriorWall` and, if its floor lies lower, `level`; a roof
   part; gates as doors with `"operation": "overhead"` in its walls; the area (and roof part)
   tagged with one group name ("Garage") — see *Surroundings* in
   [reference/format.md](reference/format.md). It is then a room like the house's (active in a
   view, presence, labels). Its walls will hide part of the house in some views; tell the user
   that a view can make them transparent, leave the group out (`hide`) or give it a switch
   (`toggles`).
   **Tag** what the kind cannot tell (see *Tags* in [reference/format.md](reference/format.md)):
   `outside` on freestanding parts (garden wall, carport roof and posts), group names for parts
   that belong together ("Carport", "Garage"). Show the tag list from `house:check`.
6. **Visual comparison.** `"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --house-check … --views`, then `"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --house-check --render …`.
   Put each `view-<direction>.png` next to the matching facade photo (`photo-<side>` and
   `view-<side>` both mean the house seen from that side; if a photo plainly shows another
   facade than its name says, ask instead of trusting the name) and list the differences
   (missing window, wrong roof overhang, wrong facade color). At most 2–3 rounds of corrections;
   after that, list what remains and let the user decide.
   Things that are easy to miss: frame colours per window (one room may have white frames, the
   rest anthracite), the door's hinge side (the handle sits on the opposite side, often next to
   the sidelight), roof and flat-roof colours, which part of a lift-and-slide door moves.
   The Read tool shows photos scaled down; for details, cut the photo into full-resolution tiles
   with `"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --plan-tiles <photo> --tile 800` and read the tile you need. A
   photo taken square-on to a facade can be measured: scale the pixels against a known length
   (e.g. the facade width) and compare window centres with the model (±5–10 cm is a match).
7. **Loxone mapping.** As in [reference/loxone-mapping.md](reference/loxone-mapping.md): needs
   `loxone-inventory.json` (the user runs `"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --inventory` after connecting HouseVis to
   the Miniserver). Suggest, the user confirms. Include the house-wide `data` (see *Weather* in
   [reference/loxone-mapping.md](reference/loxone-mapping.md)): `outsideBrightness` (the scene's
   daylight follows it, so lit rooms and lamps stand out at night) and the weather values for the
   weather station's panel and a screen's sky (temperature, felt temperature, humidity, wind,
   wind warning, rain, sunshine, weather type). Ask which family each comes from when Loxone has
   several (weather station, weather service, System).
8. **Light sources** (optional; offer it, the user decides). Lamps besides the room light,
   see *Light sources* in [reference/format.md](reference/format.md). Start from what exists:
   the light outputs in the inventory (outside ones usually sit in an "Außen…" room: wall
   spots, ground-cable lights, path lights; inside ones as extra outputs of a room's light
   controller, e.g. "Essplatz", "Vitrine"), the luminaire symbols of an electrical plan, facade
   photos, the user's description ("two bollards along the path to the door"). List them as a
   table (Loxone output → kind → where → height → source) and ask for what is unclear: where
   exactly, which lamps one output switches, wall light up, down or both. Then write them with
   their `data` (block 7's datapoints), tag outside groups ("Garten") and run `house:check`
   (each light's line, amber markers in the plans). Render with `"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --house-check --render …
   --lit=lights --lux 1` (lamps on, rooms dark, night) and show it; check that no wall light hangs
   in a window and that facade lights sit on the right facade. Lamps on the room's own output
   (spots on the room's dimmer) need no data: they follow the room.
9. **Weather station** (optional; offer it when Loxone has weather values, the user decides).
   Place `weatherStation` where the station really stands, see *Weather station* in
   [reference/format.md](reference/format.md): on the roof (a Loxone Weather Station Tree sits
   on a short mast there, often near the ridge or a gable), on a wall bracket or a post in the
   garden. Sources: roof or facade photos, the user's description ("on the ridge, two metres from
   the east gable"). The app draws a cup anemometer there that turns with the wind and the
   weather panel above it (outside temperature, felt temperature, wind, humidity from block 7's
   `data`). Run `house:check` and render; show where it stands. Then tell the user: a page can
   show the current weather as its sky with rain or snow over it, switched on per page in the
   device editor ("Wetter als Himmel"); the menu's "Wetter-Demo" shows every weather type.

Views and devices (pages) for everyday use are not part of this skill: the user makes them in the
Windows app (menus "Ansichten bearbeiten …" and "Geräte / Seiten bearbeiten …"); the check views are only
for comparison. Mention that the user's own views must list the room ids
of the new model. Exception: on request a view may get `floors` (only some storeys) and `hide`
(leave out tags such as `outside` or `roof`), see *Views that leave storeys or tags out* in
[reference/format.md](reference/format.md).

## Finishing

- Run `"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --house-check <model folder>` a last time: no errors.
- Unless the `<model folder>` is the app's folder (`"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --config-folder`): tell the user to open
  it in HouseVis, menu **Konfigurationsordner …** (the app restarts with it and keeps that folder).
  Copying the model over a house the app already shows is a separate step only the user asks for
  (back up the existing files first, keep their `views/` and `devices.json` if wanted).
- `model-notes.md`: final list of estimated values and open questions.
- **Feedback in the forum.** Once a whole house is finished (not after small corrections), mention
  once that the HouseVis forum (in German) collects how it went in *Haus-Assistent &
  Skill-Feedback*, https://forum.housevis.de/c/skill-feedback: what was right at once, where you
  got it wrong, cost or usage limits, duration. That is how this skill gets better. Offer to
  draft the post in the user's language from `model-notes.md` (route, kind of house, material,
  the corrections the user had to make); it must contain no address, names or complete plans.
  If the user gets stuck earlier on something you cannot resolve, point to *Hausmodell & Pläne*,
  https://forum.housevis.de/c/hausmodell, with the same caution about plans. Never more than one
  mention per occasion.
- **Extend this skill.** Every real house teaches something (a plan convention, a Loxone naming
  habit, a render pitfall, a new kind of input file). Propose to the user what to add, as general
  rules with the house only as an example, into the reference file where a later session would
  look for it; house-specific values stay in `model-notes.md`. Commit it only when asked.

## Updating an existing model

For "the bathroom window is wrong", "add the garage", "the walls are thicker": read
`model-notes.md` and the model, change only what is affected, keep ids stable (views and
datapoints refer to them), run `house:check`, show the affected plan or render, and add the
change with its source to `model-notes.md`.

The weather station too: "the weather station stands on the garage roof", "move it a metre to
the west", "the station is 1.5 m above the ridge"; and the weather values: "take the System
temperature instead of the station's".

Light sources are changed only here (the app has no editor for them): "add a pendant over the
dining table", "the lantern stands 50 cm further left", "the wall light also shines up",
"remove the garage spot". Keep their ids, map new ones as in block 8, and render with `--lit=lights`.

New elements get their tags like in block 5. Requests about groups ("make a group Garten with the
garden wall and the pergola", "move the pergola into Garten", "rename Garten", "the garden wall is
outside", "show only the ground floor in view X") follow *Tags* and *Views that leave storeys or
tags out* in [reference/format.md](reference/format.md); a renamed or removed tag is also renamed
or removed in the views' `hide`. Older models have no added tags: when the user asks for groups or
for views without the outside, offer to go through the structures, roof parts and pavilions once
and tag them. Older models may also have a garage without walls, or with its walls as
`structures` boxes: offer to convert it into an outbuilding with `exteriorWall` (and `level`),
shrinking its polygon to the inside, moving its gates onto the inner edge and removing the boxes
(update views that set the boxes: their walls now have wall ids).
