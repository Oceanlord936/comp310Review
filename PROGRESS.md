# COMP 310 Review — Progress

Last updated: 2026-09-22

**Overall: 52 of 58 in-scope sub-items covered (~90%; ~76% if counting the 10 skipped Part 0/1 items).**
Notes for Parts 2–6 live in `review_understand/<section>.md`. Notes for Parts 7–9 (tutorial/lab material, not lecture) live in `side_tutorial_review/<section>.md`.

| Part | Title | Status |
|---|---|---|
| 0 | GINI Orientation | Skipped by choice (0.1–0.5) |
| 1 | What an Operating System Is | Skipped by choice (1.1–1.5) |
| 2 | Traps: System Calls, Interrupts, and Faults | Done, 7/7 |
| 3 | Processes: Concept, State, and the PCB | Done, 7/7 |
| 4 | The Real Address Space in xv6/GINI | Done, 6/6 |
| 5 | Process Creation and Control | 6/7 — **missing 5.3** (fork+exec+wait shell loop; not covered yet, gap noticed while updating this file) |
| 6 | CPU Scheduling, through Stride Scheduling | Done, 10/10 |
| 7 | GINI's Practice Programs | Done, 5/5 (in `side_tutorial_review/`) |
| 8 | T02 Lab: The sysinfo Syscall | Done, 11/11 — 8.1–8.3 have their own files; 8.4–8.11 were consolidated into a single `side_tutorial_review/8.4.md` at your request, since they're all one continuous walkthrough |
| 9 | Hands-On Practice in GINI | 0/5 — not started |

## Next up
**9.1** — orientation gut-check (open Syscalls/Traps/Scheduler faces, confirm reboot-vs-rebuild before touching kernel source). This is hands-on: you'll be typing real kernel edits with me explaining as you go, so pacing goes back to one small step at a time.

Also worth circling back to **5.3** at some point — it's the one gap in an otherwise-finished Part 5.

`review_understand/trap_glossary.md` is a reference table of the trap terms (uservec, sepc, satp, PTE bits, etc.) from Part 4.

## Working style
- Parts 2–3 were brief review; Part 4 onward is new material, so more detail.
- Explanations in chat only. Notes and all code are typed by hand.
- One small sub-section at a time; move on only when "next" is said. (Exception made once for Part 8's 8.4–8.11, at your request, since they're all one continuous mechanism.)
- Parts 2–6 = lecture material → `review_understand/`. Parts 7–9 = tutorial/lab material → `side_tutorial_review/`.
