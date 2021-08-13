# Blob resource

This is a completely generic resource type and the content data consists of an unstructured sequence of bytes. The type specific data only contains the expected uncompressed size of the content data:

Name                   | Format     | Description
-----------------------|------------|-----------------------------
Uncompressed size      | uint64     | Uncompressed resource size
Rsvd                   | uint64     | Reserved
Rsvd2                  | uint16     | Reserved

The supercompression scheme is applied to the entire data blob.

*Note: The actual size of the uncompressed data may differ from the expected uncompressed size and should not be trusted.*
