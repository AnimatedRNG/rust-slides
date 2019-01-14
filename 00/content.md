# Example-Driven Rust #

![Ferris](http://rustacean.net/assets/rustacean-orig-noshadow.png)

#### Srinivas Kaza and Matthew Pfeiffer ####

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

# Example Time! #


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

# Installing Rust #

- The easiest way to install Rust is with rustup. Go to https://rustup.rs/ or:

`curl https://sh.rustup.rs -sSf | sh`

- To keep your rust up-to-date with the latest stable version of rust:

`rustup update`

- To check which version of Rust you have:

`rustc --version`

---

# Cargo #

Cargo is a tool that helps you develop Rust programs. It does several things:
    - Runs tasks: cargo build (compile your app), cargo test (test your app), cargo run (run your app)
    - Start a project: cargo new, cargo init

Cargo is also the package manager for Rust. This means that you can use Cargo to install and manage bits of other people's code.
    - A program or library is called a "crate".
    - A package contains one or more crates.
    - You can find Crates on http://crates.io
    - You list the Crates you want to use in the Cargo.toml file
    - Your app keeps track of what crates you are using in the Cargo.lock file

---

# Creating a New Project #

- `cargo new --bin name-of-my-project`
    - Use `--lib` if you are writing a library (you don't want to compile your code into an executable).
- `cd name-of-my-project`

This creates several files and folders for you automatically:
    - `Cargo.toml`: metadata about your project and its dependencies
    - `.gitignore`: ignores compiled files built by Rust
    - `src/*.rs`: where your Rust code goes

---

# Nice Tool: `rustfmt` to Format your Code #

To install:

`rustup component add rustfmt`

To reformat all your code:

`cargo fmt`
