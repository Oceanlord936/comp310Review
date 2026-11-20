===========================================

Another question 

a file

open(xxxx , RDCOR)

seek(fd , 13500) from the beginning to the byte location some random loaction 

write(fd , buf , 1) ; 

Notice that file is just 1 byte

but if we do ls -l it will still show the length of the file , not the size of file 


only when doing ls -ls , that will show 13500  1 	

this is a "sparce" file , a large file but most blank 

from i -Node point of file 

13500 / 1024 = block 13 

it has structure of i-node
taking the block , split you bytes -> but only the last block that holds the 1 byte data 

that is the only block u need to write 

but when comes to i-Node 

the i-Node need to remeber the file length ,  that is 13500 , i-Node also has the 12 direct pointers 
they will be NULL 

There is also single direct , that will be the block 13 , the first one an that is the value of bytes 
the reset inside that block is NULL

if someone try to read , it find out the "hole" 

the read fall into a hole , how many disk read Do it need , if the i-node is already in memory ?

-> 0 , as aleady form I/O it is alreayd in memory when u open() it 


=============================
another example:


/user/raj/test          open for write only 

seek(df , 7500);
write (fd , buf , 1);

... after a while 

seek(fd , 2500);   // seek does not read 
read(fd , buf , sizeof(buff));

ask: how many read operation have we down at this section (after after a while , the last 2 line section) ??



---
what is the read for each seek() + read() section ? (same question diff veriation )

seek(df , 7400);
read (fd , buf , 1);

as 7.32 block 

1R
---

seek(fd , 7165)
read(fd , buff ,)

0R

as 6.95 block 
---

understand the above question :

i-Node 

there is 12 emotyblock , the 13th is the one with data 

7500/1024 = block 7 

using the direct pointers

as we did seek(fd , 7500)

0 1 2 3 4 5 6  for the blocks (look vertical when drawing)
the 7th block has direct pointer sticking out 

0 disk read . as we count only R-> as the i-node alread stored in the memory 


reacall we look at the i-Node , it has 12 direct pointers , block size 1024
given offset 7500

that is 7500/ 1024 = 7.xxxx


7165/ 1024 = 6.xxxx

we know the first 7 block is null , as we did seek(fd 7500) and write () 
-> start from all 0 i -Node (when we new create it is all zero ) 
as we write to there  , only that dtablock and linking of to i-node it is the index 7 block has the data 

for the creating of i-Node , assume nothing is there -> so no additinal R 


---

assume tracking the "notes" file 

same as previous setup

rag/notes   11500 bytes 

which is 0 to 11,499 

writes 3000 bytes 

beocme appended to end 

11500 | 3000 

the last block in the 11500 part is half full (from previous ?)
the the rest go to the next block 

How much room in the last block ???

11500 mod 1024 = 236 

that is the taken not full block at end (before pending)
1024 - 236 = 788 remian empty in there ... 

so it is :
```
 11500 (end with 788 empty free) | ... want to appedn 3000
```

as we need wriet 3000 -> that need 2 full block 
so that is 
788 filll up the empty oart (need to read the last has empty space block first) then wiret teh reamin 2 block
end at offset 164 

so how many Read have we done ?? 


---
explain form the i-node perspective 

11500 notes length 

11500/1024 , that is 11.23 
the i-node dierect pointer -> already taken -> all 12 of them 

all of them point to 12 data blocks (12 of them)

all if them ahs teh datat for the node parts 

the last one has 788 free space (as we show above the calculation)
we first fill it up 

-> for write 3000 , nede to create 3 bock 
but we see there is no more direct pointer 
-> need to create a index block , that index block will point to the 3 new data blocks 

that is R + W for the first 11500


R + w FOR THE 13th  after the 12 direct pointer (as we do not know if there is anything in there )
W for the index block , and 3 W for the 3 new data block 

So , in total ... 6 W , 2 R 


