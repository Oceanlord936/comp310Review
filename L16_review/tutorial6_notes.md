# Tutorial 6 - counting disk reads / writes (hand-typed redo)

clean reference : Notes/tutorial6.md
i-node basics : i-node-underatnding.md

---

## 1. setup , formula , rules

setup :

```
block size 1 KB = 1024 bytes , one i-node = 128 bytes
1024 / 128 = 8 i-nodes per block

block 0       block 1        block 2 , 3 , 4 ...      later blocks
[ boot block | super block | i-node blocks ......... | data blocks ... ]

i-node block of x = 2 + floor( x / 8 )
eg i-node 19 -> 2 + floor(19/8) = 2 + 2 = block 4
```

the counting rules :

```
(1) every block READ  = 1 R

(2) every block WRITE = 1 W
    NO caching : every system call starts from scratch

(3) change PART of a block = 1 R + 1 W
    (the disk only reads / writes WHOLE blocks , the other 7 i-nodes must be kept)

(4) after open() , the file's i-node is cached -> read() / write() don't read it again

(5) ignore time stamps
```

the example file system :

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

a directory is just a file whose data is a list of (name , i-node number) pairs

---

## 2. path lookup

for each directory on the path , read its i-node block , then its data block .
at the end , read the file's own i-node block

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

symbolic link m (i-node 20) :

```
2 , 40 , 3 , 41 , 4 , 42   reach raj , find m = 20                   6 R
4                          i-node 20 (block 4 AGAIN , no caching)    1 R
60                         the link's data = a path                  1 R
lookup that path from the start                                      9 R
= 17 R
```
