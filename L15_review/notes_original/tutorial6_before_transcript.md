## tutorial 6 - counting disk reads / writes in a file system

---

## the setup

given 1 KB blocks (disk blocks)

i-node with direct , single indirect , double indirect , triple indirect pointers
one i-node is 128 bytes

file system structure on disk :

```
block 0       block 1        block 2 , 3 , 4 ...      later blocks
[ boot block | super block | i-node blocks ......... | data blocks ... ]
```

how many i-nodes fit in one block :

```
1 KB / 128 bytes = 1024 / 128 = 8 i-nodes per block
```

which block holds i-node number x :

```
i-node block of x = 2 + floor( x / 8 )
```

the i-node area starts at block 2 (after boot + super) , and every block holds 8 i-nodes .
it gives the i-node BLOCK , not the data block number

check with the table below :

```
i-node 1   -> 2 + floor(1/8)  = 2 + 0 = block 2
i-node 10  -> 2 + floor(10/8) = 2 + 1 = block 3
i-node 19  -> 2 + floor(19/8) = 2 + 2 = block 4
i-node 20  -> 2 + floor(20/8) = 2 + 2 = block 4      (19 and 20 share block 4)
i-node 37  -> 2 + floor(37/8) = 2 + 4 = block 6
i-node 42  -> 2 + floor(42/8) = 2 + 5 = block 7
i-node 52  -> 2 + floor(52/8) = 2 + 6 = block 8
```

i-node 0 to 7 are in block 2 , 8 to 15 in block 3 , 16 to 23 in block 4 , and so on

---

## the counting rules

we count reads / writes of DISK BLOCKS

(1) every block read counts 1 , call it R

(2) every block write counts 1 , call it W

assume there is NO caching
every system call starts from scratch (nothing is remembered from the previous system call)

(3) changing PART of a block = read the whole block , change that part in memory , write the whole block back

```
change part of a block = 1 R + 1 W
```

why : the disk can only read and write WHOLE blocks , never 128 bytes alone .
eg an i-node is 128 bytes , but its block holds 8 i-nodes .
to change one i-node , read the whole block into memory , change your 128 bytes ,
write the whole block back (the other 7 i-nodes are written back unchanged)

same for a directory : adding / removing one entry changes part of the directory's data block -> 1 R + 1 W

(4) after open() , the file's i-node is cached in memory
-> it costs 1 read at open , then the read() / write() calls don't read it again

(5) ignore time stamps (don't count writing access / modify times into the i-node)

=======================================

## the example file system

```
path                     i-node   i-node block   data block(s)   contents
/                        1        2              40              .  1   ..  1   users  10
/users                   10       3              41              .  10  ..  1   raj  19
/users/raj               19       4              42              .  19  ..  10  notes  42  lab  37  m  20
/users/raj/notes         42       7              43 - 54         file , 11500 bytes = 12 blocks
/users/raj/lab           37       6              55              .  37  ..  19  main.c  52
/users/raj/lab/main.c    52       8              56 - 59         file , 4 blocks
/users/raj/m             20       4              60              symbolic link , block 60 holds the
                                                                 path "/users/raj/lab/main.c"
```

the same table , drawn on the disk , block by block :

```
block   area            what is inside (from the table)
0       boot block      -
1       super block     the layout of the disk (sizes , where each area starts)
------- i-node area : 8 i-nodes per block , block = 2 + floor(x / 8) -------
2       i-nodes 0-7     i-node 1  = /
3       i-nodes 8-15    i-node 10 = /users
4       i-nodes 16-23   i-node 19 = /users/raj   AND   i-node 20 = /users/raj/m
5       i-nodes 24-31   (none used in this example)
6       i-nodes 32-39   i-node 37 = /users/raj/lab
7       i-nodes 40-47   i-node 42 = /users/raj/notes
8       i-nodes 48-55   i-node 52 = /users/raj/lab/main.c
...     more i-nodes    (the i-node area ends somewhere before block 40)
------- data area : starts at block 40 in this example -------
40      dir data        /                  : .  1   ..  1   users  10
41      dir data        /users             : .  10  ..  1   raj  19
42      dir data        /users/raj         : .  19  ..  10  notes  42  lab  37  m  20
43-54   file data       /users/raj/notes   : file blocks 0 - 11 (12 blocks)
55      dir data        /users/raj/lab     : .  37  ..  19  main.c  52
56-59   file data       main.c             : file blocks 0 - 3 (4 blocks)
60      link data       /users/raj/m       : the text "/users/raj/lab/main.c"
```

so the two kinds of blocks you read are :
- an i-NODE block (2 - 8) : holds the i-node -> tells you WHERE the data is (and size , type , link count)
- a DATA block (40 - 60) : a directory's list of names , OR a file's bytes , OR a link's path

a directory is just a file whose data is a list of (name , i-node number) pairs

how to read a row : "/users has i-node 10 . i-node 10 is stored in disk block 3 .
the directory's data (its list of entries) is in disk block 41 . inside : . is me (10) ,
.. is my parent / (1) , raj is i-node 19"

- the root is its own parent , so its .. is also 1
- notes : 11500 / 1024 = 11.2 -> round UP -> 12 blocks , file blocks 0 ... 11 , all in the 12 DIRECT pointers

---

## path lookup (worked examples , same rules)

to get from a path to a file , for EACH part of the path :

```
read the i-node block of the directory   -> find where its data block is
read the directory's data block          -> find the next name's i-node number
```

and at the end , read the i-node block of the file itself

example A : open("/users/raj/lab/main.c")

```
block 2    read i-node 1 (root)          -> root's data is block 40
block 40   read root's entries           -> users = i-node 10
block 3    read i-node 10                -> data is block 41
block 41   read /users entries           -> raj = i-node 19
block 4    read i-node 19                -> data is block 42
block 42   read /users/raj entries       -> lab = i-node 37
block 6    read i-node 37                -> data is block 55
block 55   read /users/raj/lab entries   -> main.c = i-node 52
block 8    read i-node 52 (main.c)       -> now cached (rule 4)

open = 9 R
```

then reading the whole file : 4 data blocks (56 - 59) , i-node already cached -> 4 R

```
open + read whole main.c = 9 + 4 = 13 R
```

example B : open("/users/raj/m") , m is a SYMBOLIC link

```
blocks 2 , 40 , 3 , 41 , 4 , 42    get to /users/raj and find m = i-node 20     6 R
block 4                            read i-node 20 (no caching -> read block 4 again)   1 R
block 60                           read the link's data = the path "/users/raj/lab/main.c"   1 R
then look up that path FROM THE START (example A)   9 R

open via the symbolic link = 6 + 1 + 1 + 9 = 17 R
```

a symbolic link is a file that holds a PATH , so following it means a whole new lookup .
a hard link would point straight at i-node 52 , no second lookup

notice : i-node 19 (raj) and i-node 20 (m) are both in block 4 .
with no caching you still read block 4 twice , once for each i-node

===========================================

## problem 2 : a sparse file

```
open(path , O_RDWR | O_CREAT)     new file , read/write
seek(fd , 13500)                  jump to byte 13500 , nothing is written on the way
write(fd , buf , 1)               write ONE byte there
```

what the file looks like :

```
byte 0 .................................. 13499 | 13500
[        hole : never written , no blocks      ] [1 byte]
```

- length of the file = 13501 bytes (bytes 0 ... 13500)
- real data = 1 byte
- this is a SPARSE file : a large length , but almost all of it is a hole (never written -> takes no disk blocks)

ls shows two different numbers :

```
ls -l     the LENGTH of the file             13501 bytes
ls -s     the SPACE it really takes on disk  in 1 KB blocks
ls -ls    both
```

the space is 2 blocks : the data block + the single indirect (index) block

which file block holds byte 13500 :

```
file block = floor(13500 / 1024) = 13        (13 * 1024 = 13312 <= 13500 < 14336 = 14 * 1024)
```

the i-node after the write :

```
direct 0 - 11      NULL NULL ... NULL          file blocks 0 - 11 , all hole
single indirect -> [ index block ]
                     entry 0   NULL            file block 12 , hole
                     entry 1   -> data block   file block 13 , holds the 1 byte
                     entry 2 ...  NULL
double , triple    NULL
length             13501
```

only ONE data block holds data - the one with the byte (plus the index block that points to it)

reading a hole :
- the pointer is NULL -> there is no block on disk -> the OS just returns zeros
- the i-node is already in memory (cached at open , rule 4) -> if the hole is under a DIRECT pointer : 0 R
- if the hole is under the single indirect (eg file block 12 here) , you must first read the index
  block to find out entry 0 is NULL -> 1 R

=============================

## problem 3 : seek + read in a file with a hole

```
open("/users/raj/test" , O_RDWR | O_CREAT)    new file , the i-node starts all 0 (every pointer NULL)
seek(fd , 7500)
write(fd , buf , 1)              one byte at offset 7500

... after a while

seek(fd , 2500)                  seek only changes the offset kept in memory -> NO disk access
read(fd , buf , sizeof(buf))
```

creating the i-node : assume it starts empty (all 0) -> no extra R for it

the i-node after the write :

```
direct pointer   0      1      2      3      4      5      6      7            8 - 11
file block       hole   hole   hole   hole   hole   hole   hole   DATA         NULL
bytes            0-1023                                           7168 - 8191
```

7500 / 1024 = 7.32 -> file block 7 -> direct pointer 7 (the 8th one , counting from 0) is the only one set .
length = 7501

the question : how many disk reads for the seek + read at the end ? (i-node cached since open)

```
rule : file block = floor(offset / 1024)
       pointer NULL (hole) -> 0 R
       pointer set         -> 1 R (read that data block)

seek    offset / 1024    file block   pointer        reads
2500    2.44             2            NULL (hole)    0 R
7400    7.23             7            DATA           1 R
7165    6.997            6            NULL (hole)    0 R
```

the trick in 7165 : block 7 starts at 7 * 1024 = 7168 . 7165 is just 3 bytes BEFORE it -> still block 6 , a hole .
-> always compute floor(offset / 1024) , don't eyeball "about 7000"

(this assumes the read stays inside one block . if read() asks for more than 3 bytes at 7165 ,
it crosses into block 7 (byte 7168 and up) -> 1 R)

=============================

## problem 4 : appending to "notes"

/users/raj/notes is 11500 bytes (bytes 0 ... 11499) , file blocks 0 - 11 , all 12 direct pointers used
(see the table above) . append 3000 bytes at the end

how much room is left in the last block :

```
11500 mod 1024 = 11500 - 11 * 1024 = 11500 - 11264 = 236 bytes used in file block 11
1024 - 236 = 788 bytes free
```

where the 3000 bytes go :

```
788 bytes      fill the rest of file block 11     old block , already has 236 bytes
1024 bytes     file block 12                      new
1024 bytes     file block 13                      new
164 bytes      file block 14                      new , only partly filled
----
3000 = 788 + 1024 + 1024 + 164

new length = 11500 + 3000 = 14500 , 14500 mod 1024 = 164 -> ends at offset 164 inside block 14
```

3 NEW blocks , 2 of them full and 1 with 164 bytes

the i-node view :

```
direct 0 - 10      full blocks , unchanged
direct 11          file block 11 : 236 old + 788 new -> now full
single indirect -> NEW index block
                     entry 0  -> new data block (file block 12)
                     entry 1  -> new data block (file block 13)
                     entry 2  -> new data block (file block 14)
```

all 12 direct pointers are taken -> file blocks 12 , 13 , 14 must go through the single indirect -> a new index block

counting (the i-node was cached at open) :

```
what                              why                                            R    W
file block 11                     change PART of a block with old data (rule 3)  1    1
new data blocks 12 , 13 , 14      brand new , nothing to keep -> just write      0    3
new index block                   brand new -> just write                        0    1
the i-node                        set its 13th pointer (the single indirect)     1    1
                                  + the new length . the i-node is only part
                                  of its block (8 per block) -> R + W (rule 3)
-----
total                                                                            2    6
```

append 3000 bytes = 2 R + 6 W

- the 13th POINTER is the slot inside the i-node (after the 12 direct) . changing it = changing the i-node -> R + W
- the index BLOCK is a new data-area block that the pointer points to -> only W
- block 14 holds only 164 bytes but it is NEW , so no read first . rule 3's read is only needed when the
  block has OLD data you must keep (like block 11)
- not counted : the free-block bitmap that records the 4 new blocks

=============================

## what these problems teach

```
situation                                   cost
seek()                                      0 , only moves the offset in memory
read a hole (direct pointer NULL)           0 R , returns zeros
read a block that has data                  1 R
reach a block past the 12 direct pointers   + 1 R for the index block (reading)
                                            + 1 W for a new index block (growing the file)
change part of an OLD block                 1 R + 1 W
write a brand new block                     1 W only
change the i-node (pointer , length)        1 R + 1 W of its i-node block
```

always find the file block first : file block = floor(offset / 1024) , then look at its pointer

---

to check when the transcript comes :
- does the root i-node count as a read in path lookup (counted here)
- main.c size : 4 blocks (56 - 59) hold at most 4096 bytes
- problem 3 : the file must be open for reading too , or read() fails
- does the T-lab count the free-block bitmap
