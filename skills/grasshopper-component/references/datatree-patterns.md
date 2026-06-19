# Grasshopper DataTree Patterns

## DataTree vs GH_Structure

| Type | Used for | Context |
|------|----------|---------|
| `DataTree<T>` | Building output trees, working with raw types | In SolveInstance, output |
| `GH_Structure<T>` | Reading input trees with GH wrappers | From DA.GetDataTree |
| `GH_Path` | Path indices like {0;1;2} | Used by both |

## GH_Path

```csharp
// Simple paths
GH_Path path0 = new GH_Path(0);           // {0}
GH_Path path1_2 = new GH_Path(1, 2);      // {1;2}
GH_Path nested = new GH_Path(0, 1, 0);    // {0;1;0}

// Path arithmetic
GH_Path appended = path0.AppendElement(3);  // {0;3}
GH_Path incremented = path0.Increment(0);   // {1}

// Path from string
GH_Path fromStr = new GH_Path(GH_Path.SyntaxParser("{0;1;2}"));
```

## Building Output Trees

```csharp
// Pattern 1: Mirror input tree structure
GH_Structure<GH_Curve> inputTree;
DA.GetDataTree(0, out inputTree);

DataTree<Curve> outputTree = new DataTree<Curve>();
for (int bi = 0; bi < inputTree.Branches.Count; bi++)
{
    GH_Path path = inputTree.Paths[bi];
    foreach (GH_Curve ghCrv in inputTree.Branches[bi])
    {
        Curve crv;
        ghCrv.CastTo(out crv);
        outputTree.Add(crv.Offset(Plane.WorldXY, 1.0, 0.001, 
                       CurveOffsetCornerStyle.Sharp)[0], path);
    }
}
DA.SetDataTree(0, outputTree);

// Pattern 2: Graft (each item becomes its own branch)
DataTree<Point3d> graftedTree = new DataTree<Point3d>();
List<Point3d> flatList = new List<Point3d> { pt1, pt2, pt3 };
for (int i = 0; i < flatList.Count; i++)
{
    graftedTree.Add(flatList[i], new GH_Path(i));
}

// Pattern 3: Flatten (all to {0})
DataTree<Point3d> flatTree = new DataTree<Point3d>();
foreach (var branch in inputTree.Branches)
    foreach (var item in branch)
    {
        Point3d pt; item.CastTo(out pt);
        flatTree.Add(pt, new GH_Path(0));
    }

// Pattern 4: Add range to a branch
List<Point3d> points = ComputePoints();
outputTree.AddRange(points, new GH_Path(branchIndex));
```

## Reading Input Trees

```csharp
GH_Structure<GH_Point> tree;
DA.GetDataTree(0, out tree);

// Iterate all paths and branches
foreach (GH_Path path in tree.Paths)
{
    List<GH_Point> branch = tree.Branches[tree.PathIndex(path)];
    foreach (GH_Point ghPt in branch)
    {
        Point3d pt;
        ghPt.CastTo(out pt);
        // process pt
    }
}

// Or by index
for (int i = 0; i < tree.Branches.Count; i++)
{
    GH_Path path = tree.Paths[i];
    List<GH_Point> branch = tree.Branches[i];
    // ...
}

// Check if tree has data
if (tree.IsEmpty) return;
int totalItems = tree.DataCount;
```

## DataTree Utilities

```csharp
// Merge two trees (combining paths)
DataTree<Point3d> merged = new DataTree<Point3d>();
for (int i = 0; i < treeA.BranchCount; i++)
    merged.AddRange(treeA.Branches[i], treeA.Paths[i]);
for (int i = 0; i < treeB.BranchCount; i++)
{
    // Offset paths to avoid collision
    GH_Path newPath = treeB.Paths[i].Increment(0);
    merged.AddRange(treeB.Branches[i], newPath);
}

// Simplify paths (remove leading common indices)
tree.SimplifyPaths(); // mutates in place

// Flatten to list
DataTree<T> anyTree;
List<T> flat = anyTree.AllData(); // extension method via Grasshopper.DataTree

// Check if branch exists
bool hasBranch = tree.PathExists(new GH_Path(0, 1));

// Get or create branch
if (!tree.PathExists(path))
    tree.EnsurePath(path); // creates empty branch
List<T> branch2 = tree.Branch(path);
```

## GH_Structure Operations (Input Side)

```csharp
GH_Structure<GH_Brep> structure;
DA.GetDataTree(0, out structure);

// Duplicate for modification
GH_Structure<GH_Brep> copy = structure.Duplicate();

// Get specific branch
if (structure.PathExists(new GH_Path(0)))
{
    List<GH_Brep> branch = structure.get_Branch(new GH_Path(0)) as List<GH_Brep>;
}

// Graft (each item becomes its own path)
GH_Structure<GH_Brep> grafted = structure.Graft(GH_Conversion.Both);

// Flatten
GH_Structure<GH_Brep> flat = structure.Flatten();
```

## Common DataTree Mistakes

1. **Forgetting to AppendElement when nesting**: When iterating outer loops, the inner items should be at `outerPath.AppendElement(innerIndex)`.

2. **Using same GH_Path instance in a loop**: `GH_Path` is a struct — safe to reuse, but be explicit:
   ```csharp
   // Good
   for (int i = 0; i < outer; i++)
       for (int j = 0; j < inner; j++)
           tree.Add(val, new GH_Path(i, j)); // fresh path each time
   ```

3. **Calling DA.SetDataTree with a GH_Structure instead of DataTree**: These types are different. Build a `DataTree<T>` and pass it.

4. **Not unwrapping GH types**: `DA.GetData(0, ref point)` works with `Point3d` directly. But in `GH_Structure`, items are `GH_Point` — must call `.CastTo()`.

## Type Casting Cheat Sheet

```csharp
// GH_Point → Point3d
GH_Point ghPt;
Point3d rawPt; ghPt.CastTo(out rawPt);
// or: rawPt = ghPt.Value;

// GH_Curve → Curve
GH_Curve ghCrv;
Curve rawCrv; ghCrv.CastTo(out rawCrv);

// GH_Brep → Brep
GH_Brep ghBrep;
Brep rawBrep; ghBrep.CastTo(out rawBrep);

// GH_Number → double
GH_Number ghNum;
double val; ghNum.CastTo(out val);
// or: val = ghNum.Value;

// Generic IGH_Goo → any type
IGH_Goo goo;
Point3d pt = Point3d.Unset;
if (goo.CastTo(out pt)) { /* success */ }
```
