
## reacall last tutorial 
The Dinning philospheor problem :

---

We have 5 philospheor and there are 5 forks 
for each to eat , need 2 forks;

each philospheor want to do a loop of :
think();
eat();

---

we find out the if all of them pick up a fork , they will all wait for the other fork to be free
and that is all waiting , the dead lock ;

solution 1: 
to fix this , we allow max of 4 philospheor to go to eat stage ;
in that case , no dead lock 

if there is philospheor without 2 forks he will simply wait ;

this is the simple semaphore solution:

```

maxP = Semaphore(4); // start from 4 , allow 4

maxP.wait();

fork[i].wait();		// first fork 
fork[(i +1) mod 5].wait(); // second fork() 

eat();

fork[i].signal();
fork[(i +1) mod 5].signal();

mapP.signal(); 


```

## how to max concurency ???

we can think of philospheor to be all "right handed"

the deadlock comes from their all being left handed or right handed:

we can assume 4 right handed and 1 left handed philospheor !

 in this case the right hand and left hand philospheor will fight for the same fork 
 at some point 
 
 the idea is when the conflict happens , the left hand and that right hand 
 loser will no go pick the other one . So this is similar to the semaphore
 
 
==============
 
 
## Monitor :

recall inside Monitor there should be only 1 tread running at time !

attempt 1 (wrong): 

```


Monitor Event {
	
	condition  c ;
	
	void push() {
		signal(c);
	}
	void wait(){
		wait(c);
	} 

} 


```

fix : 


```

Monitor event {

boolean done = false; // the flag variable 

void push() {
	done = true; 
	signal(c);
	
}

avoid wait(){
	while(!done){
		; // spin wait 
	}
	
}


```

Notice : this is a one time use Monitor , once used 
no matter how many times the wait() is called it will just go through 


=================================

## readers - writers problem 

assume we have many writers and readers to a database

we can allong allow one of them goes into and do the thing 

wirte also need "exclusive" access: as we want to avoid wirte into 
critical section at same time -> data race !


but a do want reader to have shared access with other readers !
as for read , reading at the same time is safe .

if readers and writers waiting , which one should get the higher priority ???



---
solution : use Monitor

Notice : there is NO "semaphore" inside Monitor !
there is conditional variable inside Monitor !


```
	Monitor RW {
		condition_variables okToRead ; 
		condition_variables okToWrite ; 

		int readers = 0 ;
		int waitingWriters = 0 ; 
		boolean writing = fasle; 
		
		
		// notice : reader is not wait inside the Monitor!
		// startRead() gives the previlage to read
		// if there is another reader , it will also go through 
		// make reader++ , this ensure the reader is concurent 
		void startRead(){
			while(writing || waitingWriters > 0 ){
			
				cwait(okToRead); 
				
			}
			
			readers++; 
		}

		void endRead() {
			// reduce reader count 
			reasers--; 
			
			// when already reach 0 , it is okay to wirte now 
			if(reasers == 0){
				csignal(okToWrite); 
			}
		}


		// simiar to teh startRead() , it also just gives the preliage of wirte whe no reader 
		// OR when the wirtting boolean is on 
		void startWrite() {
		
			while(Writting || readers >0 ){
				cwait(okToWrite);
				
			}
			
			waiting writers--;
			writing = true; 
		}
		
		
		void endWrite() {
		
			// trun off wirte flag 
			writting = false;
			
			// next writer can comes 
			csignal(okToWrite);
			
			// wakeup all the readers can be realse from waiting 
			cBoardcast(okToRead); 
		
		}
		
	}


```
====================

synconization :

---

use semaphore to make a Monitor

```
Monitor semaphore {


	int value ;
	condition_variables queue; 

	void wait () {
	
		while( value == 0 ){
		
			cwait(queue);
			
		}
		
		value--;
	}
	
	void signal() {
	
		value ++;
		
		csignal(queue); 
	}


}


```

---

use monitor to make semaphore ?

here we use Mesa Monitor 


asusme here is a trhead p comes in , does c.signal()

q is sleeping insde the Monitor

p comes throught , while the other thread also comes into the Monitor

call that second thread m 

notice the p thread wake up the condition_variables q 
but the thread m is also here , which one should it run ???


with mesa Monitor -> we actully do not know 

but the Monitor will only allow one 

if lose , the q will be waiting 

sleep inside the Monitor


====================


use semaphore for Monitor:

struct c{
	int waiters;
	semaphore sem(0);
	semaphore mutex(1);
	
}


cwait(c) {
	// c here is the struct c
	mutex wait();
	
	c.waiters++;
	

	
	mutax signal(); 

}

csignal(c) {

	mutex.wait();
	if(c.wait >0){
		c.waiters--;
	}
	
	mutex.signal(); 
}


=============================

File system lecture 


boot_block | super block | i-node table | data blocks 


execuret a command :

open ("/suer/abc/xyz.c" , "r"); 

the File conctrol block( FCB)

the FCB_id here is the i-node_id


for k / path directory , we need 2k +1 to access the first bytes of that file 

why 2k + 1 ??

as within the i-node table
we need to do data lookup 

for each compmenet on the path , we need to do lookup 
-> that is 2 times visite and the final byte access need the +1 

=========

eg: read n-th byte

do seek opeartaion 

seek(fd , n);

read(df , boot ,1 )

if n is 5000 

assume block is 1024 bytes 

we do 5000 / 1024 

that is 4096 and need additainal 904 in another block 

so we have blocks 

0th 1st 2nd 3rd 4th , within the 4th has offset 904 



---

for 100,000 

that is 

12 * 1024 + 256 * 1024  


recall the formula:

```
12B + (B/4) B + (B/4)^2 B + (B/4) ^3 B ...

```

for less than 12 B it is direct access region 
and when more than that , goes to next layer ...