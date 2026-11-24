# Version history

[← Overview](README.md) · [Deutsch](../de/versionen.md)

What changed for you in each version. Update the plugin together with the app (`/plugin marketplace update
housevis`, [chapter 2](2-house-model.md)).

## 1.0.0

The first version for everyone: the downloads are now freely available on
[www.housevis.de](https://www.housevis.de/download/), and the Windows installer and app are signed
(publisher "Martin Korndörfer"). The Android app is in testing on Google Play: sign up on the
download page. The tablet app offers the demo house on its first start and, like the Windows
app, under **Playbacks …** at any time. New since the beta 0.9.5:

**Important for the tablet:** a configuration with a dormer, a half-hip roof, a room open
through two storeys, an interior door, stairs, railings, a room a few steps down, a sloped terrain or a shape at an angle outside the house does not load in an app from before 1.0.0. Update the Android app together with the Windows app
before you export such a configuration.

**Rooms**
- **Interior doors, passages and glass walls:** doors between rooms, open passages without a door,
  glass partitions and glass doors, also frameless (a sauna's glass front), shown in both rooms'
  walls when either is cut open; the skill reads them from the floor plans.
- **Double-height rooms:** a room can be open through the storeys above (a Luftraum over the
  living room, a two-storey hall or bay): no floor above it, its walls up to the ceiling or roof
  of the highest storey, windows as tall as the elevation shows. A view showing only the lower
  storey shows it as an ordinary room; its indicators stay at the usual height. The model check
  crosses it out on the upper plan; the skill and the assistant know it.
- **Stairs:** stairs between storeys, written by their walking line as the floor plan draws it
  (straight, L or U, turning on a landing or on winders), shown as solid steps when their room
  is opened, with their opening cut into the storey above. The model check prints rises and
  goings like a plan's stair label and draws the stair; the skill and the assistant know it.
- **Railings:** glass, bars or a solid parapet, along a gallery's edge, round a balcony or beside
  a stair (following its slope). A gallery's railing opens the wall that stood at its edge, so the
  hall below shows through.
- **Split levels:** a room a few steps down or up (a sunken living room) with its steps; its
  ceiling stays where it is. On a slope the model check lists the walls in the ground and names
  windows or doors under the terrain without a light well, for a basement opening to the garden.
- **Outside at any angle:** terraces, paths, light wells, steps, canopies and conservatories can
  have any shape, not only rectangles, and the plot's edge may run at an angle; the terrain meets
  a turned garden house cleanly.

**Roof**
- **Half-hip roofs (Krüppelwalm):** a gable roof whose gable walls end below the ridge in a small
  hip, at one or both ends; the skill and the assistant know it.
- **Dormers:** shed and gable dormers (Schleppgaube, Giebelgaube) on a roof slope, written by their
  front; the roof is cut away under them, the room below gets their ceiling, the windows in their
  front are ordinary windows with datapoints and shades. Their roof and walls can have colours of
  their own (slate or zinc cheeks). A view's setting for the roof covers its dormers.
- **Bays:** a bay stepping out of a gable end no longer takes the gable out with it (the gable
  stayed open); the model check warns about a bay standing under the eave, where the roof is lower
  than its walls. The skill and the assistant know how to model bays and dormers.

**Surroundings**
- **Sloped plots:** the terrain can follow the site plan's spot heights instead of one level;
  terraces, drives or a seating area can be cut into the slope as flat platforms, with their
  retaining edges shown. Light wells, garden lights and the weather station's mast stand on the
  slope. The model check draws the spot heights in the plan; the skill and the assistant know it.

**Cameras and tablet**
- The still pictures beside the house no longer make the tablet app grow until Android closes it
  (a picture every 5 s used up about 57 MB a minute).
- Camera sign-ins stored with the first camera builds keep working (all cameras showed as signed
  out after an update).
- The **Loxone page stays signed in:** sign in once directly on it, and HouseVis signs it in with
  a token from then on – without a password and without the Miniserver's login mail.
- The **Loxone Intercom** needs no sign-in of its own any more: HouseVis fetches its picture with
  its own sign-in to the Miniserver, without a password. Since Loxone 17.3 the Miniserver otherwise
  sent a login mail for every picture. A stored intercom password is deleted.
- While the screen is dimmed for nobody being there, the app pauses: no redrawing, no weather
  animation, no camera pictures; it carries on at a touch.

**Demo house and assistant**
- The demo house is now a believable family house: a T-shaped plan with a basement opening onto
  the garden, an open stair, a gallery, a living room two steps down, a conservatory, a roof
  terrace over the garage and a pool. The demonstrations and pictures show the new house.
- **Playbacks …** is tidied up: without "Look around the demo house" (still on the start screen)
  and the fly-around; in the weather demo a click on the house shows the next weather type.
- The skill and the assistant keep the model calm: furniture, extra lights, stairs, railings and
  small parts only where you want them; they offer them rather than just adding them. Each such
  part gets a tag of its own ("Sofa Wohnzimmer"), so a view can hide it alone or with all of its
  kind.
- The house assistant shows the pictures Claude looks at (plan details, rendered views) in the
  conversation; its panel is wider.
- A new house gets three views with a page each on its first save: All rooms, Exterior and Glass
  model (without values), and a page with the Loxone interface.

**Loxone page**
- **Loxone page on doorbell** (device setting, on from the start): when a Loxone Intercom rings, the device shows its
  Loxone page with the door dialog and wakes a dimmed screen; after the ring it goes back to the
  page before.
- A new device in the device editor no longer starts empty: it has a page showing the house and a
  Loxone page.
- Saving in **Loxone UI sign-in …** on a device without a Loxone page adds one (Windows); the
  sign-in is tried there at once.

## 0.9.5

**Important for the tablet:** a configuration with a camera, PV panels, a roof with an L, T or U
outline, room humidity or a value on an indicator does not load in an app from before 0.9.5.
Update the Android app together with the Windows app before you export such a configuration.

**Cameras**
- **Camera pictures on a page:** beside the house a still picture renewed every few seconds, on a
  page of cameras the live picture. Reolink cameras (still and live picture), the **Loxone
  Intercom** (2nd generation) through the Miniserver, other cameras with a still picture. Set up
  under menu → **Cameras …**, put on a page with the device editor
  ([chapter 3](3-using-the-app.md#cameras)). The demo house has a camera to try.

**House model (from the beta feedback)**
- **PV panels** on the roof: Claude draws them from roof photos or drone shots; an indicator above
  each array can show what it delivers right now ([chapter 2](2-house-model.md)).
- **Roofs with an L, T or U outline** are one roof part now, with hips and valleys where they
  belong; Claude merges an older model's roof pieces into one on request.
- **Room humidity** in the small line below the temperature, beside the target temperature, both
  with an icon you can change or switch off under **Global settings …**.
- **Values on indicators:** an indicator can show a Loxone value with its unit, e.g. the pond's
  temperature or the heat pump's power.
- **All 2,030 Lucide icons** in the icon picker: the house groups first, the rest under **More
  groups …**.
- The AI assistant and the skill place labels and indicators in a view by looking at the rendered
  page.
- The model check no longer warns about the inner end of a cross gable that runs into the main
  roof.

**Fixes**
- Windows: **Alt** no longer brings up a menu bar (Alt+click in the view editor works again).
- View editor: walls an inactive room draws behind an opening (the side wall behind an open garage
  gate) can be clicked.
- Device editor: copying all pages onto a device with one orientation keeps that orientation's
  pages.
- Formats with a width (`%5.1f`, `%02d`) are shown formatted.
- Android: text fields in dialogs stay above the on-screen keyboard.

## 0.9.4

**Important for the tablet:** a configuration with a data panel or an in-ground spot does not
load in an app from before 0.9.4. Update the Android app together with the Windows app before you
export such a configuration.

**Pages and devices**
- **Data panels:** any Loxone value on a page (PV power, grid, battery …), each with icon, name and
  the format Loxone gives it; picked in the device editor from the list of the Miniserver's values,
  filtered by room and category and searchable ([chapter 3](3-using-the-app.md#data-panels)).
- **Device editor tidied up:** devices, pages and panels in three columns; **+** opens a choice
  (HouseVis or Loxone page; view, clock, date or data panel); panels fold open and shut; the panel
  list is the order from front to back, arrows move one forward or back. **Separate
  orientations** is now a setting of the device.
- **AI assistant with DeepSeek:** besides an Anthropic key, the assistant takes another service
  with Anthropic's API, e.g. DeepSeek (`deepseek-flash`, its model that can see plans): tick the
  box in the key form, enter address, model and key. Not tested by us; no cost is shown then.
- **The skill for Codex, Gemini CLI and Mistral Vibe:** the house-model skill also comes as a
  plain skill folder for these agents ([chapter 2](2-house-model.md#other-ai-agents-codex-gemini-cli-mistral-vibe));
  not tested by us.
- Its commands are written for PowerShell, the shell these agents use on Windows; a `plans\`
  folder in a clone of this repository stays out of Git.

**From the first feedback**
- **In-ground spots** as a light kind of their own: set into the ground or a terrace, shining up
  (facade, pillar, tree).
- **Rooms without datapoints** are named by the model check; when the assistant is connected to
  the Miniserver (or the skill finds `loxone-inventory.json`), it warns about each one whose Loxone
  room has lights, a room controller or presence: a room that stays dark because its mapping got
  lost shows up at once.
- **AI assistant:** error messages say how to fix the problem (e.g. the lower-case id, `lightColor`
  as a list for several light controllers in a room); a change that removes datapoints says so at
  once. The test pictures no longer fail so easily at the start ("Starting the capture window did
  not finish"): more time and a second try.
- The skill asks right away which side photos without one in their name show, checks after a
  refused change that everything arrived, and combines several light controllers of a room.
- **Robot mowers, vacuum robots and cameras**: the skill offers them as an icon at their place
  (charging station, gate) showing their state: mowing, charging, error, doorbell.

## 0.9.3

**Important for the tablet:** a configuration using the new icons (from another set, with a
colour, or a page without an icon) does not load in an app from before 0.9.3. Update the Android
app together with the Windows app before you export such a configuration. Unchanged
configurations keep loading in older apps too.

**Viewing the house**
- **Playbacks …** (menu): the demo house's clips from the forum play in the app, plus your house's
  own, made by the AI assistant, with made-up or the current values; "Continuous loop" keeps one
  running. Replaces Demo, Weather demo and Showcase in the menu.
- **Room names on the floor:** views looking down on a storey put names on the floor by
  themselves where no wall shows them well; in the view editor a name can be dragged onto the
  floor and back.
- **Sloped ceilings** appear only where they are seen from inside; from above the slope no longer
  jumps from one side to the other.
- Windows and doors in solid exterior walls no longer get a shifted second frame.
- Windows and shades move equally fast at any frame rate (they were too slow on the tablet).

**Pages and settings**
- **Icons from three sets** for pages and indicators: Fluent Emoji in colour or one-colour,
  Lucide; one-colour ones in a colour of your choice; a page can do without an icon. One picker
  with groups and search for both.
- Device editor: **copy pages to another device** (right click on a page, or on a device for all
  its pages); per view on a page **hide temperatures and indicators** or change their **size**
  (30–150 %); the preview also fits portrait pages.
- **Global settings** in two pages (Options, Symbols); new: the **size of the Loxone page**
  (default 90 %), so that e.g. the Loxone app's Ambient mode has room.
- Menu in groups; "House assistant …" is now **AI assistant …**.
- **Export device …** suggests the last export's folder.

**Android tablet**
- **Full screen:** status bar and navigation buttons are hidden (a swipe from the edge shows them
  for a moment).
- **The loaded configuration file is watched:** when it changes (e.g. through a sync tool such as
  QFile Pro), the app reads it in again by itself.
  "Loading the configuration …" covers the loading.
- **Settings …** (menu): language and size of the Loxone page for this tablet only.
- Loxone UI sign-in: after saving, the Loxone page signs in at once; the password can be shown
  before it is sent.

## 0.9.2

**House model**
- **A gable roof built from two shed roofs** (the workaround for unequal overhangs before 0.9.1):
  its gables are closed again and the "gable stays open" message is gone; the skill offers to
  merge them into one gable roof. Where the message still appears, it says so: ask the skill to
  merge the two.

## 0.9.1

**House model**
- **Roofs as the plans dimension them:** different overhangs per side, eave height and pitch per
  side. Your `house.json` is converted on the first start; the old file stays beside it
  (`house.v6.json`), and older export files are still read.
- **Furniture** as simple boxes (kitchen row, cupboards, wardrobe), light strips on their edges.
- **Light strips** that rise or stand upright, also sloped along a stair; **tube pendants**.
- **Indicators at a place of their own**, e.g. the mailbox at the front door; per view movable in
  the view editor.
- **Window contacts from a "Fenster- und Türüberwachung"** (WindowMonitor), one block for many
  windows.

**Seeing the house**
- **The roof opens by itself over active rooms** where it is their ceiling (rooms under a low
  roof), down through the eave; a click on the roof makes it transparent, hidden or fully solid.
  No walls stand in the attic any more; gables under a transparent roof are transparent too.
- **Clock and date** on a page, placed like a view.
- The **badge** (top left) shows only when the values are **not** live.
- Light strips glare less in realistic mode; clouds no longer jump when the wind changes.
- Pits can be left out of a view; the ground then closes over them.

**Windows app**
- **Loxone UI sign-in …** (menu): the Loxone page can remember its login (optional, encrypted).
  On Android the password manager can fill it in.
- Device editor: the preview shows the real views; a copied view can get a page on every device.
- House assistant: the house beside the panel, with a choice of views and free to turn.

**Claude Code plugin**
- Pictures of every page (`--render`), of one device or page, also with the **live values** of
  your Miniserver.
- The plugin checks that it matches the installed HouseVis and says what to update.
- The skill learned from the first houses built with it.

## 0.9.0

The first public beta: Windows app with the view, device and global-settings editors, the house
assistant, the Android app for wall tablets, the demo house, and the house-model plugin for
Claude Code.
