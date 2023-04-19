# GCF - Global Container Format

The Global Container Format (GCF) is a container format for deployment of resources. Its main purpose is to be linear and simple to parse while maintaining a feature-set oriented towards efficient runtime resource loading. It draws inspiration from both the [DDS](https://docs.microsoft.com/en-us/windows/win32/direct3ddds/dx-graphics-dds-pguide) and [KTX](https://github.khronos.org/KTX-Specification) file formats. While the DDS format is extremely simple to parse, it feels clunky, limited and outdated; the KTX format on the other hand is great and flexible but can't be parsed without writing a lot of code. The GCF format attempts to strike a balance between the two.

**Format version**: 2.0.1

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
|Type Info|A portion of resource metadata that is specific for a given resource type.

## Container

![Container](images/container.svg)

The general structure of the format is:

* Header
* 1..n Resource

The header is immediately followed by `Resource Count` resources.
Within each resource and between any two resources, padding and reserved bytes should be set to `0`.

## Header

The container header consists of the following fields:

Name           | Format  | Description
---------------|---------|------------------------------------------
Magic          | uint32  | Format identifier
Resource Count | uint16  | Number of resources following the header
Flags          | uint16  | Container flags

The format identifier is the string `GC##` encoded as a single 32 bits unsigned integer,
where `##` is a double digit unsigned integer number representing the version.

For GCF version 2, this is equal to the string "GC02", encoded as `0x32304347`.

Files can be stored both as big-endian and little-endian. File endianness can be inferred
by inspecting the first byte of the file. For little-endian encoded files, the first byte
will always be `0x47` (the equivalent ASCII character code for the letter "G"). Conforming
read-only implementations are not required to support both byte orders, while read-write
implementations should.

### Magic number computation

The magic number can be computed for any version by the following Python script:

```python
from struct import unpack

unpack('<I', b'GC02')[0] # Replace "02" with correct version number
```

### Container flags

Name           | Bit     | Description
---------------|--------:|------------------------------------------
Unpadded       | 0       | Enabled when there is no padding between resources

The `Unpadded` flag, when enabled, requires no padding to be present between any two consequent resources.

![Padded vs unpadded](images/padding.svg)

## Resources

Each resource consists of a 256 bits descriptor and some associated content data. The resource descriptor has the following structure:

Name                   | Format     | Description
-----------------------|------------|-----------------------------
Type                   | uint32     | Type of resource contained
Format                 | uint32     | Data format
Size                   | uint32     | Size of content data
Supercompression Scheme| uint16     | Data supercompression scheme
Reserved               | uint16     | Reserved
Type Info              | *          | Resource-type-specific info

The `Type` field is an enumeration specifying the type of resource this descriptor refers to.

The `Format` field is an enumeration specifying how to interpret the resource data. Valid values for this field depend on the resource type and are informational only. Format values are not directly used by reader implementations and an unknown format value should not generate an error. Supported values are listed in the [format table](./format.md). The format range between `[0x70000000-0xffffffff)` is available for private application use. Format `0xffffffff` is meant for testing.

`Size` specifies the size, in bytes, of the compressed content data following the descriptor, without taking in account any padding.

`Supercompression Scheme` defines a compression scheme used within the resource to compress the content data. What part of the content data is compressed, depends on the resource type.

`Type Info` is a 16 bytes long extension structure whose meaning and format depend on the resource type.

*Any unused bit of `Type Info` must be set to 0*.

Resource descriptor structures must be aligned on a 64 bits boundary. Padding must be added **after** the resource content data to ensure the next resource descriptor is properly aligned, unless the `Unpadded` flag is enabled. In this case, no padding must be placed between the resource content data and the next resource descriptor header. In any case the last resource does not require any padding.

![Resource Descriptor](images/resource-descriptor.svg)

### Resource Types

The following resource types are standardized:

Type #      | Name                                               | Format
-----------:|----------------------------------------------------|:------:
0           | [Blob](resources/blob.md)                          | ❌
1           | [Image](resources/image.md)                        | ✅
0xffffffff  | Test                                               | ✅

In the table above, the `Format` column specifies whether the format field is meaningful or should be set to `VK_FORMAT_UNDEFINED`.

The resource type range between `[0x70000000-0xffffffff)` is available for private application use. When reading resource descriptors, any resource having an unknown descriptor should be ignored.

The resource type `0xffffffff` is meant for testing. Applications should skip a resource with such type. Reader implementations may support only a subset of resource types but writer implementations should support all.

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

The supercompression scheme `0xffff` is meant for testing. Applications should return an error when presented with a resource with such supercompression scheme. Reader implementations may support only a subset of supercompression schemes but writer implementations should support all.

## Bugs

File any issue on the [GitHub repository](https://github.com/global-container-format/gcf-spec). Before doing so,
read the [FAQ](FAQ.md) to see if your question was already answered.
