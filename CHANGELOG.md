# Changelog

All notable changes to **rhino-dev-skills** are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2026-06-19

### Added

- Initial release.
- Five Claude Code skills covering the full Rhino/Grasshopper plugin-development lifecycle:
  - `rhinocommon-setup` — project setup, `.csproj` multi-targeting (.NET 4.8 / .NET 7), VS2022 debug configuration, Yak packaging.
  - `rhinocommon-geometry` — `Point3d`/`Vector3d`, `Curve`, `Surface`, `Brep`, `Mesh`, transforms, intersections, boolean operations.
  - `rhinodoc-objects` — `RhinoDoc` object table, layers, attributes, document events, user strings, `CommitChanges` pattern.
  - `grasshopper-component` — `GH_Component`, `SolveInstance`, `DataTree`/`GH_Path`, parameter types, custom preview.
  - `rhino-eto-ui` — Eto.Forms dialogs, dockable panels, `DynamicLayout`, `UseRhinoStyle`, UI thread safety.
- `.claude-plugin/marketplace.json` so the repository is installable via `/plugin marketplace add`.
- MIT `LICENSE`, `.gitignore`, and a `README.md` with installation instructions (English + 中文).
- `NOTICE` file with attribution (AI-generated content, source references, third-party trademarks).
- `## Notice` section in `README.md` covering AI authorship, non-affiliation, trademark ownership, and accuracy caveats.

[0.1.0]: https://github.com/bcshih/rhino-dev-skills/releases/tag/v0.1.0
