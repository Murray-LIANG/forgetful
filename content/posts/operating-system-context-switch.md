---
title: "Operating System Context Switch"
date: 2020-03-05T15:41:55+08:00 
lastmod: 2020-03-05T15:42:07+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "operating system",
    "context switch",
]

comment: true
toc: true
auto_collapse_toc: true
---

Context switch mainly involves two mechanisms:

## 1. Kernel entry/exit mechanism

It switches a single running thread from **running usermode code** to **running kernel code** in the context of that thread, and back again.

## 2. Context switch

It switches in kernel mode from running in the context of one thread to another.

## In detail
When `Thread A` calls `sched_yield` and is replaced by `Thread B`, what happens is:
1. Thread A enters the kernel, changing from user mode to kernel mode;
2. Thread A in the kernel context-switches to Thread B in the kernel;
3. Thread B exits the kernel, changing from kernel mode back to user mode.

### Save from user-mode stack to kernel-mode stack when entering the kernel
Each user thread has both a **user-mode stack** and a **kernel-mode stack**. When a thread enters the kernel, the current value of the user-mode stack (`SS:ESP`) and instruction pointer (`CS:EIP`) are saved to the thread's kernel-mode stack, and the CPU switches to the kernel-mode stack - with the `int $80` syscall mechanism, this is done by the CPU itself. The remaining register values and flags are then also saved to the kernel stack.

### Restore from kernel-mode stack to user-mode stack when leaving the kernel
When a thread returns from the kernel to user-mode, the register values and flags are popped from the kernel-mode stack, then the user-mode stack and instruction pointer values are restored from the saved values on the kernel-mode stack.

### Context switch just switches the kernel stacks 

When a thread context-switches, it calls into the scheduler (the scheduler does not run as a separate thread - it always runs in the context of the current thread). The scheduler code selects a process to run next, and calls the `switch_to()` function. This function essentially **just switches the kernel stacks** - it saves the current value of the stack pointer into the TCB for the current thread (called `struct task_struct` in Linux), and loads a previously-saved stack pointer from the TCB for the next thread. At this point it also saves and restores some other thread state that isn't usually used by the kernel - things like floating point/SSE registers. If the threads being switched don't share the same virtual memory space (ie. they're in different processes), the page tables are also switched.

So you can see that the core user-mode state of a thread isn't saved and restored at context-switch time - it's saved and restored to the thread's kernel stack when you enter and leave the kernel. **The context-switch code doesn't have to worry about clobbering the user-mode register values - those are already safely saved away in the kernel stack by that point**.