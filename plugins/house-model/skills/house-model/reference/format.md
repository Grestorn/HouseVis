# From plans to house.json

The authoritative definition is `${CLAUDE_PLUGIN_ROOT}/skills/house-model/schema/house.schema.json` (every field has a description) and the
reasoning is in `${CLAUDE_PLUGIN_ROOT}/skills/house-model/reference/model-format.md`. This file explains how to get the values **out of the
plans**. Read both schema and this file before block 1.

## Skeleton

`<model folder>/house.json`:

```json
{
  "version": 7,
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
- **Split levels**: an area whose floor lies a few steps below or above its storey's (a sunken
  living room, plans print its own level, e.g. `OKFF -0,36` or "3 Stufen") gets `level` (`-0.36`).
  Its ceiling stays at the storey's by itself; give `height` only where the section shows another
  ceiling. Sills and heights in it still count from the **storey's** floor: a terrace door at the
  room's floor has `"sill": -0.36`; a BRH printed in the room counts from its own floor, so add
  `level`. The steps are a stair staying on the storey (`"to"` = its own floor, *Stairs*).
- Outbuildings (garage, shed) get `outbuilding: true`; they are no wall neighbours of the house
  but buildings of their own (see *Surroundings*). An open carport is no area at all: a roof part
  with posts (`structures`).
- Typical values when nothing is given (mark them estimated): storey 2.75–3.00, slab 0.20–0.25,
  basement 2.40–2.60.

### Double-height spaces (Luftraum, gallery)

A room open through more than one storey: a living room with a Luftraum, a two-storey hall or
stairwell, a two-storey bay. Plans show it on the **upper** storey's plan as an area crossed out
with two diagonals and labelled "Luftraum" (or "LR", "Luftraum über EG"), often with a railing
along a gallery; the section shows one room through both storeys and no slab.

- **One area, on the lower storey, with `openTo`**: the id of the highest storey it reaches
  through (`"openTo": "og"`). It is the room standing there (the living room under the Luftraum,
  the Loxone room with its values); the crossed-out area on the upper plan gets **no area** of its
  own. No area of the upper storey may overlap it (validation); the gallery beside it is an
  ordinary area of the upper storey, its edge towards the Luftraum an interior wall gap as usual.
- Its height comes by itself (the storeys it passes, up to their slab or roof); `height` only
  where the section shows a different one.
- **Tall windows** reaching from the lower storey into the upper one are one opening on the lower
  storey, `sill` and `top` from its floor (`"sill": 0.5, "top": 4.8`), read from the elevation.
  Only an `openTo` area's windows may reach above their storey. Glazing that crosses a slab
  between two ordinary rooms (a stair window in front of a closed stairwell) stays two windows,
  one per storey.
- A gallery's railing towards the Luftraum is a `railings` entry along the wall gap between the
  gallery and the Luftraum; it opens the wall there (*Railings*).
- Check: house:check prints `open to og (…)` on the area and crosses it out on the upper
  storey's plan; no overlap error. Render a view with the room active and compare with the
  section.

```json
{ "id": "wohnen", "roomId": "wohnen", "floor": "eg", "openTo": "og", "polygon": [[0.365, 0.365], [5.2, 0.365], [5.2, 4.4], [0.365, 4.4]] }
```

### Stairs (Treppen)

A stair between two storeys, read from the floor plan of the storey it starts on: its outline,
the step lines, the **walking line (Lauflinie)** with a dot at the bottom and an arrow at the top,
and the label `16 St. 17,8/27` (16 rises of 17.8 cm, going 27 cm). The plan above draws its
opening dashed or crossed.

- `stairs`: `{ "id", "floor", "to", "line", "width", "rises", "turns" }`. `floor` the storey it
  starts on, `to` the one it arrives on (default the next one up; its own storey for steps between
  split levels; more than one storey up only from an area open through the storeys between). `line`: the walking line's
  points in plan coordinates, **from the front edge of the first step to the edge of the upper
  floor**, a corner wherever it turns; it runs in the middle of the stair. Two points straight,
  three an L, four a U. Every corner turns by 90°.
- `rises`: the label's step count (Steigungen). The rise comes from the floor-to-floor height;
  compare it with the label's (`house:check` prints `16 rises 17.8 cm, goings …`).
- `turns`, one per corner: `"landing"` where the plan shows a square landing (Podest) in the
  corner, `"winders"` where the steps fan round it (gewendelte Treppe, step lines meeting at the
  inner corner). Spiral stairs are not in the format; tell the user.
- `width` the stair's width (often 0.9–1.1); `color` of the steps if known (wood, stone).
- The opening in the storey above is cut by itself. The stair must start in an area of its
  storey and reach an area of `to` (or a Luftraum passing it) at its end, within 0.5 m (a
  passage in a wall may lie between). Validation refuses rises outside 12–25 cm and goings
  outside 18–40 cm: usually a wrong `rises` or a line drawn along the wrong edge.
- Check: the storey's plan draws the treads, the walking line and the label; compare with the
  floor plan. The railing along its open side is a `railings` entry with `stair` and `side`
  (*Railings*).

```json
{ "id": "treppe", "name": "Treppe", "floor": "eg", "to": "og", "line": [[5.0, 5.6], [6.1, 5.6], [6.1, 8.1], [4.32, 8.1]], "width": 1.0, "rises": 16, "turns": ["landing", "winders"] }
```

### Railings (Geländer, Brüstungen)

Plans draw a railing as a thin line (often doubled or dashed) along a gallery's edge, round a
balcony or a roof terrace, beside a stair; the elevations and photos show its kind.

- `railings`: `{ "id", "floor", "line", "kind" }` along a line on a storey, at any angle, or
  `{ "id", "stair", "side", "kind" }` along a stair's side (`left` or `right` seen walking up; the
  side away from the wall, usually the inner one of a turning stair). `kind`: `glass`, `bars`
  (handrail and upright bars) or `solid` (a closed parapet, e.g. a masonry balcony parapet).
  `height` defaults to 1.0 (0.9 above a stair's nosings); `base` lifts a line railing's foot
  (a balcony slab standing above the storey's floor).
- **A gallery's edge**: put the line in the middle of the wall gap between the gallery and the
  Luftraum (or between two areas): a line railing at most 0.5 m off an interior wall, parallel to
  it, **opens that wall** along it (`house:check` lists the walls each railing opens). Exterior
  walls are never opened. Where a stair arrives through the railing it leaves the stair's width
  free by itself; the wall stays open there.
- A balcony: the slab stays a `structures` box; its railing runs round the slab's free edges
  (not along the house wall), outside every area, so it carries `outside`.
- Check: the plan draws railings as heavy lines (glass dashed); a render with the gallery or the
  hall active shows the open edge.

```json
{ "id": "galerie", "floor": "og", "line": [[4.2, 4.52], [4.2, 9.135]], "kind": "glass" }
```

## Openings (windows and doors)

Glazed doors to terraces or balconies are `kind: "window"` with `sill: 0` (they behave like
windows: contacts, shades); `kind: "door"` is for opaque or mostly opaque doors (front door, side
door, basement door), and for interior doors.

**Interior doors, passages and glass partitions** are openings too, written on either room's face
of the interior wall (the room across is found by itself; both get the hole):
- **Doors between rooms** from the floor plan: the door symbol (leaf line and swing arc) gives the
  position and width, often labelled `0,885/2,01` (width/height, sill 0) or `1,01/2,01`; `side`
  from the arc (the hinge, seen from the room the door is written in). Ids
  `<room>-<other room>` (`diele-wohnen`), stable once the user has seen them.
- **Passages** (a gap in the wall with no door symbol, "Durchgang", often a lintel line dashed
  above): a door with `"operation": "none"`, `top` from the section or 2.10–2.26.
- **Glass doors** (a living room door with glazing, a shower or sauna door): `glass: true`;
  **frameless** glass held by fittings (sauna fronts, glass partitions): `frame: false`.
  A **glass partition** is a window with `"operation": "fixed"`, `sill` 0 for full height.
- Plans with no door symbols: ask whether the user wants interior doors at all, and where; do not
  invent them.

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

`roofs` is a list of parts (`gable`, `halfHip`, `hip`, `shed`, `flat` on a rectangle; `polygon`
on any outline); see the table in `${CLAUDE_PLUGIN_ROOT}/skills/house-model/reference/model-format.md`. Most houses are one `gable` or `hip`.
A **half-hip (Krüppelwalm, Krüppelwalmdach)**, where the gable wall stops below the ridge in a
small hip, is `halfHip`: the gable's values plus `hipEaveHeight` (the top of the gable wall at
the gable's wall line, from the section or the gable elevation) and `hipPitch` (the hip's own
pitch, usually steeper, often printed as "DN 50°" at the hip); per gable side where the ends
differ, or only one side for a half-hip at one end. An L, T
or U house, a main roof with a cross gable, a bay with its own small roof running into the main
one, a canopy wrapping round a corner: **one `polygon` part** (*Polygon parts* below), not
several rectangles. Parts that really are separate roofs stay separate: a wing one storey lower
(its own `floor`), a garage with its own `flat` or `shed`; where such parts overlap, the higher
one wins.

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
- **PV arrays** (`pvArrays`, optional): a rectangle of panels lying in one slope, placed exactly
  like a roof window: `roof` = part id, `center` in plan coordinates, `width` along the eave,
  `depth` across the eave **in the plan** (length on the slope × cos(pitch)), `columns` panels
  along the eave and `rows` up the slope (default 1 each). One array per block of panels: a slope
  with a roof window between two blocks gets two arrays (an array must not cover a roof window,
  and must lie within one slope). They come from roof photos and drone shots, rarely from plans:
  count the panels per row and column and take the size from the count (a panel is about
  1.0–1.15 × 1.7–1.8 m, portrait unless the photo shows them lying across) rather than from the
  photo's edges; place the block against what you can measure on the roof (the verge, the ridge,
  a roof window). In a drone shot taken at an angle, the roof's top shifts away from the camera
  against the ground, so measure along the roof, never against the ground below.
  `house:check` lists each under its roof part (slope it faces, panels, size on the slope, the
  panel size it implies: check that against the real panel) and draws it dark blue in the plan.
  The panels are drawn flat on the slope; a view that cuts the roof open cuts them too.
  ```json
  { "id": "pv-sued", "roof": "main", "center": [2.9, 2.0], "width": 5.2, "depth": 2.8, "columns": 5, "rows": 2 }
  ```
- Dormers: `dormers` (*Dormers (Gauben)* below), not small roof parts.
- Not representable yet: mansard roofs, hipped, flat, eyebrow and round dormers. Approximate
  only after asking (a hipped dormer as a gable dormer, a flat one as a shed with a small pitch);
  list them in `model-notes.md`.

### Polygon parts

For an outline that is not a rectangle. Every edge of the outline carries a slope (rising
inwards from its edge) or ends in a gable wall; where two slopes meet at an outside corner they
form a hip, at an inside corner a valley, and a narrower wing at the same pitch gets a lower
ridge that runs into the main slope by itself, as on the real roof.

```json
{ "id": "main", "kind": "polygon", "floor": "og",
  "outline": [[0, 0], [12, 0], [12, 4], [8, 4], [8, 10], [4, 10], [4, 4], [0, 4]],
  "eaveHeight": 1.0, "pitch": 35, "overhang": 0.6,
  "edges": ["slope", "gable", "slope", "slope", { "gable": true, "overhang": 0.3 }, "slope", "slope", "gable"],
  "color": "#3a3d40" }
```

- `outline`: the wall line corner by corner (outer wall faces, as for a rectangle part, within
  2 cm), in either direction. Edge i runs from corner i to the next, the last back to the first.
- `edges`: **one entry per edge, in the same order** (the check refuses another count):
  `"slope"`, `"gable"`, or an object giving that edge its own values (`{ "gable": true,
  "overhang": 0.3 }`, `{ "pitch": 45 }`, `{ "eaveHeight": 2.0 }`). Gable edges are where the
  plans show a gable wall (Giebel): the ends of the main roof and of a cross gable. Eaves
  (Traufe) are slopes. When you add or remove a corner, add or remove its edge too.
- `eaveHeight` and `pitch` for every slope, `overhang` for every edge (default 0); per edge only
  where it differs (a verge flush with the wall, a steeper cross gable). **No `ridgeHeight`**:
  where the section gives eave and ridge heights, the pitch is
  `atan((ridgeHeight − eaveHeight) / half the width across that ridge)` in degrees. Several
  ridges at different heights from one pitch (a narrower wing) are expected; different pitches
  per wing go on that wing's edges.
- `pitch` 0 on every edge is a flat roof at `eaveHeight` (an L-shaped flat roof).
- No `rotation`: the corners are plan coordinates already.
- house:check prints a polygon part's corners, defaults and top, and every edge in order (slope
  or gable, length, its own values): compare the top with the section's ridge, the edge lengths
  with the dimension chains. Open gables are named by edge (`GABLE OPEN on edge 4 (…)`). The
  slopes must cover the whole outline; an error names the gap.
- A wrap-around canopy over no walls: its outer edges are slopes, the edges against the house
  walls gables with `"overhang": 0` (it must not reach into the house), the free ends gables or
  slopes as built; posts as `structures`, tagged together with the part.
- **An existing roof built from several rectangles** that is one roof (a cross gable from
  overlapping parts, a main roof split into pieces so an eave would not run across a gable, two
  sheds meeting in a valley where the real roof has a hip): offer to merge them into one
  polygon. The outline is the outer boundary of the walls under all pieces; the slopes and
  gables as the real roof has them; keep the id the views and roof windows use most and re-point
  the others' `roofWindows[].roof`, `pvArrays[].roof` and views' `roofs` entries. Check with
  house:check (same top, no `GABLE OPEN`) and renders before and after.

## Bays (Erker)

A bay is a part of a room that steps out of the facade: in the floor plan the outline steps out
(rectangular, or with angled sides: three- and five-sided bays), the elevations show its windows
and its own small roof, the section whether it stands on the storey below or hangs out over
nothing (cantilevered, "auskragend").

- **Plan: part of its room's polygon**, not a space of its own (unless the plans name it as a
  room of its own): the polygon steps out by the bay's inner depth; angled sides are angled
  edges. Its walls get the storey's `exteriorWall` like every exterior wall, so the bay's outer
  face lies a wall thickness outside its polygon. Its windows are ordinary `openings` on its
  walls (`start`/`end` on the inner face). A cantilevered bay needs nothing more: the room
  polygon simply reaches beyond the storey below, and the underside is drawn by itself.
- **Its roof is a roof part on the bay's storey, its outline the bay's wall line** (the outer
  faces of its walls, and the facade line where it meets the house):
  - **Shed against a taller facade** (a one-storey bay with the upper storey above it): `shed`,
    `low` the side facing away from the house; `eaveHeight` at the front wall, `ridgeHeight` at
    the facade line (the height where the roof meets the wall, below the upper storey's
    windows). Overhangs on the front and the sides; none on the house side.
  - **Hipped against the facade** (slopes on three sides), or an angled bay's roof: a `polygon`
    part, every outer edge a slope, the edge on the facade a `"gable"` with `"overhang": 0`.
  - **Flat** (often with a balcony above): `flat` with its `height`.
  - **A bay through every storey with a gable of its own reaching into the main roof** (a
    Zwerchhaus): no part of its own: the main roof becomes one `polygon` whose outline steps
    out with the bay, the bay's front edge a `"gable"` (*Polygon parts*).
  - **An upper-storey bay on a gable end**, under the main roof's verge: where the verge does not
    reach over all of it, give it a `flat` (or `shed`) part of its own over its wall line, lower
    than the verge.
- **A bay on the roof's own storey under the eave cannot be**: the roof there is lower than its
  walls. house:check then warns that the walls of the area "stand under its overhang … they would
  reach through the roof". Read the plans again: it is usually a misread storey, a bay of the
  storey below, or a dormer (*Dormers*) or Zwerchhaus with a roof of its own.
- Check: house:check shows the bay in `plan-<floor>.png` and prints the bay's roof part; no
  `GABLE OPEN`, no "under its overhang" warning. Render the view facing the bay and compare
  with the elevation and photos.

```json
{ "id": "erker-kueche", "kind": "shed", "floor": "eg", "outline": { "x0": 0.835, "x1": 3.165, "y0": 9.5, "y1": 10.5 },
  "low": "north", "eaveHeight": 3.0, "ridgeHeight": 3.5, "overhang": { "north": 0.2, "east": 0.1, "west": 0.1 } }
```

## Dormers (Gauben)

A dormer stands on a roof slope with a vertical front and two side walls (cheeks), and a window
in its front. Recognise it in the roof plan (a rectangle drawn on a slope, often "Gaube"), in the
section through it (its front wall standing on the rafters, its own roof meeting the main roof
higher up) and in the elevations and photos. Only shed (*Schleppgaube*: one slope falling to the
front, flatter than the roof) and gable dormers (*Giebelgaube*: a small gable over the front) can
be written; for others see *Roof*.

- **Written by its front, as plans dimension it**, under `dormers` (not `roofs`):
  - `roof`: the part whose slope it stands on; `kind`: `shed` or `gable`.
  - `front`: the outer face of the front wall in the plan, its two ends (either order). The front
    runs along the slope's eave (within 5°); the dormer faces down the slope.
  - `eaveHeight`: above the part's storey floor, at the outer face: a shed's front wall top, a
    gable's cheek top. The section gives it; else the elevation (window top plus lintel).
  - `pitch` of its own roof in degrees; a shed's must be flatter than the slope (often 10–25°),
    a gable's is often close to the main roof's.
  - `overhang`: one value or `{ "front": 0.3, "sides": 0.15 }`.
  - `color` for its roof (default the part's); `wallColor` for front and cheeks (default the
    facade's), e.g. `"#5a5e63"` for slate or zinc cladding.
  How far back it reaches is not written: it follows from where its eave (gable) or its roof
  (shed) meets the slope. Compare the depth house:check prints with the roof plan.
- **Its windows are ordinary `openings`** on the part's storey (`floor`), `start`/`end` on the
  front's outer face (as `front` gives it, the simplest) or its inner face; `sill` and `top`
  above the storey floor. The sill must lie above the roof at the front's foot, the top below the
  front wall's top (a gable dormer's window may reach into its triangle).
- **The room below** needs nothing: its ceiling rises into the dormer by itself, inside the
  dormer's walls. A dormer over a partition is fine; its windows belong to the room behind them.
- **Views** have no setting of their own for a dormer: it goes with its roof part.
- Check: house:check lists it under its roof part (front, width, how far up the slope, eave, top,
  pitch) and draws it violet in `plan-<floor>.png`; errors name what keeps it from standing
  (eave below the roof at the front, a shed steeper than the slope, across the ridge or the roof's
  edge, front over the overhang, on a roof window or PV array). Render the view facing it and
  compare with the elevation.

```json
"dormers": [
  { "id": "gaube-sued", "roof": "main", "kind": "shed", "front": [[7.1, 1.0], [8.7, 1.0]],
    "eaveHeight": 2.4, "pitch": 10, "overhang": { "front": 0.3, "sides": 0.1 } },
  { "id": "gaube-nord", "roof": "main", "kind": "gable", "front": [[5.2, 8.5], [6.8, 8.5]],
    "eaveHeight": 2.3, "pitch": 40, "overhang": 0.2, "wallColor": "#5a5e63" }
]
```
```json
{ "id": "gaube-sued-fenster", "kind": "window", "floor": "og", "start": [7.5, 1.0], "end": [8.3, 1.0], "sill": 1.9, "top": 2.3, "leaves": 2 }
```

## Surroundings

- `ground`: `floor` = the storey whose FFB the terrain refers to (usually the ground floor) and
  either `level` for a flat terrain (e.g. `-0.30` if the site plan gives terrain at −0.30
  relative to ±0.00) or `heights` for a slope: the site plan's spot heights (Höhenkoten) as
  `[x, y, level]`, each level converted to metres relative to ±0.00 (a plan's `NN +412,35` minus
  the ±0.00 it states in NN). The terrain runs linearly between neighbouring points (triangles);
  two points give a slope along their line, level across it. Take the plot corners and the
  points round the house; where the slope changes along a row, give points along it about as far
  apart as the rows are, or the triangles cut across (the ground floor's plan from `house:check`
  draws them dotted, with each spot height as a cross: a long thin triangle across the house
  means a missing point). Without spot heights but with a visibly sloping plot (photos), ask.
- `ground.outline` (required): **where the terrain ends**, its corners in plan coordinates in
  order, at any angle (copy the site plan's plot boundary, also where a street runs slanted). The terrain is exactly this; nothing is derived from the
  house, the garage or the areas, so decide each edge yourself: from the site plan's plot
  boundary (Lageplan), or as the user describes it ("the plot ends at the street in the north",
  "one metre of lawn east of the garage"). Everything that lies outside (the house, outbuildings,
  the terrace, a forecourt) belongs inside it; a long garden may be cut short for compact views
  (say so). Parts of areas past the outline are cut off. A street or pavement along the plot is
  an area of its own inside the outline. `house:check` prints the outline's corners and extent
  and draws it as a green dashed line in the ground floor's plan; compare it with the site plan.
  When the user changes the surroundings ("the lawn reaches only a metre past the garage"), move
  the affected outline corners, not other parts.
- **Footprints** of pits, ground areas, `structures` and `pavilions`: `x0`/`x1`/`y0`/`y1` for a
  rectangle along the plan axes, or `"polygon": [[x, y], …]` at any angle (a terrace along a
  slanted wall, a balcony slab on a bay, a rounded light well as a few corners), never both.
  A polygon pavilion's roof still falls towards `low`.
- `ground.pits`: light wells, sunken areas in front of basement windows, exterior basement stairs
  (`stairs.down`, only in a rectangular pit). Their `bottom` is relative to the ground floor's FFB.
- `ground.areas`: terrace paving, gravel strips, drive, with a color. On a slope an area follows
  the terrain; with `level` it is a **platform** lying flat at that level (a terrace, a drive or
  a seating area cut into the slope, levelled to the garage floor), and an upright edge (a
  retaining wall) closes the gap to the terrain round it. `house:check` prints each platform with
  the terrain's range at its corners: that is how high its edges are.
- **Walk-out basement** (Hanggeschoss): on a slope the basement's downhill side lies free; its
  garden door and windows are ordinary openings, the patio in front a platform at the basement
  floor. `house:check` prints the terrain along every wall in the ground, relative to the room's
  floor, and names openings below the terrain without a light well: compare with the sections and
  add the light well (`ground.pits`) or correct the spot heights.
- On a slope, pits reach up to the terrain at each corner, and lanterns, ground spots and the
  weather station's mast stand on the terrain where they are; validation checks them there.
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
`${CLAUDE_PLUGIN_ROOT}/skills/house-model/reference/model-format.md` ("Light sources"), schema `lights` in `${CLAUDE_PLUGIN_ROOT}/skills/house-model/schema/house.schema.json`, an example of every kind
in `${CLAUDE_PLUGIN_ROOT}/skills/house-model/example/house.json`.

| `kind` | For | Placed by | `height` (above the storey's floor) |
|---|---|---|---|
| `pendant` | lamp hanging from the ceiling: a shade (default) or a slim tube (`"shape": "tube"`, `size` the diameter, default 0.06; `length`, default 0.3; narrow beam) | `position` under the lamp | lower edge of the lamp (over a table about 1.45–1.6; several tubes often at staggered heights) |
| `ceilingSpot` | recessed spot | `position` | none inside (the area's ceiling); outside every area the ceiling it sits in, e.g. the bottom of a canopy (`structures`) |
| `lantern` | bollard or lamp post | `position` where it stands | the lamp head (a 0.8 m bollard on terrain at -0.3 has 0.5) |
| `groundSpot` | spot set flush into the ground, a terrace or path, shining up (at the facade, a pillar, a tree) | `position` where it lies | none: it sits in the floor of the area there (an outdoor area), otherwise in the terrain (its height there) |
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
reads at house scale) and the weather panel above it. Format: `${CLAUDE_PLUGIN_ROOT}/skills/house-model/reference/model-format.md` ("House-wide data and the weather station"), schema
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
- **Where it shows: two ways, and the user chooses.** For every new indicator that belongs to an
  item in a room (the stove, the dishwasher, the cooker hood, a sensor's value), name both and ask
  which they prefer; do not settle it yourself. Several at once may share one question ("the
  kitchen's stove, hood and dishwasher: in the kitchen's row with the music icon, or each at its
  appliance?"). Only for what concerns the room as a whole (its music, a window summary) the row
  goes without asking, and for a thing outside every room (the mailbox, a mower's station) the
  point does.
  - **In the room's row** (`room`): beside the room's other indicators (music …). Every indicator
    of a room shares the row, one spot and one height per view (the views place the row), so the
    row says *which room*, not *where in it*.
  - **At a point of its own** (`position` instead of `room`): at the item, where it is in the
    room or outside; the mailbox by the front door, the washing machine in a cellar corner, the
    garage gate, a robot mower's station, a camera (SKILL.md block 10).
    `{ "floor": "eg", "x": 6.1, "y": -0.4, "height": 1.2 }`, `height` the icon's middle above the
    storey's floor (default 1.5; inside a room it must stay below the ceiling; an appliance's
    top, e.g. 1.0 above a hob, reads well). Take the point from the furniture you modelled
    (block 8) or the plans, else ask where. It shows with the room its point lies in; outside
    the building with `outside`. `tags` (only here) for a group of the user's. `house:check`
    lists it per storey and marks it as a purple diamond in the plan; `house:render` shows the
    icon there (the simulation shows every indicator).
    Where the icon covers something in one view, the user drags it in the view editor ("Ansichten
    bearbeiten …"); that view stores `"indicators": { "<id>": { "offset": [east, north], "height": h
    } }` relative to the point. Leave it to them unless they ask you to place it (*Placing labels
    and indicators in a view* below); move the point itself when it is wrong everywhere.
  - **At a PV array** (`"pvArray": "<array id>"` instead of `room` or `position`): for what that
    array produces, with a `value` (its power). The icon floats above the array's centre; it shows
    while the view draws the array's roof part and goes with its tags (`roof`, so a view or switch
    without the roof hides it). No question needed: an array's power belongs at the array. Where
    one Loxone meter measures several arrays (one string over a roof window), put the indicator at
    the largest of them. The per-view nudge works as for a point.
- **A value** (`"value": { "datapoint": … }`): the indicator also writes a datapoint's value
  beside its icon, e.g. a pond's or a wine cellar's temperature, a heat pump's power (a room's
  humidity is the room role `humidity`, shown in its temperature label; loxone-mapping.md). Use it when the user wants a Loxone value shown at a place in the 3D view; where it
  shows is asked as above (in the room's row, or at a point, also outside). The datapoint must
  give a number or a text (not `"boolean"` or `"color"`); give the datapoint the **format Loxone
  has for that state** (`"format": "%.1f°"`, as `loxone_controls` / the inventory reports it, a
  Meter's per state: `actualFormat` for `actual`, `totalFormat` for `total…`), so the app writes
  it as Loxone does with the app language's decimal separator. Only when Loxone has none, set
  `"unit"` (and `as.decimals`) on the datapoint instead. A rule decides whether it shows:
  `"rules": [{}]` always. To show only the value, without an icon, the kind's icon is set to
  `"hidden"` in `global.json` (menu "Globale Einstellungen …", "Nicht anzeigen"); say so rather
  than inventing a kind for it. A value that belongs to no place (the whole plant's PV power, the
  day's energy) goes on a page as a **data panel** instead, which the user adds in the device editor
  ("Geräte bearbeiten …", "+ Datenpanel"); point them there.

  ```json
  { "id": "teich", "kind": "thermometer", "position": { "floor": "eg", "x": 6.5, "y": -5.2, "height": 0.4 },
    "value": { "datapoint": "aussen.teich" }, "rules": [{}] }
  { "id": "pv-sued-leistung", "kind": "pv", "pvArray": "pv-sued", "value": { "datapoint": "pv.sued" }, "rules": [{}] }
  ```

## Tags

Tags let a view leave groups of elements out (`"hide": ["outside"]` in a view file) and let the
user name groups of their own. Background and rules: `${CLAUDE_PLUGIN_ROOT}/skills/house-model/reference/model-format.md` ("Tags").

- **Implied tags come from the kind; never write them where the kind already says it.** Roof
  parts are `roof`; structures `structure`; pavilions `pavilion` + `outside`; areas with
  `outbuilding: true` `outbuilding` (without `exteriorWall` also `outside`); outdoor areas (`height: 0`) `outside`; the
  terrain `ground` + `outside` (its pits too); lights `light`; furniture `furniture`; stairs
  `stairs`; railings `railing`; the weather station `weatherStation`; a roof part over
  outbuildings only also gets their tags. Lights, furniture, stairs and railings also carry the
  tags of the area they stand in. Openings, roof windows, PV arrays and gables go with their area
  or roof part and have no tags of their own.
- **Every component gets one specific tag of its own**, so the user can switch it alone, while the
  generic implied tag switches all of its kind (`"hide": ["furniture"]` leaves out all furniture,
  `"hide": ["Sofa Wohnzimmer"]` only the sofa). A component is a thing the user would name: a piece
  of furniture, a lamp or a row of lamps, a stair, a railing, a structure or a group of
  structures, a pavilion, a pit, an outbuilding, a roof part other than the house's own. Name it as
  the user calls it, in their language, specific enough to stand alone in a list ("Sofa
  Wohnzimmer", "Treppe Diele", "Galeriegeländer", "Lichtschacht Technik"); parts of one thing share
  it (the kitchen row and its wall cabinets "Küchenzeile", the carport roof and its posts
  "Carport"). Rooms (a view switches them with `rooms`), the house's own roof parts (`roof`), the
  terrain and openings get none.
- **Further tags for what the kind cannot tell** (`"tags"` is optional on `spaces`, `roofs`,
  `structures`, `pavilions`, `lights`, `furniture`, `stairs`, `railings`, `ground.pits`, the
  `weatherStation` and `indicators` at a point or a PV array):
  - On pits a group name such as `"Lichtschacht"` when the user wants to leave the light wells out
    with the basement (a view with `floors` without the basement still draws the terrain and its
    pits; `"hide": ["Lichtschacht"]` leaves them out, and the terrain closes over them). Exterior
    basement stairs are no light well: a tag of their own or none.
  - `outside` on structures and roof parts that stand apart from the building: garden walls,
    freestanding steps or a pergola, a carport roof and its posts. Structures attached to the
    house (entrance pillars and lintels, a balcony slab, steps at the front door) are
    part of the building: no `outside`.
  - A group name for components that belong together, when the user names one or it is obvious
    from the plans: `"Garten"` on the garden's lamps, pool and shed, `"Garage"` on the garage area
    and its roof part (a roof over outbuildings only would inherit it, but tag it anyway, so the
    group is obvious in the file). Names as the user calls them, in their language, like room
    names; reuse an existing tag instead of a near-duplicate ("Garten", not also "Gartenbereich").
  - An element can carry several tags (`["Carport", "outside"]`); a view hides it if any of them
    is hidden.
- Tag as you go (blocks 2, 5, 8) and list the tags in the summary for the user (`house:check`
  prints every tag with its elements); ask about a group name only where a group is plain
  (carport, garage, garden). Each component's specific tag needs no question: use its name.
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

## Placing labels and indicators in a view

Where an icon or label sits in one view is normally set by dragging it in the view editor. When
the user asks you to do it ("the mailbox icon covers the door in the garden view", "line up the
indicators on the ground-floor page"), you can: the renders show every indicator (the
simulation shows them all, whatever their rules), the room labels and the temperatures, so you
see what the user sees.

1. Render the page: `"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --house-check --render <model folder> --page <n|id|title>` (`--device <id>`
   for another device's pages). Look at `view-<id>.png` of the view in question (the page's
   picture `page-<n>-<title>.png` shows all its views side by side).
2. Change only that view's file `views/<id>.json`; every setting there is relative, so it follows
   when the house is edited:
   - **An indicator at a point** (house.json `position`): `"indicators": { "<indicator id>": {
     "offset": [east, north], "height": h } }`, the offset in meters from the indicator's point,
     the height above the storey's floor (default the point's).
   - **A room's indicator row** (indicators with `room`) and its **temperature**:
     `"labels": { "<room id>": { "indicators": { "offset": [east, north], "height": h },
     "temperature": { … } } }`, the offset in meters from the room's centroid (without one the
     spot floats over the middle of the visible floor), the height above the room's floor
     (row default 30 cm below the ceiling, temperature 1.35). The row and the temperature start
     from the same spot; give the row its own offset or height to separate them.
   - **A room's name**: `labels.<room id>.name` (on a wall with `wall`, `shift`, `height`, or
     `{ "floor": true, "offset": … }`, or `"hidden"`); see `${CLAUDE_PLUGIN_ROOT}/skills/house-model/schema/view.schema.json`.
   Offsets are in the house's directions (east, north), not the picture's: work out from the
   view's camera which way is left/right/up in the image before moving anything. Only rooms the
   view cuts open get labels.
3. `"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --house-check <model folder>`, render the page again, compare, repeat. Show the user
   the final `view-<id>.png`.

Move the point in house.json instead when the icon is wrong in every view. Never touch views
the user did not name.

## Checking

After every change: `"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --house-check <model folder>`. It validates against the
schema and `validate.ts` (overlaps, openings on no edge, roof windows across slopes, …), prints per
storey every area with area, bounding box and **edge lengths in polygon order** (compare them with
the dimension chains), every opening with width/sill/top, every roof part with its pitch, and
draws `test-output/house-check/plan-<floor>.png` (north up, 1 m grid in plan coordinates, room
edges labelled, interior walls grey, exterior walls dark grey, openings blue/brown, roof faces red dashed). Look at the
PNG next to the original plan before showing it to the user.
