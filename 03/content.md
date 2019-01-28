# Rust: Generics and Traits #

![Ferris](http://rustacean.net/assets/rustacean-orig-noshadow.png)

#### Srinivas Kaza and Matthew Pfeiffer ####
###### *Sponsored by SIPB and Course 6* ######

---

# Generic Enums/structs

Same as lifetimes. std implementation of Option:

```rust
pub enum Option<T> {
    None,
    Some(T),
}


impl<T> Option<T> {
    pub fn unwrap(self) -> T {
        match self {
            Some(val) => val,
            None => panic!("called `Option::unwrap()` on a `None` value"),
        }
    }}
}
```

---

# Generic Function Params

Easy peasy code to get the max element in an array.

```rust
fn main() {
    let numbers = vec![34, 50, 25, 100, 65];

    let mut largest = numbers[0];

    for number in numbers {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);
}
```

---

# Generic Function Params

Let's make it a function...

```rust
fn largest(list: &[i32]) -> i32 {
    let mut largest = list[0];

    for &item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let numbers = vec![34, 50, 25, 100, 65];

    let result = largest(&numbers);
    println!("The largest number is {}", result);

}
```

---

# Generic Function Params

Let's let it take in ANYTHING!

```rust
fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}
```

---

# Generic Function Params

Let's let it take in ANYTHING!

```rust
fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}
```

```
error[E0369]: binary operation `>` cannot be applied to type `T`
 --> src/main.rs:7:16
  |
7 |             if item > largest {
  |                ^^^^^^^^^^^^^^
  |
  = note: `T` might need a bound for `std::cmp::PartialOrd`
```

---

# Generic Function Params

Solution: Traits!

```rust
fn largest<T: std::cmp::PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}
```

Let's explain.

---

# Traits

Traits are similar to Java interfaces or Haskell typeclasses.

```rust
trait PrettyPrint {
    fn format(&self) -> String;
}
```

They mostly just contain method signatures without definitions.

They are not the same as an `impl` block.


---

You may implement a trait for a type as follows:

```rust
struct Point {
    x: i32,
    y: i32,
}
impl PrettyPrint for Point {
    fn format(&self) -> String {
        format!("({}, {})", self.x, self.y)
    }
}
```

You must define all methods specified by the trait.

Otherwise, the `impl` block is pretty similar to regular `impl` blocks.

---

# Trait Bounds

We can constrain generic types to those that implement one or more traits:

```rust
fn largest<T: PartialOrd>(list: &[T]) -> T;

fn largest<T: PartialOrd + Copy>(list: &[T]) -> T;

fn largest<T>(list: &[T]) -> T
    where T: PartialOrd + Copy;
```

---

# Trait Bounds in `impl`

```rust
enum Result<T, E> {
   Ok(T),
   Err(E),
}
trait PrettyPrint {
   fn format(&self) -> String;
}
impl<T: PrettyPrint, E: PrettyPrint> PrettyPrint for Result<T, E> {
   fn format(&self) -> String {
      match *self {
         Ok(t) => format!("Ok({})", t.format()),
         Err(e) => format!("Err({})", e.format()),
      }
   }
}
```

---

# Example: Equality

```rust
enum Result<T, E> { Ok(T), Err(E), }
// This is not the trait Rust actually uses for equality
trait Equals {
   fn equals(&self, other: &Self) -> bool;
}
impl<T: Equals, E: Equals> Equals for Result<T, E> {
   fn equals(&self, other: &Self) -> bool {
      match (*self, *other) {
         Ok(t1), Ok(t2) => t1.equals(t2),
         Err(e1), Err(e2) => e1.equals(e2),
         _ => false
      }
   }
}
```

`Self` is a special type that refers to the type of `self` in an `impl` block.

---

# Trait "Inheritance"

Some traits may depend on other traits.

* e.g. `Eq` requires `PartialEq`, and `Copy` requires `Clone`.

```rust
trait Parent {
    fn foo(&self) {
        // ...
    }
}
trait Child: Parent {
    fn bar(&self) {
        self.foo();
        // ...
    }
}
```

To implement Child, you must first implement Parent.

---

# Default Methods

If a default implementation for a trait method is provided in the `trait` block, the implementor doesn't have to define that method.

```rust
trait PartialEq<Rhs: ?Sized = Self> {
    fn eq(&self, other: &Rhs) -> bool;
    fn ne(&self, other: &Rhs) -> bool {
        !self.eq(other)
    }
}
trait Eq: PartialEq<Self> {}
```

These may also be overridden by the implementor.

---

# Deriving Traits

Rust can automatically implement some easy traits for you, so you don't have to manually write e.g. `Clone` over and over.

```rust
#[derive(Eq, PartialEq, Debug)]
struct Point {
    x: i32,
    y: i32,
}
```

You can do this for many core traits: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`.

In general, you can only derive a trait if all your members have that trait defined.

---

# Lots of the Core traits

They're useful, so we're gonna briefly dive into each of them:

- `Clone`, `Copy`
- `Debug`, `Display`
- `Default`
- `Drop`
- `Eq`, `PartialEq`
- `Hash`
- `Ord`, `PartialOrd`

---

### Clone

```rust
pub trait Clone: Sized {
    fn clone(&self) -> Self;

    fn clone_from(&mut self, source: &Self) { ... }
}
```
- A trait which defines how to duplicate a value of type `T`.
- This can solve ownership problems.
    - You can clone an object rather than taking ownership or borrowing!

---
### Clone

```rust
#[derive(Clone)] // without this, Bar cannot derive Clone.
struct Foo {
    x: i32,
}

#[derive(Clone)]
struct Bar {
    x: Foo,
}
```

---
### Copy
```rust
pub trait Copy: Clone { }
```
- `Copy` denotes that a type has "copy semantics" instead of "move semantics."
- Type must be able to be copied by copying bits (`memcpy`).
    - Types that contain references _cannot_ be `Copy`.
- Marker trait: does not implement any methods, but defines behavior instead.
- In general, if a type _can_ be `Copy`, it _should_ be `Copy`.

---
### Debug

```rust
pub trait Debug {
    fn fmt(&self, &mut Formatter) -> Result;
}
```

- Defines output for the `{:?}` formatting option.
- Generates debug output, not pretty printed.
- Generally speaking, you should always derive this trait.

```rust
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

let origin = Point { x: 0, y: 0 };
println!("The origin is: {:?}", origin);
// The origin is: Point { x: 0, y: 0 }
```

---

### Display

```rust
pub trait Display {
    fn fmt(&self, &mut Formatter) -> Result<(), Error>;
}

impl Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "Point {}, {})", self.x, self.y)
    }
}
```

- Defines output for the `{}` formatting option.
- Like Debug, but should be pretty printed.
    - No standard output and cannot be derived!
- You can use `write!` macro to implement this without using Formatter.

---
### Default

```rust
pub trait Default: Sized {
    fn default() -> Self;
}
```
- Defines a default value for a type.

---
### Drop

```rust
pub trait Drop {
    fn drop(&mut self);
}
```
- A trait for types that are destructable (which is all types).
- You never need to manually implement or derive Drop.
- Question: when would you want to implement Drop yourself?

---
### Drop
- Example: `Rc<T>` is Rust's reference-counted pointer type.
- It has special `Drop` rules:
    - If the number of references to the `Rc` pointer is greater than 1, `drop` decrements the reference count.
    - If the reference count drops to 0, the data is actually deleted.

---
### Eq vs. PartialEq

```rust
pub trait PartialEq<Rhs: ?Sized = Self> {
    fn eq(&self, other: &Rhs) -> bool;

    fn ne(&self, other: &Rhs) -> bool { ... }
}

pub trait Eq: PartialEq<Self> {}
```
- Traits for defining equality via the `==` operator.

---
### Eq vs. PartialEq

- `PartialEq` represents a _partial equivalence relation_.
    - Symmetric: if a == b then b == a
    - Transitive: if a == b and b == c then a == c
- `ne` has a default implementation in terms of `eq`.
- `Eq` represents a _total equivalence relation_.
    - Symmetric: if a == b then b == a
    - Transitive: if a == b and b == c then a == c
    - **Reflexive: a == a**
- `Eq` does not define any additional methods.
    - (It is also a Marker trait.)
- Question: why don't floating point types define `Eq`?

---
### Hash

```rust
pub trait Hash {
    fn hash<H: Hasher>(&self, state: &mut H);

    fn hash_slice<H: Hasher>(data: &[Self], state: &mut H)
        where Self: Sized { ... }
}
```
- A hashable type.
- The `H` type parameter is an abstract hash state used to compute the hash.
- If you also implement `Eq`, there is an additional, important property:
```rust
k1 == k2 -> hash(k1) == hash(k2)
```

&sup1;taken from Rustdocs

---
### Ord vs. PartialOrd

```rust
pub trait PartialOrd<Rhs: ?Sized = Self>: PartialEq<Rhs> {
    // Ordering is one of Less, Equal, Greater
    fn partial_cmp(&self, other: &Rhs) -> Option<Ordering>;

    fn lt(&self, other: &Rhs) -> bool { ... }
    fn le(&self, other: &Rhs) -> bool { ... }
    fn gt(&self, other: &Rhs) -> bool { ... }
    fn ge(&self, other: &Rhs) -> bool { ... }
}
```
- Traits for values that can be compared for a sort-order.

---
### Ord vs. PartialOrd

- The comparison must satisfy, for all `a`, `b` and `c`:
  - Antisymmetry: if `a < b` then `!(a > b)`, as well as `a > b` implying `!(a < b)`; and
  - Transitivity: `a < b` and `b < c` implies `a < c`. The same must hold for both `==` and `>`.
- `lt`, `le`, `gt`, `ge` have default implementations based on `partial_cmp`.

&sup1;taken from Rustdocs

---
### Ord vs. PartialOrd

```rust
pub trait Ord: Eq + PartialOrd<Self> {
    fn cmp(&self, other: &Self) -> Ordering;
}
```
- Trait for types that form a total order.
- An order is a total order if it is (for all `a`, `b` and `c`):
  - total and antisymmetric: exactly one of `a < b`, `a == b` or `a > b` is true; and
  - transitive, `a < b` and `b < c` implies `a < c`. The same must hold for both `==` and `>`.
- When this trait is derived, it produces a lexicographic ordering.
- Question: why don't floating point types define `Eq`?

&sup1;taken from Rustdocs

---
# Associated Types

Say we have a generic graph trait with node and edge types...
```rust
trait Graph<N, E> {
    fn edges(&self, &N) -> Vec<E>;
    // etc
}
```

If we write a function taking in a graph, it has to be generic over *any* N and E. What if our implementation only uses specific Node and Edge types?
```rust
fn distance<N, E, G: Graph<N,E>>(graph: &G, start: &N, end: &N)
    -> u32
```

---

# Associated Types

This can be improved by declaring *associated types* in a trait block.

An implementor may specify what the associated types correspond to.

```rust
trait Graph {
  type N;
  type E;
  fn edges(&self, &Self::N) -> Vec<Self::E>;
}
impl Graph for MyGraph {
  type N = MyNode;
  type E = MyEdge;
  fn edges(&self, n: &MyNode) -> Vec<MyEdge> { /*...*/ }
}
```

```rust
fn distance<G: Graph>(graph: &G, start: &G::N, end: &G::N) -> u32 { ... }
```

---

# Trait Scope

* You can implement your traits for any type in Rust, including types you don't own.

```rust
trait Foo {
   fn bar(&self) -> bool;
}
impl Foo for i32 {
    fn bar(&self) -> bool {
        true
    }
}
```

But this can be bad practice, depending on the situation.

---

# Trait Scope Rules

You need to `use` a trait in order to access its methods.

In order to write an `impl`, you must have defined either the type or the trait yourself.
