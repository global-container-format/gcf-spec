# Texture resource

The texture resource type holds texture content data, supports layers and mip maps. The dimensions in the resource descriptors are expressed in texels for 1D and 2D textures, and in voxels for 3D textures. The representable textures are:

* 1D textures
* 2D textures
* 3D textures

The resource format value must not be `FORMAT_UNDEFINED` or a non-color format.

## Resource Extended Descriptor

The extended descriptor structure of texture resources is the following:

Name                   | Format     | Description
-----------------------|------------|-----------------------------
Base Width             | uint16     | Base level width
Base Height            | uint16     | Base level height
Base Depth             | uint16     | Base level depth
Layer Count            | uint8      | Resource layer count
Mip Level Count        | uint8      | Number of mip levels
Flags                  | uint16     | Texture flags
Texture Group          | uint16     | The group this texture belongs to
Reserved 2             | uint32     | Reserved

The `Base Width`, `Base Height` and `Base Depth` fields describe the resource dimensions, in texels. For mip mapped textures, these dimensions refer to the largest mip level.
`Layer Count` describe the number of layers in each mip level.
`Mip Level Count` describe the number of mip levels in the texture.
`Flags` describe the texture flags.
`Texture Group` represents an application-specific identifier specifying the texture group this texture belongs to.

In a texture resource, each mip level is represented by a collection of `Layer Count` layers of the same type and size. The layers within the same mip level are all supercompressed together.

### Flags

The following resource descriptor flags are available:

Name           | Value     | Description
---------------|----------:|------------------------------------------
Texture 1D     | 0x0001    | The texture is a 1D texture (X axis only)
Texture 2D     | 0x0003    | The texture is a 2D texture (X and Y axes)
Texture 3D     | 0x0007    | The texture is a 3D texture (X, Y and Z axes)

1D textures only extend along the *X* axis (width). 2D textures extend along the *X* and *Y* axes (width and height) and 3D textures extend along the *X*, *Y* and *Z* axes - width, height and depth. If the texture has no depth or no height, `Base Height` and/or `Base Depth` must be set to 1 in the extended descriptor.

1D textures are stored texel after texel. 2D textures are stored as a sequence of 1D rows. 3D textures are stored as a sequence of 2D texture slices.

## Layers and mip maps

Each mip level is individually super-compressed but all the layers in the same mip level are super-compressed together.

The structure of a mip level is the following:

1. Mip Level Descriptor
2. Compressed Data

The Mip Level Descriptor section contains information about the compressed and uncompressed mip level, while the Compressed Data section contains the compressed raw mip level data.

Textures are made of a sequence of mip layers, each of which is made of a sequence of layers.

For example, to iterate across all the layers, one must do so following the order as shown in the pseudo-code below:

```C
for(uint32_t mip_level = 0; mip_level < resource_descriptor.mip_level_count; ++mip_level) {
    for(uint32_t layer = 0; layer < resource_descriptor.layer_count; ++layer) {
        ...
    }
}
```

Textures that don't take advantage of layers must have the `Layer Count` extended descriptor field set to 1.
Textures that don't take advantage of mip-mapping must have the `Mip Level Count` extended descriptor field set to 1.

Mip levels are stored in order from the base (largest) level to the n-th (smallest) one. Each level dimension (width, height or depth) must be computed by the following formula:

```python
round(max(1, x * 0.5 ** mip_level))
```

Where `x` is the value of the dimension to scale and `mip_level` is the mip level number, 0 being the base level. The `**` symbol represents the power operator.

### The rounding function

The rounding function used to compute the dimensions of the mip levels returns an integer value that is the value of its only argument rounded to the closest integer. For values equidistant from two integers (i.e. 1.5, equally distant from both 1 and 2), the closest even integer will be used.

### Mip Level Descriptor

Name                   | Format  | Description
-----------------------|---------|-----------------------------
Compressed Size        | uint32  | The compressed size of the mip level raw data
Uncompressed Size      | uint32  | The expected uncompressed size of the mip level raw data
Row Stride             | uint32  | The row stride in texels
Slice Stride           | uint32  | The slice stride in texels
Layer Stride           | uint32  | The layer stride in texels
Reserved               | uint32  | Reserved

The bytes within the strides acting as padding must all be set to 0. `Row Stride` represents the stride of each texel row (along the width axis), while `Layer Stride` represents the stride of each texture layer. Layer strides apply to all layers within the mip level and must equal the level's slice stride times the its depth for non-array textures. `Slice Stride` applies to 3D textures, represents the stride of each voxel slice and must be equal to the row stride times the height of the mip level for non-3D textures. While readers can ignore stride values that don't apply to their texture type, writers must still set all of them.

The relationship between the different strides within the uncompressed mip level data satisfies the following equation, which also encodes how the mip level data must be laid out:

```
element_address(x, y, z, layer) = layer_stride * layer + slice_stride * z + row_stride * y + x * element_size
```

Where `element_size` is the texel (or voxel) size, in bytes, according to the resource format and `element_address` is the offset (from the start of the mip level data) required to reach the texel (voxel) located at `x, y, z` in layer `layer` within a given mip level.
