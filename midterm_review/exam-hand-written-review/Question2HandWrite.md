## Questions_2 review (prof's question bank, red-text questions removed)

# Part A: Intro, processes (p1)

A1. What is the difference between uni- and multi-programming systems?

Uni-programming: only ONE program is in memory at a time. It runs until it finishes, and while it waits for I/O the CPU sits idle. It can still run a batch of jobs, but strictly one after another.

Multi-programming: SEVERAL programs are in memory at the same time. When the running one blocks on I/O, the OS switches the CPU to another one, so the CPU is kept busy. (With a timer added, i.e. time-sharing, it also gives better responsiveness.)

=======================================

A2. ★ Why is multi-programming useful even in hardware with a single processing core?

Because a single core would otherwise sit idle while a program waits for I/O. Multi-programming lets another program use the CPU during that wait, so I/O and computation overlap. This raises CPU utilization and throughput, and the jobs finish sooner overall.

Scenario: A and B each do compute 10 ms, I/O 10 ms, compute 10 ms.
Uni-programming: A takes 30 ms, then B takes 30 ms = 60 ms total.
Multi-programming: while A does its I/O, B computes (and the other way round), so both finish in about 40 ms.

===================================

A3. Would multi-programming need more memory than uni-programming? Briefly explain your reasoning.

Yes. Several processes are resident at the same time, so memory must hold all of their images (code, data, heap, stack) at once, plus the OS's per-process structures (PCB, page table) for each one.

Uni-programming only ever holds one program, so it needs less memory in total, but it wastes the CPU while that program waits.

(Side note: virtual memory reduces how much each process needs, because only its working set has to be in RAM, but multi-programming still needs room for more than one process.)

=======================================

A4. You have a single-core computer. Give the type of programs (or example programs) where multi-programming would NOT give better overall performance than uni-programming. Very briefly explain why.

Pure CPU-bound programs with little or no I/O, e.g. a long calculation like finding the next large prime number.

There is no I/O waiting to fill, so switching between programs gains nothing. It only adds context-switch overhead (saving/restoring state, timer interrupts, cold caches and TLB). With several such jobs, each one also finishes later because they take turns, while the total work stays the same.

===================================

A5. Give an example of an I/O-bound application (an application that would create an I/O-bound process).

A web browser: it spends most of its time waiting for the network or for the user's input, with only short CPU bursts in between. (Other examples: a file-copy program, a text editor, a web server.)

=================================

A6. Give an example of a CPU-bound application

A long computation such as finding large prime numbers (or video encoding, compiling a big project, training a machine-learning model). It has long CPU bursts and rarely blocks for I/O.

==================================

A7. ★ Pick a simple C program and determine where each of its variables is located in the memory layout (text, data, bss, heap, stack).

Sample program (find where each numbered item lives: text, data, bss, heap, or stack):

```c
#include <stdlib.h>

int count = 5;              // (1) count
int table[100];             // (2) table
const char *msg = "hi";     // (3) msg   and   (4) the string "hi" itself

int square(int x) {         // (5) the code of square()   (6) parameter x
    int result = x * x;     // (7) result
    return result;
}

int main(void) {            // (8) the code of main()
    static int calls = 0;   // (9) calls
    int local = 10;         // (10) local
    int *p = malloc(4 * sizeof(int));   // (11) the pointer p   (12) the 16 bytes p points to
    p[0] = square(local);
    calls++;
    free(p);
    return 0;
}
```

Answer:


