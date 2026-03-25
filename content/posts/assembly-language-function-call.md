---
title: "Assembly Language Function Call"
date: 2020-02-23T16:16:24+08:00
lastmod: 2020-02-23T16:16:24+08:00
draft: false
description: ""
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "assembly language",
]

featured_image: ""
featured_image_preview: ""

comment: true
toc: true
auto_collapse_toc: true
math: false
---

## Addressing mode

| Type	    | Form           | Operand Value            | Name                      |
| --------- | -------------- | ------------------------ | ------------------------- |
| Immediate | `$Imm`         | `Imm`                    | Immediate                 |
| Register  | `Ea`	         | `R[Ea]`                  | Register addressing       |
| Memory    | `Imm`          | `M[Imm]`	                | Direct addressing         |
| Memory    | `(Ea)`         | `M[R[Eb]]`	            | Indirect addressing       |
| Memory    | `Imm(Eb,Ei,s)` | `M[Imm+R[Eb]+(R[Ei]⋅s)]` | Scaled indexed addressing |


- `Imm` refers to a constant value, e.g. `0x8048d8e` or `48`
- `Ex` refers to a register, e.g. `%eax`
- `R[Ex]` refers to the value stored in register `Ex`
- `M[x]` refers to the value stored at memory address `x`

## Terms

- function name
- function arguments
- local variables
- return address: a parameter which tells the function where to resume executing after the function is completed.
    It is **invisible** in that it isn't directly used during the function. In most programming languages, this parameter is passed automatically when the function is called. In assembly language, the `call` instruction handles passing the return address for you, and `ret` handles using that address to return back to where you called the function from.
- return value

The way that the variables are stored and the parameters and return values are transferred by the computer **varies** from language to language. This variance is known as a language's **calling convention**, because it describes how functions expect to get and receive data when they are called.

## Stack

Each computer program that runs uses a region of memory called the **stack** to enable functions to work properly. Machine uses the stack to pass function arguments, to store return information, to save registers for later restoration, and for local variables. The portion of the stack allocated for a single function call is called a **stack frame**.

The computer's stack lives at the very top addresses of memory. As the name suggests, stack is a stack data structure with the "top" of the stack growing **from the high value addresses towards low value addresses**.

We use the stack register, `%esp` as a pointer to the top of the stack. To access the value on the top of the stack without removing it, we can do `(%esp)`, which is indirect addressing mode.

![](forgetful/images/stack-frame-structure.png)

## Calling a function
Before executing a function, a program pushes all of the parameters for the function onto the stack in the reverse order that they are documented. Then the program issues a `call` instruction indicating which function it wishes to start. The `call` instruction does two things:
1. First it pushes the address of the next instruction, which is the return address, onto the stack.
2. Then, it modifies the instruction pointer `%eip` to point to the start of the function.

So, at the time the function starts, the stack looks like this:

```
+---------------+
|Argument N     |
+---------------+
|...            |
+---------------+
|Argument 2     |
+---------------+
|Argument 1     |
+---------------+
|Return Address |
+---------------+ <---+ %esp
```

## Setup, Perform task, Cleanup

The setup and cleanup steps are the same across all the function calls.

### Setup

```
pushl %ebp
movl %esp %ebp
```

The first instruction is to save the current base pointer register, `%ebp`. It's the caller's base pointer. When we are done with callee function we need to restore the execution of the caller function. So, we need to save the current base pointer register.

The stack frame is delimited by two pointers: `%ebp` serves as the pointer pointing to the bottom of the stack frame and `%esp` serves as the pointer pointing to the top of the stack frame.

The second instruction is to set `%ebp` for the callee's stack frame. Then we can be able to access the function parameters that are pushed earlier onto the stack by caller function as fixed indexes from the base pointer.

For 32bit systems:
```
 +-----> +---------------+
 |       |Argument N     |
 |       +---------------+ <---+ N*4+4(%ebp)
 |       |...            |
 +       +---------------+
caller's |Argument 2     |
 +       +---------------+ <---+ 12(%ebp)
 |       |Argument 1     |
 |       +---------------+ <---+ 8(%ebp)
 |       |Return Address |
 +-----> +---------------+ <---+ 4(%ebp)
 |       |Old %ebp       |
 |       +---------------+ <---+ (%esp) and (%ebp)
 +
callee's
```

### Using the stack frame
The stack frame could be used for:

- save registers, push all the **callee-save** registers by convention onto the stack.

    **Caller-save registers**: the caller function is responsible saving these register values because the callee is free to override these register values.

    **Callee-save registers**: the callee function must save those registers values by pushing them onto the stack before overwriting them, and restore them before the returning because the caller may need these values for its future computations.

    By convention, registers `%eax`, `%edx`, `%ecx` are classified as caller-save registers, and `%ebx`, `%esi` and `%edi` are callee-save registers.

- local variables
    
    Space for data with no specified initial value can be allocated on the stack by simply decrementing the stack pointer by an appropriate amount.

```
 +-----> +---------------+
 |       |Argument N     |
 |       +---------------+ <---+ N*4+4(%ebp)
 |       |...            |
 +       +---------------+
caller's |Argument 2     |
 +       +---------------+ <---+ 12(%ebp)
 |       |Argument 1     |
 |       +---------------+ <---+ 8(%ebp)
 |       |Return Address |
 +-----> +---------------+ <---+ 4(%ebp)
 |       |Old %ebp       |
 |       +---------------+ <---+ (%ebp)
 +       |%ebx           |
callee's +---------------+ <---+ -4(%ebp)
 +       |Local var 1    |
 |       +---------------+ <---+ -8(%ebp)
 |       |Local var 2    |
 +-----> +---------------+ <---+ -12(%ebp) and (%esp)
```

### Cleanup

When a function is done executing, it does the following three things:
1. It stores its return value in %eax
2. It frees the stack space it allocated by adding the same amount to the stack pointer (i.e., `addl $8 %esp`)
3. It pops off the registers it saved earlier (i.e., `popl %ebx`)
4. It resets the stack to what it was when it was called (it gets rid of the current stack frame and puts the stack frame of the caller back into effect)
5. It returns control back to wherever it was called from. This is done using the `ret` instruction, which pops whatever value is at the top of the stack, and sets the instruction pointer `%eip` to that value

```
movl %ebp, %esp

 +-----> +---------------+
 |       |Argument N     |
 |       +---------------+ <---+ N*4+4(%ebp)
 |       |...            |
 +       +---------------+
caller's |Argument 2     |
 +       +---------------+ <---+ 12(%ebp)
 |       |Argument 1     |
 |       +---------------+ <---+ 8(%ebp)
 |       |Return Address |
 +-----> +---------------+ <---+ 4(%ebp)
callee's |Old %ebp       |
 +-----> +---------------+ <---+ (%ebp) and (%esp)

popl %ebp  # Set %ebp to old %ebp

 +-----> +---------------+
 |       |Argument N     |
 |       +---------------+ <---+ (%ebp)
 |       |...            |
 +       +---------------+
caller's |Argument 2     |
 +       +---------------+
 |       |Argument 1     |
 |       +---------------+
 |       |Return Address |
 +-----> +---------------+ <---+ (%esp)

ret  # Set %eip to Return Address

 +-----> +---------------+
 |       |Argument N     |
 |       +---------------+ <---+ (%ebp)
 |       |...            |
 +       +---------------+
caller's |Argument 2     |
 +       +---------------+
 |       |Argument 1     |
 +-----> +---------------+ <---+ (%esp)
```