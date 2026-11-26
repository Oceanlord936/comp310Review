# Nov 24 lecture - end of L16 (RMS vs EDF) + N17 overview (hand-typed)

class notes : Notes/class23.md

---
##  0. Background: real-time scheduling, the setup for this lecture


What real-time means: 

some tasks must finish before a deadline, every time.

```
soft real-time : missing a deadline = bad quality (video stutters , game lags)
hard real-time : missing a deadline = failure (ABS brakes , robot arm) -> NEVER allowed
```


What a real-time task looks like: 
it's periodic. It shows up again and again, at a fixed rate, and declares its needs up front:

```
p = period   : a new copy of the task arrives every p time units
t = burst    : CPU time it needs each time (the same every time)
d = deadline : must finish before this ; here always d = p (before the next copy arrives)

utilization of one task = t / p      (the fraction of the CPU it needs)

total U = sum of t / p over all tasks

```
Example: T1 (p = 40, t = 20) needs the CPU half the time, U = 0.5.

---

Admission control: 

before accepting tasks, the OS checks whether all deadlines can be met. 
If not, it rejects the new task. It must keep its promise to the tasks already accepted.

The two algorithms:
```
RMS (rate monotonic)       

-> STATIC priority : shorter period = higher priority , fixed forever
                           a higher-priority arrival PREEMPTS a lower one
                           simple , cheap , predictable
						   
EDF (earliest deadline)    
DYNAMIC priority : whoever's deadline is EARLIEST runs , recomputed at every arrival

```

When each one is guaranteed to work:

```
RMS : 

U <= N ( 2^(1/N) - 1 )      N = number of tasks
      N = 1 -> 1.000 , N = 2 -> 0.828 , N = 3 -> 0.780 , ... -> ln 2 = 0.693
      U under the bound -> a schedule DEFINITELY exists
      U above the bound -> MAYBE -> draw the schedule and check
	  
EDF : U <= 1 -> always works

U > 1 : the tasks need more CPU than exists -> NOTHING works (overload)
```

"Guaranteed to work" means you can say yes, every deadline will be met just by computing U, without drawing the schedule. It's a shortcut test.

```
RMS: the bound 


U < N(2^(1/N) − 1)
```

Someone proved (Liu and Layland) that if U is under this number, RMS meets every deadline, whatever the periods are. So:

U <= bound  -> YES , guaranteed . no need to draw
U >  bound  -> the shortcut says NOTHING . RMS might still work , or might not -> draw it to find out


---

Examples from class, with 2 tasks and a bound of 0.828:

```
P1 (50 , 20) , P2 (100 , 35)   

U = 0.75 <= 0.828 -> guaranteed , RMS works

T1 (30,10) T2 (40,10) T3 (60,15)  

U = 0.833 > 0.780 (3 tasks) -> not guaranteed ,
                                  but drawing it shows RMS DOES work (lucky)
								  
								  
P1 (50 , 25) , P2 (80 , 35)   
 U = 0.94 > 0.828 -> not guaranteed ,
                                  drawing shows RMS FAILS (P2 misses at 80)

```

That's why the bound is called sufficient, not necessary: 
being under it is enough to be safe, but being over it doesn't prove failure.

EDF: U ≤ 1

For EDF the test is exact:

```
U <= 1  -> EDF meets every deadline , always
U >  1  -> it can't (and neither can anything else)
```

Why U > 1 is impossible for everyone:
 U is the fraction of the CPU the tasks need. Over 1 means they need more than 100% of the CPU, 
 
 ```
 0 ---------- RMS bound (0.828) ------------------ 1.0 ----------->
   RMS : sure         RMS : maybe (draw it)        nobody can
   EDF : sure         EDF : sure                   (overload)
 ```
 
 ===========================
 
 
 RMS (rate monotonic)

Before running, once:

```
sort the tasks by period : shortest period -> priority 1 (highest) , next -> priority 2 , ...
(optional admission test : U <= N (2^(1/N) - 1) ?)
```

While running:

```
whenever a task ARRIVES or FINISHES :

    among the tasks that still have work in this period ,
    run the one with the HIGHEST (fixed) priority
    if a higher-priority task arrives while a lower one runs -> PREEMPT the lower one
```

idea:

-> Before running (once, for the whole batch of tasks):

```
1. can they be done ?  compute U = sum t/p
       U <= bound -> yes , accept them all
       U >  bound -> not sure -> draw the schedule to check (or reject)
	   
2. sort them by PERIOD -> that IS the priority (shortest period = highest) , fixed forever
```

While running:

```
run the highest-priority task that still has work
a HIGHER-priority task arrives -> it runs first : PREEMPT the running one
the preempted task continues later , from where it stopped
```

========================

EDF, in the same two phases

Before running:

```

1. can they be done ?  compute U = sum t/p
       U <= 1 -> yes , always (no "maybe" zone like RMS)
       U >  1 -> no , reject
	   
2. NO sorting , no fixed priority

```
While running:

```
each copy of a task gets a deadline when it arrives : 

its next period boundary
    (a task with period 40 : copy at 0 -> deadline 40 , copy at 40 -> deadline 80 , ...)
	
run the task whose CURRENT deadline is EARLIEST

a newly arrived task with an EARLIER deadline -> PREEMPT the running one
a newly arrived task with a LATER deadline     -> the running one CONTINUES

```
EDF
before : U <= 1 ? -> yes , the batch can be done
running: always work on the one whose deadline is EARLIEST
         new work arrives with an EARLIER deadline -> switch to it (preempt)
         new work arrives with a LATER deadline    -> keep going with the current one
		 
========================

question: what ensure this is a MUST when the RMS is not ? is that only for U or what???

Answer: 
It is only about U, but why U is enough for EDF and not for RMS comes from how each one chooses.

Why EDF ≤ 1 is a sure thing:

```
EDF always works on the most urgent job.

Suppose some deadline were missed while U ≤ 1. 
Look at the time just before the miss: 

EDF was busy the whole time with jobs whose deadlines were as early or earlier. 
So there was more work due in that window than time available, which would mean U > 1. That's a contradiction.
```

So with U ≤ 1, EDF never wastes time on a less-urgent job while an urgent one waits. 
If any schedule can meet the deadlines, EDF will. That's why EDF is called optimal.

```
Why RMS can fail even with U ≤ 1: 
RMS uses a fixed priority (the period), not the real urgency. 
So sometimes it runs a job that isn't the most urgent:

```



```
EDF : picks by urgency   -> only total demand matters -> U <= 1 is enough
RMS : picks by period    -> order can be wrong -> needs spare room -> U <= bound
```

=======================

is the miss or not the deadline only apply to the one being computed from the goiven task? 
like when u run things already a new task comes , 
there is no garentted to finish that and the new garentted frame need to be recompute ???

Answer: 

Yes, the guarantee only covers the set of tasks that was checked. Two kinds of "new" are different, though:

(1) A new copy of a task already in the set: 
->covered automatically. 

Tasks are periodic, so T1 arriving again at 40, 80, 120 ... is not new work. 

It was already counted in U (that's what t/p means). 
The guarantee covers every copy, forever.

2. A brand-new task that wasn't in the set: 
NOT covered. 

Before accepting it, the OS recomputes U with it included and re-checks. 
That's admission control (slide 37):


```
running : T1 (50 , 10) , T2 (60 , 15)       U = 0.45

T3 (100 , 25) asks to join :  

	U = 0.70 <= 0.780 (RMS , 3 tasks) -> admit

T4 (40 , 10)  asks to join :  
	U = 0.95 >  0.757 (RMS , 4 tasks) -> RMS : no guarantee -> REJECT
                                       
									   0.95 <= 1                 -> EDF : ADMIT
```

======================

##2.     Overload, U > 1: both fail, but differently


The tasks:

```
T1 : period 10 , burst 6      

T2 : period 15 , burst 7

U = 6/10 + 7/15 = 0.6 + 0.467 = 1.07 > 1

-> they need more CPU than exists (1 max)
-> SOME deadline must be missed , by any algorithm
```

The question is no longer "does it work?" but "who misses?"

---

RMS (T1 has the shorter period, so T1 always wins):

```
0  - 6    T1         copy 1 done (deadline 10 ✓)

6  - 10   T2         4 of 7 done

10 - 16   T1         new T1 at 10 preempts T2 ; done at 16 (deadline 20 ✓)

15        T2's deadline : it has only 4 of 7 -> T2 MISSES

```

RMS misses T2, the lower-priority task. T1, the more important one, is protected.
===============

EDF (earliest deadline wins):

given:

T1 : period 10 , burst 6      

T2 : period 15 , burst 7

```
0  - 6    T1   (deadline 10)        ✓

6  - 13   T2   (deadline 15)        at 10 : new T1 deadline 20 > 15 -> T2 keeps going , done at 13 ✓

13 - 19   T1   (deadline 20)        ✓

19 - 20   T2 copy 2 (deadline 30)   runs 1 unit

20        T1 copy 3 arrives , deadline 30 = T2's 30 -> a tie , T2 is running and keeps the CPU

20 - 26   T2 finishes (6 more) ✓ at 26

26 - 30   T1 gets 4 of its 6 -> T1 MISSES at 30

```

EDF gets further (it reaches 30 before missing), 
-> but it misses T1, the more important task.


================================

The lesson:

```
             first miss    who misses
RMS          at 15         T2 - always the LOW-priority task (predictable)
EDF          at 30         T1 - could be ANY task , and one late job makes the next late (domino effect)
```


===========================

undersatdning :

so it is like doing homework assignment . when there are alot , u either do them in order of ealier deadline (EDF) or u do it in whihc worth more , the RMS ... the EDF get u not missing fither but u might miss more important things

That's a good analogy. Two refinements make it exact:

1. In RMS, "worth more" means "comes back more often". Priority = period: a weekly quiz (short period) beats a monthly project (long period), fixed forever. RMS assumes the task that repeats most often is the most important.

2. "EDF doesn't miss" is true only when it's not overloaded:

```
U <= 1 (enough time for everything) : EDF misses NOTHING , RMS might (if above its bound)

U >  1 (too much homework)          : both miss something
       RMS : drops the low-priority work (the monthly project) - you know in advance what suffers
       EDF : keeps up longer , but then can drop ANYTHING , even the important one ,
             and being late on one makes the next late (domino)
```

homework analogy

EDF = do whatever is due soonest -> best use of time , no misses if there's enough time ,
      but when overloaded you might miss the important one
	  
RMS = fixed order by how often it comes (more often = more important) -> wastes time sometimes ,
      but when overloaded the least important one is what you miss
	  
"shorter repeating = more important" is only RMS's assumption.


=============================

3. RMS vs EDF: the summary table (slide 49)

```
                      RMS                                   EDF
priority              STATIC : by period                    DYNAMIC : by the next deadline
                      (shorter period = higher , fixed)     (re-decided at every arrival)

guaranteed when       U <= N ( 2^(1/N) - 1 )                U <= 1
                      above it : maybe , draw to check       (exact , no "maybe" zone)

needs from the task   its period                            its deadline , every time it arrives

run-time cost         very LOW : fixed order ,              HIGHER : a priority queue (heap)
                      just pick the highest                 re-sorted at every arrival
                                                            (fine in practice : few real-time tasks)

under overload (U>1)  the LOWEST-priority tasks miss        ANY task can miss + domino effect
                      (predictable)

optimal among ...     STATIC-priority algorithms            ALL algorithms (on one CPU)
                      (if RMS fails , no fixed-priority
                       order works)

```

Extra notes from class:

The RMS bound comes from a famous paper by Liu and Layland.
Harmonic periods (every period divides the longer ones) let RMS reach 100%. That's not on the exam.
Linux has both: fixed real-time priorities (RMS-style), and an EDF class called SCHED_DEADLINE.


=====================


How to answer an exam question about them:

```
1. compute U = sum of t/p

2. RMS : compare with the bound for N tasks -> under = "guaranteed" ; over = draw it
   EDF : compare with 1
   
3. draw the schedule to the hyperperiod (lcm of periods) if needed :
   RMS -> at each arrival , shortest PERIOD wins
   EDF -> at each arrival , earliest DEADLINE wins
   
4. mark each deadline , say who misses (if anyone) and when
```

=====================

How to draw an RMS schedule, step by step

---

Step 1: list the tasks and their priority.

```
P1 : p = 50 , t = 20      shorter period -> priority 1 (HIGH)
P2 : p = 100 , t = 35                     -> priority 2 (low)
```

Step 2: list every arrival and deadline up to the hyperperiod (lcm(50, 100) = 100):

lcm means "least common multiple"
the smallest number that every period divides into evenly.

-> Why we use it: 

at the lcm, all tasks arrive together again, just like at time 0. 

So the schedule from then on is an exact repeat. 
Drawing from 0 to the lcm (the hyperperiod) covers every possible situation,
 and if every deadline is met there, it's met forever.
 
 A quick way to find it:

->  list the multiples of the bigger period until one is divisible by the smaller:

example : lcm(40 , 60) : 
```
60 (60/40 no) , 120 (120/40 = 3 yes) -> 120
```
so forthe bigger of one , check if that is lcm , if not 2* that bigger ...

Yes. Try the bigger period, then 2×, 3×, ... until the smaller one divides it:

```
lcm(20 , 50) :  50 / 20 = 2.5 no  ->  100 / 20 = 5 yes  ->  lcm = 100
lcm(30 , 40) :  40 no , 80 no (80/30 = 2.67) , 120 / 30 = 4 yes  ->  lcm = 120
```

=============

Step 2: list every arrival and deadline up to the hyperperiod (lcm(50, 100) = 100):


```
P1 copies : arrive 0 , 50     deadlines 50 , 100
P2 copies : arrive 0          deadline 100
```
Step 3: draw a timeline. At every arrival or finish, ask: "who is the highest priority that still has work?"

```
time    event                      who has work         runs

0       P1 , P2 arrive             P1 (20) , P2 (35)    P1   (priority 1)

20      P1 done                    P2 (35)              P2

50      P1 copy 2 arrives          P1 (20) , P2 (5)     P1   (PREEMPTS P2 : higher priority)
70      P1 done                    P2 (5)               P2   (continues where it stopped)
75      P2 done                    nobody               idle
100     end of hyperperiod
```

Step 4: draw it as a bar (time on the axis, one block per run):

```
time   0        20            50        70  75       100
       |   P1   |      P2     |   P1    |P2 |  idle   |
	   
deadlines :              P1 ^50                 P1 , P2 ^100
```

Step 5: check every deadline:
```
P1 copy 1 : done at 20 <= 50 ✓      

P1 copy 2 : done at 70 <= 100 ✓

P2        : done at 75 <= 100 ✓     -> every deadline met
```

The rules to remember while drawing:

(1)Only make a decision when something arrives or finishes.

(2)A preempted task keeps its remaining work and continues later.

(3)If a deadline comes and its task still has work left, that's a miss. Write who and when.