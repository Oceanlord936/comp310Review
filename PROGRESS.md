# COMP 310 Review — Progress

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
