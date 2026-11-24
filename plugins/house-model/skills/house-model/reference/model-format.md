# House model format (house.json)

Format **version 7** (`${CLAUDE_PLUGIN_ROOT}/skills/house-model/schema/house.schema.json`). This document describes the format and the
reasoning behind it; the house-model skill's `reference/format.md` explains how to get the values
out of plans and photos.

## Principles

- **Versioned**: `"version": 7`. Bump the version only for breaking changes, i.e. when an old file
  would be read wrongly or rejected. New optional fields and new `kind`s are additive and need no
  new version; the schemas are strict, so a file using one does not load in an older app.
- **Authored vs. internal.** `HouseFile` is what people (and the skill) write; `normalizeHouse()`
  turns it into the internal `House` once, resolving shorthands and defaults. Geometry and scene
  code only see the normalized model.
- **Element families are discriminated by `kind`** (roof parts, openings, lights, …), so new kinds
  can be added without touching the existing ones.
- **Stable ids** on everything views, the editor or datapoints refer to. Ids written by hand match
  `^[a-z0-9][a-z0-9-]{0,39}$` where the schema says so.
- **Placed by plan points, never by derived ids.** Openings, lights and furniture give coordinates;
  wall ids are derived from coordinates and change with the room, so nothing in house.json names
  one.
- Coordinates in meters, plan coordinates `[x, y]` with x to the east and y to the north; heights
  relative to the floor of the element's `floor`. Rooms are interior dimensions; walls result from
  the gaps between rooms.

## Versions and conversion

Older formats are **converted**, since users have files of their own: `src/model/migrate.ts` holds
one plain-JSON step per version (from 6 on), and every load runs them in memory (`checkedHouse`,
also on Android and for imported export files, `bundleCheck.ts`). The Windows app and
`--house-check` write the result back once and keep the old file beside it as `house.v<n>.json`
(`electron/houseUpgrade.ts`), so the editors, the house assistant and the skill only ever see the
current format. The new file is written first and the backup made once that worked (an identical
earlier one is reused); a failure leaves the folder as it was, and a conversion that lacked
something (the rooms, when the rest of the house is broken) is only shown, never written. Versions
before 6 and newer ones are refused with a clear message (`src/model/version.ts`).

A breaking change adds a step and keeps the old ones; its test converts a frozen file of the old
format (`src/testing/house-v6.json`) and checks the result against the test house. The one step so
far is 6 → 7: roof parts written as plans dimension them (wall line, overhang per side, eave height
at the wall line) instead of a footprint with the eave at its edge; the conversion keeps the old
roof planes exactly.

## Floors and areas

```json
{ "version": 7,
  "floors": [ { "id": "eg", "name": "EG", "height": 2.9, "exteriorWall": 0.365 } ],
  "rooms": [ { "id": "kueche", "name": "Küche", "data": { "temperature": "kueche-temp" } } ],
  "spaces": [ { "id": "kueche", "roomId": "kueche", "floor": "eg", "polygon": [[0.37, 6.32], …] } ] }
```

- `floors` from bottom to top; `height` is floor to floor including the slab.
- `rooms` are rooms as the home automation names them; `spaces` (areas) are their floor areas. A
  room may have several areas (a bedroom with its walk-in closet); an area with a plan `name` of
  its own belongs to another room and gets no label of its own (unless it is the room's only one).
- `spaces[].id` is required and unique; views and wall ids refer to it, so keep it stable.
- Area polygons may have edges in any direction; validation reports overlapping areas and
  outlines that cross themselves. `height` overrides the storey's room height; `height: 0` is an
  outdoor area (terrace, balcony). `facadeColor` is the wall seen from outside (default white).
- `level` on any area: its floor that far below (negative) or above its storey's, a **split
  level** (a sunken living room, a raised dining area). Its default height keeps its ceiling at
  the storey's; the storey's slab steps down to it. Sills and the heights of lights, furniture and
  indicators still count from the storey's floor (a terrace door in a room 0.36 m down has
  `"sill": -0.36`), down to the area's floor. Not together with `openTo`; less than a storey. The
  steps between levels are a stair staying on its storey, see *Stairs*
 .
- The floor slab with the interior walls standing in it is built from polygons (`src/model/slab.ts`,
  Clipper2): the areas are grown by half the maximum wall gap (0.7 m), merged and shrunk back, so
  gaps between areas close at any angle; slab minus areas is where the interior walls stand.

### Double-height spaces

```json
{ "id": "wohnen", "roomId": "wohnen", "floor": "eg", "openTo": "og", "polygon": [[0.37, 0.37], …] }
```

- `openTo` on an area makes it a double-height space ("Luftraum", a two-storey hall or bay):
  open through the storeys above its own up to and including `openTo`. Write it on the **lower**
  area, the room standing there; the crossed-out area on the upper plan gets no area of its own.
- Those storeys have no floor over it, and no area of theirs may overlap it; their outline still
  closes over it. Its walls run through to the slab or roof of `openTo` (its height defaults to the
  storeys' heights added up; `height` still overrides it).
- Its windows are ordinary `openings` on its own storey and may reach up to its full height
  (`"sill": 0.5, "top": 4.8`); every other opening stays below its storey's height. A window across
  a slab between two ordinary rooms is drawn as two windows.
- Not on outbuildings or outdoor areas.

### Stairs

```json
{ "id": "treppe", "floor": "eg", "to": "og", "line": [[5.0, 5.6], [6.1, 5.6], [6.1, 8.1], [4.32, 8.1]], "width": 1.0, "rises": 16, "turns": ["landing", "winders"] }
```

- `stairs`: a stair between two storeys, written by its walking line (Lauflinie) as a plan
  draws it: from the front edge of the first step to the edge of the upper floor, a corner
  wherever it turns, every corner 90°. `floor` where it starts, `to` where it arrives (default
  the next storey up), `width`, `rises` (the rise follows from the floor-to-floor height),
  `turns` per corner (`landing`, the default, or `winders`), `color`, `tags`. `to` may be its own
  storey: steps between split levels; the rise is always from the floor where it starts to the
  floor where it arrives, each with its area's `level`. A `to` more than one storey up needs a
  double-height space the stair stands in, open through the storeys between.
- The treads go on the flights and winder corners by walking length (a landing is one tread);
  the storey above loses the stair's footprint from its slab and floors (the Treppenauge).
- Shown like furniture (its storey, its room opened, implied tag `stairs`). Validation keeps the
  rise at 12–25 cm and the goings at 18–40 cm, the start in an area of its storey and the end at
  one of `to`.

### Railings

```json
{ "id": "galerie", "floor": "og", "line": [[4.2, 4.52], [4.2, 9.135]], "kind": "glass" }
```

- `railings`: along a plan `line` on a `floor` (any angle; `base` its foot above the floor,
  `height` default 1.0), or along a `stair`'s `side` (`left`/`right` walking up; `height` above the
  nosings, default 0.9). `kind`: `glass`, `bars` or `solid`; `color`, `tags`.
- A line railing parallel to an interior wall and at most 0.5 m off it on its outer side (in the
  gap) **opens the wall** there: `Wall.open` stretches, from the area's floor, or above the
  storey's floor for a double-height space's face rising through it. The face gets a notch, the
  gap's wall top goes, the slab stays. Exterior walls are never opened. Where a stair arrives
  through it, the railing leaves the stair's width free.
- Shown with a room beside it that the view opens, a stair's with its stair, outside always;
  implied tag `railing`.

## Rooms and their values

`rooms[].data` maps a room's **roles** to datapoints in `datapoints.json`: `temperature`,
`targetTemperature`, `humidity`, `lightLevel`, `climateMode`, `climateOverride` (numbers);
`heating`, `cooling`, `light`, `presence` (true/false); `lightColor` (a colour, also a list).

- `climateMode` (IRoomControllerV2 `activeMode`) and `climateOverride` (`overrideReason`) show the
  mode as an icon left of the temperature, "paused" for an open window (override 2) or the pause
  timer (15) (`src/model/climateMode.ts`). "heating", "cooling" and the modes ("comfort", "eco",
  "paused", "protection") are indicator kinds of the rooms' own, so global.json picks their icons.
- `humidity` (percent) is read like the temperature: from the room's `IRoomControllerV2`
  (`humidityActual`) when its humidity input is wired (`details.connectedInputs` bit 14, the
  inventory's `humidityInput`), else from the room's own sensor, usually its Touch. The temperature
  label's small line shows it after the target temperature, rounded to whole percent. The target
  and the humidity get icons from global.json's kinds `target-temperature` (default →|) and
  `humidity` (default droplet; the same kind as an indicator writing a humidity value, so a house
  picks one icon for both); "hidden" leaves the bare number. A room's humidity is this role, not an
  indicator with a value in the room's row: that stays for values without a room role (a pond, a
  heat pump).

## Openings

```json
{ "id": "kueche-west-1", "kind": "window", "floor": "eg",
  "start": [0.36, 4.9], "end": [0.36, 5.9], "sill": 0.85, "top": 2.2, "operation": "turn" }
```

- `start`/`end` lie on the inner face of the wall (an edge of the area polygon, tolerance 5 mm);
  their order does not matter. The owning room and the facing direction are derived
  geometrically; validation reports openings on no area edge. This works for walls at any angle.
- `kind` (`window`/`door`) and `id` are required. `side` (`left`/`right` seen from inside),
  `operation`, `leaves`, `sidelight`, `fixedBelow`, `color`, `data` (`open`, `tilted`, `position`)
  and `shades` (`roller`/`blind`; `from`/`to` are distances from `start`) are optional.
- `sill` is above the storey's floor; negative only in an area whose floor lies lower (an
  outbuilding's or a split level's `level`), and never below that floor.

### Interior doors, passages and glass partitions

```json
{ "id": "diele-wohnen", "kind": "door", "floor": "eg", "start": [4.32, 4.9], "end": [4.32, 5.8], "sill": 0, "top": 2.1, "glass": true },
{ "id": "kueche-essen", "kind": "door", "floor": "eg", "start": [2.4, 6.32], "end": [3.4, 6.32], "sill": 0, "top": 2.1, "operation": "none" }
```

- An opening on an interior wall is written like any other, `start`/`end` on **either** room's
  face; the room across the wall gap (up to 0.7 m) follows by itself, both faces get the hole and
  the scene draws the reveals across. It belongs to the room whose face carries it (its data, its
  id). Where nothing faces it across (a wall that borders a room elsewhere) it is an exterior one.
- `"operation": "none"` on a door: a passage without a leaf (only the hole). `glass: true` on a
  door: a glazed leaf. `frame: false`: frameless glass held by fittings (a sauna's glass front and
  door), on doors and windows. A glass partition is a `fixed` window, `sill` 0 for full height.
- `top` stays below the room across too (a sauna lower than the bathroom it stands in). The room
  across may be a double-height space: a partition from the gallery into it counts from the
  gallery's storey.

## Wall ids

Derived for views and the editor, never written in house.json: `<spaceId>@<x1>,<y1>-<x2>,<y2>`,
endpoints rounded to centimeters and sorted, e.g. `kueche@0.36,4.29-0.36,8.05`. Moving another
corner of the room keeps the id; a view setting whose wall no longer exists becomes a validation
warning.

Faces of the outer outline that belong to no room wall (interior wall ends, steps in the facade)
get the same kind of id with the floor in place of the area: `eg@1,4.21-1,4.34`. They cannot clash
with a room wall, since they exist only where no room face lies. `"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --house-check --walls`
lists every id a view can set.

## Exterior walls

`floors[].exteriorWall`: structural thickness in meters, default 0 (the facade then lies on the
area edges). Areas stay interior dimensions; the exterior walls lie outside them.

- **Geometry** (`src/model/shell.ts`): the storey's slab (areas plus interior wall gaps) is grown
  by the thickness with mitred corners; the strip gained is the ring of exterior walls. Each
  exterior wall of an area gets an outer face (its edge moved out, mitred at the area's own
  exterior corners). Stretches of the outer outline no room face covers (interior wall ends, steps
  where an edge is only partly exterior) become filler faces; one with a room right behind it looks
  like that room's wall.
- **Scene**: facades and the glass walls of open rooms stand at the outer face; the back walls stay
  on the area edge. Openings get reveals through the wall; windows sit 12 cm behind the outer face.
  Gable walls close at the outer face; the terrain is cut out down to it.
- **Consequence for authored coordinates**: everything placed against the house (terrace, gravel
  strip, pits, conservatory, steps) must touch the outer face, not the area edge. A roof's
  `outline` is the outer faces, its `overhang` is measured from them.

## Outbuildings

An area with `outbuilding: true` (a garage) lies outside the main building and is no wall
neighbour of the house or of other outbuildings. It is a room like any other and can be active in
a view.

- `exteriorWall` on an outbuilding area gives it walls outside the polygon like the house's (the
  polygon is the inside): its own shell (`outbuildingShell`: slab, wall ring, outline), facades when
  inactive, room walls when active, walls settable per view. Without it the outbuilding has no
  walls (an open shelter): it can be active, showing floor and labels, but no name (that needs a
  wall).
- `level` on an outbuilding area: its floor relative to the storey's (a garage at −0.32). `height`
  counts from there; the storey's roof parts still clip the walls (`BuildContext.spaceTop`);
  openings keep counting from the storey's floor, so a gate at the garage floor has
  `"sill": -0.32`. Unlike a split level's, an outbuilding's default height does not change with
  `level`. `exteriorWall` on an area only on outbuildings (validation).
- The terrain leaves out the footprint of outbuildings with walls, like the house's, so a garage
  floor below the terrain shows.
- An outbuilding with walls is a building, not part of the grounds: it carries the implied tag
  `outbuilding` but not `outside`, so "everything outside" does not hide the garage. One without
  walls is `outside` too.

## Roof parts

`roofs` is a list of parts, each written as plans dimension it, here a gable roof with unequal
overhangs:

```json
"roofs": [
  { "id": "main", "kind": "gable", "floor": "og",
    "outline": { "x0": 0, "x1": 11, "y0": 0, "y1": 9.5 },
    "overhang": { "south": 0.8, "north": 0.3, "east": 0.6, "west": 0.6 },
    "ridge": "east-west", "eaveHeight": 1.2, "ridgeHeight": 4.5 }
]
```

| kind | extra fields | shape |
|---|---|---|
| `gable` | `ridge` (`north-south`/`east-west`), `eaveHeight` (one or per side), `ridgeHeight` or `pitch` (one or per side) | two slopes on the sides along the ridge, gable walls at the ends |
| `halfHip` | like `gable`, plus `hipEaveHeight` and `hipPitch` (one or per gable side) | a half-hip roof (Krüppelwalm): a gable whose gable walls end at `hipEaveHeight` (on the gable's wall line) in a small hip at `hipPitch`; a gable side without a value stays a full gable |
| `hip` | like `gable` | a slope on every side; with `ridgeHeight` one pitch everywhere and the ridge shorter than the outline by its width |
| `shed` | `low` (side), `eaveHeight` (at the low wall line), `ridgeHeight` (at the high one) or `pitch` | one slope |
| `flat` | `height` | flat roof |
| `polygon` | `outline` as corners, `eaveHeight`, `pitch`, `overhang`, `edges` (one per outline edge: `"slope"`, `"gable"` or its own values) | a slope on every edge that is not a gable; hips at outside corners, valleys at inside ones |

- `outline` is the **wall line** under the part (outer wall faces); `overhang` reaches beyond it,
  one value or per side (missing sides 0). `eaveHeight` is the height **at the wall line**, where
  sections give it. Each slope is a plane through its side of the outline at its eave height,
  rising at its pitch; the surface is the lowest of the planes, cut off at the outline plus the
  overhangs (`roofFootprint`). With `ridgeHeight`, one pitch for every slope: the one at which the
  two slopes across the ridge meet at that height (so different eave heights move the ridge off the
  middle); `pitch` instead gives it per side.
- Optional `rotation` (degrees, counter-clockwise around the outline's middle; not on polygons) for
  houses not aligned with the axes; `ridge`, `low` and the overhang sides refer to the outline
  before rotation. Heights are relative to the part's `floor`.
- Optional `color` (`"#rrggbb"`) for the covering seen from outside (default dark red). The seams
  are drawn 0.6× darker in sRGB, the underside keeps its light tone.
- **Geometry** (`src/model/roof.ts`): every part is broken down into flat faces (plan polygon +
  plane). Within a part the surface is the *lowest* of its planes; where parts overlap, the
  *higher* part counts, so a wing running into the main roof forms the valleys by itself.
  `roofHeightAt(point, floor)` considers the parts of that floor; wall tops, divider tops, gable
  and attic walls and sloped ceilings all derive from it.
- Walls of a floor reach up to the roof instead of stopping under a slab only if the floor has a
  roof part and no floor above has rooms (a porch roof on the ground floor does not count). Under a
  sloped part only walls on its wall line (within 2 cm) close up to the roof; every other wall ends
  at its room's ceiling, since the attic is not modelled. A gable side without such a wall stays
  open and is named by `house:check` and the configuration warnings; a side where the roof carries
  on at the same height as another part needs none.
- **Views** draw a part `solid`, `cutaway` (the default: see-through above the view's open rooms
  where the slope is their ceiling), `transparent` or `hidden`.
- Known limits: overlapping parts are drawn completely (the hidden piece lies under the higher
  roof); a wing whose roof reaches beyond the rooms shows its open end.

### Polygon roof parts

`kind: "polygon"`: `outline` a list of corners (the wall line, either direction), defaults
`eaveHeight`, `pitch` and `overhang` on the part, and `edges` with one entry per outline edge (edge
i from corner i to the next): `"slope"`, `"gable"`, or an object with its own `gable`,
`eaveHeight`, `pitch`, `overhang`. No `ridgeHeight` (an arbitrary outline has no one ridge that
would fix the pitches) and no `rotation`. Pitch 0 everywhere is a flat roof. One part covers L, T
and U shapes, cross gables and wrap-around canopies with the right hips and valleys; a rectangle
written as a polygon gives the rectangle kinds' heights exactly.

- Geometry (`src/model/roofPolygon.ts`): each slope's plane applies only within its *reach*, the
  footprint bounded at each end of its edge by the line where it meets the neighbouring slope, or
  by a neighbouring gable's line; the surface is the lowest plane reaching a point.
- Validation: one edge entry per corner, at least one slope, pitches below 90° and not 0 beside
  sloped ones, a simple outline, slopes covering the footprint. Gable walls and the open-gable
  warning work per gable edge, named by number.

### Roof windows

`roofWindows`: `{ "id", "roof": "main", "center": [x, y], "width": 1.14, "depth": 1.42 }`, `width`
along the eave, `depth` across it in the plan. The slope under `center` holds the window;
validation reports windows that reach into another slope. Shade `from`/`to` run along the width
from the left edge seen from outside looking up the slope.

### Dormers

`dormers`: `{ "id", "roof", "kind": "shed" | "gable", "front": [[x, y], [x, y]], "eaveHeight",
"pitch", "overhang"?, "color"?, "wallColor"? }`, written as plans dimension a dormer: by its front
(the front wall's outer face, its ends in either order), the height of the front wall's top (shed)
or the cheeks' top (gable) above the host part's storey, and the pitch of its own roof.

- It stands on the slope of roof part `roof` under the front's middle and faces down it; the front
  runs along that slope's eave. How far back it reaches follows from where its eave (gable) or its
  roof (shed) meets the slope. `overhang` one value or `{ "front", "sides" }`. `color` its roof
  (default the part's), `wallColor` its front and cheeks (default the facade colour of the room
  below). Ids unique among roof parts and dormers.
- The host slope is cut away within the dormer's wall line; the rooms below get its roof as
  ceiling inside the walls' inner faces.
- Windows in its front are ordinary `openings` on the host's storey, their ends on the front's inner
  face or, as `front` is written, its outer face; their room is the one behind them. `sill` must
  not lie below the slope at the front's foot, `top` not above the front wall.
- A dormer goes with its roof part in views (its `roofs` entry, tags, storey filter); it has no
  entry of its own.
- Validation: host present and sloped at the front, front at least 0.1 m and along the eave
  (within 5°), eave above the slope there, a shed's pitch below the slope's, the whole dormer on
  that one slope (not across a ridge, hip, valley or the edge), the front within the part's wall
  line, nothing under it (roof windows, PV arrays, other dormers).

### PV arrays

`pvArrays`: a rectangle of panels lying in one slope of a roof part, placed like a roof window
(`roof`, `center` in the plan, `width` along the eave, `depth` across it in the plan) with
`columns` × `rows` panels. Validation: within one slope, not over a roof window of that slope, ids
unique. Drawn without thickness just above the roof; the roof's cut-away cuts the panels too,
leaving a faint outline of the array, and a see-through roof part makes them see-through. An
indicator may sit at an array (see Indicators).

## Structures and pavilions

- **Footprints** of structures, pavilions, pits and ground areas: a `polygon` at any angle, or the
  rectangle `x0`/`x1`/`y0`/`y1` as shorthand, one or the other
 .
- `structures`: simple prisms outside the house (steps, landing, pillars, the lintel above the
  entrance, canopies, a balcony's slab), a footprint with `bottom`/`top`; railings and parapets
  are `railings`. Every entry has a
  required, unique `id`, so a view can make it transparent or hide it (`structures` in the view,
  like `walls` and `roofs`); `transparent` in house.json is its automatic state.
- `pavilions`: light glass extensions such as a conservatory, hinted by their outer edges, with a
  sloped top falling towards the compass side `low` (`top` at the footprint's point furthest from
  it, `lowTop` at the one furthest towards it); edges facing away from `low` (the house side) get
  no glass and no posts. `roomId` lets one show a room's values (light).

## Terrain

`ground`: `floor`, `level` or `heights` (relative to that floor), `outline`, `pits`, `areas`.

- **The terrain's height** (`src/model/terrain.ts`, the one function every user asks): either
  `level`, flat everywhere, or `heights`, spot heights `[x, y, level]` as a site plan gives them
  (Höhenkoten); one of the two, not both. Between spot heights the terrain is linear within the
  triangles of their Delaunay triangulation; outside their hull it continues from the nearest
  point of the hull with the gradient of the triangle there, blended round a hull corner so it
  has no step (three points: one plane everywhere). Points on one line make a profile, level
  across it. Spot heights closer than 0.1 m are refused. *Why spot heights:* they are what a
  site plan prints, so the skill copies rather than invents; two or three give a plane, more any
  shape. A single tilted plane could not flatten out at the house, and terraces alone read as
  stairs. *Pitfall:* Delaunay joins the nearest points, so sparse points (plot corners only) can
  span one long triangle across the house; the plan from `house:check` draws the triangles.
- **Platforms**: an area with `level` lies flat at it (a terrace, a drive, a seating area cut into
  the slope); the last area listed wins where they overlap, also for the level, so a path drawn on
  a platform stays on it. The scene closes the gap to the terrain round it with an upright edge in
  the pits' grey (retaining wall or fill), along the platform's own edges, wherever the terrain
  beyond lies higher or lower.
- **The scene** (`src/scene/builders/ground.ts`): a grid with every corner as a grid line; on a
  slope the cells are at most 1 m wide and every corner takes the terrain's height. A cell a
  slanted edge crosses (a plot edge, a turned outbuilding, an area at an angle) is cut with
  Clipper2: the terrain region in it, split by platforms and areas, triangulated. Pits
  reach up to the terrain at each corner (walls, railing; stairs start at their top end's
  middle). Lanterns, ground spots and the weather station's mast stand on the terrain at their
  position; light pools on the terrain tilt with it. Structures and pavilions keep their authored
  `bottom`; the model places them.
- Validation: a pit's `bottom` below the terrain at every corner, a lantern's head and the
  weather station's cups above it where they stand. `house:check` prints the terrain at the
  outline's corners, the spot heights and each platform with the terrain's range round it, and
  draws spot heights and triangles in the ground floor's plan.

- `outline` (required): the terrain's corners in plan coordinates, in order, at any angle (a
  plot ending at a street that does not run north-south). The terrain is exactly that; nothing is
  derived from other parts, so paved areas or a garage do not move the edge and the plot can end at
  the street. Validation: at least three corners, no zero-length or crossing edge, a real area.
  Areas reaching past it are cut off.
- `pits`: excavated areas (in front of basement windows) with vertical walls down to `bottom`,
  optional `railing` and `stairs`; sides along the house stay open.
- `areas`: patches with their own colour (terrace, gravel strip, paving), following the terrain or,
  with `level`, platforms (above); nothing is drawn below the house.

## Tags

Tags group elements so a view can leave groups out or switch them live. Every element has the tags
its kind implies (`src/model/tags.ts`); house.json adds more with an optional `"tags": [...]` on
areas, roof parts, structures, pavilions, pits, lights, furniture, stairs, railings, the weather
station and indicators at a point or PV array:

| Element | Implied tags |
|---|---|
| area of the house | none (a view shows or leaves out rooms with `rooms` and `floors`) |
| outbuilding (`outbuilding: true`) | `outbuilding`; without walls of its own also `outside` |
| outdoor area (`height: 0`) | `outside` |
| roof part | `roof`; over outbuildings only, their tags too (the garage roof goes with the garage) |
| structure | `structure` |
| pavilion | `pavilion`, `outside` |
| the terrain (`ground`, with its areas) | `ground`, `outside` |
| pit (`ground.pits`) | `ground`, `outside` (a view hiding an added tag leaves the pit out and the terrain closes over it; a page's switch hides the pit but leaves its opening) |
| light | `light`; the tags of the area it lies in; `outside` when outside the building |
| furniture | `furniture`; the tags of the area it stands in; `outside` when outside the building |
| stair | `stairs`; the tags of the area it starts in |
| railing | `railing`; the tags of the area it borders; `outside` when it borders none |
| weather station | `weatherStation`, `outside`; `roof` on a roof |
| indicator at a point | the area's tags, `outside` outside the building; no implied tag of its own |

- Openings go with their area; roof windows, gables, dormers and PV arrays with their roof part.
- Implied tags are fixed English ids (the view editor translates them); added tags are names as the
  user calls them (`"Carport"`, `"Garten"`), like room names, and may repeat an implied one
  (`"outside"` on a freestanding garden wall, which as a structure is not outside by its kind).
- **Tags are derived where the model already says it, authored only where it cannot**: authored
  tags duplicating derivable ones would drift (a roof part tagged with the wrong storey).
- **Every component carries one specific tag of its own** besides its implied generic one (decided
  by the owner): a piece of furniture, a lamp or row of lamps, a stair, a railing, a structure, a
  pavilion, a pit, an outbuilding, a roof part other than the house's own, named as the user calls
  it ("Sofa Wohnzimmer", "Treppe Diele"); parts of one thing share it ("Küchenzeile", "Carport").
  So a view or a page's switch can leave out all furniture (`furniture`) or just the sofa. A
  name cannot be derived, so this follows the rule above. Rooms, the house's own roof parts, the
  terrain and openings get none: views switch them by other means.
- **Several tags per element**: the carport's roof part carries `roof`, `outside` and `Carport`;
  hiding any of them hides it. Groups overlap naturally ("everything outside" and "the carport"),
  so one tag per element would force a hierarchy that does not exist.

## Views: storeys, tags and labels

Views live in `views/<id>.json` (`${CLAUDE_PLUGIN_ROOT}/skills/house-model/schema/view.schema.json`); the parts that depend on the house
format are these.

- **Leaving storeys and groups out**: `"floors": ["eg"]`, `"hide": ["outside", "roof"]`. `floors`
  is absolute; a hidden tag only changes the *automatic* state, so a roof part's or structure's own
  setting in the view beats it (`"hide": ["roof"], "roofs": { "wing": "solid" }`). Left-out
  elements are not built and do not count for the automatic framing; an element hidden by its own
  setting still counts, so hiding one pillar does not move the picture. Geometry uses the whole
  house: a ground floor shown alone keeps its wall heights, walls under a hidden roof part still
  follow its slope. Active rooms on storeys left out stay in `rooms` with their label settings.
- **Live switches**: `"toggles": ["roof", "outside"]` gives those groups a switch on the page (not
  saved). It only hides the objects carrying the tag, with no rebuild: instant on a tablet, the
  camera stays put, and nothing else adapts (gable walls stay when the roof is switched off). A
  tag in `hide` gets no switch.
- **Labels** per area id, relative so they follow room edits:

  ```json
  "labels": {
    "kueche": { "name": { "shift": -0.4 }, "temperature": { "height": 0.2 }, "indicators": { "offset": [-0.3, 0], "height": 2 } },
    "flur-og": { "name": "hidden" }
  }
  ```

  Offsets are `[east, north]` meters from the area's centroid, fixed in the room whatever the
  camera does; a label without offset floats over the visible floor. A room name pinned to a wall
  (`"name": { "wall", "shift", "height" }`) has its `shift` along the wall in the order of the wall
  id's end points. `"name": { "floor": true, "offset": [east, north] }` lays it on the floor along
  the room edge closest to left-right in the image, reading left to right. Without a name setting,
  views looking down at least about 20° put the name on the floor where the camera sees the floor
  more squarely than the wall it would take. The settings sit in the views because with a free
  camera "the view in which the room is open" is no longer one direction.
- **Indicators at a point** are nudged per view: `"indicators": { "<id>": { "offset": [east, north],
  "height": h } }`, the offset from the point; moving the point in house.json moves it in every
  view.
- Settings that have no effect only warn (unknown storey, area or tag, no active room left, a
  switch for a tag that is left out): editing the house never stops a view from loading.

## Light sources

`lights`: a list with `kind` `pendant`, `ceilingSpot`, `lantern`, `groundSpot`, `wallLight` or
`strip`, each with its own live light.

- Placed by **plan points**: `position` (a strip: `from`, `to`). A `wallLight`'s point lies on the
  inner face of its wall (an edge of an area polygon, like an opening's ends), and `side`
  (`inside`/`outside`) says which face it hangs on; the outer face only of an exterior wall.
- A pendant is a shade (`shape` `bell`, the default) or a slim closed tube (`shape` `tube`: `size`
  its diameter, `length` its length, default 0.3; light only from its lower end in a narrow beam).
  A tube's top (`height` + `length`) lies below the ceiling.
- Heights above the storey's floor: a pendant's lower edge, a lantern's head (its post reaches down
  to the terrain), a wall light's center, a strip's line (`height` at `from`; an optional `toHeight`
  at `to` makes it rise, e.g. under a sloped ceiling or along stairs, or stand upright with `from`
  and `to` the same point). A ceiling spot sits in its area's ceiling; outside every area (under a
  canopy) it needs `height`. A ground spot has no height: it sits in the floor of the area it lies
  in, otherwise in the terrain, and shines up. Validation keeps inside lights between the area's
  floor and ceiling (an outbuilding's `level` counts).
- `data` with the room's light roles (`light`, `lightColor` also as a list, `lightLevel`). Without
  data a light follows its room's light: `room`, else the room of the area it lies in. A light on a
  facade or outside every area has no room; with no data either it stays dark, which only warns,
  since the skill places lights before it maps them to Loxone.
- Derived in one place (`src/model/lights.ts`): the area, the wall's edge, whether it is outside
  (no area, an outdoor area, or a facade), the room it follows.
- Optional per light: `color` (the body when off), `size`, `range`, `beamAngle`; the scene supplies
  defaults per kind.
- Only the house-model skill places lights; the app's editors do not edit them.

## Furniture

`furniture`: boxes, drawn as lines with a faint fill, whose edges can carry light strips. Two forms,
both with `bottom` and `top` above the storey's floor:

- against a wall, also a slanted one: `from`, `to` (the back edge as plan points, `from` its left
  end seen standing in front of the piece) and `depth` towards the front, which lies to the right
  of `from`→`to`;
- along the plan axes: `x0`/`x1`/`y0`/`y1` and `front` (`north`/`south`/`east`/`west`).

- **A front, so edges keep their names when the piece turns**: left and right are as seen from the
  front, the same in both forms (`src/model/furniture.ts` brings both to one frame).
- `color` (lines and fill) and `opacity` (0 lines only, 1 solid; default faint).
- **Room**: from the middle of the footprint, as for lights; `room` overrides it. The piece shows
  when a view opens that room. Validation keeps it between its area's floor and ceiling; outside
  every area (a garden bench) no height applies.
- **Strips on edges**: a `strip` light with `"on": { "furniture": "<id>", "edge": "<edge>" }` instead
  of `floor`, `from`, `to`, `height`, `toHeight` runs along that edge's whole length; loading fills
  in its line from the piece, so it moves with the piece. It goes with the piece's tags and is
  built only where the piece is. Light falls down from a bottom edge, rises from a top edge and
  spreads outwards from an upright one.

## Indicators

`indicators`: icons that appear depending on datapoints.
One thing is one indicator; its states are `rules`, checked in order, the first matching one
applies (a rule without `when` always matches, e.g. "no mail" last); if none matches, the indicator
is hidden.

- An indicator names its **kind** (`"kind": "music"`, `^[a-z][a-z0-9-]{0,39}$`); a rule may switch
  to another kind. The icon per kind is a global setting (`global.json`, `${CLAUDE_PLUGIN_ROOT}/skills/house-model/schema/global.schema.json`,
  dialog "Globale Einstellungen …"), chosen from Fluent Emoji (colour) that look the same on every
  platform, or from the other icon sets as `"set:name"` with a colour for one-colour ones (a bare
  name is Fluent). A kind instead of an emoji in house.json: a text emoji looks different on every
  platform, and the icon is chosen once per kind for the whole house.
- **Where it shows**, exactly one of:
  - `room`: in that room's row of indicators, placed by each view (`labels` in the view);
  - `position` (`floor`, `x`, `y`, `height` of the icon's middle, default 1.5): a point of its own
    for something that belongs to a place, e.g. the mailbox by the front door
    (`src/model/indicators.ts`). It shows with the room of the area the point lies in; outside the
    building it is `outside`. Validation keeps it between a walled area's floor and ceiling.
  - `pvArray`: above that PV array's centre, shown while the view draws its roof part, with that
    part's tags.
  At a point or array it takes `tags` of its own; views can nudge it (`indicators` in the view).
- **With a value**: `"value": { "datapoint": … }` writes that datapoint's value (number or text)
  beside the icon. A rule still decides whether it shows (`"rules": [{}]` always); a kind whose
  icon global.json hides (`"hidden"`) shows the value alone. How it is written belongs to the
  datapoint, not to the indicator: `datapoints.json` has `format` (Loxone's printf-like text,
  `"%.1f°"`), which takes precedence over `unit` and `as.decimals`, written with the app language's
  decimal separator (`src/model/valueFormat.ts`).

## House-wide data and the weather station

- `data` at the top of house.json holds house-wide roles: `outsideBrightness` (lux, e.g. the
  weather station's brightness sensor; the scene's daylight follows it, so by night facades, roof
  and terrain darken while rooms keep their look; without it the scene stays in daylight), and the
  weather: `outsideTemperature`, `feltTemperature`, `humidity`, `windSpeed` (km/h),
  `windDirection` (degrees, where the wind comes from) as numbers; `windWarning`, `rain`,
  `sunshine` as switches; `weatherType` as the Loxone weather service's number (codes 1–29; their
  meaning lives in `src/model/weatherType.ts`, not in a Miniserver's texts). All optional: a
  screen's sky uses what is there.
- `weatherStation`: `{ floor, position, height }` plus optional `tags`; the cups' height above the
  storey's floor, the mast reaching down to the roof or terrain below. Validation keeps the cups
  above both.
- Pages (`devices.json`, not house.json) get `weather: true` for the sky behind their
  views.

## Objects (planned)

`objects` is sketched in the schema but not built: a 3D model or a basic shape with variants by
condition (a floor lamp that looks different when on). Simple furniture covers the common case.

## Not in house.json

- **Datapoints** (`datapoints.json`, `${CLAUDE_PLUGIN_ROOT}/skills/house-model/schema/datapoints.schema.json`): the mapping from roles to
  Loxone controls and states, with conversions (`as`) and display `format`; also the `cameras`.
- **Devices and pages** (`devices.json`, `${CLAUDE_PLUGIN_ROOT}/skills/house-model/schema/devices.schema.json`): which views, data panels,
  camera pictures and Loxone pages each screen shows; cameras are no part of the house
  model.
- **Global settings** (`global.json`): language, icon per indicator kind, the Loxone page's scale.

## Later

- Optional floor `elevation` for half-storey houses (default: sum of the floor heights below);
  split levels within a storey are areas with `level`.
- Mansard roofs; hipped and flat dormers if asked.
- 3D models with variants (`objects`).
