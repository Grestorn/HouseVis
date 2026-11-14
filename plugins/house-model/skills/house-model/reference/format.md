# From plans to house.json

The authoritative definition is `${CLAUDE_PLUGIN_ROOT}/skills/house-model/schema/house.schema.json` (every field has a description) and the
reasoning is in `${CLAUDE_PLUGIN_ROOT}/skills/house-model/reference/model-format.md`. This file explains how to get the values **out of the
plans**. Read both schema and this file before block 1.

## Skeleton

`<model folder>/house.json`:

```json
{
  "version": 6,
  "_comment": "Built by the house-model skill from plans/<project>/; sources and estimates in model-notes.md.",
  "floors": [{ "id": "eg", "name": "Erdgeschoss", "height": 2.9, "exteriorWall": 0.365 }],
  "rooms": [], "spaces": [], "openings": [], "roofWindows": [], "roofs": []
}
```

- Comments are only allowed as top-level keys starting with `_`; nested objects reject unknown
  fields. Record sources and estimates in `model-notes.md`, not in the JSON.
- Ids: `^[a-z0-9][a-z0-9-]{0,39}$`. Derive them from the room names (`küche` → `kueche`,
  `Wohnen/Essen` → `wohnen-essen`). Names (`name`) are written as the user calls the rooms, in
  their language, as short as possible (they are drawn on the wall).
- Units: meters, two or three decimals. Plans in cm or with superscript half-centimeters
  (`1.01⁵` = 1.015 m) are converted.

## Coordinates

- x to the east, y to the north, z up; plan coordinates `[x, y]`.
- **Origin: outer south-west corner of the main building's shell** (exterior face of the
  structural wall). The exterior dimension chains start there, so every coordinate is a running
  sum of printed dimensions.
- **Rooms are interior dimensions** (structural, without plaster: German "Rohbaumaß"). The
  exterior walls lie outside the room polygons: `floors[].exteriorWall` is their structural
  thickness (e.g. `0.365`), read from the dimension chains (`36.5 | 4.01 | …`). So a room behind a
  36.5 cm exterior wall starts at x = 0.365, and the scene draws the facade at x = 0. Basement
  walls often differ from the storeys above (concrete 0.30, insulation not counted); if one
  storey's walls differ on different sides, take the dominant value and note the others.
- **Everything placed against the house touches the outer face**, not the room edge: terrace,
  gravel strips, pits, conservatory, steps, and the roof overhang is measured from it.
- **Walls come from the gaps between rooms**: a gap narrower than 0.7 m between two areas of the
  same storey becomes an interior wall. Therefore **every** area becomes a space — hallways,
  stairs, storage rooms, technical rooms — even if Loxone does not know it; otherwise a hole
  appears. Shafts and chimneys narrower than 0.7 m simply become wall.
- **Houses not aligned with north**: if the walls deviate from north by less than about 25°, align
  the model axes with the walls and treat the nearest compass direction as north (the views are
  compass directions, and the facades should be recognisable as "south", "west"…). Note the real
  angle in `model-notes.md`. For larger deviations ask the user; then use angled polygon edges and
  roof `rotation`.

### Reading the dimension chains

German plans (and most others) have several chains per side: the outermost gives the overall
length, the next one the wall axes/projections, the innermost the openings. Inside the rooms,
chains give the room dimensions and wall thicknesses (`36.5 | 4.01 | 11.5 | 3.51 | 36.5`).

1. Pick one chain per side that runs through the rooms; build the cumulative positions.
2. **Check that every chain adds up** to the overall dimension of that side (and that opposite
   sides agree). Differences above 2 cm are reported to the user with both readings; never
   silently average.
3. Cross-check with the room areas printed in the rooms (`m²`, usually finished dimensions, so a
   little smaller than the structural area) — house:check prints the areas.
4. Where a dimension is missing, take it from a section or the storey above/below; only as a last
   resort scale it from the drawing (say so and mark it estimated).
5. Storeys above each other share the exterior walls: reuse the coordinates, don't re-derive them.
6. For positions without a printed dimension, calibrate the tiles per sheet: tile *n* starts at
   `n × (tile size − overlap)` pixels (the `plans:tiles` output lists every tile's range), so a
   pixel in a tile has an absolute sheet position. Fit `x = x0 + (px − px0) / scale` (and the same
   for y, which usually runs downwards) through two printed points of the sheet; 1:50 at 300 dpi
   is about 237 px/m, 1:100 about 118 px/m. Note the calibration per sheet in `model-notes.md`.
7. A sheet that lacks a chain in one direction (a basement plan often has none from north to
   south): calibrate it against a storey that has one, through features both sheets show at the
   same place (exterior wall faces, the stair, a shaft), and mark the positions taken that way as
   estimated.

### Storeys and heights

- `floors` from bottom to top; `height` = floor-to-floor (finished floor to finished floor,
  including the slab). Sections give it directly or as level marks: `±0.00 = OK FFB EG`, `+2.90`
  (OK FFB OG) → ground floor height 2.90.
- All heights in the model are relative to the **finished floor (FFB, "OK FFB")** of the element's
  own storey. Level marks relative to ±0.00 are converted by subtracting the storey's level.
- `spaces[].height` only where a room differs from the floor height (garage, lowered ceiling);
  `0` marks an outdoor area without walls (e.g. a terrace that is a Loxone room for its lights,
  together with `outbuilding: true`).
- Outbuildings (garage, shed) get `outbuilding: true`; they are no wall neighbours of the house
  but buildings of their own (see *Surroundings*). An open carport is no area at all: a roof part
  with posts (`structures`).
- Typical values when nothing is given (mark them estimated): storey 2.75–3.00, slab 0.20–0.25,
  basement 2.40–2.60.

## Openings (windows and exterior doors)

Interior doors are not modelled. Glazed doors to terraces or balconies are `kind: "window"` with
`sill: 0` (they behave like windows: contacts, shades); `kind: "door"` is for opaque or mostly
opaque doors (front door, side door, basement door).

- `start`/`end` lie on the room polygon edge (inner face) of the wall; within the 5 mm tolerance.
  The scene places the window in the wall's reveal by itself.
  Along the wall the position is the same as on the exterior chain (jambs are perpendicular), so
  take the opening chain values directly and put the cross coordinate on the room edge.
- Width: the structural opening width from the plan (`1.01` for a 1.00 m window is fine).
  Plans often write `width / height` at the opening (`1.01 / 1.385`) and the sill height as
  `BRH 85` (Brüstungshöhe, cm). Permit drawings usually count it from the finished floor (FFB);
  **construction drawings (Werkplanung) often from the raw floor (RFB, Rohfußboden)**, because
  the shell is built from them. Check the legend or a section where both stand side by side
  (`1,02` raw next to `87` finished); if raw, subtract the floor build-up (the difference
  FFB − RFB from the section, often 0.12–0.18 m) and note it in `model-notes.md`.
- `sill` = BRH; `top` = sill + opening height, or from the elevation / section. Typical values:
  living rooms sill 0.85–0.90, kitchens/bathrooms 1.10–1.45, tops 2.10–2.30, terrace doors sill 0.
- Ids: `<spaceId>-<facing side>-<n>`, numbered left to right as seen from outside, e.g.
  `kueche-west-1`. Keep them stable once the user has seen them.
- From elevations and photos: `operation` (`turn` default, `slide` for lift-and-slide doors,
  `fixed` for fixed glazing, `top-hung` for windows hinged at the top, `overhead` for garage gates,
  up-and-over or sectional, which open upwards into the garage), `side` (hinge side seen from inside), `leaves: 2` for
  two-sash windows, `sidelight` (fixed side panel at front doors, its width counts towards the
  opening), `fixedBelow` (fixed lower part, e.g. a balustrade glazing), `color`: on a door the
  leaf's colour, on a window the frame's (frames, sashes, fixed parts), wherever it differs from
  the default (dark brown-grey door leaves, grey window frames); check every window in the
  photos, one room may have white frames and the rest anthracite.
- `side` and `sidelight.side` are **seen from inside**. From outside, the handle is opposite the
  hinges; at a front door with a sidelight the handle is usually next to the sidelight. For
  `slide`, `side` is the direction in which the moving sash opens, seen from inside.
- `shades`: `roller` for roller shutters, `blind` for venetian blinds (visible slats, a box above
  the window). Only add them where photos, elevations ("RL", "Raff") or the user say so. Several
  shades on one opening with `from`/`to` (meters from `start`) when parts have separate ones.

## Roof

`roofs` is a list of parts (`gable`, `hip`, `shed`, `flat`); see the table in
`${CLAUDE_PLUGIN_ROOT}/skills/house-model/reference/model-format.md`. Most houses are one `gable` or `hip`; an L-shaped house is two parts that
overlap (the higher one wins, valleys form by themselves); a garage has its own `flat` or `shed`.

- `floor`: the storey whose rooms reach into the roof (the attic, or the upper floor with knee
  walls). Its heights are relative to that storey's FFB. For a cold, unused attic above a full
  storey, put the roof on the top full storey and give heights relative to it.
A part is written as the plans dimension it (format 7); nothing needs converting:

```json
{ "id": "main", "kind": "gable", "floor": "og", "outline": { "x0": 0, "x1": 11, "y0": 0, "y1": 9.5 },
  "overhang": { "south": 0.8, "north": 0.3, "east": 0.6, "west": 0.6 },
  "ridge": "east-west", "eaveHeight": 1.2, "ridgeHeight": 4.5, "color": "#3a3d40" }
```

- `outline`: the **wall line** the roof sits on, the outer faces of the walls (Außenkante), as the
  floor plan's outer dimension chain gives it. A roof over no walls (carport, canopy): its own
  edges, no overhang. It must lie on the modelled outer wall faces **within 2 cm**: only walls on
  it close up to the roof as gable walls. `house:check` marks a roof line `GABLE OPEN on <side>`
  when no exterior wall stands on that side of the outline; move the outline (or the walls) until
  it is gone.
- `overhang`: how far the roof reaches beyond the outline — eave overhang on the eave sides, verge
  overhang (Ortgang) on the gable sides; sections and elevations dimension them (typical
  0.3–0.8 m). One number when all sides are equal, else per side (`north`/`south`/`east`/`west`,
  missing sides 0): **different overhangs per side are normal** (an overhang only to the south, a
  verge flush with the wall).
- `ridge`: `north-south` or `east-west` (direction of the ridge line). A gable's slopes lie on the
  two sides along it (east-west: `south` and `north`); a hip slopes on all four; a shed on `low`.
- **`eaveHeight` is the height where the slope meets the wall line** (the outer wall face), above
  the storey's FFB — what sections give (the height at the outer wall, or the knee wall height
  plus the rafter seat). One number for every slope, or per sloped side when they differ
  (`{ "south": 1.2, "north": 2.0 }`). The slope carries on down over the overhang by itself.
- Then **either `ridgeHeight` or `pitch`**:
  - `ridgeHeight` (from the section, `First`, often an absolute level mark → subtract the
    storey's level): every slope gets the same pitch, the one at which the slopes across the ridge
    meet at that height. With different eave heights the ridge moves off the middle by itself.
    For a shed it is the height at the high side's wall line.
  - `pitch` in degrees (`DN 30°`), one or per side (`{ "south": 38, "north": 45 }`): for slopes of
    different pitch, or when the section gives the pitch but no ridge height.
  If the section gives all three (eave, ridge, pitch), write eave and ridge and check the pitch:
  house:check prints it per side (and the resulting top); differences above 1° mean a misread
  value.
- `rotation`: degrees counterclockwise around the outline's middle, for houses not aligned with
  the axes; `ridge`, `low` and the overhang sides refer to the outline before rotation.
- `color`: the roof covering as seen in the photos (`"#rrggbb"`, per part), e.g. anthracite
  tiles `#3a3d40`, red clay `#8a3a2a`, a grey flat roof or gravel `#8c8f92`. Default dark red, so
  set it whenever photos show the roof.
- Older files (format 6, `footprint` with the overhangs in it and the eaves at its edge) are
  converted when the app or house:check opens them; the old file stays beside it as
  `house.v6.json`. Never write `footprint` any more.
- **Two sheds that are one gable** (a format 6 workaround for unequal overhangs): two `shed` parts
  on the same `floor` with opposite `low` sides, outlines meeting at the ridge line (within 2 cm)
  and spanning the same length along it, both reaching the same height there. house:check
  accepts them (the seam needs no wall), but merge them into one `gable`:
  - `outline`: both outlines together (along the ridge the same; across it from the one half's
    low side to the other's). `ridge`: the seam's direction (a seam along x is `east-west`).
  - `overhang`: each eave side from its own half; the verge sides from either half. If the halves'
    verges differ, the gable cannot keep both: show the two values and ask which to keep.
  - `eaveHeight` per side: each half's `eaveHeight` (both at the wall line since format 7).
  - Pitch: equal eaves and equal half widths give `ridgeHeight` (the halves' height at the seam).
    Otherwise write `pitch` per side, each half's `atan((ridgeHeight − eaveHeight) / width)` in
    degrees, width = the half's depth across the ridge: the planes then stay exactly where they
    were and meet at the old seam.
  - Halves at different heights at the seam (a step in the ridge) are not one gable: leave them.
  - Keep the id the views and roof windows use most, re-point every `roofWindows[].roof` and every
    view's `roofs` entry of the other id to it, merge `tags`, keep `color`.
  - Check: house:check prints the same pitch per side as before and no `GABLE OPEN`; render
    before and after and compare (the roof should look unchanged, apart from a verge that had to
    give up one value).
- **Roof windows** (`roofWindows`): `roof` = part id, `center` in plan coordinates, `width` along
  the eave, `depth` = extent across the eave **in the plan**, i.e. window length × cos(pitch).
  Typical sizes (width × length): 0.78 × 0.98, 0.78 × 1.18, 0.78 × 1.40, 1.14 × 1.18, 1.14 × 1.40,
  1.34 × 0.98. Plans mark them as rectangles with crossed lines or "DFF".
- Not representable yet: dormers, mansard roofs, polygon footprints. Approximate (e.g. a dormer
  by a small gable part) only after asking; list them in `model-notes.md`.

## Surroundings

- `ground`: `floor` = the storey whose FFB the terrain refers to (usually the ground floor) and
  `level` (e.g. `-0.30` if the site plan gives terrain at −0.30 relative to ±0.00).
- `ground.outline` (required): **where the terrain ends**, its corners in plan coordinates in
  order, every edge east-west or north-south (right angles; an L-shaped plot or a cut-out corner
  is fine, slanted edges are refused). The terrain is exactly this; nothing is derived from the
  house, the garage or the areas, so decide each edge yourself: from the site plan's plot
  boundary (Lageplan), or as the user describes it ("the plot ends at the street in the north",
  "one metre of lawn east of the garage"). Everything that lies outside (the house, outbuildings,
  the terrace, a forecourt) belongs inside it; a long garden may be cut short for compact views
  (say so). Parts of areas past the outline are cut off. A street or pavement along the plot is
  an area of its own inside the outline. `house:check` prints the outline's corners and extent
  and draws it as a green dashed line in the ground floor's plan; compare it with the site plan.
  When the user changes the surroundings ("the lawn reaches only a metre past the garage"), move
  the affected outline corners, not other parts.
- `ground.pits`: light wells, sunken areas in front of basement windows, exterior basement stairs
  (`stairs.down`). Their `bottom` is relative to the ground floor's FFB.
- `ground.areas`: terrace paving, gravel strips, drive, with a color.
- `structures`: steps, landings, entrance canopy, lintels, pillars, balcony slabs — simple boxes
  with `bottom`/`top` relative to their `floor`. Each needs a unique `id` (lower case, digits,
  hyphens, e.g. `pfeiler-eingang`): views refer to it to make a part transparent or hide it, so
  keep ids stable when the model is corrected; `name` says in the user's language what it is.
- **Flat-roofed parts** (bump-outs, a single-storey wing): the app carries their facade, in its
  `facadeColor`, on up to a roof part of the same storey, so nothing needs closing below the
  roof. A visible parapet (Attika) is drawn as `structures` boxes along the outer faces, from the
  storey height up. Boxes and roof must never share a face (the roof draws a 16 cm
  edge band at its outer edge, coplanar faces flicker): let the boxes stand 3 cm proud of the
  roof's edge (outline plus overhang) and end 1 cm below the roof surface. If the parapet frame
  also runs over a recessed entrance, the flat roof's outline covers the recess too.
- `pavilions`: conservatories and glass extensions (only hinted); `roomId` if it is a Loxone room.
  `bottom`/`top`/`lowTop` are relative to the floor's FFB, not to the terrace: a conservatory
  2.75 m high at the house on a terrace at −0.15 gets `top: 2.60`. Manufacturer drawings give width,
  depth and both heights.
- An outbuilding (garage, shed) is a building of its own, never a wall neighbour of the house:
  - Its polygon is its **inside**, like a room's; `exteriorWall` (e.g. 0.12 for a prefab
    garage, 0.24 for masonry) puts its walls outside it. Without `exteriorWall` it has no walls
    (only for something open; a closed garage always gets them). Its outer faces must keep their
    gap to the house (a prefab garage usually stands 2 cm off): inside = outer size minus twice
    the wall.
  - `level`: its floor relative to the storey's, e.g. `-0.32` when the garage floor lies 32 cm
    below the ground floor. `height` counts from that floor up to the underside of its roof
    (printed outer height of a prefab garage minus roof build-up, or up to the roof part's
    height: the roof clips the walls anyway).
  - Its roof is a roof part of its own over it (it inherits the area's tags).
  - Its gates, doors and windows sit on the polygon edges like the house's; their `sill` and
    `top` still count from the storey's floor, so a gate starting at a garage floor at −0.32 has
    `"sill": -0.32` (negative sills only in outbuildings).
  - It is a room like any other: a view can make it active (presence, light, labels, its name on a
    back wall), its walls can be set per view, and it gets a group tag (`"Garage"`) so views can
    leave it out (`hide`) or switch it (`toggles`) where it would hide the house.
- Garage gates are doors with `"operation": "overhead"` (never `turn`: a 2.5 m leaf swinging out
  looks absurd); map their Loxone `position` (0 closed, 1 open) as for motorised windows.
  Model their roof as a roof part of its own: a view that looks across it hides it with
  `"roofs": { "<id>": "hidden" }` in its view file (or `"transparent"`, `"solid"`). Without a
  setting a roof part is `cutaway`: the slope facing the camera opens above the view's cut-open
  rooms up to their ceiling, so rooms under a low roof (knee wall) need no setting.
- `spaces[].facadeColor` for facade parts of another color (dark grey wood cladding, colored
  plaster); from the photos.

## Light sources

Lamps besides the room light: lanterns along a walkway, wall lights on the facade, lit furniture
inside (pendant over the dining table, lit cupboards, shelves). Each shows its own light, or its
room's. Only this skill places them; the app has no editor for them (decided by the owner), so
"add a light", "move the lantern", "the wall light hangs higher" all come here. Format:
`${CLAUDE_PLUGIN_ROOT}/skills/house-model/reference/model-format.md` L, schema `lights` in `${CLAUDE_PLUGIN_ROOT}/skills/house-model/schema/house.schema.json`, an example of every kind
in `${CLAUDE_PLUGIN_ROOT}/skills/house-model/example/house.json`.

| `kind` | For | Placed by | `height` (above the storey's floor) |
|---|---|---|---|
| `pendant` | lamp hanging from the ceiling: a shade (default) or a slim tube (`"shape": "tube"`, `size` the diameter, default 0.06; `length`, default 0.3; narrow beam) | `position` under the lamp | lower edge of the lamp (over a table about 1.45–1.6; several tubes often at staggered heights) |
| `ceilingSpot` | recessed spot | `position` | none inside (the area's ceiling); outside every area the ceiling it sits in, e.g. the bottom of a canopy (`structures`) |
| `lantern` | bollard or lamp post | `position` where it stands | the lamp head (a 0.8 m bollard on terrain at -0.3 has 0.5) |
| `wallLight` | wall lamp, inside or on the facade | `position` on the wall's **inner face** (an edge of a room polygon, like an opening's ends), `side` `inside`/`outside` | the lamp's center (outside typically 1.8–2.2, beside a door about door height) |
| `strip` | line of light under a cupboard, in a shelf, a cove; also rising (under a sloped ceiling, along stairs) or upright (in a shelf's side, a door frame) | `from`, `to` | the `from` end (under wall cupboards about 1.4); `toHeight` for the `to` end when it rises; upright: `from` and `to` the same point, `height` the bottom, `toHeight` the top |

- **Plan points, never wall ids.** A facade light uses the point on the inner face of its
  exterior wall with `"side": "outside"`; the scene puts it on the outer face. `house:check`
  reports a wall light whose point lies on no edge. An interior wall has no outside: put the
  light on the neighbouring room's face with `"side": "inside"`.
- **Not in an opening:** a wall light must not lie within a window or door (width and height);
  `house:check` reports it. Beside a front door is typical (check the door's `start`/`end`).
- **Room:** derived from the point (the area it lies in, or whose wall carries it inside). Give
  `room` only where the point cannot tell. Outside every area and on a facade a light has no room.
- **Data:** the room's light roles, `light`, `lightColor` (also a list), `lightLevel`, filled in
  block 7/8 like a room's. Without data a light follows its room's light: right for lamps on the
  room's own output (ceiling spots on the room's dimmer). Outside lights need data; without it
  they stay dark (`house:check` warns, it does not fail, so lights can be placed first and mapped
  afterwards).
- **One output, several lamps:** three lanterns on one switch each name the same datapoint.
- **Strips on furniture:** a strip under a wall cupboard, on top of a wardrobe or up a shelf's
  edge sits on the piece (see *Furniture*): `"on": { "furniture": "<id>", "edge": "<edge>" }`
  instead of `floor`, `from`, `to`, `height` and `toHeight`. It runs the edge's whole length,
  moves with the piece and goes with its tags. Light falls down from a bottom edge (under a
  cupboard: `bottom-front`), rises from a top edge (a cove: `top-front` or `top-back`) and
  spreads outwards from an upright one (`front-left` …). Prefer this to a free strip whenever
  the light belongs to a piece.
- **Strips along stairs** rise with `toHeight`, the plan line following the stair's edge. Both
  ends must lie inside the area the strip's middle is in: above its floor (0 is refused, use
  e.g. 0.05) and at most at its ceiling. A storey's stairs are an area of that storey, not an
  opening to the next one, so a strip that climbs a whole storey is **two strips**: one in the
  lower storey from the bottom step to just below its ceiling, one in the upper storey from just
  above its floor on along the stair line, both with the same `data`. The short piece through the
  slab is hidden anyway. Ids like `treppe-eg`, `treppe-og`; render with `--lit=lights` and check
  that the two pieces line up.
- **Tags:** implied `light`, the area's tags (a lamp in the garage goes with "Garage"), and
  `outside` for lights outside the building. Add group names like for other elements ("Garten"
  for the walkway lanterns) so a view can switch them.
- **Optional:** `color` (the body when off), `size`, `range`, `beamAngle` (the scene's defaults
  per kind are usually right); `beam` for wall lights: `down` (default), `up`, `both`.
- **Typical sources:** the electrical plan (Elektroplan, Elektroinstallationsplan) marks every
  luminaire with a symbol (circle with a cross, often with a letter for the type); facade photos
  show wall lights; photos at dusk show which lamps exist outside; the Loxone inventory's light
  outputs name what is switched ("Wandstrahler Haustür", "Erdkabel Leuchten Eingang"). Positions
  from photos and descriptions are *estimated* (to about ±20 cm), say so.

## Furniture

Simple boxes (lines with a faint fill) that show where furniture stands, mainly so lit furniture
can carry its strips. Not a furnishing plan: model the pieces that matter (kitchen rows, wall
cupboards, wardrobes, shelves with light, the dining table under a pendant), each as one box.

- **Two forms**, both with `bottom` and `top` above the storey's floor:
  - against a wall, also a slanted one: `from` and `to` are its **back edge** (plan points, e.g.
    on the inner face of the wall it stands against), `from` the left end **as seen standing in
    front of it**; `depth` towards the front. The front lies to the right of `from`→`to`. A
    free-standing piece at an angle uses one long side as its back.
  - along the plan axes: `x0`/`x1`/`y0`/`y1` and `front` (`north`/`south`/`east`/`west`, the
    side facing into the room).
- **Typical sizes:** kitchen row 0.6 deep, `top` 0.9; wall cupboard 0.35 deep, `bottom` about
  1.45–1.5, `top` 2.1–2.3; wardrobe 0.6 deep, `top` 2.0–2.4; dining table `top` 0.75.
- **Edges** (for strips) are named from the front, left and right as seen standing in front:
  `top-front`, `bottom-front`, `top-back`, `bottom-back`, `top-left`, `bottom-left`,
  `top-right`, `bottom-right`, and the upright `front-left`, `front-right`, `back-left`,
  `back-right`.
- `color` (lines and fill; default light grey) and `opacity` (0 lines only, 1 solid; default
  0.25). Give a color only when it helps tell pieces apart (wood for a table).
- **Room:** from the middle of the footprint; the piece shows when a view opens that room. `room`
  only where the middle cannot tell. It must fit between its area's floor and ceiling.
- **Tags:** implied `furniture` (one switch for all of it), the area's tags and `outside`. Add
  group names on request ("Küche", "Einbauschränke").
- **Sources:** floor plans usually draw kitchen rows, wardrobes and fixed furniture (built-in
  units with their depth); photos and the user's words for the rest. Positions from photos are
  estimated; say so. `house:check` lists each piece and draws its footprint with the front edge
  thick: check that the thick edge faces into the room.

## Weather station

Where the weather station stands; the app draws a cup anemometer there (larger than real, so it
reads at house scale) and the weather panel above it. Format: `${CLAUDE_PLUGIN_ROOT}/skills/house-model/reference/model-format.md` M, schema
`weatherStation` in `${CLAUDE_PLUGIN_ROOT}/skills/house-model/schema/house.schema.json`, an example in `${CLAUDE_PLUGIN_ROOT}/skills/house-model/example/house.json` (on the
main roof's ridge). At most one.

```json
"weatherStation": { "floor": "og", "position": [10.4, 4.75], "height": 5.3 }
```

- `position`: where the mast stands, a plan point. `floor`: the storey its height refers to; on a
  roof, the storey the roof part belongs to (the top storey for the main roof, the ground floor
  for a garage's flat roof).
- `height`: the cups above that storey's floor. The mast reaches down by itself to the roof
  below (or the terrain), so only the top is given. On a pitched roof: the roof height at the
  point plus the mast (a Weather Station Tree typically 0.8–1.5 m above the roof). `house:check`
  prints the station's line (the roof height at its point and how far above it the cups are, its
  tags, which weather roles are mapped), marks it in blue in the storey's plan, and reports cups
  at or below the roof or terrain.
- Tags: implied `weatherStation` and `outside`, plus `roof` when it stands on a roof (it goes with
  the roof when a view leaves the roof out). The view's tag buttons can show or hide it.
- The weather values themselves are house-wide `data` (block 7), not part of the station: a house
  can show the sky without a station, and a station without values shows an empty panel.
- Positions from photos are *estimated*, say so.

## Indicators

Icons that appear while a datapoint is in some state: music playing, mail in the mailbox, the
washing machine done. Mapping the datapoints: `loxone-mapping.md` (audio zones there). The
icon per kind is the user's choice (menu "Globale Einstellungen …"); a kind you invent gets a
placeholder until they pick one, so say so. If they ask you to set one, `global.json`
`indicators` takes per kind a Fluent Emoji name (`"fire"`), another set's icon as `"set:name"`
(`"lucide:flame"`, a one-colour outline icon), the latter with a colour as
`{ "icon": "lucide:flame", "color": "#ff6b5b" }`, or `"hidden"`; only names the app carries load
(the check names a wrong one).

- **One thing is one indicator; its states are rules.** The rules are checked in order, the first
  matching one applies, and it may name another `kind` (another icon) for its state. A rule
  without `when` always matches: put it last as the default. Never make one indicator per state
  ("mail", then a second for "no mail"); give the most important state the first rule (a package
  before plain mail, so both at once shows the package):

  ```json
  { "id": "postkasten", "kind": "mail", "room": "vorraum", "rules": [
    { "when": { "datapoint": "postkasten.packet", "equals": 1 }, "kind": "package" },
    { "when": { "datapoint": "postkasten.mail", "equals": 1 } },
    { "kind": "no-mail" }
  ] }
  ```

  No rule matching hides it (stopped music shows nothing). `"show": false` in a rule hides it for
  that state explicitly, ahead of later rules.
- **In a room's row** (`room`): the usual case, for what concerns the room (music, a window
  contact summary). Every indicator of a room shares its row; the views place the row.
- **At a point** (`position` instead of `room`) for an indicator that belongs to a place of its own:
  the mailbox by the front door, the washing machine in a cellar corner, the garage gate.
  `{ "floor": "eg", "x": 6.1, "y": -0.4, "height": 1.2 }`, `height` the icon's middle above the
  storey's floor (default 1.5; inside a room it must stay below the ceiling). It shows with the
  room its point lies in; outside the building with `outside`. `tags` (only here) for a group of
  the user's. Suggest it when the user names a place ("at the mailbox"); ask where if the plans
  do not show it. `house:check` lists it per storey and marks it as a purple diamond in the plan;
  `house:render` shows the icon there (the simulation shows every indicator).
  Where the icon covers something in one view, the user drags it in the view editor ("Ansichten
  bearbeiten …"); that view stores `"indicators": { "<id>": { "offset": [east, north], "height": h
  } }` relative to the point. Leave it to them; move the point itself when it is wrong everywhere.

## Tags

Tags let a view leave groups of elements out (`"hide": ["outside"]` in a view file) and let the
user name groups of their own. Background and rules: `${CLAUDE_PLUGIN_ROOT}/skills/house-model/reference/model-format.md` J.

- **Implied tags come from the kind; never write them where the kind already says it.** Roof
  parts are `roof`; structures `structure`; pavilions `pavilion` + `outside`; areas with
  `outbuilding: true` `outbuilding` (without `exteriorWall` also `outside`); outdoor areas (`height: 0`) `outside`; the
  terrain `ground` + `outside` (its pits too); a roof part over outbuildings only also gets their tags.
  Openings, roof windows and gables go with their area or roof part and have no tags of their own.
- **Add `"tags"` only for what the kind cannot tell** (optional on `spaces`, `roofs`,
  `structures`, `pavilions`, `lights`, `furniture`, `ground.pits`, and `indicators` at a point):
  - On pits a group name such as `"Lichtschacht"` when the user wants to leave the light wells out
    with the basement (a view with `floors` without the basement still draws the terrain and its
    pits; `"hide": ["Lichtschacht"]` leaves them out, and the terrain closes over them). Exterior
    basement stairs are no light well: a tag of their own or none.
  - `outside` on structures and roof parts that stand apart from the building: garden walls,
    freestanding steps or a pergola, a carport roof and its posts. Structures attached to the
    house (entrance pillars and lintels, a balcony slab, a parapet, steps at the front door) are
    part of the building: no `outside`.
  - A group name for elements that belong together across kinds, when the user names one or it
    is obvious from the plans: `"Carport"` on the carport roof and its posts, `"Garage"` on the
    garage area and its roof part (a roof over outbuildings only would inherit it, but tag it
    anyway, so the group is obvious in the file). Names as the user calls them, in their language, like
    room names; one or two words; reuse an existing tag instead of a near-duplicate ("Garten",
    not also "Gartenbereich").
  - An element can carry several tags (`["Carport", "outside"]`); a view hides it if any of them
    is hidden.
- In block 5 (surroundings), tag as you go and list the tags in the summary for the user
  (`house:check` prints every tag with its elements); ask about a group name only where a group
  is plain (carport, garage, garden), don't invent groups for single elements.
- **Requests about tags** (also in an existing model):
  - "Create a group Garten with the garden wall and the pergola" / "put X into Y": add the tag
    to each element's `tags`; show the `house:check` tag list afterwards.
  - "Rename the group Garten to Außenanlage": rename the tag on every element, then search
    `views/*.json` for it in `hide` and `toggles` and rename it there too (a view naming a tag no element
    carries only warns, so a missed one fails silently).
  - "Take the pergola out of Garten" / "delete the group": remove the tag from the element(s);
    say which views hid it.
  - "The garden wall is outside": add `outside`. "The carport posts belong to the house": remove
    an added `outside`; an implied tag cannot be removed (say so: e.g. a pavilion is always
    `outside`, a view can still show it with its own setting where the view has one).
  - Never rename or remove an implied tag; never change element ids for a tag change.

## Views that leave storeys or tags out

Views are the user's: the view editor sets both in its block "Angezeigte Elemente". On
request (or from the house assistant, which has no editor at hand) a view file may get `floors`
and `hide` directly:

- `"floors": ["eg"]` shows only those storeys (ids from `floors` in house.json); everything on
  the others is left out, the terrain stays (add `"hide": ["ground"]` to drop it). Active rooms
  on storeys left out stay in `rooms` (they come back when the storey is shown again), but at
  least one active room must be on a shown storey; activate the shown storey's rooms the user
  wants to see.
- `"hide": ["outside", "roof"]`: every element carrying one of the tags is left out, unless the
  view sets it itself in `roofs` or `structures` (`"hide": ["roof"], "roofs": { "wing": "solid" }`
  keeps one roof part).
- `"toggles": ["roof", "outside"]` gives those groups a switch on the page ("I want to switch the
  roof off on the tablet"): live, not saved, the camera stays put; gable walls stay when the roof
  is off. A tag in `hide` gets no switch. When a tag is renamed or removed, rename or remove it in
  `toggles` as well as in `hide`.
- Check with `house:check` (it warns about unknown storeys, tags no element carries, a view
  without a visible active room, switches for tags that are left out) and render the view.

## Checking

After every change: `"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --house-check <model folder>`. It validates against the
schema and `validate.ts` (overlaps, openings on no edge, roof windows across slopes, …), prints per
storey every area with area, bounding box and **edge lengths in polygon order** (compare them with
the dimension chains), every opening with width/sill/top, every roof part with its pitch, and
draws `test-output/house-check/plan-<floor>.png` (north up, 1 m grid in plan coordinates, room
edges labelled, interior walls grey, exterior walls dark grey, openings blue/brown, roof faces red dashed). Look at the
PNG next to the original plan before showing it to the user.
