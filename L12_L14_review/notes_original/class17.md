## continue lecture 12

Computing Systems Performance
How to measure performance and maximize it

==============================================

## Amdahl's Law

(fix: the slide spells it "Amdhal", the correct spelling is Amdahl)

the idea : if you speed up only part of a task , the part you didn't touch still takes just as long .
that untouched part limits how much faster the whole task can get .

Portion that can be enhanced | Portion that cannot be enhanced

```
Told :  [ black: a * Told ][ teal: (1 - a) * Told ]
                 |                    |
          divided by k          stays the same
                 v                    v
Tnew :  [black][ teal: (1 - a) * Told ]
```

formula (from the slide) :

```
Tnew = (1 - a) * Told + (a * Told) / k

Tnew = Told * [(1 - a) + a / k]

Speedup S = Told / Tnew = 1 / [(1 - a) + a / k]
```

a = fraction of the ORIGINAL TIME that can be enhanced (the black bar)
k = how many times faster that part gets
1 - a = the part that stays the same (the teal bar)

(fix: your notes had two notations, f/s from me and a/k from the slide. They are the same thing
f = a , s = k . Use a and k on the test since that is what the slide uses)

tip : always work in TIME , not distance or speed . a is a fraction of time .

tip : when solving for a or k , use the flipped form , no fraction to deal with :

```
1 / S = Tnew / Told = (1 - a) + a / k
```

upper limit : even if k -> infinity , the enhanced part takes 0 time

```
S_max = 1 / (1 - a)
```

the part that cannot be enhanced caps the speedup .
this is the orange box on the slide : "improvement is limited by the fraction of the time the
enhancement can be applied"

==============================================

## Example Problems

Speedup computation using Amdahl's Law

```
Suppose you work as a truck driver, and you are driving
from Boise, Idaho to Minneapolis, Minnesota, a total
distance of 2500 kms. You estimate you can average 100
km/hr driving within speed limits for a total of 25 hours
for the trip.
```

---

### Q1 : Montana has no speed limit (1500 km of the trip), the truck can do 150 km/hr. What is the speedup ?

2500 / 100 = 25 h original

split it into time :
Montana : 1500 km at 100 km/h = 15 h <- can be enhanced
rest : 1000 km at 100 km/h = 10 h <- cannot be enhanced

new trip :
Montana : 1500 / 150 = 10 h
rest : still 10 h
total = 20 h

S = 25 / 20 = 1.25

with the formula : a = 15 / 25 = 0.6 , k = 150 / 100 = 1.5

```
S = 1 / [0.4 + 0.6 / 1.5] = 1 / [0.4 + 0.4] = 1 / 0.8 = 1.25
```

Montana gets 50% faster but the whole trip only gets 25% faster , because 10 h never changed

---

### Q2 : with a turbocharger , how fast through Montana for an overall speedup of 1.67 ?

(fix: your notes had "total = 25 / 1.67" sitting under Q1. That line is the start of Q2)

target time = 25 / 1.67 = about 15 h

the 10 h outside Montana is fixed , so Montana can only take 15 - 10 = 5 h

1500 km / 5 h = 300 km/h

with the formula : a = 0.6 is known , solve for k

```
1 / 1.67 = 0.6 = 0.4 + 0.6 / k
0.6 / k = 0.2
k = 3            -> 3 * 100 = 300 km/h
```

max possible : S_max = 1 / 0.4 = 2.5 (crossing Montana in 0 time still leaves 10 h)

---

### Q3 : application takes 30 min on 1 core , 18 min on a 4 core machine. What portion could be enhanced ?

this time S and k are known , solve for a

k = 4 (4 cores -> the parallel part runs 4 times faster)

```
Tnew / Told = 18 / 30 = 0.6 = (1 - a) + a / 4
0.6 = 1 - 0.75a
0.75a = 0.4
a = 0.533
```

about 53% of the application can be parallelized (16 of the 30 min)

check : serial part = 14 min , parallel part = 16 / 4 = 4 min , 14 + 4 = 18 min ✓

max possible on infinite cores : S_max = 1 / (1 - 0.533) = about 2.14

---

pattern for all three :

1. split the original TIME into enhanced part and unchanged part
2. shrink only the enhanced part (divide by k)
3. S = Told / Tnew
4. unknown a or k -> 1 / S = (1 - a) + a / k , solve

==============================================

notice Amdahl's law does not apply to newer workloads

newer is more Gustafson's law

(fix: your notes said "newer unit". The point is that Amdahl assumes the problem size is FIXED .
you only ask "how much faster does the same job finish". With big machines we usually grow the
problem instead (more data , more images , bigger models) . the parallel part grows and the
serial part stays small , so adding machines keeps paying off . that is Gustafson's law)

(my addition, not on this slide set yet , check if the prof gives a formula) Gustafson's scaled speedup :

```
S = (1 - a) + a * N
```

N = number of processors , a = parallel fraction measured on the N-processor run

Amdahl : fixed problem , speedup capped at 1 / (1 - a)
Gustafson : problem grows with N , speedup grows about linearly with N

=============================

Scalable Multi Processing Systems

• SMP has a performance bottleneck
(restriction) - system bus
• Instead of a giant shared memory - distribute
the memory - but retain a shared address space
• Portions of the address space is local and rest
is global
•OS can manage locality to do memory
accesses fast
•Non-uniform memory access (NUMA)
machines
Symmetric Multi Processing is Not Scalable

================================

Cluster Computing Systems
Cheap Approach to High Performance for Some Applications

Clusters are cheap way of obtaining high
performance - you can repurpose existing
machines - connecting all lab machines

• Loosely coupled - takes longer to send a message
from one machine to another than in a SMP or
NUMA

• For certain applications that send few but large
messages, cluster computing can provide high
performance

==========================

## lecture 13

Motivation for Multi-Threading

Most modern applications are multi-threaded
• Threads run within a process
• Multiple tasks within the application can be
implemented by separate threads

Update display
• Fetch data
• Spell checking
• Answer a network request

---

Motivation for Lightweight Processes

Process creation is heavy-weight — application can be slowed down by process
creation, termination, and management overhead

• We can use process pools to reduce the overhead of creation on demand

• Thread creation is much more lightweight compared to process creation

• Threads also simplify code (modularization) and increase efficiency

• Kernels are generally multithreaded

===========================

Process Vs. Threads

Processes

• Create a new address  
space at process creation
• Allocate resources at
creation
• Need IPC to share data
• Deeper isolation for
security and fault
tolerance

---

Threads

• Same address space
• Quicker creation times —
actual times depend on
kernel versus user threads
• Sharing through shared
memory
• Fault sharing between all
threads within a process

===================

User Vs. Kernel Threads

• User threads - management done by user-level thread library (GNU Pth)

• Kernel threads - supported by the kernel

Examples - virtually all general purpose OSes provide kernel level threads including:
Windows, Linux, MacOS

• POSIX Threads - is a thread programming standard - most of the time implemented
as kernel-level threads

==========================

Different Threading Models

User-Level Threads
Many-to-One
• Many user-level threads mapped to
single kernel-level thread
• One thread blocking causes all to
block
• Multiple threads cannot run in
parallel in multi-core system
because kernel level has one thread
• GNU Pth is one example

---

Kernel-Level Threads
One-to-One
• Each user-level thread maps to a
kernel thread
• Creating a user-level thread creates a
kernel thread
• More concurrency than many-to-one
• Number of threads per process
sometimes restricted due to
overhead

---

Hybrid Threads
Many-to-Many
• Allows many user level threads to be
mapped to many kernel threads
• Allows the OS to create a sufficient
number of kernel thread
• At any given time a user thread is
mapped onto a single kernel level
thread
• Locality of mapping is a concern

====================
Example Problems
User-Level Versus Kernel-Level Threads

```

• We have an application that is implemented using a
threading library. The application is written with 2
threads. Each thread has 500 sec completion time (that
is a thread would take 500 sec of CPU time once it is
started - note: a thread cannot run on multiple CPU
cores). The application completes when both threads
are done.
• What is the application completion time in a single core
machine if it is implemented using user-level threads?
• What is the application completion time in a single core
machine if it is implemented using kernel-level threads?

```

======================

Asynchronous Vs. Synchronous Creation

Asynchronous threading

• Parent spawns a child thread, parent
resumes execution

• Parent and child continue concurrently
and independently of one another

• In a typical deployment, parent and child
share little data

---

Implicit Threading

• We have CPUs with many cores - how to parallelize code to exploit the hardware
parallelism (we are ignoring the GPU side of things)

• Option A:  
• Ask the programmer to explicitly organize the code using threads
• Ask programmer to split the data

• Option B:
• Let programmer identify tasks - use automatic ways of running tasks concurrently
by mapping them to available threads - task is a function

---

end at page 19
