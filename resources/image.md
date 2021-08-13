# Image resource

The image resource type holds image content data, supports layers and mip maps. The dimensions in the resource descriptors are expressed in pixels for 1D and 2D images, and in voxels for 3D images. The representable images are:

* 1D images
* 2D images
* 3D images

Each mip level is individually super-compressed but all the layers in the same mip level are super-compressed together.

The structure of each mip level is the following:

* Mip Level Descriptor
* Compressed Data

The Mip Level Descriptor contains information about the compressed and uncompressed mip level, while the compressed data is the compressed raw mip level data. The resource format value must not be `VK_FORMAT_UNDEFINED`.

The format for the `Type Data` structure of image resource types is as follows:

Name                   | Format     | Description
-----------------------|------------|-----------------------------
Width                  | uint16     | Resource width
Height                 | uint16     | Resource height
Depth                  | uint16     | Resource depth
Layer Count            | uint8      | Resource layer count
Mip Level Count        | uint8      | Number of mip levels
Flags                  | uint16     | Image flags
Reserved               | uint64     | Reserved

The `Width`, `Height` and `Depth` fields describe the resource dimensions, in pixels.
`Layer Count` specifies the number of layers in layered images.
`Mip Level Count` specifies the number of mip levels for mip mapped images.
`Flags` specifies the image flags.

For each image resource, each mip level is represented by a collection of `Layer Count` images of the same type and size. The images within the same mip level are all supercompressed together.

## Flags

The following resource descriptor flags are available:

Name           | Value     | Description
---------------|----------:|------------------------------------------
Image 1D       | 0x0001    | The image is a 1D image
Image 2D       | 0x0003    | The image is a 2D image
Image 3D       | 0x0007    | The image is a 3D image

1D images only extend along the `Width` axis. 2D images extend along the `Width` and `Height` axes and 3D images extend along the `Width`, `Height` and `Depth` axes. Whenever not used, `Height` and `Depth` must be set to 1.

1D images are stored one texel after the other in a linear fashion. 2D images are stored as a sequence of rows. 3D images are stored as a sequence of 2D images.

## Layers and mip maps

Layered, mip-mapped images must be accessed following the order as shown in the pseudo-code below:

```C
for(uint32_t mip_level = 0; mip_level < resource_descriptor.mip_level_count; ++mip_level) {
    for(uint32_t layer = 0; layer < resource_descriptor.layer_count; ++layer) {
        ...
    }
}
```

Images that don't take advantage of layers must have the `Layer Count` type data field set to 1.
Images that don't take advantage of mip-mapping must have the `Mip Level Count` type data field set to 1.

Mip levels are stored in oder from the base (largest) level to the n-th (smallest) one. Each level dimension (width, height and depth) must be computed by the following formula:

```python
round(max(1, x * 0.5 ** (mip_level - 1)))
```

Where `x` is the dimension to scale and `mip_level` is the mip level number, with 1 being the number for the base level, 2 the number for the second level and so on. The `**` symbol represents the power operator.

### The rounding function

The rounding function used to compute the dimensions of the mip levels returns an integer value that is the value of its only argument rounded to the closest integer. For values equally close to two integers (i.e. 1.5, equally close to both 1 and 2), the closest even integer will be used.

### Mip Level Descriptor

Name                   | Format  | Description
-----------------------|---------|-----------------------------
Compressed Size        | uint32  | The compressed size of the mip level raw data
Uncompressed Size      | uint32  | The uncompressed size of the mip level raw data
Row Stride             | uint32  | The row stride in bytes
Depth Stride           | uint32  | The depth stride in bytes
Layer Stride           | uint32  | The layer stride in bytes
Reserved               | uint32  | Reserved
Reserved2              | uint64  | Reserved

The bytes of the strides that act as padding must all be set to 0. `Row Stride` represents the stride of each image row (along the width axis), while `Layer Stride` represents the stride of each 2D image, hence the size of each image in bytes, including any padding. Layer strides apply to all images within the array and must be non-zero for both array and non-array images. `Depth Stride` only applies to 3D images and represents the stride of each voxel layer and must be 1 for non-3D images.

The relationship between the different strides within the uncompressed mip level data satisfies the following equation, which also encodes how the mip level data must be encoded:

```
address(x, y, z, layer) = layer_stride * layer + depth_stride * z + row_stride * y + x * element_size
```

Where `element_size` is the texel or voxel size, in bytes, according to the resource format.
