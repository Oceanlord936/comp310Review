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
| 6 | L06 Memory management | **in progress** — notes typed for protection, base+limit, binding (1–3); left: MMU reloc+limit notes, EXP 6.1–6.5, part 2 (linking, fits, fragmentation) |
| 7 | L07 Paging + TLB | 75–90 min |
| 8 | L08 Virtual memory | 75–90 min |

**~63% of rounds done (5 of 8; ~55% by time, L06–L08 are the longer ones); ~3.5–4 h left.** Plan: finish rounds by ~Oct 14 → sim test 1 ~Oct 15 →
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
