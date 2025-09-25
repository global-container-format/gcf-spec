# Geometry resource

The geometry resource type holds opaque geometry data. The resource format value must be `FORMAT_UNDEFINED`.

## Resource Extended Descriptor

The extended descriptor structure of texture resources is the following:

Name                     | Format     | Description
-------------------------|------------|-----------------------------
Vertex stride            | uint32     | Stride of an individual vertex data element
Vertex count             | uint32     | Number of vertices stored in the vertex data
Index count              | uint32     | Number of indices stored in the index data
Flags                    | uint32     | Geometry flags
Level of detail          | uint8      | Level of detail of this geometry resource
Topology                 | uint8      | Indicates the topology of the geometry data
Reserved                 | uint16     | Reserved, must be 0

Each geometry resource data consists of vertex data, immediately followed by index data. The `Level of detail` indicates the LOD of the given resource. This allows packing multiple LODs in the same GCF file.

The vertex stride indicates the stride of a single vertex element in the vertex data. The vertex data is made of `Vertex count` items of `Vertex stride` stride. It is followed by `Index count` vertex indices, where each index is represented as an unsigned integer (see [Flags](#Flags)).

### Flags

The following resource descriptor flags are available:

Name           | Value      | Description
---------------|-----------:|------------------------------------------
Index 16 bits  | 0x00000001 | When set, indices are 16 bits unsigned integers, when unset indices are 32 bits integers
Compressed     | 0x00000002 | When set the mesh data is compressed

### Topology

The following topologies are supported:

Name           | Value     
---------------|----------:
Point List     | 0
Line List      | 1
Line Strip     | 2
Triangle List  | 3
Triangle Strip | 4
Triangle Fan   | 5

