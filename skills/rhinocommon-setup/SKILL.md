---
name: rhinocommon-setup
description: |
  Complete guide for setting up, structuring, debugging, and packaging Rhino/Grasshopper plugins using RhinoCommon (.NET). Use this skill whenever the user asks about creating a new Rhino plugin project, setting up Visual Studio for Rhino development, configuring .csproj for RhinoCommon, building .rhp/.gha files, debugging a Rhino plugin, or publishing with the Yak package manager. Triggers on: "create rhino plugin", "setup rhinocommon", "rhino plugin project", "debug rhino plugin", "yak package", "publish rhino plugin", ".gha file setup", "rhino .csproj", "rhinocommon nuget".
---

# RhinoCommon Plugin Setup

## Project Types

| Type | Output | Description |
|------|--------|-------------|
| RhinoCommon Plugin | `.rhp` | Adds Commands to Rhino |
| Grasshopper Plugin | `.gha` | Adds Components to Grasshopper |
| Combined | `.rhp` + `.gha` | Both (separate projects, shared library) |

## .csproj Configuration

### Rhino Plugin (.rhp) — Windows-only, no Visual Studio required

For **Windows-only plugins** (Rhino 8, net48), reference Rhino DLLs directly via HintPath — no NuGet needed since the DLLs ship with Rhino:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net48</TargetFramework>
    <AssemblyName>MyPlugin</AssemblyName>
    <RootNamespace>MyPlugin</RootNamespace>
    <LangVersion>9.0</LangVersion>
    <Nullable>disable</Nullable>
    <Platforms>AnyCPU</Platforms>
    <OutputPath>bin\$(Configuration)\</OutputPath>
  </PropertyGroup>

  <ItemGroup>
    <!-- Rhino SDK — reference directly, never copy to output -->
    <Reference Include="RhinoCommon">
      <HintPath>C:\Program Files\Rhino 8\System\RhinoCommon.dll</HintPath>
      <Private>False</Private>
    </Reference>
    <Reference Include="Eto">
      <HintPath>C:\Program Files\Rhino 8\System\Eto.dll</HintPath>
      <Private>False</Private>
    </Reference>
    <Reference Include="Rhino.UI">
      <HintPath>C:\Program Files\Rhino 8\System\Rhino.UI.dll</HintPath>
      <Private>False</Private>
    </Reference>
    <Reference Include="Newtonsoft.Json">
      <HintPath>C:\Program Files\Rhino 8\System\Newtonsoft.Json.dll</HintPath>
      <Private>False</Private>
    </Reference>
    <!-- These BCL assemblies must be listed explicitly for net48 SDK-style projects -->
    <Reference Include="System.Security" />
    <Reference Include="System.Net.Http" />
    <Reference Include="System.Drawing" />
  </ItemGroup>

  <!-- Post-build: rename .dll → .rhp so Rhino recognizes it as a plugin -->
  <!-- ONLY rename in output dir — do NOT auto-copy to AppData while Rhino may be running -->
  <Target Name="PostBuild" AfterTargets="PostBuildEvent">
    <Copy SourceFiles="$(OutputPath)$(AssemblyName).dll"
          DestinationFiles="$(OutputPath)$(AssemblyName).rhp" />
  </Target>
</Project>
```

> **Important:** `System.Net.Http`, `System.Drawing`, and `System.Security` must be listed explicitly in SDK-style net48 projects — they are NOT auto-included like in old-style .csproj.

### Grasshopper Plugin (.gha) — cross-platform with NuGet
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFrameworks>net48;net7.0</TargetFrameworks>
    <AssemblyName>MyGrasshopperPlugin</AssemblyName>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="RhinoCommon" Version="8.*" ExcludeAssets="runtime" />
    <PackageReference Include="Grasshopper" Version="8.*" ExcludeAssets="runtime" />
  </ItemGroup>
</Project>
```

`ExcludeAssets="runtime"` is critical — RhinoCommon.dll is provided by Rhino at runtime, never ship it with your plugin.

## GUID Requirements — Critical

Rhino uses GUIDs to uniquely identify plugins and panels. **You must generate real random GUIDs** — sequential or made-up values will cause load errors.

```powershell
# Generate a proper GUID in PowerShell:
[System.Guid]::NewGuid()
# Example output: d7ae223c-1bbc-4a4c-a664-74176c2dc419
```

### Assembly GUID (plugin identity)
```csharp
// At the top of the plugin file, outside any namespace:
[assembly: System.Runtime.InteropServices.Guid("d7ae223c-1bbc-4a4c-a664-74176c2dc419")]
```

This is what Rhino reads to identify the plugin. Without it, or with `Guid.Empty`, you get:
> `plugInId Can't be Guid.Empty`

### Panel class GUID (IPanel identity)
Every class implementing `IPanel` must also have its own `[Guid]` attribute, and its `PanelId` static field must match:

```csharp
[System.Runtime.InteropServices.Guid("a0b2b927-b6f3-4323-ab71-0079e019ec40")]
public class MyPanel : Panel, IPanel
{
    // PanelId must match the Guid attribute above exactly
    public static readonly Guid PanelId = new Guid("a0b2b927-b6f3-4323-ab71-0079e019ec40");
    // ...
}
```

Missing `[Guid]` on the panel class → `type must have a GuidAttribute`

## Plugin Class (Rhino Plugin)

```csharp
using System.Runtime.InteropServices;
using Rhino;
using Rhino.PlugIns;
using Rhino.UI;

[assembly: Guid("YOUR-RANDOMLY-GENERATED-GUID")]
[assembly: PlugInDescription(DescriptionType.Organization, "Your Org")]
[assembly: PlugInDescription(DescriptionType.WebSite, "")]
[assembly: PlugInDescription(DescriptionType.UpdateUrl, "")]

namespace MyPlugin
{
    public class MyPluginClass : Rhino.PlugIns.PlugIn
    {
        public MyPluginClass() { Instance = this; }
        public static MyPluginClass Instance { get; private set; }

        protected override LoadReturnCode OnLoad(ref string errorMessage)
        {
            // Register dockable panel — use 4-parameter overload (no PanelType param needed)
            Panels.RegisterPanel(this, typeof(MyPanel), "My Panel", null);
            return LoadReturnCode.Success;
        }

        public override PlugInLoadTime LoadTime => PlugInLoadTime.AtStartup;
    }
}
```

> **Note:** `Panels.RegisterPanel` 4-parameter overload works in Rhino 8. The 5-parameter overload with `PanelType` enum may not be available — use the 4-parameter form.

## Command Class

```csharp
using Rhino;
using Rhino.Commands;

namespace MyPlugin
{
    public class MyCommand : Command
    {
        public override string EnglishName => "MyCommandName";

        protected override Result RunCommand(RhinoDoc doc, RunMode mode)
        {
            // mode: RunMode.Interactive (user ran it) vs RunMode.Scripted (macro/script)
            RhinoApp.WriteLine("Hello from MyCommand!");
            return Result.Success;
        }
    }
}
```

## Grasshopper Plugin Bootstrapper

```csharp
using Grasshopper;
using Grasshopper.Kernel;

namespace MyGHPlugin
{
    public class MyGHPluginInfo : GH_AssemblyInfo
    {
        public override string Name => "My GH Plugin";
        public override System.Drawing.Bitmap Icon => null; // 24x24 icon
        public override string Description => "My Grasshopper components";
        public override System.Guid Id => new System.Guid("YOUR-RANDOMLY-GENERATED-GUID");
        public override string AuthorName => "Your Name";
        public override string AuthorContact => "email@example.com";
    }
}
```

## Plugin Installation — Use PlugInManager, Not File Copy

**Do NOT** copy the `.rhp` to `%AppData%\McNeel\Rhinoceros\8.0\Plug-ins\` manually. Rhino tracks plugins by GUID and the path must be registered properly.

**Correct installation:**
1. Build the project → `bin\Release\MyPlugin.rhp` is produced
2. In Rhino: type `_PlugInManager` → click **Install...** → browse to the `.rhp` file
3. Restart Rhino if prompted

**Re-installing after rebuild:**
- Close Rhino completely (it locks the `.rhp` file while running)
- Rebuild
- Rhino remembers the path — just restart Rhino and it reloads automatically

> **File locking:** Rhino holds a file lock on the `.rhp` while running. The PostBuild rename-only pattern (no auto-copy) avoids build failures caused by locked files.

## Build Without Visual Studio

If VS is not available, install the .NET SDK:
```bash
winget install Microsoft.DotNet.SDK.8
```

Then build:
```bash
dotnet build RhinoAI.csproj -c Release
```

## Visual Studio 2022 Debug Configuration

In project Properties → Debug → Launch Profiles:
- **Launch**: Executable
- **Executable**: `C:\Program Files\Rhino 8\System\Rhino.exe`
- **Application arguments**: `/nosplash /notemplate`

Or in `launchSettings.json`:
```json
{
  "profiles": {
    "Rhino 8": {
      "commandName": "Executable",
      "executablePath": "C:\\Program Files\\Rhino 8\\System\\Rhino.exe",
      "commandLineArgs": "/nosplash /notemplate"
    }
  }
}
```

## Output File Renaming (.rhp)

Copy (not Move) dll → rhp in the output directory. Use Copy so the original `.dll` remains for incremental build tracking:

```xml
<Target Name="PostBuild" AfterTargets="PostBuildEvent">
  <Copy SourceFiles="$(OutputPath)$(AssemblyName).dll"
        DestinationFiles="$(OutputPath)$(AssemblyName).rhp" />
</Target>
```

**Do not** use `Move` (breaks incremental builds) or auto-copy to AppData (fails if Rhino is running and has a lock).

## Multi-targeting Pitfalls

- `net48`: Windows only, uses .NET Framework 4.8. Supports older Rhino features.
- `net7.0`: Cross-platform (.NET Core). Required for Rhino 8 on Mac/Apple Silicon.
- Use `#if NETFRAMEWORK` / `#if NET` preprocessor to handle API differences.
- `net7.0-windows` targets Windows-only with .NET 7 (use for WPF/WinForms if needed, but prefer Eto.Forms for cross-platform).

## Yak Package Manager

See `references/yak-packaging.md` for complete packaging workflow.

Quick reference:
```bash
# Install Yak CLI (comes with Rhino)
# Located at: C:\Program Files\Rhino 8\System\yak.exe

# Create manifest
yak spec

# Build package
yak build --platform win

# Push to Yak server
yak push MyPlugin-0.1.0-rh8_0-win.yak
```
