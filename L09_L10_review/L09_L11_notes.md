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

---

5. Hoare vs Mesa: same code, different order:

bounded buffer with N = 3 . using the class example . 
The buffer is full , count = 3 .
P4 called append and is asleep on notfull
Now C1 calls take() 

P is produce , C is consume 

Hoare:

```
C1  take: count 3 -> 2, csignal(notfull)

P4  runs AT ONCE, C1 waits in the urgent queue

P4  the condition is guaranteed true. appends, count 2 -> 3, leaves

C1  continues, leaves

```

Mesa:


```
C1  take: count 3 -> 2, csignal(notfull)   
	P4 becomes ready but does not run

C1  keeps running, leaves

??? anyone in the entry queue may get the box before P4

P4  gets the box later. the condition MAY be false again


```

So under Mesa it must be while: 

being woken only means 
"it was true at some point". 

Under Hoare, if is enough. 

while also works there, it just re-checks for nothing. 

That's why while is the safe habit, and almost every real system is Mesa.

---

The difference is who runs right after csignal.

```
Hoare: the woken thread runs immediately, and the signaller waits.

Mesa: the signaller keeps running, and the woken thread runs later.

Only one thread can be active in the monitor, so when a thread signals, one of the two has to give way.
```

That's why the code differs:

```
// enough under Hoare: the condition is guaranteed when you wake

if (count == N) {
    cwait(notfull);
}


----------------


// needed under Mesa: re-check after waking, sleep again if false

while (count == N) {
    cwait(notfull);
}


```

---

6. csignal or cbroadcast???

csignal: 
	any one woken thread can use the change (one free slot: one producer).


cbroadcast: 
	several can continue (the Event flag, 
	all the readers after a writer), 
	or you can't tell which one can (see B5). 
	With while loops, waking too many is only a little slow. 
	Waking too few can leave someone asleep forever.

---

7. Bank monitor: why no race

Only one thread is active in the box, 

so balance = balance - x 

can't interleave with 

balance = balance + y.

 Your class notes have this version. 
 
 Its weak point: when the balance is too low, withdraw just does nothing and returns. 
 
 The prof's exercise is to make it WAIT instead. That's B4.



===================================

## Round C   the Tutorial 4 examples


1. Dining philosophers

Deadlock needs all 5 holding one fork and waiting in a circle. Both fixes break the circle:

fix 1 : 
```
 the footman: maxP = Semaphore(4). 
 
 Only 4 at the table, 
 so 5 forks among 4 people. 
 
 By counting, someone gets two forks.
```

Fix 2, :

```
a left-hander: 

4 pick right first, 1 picks left first. 

Two neighbours go for the SAME first fork. 

The loser holds nothing while waiting, so the circle can't close.
```

Neither gets past 2 eating at once: 5 forks / 2 per eater. 
The fixes remove deadlock, they don't create forks.


fix 3.  use monitor:



---

code demo: 

Fix 1 : 4 max peopel allowed ,footman 

```
	semaphore maxP = 4;

	philosopher(i) {
		while (true) {
		
			think();
			
			
			wait(&maxP);    // as allow 4 max , can run 4 thread after that < 0 block 5th 
				wait(&fork[i]);
				wait(&fork[(i + 1) % 5]);
				
				eat();
				
				signal(&fork[i]);
				signal(&fork[(i + 1) % 5]);
			signal(&maxP);
		}
	}


```

fix 2 :  left-hander (one picks up in the opposite order)


```
philosopher(i) {
    while (true) {
	
        think();
		
        if (i == 4) {
			// wait left (can be 2 of them #0 and #4)
            wait(&fork[0]);            
            wait(&fork[4]);
        } else {
			// wait right 
			
            wait(&fork[i]);
            wait(&fork[(i + 1) % 5]);
        }
		
        eat();
		
        signal(&fork[i]);
        signal(&fork[(i + 1) % 5]);
    }
}

Philosophers 0 and 4 both go for fork 0 first. 
The loser holds nothing while waiting, so the circle can't close.

```

---

Fix 3: monitor (take both forks or none)

This one doesn't use the fork semaphores. It tracks what each philosopher is doing instead:

```
monitor Table {

    enum { THINKING, HUNGRY, EATING } state[5];   // all start THINKING
    
	condition canEat[5];                          // one room per philosopher

    void test(i) {
	
		// assume Hoare monitor 
		// when left right not eating 
		// this philosopher eat -> udpate state(Conditinal_variable) to eat 
		// also csignal canEat for its "room"
		
        if (state[i] == HUNGRY &&
            state[(i + 4) % 5] != EATING &&       // left neighbour
            state[(i + 1) % 5] != EATING) {       // right neighbour
            
			state[i] = EATING;
            
			csignal(canEat[i]);
        }
    }

    void pickup(i) {
        state[i] = HUNGRY;
        test(i);
        while (state[i] != EATING) {
            cwait(canEat[i]);
        }
    }

    void putdown(i) {
        state[i] = THINKING;    // always save conditioonla variable state first 
        test((i + 4) % 5);                        // can the left neighbour eat now?
        test((i + 1) % 5);                        // can the right neighbour eat now?
    }
}


```


---

2. The Event monitor: one-time "it has happened"

The fix adds done (Conditinal_variable). 

A wait() after the push sees done == true and doesn't sleep at all:


```
void push() { 
	done = true; 
	cbroadcast(c); 
}

void wait() { 
	while (!done) {
		cwait(c); 
	}
}


```

---

3. Readers-writers (NOT procedures consumers)

The trick: the monitor doesn't protect the database.
 It protects the bookkeeping. 
 
 -> grant the permission to enter 

The actual read happens OUTSIDE:

```
RW.startRead();    <- box, only to update counters. leaves the box

read the database  <- many readers can be here at once

RW.endRead();      <- box again


```

---

readers-writers code. The monitor first:

```
Monitor RW {


	condition okToREAD;
	condition okToWrite; 
	
	int readers = 0 ; 		// how many are reading right now
	int waitingWriters  = 0 ;     // how many writers are waiting
	
	boolean writing = false;  		// is a writer writing right now?
	

	void startRead() {
	
		// assume use Mesa monitor , use while loop for  cwait 
		
		while(writing || waitingWriters  >0){
		
			cwait(okToRead);
		}
		
		readers++; 
	}
	
	// keep till reader is actully reach 0 then signal okToWrite
	void endRead() {
	
        readers--;
		
        if (readers == 0) {
            csignal(okToWrite);   // last reader out lets a writer in
        }
		
    }
	
	
	 void startWrite() {
        waitingWriters++;
		
		// when still has reader , wirter wait 
		
        while (writing || readers > 0) {
            cwait(okToWrite);
        }
		
        waitingWriters--;
        writing = true;
    }
	
	void endWrite() {
        writing = false;
        csignal(okToWrite);       // wake one writer
        cbroadcast(okToRead);     // wake all readers
    }



}

============================

The threads that use it. The real work is outside the monitor:


reader() {
    while (true) {
        RW.startRead();
        read the database;    // many readers can be here at once
        RW.endRead();
    }
}

writer() {
    while (true) {
        RW.startWrite();
        write the database;   // only one writer, and no readers
        RW.endWrite();
    }
}

```
Notice:

```
The writing flag and the readers count are what keep writers exclusive even though the work happens outside the monitor. 
A thread only gets through startWrite() when nobody is reading or writing,

 and writing = true then stays set after it leaves the monitor, which keeps everyone else out.

```

4. A semaphore built from a monitor

```
void wait()   { 

	// wait , use while loop as Mesa 
	// when reach 0 
	
	while (value == 0) {
		cwait(queue); 
	}

	 value--; 

}

void signal() { 
	// similar to count++ and then wakeup 
	value++;  
	csignal(queue); 

}


```


---

5. A condition variable built from semaphores

The other direction. 

A semaphore remembers, so it would be wrong to always call sem.signal(). 

The waiters count exists to make a signal do nothing when nobody is waiting:

```
csignal(c) {

    c.mutex.wait(); // lock 
	
		// critical section 
	
		// only if someone is really waiting
		if (c.waiters > 0) {
			c.waiters--;
			c.sem.signal();  // wake up 
		}   
	
    c.mutex.signal();
	
}

```

Notice:

cwait() must leave the monitor before sleeping, 
then come back in. 

Sleeping while holding the box would block everyone.

So semaphores and monitors have the same power: 
each can build the other.

===================================

## Round D :			 L10 file system concepts

---


1. what a file is :


A named collection of related information. 
It can be accessed by addressing.

Logical view: what users and apps see. 

Physical view: how it sits on storage.

Attributes: 
	name, identifier, location, size, protection, timestamps.


---

2. Where a file operation happens 

```
Process  -> triggers open / read / write, gives the buffers

Kernel   -> really implements them (privileged), 
holds buffers + tables

Disk     -> all the persistent structures live here

```


--- 

3. The operations 

(1) create () :
what doe sit does ??

check permission on the directory , 
insert the name , allocate blocks , point the entry at them . 

(2) open() : 

check permission . Must succeed before any other operation


(3) write():
The permission from open must include write .
Location matters : 
-sequeation 
-random


(4) Read() :

permission from open must include read, needs a memory buffer, 
can return LESS than you asked, moves the pointer

(5) Reposition (seek) :
moves the file pointer to a given value

(6) Delete :
check permission, remove the entry. 
If other links remain the blocks stay, otherwise they are recovered


(7) Truncate : 
length set to the given size, 
blocks holding user data are recovered


---

The information the OS keeps for an open file:

(1) File pointer: 

where the next read or write goes. Unique to each process


(2) Open count: 
how many have it open, so the table entry can be released

(3) Location: 
which disk blocks, so you don't re-read the directory every time

(4) Access rights: 
the access it was opened for. Anything else is denied

---

Sequential: 

```
read_next() and write_next(), 

the pointer moves by itself. Editors and compilers use it.
```

my quetsion : 

```
let's say i get a new disk attact to my diy pc , 
i install the offical disk managemet software and there is a migration feature . is that sequenctail ???

```
Yes, mostly, but it's sequential at the disk level, which isn't quite what that slide means.

```
What the slide means: 
the sequential access method is about how a program reads one file. It reads record by record, 
and the file pointer moves by itself.
```

What migration usually does: 
it's usually a block-level copy. It works on the disk's blocks and doesn't look at individual files:

```

for (block = 0; block < last_block; block++) {
    read block from the old disk;
    write block to the new disk;
}

```

---

Direct: 
fixed-length records, read or write any record. 

You can emulate sequential with it by keeping a position, and it can be used to build indexed files.

---

Types: 
most are shown by extension. 
The OS really only cares about a few, like executables. 

UNIX identifies them with magic numbers (a fingerprint in the file).

a magic number is usually a few fixed bytes at the very start of the file.
The length depends on the file type, often 2 to 8 bytes.

A magic number also means the name doesn't decide the type. 
If you rename photo.png to photo.txt, its first bytes are still 89 50 4E 47, so file still reports a PNG.

---

Windows mostly uses the extension to decide what to do with a file. When you double-click photo.png
 Windows looks up .png in its list of file associations and opens the app linked to it.

if you rename photo.png to photo.txt, Notepad opens it and shows garbage

---

5. Directories


A directory is a symbol table: 

name -> file control block. 

The FCB (File control block) says where the data is.

Operations: create, delete, list, rename, traverse.

```
Single-level : 
it is jsut one list for all , 
unique names, only good for small embedded devices


Two-level: 
MFD indexed by user -> each user's UFD
weak sharing, system files are awkward

Tree:
sub-directories, current directory, relative or absolute paths
no sharing


Acyclic graph:
links to the same instance, not copies
must keep cycles out


```

---
hard vs soft : 

```
Hard link: another name for the same file, 

files only, so no cycles. 

Soft link: may point to directories, so cycles are possible.
```

6. Memory-mapped files:

Map the file into the address space with virtual memory. 

Plain loads and stores replace read() and write(). 

A write may stay in memory until close or until the page is flushed.

===================================

## Round E :     L11 implementation

1. The layout 

```
whole disk:

MBR | partition table | partition A | partition B | partition C

one partition:

boot block | super block | free space mgmt | ... i-nodes ... | ... data blocks ...



```

meaning :

```
Boot block: 
bootstrap code for the OS in the partition

Super block: 
geometry of the partition. Where is everything?


i-nodes: 
this is where the FCBs are

Data blocks: 
file contents AND directories

---

A directory is a file, and its rows are name : 
i-node number.

```

---


Directory as a linear list:
 searched at creation for uniqueness, 
 slow on big directories.

 Directory as a hash table: 
 
 fast lookup, but you must deal with table size and collisions.

---

2. The allocation methods

```
(1) Contiguous	

	the file  sits in consecutive blocks
	good for Sequential
	goof for direct access , as  just compute
	has External fragmentation -> need compaction
	hard to Grow ! 
	
	Weak spot: 	finding a hole
	
(2) Linked
	each block points to the next
	good for Sequential
	Bad for direct access , as need to walk the chain 
	No external fragmentation
	easy to grow 

Weak spot: 	one bad block breaks the chain

(3) FAT
	the links moved into one table
	Good for sequential
	good ofr direct access , if teh FAT table is in memory
	No external fragmentation
	easy to grow 

Weak spot: 	the table itself is critical


(4) Indexed (i-node)
	one block holds all pointers
	Good for sequenctail
	Good for Direct access 
	No External fragmentation
	easy to Grow (until the index is full)
	
Weak spot : 		pointer overhead, internal fragmentation



```

---

3.  The i-node

```
12 direct pointers, 

1 single indirect, 

1 double, 1 triple. 

With block size B and pointer size d:

```

Formula :
```
max size = 12*B + (B/d)*B + (B/d)^2*B + (B/d)^3*B

```

meaning : 
Each layer multiplies by B/d, the number of pointers that fit in one block.

---

Worked, with B = 1024, d = 4: B/d = 256

```
B 	is the block size in bytes, so each disk block holds 1024 bytes.

d 	is the size of one disk pointer in bytes. 
A disk pointer is a block number saying where another block is, and it takes 4 bytes here.

B/d is how many pointers fit in one block: 1024 / 4 = 256.


```

B/d matters because of indirect blocks. 
An indirect block doesn't hold file data. It's a block filled with pointers:

```
single indirect block (1024 bytes):
+-----------+-----------+-----------+-----+-------------+
| pointer 0 | pointer 1 | pointer 2 | ... | pointer 255 |
+-----------+-----------+-----------+-----+-------------+
  4 bytes     4 bytes     4 bytes             4 bytes
     |           |           |
     v           v           v
   data        data        data


```

So one indirect block points to 256 data blocks,

 each holding 1024 bytes. 
 
 That means it adds 256 × 1024 = 262,144 bytes to the largest possible file.

---
Notice : 

Each extra layer multiplies by another 256:

```
Direct -> 12 datablocks -> Bytes : 12 × 1024 = 12,288

Single indirect -> 256 block -> Bytes : 256 × 1024 = 262,144

Double indirect -> 256² block -> Bytes :  256² × 1024 = 67,108,864

Triple indirect ->  256³ blocks -> Bytes : 256³ × 1024 = about 17 GB

```

Adding those gives the formula from the slide:


```
max size = 12*B + (B/d)*B + (B/d)^2*B + (B/d)^3*B

```

---

4.  Worked: finding the n-th byte (Tutorial 4)

Assume the file is open, so its i-node is in memory.

```
given 	n = 5000

Thus :
5000 / 1024 = block 4, 
offset 904. 

block 4 < 12, so direct. 1 disk read (the data).

------------------

when n = 100,000: 

100,000 / 1024 = block 97, offset 672.

97 >= 12, so subtract: 97 - 12 = 85

this land inside the second layer as 85 < 256

so single indirect, entry 85

-> 2 disk reads

(the indirect block, then the data)

```

The steps:

```
block number = n / B
offset = n % B

subtract 12, then 256, and so on, until it fits a region

reads times needed from disk 
=>  the number of pointer blocks + 1 for the data
```


---

5. Worked: how big is the i-node table? (class 16)

Given : 
2000 blocks of 4096 bytes.
 i-node 68 bytes. 
 Average file 3 blocks.
 
 
```

files: 2000 / 3 = about 666 -> round up to 700 i-nodes (the spare 34 are for directories)

i-nodes per block: 4096 / 68 = about 60 rows 

table: 700 / 60 = about 11.6 
-> Need 12 blocks


```

---

6. Worked: where is i-node 100? (class 16)

```
byte: 100 * 68 = 6800
block: 6800 / 4096 = 1 (so the 2nd block of the table)
offset: 6800 - 4096 = 2704

```

Disk reads always take a whole block, 
so you read that block and look at offset 2704. 

An i-node number gives you the disk block directly, no searching.

---

7. Worked: the path walk (class 16 + Tutorial 4)

/user/abc/xyz.c:
 k = 3 names (user, abc, xyz.c). 

No caching. Each directory is one block.

```
(i-node 0, root)      read from disk only if it's not already in memory

1  root data block    "user : 10"

2  i-node 10

3  user data block    "abc : 21"

4  i-node 21

5  abc data block     "xyz.c : 30"

6  i-node 30          <- open() finishes here

7  first data block   <- the first byte

```

Notice : 
```
2 reads per name (the directory's data, then the next i-node) + 1 for the byte 

that is : 

 2k + 1 = 7.
 
```

If the root i-node must also be read: 8. 

Always say which one you assumed.

---

Notice:

```
i-node then data, i-node then data, always alternating.

Deep paths cost more. Caches help, and a relative path from the current directory skips the upper levels.
```


---

8. Worked: rm and the link count (class 16)

The i-node stores a link count: 
how many directory entries point to it.

eg:
/home/xy.c 
-> i-node 31, count 1, blocks 251, 3090, 3100.

```
given command 
rm /home/xy.c:

remove the row "xy.c : 31" from home's directory block

count 1 -> 0, so nobody uses the file now

free blocks 251, 3090, 3100: set their bits in the bitmap (1 = free)

free i-node 31


```

Hard link: /user/abc/xyz.c also -> i-node 31. 

Now the count is 2, and rm /home/xy.c only makes 2 -> 1. 

No blocks are freed.


---

9. Free space 

```
Bitmap: 1 bit per block, 

1 means free. 

CPUs have instructions that scan bits fast.
```

Size: (disk size / block size) bits.

 1 TB with 4 KB blocks: 
 ```
 1TB/ 4KB 
 that is : 
 
 2^40 / 2^12 = 2^28 bits 
 
 = 2^25 bytes = 32 MB.    for bit-map 

```

Linked free list: 

each free block points to the next. 
Adding is cheap, 
and traversal isn't done often. 

For contiguous allocation, keep a pointer + a count of contiguous free blocks.


===================================

## Round F   :    L12 so far

1. Ways to get compute

```
(1)Cluster: commodity machines, loosely connected. Cheap to scale, slow messages. Good when each task is relatively big.

(2)Parallel computer: expensive because of the tight connections (see top500.org).

(3)Cloud: massive clusters, split into smaller nodes with virtualization. Pay for what you use, so cost follows demand. 

```

---

2. Scaling

```
(1)Horizontal: more servers. The problem must split across machines, needs distribution of data and accumulation of the results.

(2)Vertical: a bigger server. The application doesn't change, but cost rises rapidly with machine size.

```
Which is better? It depends if the problem can be split.


Finance company: 

buying 6 servers is fine while customers pay, but when students leave for summer the cost stays high. 
Outsourcing lets the cost follow the demand.


---

3. Amdahl's law

a = the fraction of the OLD TIME that can be enhanced.

 k = how many times faster that part gets.

formula
```
Tnew = Told * [(1 - a) + a / k]

S = Told / Tnew = 1 / [(1 - a) + a / k]


```

The part you can't enhance limits you, no matter how big k is.

Worked: an app takes 100 s, 40 s of it can run 2x faster. a = 0.4, k = 2.

```
Tnew = 100 * (0.6 + 0.2) = 80 s

S = 100 / 80 = 1.25

```


