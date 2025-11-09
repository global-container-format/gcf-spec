# Volume Resource

Volume resources represent parametric volumes. These can be used, for example, to store the bounding box structure for a mesh asset. The super-compression scheme for volume resources must be `None`. Orientations are expressed as normalized quaternions in the form `Q = w + xi + yj + zk`

## Resource Extended Descriptor

The extended descriptor structure of volume resources is the following:

Name                   | Format     | Description
-----------------------|------------|-----------------------------
Shape                  | uint16     | The type of volume
Rsvd1                  | uint16     | Reserved
Offset X               | float32    | Position offset
Offset Y               | float32    | Position offset
Offset Z               | float32    | Position offset

## Shape

Name                   | Value      | Description
-----------------------|-----------:|-----------------------------
AA Box                 | 0          | Axis aligned box
Box                    | 1          | A non-axis-aligned box
Sphere                 | 2          | Spherical volume
Frustum                | 3          | Frustum volume
Capsule                | 4          | Capsule volume
Cylinder               | 5          | Cylinder volume
Convex Hull            | 6          | Convex hull volume
Test                   | 0xffff     | Test

Shape values in the range `[0x7000-0xffff)` are available for private application use. Shape `0xffff` is intended for testing.

### AA Box

The parameter data for an AA box is

Name                   | Format
-----------------------|------------
Width                  | float32
Height                 | float32
Depth                  | float32
Reserved               | float32

### Box

The parameter data for a non-AA box is

Name                   | Format
-----------------------|------------
Width                  | float32
Height                 | float32
Depth                  | float32
Reserved               | float32
Orientation W          | float32
Orientation X          | float32
Orientation Y          | float32
Orientation Z          | float32

### Sphere

The parameter data for a sphere is

Name                   | Format
-----------------------|------------
Radius                 | float32
Reserved               | float32

### Frustum

The parameter data for a frustum is

Name                   | Format
-----------------------|------------
Orientation W          | float32
Orientation X          | float32
Orientation Y          | float32
Orientation Z          | float32
Left Slope             | float32
Right Slope            | float32
Top Slope              | float32
Bottom Slope           | float32
Near Plane             | float32
Far Plane              | float32

### Capsule

The parameter data for a capsule is

Name                   | Format
-----------------------|------------
Height                 | float32
Radius 1               | float32
Radius 2               | float32
Reserved               | float32
Orientation W          | float32
Orientation X          | float32
Orientation Y          | float32
Orientation Z          | float32

### Cylinder

The parameter data for a cylinder is

Name                   | Format
-----------------------|------------
Height                 | float32
Reserved               | float32
Orientation W          | float32
Orientation X          | float32
Orientation Y          | float32
Orientation Z          | float32

### Convex Hull

The parameter data for a convex hull is

Name                   | Format
-----------------------|------------
Index                  | uint32
Reserved               | uint32

Index refers to the index of a geometry resource containing the data for the hull.
