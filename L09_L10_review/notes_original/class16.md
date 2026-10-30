## recall lecture 11

i - Node 


==============================================


eg:

let' say each block size is B and a disk pointer taskes 8 bytes ,

12 * 4 = 48 

68 bytes for the  i node

12 direct  

and we do multi level 

find the total size of iNode 

given i node table 

2000 blocks 

each is 4096 bytes 

total capacity is 4096 * 2000  = 9192000 bytes 


How many : i node are needed 

number of files are we going to store , 
i - node for direct 

for each block : (4096 / 68 ) 

and we have 2000 blcoks / 3 close 666 , round up to 700 

the average file size , assume it is 4096 * 3 

700 * (4096 / 68) around 12 blocks

that has the flexibility of at least 30 directories 

---

take care of files and directoris .

notice : the directories is also files

it is like table , each row structure depends on OS

for the minimum , need 2 

the name and the i_node 

==========================================

File System Layout

master boot record -> partation table 

-> parttaion A ->

boot block | super block | free space manage .. i-Nodes .. datat blocks
(file and dir4ecories over here) | ...

then partationB ... 


==================================================



eg: given a file directory and  a c file 

/user/acb/xyz.c 


put a pointer at the super block :
point towards the i node table 


assume 12 blocks i node table


given i node 100 

100 times 68 = 6800 
that takes 2 blocks

when visite the disk -> must take the full block into the memory t, then check the offset 

for 6800 , as each block is 4096 , 
first block is 0 to 4069 , then also need next bock 

when with give i-node number , a disk block number -> can be directly grabed 





---

eg: use the i-Node 0 , to hold the root directory (given info )

assume the inode 0 is point to futher data blocks (as many layer )

how many are there depedns on how many diretcories 

we assume the dirctories (the diretcory file itself ) is small -> a single block for that diretcory file 


for /user/acb/xyz.c 

find how many bynver if disk access to get the first byte of file xyz.c ???

first access : i-node 0 

second access : the data block - each row is a directory entry 

third access : for "user : 10" the i-node #10 
 
fourth access : within look at a datablock , there are many row entries , 
look at the "abc" , the further diretory 

it has i-node 21

fifth access: i-node 21 

inside , it has xyz.c 

xyz.c : 30 , the further i-node 

that is the one point to the file 

 
so we get the i-node 30 

now the 6th access : look at i node 30 , point ot many data block 

the first byte is in the first data block 

so , we walk many steps to resolve the path directortory , to get the first byte

takes 6 access 

---

notice : the further we go , the more access it takes to get the data 
there are cache to help , but think of how user should organize their file directories 


============================

super block point to all the configergration , 
itself is also an inode , but point to the root dir 


==============================



eg:

run command

rm ./home/xy.c 

how many disk access do we need ???

/home/xy.c 

the i-node number 31

assume it linked to e blocks

when remove the file 

the inode also has meta information 

within the meta info there is "link count" 
it tells how many diretcories is pointed to the file !

assume the linke-count is 1 

when we do the rm commnad , that count is now 0 
also remove the direct entry as well


assuume the first row data block within i node 31 is :
251 , the next is 3090 , and next is 3100

when remove all of that .

look at the free byte map

find the bit positions , we flip the bit position ->this releasing it 

as we use "1" for free , becomes 0 once we release it 


---

when we create it , it will be the opposite of this process 

need to create the row in directory file... 



===================================================



for the other file 
/user/abc/xyz.c 

assume it point to same i node 31

when we remove it , the link count is from 1 to 0 after we call rm /home/xy.c

but the /user/abc/xyz.c still holds the i-node 

this is the "hard link"


====================================================




## Lecture 12 Compute Performance   


---


## Cluster computing 

• Connection of commodity machines (off-the-shelf computers) into clusters to do 
high-throughput computing

Cheap to scale because commodity machines are cheap - priced according to the 
market 

• Not connected tightly to each other — using off-the-shelf networks so the latency 
and throughput are not the best possible 

• Clusters are good for problems where the individual tasks are relatively big

---

##  Parallel computers 
• These are specialized machines using COTS (commodity off-the-shelf) parts 
sometimes, but lot of expenses are spent on tightly connecting them 

• Look at www.top500.org for the latest Top 500 parallel supercomputers 

• Used for science and technology projects and may be AI now 

• Cloud computers 

• Massive clusters 

• Split into smaller clusters or nodes via virtualization

=============================


## Motivating Example - Compute Scaling

Consider a finance company that uses 

computers to process customer transactions
 
• It has 1000 customers and uses 1 server 

• Suddenly, number of customers increase 
to 6000 

• Finance company decided to buy more 
computers to process customer transactions - it got 6 servers 

• Investment increased 6 fold - not a problem 
they are making more money as well


---

What Happens with Demand Fluctuation?


Most of the customers at the finance 
company are students at the nearby 
university 

• Students go for summer break - activity 
drops to almost previous levels 
Time

• Cost for the finance company remains 
high 

• Not a good situation because cost 
remains high even when demand drops 


============================================

solution :

Finance company reduces the cost by 
outsourcing computing 

• Another company provides the customer 
record processing service - paid per 
customer 
On-premises  
solution

• Another company provides computing 
capacity - paid for server usage 

• Cost can dynamically change


=============================

Type of scaling 

Finance company can buy more machines 
to scale up computing capacity 

• Finance company can buy a much bigger 
machine to replace the machine it was 
using 
Horizontal Scaling
More Servers

• Which solution is better? 

• Answer: It depends on the problem - if 
the problem can be split across multiple 
machines we like the first approach


===================================

How to Scale Computing Power???

• Vertical scaling is easier to manage from 
the application side - finance company 
does not need to modify the way records 
are processed 

• Cost increases rapidly with machine size 

• Horizontal scaling needs distribution of 
data 

• Accumulation of results


====================================


How to measure performance and maximize it?

• A computers performance can be measured in 
different ways. 

• One well understood way is to measure the 
execution time of an application 

• The user wants to minimize the execution time 
of the application so they can wait the least 
amount of time to get the results 

• How to reduce the application execution times?

• Apply enhancements to 
reduce application 
execution times 

• Effectiveness of the 
enhancement depends on 
the application

===================================


Portion that can be enhanced | Portion that cannot be enhanced

Amdhal’s Law 
illustrated

if the "idea" can only be appied to some section , 
 only can be appied to some part of it 

the application is not going to be improved


Improvement that can be obtained by  
applying an enhancement is limited by  
the fraction of the time the enhancement  
can be applied to the problems


===========================================

