# Version history

[← Overview](README.md) · [Deutsch](../de/versionen.md)

What changed for you in each version. Update the plugin together with the app (`/plugin marketplace update
housevis`, [chapter 2](2-house-model.md)).

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
