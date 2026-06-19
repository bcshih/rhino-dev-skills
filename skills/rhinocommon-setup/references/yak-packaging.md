# Yak Package Manager — Complete Packaging Guide

## What is Yak?

Yak is Rhino's official package manager. Packages (`.yak` files) are distributed via the Rhino Package Manager (accessible from `_PackageManager` command in Rhino). 

## manifest.yml

Create at your project root:
```yaml
name: my-plugin
version: 0.1.0
authors:
  - Your Name
description: A short description of your plugin.
url: https://github.com/you/my-plugin

# List all files to include
# Yak auto-discovers .rhp, .gha, .dll, .rui, .ghlink files
```

## Build Command

```bash
# Path to Yak CLI (Rhino 8)
$YAK = "C:\Program Files\Rhino 8\System\yak.exe"

# From directory containing manifest.yml and your built plugin files
& $YAK build --platform win

# For Mac
yak build --platform mac

# For both (run on each platform)
yak build --platform any
```

Output: `my-plugin-0.1.0-rh8_0-win.yak`

Filename format: `{name}-{version}-rh{rhino_version}_{build}-{platform}.yak`

## File Structure Inside .yak

```
my-plugin-0.1.0-rh8_0-win.yak
├── manifest.yml
├── MyPlugin.rhp          # or .dll/.gha
├── MyPlugin.pdb          # optional debug symbols
└── dist/                 # optional: additional files
```

## Versioning

Follow semantic versioning: `MAJOR.MINOR.PATCH`
- MAJOR: breaking changes
- MINOR: new features, backwards compatible
- PATCH: bug fixes

Pre-release: `0.1.0-beta1`

## Multi-platform Packages

Build separate packages per platform:
```bash
# On Windows
yak build --platform win

# On Mac
yak build --platform mac
```

Both packages share the same name/version but different platform suffix.

## Publishing

```bash
# Login first (requires Rhino account)
yak login

# Push package
yak push my-plugin-0.1.0-rh8_0-win.yak

# List your packages
yak search --owned

# Delete a package version
yak yank my-plugin 0.1.0 --platform win --rhino 8
```

## Installing Locally for Testing

Method 1 — Use `_PackageManager` in Rhino, browse to `.yak` file.

Method 2 — Direct installation:
```powershell
# Rhino 8
& "C:\Program Files\Rhino 8\System\yak.exe" install my-plugin
```

Method 3 — Copy plugin files directly:
```
# Rhino plugin (.rhp)
%APPDATA%\McNeel\Rhinoceros\8.0\Plug-ins\<PluginName> (<GUID>)\
  └── <version>\MyPlugin.rhp

# Grasshopper component (.gha)  
%APPDATA%\Grasshopper\Libraries\
  └── MyGHPlugin.gha
```

## Rhino Version Targeting

In manifest.yml:
```yaml
# Target Rhino 8 only
# (controlled by build platform, not manifest directly)
```

The Yak filename encodes Rhino version compatibility:
- `rh8_0` = Rhino 8.0+
- `rh7_0` = Rhino 7.0+

## Required Assembly Attributes

For Rhino plugins, add to `AssemblyInfo.cs` or directly in `.cs`:
```csharp
[assembly: System.Reflection.AssemblyVersion("0.1.0.*")]
[assembly: System.Reflection.AssemblyInformationalVersion("0.1.0")]
```

## .rhp vs .gha Packaging

- `.rhp` = Rhino plugin (rename from `.dll` in post-build)
- `.gha` = Grasshopper component assembly (rename from `.dll` in post-build)
- Both are standard .NET assemblies with renamed extensions

Post-build rename for Grasshopper:
```xml
<Target Name="RenameGHA" AfterTargets="Build">
  <Copy SourceFiles="$(TargetPath)" 
        DestinationFiles="$(TargetDir)$(AssemblyName).gha" />
</Target>
```
