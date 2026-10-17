# Questions_2 — Part E: Virtual memory (p9–10)

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
