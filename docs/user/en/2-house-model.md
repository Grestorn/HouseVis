# 2 – Creating the house model

[← Overview](README.md) · [Deutsch](../de/2-hausmodell.md)

> [!TIP]
> **Stuck, or is the model done?** The forum helps under
> [Hausmodell & Pläne](https://forum.housevis.de/c/hausmodell) (house model & plans), and under
> [Haus-Assistent & Skill-Feedback](https://forum.housevis.de/c/skill-feedback) you tell us how it went
> ([more](#help-and-feedback)).

HouseVis needs a model of your house: storeys, rooms, walls, windows and doors, roof, terrace,
garage – and which Loxone controls belong to which room and window. You don't draw it yourself:
**Claude builds it with you from your building plans and photos**, asks about what the plans
leave open, and shows you the result step by step.

## Which way?

| | **A: the house assistant in the app** (recommended) | **B: Claude Code with the HouseVis plugin** |
|---|---|---|
| You need | an API key from Anthropic, billed per use | a **Claude Pro** subscription (or Max) |
| Costs | a whole house from its plans **under $20** (measured on a large, far from simple house); a small change a few cents | your subscription, nothing extra |
| Waiting | none: build the house in one sitting | with Pro, a house hits the **5-hour usage limit** several times; you wait, then continue |
| You see | the house **grows in the 3D view** while Claude builds it | plan drawings and pictures from Claude; HouseVis shows each saved step |
| Where the plans go | attached in the chat | a folder on your PC |

Both use the same knowledge and produce the same files; you can switch between them any time.
**Take way A**, unless you already pay for Claude Pro or Max and would rather wait than pay for
the API.

## 1. Collect your material

Put everything into one folder per house, e.g. `C:\Users\<you>\HouseVis-Projects\plans\my-house\`.

| Material | Needed? |
|---|---|
| Floor plan of every storey (incl. basement) | **yes** |
| At least one section | **yes** |
| North direction (north arrow, site plan, or just tell Claude) | **yes** |
| Elevations (one per side) | strongly recommended |
| Photos of every facade | strongly recommended |
| What was built differently from the plans, as a text file `plan-changes.txt` ("the garage moved 70 cm north") | strongly recommended |
| Site plan (with its spot heights if the plot slopes), your own sketch of terrace and paths | recommended |
| Electrical plan, photos at dusk (for outdoor and special lamps) | optional |
| Photos or drone shots of the roof (for the PV panels) | optional |

- **The architect's PDF set goes in as it is**, under any name. Claude finds out which page shows
  what.
- **Name photos by what they show**: `photo-north-1.jpg` is the north facade (you stand north of
  the house). Inside: `photo-inside-living-room.jpg`.
- Formats: PDF, JPG, PNG; notes as text files (TXT, MD). Plans and photos stay on your PC (see [Privacy](#privacy)).

## 2. Way A: the house assistant in the app (recommended)

### Set up once

1. Sign in at [platform.claude.com](https://platform.claude.com); without an Anthropic account
   you create one there and add a payment method under **Billing**. Create a key under **API
   Keys**; it starts with `sk-ant-`. The app shows each conversation's cost at the top of the
   window, updated after every step.
2. HouseVis: menu → **AI assistant …**. With no Miniserver connected yet, the app asks for it
   first: connected, the assistant reads your Miniserver's rooms and controls to link the house
   with Loxone; **Continue without Miniserver** works too, you can connect later.
3. Paste the key. It is stored encrypted on this PC only.

**Another service instead of Anthropic (e.g. DeepSeek):** tick **Another service with
Anthropic's API** in the same form and enter its address, model and key; for DeepSeek
`https://api.deepseek.com/anthropic` and `deepseek-flash` (its model that can see plans). Not
tested by us; the app shows no cost then, and your house model, messages, plans and photos go to
that service, under its terms (DeepSeek processes them in China).

### Build the house

1. **Attach …** your plans, photos and notes (e.g. `plan-changes.txt`), or drop them onto the
   panel; describe your house in a few words, send.
2. Claude reads the plans, says what is there and what is missing, and asks its questions.
   "I don't know" is a fine answer; Claude then estimates.
3. Claude builds the model step by step; the 3D view follows every change. **Save** writes it;
   until then **Undo** and **Discard** take changes back.

The panel shows the cost of the conversation so far. A small change costs cents; building a
large, complex house from its full plan set cost less than $20, a simpler house costs less. For a
hard cap, set a monthly spend limit in the Console.

### Connect to Loxone

1. HouseVis must be connected to your Miniserver ([chapter 1](1-install.md)).
2. Ask: *"Map the rooms, windows and lights to Loxone."* Claude reads the controls of your
   Miniserver (read only) and suggests which one belongs where; the live values show in the 3D
   view before you save. Confirm or correct. Windows may come from their own contacts or from one
   **Fenster- und Türüberwachung** (window and door monitor) for all of them.
3. On request Claude also places **lamps** (path lights, wall lights, a pendant over the table),
   **furniture** as simple boxes (kitchen row, wall cupboards, wardrobe; light strips sit on their
   edges, e.g. *"a light strip under the wall cupboards"*) and your **weather station**.
4. **Indicators** show a state as an icon: music playing or paused, the washing machine running,
   post in the mailbox. One device gets one indicator with an icon per state (*"show the mailbox:
   a letter for post, a parcel for a parcel"*). It either sits in its room's row, beside the
   other icons (music …), or floats at a place of its own (*"show the mailbox at the front door"*,
   *"the stove above the hob"*); for a device in a room Claude asks which you prefer.
5. An indicator can also show a **value** from Loxone with its unit, e.g. *"show the pond's
   temperature in the garden"* or *"the heat pump's power"*: in the format Loxone gives for the
   value (e.g. "21.4°", "55%"), temperatures in their colour. For the value alone without an icon,
   set its kind's icon to "Do not show" under **Global settings …**. Values that belong to no
   place (the whole plant's PV power, the day's energy) go on a page as a [data panel](3-using-the-app.md#data-panels).
   Claude draws **PV panels** on the roof from roof photos or drone shots (*"the PV panels on the
   south roof, 5 × 2"*), one array per block of panels. If Loxone measures the arrays separately
   (a meter per roof side), an indicator above each array shows what it delivers right now
   (*"show each PV array's power"*). Views that cut the roof open cut the panels too; only their
   outline stays there.
6. A room's **humidity** is usually known to its room controller, otherwise its Touch switch
   (T5) measures it; as with the temperature, Claude takes the room controller first. Say *"show the humidity
   with the room temperatures"*, and it appears in the small line below the temperature, beside
   the target temperature: "→| 21.0° 💧 48%". Change the two icons (arrow to a line, droplet) under
   **Global settings …** ("Heating: target temperature", "Humidity"), or set them to "Do not
   show" for the bare numbers.

### Later changes

Open the house assistant and say what is wrong, e.g. *"The bathroom window is 20 cm too far
right"* or *"The garage has a flat roof."* Claude changes only that. A roof that an older
model built from several pieces (say a cross gable from overlapping parts) Claude merges into
one on request: *"Make the main roof and the cross gable one roof."* A bay is part of its room
and gets its own small roof: *"The kitchen has a bay to the north, 1 m deep and 2.3 m wide, with a
shed roof against the wall."* Dormers too: *"On the south slope a shed dormer, front 1.6 m wide
from 7.1 m, 1 m in from the eave, front wall 2.4 m high, with a two-leaf window."* A room open
through two storeys (a Luftraum over the living room, a two-storey hall) is one room reaching up,
its windows as tall as in the elevation: *"The living room is open up to the upper floor, with a
window in the south wall from 0.5 m to 4.8 m."* Doors between rooms, open passages and glass
walls work the same way: *"A glass door from the hall to the living room, 90 cm wide"*, *"The
sauna has a frameless glass front with a glass door towards the bathroom."* So do stairs and
railings: *"A stair in the hall up to the gallery, 16 steps, turning left on a landing"*, *"A glass
railing along the gallery"* (it takes the place of the wall that stood there), *"Bars on the
stair's open side"*. A room a few steps down: *"The living room lies two steps lower than the
hall."* A sloping plot follows the site plan's spot heights (*"The garden falls 1.30 m to the
south"*), with a terrace or a seat cut into the slope; terraces, paths, light wells and
conservatories may have any shape, also at an angle. Left of the assistant you
see the house with every change: at the top choose one of your views, and while a new room is open
in no view yet also **Whole house** (every room open); drag to turn, the wheel zooms, **Back to the
view** resets the camera. Nothing of this is saved to the views. Pictures Claude looks at (plan
details, rendered views) appear in the conversation under their line; a click enlarges them. On the
first save of a new house the app creates three views with a page each: **All rooms** (every room
open, with values), **Exterior** (the house closed) and **Glass model** (every room open, without
values), and a page **Loxone** with the Loxone interface.

## 3. Way B: Claude Code

For you if you already have a Claude Pro or Max subscription and don't want to pay for the API
on top.

> [!IMPORTANT]
> **A Pro subscription is the minimum.** With the free plan the usage limit is far too small for
> a house. Even with Pro, building a house will hit the 5-hour usage limit several times: Claude
> stops and tells you when you can continue. **Just wait** and carry on then – nothing is lost,
> Claude keeps notes and continues where it stopped. If you don't want to wait, use way A.

### Set up once

1. Install **Claude Code**: the easiest is the **Claude desktop app**, tab **Code**. (Also
   possible: the terminal, see the [Claude Code quickstart](https://code.claude.com/docs/en/quickstart),
   or VS Code.) Sign in with the account that has the subscription.
2. Install HouseVis on the same PC ([chapter 1](1-install.md)). Claude uses it to check and
   render the model.
3. Install the **HouseVis plugin** in Claude Code. These are **two** commands, both are needed:

   ```
   /plugin marketplace add Grestorn/HouseVis
   /plugin install house-model@housevis
   ```

   The first only tells Claude Code where the plugin is; the second installs it. Check with
   `/plugin` that **house-model** is listed as installed, then start a new session.

   **Updates:** Claude Code does not update the plugin by itself. After installing a new HouseVis
   version, also run `/plugin marketplace update housevis` and start a new session. If you forget,
   Claude notices that the plugin and HouseVis have different versions and reminds you.

### Build the house

1. Open Claude Code on the folder that holds `plans\` (e.g. `HouseVis-Projects`).
2. Write, in your own language: *"Create the house model from plans/my-house."*
3. Claude first asks **where the model goes**. Normally into the folder HouseVis shows
   (`Documents\HouseVis`), so you see the house in the app right away. If a house is already
   there, Claude never overwrites it and suggests another folder; then open that one in HouseVis
   (menu → **Configuration folder …**).
4. Claude reads the plans, lists what is there and what is missing, and asks its questions – all
   at once. Answer them; "I don't know" is a fine answer, Claude then estimates and notes it.
5. Claude builds the house **block by block**: storeys and rooms, walls, windows and doors, roof,
   surroundings. After each block it shows you plan drawings or 3D pictures and **waits for your
   OK**. Look closely – mistakes in the rooms carry into everything after them.
6. HouseVis shows every saved change at once. Keep it open next to Claude.

Took a break or hit the usage limit? Just say *"Continue with plans/my-house."* Claude keeps its
notes in `plans/my-house/model-notes.md`.

### Connect to Loxone

1. HouseVis must be connected to your Miniserver ([chapter 1](1-install.md)).
2. Ask Claude: *"Map the rooms, windows and lights to Loxone."* Claude reads all controls of
   your Miniserver (read only) and suggests which control belongs where. Confirm or correct.
3. On request Claude also places **lamps** and your **weather station**.

### Later changes

Say what is wrong, e.g. *"The bathroom window in plans/my-house is 20 cm too far right."* Claude
changes only that and shows you the result. *"Show me the pages with the live values"* gives
pictures of your pages as the app shows them right now.

### Other AI agents (Codex, Gemini CLI, Mistral Vibe)

The same skill is also there as a plain skill folder for agents that read skills
(`SKILL.md`): **OpenAI Codex**, **Gemini CLI**, **Mistral Vibe**. **Not tested by us:** the skill
was built and measured with Claude, and how well another model reads building plans is up to
that model. It must be able to look at pictures; if it cannot, it says so.

1. Install HouseVis on the same PC ([chapter 1](1-install.md)).
2. Download [this repository](https://github.com/Grestorn/HouseVis) (**Code → Download ZIP**) and
   copy its folder `skills\house-model` into the skills folder of your agent:

   | Agent | Skills folder |
   |---|---|
   | OpenAI Codex | `%USERPROFILE%\.codex\skills\` |
   | Gemini CLI | `%USERPROFILE%\.gemini\skills\` |
   | Mistral Vibe | `%USERPROFILE%\.vibe\skills\` |

3. Start the agent in the folder that holds `plans\` and go on as under
   [Build the house](#build-the-house-1) above.

**Updates:** after a new HouseVis version, copy the folder again. The skill notices a different
version and tells you. **Privacy:** your plans and photos then go to that agent's provider, under
its terms.

## 4. After the model: views and pages

The model alone shows nothing yet – a **view** decides from where you look and which rooms are
open. Click **Create views** on the start screen, then see [chapter 3](3-using-the-app.md).

## Help and feedback

The forum [forum.housevis.de](https://forum.housevis.de) is in German.

- **Stuck?** Ask under [Hausmodell & Pläne](https://forum.housevis.de/c/hausmodell). Do **not post complete building
  plans** there, only the part in question, and black out address and names in the title block.
- **Tell us how it went:** under [Haus-Assistent & Skill-Feedback](https://forum.housevis.de/c/skill-feedback) we
  collect what was right at once, where Claude got it wrong, what it cost and how long it took.
  That makes the skill better for everyone.
- **Show your house** under [Eure Häuser](https://forum.housevis.de/c/eure-haeuser) (your houses), without address or
  names in the pictures.

## Privacy

- Way A sends the model, your messages and the parts of plans and photos Claude looks at to
  Anthropic; way B sends what Claude Code reads (plans, photos, files in your working folder)
  under your subscription. HouseVis itself has no server and sends nothing anywhere else.
- **Way A (API):** Anthropic does not train its models on data sent through the API and deletes
  it after 30 days. There is nothing to set.
- **Way B (Pro or Max subscription):** whether your sessions are used for training is up to you,
  in [claude.ai's privacy settings](https://claude.ai/settings/data-privacy-controls) (the model
  improvement switch). Off, they are kept for 30 days; on, for up to 5 years. Check the setting before
  you hand over your plans.
- **Ratings can always go into training:** thumbs up/down or `/feedback` in Claude Code send the
  whole conversation along, plans included; Anthropic keeps it for up to 5 years and may train on
  it, whatever the setting says. Tell us how it went in the [forum](#help-and-feedback) instead.
- Plans are personal data and usually copyrighted by your architect: don't put them into public
  repositories or shared folders.
- As of November 2026; Anthropic's [privacy pages](https://privacy.claude.com) are what counts.

Next: [3 – Using the app](3-using-the-app.md)
