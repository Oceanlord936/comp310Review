## what is i-node , and other terminolagy 
---

facts: 

An i-node is a small struct (128 bytes) that holds the info about one file.

8 of them are packed into one disk block in the i-node area.

The disk only reads and writes whole blocks.

The file's actual data is not in the i-node. It's in the data area.

---

Fix 1: one block never holds both kinds. 
Every disk block is the same size (1024 bytes), 

but each one is either an "i-node block" or a "data block":

```
block 2    [ i-node | i-node | ... 8 i-nodes ]      i-node area : ONLY i-nodes
block 3    [ i-node | i-node | ... 8 i-nodes ]
...

block 40   [ 1024 bytes of data              ]      data area : ONLY data
block 41   [ 1024 bytes of data              ]

```
-> They're the same size, but they live in different sections and are never mixed.

Fix 2: a big file still has exactly ONE i-node. 
When it runs out of direct pointers, the indirect pointer points to an index block. 
That's a block in the data area that holds only a list of more block numbers. 
It isn't an i-node, and it doesn't use i-node numbers.

```
i-node 42 (the ONLY i-node for this file)
  direct 0 - 11    -> 12 data blocks
  single indirect  -> index block (in the data area)
                        [ 61 | 62 | 63 | ... ]   <- just block numbers
                           |    |    |
                           v    v    v
                         data blocks

```

=======================
Questions:

1. Does the i-node's structure set the max file size? 
->Yes. The number of pointers and levels decides it:

```

block 1 KB , a block number = 4 bytes -> one index block holds 1024 / 4 = 256 numbers

direct            12                 blocks
single indirect   256                blocks
double indirect   256 * 256          = 65,536 blocks
triple indirect   256 * 256 * 256    = 16,777,216 blocks
total            ≈ 16.8 million blocks * 1 KB ≈ 16 GB max file size

```

---

2. What sets the size of the i-node section?
 It's chosen once, when the disk is formatted (mkfs), 
 and recorded in the super block. After that it's fixed.

i-node section = K blocks  ->  K * 8 i-nodes  ->  at most K * 8 files (and dirs , links)

========================

All of these are the same physical thing: 

a 1024-byte piece of the disk. 
The disk is just disk block 0, 1, 2, ..., N. 

The different names only say what is stored inside that piece:

```
name           = a physical disk block that holds ...

boot block     startup code                         (disk block 0)
super block    the map of the disk                  (disk block 1)

i-node block   8 i-node structs                     (disk blocks 2 ... 39)
data block     a file's bytes , or a dir's list ,   (disk blocks 40 ...)
               or a link's path
			   
index block    a list of disk block numbers         (also in the data area ,
               (used by the indirect pointers)       a data block used for pointers)

```

=================

read a line :

```
path              i-node   i-node block   data block(s)   contents
/users/raj/lab    37       6              55              .  37  ..  19  main.c  52


```

lab is a directory, which is a kind of file.

Its i-node is #37, and that i-node sits on disk block 6.

Its data is on disk block 55.

 a directory's contents are always "name → i-node number "
 
 .       37    "me"         = i-node # 37  (NOT block 37)
..      19    "my parent"  = raj's i-node #19  (NOT block 19)
main.c  52    "my child"   = main.c's i-node 52  (NOT block 52)

============================

The i-node number is the location, written in a short form.
It turns into the disk block and the slot inside it with no extra lookup:

```

i-node 52
  disk block = 2 + floor(52 / 8) = 8
  slot       = 52 mod 8 = 4        -> the 5th i-node in disk block 8
  byte offset in the block = 4 * 128 = 512

```


















