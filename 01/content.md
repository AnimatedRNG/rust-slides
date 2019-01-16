# Spot the Python Bug! #

```python
# we want to create a 2D NxN grid full of ones
grid = []
EMPTY = []
for x in range(N):
    grid.append(EMPTY)
    for y in range(N):
    grid[-1].append(1)
print(grid)
```

---

# Spot the Python Bug! #

```python
# we want to create a 2D NxN grid full of ones
grid = []
EMPTY = []
for x in range(N):
    grid.append(EMPTY)
    for y in range(N):
    grid[-1].append(1)
print(grid)
```

result (N=3):
```python
[[1, 1, 1, 1, 1, 1, 1, 1, 1],
 [1, 1, 1, 1, 1, 1, 1, 1, 1],
 [1, 1, 1, 1, 1, 1, 1, 1, 1]]
```

---

# Ownership & Borrowing #

![Ferris](http://rustacean.net/assets/rustacean-orig-noshadow.png)

#### Srinivas Kaza and Matthew Pfeiffer ####
###### *Sponsored by SIPB and Course 6* ######

---


# Ownership & Borrowing

* Explicit ownership is the biggest new feature Rust brings to the table!
* Checked at compile time!
* Easy to find yourself "fighting the borrow checker" at first.

But first, some important language features that might be familiar...

---

# Vectors #

* Vectors are like fixed-size arrays but they are dynamically-sized.

```rust
let mut tea_party = vec!["Mad Hatter", "March Hare", "Dormouse"];
tea_party.push("Alice");

// Equivalent to
// let mut tea_party = Vec::new();
// tea_party.push("Mad Hatter");
// tea_party.push("March Hare");
// tea_party.push("Dormouse");
// tea_party.push("Alice");

```

* `Vec::new` infers the type of the vector!

---

# Lambdas

* Lambdas are inline functions

```rust
let add_alice = |mut people: Vec<&str>| people.push("Alice");
let mut tea_party = vec!["Mad Hatter", "March Hare", "Dormouse"];
add_alice(tea_party);
```

* Parameters are specified between the `|`s.

* The compiler always infers the return type of the lambda and can often infer the parameters as well!

---

# Closures

* Closures are lambdas that "capture" variables in their environment

```rust
let mut tea_party = vec!["Mad Hatter", "March Hare", "Dormouse"];
let mut add_alice = || tea_party.push("Alice");
add_alice();
```

* Now we can refer to variables that are defined outside of the closure.

* But what happens if the closure outlives the variables that it is referencing?

---

# Iterators

* Iterators are a nice functional-programming alternative to `for` loops.

* `iter` on a slice, array, or vector creates an iterator.

```rust
let people: Vec<(&str, usize)> = vec![
    ("Mad Hatter", 43),
    ("March Hare", 2),
    ("Dormouse", 7),
    ("Alice", 7),
];
let old_enough: Vec<&str> = people
    .iter()
    .filter(|(_, age)| age >= &18)
    .map(|(name, _)| *name)
    .collect();
old_enough
```

* `map`, `filter`, `collect`

---

# This isn't slow

* Iterator version takes 0.479 sec for 10M iterations.
* Fairly optimized `for` loop takes 0.448 sec for 10M iterations.
* With a bit more care, we can get even closer to the `for` loop performance.

---

# Ownership
* A variable binding *takes ownership* of its data.
* A piece of data only has *one* owner at a time.
* When a binding goes out of scope, the data is released.
* Data *must be guaranteed* to outlive its references.

```rust
fn foo() {
    let mut v1 = vec![1, 2, 3];
    v1.push(4);
}
```

---

# Some Variables get Copied

```rust
let x = 5;
let mut y = x;
y += 1;
println!("x: {}, y: {}", x, y);
// output: x: 5, y: 6
```

Integers are tiny values with a known size, so Rust implicitly copies them.

---

# Move Semantics
```rust
fn foo() {
    let v1 = vec![1, 2, 3];
    
    // ownership of the Vec object moves to v2
    let mut v2 = v1;
    v2.push(4);
    
    // error: use of moved value 'v1'
    println!("v1: {:?}, v2: {:?}", v1, v2);
}
```
* Data cannot have multiple owners, so we cannot have v1 and v2 refer to the same Vec.
* Copying data could be expensive.
* So, Rust says v2 now owns the Vec, and v1 is invalid.

* Note: moving ownership is just how Rust reasons about the data, it doesn't involve moving data around in memory.

---

# Ownership and Functions

```rust
fn string_length(s: String) -> usize {
    s.len()
}

fn main() {
    let alice = String::from("Alice");
    let len = string_length(alice);
    
    // Error: value used after move
    println!("The length of {} is {}", alice, len);
}
```

Function arguments get ownership moved into the called function.

Return values get ownership moved back to the caller.

But how do we keep alice?

---

# Ownership and Functions

```rust
fn string_length(s: String) -> (String, usize) {
    (s, s.len())
}

fn main() {
    let alice = String::from("Alice");
    let (alice, len) = string_length(alice);
    
    println!("The length of {} is {}", alice, len);
}
```

This works, but it's kinda tedious...

---

# Borrowing with References

```rust
fn string_length(s: &String) -> usize {
    // Note: this automatically "dereferences" the string reference s.
    // It is the same as (*s).len()
    s.len()
}

fn main() {
    let alice = String::from("Alice");
    let len = string_length(&alice);
    
    println!("The length of {} is {}", alice, len);
}
```

The string_length() function can *borrow* the string, using it while the alice variable in main() continues to own it!

The *borrow* lasts until the reference goes out of scope (at the end of string_length()).

---

# Borrowing with References

```rust
fn add_to_str(s: &String) {
    // error: cannot borrow immutable borrowed content `*s` as mutable
    s.push_str(" in Wonderland");
}

fn main() {
    let alice = String::from("Alice");
    add_to_str(&alice);
    
    println!("{}", alice);
}
```

References are *immutable* by default, just like bindings.

---

# Borrowing with References

```rust
fn add_to_str(s: &mut String) {

    s.push_str(" in Wonderland");
}

fn main() {
    let mut alice = String::from("Alice");
    add_to_str(&mut alice);
    
    println!("{}", alice);
}
```

References are *immutable* by default, just like bindings.

But you can have mutable references!

---

# Mutable Borrowing Rules

```rust
fn remove_alice_from_wonderland(people: &mut Vec<String>) {
    let alice = String::from("Alice");
    let alice_index: Option<usize> = people
        .iter()
        .position(|element| element == &alice);
    match alice_index {
        Some(alice_index) => {
            people.remove(alice_index);
        }
        None => (),
    };
}

fn main() {
    let wonderland = vec![String::from("Alice")];
    remove_alice_from_wonderland(&mut wonderland);
}
```

What's wrong here?

---

# Borrowing with References

```rust
fn add_to_str(s: &mut String) {
    // Error: cannot move out of borrowed content.
    let mut alice = *s;
    alice.push_str(" in Wonderland");
}

fn main() {
    let mut alice = String::from("Alice");
    add_to_str(&mut alice);
    
    println!("{}", alice);
}
```

Error! You cannot dereference alice into a variable binding, because that would change ownership of the data.

---

# Dereferencing

Rust will auto-dereference when making method calls or passing references into functions.

```rust
fn string_length(s: &&String) -> usize {
    s.len()
}

fn main() {
    let alice = String::from("Alice");
    let len = string_length(&&&&&&&&&&&&alice);

    println!("The length of {} is {}", alice, len);
}
```

---

# Dereferencing

You might need to explicitly dereference variables...
* when writing to them
* when there is other ambiguity

```rust
let mut foo = 10;
let ref_foo = &mut foo;
ref_foo = 100;
println!("{}", *ref_foo + 4);
```

---

# Borrowing Rules

##### *"The Holy Grail of Rust"*

1. One object may have many immutable references to it (&T).
   OR exactly one mutable reference (&mut T) (not both).
2. You can't keep borrowing something after it stops existing.

![Grail](https://cis198-2016s.github.io/slides/01/img/holy-grail.jpg)

---

# Python: Runtime Bug
```python
x = [1, 2, 3, 4, 5]
for y in x:
    x.remove(y)
print(x)
# output: [2, 4]
```

# Rust: Compile-time Error
```rust
let mut v = vec![1, 2, 3, 4, 5];
for x in &v {
    v.remove(*x);
}
```
Error: cannot borrow `v` as mutable because it is also borrowed as immutable.

---

# C/C++: Runtime Bug

```c
int *func(void) {
    int x = 1234;
    return &x;
}
```

# Rust Compile-time Error

```rust
fn func() -> &isize {
    let x = 1234;
    &x
}
```
Error: "missing lifetime specifier"
