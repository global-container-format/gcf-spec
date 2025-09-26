# GCF - Global Container Format

The Global Container Format (GCF) is a container format for deployment and exchange of media resources especially meant for real-time applications. Its main purpose is to be linear and simple to parse while maintaining a feature-set oriented towards efficient runtime resource loading. It draws inspiration from both the [DDS](https://docs.microsoft.com/en-us/windows/win32/direct3ddds/dx-graphics-dds-pguide) and [KTX](https://www.khronos.org/ktx/) file formats. The GCF format attempts to strike a balance between the two in terms of speed of development and flexibility.

**Format version**: 4.0.0

**Format stability: 🧪 WIP - UNSTABLE 🧪**

This repository contains both the spec and the reference C implementation.

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

The header is immediately followed by `Resource Count` resource descriptors, then `Resource Count` resources. Header and resource descriptors are all stored as little-endian.

Any padding and reserved bytes must be set to `0`.

![Container](images/container.svg)

## Header

The container header consists of the following fields:

Name           | Format  | Description
---------------|---------|------------------------------------------
Magic          | uint32  | Format identifier
Resource Count | uint16  | Number of resources contained in the GCF file
Alignment      | uint16  | Resource alignment exponent term

The format identifier is the string `GC##` encoded as a single 32 bits unsigned integer,
where `##` is a double digit unsigned integer representing the version.

For GCF version 4, this is equal to the string "GC04", encoded as `0x34304347`.

### Alignment

Resource alignment is expressed as the formula

    alignment = 2 ^ x

Where `x` is the exponent term and `^` represents the power operator. Alignment values greater than 1 require inserting padding in between two resources and before the first resource so that each resource start offset is aligned to the given value. The "resource start offset" indicates the offset from the start of the GCF file.

## Resources

Each resource consists of a descriptor and some associated content data. The resource descriptor has the following structure:

Name                   | Format     | Description
-----------------------|------------|-----------------------------
Type                   | uint32     | Type of resource contained
Format                 | uint32     | Data format
Content Size           | uint32     | Size of content data
Extension Size         | uint16     | Size of the extra fields
Supercompression Scheme| uint16     | Data supercompression scheme

The `Type` field is an enumeration specifying the type of resource this descriptor refers to.

The `Format` field is an enumeration specifying how to interpret the resource data. Valid values for this field depend on the resource type and are informational only. Format values are not directly used by reader implementations and an unknown format value must not generate an error. Supported values are listed in the [format table](./format.md). The format range between `[0x70000000-0xffffffff)` is available for private application use. Format `0xffffffff` is meant for testing.

`Content Size` specifies the size, in bytes, of the supercompressed content data indicated by the descriptor, without accounting for padding.

`Supercompression Scheme` defines a compression scheme used within the resource to compress the content data. What part of the content data is compressed, depends on the resource type.

The above is known as the *standard descriptor* and is the same for every resource type. When needed, resources may extend their descriptor by appending extra fields, generating a *combined descriptor* made of the standard descriptor as specified above, followed by the *extended descriptor*. When this happens, `Extension Size` is the size, in bytes of the extended descriptor. If a resource has no extended descriptor, `Extension Size` must be 0.

Resource descriptor structures must be aligned to 8 bytes by adding reserved fields as necessary.

Resources are stored in the same order as the descriptors.

![Resource Descriptor](images/resource-descriptor.svg)

### Resource Types

The following resource types are specified:

Type #      | Name                                               | Format
-----------:|----------------------------------------------------|:------:
0           | [Blob](resources/blob.md)                          | ❌
1           | [Texture](resources/texture.md)                    | ✅
2           | [Geometry](resources/geometry.md)                  | ❌
0xffffffff  | Test                                               | ✅

In the table above, the `Format` column specifies whether the format field is meaningful or should be set to `FORMAT_UNDEFINED (0)`.

The resource type range between `[0x70000000-0xffffffff)` is available for private application use. When reading resource descriptors, any resource having an unknown descriptor should be skipped.

The resource type `0xffffffff` is meant for testing. Applications should skip a resource with such type. Implementations may support only a subset of resource types.

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

The supercompression scheme `0xffff` is meant for testing. Reader implementations may support only a subset of supercompression schemes but writer implementations should support all.

## Bugs, Feedback and Further Information

File an issue on the [GitHub repository](https://github.com/global-container-format/gcf-spec). Before doing so, read the [FAQ](FAQ.md) to see if your question was already answered.

If you are interested in upcoming features, check the [roadmap](./roadmap.md).
