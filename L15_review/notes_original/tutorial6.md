## tutorial 6

---

example question:

given 1KB blcoks  (disk blocks)

 i-Node with direct , indirect , bouble , triple 
 that is 128 Bytes

given a file system structure looks like

```
Boot block | super block | i-Nodes | disk blocks  
```

if given i-Node "x" amount 

x - 2 + floor[x/8]     

find the data block number 

---

recall the counting :   reads / wirtes to the file system

The counting rules:

(1) every block read . call it R  

(2) every blcok write call it W 

assume there is no no caching 
every system call start form scall

(3) changing a part of the block means reading the blokc and then 



take a smaller bytes on disk 
need reda the block of whole thing into memeort , change that portion 
then read it back 


eg: in the i-Nodes section , there are a lot of i-Node
if we chaneg teh i-NOde , we cannot just chage the section and wriet it back
as some other thing could still looking at  that section .
 To hadle it we need to read the whole thing , change and then wiret it back 

so each change is read + write 

(4) Assume open file would have teh i-Node cached 

that i-node is already there  only once access to bring it in  


(5) just ugone  time stemps 

=======================================

exmaple situation:

has a root name 

here is a table :

 "name" / has i-Node , i-Node block , Data blocks , contents 
 
 normally i-Node is given 1 , i-node block 2 , data blocks 40 and conetnst   ,1 , ...1 user IO
 
 
 /users  10 , 3  , 41 ,  . 10 , ... 1, ... raj 19
 
 /users/rag   19    , 4,   , 42 , .19 , ... 10 .    mates 42 , lab37 , m20
 
 /users/rag/notes 42  , 7 , 43 - 59  , 11500 bytes , file block  0 ... 11 in direct  [0 ....11]
 
 /users/raj/lab    37  , 6 , 55 , .37,...19, main.c 52
 
 /users/raj/ lab/mian.c    52 , 8 , 56 - 59 . 4580 bytes : 4 blocks 
 
 /users/raj/ m    20 ,     4 , 60 ,  sysblic link block 60 holds to path to mian.c 