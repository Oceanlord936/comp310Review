# COMP 310 round guide: recall L09-L12, rest of L12, L13 threads, L14 deadlocks (to slide 14)

How to use this (same as last time):

- One round at a time: read the concepts, type your own notes in `L12_L14_notes.md`, then try every question without looking back.
- Answers go in `L12_L14_answers.md` or in chat. Then say "check round G" (or R, H, I).
- Guessing or leaving a blank is fine. I check, explain, and put the right answer in your answers file.
- Answers are NOT in this guide. Only the worked examples are solved.

| Round | Material                                                         | Source                      | Time      |
| ----- | ---------------------------------------------------------------- | --------------------------- | --------- |
| R     | Recall: 15 picked questions from rounds A-F (L09 to L12 Amdahl)  | L09_L11_notes.md            | 40 min    |
| G     | Rest of L12: SMP, NUMA, clusters, EXP-17                         | N12 p12-15, class17         | 20 min    |
| H     | L13 multi-threading, all of it                                   | N13, class17-18             | 60-70 min |
| I     | L14 deadlocks, slides 1-14                                       | N14 p1-14, class18          | 40 min    |

**Today (2026-11-07): concepts only, no questions.** Read rounds G, H, I and type your notes, so you can keep up in class next week. All the questions (R, G, H, I) are pushed to later, before the exam. Then A-Lab-03.

Scope: N14 stops at slide 14 (Dealing with Deadlocks). Prevention details (slide 15 on) are left out, the prof hasn't taught them yet.

===================================

## The mistakes I fixed in your class17 / class18 notes

The originals are in `L12_L14_review/notes_original/`. Each fix is marked "(fix: ...)" in place.

- pthread code: `pthread_attr_init($attr)` is `&attr`, and `Pthreads_create` is `pthread_create`.
- Solving deadlocks (2), "abort processes which request a resource in use": you left off what it negates. It negates hold and wait.
- Fork-join example: your reasoning was right, but there were no final answers. 4 levels, 16 processors, 64 s, 64.08 s with combining, speedup about 16.
- Asynchronous vs synchronous creation (class17): only asynchronous was written down. Synchronous added.
- Resource allocation graph: a request edge (P waits for R) is DASHED, an allocation edge (R given to P) is SOLID. You had only the solid one marked.

===================================

# Round R: recall L09 to L12

No new concepts. Answer from memory first, then check against `L09_L10_review/L09_L11_notes.md` only if stuck. These are the questions from the old guide that hit the ideas most likely to be tested.

R1 (A1). counter = 10. T1 and T2 both run counter++. Write an interleaving of the load / add / store lines that ends at 11.

R2 (A6). A blocking semaphore starts at 2. Five threads call wait(). Then another thread calls signal(). Give the count after all the waits, the count after the signal, and how many are still blocked.

R3 (A8). Bounded buffer, N = 4. The producer inserts 3 items, the consumer removes 2. Give mutex, empty, full.

R4 (A9). Swap the CONSUMER's wait(&full) and wait(&mutex). Describe a run that deadlocks, starting from the empty buffer.

R5 (B1). In this exact time order: T1 signals, then T2 waits.
(a) With a semaphore s(0): does T2 block? What is the count at the end?
(b) With a condition variable c: does T2 block? Why?

R6 (B2). Monitor bounded buffer with `if`, Mesa, N = 2.
P1 append, P2 append, P3 append (sleeps), C1 take (signals), C1 leaves, P4 enters BEFORE P3 runs and appends, then P3 runs.
(a) What goes wrong with `if`? What is count at the end?
(b) What does P3 do with `while`, and what is count at the end?

R7 (B5). Bank monitor, Mesa. balance = 0. W1 calls withdraw(100), then W2 calls withdraw(10). Both sleep. Then deposit(50) runs.
(a) If deposit uses csignal and the queue is FIFO, what happens to W1 and W2? Is that a problem?
(b) What should deposit use instead, and what happens then?

R8 (C1). Footman solution: each philosopher takes fork[i] then fork[(i+1)%5]. They arrive P0, P1, P2, P3, P4, each getting as far as it can before the next arrives. Give maxP's count, who holds which fork, who is blocked on what, and who eats.

R9 (C7). Why does endRead use csignal(okToWrite), but endWrite uses cbroadcast(okToRead)?

R10 (D4). A file is 1000 bytes and the pointer is at 800. What does read(fd, buf, 500) return? And the read after that?

R11 (D9). Why do hard links to directories create a problem that hard links to files don't? What does a soft link risk instead?

R12 (E3). B = 4 KB, d = 4 bytes, 12 direct + single + double + triple indirect. What is the largest file? Give each region's size, then the total.

R13 (E8 a, b). Path /home/alice/docs/report.txt. No caching, each directory is one block.
(a) How many disk reads to get the first byte, with the root i-node in memory?
(b) And if the root i-node must also be read?

R14 (E10 a). Only /user/abc/xyz.c exists, i-node 31, count 1. Run `ln /user/abc/xyz.c /home/xy.c`, `rm /user/abc/xyz.c`, `cat /home/xy.c`, `rm /home/xy.c`. Give the link count after each, and say whether the cat works.

R15 (F4 + F5). An app takes 30 min on one core and 18 min on 4 cores. What fraction a can be enhanced? With that a, what is the best speedup with infinitely many cores?

===================================

# Round G: rest of L12 (multiprocessor systems, clusters)

1. Why multiple processors

Physics makes it hard to build a faster single CPU (heat, clock speed). Adding processors is the easiest way to scale performance.

---

2. SMP (Symmetric Multi Processing)

- Every processor is equal and can do every task (any CPU can run the kernel, any process).
- All processors share ONE physical memory over a system bus.
- Problem: the bus is shared. As you add CPUs they all fight for the same bus, so SMP does not scale.

---

3. NUMA (Non-Uniform Memory Access)

- Fix for SMP: distribute the memory, give each processor (or group) its own local memory.
- But keep ONE shared address space. Any CPU can still address all of memory.
- Part of the address space is local (fast) and the rest is remote/global (slower). That is the "non-uniform".
- The OS manages locality: it tries to keep a process's memory on the node where it runs, so most accesses are local and fast.

Example on the slide: Fugaku (Japan), 158,976 A64FX ARM CPUs, once the world's fastest.

---

4. Message passing machines

No shared memory and no shared address space. Processors only talk by sending messages.

---

5. Clusters

- Cheap: repurpose existing commodity machines (connect all the lab machines).
- Loosely coupled: sending a message between machines takes much longer than in SMP or NUMA.
- So clusters give high performance only for applications that send FEW but LARGE messages, where each task is big compared to the cost of talking.

Order by message cost, fastest first: SMP (same bus), NUMA (local fast, remote slower), cluster (a network between machines).

---

6. EXP-17: how to think about cluster size

The question is always: does adding a machine still pay for itself? Use Amdahl.

- If most of the job is serial, extra machines barely help (Amdahl's cap 1 / (1 - a)). Buy few.
- If the job splits into many independent pieces (each image, each record), the speedup keeps growing with machines. Fill the space you have.
===================================

## Round G questions

G1. Why doesn't SMP scale? What does NUMA change, and what does it keep the same as SMP?

G2. In a NUMA machine, why does the OS care which node a process's memory pages are on?

G3. Rank SMP, NUMA, and a cluster by how long it takes one processor to send data to another. Which kind of application is a good fit for a cluster, and which is a bad fit?

G4. EXP-17 part 1. Neural network training takes 10 h on one machine. 30% is parallel, 70% serial.
Price per machine: 1 machine $1000; 2 or more $900 each; 4 or more $800 each; 16 or more $700 each.
(a) Give the runtime and the total cost for 1, 2, 4 and 16 machines.
(b) What is the best possible runtime, with infinitely many machines?
(c) Which cluster size would you pick, and why?

G5. EXP-17 part 2. Image processing: many thousands of images, each takes 1 h, images are independent. The rack has space for 64 machines. Which cluster size do you use, and why is this answer so different from G4?

===================================

# Round H: L13 multi-threading

1. Why threads

- Most modern applications are multi-threaded. Separate tasks inside one app get their own thread: update the display, fetch data, spell check, answer a network request.
- Process creation is heavy-weight: new address space, new resources, slow to create, terminate and manage. Process pools reduce the cost a bit.
- Thread creation is much lighter. Threads also make code more modular. Kernels themselves are multi-threaded.

---

2. Process vs thread

Processes:

- A new address space at creation, resources allocated at creation.
- Need IPC (pipes, shared memory, messages) to share data.
- Deeper isolation: one crashing does not take the others down. Good for security and fault tolerance.

Threads:

- Same address space as the other threads of the process.
- Quicker to create (how much quicker depends on user vs kernel threads).
- Share data just by using the same variables.
- Faults are shared: one bad thread can crash the whole process.

What each thread has of its OWN: thread ID, program counter, registers, stack.
What threads SHARE: code, global data, heap, open files.
(This is why the Linux slide says "threads have specific stacks".)

---

3. The slide's x / y example

Globals x = 10, y = 20. func_a does x = x + 10 and y = y - 10. func_b prints x and y.

One process: main calls func_a then func_b. Prints x = 20, y = 10.

Two processes:

```c
main() {
    if (fork() == 0)
        func_a();      // only the child changes ITS copy
    func_b();          // both parent and child print
    wait(..);
}
```

The child prints 20, 10. The parent prints 10, 20, because the child changed its own copy of the address space.

Two threads:

```c
main() {
    thread_create_and_start(func_a);
    func_b();
    thread_join();
}
```

Same x and y for both. But func_b may run before, during or after func_a, so the output is not fixed: that's the "???" on the slide. To always print 20, 10, join BEFORE calling func_b.

---

4. Multi-threaded server

A server wants throughput (connections per second). Threads are light, so creating a handler per request, sharing data between handlers, and cleaning up old handlers costs little.

Benefits of multi-threading (4 words to remember):

- Responsiveness: if one part blocks (a long download), the rest (the UI) keeps going.
- Resource sharing: threads share the process's memory, easier than shared memory or message passing between processes.
- Economy: cheaper to create, and switching threads costs less than switching processes.
- Scalability: one process can use many cores.

---

5. Concurrency vs parallelism

- Concurrency: more than one task is making progress. A single core does this by switching between tasks (the scheduler).
- Parallelism: more than one task runs AT THE SAME TIME. Needs more than one core.

Parallel implies concurrent. Concurrent does not imply parallel.

Challenges of multi-core programming: dividing the app into tasks, balancing the work, splitting the data, data dependencies between tasks, testing and debugging.

Two types of parallelism:

- Data parallelism: split the DATA across cores, every core does the SAME operation on its part. (Sum the first half of an array on core 0, the second half on core 1.)
- Task parallelism: split the WORK across cores, each thread does a DIFFERENT operation. (One thread computes the mean, another the max.)

---

6. User threads vs kernel threads

- User threads: managed entirely by a user-level library (example GNU Pth). Creating or switching a thread is a function call, no system call. The kernel does not know they exist.
- Kernel threads: the kernel knows and schedules each one. All general purpose OSes have them (Windows, Linux, macOS).
- POSIX threads (Pthreads) is a standard, a SPECIFICATION of behaviour, not an implementation. Usually implemented with kernel threads.

---

7. Threading models (how user threads map to kernel threads)

Many-to-one (user-level threads):

- Many user threads on ONE kernel thread.
- One thread makes a blocking system call, the kernel blocks the one kernel thread, so ALL the user threads block.
- No parallelism on multi-core: the kernel sees one thread, so it uses one core.
- Example: GNU Pth.

One-to-one (kernel-level threads):

- Each user thread gets its own kernel thread. Creating a user thread creates a kernel thread.
- More concurrency: one blocks, the others keep running, and they can run on different cores.
- Cost: each thread is a kernel object, so some systems limit the number of threads per process.
- Linux and Windows work this way.

Many-to-many (hybrid):

- Many user threads on many (usually fewer) kernel threads.
- The OS creates enough kernel threads. At any moment, a user thread sits on one kernel thread.
- Concern: locality of the mapping (a user thread moving between kernel threads and cores loses its cache).

---

8. EXP-71 first problem: how to think about it

2 threads, each needs 500 s of CPU, a thread can't use 2 cores at once.

- On ONE core, nothing runs in parallel, whatever the model. Total CPU work is 1000 s.
- On TWO cores, the question is whether the kernel can SEE two threads. User-level: the kernel sees one kernel thread, so it uses one core. Kernel-level: two kernel threads, two cores.
- Thread completion time depends on how the CPU is shared. A user library usually runs a thread until it yields or ends (run one, then the other). A kernel time-slices (round robin), so both progress together and finish near the end.
- Non-zero switch time: add the cost of every switch. A user-level switch is cheap (a function call), a kernel-level switch is more expensive (trap into the kernel).

---

9. Asynchronous vs synchronous thread creation

Asynchronous:

- The parent spawns a child and then resumes its own work.
- Parent and child run concurrently and independently.
- They usually share little data. (A server's listener spawning a handler.)

Synchronous:

- The parent spawns children and WAITS for all of them to finish (join) before it goes on.
- The children may share a lot of data among themselves, but little with the parent. (Fork-join is this.)

---

10. Implicit threading

With many cores, how do you parallelize code?

- Option A (explicit): the programmer organizes the code into threads and splits the data by hand.
- Option B (implicit): the programmer only names TASKS (a task is a function). A library or the compiler maps the tasks onto threads automatically.

Implicit threading tools: thread pools, fork-join, OpenMP.

---

11. Thread pools

Two problems with creating a thread on demand:

- Latency: each request waits while its thread is created.
- No limit: too many requests create too many threads and exhaust resources.

A thread pool fixes both:

- Create a fixed set of threads ahead of time.
- A new request takes a free thread from the pool.
- If there is NO free thread, the request waits in a queue.

---

12. Fork-join parallelism

- Fork: split a task into sub-tasks, spawn a thread for each.
- Join: wait for the children and combine their results.
- Applied recursively, it forms a tree: each level splits every problem in two, until the pieces are small enough. The leaves run in parallel. Then results combine back up the tree, level by level.

Worked example (EXP-71 second problem, from class18): 1024 records, split until each piece has 64 records.

```
1024 -> 512 -> 256 -> 128 -> 64       4 levels of splitting
1024 / 64 = 16 pieces                 16 leaves = 16 threads
```

- 32 processors: only 16 are used, one per piece.
- 1 s per record, ignoring split and combine: each piece takes 64 s, all in parallel, so runtime = 64 s.
- Combining: 16 -> 8 -> 4 -> 2 -> 1 is 4 levels. The combines inside one level run in parallel, so each level costs 20 ms once.

```
total = 64 s + 4 * 20 ms = 64.08 s
serial = 1024 * 1 s = 1024 s
speedup = 1024 / 64.08 = about 15.98      (16 without combining)
```

---

13. OpenMP

A set of compiler directives (pragmas, special comments) for C, C++, Fortran. The programmer marks the code to parallelize, the compiler makes the threads.

```c
#pragma omp parallel for
for (i = 0; i < N; i++)
    c[i] = a[i] + b[i];
```

A compiler without OpenMP ignores the pragma and the code still runs, just serially. That's what you wrote in class.

---

14. Pthreads

```c
int sum;                          // shared by all threads (global)

void *runner(void *param);        // the thread's start function

int main(int argc, char *argv[]) {
    pthread_t tid;                // thread ID
    pthread_attr_t attr;          // thread attributes

    pthread_attr_init(&attr);                     // default attributes
    pthread_create(&tid, &attr, runner, argv[1]); // new thread runs runner(argv[1])
    pthread_join(tid, NULL);                      // wait for it to finish
    printf("sum = %d\n", sum);
}
```

- After pthread_create, the new thread runs the start function and main continues with the next line.
- Each thread has a unique ID, returned to the creator, or from pthread_self(). IDs let you check if two threads are the same.
- pthread_join waits for another thread.

A thread ends when:

- its start function returns,
- it calls pthread_exit(),
- another thread cancels it with pthread_cancel(),
- ANY thread calls exit(), or main returns. That ends the whole process, all threads with it.

---

15. Thread cancellation

The target thread is the one being cancelled (the user hits Cancel on a download, kill the thread doing it).

- Asynchronous cancellation: the target is terminated immediately. Risky: it may be holding a lock or be halfway through updating shared data.
- Deferred cancellation: the target checks, at points of its own choosing, whether it should stop, so it stops in an orderly way.

Deferred in Pthreads:

- pthread_cancel(tid) only SENDS the request.
- The target checks at cancellation points, for example a call to pthread_testcancel().
- If a request is pending, pthread_testcancel() does not return: the thread terminates there. Otherwise it returns and the thread keeps going.

===================================

## Round H questions

H1. Name what each thread has of its own, and what the threads of one process share. Why is a bug in one thread worse than a bug in one of several processes?

H2. The x / y example (globals x = 10, y = 20; func_a: x += 10, y -= 10; func_b prints).
(a) With fork: what do the parent and the child print?
(b) With threads, as on the slide (create, func_b, join): list two different outputs that can happen.
(c) Move one line so the thread version always prints x = 20, y = 10.

H3. True or false, one line why:
(a) A single-core machine can run threads concurrently.
(b) A single-core machine can run threads in parallel.
(c) Summing the two halves of an array on two cores is task parallelism.

H4. EXP-71. An app has 2 threads, each needs 500 s of CPU. One thread can't use two cores at once.
(a) Single core, user-level threads: application completion time?
(b) Single core, kernel-level threads: application completion time?
(c) Two cores: application completion time with user-level threads, and with kernel-level threads?
(d) Give each thread's completion time for (a), (b) and (c).
(e) Now each thread switch costs s seconds. Which model pays more per switch, and why?

H5. Many-to-one: one thread calls a blocking read() on the keyboard. What happens to the other threads, and why? What happens in one-to-one?

H6. Asynchronous or synchronous creation, and why:
(a) A web server's main thread spawns a handler for each connection.
(b) Fork-join sorting of an array.

H7. Thread pool with 4 threads. 6 requests arrive at the same moment, each takes 10 s. When does each finish? What two problems of on-demand threads does the pool avoid?

H8. Fork-join, new numbers. 2048 records, split until each piece has 128 records. 8 processors. 1 s per record. Each combine takes 10 ms.
(a) How many levels of splitting, and how many pieces?
(b) How many processors are used, and what is the runtime ignoring combining? (careful)
(c) Total runtime with combining, and the speedup.

H9. Why is asynchronous cancellation dangerous for a thread that holds a mutex? What does deferred cancellation do differently, and what does pthread_testcancel() do when a cancel is pending, and when none is?

H10. main() calls pthread_create() and then returns right away, without pthread_join(). What happens to the new thread?

===================================

# Round I: L14 deadlocks, slides 1-14

1. What a deadlock is

- Permanent blocking of a set of processes that compete for resources.
- Each process in the set is waiting for a resource held by another WAITING process of the set, so nobody can move.
- Always involves conflicting needs of two or more processes.
- There is no efficient solution in the general case.

Your class example: customer C1 locks account A1, customer C2 locks account A2. Now C1 needs A2 to finish the transfer and C2 needs A1. Each waits for the other forever.

---

2. Classes of resources

Reusable vs consumable:

- Reusable: used safely by one process at a time and NOT used up. Released for others afterwards. CPU, memory, I/O devices, files.
- Consumable: created and destroyed. Once acquired it no longer exists. Interrupts, signals, messages.

Preemptable vs non-preemptable:

- Preemptable: can be taken away with no ill effect, if you save and restore its state. CPU, memory.
- Non-preemptable: taking it away makes the computation fail. Printer (half a page printed), floppy disk.

Deadlocks mostly occur with REUSABLE and NON-PREEMPTABLE resources.

---

3. The 4 conditions

1. Mutual exclusion: a resource is held exclusively, not shared.
2. Hold and wait: a process can wait for a resource while holding others.
3. No preemption: a process won't give a resource up until it's finished with it.
4. Circular wait: a closed chain where each process holds a resource the next one is requesting.

Plus: a process can't be reset to an earlier state where it held no resources (irreversible).

Conditions 1-3 are necessary but NOT sufficient. All 4 are needed for a deadlock.

---

4. Solving deadlocks: break one condition

Break any necessary condition and deadlock can't happen.

| Example system                                                         | Condition it negates |
| ---------------------------------------------------------------------- | -------------------- |
| Only shared resources                                                  | mutual exclusion     |
| Abort a process that requests a resource in use                        | hold and wait        |
| Preempt resources from a process that hasn't started using them yet   | no preemption        |
| Detect or avoid deadlocks                                              | circular wait        |
| Checkpoints, so a transaction can back out                             | irreversibility      |

---

5. Resource allocation graphs

- Process = circle. Resource = square, with one dot per unit (a resource can have several units).
- Request edge P -> R, DASHED: P waits for (has requested) R.
- Allocation edge R -> P, SOLID, from a dot: one unit of R is held by P.

Slide 9 example, every resource has the units you see:

```
P1 waits for R2      R2 held by P2
P2 waits for R4      R4 held by P3
P3 waits for R3      R3 (2 units) held by P1 and P2

cycle: P1 -> R2 -> P2 -> R4 -> P3 -> R3 -> P1
```

All three processes are blocked and none can release anything: deadlock.

---

6. Cycle vs knot

- A cycle is NECESSARY for deadlock.
- With single-unit resources, a cycle is also enough: deadlock.
- With multi-unit resources a cycle is NOT enough. You need a KNOT: a cycle with no path leading OUT of it from any node in it.

Slide 10 example: R1 and R2 have 2 units each.

```
P1 waits for R1      R1 held by P2 and P3
P3 waits for R2      R2 held by P1 and P4
P2 and P4 are active (not waiting)

cycle: P1 -> R1 -> P3 -> R2 -> P1
```

No deadlock yet: R1 has an edge out to P2, and R2 to P4. Those are active, so one of them can finish and release a unit, which unblocks P1 or P3. Your scanner / printer example is the same graph.

Slide 12, further requests on the same graph:

- (a) P4 requests R1: still no deadlock. P2 is active, it can release R1, then P4 gets it and finishes.
- (b) P2 requests R2: deadlock. Every process is now waiting, the cycle has no way out (a knot). No active process is left to release anything.

The slide's assumption for this: a process stops as soon as it waits for one resource, and waits for only one at a time.

---

7. Four strategies

1. Ignorance: pretend there is no problem.
2. Prevention: design the system so deadlock can never happen (a priori, by excluding a condition).
3. Avoidance: decide at each request, dynamically, whether granting it could lead to deadlock. Refuse (delay) the ones that could.
4. Detection: let deadlock happen, detect it, then recover.

Prevention vs avoidance: prevention fixes the rules up front for everyone. Avoidance looks at the current state for each request.

---

8. Who picks what (slide 14)

- Mathematicians: deadlock is unacceptable, prevent it at all costs.
- Engineers: how serious is it? Don't pay in performance and convenience for something rare.
- UNIX: ignores the problem. Prevention has a high price: inconvenient restrictions on processes.

===================================

## Round I questions

I1. Classify each as reusable or consumable, and preemptable or non-preemptable where it applies: CPU, memory, printer, a message, a file being written, an interrupt.

I2. Why are conditions 1 to 3 not enough for a deadlock? Give a situation where mutual exclusion, hold and wait and no preemption all hold, and no deadlock happens.

I3. The bank transfer. C1 runs lock(A1) then lock(A2). C2 runs lock(A2) then lock(A1).
(a) Write the interleaving that deadlocks.
(b) Draw the resource allocation graph at the deadlock (as text, like in section 5).
(c) For each of the 4 conditions, say how it shows up here.
(d) Which condition is easiest to break here, and how? One idea is enough.

I4. Single-unit resources R1, R2, R3. P1 holds R1 and waits for R2. P2 holds R2 and waits for R3. P3 holds R3 and is running.
(a) Is there a cycle? A deadlock?
(b) Now P3 requests R1. Cycle? Deadlock? Why is a cycle enough here?

I5. R1 has 1 unit, R2 has 2 units. P1 holds R1 and waits for R2. P2 holds one unit of R2 and waits for R1. P3 holds the other unit of R2 and is running.
(a) Is there a cycle? Is there a knot? Deadlock?
(b) What happens next if P3 finishes?
(c) Instead, P3 requests R1. Knot? Deadlock?

I6. For each system, which condition does it negate?
(a) A printer spooler: processes write to a spool file, only the spooler touches the printer.
(b) A process must ask for every resource it will ever need in one request at start-up.
(c) A database that rolls back a transaction to its last checkpoint.

I7. Prevention or avoidance? (a) Every process must lock accounts in increasing account number. (b) The OS checks, at each request, whether granting it leaves a safe state, and makes the process wait if not.

I8. Why does UNIX choose ignorance? Give one cost of prevention that an engineer would not want to pay.
