# COMP 310 round guide: L09 recap, Tutorial 4, L10, L11, L12 so far

How to use this:

- Do one round at a time: read the concepts, then try every question without looking back.
- Type your notes in `L09_L11_notes.md`, and your answers in `L09_L11_answers.md` or in chat. Then say "check round B".
- Guessing or leaving a blank is fine. I check, explain, and put the right answer in your answers file.
- Answers are NOT in this guide. Only the worked examples are solved.
- Stuck on a concept? Ask me before the questions, that is fine too.

| Round | Material                                                              | Source                        |
| ----- | --------------------------------------------------------------------- | ----------------------------- |
| A     | L09 quick recap                                                       | N09, L09.md, L09b.md, L09c.md |
| B     | Semaphore vs monitor vs condition variable, in depth (your weak spot) | N09 slides 22-27, class14     |
| C     | Tutorial 4 examples, slowly                                           | tutorial4                     |
| D     | L10 file system concepts                                              | N10-File-System, class14-15   |
| E     | L11 implementation + the disk access examples                         | N11, class15-16, tutorial4    |
| F     | L12 so far (Amdahl)                                                   | N12 up to EXP-16, class16     |

Scope: N10-The-File-System-Interface is skipped, like you said. L12's SMP / NUMA / EXP-17 and N13 are left out because they aren't in your notes yet.

===================================

## The mistakes I fixed in your notes

Your files in `Notes/` are now corrected. The originals are in `L09_L10_review/notes_original/`. Each real fix is marked "(fix: ...)" in place. These ones change the meaning, so read them:

| File      | Your notes said                                                        | Correct                                                                   |
| --------- | ---------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| tutorial4 | Event monitor: `while (!done) ;` spin                                  | `while (!done) cwait(c);` Spinning inside a monitor blocks push() forever |
| tutorial4 | startWrite() had no waitingWriters++                                   | it needs `waitingWriters++` before the while                              |
| tutorial4 | writer goes "OR when writing is on"                                    | writer goes only when no readers AND not writing                          |
| tutorial4 | heading "use semaphore to make a Monitor"                              | that code is a monitor making a semaphore                                 |
| tutorial4 | csignal() body was missing sem.signal(), cwait() was missing its sleep | filled in (my reconstruction, check with the TA)                          |
| class16   | /user/abc/xyz.c takes 6 accesses                                       | 7 with the root i-node in memory (2k+1), 8 without                        |
| class16   | 700 \* (4096/68) = 12 blocks                                           | it is 700 / (4096/68)                                                     |
| class16   | 4096 \* 2000 = 9192000                                                 | 8,192,000                                                                 |
| class16   | i-node 100 "takes 2 blocks"                                            | it is IN the 2nd block, you read just that 1 block                        |
| class16   | super block is an i-node                                               | it is its own block: the geometry of the partition                        |
| class16   | release a block: bit 1 -> 0                                            | with 1 = free, release sets the bit to 1 (the slide agrees)               |
| class16   | hard link: count 1 -> 0, file survives                                 | two names means count 2, rm makes 2 -> 1                                  |
| class14   | semaphore empty = 0, an extra wait(empty)                              | empty = N, one wait(empty)                                                |
| class14   | bank `if (balance > x)`                                                | `>=`, you "need at least x"                                               |
| class14   | third append: "n = 2 -> 5"                                             | 2 -> 3                                                                    |

Two things to confirm with the prof or TA:

- Whether the root i-node counts as a disk access. Until you hear otherwise, use 2k+1 and write your assumption down.
- The "68 byte i-node" breakdown. Only 48 and 68 were in your notes, the split of the rest is my guess.

Also: your T3.1-T3.5 problems aren't saved anywhere in the repo. Rounds B and C replace them.

===================================

# Round A: L09 quick recap

1. The problem

counter++ is 3 instructions: load, add, store. A switch between a load and its store loses an update. Whoever stores LAST wins.

Race condition = shared data + the result depends on who runs when. The cure is mutual exclusion on the critical section.

---

2. The 4 requirements

(1) mutual exclusion: no two inside at once
(2) no assumptions about speed or number of CPUs
(3) progress: nobody OUTSIDE the critical section blocks another
(4) bounded waiting: nobody waits forever

---

3. The failed software attempts

| Attempt                   | What goes wrong                                    | Breaks       |
| ------------------------- | -------------------------------------------------- | ------------ |
| Disable interrupts        | not allowed for user code, other CPUs keep running | -            |
| Lock variable             | both read 0, both set 1, test and set are separate | (1)          |
| Strict alternation (turn) | a process outside still holds the turn             | (3)          |
| Check flag, then set      | both check before either sets                      | (1)          |
| Set flag, then check      | both set, both wait                                | (4) deadlock |

Peterson fixes it for 2 processes:

```c
flag[i] = true;   // I want in
turn = j;         // but you go first
while (flag[j] && turn == j)
    ;
/* critical section */
flag[i] = false;
```

The last one to write turn is the one that waits. It only works if loads and stores aren't reordered, and it still spins.

---

4. Hardware helps

Barrier: every load and store before it completes before any after it.

```c
// test-and-set: atomic. returns the old value, writes 1
while (test_and_set(&lock)) ;     // whoever sees false got the lock

// compare-and-swap: atomic. writes new only if the value == expected. always returns the old value
while (compare_and_swap(&lock, 0, 1) != 0) ;

// atomic increment: retry until nobody changed v between the read and the CAS
do { temp = *v; } while (temp != compare_and_swap(v, temp, temp + 1));
```

xv6 spinlock: acquire() turns interrupts off on this CPU, then does an atomic swap. Interrupts must be off, or a handler on the same CPU that wants the same lock spins forever.

---

5. Spin or sleep

Short wait: spin. Long wait: sleep. On one CPU spinning only wastes the time slice.

```c
struct semaphore { int count; queue q; };

wait(s)   { s.count--; if (s.count < 0)  { put me in s.q; block me; } }
signal(s) { s.count++; if (s.count <= 0) { take P from s.q; make P ready; } }
```

Rules:

- wait may block, signal NEVER blocks
- you can't read a semaphore's value
- a negative count = how many are blocked
- the starting value has a meaning: 0 = event not happened yet, 1 = lock, N = N free units

---

6. The semaphore patterns

| Pattern        | Semaphores                  | Key line                                                       |
| -------------- | --------------------------- | -------------------------------------------------------------- |
| Signalling     | s(0)                        | A: a1; s.signal() / B: s.wait(); b1                            |
| Rendezvous     | a(0), b(0)                  | signal your own FIRST, then wait for the other                 |
| Mutex          | m(1)                        | wait; critical section; signal                                 |
| Barrier        | mutex(1), barrier(0), count | last one signals, then everyone does wait + signal (turnstile) |
| Bounded buffer | mutex(1), empty(N), full(0) | wait for the resource BEFORE the mutex                         |

Never block on a resource while holding the mutex.

---

7. xv6 has no semaphores or monitors, only sleep(channel) / wakeup(channel), with a while re-check (Mesa style). sleep_prepare() registers before the lock is released, so no wakeup gets lost.

===================================

## Round A questions

A1. counter = 10. T1 and T2 both run counter++. Write an interleaving of the load / add / store lines that ends at 11.

A2. Which of the 4 requirements does each break, and with what interleaving in one line: (a) lock variable, (b) strict alternation, (c) set flag then check.

A3. Peterson: P0 and P1 both want in. P0 writes turn first, then P1 writes turn. What is turn's final value, and who enters?

A4. v = 7. What does compare_and_swap(&v, 5, 6) return, and what is v after? Then compare_and_swap(&v, 7, 8)?

A5. Why does xv6's acquire() turn off interrupts, and why would spinning on a 1-CPU machine with no preemption hang?

A6. A blocking semaphore starts at 2. Five threads call wait(). Then another thread calls signal(). Give the count after all the waits, the count after the signal, and how many are still blocked.

A7. Rendezvous: why does putting wait before signal in BOTH threads deadlock?

A8. Bounded buffer, N = 4. The producer inserts 3 items, the consumer removes 2. Give mutex, empty, full.

A9. Swap the CONSUMER's wait(&full) and wait(&mutex). Describe a run that deadlocks. Start from the empty buffer.

===================================

# Round B: semaphore, monitor, condition variable

This is the part you said you need to go through. The main idea first:

A semaphore is a counter that REMEMBERS. A condition variable remembers NOTHING. All the differences follow from that.

---

1. The three tools

|                            | Semaphore                               | Monitor                                   | Condition variable            |
| -------------------------- | --------------------------------------- | ----------------------------------------- | ----------------------------- |
| What it is                 | counter + queue                         | a box: shared data + its procedures       | a waiting room inside the box |
| Stands alone?              | yes                                     | yes                                       | no, only inside a monitor     |
| Mutual exclusion           | you write wait(mutex) ... signal(mutex) | automatic, one active thread              | no, the monitor gives it      |
| Wait                       | wait(s): blocks only if count < 0       | -                                         | cwait(c): ALWAYS sleeps       |
| Signal with nobody waiting | count goes up, it is remembered         | -                                         | LOST                          |
| Where's the state?         | inside the count                        | your own variables (count, done, balance) | none                          |

---

2. The picture of a monitor

```
   entry queue            +------------- monitor --------------+
 T5 T6 T7  ------------>  |  shared data: count, buffer ...    |
 (waiting to come in)     |  procedures: append(), take()      |
                          |                                    |
                          |  only ONE active thread in here    |
                          |                                    |
                          |  notfull  room:  P2 P3 (asleep)    |
                          |  notempty room:  (nobody)          |
                          +------------------------------------+
```

Threads asleep in a room are not active, so they don't block the box. That answers the class question "if we sleep inside the monitor, how does anyone get in to wake that thread?"

---

3. What cwait and csignal do

cwait(c):

- go to sleep in room c
- give up the box, so another thread can enter
- when woken, get the box back, and carry on from the line AFTER cwait

csignal(c):

- wake ONE thread in room c
- if the room is empty, nothing happens at all

cbroadcast(c):

- wake EVERY thread in room c

---

4. Why a flag or count is always there

A condition variable has no memory, so YOU keep the state in a normal variable and check it before sleeping:

```c
while (I can't continue)    // look at your own variables
    cwait(room);
// change the shared state
// csignal or cbroadcast whoever may be able to continue now
```

This is the recipe for every monitor. Bounded buffer, readers-writers and the Event monitor in round C all follow it.

---

5. Hoare vs Mesa: same code, different order

Bounded buffer with N = 3, using the class example. The buffer is full, count = 3. P4 called append and is asleep on notfull. Now C1 calls take():

Hoare:

```
C1  take: count 3 -> 2, csignal(notfull)
P4  runs AT ONCE, C1 waits in the urgent queue
P4  the condition is guaranteed true. appends, count 2 -> 3, leaves
C1  continues, leaves
```

Mesa:

```
C1  take: count 3 -> 2, csignal(notfull)   P4 becomes ready but does not run
C1  keeps running, leaves
??? anyone in the entry queue may get the box before P4
P4  gets the box later. the condition MAY be false again
```

So under Mesa it must be while: being woken only means "it was true at some point". Under Hoare, if is enough. while also works there, it just re-checks for nothing. That's why while is the safe habit, and almost every real system is Mesa.

---

6. csignal or cbroadcast?

csignal: any one woken thread can use the change (one free slot: one producer).

cbroadcast: several can continue (the Event flag, all the readers after a writer), or you can't tell which one can (see B5). With while loops, waking too many is only a little slow. Waking too few can leave someone asleep forever.

---

7. Bank monitor: why no race

Only one thread is active in the box, so balance = balance - x can't interleave with balance = balance + y. Your class notes have this version. Its weak point: when the balance is too low, withdraw just does nothing and returns. The prof's exercise is to make it WAIT instead. That's B4.

===================================

## Round B questions

B1. Two threads, in this exact time order: T1 signals, then T2 waits.
(a) With a semaphore s(0): does T2 block? What is the count at the end?
(b) With a condition variable c: does T2 block? Why?

B2. Monitor bounded buffer with `if`, Mesa, N = 2. Trace it and give count after each step:
P1 append, P2 append, P3 append (sleeps), C1 take (signals), C1 leaves, P4 enters BEFORE P3 runs and appends, then P3 runs.
(a) What goes wrong with `if`? What is count at the end?
(b) What does P3 do with `while`, and what is count at the end?

B3. The prof's question: how is the monitor version of producer-consumer similar to the semaphore version, and how is it different? Compare at least: who provides the mutual exclusion, what stops the producer when full, where the count lives, and what mistake each one makes easy or hard.

B4. The prof's exercise: write the bank monitor with a condition variable, so withdraw(x) WAITS until balance >= x. Assume Mesa.

B5. Bank monitor, Mesa. balance = 0. W1 calls withdraw(100), then W2 calls withdraw(10). Both sleep. Then deposit(50) runs.
(a) If deposit uses csignal and the room is FIFO, what happens to W1 and W2? Is that a problem?
(b) What should deposit use instead, and what happens then?

B6. True or false, with one line why:
(a) A thread asleep on cwait still counts as active in the monitor.
(b) You should put wait(mutex) at the top of each monitor procedure.
(c) Semaphore wait() always blocks.
(d) cwait() always blocks.
(e) `while` around cwait is wrong under Hoare.
(f) A semaphore signal() can block the caller.

B7. Make sure a1 runs before b1, using a monitor instead of a semaphore. Write it. Why do you need one more variable than the semaphore version needs?

===================================

# Round C: the Tutorial 4 examples

Keep your fixed `tutorial4.md` open next to this.

1. Dining philosophers

Deadlock needs all 5 holding one fork and waiting in a circle. Both fixes break the circle:

Fix 1, the footman: maxP = Semaphore(4). Only 4 at the table, so 5 forks among 4 people. By counting, someone gets two forks.

Fix 2, a left-hander: 4 pick right first, 1 picks left first. Two neighbours go for the SAME first fork. The loser holds nothing while waiting, so the circle can't close.

Neither gets past 2 eating at once: 5 forks / 2 per eater. The fixes remove deadlock, they don't create forks.

---

2. The Event monitor: one-time "it has happened"

Attempt 1 has no state, so it hits round B's main idea:

```
push() runs first   -> csignal, nobody waiting, LOST
wait() runs later   -> cwait, sleeps forever
```

The fix adds `done`. A wait() after the push sees done == true and doesn't sleep at all:

```c
void push() { done = true; cbroadcast(c); }
void wait() { while (!done) cwait(c); }
```

This is a very common shape: a condition variable plus a flag behaves like a semaphore's memory.

Why spinning in there is fatal: the spinner stays ACTIVE in the box, and only one active thread is allowed. push() waits in the entry queue forever, so done never becomes true.

---

3. Readers-writers

The rules: many readers at once, OR one writer alone.

The trick: the monitor doesn't protect the database. It protects the bookkeeping. The actual read happens OUTSIDE:

```
RW.startRead();    <- box, only to update counters. leaves the box
read the database  <- many readers can be here at once
RW.endRead();      <- box again
```

If the read were inside the box, readers would run one at a time and you'd lose the whole point.

State: readers (reading now), writing (true or false), waitingWriters.

The priority is in one line of startRead: `while (writing || waitingWriters > 0)`. A new reader waits when a writer is waiting. So this version favours writers.

---

4. A semaphore built from a monitor

```c
void wait()   { while (value == 0) cwait(queue);  value--; }
void signal() { value++;  csignal(queue); }
```

value stores what a semaphore remembers, so the signal is no longer lost: if nobody is waiting, value still goes up. It doesn't go negative: here sleepers are counted by the room, not by the number.

---

5. A condition variable built from semaphores

The other direction. A semaphore remembers, so it would be wrong to always call sem.signal(). The waiters count exists to make a signal do nothing when nobody is waiting:

```c
csignal(c) {
    c.mutex.wait();
    if (c.waiters > 0) { c.waiters--; c.sem.signal(); }   // only if someone is really waiting
    c.mutex.signal();
}
```

cwait() must leave the monitor before sleeping, then come back in. Sleeping while holding the box would block everyone.

So semaphores and monitors have the same power: each can build the other.

===================================

## Round C questions

C1. Footman solution. Each philosopher takes fork[i] then fork[(i+1)%5]. They arrive in order P0, P1, P2, P3, P4, and each one gets as far as it can before the next arrives. Give maxP's count at the end, who holds which fork, who is blocked on what, and who eats.

C2. Left-hander solution. P4 takes fork[0] first and fork[4] second, the others as in C1. Same arrival order. Who holds which fork, who is blocked on what, and who eats?

C3. Event monitor.
(a) Three threads are in wait() and push() uses csignal instead of cbroadcast. What happens to them?
(b) `done` never goes back to false. So would `if (!done) cwait(c);` be safe here? Why, and why does the bounded buffer need while when this doesn't?

C4. Readers-writers. Fill in this table after each event. Use the fixed code.

| Event                      | readers | writing | waitingWriters | asleep on okToRead | asleep on okToWrite |
| -------------------------- | ------- | ------- | -------------- | ------------------ | ------------------- |
| R1 startRead               |         |         |                |                    |                     |
| R2 startRead               |         |         |                |                    |                     |
| W1 startWrite              |         |         |                |                    |                     |
| R3 startRead               |         |         |                |                    |                     |
| R1 endRead                 |         |         |                |                    |                     |
| R2 endRead (W1 then runs)  |         |         |                |                    |                     |
| W1 endWrite (R3 then runs) |         |         |                |                    |                     |

C5. (a) In the code above, who can starve, and with what kind of arrival pattern?
(b) Change one line so that readers get priority. Who can starve now?

C6. Your original notes had no waitingWriters++ in startWrite. With that bug, what value does waitingWriters take after a writer gets through? How does startRead's priority check behave?

C7. Why does endRead use csignal(okToWrite), but endWrite uses cbroadcast(okToRead)?

C8. The semaphore from a monitor, value = 1, Mesa. Give value and who is where after each step:
T1 wait, T2 wait, T3 wait, T1 signal, then T4 calls wait BEFORE T2 gets to run, then T2 runs.
Why does this show the while is needed? In the real blocking semaphore from L09, what would the count be after the three waits, and how does it differ from value here?

C9. The condition variable from semaphores. Someone deletes the `waiters` check, so csignal always calls c.sem.signal(). Nobody is waiting, csignal runs, and later a thread calls cwait. What happens? Which rule of condition variables did this break?

C10. Write a monitor for a barrier: arrive() blocks until n threads have called it, then all continue. It only needs to work once. Assume Mesa.

===================================

# Round D: L10 file system concepts

1. What a file is

A named collection of related information. It can be accessed by addressing.

Logical view: what users and apps see. Physical view: how it sits on storage.

Attributes: name, identifier, location, size, protection, timestamps.

---

2. Where a file operation happens

```
Process  -> triggers open / read / write, gives the buffers
Kernel   -> really implements them (privileged), holds buffers + tables
Disk     -> all the persistent structures live here
```

---

3. The operations

| Operation         | What it does                                                                                                     |
| ----------------- | ---------------------------------------------------------------------------------------------------------------- |
| Create            | check permission on the directory, insert the name, allocate blocks, point the entry at them                     |
| Open              | check permissions. Must succeed before any other operation                                                       |
| Write             | the permission from open must include write. Location matters: sequential or random                              |
| Read              | permission from open must include read, needs a memory buffer, can return LESS than you asked, moves the pointer |
| Reposition (seek) | moves the file pointer to a given value                                                                          |
| Delete            | check permission, remove the entry. If other links remain the blocks stay, otherwise they are recovered          |
| Truncate          | length set to the given size, blocks holding user data are recovered                                             |

The information the OS keeps for an open file:

- File pointer: where the next read or write goes. Unique to each process
- Open count: how many have it open, so the table entry can be released
- Location: which disk blocks, so you don't re-read the directory every time
- Access rights: the access it was opened for. Anything else is denied

---

4. Access methods and types

Sequential: read_next() and write_next(), the pointer moves by itself. Editors and compilers use it.

Direct: fixed-length records, read or write any record. You can emulate sequential with it by keeping a position, and it can be used to build indexed files.

Types: most are shown by extension. The OS really only cares about a few, like executables. UNIX identifies them with magic numbers (a fingerprint in the file).

---

5. Directories

A directory is a symbol table: name -> file control block. The FCB says where the data is.

Operations: create, delete, list, rename, traverse.

| Structure     | Idea                                                           | Limit                                              |
| ------------- | -------------------------------------------------------------- | -------------------------------------------------- |
| Single-level  | one list for all                                               | unique names, only good for small embedded devices |
| Two-level     | MFD indexed by user -> each user's UFD                         | weak sharing, system files are awkward             |
| Tree          | sub-directories, current directory, relative or absolute paths | no sharing                                         |
| Acyclic graph | links to the same instance, not copies                         | must keep cycles out                               |

Hard link: another name for the same file, files only, so no cycles.
Soft link: may point to directories, so cycles are possible.

---

6. Memory-mapped files

Map the file into the address space with virtual memory. Plain loads and stores replace read() and write(). A write may stay in memory until close or until the page is flushed.

===================================

## Round D questions

D1. The prof's opening questions:
(a) We have in-memory data structures. Why do we need a different abstraction, files, for secondary storage?
(b) Why is open a key operation?
(c) What kinds of failures can a file system have?

D2. The prof's question: a file system can be copied at file level or at block level.
(a) When would you use each?
(b) Which is faster at copying a whole file system, and why?

D3. A program opens /a/b/c and reads it with ten read() calls. How many times is the path resolved? How many if there were no open() and each read() took the path? (from the skipped deck, but it's the point of open)

D4. A file is 1000 bytes and the pointer is at 800. What does read(fd, buf, 500) return? And the read after that?

D5. A process opens a file for reading. The owner then removes read permission. Can the process keep reading? Why?

D6. Records are 64 bytes. At which byte does record 7 start? Which record holds byte 1000?

D7. Why is the file pointer kept per process, while the location of the file's blocks can be shared?

D8. Which directory structure does each problem come from, and what solved it?
(a) Two users both want a file named notes.txt.
(b) One user has 10,000 files and can't organize them.
(c) Two users want to share one live copy of a project folder.

D9. Why do hard links to directories create a problem that hard links to files don't? What does a soft link risk instead?

D10. A program uses mmap to change a file, and the machine loses power before close. What could be lost, and why doesn't this happen in the same way with a sync write?

===================================

# Round E: L11 implementation

1. The layout

```
whole disk:
MBR | partition table | partition A | partition B | partition C

one partition:
boot block | super block | free space mgmt | ... i-nodes ... | ... data blocks ...
```

- Boot block: bootstrap code for the OS in the partition
- Super block: geometry of the partition. Where is everything?
- i-nodes: this is where the FCBs are
- Data blocks: file contents AND directories

A directory is a file, and its rows are name : i-node number.

Directory as a linear list: searched at creation for uniqueness, slow on big directories. Directory as a hash table: fast lookup, but you must deal with table size and collisions.

---

2. The allocation methods

|                        | Contiguous                      | Linked                         | FAT                            | Indexed (i-node)                         |
| ---------------------- | ------------------------------- | ------------------------------ | ------------------------------ | ---------------------------------------- |
| Idea                   | file sits in consecutive blocks | each block points to the next  | the links moved into one table | one block holds all pointers             |
| Sequential             | good                            | good                           | good                           | good                                     |
| Direct                 | good, just compute              | bad, walk the chain            | good if the FAT is in memory   | good                                     |
| External fragmentation | yes, needs compaction           | no                             | no                             | no                                       |
| Growing                | hard                            | easy                           | easy                           | easy, until the index is full            |
| Weak spot              | finding a hole                  | one bad block breaks the chain | the table itself is critical   | pointer overhead, internal fragmentation |

---

3. The i-node

12 direct pointers, 1 single indirect, 1 double, 1 triple. With block size B and pointer size d:

```
max size = 12*B + (B/d)*B + (B/d)^2*B + (B/d)^3*B
```

Each layer multiplies by B/d, the number of pointers that fit in one block.

Worked, with B = 1024, d = 4: B/d = 256

| Region | Blocks | Bytes it adds | Offsets it covers |
| ------ | ------ | ------------- | ----------------- |
| direct | 12     | 12,288        | 0 to 12,287       |
| single | 256    | 262,144       | 12,288 to 274,431 |
| double | 256^2  | 67,108,864    | from 274,432      |

---

4. Worked: finding the n-th byte (Tutorial 4)

Assume the file is open, so its i-node is in memory.

n = 5000: 5000 / 1024 = block 4, offset 904. 4 < 12, so direct. 1 disk read (the data).

n = 100,000: 100,000 / 1024 = block 97, offset 672. 97 >= 12, so subtract: 97 - 12 = 85. 85 < 256, so single indirect, entry 85. 2 disk reads (the indirect block, then the data).

The steps:

- block number = n / B, offset = n % B
- subtract 12, then 256, and so on, until it fits a region
- reads = the number of pointer blocks + 1 for the data

---

5. Worked: how big is the i-node table? (class 16)

2000 blocks of 4096 bytes. i-node 68 bytes. Average file 3 blocks.

```
files: 2000 / 3 = about 666 -> round up to 700 i-nodes (the spare 34 are for directories)
i-nodes per block: 4096 / 68 = about 60
table: 700 / 60 = about 11.6 -> 12 blocks
```

---

6. Worked: where is i-node 100? (class 16)

```
byte: 100 * 68 = 6800
block: 6800 / 4096 = 1 (so the 2nd block of the table)
offset: 6800 - 4096 = 2704
```

Disk reads always take a whole block, so you read that block and look at offset 2704. An i-node number gives you the disk block directly, no searching.

---

7. Worked: the path walk (class 16 + Tutorial 4)

/user/abc/xyz.c: k = 3 names (user, abc, xyz.c). No caching. Each directory is one block.

```
(i-node 0, root)      read from disk only if it's not already in memory
1  root data block    "user : 10"
2  i-node 10
3  user data block    "abc : 21"
4  i-node 21
5  abc data block     "xyz.c : 30"
6  i-node 30          <- open() finishes here
7  first data block   <- the first byte
```

2 reads per name (the directory's data, then the next i-node) + 1 for the byte = 2k + 1 = 7.

If the root i-node must also be read: 8. Always say which one you assumed.

Notice:

- i-node then data, i-node then data, always alternating.
- Deep paths cost more. Caches help, and a relative path from the current directory skips the upper levels.

---

8. Worked: rm and the link count (class 16)

The i-node stores a link count: how many directory entries point to it.

/home/xy.c -> i-node 31, count 1, blocks 251, 3090, 3100.

```
rm /home/xy.c:
remove the row "xy.c : 31" from home's directory block
count 1 -> 0, so nobody uses the file now
free blocks 251, 3090, 3100: set their bits in the bitmap (1 = free)
free i-node 31
```

Hard link: /user/abc/xyz.c also -> i-node 31. Now the count is 2, and rm /home/xy.c only makes 2 -> 1. No blocks are freed.

---

9. Free space

Bitmap: 1 bit per block, 1 = free. CPUs have instructions that scan bits fast.

Size: (disk size / block size) bits. 1 TB with 4 KB blocks: 2^40 / 2^12 = 2^28 bits = 2^25 bytes = 32 MB.

Linked free list: each free block points to the next. Adding is cheap, and traversal isn't done often. For contiguous allocation, keep a pointer + a count of contiguous free blocks.

===================================

## Round E questions

E1. Put in disk order, and say in a few words what each is for: i-nodes, super block, partition table, data blocks, boot block, master boot record, free space management.

E2. A file has 1000 blocks. How many disk reads to read ONLY block 500 (blocks numbered from 0)? The file's directory entry or i-node is already in memory.
(a) contiguous (b) linked (c) FAT, with the table cached in memory (d) one-level indexed

E3. B = 4 KB, d = 4 bytes. What is the largest file? Give each region's size, then the total (use 1 KB = 1024 and so on).

E4. B = 1 KB, d = 8 bytes. Same question.

E5. B = 1024, d = 4, the i-node is in memory. For each byte offset: the block number, the offset inside it, which region, and how many disk reads.
(a) 11,000 (b) 12,288 (c) 300,000

E6. The disk has 8000 blocks of 4096 bytes. i-nodes are 128 bytes. You plan for 2000 files and 48 directories. How many i-node blocks do you need? And what is the disk's capacity in bytes?

E7. Same i-node size. The i-node table starts at disk block 50. Which disk block holds i-node 100, and at what offset? Which other i-nodes come into memory with that one read?

E8. Path /home/alice/docs/report.txt. No caching, each directory is one block.
(a) How many disk reads to get the first byte, with the root i-node in memory?
(b) And if the root i-node must also be read?
(c) With (a)'s assumption, how many of those reads happen during open(), and how many during the first read()?
(d) The current directory is /home/alice/docs and its i-node is in memory. How many reads to get the first byte of ./report.txt?
(e) After open, with B = 1024 and d = 4, how many reads to get byte 100,000 of the file?

E9. The class question: how many disk accesses for rm /home/xy.c? Here reads AND writes both count. Assume: root i-node in memory, link count 1, 3 direct blocks, all 3 bits in one bitmap block, the bitmap is not cached, and don't count any tracking of free i-nodes.

E10. Start with only /user/abc/xyz.c -> i-node 31, count 1, blocks 251, 3090, 3100.
Run in order: `ln /user/abc/xyz.c /home/xy.c`, `rm /user/abc/xyz.c`, `cat /home/xy.c`, `rm /home/xy.c`.
(a) Give the link count after each one, and say whether the cat works.
(b) When do the bitmap bits change, and to what?
(c) Same idea with a SOFT link at /home/xy.c. What does the cat do after the first rm, and why?

E11. Blocks 1, 2, 5, 9, 10, 11 are free on a 16-block disk.
(a) Write the bitmap (1 = free), block 0 first.
(b) Blocks 3 and 4 are released. Write the new bitmap.
(c) How large is the bitmap of a 2 TB disk with 4 KB blocks?

E12. Why can linked allocation lose a file to one bad block, and how does keeping the links in a FAT change direct access?

E13. A directory has 100,000 entries. Why is a linear list a problem when you create a file, and what does a hash table fix? What new issue does it bring?

===================================

# Round F: L12 so far

1. Ways to get compute

- Cluster: commodity machines, loosely connected. Cheap to scale, slow messages. Good when each task is relatively big.
- Parallel computer: expensive because of the tight connections (see top500.org).
- Cloud: massive clusters, split into smaller nodes with virtualization. Pay for what you use, so cost follows demand.

---

2. Scaling

Horizontal: more servers. The problem must split across machines, needs distribution of data and accumulation of the results.

Vertical: a bigger server. The application doesn't change, but cost rises rapidly with machine size.

Which is better? It depends if the problem can be split.

Finance company: buying 6 servers is fine while customers pay, but when students leave for summer the cost stays high. Outsourcing lets the cost follow the demand.

---

3. Amdahl's law

a = the fraction of the OLD TIME that can be enhanced. k = how many times faster that part gets.

```
Tnew = Told * [(1 - a) + a / k]

S = Told / Tnew = 1 / [(1 - a) + a / k]
```

The part you can't enhance limits you, no matter how big k is.

Worked: an app takes 100 s, 40 s of it can run 2x faster. a = 0.4, k = 2.

```
Tnew = 100 * (0.6 + 0.2) = 80 s
S = 100 / 80 = 1.25
```

===================================

## Round F questions

F1. When would you scale horizontally, and when vertically? Give one reason for each.

F2. EXP-16. Driving 2500 km at 100 km/h takes 25 h. 1500 km of the trip is Montana with no speed limit, and your truck can do 150 km/h there. What is the speedup? Find it with the formula, and check it directly with the times.

F3. EXP-16, same trip. How fast must you go through Montana to get an overall speedup of 1.67?

F4. EXP-16. An app takes 30 min on one core. On a 4 core machine it takes 18 min. What portion could be enhanced?

F5. An app has a = 0.3. What is the best possible speedup, however many machines you buy?
