# Example-Driven Rust #

![Ferris](http://rustacean.net/assets/rustacean-orig-noshadow.png)

#### Srinivas Kaza and Matthew Pfeiffer ####
###### *Sponsored by SIPB and Course 6* ######

---

# Sections #

1. **What is Rust and Why Should I Care?**
2. Example Time!
3. Setting up Rust on Your Machine

---

# History #

Rust is really new!

* Introduced by Mozilla in 2010
* 1.0 released in 2015
* Currently on 1.31.1

Most languages we use first appeared in the 90s or earlier.

---

# Who is using Rust? #

![Yelp](https://s3-media2.fl.yelpcdn.com/assets/srv0/styleguide/1ea40efd80f5/assets/img/brand_guidelines/yelp_fullcolor.png)

![Dropbox](https://upload.wikimedia.org/wikipedia/commons/thumb/7/74/Dropbox_logo_%282013%29.svg/858px-Dropbox_logo_%282013%29.svg.png)

---

# What is Rust used for? #

* OS
  * redox 
* Graphics Programming
  * glium, vulkano, gfx-rs, point-cloud-viewer, etc
* Browser (mostly Firefox)
  * servo, webrender
* Back-end Web Development
  * iron, rocket, gotham, etc
* Front-end Web Development
  * yew, stdweb, etc
* Distributed Systems
  * noria, timely-dataflow, basementdb*

---

# Why use Rust? #

1. Safety
2. Performance
3. Ergonomics

**These goals are not contradictory!**

---

# Safety #

* No null pointers
* No pointer aliasing
* No default initialization
* No invalid/dangling pointers
* No double free
* Resource Acquisition is Initialization
* Out-of-bounds handling
* Overflow handling
* More explicit casting system

---

# Performance #

* No garbage collection!
* Compiles down to LLVM
* Safety/ergonomic features (usually) don't have runtime costs
  * Most of the magic happens at compile time
* Explicit and automatic SIMD support
* Eager evaluation

---

# Ergonomics #

* Variables are immutable by default
* Pattern matching
* Iterators
* First-class functions
* Algebraic datatypes

---

# Language Properties #

* Statically Typed
  * Types are determined at compile time
* Inferred Types
  * The compiler guesses the type
* Memory Managed
  * No garbage collector
* Functional (kind of)
  * Objects are generally immutable, operated on by functions
* Linear
  * Objects are used exactly once

---

# Sections #

1. What is Rust and Why Should I Care?
2. **Example Time!**
3. Setting up Rust on Your Machine

---

# Variables #

* You can "bind" a variable with the `let` keyword

```rust
let name = "Alice";
```

* The compiler guesses the type, but you can specify it explicitly if you want. `name` is a `&str` (fixed-length string).

```rust
let name: &str = "Alice";
```

* Variables are **immutable** by default. `mut` makes variables mutable.

```rust
let mut name = String::from("Alice");
name.push_str(" in Wonderland");
```

* `String` is the heap-allocated dynamic-sized string type.

---

# Bindings #

* Mutability is bad though. Is there a better solution?

```rust
let name: &str = "Alice";
let name: String = format!("Alice {}", "in Wonderland");
```

* Bindings can be shadowed. This has nothing to do with mutability!

```rust
let example = 1;
let example = 2;
let example = 3;

// `example` is never mutated!
```

---

# Primitive Types #

* Numeric types include width in the type
  * Unsigned types: `u8`, `u16`, `u32`, `u64`, `usize`
  * Signed types: `i8`, `i16`, `i32`, `i64`, `isize`
  * Floating-point types: `f32`, `f64`
  
* Fixed-size primitive arrays

```rust
let example_array: [i32; 3] = [1, 1, 1];
// Equivalent to the above
let example_array: [i32; 3] = [1; 3];
```

* Slices

```rust
let example_array: [i32; 3] = [1, 1, 1];
let example_slice: &[i32] = &example_array[0..1];
```

---

# Primitive Types #

* Rust has tuples and simple destructuring. Tuples are indexed with the `.` operator.

```rust
let book_details = ("Alice in Wonderland", "Caroll", 1865);

// Note that _ ignores a binding
let (title, _, year) = book_details;

println!("{} was printed in {}", title, year);
```

* Functions

```rust
fn get_publication_year(book_details: (&str, &str, i32)) -> i32 {
    book_details.2
}

fn main() {
    let book_details = ("Alice in Wonderland", "Caroll", 1865);

    println!("{}", get_publication_year(book_details));
}
```

---

# Expressions #

* An expression is code that can be evaluated.

* A function body can be an expression.

```rust
fn get_publication_year(book_details: (&str, &str, i32)) -> i32 {
    book_details.2
}
```

* `if` statements can be expressions too!

```rust
let title = if is_sequel(book) {
    "Through the Looking-Glass"
} else {
    "Alice in Wonderland"
};
```

---

# Structs #

* Rust doesn't have classes, but it does have structs!

```rust
struct Book {
    title: String,
    author: String,
    year: u32,
    isbn: u64,
}

let alice = Book {
    title: String::from("Alice in Wonderland"),
    author: String::from("Lewis Caroll"),
    year: 1865,
    isbn: 9780486275437
};
```

---

# Enums #

* Rust also has enums (somewhat akin to `union`s in C).

```rust
enum Suit {
    Diamonds,
    Clubs,
    Hearts,
    Spades,
}
```

* Rust also has support for tagged enumerations

```rust
enum Color {
    Rgb(u8, u8, u8),
    Rgba(u8, u8, u8, u8),
}
```

---

# Options #

* In many languages, `null` is often used to signify an empty object

```java
if (title == null) {
    //...
}
```

* This practice often causes unintended (and difficult to track) bugs. Rust introduces `Option`s to help handle these cases.

```rust
let title: Option<&str> = Some("Alice in Wonderland");
```

* An option can either be `Some` or `None`. The `unwrap` function gets the contents of an option, and will `panic` if it is `None`.

---

# Conditionals #

* Like practically every other language, Rust has `if` statements.

```rust
fn print_book_title(title: Option<(&str)>) {
    // parentheses are not needed
    if title.is_none() {
        return;
    } else {
        println!("{}", title.unwrap());
    }
}
```

* While `if` statements are useful, we have better tools at our disposal.

---

# Pattern Matching #

* The match statement/expression is like `switch`

```rust
let legs = 3;
let riddle_answer = match legs {
    2 => "adult",
    3 => "elderly",
    4 => "infant",
    _ => "idk"
};
```

* Tuples can be destructured inside of patterns

```rust
let aiw = ("Alice in Wonderland", "Caroll", 1865);
let ttlg = ("Through the Looking-Glass", "Caroll", 1865);
match aiw {
    ("Alice in Wonderland", "Caroll", 1865) => println!("Great!"),
    ("Alice in Wonderland", "Caroll", _) => println!("Wrong year!"),
    ("Alice in Wonderland", _, 1865) => println!("Wrong author!"),
    (_, "Caroll", _) => println!("A great author!"),
    _ => println!("I don't have anything to say about that."),
};
```
---

# Pattern Matching #

* Using the `match` statement on `Option`s is very common.

```rust
fn get_book_cost(book: Option<&str>) -> f32 {
    match book {
        Some("Alice in Wonderland") => 20.0,
        Some(_) => 1.0,
        None => 0.0,
    }
}
```

* You can also pattern match on enums!

```rust
enum Color {
    Rgb(u8, u8, u8),
    Rgba(u8, u8, u8, u8),
}

let red = Color::RGB(255, 0, 0);
match red {
    Color::RGBA(_, _, _, 0) => println!("Invisible"),
    _ => println!("Very colorful!"),
};
```

---

# Looping #

* `loop` loops forever

```rust
loop {
    println!("Looping forever!")
}
```

* `for` loops will loop upon an iterator (or range)

```rust
for i in 0..10 {
    println!("i is {}", i);
}
```

* Of course `while` loops are supported as well

```rust
let mut i = 0;

while i < 10 {
    println!("i is {}", i);
    i += 1;
}
```
---

# Looping #

* We can loop on anything that provides an iterator interface!

```rust
let books = ["Alice in Wonderland", "Through the Looking-Glass"];

for book in &books {
    println!("You should read {}", book);
}
```

---

# Ownership #

* Every value in Rust is owned by a variable.

* **There can only be one owner for a value at a given time**. 

* When the owner goes out of scope, the variable is **dropped**.

```rust
fn enter_wonderland(entrant: String) {
    println!("{} is in Wonderland forever!", entrant);
}

fn after_the_book_ends(person: String) {
    println!("No one cares about this part")
}

fn main() {
    let alice = String::from("Alice");
    enter_wonderland(alice);

    // Uncommenting this would be an error
    //after_the_book_ends(alice);
}
```

---

# Sections #

1. What is Rust and Why Should I Care?
2. Example Time!
3. **Setting up Rust on Your Machine**

---

# Installing Rust #

* The easiest way to install Rust is with rustup. Go to https://rustup.rs/ or:

`curl https://sh.rustup.rs -sSf | sh`

* To keep your rust up-to-date with the latest stable version of rust:

`rustup update`

* To check which version of Rust you have:

`rustc --version`

---

# Cargo #

Cargo is a tool that helps you develop Rust programs. It does several things:
* Runs tasks:
  * `cargo build` (compile your app)
  * `cargo test` (test your app)
  * `cargo run` (run your app)
* Start a project: `cargo new`, `cargo init`

Cargo is also the package manager for Rust.
  * A program or library is called a "crate".
  * A package contains one or more crates.
  * You can find Crates on http://crates.io
  * You list the Crates you want to use in the Cargo.toml file

---

# Creating a New Project #

`cargo new --bin name-of-my-project`
* Use `--lib` if you are writing a library (you don't want to compile your code into an executable).
`cd name-of-my-project`

This creates several files and folders for you automatically:
* `Cargo.toml`: metadata about your project and its dependencies
* `.gitignore`: ignores compiled files built by Rust
* `src/*.rs`: where your Rust code goes

---

## Cargo.toml is not a Makefile ##

Cargo uses Cargo.toml for project metadata and dependencies.

```cargo
[package]
name = "basementdb"
version = "0.1.0"
authors = ["Matthew Pfeiffer <spferical@gmail.com>",
           "Srinivas Kaza <kaza@mit.edu>"]

[dependencies]
bincode = "1.0.0"
data-encoding = "2.1.1"
serde = "1.0"
serde_derive = "1.0"
serde_json = "1.0"
base64 = "0.9.0"
config = "0.8"
sodiumoxide = "0.0.16"
bufstream = "0.1"
scoped_threadpool = "0.1.9"
chrono = "0.4"
rand = "0.4.2"
clap = "2.3.2"
```
---

# `cargo test`

* A test is any function annotated with `#[test]`
* `cargo test` runs all tests in your project.

```rust
#[test]
fn it_works() {
    assert_eq!(2 + 2, 4);
}
```

</br>

```bash
$ cargo test
   Compiling name-of-my-project v0.1.0 (/home/matthew/dev/name-of-my-project)
    Finished dev [unoptimized + debuginfo] target(s) in 0.65s
     Running target/debug/deps/name_of_my_project-c45ef0564e3373a0

running 1 test
test it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out```k
```

---

# `rustfmt` #

`rustfmt` is a handy tool that can format your code! To install:

`rustup component add rustfmt`

To reformat all your code:

`cargo fmt`

---

