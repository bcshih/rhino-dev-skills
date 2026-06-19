---
name: grasshopper-component
description: |
  Complete guide for developing custom Grasshopper components in C# — GH_Component class, RegisterInputParams/OutputParams, SolveInstance, DataTree patterns, parameter types, error handling, and component packaging as .gha. Use this skill whenever creating or debugging a Grasshopper component: "grasshopper component c#", "GH_Component", "SolveInstance", "DataTree", "GH_Structure", "RegisterInputParams", "grasshopper .gha", "grasshopper custom component", "GH_Path", "grasshopper data tree", "grasshopper plugin", "IGH_Param", "DA.GetData", "DA.SetData".
---

# Grasshopper Component Development

## Component Class Template

```csharp
using System;
using System.Collections.Generic;
using Grasshopper.Kernel;
using Grasshopper.Kernel.Types;
using Rhino.Geometry;

namespace MyGHPlugin
{
    public class MyComponent : GH_Component
    {
        // Constructor: (name, nickname, description, category, subcategory)
        public MyComponent()
            : base("My Component", "MyCmp",
                   "Description of what this component does",
                   "MyPlugin", "Operations")
        { }

        // Unique GUID — generate once and never change
        public override Guid ComponentGuid => new Guid("PUT-YOUR-GUID-HERE");

        // Optional: 24x24 icon
        protected override System.Drawing.Bitmap Icon => null;

        // Optional: component exposure in the ribbon
        public override GH_Exposure Exposure => GH_Exposure.primary;

        protected override void RegisterInputParams(GH_InputParamManager pManager)
        {
            // Add inputs (index order matters — matches DA.GetData calls)
            pManager.AddPointParameter("Point", "P", "Input point", GH_ParamAccess.item);
            pManager.AddNumberParameter("Distance", "D", "Distance value", GH_ParamAccess.item, 1.0); // default value
            pManager.AddCurveParameter("Curves", "C", "Input curves", GH_ParamAccess.list);
            pManager.AddBrepParameter("Brep", "B", "Input brep", GH_ParamAccess.item);
            pManager.AddGenericParameter("Any", "A", "Any type", GH_ParamAccess.item);
            
            // Make a parameter optional
            pManager[1].Optional = true;
        }

        protected override void RegisterOutputParams(GH_OutputParamManager pManager)
        {
            pManager.AddPointParameter("Result", "R", "Output point", GH_ParamAccess.item);
            pManager.AddCurveParameter("Curves", "C", "Output curves", GH_ParamAccess.list);
            pManager.AddNumberParameter("Values", "V", "Computed values", GH_ParamAccess.tree);
        }

        protected override void SolveInstance(IGH_DataAccess DA)
        {
            // 1. Collect inputs
            Point3d point = Point3d.Unset;
            if (!DA.GetData(0, ref point)) return; // required — abort if missing
            
            double distance = 1.0;
            DA.GetData(1, ref distance); // optional — don't abort on failure

            List<Curve> curves = new List<Curve>();
            if (!DA.GetDataList(2, curves)) return;

            Brep brep = null;
            DA.GetData(3, ref brep);

            // 2. Validate
            if (point == Point3d.Unset)
            {
                AddRuntimeMessage(GH_RuntimeMessageLevel.Error, "Invalid point.");
                return;
            }
            if (distance <= 0)
            {
                AddRuntimeMessage(GH_RuntimeMessageLevel.Warning, "Distance should be positive.");
                distance = Math.Abs(distance);
            }

            // 3. Compute
            Point3d result = point + new Vector3d(distance, 0, 0);

            // 4. Set outputs
            DA.SetData(0, result);
            DA.SetDataList(1, curves);
            // DA.SetDataTree(2, someTree); -- see DataTree section
        }
    }
}
```

## Parameter Types Reference

```csharp
// Primitives
pManager.AddBooleanParameter("Flag", "F", "Boolean flag", GH_ParamAccess.item);
pManager.AddIntegerParameter("Count", "N", "Integer count", GH_ParamAccess.item, 1);
pManager.AddNumberParameter("Value", "V", "Number value", GH_ParamAccess.item, 0.0);
pManager.AddTextParameter("Name", "N", "Text string", GH_ParamAccess.item, "default");
pManager.AddColourParameter("Color", "C", "Color", GH_ParamAccess.item);
pManager.AddIntervalParameter("Domain", "D", "Interval", GH_ParamAccess.item);

// Geometry
pManager.AddPointParameter("Point", "P", "3D point", GH_ParamAccess.item);
pManager.AddVectorParameter("Vector", "V", "3D vector", GH_ParamAccess.item);
pManager.AddPlaneParameter("Plane", "Pl", "Plane", GH_ParamAccess.item);
pManager.AddLineParameter("Line", "L", "Line", GH_ParamAccess.item);
pManager.AddCurveParameter("Curve", "C", "Curve", GH_ParamAccess.item);
pManager.AddSurfaceParameter("Surface", "S", "Surface", GH_ParamAccess.item);
pManager.AddBrepParameter("Brep", "B", "Brep", GH_ParamAccess.item);
pManager.AddMeshParameter("Mesh", "M", "Mesh", GH_ParamAccess.item);
pManager.AddGeometryParameter("Geo", "G", "Any geometry", GH_ParamAccess.item);
pManager.AddGenericParameter("Data", "D", "Any data", GH_ParamAccess.item);
pManager.AddTransformParameter("Transform", "T", "Transform matrix", GH_ParamAccess.item);
```

## Access Modes

| Mode | pManager call | DA call | Behavior |
|------|--------------|---------|----------|
| `GH_ParamAccess.item` | default | `DA.GetData(i, ref val)` | SolveInstance called once per item combination |
| `GH_ParamAccess.list` | `.list` | `DA.GetDataList(i, list)` | SolveInstance called once, you get all items |
| `GH_ParamAccess.tree` | `.tree` | `DA.GetDataTree(i, out tree)` | Full DataTree, maximum control |

Item access is most common — GH handles list/tree matching automatically (like Grasshopper's "data matching").

## DataTree Operations

See `references/datatree-patterns.md` for complete patterns.

```csharp
// Reading a tree
GH_Structure<GH_Point> inputTree;
DA.GetDataTree(0, out inputTree);

// Building an output tree
DataTree<Point3d> outputTree = new DataTree<Point3d>();
for (int i = 0; i < inputTree.Branches.Count; i++)
{
    GH_Path path = inputTree.Paths[i];
    List<Point3d> branch = new List<Point3d>();
    foreach (GH_Point ghPt in inputTree.Branches[i])
    {
        Point3d pt;
        ghPt.CastTo(out pt);
        branch.Add(pt + new Vector3d(1, 0, 0));
    }
    outputTree.AddRange(branch, path);
}
DA.SetDataTree(0, outputTree);
```

## GH Type Wrappers

Grasshopper wraps geometry in `GH_*` types. Most Add methods accept either:

```csharp
// These are equivalent — GH auto-wraps
DA.SetData(0, new Point3d(1, 2, 3));         // raw RhinoCommon type
DA.SetData(0, new GH_Point(new Point3d(1,2,3))); // explicit GH wrapper

// Unwrapping from GetData
Point3d pt = Point3d.Unset;
DA.GetData(0, ref pt); // works with raw type

// Or via GH wrapper (needed for GH_Structure)
GH_Point ghPt;
DA.GetData(0, ref ghPt);
Point3d raw = ghPt.Value;
```

## Runtime Messages

```csharp
// Error = red component, aborts outputs
AddRuntimeMessage(GH_RuntimeMessageLevel.Error, "Message explaining the error");

// Warning = orange component, outputs still produced
AddRuntimeMessage(GH_RuntimeMessageLevel.Warning, "Non-critical warning");

// Remark = grey (informational)  
AddRuntimeMessage(GH_RuntimeMessageLevel.Remark, "FYI note");

// Clear messages (rarely needed — GH clears them each solve)
ClearRuntimeMessages();
```

## Component with Preview

```csharp
public class MyPreviewComponent : GH_Component
{
    // Store computed geometry for preview
    private List<Curve> _previewCurves = new List<Curve>();

    protected override void SolveInstance(IGH_DataAccess DA)
    {
        _previewCurves.Clear();
        // ... compute ...
        _previewCurves.Add(resultCurve);
        DA.SetData(0, resultCurve);
    }

    // Override to draw custom preview
    public override void DrawViewportWires(IGH_PreviewArgs args)
    {
        base.DrawViewportWires(args);
        if (_previewCurves == null) return;
        foreach (var c in _previewCurves)
            args.Display.DrawCurve(c, Selected ? args.WireColour_Selected : System.Drawing.Color.Red);
    }

    public override void DrawViewportMeshes(IGH_PreviewArgs args)
    {
        base.DrawViewportMeshes(args);
        // Draw filled meshes here
    }

    public override BoundingBox ClippingBox
    {
        get
        {
            BoundingBox box = BoundingBox.Empty;
            foreach (var c in _previewCurves)
                box.Union(c.GetBoundingBox(false));
            return box;
        }
    }
}
```

## Component with Persistent State (ValueList style)

```csharp
// Store per-solve data that survives between solves
[ThreadStatic]
private static List<string> _lastResults;

// For parameters with UI context menus
protected override void AppendAdditionalComponentMenuItems(ToolStripDropDown menu)
{
    base.AppendAdditionalComponentMenuItems(menu);
    Menu_AppendItem(menu, "My Option", OnMyOptionClick, true, _myOption);
}

private bool _myOption = false;
private void OnMyOptionClick(object sender, EventArgs e)
{
    _myOption = !_myOption;
    ExpireSolution(recompute: true); // trigger re-solve
}

// Serialize component state
public override bool Write(GH_IWriter writer)
{
    writer.SetBoolean("MyOption", _myOption);
    return base.Write(writer);
}
public override bool Read(GH_IReader reader)
{
    _myOption = reader.GetBoolean("MyOption");
    return base.Read(reader);
}
```

## AssemblyInfo Requirements

```csharp
// In AssemblyInfo.cs or at top of GH_AssemblyInfo file
[assembly: AssemblyTitle("MyGHPlugin")]
[assembly: AssemblyVersion("0.1.0.0")]
[assembly: AssemblyFileVersion("0.1.0.0")]
```

The `.gha` file is discovered by Grasshopper automatically from:
- `%APPDATA%\Grasshopper\Libraries\`
- or any folder listed in Grasshopper's Library Folders

## Common Patterns

```csharp
// Null/Unset checking
if (brep == null || !brep.IsValid) 
{
    AddRuntimeMessage(GH_RuntimeMessageLevel.Error, "Invalid brep");
    return;
}

// Tolerance (always use doc tolerance, not hardcoded)
double tol = DocumentTolerance(); // inherits from active Rhino doc
double angleTol = DocumentAngleTolerance();

// Check if running headless/batch (for UI decisions)
bool isInteractive = !Rhino.RhinoApp.RunningHeadless;

// Force component to re-solve
this.ExpireSolution(true);

// Access other components' outputs (rare, avoid tight coupling)
// Instead use GH_Document and find by instance GUID
```
