## calss 14

continue lecture 9 :

=======================

theree semaphores :

mutex protects the queue
itself

empty stops the producer
when the buffer is full; it
sleeps, not spins

full stops the consumer when
the buffer is empty

Each signal() wakes a waiting
process if there is one;
otherwise it just raises the
count

---

eg:

semaphore mutex = 1; // protects the buffer
semaphore empty = N; // counts empty slots
semaphore full = 0; // counts full slots

producer() {  
while (TRUE) {  
item = produce_item();  
wait(&empty);  
wait(&mutex);  
insert_item(item);  
signal(&mutex);  
signal(&full);  
}  
}

consumer() {
while (TRUE) {
wait(&full);
wait(&mutex);
item = remove_item();
signal(&mutex);
signal(&empty);
consume_item(item);
}
}

"mutex" is mutual exclusion

=======================================

assume n - slots

given

semaphore empty = 0 ;
semaphore full = 0 ;
semaphore mutex = 1;

// producer

while(true){

wait(empty)

        item = produce_item();
        wait(&empty);
        wait(&mutex);
        insert_item(item);
        signal(&mutex);
        signal(&full);

}

// consumer

while(true){

    wait(&full);
    wait(&mutex); // wait for an item into
    item = remove_item();
    signal(&mutex);
    signal(&empty);
    consume_item(item); buffer

}

Notice: we the semaphore we can handle the "consumer" "producer" problem

===============================================

Synchronization in the type

Semaphores are easy to misuse: swap two
wait() calls and the program can deadlock

• A monitor is an abstract data type: shared
data, the only procedures that may touch
it, and initialization code

• Only one thread can be active inside a
monitor at a time, so the mutual exclusion
comes for free

• You enter by calling one of its procedures;
the rest wait in the entry queue

---

Waiting inside
• cwait(c) puts the caller to sleep on condition c and lets another thread into the monitor.

A sleeping thread is not active inside it

---

The buffer as a monitor:

// This part hid the semeaphore (Must be inside!)
monitor x = boundbuffer();

producer() {
x.append(0);
}

question: of how is his simialr / diff
from the semaphores handling of producer consumer ???

==============================
inside the monitor boundbuffer:

---

monitor boundbuffer {

<!--  counte is how many slots of the buffer is taken -->

append(x){

    if (count == N) cwait(notfull);
    buffer[nextin] = x;
    nextin = (nextin + 1) % N;
    count++;
    csignal(notempty);

}

take(){
if (count == 0) {
cwait(notempty);
}

    *x = buffer[nextout];
    nextout = (nextout + 1) % N;
    count--;
    csignal(notfull);

}

}

============================

eg: the boundered buffer
3 slots

01|02|03

assume the input is 3

bb.wait(3)

bb.append(01)

n = 0 -> 1
n = 1 -> 2
n = 2 -> 5

bb.take()

csignal(notfull); // this rease the append

comes the 4th append

nwait(mutex)

---

idea: we have 2 semaphore come to use freely

=============================

eg: Bank balance

we have

---

"using momitors :

withdraw(x) | deposite(y)

---

Monitor BankBalance() {
balance = 0

// want to withdraw x , need at least x within the bank

    withdraw(x){
        if(balance > x){
            balance = balance - x;
            return x ;
        }
    }

    deposite(y){
        balance = balance + y ;
    }

}

why we do not have race considertion here ???

# as inside "monitor" we garanuteed taht only one thread at time !!

think the "monitor" like a box
that also like a queue , inside the box there is only one active thread at any given tiem

this is why we do not have any race condition

============================================

The problem:
the thread might stay inside the monitor
if we sleep inside the monitor how does the wake that thread work ??

"CONDITION VARIABLES"

---

Waiting inside

• cwait(c) puts the caller to sleep on condition c and lets another thread into the monitor. A sleeping thread is

not active inside it

• csignal(c) wakes one thread waiting on c. If none is waiting, the signal is lost

• Hoare semantics: the woken thread runs at once, and the signaller waits in the urgent queue

• Mesa semantics: the signaller keeps running; the woken thread runs later, and the condition may be false again by then. Almost every real system is Mesa

==============================

BOUNDED BUFFER, AGAIN

The buffer as a monitor

<!--


void append(char x) {

    // condition variable "notfull"
    if (count == N) cwait(notfull);

    buffer[nextin] = x;
    nextin = (nextin + 1) % N;
    count++;
    csignal(notempty);
}

void take(char *x) {

    // condition variable "notempty"
    if (count == 0) cwait(notempty);

    *x = buffer[nextout];
    nextout = (nextout + 1) % N;
    count--;
    csignal(notfull);

}


-->

exercise :

Following this idea , try to use "conditinal varible" to help the banking probrom !

========================================

## lecture 10 - File systems

Questions

(1) We have in-memory data structures (arrays, structures, binary blobs, strings, etc).
Why do we need a different abstraction called files for secondary storage?

(2) Why “open” is a key operation with files?

(3) What kind failures can we have with file systems?

============================

File Concept:

File is a named collection of related information
that can be accessed by addressing

File can have heterogeneous types of data

• Creator is responsible for defining the embedded
data types, convention for access, etc

• File can be text, executable, media, data, etc

• File has two different views:

• Logical view - view as users, applications, see it

the way user going to view

• Physical view - as it actually resides in storage

---

why do we use "block" base view?

it make the load from disk to memeory faster
we want to reduce the amount travel times between memory and the disk

think of it like the shiping box , when have many little things better ship them together with bigger box

---

Notice: the block concept here is NOT a must
for computer , old PULM PDA devices does not have that

=====================================

Questions

• File system can be operated at file level or block level. A file system can be copied at
a file level or block level. When would you use file-based copying and when would
you use block-based copying?

• Which would copy a file system faster ???

====================================

File Attributes

• File attributes vary from OS to OS, but following are
common:

• Name: Symbolic name in human readable form
• Identifier: A unique tag for identification within the file
system
• Location: Information about the device holding the file, and
location inside the device
• Size: Current size of the file, number of blocks allocated
• Protection: Access control information, reading, writing
permissions
• Timestamps: last accessed, created, modified, etc

======================================
File Operations

• OS provides many operations on a file, here are the common ones (implemented using system calls)

(1) Creating a file: Create file dir/XYZ

Select “dir/“ and check if create permissions are available.

• Insert XYZ in the directory map.

• Space allocation on storage is done by marking certain blocks are used for XYZ

• Update directory map to point to the storage allocation

---

(2) Opening a file: Open file dir/XYZ

Check permissions to open the file

• Needs to succeed for any future operation with the file

eg:

// open() also return the privilage
fd = open("file2.text" , "r");

if(fd ...) {
// need to check the privilage
}

---

(3) Writing a file: write on file dir/XYZ

Permission obtained at open need to include write
• Location in file is important
• Can we sequential or even random

===========================

Three major part of file operation :

(1) Disk:

All persistent data structures are
stored back here.

File system operations rely on data  
stored in the disk. So, accessing
them is very different from  
other programs.

---

(2)Memory (Kernel)

All file operations are actually
fully implemented here.

Open(), Read(), Write() actually
implemented here because
they need privileged execution.
Lot of data structures like
buffers, etc are held here too.

---

(3) Process

Some file operations are  
happening here.

Open(), Read(), Write() all are
triggered from the process  
space and will get values from  
there.

=====================================

Access Methods

(1) Sequential Access Method

Information in the file is accessed in order, one record (byte stream) at a time

• Editors and compiler access files this way

• Provided two methods: read_next(), and write_next()

• Read_next() — reads the next record in the file and advances the pointer

automatically so the portion after the read one will be next

• Write_next() — appends to the end of the file and advances to the end of the newly
written portio

(2) Direct Access
