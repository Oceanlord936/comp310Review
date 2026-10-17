# COMP 310 Review — Progress

## ACTIVE: Midterm 1 cram (test Oct 20, L01–L08) — last updated 2026-10-13

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

**Round 2 (started 2026-10-17): Questions_2 review** in `midterm_review/Q2_review.md` (question list) with clean question files `exam-hand-written-review/Q2_partX_questions.md` and your answers in `Q2_partX_answers.md` (Part A: `Q2_partA.md`). **Status 2026-10-17: A ✓, B ✓, C ✓ (checked + explained in the answer files). Left: D (memory, 13), E (VM, 12).** Weak spots from C: preemptive/non-preemptive swapped; next start = previous FINISH (not turnaround); larger-number priority; RR only rotates equal priorities; C13 conflict comes from continuous arrivals — every non-red question from the prof's bank, by part (A intro/processes 12, B fork/pipes 9, C scheduling 10, D memory 13, E VM 12). You answer, Claude checks. B12–B15, B17 need the code pasted (images in the PDF). E7 buddy system optional (not in prof's notes). Then: refine mid-review-withProf.md → test on weak spots → light final pass Oct 19.
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
