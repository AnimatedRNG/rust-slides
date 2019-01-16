# Spot the Bug! #

```python
# we want to create a 2D NxN grid full of ones
grid = []
EMPTY = []
for x in range(N):
    grid.append(EMPTY)
    for y in range(N):
    grid[-1].append(1)
```

---

# Spot the Bug! #

```python
# we want to create a 2D NxN grid full of ones
grid = []
EMPTY = []
for x in range(N):
    grid.append(EMPTY)
    for y in range(N):
    grid[-1].append(1)
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

* Explicit owership is the biggest new feature Rust brings to the table!
* Checked at compile time!
* Easy to find yourself "fighting the borrow checker" at first.

---

# Variables have Scope

```rust
{
    // s is not valid here; it has not been declared
    
    let s = "hello!";
    
    // s is valid here
    
}

// the scope is now over, and s is no longer valid

println!("{}", s);

```

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
    let alice_index = people
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
