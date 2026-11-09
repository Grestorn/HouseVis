# Intake: what the user provides, and how

Present this to the user **in their language** when a project folder is new, empty or incomplete.
Translate the explanations; keep the file names exactly as written here (they are the same in every
language, so the skill can recognise them).

## Where the files go

One folder per house, in the folder Claude Code works in (the HouseVis repository, or any folder
the user opens Claude Code in):

```
plans/<project>/
```

`<project>` is a short name in lower-case letters, digits and hyphens, e.g. `plans/korn-haus/`. The
folder `plans/` is excluded from git (`.gitignore`): plans and photos are personal data and usually
copyrighted, and they never leave the machine. Accepted formats: **.pdf**, **.jpg/.jpeg**, **.png**.

The skill writes its notes next to them, `plans/<project>/model-notes.md` (sources, decisions,
estimates, open questions), so the user should not create that name. Where the model itself goes
(`house.json` and the rest) the skill asks: by default into the folder HouseVis shows its house
from, if that holds no house yet; otherwise e.g. into `plans/<project>/config/`.

## Two ways to provide the plans

**A — the architect's document set as it is (recommended when available).** Architects usually
deliver one multi-page PDF (permit set "Eingabeplanung", construction drawings "Werkplanung") in
which every sheet has a title block saying what it shows: "Grundriss EG", "Schnitt A-A",
"Ansicht Süd", "Lageplan". Put that file into the folder **unchanged, under any name**; nothing
needs renaming or splitting. The skill reads the title blocks and lists which page is what.
Several such files (e.g. permit set plus a later revision) are fine; say which one is current if
they disagree.

**B — single files without a title** (photos of plans, cut-out sheets without title block,
screenshots, facade photos). Their content cannot be recognised reliably, so they get a name
from the table below. Mixing A and B is normal: the architect's PDF plus named photos.

## File names for material without a title

Names start with a fixed prefix; after it, anything descriptive may follow (`-` instead of
spaces). Several files of the same kind get a counter or a description.

**`<side>` is the side of the house the picture shows, which is where the photographer stands —
not the direction the camera points.** Standing north of the house and looking south at the
north facade gives `photo-north-1.jpg`. Elevations follow the same rule: "Ansicht Nord" (north
elevation) shows the north facade and becomes `elevation-north`. A corner shot from the
south-west, showing the south and west facades, is `photo-southwest-1.jpg`.

| Prefix / name | What |
|---|---|
| `floor-<n>-<name>.jpg` | Floor plan of one storey. `<n>` counts **from the bottom, starting at 1**: basement `floor-1-basement`, ground floor `floor-2-ground`, upper floor `floor-3-upper`, attic `floor-4-attic`. Without a basement the ground floor is `floor-1-…`. Close-ups of one sheet: `floor-2-ground-part-1.jpg`, … |
| `section-<label>.jpg` | Section through the house, e.g. `section-a-a.jpg` |
| `elevation-<side>.jpg` | Elevation of the `<side>` facade (`north`, `east`, `south`, `west`; for angled houses the nearest one, `northeast` etc. are allowed) |
| `site-plan.jpg` | Site plan (Lageplan) with north arrow, plot, garage, terrain heights |
| `plan-<what>.jpg/.pdf` | Drawings that are not part of the architect's set: the manufacturer's drawing of a conservatory, carport or canopy (`plan-conservatory.jpg`), or the owner's own top view of the plot with measured terrace, paths and drive (`plan-topview.pdf`; small deviations are fine, say so in `notes`) |
| `sketch-<what>.jpg` | Sketches or 3D renderings of such products, e.g. the conservatory as offered |
| `photo-<side>-<n>.jpg` | Photo of the `<side>` facade, taken standing on that side of the house (the whole facade if possible), e.g. `photo-south-1.jpg`; `photo-southwest-1.jpg` for a corner shot |
| `photo-detail-<what>.jpg` | Outside details the plans don't make clear: light wells, exterior stairs, a special window |
| `photo-inside-<room>-<what>.jpg` | Inside a room, e.g. `photo-inside-living-room-facing-northeast.jpg`, `photo-inside-upper-bathroom-sauna.jpg`. Useful where the plan is ambiguous (stairs, galleries, roof slopes) and for things that may later become objects of their own in the model (sauna, shower, dining table, a light cabinet). Saying which way the camera faces helps. |
| `plan-changes.txt` (or `.md`) | **What was built differently from the plans**, one change per paragraph: what, where, by how much, and what else moved with it. Example: "The garage was moved 70 cm to the north; the door from the entrance hall to the garage is therefore also 70 cm further north than in the plans." These override the drawings. |
| `room-names.txt` (or `.md`) | **How the plan's room names map to the Loxone room names**, one line per area: `EG Arbeiten = Musikzimmer`; areas without a Loxone room of their own: `DG Ankleide belongs to Schlafzimmer`. Plans often carry old or generic names ("Arbeiten" twice, "Keller 1"); without this file the skill asks for the mapping. |
| `plan-electrical-<n>-<name>.pdf/.jpg` | The electrical plan (Elektroplan) of a storey, numbered like the floor plans: its luminaire symbols show where lamps are, for the light sources (block 8) |
| `photo-lights-<side>-<n>.jpg` | The house at dusk or night with the outside lights on, taken from `<side>`: shows which lamps exist and where they light |
| `notes.md` (or `.txt`) | Other free notes: which plan file is current, anything unusual |

PDFs work for all of them too. Anything else in the folder is read as well; if the skill cannot
tell what it is, it asks.

## What is needed

| Material | Required? |
|---|---|
| Floor plan of every storey | **yes** |
| At least one section | **yes** |
| North direction (north arrow on any plan, the site plan, or a description) | **yes** |
| Elevations, one per side | strongly recommended |
| Facade photos, at least one per side | strongly recommended |
| Site plan | recommended |
| Own top view of the plot with terrace, paths, drive (`plan-topview`) | recommended if the surroundings differ from the plans |
| Manufacturer drawings of a conservatory, carport, canopy (`plan-<what>`) | recommended if present |
| `plan-changes.txt`, if anything was built differently from the plans | strongly recommended |
| `room-names.txt` (plan name → Loxone room) | recommended; otherwise the skill asks |
| Electrical plans, photos of the outside lights (`plan-electrical-…`, `photo-lights-…`) | optional, for the light sources |
| Detail photos, notes | optional |

## What the plans should show

1. **Floor plans** (Grundrisse) of **every** storey, including basement and attic/loft, ideally
   construction or permit drawings at 1:100 or 1:50 (Werkplan, Eingabeplan). Best is the
   architect's **vector PDF** (the dimensions can be read as text); a scan at **300 dpi** or a
   photo works too. The **dimension chains** (Maßketten) must be readable: the skill uses the
   printed dimensions, not the drawing scale. Window dimensions (width/height, sill height "BRH")
   are usually written at the openings.
2. **At least one section** (Schnitt) for the heights: storey heights, slab thickness, ceiling
   heights, knee wall (Kniestock), eave and ridge height, roof pitch.
3. **Elevations** (Ansichten), one per side: sill and top heights of windows, window divisions,
   roof overhangs, terrain line. Without them the skill uses typical values and marks them as
   estimated.
4. **North direction**: a north arrow on any plan or the site plan. If none exists, the user
   describes it (e.g. "the garden side faces south-west").

## Photos

Strongly recommended, but not required. They show what plans often don't: window types (turn,
tilt, sliding, fixed, side panels), roller shutters and venetian blinds, facade colors, terrain,
steps, conservatories, things built differently than planned. Without photos the skill asks more
questions and marks those details as estimated.

- One photo per facade, as straight-on and complete as possible; corner shots (two facades at
  once) are useful too.
- Daylight, shutters open, no strong perspective distortion if avoidable.
- Interior photos (`photo-inside-…`) where the plan is ambiguous, and of fittings and furniture
  that may be shown as objects later; the first version of the skill uses them only for stairs,
  galleries and roof slopes.

## Photos and scans of plans

- Photograph the whole sheet straight from above, flat, evenly lit; include the title block (scale,
  storey).
- Large sheets (A1/A0): the dimension figures must stay legible. Either a high-resolution photo
  (the skill cuts it into tiles with `"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --plan-tiles`) or several close-ups of the same sheet
  named `floor-2-ground-part-1.jpg`, `floor-2-ground-part-2.jpg`, ….
- Hand-written changes on the plans are welcome; list them in `plan-changes.txt` too.

## Also useful later (Loxone mapping)

HouseVis must be connected to the Miniserver once (menu → connect); the skill then reads all
rooms and controls the signed-in user may see itself (`"$LOCALAPPDATA/Programs/HouseVis/HouseVis.exe" --inventory` writes
`loxone-inventory.json`). The skill uses it in the last block to connect rooms, window contacts, shades and
lights. The user should sign in with a user that has permissions for all rooms.
