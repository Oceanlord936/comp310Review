## what is an i-node , and the other terms

---

## 1. the disk is an array of blocks

the file system cuts the disk into equal pieces called blocks .
every read or write moves one WHOLE block

```
block size = 1 KB = 1024 bytes  (in this tutorial)
disk = disk block 0 , 1 , 2 , ... , N-1      every block has one number
```

"block N" with no other word = disk block #N .
the word in front only says WHAT is stored inside that block :

```
name           = a physical disk block that holds ...
boot block     startup code                         (disk block 0)
super block    the map of the disk                  (disk block 1)
i-node block   8 i-node structs                     (disk blocks 2 ... 39)
data block     a file's bytes , or a dir's list ,   (disk blocks 40 ...)
               or a link's path
index block    a list of disk block numbers         (in the data area ,
               (used by the indirect pointers)       a data block used for pointers)
```

the one exception :

```
disk block    a real place on the disk (one number for the whole disk)
file block    a position inside ONE file : file block 0 = the file's 1st KB , 1 = 2nd KB ...
              the i-node's pointers turn a file block into a disk block number
```

---

## 2. the four areas on the disk

```
block 0       block 1        block 2 ... 39           block 40 ...
[ boot block | super block | i-node blocks         | data blocks          ]
  code to       the map of    the i-node TABLE       the actual contents :
  start the     the disk :    (info about every      file bytes , directory
  OS            sizes , where file)                  lists , link paths
                each area
                starts , free
                counts
```

(1) boot block : the startup code . we never touch it
(2) super block : describes the whole file system , how big the areas are and where each one starts
(3) i-node blocks : an array of i-nodes , one per file
(4) data blocks : the contents

one block never holds both kinds . every disk block is the same size (1024 bytes) ,
but each one is EITHER an i-node block OR a data block :

```
block 2    [ i-node | i-node | ... 8 i-nodes ]      i-node area : ONLY i-nodes
block 3    [ i-node | i-node | ... 8 i-nodes ]
...
block 40   [ 1024 bytes of data              ]      data area : ONLY data
block 41   [ 1024 bytes of data              ]
```

---

## 3. what an i-node is

an i-node is a small struct (128 bytes) that holds everything about ONE file , except its NAME and its DATA

- 8 of them are packed into one disk block in the i-node area
- the file's actual data is not in the i-node , it is in the data area

```
i-node (128 bytes)
  type          file / directory / symbolic link
  size          length in bytes
  link count    how many names point to it
  owner , permissions , times
  pointers :
    12 direct pointers      -> data block numbers of file blocks 0 ... 11
    1 single indirect       -> an INDEX block full of more pointers
    1 double indirect       -> index of index blocks
    1 triple indirect       -> one more level
```

EVERYTHING has an i-node : a file , a directory , a symbolic link . the type says how to read its data :

```
i-node type     its data blocks hold
file            the file's bytes
directory       a list of (name , i-node number) pairs
symbolic link   a path string , eg "/users/raj/lab/main.c"
```

---

## 4. i-node number vs i-node block

```
i-node NUMBER   = WHICH file's record      (an ID , like a student ID)
i-node BLOCK    = WHERE on the disk that record physically sits
```

there are NO layers of i-nodes . the i-node area is one flat array , just stored across several disk blocks :

```
disk block 2   [ i-node 0 | i-node 1 | i-node 2 | ... | i-node 7  ]
disk block 3   [ i-node 8 | i-node 9 | i-node 10| ... | i-node 15 ]
disk block 4   [ i-node 16| ... | i-node 19 | i-node 20 | ... | 23 ]
...
each box = one i-node = 128 bytes , each row = one block = 1024 bytes
```

```
i-node block = 2 + floor( i-node NUMBER / 8 )      one step , always

2        the i-node area starts at disk block 2 (blocks 0 , 1 = boot , super)
x / 8    8 i-nodes per block -> which row-of-8 i-node x falls in
floor    drop the fraction (19 / 8 = 2.375 -> 2)
```

the i-node number IS the location , written in a short form :

```
i-node 52
  disk block = 2 + floor(52 / 8) = 8
  slot       = 52 mod 8 = 4        -> the 5th i-node in disk block 8
  byte offset in the block = 4 * 128 = 512
```

to read i-node 19 : read disk block 4 (the whole 1024 bytes) , then take record 19 - 16 = 3 (counting from 0)

why use a number at all :
- one file can have several names (hard links) , all pointing to the same i-node number
- the file can grow (its data blocks change) , but the number in the directory stays the same
- rename / move = just move the (name , number) pair , the file is not touched
- the OS keeps open files and the i-node cache by i-node number

---

## 5. directories and names

the name of a thing is NOT in its own i-node , it is in its PARENT directory's data

a directory is a file whose data is a list of (name , i-node number) pairs

```
/users/raj        i-node 19 , data in block 42
block 42 holds :  .  19  |  ..  10  |  notes  42  |  lab  37  |  m  20

the name "lab" lives in raj's data block (42) , not in i-node 37
```

from a name to a file : the directory gives the i-node number , the i-node gives the data blocks

```
.     = ME (this directory)        -> its number is always the directory's own i-node
..    = MY PARENT (one level up)   -> its number is the parent's i-node
name  = a CHILD inside me          -> its number is that child's i-node
```

same idea as in the terminal , . is this directory , .. is the parent :

```
terminal            what the OS does
cd ..               look up ".." in this directory's data block -> parent's i-node
./run.sh            look up "." -> this directory's i-node , then "run.sh" in its list
cd ../lab           ".." -> parent , then "lab" in the parent's list
```

---

## 6. reading one line of the table

```
path              i-node   i-node block   data block(s)   contents
/users/raj/lab    37       6              55              .  37  ..  19  main.c  52
```

- lab is a directory , which is a kind of file
- its i-node is #37 , and that i-node sits on disk block 6 (2 + floor(37/8) = 6)
- its data is on disk block 55 (i-node 37's direct pointer 0 says "55")
- a directory's contents are always "name -> i-node number" :

```
.       37    "me"         = i-node 37            (NOT block 37)
..      19    "my parent"  = raj's i-node 19      (NOT block 19)
main.c  52    "my child"   = main.c's i-node 52   (NOT block 52)
```

inside a directory's contents , every number is an i-node number .
disk block numbers only show up in the "i-node block" and "data block" columns

---

## 7. the pointers : the only place with layers

once you have ONE i-node , its pointers find the file's data blocks :

```
the i-node for notes (i-node 42)
  direct 0   -> data block 43          one hop
  ...
  direct 11  -> data block 54
  single indirect -> [ index block ] -> data blocks     two hops
  double indirect -> [ index ] -> [ index ] -> data      three hops
  triple indirect -> 3 index levels -> data              four hops
```

- small files : direct pointers only , 1 hop
- big files : indirect pointers , more levels , more block reads

a big file still has exactly ONE i-node . the indirect pointer points to an INDEX block :
a block in the data area that holds only a list of more block numbers .
it is not an i-node , and it doesn't use i-node numbers

```
i-node 42 (the ONLY i-node for this file)
  direct 0 - 11    -> 12 data blocks
  single indirect  -> index block (in the data area)
                        [ 61 | 62 | 63 | ... ]   <- just block numbers
                           |    |    |
                           v    v    v
                         data blocks
```

like page tables :

```
memory (page tables)                         file system (i-node pointers)
virtual page number -> which frame?          file block number -> which disk block?
multi-level page table                       single / double / triple indirect
one table entry = one pointer to a frame     one pointer = one disk block number
more levels = bigger address space           more levels = bigger file
```

---

## 8. limits

the i-node's pointers set the max file size :

```
block 1 KB , a block number = 4 bytes -> one index block holds 1024 / 4 = 256 numbers

direct            12                 blocks
single indirect   256                blocks
double indirect   256 * 256          = 65,536 blocks
triple indirect   256 * 256 * 256    = 16,777,216 blocks
total            ≈ 16.8 million blocks * 1 KB ≈ 16 GB max file size
```

the size of the i-node section is chosen once , when the disk is formatted (mkfs) ,
and recorded in the super block . after that it is fixed :

```
i-node section = K blocks  ->  K * 8 i-nodes  ->  at most K * 8 files (and dirs , links)
```

i-node 42 does NOT mean only 42 files , 42 is an ID , not a count
