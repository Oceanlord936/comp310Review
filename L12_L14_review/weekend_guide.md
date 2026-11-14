# COMP 310 weekend review guide: Tutorial 5, rest of L14, L15

How to use it (same as before):

- Answer in `weekend_answers.md`, or in chat. Then say "check T1" (or "check part 2 L14", and so on).
- Each Part 1 question is a Tutorial 5 question type with NEW numbers. The worked version with the old numbers is in `Notes/tutorial5.md`. Try it first, and only look there if you're stuck.
- Answers are NOT in this guide.

| Part | What | When |
| --- | --- | --- |
| 1 | Tutorial 5 question types, new numbers (T1-T10) | today |
| 2a | Rest of L14: prevention, trajectories, claim edges, Banker's, detection, recovery | after Part 1 |
| 2b | L15: file descriptors, redirection, pipes, message passing, sockets, RPC | after 2a |
| later | Old rounds R, G, H, I in `L12_L14_guide.md` (still not answered) | before the final |

Real-time guess (yours, already doubled): Part 1 about 2.5-3 h, Part 2a about 1.5 h, Part 2b about 1.5 h.

Final exam hint from class (Nov 10): questions look like the practice questions, lectures and tutorials. A few coding questions. Slide 42 (the summary of the strategies) is one the prof said to study carefully.

===================================

# Part 1: Tutorial 5 question types

## The method card (copy this into your notes first)

Banker's safety check:

```
1. need = max - hold                 (row by row)
2. avail = total - sum of hold       (column by column)
3. repeat: find a process with need <= avail (EVERY column)
           let it finish: avail = avail + its HOLD
4. everyone finished -> SAFE (write the order) ; stuck -> UNSAFE
```

Handling a request "Pi asks for r":

```
1. r <= need[i] ?   no -> ILLEGAL (asked for more than its max)
2. r <= avail ?     no -> WAIT (not enough free right now)
3. pretend: avail -= r , hold[i] += r , need[i] -= r
4. safety check on the pretend state: safe -> GRANT , unsafe -> UNDO, Pi WAITS
```

Detection: the same loop, but use REQUEST (what they ask for right now), not need. Whoever can't finish is deadlocked.

Smallest number of units that makes deadlock impossible:

```
sum of (max_i - 1) + 1
```

Amdahl:

```
Tnew = Told * [(1 - a) + a / k]     S = Told / Tnew     S_max = 1 / (1 - a)
```

===================================

## T1. Banker's, one resource type

10 identical units. Four processes:

```
        Max   Hold
P1       7     2
P2       4     1
P3       5     2
P4       3     2
```

(a) Give need and avail. Is the state safe? If it is, give a safe sequence.

(b) P1 asks for 2 units. Grant it or not? Show the pretend state and the check.

(c) Start again from (a). P1 asks for 3 units. Grant it or not?

(d) Start again from (a). P2 asks for 4 units. What happens, and at which step?

(e) In (c), is the system deadlocked right after the pretend grant? Use the words safe / unsafe / deadlock correctly.

---

## T2. Smallest number of units

(a) 4 processes share identical units. Each needs at most 4. What is the smallest number of units that makes deadlock impossible? Explain using the worst case.

(b) 3 processes need at most 3, 5 and 6 units. Same question.

(c) 6 units, 4 processes, each needs at most 2. Can a deadlock happen? Why?

---

## T3. Banker's, three resource types

Total = (A B C) = (7 5 6).

```
         Max          Hold
       A  B  C      A  B  C
P1     4  2  3      1  1  1
P2     2  3  2      2  1  0
P3     5  1  4      1  0  2
P4     3  3  1      1  2  1
```

(a) Give the need matrix, the allocation (sum of hold) and avail. Is it safe? Give the sequence.

(b) P4 asks for (1 0 0). Grant it or not?

(c) Start again from (a). P1 asks for (1 1 0). Grant it or not? Which process got closest to fitting, and in which column did it fail?

(d) Start again from (a). P2 asks for (0 0 3). What happens?

---

## T4. Detection and recovery

Two resource types. A has 4 units, B has 3. No unit is free.

```
          Hold      Request     CPU used
         A   B      A   B
P1       1   0      0   0       20 min
P2       1   1      0   1       30 min
P3       0   1      1   1        8 min
P4       1   1      0   1       12 min
P5       1   0      1   0        6 min
```

(a) Check that "no unit is free" is true. Which processes are deadlocked? Show the check.

(b) Which ONE process would you terminate? Try killing each deadlocked process and check that it really breaks the deadlock.

(c) Would killing P1 or P5 help? Why or why not? (class19 has the rule.)

(d) P3's last checkpoint was taken after it got its B. Would rolling P3 back to it help? Why?

---

## T5. Threads and a shared variable

```c
int count = 0;

void *add(void *arg) {
    for (int i = 0; i < 2; i++) {
        count = count + 5;
    }
    return NULL;
}

int main(void) {
    pthread_t tid[4];
    for (int i = 0; i < 4; i++) {
        pthread_create(&tid[i], NULL, add, NULL);
    }
    printf("%d\n", count);
    return 0;
}
```

(a) What values can it print? Name the two separate problems.

(b) Add joins (write the loop). What values can it print now? Can it still print 0?

(c) Fix it with a mutex. Write the new add(). What does it print?

(d) Fix it with NO lock, by returning values through pthread_join. Write add() and the join loop.

(e) In (d), why is there no race any more? Give the three reasons.

---

## T6. Counting threads after fork

Assume f() does not fork or create threads.

(a)

```c
fork();
pthread_create(&t1, NULL, f, NULL);
```

How many processes, and how many threads in total?

(b)

```c
pthread_create(&t1, NULL, f, NULL);
fork();
fork();
pthread_create(&t2, NULL, f, NULL);
```

How many processes, and how many threads in total? Draw the table (line, then each process's threads).

(c) Why does fork copy only the calling thread?

---

## T7. User-level vs kernel-level threads

(a) Give two advantages of user-level (many-to-one) threads.

(b) Give two advantages of kernel-level (one-to-one) threads.

(c) A many-to-one app has 3 threads on a 4-core machine. How many cores can it use at once? One thread calls a blocking read() on the keyboard. What happens to the other two?

(d) How can a thread library get around (c)'s blocking problem without kernel threads?

---

## T8. Amdahl with a disk

A job takes 60 s. 40 s of it is waiting for the disk.

(a) A new disk makes the job take 30 s. What is the speedup?

(b) How many times faster is the new disk? Find it directly with times, and with the formula.

(c) What is the best possible speedup with a disk that takes almost no time?

---

## T9. Amdahl with cores

A pipeline takes 200 min on 1 core: 10 min to load, 180 min of work that can run in parallel, 10 min to write the report.

(a) Give a.

(b) Give the time and the speedup on 1, 6 and 60 cores.

(c) What is the best possible speedup? How many cores do you need to get within 1 min of the best time?

---

## T10. Vertical vs horizontal scaling

One server costs $3,000. Bigger servers: $7,000 for 2x the power, $18,000 for 4x, $50,000 for 8x.
For horizontal scaling, a load balancer costs $1,000, each server is $3,000, and each server delivers only 80% of its power.

(a) Fill in the table: cost to get 2x, 4x and 8x the power each way. Show how many servers you need.

```
power   vertical   horizontal (servers, cost)   cheaper
2x
4x
8x
```

(b) Explain in two lines why vertical wins small and horizontal wins large.

(c) What does horizontal scaling need from the problem that vertical doesn't?

===================================

# Part 2a: rest of L14 (deadlocks)

Your class19 notes already cover this well, with fixes. This is the short version to type, then the questions.

1. Prevention: break one of the 4 conditions with a static rule

- Mutual exclusion: usually can't be broken (a printer can't be shared).
- Hold and wait: ask for ALL resources at once, block until all can be granted together.
- No preemption: if a process is refused a request, it releases what it holds and asks again for everything.
- Circular wait: put resource types in a fixed order. A process holding type R may only ask for types after R.

Prevention is conservative: it gives out fewer resources than it could.

---

2. Resource trajectories (2 processes, 1 printer, 1 scanner)

- A 2D picture: x = how far P1 has gone, y = how far P2 has gone. The state is a point that only moves right or up.
- Impossible region (X): both would hold the same resource at once.
- Same order of grabbing in both processes: you can always go around the X block, so no deadlock is possible.
- Different orders: a reachable cell U appears where each holds one and wants the other. It isn't deadlocked YET, but deadlock is certain. That's an UNSAFE state.
- Avoidance = refuse the move from safe into unsafe.

---

3. Claim edges (single-unit resources)

- Claim edge P ....> R (dotted): P may ask for R later. Declared before it runs.
- Request edge (dashed) and allocation edge (solid) as before.
- Grant a request only if turning it into an allocation does NOT make a cycle, counting claim edges.
- Claims let you see the danger before the second request arrives.

---

4. Banker's algorithm = the same idea with multiple units. It's the method card in Part 1.

- safe = at least one order where everyone finishes
- very safe = everyone can finish in any order (every need <= avail already)
- unsafe = deadlock may come, not that it is here

---

5. Detection

- Grant whenever possible, let deadlock happen, find it (cycle / knot, or the detection loop), then recover.
- When to check: on every request (early, but costs CPU), periodically (cheaper, later), or a watchdog timer (fires when the system stops responding).

---

6. Recovery

- Preemption: take the resource away. Not always possible.
- Rollback: go back to a checkpoint from before the resource was taken.
- Termination: kill a process IN the cycle, as few as possible, the cheapest one.

---

7. Slide 42 summary (study this for the final)

- Detection: very liberal. Never delays a start, good when deadlock is rare. Costs lost work when you kill or preempt.
- Prevention: conservative static rules. Each rule has its own cost (slow start, extra preemption, can't ask a bit at a time).
- Avoidance: in between. No preemption needed, but needs Max ahead of time and a check on each request. Processes can be blocked a long time.

## Part 2a questions

L1. For each of the 4 conditions, give the prevention rule that breaks it, and one cost of that rule.

L2. Two threads lock two mutexes. Thread 1 locks m1 then m2. Thread 2 locks m2 then m1.
(a) Which condition does the fix "always lock m1 before m2" break?
(b) Is that prevention or avoidance? Why?

L3. Trajectories. P1 does GS GP RS RP. P2 does GP GS RP RS.
(a) Is there an unsafe region? Explain in words which resource each one holds there.
(b) What would avoidance do when P2 asks for the printer while P1 holds the scanner?

L4. Claim edges. P1 and P2 both claim the printer R1 and the scanner R2. P2 holds R2. P1 asks for R1.
(a) Draw the graph (as text) if R1 is granted.
(b) Is there a cycle once claims are counted? Grant or not?

L5. True or false, one line why:
(a) An unsafe state is a deadlocked state.
(b) A very safe state is also safe.
(c) Banker's algorithm needs each process's max ahead of time.
(d) Detection never loses work.
(e) Checking for deadlock on every request finds it earliest.

L6. Give one situation where detection is the better choice than avoidance, and one where avoidance is better.

L7. Why must you kill a process that is IN the cycle? What happens if you kill one outside it?

===================================

# Part 2b: L15 inter-process communication

Your class20 notes follow the slides. Short version:

1. File descriptor table (FDT)

- Each process has an FDT in the kernel. Entries 0, 1, 2 are stdin, stdout, stderr, already open.
- open() returns the LOWEST free entry. printf writes to fd 1, scanf reads fd 0.
- A child created by fork gets a copy of the FDT.

---

2. Redirection

- Output: close(1), then open("abc.txt", ...) gets fd 1, so printf now goes to the file.
- Input: close(0), then open the input file, which gets fd 0.
- Only for this process and its children made after.

---

3. Pipes

- pipe(p) makes a one-way channel in the kernel. p[0] is the read end, p[1] is the write end.
- pipe, then fork: both processes have both ends.
- `ls | wc -l`: ls's stdout goes to the write end, wc's stdin comes from the read end.
- read() on a pipe returns 0 (end of file) only when EVERY write end is closed. Close the ends you don't use.

---

4. Two models of IPC

- Shared memory: both processes map the same memory. Fast, no kernel call per access, but you must synchronize.
- Message passing: send() / receive() through the kernel. Simpler, good for a small number of messages.

---

5. Message passing details

- Direct: name the other process. Indirect: send to a mailbox. Senders and receivers can change without updating the program, but it can be slower.
- A mailbox owned by a process disappears when the owner deletes it. One made by the OS lives on its own.
- Blocking (synchronous) send waits until received. Blocking receive waits for a message.
- Non-blocking (asynchronous) send just continues. Non-blocking receive gets a message or null.
- Buffer: zero capacity (sender waits for receiver: rendezvous), bounded (wait when full), unbounded (never wait).

---

6. Sockets

- An endpoint for communication, named by IP:port. IP picks the machine, port picks the process on it.
- Ports below 1024 are well-known services (SSH 22, HTTPS 443).
- A connection is unique by its 4-tuple (host1:port1, host2:port2).

---

7. RPC (remote procedure call)

- Call a function on another machine as if it were local. Message passing hides underneath.
- Client stub: packs the procedure name and parameters into a message (marshalling), sends it, waits for the reply.
- Server stub: looks up the procedure by name (registry), calls it, sends back the result or error.
- Different machine types need a common data encoding (XDR).

## Part 2b questions

P1. A process has only fds 0, 1, 2 open. It runs close(1), then fd = open("out.txt", ...), then printf("hi").
(a) What is fd?
(b) Where does "hi" go?
(c) It then forks. Where does the child's printf go?

P2. Write the order of syscalls (no full code) for a shell running `ls | wc -l`: pipe, fork twice, which closes, which redirect in each child.

P3. In the class pipe example, the child loops `while ((n = read(p[0], ...)) > 0)`.
(a) Why does the program hang if nobody closes p[1]?
(b) Which process has to close p[1] for the child to see end of file? (Careful: there are two copies.)

P4. Shared memory vs message passing: which is faster, why, and what extra work does shared memory push onto the programmer?

P5. Direct vs indirect communication. Give one advantage and one disadvantage of a mailbox.

P6. Match each: a sender that never waits, a sender that waits for every message to be received, a rendezvous.
Options: zero capacity, bounded capacity, unbounded capacity, blocking send, non-blocking send.

P7. Two browser tabs on your laptop (IP 10.0.0.5) open connections to the same web server 93.1.1.1:443.
(a) Why can't both connections use the same local port?
(b) Write the two 4-tuples, with your own port numbers above 1024.

P8. RPC.
(a) What does the client stub do, and what does the server stub do?
(b) What is marshalling?
(c) read(fd, buf, n) passes buf by reference. Why is that hard to do with RPC?
