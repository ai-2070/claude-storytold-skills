# storytold skills for Claude Code

A [Claude Code](https://code.claude.com) plugin for the [storytold Crafting Apps](https://github.com/storytold):
twelve free, open-source creative tools written in Rust. Each app ships a desktop app, a CLI and an MCP server.

The `storytold` plugin gives Claude three things:

- **Picking the right tool** (`storytold`): a router that maps a creative task to the right app.
- **Install and wiring** (`storytold-install`): download the apps, put them on PATH, check that each MCP server
  answers, and register it with Claude Code.
- **Knowing how to use it** (one skill per app): the app's mental model, MCP tools, tested recipes, formats,
  CLI one-shots, pitfalls and how to verify the result.

## Quick start

```bash
claude plugin install storytold --marketplace ai-2070/claude-storytold-skills
```

Or inside Claude Code:

```text
/plugin marketplace add ai-2070/claude-storytold-skills
/plugin install storytold@storytold-skills
```

Then ask Claude:

> Set up the storytold Crafting Apps.

The `storytold-install` skill downloads the apps, puts them on PATH, checks that their MCP servers answer, and
registers them at user scope with `claude mcp add`. **Restart Claude Code** so the MCP tools load. They appear
as `mcp__<app>__<tool>`, and `/mcp` shows their status. From then on, Claude picks the right app for each
creative task.

## Skills

| Skill | App | For |
|---|---|---|
| [`storytold`](plugins/storytold/skills/storytold/SKILL.md) | (router) | Choosing the app for a creative task |
| [`storytold-install`](plugins/storytold/skills/storytold-install/SKILL.md) | (all) | Install, PATH, MCP check, Claude Code registration |
| [`vectorcraft`](plugins/storytold/skills/vectorcraft/SKILL.md) | VectorCraft (Illustrator-style) | Logos, icons, posters, illustration, SVG / PDF / EPS |
| [`photocraft`](plugins/storytold/skills/photocraft/SKILL.md) | PhotoCraft (Photoshop-style) | Photo editing, retouching, compositing, PSD |
| [`lightcraft`](plugins/storytold/skills/lightcraft/SKILL.md) | LightCraft (Lightroom-style) | RAW development, culling, presets, batch export |
| [`effectcraft`](plugins/storytold/skills/effectcraft/SKILL.md) | EffectCraft (After Effects-style) | Motion graphics, animated titles, lower thirds |
| [`filmcraft`](plugins/storytold/skills/filmcraft/SKILL.md) | FilmCraft (Premiere Pro-style) | Video editing, transitions, titles, captions, MP4 |
| [`designcraft`](plugins/storytold/skills/designcraft/SKILL.md) | DesignCraft (InDesign-style) | Brochures, newsletters, multi-page layout, print PDF |
| [`pdfcraft`](plugins/storytold/skills/pdfcraft/SKILL.md) | PdfCraft (Acrobat-style) | Merge/split, annotate, forms, redact, sign PDFs |
| [`wordcraft`](plugins/storytold/skills/wordcraft/SKILL.md) | WordCraft (Word-style) | Letters, reports, .docx |
| [`gridcraft`](plugins/storytold/skills/gridcraft/SKILL.md) | GridCraft (Excel-style) | Spreadsheets, formulas, charts, .xlsx / .csv |
| [`deckcraft`](plugins/storytold/skills/deckcraft/SKILL.md) | DeckCraft (PowerPoint-style) | Slide decks, .pptx |
| [`soundcraft`](plugins/storytold/skills/soundcraft/SKILL.md) | SoundCraft (Pro Tools-style) | Podcast editing, mixing, mastering, sound design |
| [`cadcraft`](plugins/storytold/skills/cadcraft/SKILL.md) | CADCraft (AutoCAD-style) | Floor plans, technical drawings, DXF / DWG |

Claude loads a skill when a task matches its description. You can also invoke one directly, e.g.
`/storytold:vectorcraft`. Every session pays about 1.9k tokens for the skill list. A skill's full instructions
(about 1.4k–7.7k tokens) load only when that skill is used. Each app skill has a `references/` folder with the
app's full command, effect or function catalog, generated from the live MCP server. Claude greps those files
rather than reading them whole.

## Install the apps yourself

The helper needs Python 3.9+ and nothing else:

```bash
python plugins/storytold/skills/storytold-install/scripts/storytold_tools.py setup            # all twelve apps
python plugins/storytold/skills/storytold-install/scripts/storytold_tools.py setup vectorcraft photocraft
```

`setup` does four things:

1. **Install:** downloads each app's latest GitHub release for this OS and CPU, checks it against the release's
   `SHA256SUMS.txt`, and unpacks it.
2. **PATH:** links `<app>` and `<app>-cli` into one bin directory and adds it to the persistent PATH if it isn't
   there yet.
3. **Verify:** runs an MCP handshake (`initialize` + `tools/list`) against every server.
4. **Register:** runs `claude mcp add --scope user <app> -- <abs path>/<app>-cli mcp …`. A name that's already
   registered is left alone unless you pass `register --replace`.

| | Windows | Linux / FreeBSD | macOS |
|---|---|---|---|
| Apps | `%LOCALAPPDATA%\storytold\apps\` | `~/.local/share/storytold/apps/` | `~/Library/Application Support/storytold/apps/` (CLI only) |
| On PATH | `%LOCALAPPDATA%\storytold\bin` | `~/.local/bin` | `~/.local/bin` |

PhotoCraft is registered with file-access roots (your home folder), because without them it can't open or save
files. LightCraft gets a persistent library, because otherwise its catalog lives in memory only. See
[`storytold-install/SKILL.md`](plugins/storytold/skills/storytold-install/SKILL.md) for the other subcommands
(`status`, `install`, `path`, `verify`, `register`) and the environment variables that change locations.

## How the skills were tested

Every recipe marked **(tested)** was run against the released apps through their MCP servers or CLIs. The
outputs were checked by rendering and looking at them, by reading values back, or by measuring them with
ffprobe or ffmpeg. The versions tested were:

vectorcraft 0.7.0 · photocraft 0.5.0 · lightcraft 0.4.0 · effectcraft 0.6.0 · filmcraft 0.4.0 · designcraft 0.4.0 ·
pdfcraft 0.4.0 · wordcraft 0.3.0 · gridcraft 0.3.0 · deckcraft 0.3.0 · soundcraft 0.3.0 · cadcraft 0.3.0

Bugs and quirks found along the way, with their exact error text, are in each skill's **Pitfalls** section.
Each skill also says what wasn't tested.

A Hermes Agent version of the same skills lives at
[hermes-pro/storytold-skills](https://github.com/hermes-pro/storytold-skills).

## Repository layout

```
.claude-plugin/marketplace.json        # the "storytold-skills" marketplace
plugins/storytold/
├── .claude-plugin/plugin.json         # the "storytold" plugin
└── skills/<skill>/
    ├── SKILL.md                       # name + description frontmatter, instructions
    ├── references/                    # generated catalogs (app skills)
    └── scripts/                       # helper script (storytold-install)
```

Check changes with `claude plugin validate --strict . && claude plugin validate --strict ./plugins/storytold`.

## License

MIT. See [LICENSE](LICENSE). The Crafting Apps themselves are licensed separately by their own repositories.
