# The Rust Programming Language


## Chap 3 Common Programming Concept

### 3.1 Variables and Mutability

By default variables are immutable.

Immutable variables vs. Constants: Constants are valid for the entire time a program runs, within the scope they were declared in, making them a useful choice for values in your application domain that multiple parts of the program might need to know about.

Shadowing: We can shadow a variable by using the same variable’s name and repeating the use of the `let` keyword.

### 3.2 Data Types

Rust is a **statically typed language**, which means that it must know the types of all variables at compile time.

**Scalar Types**: integers, floating-point numbers, booleans and characters. Rust's `char` type is four bytes in size and represents a Unicode Scalar value.

**Compound Types**, two primitive compound types: tuples and arrays.

Tuples have a fixed length: once declared, they cannot grow or shrink in size. We create a tuple by writing a comma-separated list of values inside parentheses. Each position in the tuple has a type, and the types of the different values in the tuple don’t have to be the same.
```rust
let tup: (i32, f64, u8) = (500, 6.4, 1)
let five_hundred = x.0
```

Arrays in Rust are different from arrays in some other languages because arrays in Rust have a **fixed length** like tuples. Arrays are useful when **you want your data allocated on the stack rather than the heap** or when **you want to ensure you always have a fixed number of elements**.
```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];  // a is with 5 elements of i32 type.

let b = [3; 5];  // same as let b = [3, 3, 3, 3, 3]
```

### 3.3 Functions

#### Function Bodies Contain Statements and Expressions

**Statements** are instructions that perform some action and do not return a value. **Expressions** evaluate to a resulting value.

Calling a function is an expression. Calling a macro is an expression. The block that we use to create new scopes, `{}`, is an expression, for example:
```rust
fn main() {
    let x = 5;

    let y = {
        let x = 3;
        x + 1
    };

    println!("The value of y is: {}", y);
}
```

Note the `x + 1` line without a semicolon at the end, which is unlike most of the lines you’ve seen so far. **Expressions do not include ending semicolons.** If you add a semicolon to the end of an expression, you turn it into a statement, which will then not return a value. Keep this in mind as you explore function return values and expressions next.

### 3.4 Control Flows

#### Use `if` in a `let` Statement
```rust
fn main() {
    let condition = true;

    let number = if condition {
        5
    } else {
        6
    };

    println!("The value of number is: {}", number);
}
```

#### Return Values from Loops

```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;
        if counter == 10 {
            break counter * 2;
        }
    };
    println!("The result is {}", result);
}
```

## Chap 4 Understanding Ownership

### What Is Ownership?

#### Ways to manage a computer's memory
1. The language has garbage collection that constantly looks for no longer used memory as the program runs;
2. The programmer must explicitly allocate and free the memory;
3. Rust's ownership approach: memory is managed through a system of ownership with a set of rules that the compiler checks at compile time.

#### The Heap vs. The Stack

In many programming languages, you don't have to think about the stack and the heap very often. But in a systems programming language like Rust, whether a value is on the stack or the heap has more of an effect on how the language behaves and why you have to make certain decisions.

The stack:
1. Adding data is called `pushing onto the stack`.
2. FIFO.
3. All data stored on the stack must have a known, fixed size.
4. Pushing to the stack is faster than allocating on the heap. Just put at the top of the stack.
5. Accessing is faster.

The heap:
1. Data with an unknown size at compile time or a size that might change must be stored on the heap instead.
2. Less organized: when you put data on the heap, you request a certain amount of space. The OS finds an empty spot in the heap and returns a `pointer`.
3. `Allocating`, while pushing values onto the stack is not considered allocating.
4. Slower because the OS must first find a big enough space and perform bookkeeping.
5. Accessing is slower because you have to follow a pointer.

#### Ownership Rules
1. Each value in Rust has a variable that's called its `owner`.
2. There can only be one owner at a time.
3. When the owner goes out of scope, the value will be dropped.

#### Memory and Allocation

**`String` vs. `string literal`**

In the case of a string literal, we know the contents at compile time, so the text is hardcoded directly into the final executable. This is why string literals are fast and efficient. But these properties only come from the string literal's immutability.

`String` type is for supporting a mutable, growable piece of text, we need to allocate an amount of memory on the heap, unknown at compile time, to hold the contents.

When a variable goes out of scope, Rust calls a special function for us. This function is called `drop`.

**Move Ownership**

```rust
let s1 = String::from("hello");
let s2 = s1;
```
![](/forgetful/images/rust-move-ownership.svg)

`s2 = s1` would move `s1` into `s2`. It's like a `shallow copy`. But it's a move because only `s2` is valid after the move. When `s2` goes out of scope, it alone will free the memory.

**NOTE: Rust will never automatically create "deep" copies of your data. Any automatic copying can be assumed to be inexpensive in terms of runtime performance.**

**Clone Data == Deep Copy**

Clone: copy the heap data of the `String` together with the stack data.
```rust
let s1 = String::from("hello");
let s2 = s1.clone();
```

**Copy Stack-Only Data**

Types such as integers that have a known size at compile time are stored entirely on the stack, so copies of the actual values are quick to make.
```rust
let x = 5;
let y = x;
```
`x` is still valid after copy. It wasn't moved into `y`.

**Copy Trait and Drop Trait**

Rust has a special annotation called the `Copy` trait. If a type has the `Copy` trait, an older variable is still usable after assignment.

**NOTE: Rust won't let us annotate a type with the `Copy` trait if the type, or any of its parts, has implemented the `Drop` trait.**

**These Types are Copy**
In general, any group of simple scalar values can be `Copy`, and nothing that requires allocation or is some form of resource is `Copy`.
- All the integer types, such as `u32`.
- The Boolean type.
- All the floating point types.
- The character type, `char`.
- Tuples, if they only contain types that are also `Copy`. For example, (`i32`, `i32`) is `Copy`, but (`i32`, `String`) is not.

#### Ownership and Functions
Passing a variable to a function will **move** or **copy**.

```rust
fn main() {
    let s = String::from("hello");  // s comes into scope

    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so is no longer valid here

    let x = 5;                      // x comes into scope

    makes_copy(x);                  // x would move into the function,
                                    // but i32 is Copy, so it’s okay to still
                                    // use x afterward

} // Here, x goes out of scope, then s. But because s's value was moved, nothing
  // special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens.
```

#### Return Values and Scope

Returning values can also transfer ownership.


### 4.2 References and Borrowing

![](/forgetful/images/rust-references.svg)

`References` allow you to refer to some value without taking ownership of it. Because the variable is a reference and does not own the value, the value it points to will not be dropped when the reference goes out of scope.

#### Mutable References

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);  // mut here
}

fn change(some_thing: &mut String) {  // and here
    some_thing.push_str(", world");
}
```

Mutable references have one big restriction: **you can have only one mutable reference to a particular piece of data in a particular scope**.

This restriction can prevent data races at compile time. A `data race` is similar to a race condition and happens when these three behavior occur:
- Two or more pointers access the same data at the same time.
- At least one of the pointers is being used to write to the data.
- There's no mechanism being used to synchronize access to the data.

```rust
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s;  // can't compile due to data race.
```

But we can create a new scope to allow for multiple mutable references.
```rust
let mut s = String::from("hello");

{
    let r1 = &mut s;

} // r1 goes out of scope here, so we can make a new reference with no problems.

let r2 = &mut s;
```

A similar rule exists for combining mutable and immutable references.
```rust
let mut s = String::from("hello");

let r1 = &s; // no problem
let r2 = &s; // no problem
let r3 = &mut s; // BIG PROBLEM, can't compile

println!("{}, {}, and {}", r1, r2, r3);
```
But this can compile.
```rust
let mut s = String::from("hello");

let r1 = &s; // no problem
let r2 = &s; // no problem
println!("{} and {}", r1, r2);
// r1 and r2 are no longer used after this point

let r3 = &mut s; // no problem
println!("{}", r3);
```

### 4.3 The Slice Type
Another data type that does not have ownership is the `slice`.


#### String Slices
A `string slice` is a reference to part of a `String`. 
```rust
let s = String::from("hello world");

let hello = &s[0..5];
let world = &s[6..11];
```
This is similar to taking a reference to the whole `String` but with the extra `[0..5]` bit.
![](/forgetful/images/rust-string-slice.svg)

The type that signifies `string slice` is written as `&str`.

#### String Literals Are Slices

String literals are stored inside the binary.
```rust
let s = "Hello, world!";
```

The type of `s` here is `&str`: it’s a slice pointing to that specific point of the binary. This is also why string literals are immutable; `&str` is an immutable reference.

#### String Slices as Parameters

We'll write this:
```rust
fn first_word(s: &str) -> &str {
```
not this:
```rust
fn first_word(s: &String) -> &str {
```

If we have a string slice, we can pass that directly. If we have a `String`, we can pass a slice of the entire `String`. Defining a function to take a string slice instead of a reference to a `String` makes our API more general and useful without losing any functionality:
```rust
fn main() {
    let my_string = String::from("hello world");

    // first_word works on slices of `String`s
    let word = first_word(&my_string[..]);

    let my_string_literal = "hello world";

    // first_word works on slices of string literals
    let word = first_word(&my_string_literal[..]);

    // Because string literals *are* string slices already,
    // this works too, without the slice syntax!
    let word = first_word(my_string_literal);
}
```

#### Other Slices
```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];
```
This slice has the type `&[i32]`.

#### The concepts of ownership, borrowing, and slices ensure memory safety in Rust programs at compile time.
#### The Rust language gives you control over your memory usage in the same way as other systems programming languages, but having the owner of data automatically clean up that data when the owner goes out of scope means you don’t have to write and debug extra code to get this control.

## Chap 5 Using Structs to Structure Related Data

### 5.1 Structs

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

fn build_user(email: String, username: String) -> User {
    User {
        email,  // shorthand of email: email because the name is exactly same
        username,  // same here
        active: true,
        sign_in_count: 1,
    }
}

let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    ..user1  // using user1's active and sign_in_count
};

struct Color(i32, i32, i32);  // tuple struct
```

### 5.3 Methods

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // Method of Rectangle with &self, which doesn't take ownership of self.
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other. height
    }

    // Associated function without self.
    // Invoke like `let sq = Rectangle::square(3);`
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}
```

## 6 Enums

Similar to `algebraic data types` in functional languages.

```rust
enum IpAddrKind {
    V4,
    V6,
}

let four = IpAddrKind::V4;

// The enum inside another struct
struct IpAddr {
    kind: IpAddrKind,
    address: String,
}

let home = IpAddr {
    kind: IpAddrKind::V4,
    address: String::from("127.0.0.1"),
};

let loopback = IpAddr {
    kind: IpAddrKind::V6,
    address: String::from("::1"),
};
```

A more concise way using just an enum, rather than an enum inside a struct, by putting data directly into each enum variant. This new definition of the `IpAddr` enum says that **both `V4` and `V6` variants will have associated `String` values**:

```rust
enum IpAddr {
    V4(String),  // notice here
    V6(String),  // and here
}

// No need for an extra struct.

let home = IpAddr::V4(String::from("127.0.0.1"));

let loopback = IpAddr::V6(String::from("::1"));
```

Each variant can have different types and amounts of associated data. Version four type IP addresses will always have four numeric components that will have values between 0 and 255. If we wanted to store `V4` addresses as four `u8` values but still express `V6` addresses as one `String` value, we wouldn’t be able to with a struct. Enums handle this case with ease:

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);

let loopback = IpAddr::V6(String::from("::1"));
```

```rust
// This enum has four variants with different types.
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

// The following structs could hold the same data that the above enum variants hold.
struct QuitMessage;  // unit struct
struct MoveMessage {
    x: i32,
    y: i32,
}
struct WriteMessage(String);  // tuple struct
struct ChangeColorMessage(i32, i32, i32);  // tuple struct
```

The benefit of defining an enum like `Message` is that it's possible for us to define a function to take any of these kinds of messages as we could with the `Message` enum.

We could also define methods on enums like what we do on structs using `impl`.

```rust
// Method defined on enum `Message`.
impl Message {
    fn call(&self) {
        // method body.
    }
}

let m = Message::Write(String::from("hello"));
m.call();
```

```rust
enum Option<T> {
    Some(T),
    None,
}
```
The `Option` enum is so useful that it's even included in the preclude; you don't need to bring it into scope explicitly. And you can use `Some` and `None` directly without the `Option::` prefix.


### 6.2 The `match` Control Flow Operator

The code associated with each arm is an expression, and the resulting value of the expression in the matching arm is the value that gets returned for the entire `match` expression.

```rust
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1  // <<< the expression
        },
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

#### Patterns that Bind to Values

Another useful feature of match arms is that they can bind to the parts of the values that match the pattern. This is how we can extract values out of enum variants.

```rust
#[derive(Debug)] // so we can inspect the state in a minute
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {  // coin's value binds to `state`
            println!("State quarter from {:?}!", state);
            25
        },
    }
}
```

```rust
// Example of Option<T> binding
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

This pattern can be seen a lot in Rust code: **`match` against an enum, bind a variable to the data inside, and then execute code based on it.**

`match` must cover every possible case. Matches in Rust are exhaustive.

#### The `_` Placeholder

```rust
let some_u8_value = 0u8;
match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (),  // just do nothing for other possible values
}
```

### 6.3 Concise Control Flow with `if let`

The `if let` syntax lets you combine `if` and `let` into a less verbose way to handle values that match one patter while ignoring the rest.

```rust
let some_u8_value = Some(0u8);

// Normal
match some_u8_value {
    Some(3) => println!("three"),
    _ => (),
}

// Use `if let`
if let Some(3) = some_u8_value {
    println!("three");
}

```rust
let mut count = 0;
match coin {
    Coin::Quarter(state) => println!("State quarter from {:?}!", state),
    _ => count += 1,
}

// Or use `if let`.
let mut count = 0;
if let Coin::Quarter(state) = coin {
    println!("State quarter from {:?}!", state);
} else {
    count += 1;
}
```

## 7 Managing Growing Projects with Packages, Crates, and Modules

A package can contain multiple binary crates and optionally one library crate.
- Packages
- Crates
- Modules and use
- Paths

### Packages
A package is one or more crates that provide a set of functionality. It:
- must contain at least one crate (either library or binary).
- must contain zero or one library crates, and no more.
- can contain as many binary crates as you'd like.

A package contains a `Cargo.toml` file that describes how to build those crates.

### Crates
A crate is a binary or library.

The **crate root** is a source file that Rust compiler starts from and makes up the root module of your crate. It could be `src/main.rs` or `src/lib.rs`.

`src/main.rs` is the crate root of a binary crate with the same name as the package. While `src/lib.rs` is the crate root of a library crate.

A package can have multiple binary crates by placing files in the `src/bin` directory: each file will be a separate binary crate.

### Modules

Modules control the **privacy** of items.

```rust
// File: src/lib.rs
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}
```

`src/main.rs` and `src/lib.rs` are called crate roots. The reason for their name is that the contents of either of these two files form a module named crate at the root of the crate’s module structure, known as the `module tree`.

```console
# Module Tree
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

### Paths

A path is used to find an item in a module tree.

- An **absolute path** starts from a caret root by using a crate name or literal `crate`, for example, `crate::front_of_house::hosting::add_to_waitlist();`
- A **relative path** starts from the current module and uses `self`, `super` or an identifier in the current module. `super` is like `..` in filesystem navigation.

```rust
mod front_of_house {
    mod hosting {  // cannot compile when missing `pub` here
    // pub mod hosting {
        fn add_to_waitlist() {}  // cannot compile when missing `pub` here
        // pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
    // Think it as `eat_at_restaurant` is in the same MODULE as `front_of_house`.
}
```

#### Use a relative or absolute path? Our preference is to specify `absolute paths` because it’s more likely to move code definitions and item calls independently of each other.

Modules define Rust's **privacy boundary**. If you want to make an item like a function or struct private, you put it in a module.

The way privacy works in Rust is that all items (functions, methods, structs, enums, modules, and constants) are **private by default**.

#### Items in a parent module can’t use the private items inside child modules, but items in child modules can use the items in their ancestor modules.

#### If we use `pub` before a struct definition, we make the struct public, but the struct's fields will still be private.

```rust
mod back_of_house {
    pub struct Breakfast {
        pub toast: String,
        seasonal_fruit: String,  // private field
    }

    impl Breakfast {
        // Because `seasonal_fruit` is private, we need a public associated
        // function to construct an instance of `Breakfast`. Or, we couldn't
        // create an instance of `Breakfast`.
        pub fn summer(toast: &str) -> Breakfast {
            Breakfast {
                toast: String::from(toast),
                seasonal_fruit: String::from("peaches"),
            }
        }
    }
}

pub fn eat_at_restaurant() {
    // Order a breakfast in the summer with Rye toast
    let mut meal = back_of_house::Breakfast::summer("Rye");
    // Change our mind about what bread we'd like
    meal.toast = String::from("Wheat");
    println!("I'd like {} toast please", meal.toast);

    // The next line won't compile if we uncomment it; we're not allowed
    // to see or modify the seasonal fruit that comes with the meal
    // meal.seasonal_fruit = String::from("blueberries");
}
```

#### In contrast, if we make an enum public, all of its variants are then public.

### `use` Keyword

- Bring paths into scope.
- Rename with `as` keyword. `use std::io::Result as IoResult;`
- Re-export names with `pub use`.
- Use nested paths to clean up large `use` lists. `use std::{cmp::Ordering, io};`
- The glob operator. `use std::collections::*;`, use it carefully.

### Separating Modules into Different Files

```rust
// File: src/lib.rs
// `src/lib.rs` is still the crate root.
mod front_of_house;  // defined in file src/front_of_house.rs

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}

// File: src/front_of_house.rs
pub mod hosting;

// File: src/front_of_house/hosting.rs
pub fn add_to_waitlist() {}
```

## 8 Common Collections

### Vectors

```rust
let v: Vec<i32> = Vec::new();

let v = vec![1, 2, 3];  // vec! macro provided by Rust for convenience
```

Two ways to access a value in a vector, either with indexing syntax or the `get` method.
```rust
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];  // cause the program to panic when it references a nonexistent element
println!("The third element is {}", third);

match v.get(2) {  // it gives us an `Option<&T>`, will be `None` if the element doesn't exist
    Some(third) => println!("The third element is {}", third),
    None => println!("There is no third element."),
}
```

```rust
// Get immutable references to each element.
let v = vec![100, 32, 57];
for i in &v {
    println!("{}", i);
}

// Get mutable references to each element.
for i in &mut v {
    *i += 50;  // `*` to dereference `i` to get to the value
}
```

#### Using an Enum to Store Multiple Types

Vectors can only store values that are the same type. We can define an enum whose variants will hold the different value types, and then all the enum variants will be considered the same type.
```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
]
```

### Strings

Rust has only one string type in the core language, which is the string slice `str` that is usually seen in its borrowed form `&str`.

The `String` type is provided by Rust's standard library.

Both `String` and string slices are UTF-8 encoded.

```rust
// Create a String
let mut s = String::new();

// Or
let data = "initial contents";
let s = data.to_string();

// Or
let s = String::from("initial contents");
```

```rust
// Update a String
let mut s = String::from("foo");
s.push_str("bar");
s.push('l');  // add a single character

// Or
let s1 = String::from("Hello, ");
let s2 = String::from("World!");
let s3 = s1 + &s2;

// Or
let s4 = format!("{}{}", s1, s2);
```

#### Understand `+` operator
The `+` operator uses the `add` method with signature: `fn add(self, s: &str) -> String `.

`s2` has an `&`, meaning that we're adding a reference of the second string to the first one.

And `&s2` is `&String` but `+` accepts `&str` for the second string. The compiler can coerce the `&String` argument into a `&str`. When we call the `add` method, Rust uses a `deref coercion`, which here turns `&s2` into `&s2[..]`.

The first parameter of `add` method is `self`, which means `s1` will take ownership of `self`, because `self` does not have an `&`.

#### Indexing into Strings

You can't access individual characters in a string by referencing them by index.
```rust
let s1 = String::from("hello");
let h = s1[0];  // <<< compile error
```

A `String` is a wrapper over a `Vec<u8>`.
```rust
let len = String::from("Hola").len();  // 4, each letter takes 1 byte when encoded in UTF-8

let len = String::from("你好").len();  // 6, each Unicode scalar value takes 3 bytes when encoded in UTF-8
```
Therefore, an index into the string's bytes will not always correlate to a valid Unicode scalar value. `你` is `\xe4\xbd\xa0` when encoded to UTF-8, if `&"你好"[0]` returns the first byte `\xe4`, it is an unexpected value. So, Rust doesn't compile this code at all.

#### Bytes and Scalar Values and Grapheme Clusters

Three relevant ways to look at strings from Rust's perspective: as bytes, scalar values, and grapheme clusters (the closest thing to what we would call `letters`).

Look at the Hindi word “नमस्ते” written in the Devanagari script, it is:

- **bytes**: stored as a vector of u8 values that looks like this:
`[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]`.

**That's 18 bytes and is how computers ultimately store this data.**
- **Unicode scalar**: `['न', 'म', 'स', '्', 'त', 'े']`. The fourth and sixth are not letters.

- **grapheme clusters**: `["न", "म", "स्", "ते"]`. What a person would call the four letters that make up the Hindi word.

#### Slicing Strings

Rust doesn't support using `[]` with a single number, but you can use `[]` with **a range** to create a string slice containing particular bytes.

What would happen if we used `&hello[0..1]`? The answer: Rust would panic at runtime in the same way as if an invalid index were accessed in a vector.

You should use ranges to create string slices with caution, because doing so can crash your program.


#### Methods for Iterating Over Strings

If you need to perform operations on individual Unicode scalar values, the best way to do so is to use the `chars` method. Calling `chars` on “नमस्ते” separates out and returns six values of type `char`. The `bytes` method returns each raw byte.
```rust
for c in "नमस्ते".chars() {
    println!("{}", c);
}

for b in "नमस्ते".bytes() {
    println!("{}", b);
}
```

### 8.3 Hash Maps
```rust
use std::collections::HashMap;  // HashMap is not preclude

let teams  = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];

let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();
```
The type annotation `HashMap<_, _>` is needed here because it’s possible to collect into many different data structures and Rust doesn’t know which you want unless you specify.


For types that implement the `Copy` trait, like `i32`, the values are copied into the hash map. For owned values like `String`, the values will be moved and the hash map will be the owner of those values.
```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// field_name and field_value are invalid at this point, try using them and
// see what compiler error you get!
```

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name);  // Option<&V>, could be `None`

for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```

#### Only Inserting a Value If the Key Has No Value
```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

// The return value of the `entry` method is an enum called `Entry`
// that represents a value that might or might not exist.
// The `or_insert` method on `Entry` is defined to return a mutable
// reference to the value for the corresponding `Entry` key if that
// key exists, and if not, inserts the parameter as the new value 
// for this key and returns a mutable reference to the new value. 
scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores);
```

#### Updating a Value Based on the Old Value

```rust
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);  // count is mut reference
    *count += 1;
}

println!("{:?}", map);
```

## 9 Error Handling

### Rust has no exceptions.

- Recoverable: such as a file not found error, it's reasonable to report the problem to the user and retry the operation.
- Unrecoverable: symptoms of bugs.

`Result<T, E>` for recoverable errors and `panic!` macro that stops execution when the program encounters an unrecoverable error.

### `panic!`

When the `panic!` macro executes, your program will print a failure message, unwind and clean up the stack, and then quit.

#### About unwinding the stack
By default, when a panic occurs, the program starts **unwinding**, which means Rust walks back up the stack and cleans up the data from each function it encounters. But this walking back and cleanup is a lot of work. The alternative is to immediately **abort**, which ends the program without cleaning up. Memory that the program was using will then need to be cleaned up by the operating system.

### `Result`

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

#### How do we know a method/function returns a `Result`?

Take `File::open` as example, we could ask the compiler.

```rust
let f: u32 = File::open("hello.txt");
```
The compiler will tell you:
```console
error[E0308]: mismatched types
 --> src/main.rs:4:18
  |
4 |     let f: u32 = File::open("hello.txt");
  |                  ^^^^^^^^^^^^^^^^^^^^^^^ expected u32, found enum
`std::result::Result`
  |
  = note: expected type `u32`
             found type `std::result::Result<std::fs::File, std::io::Error>`
```

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");
    // `f` is `Ok` that contains a file handle if `open` succeeds,
    // otherwise, `f` is an instance of `Err` that contains more info
    // about the kind of error that happened.

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("Problem opening the file: {:?}", error)
        },
    };
}
```

#### Matching on different errors using `error.kind()`.

Error like `io::Error` has a method `kind` that we can call to ge an `io::ErrorKind` value. The enum `io::ErrorKind` has variants representing the different kinds of errors, i.e. `ErrorKind::NotFound`.
```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => panic!("Problem opening the file: {:?}", other_error),
        },
    };
}
```

#### Shortcuts for panic on error: `unwrap` and `expect`.

If the `Result` value is the `Ok` variant, `unwrap` will return the value inside the `Ok`. If the `Result` is the `Err` variant, `unwrap` will call the `panic!` macro for us.

`expect` method is similar to `unwrap`, but providing good error messages can convey your intent.

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
    let g = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

#### Propagating Errors

Give more control to the calling code, where there might be more information or logic that dictates how the error should be handled than what you have available in the context of your code.

The long version:
```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```

#### `?` Operator

The short version of propagating errors:
```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();
    let mut f = File::open("hello.txt")?;
    f.read_to_string(&mut s)?;
    // Or
    // File::open("hello.txt")?.read_to_string(&mut s)?;
    Ok(s)
}
```
The `?` placed after a `Result` value is defined to work in almost the same way as the `match` expressions. 

#### `?` Can Only Be Used in Functions That Return `Result`

You cannot use it as the return value of `main()` as below.
```rust
use std::fs::File;

fn main() {
    // Can't compile due to `main()` returns `()`.
    let f = File::open("hello.txt")?;
}
```

Use this one:
```rust
fn main() -> Result<(), Box<dyn Error>> {
    let f = File::open("hello.txt")?;

    Ok(())
}
```
The `Box<dyn Error>` type is called a trait object, meaning "any kind of error".

### 9.3 To `panic!` or Not to `panic!`

Returning `Result` is a good default choice when you're defining a function that might fail. Because the calling code could choose to attempt to recover, or it could decide that an `Err` value is unrecoverable, so it can call `panic!` and turn your recoverable error into an unrecoverable one.

In rare situations, it's more appropriate to write code that panics instead of returning a `Result`.

#### Use `unwrap` and `expect` to panic when you're writing an example, prototype or test codes.

#### Use `unwrap` and `expect` to panic when you're writing codes which are logically impossible to fail.
```rust
use std::net::IpAddr;

// Hardcoded IP address's parse will not fail.
let home: IpAddr = "127.0.0.1".parse().unwrap();
```

#### Guidelines for Error Handling

It’s advisable to have your code **panic** when it’s possible that your code could end up in a **bad state**. In this context, a **bad state** is when some assumption, guarantee, contract, or invariant has been broken, such as when invalid values, contradictory values, or missing values are passed to your code—plus one or more of the following:

- The bad state is not something that’s expected to happen occasionally.
- Your code after this point needs to rely on not being in this bad state.
- There’s not a good way to encode this information in the types you use.

#### Creating Custom Types for Validation

```rust
loop {
    // --snip--

    let guess: i32 = match guess.trim().parse() {
        Ok(num) => num,
        Err(_) => continue,
    };

    // The code to validate guess could be repeated everywhere.
    // Define a `Guess` type to check the value is between 1 and 100
    // in its constructor.
    if guess < 1 || guess > 100 {
        println!("The secret number will be between 1 and 100.");
        continue;
    }

    match guess.cmp(&secret_number) {
    // --snip--
}
```

```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess {
            value
        }
    }

    pub fn value(&self) -> i32 {
        self.value
    }
}
```

A function that has a parameter or returns only numbers between 1 and 100 could then declare in its signature that it takes or returns a `Guess` rather than an `i32` and wouldn't need to do any additional checks in its body.

## 10 Generic

### 10.1 Generic Data Types

#### In Function Definitions
```rust
fn largest<T>(list: &[T]) -> T {
    // --snip--
}
```

#### In Struct Definitions
```rust
struct Point<T> {
    x: T,
    y: T,
}
```

#### In Enum Definitions
```rust
enum Option<T> {
    Some(T),
    None,
}
```

#### In Method Definitions
```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
```

By declaring `T` as a generic type after `impl`, Rust can identify that the type in the angle brackets in `Point` is a generic type rather than a concrete type.

```rust
// No `<T>` after `impl`.
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

The method `distance_from_origin` is only on `Point<f32>` instances rather than on `Point<T>` instances with any generic type. It means the type `Point<f32>` will have that method and other instances of `Point<T>` where `T` is not of type `f32` will not have this method defined.

#### Performance of Code Using Generics

**Rust implements generics in such a way that your code doesn’t run any slower using generic types than it would with concrete types.**

Rust accomplishes this by performing `monomorphization` of the code that is using generics at compile time. `Monomorphization` is the process of turning generic code into specific code by filling in the concrete types that are used when compiled.

The compiler looks at all the places where generic code is called and generates code for the concrete types the generic code is called with.

```rust
let integer = Some(5);  // i32
let float = Some(5.0);  // f64
```

The monomorphized version of the code looks like the following. The generic `Option<T>` is replaced with the specific definitions created by the compiler:

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

### 10.2 Traits: Defining Shared Behavior

**Traits are similar to a feature often called interfaces in other languages, although with some differences.**

#### Defining a Trait
```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

#### Implementing a Trait on a Type
```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

#### Restrictions with Trait Implementations

**We can implement a trait on a type only if either the trait or the type is local to our crate.**
For example, we can implement standard library traits like `Display` on a custom type like `Tweet`, and also implement `Summary` on `Vec<T>`.

**But we can’t implement external traits on external types.**
For example, we can't implement the `Display` trait on `Vec<T>` within our crate, because `Display` and `Vec<T>` are defined in the standard library and aren't local to our crate.

#### Default Implementations
```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}

impl Summary for NewsArticle {}  // use the default implementation
```

#### Traits as Parameters
```rust
pub fn notify(item: impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

#### Trait Bound Syntax

The `impl Trait` syntax is actually syntax sugar for a longer form, which is called a `trait bound`:
```rust
pub fn notify<T: Summary>(item: T) {
    println!("Breaking news! {}", item.summarize());
}
```

The trait bound syntax can express more complexity in other cases.

```rust

// item1 and item2 could be different types as long as both types implement `Summary`.
pub fn notify(item1: impl Summary, item2: impl Summary) {
}

// item1 and item2 must be the same type `T`.
pub fn notify<T: Summary>(item1: T, item2: T) {
}
```

#### Specifying Multiple Trait Bounds with the `+` Syntax
```rust
pub fn notify(item: impl Summary + Display) {
    // Can call `summarize` and use `{}` to format item here.
}

// Or
pub fn notify<T: Summary + Display>(item: T) {
    // Can call `summarize` and use `{}` to format item here.
}
```

#### Clearer Trait Bounds with `where` Clauses

```rust
// Trait bounds is too long, making hard to read.
fn some_function<T: Display + Clone, U: Clone + Debug>(t: T, u: U) -> i32 {
}

fn some_function<T, U>(t: T, u: U) -> i32
    where T: Display + Clone,  // <<< use where
          U: Clone + Debug
{
```

#### Returning Types that Implement Traits

The ability to return a type that is only specified by the trait it implements is especially useful in the context of closures and iterators.

```rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        retweet: false,
    }
}
```

However, you can only use `impl Trait` if you’re returning a single type. For example, this code that returns either a `NewsArticle` or a `Tweet` with the return type specified as `impl Summary` wouldn’t work:
```rust
fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        NewsArticle {
            // --snip--
        }
    } else {
        Tweet {
            // --snip--
        }
    }
}
```

#### Using Trait Bounds to Conditionally Implement Methods

By using a trait bound with an `impl` block that uses generic type parameters, we can **implement methods conditionally for types that implement the specified traits**.

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

// All `Pair<T>` always implements the `new` function.
impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self {
            x,
            y,
        }
    }
}

// But only the type `T` which implements `Display` and `PartialOrd` implements `cmp_display`.
impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

**Implement a trait for any type that implements another trait.**

Implementations of a trait on any type that satisfies the trait bounds are called `blanket implementations`.

```rust
impl<T: Display> ToString for T {
    // --snip--
}
```

In **dynamically typed languages**, we would get an error at **runtime** if we called a method on a type that the type didn’t implement. But Rust moves these errors to **compile time** so we’re forced to fix the problems before our code is even able to run. Additionally, we don’t have to write code that **checks** for behavior at runtime because we’ve already checked at compile time. Doing so improves performance without having to give up the flexibility of generics.

### 10.3 Validating References with Lifetimes

#### The Borrow Checker

The borrow checker compares scopes to determine whether all borrows are valid.
```rust
{
    let r;                // ---------+-- 'a
                          //          |
    {                     //          |
        let x = 5;        // -+-- 'b  |
        r = &x;           //  |       |
    }                     // -+       |
                          //          |
    println!("r: {}", r); //          |
}                         // ---------+
```

#### Generic Lifetimes in Functions
```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}

// Can't compile
// The compiler(borrow checker) can't guarantee the reference `longest`
// returned will be still valid in scope. The compiler can't tell whether
// the reference being returned refers to `x` or `y`, what if it refers to
// `x`, but `x` is out of scope when the caller uses the returned reference.
// For example,
// let string1 = String::from("xyz");
// let result;
// {
//     let string2 = String::from("long string is long");
//     result = longest(string1.as_str(), string2.as_str());
// }
// println!("The longest string is {}", result);
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

```console
error[E0106]: missing lifetime specifier
 --> src/main.rs:1:33
  |
1 | fn longest(x: &str, y: &str) -> &str {
  |                                 ^ expected lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but the
signature does not say whether it is borrowed from `x` or `y`
```

#### Lifetime Annotation Syntax

**1. Only works on references.**

```rust
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
```

**2. Not change how long any of the references live.**

**3. Not has much meaning for one lifetime annotation.**
The annotations are meant to tell Rust how generic lifetime parameters of multiple references relate to each other. For example, let’s say we have a function with the parameter first that is a reference to an `i32` with lifetime `'a`. The function also has another parameter named second that is another reference to an `i32` that also has the lifetime `'a`. The lifetime annotations indicate that the references first and second must both live as long as that generic lifetime.

#### Lifetime Annotations in Function Signatures
```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

The function signature now tells Rust that both parameters, `x` and `y`, are string slices that live at least as long as lifetime `'a`. The function signature also tells Rust that the string slice returned from the function will live at least as long as lifetime `'a`.

**Remember**, when we specify the **lifetime parameters** in this function signature, we’re **not changing the lifetimes** of any values passed in or returned. Rather, we’re specifying that the **borrow checker** should **reject** any values that **don’t adhere to these constraints**.

When we pass concrete references to `longest`, the generic lifetime `'a` will get the concrete lifetime that is equal to the **smaller** of the lifetimes of `x` and `y`.

```rust
// Can't compile.
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {}", result);
}
```

As humans, we can look at this code and see that `string1` is longer than `string2` and therefore result will contain a reference to `string1`. Because `string1` has not gone out of scope yet, a reference to `string1` will still be valid for the `println!` statement.

However, **the compiler can’t see that the reference** is valid in this case. **We’ve told Rust** that the lifetime of the reference returned by the longest function is the same as the smaller of the lifetimes of the references passed in. Therefore, the **borrow checker disallows** the above code as possibly having an invalid reference.

#### Lifetime Annotations in Struct Definitions

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.')
        .next()
        .expect("Could not find a '.'");
    let i = ImportantExcerpt { part: first_sentence };
}
```

The generic lifetime annotation means an instance of `ImportantExcerpt` can’t outlive the reference it holds in its `part` field.

#### Lifetime Elision

You need to specify lifetime parameters for functions or structs that use references. But as the compiler (borrow checker) becoming smarter, it can infer the lifetimes in some patterns and wouldn't need explicit annotations.

The patterns programmed into Rust's analysis of references are called the `lifetime elision rules`. The compiler uses three rules to figure out what lifetimes references have when there aren’t explicit annotations. If the compiler gets to the end of the three rules and there are still references for which it can’t figure out lifetimes, the compiler will stop with an error. These rules apply to `fn` definitions as well as `impl` blocks.

- Rule #1: each parameter that is a reference gets its own lifetime parameter. In other words, a function with one parameter gets one lifetime parameter: `fn foo<'a>(x: &'a i32);` a function with two parameters gets two separate lifetime parameters: `fn foo<'a, 'b>(x: &'a i32, y: &'b i32);` and so on.
- Rule #2: if there is exactly one input lifetime parameter, that lifetime is assigned to all output lifetime parameters: `fn foo<'a>(x: &'a i32) -> &'a i32`.
- Rule #3: if there are multiple input lifetime parameters, but one of them is `&self` or `&mut self` because this is a method, the lifetime of `self` is assigned to all output lifetime parameters. This third rule makes methods much nicer to read and write because fewer symbols are necessary.

Examples.
```rust
fn first_word(s: &str) -> &str {
// After applying Rule #1: fn first_word<'a>(s: &'a str) -> &str {
// After applying Rule #2: fn first_word<'a>(s: &'a str) -> &'a str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}

// Can't compile.
fn longest(x: &str, y: &str) -> &str {
// After applying Rule #1: fn longest(x: &'a str, y: &'b str) -> &str {
// Rule #2 doesn't apply.
// Rule #3 doesn't apply either.
// The compiler worked through all lifetime elision rules but still couldn't
// figure out all the lifetimes of the references in the signature.
    // --snip--
}
```

#### The Static Lifetime

One special lifetime we need to discuss is `'static`, which means that this reference can **live for the entire duration of the program**. All string literals have the `'static` lifetime, which we can annotate as follows:
```rust
let s: &'static str = "I have a static lifetime.";
```

## 11 Writing Automated Tests

### 11.1 How to Write Tests

```rust
#[test]
fn test_case() {
    // --snip--
}
```
Each test is run in a new thread, and when the main thread sees that a test thread has died, the test is marked as failed. 

#### Adding Custom Failure Messages

Any arguments specified after the one required argument to `assert!` or the two required arguments to `assert_eq!` and `assert_ne!` are passed along to the `format!` macro, so you can pass a format string that contains `{}` placeholders and values to go in those placeholders. 

```rust
assert!(
        result.contains("Carol"),
        "Greeting did not contain name, value was `{}`", result
    );
```

#### Checking for Panics with `should_panic`
It’s also important to check that our code handles error conditions as we expect.
```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess {
            value
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    // should_panic attribute accepts `expected` parameter to make the test
    // more precise.
    // #[should_panic(expected = "Guess value must be less than or equal to 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

#### Using `Result<T, E>` in Tests
```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}
```

### 11.2 Controlling How Tests Are Run

#### Running a Subset of Tests by Name

You can choose which tests to run by passing `cargo test` the name or names of the test(s) you want to run as an argument.

We can specify part of a test name, and any test whose name matches that value will be run. 

#### Ignoring Some Tests Unless Specifically Requested
After `#[test]` we add the `#[ignore]` line to the test we want to exclude. `cargo test -- --ignored` to run only the ignored tests.
```rust
#[test]
fn it_works() {
    assert_eq!(2 + 2, 4);
}

#[test]
#[ignore]
fn expensive_test() {
    // code that takes an hour to run
}
```

### 11.3 Test Organization
The Rust community thinks about tests in terms of two main categories: `unit tests` and `integration tests`.

#### UT - testing one module in isolation at a time, and can test private interfaces.

#### IT - entirely external to your library and use your code in the same way any other external code would, using only the public interface and potentially exercising multiple modules per test.

#### Unit Tests

You’ll put unit tests in the `src` directory in each file with the code that they’re testing. The convention is to create a module named `tests` in each file to contain the test functions and to annotate the module with `cfg(test)`.

#### The Tests Module and `#[cfg(test)]`
The `#[cfg(test)]` annotation on the tests module tells Rust to compile and run the test code only when you run `cargo test`, not when you run `cargo build`. This **saves compile time** when you only want to build the library and **saves space** in the resulting compiled artifact because the tests are not included. You’ll see that because integration tests go in a different directory, they don’t need the `#[cfg(test)]` annotation. However, because unit tests go in the same files as the code, you’ll use `#[cfg(test)]` to **specify that they shouldn’t be included in the compiled result**.

#### Integration Tests

To create integration tests, **you first need a `tests` directory**.

We create a `tests` directory at the top level of our project directory, next to `src`. Cargo knows to look for integration test files in this directory.

To avoid having `common` appear in the test output, instead of creating `tests/common.rs`, we’ll create `tests/common/mod.rs`.

#### Integration Tests for Binary Crates
If our project is a binary crate that **only** contains a `src/main.rs` file and **doesn’t** have a `src/lib.rs` file, we **can’t** create integration tests in the `tests` directory and bring functions defined in the `src/main.rs` file into scope with a `use` statement. Only library crates expose functions that other crates can `use`; binary crates are meant to be run on their own.

This is one of the reasons Rust projects that provide a binary **have a straightforward `src/main.rs` file that calls logic that lives in the `src/lib.rs` file**. Using that structure, integration tests can test the library crate with use to make the important functionality available. If the important functionality works, the small amount of code in the `src/main.rs` file will work as well, and that small amount of code doesn’t need to be tested.o

## 12 An I/O Project: Building a Command Line Program

### 12.3 Refactoring to Improve Modularity and Error Handling

#### Guideline for splitting the separate concerns of a binary program when main starts getting large.

The process:
- Split your program into a `main.rs` and a `lib.rs` and move your program’s logic to `lib.rs`.
- As long as your command line parsing logic is small, it can remain in `main.rs`.
- When the command line parsing logic starts getting complicated, extract it from `main.rs` and move it to `lib.rs`.

`main` function should be limited to the following:
- Calling the command line parsing logic with the argument values
- Setting up any other configuration
- Calling a `run` function in `lib.rs`
- Handling the error if `run` returns an error

Because you can’t test the `main` function directly, this structure lets you test all of your program’s logic by moving it into functions in `lib.rs`. The only code that remains in `main.rs` will be small enough to verify its correctness by reading it.

## 13 Functional Language Features: Iterators and Closures

Programming in a functional style often includes using functions as values by passing them in arguments, returning them from other functions, assigning them to variables for later execution, and so forth.

### 13.1 Closures: Anonymous Functions that Can Capture Their Environment

You can create the closure in one place and then call the closure to evaluate it in a different context. Unlike functions, closures can capture values from the scope in which they're defined.

#### Closure Type Inference and Annotation
**Closures don’t require you to annotate the types** of the parameters or the return value like fn functions do. Type annotations are **required on functions** because they’re part of an explicit interface exposed to your users. Defining this interface rigidly is important for ensuring that everyone agrees on what types of values a function uses and returns. But closures aren’t used in an exposed interface like this: they’re **stored in variables and used without naming them and exposing them** to users of our library.

```rust
fn  add_one_v1   (x: u32) -> u32 { x + 1 }   // function
let add_one_v2 = |x: u32| -> u32 { x + 1 };  // fully annotated closure
let add_one_v3 = |x|             { x + 1 };  
let add_one_v4 = |x|               x + 1  ;  // only one expression in body
```

**Closure definitions will have one concrete type inferred for each of their parameters and for their return value.**
```rust
// can't compile
let example_closure = |x| x;

let s = example_closure(String::from("hello"));
let n = example_closure(5);
```
The first time we call `example_closure` with the `String` value, the compiler infers the type of `x` and the return type of the closure to be `String`. Those types are then locked in to the closure in `example_closure`, and we get a type error if we try to use a different type with the same closure.

#### Storing Closures Using Generic Parameters and the `Fn` Traits

We can create a **struct** that will hold the closure and the resulting value of calling the closure. The struct will **execute** the closure **only if we need** the resulting value, and it will **cache the resulting value** so the rest of our code doesn’t have to be responsible for saving and reusing the result. You may know this pattern as `memoization` or `lazy evaluation`.

Each closure instance has its own unique anonymous type: that is, even if two closures have the same signature, their types are still considered different.

All closures implement at least one of the traits: `Fn`, `FnMut`, or `FnOnce`. 
```rust
struct Cacher<T>
    where T: Fn(u32) -> u32
{
    calculation: T,
    value: Option<u32>,
}

impl<T> Cacher<T>
    where T: Fn(u32) -> u32
{
    fn new(calculation: T) -> Cacher<T> {
        Cacher {
            calculation,
            value: None,
        }
    }

    fn value(&mut self, arg: u32) -> u32 {
        match self.value {
            Some(v) => v,
            None => {
                let v = (self.calculation)(arg);
                self.value = Some(v);
                v
            },
        }
    }
}
```

**Note:** **Functions** can implement all three of the `Fn` traits too. If what we want to do **doesn’t require capturing** a value from the environment, we can use a function rather than a closure where we need something that implements an `Fn` trait.

#### Capturing the Environment with Closures

Closures have an additional capability that functions don’t have: **they can capture their environment and access variables from the scope in which they’re defined**.

Closures can capture values from their environment in **three** ways, which directly map to the three ways a function can take a parameter: **taking ownership, borrowing mutably, and borrowing immutably.** These are encoded in the three `Fn` traits as follows:

`FnOnce` consumes the variables it captures from its enclosing scope, known as the closure’s environment. To consume the captured variables, the closure must take ownership of these variables and move them into the closure when it is defined. The Once part of the name represents the fact that the closure can’t take ownership of the same variables more than once, so it can be called only once.
`FnMut` can change the environment because it mutably borrows values.
`Fn` borrows values from the environment immutably.
When you create a closure, Rust infers which trait to use based on how the closure uses the values from the environment. All closures implement `FnOnce` because they can all be called at least once. Closures that don’t move the captured variables also implement `FnMut`, and closures that don’t need mutable access to the captured variables also implement `Fn`. 

```rust
fn main() {
    let x = 4;

    let equal_to_x = |z| z == x;

    let y = 4;

    assert!(equal_to_x(y));
}
```
In this example, the `equal_to_x` closure borrows `x` immutably (so `equal_to_x` has the `Fn` trait) because the body of the closure only needs to read the value in `x`.

If you want to force the closure to take ownership of the values it uses in the environment, you can use the `move` keyword before the parameter list. This technique is mostly useful when passing a closure to a new thread to `move` the data so it’s owned by the new thread.
```rust
fn main() {
    let x = vec![1, 2, 3];

    let equal_to_x = move |z| z == x;

    // Can't compile due to `x` was moved to closure `equal_to_x` already,
    // `main` has no ownershipt of `x` here.
    println!("can't use x here: {:?}", x);

    let y = vec![1, 2, 3];

    assert!(equal_to_x(y));
}
```

**Most of the time when specifying one of the `Fn` trait bounds, you can start with `Fn` and the compiler will tell you if you need `FnMut` or `FnOnce` based on what happens in the closure body.**

Because `Fn` clousres have minimum influence to its enclosing scope, it just borrows variables immutably.

### 13.2 Processing a Series of Items with Iterators

`Iterator` trait definition:
```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // methods with default implementations elided
}
```
Notice this definition uses some new syntax: `type Item` and `Self::Item`, which are defining an `associated type` with this trait. That is, implementing the `Iterator` trait requires that you also define an `Item` type, and this `Item` type is used in the return type of the next method. In other words, the `Item` type will be the type returned from the iterator.

```rust
#[test]
fn iterator_demonstration() {
    let v1 = vec![1, 2, 3];

    let mut v1_iter = v1.iter();

    assert_eq!(v1_iter.next(), Some(&1));
    assert_eq!(v1_iter.next(), Some(&2));
    assert_eq!(v1_iter.next(), Some(&3));
    assert_eq!(v1_iter.next(), None);
}
```
Note that we needed to make `v1_iter` mutable: calling the `next` method on an iterator changes internal state that the iterator uses to keep track of where it is in the sequence. **We didn’t need to make `v1_iter` mutable when we used a `for` loop because the loop took ownership of `v1_iter` and made it mutable behind the scenes.**

Also note that the values we get from the calls to `next` are immutable references to the values in the vector. The `iter` method produces an iterator over **immutable references**. If we want to create an iterator that takes ownership of `v1` and returns owned values, we can call `into_iter` instead of `iter`. Similarly, if we want to iterate over mutable references, we can call `iter_mut` instead of `iter`.
```rust
let v1 = vec![1, 2, 3];
let mut v1_iter = v1.into_iter();
assert_eq!(v1_iter.next(), Some(1));

// Can't compile due to v1 moved into into_iter.
println!("{:?}", v1);
```

#### Methods that Consume the Iterator
Methods that call `next` are called **consuming adaptors**, because calling them uses up the iterator.

#### Methods that Produce Other Iterators
Other methods defined on the `Iterator` trait, known as **iterator adaptors**, allow you to change iterators into different kinds of iterators.

`map` is an iterator adaptor.
```rust
let v1: Vec<i32> = vec![1, 2, 3];
let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();
assert_eq!(v2, vec![2, 3, 4]);
```

#### Creating Our Own Iterators with the `Iterator` Trait
```rust
struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}

impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;

        if self.count < 6 {
            Some(self.count)
        } else {
            None
        }
    }
}
```

#### Using Other `Iterator` Trait Methods
We implemented the `Iterator` trait by defining the `next` method, so we can now use any `Iterator` trait **method’s default implementations as defined in the standard library**, because they all use the `next` method’s functionality.
```rust
#[test]
fn using_other_iterator_trait_methods() {
    let sum: u32 = Counter::new().zip(Counter::new().skip(1))
                                 .map(|(a, b)| a * b)
                                 .filter(|x| x % 3 == 0)
                                 .sum();
    assert_eq!(18, sum);
}
```

**You can use iterators and closures without fear! They make code seem like it’s higher level but don’t impose a runtime performance penalty for doing so.**

## 14 Cargo and Crates.io

### 14.2 Publishing a Crate to Crates.io

#### Making Useful Documentation Comments

1. Use three slashes, `///`.
2. Support Markdown notation.
3. Commonly used setions: `Examples`, `Panics`, `Errors`, and `Safety`.
4. Run `cargo test` will run the code examples.
```rust
/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let arg = 5;
/// let answer = my_crate::add_one(arg);
///
/// assert_eq!(6, answer);
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

#### Commenting Contained Items
`//!` adds documentation to the item that contains the comments.
```rust
//! # My Crate
//!
//! `my_crate` is a collection of utilities to make performing certain
//! calculations more convenient.

/// Adds one to the number given.
// --snip--
```

#### Exporting a Convenient Public API with `pub use`
Some structs may be deep in the hierarchy of your crate, which is not freindly to the user who wants to use it.

Could use `pub use` to re-export your types.
```rust
//! # Art
//!
//! A library for modeling artistic concepts.

pub use self::kinds::PrimaryColor;
pub use self::kinds::SecondaryColor;
pub use self::utils::mix;

pub mod kinds {
    // --snip--
}

pub mod utils {
    // --snip--
}
```

```rust
// In others crate.
use art::PrimaryColor;
use art::mix;

fn main() {
    // --snip--
}
```

### 14.5 Extending Cargo with Custom Commands
Cargo is designed so you can extend it with new subcommands without having to modify Cargo. If a binary in your `$PATH` is named `cargo-something`, you can run it as if it was a Cargo subcommand by running `cargo something`. Custom commands like this are also listed when you run `cargo --list`.

## 15 Smart Pointers
A `pointer` is a general concept for a variable that contains an address in memory. This address refers to, or "points at", some other data.

References are a kind of pointer, without any special capabilities other than referring to data. Also, they don't have any overhead.

`Smart pointers`, on the other hand, are data structures that not only act like a pointer but also have additional metadata and capabilities.

In Rust, the different smart pointers defined in the standard library provide **functionality** beyond that provided by references. For example, the `reference counting` smart pointer type enables you to have multiple owners of data by keeping track of the number of owners and, when no owners remain, cleaning up the data.

An additional difference between references and smart pointers is that **references** are pointers that **only borrow** data; in contrast, in many cases, **smart pointers** **own** the data they point to. `String` and `Vec<T>` are counted as smart pointers because they own some memory and allow you to manipulate it.

Smart pointers implement the `Deref` and `Drop` traits.
- `Deref` allows an instance of the smart pointer struct to behave like a reference so you can write code that works with either references or smart pointers.
- `Drop` allows you to customize the code that is run when an instance of the smart pointer goes out of scope.

### 15.1 Using `Box<T>` to Point to Data on the Heap

Boxes allow you to store data on the heap rather than the stack. What remains on the stack is the pointer to the heap data.

Boxed don't have performance overhead, other than storing their data on the heap instead of on the stack. But they don't have many extra capabilities either.

You'll use them most often in these situations:
- When you have a type whose size can't be known at compile time and you want to use a value of that type in a context that requires an exact size.
- When you have a large amount of data and you want to transfer ownership but ensure the data won't be copied when you do so.

**Transferring ownership** of a large amount of data can take a long time because the data is **copied around on the stack**. To improve performance in this situation, we can store the large amount of **data on the heap in a box**. Then, only the small amount of **pointer** data is **copied** around on the **stack**, while the data it references stays in one place on the heap.

- When you want to own a value and you care only that it's a type that implements a particular trait rather than being of a specific type. `Trait objects` do what as you want.


#### Using a `Box<T>` to Store Data on the Heap
```rust
fn main() {
    let b = Box::new(5);  // store an `i32` value on the heap
    println!("b = {}", b);  // we can access the data in the box similar to
                            // how we would if this data were on the stack
}
```

**When a box (`b` above) goes out of scope, the deallocation happens for the box (stored on the stack) and the data it points to (stored on the heap).**

#### Enabling Recursive Types with Boxes
At compile time, Rust needs to know how much space a type takes up. One type whose size can’t be known at compile time is a `recursive type`, where a value can have as part of itself another value of the same type.
```rust
enum List {
    Cons(i32, List),
    Nil,
}

use crate::List::{Cons, Nil};
let list = Cons(1, Cons(2, Cons(3, Nil)));
// Can't compile because Rust can't figure out how much space it needs to
// store a `List` value.
```

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```
To determine how much space to allocate for a `Message` value, Rust goes through each of the variants to see **which variant needs the most space**. Rust sees that `Message::Quit` doesn’t need any space, `Message::Move` needs enough space to store two `i32` values, and so forth. Because only one variant will be used, the most space a `Message` value will need is the space it would take to store the largest of its variants.

`List` enum above causes infinite calculation of space.
![](/forgetful/images/rust-infinite-cons-variants.svg)


```rust
enum List {
    Cons(i32, Box<List>),  // use Box now
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1,
        Box::new(Cons(2,
            Box::new(Cons(3,
                Box::new(Nil))))));
}
```
A `Box<T>` needs a pointer’s size doesn’t change based on the amount of data it’s pointing to.

The `Cons` variant will need the size of an `i32` plus the space to store the box’s pointer data. The `Nil` variant stores no values, so it needs less space than the `Cons` variant. We now know that any `List` value will take up the size of an `i32` plus the size of a box’s pointer data.

Boxes provide only the indirection and heap allocation; they don’t have any other special capabilities. They also don’t have any performance overhead that these special capabilities incur, so they can be **useful** in cases like the cons list where the **indirection is the only feature we need**.


### 15.2 Treating Smart Pointers Like Regular References with the `Deref` Trait

Implementing the `Deref` trait allows you to customize the behavior of the `dereference operator`, `*`. By implementing `Deref` in such a way that a smart pointer can be treated like a regular reference, you can write code that operates on references and use that code with smart pointers too.

#### Defining Our Own Smart Pointer
```rust
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}
```

#### Treating a Type Like a Reference by Implementing the Deref Trait

The `Deref` trait, provided by the standard library, requires us to implement one method named `deref` that **borrows** `self` and **returns a reference to the inner data**. 

```rust
use std::ops::Deref;

impl<T> Deref for MyBox<T> {
    // Defines an associated type for the Deref trait to use.
    type Target = T;

    fn deref(&self) -> &T {
        &self.0
    }
}

fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

Without the `Deref` trait, the compiler can only dereference `&` references, like `let y = &x;`, the compiler can only dereference this `y`.

The `deref` method gives the compiler the ability to take a value of any type that implements `Deref` and call the `deref` method to get a `&` reference that it knows how to dereference.

For example, `let y = MyBox::new(x);`. When we entered `*y`, Rust actually ran this code: `*(y.deref())`. `y.deref()` returned a reference to `x`, then the plain dereference `*` outside followed the reference to the data.

#### Implicit Deref Coercions with Functions and Methods
`Deref Coercion` is a convenience that Rust performs on **arguments** to **functions** and **methods**.

The deref coercion feature also lets us write more code that can work for either references or smart pointers.
```rust
fn hello(name: &str) {
    println!("Hello, {}!", name);
}

fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m);
    // Same as
    hello(&(*m)[..]);
}
```
Calling the `hello` function with `&m`, which is a reference to a `MyBox<String>` value. Deref coercions turn the reference to `MyBox` into a reference to `String` by calling `MyBox`'s `deref`. The standard library provides an implementation of `Deref` on `String` that returns a string slice. Deref coercions calls `deref` again to turn the reference to `String` into a string slice `&str`, which matches the `hello` definition.

Similar to how you use the `Deref` trait to override the `*` operator on **immutable references**, you can use the `DerefMut` trait to override the `*` operator on **mutable references**.

Rust does deref coercion when it finds types and trait implementations in three cases:

- From `&T` to `&U` when `T: Deref<Target=U>`
- From `&mut T` to `&mut U` when `T: DerefMut<Target=U>`
- From `&mut T` to `&U` when `T: Deref<Target=U>`

The last case: Rust will also coerce a mutable reference to an immutable one. But the reverse is **not** possible: immutable references will never coerce to mutable references. 

Because of the borrowing rules, if you have a **mutable** reference, that mutable reference must be **the only reference** to that data (otherwise, the program wouldn’t compile). Converting one mutable reference to one immutable reference will **never break the borrowing rules**. Converting an immutable reference to a mutable reference would require that there is only one immutable reference to that data, and the borrowing rules don’t guarantee that. Therefore, Rust can’t make the assumption that converting an immutable reference to a mutable reference is possible.

### 15.3 Running Code on Cleanup with the Drop Trait

The second trait important to the smart pointer pattern is Drop, which lets you customize what happens when a value is about to go out of scope.

You can use it to release resources like files or network connections. For example, `Box<T>` customizes `Drop` to deallocate the space on the heap that the box points to.

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer { data: String::from("my stuff") };
    let d = CustomSmartPointer { data: String::from("other stuff") };
    println!("CustomSmartPointers created.");
}
// Output would be:
// CustomSmartPointers created.
// Dropping CustomSmartPointer with data `other stuff`!
// Dropping CustomSmartPointer with data `my stuff`!
```
`data` with `String` type of `CustomSmartPointer` instances will be released by the compiler because they are on stack.

`d` was dropped before `c` because variables are dropped in the reverse order of their creation.

#### Dropping a Value Early with `std::mem::drop`

Occasionally, however, you might want to clean up a value early. One example is when using smart pointers that manage **locks**: you might want to force the `drop` method that releases the lock to run so other code in the same scope can acquire the lock. Rust doesn’t let you call the `Drop` trait’s `drop` method manually; instead you have to call the `std::mem::drop` function provided by the standard library if you want to force a value to be dropped before the end of its scope.


### 15.4 `Rc<T>`, the Reference Counted Smart Pointer
To enable multiple ownership (a single value has multiple owners), Rust has a type called `Rc<T>`, which is an abbreviation for **reference counting**. The `Rc<T>` type keeps track of the **number of references** to a value which determines whether or not a value is still in use. If there are zero references to a value, the value can be cleaned up without any references becoming invalid.

We use the `Rc<T>` type when we want to **allocate some data on the heap** for **multiple** parts of our program to **read** and we can’t determine at compile time which part will finish using the data last.

**Note that `Rc<T>` is only for use in single-threaded scenarios.**

#### Using `Rc<T>` to Share Data

Recall that we defined cons list using `Box<T>`. This time, we’ll create two lists that both share ownership of a third list. 
![](/forgetful/images/rust-rc-share-data.svg)
```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    println!("count after creating a = {}", Rc::strong_count(&a));
    let b = Cons(3, Rc::clone(&a));
    println!("count after creating b = {}", Rc::strong_count(&a));
    {
        let c = Cons(4, Rc::clone(&a));
        println!("count after creating c = {}", Rc::strong_count(&a));
    }
    println!("count after c goes out of scope = {}", Rc::strong_count(&a));
}
```
`Rc::clone(&a)` is used here rather than `a.clone()`, because `a.clone()` is more like the deep copy of most types' implementations of `clone` do. While `Rc::clone` doesn't do deep copy, it only increments the reference count, which doesn't take much time.

The implementation of the `Drop` trait decreases the reference count automatically when an `Rc<T>` value goes out of scope.

`Rc<T>` only allow you to have immutable references. With `RefCell<T>` type you can work with immutable references.

### 15.5 `RefCell<T>` and the Interior Mutability Pattern

`Interior mutability` is a design pattern in Rust that **allows you to mutate** data even when there are **immutable references** to that data; normally, this action is **disallowed** by the borrowing rules. To mutate data, the pattern uses **unsafe** code inside a data structure to bend Rust’s usual rules that govern mutation and borrowing.

We can use types that use the interior mutability pattern when we can ensure that the borrowing rules will be **followed at runtime**, even though the compiler can’t guarantee that. The unsafe code involved is then wrapped in a safe API, and **the outer type is still immutable**.

Unlike `Rc<T>`, the `RefCell<T>` type represents single ownership over the data it holds. So, what makes `RefCell<T>` different from a type like `Box<T>`?

With references and `Box<T>`, the borrowing rules’ invariants are enforced **at compile time**. With `RefCell<T>`, these invariants are enforced **at runtime**. With references, if you break these rules, you’ll get a **compiler error**. With `RefCell<T>`, if you break these rules, your program will **panic and exit**.

The `RefCell<T>` type is useful when **you**’re sure your code follows the borrowing rules but the compiler is unable to understand and guarantee that.

Similar to `Rc<T>`, `RefCell<T>` is only for use in **single-threaded** scenarios and will give you a compile-time error if you try using it in a multithreaded context. 

#### `Box<T>` vs. `Rc<T>` vs. `RefCell<T>`

- `Rc<T>` enables multiple owners of the same data; `Box<T>` and `RefCell<T>` have single owners.
- `Box<T>` allows immutable or mutable borrows checked at compile time; `Rc<T>` allows only immutable borrows checked at compile time; `RefCell<T>` allows immutable or mutable borrows checked at runtime.
- Because `RefCell<T>` allows mutable borrows checked at runtime, you can mutate the value inside the `RefCell<T>` even when the `RefCell<T>` is immutable.

#### A Use Case for Interior Mutability: Mock Objects
```rust
pub trait Messenger {
    fn send(&self, msg: &str);  // taking immutable reference to `self`
}

pub struct LimitTracker<'a, T: Messenger> {
    messenger: &'a T,
    value: usize,
    max: usize,
}

impl<'a, T> LimitTracker<'a, T>
    where T: Messenger {
    pub fn new(messenger: &T, max: usize) -> LimitTracker<T> {
        LimitTracker {
            messenger,
            value: 0,
            max,
        }
    }

    pub fn set_value(&mut self, value: usize) {
        self.value = value;

        let percentage_of_max = self.value as f64 / self.max as f64;

        if percentage_of_max >= 1.0 {
            self.messenger.send("Error: You are over your quota!");
        } else if percentage_of_max >= 0.9 {
             self.messenger.send("Urgent warning: You've used up over 90% of your quota!");
        } else if percentage_of_max >= 0.75 {
            self.messenger.send("Warning: You've used up over 75% of your quota!");
        }
    }
}
```

Then we want to mock `send` method.
```rust
// Can't compile.

#[cfg(test)]
mod tests {
    use super::*;

    struct MockMessenger {
        sent_messages: Vec<String>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger { sent_messages: vec![] }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.push(String::from(message));
            // Can't compile because `self` is immutable reference.
            // We can't change `&self` to `&mut self` either, because it's
            // defined in `Messenger`'s signature.
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        let mock_messenger = MockMessenger::new();
        let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);

        limit_tracker.set_value(80);

        assert_eq!(mock_messenger.sent_messages.len(), 1);
    }
}
```

This is a situation in which interior mutability can help.
```rust
#[cfg(test)]
mod tests {
    use super::*;
    use std::cell::RefCell;

    struct MockMessenger {
        sent_messages: RefCell<Vec<String>>,  // use RefCell for interior mutability
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger { sent_messages: RefCell::new(vec![]) }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            // `borrow_mut` gets a mutable reference to the value inside the
            // RefCell<Vec<String>>, which is the vector.
            self.sent_messages.borrow_mut().push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        // --snip--

        // Need to use `borrow` to get an immutable reference to the vector.
        assert_eq!(mock_messenger.sent_messages.borrow().len(), 1);
    }
}
```

#### Keeping Track of Borrows at Runtime with `RefCell<T>`

With `RefCell<T>`, we use the `borrow` and `borrow_mut` methods, which are part of the safe API that belongs to `RefCell<T>`. The `borrow` method returns the smart pointer type `Ref<T>`, and `borrow_mut` returns the smart pointer type `RefMut<T>`. Both types implement `Deref`, so we can treat them like regular references.

The `RefCell<T>` keeps track of how many `Ref<T>` and `RefMut<T>` smart pointers are currently active. Every time we call `borrow`, the `RefCell<T>` increases its count of how many immutable borrows are active. When a `Ref<T>` value goes out of scope, the count of immutable borrows goes down by one. Just like the compile-time borrowing rules, `RefCell<T>` lets us have many immutable borrows or one mutable borrow at any point in time.

If we try to violate these rules, rather than getting a compiler error as we would with references, the implementation of `RefCell<T>` will **panic at runtime**.
```rust
// Can't compile.
impl Messenger for MockMessenger {
    fn send(&self, message: &str) {
        // Two mutable references in the same scope isn't allowed.
        let mut one_borrow = self.sent_messages.borrow_mut();
        let mut two_borrow = self.sent_messages.borrow_mut();

        one_borrow.push(String::from(message));
        two_borrow.push(String::from(message));
    }
}
```

#### Having Multiple Owners of Mutable Data by Combining `Rc<T>` and `RefCell<T>`

Recall the cons list example above where we used `Rc<T>` to allow multiple lists to share ownership of another list. Because `Rc<T>` holds only immutable values, we **can’t change** any of the values in the list once we’ve created them.

We can gain the ability to change the values in the lists with `RefCell<T>`.
```rust
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),  // use `Rc<RefCell<i32>>` instead of `i32`
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;
use std::cell::RefCell;

fn main() {
    let value = Rc::new(RefCell::new(5));  // would change it directly
  
    // Both `a` and `value` have ownership ot the inner `5` value
    // because `Rc::clone` is used.
    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));

    // Both `b` and `c` owns `a`.
    let b = Cons(Rc::new(RefCell::new(6)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(10)), Rc::clone(&a));

    // `value` of `Rc` type is dereferenced to `RefCell` automatically,
    // then `borrow_mut` of `RefCell` is called to return a `RefMut` smart
    // pointer. And we use the dereference operator on it to change the
    // inner value.
    *value.borrow_mut() += 10;

    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}
```

The standard library has other types that provide **interior mutability**, such as `Cell<T>`, which is similar except that instead of giving references to the inner value, the value is **copied** in and out of the `Cell<T>`. There’s also `Mutex<T>`, which offers interior mutability that’s safe to use **across threads**.

### 15.6 Reference Cycles Can Leak Memory

Rust’s memory safety guarantees make it **difficult**, but **not impossible**, to accidentally create memory that is never cleaned up (known as a `memory leak`).

Preventing memory leaks entirely is **not** one of Rust’s **guarantees** in the same way that disallowing data races at compile time is, meaning memory leaks are memory safe in Rust.

We can see that Rust allows memory leaks by using `Rc<T>` and `RefCell<T>`: **it’s possible to create references where items refer to each other in a cycle**. This creates memory leaks because the **reference count** of each item in the cycle will **never reach 0**, and the values will never be dropped.

#### Creating a Reference Cycle
```rust
use std::rc::Rc;
use std::cell::RefCell;
use crate::List::{Cons, Nil};

#[derive(Debug)]
enum List {
    Cons(i32, RefCell<Rc<List>>),
    Nil,
}

impl List {
    fn tail(&self) -> Option<&RefCell<Rc<List>>> {
        match self {
            Cons(_, item) => Some(item),
            Nil => None,
        }
    }
}

fn main() {
    let a = Rc::new(Cons(5, RefCell::new(Rc::new(Nil))));

    println!("a initial rc count = {}", Rc::strong_count(&a));
    // Output: a initial rc count = 1
    println!("a next item = {:?}", a.tail());
    // Output: a next item = Some(RefCell { value: Nil })

    let b = Rc::new(Cons(10, RefCell::new(Rc::clone(&a))));

    println!("a rc count after b creation = {}", Rc::strong_count(&a));
    // Output: a rc count after b creation = 2
    println!("b initial rc count = {}", Rc::strong_count(&b));
    // Output: b initial rc count = 1
    println!("b next item = {:?}", b.tail());
    // Output: b next item = Some(RefCell { value: Cons(5, RefCell { value: Nil }) })

    if let Some(link) = a.tail() {
        *link.borrow_mut() = Rc::clone(&b);
    }

    println!("b rc count after changing a = {}", Rc::strong_count(&b));
    // Output: b rc count after changing a = 2
    println!("a rc count after changing a = {}", Rc::strong_count(&a));
    // Output: a rc count after changing a = 2

    // Uncomment the next line to see that we have a cycle;
    // it will overflow the stack
    // println!("a next item = {:?}", a.tail());
    // a: 5 -> b: 10 -> a ....
}
```
Creating **reference cycles** is **not easily done**, but it’s **not impossible** either. If you have `RefCell<T>` values that contain `Rc<T>` values or similar nested combinations of types with interior mutability and reference counting, **you** must ensure that you don’t create cycles; you **can’t rely on Rust** to catch them. Creating a reference cycle would be a logic bug in your program that you should use automated tests, code reviews, and other software development practices to minimize.

**Another solution for avoiding reference cycles** is reorganizing your data structures so that some references express ownership and some references don’t. As a result, you can have cycles made up of some ownership relationships and some non-ownership relationships, and **only the ownership relationships affect whether or not a value can be dropped**.

However, reorganizing **doesn't** work well all the time. For example, in the cons list example, we always want Cons variants to own their list, so reorganizing the data structure isn’t possible.

But in graphs which are made up of parent nodes and child nodes  **non-ownership relationships are an appropriate way to prevent reference cycles**.

#### Preventing Reference Cycles: Turning an `Rc<T>` into a `Weak<T>`

Given `let x = Rc::new...`,

`Rc::clone(&x)` increases the `strong_count` of `x`, and `x` is only cleaned up if its `strong_count` is 0.

Calling `Rc::downgrade(&x)`, you get a smart pointer of type `Weak<T>`, called a **weak reference**. It increases the `weak_count` by 1. The `weak_count` doesn't need to be 0 for `x` to be cleaned up.

**Weak references don’t express an ownership relationship.** They won’t cause a reference cycle because any cycle involving some weak references will be broken once the strong reference count of values involved is 0.

Because the value that `Weak<T>` references **might have been dropped**, to do anything with the value that a `Weak<T>` is pointing to, you must make sure the value still exists. Do this by calling the `upgrade` method on a `Weak<T>` instance, which will return an `Option<Rc<T>>`. You’ll get a result of `Some` if the `Rc<T>` value has not been dropped yet and a result of `None` if the `Rc<T>` value has been dropped. Because `upgrade` returns an `Option<T>`, Rust will ensure that the `Some` case and the `None` case are handled, and there won’t be an invalid pointer.

Example of `tree`
```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

// Node only owns its children. If a Node is dropped, its children should
// be dropped as well. However, a child should not own its parent. This 
// is a case for weak reference!
#[derive(Debug)]
struct Node {
    value: i32,

    // Can’t contain an Rc<T> for parent, because that would create a
    // reference cycle with `leaf.parent` pointing to `branch` and 
    // `branch.children` pointing to `leaf`, which would cause their
    // `strong_count` values to never be 0.
    parent: RefCell<Weak<Node>>,  // weak reference to its parent
    children: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),  // initial parent is empty
        children: RefCell::new(vec![]),
    });

    println!(
        "leaf strong = {}, weak = {}",
        Rc::strong_count(&leaf),
        Rc::weak_count(&leaf),
    );
    // Output: leaf strong = 1, weak = 0.

    {
        let branch = Rc::new(Node {
            value: 5,
            parent: RefCell::new(Weak::new()),
            children: RefCell::new(vec![Rc::clone(&leaf)]),
        });

        // Set `leaf`'s parent to `branch` using `RefCell`'s `borrow_mut`.
        *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

        println!(
            "branch strong = {}, weak = {}",
            Rc::strong_count(&branch),
            Rc::weak_count(&branch),
        );
        // Output: branch strong = 1, weak = 1.

        println!(
            "leaf strong = {}, weak = {}",
            Rc::strong_count(&leaf),
            Rc::weak_count(&leaf),
        );
        // Output: leaf strong = 2, weak = 0.
    }
    // When this scope ends, `branch` goes out of scope and its `strong_count`
    // decreases to 0. So its `Node` is dropped, while `leaf` is dropped too
    // and because `leaf` is a `Rc`, only `strong_count` decreases to 1.

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
    // Output: leaf parent = None
    println!(
        "leaf strong = {}, weak = {}",
        Rc::strong_count(&leaf),
        Rc::weak_count(&leaf),
    );
    // Output: leaf strong = 1, weak = 0.
}
```

#### Summary

- The `Box<T>` type has a known size and points to data allocated on the heap.
- The `Rc<T>` type keeps track of the number of references to data on the heap so that data can have multiple owners.
- The `RefCell<T>` type with its interior mutability gives us a type that we can use when we need an immutable type but need to change an inner value of that type; it also enforces the borrowing rules at runtime instead of at compile time.

## 16 Fearless Concurrency

`Concurrent programming`, where different parts of a program execute independently, and `parallel programming`, where different parts of a program execute at the same time, are becoming increasingly important as more computers take advantage of their multiple processors.

### 16.1 Using Threads to Run Code Simultaneously

The negative effects of using threads:
- Race conditions, where threads are accessing data or resources in an inconsistent order.
- Deadlocks, where two threads are waiting for each other to finish using a resource the other thread has, preventing both threads from continuing.
- Bugs that happen only in certain situations and are hard to reproduce and fix reliably.

Programming languages implement threads in a few different ways. Many operating systems provide an API for creating new threads. This model where a language calls the operating system APIs to create threads is sometimes called **1:1**, meaning one operating system thread per one language thread.

Many programming languages provide their own special implementation of threads. Programming language-provided threads are known as `green` threads, and languages that use these green threads will execute them in the context of a different number of operating system threads. For this reason, the green-threaded model is called the **M:N** model: there are M green threads per N operating system threads, where M and N are not necessarily the same number.

The green-threading M:N model requires a larger language **runtime** (code that is included by the language in every binary) to manage threads. As such, the **Rust** standard library only provides an implementation of **1:1** threading. Because Rust is such a low-level language, there are crates that implement **M:N** threading if you would rather trade **overhead** for aspects such as **more control** over which threads run when and **lower costs of context switching**, for example.

#### Creating a New Thread with `spawn`
```rust
use std::thread;
use std::time::Duration;

fn main() {
    // Call `thread::spawn` with a closure.
    thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

#### Waiting for All Threads to Finish Using `join` Handles
```rust
use std::thread;
use std::time::Duration;

fn main() {
    // `thread::spawn` returns a `JoinHandle`, calling the `join` on the 
    // `JoinHandle` will wait for its thread to finish.
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```

#### Using `move` Closures with Threads
```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
    
    // Cann't compile without `move`.
    // let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

### 16.2 Using Message Passing to Transfer Data Between Threads

Go's slogan: **"Do not communicate by sharing memory; instead, share memory by communicating."**

Channels in Rust.
```rust
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();  // mpsc for multiple producer single consumer

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();  // or use try_recv which doesn't block
    println!("Got: {}", received);
}
```

The `send` method returns a `Result<T, E>` type, so if the receiving end has already been dropped and there’s nowhere to send a value, the `send` operation will return an error.

The receiving end of a channel has two useful methods: `recv` and `try_recv`. We’re using `recv`, short for receive, which will **block** the main thread’s execution and wait until a value is sent down the channel. Once a value is sent, `recv` will return it in a `Result<T, E>`. When the sending end of the channel closes, `recv` will return an error to signal that no more values will be coming.

The `try_recv` method **doesn’t block**, but will instead return a `Result<T, E>` immediately: an `Ok` value holding a message if one is available and an `Err` value if there aren’t any messages this time. **Using `try_recv` is useful if this thread has other work to do while waiting for messages: we could write a loop that calls `try_recv` every so often, handles a message if one is available, and otherwise does other work for a little while until checking again.**

When the `val` is sent down the channel via `tx.send`, the `send` function takes ownership of its parameter, and when the value is moved, **the receiver takes ownership of it**.

#### Creating Multiple Producers by Cloning the Transmitter
```rust
// --snip--

let (tx, rx) = mpsc::channel();

let tx1 = mpsc::Sender::clone(&tx);  // now we have two producers, `tx` and `tx1`
thread::spawn(move || {
    let vals = vec![
        String::from("hi"),
        String::from("from"),
        String::from("the"),
        String::from("thread"),
    ];

    for val in vals {
        tx1.send(val).unwrap();
        thread::sleep(Duration::from_secs(1));
    }
});

thread::spawn(move || {
    let vals = vec![
        String::from("more"),
        String::from("messages"),
        String::from("for"),
        String::from("you"),
    ];

    for val in vals {
        tx.send(val).unwrap();
        thread::sleep(Duration::from_secs(1));
    }
});

for received in rx {
    println!("Got: {}", received);
}

// --snip--
```

### 16.3 Shared-State Concurrency

Channels are similar to single ownership, because once you transfer a value down a channel, you should no longer use that value. Shared memory concurrency is like multiple ownership: multiple threads can access the same memory location at the same time.

#### Using Mutexes to Allow Access to Data from One Thread at a Time
```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap();
        *num = 6;
    }  // num goes out of scope with unlocking the lock

    println!("m = {:?}", m);
}
```
The `lock` method is used to acquire the lock. It would block the current thread until it's our turn to have the lock. The call to `lock` would fail if another thread holding the lock panicked.

`Mutex<T>` is a smart pointer. More accurately, the call to `lock` returns a smart pointer called `MutexGuard`, wrapped in a `LockResult` that we handled with the call to `unwrap`. The `MutexGuard` smart pointer implements `Deref` to point at our inner data; the smart pointer also has a `Drop` implementation that releases the lock automatically when a `MutexGuard` goes out of scope, which happens at the end of the inner scope above.

#### Multiple Ownership with Multiple Threads
```rust
use std::sync::{Mutex, Arc};
use std::thread;

fn main() {
    // let counter = Mutex::new(0); doesn't work here because counter can't
    // move to multiple threads' closures.
    // let counter = Rc::new(Mutex::new(0)); doesn't work either because `Rc`
    // is not safe to share across threads. 
    // `Arc` means atomic reference counting, which is safe to use in concurrent
    // situations.
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

The reason is that thread safety comes with a performance penalty that you only want to pay when you really need to. If you’re just performing operations on values within a single thread, your code can run faster if it doesn’t have to enforce the guarantees atomics provide.

#### Similarities Between `RefCell<T>/Rc<T>` and `Mutex<T>/Arc<T>`

The `counter` is immutable but we could get a mutable reference to the value inside it; this means `Mutex<T>` provides interior mutability, as the `Cell` family does. In the same way we used `RefCell<T>` to allow us to mutate contents inside an `Rc<T>`, we use `Mutex<T>` to mutate contents inside an `Arc<T>`.

Using `Rc<T>` came with the risk of creating reference cycles, where two `Rc<T>` values refer to each other, **causing memory leaks**. Similarly, `Mutex<T>` comes with the risk of **creating deadlocks**. These occur when an operation needs to lock two resources and two threads have each acquired one of the locks, causing them to wait for each other forever. 

### 16.4 Extensible Concurrency with the `Sync` and `Send` Traits

#### Allowing Transference of Ownership Between Threads with `Send`
The `Send` marker trait indicates that **ownership** of the type implementing `Send` can be **transferred between threads**. Almost every Rust type is `Send`, but there are some exceptions, including `Rc<T>`: this cannot be `Send` because if you cloned an `Rc<T>` value and tried to transfer ownership of the clone to another thread, **both threads might update the reference count at the same time**. For this reason, `Rc<T>` is implemented for use in single-threaded situations where you don’t want to pay the thread-safe performance penalty.

Any type composed **entirely** of `Send` types is **automatically** marked as `Send` as well. Almost all **primitive** types are `Send`, aside from raw pointers.

#### Allowing Access from Multiple Threads with Sync
The `Sync` marker trait indicates that it is safe for the type implementing `Sync` to be referenced from multiple threads. In other words, any type `T` is `Sync` if `&T` (a reference to `T`) is `Send`, meaning the reference can be sent safely to another thread. Similar to `Send`, **primitive types** are `Sync`, and types **composed entirely** of types that are `Sync` are also `Sync`.

## 17 Object Oriented Programming Features of Rust

### 17.1 Characteristics of Object-Oriented Languages

OOP languages share certain common characteristics, **namely objects**, **encapsulation**, and **inheritance**.

#### Objects Contain Data and Behavior
`The Gang of Four` book defines OOP this way:
    
    Object-oriented programs are made up of objects. An object packages both data and the procedures that operate on that data. The procedures are typically called methods or operations.

Using this definition, Rust is object oriented: structs and enums have data, and `impl` blocks provide methods on structs and enums.

#### Encapsulation that Hides Implementation Details
If encapsulation is a required aspect for a language to be considered object oriented, then Rust meets that requirement. The option to use `pub` or not for different parts of code enables encapsulation of implementation details.

#### Inheritance as a Type System and as Code Sharing

If a language must have inheritance to be an object-oriented language, then Rust is not one. There is no way to define a struct that inherits the parent struct’s fields and method implementations.

You choose inheritance for two main reasons. **One is for reuse of code**: you can implement particular behavior for one type, and inheritance enables you to reuse that implementation for a different type. You can share Rust code **using default trait method implementations** instead.

**The other reason to use inheritance** relates to the type system: to enable a child type to be used in the same places as the parent type. This is also called **polymorphism**, which means that you can substitute multiple objects for each other at runtime if they share certain characteristics. Rust instead uses **generics to abstract over different possible types** and **trait bounds** to impose constraints on what those types must provide. This is sometimes called `bounded parametric polymorphism`. Also Rust provides **trait objects** to enable polymorphism.

Inheritance has recently fallen out of favor as a programming design solution in many programming languages.

### 17.2 Using Trait Objects That Allow for Values of Different Types

#### Defining a Trait for Common Behavior

Trait objects are more like objects in other languages in the sense that they **combine data and behavior**. But trait objects differ from traditional objects in that we can’t add data to a trait object. Trait objects aren’t as generally useful as objects in other languages: their specific purpose is to allow **abstraction across common behavior**.

```rust
// First we need a trait.
pub trait Draw {
    fn draw(&self);
}

// A struct holds a vector of components.
pub struct Screen {
    // Here `Box<dyn Draw>` is a trait object.
    pub components: Vec<Box<dyn Draw>>,  // any type inside a `Box` that
                                         // implements the `Draw` trait
}

impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```
In this example, the trait object is a `Box`. We create a trait object by specifying some sort of pointer, such as a `&` reference or a `Box<T>` smart pointer, then the `dyn` keyword, and then specifying the relevant trait.

This works differently from defining a struct that **uses a generic type parameter with trait bounds**. **A generic type parameter can only be substituted with one concrete type at a time, whereas trait objects allow for multiple concrete types to fill in for the trait object at runtime**. For example, we could have defined the `Screen` struct using a generic type and a trait bound as this:
```rust
pub struct Screen<T: Draw> {
    pub components: Vec<T>,
}

impl<T> Screen<T>
    where T: Draw {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

This restricts us to a `Screen` instance that has a list of components all of type `Button` or all of type `TextField`. If you’ll only ever have **homogeneous collections**, using **generics and trait bounds** is preferable because the definitions will be monomorphized at compile time to use the concrete types.

On the other hand, with the method using **trait objects**, one `Screen` instance can hold a `Vec<T>` that contains a `Box<Button>` as well as a `Box<TextField>`.

#### Implementing the Trait
```rust
// Published with our gui crate.
pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        // code to actually draw a button
    }
}
```

```rust
// Our library's user defines their own type.
use gui::Draw;

struct SelectBox {
    width: u32,
    height: u32,
    options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        // code to actually draw a select box
    }
}

use gui::{Screen, Button};

fn main() {
    let screen = Screen {
        components: vec![
            Box::new(SelectBox {
                width: 75,
                height: 10,
                options: vec![
                    String::from("Yes"),
                    String::from("Maybe"),
                    String::from("No")
                ],
            }),
            Box::new(Button {
                width: 50,
                height: 10,
                label: String::from("OK"),
            }),
        ],
    };

    screen.run();
}
```
When we wrote the library, we didn’t know that someone might add the `SelectBox` type, but our `Screen` implementation was able to operate on the new type and draw it because `SelectBox` implements the `Draw` trait, which means it implements the `draw` method.

This concept—of being concerned only with the messages a value responds to rather than the value’s concrete type—is similar to the concept **duck typing** in dynamically typed languages: if it walks like a duck and quacks like a duck, then it must be a duck! In the implementation of `run` on `Screen` in above code, `run` doesn’t need to know what the concrete type of each component is. It doesn’t check whether a component is an instance of a `Button` or a `SelectBox`, it just calls the `draw` method on the component. By specifying `Box<dyn Draw>` as the type of the values in the components vector, we’ve defined `Screen` to need values that we can call the `draw` method on.

The advantage of using **trait objects** and **Rust’s type system** to write code similar to code using duck typing is that we **never** have to check whether a value implements a particular method at **runtime** or worry about getting errors if a value doesn’t implement a method but we call it anyway. Rust **won’t compile** our code if the values don’t implement the traits that the trait objects need.

#### Trait Objects Perform Dynamic Dispatch
The monomorphization process performed by the compiler when we use trait bounds on generics: the compiler **generates** nongeneric implementations of functions and methods **for each concrete type** that we use in place of a generic type parameter. The code that results from monomorphization is doing **static dispatch**, which is when the compiler knows what method you’re calling **at compile time**. This is opposed to **dynamic dispatch**, which is when the compiler can’t tell at compile time which method you’re calling. In dynamic dispatch cases, the compiler emits code that **at runtime** will figure out which method to call.

When we use **trait objects**, Rust must use **dynamic dispatch**. The compiler doesn’t know all the types that might be used with the code that is using trait objects, so it doesn’t know which method implemented on which type to call. Instead, at runtime, Rust uses the **pointers** inside the trait object to know which method to call. There is **a runtime cost** when this lookup happens that doesn’t occur with static dispatch. **Dynamic dispatch also prevents the compiler from choosing to inline a method’s code, which in turn prevents some optimizations.**

#### Object Safety Is Required for Trait Objects

TODO: NOT CLEAR

### 17.3 Implementing an Object-Oriented Design Pattern

Implemented here: `~/git/playground/trpl/chap_17/blog`.

This example shows that Rust is capable of implementing the object-oriented state pattern to encapsulate the different kinds of behavior a post should have in each state.

One **downside** of the state pattern is that, because the states implement the transitions between states, some of the states are coupled to each other. 

Another **downside** is that we’ve duplicated some logic.


By implementing the state pattern exactly as it’s defined for object-oriented languages, we’re **not taking as full advantage of Rust**’s strengths as we could.

#### We could leverage Rust's type checking system more to issue compiler errors.

The steps to refactor our state OOP pattern to a Rust one:
#### Encoding States and Behavior as Types

#### Implementing Transitions as Transformations into Different Types

Although you might be very familiar with object-oriented patterns, **rethinking** the problem to take advantage of Rust’s features can provide benefits, such as preventing some bugs at **compile time**. Object-oriented patterns won’t always be the best solution in Rust due to certain features, like ownership, that object-oriented languages don’t have.

## 18 Patterns and Matching

### 18.1 All the Places Patterns Can Be Used

#### `match` Arms
```rust
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```
Note that it is a expression followed the pattern.

A particular pattern `_` will **match anything**, but it never binds to a variable, so it’s often used in **the last match arm**.

#### Conditional `if let` Expressions

`if let` expressions mainly as a shorter way to write the equivalent of a match that only matches one case.
```rust
fn main() {
    let favorite_color: Option<&str> = None;
    let is_tuesday = false;
    let age: Result<u8, _> = "34".parse();

    if let Some(color) = favorite_color {
        println!("Using your favorite color, {}, as the background", color);
    } else if is_tuesday {
        println!("Tuesday is green day!");
    } else if let Ok(age) = age {  // introduce a new shadowed `age` as well
        // But can't use `else if let Ok(age) = age && age > 30` here
        // because the shadowed `age` isn't valid until the new scop starts
        // with the curly bracket. 
        if age > 30 {
            println!("Using purple as the background color");
        } else {
            println!("Using orange as the background color");
        }
    } else {
        println!("Using blue as the background color");
    }
}
```

#### `while let` Conditional Loops
Similar in construction to `if let`, the `while let` conditional loop allows a while loop to run for as long as a pattern continues to match.
```rust
let mut stack = Vec::new();

stack.push(1);
stack.push(2);
stack.push(3);

while let Some(top) = stack.pop() {
    println!("{}", top);
}
```

#### `for` Loops

In a `for` loop, the pattern is the value that directly follows the keyword `for`, so in `for x in y` the `x` is the pattern.
```rust
let v = vec!['a', 'b', 'c'];

for (index, value) in v.iter().enumerate() {
    println!("{} is at index {}", value, index);
}
```

#### `let` Statements
```rust
let PATTERN = EXPRESSION;

// For example:
let (x, y, z) = (1, 2, 3);
```
Use `_` or `..` to ignore one ro more of the values in the tuple.


#### Function Parameters
Function parameters can also be patterns as `let` statements.

### 18.2 Refutability: Whether a Pattern Might Fail to Match

Patterns come in two forms: **refutable** and **irrefutable**. Patterns that will match for any possible value passed are **irrefutable**. An example would be `x` in the statement `let x = 5;` because `x` matches anything and therefore cannot fail to match. Patterns that can fail to match for some possible value are **refutable**. An example would be `Some(x)` in the expression `if let Some(x) = a_value` because if the value in the `a_value` variable is `None` rather than `Some`, the `Some(x)` pattern will not match.

Irrefutable patterns: function parameters, let statements, and for loops. Because the program cannot do anything meaningful when values don’t match.

Refutable patterns: `if let`, `while let`.

### 18.3 Pattern Syntax

#### Matching Literals

#### Matching Named Variables

#### Multiple Patterns
```rust
let x = 1;

match x {
    1 | 2 => println!("one or two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

#### Matching Ranges of Values with `..=`
The only types for which Rust can tell if a range is empty or not are `char` and numeric values.
```rust
let x = 5;

match x {
    1..=5 => println!("one through five"),
    _ => println!("something else"),
}
```

#### Destructuring to Break Apart Values 
```rust
// Destructuring structs.
struct Point {
    x: i32,
    y: i32,
}
let p = Point { x: 0, y: 7 };

// Shortcut for `let Point {x: x, y: y} = p;`
let Point { x, y } = p;
assert_eq!(0, x);
assert_eq!(7, y);

match p {
    // Destructuring and matching part of the struct.
    Point { x, y: 0 } => println!("On the x axis at {}", x),
    Point { x: 0, y } => println!("On the y axis at {}", y),
    Point { x, y } => println!("On neither axis: ({}, {})", x, y),
}


// Destructuring Enums.
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
let msg = Message::ChangeColor(0, 160, 255);
match msg {
    Message::Quit => {
        println!("The Quit variant has no data to destructure.")
    },
    Message::Move { x, y } => {
        println!(
            "Move in the x direction {} and in the y direction {}",
            x,
            y
        );
    }
    Message::Write(text) => println!("Text message: {}", text),
    Message::ChangeColor(r, g, b) => {
        println!(
            "Change the color to red {}, green {}, and blue {}",
            r,
            g,
            b
        )
    }
}


// Destructuring Nested Structs and Enums
enum Color {
   Rgb(i32, i32, i32),
   Hsv(i32, i32, i32),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
}

let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

match msg {
    Message::ChangeColor(Color::Rgb(r, g, b)) => {
        println!(
            "Change the color to red {}, green {}, and blue {}",
            r,
            g,
            b
        )
    },
    Message::ChangeColor(Color::Hsv(h, s, v)) => {
        println!(
            "Change the color to hue {}, saturation {}, and value {}",
            h,
            s,
            v
        )
    }
    _ => ()
}


// Destructuring Structs and Tuples.
let ((feet, inches), Point {x, y}) = ((3, 10), Point { x: 3, y: -10 });

```

#### Ignoring Values in a Pattern
1. Ignoring an Entire Value with `_`

We’ve used the underscore (`_`) as a **wildcard** pattern that will match any value but **not bind to the value**.
```rust
// Using `_` to ignore a function parameter.
fn foo(_: i32, y: i32) {
    println!("This code only uses the y parameter: {}", y);
}

fn main() {
    foo(3, 4);
}
```

2. Ignoring Parts of a Value with a Nested `_`
```rust
let numbers = (2, 4, 8, 16, 32);

match numbers {
    (first, _, third, _, fifth) => {
        println!("Some numbers: {}, {}, {}", first, third, fifth)
    },
}
```

3. Ignoring an Unused Variable by Starting Its Name with `_`

Note that there is a subtle difference between using only `_` and using a name that starts with an underscore. The syntax `_x` still binds the value to the variable, whereas `_` doesn’t bind at all. 

```rust
let s = Some(String::from("Hello!"));

// Can't compile if using `_s` like below, because `s` is moved to `_s`.
// if let Some(_s) = s {
if let Some(_) = s {
    println!("found a string");
}

println!("{:?}", s);
```

4. Ignoring Remaining Parts of a Value with `..`
```rust
struct Point {
    x: i32,
    y: i32,
    z: i32,
}

let origin = Point { x: 0, y: 0, z: 0 };

match origin {
    // Equivalent to `Point { x, y, z }`.
    Point { x, .. } => println!("x is {}", x),
}

let numbers = (2, 4, 8, 16, 32);

match numbers {
    (first, .., last) => {
        println!("Some numbers: {}, {}", first, last);
    },
}
```

#### Extra Conditionals with Match Guards
```rust
let num = Some(4);

match num {
    Some(x) if x < 5 => println!("less than five: {}", x),
    Some(x) => println!("{}", x),
    None => (),
}
```

#### `@` Bindings

An example where we want to **test** that a `Message::Hello` `id` field is within the range `3..=7`. But we also want to **bind** the value to the variable `id_variable` so we can use it in the code associated with the arm.
```rust
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello { id: id_variable @ 3..=7 } => {
        println!("Found an id in range: {}", id_variable)
    },
    Message::Hello { id: 10..=12 } => {
        println!("Found an id in another range")
    },
    Message::Hello { id } => {
        println!("Found some other id: {}", id)
    },
}
```

## 19 Advanced Features

Reference when you use them.

### 19.1 Unsafe Rust

Unsafe Rust exists because:
1. Static analysis is conservative. In some cases, you can tell the compiler "Trust me, I know what I'm doing" by using unsafe code. Which means that you use it at your own risk.
2. Underlying computer hardware is inherently unsafe. If Rust didn’t let you do unsafe operations, you couldn’t do certain tasks. Rust needs to allow you to do low-level systems programming, such as directly interacting with the operating system or even writing your own operating system.

#### Unsafe Superpowers

These superpowers include the ability to:

- Dereference a raw pointer
- Call an unsafe function or method
- Access or modify a mutable static variable
- Implement an unsafe trait
- Access fields of `union`s

To switch to unsafe Rust, use the `unsafe` keyword and then start a new block that holds the unsafe code. 

Keep `unsafe` blocks small.

To isolate unsafe code as much as possible, it’s best to enclose unsafe code within a safe abstraction and provide a safe API.

#### Dereferencing a Raw Pointer
Raw pointers can be immutable or mutable and are written as `*const T` and `*mut T`, respectively.

Different from references and smart pointers, raw pointers:

- Are allowed to ignore the borrowing rules by having both immutable and mutable pointers or multiple mutable pointers to the same location
- Aren’t guaranteed to point to valid memory
- Are allowed to be null
- Don’t implement any automatic cleanup

```rust
let mut num = 5;

let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;

unsafe {
    println!("r1 is: {}", *r1);
    println!("r2 is: {}", *r2);
}
```
If we instead tried to create an immutable and a mutable reference to `num`, the code would not have compiled because Rust’s ownership rules don’t allow a mutable reference at the same time as any immutable references. With raw pointers, we can create a mutable pointer and an immutable pointer to the same location and change data through the mutable pointer, potentially creating a data race.

Mainly in two use cases:
1. Interfacing with C code.
2. Building up safe abstractions that the borrow checker doesn’t understand.

#### Calling an Unsafe Function or Method
```rust
unsafe fn dangerous() {}

unsafe {
    dangerous();
}
```

#### Creating a Safe Abstraction over Unsafe Code
Wrapping unsafe code in a safe function is a common abstraction.
```rust
use std::slice;

fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = slice.len();
    let ptr = slice.as_mut_ptr();

    assert!(mid <= len);

    unsafe {
        (slice::from_raw_parts_mut(ptr, mid),
         slice::from_raw_parts_mut(ptr.offset(mid as isize), len - mid))
    }
}
```
The **unsafe** code: the `slice::from_raw_parts_mut` function takes a raw pointer and a length, and it creates a slice. The function `slice::from_raw_parts_mut` is unsafe because it takes a raw pointer and must trust that this pointer is valid. The `offset` method on raw pointers is also unsafe, because it must trust that the offset location is also a valid pointer.

Note that we don’t need to mark the resulting `split_at_mut` function as `unsafe`, and we can call this function from safe Rust. This way we created a safe abstraction to the unsafe code with an implementation of the function that uses `unsafe` code in a safe way, because it creates only valid pointers from the data this function has access to.

#### Using `extern` Functions to Call External Code

```rust
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```
Keyword, `extern`, facilitates the creation and use of a `Foreign Function Interface` (`FFI`). An `FFI` is a way for a programming language to define functions and enable a different (foreign) programming language to call those functions.

Within the `extern "C"` block, we list the names and signatures of external functions from another language we want to call. The `"C"` part defines which `application binary interface` (`ABI`) the external function uses: the `ABI` defines how to call the function at the assembly level. The `"C"` ABI is the most common and follows the C programming language’s ABI.

#### Calling Rust Functions from Other Languages
We can also use `extern` to create an interface that allows other languages to call Rust functions. Instead of an `extern` block, we add the `extern` keyword and specify the `ABI` to use just before the fn keyword. We also need to add a `#[no_mangle]` annotation to tell the Rust compiler not to mangle the name of this function. Mangling is when a compiler changes the name we’ve given a function to a different name that contains more information for other parts of the compilation process to consume but is less human readable. Every programming language compiler mangles names slightly differently, so for a Rust function to be nameable by other languages, we must disable the Rust compiler’s name mangling.

In the following example, we make the `call_from_c` function accessible from C code, after it’s compiled to a shared library and linked from C:
```rust
#[no_mangle]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```
This usage of `extern` does not require unsafe.

#### Accessing or Modifying a Mutable Static Variable
```rust
static HELLO_WORLD: &str = "Hello, world!";

fn main() {
    println!("name is: {}", HELLO_WORLD);
}
```
**Static variables are similar to constants.**

**Static variables vs. Constants**

- Values in a static variable have a fixed address in memory. Using the value will always access the same data. Constants, on the other hand, are allowed to duplicate their data whenever they’re used.

- Static variables can be mutable. Accessing and modifying mutable static variables is **unsafe**. 
```rust
static mut COUNTER: u32 = 0;

fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}

fn main() {
    add_to_count(3);

    unsafe {  // note: it's unsafe
        println!("COUNTER: {}", COUNTER);
    }
}
```

#### Implementing an Unsafe Trait

A trait is unsafe when at least one of its methods has some invariant that the compiler can’t verify.

Adding the `unsafe` keyword before trait and marking the implementation of the trait as `unsafe` too.
```rust
unsafe trait Foo {
    // methods go here
}

unsafe impl Foo for i32 {
    // method implementations go here
}
```

### 19.3 Advance Traits

#### Specifying Placeholder Types in Trait Definitions with Associated Types

```rust
pub trait Iterator {
    type Item;  // this is an associated type

    fn next(&mut self) -> Option<Self::Item>;  // use the associated type here
}
```
**Associated types** connect a type **placeholder** with a trait such that the trait method definitions can use these placeholder types in their signatures.

#### Default Generic Type Parameters and Operator Overloading

**Operator overloading** is customizing the behavior of an operator (such as `+`) in particular situations.

Rust **doesn’t allow** you to create your own operators or overload arbitrary operators. But you can overload the operations and corresponding traits listed in `std::ops` by implementing the traits associated with the operator.

```rust
use std::ops::Add;  // only allow to overload operators in `std::ops`

#[derive(Debug, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {  // need implement `Add` trait to overload `+`
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    assert_eq!(Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
               Point { x: 3, y: 3 });
}

// The defination of `Add` trait.
trait Add<RHS=Self> {  // default type parameter `RHS=Self`
    type Output;

    fn add(self, rhs: RHS) -> Self::Output;
}
```

The `RHS` generic type parameter allows us to add two different types. But the default type parameter allows us not have to specify the extra parameter most of the time.
```rust
use std::ops::Add;

struct Millimeters(u32);
struct Meters(u32);

impl Add<Meters> for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
```

Another useful case of default type parameters is **to extend a type without breaking existing code**, that is, if you want to add a type parameter to an existing trait, you can give it a default to allow extension of the functionality of the trait without breaking the existing implementation code.

#### Fully Qualified Syntax for Disambiguation: Calling Methods with the Same Name

You could implement two traits on one type and there are methods with the same name from these two traits and the type.
```rust
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
}

impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
}

fn main() {
    let person = Human;
    Pilot::fly(&person);  // need to tell Rust which method to call
    Wizard::fly(&person);  // need to tell Rust which method to call
    person.fly();  // output: *waving arms furiously*
}
```
Three `fly` methods are implemented.

Because the `fly` method accepts `self` as a parameter, if two types, `Human` and `Dog` implement `Pilot` trait, Rust can tell which type's `fly` to call based on the type of `self`.

But if the trait method doesn't accept `self` as its parameter, in which case, the method is an associated function. **Fully qualified syntax** is needed in this case.
```rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from("Spot")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from("puppy")
    }
}

fn main() {
    // Not compile. Rust can't figure out which implementation to use.
    // println!("A baby dog is called a {}", Animal::baby_name());
    println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
}
```

#### Using Supertraits to Require One Trait’s Functionality Within Another Trait
`Display` trait is the **supertrait** of `OutlinePrint` trait.
```rust
use std::fmt;

trait OutlinePrint: fmt::Display {  // OutlinePrint depends on Display
    // Prints like:
    // **********
    // *        *
    // * (1, 3) *
    // *        *
    // **********
    fn outline_print(&self) {
        let output = self.to_string();  // to_string is Display's function
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}

struct Point {
    x: i32,
    y: i32,
}

impl OutlinePrint for Point {}

impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}
```

#### Using the Newtype Pattern to Implement External Traits on External Types

Rust has the orphan rule that states we’re allowed to implement a trait on a type as long as either the trait or the type are local to our crate. It’s possible to get around this restriction using the **newtype pattern**, which involves creating a new type in a **tuple struct**. The tuple struct will have one field and be a thin wrapper around the type we want to implement a trait for. Then the wrapper type is local to our crate, and we can implement the trait on the wrapper.

As an example, let’s say we want to implement `Display` on `Vec<T>`, which the orphan rule prevents us from doing directly because the `Display` trait and the `Vec<T>` type are defined outside our crate. We can make a `Wrapper` struct that holds an instance of `Vec<T>`; then we can implement `Display` on `Wrapper` and use the `Vec<T>` value.
```rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))  // notice the `self.0`
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```

The downside of using this technique is that `Wrapper` is a new type, so it **doesn’t have the methods** of the value it’s holding. We would have to implement all the methods of `Vec<T>` directly on `Wrapper` such that the methods delegate to `self.0`, which would allow us to treat `Wrapper` exactly like a `Vec<T>`. If we wanted the new type to have every method the inner type has, implementing the `Deref` trait on the `Wrapper` to return the inner type would be a solution. If we don’t want the `Wrapper` type to have all the methods of the inner type—for example, to restrict the `Wrapper` type’s behavior—we would have to implement just the methods we do want manually.

### 19.4 Advanced Types

#### Using the Newtype Pattern for Type Safety and Abstraction

**Type Safety**
```rust
struct Millimeters(u32);
struct Meters(u32);
```
`Millimeters` and `Meters` wrap `u32` values in a newtype, which enforces these two are different types.

If we worte a function with a parameter of type `Meters`, we couldn't compile a program that accidentally tried to call that function with a value of type `Millimeters` or a plain `u32`.

**Abstraction**
Another use of the newtype pattern is in **abstracting** away some implementation details of a type: the new type can expose a **public API** that is different from the API of the **private inner type** if we used the new type directly to restrict the available functionality, for example.

Newtypes can also hide internal implementation. For example, we could provide a `People` type to wrap a `HashMap<i32, String>` that stores a person’s ID associated with their name. Code using `People` would only interact with the public API we provide, such as a method to add a name string to the `People` collection; that code wouldn’t need to know that we assign an `i32` ID to names internally.

#### Creating Type Synonyms with Type Aliases
Along with the newtype pattern, Rust provides the ability to declare a **type alias** to give an existing type another name.
```rust
type Kilometers = i32;  // the alias `Kilometers` is a synonym for `i32`

let x: i32 = 5;
let y: Kilometers = 5;

println!("x + y = {}", x + y);
```
Unlike the `Millimeters` and `Meters` types above, `Kilometers` is **not** a separate, **new type**. Values that have the type `Kilometers` will be treated the same as values of type `i32`.

A type alias makes codes more manageable by reducing the repetition.
```rust
type Thunk = Box<dyn Fn() + Send + 'static>;

let f: Thunk = Box::new(|| println!("hi"));

fn takes_long_type(f: Thunk) {
    // --snip--
}

fn returns_long_type() -> Thunk {
    // --snip--
}
```

It is also commonly used in the `std::io` module.
```rust
type Result<T> = std::result::Result<T, std::io::Error>;
```

#### The Never Type that Never Returns
`!`, the `never type`, stands in the place of the return type when a function will never return. 
```rust
fn bar() -> ! {  // the function `bar` returns never.
    // --snip--
}
```
What the `never type` used for?
```rust
let guess: u32 = match guess.trim().parse() {
    Ok(num) => num,
    Err(_) => continue,
};
```
`continue` has a `!` value, or the code can't compile because `match` arms must all return the same type. Because `!` can never have a value, Rust decides that the type of `guess` is `u32`.

The formal way of describing this behavior is that expressions of type `!` can be **coerced into any other type**.

`panic!` macro is a useful never type as well.
```rust
impl<T> Option<T> {
    pub fn unwrap(self) -> T {
        match self {
            Some(val) => val,
            None => panic!("called `Option::unwrap()` on a `None` value"),
        }
    }
}
```

One final expression that has the type `!` is a loop:
```rust
print!("forever ");

loop {  // the loop never ends, so `!` is the value of the expression
    print!("and ever ");
}
```

#### Dynamically Sized Types and the `Sized` Trait
Due to Rust’s need to know certain details, such as how much space to allocate for a value of a particular type, there is a corner of its type system that can be confusing: the concept of **dynamically sized types**. Sometimes referred to as DSTs or unsized types, these types let us write code using values whose **size** we can know only **at runtime**.

**The dynamically sized type, `str`**

Not `&str`, but `str` on its own, is a DST. We can’t know how long the string is until runtime, meaning we can’t create a variable of type `str`, nor can we take an argument of type `str`.

So what do we do? We can make the type a `&str` rather than a `str`. The slice data structure stores the starting position and the length of the slice. 

The golden rule of dynamically sized types is that we must always put values of dynamically sized types behind a **pointer** of some kind.

We can combine `str` with all kinds of pointers: for example, `Box<str>` or `Rc<str>`. Every trait is a dynamically sized type we can refer to by using the name of the trait. To use traits as trait objects, we must put them behind a pointer, such as `&dyn Trait` or `Box<dyn Trait>` (`Rc<dyn Trait>` would work too).


To work with DSTs, Rust has a particular trait called the `Sized` trait to determine whether or not a type’s size is known at compile time. This trait is automatically implemented for everything whose size is known at compile time. In addition, Rust implicitly adds a bound on `Sized` to every generic function. That is, a generic function definition like this:
```rust
fn generic<T>(t: T) {  // same as `fn generic<T: Sized>(t: T)`
    // --snip--
}
```
By default, generic functions will work only on types that have a known size at compile time. However, you can use the following special syntax to relax this restriction:
```rust
fn generic<T: ?Sized>(t: &T) {
    // --snip--
}
```
A trait bound on `?Sized` is the **opposite** of a trait bound on `Sized`: we would read this as “T may or may not be Sized.” **This syntax is only available for `Sized`, not any other traits.**

Also note that we switched the type of the `t` parameter from `T` to `&T`. Because the type might not be `Sized`, we need to use it behind some kind of pointer. In this case, we’ve chosen a reference.

### 19.5 Advanced Functions and Closures

#### Function Pointers

Closures can be passed to functions. Regular functions can too, via **function pointers**.

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}

fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {  // `fn` type is function pointer
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(add_one, 5);

    println!("The answer is: {}", answer);
}
```

Unlike closures, `fn` is a type rather than a trait, so we specify `fn` as the parameter type directly rather than declaring a generic type parameter with one of the `Fn` traits as a trait bound.

Function pointers implement all three of the closure traits (`Fn`, `FnMut`, and `FnOnce`), so you can **always pass** a function pointer as an argument for a function that expects a closure. It’s best to write functions using a generic type and one of the closure traits so your functions can accept either functions or closures.

An example of where you would want to **only accept `fn` and not closures** is when interfacing with external code that doesn’t have closures: `C` functions can accept functions as arguments, but `C` doesn’t have closures.

```rust
let list_of_numbers = vec![1, 2, 3];
let list_of_strings: Vec<String> = list_of_numbers
    .iter()
    .map(|i| i.to_string())
    .collect();

let list_of_strings: Vec<String> = list_of_numbers
    .iter()
    .map(ToString::to_string)  // using a function pointer
    .collect();
```

We have another useful pattern that exploits an implementation detail of tuple structs and tuple-struct enum variants. These types use `()` as initializer syntax, which looks like a **function call**. The initializers are actually implemented as functions returning an instance that’s constructed from their arguments. **We can use these initializer functions as function pointers that implement the closure traits**, which means we can specify the initializer functions as arguments for methods that take closures, like so:
```rust
enum Status {
    Value(u32),
    Stop,
}

let list_of_statuses: Vec<Status> =
    (0u32..20)
    .map(Status::Value)
    .collect();
```
Some people prefer this style, and some people prefer to use closures. They **compile to the same code**, so use whichever style is clearer to you.

#### Returning Closures
Closures are represented by traits, which means you **can’t return closures directly**. In most cases where you might want to return a trait, you can instead use the **concrete type** that implements the trait as the return value of the function. But you can’t do that with closures because they don’t have a concrete type that is returnable; you’re not allowed to use the function pointer `fn` as a return type, for example.

The following code tries to return a closure directly, but it won’t compile:
```rust
fn returns_closure() -> Fn(i32) -> i32 {
    |x| x + 1
}
```
The compiler error is as follows:
```console
error[E0277]: the trait bound `std::ops::Fn(i32) -> i32 + 'static:
std::marker::Sized` is not satisfied
 -->
  |
1 | fn returns_closure() -> Fn(i32) -> i32 {
  |                         ^^^^^^^^^^^^^^ `std::ops::Fn(i32) -> i32 + 'static`
  does not have a constant size known at compile-time
  |
  = help: the trait `std::marker::Sized` is not implemented for
  `std::ops::Fn(i32) -> i32 + 'static`
  = note: the return type of a function must have a statically known size
```
The error references the `Sized` trait again! Rust doesn’t know how much space it will need to store the closure. We saw a solution to this problem earlier. We can use a **trait object**:
```rust
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
```

### 19.6 Macros

The term `macro` refers to a family of features in Rust: `declarative` macros with `macro_rules!` and three kinds of `procedural` macros:

- Custom `#[derive]` macros that specify code added with the derive attribute used on structs and enums
- Attribute-like macros that define custom attributes usable on any item
- Function-like macros that look like function calls but operate on the tokens specified as their argument

#### The Difference Between Macros and Functions
Fundamentally, macros are a way of writing code that writes other code, which is known as **metaprogramming**. 

Metaprogramming is useful for reducing the amount of code you have to write and maintain, which is also one of the roles of functions. However, macros have some additional powers that functions don’t.

A function signature must declare the **number** and type of parameters the function has. Macros, on the other hand, can take a **variable number** of parameters: we can call `println!("hello")` with one argument or `println!("hello {}", name)` with two arguments. Also, **macros are expanded before the compiler interprets the meaning of the code**, so a macro can, for example, **implement a trait** on a given type. A function can’t, because it gets called at runtime and a trait needs to be implemented at compile time.

The **downside** to implementing a macro instead of a function is that macro **definitions are more complex** than function definitions because **you’re writing Rust code that writes Rust code.** Due to this indirection, macro definitions are generally more difficult to read, understand, and maintain than function definitions.

Another important difference between macros and functions is that you must **define macros or bring them into scope before you call them in a file**, as opposed to functions you can define anywhere and call anywhere.

#### Declarative Macros with `macro_rules!` for General Metaprogramming
```rust
#[macro_export]
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec

            /*
            let mut temp_vec = Vec::new();
            temp_vec.push(1);
            temp_vec.push(2);
            temp_vec.push(3);
            temp_vec
            */
        }
    };
}

let v: Vec<u32> = vec![1, 2, 3];
```

#### Procedural Macros for Generating Code from Attributes
The second form of macros is **procedural macros**, which act more like **functions** (and are a type of procedure). Procedural macros accept some **code as an input**, **operate** on that code, and produce some **code as an output** rather than matching against patterns and replacing the code with other code as declarative macros do.

Three kinds:
- Custom derive
- Attribute-like
- Function-like

When creating procedural macros, the definitions must reside in their own crate with a special crate type. 

#### How to Write a Custom `derive` Macro

```rust
use hello_macro::HelloMacro;
use hello_macro_derive::HelloMacro;

// Annotate their type with `#[derive(HelloMacro)]` to get a default implementation of the `hello_macro` function.
#[derive(HelloMacro)]
struct Pancakes;

fn main() {
    Pancakes::hello_macro();
    // Prints: Hello, Macro! My name is Pancakes!
}
```

```rust
// src/lib.rs
pub trait HelloMacro {
    fn hello_macro();
}
```

If we don't use `derive`, we need to write the implementation block for each type we wanted to use with `hello_macro`, like:
```rust
// If there is a type `struct Pie`, we need to `impl HelloMacro for Pie`.
impl HelloMacro for Pancakes {
    fn hello_macro() {
        // Because the type name is used here, `hello_macro` with default
        // implementation can't do it.
        println!("Hello, Macro! My name is Pancakes!");
    }
}
```
We can’t yet provide the `hello_macro` function with **default implementation** that will print the **name of the type** the trait is implemented on: **Rust doesn’t have reflection capabilities**, so it **can’t look up the type’s name at runtime**. We need a macro to generate code at compile time.

Refer to `ch19-06` about the implementation of `derive` macro.

#### Attribute-like macros
```rust
#[route(GET, "/")]
fn index() {
```
This `#[route]` attribute would be defined by the framework as a procedural macro. The signature of the macro definition function would look like this:
```rust
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```
The first `TokenStream` is for the contents of the attribute: the `GET, "/"` part. The second is the body of the item the attribute is attached to: in this case, `fn index() {}` and the rest of the **function’s body**.

#### Function-like macros
```rust
let sql = sql!(SELECT * FROM posts WHERE id=1);
```
Defination of `sql!` macro:
```rust
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

### 20 Final Project: Building a Multithreaded Web Server

Rather than spawning unlimited threads, we’ll have a **fixed number of threads waiting in the pool**. As requests come in, they’ll be sent to the pool for processing. The pool will maintain a queue of incoming requests. Each of the threads in the pool will pop off a request from this queue, handle the request, and then ask the queue for another request. With this design, we can process N requests concurrently, where N is the number of threads. If each thread is responding to a long-running request, subsequent requests can still back up in the queue, but we’ve increased the number of long-running requests we can handle before reaching that point.

This technique is just one of many ways to improve the throughput of a web server. Other options you might explore are the **fork/join model** and the **single-threaded async I/O model**. If you’re interested in this topic, you can read more about other solutions and try to implement them in Rust; with a low-level language like Rust, all of these options are possible.

Before we begin implementing a thread pool, let’s talk about what using the pool should look like. When you’re trying to design code, **writing the client interface first** can help guide your design. Write the API of the code so it’s structured in the way you want to call it; **then implement the functionality within that structure** rather than implementing the functionality and then designing the public API.

Similar to how we used test-driven development in the project in Chapter 12, we’ll use **compiler-driven development** here. We’ll write the code that calls the functions we want, and then we’ll look at errors from the compiler to determine what we should change next to get the code to work.
