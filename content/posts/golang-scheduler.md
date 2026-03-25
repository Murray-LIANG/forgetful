---
title: "Golang Scheduler"
date: 2020-02-15T15:06:55+08:00
lastmod: 2020-02-15T15:07:06+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "golang",
    "scheduler",
    "schedule",
]

comment: true
toc: true
auto_collapse_toc: true
---

## What does the Go runtime need with a scheduler?

Why create a userspace scheduler when the OS can schedule threads for you?

### 1. Overhead of kernel threads

The POSIX thread API is very much **a logical extension to the existing Unix process model** and as such, **threads get a lot of the same controls as processes**. Threads have:
- their own signal mask 
- can be assigned CPU affinity
- can be put into cgroups
- can be queried for which resources they use

All these controls add overhead for features that are simply not needed for how Go programs uses goroutines and they quickly add up when you have 100,000 threads in your program.

### 2. Kernel threads scheduling can't be informed
Another problem is that the OS can't make informed scheduling decisions, based on the Go model. For example, the Go GC requires that all threads are stopped when running a collection and that memory must be in a consistent state. This involves waiting for running threads to reach a point where we know that the memory is consistent.

When you have many threads scheduled out at random points, changes are that you're going to have to wait for a lot of them to reach a consistent state. The Go scheduler can make the decision of only scheduling at points where it knows that memory is consistent. This means that when we stop for GC, we only have to wait for the threads that are being actively run on a CPU core.

## Threading models

- N:1, several userspace threads are run on one OS thread.

    Quick to context switch but cannot take advantage of multi-core systems.

- 1:1, one thread of execution matches one OS thread.
    
    It takes advantage of all of the cores on the machine, but context switching is slow because it has to trap through the OS.

- M:N, mix of N:1 and 1:1.

    Quick context switches and take advantage of all the cores in your system. But this model adds more complexity to the scheduler.

    Go scheduler chooses this model. It schedules an arbitrary number of goroutines onto an arbitrary number of OS threads.

## Three main entities

- **M**, for machine, represents an OS thread. It's the thread of execution managed by the OS and works pretty much like your standard POSIX thread.
- **G**, represents a goroutine. It includes the stack, the instruction pointer and other information important for scheduling goroutines, like any channel it might be blocked on.
- **P**, for processor, represents a context for scheduling. You can look at it as a localized version of the scheduler which runs Go code on a single thread.

**In order to run goroutines (G), a thread (M) must hold a context (P).**

The number of contexts is set on startup to the value of the `GOMAXPROCS` environment variable or through the runtime function `GOMAXPROCS()`. Normally this doesn't change during execution of your program. The fact that the number of contexts is fixed means that only `GOMAXPROCS` are running Go code at any point. We can use that to tune the invocation of the Go process to the individual computer, such at a 4 core PC is running Go code on 4 threads.

Goroutines that are not running but ready to be scheduled are arranged in lists called runqueues. Goroutines are added to the end of a runqueue whenever a goroutine executes a `go` statement. Once a context has run a goroutine until a **scheduling point**, it
1. pops a goroutine off its runqueue
2. sets stack and instruction pointer
3. begins running the goroutine.

To bring down mutex contention, **each context has its own local runqueue**. A previous version of the Go scheduler only had a global runqueue with a mutex protecting it. Threads were often blocked waiting for the mutex to unlocked. This got really bad when you had 32 core machines that you wanted to squeeze as much performance out of as possible.

The scheduler keeps on scheduling in this steady state as long as all contexts have goroutines to run.

## Why needs contexts?

Can't we just put the runqueues on the threads and get rid of contexts? Not really. The reason we have contexts is so that we can hand them off to other threads if the running thread needs to block for some reason.

An example of when we need to block, is when we call into a syscall. Since a thread cannot both be executing code and be blocked on a syscall, we need to hand off the context so it can keep scheduling.

![](forgetful/images/golang-scheduler-syscall.jpg)
_Figure copied from http://morsmachine.dk/go-scheduler_

Here we see a thread giving up its context so that another thread can run it. The scheduler makes sure there are enough threads to run all contexts. **M1** in the illustration above might be created just for the purpose of handling this syscall or it could come from a thread cache. The syscalling thread will hold on the goroutine that made the syscall since it's technically still executing, albeit blocked in the OS.

When the syscall returns, the thread must try and get a context in order to run the returning goroutine. The normal mode of operation is to **steal a context** from one of the other threads. If it can't steal one, it will **put the goroutine on a global runqueue**, put itself on the thread cache and go to sleep.

The global runqueue is a runqueue that contexts pull from when they run out of their local runqueue. Contexts also periodically check the global runqueue for goroutines. Otherwise the goroutines on global runqueue could end up never running because of starvation.

This handling of syscalls is why Go programs run with multiple threads, even when `GOMAXPROCS` is `1`. The runtime uses goroutines that call syscalls, leaving threads behind.

## Stealing work

Another way that the steady state of the system can change is when a context runs out of goroutines to schedule to. This can happen if the amount of work on the contexts' runqueues is unbalanced. This can cause a context to end up exhausting it's runqueue while there is still work to be done in the system. To keep running Go code, a context can take goroutines out of **the global runqueue** but if there are no goroutines in it, it'll have to get them from somewhere else.

That somewhere is the other contexts. When a context runs out, it will try to **steal about half of the runqueue from another context**. This makes sure there is always work to do on each of the contexts, which in turn makes sure that all threads are working at their maximum capacity.