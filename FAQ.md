# Frequently Asked Questions

**Why are decompressed sizes only a hint?**

Security. It's a hint to application developers not to trust these values for memory allocation, unless they can verify their GCF files were not tampered with.

**How is the magic number computed for a given version?**

The magic number can be computed for any version by the following Python script:

```python
from struct import unpack

unpack('<I', b'GC04')[0]  # Replace "04" with correct version number
```

**Why is the spec so lax in terms of what implementations must include?**

The main goals of GCF are flexibility and development speed. For example, forcing an implementation to support ZLIB supercompression when they only need Deflate is not necessary.
