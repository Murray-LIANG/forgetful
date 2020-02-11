# Non-Blocking IO in Go



If you are using Go you are probably using non-blocking IO.

What is non-blocking IO? A simple explanation: It allows you to `read()` and `write()` to a file descriptor (that is, any type of open file be it a socket, pipe, a file on disk, whatever) without having these calls block just because the file is not ready.

It's just something like this:
```go
fd, _ := syscall.Open("/foo",
    syscall.O_CREAT|syscall.O_RDWR|syscall.O_CLOEXEC|syscall.O_NONBLOCK,
    0644,
)
```

This is instructing the system to:
1. open the file at `/foo`, create it if it does not exist (`O_CREAT`)
2. close the file if executing a new processes (`O_CLOEXEC`)… this is important to not copy file descriptors between processes unexpectedly
3. open the file with both read and write access
4. use non-blocking mode (`O_NONBLOCK`)
5. set the permissions on new files

For a regular file it means not much because they are always readable and writable. But other types of files, such as a pipe this gets very interesting, so instead we can do this before we open the file:
```go
syscall.Mkfifo("/foo", 0644)
```

This will create a fix-sized pipe buffer. Without the `O_NONBLOCK` flag, when a `read()` is performed, the caller will block until there is data to read. Likewise when a `write()` is performed the caller will be blocked if the pipe is full. Here we are using `O_NONBLOCK`, and so will have slightly different semantics. Instead of blocking, a call to `read()` on an empty pipe or `write()` to a full pipe will return an `EAGAIN` error. This is a nice way of saying the pipe is not ready for that action (the error message might look like `resource temporarily unavailable`). `EAGAIN` really means `try again`, there is an alias for this error called `EWOULDBLOCK`.

From here you might want to use a polling mechanism such as `epoll` to be notified of when the pipe is ready for read or write (depending on what you need).

So, Go does all this for you. When you call `os.Open(...)`, Go opens the file with the non-blocking flag, sets up watches for the file descriptor to know when it's ready for read/write/is-closed, and provides a blocking API on top of non-blocking IO for a natural flow like so:
```go
bug := make([]byte, 32)
n, _ := f.Read(buf)
fmt.Println(string(buf[:n]))
```

So, the `fmt.Println` doesn't happen until `Read` has completed. If the file is not ready for `read()`, it pauses the goroutine and allows other goroutines to run while it waits for it to be ready, then wakes up our goroutine when it is ready so it can continue. This is really nice and simple, don't have to think about callbacks, or polling API's, or any low-level details and get all benefit of asynchronous IO.

But, a blocking API isn't always what you want. Starting with go1.11, there are two new interfaces: `syscall.Conn`, `syscall.RawConn`. These interfaces essentially allow Go to expose the raw file descriptor but without sacrificing any control by the runtime itself to do weird things and allows the caller to still utilize the built-in runtime poller so you don't have to deal with these semantics yourself.
The `Read` and `Write` methods on this interface take a function which gets called in a loop when the file descriptor is ready for the operation, and it's up to you to determine when to hand off back to Go, normally you'd do this when you receive `EAGAIN`.
