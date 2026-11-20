# L16 Advanced Scheduling + Tutorial 6 - my notes (hand-typed)

slides : lecture/16/N16-Advanced-Scheduling.pdf (taught : slides 2 - 46)
class notes : Notes/class21.md (nov 17) , Notes/class22.md (nov 19) , Notes/tutorial6.md

---

## 1. the job mix

A real workstation does not run one kind of job  . it run a mix , and each kind wants something diff

```
kind             CPU burst      wants
interactive      tiny           fast response (a keystroke must not wait)
  (editor)
  
compute + I/O    medium         CPU and disk both kept busy
  (encoder)
  
batch            long           short turnaround , high throughput
  (compile)

```
The class example has 4 jobs on 1 CPU, all ready at time 0:

```
job   what it does                                    CPU needed

E     editor : 8 keystrokes , 1 CPU then 9 waiting    8
M     encoder : 5 CPU , 5 disk I/O , 4 times          20
B1    compile : 30 CPU , no I/O                       30
B2    simulation : 30 CPU , no I/O                    30
ready queue at 0 : B1 , B2 , M , E


```
Here's the same mix under different policies:


```
policy    keystroke wait (max)   M finishes   B1,B2 avg turnaround   switches
FCFS      65                     95           45                     9
RR q=10   25                     87           67                     13
RR q=4    12                     85           78                     28
RR q=1    3                      79           86.5                   87

```

The  idea:
FCFS is best for the batch job -> 45 unit and terrible for editor , 
one key stroke wait 65 

shrink the quantum helps the editor but hurts the batch jobs 45 -> 86.5 
and the switches go from 9 to 87 

no signle policy can win , whatever helps E hurts B1 , B2 

=========================


2. RR wait , takes (n - 1) times quantum

MLQ 

why RR gets worse as jobs are added . as in RR , a job goes to the tail of the queue 
after its turn . It then waits for everyone ahead of it , and each of them uses a full quantum

```
[ T1 ] [ T2 ] ... [ Tn ]  -> CPU , runs q , back to the tail

wait before running again = (n - 1) * q

```

the editor's 1 unit of work waits behind the full quanta of the batch jobs 
```
batch jobs ahead   q     keystroke waits

4                  10    40
19                 10    190
99                 10    990
19                 1     19       (smaller q helps , but more switching)
19 , own queue     10    0        (batch jobs don't matter any more)

```

There are two ways to shrink (n − 1) × q:

(1) make the q smaller -> costs switching more . if a wsitch cost 0.1 
the overhead is now 0.1 / 10.1 close to 1 % . at q = 10 (larger q  low percent wasting on switch)

and 0.1 / 1.1 clsoe to 9 % at q = 1 (q is small , more percent wasting on just switching )

===========================

MLQ (multilevel queue) takes the second option. It uses one queue per class of job:

```
highest   
		  real-time processes
          system processes
          interactive processes     (editor)
		  
lowest    batch processes

```

(1)Within a class: 
RR, or another policy (batch can use FCFS).

-> A "class" is a group of jobs of the same kind. Each class gets its own queue:

```
class            jobs in it          its queue
interactive      E (editor)          Q0
compute + I/O    M (encoder)         Q1
batch            B1 , B2             Q2

```



(2)Across classes: strict priority, so a higher queue preempts a lower one. 
Another option is time-slicing between the queues, for example 80% interactive and 20% batch.

(3)No feedback: a job gets its class when it starts and keeps it for life. 
Someone has to tell the scheduler the class: the user, the admin, or the program.


(4) From class: higher queues are expected to have jobs rarely, 
and only short bursts. Otherwise they would take over the machine.

Here's the job mix under MLQ, 

with Q0 = E (RR q = 1), Q1 = M (RR q = 5) and Q2 = B1, B2 (FCFS):

```
policy    keystroke wait (max)   M finishes   B1,B2 avg   switches

FCFS      65                     95           45          9
RR q=1    3                      79           86.5        87
MLQ       0                      36           72          20

```

```
MLQ = sort jobs into classes by a LABEL , one queue (level) per class
      levels are ordered by priority : the higher level runs first
      inside each queue : its own policy (usually RR for short bursts , FCFS for batch)
      a job NEVER changes level -> no feedback
```

============================


3. The label problem → MLFQ rules
The catch with MLQ is that labels can be wrong:

```
mistake                    what happens
E labelled batch           first keystroke waits 80 (worse than FCFS)
M labelled batch           M finishes at 103 instead of 36
B1 labelled interactive    B1 holds Q0 for 30 units , M waits 34 for its first turn
```


Jobs also change. A compile can turn into an interactive debugging session and still be stuck in the batch class.
With strict priority, a steady stream of interactive work starves the batch class.
So we want the scheduler to find the class by itself.


---

fix MLFQ (multilevel feedback queue) 
places jobs by what they do (their CPU burst sizes), not by a label:


```
new job --> [ Q0 : RR q = 8  ] --used whole quantum--> 
            [ Q1 : RR q = 16 ] --used whole quantum--> 
            [ Q2 : FCFS      ]

```
It estimates as it goes. The queue a job is in is the scheduler's guess of its burst length:

```
queue   quantum   a job settles here if its bursts are
Q0      8         <= 8
Q1      16        9 to 16
Q2      FCFS      longer than 16

```

A new job starts in Q0, so the first guess is "short".

If it uses its whole quantum, the burst was longer than guessed, so it moves down.
If it gives up the CPU early, the guess was right, so it stays.


The rules:

```
Rule 1 : a higher queue always runs first
         a job arriving in a higher queue PREEMPTS a lower one
Rule 2 : inside a queue , RR with that queue's quantum (last queue FCFS)
Rule 3 : a new job enters the top queue Q0
Rule 4 : uses its whole quantum         -> moves DOWN one queue
         gives up the CPU early (I/O)   -> STAYS in the same queue
Rule 5 : moves UP only by priority boost or aging

```

No bubble up: a job never climbs back up by itself.
Preempted by a higher queue:
The slide says the job goes back to the front of its queue, and its slice starts over.
On Nov 19 the prof said end. That's still open; ask which one the exam uses.

======================

4. MLFQ example + starvation + boost / aging

Assume example :
```
Q0 has q = 8 and Q1 has q = 16. 

A is CPU-bound. 
B is interactive: 2 CPU, then 6 I/O, repeating.


A  0 - 8    Q0 , used all 8 -> drops to Q1
B  8 - 10   Q0 , used 2 < 8 -> stays in Q0 , goes to I/O until 16
A  10 - 16  Q1
B  16 - 18  back from I/O , in Q0 -> preempts A (Rule 1)
A  18 - 24  Q1
... repeats every 8

```
Notice: B never uses a whole quantum, so it stays in Q0 and never waits again.
MLFQ found that B is interactive with no label, and that A is not.


but issue can happens:
Starvation. If enough jobs live in Q0, they keep it busy all the time, and nothing below ever runs:

```

B and C : 4 CPU , 4 I/O , forever , both in Q0
B C B C B C ...     A (in Q2) gets 0 CPU

```
Notice the issue : Nothing in Rules 1–4 can save A, because there's no bubble up.

Need rule 5 to fix this 
There are two fixes:

```
boost : every S units , ALL jobs (from every level) jump straight to Q0
        same time for everyone , no matter how long each one waited

aging : per job , count how long it has WAITED in its current queue
        waited W -> move up ONE level , the wait count starts over
        it can keep climbing one level at a time (Q2 -> Q1 -> Q0)

```

=====================================

5. Gaming + total-use accounting


Can a CPU-bound job pretend to be interactive? 
Job G runs 7 units, then does a tiny dummy 1-unit I/O just before its quantum of 8 runs out:

```
G 0-7 | A 7-15 | G 7 | A 1 | G 7 | A 1 | G 7 | A 1 | ...

```
By Rule 4, G gave up the CPU early, so it stays in Q0 and its quantum restarts.
G gets 7 of every 8 units, and A only runs while G is in its 1-unit I/O.

The trick works because Rule 4 only looks at one continuous burst.


The fix: count the total CPU a job uses at a level, not per burst.

```
quantum   = length of ONE turn , restarts every turn

allotment = TOTAL CPU allowed at a level , adds up across turns
            I/O does NOT reset it , only changing queue resets it
            allotment used up -> down one level

```

eg:

```
G in Q0 : 7 + 1 = 8 total          -> allotment 8 used  -> Q1

G in Q1 : 6 + 7 + 3 = 16 total     -> allotment 16 used -> Q2

```

-> The rule changes from "used a whole continuous quantum" to "used its total allotment".

=========================

6. PCS vs SCS + multi-core queues + push / pull

Thread scheduling. 

The kernel schedules kernel threads.

 User threads are managed by a library, and the kernel doesn't see them:

(Many to one )

```
user threads       ~  ~  ~  ~
                    \  \ /  /      <- PCS : the library picks which user thread runs on an LWP
LWPs (kernel)        ~     ~
                      \   /        <- SCS : the kernel picks which kernel thread runs on the core
core                  [core]

```

PCS (process contention scope): 
-> threads compete only with threads of the same process.

SCS (system contention scope): 
-> threads compete with all threads in the system.

Notice: 
In a many-to-many or many-to-one setup, both happen at once, at different layers:

```
layer 1 (library , PCS) : process A has 4 user threads , 2 LWPs
                          the library picks which 2 of A's 4 threads sit on A's LWPs
                          -> A's threads compete ONLY with each other (same process)

layer 2 (kernel , SCS)  : the kernel sees LWPs from ALL processes (A's 2 + B's 1)
                          it picks which LWP runs on the core
                          -> LWPs compete with every other kernel thread in the system

```

what is lwp ???

An LWP (lightweight process) is a kernel thread that the thread library uses to run user threads. 
Think of it as a virtual CPU that the library hands out to its user threads:


```

user threads      T1   T2   T3   T4        (the kernel does NOT see these)
                    \   |    |   /
                  library picks 2 of them
                      |        |
LWPs               LWP1     LWP2           (the kernel DOES see these)
                      \      /
                   kernel schedules
                        |
                      core

```

wait  , what is the point of this LWP1 and LWP2 rather than let keernal direcly see the 4 thread ???

answer:

 it's the same trade-off as many-to-one vs one-to-one in L13
 
 Why not let the kernel see all 4 (one-to-one)? Kernel threads used to be expensive:



```
                       user thread (library only)   kernel thread / LWP
create                 cheap , no system call        system call + kernel memory
switch between them    cheap , a function call       goes into the kernel (trap)
how many you can have  thousands                     limited by kernel resources

```
Why not just 1 LWP (many-to-one)? With only one kernel thread:

if any thread blocks in a system call, the whole process blocks
it can't use more than one core

Many-to-many, with a few LWPs, was the middle ground:

4 user threads on 2 LWPs :
- create / switch user threads cheaply (no kernel)
- 2 can run on 2 cores at the same time (parallelism)
- if one blocks , the other LWP keeps the process going
- the kernel only has to manage 2 , not 4 (or 4000)

---

First, a fix: an LWP is not attached to a kernel thread. 
An LWP is one kernel thread; it's just the name for a kernel thread when the thread library uses it:


```
user threads   T1 T2 T3 T4      (many , cheap , kernel can't see)
                  |
LWP  =  kernel thread           (few , costly , kernel sees)
                  |
core

```
Now, 1 LWP vs 2 LWPs, with the same 4 user threads:

```
                        4 user threads , 1 LWP        4 user threads , 2 LWPs
how many run at once    1  (kernel sees 1 thing ,     2  (kernel can put LWP1 on core 0 ,
                            can only use 1 core)          LWP2 on core 1)
T1 makes a blocking     LWP blocks -> T2 T3 T4        LWP1 blocks , LWP2 still runs
system call (read)      ALL stuck , whole process     T2 -> process keeps going
                        waits
CPU share vs other      kernel sees 1 -> small        kernel sees 2 -> bigger share
processes (PCS)         share                         (the 2/3 in the table)
cost                    pay for 1 kernel thread       pay for 2


The difference: the number of LWPs is how many of your threads the kernel can run or block at the same time.


1 LWP   = many-to-one  : cheapest , but no parallelism , one block stops everyone
2 LWPs  = many-to-many : costs a bit more , but 2 cores + survives one block
4 LWPs  = one-to-one   : most expensive , every thread is independent

```
---

ok , so they goes back to the sheculing question of this lecture . 
why should i care . 
is it casue the so called schdeuling can only work at the LWPS level not user thread level ???

```
Yes, that's exactly the link. Every scheduler in this lecture (RR, MLFQ, boost, multi-core queues) 
is the kernel's scheduler, 
and the kernel can only schedule what it sees: 

kernel threads, which are the LWPs
```
Why you should care:

The kernel can't help a user thread directly. If T3 is an interactive thread, MLFQ can't put it in Q0, because it doesn't know T3 exists. It only sees the LWP that T3 happens to be sitting on.

Fairness is decided by the number of LWPs, not threads. This is the PCS vs SCS table:

```
A : 4 threads on 2 LWPs , B : 1 thread on 1 LWP

kernel splits the CPU by what it sees -> A 2/3 , B 1/3

then A's library splits A's 2/3 among its 4 threads -> 1/6 each

```
A priority set on a user thread only matters inside its own process (PCS). 
It never competes with other processes.

=======================

7. Real-time: soft / hard, latency, task spec, admission control

```
soft real-time   multimedia , games , not mission-critical
                 real-time tasks get priority over the others
                 missed deadline -> bad quality (lag)

hard real-time   safety critical (ABS brakes , robots)
                 NO deadline may be missed

```

Event latency is the time from an event happening 
(eg the ABS sensor fires) to the system responding. ABS needs 3–5 ms.

```
event --interrupt latency--> ISR starts --dispatch latency--> real-time task runs

interrupt latency : the kernel may MASK interrupts while doing something critical
                    -> must be very short (xv6 masks while holding any spinlock)
dispatch latency  : time to stop the running process and start the real-time one
                    -> preemptive kernels keep it minimal

```

Task spec. Real-time tasks must say up front what they need:

```
p = period     (the task shows up every p units)
t = CPU burst  (the same every period)
d = deadline   (usually d = p : finish before the next one arrives)

0 <= t <= d <= p
utilization of one task = t / p
total U = sum of t_i / p_i

```

admission control . a new task is only accepted  if the whole set still meets every deadline

```
p = period     (the task shows up every p units)
t = CPU burst  (the same every period)
d = deadline   (usually d = p : finish before the next one arrives)

0 <= t <= d <= p
utilization of one task = t / p
total U = sum of t_i / p_i

```

=================================

8. RMS (rate monotonic scheduling) + the bound

RMS uses static priorities with preemption:

```
priority = 1 / period  -> SHORTEST period = HIGHEST priority
computed ONCE , never changes
a higher-priority arrival kicks out a lower one that is running

```
Why: 
	a task that shows up often needs attention often. A shorter period also means an earlier deadline.

Easy to implement: just switch by fixed priority, with no need to look at deadlines.

Optimal among static-priority algorithms: 
if RMS can't schedule a set, no static-priority algorithm can. 
A dynamic one (EDF) still might.

The bound:

```
bound = N ( 2^(1/N) - 1 )        N = number of tasks

N     1       2       3       4       ...   inf
      1.000   0.828   0.780   0.757         ln 2 = 0.693

U <= bound   -> ALWAYS feasible (guaranteed , no drawing needed)
U >  bound   -> MAYBE -> draw the schedule and check
U >  1       -> impossible for any algorithm

```
explain : 
RMS has one job: run a set of periodic real-time tasks 
so that every copy of every task finishes before its deadline.
 It does it with the simplest possible rule.

The situation

```
P1 : every 50 units , a new copy of P1 shows up and needs 25 units of CPU ,
     it must finish before the next copy comes (deadline = 50 , 100 , 150 ...)
	 
P2 : every 80 units , needs 35 , deadline 80 , 160 ...
one CPU

---------------------------------------------------------------

step 1 , BEFORE running (admission) :
   compute U = sum t/p , compare with the bound
   U <= bound -> "I promise every deadline will be met" -> accept the tasks
   U >  bound -> no promise -> draw it to check , or reject

step 2 , WHILE running (the rule) :
   give each task a FIXED priority : shorter period = higher priority
   at any moment , run the highest-priority task that has work left
   a higher-priority task arriving preempts a lower one


```

RMS is for a fixed, known set of tasks that repeat forever at a steady rate

===============================

9. EDF (earliest deadline first)

EDF uses dynamic priorities:

rule:

```
(1) priority = whoever's deadline is EARLIEST , right now

(2) recomputed every time a task arrives

(3) a task with an earlier deadline arriving preempts the running one

(4) feasible whenever U <= 1  
-> can use 100 % of the CPU

```
Each task must announce its deadline every time it becomes ready.

---

The set where RMS failed: P1 (50, 25) and P2 (80, 35), U = 0.94.

```
time   event            deadlines of ready jobs   runs
0      P1 , P2 arrive   P1: 50    P2: 80          P1
25     P1 done          P2: 80                    P2
50     P1 arrives       P1: 100   P2: 80          P2 - NOT preempted
60     P2 done          P1: 100                   P1
80     P2 arrives       P1: 100   P2: 160         P1
85     P1 done          P2: 160                   P2
100    P1 arrives       P1: 150   P2: 160         P1 preempts P2
125    P1 done          P2: 160                   P2 , done at 145

```


```
              RMS                        EDF
priority      static , by period         dynamic , by next deadline
feasible if   U <= N(2^(1/N) - 1)        U <= 1
needs         each task's period         each deadline , every arrival
cost          simple , fixed priorities  re-sort at every arrival
```


both are for tasks with strict deadlines.

"Fail" means a deadline is missed: some copy of a task hasn't finished its CPU work by its deadline.





















