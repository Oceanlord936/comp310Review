# COMP 310 — Simulation Midterm 1 (L01–L08)

**Time: 90 minutes. Closed book. 100 points.**
Write your answers in `midterm_review/sim_test_1_answers.md` (number them), then tell Claude "grade".
Multiple-choice: circle **all and only** the correct answers (some have more than one).

Mix: Section A = 2022 midterm (adapted to this year's scope) · Section B = prof's "Questions from the Labs" ·
Section C = this year's slide-problem types with **new numbers** (the original slide problems are in
`00_quick_review.md`, solved during the lecture rounds).

---

## Section A — Concepts (old-midterm style) [34 pts]

**A1. [4]** A dual-mode CPU has privileged and normal instructions. For each of the four combinations
(normal/privileged instruction × user/kernel mode), give what the CPU does.

**A2. [5]** A C program calls a memory-manipulating function in user memory at line 99 and `read()` on a
disk file at line 100. Compare the processor mode at line 99 vs line 100 (line 100 expands into many
instructions). Which memory (user, kernel, both) is accessed during line 100, and during line 99?

**A3. [6]** For each named line, draw the user-space memory segments (TEXT, DATA, BSS, HEAP, STACK) and
mark every variable that exists at that point.
```c
int a;
int b = 10;
// LINE A
void f() {
    int z;
    int *q = malloc(sizeof(int));
    // LINE C
}
int main() {
    int x, y;
    // LINE B
    f();
}
```

**A4. [4]** Give every possible process state at each line, with a one-line reason:
```c
main() {
(A)  int x;
(B)  scanf("%d", &x);
(C)  process(x);       // long CPU-bound computation
}
```

**A5. [4]** (a) `fork()` copies which of: a. text segment  b. data segment  c. stack segment
d. file-descriptor table  e. none?  (b) `exec()` copies which of the same list? Justify each in one line.

**A6. [3]** When a process is context switched, do we need to save its memory? Why or why not?

**A7. [4]** (a) Why is interrupt processing essential for the OS itself, not just for devices?
(b) Why should I/O be asynchronous?

**A8. [4]** What is the key difference between uniprogramming and multiprogramming? A job alternates 3 s of
CPU and 2 s of I/O, once. Two such jobs arrive together. Give the finish times and CPU utilization
under each (assume perfect overlap under multiprogramming).

## Section B — Lab questions (xv6 / GINI context) [24 pts]

**B1. [3]** On an "idle machine" in the labs: how many RUNNING processes? How many RUNNABLE could there
be? How many SLEEPING? Explain who they are.

**B2. [4]** Name the three major types of trap. Which one's rate is independent of how many processes you
run, and why? You keep catching one trap type that sometimes lands in user space and sometimes in kernel
space — which type, and how is that possible?

**B3. [4]** How does the OS know why it trapped? Write the essential part of `usertrap()` that dispatches
on it (pseudo-C is fine).

**B4. [3]** What is stored in and restored from the trapframe? What happens if it can't be restored? Is
the trampoline the same thing?

**B5. [3]** Why are system calls identified by a number rather than a string or an address?

**B6. [3]** What data structure does xv6 use to keep process control blocks? Good or bad choice, and what
assumption are the designers making?

**B7. [2]** Can a process be RUNNING and RUNNABLE at the same instant? Can processes in the ready queue
have different state values?

**B8. [2]** In the A-Lab you kept a ring buffer of load samples in the kernel. Why was that necessary —
why can't `sysinfo` just measure the load average directly when called?

## Section C — Problems (slide problem types, NEW numbers) [42 pts]

*Same kinds of problems as the slide EXPs, with different data — so this checks whether you can do them,
not whether you remember the slide's answers.*

**C1. [6] Scheduling.** A arrives 0, burst 7 · B arrives 2, burst 4 · C arrives 4, burst 1 · D arrives 5,
burst 4. Gantt chart, average waiting time and average turnaround time for (a) FCFS (b) non-preemptive SJF
(c) SRTN (d) RR with q = 3. Ties: earlier arrival first; a process arriving at the same instant another is
preempted joins the queue first. (e) Which wins on average waiting time? On response time?

**C2. [3]** Exponential averaging, α = 0.5, initial guess τ₀ = 8. Actual bursts: 4, 8, 12, 2, 6.
Give τ₁ … τ₅. What does choosing α = 1 mean? α = 0?

**C3. [3]** Stride scheduling, W = 1200. Tickets: A = 400, B = 300, C = 200. Passes start at 0; ties go
alphabetically. List who runs in the first 9 quanta with the passes after each. Does the run count match
the ticket ratio?

**C4. [4]** Free holes, in order: 300K, 600K, 350K, 200K, 750K, 125K. Requests, in order: 115K, 500K, 358K,
200K, 375K. Placement under first fit, best fit, worst fit. Which fail? Sum the leftover holes for each.

**C5. [3]** Base 120000, limit 45000, addresses physical. (a) `load #164999` — allowed? Show both
comparisons. (b) `store #165000` — what happens, and who handles it? (c) With relocation 120000 / limit
45000 (addresses logical): which logical address reaches physical 150500? What happens to logical 45000?
Count comparisons and additions per access.

**C6. [3]** Tiny machine: 128-byte virtual address space, 32-byte pages, 256 bytes of physical memory.
Page table VPN→PFN: 0→5, 1→2, 2→7, 3→0. (a) Bits in VPN, offset, frame number? (b) Translate virtual 0, 45,
70, 127 (show 45 in binary). (c) Which bits changed?

**C7. [4]** (a) Memory 50 ns, 3-level page table, TLB lookup free: EAT with no TLB, and at hit rates 0.95 and
0.75. (b) `for (i = 0; i < 12; i++) sum += a[i];` with 4-byte ints, `a` at virtual address 56 (decimal),
16-byte pages, TLB empty: list hits and misses, give the hit rate. (c) How many page-table switches does xv6
do in one system call, and what happens to the TLB each time? Why doesn't Linux pay this?

**C8. [3]** (a) 32-bit addresses, 16 KB pages, 4-byte PTEs: offset bits, size of one linear page table, and
for 50 processes. (b) Same machine, multi-level so each table fits one page: entries per table page, levels,
bit split. (c) 48-bit virtual addresses, 4 KB pages, 8-byte PTEs: levels and split (which RISC-V scheme is
this?).

**C9. [3]** Memory 150 ns. A fault takes 6 ms with a free frame or clean victim, 18 ms when the victim is
dirty (40% of the time). (a) EAT if 1 access in 10,000 faults — how many times slower? (b) What fault rate
keeps the slowdown within 10%?

**C10. [4]** Reference string `2,3,2,1,5,2,4,5,3,2,5,2`, 3 frames, all empty at start. Count faults under
FIFO, OPT and LRU (show the frames). Which policy could show Belady's anomaly, and what property protects the
others?

**C11. [3]** Cache of 3 pages, references `4,7,0,7,1,0,1,2,1,2,7,1,2`. Hits, misses and hit rate under OPT,
FIFO, LRU. Then recompute each hit rate ignoring compulsory misses.

**C12. [3]** Clock with five frames P, Q, R, S, T, reference bits 0, 1, 1, 0, 1, hand at **Q**. (a) A fault
needs a frame: which page is evicted? The new page U arrives with bit 1 — give all bits and the hand position
after. (b) Another fault right after: who is evicted now, bits and hand after? (c) Enhanced second chance:
rank the four (reference, dirty) classes from best to worst victim.

---
*Bonus (not scored): explain thrashing with the CPU-utilization-vs-number-of-processes graph, and why
copy-on-write makes fork cheap.*
