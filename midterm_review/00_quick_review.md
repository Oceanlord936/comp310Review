# COMP 310 Midterm 1 — Quick Cover-All Review (Lectures 1–8)

Test: **Oct 20, 2026**. Scope: L01–L08 inclusive (confirmed in class11.md).
Sources: this year's slides N01–N08, prof's OS-Notes-L01-L08.pdf (v0.1, Oct 6), your class1–12 notes,
mid1RE (2022 midterm + "Questions from the Labs").

How to use: one pass, top to bottom. ★ = very likely on the exam (old midterm topic, lab question,
or slide EXP problem). If a line feels unfamiliar, that is the spot to reopen your detailed notes in
`review_understand/`.

---

## L01 — What an Operating System Is

- OS = **resource manager**. First job: **allocation** (who gets CPU / memory / disk, and when).
  Working definition: **the kernel**, the program that runs all the time.
- ★ Managing a resource means being able to **take it back**. Taking the CPU back from a program that
  won't yield needs hardware help: the **timer interrupt**.
- Toolchain (preprocessor, compiler, assembler, linker) = ordinary programs. An executable is just a file.
- Hardware: CPU + device controllers on a shared bus. Each controller is a small computer with its own
  buffer and its own idea of "done". **Speed mismatch**: CPU is vastly faster than devices, so the CPU
  must not wait → devices **interrupt** the CPU (→ L02).
- Storage hierarchy: fast, small, volatile at the top (registers, cache, RAM); slow, big, persistent at the
  bottom (SSD, disk). ★ Two properties a designer cares about: **speed/cost/size trade-off** and
  **volatility** (top loses data on power-off). Moving data between levels is the OS's job.
- ★ **Dual mode**: a **mode bit** (user / kernel). Boot starts in kernel mode; the OS then enables user mode.
  A syscall traps in, flips the bit, runs, flips it back.

  ★ The four scenarios (old midterm Q14):
  | | user mode | kernel mode |
  |---|---|---|
  | normal instruction | proceeds | proceeds |
  | privileged instruction | **traps** to the OS | proceeds |

  Privileged: I/O control, timer management, interrupt management (and loading base/limit, satp).
- **Ruling while absent** — the kernel runs ~1 in 10,000 instructions but stays in charge through three
  things it configures and the hardware enforces: **mode bit** (what's allowed), **page tables** (what's
  reachable), **timer** (return ticket: control comes back within one tick).
- OS services: UI (CLI / GUI / touch), program execution, I/O, file system, communication, error detection,
  resource allocation, logging, protection & security.
- No mode bit → no OS, "only a library" (any program could run privileged instructions).

> **Slide problems L01** (solve in your typed file)
> - **EXP 1.1** — One program: A = reading "hello" from the keyboard, B = loading the program into memory,
>   C = executing and printing to the screen. (1) Role of the OS in A? Which operations run on the CPU, and
>   who wrote most of them? (2) Why is the disk not used in A? (3) Is the CPU involved in initiating B? If so,
>   what does it run to set it up? (4) Why is the disk not used in C — could another program be using it
>   then? (5) Could `hello` itself use the disk, under some condition?
> - **EXP 1.2** — Designing a smartphone OS: which desktop services would you remove, and what breaks? How
>   do you keep interactive response times acceptable? Unique challenges of touch, seen from the OS side?

## L02 — Traps: System Calls, Interrupts, Faults

- ★ **One mechanism, three causes**:
  | | cause | sync? | resumes |
  |---|---|---|---|
  | system call | deliberate (`ecall`) | synchronous | **after** the instruction |
  | interrupt | external (timer, device) | asynchronous | **on** the interrupted instruction (it never ran) |
  | exception/fault | accidental (bad address, illegal instr.) | synchronous | re-run it, or kill |
- What every trap does (hardware): save PC in **sepc**, record cause in **scause** (+ **stval** = extra info,
  e.g. faulting address), switch to supervisor mode, jump to the fixed entry in **stvec**.
- ★ scause values: **8** = ecall from user · **12** instruction page fault · **13** load page fault ·
  **15** store page fault · **2** illegal instruction · interrupts (top bit set): **5** supervisor timer,
  **9** external device.
- ★ How the OS knows why it trapped: **scause**. `usertrap()` dispatches on it:
  ```c
  if (r_scause() == 8) {            // syscall
      p->trapframe->epc += 4;       // return AFTER the ecall
      intr_on(); syscall();
  } else if ((which_dev = devintr()) != 0) {     // timer / device
  } else if ((r_scause()==15 || r_scause()==13) && vmfault(...) != 0) {  // lazy page
  } else { setkilled(p); }          // fatal fault
  if (which_dev == 2) yield();      // timer → give up the CPU
  ```
- Syscalls = the service interface. **Six families**: process control, file mgmt, device mgmt, information
  maintenance, communication, protection. xv6 has **22**.
- ★ `printf` (library) ≠ `write` (syscall). xv6's printf calls `write` **once per character** →
  `printf("hello\n")` = 6 syscalls (a buffering libc makes 1). That's why `write` dominates the histogram.
- ★ **Call by number, not by name/address**: the user can't know (or be trusted with) kernel addresses, and
  the kernel must validate the request. A number indexes a kernel-owned table (`syscalls[num]`), easy to
  bounds-check, fixed ABI. Arguments go in registers `a0–a5`, number in `a7`, result back in `a0`.
  Big arguments → pass a pointer, kernel copies with `copyin/copyout`.
- Interrupts: device signals completion. **Interrupt-driven I/O**: CPU interrupted per unit of data.
  **DMA**: controller moves the whole block to memory, **one** interrupt at the end → frees the CPU.
- ★ Why interrupts matter to the OS itself: they're how the kernel **regains control** (timer → preemption)
  and learns about I/O completion without **polling**.
- ★ Why I/O must be asynchronous: devices are slow; if the CPU waited, it would sit idle. Start the I/O,
  run something else, get interrupted when done.
- Disabling interrupts is privileged: a user program that disabled them could never be preempted
  (monopolize the CPU) and devices would go unserviced.
- ★ A page fault isn't always an error: **lazy allocation**, **copy-on-write**, **demand paging**, **stack
  growth** are legit; a genuinely illegal access is fatal.
- **Trapframe** (per process, saved user registers, one page) ≠ **trampoline** (one shared code page at the
  same VA in every page table + kernel's, holds uservec/userret). ★ Saving the trapframe happens on
  **every** trap; switching to another process is a **separate, conditional** decision.

> **Slide problems L02**
> - **EXP 2.1** — A `cp` written with `open`/`read`/`write`/`exit` never checks the return value of `open()`:
>   ```c
>   int sfd = open(argv[1], O_RDONLY);
>   int dfd = open(argv[2], O_CREAT|O_WRONLY);
>   while ((n = read(sfd, buf, sizeof(buf))) > 0)
>     if (write(dfd, buf, n) != n) { (void)write(2, msg, sizeof(msg)); exit(EXIT_FAILURE); }
>   ```
>   If an `open()` fails, does the program crash or keep running? If it fails, where exactly, and why there
>   rather than at the `open()`? Advantage of catching it at the `open()` and exiting immediately?
> - **EXP 2.2** — Why must creating a file (`open`/`create`) go through the kernel? Why is using the GPU for
>   a computation *not* a system call — downside of making it one? Where is the code that actually performs
>   a `read()`? Why can't an application reimplement it and skip the call — would that be faster?
> - **EXP 2.3** — 1 MB file = 1024 blocks, disk → memory. **Interrupt-driven:** per block, controller takes
>   300 µs to fetch into its buffer (CPU free meanwhile), then the CPU spends 50 µs copying it to memory.
>   **DMA:** one operation, 500 µs CPU setup, transfer 1500 µs with no CPU. For each: total transfer time,
>   and CPU time spent on transfer work?
> - **EXP 2.4** — A simple OS lets applications disable interrupts. What breaks (beyond the kernel never
>   regaining control at the next timer)? Does it hurt the offender itself or only others? Any legitimate
>   reason for the *kernel* to disable interrupts, and for how long?
> - **EXP 2.5** — Classify (syscall / interrupt / exception; sync or async): `write()`; dereferencing a null
>   pointer (what's in stval?); the timer firing in a tight loop (what may the kernel do next?); jumping into
>   code not yet paged in (has anything gone wrong?). Which can occur at an instruction boundary the program
>   did not choose, and why is that the hardest to handle?

## L03 — Processes

- ★ **Uniprogramming** = one program in memory, runs to completion; CPU idles during its I/O.
  **Multiprogramming** = several in memory; when one waits for I/O, another runs → higher utilization.
  (Worked ex.: 3 s CPU + 2 s I/O, two jobs: uni = 10 s, CPU busy 6/10 = 60%; multi overlaps them.)
- Program = passive file; **process = program in execution** (+ PC, registers, memory, open files…).
- ★ Memory segments (old midterm Q1): **text** (code) · **data** (initialized globals/static) · **bss**
  (uninitialized / zero globals & statics) · **heap** (malloc, grows up) · **stack** (locals, args, return
  addresses, grows down). Rules: `int a;` global → bss · `int b = 10;` → data · locals → stack frame of
  their function · `static` local → bss/data · `malloc`'d block → heap (the pointer itself is on the stack) ·
  string literals → read-only, with text.
- CPU-bound (long bursts) vs I/O-bound (short bursts, waits a lot). Multiprogramming pays because they overlap.
- ★ States: New → Ready → Running → (Waiting) → Terminated.
  Running→Ready (preempted/timer), Running→Waiting (I/O or event), Waiting→Ready (event done).
  **xv6 has six**: UNUSED, USED, SLEEPING, RUNNABLE, RUNNING, ZOMBIE.
  ★ Old Q2: `int x;` → running · `scanf` → running, then **waiting/sleeping** for input, then ready ·
  `process(x)` → running (or ready if preempted). Any line can be "ready" if the timer preempts it.
- ★ A process can't be RUNNING and RUNNABLE at once. Ready queue = all RUNNABLE (same state value).
- **PCB** = everything the kernel must remember to resume: state, PID, registers/PC, scheduling info, memory
  (page table), open files, parent… ★ Nothing survives a context switch unless it's in the PCB.
- ★ xv6: `struct proc proc[NPROC]`, **fixed array of 64**, found by **scanning**. Cheap and simple when N is
  small (assumption: few processes); Linux uses a linked list of `task_struct` (scales, more complex).
  65th process → `fork` fails.
- Queues: ready queue + device/event queues. xv6 has no real ready queue: the scheduler scans the table
  for RUNNABLE.
- ★ **Context switch** in xv6: timer trap saves user regs in the trapframe → `usertrap` → `yield` (mark
  RUNNABLE) → `sched` → `swtch` saves **14 callee-saved registers** (ra, sp, s0–s11) of the old kernel
  thread and loads the new one's. PC is covered by `ra`. Switch cost = pure overhead.
- ★ **Memory is NOT saved on a context switch** (old Q15): each process has its own address space, so
  nothing the next process does can touch it. Only the address-space pointer (satp/page table) changes.

> **Slide problems L03** (the slides mislabel these "EXP 5.x"; they're Lecture 3's)
> - **EXP 3.1** — Uniprogrammed machine, app with 1 s CPU bursts and 1 s I/O bursts: CPU utilization? Two
>   instances under multiprogramming: utilization? App has 3 CPU bursts and 2 I/O bursts: when do both
>   instances complete, under uni- and multiprogramming (assume perfect interleaving)?
> - **EXP 3.2** — For a short C program, place every variable in text / initialized data / BSS / heap / stack.
>   With a function that calls another: where do the arguments and return address live? What problems must
>   be solved before two programs can share a machine? Do we need the process concept on a one-program system?
> - **EXP 3.3** — Why would a CPU-bound process occasionally use I/O? Why would an I/O-bound process
>   periodically need the CPU? What does multiprogramming buy if every process is CPU-bound?
> - **EXP 3.4** — Most likely state: waiting for the user to type; training a neural network; waiting for
>   disk data. A program prints "Hello, World" the instant it launches, then calls `read()` on a file — state
>   at each instant?
> - **EXP 3.5** — If memory isn't saved on a context switch, we trust the next process not to touch ours: what
>   could go wrong? With nobody malicious, what else? With isolation, does the problem exist? How many
>   processes does one PCB represent, and what is its information used for?

## L04 — Process Creation and Control

- ★ `fork()` returns twice: **child gets 0**, **parent gets the child's PID** (it needs it to `wait` for that
  child; the child can always call `getpid`/its parent is known).
- ★ Counting: n sequential forks → **2ⁿ** processes. Nested (second fork only in child) → 3.
  Variable set before fork, changed in child → parent still sees the old value (separate copies).
- ★ Old Q12 — fork copies: **data, stack, and the file-descriptor table** for sure. **Text** is the judgment
  call: it's never written, so real Unix **shares** it read-only rather than copying; xv6's `uvmcopy` does
  copy every page, text included. On the exam: circle b, c, d, and add one line saying text is shared
  (copied in xv6). Linux makes all of it copy-on-write.
  Old Q13 — exec "copies": **none of the above (e)**. exec **replaces** text/data/heap/stack from the file;
  the fd table **survives** (that's how redirection works) but nothing is copied.
- ★ exec keeps: PID, parent, open fds, table entry. Discards: the whole address space.
- Shell loop: `fork` → child `exec` (then `exit(1)` only if exec failed) → parent `wait`.
- ★ Redirection (old Q4, low priority this year): in the child, **between fork and exec**:
  `close(1); open("out.txt", O_WRONLY|O_CREATE);` → open returns lowest free fd = 1 → exec'd program
  writes to the file. Pipe `ls | wc -l` (old Q3): `pipe(p)`; child 1: `close(1); dup(p[1]); close both;
  exec ls`; child 2: `close(0); dup(p[0]); close both; exec wc`; parent closes both ends, waits twice.
  This is why fork and exec are separate calls.
- `exit()` → status to parent via `wait()`. Reclaim: **Linux eagerly, xv6 does not** (parent's wait frees it).
- ★ **Zombie**: child exited, parent hasn't waited → PID + exit status (a struct proc) kept. No CPU, no memory
  — but holds a PID / table slot. Leaking zombies → run out of PIDs/slots → can't start anything.
  **Orphan**: parent exited first → re-parented to **init**, which waits forever. (Or cascading termination.)
- ★ Isolation: each process its own address space → no address it can generate names another's memory.
  **Every process switch is an address-space switch** (one `satp` write in xv6) → TLB cost (L07).
  Same VA 0x1000 in two processes → different bytes (different page tables).

> **Slide problems L04**
> - **EXP 4.1** — `fork()` twice in sequence (no exec/exit): how many processes? Nested (second fork only in
>   the child's branch): how many? A variable = 5 before the fork, incremented by the child: what does the
>   parent see? Why does fork return the child's PID to the parent rather than 0 to both?
> - **EXP 4.2** — Why does an address-space switch happen with every process switch? Two processes access
>   0x1000 — same byte? (explain via address spaces, not hardware). Does a zombie still have an address
>   space? If isolation is enforced by hardware, why must the kernel be involved too?

## L05 — CPU Scheduling

- Scheduler = **policy** (who next). Dispatcher = **mechanism** (context switch, mode switch, jump);
  dispatch latency = pure overhead.
- ★ Scheduling happens at 4 moments: Running→Waiting, Running→Terminated (**non-preemptive**);
  Running→Ready, Waiting→Ready (**preemptive**).
- ★ Criteria: CPU utilization ↑, throughput ↑, turnaround ↓ (finish − arrival), waiting ↓ (time in ready
  queue = turnaround − burst), response ↓ (first run − arrival). They conflict.
- **FCFS**: FIFO, non-preemptive. **Convoy effect**: short jobs stuck behind a long one. Very order-sensitive.
- **SJF**: shortest next burst first. **Provably optimal for average waiting time** (swap argument). Needs
  the future → predict with ★ **exponential averaging**: τₙ₊₁ = α·tₙ + (1−α)·τₙ (t = actual, τ = guess;
  α=0.5 typical; α→1 = only recent, α→0 = only history).
- **SRTN** (preemptive SJF): on each arrival, compare newcomer's burst vs running job's *remaining* time.
  Can **starve** long jobs.
- **Round Robin** (what xv6 runs): quantum q, preempt on expiry, back of queue. q too big → FCFS;
  q too small → switch overhead dominates. ★ Lab: a job doesn't always use its full slice (blocks for I/O
  or yields early). Large slices → poor responsiveness.
- **Priority**: SJF is a special case. **Starvation** → fix with **aging** (raise priority while waiting).
- ★ Turnaround vs response: no ordering wins both (batch = good turnaround, RR = good response).
- **Proportional share**: state the ratio. **Lottery**: tickets, random draw each quantum, fair **on
  average**, not short-run. Mechanisms: **currency** (local tickets converted), **transfer** (give tickets
  to who you wait on), **inflation** (trusted process mints tickets).
  **Stride**: stride = W / tickets, pass starts 0, run lowest pass, pass += stride. Deterministic — same
  shares, no dice.

> **Slide problems L05**
> - **EXP 5.1** — Processes A (arrives 0, burst 10), B (5, 3), C (6, 5), D (7, 1). Gantt chart + average
>   waiting and turnaround for **FCFS, SJF, SRTN, RR q = 4**. Which wins on average waiting time, which on
>   response time — the same one? A process has bursts 6, 4, 6, 4, 13, 13, 13; with α = ½ and initial
>   prediction 10, compute each prediction. At which burst does the prediction first exceed the measurement,
>   and why there?
> - (notes) **Stride** worked example: W = 10000, tickets A 200 / B 100 / C 50 — run 7 steps.

## L06 — Memory Management

- CPU directly reaches only registers + main memory. Memory sees only address (+data) — it can't tell
  who's asking → protection must happen **in hardware before memory**, on every access.
- ★ **Base + limit** (addresses physical): legal iff `base ≤ addr < base+limit`, else trap. Loaded on every
  context switch, **privileged**. xv6 has none — its page table does this per page.
- ★ **Binding**: compile time (absolute; xv6 kernel at 0x80000000) · load time (relocatable code, loader
  patches once) · **execution time** (moved while running; needs MMU) ← enables swapping, paging, VM.
- ★ MMU: logical → physical. **Relocation + limit**: `if (la >= limit) trap; pa = la + reloc;`
  (1 comparison, 1 addition). Moving a process: code unchanged, only the base register changes.
- Dynamic loading (load a routine on first call; overlays = manual version) vs **linking** (relocation +
  resolution). Static: copied into every binary (xv6: ulib, usys, printf, umalloc). Dynamic: one shared
  copy, resolved at run time.
- Contiguous allocation → **first / best / worst / next fit**. First and best beat worst; first is fastest.
- ★ **External** fragmentation: enough free total, no single hole big enough (variable sizes). **50-percent
  rule**: first fit loses 0.5N per N → ⅓ of memory. **Internal**: rounding up inside an allocation.
  **Compaction** fixes external, needs execution-time binding, rarely done (copies lots, pauses processes,
  can't move memory under active DMA).

> **Slide problems L06**
> - **EXP 6.1** — When are addresses bound? (a) xv6 kernel linked at 0x80000000, QEMU loads it exactly
>   there. (b) A loader places a program and adjusts every address before it starts. (c) Swapped out, brought
>   back into a different region. (d) Which can move after it starts, and what does it need from hardware?
> - **EXP 6.2** — Base 32 KB (32768), bounds 2 KB (2048): translate virtual 0, 1000, 2047, 2048, 5000 (or
>   fault). Heap grows to 3000 bytes: which register changes, to what? OS moves it to physical 48 KB: what
>   changes in the code, and in the registers?
> - **EXP 6.3** — Base 300040, limit 120900, addresses physical: `load #305000` allowed? (two comparisons).
>   `store #300030`: what happens, who handles it? Relocation 300040 / limit 120900, addresses logical: what
>   logical address reaches the same byte; count comparisons and additions. On a switch A → B, what must the
>   OS do to these registers, and why must that be privileged?
> - **EXP 6.4** — Holes 100K, 500K, 200K, 300K, 600K (in order); requests 212K, 417K, 112K, 426K. Placement
>   under first / best / worst fit; which places all four; leftover holes and their sum.
> - **EXP 6.5** — 512-byte allocation units, requests 1300, 512, 2049: bytes lost to internal fragmentation?
>   Show the 50-percent rule means ⅓ of memory lost. Compaction: what must be true about binding, and why is
>   it rarely done?

## L07 — Paging and Address Translation

- ★ Fixed-size pages/frames → **no external fragmentation** (any frame fits any page); cost = **internal**,
  < 1 page per allocation (max P−1 bytes).
- ★ Address = **[page number | offset]**. Only the page number is looked up; **offset passes through**.
  `PA = frame × pagesize + offset` = `(PFN << offset_bits) | offset`. Offset bits = log₂(page size).
  xv6: 4 KB pages → 12-bit offset.
- PTE bits: **valid**, protection (R/W/X, U), **present** (L08), **dirty**, **accessed/reference**.
- Two problems: **too slow** (extra memory access per load) → **TLB**; **too big** (32-bit, 4 KB, 4-byte PTE →
  2²⁰ × 4 B = **4 MB per process**) → **multi-level tables**.
- ★ **TLB** caches VPN→PFN. Hit: no table walk. Miss: walk, fill. Value = hit rate, thanks to locality
  (spatial: first touch of each page misses; temporal: rerun all hit).
  **EAT = h·t_m + (1−h)·(k·t_m + t_m)** for a k-level table (TLB lookup free).
  10 ns, 3 levels, h=0.9 → 13 ns (vs 40 ns without).
- ★ TLB on context switch: entries belong to one address space → **flush** or tag with **ASID**.
  xv6: no ASIDs, separate kernel page table → **every trap** switches satp + `sfence.vma`, in and out
  (2 table switches per syscall). Linux maps the kernel into every process.
- ★ **Hierarchical**: the table must itself be paged → page directory. 32-bit/4 KB/4-byte PTE →
  **10 | 10 | 12**. Index bits per level = log₂(page size / PTE size). Short leftover field goes **on top**
  (only one top table). Unused regions → no inner table → saves memory. k levels = k memory accesses per
  walk.
- ★ **Sv39** (xv6): 39-bit VA, 8-byte PTEs, 512 per 4 KB page → **9 | 9 | 9 | 12**, 3 levels; satp picks it.
- Hashed (hash VPN → chain) and **inverted** (one entry per physical frame, (pid, p); small table, slow
  search). Protection is per page (valid bit). **Sharing**: map one frame into many tables (reentrant code,
  libc; xv6 shares only the trampoline).

> **Slide problems L07**
> - **EXP 7.1** — 256-byte pages; A, B, C need 1030, 750, 510 bytes: pages each, total internal
>   fragmentation? 1024-byte pages: requests of 10 and 1500 bytes — pages and waste each? With page size P,
>   max waste per allocation? What argues for small pages, and against?
> - **EXP 7.2** — Tiny machine: 64-byte space, 16-byte pages, table 0→3, 1→7, 2→5, 3→2. Translate 0, 21, 40,
>   63. Physical memory 128 bytes: bits in VPN, offset, frame number? Hex example (page 4 → frame 0x66, 4-bit
>   offset): translate 0x45 and 0x4F — which bits changed?
> - **EXP 7.3** — Memory 100 ns, 2-level table, free TLB lookup: EAT at hit rates 0.98 and 0.80. Redo the
>   ten-integer loop (array at VA 100, TLB empty) with 32-byte pages: hits/misses, hit rate. During one system
>   call, how many times does xv6 change page tables, and what happens to the TLB each time?
> - **EXP 7.4** — 32-bit, 4 KB pages, 4-byte entries: one linear page table size; 100 processes? 32-bit, 8 KB
>   pages, 4-byte entries: offset bits, entries per table page, levels needed? (20-address drawing: translate
>   logical 17 — how many memory accesses? needs the slide figure.)
> - (class9) 39-bit addresses, 4 KB pages, 4-byte PTEs: bit split, and why the short field goes on top.
>   Sv39 (8-byte PTEs): why 9 | 9 | 9 | 12?

## L08 — Virtual Memory

- ★ Only part of a program needs to be in memory → logical space ≫ physical. Works because of **locality**.
  Each page: in memory · in backing store · not allocated.
- ★ **Demand paging**: load a page on first touch. Entry not present → **page fault** → OS finds page on
  disk, gets a free frame (evict if none), reads it in (process **sleeps**, another runs), marks valid,
  **restarts the instruction**. Pure demand paging = start with zero pages.
- Present bit (OSTEP). RISC-V has only V: when V=0 the OS can store the disk address in the PTE; it tells
  "on disk" from "invalid" by its own records.
- Restarting is hard with auto-increment / overlapping block moves → hardware must undo or pre-check.
  RISC-V loads/stores: precise faults, return to sepc and re-run.
- Free-frame list; **zero-fill-on-demand** (security: don't leak the previous owner's data).
- ★ xv6: **lazy (demand-zero) only** — `sbrklazy` raises `p->sz`, first touch → scause 13/15 → `vmfault`
  maps a zeroed page if `va < p->sz` and unmapped; else kill. **No swap, never evicts.** fork copies
  eagerly (no COW).
- ★ **EAT = (1−p)·t_m + p·t_f**. 200 ns, 8 ms, p=1/1000 → 8.2 µs ≈ **40× slower**. Within 10%:
  p < 2.5×10⁻⁶ (≈1 in 400,000).
- ★ **Copy-on-write**: fork maps the same frames read-only in both; first write traps and copies that one
  page. Code pages shared forever. (xv6 doesn't; Linux does.)
- Replacement: no free frame → evict a victim (write back if **dirty**, drop if clean → prefer clean).
  Watermarks LW/HW + **page daemon** evicts in batches.
- ★ Policies on `7,0,1,2,0,3,0,4,2,3,0,3,2,1,2,0,1,7,0,1`, 3 frames: **FIFO 15 · OPT 9 · LRU 12**.
  **Belady's anomaly** (FIFO only): `1,2,3,4,1,2,5,1,2,3,4,5` → 9 faults with 3 frames, **10** with 4.
  LRU/OPT have the **stack property** (N frames' contents ⊂ N+1 frames') → no anomaly.
- LRU exact = counters (scan to find min) or stack (move to top on every reference) — too costly →
  **Clock / second chance**: reference bit; hand at 1 → clear, move on; at 0 → victim. All 1s → FIFO.
  Enhanced: prefer (ref 0, clean). RISC-V A = reference bit, D = dirty (xv6 never reads them).
- LFU (counts, shifted right to age). Prefetching (P → P+1).
- ★ **Thrashing**: Σ working sets > memory → constant paging, CPU utilization collapses as more processes
  are added. Fix: **admission control** (fewer processes) or OOM killer.

> **Slide problems L08**
> - **EXP 8.1** — Memory 200 ns, fault 8 ms: EAT when 1 in 1000 accesses faults; what fault rate keeps the
>   slowdown within 10%? OSTEP numbers (memory 100 ns, disk 10 ms, AMAT = T_M + P_miss·T_D): AMAT at hit rates
>   90% and 99.9%. Fault = 8 ms with a free frame or clean victim, 20 ms when the victim is dirty (70% of the
>   time), memory 100 ns: what fault rate keeps EAT ≤ 200 ns?
> - **EXP 8.2** — `7,0,1,2,0,3,0,4,2,3,0,3,2,1,2,0,1,7,0,1`, 3 frames: trace FIFO, OPT, LRU (check vs 15 / 9 /
>   12). FIFO on `1,2,3,4,1,2,5,1,2,3,4,5` with 3 then 4 frames; LRU with 3 then 4. Why can LRU never show
>   Belady's anomaly?
> - **EXP 8.3** — Cache of 3 pages, `0,1,2,0,1,3,0,3,1,2,1`: trace OPT, FIFO, LRU; hits, misses, hit rate.
>   Recompute ignoring compulsory misses.
> - **EXP 8.4** — Clock: A, B, C, D with bits 1, 0, 1, 1, hand at A: who's evicted, bits after (new page bit 1),
>   where does the hand stop? All bits 1, hand at A: who's evicted, which policy did it act like? B and C both
>   bit 0, B dirty, C clean: which goes, why?

---

## Lab questions to be ready for (prof's list, mid1RE)

Idle machine: RUNNING/RUNNABLE/SLEEPING counts · three trap types and which is independent of process
count (timer) · how the OS knows the trap reason (scause) · the trap-dispatch code · trap that lands in
both user and kernel space (timer interrupt — it fires wherever the CPU is) · CPU utilization estimate
from frozen traps · trapframe contents and what if it can't be restored · lottery/priority/RR handout code
correctness · large time slices · does an app use its whole slice · why syscall numbers · add a syscall in
<5 sentences · why the in-kernel load ring (can't measure an average at one instant) · is xv6
multi-threaded (multiple CPUs/harts run kernel code concurrently → locks everywhere) · xv6's PCB array:
good/bad, assumption · trampoline vs trapframe · RUNNING and RUNNABLE at once · ready-queue state values ·
the "quiet" app in T-Lab-01.

These are all in the simulation test, with answers checked against the prof's notes after you attempt them.
