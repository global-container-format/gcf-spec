# Key-Value Store Resource

The key-value store (KV) resource type holds a mapping between keys and values.

The supercompression scheme is applied to the entire resource.

## Resource Extended Descriptor

The extended descriptor structure of KV resources is the following:

Name                     | Format     | Description
-------------------------|------------|-----------------------------
Record count             | uint32     | Number of key-value pairs in the store
Reserved                 | uint32     | Reserved

The data of a KV resource consists of a sequence of records, where each key is an ASCII string of which each character is 1 byte long, and each value is of an arbitrary data type.

## Record Format

The record format is:

Name                     | Format     | Description
-------------------------|------------|-----------------------------
Key Length               | uint16     | Length of the key in characters
Value Type               | uint16     | Value data type
Value Size               | uint32     | Value size, in bytes
Key                      | uint8[]    | Key data
Value                    | uint8[]    | Value data

String keys and values are not NUL-terminated. Data for key and values must be aligned on a 64-bit boundary.

Key Length `0xffff` is reserved for testing and should not be used. Value size `0xffffffff` is reserved for testing and should not be used. Key length and value size do not include padding. Key length must always be greater than 0. A value size of 0 can be used to indicate a NULL value.

## Value Type

The value type format is:

Name                     | Value      | Size | Description
-------------------------|-----------:|-----:|----------------------
String                   | 0          | *    | A UTF-8 string
Integer                  | 1          | 8    | A 64 bit signed integer
Unsigned Integer         | 2          | 8    | A 64 bit unsigned integer
Float                    | 3          | 8    | A 64 bit signed floating point number
Vector 2f                | 4          | 8    | A vector of 2 32-bit floating point numbers
Vector 4f                | 5          | 16   | A vector of 4 32-bit floating point numbers
Vector 8f                | 6          | 32   | A vector of 8 32-bit floating point numbers
Vector 16f               | 7          | 64   | A vector of 16 32-bit floating point numbers
Blob                     | 8          | *    | A sequence of bytes.
Test                     | 0xffff     | 0    | Reserved for testing.

Value types in the range `[0x8000-0xfffe)` are available for the application to use for custom data types. Value `0xffff` is reserved for testing and should not be used.
