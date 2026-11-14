# COMP 310 Review — Progress

## ACTIVE: weekend of 2026-11-14 — Tutorial 5 + rest of L14 + L15

Guide: `L12_L14_review/weekend_guide.md`. Answers in `L12_L14_review/weekend_answers.md` or chat, then "check T1" etc.
Notes: `Notes/tutorial5.md` (checked + filled in on 2026-11-13), `class19.md` (rest of L14), `class20.md` (all of L15).

| Part | Material | Status |
|---|---|---|
| 1 | T1–T10: Tutorial 5 question types with new numbers | concepts ✓ (sections 4–10 in weekend_notes_l12_l14.md); questions later |
| 2a | Rest of L14: prevention, trajectories, claim edges, Banker's, detection, recovery, slide 42 (L1–L7) | concepts ✓ (sections 1–3, 7–8); questions later |
| 2b | L15 IPC: FDT, redirection, pipes, message passing, sockets, RPC (P1–P8) | concepts ✓ (11a–11d in L15_review/L15_notes.md); questions later |
| later | Old rounds R, G, H, I questions in `L12_L14_guide.md` | |

A-Lab-03: DONE (pass 1 + recorded pass 2).
2026-11-14: concept pass for Tutorial 5 + rest of L14 done (sections 1–10, notes in `L12_L14_review/weekend_notes_l12_l14.md`). L15 concept pass done the same day (`L15_review/L15_notes.md`). **Next: practice questions T1–T10, L1–L7, P1–P8, then old rounds R, G, H, I.** Weak spots seen today: in the Banker's pretend step change all 3 (avail, requester's hold AND need) and check the requester too; finished process returns its CURRENT hold; deadlocked = everyone left when the loop stops; a reader must close its own copy of p[1].

---

## ACTIVE: recall + L12 rest, L13, L14 (to slide 14), then A-Lab-03 (started 2026-11-07)

Guide: `L12_L14_review/L12_L14_guide.md`. Same format: you type notes in `L12_L14_review/L12_L14_notes.md`, answers in `L12_L14_answers.md` or chat, then "check round X".
class17/class18 fixed in place (originals in `L12_L14_review/notes_original/`).

| Round | Material | Time | Status |
|---|---|---|---|
| R | Recall: 15 picked questions from A–F | 40 min | |
| G | Rest of L12: SMP, NUMA, clusters, EXP-17 | 20 min | |
| H | L13 multi-threading (EXP-71 ×2) | 60–70 min | |
| I | L14 deadlocks slides 1–14 | 40 min | |
| Lab | A-Lab-03 buddy allocator + megapages (due Nov 17) | ~4 h | Part 0 / A / B / C |

The old A–F questions (54) are replaced by round R's 15.
**Today 2026-11-07 (real time):** 1) concepts only for G, H, I, typing notes, no questions (≈ 2 h) so you can keep up in class next week; 2) A-Lab-03 your usual way: pass 1 on GINI step by step + notes in `assignment/Alab/3/Alab03note.md` (Part 0 + A ≈ 2 h, B ≈ 1.5 h, C ≈ 2.5 h), pass 1 is with GINI recording OFF; pass 2 = the recorded run you submit, done fast from your notes (≈ 1 h). All questions (R, G, H, I) pushed to later, before the exam.
**Status end of 2026-11-07:** concept pass G, H, I ✓. A-Lab-03 pass 1 ✓ — Check all 10/10 (0, A1–A2, B1–B3, C1–C4). Your code backups: `A-lab3-backup/parta`, `partb`. **Next: pass 2 = recorded run from your notes, then submit (due Nov 17).** After that: questions R, G, H, I.

---

## Earlier: L09 recap → L12 guide rounds (started 2026-10-30)

Guide: `L09_L10_review/L09_L11_guide.md` (concepts + questions, no answers). You hand-type concept notes in `L09_L10_review/L09_L11_notes.md` (in Notepad++). Question answers go later in a separate `L09_L11_answers.md` or in chat, then say "check round X"; Claude checks and writes the right answers into the answers file.
Notes `class14–16.md` and `tutorial4.md` were corrected in place on 2026-10-30; originals in `L09_L10_review/notes_original/`. N10-The-File-System-Interface is skipped by the prof, so the L10 row in the table below is replaced by round D. T3.1–T3.5's text was never saved; rounds B and C replace them.

| Round | Material | Status |
|---|---|---|
| A | L09 quick recap | notes typed ✓; questions A1-A9 not done |
| B | Semaphore vs monitor vs condition variable | notes typed ✓; questions B1-B7 not done |
| C | Tutorial 4 examples | notes typed ✓; questions C1-C10 not done |
| D | L10 concepts (N10-File-System) | notes typed ✓; questions D1-D10 not done |
| E | L11 implementation + disk access examples | notes typed ✓; questions E1-E13 not done |
| F | L12 so far (Amdahl, up to EXP-16) | notes typed ✓; questions F1-F5 not done |

**Status: all concept notes hand-typed in `L09_L11_notes.md`. Next: answer the questions (54, round A first) in `L09_L11_answers.md`, then say "check round X".**

To confirm with prof/TA: does the root i-node count as a disk access (2k+1 vs 2k+2); the T4 cwait/csignal lines that were reconstructed.

---

## Earlier: L09–L10 + Tutorial 3 concept pass (started 2026-10-23, after midterm 1)

Same format as the midterm round 1: Claude explains a concept block in chat → you type your notes and solve the slide
problems (EXP N.x) in the file → Claude checks and fixes the file.
Files live in `L09_L10_review/` (L09.md, L10.md, T03.md). Sources: `lecture/9/N09-...pdf`, `lecture/10/N10-The-File-System-Interface.pdf`
(slides 1–14 only: up to Direct access + EXP 10.2), `Notes/class12–14.md` (L09 in class), `Notes/tutorial3.md`.

| Round | Material | Status |
|---|---|---|
| 9a | L09 part 0–1: races, critical section, 4 requirements, software attempts, Peterson (EXP 9.1, 9.2) | ✓ done — answers written by Claude (user asked for explanations first); notes on Peterson me/other, "later write overwrites earlier", 4-way-stop analogy. Weak spots: who STORES last decides the lost update; requirement numbers (3) progress / (4) bounded waiting; one CPU = no speed-up, spinning only wastes |
| 9b | L09 part 2 (hardware): barriers, TAS, CAS, atomics, xv6 spinlock (EXP 9.3 + in-class bonus). Notes in `L09b.md` | ✓ done (2026-10-23) — notes typed by user; code re-fenced in ```c blocks (Prettier was mangling `/* */`, `*`, `__`); EXP 9.3 answers written by Claude on request. Key points: TAS merges test+set (no gap); failed CAS re-reads and retries (lock-free); interrupts off in acquire or a same-CPU handler deadlocks (tickslock); 1 core → 0 spins |
| 9c + T3 | L09 part 3 (waiting without spinning) merged with Tutorial 3: mutex, semaphores (busy + blocking, initial values), patterns (signalling, rendezvous, mutex, barrier), bounded buffer, monitors, condition variables (Hoare vs Mesa), xv6 sleep/wakeup, dining philosophers (EXP 9.4 + T3.1–T3.5). Notes go in `L09c.md` | **mostly done (2026-10-23)** — notes typed + checked (fixed: mutex trace, barrier's two counters, spaces/items timeline). EXP 9.4: 1 ✗ (gave start values; answer mutex 1, empty 4, full 1), 2 ✓ idea + run added, 3 ✓, 4 answered by Claude (Mesa if → while). **Still open: T3.1–T3.5** (not attempted). Paused for A-Lab-02 |
| 10 | L10 up to direct access: file, attributes, operations, open-file tables, xv6 struct file, types, sequential/direct access (EXP 10.1, 10.2) | |

---

## DONE: Midterm 1 cram (test Oct 20, L01–L08) — last updated 2026-10-13

Everything lives in `midterm_review/`:
- `00_quick_review.md` — one-pass review of L01–L08 + every slide problem (questions only), by lecture
- `exam-hand-written-review/L0N.md` — your typed notes + solved slide problems, one file per lecture round
- `sim_test_1.md` — 90-min, 100-pt simulation (A: 2022 midterm adapted, B: prof's lab questions, C: slide
  problem types with new numbers). Answer in `sim_test_1_answers.md`, then say "grade".
  (Answer key was kept in the session scratchpad; if it's gone in a new session, Claude recomputes it when grading.)

Concept-pass rounds (one lecture each: Claude explains in chat → you type notes + solve slide problems → Claude checks/fixes):

| Round | Lecture | Status |
|---|---|---|
| 1 | L01 What an OS is | ✓ done (checked + fixed) |
| 2 | L02 Traps | ✓ done (checked + fixed; weak spots: syscall = sync, fault ≠ error, give final numbers) |
| 3 | L03 Processes | ✓ done (checked + fixed; weak spots: segments EXP 3.2, states, full-sentence answers) |
| 4 | L04 fork/exec/wait | ✓ done (checked + fixed; weak spot: xv6 zombie still has its address space) |
| 5 | L05 Scheduling | ✓ done — EXP 5.1 FCFS/SJF/SRTN/RR q=4 + scoreboard, prediction table (first too-low at burst 4), stride W=10000 ✓. Weak spots: α goes on the REAL burst; SJF sorts by newest guess; RR arrival doesn't preempt. Prof's "Your turn" 5.1–5.7 deferred to sim test |
| 6 | L06 Memory management | ✓ done — EXP 6.1–6.5 ✓. Weak spots: virtual 0 → base; give final numbers; best fit = smallest that fits (check every hole); round 2049 up to 5 units; context-switch regs live in PCB (not trapframe) |
| 7 | L07 Paging + TLB | ✓ done — notes 1–9, EXP 7.1–7.4 ✓ (7.4 part 2 answer given, check later). Weak spots: VPN bits from LOGICAL size, frame bits from PHYSICAL size; EAT miss = k·t_m + t_m (levels multiply); array start offset shifts page boundary (TLB loads one page per miss); table size = rows × PTE size (not page size) |
| 8 | L08 Virtual memory | ✓ done — notes 1–9, EXP 8.1 ✓ (user), 8.2 FIFO/LRU ✓ (user, finished by Claude), OPT ✓, Belady + 8.3 + 8.4 answers written by Claude (check later). Weak spots: 10% of 200 = 20; P_miss = 1 − hit rate; FIFO evicts leftmost, hits do not reorder; OPT = farthest NEXT use, not future count |

**Round 1 COMPLETE (8 of 8, 2026-10-16).**

**Round 2 (started 2026-10-17): Questions_2 review** in `midterm_review/Q2_review.md` (question list) with clean question files `exam-hand-written-review/Q2_partX_questions.md` and your answers in `Q2_partX_answers.md` (Part A: `Q2_partA.md`). **Status 2026-10-17: A ✓, B ✓, C ✓, D ✓, E ✓ — Questions_2 COMPLETE (E7 buddy skipped, optional).** Weak spots from D: page bits = log2(#pages) OR address bits − offset (add offset once); formula questions = generalize a worked example; word-addressed → count in words; keep going after a failed fit; TLB hit still pays one memory access + TLB time on every reference. Weak spots from E: TLB hit ⇒ page in memory (no fault); TLB miss = hardware walk, no state change; finish fault questions by taking the free frame; locality change ⇒ burst of faults; software reference bit via the valid bit (minor fault). **Next: refine mid-review-withProf.md → light final pass Oct 19.** Weak spots from C: preemptive/non-preemptive swapped; next start = previous FINISH (not turnaround); larger-number priority; RR only rotates equal priorities; C13 conflict comes from continuous arrivals — every non-red question from the prof's bank, by part (A intro/processes 12, B fork/pipes 9, C scheduling 10, D memory 13, E VM 12). You answer, Claude checks. B12–B15, B17 need the code pasted (images in the PDF). E7 buddy system optional (not in prof's notes). Then: refine mid-review-withProf.md → test on weak spots → light final pass Oct 19.
test 2 on weak spots ~Oct 17–18 → light final pass Oct 19.

---

## Original review plan (paused for the cram)

Last updated: 2026-09-30

**Overall: 60 of 77 in-scope sub-items covered (~78%; ~69% if counting the 10 skipped Part 0/1 items).** Parts 10–11 (Lectures 6–7) added 2026-09-30.
Notes for lecture parts (2–6, 10–11) live in `review_understand/<section>.md`. Notes for Parts 7–9 (tutorial/lab material, not lecture) live in `side_tutorial_review/<section>.md`.

| Part | Title | Status |
|---|---|---|
| 0 | GINI Orientation | Skipped by choice (0.1–0.5) |
| 1 | What an Operating System Is | Skipped by choice (1.1–1.5) |
| 2 | Traps: System Calls, Interrupts, and Faults | Done, 7/7 |
| 3 | Processes: Concept, State, and the PCB | Done, 7/7 |
| 4 | The Real Address Space in xv6/GINI | Done, 6/6 |
| 5 | Process Creation and Control | 6/7 — **missing 5.3** (fork+exec+wait shell loop) |
| 6 | CPU Scheduling, through Stride Scheduling | Done, 10/10 |
| 7 | GINI's Practice Programs | Done, 5/5 (in `side_tutorial_review/`) |
| 8 | T02 Lab: The sysinfo Syscall | Done, 11/11 — 8.4–8.11 consolidated in `side_tutorial_review/8.4.md`. **Its 8.11 debug table was wrong about `sys23`; corrected 2026-09-30 (see 9.1 / 9.5) — make sure your 8.4.md has the fixed rows** |
| 9 | Hands-On Practice in GINI | Done, 5/5 — notes written by Claude (non-lecture exception); reviewed your `assignment/Alab/practice/partA–C` code, all correct |
| 10 | Memory Management (Lecture 6) | 3/8 — 10.1–10.3 done |
| 11 | Paging and Address Translation (Lecture 7) | 0/11 (11.11 is L07 material not yet in your class notes) |

## Next up
**10.4** — logical vs. physical addresses and the MMU: relocation + limit, base-and-bounds worked (EXP 6.2, EXP 6.3 last two bullets).

Also worth circling back to **5.3** at some point — the one gap in an otherwise-finished Part 5.

ALab01 (sysinfo) is due **Oct 9**. Before answering the GINI Labs questions, see the "six things" checklist at the end of `side_tutorial_review/9.4.md` — two questions need the machine running.

`review_understand/trap_glossary.md` is a reference table of the trap terms (uservec, sepc, satp, PTE bits, etc.) from Part 4.

## Working style
- Parts 2–3 were brief review; Part 4 onward is new material, so more detail.
- Lecture parts: explanations in chat only; you type the notes. Side-tutorial parts (7–9): Claude writes the note file.
- One small sub-section at a time; move on only when "next" is said. (Exception made once for Part 8's 8.4–8.11.)
- Parts 2–6 and 10–11 = lecture material → `review_understand/`. Parts 7–9 = tutorial/lab material → `side_tutorial_review/`.
