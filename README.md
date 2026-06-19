# rhino-dev-skills

> A Claude Code plugin that gives Claude complete, hands-on knowledge for building **Rhino & Grasshopper plugins** with RhinoCommon (.NET / C#).

### Why this exists

Developing Rhino plugins means juggling RhinoCommon APIs, Grasshopper data trees, Eto.Forms quirks, and Yak packaging details all at once. Looking up the right class, method signature, or pattern mid-flow breaks concentration and slows everything down.

This plugin embeds that domain knowledge directly into Claude Code. Once installed, Claude automatically has the right context — correct API usage, common pitfalls, working patterns — so you can **focus on what you're building, not on hunting through documentation**. The goals are simple:

- **Faster development** — less time looking things up, more time writing code
- **Fewer errors** — Claude knows the gotchas (e.g. DataTree access patterns, Eto thread safety, CommitChanges requirements) and applies them proactively
- **Lower barrier to entry** — useful whether you're new to Rhino plugin development or an experienced developer who just wants quicker recall

It bundles five domain skills covering the full plugin-development lifecycle — geometry, the document model, Grasshopper components, cross-platform UI, project setup and packaging. Once installed, Claude automatically pulls in the right skill whenever you work on Rhino/Grasshopper code.

## Requirements

- [Claude Code](https://docs.claude.com/en/docs/claude-code) (CLI, desktop, or IDE extension)
- Git — only needed for the marketplace / clone install methods

## Installation

### Option A — Install as a Claude Code marketplace (recommended)

In any Claude Code session:

```
/plugin marketplace add bcshih/rhino-dev-skills
/plugin install rhino-dev-skills@rhino-dev-skills
```

Run `/plugin` to confirm `rhino-dev-skills` is listed and enabled.

### Option B — Local install (clone first)

```bash
git clone https://github.com/bcshih/rhino-dev-skills.git
```

Then point Claude Code at the local folder:

```
/plugin marketplace add /absolute/path/to/rhino-dev-skills
/plugin install rhino-dev-skills@rhino-dev-skills
```

### Option C — Drop the skills in manually

Copy the subfolders of `skills/` into your personal skills folder:

- **Windows:** `C:\Users\<you>\.claude\skills\`
- **macOS / Linux:** `~/.claude/skills/`

Each subfolder (e.g. `rhinocommon-setup/`) becomes an available skill on the next session.

## 中文快速安裝

這是一個 **Claude Code 插件**，內含 Rhino/Grasshopper 開發的五個技能。在 Claude Code 中輸入：

```
/plugin marketplace add bcshih/rhino-dev-skills
/plugin install rhino-dev-skills@rhino-dev-skills
```

輸入 `/plugin` 確認 `rhino-dev-skills` 已啟用，重開 Claude Code 即可。之後只要在處理 Rhino/Grasshopper 的 C# 程式碼，Claude 會自動載入對應技能，不需手動呼叫。

## Skills

| Skill | Trigger | Covers |
|-------|---------|--------|
| `rhinocommon-setup` | Project setup, .csproj, debug, Yak | Plugin class, Command class, multi-targeting (.NET 4.8 / .NET 7), VS2022 debug config, Yak packaging |
| `rhinocommon-geometry` | Rhino geometry API | Point3d, Vector3d, Curve, Surface, Brep, Mesh, Transform, Intersections, Boolean ops |
| `rhinodoc-objects` | RhinoDoc, ObjectTable, layers | Add/find/modify/replace objects, layers, attributes, events, user strings |
| `grasshopper-component` | GH_Component, DataTree, .gha | SolveInstance, RegisterInputParams/OutputParams, DataTree, GH_Path, parameter types, preview |
| `rhino-eto-ui` | Eto.Forms, dialogs, panels | Dialog, Form, DynamicLayout, controls, UseRhinoStyle, IPanel, thread safety |

## Coverage

Based on the "Rhinoceros 軟體生態系統開發架構與跨平台插件建構權威指南" technical handbook, covering:

- **Chapter 1:** RhinoCommon architecture, .NET Core transition, language selection
- **Chapter 2:** Geometry core (NURBS, Brep topology, Mesh)
- **Chapter 3:** RhinoDoc object system
- **Chapter 4:** Grasshopper plugin development
- **Chapter 5:** Eto.Forms cross-platform UI
- **Chapter 6:** Plugin lifecycle, debugging
- **Chapter 7:** Yak packaging and distribution

## How it works

These are Claude Code **skills** — markdown knowledge files with trigger metadata, plus deeper `references/` notes. You don't call them directly: Claude detects what you're doing (e.g. writing a `GH_Component`, transforming a `Brep`, building an Eto dialog) and loads the matching skill automatically. Just describe your Rhino/Grasshopper task and Claude brings the relevant expertise into context.

## Repository layout

```
rhino-dev-skills/
├── .claude-plugin/
│   ├── marketplace.json     # makes the repo installable via /plugin marketplace add
│   └── plugin.json          # plugin manifest (name, version, description)
├── skills/
│   ├── rhinocommon-setup/
│   ├── rhinocommon-geometry/
│   ├── rhinodoc-objects/
│   ├── grasshopper-component/
│   └── rhino-eto-ui/
├── CHANGELOG.md
├── LICENSE
└── README.md
```

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for release history.

## Contributing

Contributions are welcome and encouraged. The skills in this plugin will always be a work in progress — Rhino APIs evolve, and there are always more patterns, edge cases, and examples worth capturing.

**Ways to contribute:**

- **Fix an error** — if a skill has a wrong API signature, outdated pattern, or misleading example, open a PR with a correction.
- **Improve coverage** — add examples, expand a references file, or document a pattern that isn't covered yet.
- **Add a skill** — if there's a significant Rhino/Grasshopper domain not yet covered (e.g. Rhino.Render, RhinoMobile, CPython scripting), a new skill folder is welcome.
- **Share your experience** — found a gotcha the skills don't warn about? Open an issue or submit a PR directly.

Each skill lives in `skills/<skill-name>/SKILL.md` with optional deeper notes in `references/`. The format is plain Markdown — no build step, no toolchain required.

If you've used this plugin in real plugin development and have corrections or additions based on hands-on experience, those contributions are especially valuable.

## Notice

**AI-generated content.** The skills in this plugin were authored by [Claude Code](https://claude.ai/code) (Anthropic). Content was derived from the official Rhino developer documentation, the *Rhinoceros 軟體生態系統開發架構與跨平台插件建構權威指南* handbook, and the author's personal Rhino/Grasshopper development experience — which was used to guide, correct, and refine the AI's output.

**Not an official product.** This plugin is an independent community project and is not affiliated with, endorsed by, or supported by Robert McNeel & Associates or Anthropic.

**Trademarks.** Rhinoceros®, Grasshopper®, RhinoCommon, and the Rhino logo are trademarks of Robert McNeel & Associates. Claude™ and Claude Code™ are trademarks of Anthropic, PBC. All trademarks remain the property of their respective owners.

**Accuracy.** The skills reflect documentation and experience available at the time of authoring. Rhino/RhinoCommon APIs evolve — always cross-check against the [latest McNeel developer docs](https://developer.rhino3d.com/) for production use.

## License

MIT — see [LICENSE](LICENSE).
