## multi thread 

----

## Fork-Join Parallelism

Example Problems
Large-Scale Data Processing

```
Consider a large-scale data processing problem with 1024 
records. Each record processing is CPU intensive. We 
want to breakdown the problem until we have sub 
problems with 64 records.  

• Suppose we use the fork-join mechanism shown in the 
previous slide. How many levels would you have in the 
fork-join graph?  

• Suppose we have 32 processors in the machine, how many 
processors would we use in the above split? 

• Suppose processing a record takes 1 seconds, what is the 
runtime? Note: ignore the splitting times and combining 
times

```

start with 1024 , and we want to breakdown it till we have sub problem
where each has 64 records.

call "fork" div intp 512 , 512 

each call again , div into 256 

then 128 

then reach 64 ;

that is 2 , 4 , 8 ,  till me reach 16 thread ;

assume we have 32 processors in machine , ask for how many processors would we use in the above
sipose process each is 1 sec . 

that is 64 record , each is 1 sec, so  64 sec 

as working in paralle does use 64 plus additinal combine time 
20 ms for one combine 

for 16 of threads , i and i +1 threads will be combined 
that is 0 ,1 ;  2 , 3 ; .. 14 , 15  that is 8 pair
as work in parelle cost 20ms

and next layer still need to combine 
that is 4 , then 2 , then reach 1 

so that is 4 * 20 ms takes to combine 

============================

## Open Multi-Processing (OpenMP)


use pragma -> special class of comment 
the compiler will analyse it , 

Programmers insert 
compiler directives 
(special comments) to 
indicate code that 
should be parallelized


eg:

 OpenMP is a set of 
compiler directives to 
implicitly parallelize C, 
C++, FORTRAN code


idea: if hardware not support can still run that code 
when hardware deoes support it allow faster run 

===========

## POSIX Pthreads

Pthreads is a specification 
for thread behaviour - not an 
implementation 

```c

pthread_t tid;
pthread_attr_t attr; 

pthread_attr_init ($attr);

Pthreads_create(&tid , &attr , runner , argv[1]);

pthread_join(tid , NULL);

printf("sum = %d\n" ,sum);

```

==========

Threads in Linux

• Threads have specific stacks


=================

Thread Cancellation

 User launches a request for 
downloading a webpage 

• Hits cancel to stop the request 

• We need to cancel the thread that is 
launching the network request 

• Cancelled thread is known as the 
target thread

---
 Target can be cancelled in two different scenarios: 

• Asynchronous cancellation — 
immediately terminate the target 
thread 
• Deferred cancellation — target 
thread periodically checks whether 
it should terminate in an orderly 
manner

---

Deferred Thread Cancellation

 Cancellation is initiated 
using pthread_cancel() 

Target is checking for 
the reception of cancel 
at pre-defined points 

 If a cancel is there, 
pthread_testcancel() 
will not return - thread 
will terminate


===================

N14 deadLocks 

---

What is a deadlock?
Deadlock-- permanent blocking of a set of 
processes that compete for system resources 

No
efficient solution to the deadlock problem 
in the general case.

In deadlock
a set of processes are in a wait state, because each 
process is waiting for a resource that is held by some 
other waiting process

All deadlocks involve conflicting resource needs 
by two or more processes

======

Classification of resources:

Resources grouped into two classes:

(1) Reusable: 
something that can be safely used 
by one process at a time and is not depleted
by that use. Processes obtain resources that 
they later release for reuse by others.
E.g., CPU, memory, specific I/O devices, or files

(2) Consumable: 
these can be created and 
destroyed. When a resource is acquired by a 
process, the resource ceases to exist.
E.g., interrupts, signals, or messages

---

Classification of resources—II

Another classification of resources:

(1)Preemptable: these can be taken away from the 
process owning it with no ill effects (needs 
save/restore).
E.g., memory or CPU.

(2)Non-preemptable: cannot be taken away from its 
current owner without causing the computation to 
fail.
E.g., printer or floppy disk

Deadlocks mostly occur when sharing reusable
and non-preemptable resources

============

Conditions for Deadlock:

4 conditions that must hold for a 
deadlock to be possible:

---

(1)Mutual exclusion: processes require exclusive 
control of its resources (not sharing)

(2)Hold and wait: process may wait for a resource 
while holding others

(3)No preemption: process will not give up a 
resource until  it is finished with it

(4)Circular wait: each process in the chain holds a 
resource requested by another


=====

Conditions for deadlock

Also, a process cannot be reset to an 
earlier state where resources not held

Conditions 1—3 are necessary, but not 
sufficient. All 4 are needed

=====

Solving deadlocks

If a necessary condition is prevented a 
deadlock cannot occur. 

For example:
(1)Systems with only shared resources cannot 
deadlock.– Negates mutual exclusion.

(2)Systems that abort processes which request a 
resource that is in use

(3)Preemptions may be possible if a process 
does not use its resources until it has 
acquired all it needs.– Negates no preemption

(4)Systems that detect or avoid deadlocks.– Prevents cycle.

(5)Transaction processing systems provide 
checkpoints so that processes may back out 
of a transaction.– Negates irreversible process


=================

Resource allocation graphs

eg: the Bank balance 

have account A1 and accountA2

there are also customer C1 and customer C2

and they want to transfer some money 

C1 access A1 , C2 access A2 

then they need to wait for the other is done 
then they can access the other account to do the transfer 

think of set of resource 
```
Set of Processes     P = {P1, P2, …, Pn}

Set of Resources     R = {R1, R2, …, Rm} 
```

Process Pi waits for (has requested) Rj 

Resource Rj has been allocated to Pi

when drawing graph , use circle for process P 
use square for resource R 

P ---> R   is for P wait for R 

P <--solid-- R is  rousource R has been allocatde to P 

now we see if there is an cycle inside the graph 
it is an indication for there is Deadlock

Notice: 
has a cycle does not mean it must have a deadlock
but for deadlock to occure there Must be a cycle 

---

when there is multi instance of resource it is not sufficent to 
have a deadlock

eg:

assume we have 2 "Scanner" and 2 "Printer " :
we have multiple resource 

drawing the graph:

```
          P2<---solid----Scanner (1 , 2)---solid----> P1
										^                                    |
										 |                                     |
										 |									  v
										P3     <--solid ------ Printer  (1, 2)  ---solid---> P4
```

---

 a knot is require
 Multiple resource unit case:
No Deadlock—yet!

Because, either P2 or P4 could relinquish a resource 
allowing P1 or P3 (which are currently blocked
continue. P2 is still executing  even if P4 requests R1.
)

A knot must exist—a cycle with no non
cycle outgoing path from any involved 
node.
At the moment assume that:
a process halts as soon as it waits for one 
resource, and
processes can wait for only one resource at a 
time

====================

Strategies for deadlocks

In general, 4 strategies are used for 
dealing with deadlocks:

```

(1)Ignorance: 
pretend there is no problem at all.

(2)Prevention: 
design a system in such a way 
deadlock is excluded a priori

(3)Avoidance: 
make a decision dynamically 
checking whether a request will, if granted, 
potentially lead to a deadlock or not.

(4)Detection: 
let the deadlock occur and detect 
when it happens, and take some action to 
recover after the fact.


```





