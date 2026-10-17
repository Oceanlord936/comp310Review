// 12 questions (80 mins)

Into to Os

Os mostly in the backgournd and stll imposes control

whith given case :
eg reading hello form keyboard

launching prorgam form a disk

printing to screen

what kind of syscall does these use ??
can see these kind question in exam

---

we saw dual mode operation of the machine : user vs kernel
which was anchored on privilaged instuction supported by the cpu

---

some sample question to think about :

1. descide teh value of the timer

2. read the clock

3. clear memory

4. disable interrupts

5. switch from user mode to kernel mode

6. access an I/O devices

for each , say whether it is part of the kernel , an application , or a library
taht runs inside an app ...
identify what part is responsible

---

traps:

syscall , interrupts , exceptions

---

key point of observation

kernel does bare seesentials and important things , defer the rest to user space

how information is passed between diff componnets in a secure manner

recall the labs where we handle the syscall

eg:
"copy out" method ... ect

recall the 6 family of syscall and how they are mostly used

the majority function xv6 implemented.

---

some question :

for each event , name the casue (syscall , interrupt , execption)
say whether is synchronous or async and answer exact cases

prorgam call write()

timer fire whie a proagram is in a tight loop , what the kernel do next

a prgram jump into code that has not been loaded into memory yet . has any gone wrong ?

---

a very simple OS lets user app disable interrupts (not possible but imagin !)

-> what issue can it cause ? One is immediate , what else breaks

whould the problems affect the offending prorgam itself , or only everthing around it ?

...

---

prorcesses :

we know the UNI- prorgamming , Multi-programming

if we has single core cpu , the multi -programming is still (or even more) important , why ??

give a secnario you can illustrate that multipogramming can complete the task sonner !

memero image of progracess , major portion of memeroy map , how prorgam compenents map into them ?

given a C prorgam , place its element into memory map .
-> eg , some goes to stack , heap , text , data , bss , ....

when the prorgam already run , enter some execution what would be the memory map then

---

process creation and control

fork() , exec() , wait() are system calls that are very important in process creation and control

develop a small shell using them - you can expand on what we did in class / lecture

the sheel is a program where what you type is executed in a sub-process created by the shell , why ??? (like it first run fork() , then exec() , wait() ...

think of the details
)

know the pesudo code of a shell

when woudl the shell not do that ??
-> some cases , the sheel itself would run it
eg: "cd"

think about this , know why

how the shell is handling failures , that is program crashes
eg: the implemention of code under exec()... under child branch

---

cpu sheduling

HAS TWO aspects: mechanism and policy

a scheduler can be preemptive aor non preemptive , preemptive shceduler interuposes itself asynchonouly
(not always on regular time intervals)

policy determins what next job we would process

some polices require lot of job attributs including their resources consumption profiles , hard to get

scheduler can be measured using many metris

use cases of the OS disctate what kind of schedulers we use

---

Memory management

memory isolation , allocation

fragmentaion: both extrenal and internal - when would each be a concern ?

memory managment unit - hardware and os interplay for efficent memory management

first fit , next fit , worst fit , best fit ...
know how to use them when given an example .

---

virtual memory

locality of memory access is the key prorpety of explited by VM

locality is a proterty of the application , if there is no locality in the applications vm would not perform well

you have trashing if the working set , set of pages needed by applicatoin cannot fit in the available memory

eg: one app touch a page , need to free some page , but that page is needed by another application . it becomes perpetual page fault , keep swap , the entire thing runs slow , limited by I/O .

think of this cases

trashing is a situation where teh os is continueouly page faulting and servicing them , since page fault serving is IO intensive the cpu cannot run (stay idle)
cpu utilization is low

======================================

eg:

Consider a logical address space of 64 pages of 1024 words each, mapped onto a
physical memory of 32 frames.

a. How many bits are there in the logical address?

b. How many bits are there in the physical address?

=========================

considering a demanding paging system with the follwoing time measure utilizatoin

cpu 20%
paging disk 97.7%
other I/0 5%

for each , indicate it will improve cpu utilization or not , give reason why ?

a. install a fater cpu
-> not working , as cpu is not the bottle neck
cpu utilization is already low

b. install a biger paging disk
-> Notice the paging disk busy is 97.7% (casued by large amount of page fault
when cannot find it insdie the memory
)
it is not size related .
a larger disk not helping

c. increase degree of multiprogramming
-> will make it worst

recall the most optimal of multiprograming is not leaier

if we too less multiprograming , it will not use cpu much
when too much multiprograming , it will cause too much of memory bound issue.

how do we know we are not using enough multi prorgamming or using too much ?

-> check the paging disk usage (when usage of paging disk is high it means we are running into too much multi progamming )

d. decrese the degree of multiprogramming

it will help .

e. install more main memory

will help . when we increase memory size , it shift the cpu optimiaztion curve to the right

allowing more multipgraming , which will make it more cpu utilization

f. increase the page size

when inrease size of each page , the physical memory no change of size
this can cause more applciation that jump around , increase framentation ,
casuing more page disk fault .
-> most likely not helping

idea: as paging disk is very high . it indicates cpu is not finding the memory it want , causing page fault , need a lot of paging i/o

========================

Consider a logical address (virtual )space of 256 pages with a 4-KB page size,
mapped onto a physical memory of 64 frames.

a. How many bits are required in the logical address?

for offset bits .
think of 4 KB that is the needed offset of 4 times 1024 bytes = 4096

that is 2 to power ? of 4096
that ? is 12

so we need 12 bits of offset bits

think of

page number | offset (12 bites)

b. How many bits are required in the physical address?

c. consider a case
a RISC cpu has 39 bits

4KB page

39 bits

given PTE = 4 bytes

tell 4KB how can be break that into offset
design the map

12 bits for offset , that is 39 - 12 = 27 for remain page number

as we know we want to save space by use subpage
and we know for 10 bits makes it fit physical memory nicely

and we want to keep the uneven fit bit at top

so we have

page number | offset

total 39 bits

7 | 10 | 10 | 12

we keep the 7 at top . as keeps less internal fragmentaion

why we use 7 10 10 ? ?

if we make it leanier global page table
2 to power 27 page table . that page table becomes too big !
taking too much space of memory

we want to "chain" them together , one page root 7 bits , that map to more 10 bits table , each is PTE (page table entry) that is give PTE = 4 bytes

4096 / 4 = 1024 rows -> that is 2 to power of 10 rows . need 10 bits to reasperant taht many rows

so this is why the PTE can determin the 10 bits for each sub table bits
we want the page table to be "fully used"
that is why we want internal frag only at top level -> make it 7 | 10 | 10 | 12

=============================

Consider the following page reference string:
7, 2, 3, 1, 2, 5, 3, 4, 6, 7, 7, 1, 0, 5, 4, 6, 2, 3, 0, 1

Assuming demand paging with three frames, how many page faults would occur for the

## following replacement algorithms?

a. LRU replacement

as demand paging three frames ,can keep the 3 frame

initally miss

index:

0 1 2 3 4

7 7 7 1 1
--2 2 2 2
----3 3 3

x x x x v

at index 3 , "1" comes, kick out 7
at idnex 4, 2 comes , already inside , keep

---

b. FIFO replacement

just a queue

c. Optimal replacement

The idea is simple, for every reference we do following:

If referred page is already present, increment hit count.
If not present, find if a page that is never referenced in future. If such a page exists, replace this page with new page. If no such page exists, find a page that is referenced farthest in future. Replace this page with new page
