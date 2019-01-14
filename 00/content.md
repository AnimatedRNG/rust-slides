# Example-Driven Rust #

![Ferris](http://rustacean.net/assets/rustacean-orig-noshadow.png)

#### Srinivas Kaza and Matthew Pfeiffer ####

---

# Sections #

1. **What is Rust and Why Should I Care?**
2. Example Time!
3. Setting up Rust on Your Machine

---

# Who is using Rust? #

![Yelp](https://s3-media2.fl.yelpcdn.com/assets/srv0/styleguide/1ea40efd80f5/assets/img/brand_guidelines/yelp_fullcolor.png)

![Dropbox](https://upload.wikimedia.org/wikipedia/commons/thumb/7/74/Dropbox_logo_%282013%29.svg/858px-Dropbox_logo_%282013%29.svg.png)

---

# History #

* Introduced by Mozilla in 2010
* 1.0 released in 2015
* Currently on 1.31.1

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

* An option can either be `Some` or `None`. The `unwrap` function gets the contents of a book, and will `panic` if it is `None`.

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

* Note that we can shadow bindings within patterns!

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
* Runs tasks: cargo build (compile your app), cargo test (test your app), cargo run (run your app)
* Start a project: cargo new, cargo init

Cargo is also the package manager for Rust. This means that you can use Cargo to install and manage bits of other people's code.
* A program or library is called a "crate".
* A package contains one or more crates.
* You can find Crates on http://crates.io
* You list the Crates you want to use in the Cargo.toml file
* Your app keeps track of what crates you are using in the Cargo.lock file

---

# Creating a New Project #

* `cargo new --bin name-of-my-project`
  * Use `--lib` if you are writing a library (you don't want to compile your code into an executable).
* `cd name-of-my-project`

This creates several files and folders for you automatically:
* `Cargo.toml`: metadata about your project and its dependencies
* `.gitignore`: ignores compiled files built by Rust
* `src/*.rs`: where your Rust code goes

---

# `rustfmt` #

`rustfmt` is a handy tool that can format your code! To install:

`rustup component add rustfmt`

To reformat all your code:

`cargo fmt`

---

