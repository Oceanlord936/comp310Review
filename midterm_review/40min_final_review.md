# COMP 310 Midterm 1: 40-minute final review (L01–L08)

Exam: **12 questions in 80 min, about 6.5 min each.** Write formulas first, then plug in, and always write the final number.

| block | topic | min |
|---|---|---|
| 0 | **Your personal traps** (read this first and last) | 4 |
| 1 | L01–L02: OS, dual mode, traps, syscalls | 6 |
| 2 | L03–L04: processes, memory map, fork/exec/wait, shell | 7 |
| 3 | L05: scheduling | 7 |
| 4 | L06: base/limit, binding, fits, fragmentation | 4 |
| 5 | L07: paging, address bits, TLB, multi-level | 6 |
| 6 | L08: page faults, COW, replacement, thrashing | 6 |

---

## 0. Your personal traps (from the ✗ marks in your Q2 checks) ⏱ 4 min

1. **Preemptive = the OS CAN take the CPU away** (timer, or a better job arrives). Non-preemptive = the job keeps the CPU until it blocks or exits. You had these swapped once.
2. **Address bits: add the offset ONCE.** 256 pages × 4 KB, 64 frames → logical 8 + 12 = **20**, physical 6 + 12 = **18**. Your earlier answers were 32 and 30 because the offset was counted twice.
3. **Words stay words.** "64 pages of 1024 words, 32 frames" → logical 6 + 10 = **16**, physical 5 + 10 = **15**. Don't convert words to bytes.
4. **TLB EAT: the TLB lookup is paid on every access, and a hit still reads memory once.**
   `EAT = t_TLB + h·t_m + (1−h)·(k·t_m + t_m)` → 2 + 0.75·50 + 0.25·150 = **77 ns**
5. **Gantt chart: the next job starts at the previous FINISH time**, never at the previous turnaround. Label your columns: TA = finish − arrival, wait = TA − burst.
6. **Priority:** check which way the numbers go ("larger = higher" in some questions). With priority + RR, **RR only rotates among EQUAL priorities**. A lone high-priority job just gets another quantum.
7. **An xv6 zombie still has its memory and page table.** `freeproc` frees them only when the parent's `wait` reaps it. On Linux the memory is freed at exit.
8. **Page-fault translation questions: resolve the fault.** Take the next free frame, update the page table, then translate. Don't stop at "not in memory".
9. **Memory map:** a `static` local goes in **bss** (not the stack), a string literal goes in **rodata/text** (not data or heap), and a `malloc` pointer is on the **stack** while its block is on the **heap**.
10. **Shell:** it must **loop**. `cd` is a built-in, and `exit(1)` goes right after `exec` (it is reached only if exec failed).
11. **Process counts:** n sequential forks → 2ⁿ processes **exist**, of which 2ⁿ − 1 are **created**. Give both.
12. **FIFO:** a hit does NOT reorder the queue. **OPT:** evict by the **distance to the next use**, not by how many times a page appears later.
13. **Working set changes → the fault rate goes UP** (new pages aren't loaded). It stays high if the new set doesn't fit.
14. **A TLB miss causes no process state change.** The hardware walks the table in ns. Only a page fault that needs disk takes the process from running to waiting.
15. **No hardware reference bit? Fake it with the VALID bit** (clear it, the next touch traps). Don't use the dirty bit: it only catches writes.
16. **"Paging the page table" has a specific benefit:** the table doesn't need to be contiguous, and pieces for unused regions are never allocated. "No external fragmentation" is the benefit of paging memory in general, so it doesn't answer this.

---

## 1. L01–L02: OS, dual mode, traps, syscalls ⏱ 6 min

**The OS is a resource manager.** It has to be able to *take the CPU back*, and that needs the **timer interrupt**. The kernel "supervises nothing, configures everything" using three hardware mechanisms it sets up:
- **mode bit:** decides what an instruction may do
- **page tables:** decide what memory is reachable
- **timer:** guarantees the kernel gets control back

**Dual mode:** in user mode, a privileged instruction **traps**. Root is still user mode, because kernel mode is a CPU state, not an account.

**Privileged or not?** (prof's sample list)

| action | answer |
|---|---|
| set the timer | privileged (could keep the CPU forever) |
| read the clock | **not** privileged (it harms nobody) |
| clear memory | privileged if it is arbitrary memory; clearing your own memory is just writes |
| disable interrupts | privileged |
| switch user → kernel | **only by trapping** (ecall), which lands at a kernel-chosen address |
| access an I/O device | privileged |

**Kernel, app or library?** Picking the next process and the disk driver are **kernel**. `printf` is a **library**: it formats text in your process and then calls `write`. The shell and the compiler are **applications**.

**Three kinds of trap**

| | cause | sync? | resumes |
|---|---|---|---|
| syscall | `ecall` (deliberate) | **sync** | AFTER the ecall (`epc += 4`) |
| interrupt | timer or device | **async** | ON the interrupted instruction |
| exception | bad address, illegal instruction | **sync** | re-run if fixed, else killed |

On every trap the hardware does four things: it saves the PC in `sepc`, saves the cause in `scause`, switches to kernel mode, and jumps to `stvec`. **The program does not choose where it enters the kernel**, and that is the whole security argument.

**Prof's classify-the-event questions:**
- `write()` → syscall, sync (scause 8).
- The timer fires during a tight loop → interrupt, async (scause 5). The kernel may **yield** and switch to another process.
- A jump into code not loaded yet → exception, sync (instruction page fault, scause 12). **Nothing is wrong** under demand loading: the kernel loads the page and retries.
- Only the **interrupt** can arrive at an instruction the program didn't choose, which is why it is the hardest to handle (synchronization).

**What if apps could disable interrupts?** The **immediate** problem is that the timer can't take the CPU back, so one loop hangs the machine. **Also:** device data is lost when buffers overflow, the clock stops, and I/O completions never wake anyone. It hurts **both** the offender (it may wait forever on its own I/O) and everyone else. The kernel itself disables interrupts only for a few instructions (spinlock).

**Syscalls**
- Called by **number**: the number goes in `a7`, the args in `a0–a5`, the result in `a0`.
- Three ways to pass arguments: registers, a memory block (pass its address), or the stack.
- The kernel **never trusts a user pointer**: it goes through `copyin`/`copyout`.
- **6 families:** process (fork, exec, exit, wait, kill), file (open, read, write, close), device (mknod, dup), info (getpid, uptime), communication (pipe), protection (chdir, link).
- Reading "hello" from the keyboard uses **read(0, …)**, and the keyboard **interrupt** fills the buffer. Launching a program from disk uses **fork + exec** (exec reads the file) + **wait**. Printing uses **write(1, …)**.
- xv6 `printf` makes one `write` per character.
- A short read is **not** an error.

**DMA vs interrupt-driven I/O:** with interrupt-driven I/O there is one interrupt per block. With DMA the CPU sets up the transfer once and gets **one interrupt** at the end.

---

## 2. L03–L04: processes, memory map, fork/exec/wait, shell ⏱ 7 min

**Why multiprogramming, even on ONE core?** While one process waits on I/O, another uses the CPU, so the gaps get filled.
- **Scenario:** 1 s CPU / 1 s I/O, 3 CPU + 2 I/O bursts.
  - Uniprogramming: A finishes at 5 s, B at 10 s.
  - Multiprogramming: A at 5 s, B at **6 s**, CPU 100% busy.
- **Model:** utilization = **1 − pⁿ** (p = fraction of time spent waiting on I/O). With p = 0.8: n = 1 → 20%, n = 4 → 59%, n = 10 → 89%.
- If every job is CPU-bound, throughput doesn't improve; what you still gain is **responsiveness**.

**Memory map**

```
high  stack  ↓   locals, args, return addr; one frame per active call
       ...
      heap   ↑   malloc'd blocks
      bss        uninitialized or zero globals, static locals
      data       initialized globals/statics (int b = 10;)
low   text       code + string literals (read-only)
```

Example: `char *banner = "sq";`. The pointer `banner` goes in **data**, and the chars `"sq"` go in **rodata/text**.

While the program runs, each call pushes a stack frame (popped on return), and each malloc grows the heap.

**States:** new → ready ⇄ running → terminated, plus running → waiting → **ready** (a waiting process never goes straight to running).
- xv6 states: UNUSED, USED, RUNNABLE, RUNNING, SLEEPING, ZOMBIE.
- The **PCB** (`struct proc`) holds the state, PID, registers (trapframe/context), page table, open files, and so on.
- A **context switch** is pure overhead. `swtch` saves 14 registers (the callee-saved ones plus ra and sp).

**fork / exec / wait**
- fork: the child gets **0**, the parent gets the **child's PID**. Memory is **copied**, not shared (x = 5, the child does x++, the parent still sees 5).
- exec: replaces text, data, heap and stack. It **keeps** the PID, parent, open fds and cwd. If it succeeds it **never returns**.
- `for(i=0;i<3;i++) fork(); printf("hi\n");` prints "hi" 8 times. With a nested `if(fork()==0) fork();` you get 3 processes.
- **Zombie:** exited, but the parent hasn't called wait. **Orphan:** the parent exited first, so the child is re-parented to init, which waits on it.

**Shell pseudocode** (know it cold):
```c
while (1) {
    printf("$ "); read line; parse into argv;
    if (argv[0] == "cd")   { chdir(argv[1]); continue; }  // BUILT-IN
    if (argv[0] == "exit") exit(0);
    int pid = fork();
    if (pid < 0) { print error; continue; }
    if (pid == 0) {                 // child
        // (redirect / pipe setup goes here, between fork and exec)
        exec(argv[0], argv);
        printf("exec failed\n");    // only reached on failure
        exit(1);                    // child must NOT fall back into the shell loop
    }
    wait(&status);                  // parent (skip if background "&")
}
```

**Why run the command in a child process?**
- If the shell called exec itself, it would be **replaced**.
- A crashing command kills only the child, and the shell keeps running and gets the status back from wait.
- The gap between fork and exec is where the shell sets up redirection and pipes.

**Why is `cd` built in?** cwd belongs to each process. A `cd` child would change only its *own* directory and then exit. `exit` is built in for the same reason.

---

## 3. L05: Scheduling ⏱ 7 min

**Mechanism vs policy.** The mechanism is the dispatcher, the timer and the context switch. The policy is *who runs next*. The OS's use case (batch, interactive, phone) decides which policy fits.

**When the scheduler decides:**

| transition | type |
|---|---|
| running → waiting | **non-preemptive** |
| exit | **non-preemptive** |
| running → ready (timer) | **preemptive** |
| waiting → ready (a better job is now ready) | **preemptive** |

**Formulas:** `TA = finish − arrival`, `wait = TA − burst`, `response = first run − arrival`.

| policy | key facts |
|---|---|
| FCFS | non-preemptive; **convoy effect**; 24,3,3 → avg wait 17, but order 3,3,24 → 3 |
| SJF | non-preemptive; **optimal avg wait** (swap argument); needs future bursts |
| SRTN | preemptive SJF; on each arrival compare the newcomer's burst with the running job's **remaining** time; on a tie the running job keeps the CPU |
| RR | quantum q; the queue's front runs; **an arrival does not preempt** (it joins the back); waits ≤ (n−1)q; q too small → switch overhead, q too big → FCFS |
| Priority | ties go FCFS; **starvation → fixed by aging**; SJF = priority by predicted burst |
| Lottery | P(win) = t/T; fair on average, not short-term; no permanent starvation if tickets > 0 |
| Stride | stride = W/tickets; run the **smallest pass**, then pass += stride; deterministic |

- **Prediction:** `τₙ₊₁ = α·tₙ + (1−α)·τₙ`. With α = ½ it closes half the gap each step. It **lags** when behaviour suddenly changes.
- **Starvation:** FCFS no, RR no, SJF/SRTN **yes**, priority **yes**, lottery no.
- **Conflicts:** RR wins on response time, SJF/SRTN win on turnaround. Minimizing the average is not the same as minimizing the worst case.
- **SRTN/RR procedure:** make a table with one row per decision point (each arrival, each finish, and each quantum end for RR) and track the remaining time per job. Only the running job's number goes down.

Benchmark (A 0/10, B 5/3, C 6/5, D 7/1), with avg wait in each row:
FCFS 5.75, SJF 4, SRTN **3.25**, RR q=4 6.75.

---

## 4. L06: base/limit, binding, fits, fragmentation ⏱ 4 min

- **Base + limit (physical):** an address is legal if `base ≤ addr < base + limit`. That is **2 comparisons**, and a failure traps to the OS.
- **Relocation + limit (logical):** `if la ≥ limit trap; pa = la + reloc`. That is **1 comparison + 1 addition**.
  - Loading these registers is **privileged**. They are saved and restored from the PCB on each switch.
  - **Base changes when the process moves; limit changes when it grows.**
- **Binding time:**

| binding | example | can move after it starts? |
|---|---|---|
| compile | xv6 kernel at 0x80000000 | no |
| load | loader patches addresses once | no |
| **execution** | MMU translates every access | **yes** (needed for swapping, compaction, paging) |

- **Fits** (holes 100, 500, 200, 300, 600; requests 212, 417, 112, 426):

| policy | picks | result |
|---|---|---|
| first fit | first hole that's big enough | 426 waits |
| best fit | smallest hole that's big enough | **all four placed** |
| worst fit | largest hole | 426 waits |
| next fit | first fit, starting where the last search ended | – |

  - **Procedure:** the hole shrinks by the request size. Recheck the whole hole list at every request.
- **External fragmentation:** enough free memory in total, but no single hole is big enough. It comes from contiguous variable-size allocation.
  - **50% rule:** about ⅓ of memory is lost.
  - Fixes: compaction (needs execution-time binding, and is costly) or paging.
- **Internal fragmentation:** waste *inside* an allocation, from rounding up to a fixed unit (pages, 512-B blocks).
  - Example: 512-B units, requests 1300, 512, 2049 → 236 + 0 + 511 = **747** wasted.
- **When each matters:** external fragmentation matters with contiguous/segmented allocation. Internal fragmentation matters with fixed units or paging (at most P−1 bytes per allocation).

---

## 5. L07: Paging, address bits, TLB, multi-level ⏱ 6 min

`logical = [ page# | offset ]`, `offset bits = log₂(page size)`, `PA = frame × pagesize + offset`. The offset is copied unchanged.

**Bit recipe:**
- page# bits = log₂(**number of pages**), or address bits − offset bits
- frame# bits = log₂(**number of frames**)
- Then add the offset **once**.

Prof's 256 pages × 4 KB on 64 frames → **20 / 18**.

**Hex shortcut:**
- 4-bit offset = the last hex digit
- 8-bit offset = the last 2 digits
- 12-bit offset = the last 3 digits

Example: 0x2A1 with 256-B pages and page 2 → frame A gives **0xAA1**. On a fault, use the next free frame (e.g. 9), so 0x4E6 → **0x9E6**.

**PTE bits:** valid, R/W/X/U protection, present, dirty, accessed/reference.

**Two problems with paging, and the fixes:**
1. **Too slow** (an extra memory access for every load/store) → fixed by the **TLB**, a hardware cache in the MMU that works because of **locality**.
   - Without TLB EAT = (k+1)·t_m.
   - With a TLB: `EAT = [t_TLB] + h·t_m + (1−h)(k·t_m + t_m)`
   - Examples: 100 ns, 2 levels → h = .98 gives 104, h = .80 gives 140.
2. **Too big:** 32-bit addresses, 4 KB pages, 4 B PTEs → 2²⁰ × 4 B = **4 MB per process** (multiply by the PTE size, NOT the page size) → fixed by **multi-level** tables.

**Multi-level split:**
- index bits per level = **log₂(page size / PTE size)**
- The **short leftover field goes on TOP**, so every lower table is exactly one full page.

| machine | split |
|---|---|
| 32-bit, 4 KB, 4 B PTE | **10 \| 10 \| 12** |
| 39-bit, 4 KB, **4 B** PTE (prof's review) | 27 page bits → **7 \| 10 \| 10 \| 12** |
| Sv39 (xv6), 4 KB, **8 B** PTE | 512/page → **9 \| 9 \| 9 \| 12** |
| 32-bit, 8 KB, 4 B PTE | **8 \| 11 \| 13** |

Why multi-level saves memory: unused regions get **no lower-level table**. The cost is k memory accesses per walk.

**TLB on a context switch:** flush it, or tag entries with an **ASID**. xv6 switches `satp` and flushes **twice on every trap** (once in, once out).

**Inverted page table:** one entry per **frame**. It is small, but every lookup must **search** it (fixed with a hash + TLB).

**Page size trade-off:** small pages mean less internal fragmentation, but bigger tables and more TLB misses.

---

## 6. L08: Virtual memory ⏱ 6 min

**The key property is locality.** It belongs to the **application**: with no locality, VM performs badly. RAM acts as a cache for the disk, and a page fault is a cache miss.

**Page fault steps:**
1. Trap; check that the address is legal (if not, **kill** the process).
2. Find the page on disk.
3. Get a free frame, **evicting** one if needed. A **dirty** victim must be written back first (2 disk operations).
4. Start the disk read; the process **sleeps** and another one runs.
5. Update the PTE.
6. **Restart** the instruction.

**Lazy, demand and COW:**
- **Pure demand paging** starts with zero pages in memory.
- **xv6** does only lazy allocation (sbrk raises `p->sz`, and `vmfault` maps a zeroed page if `va < p->sz`). It has no swap and no COW.
- **COW fork:** copy only the page table, share the frames **read-only**, and on a write fault copy that one page. With fork + exec almost nothing gets copied.

**EAT with faults:** `EAT = (1−p)·t_m + p·t_f`. With 200 ns and 8 ms, p = 1/1000 gives **≈ 8.2 µs (≈ 40× slower)**. Staying within 10% (220 ns) needs p < **2.5 × 10⁻⁶**.

**Replacement:**

| policy | evicts | notes |
|---|---|---|
| FIFO | oldest arrival | hits don't reorder; **Belady's anomaly** |
| OPT | farthest next use (or never used again) | benchmark; needs the future |
| LRU | least recently used | approximates OPT; too costly exactly → **clock** |

- **Belady:** 1,2,3,4,1,2,5,1,2,3,4,5 under FIFO gives **9 faults with 3 frames and 10 with 4**. LRU and OPT can't do this (the **stack property**: the N-frame set ⊆ the N+1-frame set). So an algorithm with Belady's anomaly **cannot be optimal**.
- **Prof's string** 7,2,3,1,2,5,3,4,6,7,7,1,0,5,4,6,2,3,0,1 with 3 frames: **LRU 18, FIFO 17, OPT 13**. LRU isn't always better than FIFO.
- **Clock:**
  - bit 1 → clear it and move on.
  - bit 0 → this is the victim; the new page gets bit 1 and the hand moves past it.
  - all bits 1 → it behaves exactly like FIFO.
  - **Enhanced clock** prefers (ref 0, **clean**) over (ref 0, dirty).

**Thrashing:**
- **Cause:** Σ working sets > RAM. Processes keep evicting each other's pages, the disk is constantly busy, and the CPU sits idle.
- **Detect:** **CPU utilization low + paging disk busy / fault rate high.**
- **Fix:** **lower the degree of multiprogramming** (suspend or swap out processes, OOM killer), or add RAM. The trap is that the OS sees low CPU use and admits *more* processes.

**Prof's question: CPU 20%, paging disk 97.7%, other I/O 5%** (this is thrashing):

| option | helps? | why |
|---|---|---|
| a. faster CPU | **No** | the CPU isn't the bottleneck |
| b. bigger paging disk | **No** | the problem is that the disk is busy, not that it lacks space |
| c. more multiprogramming | **No, worse** | less memory per process |
| d. less multiprogramming | **Yes** | each process gets more frames |
| e. more RAM | **Yes** | working sets fit, which shifts the utilization curve right |
| f. bigger pages | **Probably not** | fewer frames, more internal fragmentation; doesn't attack the cause |

---

## Rapid-fire self-test (cover the answers)

1. Logical space of 64 pages × 1024 words on 32 frames: bits in the logical and physical address?
2. 2-level page table, 50 ns memory, 2 ns TLB, 75% hit rate: EAT?
3. A child calls `exec("/bin/nope")` and it fails. What happens if there is no `exit(1)` after it?
4. A timer interrupt arrives while the user is in a tight loop. What's the cause type, is it sync or async, and what may the kernel do?
5. Is an orphan the same as a zombie?
6. With RR, does a new arrival preempt the running job?
7. 39-bit VA, 4 KB pages, 4 B PTE: what's the split?
8. Can an algorithm with Belady's anomaly be optimal?
9. `static int calls;` inside main: which segment?
10. A process changes its working set. Does its fault rate go up or down?

### Answers

1. **64 pages × 1024 words, 32 frames:** the logical address is 6 + 10 = **16 bits** (64 = 2⁶ pages, 1024 = 2¹⁰ offset). The physical address is 5 + 10 = **15 bits** (32 = 2⁵ frames). Keep everything in words and don't convert to bytes.

2. **2-level table, 50 ns memory, 2 ns TLB, 75% hit rate:** EAT = 2 + 0.75 × 50 + 0.25 × (2 × 50 + 50) = 2 + 37.5 + 37.5 = **77 ns**. The TLB's 2 ns is paid on every access, and a hit still reads memory once.

3. **Failed exec with no `exit(1)` after it:** exec returns −1, so the child falls through into the rest of the shell code. You now have **two shells** running the loop, which gives duplicate prompts while the parent waits on a child that never finishes. `exit(1)` makes the failed child die and report failure through `wait`.

4. **Timer during a tight loop:** this is an **interrupt**, and it is **asynchronous** (scause 5, top bit set). The kernel may **yield**, marking the process RUNNABLE and switching to another runnable process. The loop resumes on the interrupted instruction later.

5. **Orphan vs zombie: they are not the same.**
   - A **zombie** has already exited, but its parent hasn't called `wait` yet. Only its slot, PID and exit status remain (plus its memory, in xv6).
   - An **orphan** is still running, but its parent exited first. It is re-parented to **init**, which waits on it.

6. **RR and a new arrival: no preemption.** The newcomer joins the back of the ready queue. Only the end of a quantum (or the job blocking or exiting) switches the CPU.

7. **39-bit VA, 4 KB pages, 4 B PTEs:** the split is **7 | 10 | 10 | 12**.
   - The offset is 12 bits, because 4 KB = 2¹².
   - That leaves 39 − 12 = 27 page-number bits.
   - One page holds 4096 / 4 = 1024 PTEs = 2¹⁰, so each level gets 10 bits.
   - 27 = 10 + 10 + 7, and the short 7-bit field goes on top so every lower-level table is a full page.

8. **Belady's anomaly and optimality: no.** OPT has the stack property: the pages it holds with N frames are always a subset of those it holds with N + 1. So more frames can never cause more faults under OPT. An algorithm that shows the anomaly therefore can't be OPT.

9. **`static int calls;` inside `main`:** it goes in **bss**, because it is a static, uninitialized variable and lives for the whole run. It is not on the stack, even though it's declared inside a function.

10. **Working set changes:** the fault rate **goes up**, because the new pages aren't in memory yet. It drops again once they're loaded if the new set fits. If it doesn't fit, the rate stays high and the process can start thrashing.

**Last 2 minutes before you walk in:** reread section 0.
