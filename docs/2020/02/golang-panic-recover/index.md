# Golang Panic and Recover


## Panic

Panics are similar to C++ and Java exceptions, but are only intended for run-time errors, such as following a `nil` pointer or attempting to index an array out of bounds.

A panic stops the normal execution of a goroutine:

- When a program panics, it immediately starts to unwind the call stack.
- This continues until the program crashes and prints a stack trace,
- or until the built-in `recover` function is called.

## Recover

The built-in `recover` function can be used to regain control of a panicking goroutine and resume normal execution.

- A call to `recover` stops the unwinding and returns the argument passed to `panic`.
- If the goroutine is not panicking, `recover` returns `nil`.

Because the only code that runs while unwinding is inside **deferred functions**, `recover` is only useful inside such functions.

```go

func main() {
    n := foo()
    fmt.Println("main received", n)
}

func foo() int {
    defer func() {
        if err := recover(); err != nil {
            fmt.Println(err)
        }
    }()
    m := 1
    panic("foo: fail")
    m = 2
    return m
}
```

Above codes print below lines to the console.
```console
foo: fail
main received 0
```

Since the panic occurred before `foo` returned a value, `n` still has its initial `zero` value.

To return a value during a panic, you must use a named return value.

```go

func main() {
    n := foo()
    fmt.Println("main received", n)
}

func foo() (m int) {
    defer func() {
        if err := recover(); err != nil {
            fmt.Println(err)
            m = 2
        }
    }()
    m := 1
    panic("foo: fail")
    m = 3
    return m
}
```

```console
foo: fail
main received 2
```


