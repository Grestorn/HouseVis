# House model format (house.json)

Format **version 5**. Steps A–F made format 2 (merged into `main` 2026-11-05); G (format 3)
moved the label settings into the views; H (format 4) gave structures ids; I (format 5) gave
indicators a kind instead of an emoji. This document later becomes
the reference for the house-model skill.

## Principles

- **Versioned.** `"version": 6`. HouseVis reads only the current version and refuses any other with
  a clear message (`src/model/version.ts`). Until the first public release there are **no
  migrations**: only the owner's `config/` exists, so a breaking change converts it (and the test
  fixtures) once and keeps no upgrade code. Formats 1→2 and 2→3 were converted by migrations that
  were removed afterwards (2026-11-05; they are in the git history up to commit `254df24`). Once
  other people have files of their own, migrations come back: plain JSON, one step per version.
- **Bump the version only for breaking changes**, i.e. when an old file would be read wrongly or
  rejected. New optional fields and new `kind`s are additive and need no new version.
- **Authored vs. internal.** `HouseFile` is what people (and the skill) write; `normalizeHouse()`
  turns it into the internal `House` once, resolving shorthands and defaults. Geometry and scene
  code only see the normalized model.
- **Element families are discriminated by `kind`** (roof parts, openings, …), so new kinds can be
  added without touching the existing ones.
- **Stable ids** on everything views, the editor or datapoints refer to.
- Coordinates in meters, plan coordinates `[x, y]` with x to the east and y to the north; heights
  relative to the floor of the element's `floor`. Rooms are interior dimensions; walls result from
  the gaps between rooms.

## A — versioning and space ids (done)

```json
{ "version": 2, "floors": [ … ],
  "spaces": [ { "id": "kueche", "roomId": "kueche", "floor": "eg", "polygon": [[0.37, 6.32], …] } ] }
```

`spaces[].id` is required and unique; format 1 files get the room id (plus floor/counter if a room
has several areas).

## B — openings by plan coordinates (done)

Today: `"wall": "west", "at": 0.36, "from": 4.9, "to": 5.9` — only works for walls facing exactly
north, south, east or west. New:

```json
{ "id": "kueche-west-1", "kind": "window", "floor": "eg",
  "start": [0.36, 4.9], "end": [0.36, 5.9], "sill": 0.85, "top": 2.2, "operation": "turn" }
```

- `start`/`end` lie on the inner face of the wall (= an edge of the room polygon); their order does
  not matter. The owning room and the facing direction are derived geometrically.
  Tolerance 5 mm; the validation reports openings that lie on no room edge.
- `kind` (`window`/`door`) and `id` are required; format 1 files get `opening-<n>` where the id was
  missing.
- `side` (`left`/`right` seen from inside), `operation`, `leaves`, `sidelight`, `fixedBelow`,
  `color`, `data`, `shades` stay as they are; shade `from`/`to` are distances from `start`.

## C — walls at any angle (done)

- Room polygons may have edges in any direction; the validation reports overlapping rooms and
  outlines that cross themselves.
- The floor slab with the interior walls standing in it is built from polygons (`src/model/slab.ts`,
  Clipper2): the rooms are grown by half the maximum wall gap (0.7 m), merged and shrunk back, so
  gaps between rooms close at any angle; slab minus rooms is where the interior walls stand.
- **Wall ids** (for views and the editor, not written in house.json):
  `<spaceId>@<x1>,<y1>-<x2>,<y2>`, endpoints rounded to centimeters and sorted, e.g.
  `kueche@0.36,4.29-0.36,8.05`. Moving another corner of the room keeps the id; a view override
  whose wall no longer exists becomes a validation warning.
- Faces of the outer outline that belong to no room wall (interior wall ends, steps in the facade,
  see F) get the same kind of id with the floor in place of the area: `eg@1,4.21-1,4.34`. They
  cannot clash with a room wall, since they exist only where no room face lies.
  `"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --house-check --walls` lists every id a view can set.

## D — roof parts (done)

Format 1 had one gable roof with its ridge north-south. Now `roofs` is a list of parts:

```json
"roofs": [
  { "id": "main", "kind": "gable", "floor": "og",
    "footprint": { "x0": -0.6, "x1": 11.6, "y0": -0.7, "y1": 10.2 },
    "ridge": "north-south", "eaveHeight": 1.68, "ridgeHeight": 4.49 }
]
```

| kind | extra fields | shape |
|---|---|---|
| `gable` | `ridge` (`north-south`/`east-west`), `eaveHeight`, `ridgeHeight` | two slopes, gable walls at the ends |
| `hip` | like `gable` | four slopes with equal pitch; the ridge is shorter than the footprint by its width (so the footprint must be at least as long along the ridge as it is wide) |
| `shed` | `low` (side), `eaveHeight` (low side), `ridgeHeight` (high side) | one slope |
| `flat` | `height` | flat roof |

- `footprint` includes the overhang. Optional `rotation` (degrees, counter-clockwise around the
  footprint center) for houses that are not aligned with the axes; `ridge` and `low` refer to the
  footprint before rotation. Heights are relative to the part's `floor`.
- Optional `color` (`"#rrggbb"`) for the roof covering seen from outside, per part (e.g.
  anthracite tiles on the house, a grey flat roof on the garage); default dark red. The seams are
  drawn 0.6× darker in sRGB, the underside keeps its light tone.
- **Geometry** (`src/model/roof.ts`): every part is broken down into flat faces (plan polygon +
  plane). Within a part the surface is the *lowest* of its planes; where parts overlap, the
  *higher* part counts, so a wing running into the main roof forms the valleys by itself.
  `roofHeightAt(point, floor)` considers the parts of that floor (Infinity elsewhere); wall tops,
  divider tops, gable and attic walls and sloped ceilings all derive from it. Wall profiles bend
  only on "crease" lines (planes meeting each other or the ceiling height, footprint outlines),
  so they are exact at any wall angle.
- Walls of a floor reach up to the roof instead of stopping under a slab only if the floor has a
  roof part and no floor above has rooms (a porch roof on the ground floor does not count).
- Gable walls are closed on every footprint side that faces the camera, at the outermost wall line
  of the rooms below the part; where the roof comes down below the rooms (eaves, hips) nothing
  is drawn.
- **Roof windows** refer to a part: `{ "id": …, "roof": "main", "center": [x, y], "width": 1.14,
  "depth": 1.42 }` — `width` along the eave, `depth` across it in the plan. The slope under
  `center` holds the window; the validation reports windows that reach into another slope. Shade
  `from`/`to` run along the width from the left edge seen from outside looking up the slope.
- Known limits: overlapping parts are drawn completely (the hidden piece lies under the higher
  roof); a wing whose footprint reaches beyond the rooms shows its open end.
- Later, additive: polygon footprints, dormers, mansard roofs.

## F — exterior walls with thickness (done)

Rooms stay interior dimensions; the exterior walls lie outside them. Additive, so no version bump:

```json
"floors": [ { "id": "eg", "name": "EG", "height": 2.9, "exteriorWall": 0.365 } ]
```

- `floors[].exteriorWall`: structural thickness in meters, default 0 (the facade then lies on the
  room edges, as in format 1). Outbuildings have no thickness.
- **Geometry** (`src/model/shell.ts`): the storey's slab (rooms plus interior wall gaps) is grown
  by the thickness with mitred corners; the strip gained is the ring of exterior walls. Each
  exterior wall of a room gets an outer face (its edge moved out, mitred at the room's own
  exterior corners). Stretches of the outer outline no room face covers — interior wall ends,
  steps where an edge is only partly exterior — become filler faces; one with a room right behind
  it looks like that room's wall (glass when cut open, its facade color otherwise).
- **Scene**: facades and the glass walls of cut-open rooms stand at the outer face; the back walls
  stay on the room edge. Openings get reveals through the wall; windows sit 12 cm behind the outer
  face. Gable walls close at the outer face; the terrain is cut out down to it.
- **Consequence for authored coordinates**: everything placed against the house (terrace, gravel
  strip, pits, conservatory, steps) must touch the outer face, not the room edge. Roof footprints
  measure the overhang from the outer face.
- With thickness 0 the scene renders as before (pixel diff of `smoke:live`, only the terrain line
  in front of interior wall ends changed).

## G — label settings in the views (format 3)

`hideName`, `nameShift`, `temperatureHeight`, `temperatureOffset`, `indicatorOffset` and
`indicatorHeight` left the areas of house.json. They were offsets relative to "the view in which
the room is cut open"; once a view has a free camera, that is no longer one direction, so every
view keeps its own in `labels`, keyed by area id:

```json
"labels": {
  "kueche": { "name": { "shift": -0.4 }, "temperature": { "height": 0.2 }, "indicators": { "offset": [-0.3, 0], "height": 2 } },
  "flur-og": { "name": "hidden" }
}
```

- Offsets stay relative (decided 2026-11-05), so they follow room edits. Since the editor
  (decided 2026-11-05) they are `[east, north]` meters from the area's centroid, fixed in the room
  whatever the camera does; before they were `[right, back]` from the center of the visible floor,
  which moved with the camera. `config/` and the fixtures were converted once for the same
  positions. A label without offset still floats over the visible floor. A pinned room name's
  `shift` runs along its wall (in the order of the wall id's end points), not to the right in the
  image.
- The conversion copied each area's settings into every view that cuts the area open, exactly where
  they took effect before; the picture stayed pixel-identical.

## H — structure ids (format 4)

Every entry in `structures` has a required, unique `id` (`^[a-z0-9][a-z0-9-]{0,39}$`), so a view
can make a pillar, lintel, canopy or parapet transparent or hide it (`structures` in the view,
like `walls` and `roofs`); `transparent` in house.json is the part's automatic state. The owner's
`config/` and the test fixture were converted once, ids from the names (e.g. "Pfeiler Eingang" →
`pfeiler-eingang`).

## I — indicator kinds (format 5)

An indicator names its **kind** (`"kind": "music"`, `^[a-z][a-z0-9-]{0,39}$`) instead of an emoji
`"icon"`; a rule may switch to another kind (`"kind"` in the rule) instead of another icon. The
icon per kind is a global setting (`config/global.json`, `${CLAUDE_PLUGIN_ROOT}/skills/house-model/schema/global.schema.json`, dialog
"Globale Einstellungen …"), chosen from about a thousand Fluent Emoji (colour) that look the same
on every platform (`src/model/indicatorIcons.ts` with the hand-picked suggestions, the full list
generated into `src/model/generated/indicatorIconNames.ts`, files in `src/ui/indicatorIcons/`; the
picker searches their Unicode CLDR names in German and English, `tools/indicator-icons.mjs`). The rule's
`color` is gone: it tinted a text emoji and does nothing on an image. "heating", "cooling" and the
heat controller's modes ("comfort", "eco", "paused", "protection") are kinds of the rooms' own.
The owner's `config/`, `plans/<project>/config` and the test fixture were converted once
(`"icon": "🎵"` → `"kind": "music"`).

Additive in the same step: the room roles `climateMode` (IRoomControllerV2 `activeMode`) and
`climateOverride` (`overrideReason`); the label shows the mode as an icon left of the temperature,
"paused" for an open window (override 2) or the pause timer (15), see `src/model/climateMode.ts`.

## J — tags, and views that leave storeys or tags out (additive, format 5)

A view can leave out whole storeys or groups of elements, e.g. show the ground floor alone, or
the house without terrain, pavilion and carport (`src/model/tags.ts`, roadmap "Storeys and tags in
views"):

```json
"floors": ["eg"],
"hide": ["outside", "roof"]
```

- **Tags** group elements. Every element has the tags its kind implies; house.json adds more with
  an optional `"tags": [...]` on areas, roof parts, structures, pavilions and lights:

  | Element | Implied tags |
  |---|---|
  | area of the house | none (a view shows or leaves out rooms with `rooms` and `floors`) |
  | outbuilding (`outbuilding: true`) | `outbuilding`; without walls of its own also `outside` |
  | outdoor area (`height: 0`) | `outside` |
  | roof part | `roof`; over outbuildings only, their tags too (the garage roof goes with the garage) |
  | structure | `structure` |
  | pavilion | `pavilion`, `outside` |
  | the terrain (`ground`, with its areas and pits) | `ground`, `outside` |
  | light (section L) | `light`; the tags of the area it lies in; `outside` when outside the building |

  Openings go with their area, roof windows and gables with their roof part. Implied tags are
  fixed English ids (the view editor translates them); added tags are names as the user calls them
  (`"Carport"`, `"Garten"`), like room names, and may repeat an implied one (`"outside"` on a
  freestanding garden wall, which as a structure is not outside by its kind).
- **Several tags per element** (decided 2026-11-06): the carport's roof part carries `roof`,
  `outside` and `Carport`; hiding any of them hides it. Groups overlap naturally ("everything
  outside" and "the carport"), so one tag per element would force a hierarchy that does not exist.
- **What wins.** `floors` is absolute: nothing of another storey is drawn, whatever its own
  setting. A hidden tag only changes the *automatic* state, so a roof part's or structure's own
  setting in the view (`roofs`, `structures`) beats it: "all roofs hidden except the wing" is
  `"hide": ["roof"], "roofs": { "wing": "solid" }`.
- **Left out, not hidden.** Elements left out by `floors` or a tag are not built at all and do not
  count for the automatic framing, so the picture fits what is left. An element hidden by its own
  setting still counts, as before, so hiding one pillar does not move the picture.
- **Geometry uses the whole house.** Wall heights still know about the storeys above (a ground
  floor shown alone keeps its walls at storey height minus the slab), and walls under a hidden
  roof part still follow its slope.
- Active rooms on storeys left out stay in `rooms` (with their label settings): showing the
  storey again brings them back as they were. The view editor lists only the rooms of the shown
  storeys.
- **Live switches**: `"toggles": ["roof", "outside"]` gives those groups a switch on the page; the
  viewer turns them off and on there (not saved). It only hides the objects carrying the tag, with
  no rebuild: instant on the tablet, the camera stays put, and nothing else adapts (gable walls
  stay when the roof is switched off). A tag in `hide` gets no switch.
- Validation only warns (unknown storey, a tag no element carries, no active room left, a switch
  for a tag that is left out), like the overrides: editing the house never stops a view from
  loading.

## K — outbuildings as buildings of their own (additive, format 5)

Until 2026-11-06 an outbuilding area (`outbuilding: true`) was drawn without walls, so it would
not hide the house, and could never be active. With views able to make walls transparent, leave
groups out and switch them live, that reason is gone, and a garage with its own door, presence
and light should be a room (the owner's request):

- `exteriorWall` on an outbuilding area: its walls, outside the polygon like the house's (the
  polygon is the inside). It gets its own shell (`outbuildingShell`: slab, wall ring, outline),
  facades when inactive, room walls when active, walls settable per view and clickable in the
  editor. It is still no wall neighbour of the house (nor of other outbuildings). Without
  `exteriorWall` an outbuilding stays without walls (an open shelter): it can be active, showing
  its floor and labels, but no name (that needs a wall).
- `level` on an outbuilding area: its floor relative to the storey's (a garage at −0.32).
  `height` counts from there; the storey's roof parts still clip the walls (heights shifted by
  the level, `BuildContext.spaceTop`); openings keep counting from the storey's floor, so a gate
  at the garage floor has `"sill": -0.32`. Both fields only on outbuildings (validation).
- The terrain leaves out the footprint of outbuildings with walls, like the house's, so a garage
  floor below the terrain shows.
- An outbuilding with walls is a building, not part of the grounds: it no longer carries the
  implied tag `outside` (only `outbuilding`), so "everything outside" no longer hides the garage
  (decided 2026-11-06, after the owner's EG view hid the garage they wanted to see). One without
  walls (an open shelter) still is `outside`.
- Additive: an outbuilding without the new fields behaves as before, except that it can be
  active. `config/` and `${CLAUDE_PLUGIN_ROOT}/skills/house-model/example/` were converted once: the garage areas shrank to their
  inside, got `exteriorWall`, `level` and the full height, their gates moved onto the inner edge,
  and the wall boxes (`structures`) that stood in for the walls went.

## L — light sources (additive, format 5)

Asked 2026-11-07: lanterns along the walkway, wall lights
on the facade, lit furniture inside, each with its own live light.

- `lights`: a list with `kind` `pendant`, `ceilingSpot`, `lantern`, `wallLight` or `strip`. Placed
  by **plan points**, like openings, never by wall id (wall ids are derived from coordinates and
  change with the room): `position` (a strip: `from`, `to`). A `wallLight`'s point lies on the
  inner face of its wall (an edge of a room polygon, like an opening's ends), and `side`
  (`inside`/`outside`) says which face it hangs on; the outer face only of an exterior wall.
- Heights above the storey's floor, like everywhere: a pendant's lower edge, a lantern's head (its
  post reaches down to the terrain), a wall light's center, a strip's line. A ceiling spot sits in
  its area's ceiling; outside every area (under a canopy) it needs `height`. Validation keeps
  inside lights between the area's floor and ceiling (an outbuilding's `level` counts).
- `data` with the room's light roles (`light`, `lightColor` also as a list, `lightLevel`; same value
  types). Without data a light follows its room's light: `room`, else the room of the area it lies
  in. A light on a facade or outside every area has no room; with no data either it stays dark,
  which only warns, since the skill places lights before it maps them to Loxone.
- Derived in one place (`src/model/lights.ts`): the area, the wall's edge, whether it is outside
  (no area, an outdoor area, or a facade), the room it follows. Tags: section J.
- Optional per light: `color` (the body when off), `size`, `range`, `beamAngle`; the scene
  supplies defaults per kind.
- Only the house-model skill places lights; the app's editors do not edit them (decided
  2026-11-07).
- House-wide datapoints: `data` at the top of house.json, so far `outsideBrightness` (lux, e.g.
  the weather station's brightness sensor). The scene's daylight follows it: by night facades,
  roof and terrain darken while rooms keep their look, so lit rooms and light sources stand out.
  Without it the scene keeps its daylight look.
- Additive: no version bump, no conversion of `config/`.

## M — weather (additive, format 5)

Asked 2026-11-07.

- House-wide `data` gains the weather roles: `outsideTemperature`, `feltTemperature`, `humidity`,
  `windSpeed` (km/h), `windDirection` (degrees, where the wind comes from) as numbers;
  `windWarning`, `rain`, `sunshine` as switches; `weatherType` as the Loxone weather service's
  number (codes 1–29; their meaning lives in `src/model/weatherType.ts`, not in a Miniserver's
  texts). All optional: a screen's sky uses what is there.
- `weatherStation`: `{ floor, position, height }` plus optional `tags`; the cups' height above the
  storey's floor, the mast reaching down to the roof or terrain below. Validation keeps the cups
  above both. Tags: implied `weatherStation`, `outside`, and `roof` on a roof (section J).
- Pages (`devices.json`, not house.json) get `weather: true` for the sky behind their views.
- Additive: no version bump, no conversion of `config/`.

## N — the terrain's outline (format 6)

Asked 2026-11-07 after the assistant fought the terrain on the owner's house: the edge was a box
around the house, the garage, the areas, pits and pavilions plus a `margin` per side (default 2 m),
so every paved area or the garage moved the edge, and the plot could not end at the street.

- `ground.outline` (required) replaces `margin`: the terrain's corners in plan coordinates, in
  order, every edge east-west or north-south (right angles: rectangles, L-shapes, cut-out corners).
  The terrain is exactly that; nothing is derived from other parts. Validation: at least four
  corners, no slanted or zero-length edge, no crossing edges, a real area (`outlineErrors`).
- The scene's grid takes the outline's corners as grid lines and drops cells outside it; the edge
  line follows the outline. Areas reaching past it are cut off.
- Converted once: `config/` and `${CLAUDE_PLUGIN_ROOT}/skills/house-model/example/` got the rectangle the format-5 code drew for them
  (computed with that code), so nothing changed visually. An export file from before (format 5)
  is refused on Android; export the device again.
- Slanted plot edges would need the terrain clipped against the outline; left for later.

## Later, additive (no version bump needed)

- Polygon footprints for structures, ground areas, pits and pavilions (today rectangles
  `x0/x1/y0/y1`, which stay as shorthand).
- Optional floor `elevation` for split levels (default: sum of the floor heights below).
- Furniture/objects (`objects`, already in the schema), several light channels per room.
