# Golang Stack Traces


https://www.ardanlabs.com/blog/2015/01/stack-traces-in-go.html

```go
package main

func main() {
    slice := make([]string, 2, 4)
    Example(slice, "hello", 10)
}

func Example(slice []string, str string, i int) {
    panic("Want stack trace")
}
```

You will get a stack track like this:
```console
01 goroutine 1 [running]:
02 main.Example(0x2080c3f50, 0x2, 0x4, 0x425c0, 0x5, 0xa)
           /Users/bill/Spaces/Go/Projects/src/github.com/goinaction/code/
           temp/main.go:9 +0x64
03 main.main()
           /Users/bill/Spaces/Go/Projects/src/github.com/goinaction/code/
           temp/main.go:5 +0x85
```

Line `main.Example(0x2080c3f50, 0x2, 0x4, 0x425c0, 0x5, 0xa)` is the source code `Example(slice, "hello", 10)` with **parameters showing six hexadecimal**. The key to understanding how the values do match up with the parameters requires knowing the implementation for each parameter type.

In the above example, the first parameter is a slice of strings. A slice is a reference type in Go. This means the value for a slice is a header value with a pointer to some underlying data. In the case of a slice, the header value is a three word structure that contains a pointer to an underlying array, the length of the slice and the capacity.

![](/forgetful/images/golang-stack-traces-slice.png)

The second parameter is a string. A string is also a reference type but this header value is immutable. The header value for a string is declared as a two word structure that contains a pointer to an underlying byte array and the length of the string.

![](/forgetful/images/golang-stack-traces-string.png)




