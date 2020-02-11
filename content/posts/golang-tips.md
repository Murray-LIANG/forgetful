---
title: "Golang Tips"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "golang",
    "tips",
]

comment: true
toc: true
auto_collapse_toc: true
---
# Some Tips of Golang

## Context
https://siadat.github.io/post/context

Context package was initially designed to implement:
1. Request cancellation
2. Deadline

Instead of forcing a function to stop, the caller should **inform** it that its work is no longer needed. Caller sends the information about cancellation and let the function decide how to deal with it. For example, a function could **clean up** and return early when it is informed that its work is no longer needed.

The caller prepares the context and passes it to the callee.
```go
parentContext := context.Background()
ctx, cancel := context.WithDeadline(parentContext, time)
// or
ctx, cancel := context.WithTimeout(parentContext, duration)
```

The direction of the cancellation only **goes down from parent to child**.


The callee checks if the context is cancelled.
```go
func Perform(ctx context.Context) error {
    for {
        SomeFunction()

        select {
        case <-ctx.Done():
            return ctx.Err()
        case <-time.After(time.Second):
            // wait for 1 second
        }
    }
    return nil
}
```

``context.TODO()`` returns an empty context like ``context.Background()``. ``TODO`` is used while **refactoring** functions to support context.


## time.After

Use it in goroutine to wait for some time.
```go
select {
case <-ctx.Done():
    return ctx.Err()
case <-time.After(time.Second):
    // wait for 1 second
}
```