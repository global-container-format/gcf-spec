# GCF - Global Container Format

The Global Container Format (GCF) is a container format for deployment of generic resources. The main purpose
of the format is to be linear and simple to parse while maintaining an optimized layout. It draws inspiration from both the [DDS](https://docs.microsoft.com/en-us/windows/win32/direct3ddds/dx-graphics-dds-pguide) and [KTX](https://github.khronos.org/KTX-Specification) file formats. While the DDS format feels clunky, limited and outdated, the KTX format is great and flexible but too complicated to parse quickly. This format is not meant to be an all-encompassing, highly flexible format for data exchange but rather a format optimized for asset loading in game engines.

**Format stability: 🧪 UNSTABLE 🧪**

This repository contains both the spec and the reference C implementation.

## Container

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

## Resources

Each resource consists of a 256 bits descriptor and some associated content data. The resource descriptor has the following structure:

Name                   | Format     | Description
-----------------------|------------|-----------------------------
Type                   | uint32     | Type of resource contained
Format                 | uint32     | Data format
Size                   | uint32     | Size of content data
Supercompression Scheme| uint16     | Data supercompression scheme
Type Data              | *          | Type specific data

The `Type` field is an enumeration specifying the type of resource this descriptor refers to.
The `Format` field is an enumeration with the same values and meanings as [`VkFormat`](https://www.khronos.org/registry/vulkan/specs/1.2-extensions/html/chap43.html#VkFormat) specifying the format of the data. Valid values for this field depend on the resource type.
`Size` specifies the size, in bytes, of the compressed content data following the descriptor, without taking in account any padding.
`Supercompression Scheme` defines a compression scheme used within the resource to compress the content data. What part of the content data is compressed, depends on the resource type.
`Type Data` is an 18 bytes long structure whose meaning and format depend on the resource type.

*Any unused bit of `Type Data` must be set to 0*.

Resource descriptor structures must be aligned on a 64 bits boundary. Padding must be added **after** the resource content data to ensure the next resource descriptor is properly aligned, unless the `Unpadded` flag is enabled. In this latter case, no padding must be placed between the resource content data and the next resource descriptor header. In any case the last resource does not require any padding.

### Resource Types

The following resource types are standardized:

Type #      | Name              | Format | Resource type family
-----------:|-------------------|:------:|:-------------------
0           | Blob              | ❌     | Blob
1           | Color Map         | ✅     | Image
0xffffffff  | Test              | ❌     | Test

In the table above, the `Format` column specifies whether the format field is meaningful or should be set to `VK_FORMAT_UNDEFINED`, while the `Resource type family` column specifies how the `Type Data` field in the resource descriptor is used.

The resource type range between (0x70000000-0xfffffffe) is available for private use and will not be standardized. When reading resource descriptors, any resource that has an unknown descriptor type may be skipped by advancing to the next resource descriptor.

The resource type `0xffffffff` is meant for testing. It can be supported by GCF writers but a conformig GCF reader must always skip a resource whose resource type is `0xffffffff`.

#### Blob resource type family

This is a completely generic resource type and the content data consists of an unstructured sequence of bytes. The type specific data only contains the expected uncompressed size of the content data. The supercompression scheme is applied to the entire data blob.

Name                   | Format     | Description
-----------------------|------------|-----------------------------
Uncompressed size      | uint64     | Uncompressed resource size
Rsvd                   | uint64     | Reserved
Rsvd2                  | uint16     | Reserved

*Note: The actual size of the uncompressed data may differ from the expected uncompressed size and should not be trusted.*

#### Image resource type family

The format for the `Type Data` structure of image resource types is as follows:

Name                   | Format     | Description
-----------------------|------------|-----------------------------
Width                  | uint16     | Resource width
Height                 | uint16     | Resource height
Depth                  | uint16     | Resource depth
Layer Count            | uint8      | Resource layer count
Mip Level Count        | uint8      | Number of mip levels
Flags                  | uint16     | Resource flags
Reserved               | uint64     | Reserved

The `Width`, `Height` and `Depth` fields describe the resource dimensions; their units and specific meanings depend on the resource type.
`Layer Count` specify the number of layers in layered resources such as array textures.
`Mip Level Count` specifies the number of mip levels for mip maps.
`Flags` is a bit set whose definition is entirely up to the resource type.

Layered resources that support mip-mapping are can be read in the following way:

```C
for(uint32_t mip_level = 0; mip_level < resource_descriptor.mip_level_count; ++mip_level) {
    for(uint32_t layer = 0; layer < resource_descriptor.layer_count; ++layer) {
        ...
    }
}
```

##### Currently standardised image resources

* [Color Map](resource-formats/color-map.md)

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
