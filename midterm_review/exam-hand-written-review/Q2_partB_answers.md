# Questions_2 — my answers — Part B: fork / exec / wait code, pipes (p3–5)

============================

B12. What would be the output from the program at LINE C and LINE P?

(Optional: threads haven't been taught yet, so this is unlikely on the midterm. The fork part still matters: the parent's value stays 0 because fork gives the child its own copy, same as B17.)

**LINE C: `CHILD: value = 5`**   **LINE P: `PARENT: value = 0`**

(My first try said the child prints 0. ✗ The parent part was right.)

Why the child prints 5: a thread runs INSIDE its process and shares that process's address space, including globals. The thread runs runner(), which sets the global value = 5 in the CHILD's memory. pthread_join waits for the thread to finish, so by LINE C the child's value is already 5.

Why the parent prints 0: fork gave the child its own COPY of the address space. The thread changed the child's copy only. The parent's value was never touched, so it's still 0. (wait(NULL) just makes the parent print after the child is done; it doesn't share any data.)

Rule: threads share memory (same process); fork'ed processes do NOT (each has its own copy).

===========================

B13. How many processes are created in the following program?

**16 processes in total (2⁴), i.e. 15 new ones created besides the original.** ✓

Each pass through the loop, EVERY existing process calls fork, so the count doubles: 1 → 2 → 4 → 8 → 16.
Careful with the wording: "how many processes exist at the end" = 16, "how many are created (new)" = 15. Write both to be safe.
(The children also continue the loop from where the parent was, since fork copies i too. That's why it doubles instead of adding 1 each time.)

==============================

B14. When will the program reach LINE J (i.e. execute printf("LINE J"))?

**Only if execlp fails** (e.g. /bin/ls doesn't exist or can't be run). ✓

Why: on success, exec REPLACES the child's whole memory image with the ls program. The old code, including printf("LINE J"), is gone, so exec never returns. Only on failure does exec return (-1) and the next line runs.
(Also: if fork itself fails, there's no child at all, so LINE J is never reached either.)

=============================

B15. What are the pid values in the following program? (Textbook version assumes the actual pids of the parent and child are 2600 and 2603.)

**A = 0, B = 2603, C = 2603, D = 2600.** ✓

fork() returns twice: 0 in the child, the child's pid in the parent, (-1 if it fails). getpid() always returns the CALLER's own pid.

- A (child, pid): fork returned 0 in the child → 0
- B (child, pid1 = getpid()): the child's own pid → 2603
- C (parent, pid): fork returned the child's pid → 2603
- D (parent, pid1 = getpid()): the parent's own pid → 2600

============================

B16. What is a zombie process?

A process that has **exited but has not yet been reaped by its parent** (the parent hasn't called wait for it yet).

- It no longer runs (no CPU), but its entry in the process table (PCB) stays, holding its pid and exit status, so the parent can collect them with wait.
- (My first try said "no memory resource". ✗ Careful, this was my L04 weak spot: in xv6 a zombie STILL has its address space; freeproc() frees its memory and page table only when the parent's wait reaps it.)
- If the parent never calls wait, zombies pile up and use up process-table slots.
- If the parent exits first, its children are re-parented to init, which calls wait in a loop and reaps them.

===============================

B17. What output would we get at LINE X and LINE Y in the following program?

**LINE X: `CHILD: 0 CHILD: -1 CHILD: -4 CHILD: -9 CHILD: -16`**
**LINE Y: `PARENT: 0 PARENT: 1 PARENT: 2 PARENT: 3 PARENT: 4`**

Child: nums[i] *= -i multiplies each element by −i:
0 × 0 = 0, 1 × (−1) = **−1** (my first try wrote 1 ✗, sign slip), 2 × (−2) = −4, 3 × (−3) = −9, 4 × (−4) = −16.

Parent: still the original 0 1 2 3 4. ✓ The child changed its own COPY of the global array (fork copies the address space), so the parent's nums is untouched. wait(NULL) only makes the parent print after the child finishes.

Same rule as B12: separate processes = separate memory, even for globals.

================================

Concepts first: file descriptors, redirection, pipes, sockets

(Prof's notes only touch these briefly: pipe is listed as the "communication" system call, and the fork/exec section says the gap between fork and exec is where a shell "can close a file and open another in its place to redirect output, connect a pipe, or change directory". Sockets are not in the notes. So know the ideas below; the exam won't go deep.)

**1. File descriptors (fd)**

Every process has a small table of open files, indexed by small numbers called file descriptors. By convention:
- fd 0 = standard input (normally the keyboard)
- fd 1 = standard output (normally the screen)
- fd 2 = standard error (normally the screen)

A program just does read(0, …) and write(1, …). It doesn't know or care what fd 0 and fd 1 are actually connected to: a terminal, a file, or a pipe. That's what makes redirection and pipes work.

Two rules that matter:
- open() always returns the LOWEST free fd number.
- fork() copies the fd table to the child, and exec() keeps the fd table (it replaces the code and memory, not the open files).

**2. Output redirection: `ls > out.txt`**

The shell makes ls write into a file instead of the screen, without ls knowing:

```c
int pid = fork();
if (pid == 0) {                                   // child, before exec
    close(1);                                     // free fd 1 (stdout)
    open("out.txt", O_WRONLY | O_CREATE);         // lowest free fd is 1 → out.txt is now fd 1
    exec("ls", args);                             // ls writes to fd 1 → goes into out.txt
}
wait(0);
```

(Same idea with dup: open the file, then dup it onto fd 1.) Input redirection `wc < in.txt` is the same with fd 0.

This is exactly why fork and exec are two separate calls: the child can rearrange its fds in between, then exec keeps them.

**3. Pipes: `ls | wc`**

pipe(fd) creates a one-way channel inside the kernel (a small buffer) and gives two fds: fd[0] = read end, fd[1] = write end. Bytes written into fd[1] come out of fd[0], in order.

```c
int fd[2];
pipe(fd);
if (fork() == 0) {            // child 1: ls
    close(1); dup(fd[1]);     // stdout → pipe write end
    close(fd[0]); close(fd[1]);
    exec("ls", ...);
}
if (fork() == 0) {            // child 2: wc
    close(0); dup(fd[0]);     // stdin ← pipe read end
    close(fd[0]); close(fd[1]);
    exec("wc", ...);
}
close(fd[0]); close(fd[1]);   // parent (shell) closes both ends
wait(0); wait(0);
```

Properties:
- one-way, a stream of bytes, no message boundaries
- both programs run AT THE SAME TIME; the kernel buffer connects them. If the buffer is full, the writer blocks; if it's empty, the reader blocks (the I/O wait queue idea from A9). When all write ends are closed, the reader gets end-of-file.
- only works between RELATED processes on the SAME machine: the pipe's fds are inherited through fork, there's no name to find it by.

**4. Sockets (just the idea)**

A socket is a communication endpoint that can connect processes on DIFFERENT machines over a network (or on the same machine). Two-way, and it has an address (IP + port) so unrelated programs can find each other. More setup (create, bind/connect, listen/accept) and more overhead (network protocol stack).

Pipe vs socket in one line: pipe = simple, fast, one-way, same machine, related processes (parent/child). Socket = general, two-way, any processes, even across machines.

---

B18. Give an example where pipes are more suitable than sockets.

When two RELATED processes on the SAME machine just need to pass a stream of data one way, e.g. a shell pipeline like `ls | wc -l` or `cat log.txt | grep error`, where the output of one program feeds the input of the next.

Why a pipe is better there: it's created with one call (pipe), the children inherit it through fork, it needs no address or network setup, it goes through a simple kernel buffer instead of the network stack (less overhead), and one-way is exactly what's needed. A socket would work but is overkill: it's built for two-way communication between unrelated processes, possibly on different machines.

================================

B19. Briefly explain how output redirection is implemented.

The shell does it between fork and exec, using the fd rules above. For `cmd > file`:

1. The shell forks a child.
2. In the child: close(1) (stdout), then open(file). open returns the lowest free fd, which is 1, so fd 1 now refers to the file. (Or: open the file, then dup it onto fd 1.)
3. The child calls exec(cmd). exec keeps the fd table, so the new program writes to fd 1 as usual, and the output lands in the file.
4. The parent shell waits.

The program itself never knows its output was redirected; it just writes to fd 1. This is a main reason fork and exec are separate calls.

================================

B20. Can we replace a pipe with output and input redirection?

Partly: you can get the same final result for simple cases, but it is NOT a real replacement.

`ls | wc` could be done as `ls > tmp.txt` then `wc < tmp.txt`: redirect the first program's output into a temporary file, then redirect the second program's input from that file. Same output.

But the differences:
- Sequential vs concurrent: with the temp file, ls must FINISH before wc starts. With a pipe both run at the same time, and wc starts working on data as soon as it arrives.
- Disk vs memory: the temp file is written to and read from disk (slower, uses disk space, must be deleted afterwards). A pipe is a small in-memory kernel buffer.
- Unbounded / never-ending output: a pipe works even if the first program produces endless output (e.g. `yes | head -5`), because the kernel blocks the writer when the buffer is full. A temp file would grow forever and the second program would never start.

So redirection can imitate a pipe for small, finite outputs, but a pipe gives concurrency, no disk I/O, and flow control.
