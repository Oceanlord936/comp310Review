# Questions_2 review (prof's question bank, red-text questions removed)

Source: `mid1RE/Questions_2.pdf`. Exam = 12 questions in 80 min (prof, review class Oct 16).
★ = the prof went through this exact question (or a twin) in the review class, see `exam-hand-written-review/mid-review-withProf.md`.
Skipped: the whole threads section (p2–3, all red), scheduling 8, 9e, 11, 12 (red).

Write your answer under each question. Short answers are fine for concept questions; show the steps for calculations.

---

## Part A: Intro, processes (p1)

**A1.** What is the difference between uni- and multi-programming systems?

**A2.** ★ Why is multi-programming useful even in hardware with a single processing core?

**A3.** Would multi-programming need more memory than uni-programming? Briefly explain your reasoning.

**A4.** You have a single-core computer. Give the type of programs (or example programs) where multi-programming would NOT give better overall performance than uni-programming. Very briefly explain why.

**A5.** Give an example of an I/O-bound application (an application that would create an I/O-bound process).

**A6.** Give an example of a CPU-bound application.

**A7.** ★ Pick a simple C program and determine where each of its variables is located in the memory layout (text, data, bss, heap, stack).

**A8.** ★ Your program has a function `abc()`. Where is the code of the function in the memory layout? When the function is called, what changes in the memory layout?

**A9.** Why do we have a wait queue for I/O requests?

**A10.** Why do we need to minimize the context-switching overhead?

**A11.** ★ Give C-like pseudo code for a simple shell, within 15 lines, showing all the important operations.

**A12.** Why is it important that a process has its own address space?

---

## Part B: fork / exec / wait code, pipes (p3–5)

Questions B12–B17 come with a code listing that is an **image** in the PDF. Paste the code (or a screenshot) here when we get to them.

**B12.** What would be the output from the program at LINE C and LINE P? *(code needed)*

**B13.** How many processes are created in the following program? *(code needed)*

**B14.** When will the program reach LINE J (i.e. execute `printf("LINE J")`)? *(code needed)*

**B15.** What are the pid values in the following program? *(code needed)*

**B16.** What is a zombie process?

**B17.** What output would we get at LINE X and LINE Y in the following program? *(code needed)*

**B18.** Give an example where pipes are more suitable than sockets.

**B19.** Briefly explain how output redirection is implemented.

**B20.** Can we replace a pipe with output and input redirection?

---

## Part C: CPU scheduling (p6–7)

**C1.** A scheduling algorithm decides an order for running its processes. Given n processes to schedule on one processor, how many different schedules are possible? Give a formula in terms of n.

**C2.** Explain the difference between preemptive and non-preemptive scheduling.

**C3.** These processes arrive at the times shown and run for the burst times shown. Use **non-preemptive** scheduling, and make every decision with only the information available at that moment.

- P1: arrives 0.0, burst 8
- P2: arrives 0.4, burst 4
- P3: arrives 1.0, burst 1

a. Average turnaround time with FCFS?
b. Average turnaround time with SJF?
c. SJF is supposed to improve performance, but we ran P1 at time 0 because we didn't know two shorter processes would arrive soon. Compute the average turnaround time if the CPU is left **idle for the first 1 unit**, then SJF is used. (P1 and P2 are waiting during the idle time, so their waiting time grows.) This is "future-knowledge scheduling".

**C4.** These processes all arrive at time 0, in the order P1, P2, P3, P4, P5. Burst times are in ms.

- P1: burst 2, priority 2
- P2: burst 1, priority 1
- P3: burst 8, priority 4
- P4: burst 4, priority 2
- P5: burst 5, priority 3

a. Draw four Gantt charts: FCFS, SJF, non-preemptive priority (**larger number = higher priority**), and RR with quantum 2.
b. Turnaround time of each process for each algorithm?
c. Waiting time of each process for each algorithm?
d. Which algorithm gives the minimum average waiting time?

**C5.** A scheduler favours processes that have used the **least processor time in the recent past**. Why does it favour I/O-bound programs, and yet not permanently starve CPU-bound programs?

**C6.** What is the relation between priority scheduling and SJF? Ways they are related, and ways they are not.

**C7.** What is the relation between priority scheduling and FCFS?

**C9.** Which of these can result in starvation? Briefly explain how starvation happens in each.
a. FCFS   b. SJF   c. Round robin   d. Priority   f. Lottery scheduling

**C10.** Preemptive, priority-based, round-robin scheduling. A **higher number = higher priority**. The scheduler runs the highest-priority process; processes with the same priority share the CPU round-robin with **quantum 10**. A process preempted by a higher-priority process goes to the **end** of its queue.

- P1: priority 8, burst 15, arrives 0
- P2: priority 3, burst 20, arrives 0
- P3: priority 4, burst 20, arrives 20
- P4: priority 4, burst 20, arrives 25
- P5: priority 5, burst 5, arrives 45
- P6: priority 5, burst 15, arrives 55

a. Gantt chart of the scheduling order.
b. Turnaround time of each process?
c. Waiting time of each process?

**C13.** You must design a scheduler that minimizes **average turnaround time** and also minimizes **maximum waiting time**. Is there a conflict in achieving both at once? If so, briefly explain.

---

## Part D: Memory, paging (p8)

**D1.** ★ A logical address space of **256 pages**, page size **4 KB**, mapped onto a physical memory of **64 frames**.
a. How many bits are in the logical address?
b. How many bits are in the physical address?

**D2.** A computer has 32-bit logical addresses and 4 KB pages. It supports up to 512 MB of physical memory. A PTE is 4 bytes. How many levels are in a forward (multi-level) page table?

**D3.** A logical address A (generated by the CPU) and page size P. Give an expression for the page offset and the page number (single-level page table).

**D4.** Page size 1 KB. Give the page number and offset for these addresses (decimal), using your D3 expressions:
a. 3085   b. 42095   c. 215201   d. 650000   e. 2000001

**D5.** ★ A logical address space of **64 pages of 1024 words each**, mapped onto a physical memory of **32 frames**.
a. How many bits in the logical address?
b. How many bits in the physical address?

**D6.** What is the purpose of paging the page tables?

**D7.** ★ Six memory partitions of 100 MB, 170 MB, 40 MB, 205 MB, 300 MB, 185 MB (in order). Place processes of 200 MB, 15 MB, 185 MB, 75 MB, 175 MB, 80 MB (in order) with first fit, best fit, and worst fit. Say which requests (if any) cannot be satisfied, and comment on the results.

**D8.** Compare contiguous allocation and paging on:
a. external fragmentation   b. internal fragmentation   c. ability to share code across processes

**D9.** With paging, a process cannot access memory it does not own. How does the OS enforce this? Briefly explain.

**D10.** What advantage do inverted page tables have over forward page tables?

**D11.** What is the disadvantage of inverted page tables?

**D12.** ★ What is the difference between internal and external fragmentation? (Prof: also say when each one is a concern.)

**D13.** A paging system with a **two-level** forward page table stored in memory.
a. If a memory reference takes 50 ns, how long does a paged memory reference take?
b. We add a TLB, and 75% of page-table references are found in it. What is the effective memory reference time? (Looking up the TLB takes 2 ns, and this time is spent whether or not the entry is found.)

---

## Part E: Virtual memory (p9–10)

**E1.** Under what conditions do page faults occur? Describe what the OS does when a page fault occurs.

**E2.** Order these from best to worst by average page-fault rate: a. LRU   b. FIFO   c. Optimal   d. Second chance

**E3.** ★ Reference string `7, 2, 3, 1, 2, 5, 3, 4, 6, 7, 7, 1, 0, 5, 4, 6, 2, 3, 0, 1`, demand paging with **3 frames**. How many page faults for:
a. LRU   b. FIFO   c. Optimal

**E4.** A program has just referenced an address in virtual memory. Can each of these happen? If yes, describe how; if no, explain why not.
a. TLB miss with no page fault
b. TLB miss with page fault
c. TLB hit with no page fault
d. TLB hit with page fault

**E5.** ★ A demand-paging system measures: CPU utilization 20%, paging disk 97.7%, other I/O devices 5%. For each change, say whether it will improve CPU utilization, with a one-sentence reason.
a. Install a faster CPU.
b. Install a bigger paging disk.
c. Increase the degree of multiprogramming.
d. Decrease the degree of multiprogramming.
e. Install more main memory.
f. Increase the page size.

**E6.** ★ What causes thrashing? How does the system detect it? Once detected, what can the system do to reduce or eliminate it?

**E7.** *(optional: the buddy system is not in the prof's L01–L08 notes)* In a 1024 KB segment, memory is allocated with the buddy system. Draw the tree for these requests: 5 KB, 135 KB, 14 KB, 3 KB, 12 KB. Then show the tree after these releases, coalescing whenever possible: 3 KB, 5 KB, 14 KB, 12 KB.

**E8.** A system uses pure demand paging.
a. When a process first starts, how would you describe its page-fault rate?
b. Once its working set is loaded, how would you describe the page-fault rate?
c. The process changes locality, and the new working set is larger and different from the old one. What happens to the page-fault rate?

**E9.** Simplified process states: ready, running, blocked. A process is running.
a. Does it change state if it takes a page fault? If so, to which state?
b. Does it change state if it gets a TLB miss that is resolved from the page table?

**E10.** A system has **12-bit** virtual and physical addresses and **256-byte** pages. Free frames, in order: 9, F, D. A dash means the page is not in memory. Convert each virtual address to a physical address (all numbers hex). On a page fault, take the next free frame, update the page table, then translate.

Page table (page → frame): 0 → 4, 1 → B, 2 → A, 3 → –, 4 → –, 5 → 2, 6 → –, 7 → 0, 8 → C, 9 → 1

a. 0x2A1   b. 0x4E6   c. 0x94A   d. 0x316

**E11.** A page-replacement algorithm is known to suffer from Belady's anomaly. Can it be optimal?

**E12.** You need a reference bit for your replacement algorithm, but the hardware doesn't provide one in the page table. Can you create one from information you do find in the page table?
