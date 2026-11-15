# Version history

[← Overview](README.md) · [Deutsch](../de/versionen.md)

What changed for you in each version. Update the plugin together with the app (`/plugin marketplace update
housevis`, [chapter 2](2-house-model.md)).

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
