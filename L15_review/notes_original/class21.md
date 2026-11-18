## lecture 21 The scheduling Problem 

---
What are we trying to optimize?

• A workstation does not run one kind of job - it runs a mix

• Interactive (editor, shell): tiny bursts, then wait for the user

Batch (compile, simulation): long bursts, nobody waiting
• Compute + IO (media encoder, database): medium bursts, alternate with the disk
• A policy good for one kind can be bad for another - the best scheduler depends on the job 
mix

---

For different type of job:

Interactive -> short CPU burst -> wants responsive 

compute + IO -> cpu medium burst -> wants cpu and disk both busy 

batch work -> long cpu burst -> want to shorten turnaround , throughtput 

============================

A Workstation Job Mix: Example


eg:

for jobs , one cpu , already at tiem 0 

B1 B2 where staetd first . B1 does compile , B2 does simulation 

both 30 units of cpu , no IO , need 30 cpu 


B1 and B2 were started first - the ready queue at 0 is B1, B2, M, E
CPU needed
8
20
30
30
• What we measure: how long a keystroke waits for the CPU, when M finishes, the average turnaround 
of B1 and B2, and the number of context switches
• Same jobs on every policy - only the scheduler changes

---

One Policy for Everyone ?? 

Notice : FCFS is best for the batch jobs (45) - and terrible for the editor: a keystroke waits 65

FCFS and RR, Side by Side
Where does the editor wait?

NOtice: 
(1) FCFS: E’s first keystroke waits for both batch jobs - 65 units. Gaps after 60: everyone left is waiting for I/O

 (2)RR, q = 1: E never waits more than 3 - but the CPU changes jobs 87 times, and B1, B2 now finish near 86

---

Queue Size a Problem

Why does RR get worse as jobs are added????

 Round Robin: Very simple idea, but the 
problem is a job needs to wait at most (n − 
1) × q time units before running again

However : 
Every job added to the queue adds q to 
everyone’s wait

The interactive job waits in the same line as 
every batch job - its 1 unit of work waits 
behind their full quanta

```
tail: Tn arrived

n − 1 jobs ahead of it, q each → wait (n − 1) × q
```

Can we reduce the number of jobs in the 
queue?

---

RR Waiting: The Numbers
How long does a keystroke wait behind n jobs?

```
Make q smaller? 

The wait shrinks - but the CPU switches 
more often. 

Say a switch costs 0.1: 

overhead is 0.1/10.1 ≈ 1% at q = 10

0.1/1.1 ≈ 9% at q = 1


```

The real problem: the keystroke waits in the same queue 
as every batch job


Give interactive jobs their own queue, above the batchqueue: the keystroke waits 0 - with 19 or 99 batch jobs


====================

idea:  Multilevel Queue Scheduling

One queue per class - no feedback

Not put all the jobs in a single queue - classify the jobs into 
classes of equal priority

Run RR within each class - one queue for a class (or another 
policy: batch can be FCFS)

 Priority across the classes - a higher queue preempts a lower 
one

Or time-slice across the queues - e.g., 80% of the CPU for 
interactive, 20% for batch

 No feedback: a job is put in a class when it starts - and stays 
there for life

 Someone has to tell the scheduler the class - the user, the 
administrator, or the program


============================

Multilevel Queue: The Job Mix

Top run RR , low run FCFS : 

mid also use RR 

3 Level of queue 

----


Q0 interactive, RR q = 1: E. Q1 compute + IO, RR q = 5: M. Q2 batch, FCFS: B1, B2

• E never waits. M finishes at 36. The batch jobs pay: 72 instead of 45 - but better than RR q = 1


Notice:

```
for FCFS , key schroke wait max is 65 , , M finished at 95 , B1 B2 avg turnaround 45 , context switche 9 .

for RR , q =1 ,   keystroke wait is 3 , M finishes 79 , B1 , B2 avg 86.5 , context switches 87 

MLQ , keystrok 0 , M fisniehd 36 , B1 B 2 avg turnaround 72 , context switch 20 

```

The Catch: Labels
-> What if a job is in the wrong class?

 E labelled batch by mistake: its first keystroke waits 80 - worse than FCFS
• M labelled batch: M waits 67 for its first turn, finishes at 103 instead of 36
• B1 labelled interactive: B1 holds Q0 for 30 units - M waits 34 for its first turn
• Jobs change: a compile that turns into an interactive debugging session is still in the batch class
• With strict priority, a steady stream of interactive work starves the batch class



=============================

MLFQ: The Job Mix, No Labels
Can the scheduler find the classes by itself?

• Nobody labels anything - all four jobs start in Q0. Quanta: Q0 2, Q1 8, Q2 FCFS

B1 and B2 use whole quanta → both are down in Q2 by time 25. M’s bursts (5) are too long for Q0, fit in Q1 → M 
settles in Q1. E’s bursts (1) → E stays in Q0

• The cost of learning: E’s first keystroke waits 6, M finishes at 62 instead of 36


