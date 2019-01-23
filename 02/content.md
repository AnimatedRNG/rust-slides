# Lifetimes

```rust
struct HelloPrinter {
    name: String,
}

impl HelloPrinter { // impl blocks: add methods to structs!
    fn new(name: String) -> Self {
        HelloPrinter { name }
    }
    fn print_hello(&self) {
        println!("Hello, {}!", self.name);
    }
}

fn main() {
    let world: String = "World".to_string();
    let hp = HelloPrinter::new(world.clone());
    hp.print_hello();
    println!("We still own world: {}", world);
}
```

How do we do this without copying things?

---

# Lifetimes

```rust
// this doesn't compile yet...
struct HelloPrinter {
    name: &str,
}

impl HelloPrinter {
    fn new(name: &str) -> HelloPrinter {
        HelloPrinter { name }
    }
    fn print_hello(&self) {
        println!("Hello, {}!", self.name);
    }
}

fn main() {
    let world: String = "World".to_string();
    let hp = HelloPrinter::new(&world);
    hp.print_hello();
}
```

---

# Lifetimes

```rust
// this doesn't compile yet...
struct HelloPrinter {
    name: &str,
}

impl HelloPrinter {
    fn new(name: &str) -> HelloPrinter {
        HelloPrinter { name }
    }
    fn print_hello(&self) {
        println!("Hello, {}!", self.name);
    }
}

fn main() {
    let world: String = "World".to_string();
    let hp = HelloPrinter::new(&world);
    drop(world); // this explicitly drops the value
    hp.print_hello();
}
```

---

# Rust: Lifetimes #

![Ferris](http://rustacean.net/assets/rustacean-orig-noshadow.png)

#### Srinivas Kaza and Matthew Pfeiffer ####
###### *Sponsored by SIPB and Course 6* ######


---

# Lifetimes

Up to this point, we've relied on *implicit* lifetimes.

```rust
fn string_length(s: &String) -> usize;
```

We can also *explicitly* provide one for Rust to use.

```rust
fn string_length<'a>(s: &'a String) -> usize;
```

`'a` AKA "tick-a" AKA "the lifetime a" is a named lifetime parameter.

The type `&'a String` is a reference to a String that lives at least as long as the lifetime `'a`.

---


# Lifetimes

The compiler is smart enough to guess lifetimes for simple cases.

But functions that involve multiple references or returning references may require explicit lifetimes.

Also, storing references in structs requires explicit lifetimes.

---

# Multiple Lifetime Parameters

```rust
fn borrow_x_or_y<'a>(x: &'a str, y: &'a str) -> &'a str;
```

- In this example, everything has the same lifetime.
- `x` and `y` are borrowed as long as the returned reference exists.

```rust
fn borrow_p<'a, 'b>(p: &'a str, q: &'b str) -> &'a str;```
```

- In this example, we return a reference with the same lifetime as `p`.
- `q` has a separate lifetime that may be unrelated to `p`.
- `p` is borrowed as long as the returned reference exists.

---

# Rules of Lifetimes

- Lifetimes can be pretty confusing until you use them more, but here are the important rules.
- If a reference `&x` has a lifetime `'a`, it *will not outlive* its underlying data.
- If a reference `&x` has a lifetime `'a`, anything else with the same lifetime *will live at least as long as* `&x`.

---

# Lifetimes in `structs`

```rust
struct Pizza(Vec<i32>);
struct PizzaSlice<'a> {
    pizza: &'a Pizza,  // <- references in structs must
    index: u32,        //    ALWAYS have explicit lifetimes
}
let p1 = Pizza(vec![1, 2, 3, 4]);
{
    let s1 = PizzaSlice { pizza: &p1, index: 2 }; // this is okay
}
let s2;
{
    let p2 = Pizza(vec![1, 2, 3, 4]);
    s2 = PizzaSlice { pizza: &p2, index: 2 };
    // no good - why?
}
```

---

* You can tell rust that a lifetime "outlives" another.

```rust
struct Pizza(Vec<i32>);
struct PizzaSlice<'a> { pizza: &'a Pizza, index: u32 }
struct PizzaConsumer<'a, 'b: 'a> { // says "b outlives a"
    slice: PizzaSlice<'a>, // <- currently eating this one
    pizza: &'b Pizza,      // <- so we can get more pizza
}
fn get_another_slice(c: &mut PizzaConsumer, index: u32) {
    c.slice = PizzaSlice { pizza: c.pizza, index: index };
}
let p = Pizza(vec![1, 2, 3, 4]);
{
    let s = PizzaSlice { pizza: &p, index: 1 };
    let mut c = PizzaConsumer { slice: s, pizza: &p };
    get_another_slice(&mut c, 2);
}
```

---

# Special `'static` Lifetime

* The `'static` lifetime means a reference will never go invalid.
* e.g. all `&str` literals have the `'static` lifetime because they're stored in a special portion of our program's memory.
* 
```
fn get_hello_world() -> &'static str {
    "Hello, World!"
}

fn main() {
    println!("{}", get_hello_world());
}
```

# Lifetimes in `impl` blocks.

Implementing methods requires lifetime annotations, too.

You can read this block as "the implementation using the lifetimes `'a` and `'b` for the struct `Foo` using the lifetimes `'a` and `'b`.

```rust
struct Foo<'a, 'b> {
  v: &'a Vec<i32>,
  s: &'b str,
}

impl<'a, 'b> Foo<'a, 'b> {
  fn new(v: &'a Vec<i32>, s: &'b str) -> Foo<'a, 'b> {
    Foo {
      v: v,
      s: s,
    }
  }
}
```
