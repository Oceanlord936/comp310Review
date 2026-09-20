# COMP 310 — Review Plan

Scope: Lectures 1–5 (focus on 4 and 5, stopping exactly where class stopped), the T02 sysinfo-syscall lab, and GINI orientation/practice. Built from the T02 PDF, your t2note.md, Lectures 1–5, Notes/class1–6.md + tutorial2.md, the A1 T01 orientation/address-space lab, C-Lab 01, and the six GINI_Program utility sources.

## How to use this file
- Work top to bottom — sections are small on purpose. You don't have to do them in one sitting, or even in one week.
- When you're ready for one, just say which (e.g. "let's do 2.3" or "next section"). I'll explain that piece in chat; you type your own notes into a separate md in this folder. I won't write your notes for you.
- Code is typed by hand by you, not generated for you. I'll explain what code does and why; you type it and ask questions as you go.
- Check boxes as you finish a section — `[ ]` → `[x]`. This file is the save point: if you stop mid-section, just tell me where you left off next time.
- **(GINI-specific)** tags mark things that diverge from general Unix/Linux — don't assume they generalize.
- **(future)** tags mark things beyond what's been covered in class yet — skip for now, revisit later.

---

## Corrections to fold into your notes

Found while reading your files against the official material. Worth fixing before they calcify:

1. **"xvt simulation" isn't a thing, and the framing is backwards.** GINI runs a real, unmodified xv6 kernel on **QEMU** (an emulator, explicitly *not* a VM) emulating real RISC-V hardware. Nothing is simulated. (Part 0)
2. **Trampoline doesn't "grow"** (tutorial2.md). It's one fixed page, mapped at the *identical* virtual address in every process's page table *and* the kernel's — that fixed identity is the whole point. (Part 4)
3. **class1.md**: "modern CPU can assign and execute different processes at the same clock" conflates two unrelated ideas. The lecture's point is just that CPU clock speed vastly outpaces device speed (motivating interrupts) — nothing about running multiple processes at once (that's multicore, not covered here). (Part 1)
4. **class3.md**: "argc, argn" at the top of the stack diagram → should be **argc, argv**. (Part 3)
5. **class6.md**: your first `fork()` if/else annotation has parent/child backwards. Your *second* example in the same file has it right: `if(fork())` (nonzero return) = **parent**, `else` (zero return) = **child**. (Part 5)
6. **Guard page isn't "no permissions at all"** (Tlab1_NOTE.md — self-contradicted). It has Read+Write set, just missing the **U (user)** bit. A user-mode touch faults because it's a privilege violation, not a lack of any permission. (Part 4)
7. **"Linux has no guard page" is wrong** (Tlab1_NOTE.md — self-contradicted one paragraph later). Linux has guard pages too. The real xv6-vs-Linux difference: xv6's guard sits in one fixed spot forever (stack never grows); Linux's guard *moves* as its stack grows. (Part 4)
8. **Trapframe save ≠ process switch** (Tlab1_NOTE.md — self-corrected later, confirmed by CLab1_Traps_Student.pdf). A trapframe is saved on **every** user-mode trap. Switching to a *different* process afterward is a separate, conditional decision the scheduler sometimes makes. (Part 2 / Part 4)
9. **Heap growth direction, phrasing to fix**: heap grows up **away** from the stack, toward the trapframe — not "toward" the stack. No meeting-in-the-middle in xv6; the stack is frozen at one page forever. (Part 4)

---

## Part 0 — GINI Orientation (environment, not theory)
*(You already did T01 for this — this locks the mental model in before T02.)*

- [ ] 0.1 What GINI actually is: real xv6 kernel + QEMU emulating RISC-V, inside a container, on your laptop — the "four computers" picture, and why "simulation" is the wrong word
- [ ] 0.2 Vocabulary: IMAGE, CONTAINER, QEMU, HART, THE AGENT, A FACE
- [ ] 0.3 How a click reaches the kernel: gBuilder → Agent → shell; panels are *polled* ~1×/sec without pausing the kernel (except "Step a trap," which does pause it)
- [ ] 0.4 The rule that bites: REBOOT vs REBUILD vs STOP+RUN vs QUIT — what each keeps/destroys, and why "I edited the kernel and rebooted but nothing changed" is the #1 mistake
- [ ] 0.5 The faces/tabs you'll use: System Calls, Traps & Interrupts, Process Scheduler, Virtual Memory, Locks, plus GINI Source (browse real kernel files) and GINI Labs (submit answers)

## Part 1 — What an Operating System Is (Lecture 1)
- [ ] 1.1 The OS as resource manager — why "management" means the ability to *take back*, not just hand out
- [ ] 1.2 Dual-mode operation: the mode bit, privileged instructions, what happens when user mode tries one
- [ ] 1.3 The speed mismatch and the storage hierarchy — why the CPU can't just wait on devices
- [ ] 1.4 Three ways into the kernel (named, not yet detailed): system call, interrupt, exception
- [ ] 1.5 "Who is really in charge" — mode bit / page tables / timer as the three things that let the kernel supervise without running

## Part 2 — Traps: System Calls, Interrupts, and Faults (Lecture 2 + C-Lab 01)
- [ ] 2.1 One mechanism, three causes — deliberate (syscall) / external (interrupt) / accidental (exception), one hardware trap implementing all three
- [ ] 2.2 What the trap itself does, every time: save state, record why, switch mode, jump to a fixed kernel entry point
- [ ] 2.3 System calls as a service interface: six families, xv6's 22 calls, and why `printf` (library) isn't `write` (syscall)
- [ ] 2.4 Passing arguments across the boundary: why registers-only isn't enough, and call-by-number instead of call-by-name
- [ ] 2.5 Interrupts: the uninvited trap, DMA vs. interrupt-driven I/O, why disabling interrupts is dangerous
- [ ] 2.6 Exceptions: page faults aren't all errors (lazy allocation, copy-on-write, stack growth vs. genuinely illegal access)
- [ ] 2.7 **(GINI-specific)** scause/sepc/SPP in practice — the C-Lab 01 experiment: catching the *same* timer interrupt idle vs. busy, uservec vs. kernelvec, and why "trapframe written" and "process switch happened" are separate events

## Part 3 — Processes: Concept, State, and the PCB (Lecture 3)
- [ ] 3.1 Program vs. process — memory segments recap (text/initialized data/BSS/heap/stack), where arguments/return addresses live
- [ ] 3.2 CPU-bound vs. I/O-bound, and why multiprogramming only pays off because they overlap
- [ ] 3.3 The five-state model (New/Ready/Running/Waiting/Terminated) — and why xv6 actually has *six* (UNUSED/USED/SLEEPING/RUNNABLE/RUNNING/ZOMBIE)
- [ ] 3.4 The Process Control Block — what the kernel remembers, and why nothing survives a context switch if it isn't in there
- [ ] 3.5 xv6's `struct proc`: fixed array of 64, vs. Linux's doubly-linked list of `task_struct` — what each design buys/costs
- [ ] 3.6 Scheduling queues — ready queue vs. event queues, and the four paths a process takes after running
- [ ] 3.7 Context switch mechanics — what's saved, and the key insight: memory is never saved/restored, because isolation makes that unnecessary

## Part 4 — The Real Address Space in xv6/GINI (T01 Address-Space Lab) **(GINI-specific)**
- [ ] 4.1 Why xv6 draws the address space differently from the lecture "bands" picture — the 5-row comparison table (heap-above-stack, fixed one-page stack, no growth, one data band, fixed 64-slot process table)
- [ ] 4.2 The regions in order: text → data → guard → stack → heap → gap → trapframe → trampoline
- [ ] 4.3 The guard page — what it actually blocks (missing U bit, not "no permissions"), and why it's the cheapest safety device in the kernel
- [ ] 4.4 The trapframe — whose is it, why your own process can't touch it, why saving it happens on every trap regardless of what happens next
- [ ] 4.5 The trampoline — the "same address in every table" trick, and why a page-table-switch instruction has to live somewhere special to survive its own execution
- [ ] 4.6 xv6's honest confession: no real ready queue, no hardware-enforced region types — just a table scan and permission-bit labels a human reads afterward

## Part 5 — Process Creation and Control (Lecture 4)
- [ ] 5.1 fork(): one call, two returns — why the child gets 0 and the parent gets a PID, not the other way around
- [ ] 5.2 exec(): replacing the program inside a process — what survives (PID, parent, open files), what's discarded (the whole address space)
- [ ] 5.3 The fork+exec+wait shell loop — what every command you've ever typed actually does
- [ ] 5.4 Reasoning drills: nested fork() counting, variable isolation after fork, why fork() can't return 0 to both sides
- [ ] 5.5 exit()/wait() and resource reclamation — xv6 vs. Linux difference
- [ ] 5.6 Zombies and orphans — what's kept, who keeps it, why a zombie leak eventually stops the machine from starting anything
- [ ] 5.7 Isolation via address spaces — why every process switch is also an address-space switch

## Part 6 — CPU Scheduling, through Stride Scheduling (Lecture 5 — stops where class did)
- [ ] 6.1 The objective, and the scheduler/dispatcher split
- [ ] 6.2 Four moments scheduling happens (2 preemptive, 2 not) + five criteria that conflict with each other
- [ ] 6.3 FCFS and the convoy effect
- [ ] 6.4 SJF — provably optimal, needs the future, so we predict with exponential averaging
- [ ] 6.5 SRTN (preemptive SJF) and Round Robin (what xv6 actually runs) — quantum trade-offs
- [ ] 6.6 Priority scheduling, starvation, and aging
- [ ] 6.7 Turnaround vs. response time — worked examples showing no ordering wins both
- [ ] 6.8 Proportional share: stating the ratio instead of picking a rule-based winner
- [ ] 6.9 Lottery scheduling — the algorithm, ticket mechanisms (currency/transfer/inflation), why it's fair on average but not in the short run
- [ ] 6.10 Stride scheduling — the deterministic version, worked pass/stride example *(stop point: "the same shares, without the dice")*
- [ ] **(future)** implementing lottery scheduling in xv6 (Lab A2), and Lecture 6 — Memory Management

## Part 7 — GINI's Practice Programs (what they are, before you run them)
- [ ] 7.1 `spin` and `busy` — CPU-bound load generators for the scheduler (spin freezes the PC on one instruction; busy moves it through a real loop)
- [ ] 7.2 `walker` — makes the PC visibly march across memory pages; what it needs from the kernel (`sbrkexec`) and why
- [ ] 7.3 `alloc` — lazy/demand paging made visible; what it needs from the kernel (`sbrklazy`) and why it's a Memory-face tool
- [ ] 7.4 `grind` and `forktest` — stress/regression tests you *run*, not read closely, to sanity-check a kernel after changes
- [ ] 7.5 How these connect to T01 (you already ran walker/alloc/grind there) vs. what's still ahead (memory-management labs)

## Part 8 — T02 Lab: The sysinfo Syscall, Deep Dive (this week's actual assignment)
- [ ] 8.1 The assignment in one slide: Parts A (add the call) / B (make it tell the truth) / C (make it remember)
- [ ] 8.2 Why a syscall can't just be a jump — privilege mode + separate page tables, and the one door (`ecall`)
- [ ] 8.3 The whole trip, station by station, as a map (not code yet): user call → stub → hardware → trampoline → usertrap → syscall() → your handler → back home
- [ ] 8.4 Stations 2–3 in detail: the generated stub (`usys.pl`), and the four things `ecall` does that nothing else can
- [ ] 8.5 Stations 4–6 in detail: the trampoline page, `usertrap`'s one-function-many-reasons dispatch, `syscall()`'s table lookup
- [ ] 8.6 The five registration sites — six edits, five files, five distinct failure modes (compile/assembly/link/runtime) and what each looks like when broken
- [ ] 8.7 Station 7: how a kernel-side syscall handler fetches arguments it was never "passed" (argraw/argint/argaddr/argstr)
- [ ] 8.8 Station 8: the way home — how `sret` and the trapframe make the whole trip invisible to your program
- [ ] 8.9 Part B philosophy: why walk a list on demand instead of keeping a counter — free-list walk + process-table walk + `copyout`'s safety role
- [ ] 8.10 Part C: the producer/consumer shape — `clockintr` sampling, the ring buffer, and why the math is fixed-point integers, not floats
- [ ] 8.11 Debugging checkpoints: what "nothing in the histogram" vs. "sys23" vs. "sysinfo by name" each tell you about which station is broken

## Part 9 — Hands-On Practice in GINI (you type, I explain)
- [ ] 9.1 Orientation gut-check: open the Syscalls / Traps / Scheduler faces, confirm reboot-vs-rebuild before touching kernel source
- [ ] 9.2 Implement Part A (register the call, stub returns zeros) — verify via the histogram (nothing → sys23 → sysinfo)
- [ ] 9.3 Implement Part B (`freemem` walk, process-table walk, `copyout`) — verify against the Virtual Memory / Scheduler faces
- [ ] 9.4 Implement Part C (`clockintr` sampling, ring buffer, averaging) — verify with three `spin` processes, watch `nrunnable` move 0→3
- [ ] 9.5 Whatever breaks — debug using Part 8.11's checkpoint table before asking

---

## Sources referenced
- `tut\t2\T02-ALab01-SysInfo-SysCall.pdf`, `tut\t2\t2note.md`
- `lecture\1\N01-...pdf` through `lecture\5\N05-...pdf`
- `Notes\class1.md`–`class6.md`, `Notes\tutorial2.md`
- `assignment\A1\t_lab\T01-GINI-xv6-Orientation.pdf`, `TLab1_AddressSpace_Student.pdf`, `Tlab1_NOTE.md`
- `assignment\A1\c_lab\C-Lab-01.pdf`, `CLab1_Traps_Student.pdf`
- `GINI_Program\{walker,spin,grind,alloc,forktest,busy}.pdf`
