## Deadlock prevention

---

1. Deadlock prevention

The idea: a deadlock needs all 4 conditions at once. 
Prevention is a rule fixed in advance that makes one condition impossible, so a deadlock can never happen.

(1) Mutual exclusion

To break it, make resources shareable.
It usually can't be broken. A printer can't print two jobs at once.

(2) Hold and wait

Rule: a process asks for all its resources at once, and waits until it can get all of them together.
Then it never holds some while waiting for more.
Cost: it's slow to start, and resources sit unused while the process doesn't need them yet.

(3) No preemption

Rule: if a process is refused a request, it gives back everything it holds and asks again for all of it.

Cost: work gets redone, and a process can keep restarting forever.
It only works for resources whose state can be saved and restored, like CPU or memory. It doesn't work for a printer.

(4) Circular wait

Rule: number the resource types (printer = 1, scanner = 2). A process may only ask in increasing order.
A cycle would need someone to ask "backwards", which the rule forbids.

Cost: you can't ask for resources in the order you actually use them.
Example: "always grab the printer before the scanner", even if you scan first.

---

2. Trajectories, and safe vs unsafe

The picture , 2 processses , 1 printer , 1 scanner , draw 2D graph 

The system is a point that only moves right (P1 runs) or up (P2) runs 
-> one on X axis one on Y axis 

Impossible  (X mark)  : Both would hold the same resource at the same time .

if both process grab in the same order (Printer , then sacnner ) 
-> you can always go around the X block , so NO Deadlock


If they grab in different orders (P1 printer first, P2 scanner first):

(1) all reachable cell U , appears , where P1 holds the pointer and P2 holds the scanner 

(2) from U , any move is blocked , so deadlock is certain 

(3) U is "unsafe" : Not Deadlock yet , but Nothing can avoid it 

Avoidance: 
refuse the move form safe -> to unsafe . In this example , when P1 holds the printer , refuse to gove 
the scanner to P2 , and P2 waits ..

The three words:

Safe: some order still lets everyone finish.
Unsafe: deadlock may come (and here it's certain).
Deadlock: already stuck.

---

eg: Here are the two trajectory graphs:


Same order (no deadlock):

```

P1 : GP  GS  RP  RS
P2 : GP  GS  RP  RS

 P2
  ^
RS|  .    .    .    .    .
RP|  .    .    X    X    .        X = impossible (both hold same resource)
GS|  .    X    X    X    .        . = reachable
GP|  .    X    X    .    .
  +--------------------------> P1
     0    GP   GS   RP   RS


```

Different order (unsafe cell U):

```
P1 : GP  GS  RP  RS
P2 : GS  GP  RP  RS      <- P2 grabs scanner first

 P2
  ^
RS|  .    .    .    .    .
RP|  .    .    X    X    .
GP|  .    X    X    X    .        U = unsafe (reachable, deadlock certain)
GS|  .    U    X    X    .
  +--------------------------> P1
     0    GP   GS   RP   RS

at U : P1 holds printer , P2 holds scanner
  go right -> P1 wants scanner -> blocked
  go up    -> P2 wants printer -> blocked
```

---

How to draw it yourself ???

Step 1. 
Put P1's steps on the x axis and P2's steps on the y axis.
A point (x, y) means P1 has finished its first x steps and P2 has finished its first y steps. 
0 means it hasn't started.

Step 2. 
For each position, write down what that process HOLDS right after that step.
A resource is "held" from just after its get until its release.

```
P1 : GP  GS  RP  RS

P1 position :  0    1(GP)   2(GS)   3(RP)   4(RS)
P1 holds    :  -    P       P,S     S       -

P2 : GS  GP  RP  RS

P2 position :  0    1(GS)   2(GP)   3(RP)   4(RS)
P2 holds    :  -    S       S,P     S       -

```

Step 3. 
Check every cell: do they share a resource?
If both hold the same resource at that point, 
the cell is X (impossible). 

Each resource has 1 unit, so two holders can't happen. 
Otherwise it's . (reachable).

```
cell (x=2, y=1) : P1 holds P,S   P2 holds S    -> both hold S  -> X
cell (x=1, y=1) : P1 holds P     P2 holds S    -> nothing shared -> .
cell (x=3, y=3) : P1 holds S     P2 holds S    -> both hold S  -> X
cell (x=1, y=3) : P1 holds P     P2 holds S    -> nothing shared -> .

```

Step 4. Look for unsafe cells.
An unsafe cell is a reachable cell (not the finish) where both moves go into X:

right = P1 does its next step
up = P2 does its next step

```
cell (1,1) : right -> (2,1) X   up -> (1,2) X   -> stuck = U (unsafe)
```

The result:

```
 P2
  ^
RS|  .    .    .    .    .
RP|  .    .    X    X    .
GP|  .    X    X    X    .
GS|  .    U    X    X    .
  +--------------------------> P1
     0    GP   GS   RP   RS

```
=================================

3. Claim edges (resources with a single unit)

idea: each process delcares ahead of time , which resoruces it "Might" ask for.
The OS uses these claims to see trouble coming before teh requests arrive

3kind of edges : 

```
P ......> R   claim edge      (dotted)  P MAY ask for R later
P - - - > R   request edge    (dashed)  P is asking for R now
P <------ R   allocation edge (solid)   R is given to P

claim -> becomes request -> becomes allocation

release -> allocation turns back into a claim

no claim = request is illegal

```

The rule : grant a request only if turning it into an "allocation" does Note make a cycle

"Claim edges" count as part if the cycle 

eg: (printer, scanner, and both processes claim both):

```
P1 holds printer :   printer -> P1   (printer is resource , allocated to P1 )


P2 asks scanner  :   would give   scanner -> P2    (scanner is also a resource , allocated to P2 )



check the cycle (with claims) :

	scanner -> P2 ...> printer -> P1 ...> scanner     !!! CYCLE -> refuse , P2 waits

P1 asks scanner  :   scanner -> P1 ,        no cycle -> grant !

P1 finishes, releases both -> now P2 can go

```

Question: 
when we say there is a cycle occure ,
 including the claim edges . and the edge we need to refuse is the new added one that cause the cycle???
 
 
 Answer: 
 
 Yes. You only ever decide about the new request:

```
(1)Pretend to grant it: request → allocation.

(2)Look for a cycle, counting claim edges.

(3)Cycle: refuse that request. It goes back to being a request, and the process waits.

(4)No cycle: grant it. It stays an allocation.

```

===================================


4. Banker's algorithm, one resource type (Tutorial 5, question 1)

idea: grant a request only if , afterwards , there is still some "order
in which every processes can finish 

A finished process gives back -> everything it holds 

The 4 words:

```
Max :   the most units a process will ever use ( agiven info)

Hold : the units it has now 

Need : calcualted , MAX - HOLD   (what it may still ask for )

Avail : Total - Sum of Hold   (the free resource now )

```

---

The question:

```
12 identical units , P1 to P4

        Max   Hold
P1       8     4
P2       5     2
P3       9     3
P4       3     1


```
quetsion :

(a) Is it safe?

step 1 . check the NEED , Avail  of each process 

recall , total Avail = MAX (sum of ) - HOLD (sum of hold)



 ```
         Max   Hold   Need
P1       8     4      4
P2       5     2      3
P3       9     3      6
P4       3     1      2

sum of Hold = 4 + 2 + 3 + 1 = 10
Avail       = 12 - 10 = 2

 
 ```

Step 2: the safety check. 
repeat , find somone with "need <= avail " 
let them finish , then their hold resource can be released and 
avail += their hold 

```
start                                  avail = 2
P1 need 4 <= 2 ? no
P2 need 3 <= 2 ? no
P3 need 6 <= 2 ? no

avail +XXX (release the hold after done )

P4 need 2 <= 2 ? yes   + hold 1   								      avail = 3
P2 need 3 <= 3 ? yes   + hold 2        								  avail = 5
P1 need 4 <= 5 ? yes   + hold 4                                        avail = 9
P3 need 6 <= 9 ? yes   + hold 3        avail = 12   = total , all done

SAFE , order : P4 , P2 , P1 , P3

```

!!! Why add the hold, not the need ???

P4 takes its 2 (avail 2 → 0) and now holds 1 + 2 = 3, its max. 
It finishes and returns all 3 (avail 0 → 3). 
The net effect is avail + "old hold"

Check: at the end avail must equal the total (12). If it doesn't, you made a math mistake.

---

Handling a request: 4 steps

```
Process i asks for resoruce r  :

(1) r <= Need[i] ?   no -> ILLEGAL (more than it said)

(2)  r <= Avail ?     no -> WAIT (not enough free now) , stop here

(3) pretend : Avail -= r , Hold[i] += r , Need[i] -= r

(4) safety check : safe -> GRANT , unsafe -> UNDO , Pi WAITS

```

Notice: 
Max, Hold, Need  -> are one value per process, one row each.
"Avail"  is one value for the whole system: 
	how many units are free right now. It doesn't belong to any process.

Avail is calculated from the sum of holds, but it isn't "a sum" itself:
```
Avail = Total - (sum of everyone's Hold) = 12 - 10 = 2

```
---

(b) P2 asks for 1 unit of resoruce

```
 1 <= Need P2 (3)  ok



 1 <= Avail (2)    ok
 
 pretend :
 
 
 
        Hold   Need
P1       4      4
P2       3      2        <- was 2 / 3
P3       3      6
P4       1      2
Avail = 2 - 1 = 1

4. safety check , avail = 1 :
P1 need 4 no , P2 need 2 no , P3 need 6 no , P4 need 2 no
nobody fits -> UNSAFE -> UNDO , P2 WAITS

```

In my own words :

```
P2 , ask for 1 . we look if that works . as we calculate dbefore , Avail is 2 , and ask the need of P2 , which is 3 needs max it can ask for , it is also ok . pass these 2 check , reduce Avail from 2 to 1 . 
now keep check . P1 need 4 cannot work as 4 < 1 avail , P3 6 < 1 , P4 2 < 1 avail . so we run into no other process after it is safe . so this allow P2 take 1 fail
```

clean verison :
(b) P2 asks for 1
```

1. 1 <= Need P2 (3)    ok
2. 1 <= Avail (2)      ok
3. pretend : Avail 2 -> 1 , P2 Hold 2 -> 3 , P2 Need 3 -> 2
4. check with Avail = 1 :
   P1 need 4 > 1 no
   P2 need 2 > 1 no
   P3 need 6 > 1 no
   P4 need 2 > 1 no
   nobody fits -> UNSAFE -> undo , P2 WAITS

```

---

(c) P4 ask for 1 unit of resource , is this okay ? 

```
recall , avail =2 form start . 
P4 need max is 2 , so ask 1 is fine . ask 1 also < avail = 2 , also okay . 
reduce avail -> 2 -1 = 1 

now check next 3 process:
P1 need 4 > 1 , no 
P2 need 3 > 1 , no 
P3 need 6 >1 no 

// keep checking p4 -> so it can be finised and release the old hold 

P4 need 1 <= 1  yes   + hold 2        avail = 3
P2 need 3 <= 3  yes   + hold 2        avail = 5
P1 need 4 <= 5  yes   + hold 4        avail = 9
P3 need 6 <= 9  yes   + hold 3        avail = 12   all done

```

---

(d) P3 asks for 3

```
check 3 <= Need P3 (6)    ok   (legal)
check  3 <= 2 Avail ?  that is no , refuese 


what to do : -> P3 WAITS , stop here (no pretend , no safety check)

```

=====================

5. Banker's algorithm, several resource types (Tutorial 5, question 3)

What changes: every value becomes a vector, with one number per resource type (A, B, C).

```
Need_i <= Avail   means EVERY column : A <= A , B <= B , C <= C

one column too big -> that process does not fit

add / subtract column by column

```

Everything else is the same as one resource type: 
the same 4 request steps, and the same "avail += hold" when a process finishes.

The question:

```
Total = (A B C) = (6 8 10) , the give resoruces , we have 3 type of resoruces A , B , C 

Notice: for the MAX , HOLD , NEED , and avail , the table is now one type of resource each column 

           Max            Hold
process  A   B   C      A   B   C
P1       3   2   2      2   0   2
P2       1   2   4      1   0   2
P3       3   6   6      0   2   4
P4       6   6   3      1   1   1
P5       4   3   5      1   3   0


```

step 1. need = max - hold of each type , do it row by row , column by column

this gives the need table 
```
           Need                 
process  A   B   C
P1       1   2   0      (3-2 , 2-0 , 2-2)
P2       0   2   2
P3       3   4   2
P4       5   5   2
P5       3   0   5

```

Step 2: avail = total − allocation. Allocation is the sum of each hold column:

Notice: now the Avail is not a single number as we have many types of resources


```
for each type , check the "allocation" , which is hold sum for that column 
allocation  A = 2+1+0+1+1 = 5
            B = 0+0+2+1+3 = 6
            C = 2+2+4+1+0 = 9

total       (6 8 10) given info from start 
allocation -(5 6  9)
avail      =(1 2  1)


```

Step 3: safety check. Need ≤ avail must hold in all 3 columns:

```
start                                          avail = (1 2 1)
P1 (1 2 0) <= (1 2 1)  yes  + hold (2 0 2)     avail = (3 2 3)
P2 (0 2 2) <= (3 2 3)  yes  + hold (1 0 2)     avail = (4 2 5)
P3 (3 4 2) <= (4 2 5)  no , B 4 > 2
P4 (5 5 2) <= (4 2 5)  no , A and B too big
P5 (3 0 5) <= (4 2 5)  yes  + hold (1 3 0)     avail = (5 5 5)
P3 (3 4 2) <= (5 5 5)  yes  + hold (0 2 4)     avail = (5 7 9)
P4 (5 5 2) <= (5 7 9)  yes  + hold (1 1 1)     avail = (6 8 10)  = total

SAFE , order : P1 , P2 , P5 , P3 , P4

Check: avail ends at (6 8 10), the total. so this is correct 


```

---

(b) P5 asks for (0 0 1). Use the same 4 steps, and remember to check P5 itself with its new need.


my first issue try 
```
as P5 ask for (0  0  1)


check again its need -> (3 0 5 ) that is legal OK 

check Avail , (0 0 1) <= (1 2 1) OK 
now we have (1 2 0 ) for avail 

check other process
P1 (1 2 0) <= ( 1 2 0 ) Ok , give that P1 , then give back old hold and (12 0)+ (2 0 2) = (3 2 2) avail 

check P2
p2 need (0 2 2) <= (3 2 2) , can give that P2 , ( 3 2 2 ) + (1 0 2) = (4 2 4)

check P3 need (3   4   2) <= (4 2 4 ) ?  NO , wait 

check P4 need ( 5   5   2) <= (4 2 4) ? NO , wait 

check P5 need remain (3 0 4) <= (4 2 4) , OK . take that P5 . 

	now avail is :  (4 2 4) + (1 3 0) = (5 5 4) <- Problem here ! as P5 prevoussly hold +1 , reseal it as well so + (1 3 1 )
	
check remain p3 and p4 

P3 need (3   4   2) <= (5 5 4) , OK . 
	now avail:  (5 5 4) + ( 0   2   4) = (5 7 8 )
	
oly P4 retemain 

P4 need ( 5   5   2) <= (5 7 8 ) , OK 

now avail (5 7 8 ) + (1 1 1 ) = (6 8 9 ) 

wait , we start with (6 8 10)  where is some missing  

```

fixed :

```
(b) P5 asks for (0 0 1)

1. (0 0 1) <= Need P5 (3 0 5)   ok
2. (0 0 1) <= Avail (1 2 1)     ok


3. pretend : Avail (1 2 1) -> (1 2 0)
             P5 Hold (1 3 0) -> (1 3 1)
             P5 Need (3 0 5) -> (3 0 4)
			 
			 
4. check :
   P1 (1 2 0) <= (1 2 0)  yes  + (2 0 2)  avail = (3 2 2)
   P2 (0 2 2) <= (3 2 2)  yes  + (1 0 2)  avail = (4 2 4)
   P3 (3 4 2) <= (4 2 4)  no , B
   P4 (5 5 2) <= (4 2 4)  no , A B
   
   P5 (3 0 4) <= (4 2 4)  yes  + (1 3 1)  avail = (5 5 5)
   P3 (3 4 2) <= (5 5 5)  yes  + (0 2 4)  avail = (5 7 9)
   P4 (5 5 2) <= (5 7 9)  yes  + (1 1 1)  avail = (6 8 10)
   SAFE -> GRANT , order P1 , P2 , P5 , P3 , P4

```

---

(c) P2 asks for (0 1 0)

```

1. (0 1 0) <= Need P2 (0 2 2)   ok
2. (0 1 0) <= Avail (1 2 1)     ok

3. pretend : Avail (1 2 1) -> (1 1 1)
             P2 Hold (1 0 2) -> (1 1 2)
             P2 Need (0 2 2) -> (0 1 2)
			 
			 
4. check with Avail = (1 1 1) :

   P1 (1 2 0) <= (1 1 1)  no , B 2 > 1
   P2 (0 1 2) <= (1 1 1)  no , C 2 > 1
   P3 (3 4 2) <= (1 1 1)  no , A B C
   P4 (5 5 2) <= (1 1 1)  no , A B C
   P5 (3 0 5) <= (1 1 1)  no , A C
   
   nobody fits -> UNSAFE -> undo , P2 WAITS


```

========================

6. Smallest number of units that makes deadlock impossible (Tutorial 5, question 2)

The idea: find the worst case, the closest the system can get to a deadlock, then "add one unit".

Worst case L every process hold "one less" Than its max 
and each one wiats for its last Unit . if no unit is free at that point . nobody can finish , and this is a Deadlock
 
 Fix: have 1 extra unit. 
 Then someone gets their last unit, finishes, and gives everything back, and the rest follow.

```
smallest # of units = sum of (max_i - 1) + 1

```

--- 

Question 1:        5 processes, each max 3

```
worst case : P1  P2  P3  P4  P5
holds        2   2   2   2   2     = 10 , all waiting for 1 more

10 units -> all held , avail 0 -> deadlock possible
11 units -> avail 1 -> one finishes , gives back 3 -> avail 3 -> next ... all finish

answer = 5 * (3 - 1) + 1 = 11

```

SO here the number = 11 for number of resoruces is the min to prvent Deadlock

---

Question 2: 			3 processes, max 2, 3, 4

```
worst case : P1  P2  P3
max          2   3   4
holds        1   2   3     = 6 , all waiting for 1 more

6 units -> deadlock possible

7 units -> avail 1 -> P1 gets its 2nd , finishes , gives back 2 -> avail 3
        -> P2 finishes , gives back 3 -> avail 5 -> P3 finishes

answer = (2-1) + (3-1) + (4-1) + 1 = 7


```

Why "one less than max"????


Holding the full max means the process can finish, so that's not stuck.

Holding less than max − 1 leaves more units free, so it's not the worst case either.

Same max for everyone: n × (m − 1) + 1.


=======================

7. Detection and recovery (Tutorial 5, question 4)

detection: don not check requets ahead of time . Let a dadlock happen , then find it and fix it 

How to find it : Use the same loop as Banker's , 
but compare "request"  (what the process is asking for right now), not need.

```
repeat : find a process with Request <= Avail
         it finishes -> Avail += its Hold
		 
whoever is left at the end = DEADLOCKED

```

The question:

```
A has 5 units , B has 4 . No unit is free.

          Hold      Request    CPU used   last checkpoint
         A   B      A   B
P1       1   0      0   0      12 min     12 min
P2       2   1      0   1      40 min     20 min
P3       0   2      2   1       5 min      5 min
P4       1   1      1   1      15 min     15 min
P5       1   0      1   0       3 min      3 min
-----------------------------------
held sum 5   4    -> Avail = (0 0)


```

(a) Which processes are deadlocked?

```
start                                     Avail = (0 0)
P1 req (0 0) <= (0 0)  yes  + hold (1 0)  Avail = (1 0)
P5 req (1 0) <= (1 0)  yes  + hold (1 0)  Avail = (2 0)
P2 req (0 1)  no , B 1 > 0
P3 req (2 1)  no , B
P4 req (1 1)  no , B
nobody else fits -> stop

DEADLOCKED : P2 , P3 , P4

```
Why: P1 and P5 only hold A, so finishing them never frees a B. 
All 4 B's are held by P2, P3 and P4, 
and each of them is waiting for one more B.

Notice: Every process that's still unfinished when the loop gets stuck is deadlocked.

```
finished : P1 , P5
left     : P2 , P3 , P4    <- ALL three are deadlocked

```

P4 isn't special because it was checked last. P2, P3 and P4 all failed against avail (2 0), and nothing more will ever be released, so all three wait forever.

The rule:
```
Run the loop until nobody fits.
Finished processes are fine.
Not finished processes are deadlocked.
If everyone finished, there's no deadlock.

```

================
(b) Which one process would you kill? 

Try killing each of P2, P3 and P4: 
add its hold to avail (2 0), then run the check again. Then pick the cheapest by CPU used.

my attempt:

```
try kill P2 , free resources (2  1) + (2  0 ) , now avail = (4 1)

check P3 request (2   1) <= (4  1) OK
now avail  (0 2) + (4  1) = (4  3)

check P4 request  (1 1) , <= (4 3) OK 
now avail  (1  1) + (4  3) = (5  4) check == total , GOOD 

all done 

-> works , but loses P2's 40 min

```

the other 2 opetion , try to kill p3 , or kill p4 

```
kill P3 : avail (2 0) + P3's hold (0 2) = (2 2)
P2 req (0 1) <= (2 2)  yes  + P2's hold (2 1)  avail = (4 3)
P4 req (1 1) <= (4 3)  yes  + P4's hold (1 1)  avail = (5 4)   all done
-> works , loses P3's 5 min

kill P4 : avail (2 0) + P4's hold (1 1) = (3 1)
P2 req (0 1) <= (3 1)  yes  + P2's hold (2 1)  avail = (5 2)
P3 req (2 1) <= (5 2)  yes  + P3's hold (0 2)  avail = (5 4)   all done
-> works , loses P4's 15 min


```

Conclusion:

```
kill P2 -> works , lose 40 min
kill P3 -> works , lose  5 min   <- cheapest
kill P4 -> works , lose 15 min

answer : kill P3

```

Why every option works here: 
each of the three holds at least one B. Killing any one of them frees a B, and that's enough to start the chain.

That isn't always true. 
If killing a process doesn't free what the others are waiting for, it doesn't help, 
and you'd have to kill more. 
That's why you check each option before choosing.

---

(c) cost: P3 starts over from the beginning, so its 5 min of CPU work is lost. 
It may also have left shared data half-updated.

Now try the rollback question: 
would rolling P3 back to its last checkpoint help???
Compare its checkpoint time (5 min) with its CPU used (5 min).

```
Anaswer : No, rolling P3 back doesn't help.


P3 : CPU used 5 min , last checkpoint 5 min

-> the checkpoint was taken just now , AFTER P3 got its 2 B
-> rolling back to it gives the same state : P3 still holds 2 B
-> still deadlocked

```

The rule: a checkpoint saves everything, including what the process holds. 
Rollback only frees a resource if the checkpoint is from before the process got it.

Checking all three:

```
process  CPU used  checkpoint  rollback loses  useful?
P2       40 min    20 min      20 min          maybe (only if P2 got its B before minute 20)
P3        5 min     5 min       0 min          no , checkpoint = now
P4       15 min    15 min       0 min          no , checkpoint = now

```
Notice: 
Only P2 has an older checkpoint. 
Even if rolling P2 back works, it loses 20 min, while killing P3 loses only 5 min.


Final answer: 
```
kill P3. Rollback is the better choice when a process has a checkpoint from before it took the resource, 
and going back to it loses less work than killing the cheapest process.
```

=====================================

8. Slide 42: comparing the three strategies


The prof said to study this one carefully for the final.


```
DETECTION   (very liberal : grant whatever is asked)
  + never delays a process from starting
  + good when deadlock is rare : better performance
  - recovery (kill / preempt / rollback) loses work
  - if a real deadlock is ignored -> can be a disaster




PREVENTION  (conservative : static rules , gives out less than it could)
  ask for all at once  : good for one-burst processes , but slow start
  preemption           : good when state can be saved , but preempts more than needed ,
                         processes may restart forever
  resource ordering    : checked at compile time , no runtime cost ,
                         but can't ask for resources a bit at a time





AVOIDANCE   (in between : always keep at least one safe path)
  + no preemption needed
  - must know Max ahead of time
  - checking every request costs time
  - processes can be blocked for a long time


```

One line each:

(1)Detection: "let it happen, then clean up."

(2)Prevention: "rules that make it impossible."

(3)Avoidance: "check each request, refuse unsafe ones." (Banker's, claim edges, trajectories)

(4)Ignorance: "pretend it doesn't exist." (UNIX)

--- 

Which is which (likely exam question):

```
Banker's algorithm                  -> avoidance

claim edges / no-cycle rule         -> avoidance

"always lock printer before scanner" -> prevention (circular wait)

"ask for everything at start"       -> prevention (hold and wait)

find cycle, kill a process          -> detection + recovery

do nothing                          -> ignorance (UNIX)

```

---

When to check for deadlock, in detection:

```
every request -> finds it early , costs a lot of CPU

periodically  -> cheaper , finds it later

watchdog      -> fires when the system stops responding

```

=======================

9a. Threads: the shared variable race (Tutorial 5)

The question:

```
int param = 0 ;

void * sum (void  * arg) {

	param = param +10 ; 
}

int main(void) {

	 pthread_t tid[10];
	
	// create 10 threads each has address in thread id array 
	 for (int i = 0; i < 10; i++) {
        pthread_create(&tid[i], NULL, sum, NULL);
     }

	 printf("parameter %d\n", param);
    return 0;
}

```
What can it print??? Any multiple of 10 from 0 to 100. That comes from two separate problems:

```
problem 1 : no join
  main does not wait for the threads , it can print before some or all of them ran
  no thread ran yet -> prints 0

problem 2 : race on param
  param = param + 10  is really 3 steps : load , add , store
  
  T1 load 30          T2 load 30
  T1 store 40         T2 store 40     -> two +10 , but param only went up once
  one update is LOST


```

Fix 1: join (fixes problem 1 only)

```
for (int i = 0; i < 10; i++) {
    pthread_join(tid[i], NULL);      // wait for every thread
}
printf("parameter %d\n", param);

```
notice:

no early print any more , but updates can still be lost
-> prints 10 , 20 , ... , 100

can not print 0 : every store writes (some value + 10) , so at least 10

---

Fix 2: mutex (fixes problem 2)

```
pthread_mutex_t    lock = PTHREAD_MUTEX_INITIALIZER;

void *sum(void *arg) {

    pthread_mutex_lock(&lock);       // one thread at a time
	
    param = param + 10;              // critical section
	
    pthread_mutex_unlock(&lock);
	
    return NULL;
}

```
now: 
join + mutex -> always 100


---

9b. Fixing it with no lock: return values

The idea: the race only exists because all the threads write the same variable. So don't share it:

2 key points : 

	(1)each thread computes its own result and returns it;
	(2)only main adds the results up, at pthread_join.

```
void *sum(void *arg) {

    int *result = malloc(sizeof(int));   // the thread's OWN memory
	
    *result = 10;                        // the thread's work
	
    return result;                       // handed to whoever joins it
}

int main(void) {
    pthread_t tid[10];
    int total = 0;  // notice the sum now happens in main () 
	
	// create 10 threads 
    for (int i = 0; i < 10; i++) {
        pthread_create(&tid[i], NULL, sum, NULL);
    }

    for (int i = 0; i < 10; i++) {
	
        void *val;
		
        pthread_join(tid[i], &val);      // wait , and receive the returned pointer
		
        total += *(int *)val;            // only main touches total
        free(val); 
    }

    printf("parameter %d\n", total);     // always 100
    return 0;
}


```

Notice: 

```
pthread_create(&tid[i], NULL, sum, NULL);
   -> starts a NEW thread , that thread runs sum()
   -> main does NOT wait , it moves on right away

pthread_join(tid[i], &val);
   -> main WAITS until thread tid[i] has finished sum()
   -> then copies sum's return value into val

```

---

How pthread_join() passes the value back:

```
thread : return result;          (a pointer , void *)
main   : pthread_join(tid, &val) -> val = that same pointer
         *(int *)val             -> turn it back into an int pointer , read the number

```
Why there's no race:

```
1. each thread writes only its OWN malloc'd int , nothing shared
2. only ONE thread (main) ever writes total
3. join waits , so each result is finished before main reads it

```

==============================

9c. Counting threads after fork (Tutorial 5)

The rule: 
fork() copies the whole address space (all memory, including variables like t1), 
but only the thread that called fork. 
The other threads don't exist in the child.

Every process starts with one thread, the main thread that runs main().


The question:
```
pthread_create(&t1, NULL, f, NULL);
fork();
pthread_create(&t2, NULL, f, NULL);

```


Step by step:

```
line                   parent              child

start                  main                -

-------------------------------------------------------------
pthread_create(&t1)    main , t1           -

fork()                 main , t1           main         							 (copy of the caller only)

pthread_create(&t2)    main , t1 , t2      main , t2     			(BOTH run this line)

parent : 3 threads
child  : 2 threads

total  : 5 threads


```

The two things that make it tricky:

```
(1) every line AFTER fork() runs twice (once in parent , once in child) -> t2 made twice
(2) t1 was made BEFORE fork and is NOT copied -> child has the variable t1 , but no thread

```

idea : How to count any version

```
(1)  go line by line
(2)  keep a column per process : which threads it has
(3)  fork -> new process , with only the calling thread (main)
(4)  a line after fork -> runs in EVERY process that exists

```

=========================

9d. User-level vs kernel-level threads (Tutorial 5)

User-level (many-to-one): many user threads run on 1 kernel thread.

```
advantages :
+ fast : create / switch = a library function call , no syscall , no trap
+ portable : the library runs on any OS (the kernel never sees the threads)
+ the app can pick its own scheduling

problems :
- one blocking syscall (eg read()) blocks the ONE kernel thread -> ALL threads stop
- no parallelism : kernel sees 1 thread -> uses 1 core , even on a multi-core machine

```

Kernel-level (one-to-one): each user thread gets its own kernel thread.

```
advantages :
+ parallelism : threads run on DIFFERENT cores at the same time
+ a blocking syscall blocks only THAT thread , the others keep running

problems :
- slower to create / switch (needs the kernel)
- each thread is a kernel object -> number of threads can be limited

```


The fix for many-to-one blocking (your "non-blocking re-implementation" note):
```
the thread library rewrites read() etc. with NON-BLOCKING calls :
  try to read -> no data yet? -> switch to another user thread -> try again later
so one waiting thread does not stop the whole app

JavaScript / Node.js : one thread , all I/O non-blocking (callbacks / promises) -> same idea

```

Quick compare:


```
                       user-level (M:1)      kernel-level (1:1)
create / switch        fast (no syscall)     slower (syscall)
uses many cores        no                    yes
one thread blocks      all block             only that one
```

==================================

10a. Amdahl's law (Tutorial 5: SSD and genome)

The formula:

```
a = fraction of the OLD TIME that gets faster
k = how many times faster that part gets
1 - a = the part that does not change

Tnew  = Told * [ (1 - a) +  a / k]

S     = Told / Tnew

S_max = 1 / (1 - a)              (when k reach -> infinity : that part takes 0 time)

```

How to solve any of these:

```
(1) split the OLD time : part that gets faster + part that does not

(2) shrink only the faster part

(3) S = Told / Tnew

(4) unknown k or a -> use 1/S = (1 - a) + a/k

```
---

SSD question

given : 
A job takes 40 s, half of it waiting for the disk. With an SSD it takes 25 s.

```
split : CPU 20 s (no change) + disk 20 s (gets faster)    a = 20/40 = 0.5

speedup : S = 40 / 25 = 1.6 times 

SSD k :   new disk time = 25 - 20 = 5 s

          k = 20 / 5 = 4                  -> SSD is 4 times faster
		  
  formula check :
   1/S = (1 - a) + a/k
  1/1.6 = 0.625 = 0.5 + 0.5/k -> k = 4

near-zero disk : only CPU 20 s left 

 -> S_max = 40 / 20 = 2 Times 


  formula :  use  1/S = (1 - a) + a/k
  
  1 / (1 - 0.5) = 2 TIMES  

```
Key point: the disk got 4× faster, but the job only got 1.6× faster. The CPU half never changed.

---

Genome question:
given 
120 min on 1 core: load 15 min + align 100 min (parallel) + write 5 min.

```
the load and write cannot be done paralle , only align can be done paralle 

split : serial 15 + 5 = 20 min , parallel 100 min   , total 120 min 
   
   calcualte "a" 
  ( a is the can be speed up ratio)

a = 100/120 = 0.833 

on N cores : T(N) = 20 + 100 / N     where 100 is the can be speed up part 

cores   time              speedup = 120 / T
1       20 + 100 = 120    1
10      20 + 10  =  30    4
100     20 + 1   =  21    about 5.71
inf     20 + 0   =  20    6   (S_max = 1/(1-a) = 120/20)


```

Key point: 
going from 10 to 100 cores (10 times the hardware) only goes from 30 min to 21 min. 
The 20 serial minutes cap the speedup at 6.

===============================

10b. Vertical vs horizontal scaling (Tutorial 5)

(1)Vertical: buy one bigger machine. 
The app doesn't change, but the price grows much faster than the power.

(2)Horizontal: buy more normal machines plus a load balancer. 
The balancer receives each request and sends it to one of the servers. 
The price grows roughly in line with the number of machines, 
but the problem has to be split across them, and each machine loses some efficiency.

The question:
```
vertical   : 1x $3,000   2x $7,000   4x $18,000   8x $50,000
horizontal : load balancer $2,000 + $3,000 per server , each server gives only 90%

```

How to compute horizontal:
->  each server gives 0.9, so to get X times the power:

```
servers = ceil( X / 0.9 )             (round UP , can't buy part of a server)

cost    = $2000 + servers * $3000

```

```
power   vertical    horizontal                         cheaper
1x      $3,000      1 server , no balancer = $3,000    same
2x      $7,000      ceil(2.22)=3  2000+9000  = $11,000  vertical
4x      $18,000     ceil(4.44)=5  2000+15000 = $17,000  horizontal
8x      $50,000     ceil(8.89)=9  2000+27000 = $29,000  horizontal

```

Notice
Why the answer flips:

```
small : horizontal pays extra for the balancer + loses 10% per server -> vertical cheaper

large : big machines get very expensive (8x power = 16.7x price)       -> horizontal cheaper

```
Cost per 1× of power (another way to see it):

```
power   vertical $ per 1x     horizontal $ per 1x
2x      3,500                 5,500
4x      4,500                 4,250
8x      6,250                 3,625

```
Vertical gets more expensive per unit as it grows. 
Horizontal gets cheaper, because the balancer's cost is spread over more servers.

Link to L12: the vertical cost curve is the "exponential" one on the slide, 
and horizontal is "linear". 

Horizontal only works if the problem can be split:  you distribute the data and collect the results.






