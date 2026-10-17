# Questions_2 — my answers — Part C: CPU scheduling (p6–7)

=================================

C1. A scheduling algorithm decides an order for running its processes. Given n processes to schedule on one processor, how many different schedules are possible? Give a formula in terms of n.

It's the number of permutations of n items, which is n! = n × (n − 1) × … × 2 × 1.

For the exam, write "n!" with that one-line reason.

**✓ Check:** correct. n! (n choices for the first, n − 1 for the second, …, 1 for the last). Point of the question: even 10 processes have 10! ≈ 3.6 million orders, so schedulers use a policy instead of searching for the best one.

================================

C2. Explain the difference between preemptive and non-preemptive scheduling.

preemptive means when a new comes , it has to wait for the current running process to be done , cannot swich and run the new one

non-preemptive is the oposite

**✗ Check: the two definitions are swapped.**

- **Preemptive**: the OS CAN take the CPU away from the running process before it finishes its burst, e.g. a timer interrupt (RR) or a higher-priority / shorter arrival (SRTN, preemptive priority). The running process goes back to the ready queue.
- **Non-preemptive**: once a process gets the CPU it keeps it until it finishes its burst or blocks itself (I/O, exit). A new arrival just waits in the ready queue. (FCFS, SJF.)

What you described ("the new one has to wait for the current one to be done") is non-preemptive.
Extra: preemptive needs hardware support (timer interrupt) and costs more context switches, but gives better response time.

============================

C3. These processes arrive at the times shown and run for the burst times shown. Use non-preemptive scheduling, and make every decision with only the information available at that moment.

P1: arrives 0.0, burst 8
P2: arrives 0.4, burst 4
P3: arrives 1.0, burst 1

a. Average turnaround time with FCFS?

as first come first server

trace each

P1 arrive 0 start 0 , end at 8 . trunaround is 8

P2 arrive 0.4 , start at 8 , end at 12
turnaound is 12 - 0.4 = 11.6

P3 arrive 1 , start at 12 , end at 13
turnaround is 13 -1 = 12

avg = (8 + 11.6 + 12 ) / 3 = 10.53

---

b. Average turnaround time with SJF?

with SJF , shortest job first

P1 arrive 0 , start 0 , end 8 , truanorud 8

P2 arirve 0.4 ,

P3 arrive 1 ,

run P3 first as shroter , end 9 turnaround
9 -1 = 8

p2 run at 9 end at 13
turn around is 13 - 0.4= 12.6

avg = (8 + 8 + 12.6) /3 = 9.53

---

c. SJF is supposed to improve performance, but we ran

P1 at time 0 because we didn't know two shorter processes would arrive soon.

Compute the average turnaround time if the CPU is left idle for the first 1 unit, then SJF is used. (P1 and P2 are waiting during the idle time, so their waiting time grows.) This is "future-knowledge scheduling".

if cpu idle for 1 unite , that is

all 3 has been arrive
it run P3

arrive 1 , end at 2 , turnaround 1

run p2 , arrive 0.4 , end at 6 , trunaroudn 5.6

run p1 , arrive 0 , end at 5.6 + 8 = 13.6

turnaround 13.6

avg = (1 + 5.6 + 13.6) /3 = 6.73

**Check:**

a. ✓ FCFS: TA 8, 11.6, 12 → avg 31.6 / 3 ≈ **10.53**.

b. ✓ SJF (non-preemptive): at t = 0 only P1 is there, so P1 runs 0–8. At 8, P3 (1) beats P2 (4): P3 8–9 (TA 9 − 1 = 8), P2 9–13 (TA 13 − 0.4 = 12.6). Avg 28.6 / 3 ≈ **9.53**.

c. ✗ one slip at P1. Idle 0–1, then SJF with all three waiting:
- P3 runs 1–2, TA = 2 − 1 = **1** ✓
- P2 runs 2–6, TA = 6 − 0.4 = **5.6** ✓
- P1 runs **6–14**, TA = 14 − 0 = **14** (you wrote end 5.6 + 8 = 13.6: you added the burst to P2's TURNAROUND 5.6, but P1 starts at P2's FINISH time 6)

Avg = (1 + 5.6 + 14) / 3 = 20.6 / 3 ≈ **6.87** (not 6.73).

Rule: next start = previous FINISH time, never previous turnaround. Turnaround is only for the answer, not for the timeline.

Why this is interesting: idling 1 unit on purpose beats normal SJF (6.87 vs 9.53), because the long job no longer blocks the two short ones. But it needs knowledge of the future, so a real scheduler can't do it.

=================================

C4. These processes all arrive at time 0, in the order P1, P2, P3, P4, P5. Burst times are in ms.

P1: burst 2, priority 2
P2: burst 1, priority 1
P3: burst 8, priority 4
P4: burst 4, priority 2
P5: burst 5, priority 3

a. Draw four Gantt charts: FCFS, SJF, non-preemptive priority (larger number = higher priority), and RR with quantum 2.

FCFS

P1 run at 0 end 2

P2 run at 2 end at 3

P3 run at 3 end at 11

P4 run at 11 end at 15

P5 run at 15 end at 20

---

SJF

P2 run at 0 , end at 1

P1 run at 1 , end at 3

P4 run at 3 , end at 7

P5 run at 7 , end at 12

P3 run at 12 , end at 20

---

non-preemptive priority

P2 , run 0 end 1
P1 , run 1 end 3
P4 , run 3 end 7
P5 , run 7 end 12
P3 , run 12 end 20

---

RR with quantum 2

P1 run 0 , end 2

P2 run 2 end 3

P3 run 3 end 5

P4 run 5 end 7

P5 run 7 end 9

P3 run 9 end 11

P4 run 11 end 13

P5 run 13 end 15

P3 run 15 end 17

P5 run 17 end 18

P3 run 18 end 20

---

b. Turnaround time of each process for each algorithm?

FCFS

as all arrive 0 , the wait = trunaround - burst
P1 2 -2 = 0

P2 3-1 = 2

P3 11 - 8 = 3

P4 15 -4 = 11

P5 20 - 5 - 15

---

...

---

d. Which algorithm gives the minimum average waiting time?

for min wait time SJF

**Check:**

**a. Gantt charts**

FCFS ✓: | P1 0–2 | P2 2–3 | P3 3–11 | P4 11–15 | P5 15–20 |

SJF ✓: | P2 0–1 | P1 1–3 | P4 3–7 | P5 7–12 | P3 12–20 |

Non-preemptive priority ✗ (you copied the SJF order). LARGER number = higher priority: P3 (4) > P5 (3) > P1 (2) = P4 (2) > P2 (1). Tie P1/P4 → FCFS order, P1 first.
| P3 0–8 | P5 8–13 | P1 13–15 | P4 15–19 | P2 19–20 |

RR q = 2 ✓ (all correct): | P1 0–2 | P2 2–3 | P3 3–5 | P4 5–7 | P5 7–9 | P3 9–11 | P4 11–13 | P5 13–15 | P3 15–17 | P5 17–18 | P3 18–20 |

**b. Turnaround** (everyone arrives at 0, so TA = finish time)

(✗ your part b computed TA − burst, which is the WAITING time, under the turnaround heading; and "20 − 5 − 15" should be 20 − 5 = 15.)

- FCFS: P1 2, P2 3, P3 11, P4 15, P5 20 → avg 51 / 5 = 10.2
- SJF: P1 3, P2 1, P3 20, P4 7, P5 12 → avg 43 / 5 = 8.6
- Priority: P1 15, P2 20, P3 8, P4 19, P5 13 → avg 75 / 5 = 15
- RR: P1 2, P2 3, P3 20, P4 13, P5 18 → avg 56 / 5 = 11.2

**c. Waiting time** (= TA − burst; bursts P1 2, P2 1, P3 8, P4 4, P5 5)

- FCFS: P1 0, P2 2, P3 3, P4 11, P5 15 → avg 31 / 5 = 6.2 (your numbers, now under the right heading)
- SJF: P1 1, P2 0, P3 12, P4 3, P5 7 → avg 23 / 5 = 4.6
- Priority: P1 13, P2 19, P3 0, P4 15, P5 8 → avg 55 / 5 = 11
- RR: P1 0, P2 2, P3 12, P4 9, P5 13 → avg 36 / 5 = 7.2

**d.** ✓ **SJF**, average wait 4.6 ms (SJF is provably optimal for average waiting time when everyone arrives together).

**Understanding: non-preemptive priority, and which number is "high"**

- Non-preemptive priority = same as SJF, but sort by priority number instead of burst. When the CPU is free, pick the highest-priority process among those that have arrived (tie → earlier arrival, FCFS), and let it run its WHOLE burst with no interruption, even if a higher-priority one arrives meanwhile. The others wait in the ready queue, which here acts like a pool sorted by priority (arrival order only breaks ties).
- "High priority" always means "picked first". Only the numbering differs: many textbooks / Linux nice use smaller number = higher; Questions_2 C4 and C10 say LARGER number = higher. Always read the question; if it doesn't say, write your assumption in one line.
- Shortcut when everyone arrives at 0: turnaround = finish − 0 = finish time, and wait = finish − burst (= start time if the job never stops once started). Only works when ALL arrivals are 0 (not C3 or C10).

==============================

C5. A scheduler favours processes that have used the least processor time in the recent past. Why does it favour I/O-bound programs, and yet not permanently starve CPU-bound programs?

a schedulare let least process time one run first
this is close to the future perdiction of SJF

it does not start cpu bound process forever
as there is aging , when a less priority one age too long it gain priority

**Check: partly right.** Aging is the right idea for the second half, but explain both halves:

- **Why it favours I/O-bound programs:** an I/O-bound program runs a short CPU burst and then blocks for I/O, so it has used very LITTLE CPU time recently. The scheduler picks it first. (Good result: it quickly issues its next I/O and gets out of the way, keeping devices busy.)
- **Why CPU-bound programs don't starve:** they have used a lot of CPU recently, so they get low priority. But the measure is only the RECENT past. While a CPU-bound program waits without running, its recent usage keeps dropping (old usage ages out of the window), so its priority keeps rising until it is picked. That's built-in aging.

(Your "close to SJF prediction" is a fair link: low recent CPU use ≈ short predicted burst.)

================================

C6. What is the relation between priority scheduling and SJF? Ways they are related, and ways they are not.

SJF treat shortest cpu burst as high priority

they are not the same when some high priority process
is i/o related

**Check: first line right, second line vague.**

Related:
- SJF IS a priority scheduler where the priority = the (predicted) next CPU burst, and shorter burst = higher priority.
- Both pick the next job by a number, and both can starve the jobs at the bottom (long jobs in SJF, low priority in priority scheduling). Both are fixed by aging.

Not related:
- General priority can come from anything: importance, user, payment, deadlines, set externally by the OS or admin. It has nothing to do with burst length.
- So a high-priority job can be very long, and SJF would run it last while priority scheduling runs it first.
- SJF needs a burst prediction (exponential averaging); priority scheduling just reads the assigned number.

============================

C7. What is the relation between priority scheduling and FCFS?

when the priority of process is same as their arrive time , the FCFS match proority schdeuling

when they are not it is not

**Check: ✓ right idea.** FCFS = priority scheduling where priority is the arrival time (earlier = higher).

Add: if all processes have the SAME priority, priority scheduling becomes FCFS, because ties are broken first-come-first-served. Not the same when priorities differ: a late high-priority process jumps ahead of earlier ones.

==========================

C9. Which of these can result in starvation?

Briefly explain how starvation happens in each.

a. FCFS b. SJF c. Round robin d. Priority f. Lottery scheduling

priority can cause starvation , when high priority process keeps coming , shortest priority get no change to run

**Check: priority ✓, but the other four are missing.** Answer each:

- a. **FCFS: no starvation.** Every process is eventually reached in arrival order (as long as each one terminates). It can wait a long time behind a long job (convoy effect), but that's not starvation.
- b. **SJF: yes.** If short jobs keep arriving, a long job keeps getting pushed back and may never run.
- c. **Round robin: no.** Each process gets a quantum every round, so with n processes it waits at most (n − 1) × q.
- d. **Priority: yes** (your answer). If higher-priority processes keep arriving, a low-priority one never runs. Fix: aging.
- f. **Lottery: no (not permanently).** Any process holding at least one ticket has a non-zero chance every time slice, so the chance of never winning goes to 0 over time. It can be unlucky in the short run. (Only a process with 0 tickets would never run.)

=======================================

C10. Preemptive, priority-based, round-robin scheduling.

A higher number = higher priority.

The scheduler runs the highest-priority process; processes with the same priority share the CPU round-robin with quantum 10.

A process preempted by a higher-priority process goes to the end of its queue.

P1: priority 8, burst 15, arrives 0
P2: priority 3, burst 20, arrives 0
P3: priority 4, burst 20, arrives 20
P4: priority 4, burst 20, arrives 25
P5: priority 5, burst 5, arrives 45
P6: priority 5, burst 15, arrives 55

a. Gantt chart of the scheduling order. b. Turnaround time of each process? c. Waiting time of each process?

P1 and P2 arribe , P1 higher priority

run for 10 for P1

{P2 in queue}

switch

run P2 for 10 {P1} in queue

P3 comes

in queue {P1 P2 P3}

P1 higher prority

run P1 for 5 done
P4 arrive , in queue {P2 P3 P4}

P4 run , till 30 queue {P2 P3}

P3 run , till 40 queue{P2 P3}

p2 Run, at 45 P5 comes {P3 P5}

at 50 P2 done

{P3 P5}
p5 run , till 55 p6 comes {P6}
and P5 done

at 55 P6 run ,
till 60 , no other in queue

keep run P6 till 65 , done

**Check: ✗ the first step is wrong, and that shifts everything.**

The mistake: at t = 10 you switched from P1 to P2. RR only rotates among processes of the SAME priority. P1 (8) is the only priority-8 process, so when its quantum ends it just gets another one. P2 (3) can't run while a higher-priority process is ready.

Rules: always run the highest priority that's ready; a higher-priority arrival preempts immediately (the preempted process goes to the END of its own priority's queue); equal priorities rotate every 10.

Trace:
- 0–15: P1 (8) runs all 15 (no other priority 8). Done at 15.
- 15–20: only P2 (3) is ready. P2 runs 5 (15 left).
- 20: P3 (4) arrives, 4 > 3 → preempts P2. 20–30: P3 runs (quantum started at 20).
- 25: P4 (4) arrives, same priority → waits in the priority-4 queue (no preemption).
- 30: P3's quantum ends (10 left) → P4 runs 30–40 (10 left), P3 to the back.
- 40: P4's quantum ends → P3 runs 40–45.
- 45: P5 (5) arrives, 5 > 4 → preempts P3 (5 left), P3 goes to the end: queue P4, P3. P5 runs 45–50, done.
- 50: P4 runs 50–55.
- 55: P6 (5) arrives → preempts P4 (5 left), queue P3, P4. P6 runs 55–70 (no other priority 5 left, so it keeps going after its quantum). Done at 70.
- 70–75: P3 finishes. 75–80: P4 finishes. 80–95: P2 finishes its last 15.

**a. Gantt:** | P1 0–15 | P2 15–20 | P3 20–30 | P4 30–40 | P3 40–45 | P5 45–50 | P4 50–55 | P6 55–70 | P3 70–75 | P4 75–80 | P2 80–95 |

(Check: total burst 15 + 20 + 20 + 20 + 5 + 15 = 95, no idle time, ends at 95 ✓)

**b. Turnaround** (finish − arrival): P1 15 − 0 = **15**, P2 95 − 0 = **95**, P3 75 − 20 = **55**, P4 80 − 25 = **55**, P5 50 − 45 = **5**, P6 70 − 55 = **15**

**c. Waiting** (TA − burst): P1 **0**, P2 **75**, P3 **35**, P4 **35**, P5 **0**, P6 **0**

Notice P2 (lowest priority) waits 75: priority scheduling makes the low-priority job pay.

**Understanding: how preemptive priority + RR behaves (rules that confused me)**

When a process ARRIVES, compare it with the running one right away (not at the next timer tick):
- newcomer has HIGHER priority → preempt immediately, even mid-quantum. The interrupted one goes to the end of its own priority's queue.
- newcomer has EQUAL priority → no switch now. It waits; the running one finishes its current quantum, THEN RR switches to it.
- newcomer has LOWER priority → it just waits.

When a QUANTUM ENDS:
- another process of the SAME priority is waiting → switch (running one goes to the back).
- nobody else at that priority → keep running with a new quantum (P1 at 10, P6 at 65).
- (a higher-priority one is never left waiting at this point: it would already have preempted when it arrived.)

Quantum timer: every time a process GETS the CPU (first run, after preemption, or its next RR turn) it starts a FRESH full quantum. An arrival that doesn't cause a switch does NOT reset it (P4 arriving at 25 doesn't change P3 stopping at 30). A preempted process loses the rest of its quantum and gets a new full one next time (P3 at 40, P4 at 50).

Tie at the same instant (quantum ends AND a process arrives, same priority): handle both events first, then pick. Usual convention (same as L05 RR): the new arrival joins the queue first, the preempted process goes behind it, so the newcomer runs next. The question should state it; if not, write the convention you used. (C10 never has this tie: quanta end at 10, 30, 40, 65; arrivals at 20, 25, 45, 55.)

Link to SRTN: same pattern (check on arrival, switch if the newcomer "wins"), only the comparison differs: SRTN compares newcomer's burst vs running job's REMAINING time (which shrinks as it runs); preemptive priority compares fixed priority numbers. SRTN = preemptive priority where priority = remaining time. On a tie SRTN keeps the running job; here the tie waits for the quantum end.

Mixing rules adds them up, they don't cancel: RR only shares the CPU among EQUAL priorities (good response time within a level), but between levels the higher one always wins. So a very long, high-priority job keeps the CPU until it's done: every quantum the timer fires, the scheduler looks again, and picks it again. Lower priorities starve meanwhile (P2 arrived at 0, finished at 95). Fix: aging (raise the priority of processes that have waited long).

**Fun fact: who schedules the scheduler? Do we even need the timer?**

- The kernel is NOT a process waiting in the ready queue (in xv6), so it doesn't need a priority. Kernel code runs whenever a TRAP happens (syscall, interrupt, fault); the hardware forces the CPU into the kernel immediately, whatever user process is running.
- The scheduler itself is kernel code that runs on every timer interrupt and every yield. User processes can't starve it, because the timer interrupt guarantees the kernel gets control back at least once per tick, even if a user program loops forever. The kernel's protection comes from hardware (traps + timer), not from a priority number. (Real Linux also has some kernel threads that are scheduled like processes, but time-critical work happens directly in trap handlers.)
- Mechanism vs policy (prof's review): the timer, traps, and context switch are the MECHANISM (how the kernel takes the CPU and switches). FCFS / SJF / RR / priority are the POLICY (who runs next). Same mechanism, different decision rule.
- Preemptive policies (RR, SRTN, preemptive priority) need the timer (or other interrupts, e.g. an I/O-completion interrupt waking a higher-priority process) to take the CPU away.
- Non-preemptive policies (FCFS, SJF, non-preemptive priority) do NOT strictly need a timer: they only decide when the running process gives up the CPU itself, by exiting or blocking on I/O, and both are system calls (traps) anyway. Real examples: early batch systems; cooperative multitasking in Windows 3.1 and classic Mac OS, where programs had to yield voluntarily.
- The catch without a timer: one program that never yields (infinite loop, bug) freezes the whole machine. That's exactly why modern OSes always have the timer (the L01 "why do we need a timer" answer).

=======================================

C13. You must design a scheduler that minimizes average turnaround time and also minimizes maximum waiting time.

Is there a conflict in achieving both at once? If so, briefly explain.

to min turaound time , and minimiza waiting time

they are not conflicting?
as waiting time minimiszation is
rountime - cpu burst where cpu brust cannot be chaneg
so round time also need to be small

**✗ Check: yes, there IS a conflict.** (Your reasoning used wait = TA − burst for ONE job. But the two goals are about different things: the AVERAGE over all jobs vs the WORST single job.)

- To minimize **average turnaround**, run the shortest jobs first (SJF/SRTN). But then a long job can be pushed back again and again → its waiting time becomes huge (even starvation). Bad for **maximum** waiting time.
- To minimize **maximum waiting time**, be fair: FCFS or RR, so no job waits too long. But then short jobs get stuck behind long ones → the average turnaround goes up.

So improving one hurts the other. Same lesson as the L05 notes: "no ordering wins both".
