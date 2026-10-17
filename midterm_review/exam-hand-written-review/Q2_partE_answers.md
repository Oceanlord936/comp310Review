# Questions_2 — my answers — Part E: Virtual memory (p9–10)

====================================

E1. Under what conditions do page faults occur? Describe what the OS does when a page fault occurs.

the lazy allocation skbr()
when access within own memory space rech a non allocated page section triggered page faults

OS check if it is valid , use kernel privilage

if not valide OS kill it

**Check: partly. Lazy allocation is one case, but give all the conditions and all the OS steps.**

When does a page fault happen? When a process touches a page whose PTE is not valid / not present:
- the page is on disk (demand paging: never loaded yet, or evicted earlier),
- the page was lazily allocated (sbrk raised the size but nothing is mapped yet, your case; xv6's only kind),
- a write to a read-only copy-on-write page,
- or a truly illegal address (outside the process's memory).

What the OS does (L08 section 2):
1. Trap into the kernel; check the address is legal. Illegal → kill the process. ✓ (yours)
2. Find where the page's contents are (on disk, or "should be zero" for a new page).
3. Get a free frame. If none, evict a victim page (write it back first if dirty).
4. Read the page in from disk. This is slow, so the process sleeps (blocked) and another process runs.
5. When the I/O finishes, update the PTE (frame number, valid bit).
6. Restart the instruction that faulted. The process never knows it happened.

=================================

E2. Order these from best to worst by average page-fault rate:

a. LRU b. FIFO c. Optimal d. Second chance

the LRU is hard to implment

FIFO the worst

Optimal also need perfict futrue

second is better then FIFO easy to implement

form best to worst

Optimal -> LRU -> second chance -> FIFO

**✓ Check: Optimal → LRU → Second chance → FIFO.** Correct order.

Reasons: Optimal is the best possible by definition (but needs the future). LRU uses the past to predict the future and is usually close to optimal. Second chance (clock) is an approximation of LRU using one reference bit, so slightly worse than LRU. FIFO ignores usage completely, so it's the worst (and can even show Belady's anomaly).

==================================

E3. ★ Reference string 7, 2, 3, 1, 2, 5, 3, 4, 6, 7, 7, 1, 0, 5, 4, 6, 2, 3, 0, 1, demand paging with 3 frames. How many page faults for:

a. LRU

{7 2 3 } all miss

1 comes miss
{2 3 1}

2 hit
{3 1 2}

5 miss

{1 2 5}

3 miss
{2 5 3}

4 miss

{5 3 4}

6 miss

{3 4 6 }

7 miss

{4 6 7 }

7 hit

{4 6 7 }

1 miss

{6 7 1}

0 miss

{7 1 0 }

...

---

★ Reference string 7, 2, 3, 1, 2, 5, 3, 4, 6, 7, 7, 1, 0, 5, 4, 6, 2, 3, 0, 1

b. FIFO

{7 2 3} miss

1 miss

{2 3 1}

2 hit

5 miss
{3 1 5}

3 hit

4 miss
{ 1 5 4}

...

---

★ Reference string 7, 2, 3, 1, 2, 5, 3, 4, 6, 7, 7, 1, 0, 5, 4, 6, 2, 3, 0, 1

c. Optimal

{7 2 3 } all miss

1 miss , 7 is farthese

{ 2 3 1 }

2 hit

5 miss , 2 is the farthese

{ 3 1 5}

3 hit

4 miss, 3 is the farthese

{1 5 4}

...

**Check: every step you wrote is correct. Here are the full traces and totals.**

**a. LRU** (least recent on the left; a hit moves the page to the right)
- 7, 2, 3: miss → {7 2 3} (3)
- 1: miss, kick out 7 → {2 3 1} (4)
- 2: hit → {3 1 2}
- 5: miss, kick out 3 → {1 2 5} (5)
- 3: miss, kick out 1 → {2 5 3} (6)
- 4: miss, kick out 2 → {5 3 4} (7)
- 6: miss, kick out 5 → {3 4 6} (8)
- 7: miss, kick out 3 → {4 6 7} (9)
- 7: hit
- 1: miss, kick out 4 → {6 7 1} (10)
- 0: miss, kick out 6 → {7 1 0} (11)
- 5: miss, kick out 7 → {1 0 5} (12)
- 4: miss, kick out 1 → {0 5 4} (13)
- 6: miss, kick out 0 → {5 4 6} (14)
- 2: miss, kick out 5 → {4 6 2} (15)
- 3: miss, kick out 4 → {6 2 3} (16)
- 0: miss, kick out 6 → {2 3 0} (17)
- 1: miss, kick out 2 → {3 0 1} (18)

LRU: **18 faults**.

**b. FIFO** (oldest on the left; a hit changes nothing)
- 7, 2, 3: miss → {7 2 3} (3)
- 1: miss, kick out 7 → {2 3 1} (4)
- 2: hit
- 5: miss, kick out 2 → {3 1 5} (5)
- 3: hit
- 4: miss, kick out 3 → {1 5 4} (6)
- 6: miss, kick out 1 → {5 4 6} (7)
- 7: miss, kick out 5 → {4 6 7} (8)
- 7: hit
- 1: miss, kick out 4 → {6 7 1} (9)
- 0: miss, kick out 6 → {7 1 0} (10)
- 5: miss, kick out 7 → {1 0 5} (11)
- 4: miss, kick out 1 → {0 5 4} (12)
- 6: miss, kick out 0 → {5 4 6} (13)
- 2: miss, kick out 5 → {4 6 2} (14)
- 3: miss, kick out 4 → {6 2 3} (15)
- 0: miss, kick out 6 → {2 3 0} (16)
- 1: miss, kick out 2 → {3 0 1} (17)

FIFO: **17 faults**. (Yes, FIFO beats LRU on this string. LRU is better on average, not on every string; this string has almost no reuse, so LRU's "recent = useful" guess keeps being wrong.)

**c. Optimal** (on a miss, kick out the page whose next use is farthest away / never)
- 7, 2, 3: miss → {7 2 3} (3)
- 1: miss. Next uses: 2 soon, 3 next, 7 farthest → kick out 7 → {2 3 1} (4)
- 2: hit
- 5: miss. Next uses: 3 soon, 1 later, 2 farthest → kick out 2 → {3 1 5} (5)
- 3: hit
- 4: miss. Next uses: 1, then 5, 3 farthest → kick out 3 → {1 5 4} (6)
- 6: miss. Next uses: 1, then 5, 4 farthest → kick out 4 → {1 5 6} (7)
- 7: miss. Next uses: 1, then 5, 6 farthest → kick out 6 → {1 5 7} (8)
- 7: hit
- 1: hit
- 0: miss. Next uses: 5 soon, 1 later, 7 never → kick out 7 → {1 5 0} (9)
- 5: hit
- 4: miss. Next uses: 0, then 1, 5 never → kick out 5 → {1 0 4} (10)
- 6: miss. 4 never used again → kick out 4 → {1 0 6} (11)
- 2: miss. 6 never used again → kick out 6 → {1 0 2} (12)
- 3: miss. 2 never used again → kick out 2 → {1 0 3} (13)
- 0: hit
- 1: hit

Optimal: **13 faults**.

Summary: **LRU 18, FIFO 17, Optimal 13.**

================================

E4. A program has just referenced an address in virtual memory.

Can each of these happen?

If yes, describe how;

if no, explain why not.

a. TLB miss with no page fault

when tlb miss , no page fault ,
this can happens when tlb cache miss

need to walk throw level of page table to find referece

b. TLB miss with page fault

miss , and page fault , we can allocation new memeory space

c. TLB hit with no page fault

what we want , direct access to referece from TLB

d. TLB hit with page fault

a new empty

**Check: a ✓, b partly, c ✓, d ✗.**

- a. **Yes.** ✓ The translation isn't in the TLB, but the page IS in memory. The hardware walks the page table, finds a valid PTE, fills the TLB, and continues. No page fault (no OS involvement).
- b. **Yes.** Not in the TLB, and the page-table walk finds the PTE not valid/present (the page is on disk, or not allocated yet). → page fault → the OS loads/allocates the page, updates the PTE, and restarts the instruction; this time the walk succeeds and the TLB is filled. (Your "allocate new memory" is one case: lazy allocation; the main case is loading the page from disk.)
- c. **Yes.** ✓ The normal, fast case: the TLB has the translation, so the page is in memory; go straight to the frame.
- d. **No.** The TLB only caches translations of pages that are IN memory. When the OS evicts a page, it also invalidates (flushes) that page's TLB entry. So if the TLB hits, the page must be in memory → no page fault. (Your "a new empty" isn't an answer to this one.)

============================

E5. ★ A demand-paging system measures:

CPU utilization 20%, paging disk 97.7%, other I/O devices 5%. For each change, say whether it will improve CPU utilization, with a one-sentence reason.

a. Install a faster CPU.

no , as already low on cpu utilization

b. Install a bigger paging disk.

not larger disk size does not improve access time

c. Increase the degree of multiprogramming.

no , we likly already reach too much process where the memory trashing occure

d. Decrease the degree of multiprogramming.

yes , this can help as less process means less page fault , less disk i/o usage

and more cpu work

e. Install more main memory.

can work , as larger main memory shift the curve of cpu optimization utilization to right , increse the cpu utilization

f. Increase the page size.

not realy , as it can casue more internal fragmenation . not resolving page fault

**Check: a–e ✓ (matches the prof's review), f: right answer, better reason below.**

Big picture (prof): CPU 20% but paging disk 97.7% = the system is THRASHING. Processes spend their time waiting for page faults, not computing. Anything that reduces paging helps; anything that adds load hurts.

- a. No ✓. The CPU isn't the bottleneck; a faster CPU would just wait faster.
- b. No ✓. The problem is too many page faults (disk BUSY), not disk SPACE.
- c. No ✓. More processes = even less memory per process = more page faults = worse thrashing.
- d. Yes ✓. Fewer processes → each gets more frames → its working set fits → fewer faults → CPU works more.
- e. Yes ✓. More frames → working sets fit → fewer faults (the prof: shifts the utilization curve to the right).
- f. **Probably not.** Memory size stays the same, so bigger pages means FEWER frames. Each fault brings in more data (helps only if spatial locality is good), but there's more internal fragmentation (more useless data per page), so working sets take more memory and thrashing isn't fixed. It doesn't attack the cause (too many processes for the memory).

=========================

E6. ★ What causes thrashing? How does the system detect it? Once detected, what can the system do to reduce or eliminate it?

when too much process , the os need to free physciall memory space by place other process into disk , take long time and after that we switch into the other process that need to retrive the data from the disk take long time and this goes on . CPU use low but disk use hight , waiting for i/o

**Check: cause ✓ (described well), but "how detected" and "what to do" are missing.**

- **Cause:** the sum of the processes' working sets is bigger than physical memory (degree of multiprogramming too high). Each process keeps evicting pages another one needs, so almost every access faults; everyone waits on the paging disk. ✓
- **Detection:** CPU utilization is LOW while the paging disk is very BUSY / the page-fault rate is very high (exactly E5's numbers). (Some systems watch each process's page-fault frequency against an upper bound.)
- **What to do:** reduce the degree of multiprogramming: suspend or swap out some processes entirely (admission control), or kill one (OOM killer), so the remaining ones' working sets fit. Longer term: add memory. (Trap: the OS must NOT react to the low CPU use by admitting MORE processes.)

=========================

E8. A system uses pure demand paging.

a. When a process first starts, how would you describe its page-fault rate?

it will has alot of page fault as the data is not here
need to find from disk

b. Once its working set is loaded, how would you describe the page-fault rate?

it will has less page fault rate
only cause page fault when malloc or need to do write change and changed

c. The process changes locality, and the new working set is larger and different from the old one. What happens to the page-fault rate?

the new working set is lager , means the whole memory space sum is lager , page fault rate should not change as we only take when needed

**Check: a ✓, b ✓ (but the reason needs fixing), c ✗.**

- a. ✓ **Very high.** Pure demand paging starts with zero pages in memory, so every first touch of a page (code, data, stack) faults.
- b. ✓ **Low.** The pages it is using are all in memory now, so almost every access hits. Faults happen only when it touches a page outside its working set.
  (Fix your reason: writing to a page that's already loaded does NOT fault, and malloc doesn't fault by itself. Only touching a page that isn't in memory faults.)
- c. ✗ **It goes UP.** The new working set's pages aren't in memory, so there's a burst of faults while they load. If the new working set is bigger than the frames the process has, the fault rate STAYS high (it can start thrashing); if it fits, the rate drops back down once it's loaded.
  ("We only load when needed" is true, but that's exactly why there are faults: every needed page that isn't there yet is a fault.)

========================

E9. Simplified process states: ready, running, blocked. A process is running.

a. Does it change state if it takes a page fault?
If so, to which state?

yes , it change from running to blocked
as it will take time to resolve teh page fault

b. Does it change state if it gets a TLB miss that is resolved from the page table?

it chaneg from running to ready

wait for page tabel walk through but faster than a page fault that need slow disk access

**Check: a ✓, b ✗.**

- a. ✓ **Yes: running → blocked** (waiting for the disk read). When the I/O completes, it goes blocked → ready, and later runs again and restarts the instruction.
- b. ✗ **No state change.** On RISC-V/x86 the hardware walks the page table itself in a few memory accesses (nanoseconds), so the process just keeps running; the OS isn't even involved. Running → ready only happens when the OS takes the CPU away (timer/preemption), and running → blocked only for slow waits like disk I/O.

===============================

E10. A system has 12-bit virtual and physical addresses and 256-byte pages.

Free frames, in order: 9, F, D. A dash means the page is not in memory.

Convert each virtual address to a physical address (all numbers hex).

On a page fault, take the next free frame, update the page table, then translate.

Page table (page → frame):

0 → 4, 1 → B, 2 → A, 3 → –, 4 → –, 5 → 2, 6 → –, 7 → 0, 8 → C, 9 → 1

a. 0x2A1

offset is same

256 byte per page that is 2 to power of 8 , 8 bits

0x 2 map to A

physical is

## 0x A A1

b. 0x4E6

4 map to -
not in memory

---

c. 0x94A

9 map to 1

physicla is

0x 1 4A

---

d. 0x316

3 map to -
not in memory

**Check: a ✓, c ✓, b and d: you stopped at "not in memory", but the question says to resolve the fault.**

(Check this page table against the PDF; the text extraction was messy.)

256-byte pages = 2⁸ → 8 offset bits = the last **2 hex digits**. 12-bit address = 3 hex digits, so the **first hex digit is the page number**. ✓ (your method)

- a. 0x2A1: page 2 → frame A → **0xAA1** ✓
- b. 0x4E6: page 4 → "–" → page fault. Take the first free frame, **9**; update the table (4 → 9) → **0x9E6**
- c. 0x94A: page 9 → frame 1 → **0x14A** ✓
- d. 0x316: page 3 → "–" → page fault. Take the next free frame, **F**; update the table (3 → F) → **0xF16**

Free frames are used in the given order (9, then F; D stays free). The offset digits are always copied unchanged.

=======================================

E11. A page-replacement algorithm is known to suffer from Belady's anomaly. Can it be optimal?

Belady's anomaly has been skimped by prof , explain

**Concept: Belady's anomaly** (from L08 part 2): with some algorithms, giving a process MORE frames can cause MORE page faults. Example: FIFO on 1,2,3,4,1,2,5,1,2,3,4,5 has 9 faults with 3 frames but 10 with 4.

Why it happens: FIFO evicts by arrival order and ignores use, so the set of pages it keeps with 4 frames is not simply "the 3-frame set plus one more page". The 4-frame set can be missing a page the 3-frame set has, and that page is exactly the next one needed.

Algorithms with the **stack property** (LRU, Optimal) can't have it: their N-frame set is always a subset of their (N+1)-frame set, so more frames can only add hits.

**Answer E11: No, it can't be optimal.** The optimal algorithm (OPT) gives the minimum possible faults for every number of frames, and OPT has the stack property, so its fault count never goes up when frames are added. An algorithm whose faults go UP with more frames does worse than OPT in that case, so it isn't optimal.

=======================

E12. You need a reference bit for your replacement algorithm,
but the hardware doesn't provide one in the page table.

Can you create one from information you do find in the page table?

there is dity bits or somehing

**Check: ✗ not the dirty bit. Yes, you can simulate a reference bit with the VALID bit.**

Why not the dirty bit: it's only set on WRITES, so a page that's only read would look unused. (And on most hardware the dirty bit is also set by hardware, so if there's no reference bit there may be no dirty bit either.)

How (software reference bit):
1. The OS keeps its own table: "software reference bit" per page.
2. Periodically (or as the clock hand passes), for each resident page: set software bit = 0 and mark the PTE INVALID on purpose, while remembering the page is actually in memory.
3. When the process touches that page, the hardware sees "invalid" → page fault → trap to the OS.
4. The OS sees the page is really in memory: sets software bit = 1, marks the PTE valid again, restarts the instruction. No disk I/O.

So "was it touched since I last cleared it?" = software bit. Think of it as a deliberate false alarm: the page is really in RAM, the trap just tells the OS "this page was used". The replacement algorithm (e.g. clock) later reads these software bits to pick a victim (bit 0 = not touched recently). A fault resolved without disk I/O like this is a minor (soft) page fault; one that reads from disk is a major (hard) page fault. Cost: one extra (cheap, no-disk) fault on the first access after each clearing. Same idea can be done with protection bits (no access) instead of the valid bit.
