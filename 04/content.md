# Rust: Concurrency #

![Ferris](http://rustacean.net/assets/rustacean-orig-noshadow.png)

#### Srinivas Kaza and Matthew Pfeiffer ####
###### *Sponsored by SIPB and Course 6* ######

---

# Overview

There are a few important language features that will be useful for writing threaded programs

* Function pointers -- what are `Fn`, `FnOnce`, and `FnMut`?

* Smart pointers -- what are `Box` and `Rc`?

* Interior Mutability -- what is `RefCell`?

Finally, we will discuss Rust's concurrency primitives and how to use them.

---

# Function Pointers

Remember closures?

```rust
let count_items = |x: &Vec<_>| {
    x.iter().count()
};

assert_eq!(count_items(&vec![1, 2, 3]), 3);
```

They were cool because we could capture variables in the environment.

```rust
let x = vec![1, 2, 3];

let count_items_in_x = || {
    x.iter().count()
};

assert_eq!(count_items_in_x(), 3);
```

---

# Function Pointers

Given that closures are objects, you might wonder how we pass them into functions.

But first, let's cover a simpler case -- function pointers.

```rust
fn multiply_all<T>(x: &Vec<T>, y: T) -> Vec<T>
where
    T: Mul<T, Output = T> + Copy,
{
    x.iter().map(|&elem| elem * y).collect()
}

fn chain_ops<T>(
    ops: &Vec<fn(&Vec<T>, T) -> Vec<T>>,
    data: &Vec<T>,
) -> Vec<T> {
    // ...
}
```
---

# Closures

Closures aren't function pointers -- in fact, they don't have a concrete type!

Instead, they implement one or more of the following traits:

* `FnOnce(self)` -- functions that can be called once
* `Fn(&self)` -- functions that can be called many times
* `FnMut(&mut self)` -- functions that can be called many times with mutable environment access

Function pointers are concrete types that implement all three. 

---

# Closures as arguments


We can pass in closures as arguments -- just remember that `Fn` is a trait and unsized.

```rust
fn chain_ops<T>(ops: &Vec<&Fn(&Vec<T>, T) -> Vec<T>>, 
                data: &Vec<T>) -> Vec<T> {
    Vec::new()
}
```

For an operation like `chain_ops`, why is `Fn` the right choice?

---

# Closures as return types

We could pass in closures as references before, but usually we cannot (easily) return references.

```rust
fn returns_closure() -> Fn(i32) -> i32 {
    |x| x + 1
}
```

We also can't just return a `Fn` object -- why?

---

# Closures as return types

We could pass in closures as references before, but usually we cannot (easily) return references.

```rust
fn returns_closure() -> Fn(i32) -> i32 {
    |x| x + 1
}
```

We also can't just return a `Fn` object -- why?

```rust
error[E0277]: the trait bound `std::ops::Fn(i32) -> i32 + 'static:
std::marker::Sized` is not satisfied
 -->
  |
1 | fn returns_closure() -> Fn(i32) -> i32 {
  |                         ^^^^^^^^^^^^^^ 
  does not have a constant size known at compile-time
```

---

# The `move` keyword

TODO: Fill me in!

---

# Smart Pointers: `Box<T>`

`Box<T>` is useful for storing data on the heap. It is similar to C++'s `unique_ptr<T>`.

```rust
let alice_in_box = Box::new("alice is in the heap");
```

For the most part, we can use objects in smart pointers just as we would normally

```rust
fn inspect_box<T: std::fmt::Debug>(boxed_stuff: &Box<T>) {
    println!("{:?} is in a box", boxed_stuff);
}

inspect_box(&Box::new(3u32));
```

`Box::clone()` will clone the underlying contents of the `Box`.

---

# Recursive Data Structures

```rust
enum List {
    Cons(i32, List),
    Nil,
}
```

This won't work.

---

# `Box<T>` to the Rescue!

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}
```

This will. Why?

---

# Multiple Ownership

Sometimes you want to share an object between multiple owners.


```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let a = Cons(5,
        Box::new(Cons(10,
            Box::new(Nil))));
    let b = Cons(3, Box::new(a));
    let c = Cons(4, Box::new(a));
}
```

Why won't this work?

---

# Smart Pointers -- `Rc<T>`

`Rc<T>` is a *reference-counted* smart pointer. It's like C++'s `shared_ptr<T>`.

That means that it **increments** a reference count when `Rc::new()` is called, and **decrements** a reference count when the `Rc` is dropped.

Thus, when the reference count is zero, it is safe to drop the underlying object.

We use `Rc::clone()` to create a new reference (i.e increase the reference count).

`Rc` only represents immutable references -- why?

---

# `Rc<T>` to the Rescue!

```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a));
    let c = Cons(4, Rc::clone(&a));
}
```

Both `b` and `c` can have a reference to `a`

---

# Interior Mutability

`Rc` is great and all, but what if we want to share a mutable reference?

That could be quite unsafe -- Rust forbids having multiple mutable references out at a time!

> * At any given time, you can have either (but not both of) one mutable reference or any number of immutable references.
> * References must always be valid.

But what if we know that we never have two mutable references out at the same time?

---

# `RefCell<T>`

`RefCell<T>` allows us to delay Rust's mutable borrowing rules until runtime.

The `RefCell::borrow_mut` returns a mutable reference to the contents of the RefCell, even though the `RefCell` is immutable itself.

`RefCell` checks that we don't have two mutable references out at a time!

---

# `Rc<T>` and `RefCell<T>`

```rust
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use List::{Cons, Nil};
use std::rc::Rc;
use std::cell::RefCell;

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));
    let b = Cons(Rc::new(RefCell::new(6)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(10)), Rc::clone(&a));
    *value.borrow_mut() += 10;

    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}
```

---

# Concurrency (finally)

We can spawn a new thread with the `thread::spawn` function.

```rust
use std::thread;
use std::time::Duration;

fn main() {
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

`thread::spawn` takes a closure as an argument

---

# Concurrency

`thread::spawn` returns a `JoinHandle`, upon which we can call `.join()` to wait for the other threads

```rust
use std::thread;
use std::time::Duration;

fn main() {
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

---

# Revisiting Closures

```rust
let v = vec![1, 2, 3];

let handle = std::thread::spawn(|| {
    println!("Here's a vector: {:?}", v);
});

handle.join().unwrap();
```

What's wrong with this code?

---

# Revisiting Closures

```rust
let v = vec![1, 2, 3];

let handle = std::thread::spawn(|| {
    println!("Here's a vector: {:?}", v);
});

handle.join().unwrap();
```

What's wrong with this code?

```rust
error[E0373]: closure may outlive the current function, but it borrows `v`,
which is owned by the current function
 --> src/main.rs:6:32
  |
6 |     let handle = thread::spawn(|| {
  |                                ^^ may outlive borrowed value `v`
7 |         println!("Here's a vector: {:?}", v);
  |                                           - `v` is borrowed here
  |
```

---

# Lifetimes and Threads

We don't know how long our new thread will run, so we don't know if our reference will always be valid!

We can make even more dangerous scenarios this way

```rust
let v = vec![1, 2, 3];

let handle = std::thread::spawn(|| {
    println!("Here's a vector: {:?}", v);
});

drop(v);

handle.join().unwrap();
```

---

# `move` to the Rescue!

```rust
let v = vec![1, 2, 3];

let handle = std::thread::spawn(move || {
    println!("Here's a vector: {:?}", v);
});

handle.join().unwrap();
```

`move` transfers ownership of captured variables into the closure.

---

# Concurrency

Data races are impossible in safe Rust.

A data race happens when two threads try to access the same location at memory at once, and

* One of the accesses is a write
* One of the accesses is unsynchronized

Rust cannot prevent general race conditions. This is a tall order.
