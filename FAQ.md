# Frequently Asked Questions

**Why are decompressed sizes only a hint?**

Security. It's a hint to application developers not to trust these values for memory allocation, unless they can verify their GCF files were not tampered with.

**How is the magic number computed for a given version?**

The magic number can be computed for any version by the following Python script:

```python
from struct import unpack

unpack('<I', b'GC02')[0] # Replace "02" with correct version number
```

The magic number is always computed with little-endian-based code. This way you can use it
to identify endianness of each file by checking whether the first byte is a "G" ASCII character.

**Is the GCF format specific for Vulkan?**

No. The GCF format has nothing to do with Vulkan. GCF simply borrows the format numbers and naming as it was convenient to
adopt their convention instead of inventing a new one. GCF readers and writers don't need Vulkan headers or libraries.

**Why support multiple resources?**

Multiple resources can be useful for some use cases. You may want to have the fastest loading time of related resources, so you
can put them sequentially in the same file - ex. a 3D model and its textures or a set or related textures. Another use case would be storing multiple versions of the same resource, each with different quality. You don't have to support multiple resources in your application but if you need it, it's there.

**Why include a blob resource type?**

While the blob provides little value in a single-resource scenario, it can be useful to embed custom resources such as extra metadata, license files or the original file used to create the resource.

**Why doesn't GCF support data formats such as scalars or vectors?**

These can be expressed using existing formats. For example a sequence of normalised 3D vectors with each entry being 8-bits wide, could be stored with format `B8G8R8_SNORM`. For anything more specific to your use case, a range is allocated for application formats.

**Why is the spec so lax in terms of what implementations must include?**

The main goals of GCF are flexibility and development speed. For example, forcing an implementation to support ZLIB supercompression when they only need Deflate is not necessary.
