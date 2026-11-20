# Tutorial 6 - counting disk reads / writes (hand-typed redo)

clean reference : Notes/tutorial6.md

---

## 1. setup , formula , rules

Setup:

```
block size 1 KB = 1024 bytes , one i-node = 128 bytes
1024 / 128 = 8 i-nodes per block

block 0       block 1        block 2 , 3 , 4 ...      later blocks
[ boot block | super block | i-node blocks ......... | data blocks ... ]

i-node block of x = 2 + floor( x / 8 )
eg i-node 19 -> 2 + floor(19/8) = 2 + 2 = block 4

```

The counting rules:

```
(1) every block READ  = 1 R

(2) every block WRITE = 1 W
    NO caching : every system call starts from scratch
	
(3) change PART of a block = 1 R + 1 W
    (the disk only reads / writes WHOLE blocks , the other 7 i-nodes must be kept)
	
(4) after open() , the file's i-node is cached -> read() / write() don't read it again

(5) ignore time stamps

```

The example file system:

```
path                     i-node   i-node block   data block(s)   contents
/                        1        2              40              .  1   ..  1   users  10
/users                   10       3              41              .  10  ..  1   raj  19
/users/raj               19       4              42              .  19  ..  10  notes  42  lab  37  m  20
/users/raj/notes         42       7              43 - 54         file , 11500 bytes = 12 blocks
/users/raj/lab           37       6              55              .  37  ..  19  main.c  52
/users/raj/lab/main.c    52       8              56 - 59         file , 4 blocks
/users/raj/m             20       4              60              symbolic link -> "/users/raj/lab/main.c"


```

A directory is just a file whose data is a list of (name, i-node number) pairs.

Path lookup. For each directory on the path, read its i-node block, then its data block. At the end, read the file's own i-node block.

```
open("/users/raj/lab/main.c")
block 2    i-node 1 (root)       -> data in 40
block 40   root entries          -> users = 10
block 3    i-node 10             -> data in 41
block 41   /users entries        -> raj = 19
block 4    i-node 19             -> data in 42
block 42   /users/raj entries    -> lab = 37
block 6    i-node 37             -> data in 55
block 55   lab entries           -> main.c = 52
block 8    i-node 52             -> cached now
open = 9 R ,  + read whole file (56 - 59) = 4 R  ->  13 R

```

Symbolic link m (i-node 20):

```
2 , 40 , 3 , 41 , 4 , 42   reach raj , find m = 20     6 R
4                          i-node 20 (block 4 AGAIN , no caching)   1 R
60                         the link's data = a path    1 R
lookup that path from the start                        9 R
= 17 R

```

=============================================

##0. Quick i-node review (before chunk 10)

quick review for i-node:
The disk is just an array of blocks. 
The file system cuts the disk into equal pieces called blocks. 
Every read or write moves one whole block.

```
block size = 1 KB = 1024 bytes  (in this tutorial)
disk = block 0 , block 1 , block 2 , ... block N-1

```

The four areas on the disk:

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
what each section are:

(1) boot block :
	the start up code . We never touch it 
	
(2) super block: 
	describes the whole file system: how big the areas are and where each one starts.

(3) i-node blocks: 
	an array of i-nodes, one per file.

(4) data blocks: 
	the contents.

=========================

What an i-node is: 
one small record per file (or directory) that holds everything about the file except its name and its data.

```
i-node (128 bytes in this tutorial)
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

Every i-node has a number (its index in the i-node table): i-node 1, 10, 19, ...

Where's the name? In the directory. 
A directory is a file whose data is a list of name → i-node number pairs:

/users/raj data block :   .  19  |  ..  10  |  notes  42  |  lab  37  |  m  20

So to go from a name to a file: 
	the directory gives you the i-node number, the i-node gives you the data blocks.

 -> a directory is an i-node, plus data blocks that hold a list of names.

->The name of a thing is not in its own i-node. It's in its parent directory's data:

```
/users/raj        i-node 19 , data in block 42
block 42 holds :  notes 42 | lab 37 | m 20      <- the NAMES of raj's children

the name "lab" lives in raj's data block (42) , not in i-node 37

```

======================================

Reading the table, column by column

Take the /users row:

```
path     i-node   i-node block   data block(s)   contents
/users   10       3              41              .  10  ..  1   raj  19

```

```
i-node 10        the ID number of /users (its index in the i-node table)

i-node block 3   WHERE i-node 10 sits on disk (2 + floor(10/8) = 3)
                 -> read block 3 to get the i-node record
				 
data block 41    WHERE /users's content sits on disk
                 -> the i-node's pointer says "41"
				 
contents         WHAT is inside block 41

```

The contents column for a directory is a list of pairs, name i-node:

```
	.  10   ..  1   raj  19

	read as 3 pairs :
	  name "."    -> i-node 10
	  name ".."   -> i-node 1
	  name "raj"  -> i-node 19

```
meaning of each 
```
.     = ME (this directory)        -> its number is always the directory's own i-node
..    = MY PARENT (one level up)   -> its number is the parent's i-node
name  = a CHILD inside me          -> its number is that child's i-node

```
so , it is same idea as the on u tyoe in commadn terminal , ./ is relative this , .. the parent ,

```
terminal            what the OS does

cd ..               look up ".." in this directory's data block -> parent's i-node

./run.sh            look up "." -> this directory's i-node , then "run.sh" in its list

cd ../lab           ".." -> parent , then "lab" in the parent's list

```

==============================

The big fix first

There are no layers of i-nodes. 
The i-node area is one flat array of i-nodes, just stored across several disk blocks.

The formula is

```
i-node block = 2 + floor( i-node NUMBER / 8 )

```
It has one step, always, and there are never more layers.

---

1. i-node number vs i-node block: two different questions

```
i-node NUMBER   = WHICH file's record      (an ID , like a student ID)

i-node BLOCK    = WHERE on the disk that record physically sits

```

On the disk:

```
disk block 2   [ i-node 0 | i-node 1 | i-node 2 | ... | i-node 7  ]
disk block 3   [ i-node 8 | i-node 9 | i-node 10| ... | i-node 15 ]
disk block 4   [ i-node 16| ... | i-node 19 | i-node 20 | ... | 23 ]
...
each box = one i-node = 128 bytes , each row = one block = 1024 bytes

```
-> To read i-node 19, 
read disk block 4 (the whole 1024 bytes), 
then take the 4th record in it (19 − 16 = 3, counting from 0).

Where each piece of the formula comes from:

```
2           the i-node area starts at disk block 2 (blocks 0 , 1 = boot , super)

x / 8       8 i-nodes per block -> which row-of-8 i-node x falls in

floor       drop the fraction (19 / 8 = 2.375 -> 2)

```

---

2. Where real layers do exist: inside one i-node (the pointers)


This is the part that's like page tables.
 Once you have one i-node, its pointers find the file's data blocks:

```
the i-node for notes (i-node 42)
  direct 0   -> data block 43          one hop
  direct 1   -> data block 44
  ...
  direct 11  -> data block 54
  single indirect -> [ index block ] -> data blocks     two hops
  double indirect -> [ index ] -> [ index ] -> data      three hops
  triple indirect -> 3 index levels -> data              four hops

```

Small files: direct pointers only, so 1 hop.
Big files: indirect pointers, so more levels and more block reads.


---

Compared with memory management:

```
memory (page tables)                         file system (i-node pointers)
virtual page number -> which frame?          file block number -> which disk block?
multi-level page table                       single / double / triple indirect
one table entry = one pointer to a frame     one pointer = one disk block number
more levels = bigger address space           more levels = bigger file

```

















