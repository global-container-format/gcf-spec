# Frequently Asked Questions

**Why are decompressed sizes only a hint?**

Security. It's a hint to application developers not to trust these values for memory allocation, unless they can verify their GCF files
were not tampered with.

**Is the code shown to compute the magic number the same regardless of endianness?**

Yes. The magic number is always computed with little-endian-based code. This way you can use it
to identify endianness of each file by checking whether the first byte is a "G" ASCII character.

**Is the GCF format specific for Vulkan?**

No. The GCF format has nothing to do with Vulkan. GCF simply borrows the format numbers and naming as it was convenient to
adopt their convention instead of inventing a new one. GCF readers and writers don't need Vulkan headers or libraries.

**Why support multiple resources?**

Multiple resources can be useful for some use cases. You may want to have the fastest loading time of related resources, so you
can put them sequentially in the same file - ex. a 3D model and its textures or a set or related textures. Another use case would be
storing multiple versions of the same resource, each with different quality. You don't have to support multiple resources in your
application but if you need it, it's there.

**Why include a blob resource type?**

While the blob provides little value in a single-resource scenario, it can be useful to embed custom resources such as extra metadata or
license files. In some cases it could even be used to include the original file used to create the resource.
