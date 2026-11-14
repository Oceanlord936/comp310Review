## L15 Inter-Process Communication

---

11a. File descriptor table and redirection

Every process starts with 3 streams already open:

```
fd 0  stdin    input  (keyboard)

fd 1  stdout   output (terminal screen)

fd 2  stderr   error messages (also the screen)

```

The file descriptor table (FDT):

	(1)Each process has one, kept in the kernel.
	
	(2)An fd is just an index into that table. Each entry points to an open file or device.
	
	(3)open() returns the lowest free index.
	
	(4)printf writes to fd 1 (it calls write(1, ...)), and scanf reads fd 0 (read(0, ...)).
	
	(5)A child made by fork() gets a copy of the table, so it uses the same files.

```
FDT of a process
index   points to
0       keyboard
1       screen
2       screen
3       abc.txt     <- after fd = open("abc.txt", ...) , fd = 3 (lowest free)

```
---

Output redirection

```
close(1);                        // fd 1 no longer points to the screen , index 1 is free

int fd = open("abc.txt", ...);   // lowest free index is 1 -> fd = 1

printf("hi");                    // printf writes to fd 1 -> goes into abc.txt


before            after close(1) + open
0  keyboard       0  keyboard
1  screen         1  abc.txt     <- printf now goes here
2  screen         2  screen

```
printf doesn't know anything changed. It still writes to fd 1, and fd 1 now points to the file.

---

Input redirection

The same trick with fd 0: close(0), then open the input file, which becomes fd 0. 

Now scanf reads from the file.

Scope: redirection only affects this process, 
plus any children it forks afterwards, since they copy the table. 

This is how the shell does ls > out.txt: in the child, before exec, it redirects fd 1.


============================

11b. Pipes

What it is: a one-way channel inside the kernel. pipe(p) gives you two fds:

```
p[0] = read end
p[1] = write end

write into p[1]  ->  read out of p[0]
```

How ls | wc -l works:

```
1. pipe(p)        both ends open in the shell

2. fork           child copies the FDT -> child has both ends too

3. in ls     : stdout -> write end   (close(1) , then put p[1] at fd 1)

4. in wc     : stdin  -> read end    (close(0) , then put p[0] at fd 0)

5. ls prints normally -> goes into the pipe -> wc reads it as its input

```
---

The one rule that matters:  read() on a pipe returns 0 (end of file) 
-> only when every write end is closed

Close the ends you don't use.

The class code: the parent writes 3 messages, and the child reads in a loop:

```
while ((nbytes = read(p[0], inbuf, MSGSIZE)) > 0)
    printf("%s\n", inbuf);
```

```
why it hangs :
after 3 messages the child calls read again -> pipe is empty
read returns 0 only if NO write end is open
but the child itself still has p[1] open (it got a copy at fork)
-> read waits forever -> child never ends -> parent's wait(NULL) waits forever

fix : child does close(p[1]) before its loop , parent does close(p[1]) after writing
-> no write end left -> read returns 0 -> loop ends

```

---

Question: what u mean close , close(1) and close(0) does what ???

ans: 
	close(fd) empties that slot in the process's file descriptor table. 
	The process stops using that file or device, and the slot number becomes free again.
	
	Nothing is deleted: the screen, the keyboard or the file itself is untouched. 
	Only this process's link to it is removed.
	
```
	close(1) :                     close(0) :
0  keyboard                    0  (empty)   <- slot 0 free
1  (empty)   <- slot 1 free    1  screen
2  screen                      2  screen
```

so , u use what ??? like for read u use the p[0] for the write side u use the p[1] 
so the other give fd for that side is useless , 
like for wirter , the p[0] is not useful and for reader the p[1] is not useful so you close them ???

```
Yes, exactly:

writer (parent) : uses p[1] to write   -> p[0] is useless -> close(p[0])
reader (child)  : uses p[0] to read    -> p[1] is useless -> close(p[1])

```

=========================

11c. Message passing

Two models of IPC:

```
shared memory   : both processes map the SAME memory
                  + faster : no kernel call on each access
                  - you must synchronize yourself (locks , semaphores)

message passing : send() / receive() through the kernel , no shared memory
                  + simpler , good for a small number of messages
                  - each message goes through the kernel -> slower

```

Naming: who you send to.

```
direct   : name the other process   send(B, msg) / receive(A, msg)
           each side needs a handle to the other

indirect : send to a MAILBOX         send(box, msg) / receive(box, msg)
           + sender / receiver can change , program does not need updating
           + one mailbox , many receivers -> load balance (each msg to ONE receiver)
           - more delay , lower throughput

```

---

Mailboxes:

(1)They have to be created and deleted.
(2)If a process creates one, it owns it, and the mailbox is gone when the owner deletes it.
(3)If the OS creates one, it exists independently of the processes.

Blocking vs non-blocking:
```
blocking     = synchronous
  send    : wait until the message is received
  receive : wait until a message is there

non-blocking = asynchronous
  send    : send and continue
  receive : get a message , or null if none
  + more responsive

```

Buffering, based on how many messages the link can hold:

```
zero capacity  : holds nothing -> sender waits for receiver (rendezvous)

bounded (n)    : sender waits only when the buffer is full

unbounded      : sender never waits

```

--- 

analogy in js 

(1) Non-blocking (asynchronous): start the operation, keep going, and get the result later.

```
fetch(url).then(res => { ... });   // send request , DON'T wait

console.log("keeps going");        // runs right away

```
That's like a non-blocking send: the program doesn't stop.

---

Blocking (synchronous): the whole thread stops until the result is there.
```
const data = fs.readFileSync("a.txt");   // whole thread waits here
```

That's like a blocking receive.

async / await is in between:

```
async function load() {
  const res = await fetch(url);   // looks like it waits ...
  ...
}

```

It looks blocking, because the code reads top to bottom.
But it only pauses this function. The JS thread itself isn't blocked: the event loop keeps running other code, and the function resumes when the data arrives.
So underneath, it's still non-blocking. await just makes asynchronous code easier to read.

========================

11d. Sockets and RPC

Sockets

What it is: an endpoint the OS provides for communication, usually across the network.

```
a socket is named by  IP : port
IP   -> which machine          (box to box)
port -> which process on it    (process to process inside the box)
a port is like a simple mailbox


ports below 1024 : well-known services   SSH 22 , HTTPS 443 , SMTP (email)
your programs    : use ports above 1024

```

A connection = a pair of sockets. I
t must be unique, so it's identified by the 4-tuple:

```
(client IP : client port , server IP : server port)

process A on host X : (146.86.5.20:1625 , 161.25.19.8:80)
process B on host X : (146.86.5.20:2625 , 161.25.19.8:80)

same machines , same server port -> the CLIENT PORT makes them different

```

The OS gives each client connection its own port above 1024 (here 1625 and 2625). 
That's how two processes, or two browser tabs, can talk to the same server at once.

---

Client code shape (TCP):

```
sock = socket(PF_INET, SOCK_STREAM, IPPROTO_TCP);    // make the endpoint
connect(sock, (struct sockaddr *)&servAddr, sizeof(servAddr));  // link to server
send(sock, msg, len, 0); 

```

---

RPC (remote procedure call)

The idea: call a function on another machine as if it were local. 
Message passing happens underneath, but the programmer just sees a normal function call.

```
client                                      server
add(3, 4)
 -> client stub : pack "add" , 3 , 4        
    into a message (= MARSHALLING)  ----->  server stub : unpack
                                            look up "add" in its registry
                                            call the real add(3, 4) -> 7
    unpack result , return 7       <-----   pack result (or error) , send back
add returns 7

------------------

client stub : packs name + parameters , sends , waits for the reply

server stub : finds the procedure by name (registry / lookup table) ,
              calls it , sends back the result or the error
			  
marshalling : packing parameters into a message

XDR         : a common data format , needed when client and server
              are different kinds of machines (eg different byte order)

```

Why pass-by-reference is hard: 
in read(fd, buf, n), buf is a pointer. 

A pointer is an address in the client's memory, and it means nothing on the server. 

So RPC has to send the data itself, and copy results back.











