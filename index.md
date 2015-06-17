---
layout:         "default"
title:          "BinPack"
lead:           "Fastest / Refined"
---

#BinPack

---

###what is it

`Binpack` is a binary serialize format, it is json like, but smaller and faster.

###Support data format

*   Null
*   Bool
*   [Integer](http://en.wikipedia.org/wiki/Integer_%28computer_science%29)
*   Blob
*   String
*   Float:  Single and Double pecision
*   List
*   Dict, Key value data set.

#Encoding
---

### Null

* Encode into one byte

    ```
    +-----------+
    | 0000 1111 |   0x0f
    +-----------+
    ```

### Bool

*   `true`

    ```
    +-----------+
    | 0000 0100 |   0x04
    +-----------+
    ```

*   `false`

    ```
    +-----------+
    | 0000 0101 |   0x05
    +-----------+
    ```

### Integer

Integer will be encoded into one or more bytes.

* The last byte is used to store the type and sign information of the Integer.

    The type and sign information is encode into the first 3 bits:

    *   `positive`
    
        ```
        +-----------+
        | 010x xxxx |  0x40
        +-----------+
        ```
    
    *   `negative`
    
        ```
        +-----------+
        | 011x xxxx |  0x60
        +-----------+
        ```
    
* Except the last byte, the first bit of each byte will be `1`. The remain 7 bits in these bytes and the remain 5 bits in the last byte will be used to store the value of the `Integer`, For example:

    ```
        7 bits                  5 bits
    +-----------+...........+-----------+
    | 1xxx xxxx | 1xxx xxxx | ...x xxxx |
    +-----------+...........+-----------+
    ```

    The absolte value of `Integer` in the Two's Complement binary format will be grouped by 7, the LSB bits in front and the MSB in end.

    ```c
    // allocate buf
    char *p = buf;
    // pack
    while (num >= 0x08)
    {
        *p++ = 0x80 | num;
        num >>= 7;
    }
    *p++ = type | num;
    ```

    1 will be encoded to:

    ```
    +-----------+
    | 0100 0001 |
    +-----------+
    ```

    -16 will be encoded to:

    ```
    +-----------+-----------+
    | 1001 0000 | 0110 0000 |  
    +-----------+-----------+
    ```

### Blob

*   `Blob` will be encoded into 2 parts. First part is the `length of Blob`, the second part is the binary data.

    ```
    +----------------+
    | length + data  |  
    +----------------+
    ```

The length will be encoded like `Integer`, but there are two differents:

1.  The type information of `Blob` is `0x01` which will be encoded into the last byte:

    ```
                  0x10 + 4bits
    +...........+-----------+
    | 1xxx xxxx | 0001 xxxx |
    +...........+-----------+
    ```

2.  There are 4 bits in the lat byte can be used to store the value of `length of Blob`.

    ```
                                         binary data
    +-----------+...........+-----------+============+
    | 1xxx xxxx | 1xxx xxxx | 0001 xxxx |    data    |
    +-----------+...........+-----------+============+
    ```
    
### String

*   `String` is also encoded into `length + data` like `Blob`.

    The type of `String` is `0x20`, it also will be encoded into the last byte of the encoded bytes of length.

    ```
                  0x20 + 4 bits
    +...........+-----------+
    | 1xxx xxxx | 0010 xxxx |
    +...........+-----------+

    +-----------+...........+-----------+============+
    | 1xxx xxxx | 1xxx xxxx | 0010 xxxx |    data    |
    +-----------+...........+-----------+============+
    ```

### Float

*   The `Float` type information will be encoded into the first byte, followed by bytes of the Float in the `IEEE-754` format, in `Big Endian`.

    `Double` will be encoded into 9 bytes, `Single` will be 5 bytes.

    ```
      0x06        8 bytes
    +-----------+===========+
    | 0000 0110 |    data   |  Double precision.
    +-----------+===========+

      0x07        4 bytes
    +-----------+===========+
    | 0000 0111 |    data   |  Single precision.
    +-----------+===========+

### List

*   For encoding `List` and `Dict`, we define a `Closure` byte.

    ```
    +-----------+
    | 0000 0001 |   0x01, Closure
    +-----------+
    ```
*   List type is encoded to one byte:

    ```
    +-----------+
    | 0000 0010 |   0x02, List
    +-----------+
    ```

*   `List` type information will be encoded into the first byte, then following every element in `List`.

    The last byte is `Closure`.

    ```
    +-----------+
    | 0000 0010 | 
    +-----------+----------------------------
    |          element 1                       
    +----------------------------------------
    |          element 2                       
    +----------------------------------------
    .    .    .
    .    .    .
    .    .    .
    +----------------------------------------
    |          element N                       
    +-----------+----------------------------
    | 0000 0001 | Closure
    +-----------+
    ```

### Dict

*   Dict type:

    ```
    +-----------+
    | 0000 0011 |   0x03, Dict
    +-----------+
    ```

*   Like `List`, the encoded data will begin with type information and end with `Closure`.

    The key and value of every Entry of the Dictionary will be encoded like following:

    ```
    +-----------+
    | 0000 0011 | Dict
    +-----------+----------------------------
    |            key 1   
    +----------------------------------------
    |          value 1
    +----------------------------------------
    |            key 2   
    +----------------------------------------
    |          value 2
    +----------------------------------------
    .    .    .
    .    .    .
    .    .    .
    +----------------------------------------
    |            key N   
    +----------------------------------------
    |          value N
    +-----------+----------------------------
    | 0000 0001 | Closure
    +-----------+
    ```

### Some 

*   The types:

    ```c
    typedef enum {
        BIN_TYPE_CLOSURE	            = 0x01,
        BIN_TYPE_LIST   	            = 0x02,
        BIN_TYPE_DICT   	            = 0x03,
        BIN_TYPE_BOOL   	            = 0x04,     /* 0000 0100 T */
        BIN_TYPE_BOOL_FALSE             = 0x05,     /* 0000 0101 F */
        
        BIN_TYPE_FLOAT_DOUBLE           = 0x06,     /* 0010 0110   */
        BIN_TYPE_FLOAT_SINGLE           = 0x07,     /* 0000 0111   */
        
        BIN_TYPE_NULL   	            = 0x0f,
        
        BIN_TYPE_BLOB   	            = 0x10,		/* 0001 xxxx   */
        BIN_TYPE_STRING   	            = 0x20,		/* 0010 xxxx   */
        
        BIN_TYPE_INTEGER 	            = 0x40,		/* 010x xxxx + */
        BIN_TYPE_INTEGER_NEGATIVE    	= 0x60      /* 011x xxxx - */
    } bin_type_t;
    ```

###Implemetation

*   php: [https://github.com/binpack/binpack-php](https://github.com/binpack/binpack-php) @2014 May 9

*   Java: [https://github.com/binpack/binpack-java](https://github.com/binpack/binpack-java) @2014 May 9

*   Python: [https://github.com/binpack/binpack-java](https://github.com/binpack/binpack-pthon) @2014 June 13

*   others: TODO

###performance

*   Compared with msgpack, the implemetation of php, the data size after encoded almost the same, but the time is nearly 3/4.

