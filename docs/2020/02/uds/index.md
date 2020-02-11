# Unix Domain Socket


When it comes to inter-process communication (IPC) between processes on the same Linux host, there are multiple options: 
- FIFOs
- pipes
- shared memory
- sockets and so on
  
`Unix Domain Sockets` combine the convinient API of sockets with higher performance of other single-host methods.

UDS supports streams (TCP equivalent) and datagrams (UDP equivalent).

IPC with UDS looks very similar to IPC with regular TCP sockets using the loop-back interface (`localhost` or `127.0.0.1`), but there is a key difference: performance. While TCP loop-back interface can skip some of the complexities of the full TCP/IP network stack, it retains many others (ACKs, TCP flow control, and so on). These complexities are designed for reliable cross-machine communication, but on single host they're an unnecessary burden.

There are some additional differences. For example, since UDS use paths in the filesystem as their addresses, we can use directory and file permissions to control access to sockets, simplifying authentication.
