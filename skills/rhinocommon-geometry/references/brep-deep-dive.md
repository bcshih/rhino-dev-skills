# Brep Topology — Deep Dive

## Brep Data Model

A Brep has two parallel hierarchies:
- **Topology**: Face → Loop → Trim → Edge → Vertex (how parts connect)
- **Geometry**: Surface → 2D Curve (trim) → 3D Curve (edge) → Point (vertex)

```
Brep
├── Faces[]        → each references a Surface
│   └── Loops[]    → each Loop is a closed boundary on the face
│       └── Trims[] → 2D curves in UV space of the face's surface
├── Edges[]        → shared between faces, have 3D curve geometry
└── Vertices[]     → shared between edges, have Point3d geometry
```

## Key Properties

```csharp
Brep brep;

// Topology counts
int nFaces    = brep.Faces.Count;
int nEdges    = brep.Edges.Count;
int nVertices = brep.Vertices.Count;
int nTrims    = brep.Trims.Count;
int nLoops    = brep.Loops.Count;
int nSurfaces = brep.Surfaces.Count;
int nCurves2  = brep.Curves2D.Count; // UV-space curves
int nCurves3  = brep.Curves3D.Count; // 3D-space curves

// Validity
string whyNotValid;
bool valid = brep.IsValidWithLog(out whyNotValid);
brep.Repair(tolerance: 0.001);
```

## Iterating Faces

```csharp
for (int fi = 0; fi < brep.Faces.Count; fi++)
{
    BrepFace face = brep.Faces[fi];
    
    // Underlying (untrimmed) surface
    Surface srf = face.UnderlyingSurface();
    // or: int srfIndex = face.SurfaceIndex; Surface srf = brep.Surfaces[srfIndex];
    
    // Face orientation relative to surface
    bool reversed = face.OrientationIsReversed;
    
    // Loops: OuterLoop + inner loops (holes)
    BrepLoop outerLoop = face.OuterLoop; // CCW in UV space (for non-reversed face)
    foreach (BrepLoop loop in face.Loops)
    {
        bool isOuter = loop.LoopType == BrepLoopType.Outer;
        bool isHole  = loop.LoopType == BrepLoopType.Inner;
        
        // Trims in this loop
        foreach (BrepTrim trim in loop.Trims)
        {
            // UV curve
            Curve uvCurve = trim.TrimCurve; // 2D curve in surface param space
            
            // Corresponding 3D edge
            int edgeIndex = trim.Edge?.EdgeIndex ?? -1;
            
            // Trim orientation vs edge
            bool trimReversed = trim.IsReversed();
        }
    }
    
    // Get face mesh for analysis
    Mesh faceMesh = face.GetMesh(MeshType.Render);
    
    // Evaluate face at UV
    face.Evaluate(u, v, 1, out Point3d evalPt, out Vector3d[] evalDeriv);
    face.NormalAt(u, v, out Vector3d faceNormal);
    // Note: face normal = surface normal * (reversed ? -1 : 1)
}
```

## Iterating Edges

```csharp
foreach (BrepEdge edge in brep.Edges)
{
    // 3D curve
    Curve edgeCurve = edge.EdgeCurve;
    // or: int curveIndex = edge.EdgeCurveIndex; Curve c = brep.Curves3D[curveIndex];
    
    // Adjacent faces (naked edge = 1 face, manifold edge = 2 faces)
    int[] adjFaces = edge.AdjacentFaces();
    bool isNaked   = edge.Valence == EdgeAdjacency.Naked;    // 1 face
    bool isManifold = edge.Valence == EdgeAdjacency.Interior; // 2 faces
    bool isNonManifold = edge.Valence == EdgeAdjacency.NonManifold; // 3+ faces
    
    // Edge endpoints (topological vertices)
    BrepVertex startVertex = brep.Vertices[edge.StartVertex.VertexIndex];
    BrepVertex endVertex   = brep.Vertices[edge.EndVertex.VertexIndex];
}
```

## Common Brep Operations

```csharp
// Cap planar holes
Brep capped = brep.CapPlanarHoles(tolerance: 0.001);

// Merge coplanar faces
brep.MergeCoplanarFaces(tolerance: 0.001);

// Extract sub-Brep from specific faces
Brep subBrep = brep.DuplicateSubBrep(new[] { faceIndex1, faceIndex2 });

// Explode into individual face Breps
Brep[] faces2 = brep.DuplicateFaces(duplicateMeshes: false);

// Join Breps (like BooleanUnion for open surfaces)
Brep[] joined = Brep.JoinBreps(new[] { brepA, brepB }, tolerance: 0.001);

// Offset surface
Brep offsetBrep = brep.Faces[0].CreateOffsetBrep(distance: 1.0, solid: false, extend: false,
                                                   tolerance: 0.001, out Brep[] blends, out Brep[] walls);

// Trim with plane
Brep[] trimmed = brep.Trim(plane, tolerance: 0.001);

// Split with surface
Brep[] split = brep.Split(cutter, tolerance: 0.001);
```

## Why Boolean Operations Fail

1. **Non-manifold input**: Check `brep.IsManifold` before booleans
2. **Open Brep**: `brep.IsSolid` must be true for solid booleans
3. **Intersecting geometry at tolerance edge**: Try increasing tolerance
4. **Degenerate faces**: Use `brep.IsValidWithLog()` to diagnose
5. **Duplicate/coincident faces**: MergeCoplanarFaces first

```csharp
// Safe boolean union pattern
bool AIsValid = brepA.IsValid && brepA.IsSolid;
bool BIsValid = brepB.IsValid && brepB.IsSolid;
if (AIsValid && BIsValid)
{
    Brep[] result = Brep.CreateBooleanUnion(new[] { brepA, brepB }, 0.001);
    if (result == null || result.Length == 0)
    {
        // Boolean failed — try repair
        brepA.Repair(0.001);
        brepB.Repair(0.001);
        result = Brep.CreateBooleanUnion(new[] { brepA, brepB }, 0.01);
    }
}
```

## Trimmed Surfaces Explained

A `BrepFace` wraps a `Surface` but may only expose part of it via trim curves in UV space. The face's UV domain is the same as the surface's UV domain — trims restrict which part is "active."

```csharp
BrepFace face;
Surface srf = face.UnderlyingSurface();

// Surface UV domain (full extent)
Interval uFull = srf.Domain(0);
Interval vFull = srf.Domain(1);

// A point may be inside the surface domain but outside the trim boundary!
// Use face.IsPointOnFace() instead of srf.Domain check
PointFaceRelation pfr = face.IsPointOnFace(u, v);
// PointFaceRelation.Interior, Boundary, Exterior
```
