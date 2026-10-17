# Questions_2 — Part C: CPU scheduling (p6–7)

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
