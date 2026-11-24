# HouseVis

A 3D view of your house with live values from a Loxone Miniserver: room temperatures and heating
modes, light including its colour, presence, open windows and doors, shading, and indicators such
as music playing. The house is cut open per view, so you look into the rooms.

> [!IMPORTANT]
> ### 💬 The HouseVis forum: [forum.housevis.de](https://forum.housevis.de) (in German)
>
> Ask questions, get help with your house model, report bugs, suggest ideas, follow the news.
>
> [Help](https://forum.housevis.de/c/hilfe) · [House model & plans](https://forum.housevis.de/c/hausmodell) ·
> [Feedback on the house assistant](https://forum.housevis.de/c/skill-feedback) · [Report a bug](https://forum.housevis.de/c/fehlerberichte) ·
> [Ideas](https://forum.housevis.de/c/ideen)

- **Windows app**: the full app, with editors for views, devices and pages, and the house
  assistant.
- **Android app**, for wall tablets: shows a configuration exported from the Windows app.

**Downloads: [www.housevis.de/download](https://www.housevis.de/download/)** (in German). The
[website](https://www.housevis.de) also has a demo in the browser. This repository holds the guide and
the house-model skill.

## User guide

Installation, the house model, using the app, troubleshooting:
**[Deutsch](docs/user/de/README.md)** · **[English](docs/user/en/README.md)**. What is new in each
version: [Versionen](docs/user/de/versionen.md) · [Version history](docs/user/en/versions.md).

## Your house model

HouseVis needs a model of your house and its mapping to Loxone. Claude builds both with you from
your architect's plans and photos:

- **A – the house assistant in the app** (recommended): needs an Anthropic API key; under $20 for
  a whole house.
- **B – Claude Code with the HouseVis plugin**, if you already pay for a Claude Pro or Max
  subscription. This repository is the plugin's marketplace; in Claude Code (two commands, both
  are needed):

  ```
  /plugin marketplace add Grestorn/HouseVis
  /plugin install house-model@housevis
  ```

- **Other AI agents** (OpenAI Codex, Gemini CLI, Mistral Vibe): the same skill as a plain skill
  folder, [`skills/house-model`](skills/house-model/SKILL.md); not tested by us. Install HouseVis,
  copy that folder into your agent's skills folder (`%USERPROFILE%\.codex\skills\`,
  `%USERPROFILE%\.gemini\skills\` or `%USERPROFILE%\.vibe\skills\`) and start the agent in the
  folder that holds your `plans\`; no need to clone this repository.

To see what material works best, or to try it before you gather your own: a complete made-up
example, [`examples/haus-sonnenhang`](examples/haus-sonnenhang/README.md) (plan set, a maker's
drawing, room names, changes).

Details on these ways, costs and what happens with your data: chapter 2 of the guide
([Deutsch](docs/user/de/2-hausmodell.md) · [English](docs/user/en/2-house-model.md)).

## Licence

HouseVis is free to use but not open source: see [LICENSE](LICENSE). It may not be redistributed,
modified, reused in part or recreated (also not with AI tools) without permission.
