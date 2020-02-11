# Multi-Threading and Multi-Processing


**Multithreading** refers to the general task of running more that one thread of
execution within an operating system. **Multithreading** is more generically
called **multiprocessing**, which can include multiple **system processes** (a
simple example on Windows would be, e.g., running Internet Explorer and 
Microsoft Word at the same time), or it can consist of one process that has 
multiple **threads** within it.

**Multithreading** (or should I say, **multiprocessing**) is a **software** concept.
Practically any Turing-complete CPU can perform multithreading, even if the
computer only has one CPU core and that core does not support **hyperthreading**.
In order to support multiprocessing, the CPU will **interleave** execution of 
different threads of execution, by executing one, then another, then another, where
the operating system will divide up the **time** available into "**slices**" and 
give a roughly **equal amount of time to each thread** (the time doesn't have to be 
equal, but that's typically how it's done unless a process requests a higher 
priority).

Note that, whenever there are **more software threads** of execution trying to 
execute at any given time **than** there are available **hardware (simultaneous) 
threads** of execution, then these **software threads** will be "**interleaved**" 
among the **available cores**. In the case of a "uniprocessor" (one CPU core with 
no hyperthreading), if you have more than one software thread, they will always be 
interleaved. If you have a **4-core CPU with hyperthreading**, that's **8 "hardware
threads"**, meaning the CPU can execute 8 simultaneous threads of execution at the 
same instant, so if you had 8 software threads trying to run, they could all run at 
once; but if you had 9 software threads, one of the hardware threads would have to 
interleave a pair of threads (the exact pair of threads chosen would depend on the 
operating system's scheduler implementation).
