---
topic: rust
status: wip
---

# structs and enums

Notes from the Rust Book / Rust By Practice chapters on structs and enums.

## struct

A struct groups related fields under one name. Three forms:
- named-field: `struct User { name: String, age: u8 }`
- tuple struct: `struct Point(i32, i32)`, fields by position, no names
- unit struct: `struct Marker;`, no fields, used as a marker

mut is on the whole instance, not per field.

```rust
struct User { name: String, active: bool }
let mut u = User { name: String::from("mj"), active: true };
u.active = false;   // needs the whole binding to be mut
```

Field init shorthand: when a variable name matches the field name, write `User { name, active }` instead of `name: name`.

Struct update syntax: `User { active: false, ..other }` fills the rest from another instance. It moves the non-Copy fields out of `other`.

## associated functions and methods

An `impl` block hangs functions off a type.
- method: first param is `&self` / `&mut self` / `self`. Called with `u.method()`.
- associated function (no self): called with `Type::func()`. Constructors like `String::from` and `String::new` are these.

```rust
impl User {
    fn new(name: String) -> Self {   // associated fn, the constructor
        User { name, active: true }
    }
    fn deactivate(&mut self) {        // method, borrows self mutably
        self.active = false;
    }
}
```

`Self` is the type the impl is for. `&self` is sugar for `self: &Self`.

## enum

An enum is a type that's exactly one of several variants. The power: each variant carries its own data, and different variants can carry different types.

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),   // four bytes
    V6(String),           // a string
}
let home = IpAddr::V4(127, 0, 0, 1);
let loop6 = IpAddr::V6(String::from("::1"));
```

V4 and V6 are the same type (IpAddr) but hold different shapes of data. A struct forces every instance to carry every field. An enum says "one of these, with the data that variant needs".

A variant can carry nothing (`Penny`), a tuple of values (`V4(u8,u8,u8,u8)`), a struct-like body (`Move { x: i32, y: i32 }`), or another type (`Quarter(UsState)`). Enums get `impl` blocks too, same as structs.

## Option

Rust has no null. Instead, a std enum:

```rust
enum Option<T> {
    None,
    Some(T),
}
```

A value that might be absent has type `Option<T>`, not `T`. You can't accidentally use a missing value as if it were present. The compiler forces you to handle `None` before you reach the `T` inside. That's how Rust kills null-pointer bugs at compile time.

`Some(5)` and `None` are values. To get at the inner T you must match, or use helpers like `unwrap`, `unwrap_or`, `?`.

## match

`match` compares a value against patterns top to bottom and runs the first arm that fits. It's an expression: the whole match evaluates to the chosen arm's value.

Each arm is a single expression or a `{}` block. A block returns its last expression, no semicolon on that line:

```rust
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1                       // block's value, returned from the arm
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

A pattern can pull the data out of a variant:

```rust
match coin {
    Coin::Quarter(state) => println!("quarter from {:?}", state),  // state is the UsState
    _ => {}
}
```

### matches are exhaustive

You must cover every variant. Miss one and it won't compile, and the compiler lists what you forgot. That's the point: add a variant later and every match that didn't handle it becomes a compile error, so you can't forget to update them.

`_` is the catch-all for everything else. Use it last.

## if let

When you only care about one pattern, `match` is noisy. `if let` is the shorthand:

```rust
if let Some(x) = maybe {
    println!("{}", x);
}
// optional else for the None / non-matching case
```

`if let` is a match with one real arm and a `_ => ()`. You trade exhaustiveness for brevity. Use it when you genuinely don't care about the other cases.

## let...else

Staying on the happy path. `let...else` binds a pattern, and if it doesn't match, the `else` block must diverge (return, break, panic). After it, the binding stays in scope for the rest of the function, no nesting.

```rust
fn describe(maybe: Option<i32>) -> String {
    let Some(x) = maybe else {
        return String::from("nothing");   // must diverge
    };
    // x is available here, unindented
    format!("got {}", x)
}
```

Versus `if let`: `if let` puts the success path inside the block (indented). `let...else` puts the failure path in the block and keeps the success path flat. Reach for it when the missing case is an early exit and you want the main logic un-nested.

## derive(Debug) and dbg!

By default a struct or enum can't be printed. `#[derive(Debug)]` auto-generates a debug formatter so `{:?}` (or `{:#?}` for pretty) works in `println!`.

```rust
#[derive(Debug)]
struct Point { x: i32, y: i32 }
let p = Point { x: 1, y: 2 };
println!("{:?}", p);     // Point { x: 1, y: 2 }
```

`{:#?}` is the same thing pretty-printed: one field per line, indented. Use it for anything nested or wider than a glance.

```rust
println!("{:#?}", p);
// Point {
//     x: 1,
//     y: 2,
// }
```

`dbg!` prints a value with its file:line and the expression text, then hands the value back. It goes to stderr. Handy for a quick look without restructuring code:

```rust
let w = dbg!(3 * 2);   // prints [src/main.rs:2] 3 * 2 = 6, then w = 6
```

The difference: `println!("{:?}", ..)` takes a reference and is for normal output. `dbg!` takes ownership and hands it back, prints the location, and is for temporary debugging.
