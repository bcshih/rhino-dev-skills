---
name: rhinodoc-objects
description: |
  Guide for managing Rhino document objects via RhinoDoc API — adding, modifying, deleting, selecting geometry in the document, object attributes, layers, and event handling. Use this skill for any RhinoDoc-related code: adding geometry to the scene, modifying existing objects, working with layers, GUID management, object selection, document events, CommitChanges pattern. Triggers on: "rhinodoc", "doc.objects", "add object rhino", "rhino layer", "rhinoobject attributes", "guid rhino", "object table", "commitchanges", "rhino document", "objecttable", "rhinodoc events", "replace geometry rhino".
---

# RhinoDoc — Document Object System

## Core Concept: Geometry vs Document Objects

- **Rhino.Geometry.*** — pure math (no layer, color, GUID). Lives in memory.
- **Rhino.DocObjects.RhinoObject** — geometry + attributes in the document. Has a GUID.

Always duplicate geometry before modifying it; never edit geometry still referenced by the document.

## Accessing the Active Document

```csharp
using Rhino;
using Rhino.DocObjects;

// In a Command:
RhinoDoc doc = RhinoDoc.ActiveDoc;
// Or via the RunCommand parameter:
protected override Result RunCommand(RhinoDoc doc, RunMode mode) { ... }
```

## Adding Geometry to the Document

```csharp
// Returns Guid.Empty on failure
Guid id = doc.Objects.AddPoint(new Point3d(0, 0, 0));
Guid id = doc.Objects.AddLine(new Line(pt1, pt2));
Guid id = doc.Objects.AddArc(new Arc(circle, angle));
Guid id = doc.Objects.AddCircle(new Circle(plane, radius));
Guid id = doc.Objects.AddCurve(curve);
Guid id = doc.Objects.AddSurface(surface);
Guid id = doc.Objects.AddBrep(brep);
Guid id = doc.Objects.AddMesh(mesh);
Guid id = doc.Objects.AddExtrusion(extrusion);
Guid id = doc.Objects.AddTextDot("label", point);

// With attributes
ObjectAttributes attr = new ObjectAttributes();
attr.LayerIndex = doc.Layers.FindName("MyLayer").Index;
attr.ColorSource = ObjectColorSource.ColorFromObject;
attr.ObjectColor = System.Drawing.Color.Red;
attr.Name = "MyObject";
Guid id = doc.Objects.AddBrep(brep, attr);
```

## Finding Objects

```csharp
// By GUID (most reliable)
RhinoObject obj = doc.Objects.Find(guid);
if (obj == null) return; // object was deleted

// By name
ObjectEnumeratorSettings settings = new ObjectEnumeratorSettings();
settings.NameFilter = "MyObject";
foreach (RhinoObject ro in doc.Objects.GetObjectList(settings)) { ... }

// All objects of a type
settings.ObjectTypeFilter = ObjectType.Brep;
var breps = doc.Objects.GetObjectList(settings);

// Selected objects
RhinoObject[] selected = doc.Objects.GetSelectedObjects(includeLights: false, includeGrips: false);

// Objects on layer
int layerIndex = doc.Layers.FindName("MyLayer").Index;
settings.LayerIndexFilter = layerIndex;
```

## Extract-Modify-Replace Pattern

This is the canonical way to modify existing geometry. It preserves the Undo system.

```csharp
// 1. Find the object
RhinoObject obj = doc.Objects.Find(guid);
if (obj == null) return;

// 2. Duplicate its geometry (never modify in-place)
GeometryBase geo = obj.Geometry.Duplicate();

// 3. Modify the duplicate
Transform xf = Transform.Translation(new Vector3d(10, 0, 0));
geo.Transform(xf);

// 4. Replace in document
doc.Objects.Replace(guid, (Brep)geo);  // or Replace(obj, geo)

// 5. Redraw
doc.Views.Redraw();
```

## Modifying Object Attributes

```csharp
RhinoObject obj = doc.Objects.Find(guid);

// Get a writable copy of attributes
ObjectAttributes attr = obj.Attributes.Duplicate();
attr.LayerIndex = doc.Layers.FindName("NewLayer").Index;
attr.ObjectColor = System.Drawing.Color.Blue;
attr.ColorSource = ObjectColorSource.ColorFromObject;
attr.Name = "Renamed";

// Commit back (doesn't change geometry)
doc.Objects.ModifyAttributes(obj, attr, quiet: false);
```

## Object Selection

```csharp
// Select/deselect single object
obj.Select(true);  // select
obj.Select(false); // deselect

// Select all
doc.Objects.UnselectAll();
doc.Objects.Select(guid);

// Get user selection (interactive)
Rhino.Input.Custom.GetObject go = new Rhino.Input.Custom.GetObject();
go.SetCommandPrompt("Select objects");
go.GeometryFilter = ObjectType.Brep | ObjectType.Mesh;
go.GroupSelect = true;
go.SubObjectSelect = false;

GetResult res = go.GetMultiple(minCount: 1, maxCount: 0);
if (res != GetResult.Object) return Result.Cancel;

for (int i = 0; i < go.ObjectCount; i++)
{
    ObjRef objRef = go.Object(i);
    Brep brep = objRef.Brep();
    RhinoObject rhinoObj = objRef.Object();
    Guid objGuid = objRef.ObjectId;
}
```

## Layers

```csharp
// Find layer by full path (Parent::Child)
Layer layer = doc.Layers.FindName("Design::Structure");
if (layer == null)
{
    // Create layer
    Layer newLayer = new Layer();
    newLayer.Name = "Structure";
    newLayer.Color = System.Drawing.Color.Green;
    // Find parent
    Layer parent = doc.Layers.FindName("Design");
    if (parent != null) newLayer.ParentLayerId = parent.Id;
    int layerIdx = doc.Layers.Add(newLayer);
}

// Get layer index
int idx = doc.Layers.Find("Design::Structure", ignoreDeletedLayers: true);

// Current layer
int currentLayerIdx = doc.Layers.CurrentLayerIndex;
doc.Layers.SetCurrentLayerIndex(idx, quiet: false);

// Layer properties
layer.IsVisible = true;
layer.IsLocked = false;
layer.CommitChanges(); // required after modifying layer properties
```

## Object Delete and Purge

```csharp
doc.Objects.Delete(guid, quiet: true);
doc.Objects.Delete(obj, quiet: true);

// Delete multiple
doc.Objects.Delete(new[] { guid1, guid2 }, quiet: true);

// Purge deletes permanently (no undo)
// Use doc.Objects.Delete normally (supports undo)
```

## Document Events

```csharp
// Subscribe in OnLoad, unsubscribe on plugin unload
protected override LoadReturnCode OnLoad(ref string errorMessage)
{
    RhinoDoc.AddRhinoObject += OnAddObject;
    RhinoDoc.DeleteRhinoObject += OnDeleteObject;
    RhinoDoc.ModifyObjectAttributes += OnModifyAttributes;
    RhinoDoc.ReplaceRhinoObject += OnReplaceObject;
    RhinoDoc.DocumentPropertiesChanged += OnDocPropsChanged;
    RhinoApp.Closing += OnRhinoClosing;
    return LoadReturnCode.Success;
}

private void OnAddObject(object sender, RhinoObjectEventArgs e)
{
    RhinoObject obj = e.TheObject;
    RhinoDoc doc2 = e.Document;
    // Don't modify document inside events — queue for later
}

private void OnDeleteObject(object sender, RhinoObjectEventArgs e) { ... }
```

## User Strings (Custom Data on Objects)

```csharp
// Attach arbitrary key-value data to a RhinoObject
obj.Attributes.SetUserString("my_plugin_key", "some_value");
obj.Attributes.CommitChanges();

string val = obj.Attributes.GetUserString("my_plugin_key");

// All keys
string[] keys = obj.Attributes.GetUserStrings(out string[] values);
```

## Get/Set User Data (Persistent Plugin Data)

```csharp
// For structured data, derive from UserData
public class MyData : Rhino.DocObjects.Custom.UserData
{
    public string Value { get; set; }
    public override string Description => "My Plugin Data";
    public override string ToString() => Value;

    protected override void ReadDataFrom(BinaryArchiveReader archive)
    {
        Value = archive.ReadString();
    }
    protected override void WriteDataTo(BinaryArchiveWriter archive)
    {
        archive.WriteString(Value);
    }
}

// Attach to object
MyData data = new MyData { Value = "hello" };
obj.Attributes.UserData.Add(data);
obj.Attributes.CommitChanges();

// Read back
MyData retrieved = obj.Attributes.UserData.Find(typeof(MyData)) as MyData;
```

## Redraw and Display

```csharp
doc.Views.Redraw(); // redraw all views
RhinoApp.Wait();    // process pending events (use sparingly in tight loops)

// Disable/enable redraw for batch operations
doc.Views.RedrawEnabled = false;
// ... add many objects ...
doc.Views.RedrawEnabled = true;
doc.Views.Redraw();
```
