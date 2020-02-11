---
title: "Golang Interview Questions"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "golang",
]

comment: true
toc: true
auto_collapse_toc: true
---


## Explain what is go routine in GO? How you can stop go routine?

A goroutine is a function which is capable of running **concurrently** with other functions.

To stop goroutine, you pass the goroutine a signal channel, that signal channel is used to push a value into when you want the goroutine to stop. The goroutine polls that channel regularly as soon as it detects a signal, it quits.

```go
Quit := make(chan bool)

go func() {
    for {
        select {
        case <- quit:
            return
        default
            // do other stuff
        }
    }
}()

// Do stuff

// Quit goroutine
Quit <- true
```

## Why would you prefer to use an empty struct{}? Provide some examples of the good use of the empty struct{}.

1. Save memory. Empty structs do not take any memory for its value.
```go
a := struct{}{}
println(unsafe.Sizeof(a))
// Output: 0
```
2. Show a reader you do not need a value at all.
    - When implementing the `set` data structure.
    ```go
    set := make(map[string]struct{})
    for _, value := range []string{"apple", "orange", "apple"} {
        set[value] = struct{}
    }
    fmt.Println(set)
    // Output: map[orange:{} apple:{}]
    ```

    - When building an object, and only being interested in a grouping of methods and no intermediary data.
    ```go
    type Lamp struct{}

    func (l Lamp) On() {
        println("On")
    }

    func (l Lamp) Off() {
        println("Off")
    }

    func main() {
        // Case #1.
        var lamp Lamp
        lamp.On()
        lamp.Off()
        // Output:
        // on
        // off
    }
    ```

    - When you need a channel to signal an event, but do not really need ot send any data.
    ```go
    func worker(ch chan struct{}) {
	    // Receive a message from the main program.
	    <-ch
	    println("got in worker")
	
	    // Send a message to the main program.
	    close(ch)
    }

    func main() {
	    ch := make(chan struct{})
	    go worker(ch)
	
	    // Send a message to a worker.
	    ch <- struct{}{}
	
	    // Receive a message from the worker.
	    <-ch
	    println("got in main")
	    // Output:
	    // got in worker
	    // got in main
    }
    ```

## How do you compare two structs? What about two interfaces?

- You can compare two structs with the `==` operator, as you would do with other simple types. Just make sure they do **not** contain any slices, maps, or functions, in which case the code will **not be compiled**.
- You can compare two interfaces with the `==` operator as long as the underlying types are "simple" and identical. Otherwise the code will **panic at runtime**.
- Both structs and interfaces which **contain maps, slices (but not function)** can be compared with the `reflect.DeepEqual()` function.
- For **comparing byte slices**, there are nice helper functions in the **bytes** package: `bytes.Equal()`, `bytes.Compare()`, and `bytes.EqualFold()`. The latter for comparing text strings disregarding the case, which are much faster than the `reflect.DeepEqual()`.