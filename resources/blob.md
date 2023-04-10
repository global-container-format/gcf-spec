# Blob resource

This is a generic resource type and the content data consists of an unstructured sequence of bytes. The type info only contains the expected uncompressed size of the content data:

Name                   | Format     | Description
-----------------------|------------|-----------------------------
Uncompressed size      | uint64     | Expected uncompressed resource size
Reserved               | uint64     | Reserved

The supercompression scheme is applied to the entire data blob. If no supercompression scheme is used, the value of uncompressed size should be equal to the resource size but it has to be ignored when reading the resource.

*Note: The actual size of the uncompressed data may differ from the expected uncompressed size as this is just a hint. Do not trust this value for allocating memory unless you can verify it.*
