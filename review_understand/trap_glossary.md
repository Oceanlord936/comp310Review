# Trap glossary (xv6 / RISC-V)

Reference for the terms in 4.2–4.5. The "how it fits together" section at the end ties them into one trip.

## Trap code (functions and assembly labels)

| Name | Written in | Where it lives | What it does |
|---|---|---|---|
| **uservec** | assembly | trampoline page | First code that runs on a trap from user mode. Saves user registers into the trapframe, switches to the kernel stack and page table, jumps to `usertrap`. |
| **usertrap** | C | kernel | Reads `scause` and decides what the trap was (syscall, device interrupt, fault). Calls `syscall()`, `clockintr`, `yield` and so on. |
| **usertrapret** | C | kernel | Prepares the return to user mode: points `stvec` back at `uservec`, fills in the kernel values in the trapframe, sets up `sepc` and `SPP`. Then calls `userret`. |
| **userret** | assembly | trampoline page | Switches to the user page table, restores registers from the trapframe, runs `sret`. |
| **kernelvec** | assembly | kernel | Trap entry for traps that happen **while already in the kernel**. Saves registers on the kernel stack, calls `kerneltrap`. |
| **swtch** | assembly | kernel | Saves the current kernel `context` and loads another one. This is the actual process switch (3.7). |

## CSRs (special CPU registers)

| Name | Meaning |
|---|---|
| **stvec** | Address the CPU jumps to on a trap. Points at `uservec` in user mode and `kernelvec` in the kernel. |
| **sepc** | The PC at the moment of the trap, saved by hardware. |
| **scause** | Why the trap happened (syscall, timer, page fault, and so on). |
| **sstatus** | Status bits. Includes the three below. |
| **SIE** | Bit in `sstatus`. Supervisor interrupts enabled. Hardware clears it on a trap. |
| **SPIE** | Bit in `sstatus`. Holds the old SIE so `sret` can restore it. |
| **SPP** | Bit in `sstatus`. Previous privilege mode (user or supervisor) that `sret` returns to. |
| **sscratch** | A spare register. Holds `TRAPFRAME` while in user mode so `uservec` has a base address to save registers to. |
| **satp** | Points at the current page table. Writing it switches address spaces. |

## Instructions

| Name | Meaning |
|---|---|
| **ecall** | User instruction that deliberately traps into the kernel (a system call). |
| **sret** | Return from a trap: restores the mode from `SPP`, `SIE` from `SPIE`, and the PC from `sepc`. |
| **csrw / csrr / csrrw** | Write, read, or swap a CSR. `csrrw a0, sscratch, a0` swaps `a0` and `sscratch`. |
| **sfence.vma** | Flushes cached address translations (the TLB) after changing `satp`. |

## Names for memory and structs

| Name | Meaning |
|---|---|
| **TRAMPOLINE** | Fixed virtual address of the trampoline page (`MAXVA - PGSIZE`). |
| **TRAPFRAME** | Fixed virtual address of the trapframe page, just below `TRAMPOLINE`. |
| **p->trapframe** | Kernel's pointer to the process's trapframe physical page. |
| **epc** | Field in the trapframe. Saved user PC, copied from `sepc`. |
| **kernel_satp / kernel_sp / kernel_trap** | Trapframe fields holding the kernel page table, kernel stack pointer, and address of `usertrap`. |
| **context** | Struct of kernel registers saved by `swtch`. Different from the trapframe. |

## Registers

| Name | Meaning |
|---|---|
| **sp** | Stack pointer (`x2`). |
| **a0–a7** | Argument and return-value registers. |
| **t0–t6, s0–s11** | Temporary and saved registers. |

## Page-table permission bits (PTE)

| Bit | Name | Meaning |
|---|---|---|
| V | Valid | Entry in use. If 0, any access faults. |
| R | Read | Loads allowed. |
| W | Write | Stores allowed. |
| X | eXecute | Instruction fetch allowed (marks code). |
| U | User | User mode may access. Without it, only supervisor mode can. |
| G / A / D | Global / Accessed / Dirty | Exist in hardware; xv6 labs don't rely on them. |

## How it fits together

**Trap from user mode (e.g. timer interrupt):**

1. **Hardware, all at once:** copy PC to `sepc`, write `scause`, record old mode in `SPP`, clear `SIE`, switch to supervisor mode, set PC = `stvec` (which is `uservec`). It does *not* change `sp` or `satp`.
2. **uservec** (trampoline): swap `a0` with `sscratch` to get the trapframe address, save all registers there, load `kernel_sp`, `kernel_trap`, `kernel_satp`, write `satp` (page-table switch, safe because the trampoline is at the same VA in both tables), jump to `usertrap`.
3. **usertrap** (C): read `scause`, handle the cause, possibly call `yield` (which goes through `swtch` to another process).
4. **usertrapret** (C): set `stvec` back to `uservec`, fill trapframe kernel values, set `SPP` and `sepc`, call `userret`.
5. **userret** (trampoline): write user page table to `satp`, restore registers from the trapframe, `sret` (restores mode, `SIE`, and PC).

**Key points to remember:**
- The trampoline is **code**, one physical page shared by every page table at the same VA. The trapframe is **data**, one page per process.
- Both have no U bit: hardware in supervisor mode can use them, user mode can't.
- The trapframe is saved on **every** user-mode trap, before the cause is known. A process switch is a separate, conditional decision.
- Page tables map VA to PA one way. Several VAs can map to one PA (that's the trampoline case).
- Traps taken while already in the kernel use `kernelvec` and the kernel stack, not the trapframe.
