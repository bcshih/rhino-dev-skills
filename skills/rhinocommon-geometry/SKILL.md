---
name: rhinocommon-geometry
description: |
  Deep reference for Rhino.Geometry namespace — all geometry types, operations, and patterns in RhinoCommon. Use this skill whenever working with Rhino geometry in C# or Python: Point3d, Vector3d, Curve, Surface, Brep, Mesh, NURBS, intersections, boolean operations, transforms, planes, bounding boxes. Triggers on: "rhinocommon geometry", "rhino brep", "rhino curve", "rhino surface", "rhino mesh", "point3d vector3d", "nurbs rhino", "rhino intersection", "boolean union rhino", "rhino transform", "rhino plane", "geodesic", "geometry api rhino".
---

# Rhino.Geometry — Complete Reference

## Core Value Types (Structs)

```csharp
// Points
Point3d pt = new Point3d(1.0, 2.0, 3.0);
Point2d uv = new Point2d(0.5, 0.5);
Point3f ptF = new Point3f(1f, 2f, 3f); // float precision (mesh vertices)

// Vectors
Vector3d v = new Vector3d(0, 0, 1);    // Z-axis direction
v.Unitize();                            // normalize in place (returns bool)
double len = v.Length;
Vector3d cross = Vector3d.CrossProduct(v1, v2);
double dot = v1 * v2;                   // dot product operator

// Point - Vector arithmetic
Point3d moved = pt + v * 5.0;          // translate point
Vector3d diff = pt2 - pt1;             // vector from pt1 to pt2

// Interval
Interval domain = new Interval(0.0, 1.0);
double mid = domain.Mid;               // 0.5
double t = domain.ParameterAt(0.25);   // remap 0..1 to domain

// BoundingBox
BoundingBox bbox = new BoundingBox(Point3d.Origin, new Point3d(10, 10, 10));
bool contains = bbox.Contains(pt);
bbox.Inflate(1.0);                      // expand uniformly
Point3d center = bbox.Center;
```

## Plane

```csharp
// Common planes
Plane world = Plane.WorldXY;
Plane worldYZ = Plane.WorldYZ;
Plane worldZX = Plane.WorldZX;

// Custom plane (origin, xAxis, yAxis — must be orthonormal)
Plane plane = new Plane(origin, xAxis, yAxis);

// Plane from point + normal
Plane fromNormal = new Plane(origin, normal);

// Plane at point on surface
Surface srf; double u, v;
srf.FrameAt(u, v, out Plane srfFrame);

// Coordinate transform
plane.ClosestPoint(worldPt, out double px, out double py);
Point3d localPt = plane.PointAt(px, py);
```

## Transform

```csharp
// Identity
Transform xf = Transform.Identity;

// Translation
Transform move = Transform.Translation(new Vector3d(10, 0, 0));

// Rotation (angle in radians)
Transform rot = Transform.Rotation(Math.PI / 4, Vector3d.ZAxis, Point3d.Origin);

// Scale
Transform scale = Transform.Scale(Point3d.Origin, 2.0);
Transform nonUniform = Transform.Scale(new Plane(origin, xAxis, yAxis), 2, 1, 1);

// Mirror
Transform mirror = Transform.Mirror(Plane.WorldYZ);

// Apply to geometry (always work on a duplicate!)
GeometryBase geo = original.Duplicate();
geo.Transform(xf);

// Compose transforms
Transform combined = rot * move; // applied right-to-left: move then rot
```

## Curves

```csharp
// Line
Line line = new Line(pt1, pt2);
LineCurve lc = new LineCurve(line);

// Arc
Arc arc = new Arc(circle, Math.PI);     // half-circle
ArcCurve ac = new ArcCurve(arc);

// Circle
Circle circle = new Circle(Plane.WorldXY, radius);
Circle circlePt = new Circle(pt, radius); // in world XY through point

// Polyline
Polyline pline = new Polyline(new List<Point3d> { pt1, pt2, pt3 });
pline.Add(pt4);
PolylineCurve plc = pline.ToPolylineCurve();

// NurbsCurve — construct from points
NurbsCurve nc = NurbsCurve.CreateFromPoints(points, degree: 3);
NurbsCurve interpolated = NurbsCurve.CreateInterpolatedCurve(points, degree: 3);

// Curve operations
Curve crv; // any Curve subtype
double length = crv.GetLength();
crv.ClosestPoint(testPt, out double t);
Point3d ptOnCrv = crv.PointAt(t);
Vector3d tangent = crv.TangentAt(t);
crv.Reverse();
Curve[] segments = crv.DuplicateSegments();
bool isClosed = crv.IsClosed;
bool isPeriodic = crv.IsPeriodic;
Interval domain2 = crv.Domain;

// Divide curve
double[] divParams = crv.DivideByCount(10, includeEnds: true);
Point3d[] divPts = crv.DivideByLength(targetLength, includeEnds: true, out double[] divT);

// Offset curve (in plane)
Curve[] offsets = crv.Offset(Plane.WorldXY, distance: 1.0, tolerance: 0.001, 
                              cornerStyle: CurveOffsetCornerStyle.Sharp);

// Join curves
Curve[] joined = Curve.JoinCurves(curveList, joinTolerance: 0.001);

// Boolean region operations
Curve[] regions = Curve.CreateBooleanRegions(curves, Plane.WorldXY, 
                    selectPoints, tolerance: 0.001, returnRegions: true)?.Loops;
```

## Surfaces

```csharp
// NurbsSurface
NurbsSurface ns = NurbsSurface.CreateFromPoints(points, uCount, vCount, uDegree, vDegree);
NurbsSurface loft = NurbsSurface.CreateRuledSurface(crv1, crv2);

// Evaluate surface
ns.Evaluate(u, v, 1, out Point3d srfPt, out Vector3d[] derivs);
// derivs[0] = dS/du, derivs[1] = dS/dv

// Normal at UV
ns.NormalAt(u, v, out Vector3d normal);

// Closest point  
ns.ClosestPoint(testPt, out double closestU, out double closestV, 
                out double closestDist, 0.001);

// Surface domain
Interval uDomain = ns.Domain(0); // U domain
Interval vDomain = ns.Domain(1); // V domain

// PlaneSurface
PlaneSurface planeSrf = new PlaneSurface(Plane.WorldXY, 
                          new Interval(-5, 5), new Interval(-5, 5));

// Extrusion
Extrusion extr = Extrusion.Create(profile, height, cap: true);
```

## Brep (Boundary Representation)

See `references/brep-deep-dive.md` for complete Brep topology details.

```csharp
// Create Brep from primitives
Brep box = Brep.CreateFromBox(new Box(Plane.WorldXY, 
                               new Interval(0,10), new Interval(0,10), new Interval(0,10)));
Brep sphere = Brep.CreateFromSphere(new Sphere(Point3d.Origin, radius: 5));
Brep cylinder = Brep.CreateFromCylinder(new Cylinder(circle, height), capBottom: true, capTop: true);

// Boolean operations (returns array — may be empty on failure)
Brep[] union = Brep.CreateBooleanUnion(new[] { brepA, brepB }, tolerance: 0.001);
Brep[] diff  = Brep.CreateBooleanDifference(new[] { brepA }, new[] { brepB }, tolerance: 0.001);
Brep[] inter = Brep.CreateBooleanIntersection(new[] { brepA }, new[] { brepB }, tolerance: 0.001);

// Validity check
bool isValid = brep.IsValid;
bool isSolid = brep.IsSolid;
bool isManifold = brep.IsManifold(out bool isOriented, out bool hasBoundary);

// Iterate faces
foreach (BrepFace face in brep.Faces)
{
    Surface srf = face.UnderlyingSurface();
    bool isTrimmed = face.IsSurface; // false = trimmed face
    BrepLoop outerLoop = face.OuterLoop;
}

// Iterate edges
foreach (BrepEdge edge in brep.Edges)
{
    Curve edgeCurve = edge.EdgeCurve;
    int faceA = edge.AdjacentFaces()[0];
    int faceB = edge.AdjacentFaces().Length > 1 ? edge.AdjacentFaces()[1] : -1;
    bool isNakedEdge = edge.Valence == EdgeAdjacency.Naked;
}

// Volume / Area
VolumeMassProperties vmp = VolumeMassProperties.Compute(brep);
double volume = vmp.Volume;
AreaMassProperties amp = AreaMassProperties.Compute(brep);
double area = amp.Area;
```

## Mesh

```csharp
// Create mesh
Mesh mesh = new Mesh();
mesh.Vertices.Add(0, 0, 0);
mesh.Vertices.Add(1, 0, 0);
mesh.Vertices.Add(1, 1, 0);
mesh.Vertices.Add(0, 1, 0);
mesh.Faces.AddFace(0, 1, 2, 3); // quad face
mesh.Faces.AddFace(0, 1, 2);    // triangle face
mesh.Normals.ComputeNormals();
mesh.Compact();

// Mesh from Brep
Mesh[] meshes = Mesh.CreateFromBrep(brep, MeshingParameters.Default);
Mesh joined = new Mesh();
foreach (var m in meshes) joined.Append(m);

// MeshingParameters
var meshParams = new MeshingParameters(0.1); // tolerance
meshParams.MaximumEdgeLength = 1.0;
meshParams.MinimumEdgeLength = 0.01;
meshParams.RefineGrid = true;

// Mesh topology
MeshTopology topo = mesh.TopologyVertices;
int[] neighbors = mesh.TopologyVertices.ConnectedFaces(vi);
int[] connectedVerts = topo.ConnectedTopologyVertices(vi);

// Queries
bool isClosed = mesh.IsClosed;
double meshArea = AreaMassProperties.Compute(mesh).Area;
BoundingBox meshBBox = mesh.GetBoundingBox(accurate: true);

// Ray intersection
Ray3d ray = new Ray3d(rayOrigin, rayDirection);
int faceIndex = Rhino.Geometry.Intersect.Intersection.MeshRay(mesh, ray, out double t);
```

## Intersections

```csharp
using Rhino.Geometry.Intersect;

// Curve-Curve
CurveIntersections ccX = Intersection.CurveCurve(crv1, crv2, tolerance: 0.001, overlapTolerance: 0.0);
foreach (IntersectionEvent e in ccX)
{
    if (e.IsPoint)
    {
        Point3d pt = e.PointA;
        double tA = e.ParameterA;
        double tB = e.ParameterB;
    }
    else if (e.IsOverlap)
    {
        Interval overlapA = e.OverlapA;
    }
}

// Curve-Surface
CurveIntersections csX = Intersection.CurveSurface(crv, srf, tolerance: 0.001, overlapTolerance: 0.0);

// Curve-Brep
Curve[] overlapCurves;
Point3d[] intersectPts;
bool ok = Intersection.CurveBrep(crv, brep, tolerance: 0.001, out overlapCurves, out intersectPts);

// Surface-Surface
Curve[] intCurves;
Point3d[] intPts2;
bool ssOk = Intersection.SurfaceSurface(srfA, srfB, tolerance: 0.001, out intCurves, out intPts2);

// Brep-Brep
Curve[] bbCurves;
Point3d[] bbPts;
bool bbOk = Intersection.BrepBrep(brepA, brepB, tolerance: 0.001, out bbCurves, out bbPts);

// Line-Plane
double lineT;
bool hitPlane = Intersection.LinePlane(line, plane, out lineT);
Point3d hitPt = line.PointAt(lineT);

// Ray-Mesh
double rayT;
int hitFace = Intersection.MeshRay(mesh, ray, out rayT);
```

## Geometry Utilities

```csharp
// Duplicate before modifying
GeometryBase geo2 = original.Duplicate(); // safe deep copy

// Type checking
if (geo is Brep brep2) { ... }
if (geo is Curve curve2) { ... }
if (geo is Mesh mesh2) { ... }

// Distance
double dist = pt1.DistanceTo(pt2);

// Project point to plane
Point3d proj = plane.ClosestPoint(worldPt);

// Point containment (Brep/Curve)
PointContainment pc = brep.IsPointInside(testPt, tolerance: 0.001, strictlyIn: false);
// PointContainment.Inside, Outside, Coincident, Unset

RegionContainment rc = Curve.PlanarClosedCurveRelationship(crvA, crvB, Plane.WorldXY, tolerance: 0.001);
// RegionContainment.AInsideB, BInsideA, Disjoint, MutualIntersection
```
