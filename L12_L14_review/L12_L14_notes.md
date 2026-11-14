## Round G: rest of L12


1. Why multiple processors
It's getting very hard to make one CPU faster: i
t overheats, and clock speeds have hit their limits.
Adding more processors is the easiest way to get more performance.

---

2. SMP (Symmetric Multi Processing)

All processors are equal, and any of them can do any task, including running the kernel.
They all share one physical memory through one system bus.

The problem is that every CPU goes through that same bus. 
Add more CPUs and they start waiting for each other at the bus. 
That's why SMP doesn't scale

---

3. NUMA (Non-Uniform Memory Access)

This is the fix for SMP. Memory is split up so each processor,
 or group of processors, has its own local memory.
 
 But there is still one shared address space, so any CPU can reach any address.

 Local memory is fast and remote memory is slower. 
 "Non-uniform" means not every access costs the same.
 
 The OS manages locality: it tries to keep a process's pages on the node where that process runs, so most of its accesses are fast.

---

4. Message passing machines
They have no shared memory and no shared address space. Processors can only share data by sending messages.

---

5. Clusters

They're cheap, because you can reuse ordinary machines 
(for example, link all the lab PCs together).

The machines are loosely coupled: sending a message 
between them over the network is much slower than in SMP or NUMA.

So a cluster only pays off for jobs that send few but large messages, 
where each task is big compared to the cost of the messages.

---

From fastest to slowest 
at moving data between processors: 

SMP, then NUMA, then cluster.


---

6. How big a cluster to build (EXP-17)
Ask whether one more machine still pays for itself, and use Amdahl's law to check.

Mostly serial job: extra machines barely help, because the speedup is capped at 1 / (1 − a). 

Buy only a few.
Many independent pieces 
(one per image, one per record): 
the speedup keeps growing as you add machines, 
so fill all the space you have. 

This is the Gustafson idea from class17: 
when the problem grows, more machines keep paying off.

================

6. Cluster size with Amdahl's law

```
Tnew = Told * [(1 - a) + a / k]

Tnew = Told * [ (1 - a)    +    a / k ]
                   |              |
       part that can't change   part that can, made k times faster


```

on a cluster , k = N , the number of machines 
cause the parallel part is split across N machines

a is the fraction of the old running time that the improvement can speed up.
-> So 1 − a is the part of the time that can't be sped up:

k is how many times faster that improvable part becomes.


---

Case 1: a job that is mostly serial (EXP-17 part 1)
Training takes 10 h, and a = 0.3 of it can run in parallel.

```
Tnew = 10 * [0.7 + 0.3 / N] = 7 h + 3 h / N

```

Notice:
```
N	Runtime
1	10 h
2	8.5 h
4	7.75 h
16	about 7.2 h
∞	7 h, the cap
```
so Best possible speedup = 1 / (1 − a) = 1 / 0.7 ≈ 1.43.
where the a /N part is 0 

---

Case 2: independent pieces (EXP-17 part 2)

Thousands of images, 1 h each, and no image depends on another.

The work is almost all parallel, so a ≈ 1 and the serial part is close to 0.

That makes Tnew ≈ Told / N: double the machines and you halve the time.

There's no cap that matters here, so fill all the space you have (64 machines in the question).

```
The rule to remember: the serial part (1 − a) decides how big a cluster is worth building.

Large serial part: few machines.
Serial part near 0: as many machines as you can fit.
```


==============================

Round H, part 1: threads (sections 1–7)

1. Why threads

Apps do several things at once: update the display, fetch data, spell-check, answer network requests. Each of these can be its own thread.
Creating a process is heavy: it needs a new address space and new resources. Process pools reduce this cost only a little.
Creating a thread is much lighter. Threads also make code more modular, and kernels themselves are multi-threaded.

---


2. Process vs thread

Processes:
```
Each one gets a new address space and its own resources.
They need IPC (pipes, messages) to share data.
They are strongly isolated: if one crashes, the others keep running. That's good for security and fault tolerance.
```

Threads:
```
They share the same address space.
They are faster to create.
They share data just by using the same variables.
A fault is shared: one bad thread crashes the whole process
```

What a thread has of its own: 
	thread ID, program counter, registers, stack.
	
What the threads of a process share: 
	code, global variables, heap, open files.
	
---

3. The x / y example on the slide

The globals start as x = 10 and y = 20. 

func_a does x += 10 and y -= 10. 

func_b prints x and y.

(1)One process, no threads: func_a runs, then func_b. It prints 20, 10.

(2)fork: the child runs func_a, then both the parent and the child run func_b.
The child prints 20, 10.
The parent prints 10, 20, because the child only changed its own copy of x and y.

(3)Threads: main starts a thread that runs func_a, then main runs func_b right away. 

They share x and y, but func_b may run before or after func_a, 
so the output isn't fixed. 
That's the "???" on the slide. Call join before func_b and it always prints 20, 10.

---

4. Multi-threaded server and the benefits

A server wants the most connections handled per second. 
Threads are cheap, so it can give each request its own handler thread.

The 4 benefits:

(1)Responsiveness: 
	if one part blocks, the rest (like the UI) keeps going.

(2)Resource sharing: 
	threads share memory, which is easier than IPC.

(3)Economy: creating and switching threads is cheaper than for processes.

(4)Scalability: one process can use many cores.

---

5. Concurrency vs parallelism

"Concurrency" means several tasks are making progress. 
One core can do this by switching between them.

"Parallelism" means several tasks run at the same instant. 
That needs more than one core.

Parallel implies concurrent, but concurrent doesn't imply parallel.


The hard parts of multi-core programming: 
dividing the work, balancing the load, splitting the data, data dependencies, and testing and debugging.

---

There are two kinds of parallelism:

(1) Data parallelism: split the data; every core does the same operation on its own part. 
Example: each core sums half of an array.


(2)Task parallelism: split the work; each thread does a different operation.
Example: one thread computes the average while another finds the max.

---

6. User threads vs kernel threads

User threads are managed by a library in user space (for example GNU Pth). 
Creating or switching one is just a function call, with no system call. The kernel doesn't know they exist.

Kernel threads are known to the kernel, 
which schedules each one. Windows, Linux and macOS all have them.

Pthreads (POSIX threads) is a specification, not an implementation. 
It's usually built on kernel threads.

---

7. The 3 threading models

These describe how user threads are mapped onto kernel threads.

(1)Many-to-one (user-level threads):

Many user threads run on 1 kernel thread

If one thread makes a blocking call, all of them block, because the kernel blocks its only thread.
There's no parallelism on a multi-core machine: the kernel sees 1 thread, so only 1 core is used.
Example: GNU Pth.


(2) One-to-one (kernel-level threads):
Each user thread gets its own kernel thread.
One thread can block while the others keep running, and they can run on different cores.
The cost: every thread is a kernel object, so the number of threads per process is sometimes limited.
Linux and Windows use this model.


(3)Many-to-many (hybrid):

Many user threads are mapped onto many (usually fewer) kernel threads.
The OS creates enough kernel threads, and at any moment each user thread sits on one of them.
The concern is locality: a user thread that moves between kernel threads loses its cache.


============
Round H, part 2 (sections 8–15)

8. EXP-71: user vs kernel threads, how to think about it

The setup: 
2 threads, each needs 500 s of CPU time, and one thread can't use 2 cores at once.

(1)One core: nothing can run in parallel, whatever the threading model. The total work is 1000 s of CPU.

(2) Two cores: the question is whether the kernel can see 2 threads.
User-level (many-to-one): the kernel sees only 1 thread, so it uses only 1 core.
Kernel-level (one-to-one): the kernel sees 2 threads, so it can put one on each core.

(3) When each thread finishes: it depends on how the CPU is shared.
A user-level library usually runs one thread until it finishes or gives up the CPU, then runs the next.
A kernel time-slices (round robin), so both threads move forward together and finish near the end.

(4)Switch time that isn't zero:
add the cost of every switch. 
A user-level switch is cheap, about a function call. 
A kernel-level switch costs more, because it has to go into the kernel.

---

9. Asynchronous vs synchronous thread creation

(1)Asynchronous:
	The parent creates a child thread and carries on with its own work.
	Parent and child run independently and share little data.
	Example: a server's listener creates a handler for each connection.

(2)Synchronous:
	The parent creates children and waits (join) until all of them finish.
	The children share a lot of data with each other, but little with the parent.
	Example: fork-join (section 12).


---

10. Implicit threading

How do you use all the cores?

Option A, explicit: 
the programmer writes the threads and splits the data by hand.

Option B, implicit:
 the programmer only marks the tasks (a task is a function). 
 A library or the compiler maps those tasks onto threads automatically.
 
 -> Tools for implicit threading: thread pools, fork-join, OpenMP.

---

quetsion for 10 :
 in avdoird studio when program android app , 
 some ui thing on main theread and to handle data or internet needed loading 
 often use io thread so that is like android implicit handle those ???
 
 Ans:
 
Yes, that's implicit threading, and it's a good example.

When you write

```
viewModelScope.launch(Dispatchers.IO) {
    val data = api.fetch()          // runs on an IO thread
    withContext(Dispatchers.Main) {
        textView.text = data        // back on the UI thread
    }
}

```
you only mark the task and where it should run. You never create a thread yourself.
Dispatchers.IO is a thread pool (section 11). Its threads already exist, your task borrows a free one, 
and if none is free the task waits in a queue. 
That's why launching thousands of these doesn't create thousands of threads.

Option B from the slide: the programmer identifies the task, and the library maps it onto available threads.


launch is asynchronous creation (section 9). The caller doesn't wait, it carries on right away.

---

11. Thread pools

Creating a new thread for every request has two problems:

(1)Latency: each request waits while its thread is being created.
(2)No limit: too many requests create too many threads, and the system runs out of resources.

A thread pool fixes both:
A fixed set of threads is created ahead of time.
Each request takes a free thread from the pool.
If no thread is free, the request waits in a queue.

---

12. Fork-join

(1)Fork: split a task into subtasks and start a thread for each.
(2)Join: wait for the children, then combine their results.

Done recursively, this makes a tree. Each level splits every piece in half until the pieces are small enough. 
The leaves run in parallel, then the results are combined back up the tree, one level at a time.

eg:  1024 records, split until each piece has 64 records.

```
1024 -> 512 -> 256 -> 128 -> 64      4 levels, 1024 / 64 = 16 pieces

```

There are 32 processors, but only 16 are used, one per piece.

At 1 s per record, each piece takes 64 s. All the pieces run in parallel, so the runtime is 64 s.

Combining goes 16 → 8 → 4 → 2 → 1, which is 4 levels. 
The combines on one level happen in parallel, so each level costs 20 ms.

```
total   = 64 s + 4 * 20 ms = 64.08 s
speedup = 1024 / 64.08 ≈ 15.98   Times  (16 if you ignore combining)

```

---

13. OpenMP

OpenMP is a set of compiler directives (pragmas, which are special comments) for C, C++ and Fortran. 
The programmer marks the code to run in parallel, and the compiler creates the threads.

```
#pragma omp parallel for
for (i = 0; i < N; i++)
    c[i] = a[i] + b[i];

```
Notice: A compiler without OpenMP just ignores the pragma, and the loop still runs normally on one thread.

---

14. Pthreads

```c

int sum;                         // global, shared by all threads
void *runner(void *param);       // the function the new thread starts in

int main(int argc, char *argv[]) {
    pthread_t tid;
    pthread_attr_t attr;
    pthread_attr_init(&attr);
    
	pthread_create(&tid, &attr, runner, argv[1]);  // new thread runs runner(argv[1])
    pthread_join(tid, NULL);                       // wait for it to finish
    
	printf("sum = %d\n", sum);
}

```

After pthread_create, the new thread runs runner while main carries on to the next line.

(1)Each thread has a unique ID. pthread_create gives it back to the caller, 
and a thread can get its own with pthread_self(). 
(2)IDs let you check whether two threads are the same.

pthread_join makes the caller wait for another thread to finish.

A thread ends when:

its start function returns,

it calls pthread_exit(),

another thread cancels it with pthread_cancel(),
any thread calls exit(), or main returns.

 Either one ends the whole process, so every thread dies with it.

---

15. Thread cancellation

The target thread is the one being cancelled. 
For example, the user clicks Cancel on a download, so the thread doing that download should stop.

(1)Asynchronous cancellation: 
the target is killed immediately. 
This is risky, because it may be holding a lock or be halfway through updating shared data.

(2)Deferred cancellation: the target checks for itself, at points it chooses, 
whether it should stop. That way it stops cleanly.

How deferred cancellation works in Pthreads:

pthread_cancel(tid) only sends the request.

The target checks for it at cancellation points, for example with pthread_testcancel().
If a cancel request is waiting, pthread_testcancel() does not return: the thread ends right there. If there's no request, it returns and the thread keeps going.

==========================
Round I: L14 deadlocks (slides 1–14)

1. What a deadlock is

A deadlock is when a set of processes that compete for resources are permanently blocked.

Each process in the set is waiting for a resource that another waiting process in the set holds, so none of them can ever continue.
It always involves two or more processes whose resource needs conflict.
There is no efficient solution for the general case.

---
Your bank example from class: 
C1 locks account A1 and C2 locks A2. To finish its transfer,
 C1 now needs A2 while C2 needs A1, so both wait forever.


---

2. Kinds of resources


The first split is reusable vs consumable:

(1)Reusable: one process uses it at a time, it isn't used up, 
	and it's released for others afterwards. 
	Examples: CPU, memory, I/O devices, files.

(2)Consumable: it is created and destroyed, so once a process takes it, 
	it's gone. 
	Examples: interrupts, signals, messages.

The second split is preemptable vs non-preemptable:

(1)Preemptable: it can be taken away from a process without harm,
	as long as its state is saved and restored. Examples: CPU, memory.

(2)Non-preemptable: taking it away makes the computation fail. 
	Examples: a printer (it would leave half a page printed), a floppy disk.


Deadlocks mostly happen with reusable + non-preemptable resources.

---

3. The 4 conditions of deadlock

(1)Mutual exclusion: a resource is held by only one process at a time; it isn't shared.

(2)Hold and wait: a process can wait for a resource while still holding others.

(3)No preemption: a process won't give up a resource until it's finished with it.

(4)Circular wait: there is a closed chain where each process holds a resource that the next one is waiting for.

A process also can't be reset to an earlier state where it held nothing; the step can't be undone.

Conditions 1–3 are necessary but not sufficient. A deadlock needs all 4.

---

4. Solving deadlocks: break one condition

If any necessary condition can't hold, a deadlock can't happen. The slides give one example system for each:

```
Example system:																			Condition it breaks:

Only shared resources																	mutual exclusion
Abort any process that asks for a resource already in use				hold and wait
A process doesn't use its resources until it has all it needs, so they can be taken back meanwhile	   no preemption
The system detects or avoids deadlocks											circular wait
Checkpoints let a transaction back out										can't undo (irreversible)


```

---

5. Resource allocation graphs

A process is a circle.
A resource is a square, with one dot per unit.
A request edge goes P → R and is dashed: P is waiting for R.
An allocation edge goes R → P and is solid, starting from a dot: P holds one unit of R.

Slide 9:

```
P1 waits for R2      R2 held by P2
P2 waits for R4      R4 held by P3
P3 waits for R3      R3 (2 units) held by P1 and P2

cycle: P1 -> R2 -> P2 -> R4 -> P3 -> R3 -> P1


All 3 processes are blocked, so none can release anything. That's a deadlock.

```

---

6. Cycle vs knot

(1)A cycle is necessary for a deadlock.
(2)If every resource has only 1 unit, a cycle is also enough for a deadlock.

(3)If resources have several units, a cycle is not enough. 
You need a knot: 
	means a cycle where no node in the cycle has a path leading out of it.

Slide 10 (R1 and R2 have 2 units each):

```
P1 waits for R1      R1 held by P2 and P3
P3 waits for R2      R2 held by P1 and P4
P2, P4 are active (not waiting)

cycle: P1 -> R1 -> P3 -> R2 -> P1


```
There's no deadlock yet

R1 also has an edge out to P2, 
and R2 one out to P4. 

Both of them are still running, so one can finish and release a unit, 
and that unblocks P1 or P3. 
Your scanner/printer example has the same shape.

---

Slide 12 adds new requests to that same graph:

(a) P4 requests R1: still no deadlock. P2 is still running, so it can release R1 and P4 gets it.

(b) P2 requests R2: deadlock. 
Now every process is waiting and there's no way out of the cycle, 
so it's a knot. No running process is left to release anything.

These slides assume a process stops as soon as it waits for a resource, and waits for only one at a time.

---

7. Four strategies

(1)Ignore it: pretend the problem doesn't exist.

(2)Prevention: design the system up front so a deadlock can never happen, by ruling out one of the conditions.

(3)Avoidance: at each request, decide on the spot whether granting it could lead to a deadlock, and make the process wait if it could.

(4)Detection: let a deadlock happen, detect it, then recover.

Prevention sets fixed rules ahead of time for everyone. Avoidance checks the current state each time a request comes in.

---

8. Who picks what (slide 14)

Mathematicians: a deadlock is unacceptable and must be prevented at any cost.

Engineers: how serious is it? They don't want to pay in performance and convenience for something rare.

UNIX ignores it. Prevention is expensive: it puts inconvenient restrictions on processes.