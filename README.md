# GCF - Global Container Format

The Global Container Format (GCF) is a container format for deployment and exchange of media resources especially meant for real-time applications. Its main purpose is to be linear and simple to parse while maintaining a feature-set oriented towards efficient runtime resource loading. While it can be used as-is, the GCF format is primarily intended as a low level specification to build application-specific formats for higher level constructs. An application can rely on GCF to efficiently store its textures, define more complex resource layouts to accommodate higher-level resource types (3D models, scenes, LODs, etc..) or extend it to support its own proprietary data types, without ever breaking compatibility.

**Format version**: 4.0.0

**Format stability: 🧪 WIP - UNSTABLE 🧪**

## Vocabulary

|Name|Meaning
|----|----
|Implementation|Unless otherwise stated, this word refers to a piece of software intended to read and/or write GCF files.
|Reader|A piece of software intended to read GCF files.
|Writer|A piece of software intended to write GCF files.
|Container|The data, including any metadata, stored in a GCF file. A container is a collection of resources.
|Resource|An indivisible unit of data, along with its metadata, within the container.

## Container

The general structure of the format is:

* Header
* 1..n Resource Descriptor
* 1..n Resource

The header is immediately followed by `Resource Count` resource descriptors, then `Resource Count` resources. GCF files follow the little-endian convention.

Any padding, reserved bytes must be set to `0`. Any flag bits intended for application-specific usage that are not used by the application must be set to 0.

![Container](images/container.svg)

## Header

The container header consists of the following fields:

Name           | Format  | Description
---------------|---------|------------------------------------------
Magic          | uint32  | Format identifier
Resource Count | uint16  | Number of resources contained in the GCF file
Alignment      | uint16  | Resource alignment exponent term

The format identifier is the string `GC##` encoded as a single 32 bits unsigned integer,
where `##` is a double digit unsigned integer representing the major version.

For GCF version 4, this is equal to the string "GC04", encoded as `0x34304347`.

GCF major versions are not backwards-compatible and applications should not blindly attempt reading a container with a format version different from the reader's supported one.

### Alignment

Resource alignment is expressed as the formula

    alignment = 2 ^ x

Where `x` is the exponent term and `^` represents the power operator. Alignment values greater than 1 require inserting padding in between two resources and before the first resource so that each resource start offset is aligned to the given value. The "resource start offset" indicates the offset from the start of the GCF file.

## Resources

Each resource consists of a descriptor and some associated content data. The resource descriptor has the following structure:

Name                   | Format     | Description
-----------------------|------------|-----------------------------
ID                     | uint16     | Resource identifier
Type                   | uint16     | Type of resource contained
Flags                  | uint32     | Resource flags
Content Size           | uint32     | Size of content data
Extension Size         | uint16     | Size of the extra fields
Supercompression Scheme| uint16     | Data supercompression scheme

The `Type` field is an enumeration specifying the type of resource this descriptor refers to.

`Content Size` specifies the size, in bytes, of the content data indicated by the descriptor as stored on disk, without accounting for padding.

`Supercompression Scheme` defines a compression scheme used within the resource to compress the content data. What part of the content data is compressed, depends on the resource type.

`ID` is an integer resource identifier used to refer to the resource within the GCF file. This allows resources to be position-independent, simpler to refer to and re-organize. IDs are not required to be unique. Any value in the allowed range can be used - this allows, for example, multiple-LODs to share the same ID and to query the resource via ID + LOD level. ID `0xffffffff` is reserved for testing and must not be used. When resources are not referenced by ID, the field is meaningless.

The above is known as the *standard descriptor* and is the same for every resource type. When needed, resources may extend their descriptor by appending extra fields, generating a *combined descriptor* made of the standard descriptor as specified above, followed by the *extended descriptor*. When this happens, `Extension Size` is the size, in bytes of the extended descriptor. If a resource has no extended descriptor, `Extension Size` must be 0.

Standard and extended resource descriptor structures must be aligned to 8 bytes by adding reserved fields as necessary.

Resources are stored in the same order as the descriptors.

![Resource Descriptor](images/resource-descriptor.svg)

### Resource Types

The following resource types are specified:

Type #      | Name                                               
-----------:|----------------------------------------------------
0           | [Blob](resources/blob.md)                          
1           | [Texture](resources/texture.md)                    
2           | [Geometry](resources/geometry.md)                  
3           | [Volume](resources/volume.md)                      
0xffff      | Test                                               

The resource type range between `[0x7000-0xffff)` is available for private application use. When reading resource descriptors, any resource having an unknown descriptor should be skipped.

The resource type `0xffff` is meant for testing. Applications should skip a resource with such type. Implementations may support only a subset of resource types.

### Resource Flags

The following resource types are specified:

Flag        | Name                 | Description
-----------:|----------------------|-----------------------------
0x00000001  | Private              | The resource is private
0x00008000  | Test                 | Test

A private resource is not part of the main exported data. Private resources should be skipped when loading the GCF file, unless referenced explicitly by another resource. For example, a mesh file could contain bounding volume information as Volume resources. The Geometry resource associated with a convex hull volume, will have its Private flag set to indicate the stream is not part of the primary mesh data.

The test flag is intended for testing purposes only.

Flags from 0x00010000 onwards are available for internal application use.

### Supercompression Scheme

Scheme # | Name
--------:|------
0        | None
1        | [ZLIB](https://datatracker.ietf.org/doc/html/rfc1950)
2        | [DEFLATE](https://datatracker.ietf.org/doc/html/rfc1951)
0xffff   | Test

When the `None` supercompression scheme is used, no data is compressed and the compressed size of the data equals
its uncompressed size.

The supercompression scheme range between `[0x7000-0xffff)` is available for private application use.

The supercompression scheme `0xffff` is meant for testing. Reader implementations may support only a subset of supercompression schemes but writer implementations should support all. Applications reading a resource with unknown supercompression scheme, should throw an error.

## Bugs, Feedback and Further Information

File an issue on the [GitHub repository](https://github.com/global-container-format/gcf-spec). Before doing so, read the [FAQ](FAQ.md) to see if your question was already answered.
