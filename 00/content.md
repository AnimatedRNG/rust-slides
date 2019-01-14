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
let name: String = format!("Alice {}", "and Bob");
```

* Bindings can be shadowed. This has nothing to do with mutability!

```rust
let example = 1;
let example = 2;
let example = 3;

// `example` is never mutated!
```
