# Blob resource

The blob type represents a generic raw resource. Its content data is an unstructured sequence of bytes. The extended descriptor of a blob resource contains the expected uncompressed size of the content data.

Name                   | Format     | Description
-----------------------|------------|-----------------------------
Uncompressed size      | uint64     | Expected uncompressed resource size

The supercompression scheme is applied to the entire data blob. If no supercompression scheme is used, the value of uncompressed size must be equal to the resource size.

*Note: The actual size of the uncompressed data may differ from the expected uncompressed size as this is just a hint. Do not trust this value for allocating memory unless you can verify it.*
