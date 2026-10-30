# My notes: L09 recap, Tutorial 4, L10, L11, L12

Concepts are in L09_L11_guide.md. Answers to the questions go in L09_L11_answers.md (separate file, later).

## Round A


L09 recap

1. The problem 

counter++ is 3 instructions: 
load, add, store. 

A switch between a load and its store loses an update. Whoever stores LAST wins.


race condition = shared data + the reuslt depends on who runs when . 
the cure is mutula exclusion on th ecirtical section .


---

2. the 4 requirements

(1) mutual exclusion:
no two inside at once 

(2) no assumption about speed or numbers of cpus

(3) progress: nobody OUTSIDE the critical section blocks another 


(4) boundeed waiting : noboady wait forever .


---


3. the fialed software attemps 

(1)attemps diable interrups 
-> not working as it no alloed for user code , other CPU keeps urnning 


(2) use lock variable :

this is wrong as both read 0 , both set 1 ,
test and sets are multiple line of assembly code 
breaks 

(3) strict alterntipon , take truns 
-> this also wrong , as a process outsid estill hols the trun 
it assume cpu speed or strick truns

(4)  check flags then set 
-> this is still worng , as both can check before either sets

(5) set flags , then check 

this is still wrong , as both set , and both can wait 
-> can casue dead lock 

========

the actuall fix : "Peterson "
this only works for 2 thread

we have flag[0] and flag[1] each is a boolean want flag

we do "want" before yead the other 

```
 flag[i] = true; // I want in 
 turn = j ; // but you go first 
 
 // the while loop wait onl check the other thread j 
 // the flag of j check if other want 
 // the trun of j is the atcull turn 
 
 // this works as the later comes thread 
 // would overwirte the turn into the previous comes
 // so if comes ealier , the flag[other] handles its
 // if comes later , the turn == other handles it 
 
 // only when both happends we wait 
 // which means only when other comes first (other want)
 // AND we are indeed late -> turn is also other
 // this case we spin wait 
 
 while(flag[j] && turn == j){
	;// spin wait 
 }

  // critical section 
  doSomethingCritical();
  
  // flag i am out , no longer want 
  flag[i] = false ;
  
  
	

```

---

4. hardware helps

Barrier:

every load and store before it complelets before any after it .



```

// hardware previous "test and set"
// called "atomic" , return the old value , wirte 1 


// whoever sees false (lock not yet taken) got the lock 

while(test_and_set(&lock)) {
; // spin wait 
}


```

```
// compare and swap : atomic 
// wirtes new only if the value == expected 

// if have been chancged somehow -> no longer expected 
// always return teh old value and retry 

whiel(comapre+and_swap(&lock , 0 , 1) != 0){
 // spin wait 
}


```

```
// atomic increment : retry until nobody changed v between the read and the CAS 

do{
	temp = *v;


} 
// onlly when temp is still temp we can do temp +1 increment 
while ( temp != compare_and_swap(v, temp , temp +1)){

	; // spin wait 
}


```

Notice: 
xv6 spinlock: acquire() turns interrupts off on this CPU, then does an atomic swap. 
Interrupts must be off, or a handler on the same CPU that wants the same lock spins forever.

---

5. Spin or sleep 

Short wait: spin. 

Long wait: sleep. 

On one CPU spinning only wastes the time slice.

---

semaphore :

itself is just a struct , with field : count and queue 
```

struct semaphore {
	int count ;
	queue q ; 
}
	
	wait(s) {
		s.count--;
		if(scoun <0){
		
			put me (cur thread ) in s.q ;
			
			block me ;
		}
	
	signal(s) {
			
		// FCFS order 
		taken the thread P from s.q ;
		
		make P ready;
	
	}

}


```
Rules:

(1) wait may block(when <0), signal NEVER blocks(as it is used to wake up)

(2) you can't read a semaphore's value (that is the whole point )

(3) a negative count = how many are blocked;
 as all the <0 one get place in queue and blocked 

(4) the starting value has a meaning:

 0 = event not happened yet, 
 1 = lock, 
 N = N free units

---

6. The semaphore patterns

(1) signalling 

semaphores s(0);

take 2 thread A , B 

```
A: a1 ; 
s.signal() 

B : s.wait() ;
b1; 

```
so the A wake up B with signal() 

---

(2) Rendezvous

for example: there is a(0) , b(0) 2 semaphores

```
signal your own FIRST 

then wait for the other 

---
aArrived = Semaphore(0);
bArrived = Semaphore(0);


/* thread A */            /* thread B */
a1;                       		  b1;
aArrived.signal();        bArrived.signal();   // "I'm here"
bArrived.wait();          aArrived.wait();     // wait for the other
a2;                       		  b2;


```
Why it works: each thread says "I'm here" first, then waits for the other.

if A comes first , signal , so aArrived goes to 1 (bumped up by signal)
then blocks on bArrived.wait();   
as bArrived is now -1 < 0 

if both wait first :
each one blocks waiting for a singal the other never reaches 

so they deadlock -> this is why we call singal() first then wait !

Compared with signalling: signalling is one-way, 
so A goes first and B waits. Rendezvous is two-way, 
so it needs two semaphores, one for each direction. 

A barrier is the same idea for n threads.

---

(3)  mutex 

Semaphore m(1) 

```
wait ();

do-critical();

signal() 

```

---

(4) Barrier: 

similar idea as the Rendezvous , but for n many thread 

it has :
	mutex(1), barrier(0), count
	
	eg:
	
```

	n = 4 ;  count = 0 ; 


	mutex  = Semaphore (1);  // protects count 
	barrier = Semaphore(0);  // locked till every one is here 

	mutex.wait();
		count++;  // count is with in the critical section 
						// use mutext lock protect 
	mutex.signal() 

	if(count == n){
		barrier.signal();  //  the last one to arrive opens the barrier!
	}
	
barrier.wait();     // line 1: stop here until a permit is available
barrier.signal();   // line 2: pass the permit to the next thread

	
```

what is the last 2 line for ???

```
ok , so thet wait and singal is actullt to capture the previous thread before we satsify the if branch ? 

like when count is not n , there has to be a wait somewhere , when count finnaly reach n it breack the barrier wait line 1 . 
and the barrie signal - line 2 is for what ???

```

answer again my understanding :

```
Yes, you've got line 1 right

Line 2 is there because that one signal only wakes one thread. 

A, B and C are all asleep, but the if only runs once, and one signal() wakes exactly one thread. Line 2 is how the others get woken: each thread wakes the next one on its way out.
```

```
ok , so think of it like this block of code is itendtial for all thread ,
 wake by reach the  barrier.signal();  
 
 in the if branch only free one thread , to wake the other still need to do "chain wake up "
 when one is already waken it run that line 2

 to wake up another by increase barrier number ++
 
 One small correction: line 2 doesn't wake a thread just because the number goes up. signal() does two things:
 
 -------------------
 
 signal(s) {
    s.count++;
    if (s.count <= 0) {          // someone is still asleep
        take P from s.q;         // this is the actual wake-up
        make P ready;
    }
}

so , the wake up is also inside the signal code itself , where it make P read and remove from queue


```
---

(5) Bounded buffer:

producers put items into a buffer with N slots , and consumers take them out .

There are three things to get right :

1) full buffer :
the producer must sleep till there is a free slot 

2) Empty buffer :
the consumer must sleep till where is an items

3) changing the buffer : 
only one thread can touch it at the time , or u get a race ! 


---
quick recap for "wait()" and "signal()"

why do we push things into them ???

```
	struct semaphore {
		int count;
		queue q;
	};

```

wait (s: Semaphore) 
```
	void wait(struct semaphore *s) {
		s->count--;
		if (s->count < 0) {
			put this thread in s->q;
			block this thread;
		}
	}

```

signal (s : Semaphore) 

```

	void signal(struct semaphore *s) {
		s->count++;
		if (s->count <= 0) {
			remove a thread P from s->q;
			make P ready;
		}
	}

```






---


eg:

```
semaphore mutex = 1;   // use as a lock , ensure one thread in the buffer at a time
semaphore empty = N;   // how many free slots
semaphore full  = 0;   // how many items

producer() {

	 while (TRUE) {
		item = produce_item();
		
		wait($empty);  // pass in the semaphore . when empty N < 0 blocked
		
		wait(&mutex);
			// critical section , need protect by mutex 
			insert_item(item);
			
		signal(&mutex);
			
			
		signal(&full);        // pass in full semaphore , increment item number 
		// if the buffer is empty this also wake up the consumer 

	}

}


```


consumer : 

```

	consumer() {
		while (TRUE) {
		
		
			wait(&full);          // sleep if no items , as when full is 0 , -1 make it <0 
									// block consumer 
			
			wait(&mutex);
				// crtical section 
				item = remove_item();
			signal(&mutex);
			
			signal(&empty);       // one more free slot , as empty is N 
			// when call it get N +1 count , 
			//also wake up the producer who was block when buffer was full 
			
			
			//  safe as it is just read , as long as there is item 
			consume_item(item);
			
			
		}
	}

```


---

7. xv6 has no semaphores or monitors: 

only sleep(channel) / wakeup(channel), 

with a while re-check (Mesa style). sleep_prepare() registers before the lock is released, 
so no wakeup gets lost.


===================================

## Round B  semaphore, monitor, condition variable


A Semaphore is a counter that "remembers" 

A condition _variable remebers Nothing 

A monitor is a box , shared data + procedures 

---

1. the three tools 


(1) Semaphore 

```
a struct holding counter + queue 

it is a stand alow , 

for mutal exclusion , use wait(mutex) .. singnal (mutex)

wait(s) :
counter -- for that s , and only block if count < 0 

signal with noboday waiting :
count goes up , it is remembered , and it wake up when count <= 0 
```

---

(2) Monitor ;

```
it is a box with shared data + its procedures

it is a stand alone 

it use automitic for mutual exclusion , one active thread inside 

no need for mutex lock inside !!

the state is in your own variables , count  , done , balance ... 
```

---

(3) Conditinal_variable :

```
It is a waiting room isnide the box ;

It IS NOT stand alone , only inside a monitor !

it have no mutal exclusion : he monitor gives it !

it use "cwait(c)" which ALWAYS sleep ,
ulike the wait(s) for Semaphore could only sleep when < 0 

what happend when do signal() when no one is waiting ???
-> it get LOST 

ulike in Semaphore when call signal(s) when no one waiting
it still count ++ , as Semaphore remebers !

Conditinal_variable has No state !

```
---

picture of monitor : 

```
   entry queue            +------------- monitor --------------+
 T5 T6 T7  ------------>  |  shared data: count, buffer ...    |
 (waiting to come in)     |  procedures: append(), take()      |
                          |                                    |
                          |  only ONE active thread in here    |
                          |                                    |
                          |  notfull  room:  P2 P3 (asleep)    |
                          |  notempty room:  (nobody)          |
                          +------------------------------------+

```

Threads asleep in a room are not active, 
so they don't block the box. 

hat answers the class question "if we sleep inside the monitor, how does anyone get in to wake that thread?"

so how does it get waken up when sleep inside monitor ???

```
Another thread wakes it. A sleeping thread can't wake itself, so it stays asleep until a different thread enters the monitor and calls csignal on that room.

The other thread can get in because the sleeper gave up the box when it called cwait.
```
-- 

eg : 

Here's the bounded buffer, with N = 3 and the buffer full:

```
append(x) {
    if (count == N) cwait(notfull);   // P4 sleeps here
	
    buffer[nextin] = x;  // save into buffer 
    nextin = (nextin + 1) % N; // move index forward 
    count++;      // increase number of item 
	
    csignal(notempty);      // signal that it is indeed no longer empty 
		// this wake up the consumer sider sleep 
	
}

```

```
	take(*x) {
		if (count == 0) cwait(notempty); // sleep when empty 
		
		*x = buffer[nextout];
		nextout = (nextout + 1) % N;
		count--;
		
		csignal(notfull);                 // wakes P4
	}
```

Notice 

```
You need just one monitor, not two. append and take are two procedures inside the same monitor, and that monitor 
has two condition variables (rooms). There must be one monitor, 
```


HERE is the whole  thing :
the monitor version of producer consumer 
```
	monitor boundedbuffer {
	
		char buffer[N];
		
		// index for the producer 
		int nextin = 0;
		
		// index for the consumer 
		nextout = 0;
		
		// item inside buffer count 
		count = 0;
		
		// the 2 Conditinal_variable
		
		condition notfull;    // producers sleep here
		condition notempty;   // consumers sleep here

		append(x) { ... }
		take(*x)  { ... }
	}


```
Notice :
as we use 2 Conditinal_variable , that means there are 2 rooms 
inside one monitor ! 

cwait(notfull);    // sleep in the notfull room
cwait(notempty);   // sleep in the notempty room

The rooms are separate because each holds threads waiting for a different thing:

```
+----------------- monitor boundedbuffer -----------------+
|  buffer, count, nextin, nextout                         |
|  only 1 active thread in here                           |
|                                                         |
|  notfull room:   producers waiting for a free slot      |
|  notempty room:  consumers waiting for an item          |
+---------------------------------------------------------+

```

This lets a signal wake the right kind of thread:
```
csignal(notempty);   // wakes a consumer, never a producer
csignal(notfull);    // wakes a producer, never a consumer
```

---

3. What cwait and csignal do

cwait(c):

```
go to sleep in room c
give up the box, so another thread can enter
when woken, get the box back, and carry on from the line AFTER cwait
```

csignal(c):

```
wake ONE thread in room c

if the room is empty, nothing happens at all
```

cbroadcast(c):

```
wake EVERY thread in room c 
```

---

4. Why a flag or count is always there ???

A condition variable has no memory, so YOU keep the state in a normal variable and check it before sleeping:

```
while (I can't continue)    // look at your own variables

    cwait(room);
	
// change the shared state
// csignal or cbroadcast whoever may be able to continue now

```

This is the recipe for every monitor. Bounded buffer,
 readers-writers and the Event monitor in round C all follow it.
 
 ---
 
 example for use flag for monitor as Conditinal_variable has no memory
 
 Without a flag (broken):


```
	monitor Event {
		condition c;

		void push() {
			csignal(c);
		}

		void wait() {
			cwait(c);
		}
	}

```
 If push() runs first:

```
T1 push()  -> csignal(c), the room is empty, so the signal is LOST
T2 wait()  -> cwait(c), T2 sleeps, and nobody will ever wake it
```

With a flag (fixed):


```
monitor Event {

    condition c;
	
    boolean done = false;      // remembers that it happened

    void push() {
        done = true;           // write it down into the flag first
		
        cbroadcast(c);         // wake anyone already asleep
    }

    void wait() {
        while (!done) {        // check the memory before sleeping
            cwait(c);
        }
    }
}


```

Notice :

cwait(c) puts the thread to sleep, 
so the thread stops at that line and uses no CPU. 
The loop only goes around again when someone wakes the thread, which is usually just once or twice.

 
 The loop is there to re-check after waking.
 Under Mesa, 
 ```
 being woken only means the condition was true at some point. 
 It may be false again by the time you run.
 ```

===================================

## Round C

===================================

## Round D

===================================

## Round E

===================================

## Round F
