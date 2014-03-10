---
layout:         "default"
title:          "binpack"
lead:           "Most fast, json like"
---

#Binpack

###what is it

`Binpack` is a binary serialize format, it is json like, but smaller and faster.

###Support data format

*   Null
*   Bool
*   Integer
*   Float
*   String
*   Blob
*   List
*   Dict

#Encoding


	BIN_TYPE_CLOSURE	            = 0x01,	
	BIN_TYPE_LIST   	            = 0x02,
	BIN_TYPE_DICT   	            = 0x03,
	BIN_TYPE_BOOL   	            = 0x04,     /* 0000 0100 T */
    BIN_TYPE_BOOL_FALSE             = 0x05,     /* 0000 0101 F */

	BIN_TYPE_REAL_DOUBLE            = 0x06,     /* 0010 0110   */
    BIN_TYPE_REAL_FLOAT             = 0x07,     /* 0000 0111   */

	BIN_TYPE_NULL   	            = 0x0f,

	BIN_TYPE_BLOB   	            = 0x10,		/* 0001 xxxx + */
	BIN_TYPE_STRING   	            = 0x20,		/* 0010 xxxx + */

	BIN_TYPE_INTEGER 	            = 0x40,		/* 010x xxxx + */
	BIN_TYPE_INTEGER_NEGATIVE    	= 0x60,     /* 011x xxxx - */
