# GCF - Global Container Format

The Global Container Format (GCF) is a container format for deployment of generic resources. The main purpose
of the format is to be linear and simple to parse while maintaining an optimized layout. It draws inspiration from both the [DDS](https://docs.microsoft.com/en-us/windows/win32/direct3ddds/dx-graphics-dds-pguide) and [KTX](https://github.khronos.org/KTX-Specification) file formats. While the DDS format feels clunky, limited and outdated, the KTX format is great and flexible but too complicated to parse quickly. This format is not meant to be an all-encompassing, highly flexible format for data exchange but rather a format optimized for asset loading in game engines.

**Format stability: 🧪 WIP - UNSTABLE 🧪**

This repository contains both the spec and the reference C implementation.

## Container

![Container](images/container.svg)

The general structure of the format is:

* Header
* 1..n Resource

The header is immediately followed by `Resource Count` resources with no gaps in between.
Any padding or reserved bytes must be set to `0`.

## Header

The container header consists of the following fields:

Name           | Format  | Description
---------------|---------|------------------------------------------
Magic          | uint32  | Format identifier
Resource Count | uint16  | Number of resources following the header
Flags          | uint16  | Container flags

The format identifier is the string `GC##` encoded as a single 32 bits unsigned integer,
where `##` is a double digit unsigned integer number representing the version.

For GCF version 2, this is equal to the string "GC02".

Files can be stored both as big-endian and little-endian. File endianness can be inferred
by inspecting the first byte of the file. For little-endian encoded files, the first byte
will always be 0x47 (the equivalent ASCII character code for the letter "G"). Conforming
read-only implementations are not obliged to support both byte orders, while read-write
implementations should.

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
Type Data              | *          | Type specific data

The `Type` field is an enumeration specifying the type of resource this descriptor refers to.
The `Format` field is an enumeration with the same values and meanings as [`VkFormat`](https://www.khronos.org/registry/vulkan/specs/1.2-extensions/html/chap43.html#VkFormat) specifying the format of the data. Valid values for this field depend on the resource type.
`Size` specifies the size, in bytes, of the compressed content data following the descriptor, without taking in account any padding.
`Supercompression Scheme` defines a compression scheme used within the resource to compress the content data. What part of the content data is compressed, depends on the resource type.
`Type Data` is an 16 bytes long structure whose meaning and format depend on the resource type.

*Any unused bit of `Type Data` must be set to 0*.

Resource descriptor structures must be aligned on a 64 bits boundary. Padding must be added **after** the resource content data to ensure the next resource descriptor is properly aligned, unless the `Unpadded` flag is enabled. In this latter case, no padding must be placed between the resource content data and the next resource descriptor header. In any case the last resource does not require any padding.

![Resource Descriptor](images/resource-descriptor.svg)

### Resource Types

The following resource types are standardized:

Type #      | Name                                               | Format
-----------:|----------------------------------------------------|:------:
0           | [Blob](resources/blob.md)                          | ❌
1           | [Image](resources/image.md)                        | ✅
0xffffffff  | Test                                               | ✅

In the table above, the `Format` column specifies whether the format field is meaningful or should be set to `VK_FORMAT_UNDEFINED`.

The resource type range between (0x70000000-0xfffffffe) is available for private use and will not be standardized. When reading resource descriptors, any resource that has an unknown descriptor type may be skipped by advancing to the next resource descriptor.

The resource type `0xffffffff` is meant for testing. It can be supported by GCF writers but a conformig GCF reader must always skip a resource whose resource type is `0xffffffff`.

### Supercompression Scheme

Scheme # | Name
--------:|------
0        | None
1        | [ZLIB](https://datatracker.ietf.org/doc/html/rfc1950)
2        | [DEFLATE](https://datatracker.ietf.org/doc/html/rfc1951)
0xffff   | Test

When the `None` supercompression scheme is used, no data is compressed and the compressed size of the data equals
its uncompressed size.

The supercompression scheme range between (0x7000-0xfffe) is available for private use and will not be standardized.

The supercompression scheme `0xffff` is meant for testing. It can be supported by GCF writers but a conforming GCF reader must always return an error when presented with a resource whose supercompression scheme is `0xffff`.

## Bugs

File any issue on the [GitHub repository](https://github.com/global-container-format/gcf-spec).
