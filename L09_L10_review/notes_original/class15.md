## lecture 10 file system

Access Methods
Sequential Access Method

---

Information in the file is accessed in order,
one record (byte stream) at a time

• Editors and compiler access files this way

• Provided two methods: read_next(), and write_next()

• Read_next() — reads the next record in the file and advances the pointer
automatically so the portion after the read one will be next

• Write_next() — appends to the end of the file and advances to the end of the newly
written portion

===============================

Direct Access

File is made up of fixed length logical records

• Allows the program to read and write any
record in the file

• We can emulate sequential access using direct
access files

• Direct access files can be used to build other
types — indexed files, etc

==================================

Directory Structure

Directory is a symbol table translating the file names to file control blocks - file
control blocks will tell us where to find the data in the file

• Directory must allow the following operations:

• Create a file: new files added to the directory

• Delete a file: deleted file can leave hole in the directory

• List a directory: we need to list the contents of a directory

• Rename a file: change name and other attributes of a file

• Traverse the file system:

===================================

Single-Level Directory

• Simplest directory structure
• Files must have unique names
• Could be useful for embedded devices

===================================

Two-Level Directory

In a two-level directory, each user has a single level user file directory (UFD)

• UFD lists files for a particular user

• Master file directory (MFD) links all the users

• The MFD is indexed by user name and points to the different UFDs

• Access control for inter-user file access is not well developed

• Dealing with system files such as compilers, loaders, etc is also not well supported

======================================

Tree-Structured Directories

Directory contains a set of sub-directories and files

• Directories are implemented as files with a special
(OS defined) structure

• Process has a current directory - location in the
tree

• Files and directories can be references with relative
paths with respect to current directory or absolute
paths

• Directories can be deleted recursively in some
OSes and others do not allow recursive delete

=======================================

Acyclic Graph Directories

• Acyclic graph directories allows sharing of files and
directories - not copies - links to the same instance

• Modification of a file or directory will be visible
from all locations that link to the instance

• Challenge: to keep cycles out

• Simple idea: link to only files (used by hard links in
UNIX/Linux)

• Soft links in UNIX/Linux allow linking to directories - can lead to cycles - and result in chaos!

======================================

Memory-Mapped Files

• Leverage virtual memory to map files into a process’
memory space

• No need to use read() and write() system calls to
access file - just reading and writing memory is
sufficient

• Writes to a file may not be synchronously written to
disk - in the memory - until the memory mapped file is
closed or memory flushed to disk in page replacement

===================================

leture 11

File System Implementation

---

Storage System Overview

Storage system is the persistent layer in the
memory hierarchy
\
• Storage system is also much slower than the
volatile memory

• Storage system is still directly addressable

• We need some way of mitigating the
performance gap - use blocking

================================
Storage System Overview
why do ue use small box .

==============================
Storage System Overview
/Free Space Management

==============================

Directory is the symbol table that holds information about all directories and files -
points to the file control block - telling you where the data blocks are

Partition A
Boot Block
Partition B
Super
Block
Free Space
Manag.
Partition

=======================================

File Space Allocation

• Directory links the name of a file or subdirectory with a file control block (FCB)

• FCB points to all data blocks

• We need a way of allocating data blocks - file space allocation problem

• File space allocation can be done in three major ways

• Contiguous allocation

• Linked allocation

• Indexed allocation

================================

Contiguous Allocation

File must occupy a set of contiguous blocks

Accessing is easy for any part of the file: both sequential and
direct access are supported
• Finding space for a new allocation is hard due to external
fragmentation
• Growing a file is also hard - no space to grow where the file is
located
• Can copy the whole file to a new location and release the old
blocks
• Compacting (a time consuming activity) is necessary to reduce
external fragmentation

===============================

Linked Allocation

File is a linked list of storage

• Directory contains the first and last blocks of the file

• Each block contains a pointer to the next block

• Linked allocation is effective for sequential access,
but not for direct access

• No external fragmentation

• Block includes a pointer - small storage is lost for
pointers in each block

• File system integrity is lost quite easily - single
corrupt block can destroy files and directories

=====================================

File Allocation Table

• Some blocks on disk are set aside for a file
allocation table

• Directory entry points to the FAT entry for the
file

• Add a new block - insert the new block into the
linked list (could be at the end or in the middle)

• FAT can be cached in memory and disk seeks
can be avoided

• FAT allows efficient direct access

========================================

Indexed Allocation

All the indexes are collected into an index block

• Indexed allocation supports direct access, no external
fragmentation

• Indexed allocation can waste space - internal
fragmentation inside data blocks and index blocks

• Pointer overhead for indexed allocation is more than
linked allocation

=======================================

i-Nodes (Index Allocation in UNIX)

• UNIX i-node is organized using the indexed
allocation scheme

• 12 direct block pointers, 1 single indirect block
pointer, 1 double indirect block pointer, 1 triple
indirect block pointer
B

• Lets say block size is and a disk pointer takes
4 bytes, the largest file size is given by
12 ×B+B/4×B+(B/4)2×B+(B/4)3×B

---

(B/d) to power of how many layer

=================================

Free Space Management

Secondary storage has finite space - so we need
to reclaim storage
• Keep track of free storage blocks and reuse them
in future allocations
• Bitmap or bit vector - each block is represented
by 1 bit
• Many CPUs have bit manipulation instructions
that can scan for free blocks in the bit map
efficiently
