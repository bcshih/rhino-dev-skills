# rhino-plugin-dev

> A Claude Code plugin that gives Claude complete, hands-on knowledge for building **Rhino & Grasshopper plugins** with RhinoCommon (.NET / C#).

It bundles five domain skills covering the full plugin-development lifecycle — geometry, the document model, Grasshopper components, cross-platform UI, project setup and packaging. Once installed, Claude automatically pulls in the right skill whenever you work on Rhino/Grasshopper code.

## Requirements

- [Claude Code](https://docs.claude.com/en/docs/claude-code) (CLI, desktop, or IDE extension)
- Git — only needed for the marketplace / clone install methods

## Installation

> This is a **private** repository. Whoever installs it must first be granted read access on GitHub (added as a collaborator), and their local `git` must be authenticated (SSH key or GitHub credential manager).

### Option A — Install as a Claude Code marketplace (recommended)

In any Claude Code session:

```
/plugin marketplace add bcshih/rhino-plugin-dev
/plugin install rhino-plugin-dev@rhino-plugin-dev
```

If the `owner/repo` shorthand fails for the private repo, use the explicit git URL instead:

```
/plugin marketplace add git@github.com:bcshih/rhino-plugin-dev.git
/plugin install rhino-plugin-dev@rhino-plugin-dev
```

Run `/plugin` to confirm `rhino-plugin-dev` is listed and enabled.

### Option B — Local install (clone first)

```bash
git clone git@github.com:bcshih/rhino-plugin-dev.git
```

Then point Claude Code at the local folder:

```
/plugin marketplace add /absolute/path/to/rhino-plugin-dev
/plugin install rhino-plugin-dev@rhino-plugin-dev
```

### Option C — Drop the skills in manually

Copy the subfolders of `skills/` into your personal skills folder:

- **Windows:** `C:\Users\<you>\.claude\skills\`
- **macOS / Linux:** `~/.claude/skills/`

Each subfolder (e.g. `rhinocommon-setup/`) becomes an available skill on the next session.

## 中文快速安裝

這是一個 **Claude Code 插件**，內含 Rhino/Grasshopper 開發的五個技能。

因為是 **private（私有）repo**，對方需先被你加入為 GitHub collaborator，且本機 `git` 已登入（SSH 金鑰或 GitHub 認證）。

1. 在 Claude Code 中輸入：
   ```
   /plugin marketplace add bcshih/rhino-plugin-dev
   /plugin install rhino-plugin-dev@rhino-plugin-dev
   ```
2. 輸入 `/plugin` 確認 `rhino-plugin-dev` 已啟用，重開 Claude Code 即可。

之後只要在處理 Rhino/Grasshopper 的 C# 程式碼，Claude 會自動載入對應技能，不需手動呼叫。

> 若 `owner/repo` 簡寫對私有 repo 無效，改用完整網址 `git@github.com:bcshih/rhino-plugin-dev.git`，或先 `git clone` 後用本機路徑安裝（Option B）。

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
rhino-plugin-dev/
├── .claude-plugin/
│   ├── marketplace.json     # makes the repo installable via /plugin marketplace add
│   └── plugin.json          # plugin manifest (name, version, description)
├── skills/
│   ├── rhinocommon-setup/
│   ├── rhinocommon-geometry/
│   ├── rhinodoc-objects/
│   ├── grasshopper-component/
│   └── rhino-eto-ui/
├── LICENSE
└── README.md
```

## License

MIT — see [LICENSE](LICENSE).
