# Goroutines


Goroutines are a way of doing tasks concurrently in golang. They allow
us to create and run multiple methods or functions concurrently in the
**same address space** inexpensively.

Goroutines are lightweight abstractions over threads because their 
creation and destruction are very **cheap** as compared to threads, and 
they are **scheduled over OS threads**.


## Goroutines vs. Threads
Goroutines are **NOT** any faster than threads.

### Memory consumption

The creation of Goroutines require **much less memory** as compared to threads. They are designed in a way that stack size of Goroutines can **grow and shrink** according to the need of an application.

### Setup and Teardown cost
Threads have significant setup and teardown costs because it has to **request resources from the OS** and **return it once it's done**. While Goroutines are created and destroyed by the **go runtime** (it manages scheduling, garbage collection, and the runtime environment for Goroutines) and those operations are pretty cheap.

### Switch cost
**Threads** are scheduled **preemptively** (if a process is running for more than a scheduler **time slice**, it would preempt the process and schedule execution of another runnable process on the same CPU), the scheduler needs to save/restore all registers i.e. 16 general purpose registers, PC (Program Counter), SP (Stack Pointer), segment registers etc.
While **Goroutines** are scheduled **cooperatively**, they do not directly talk to the OS kernel. When a Goroutine switch occurs, very few registers like PC and SP need to be saved/restored. https://electronics.stackexchange.com/questions/115286/what-processor-registers-are-saved-and-recovered-in-a-context-switch

## Scheduling of Goroutines
In cooperative scheduling Goroutines yield the control periodically when they are idle or logically blocked. The switch between Goroutines happen only at well defined points - when an explicit call is made to the go runtime scheduler. And those well defined points are:
- Channels send and receive operations, if those operations would block.
- The Go statement, although there is no guarantee that the new Goroutine will be scheduled immediately.
- Blocking syscalls like file and network operations.
- After being stopped for a garbage collection cycle.

Go uses three entities to explain the Goroutine scheduling.
1. Processor (P)
2. OSThread (M)
3. Goroutine (G)

In any particular Go application the number of threads available for Goroutines to run is equal to the GOMAXPROCS, which by default is equal to the number of cores available for that application.

![](/forgetful/images/golang-goroutine-scheduling.png)
