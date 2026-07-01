---
topic: rust
status: wip
---

# error handling

Notes from the Rust Book chapter on error handling. Rust splits errors into two kinds: **recoverable** (`Result<T, E>`, expected failures the caller can respond to) and **unrecoverable** (`panic!`, bugs).

## panic and unwinding

`panic!` stops the current thread. By default Rust **unwinds**: it walks back up the stack, running destructors and cleaning up each frame's data. That's tidy but adds code to the binary.

The alternative is **abort**: stop immediately, hand cleanup to the OS, no unwinding. Smaller binary, faster exit. Opt in per profile:

```toml
[profile.release]
panic = "abort"
```

Reach for this when you want a leaner release build and don't need the graceful unwind.

## Result

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`Ok` holds the success value, `Err` holds the error. The caller has to deal with both before touching the `T`.

Handle it explicitly with `match`, or use the shorthands:

- **`unwrap()`**: give me the `Ok` value, or panic on `Err`.
- **`expect("msg")`**: same, but panics with your message. Prefer over `unwrap` so the panic says why.
- **`unwrap_or_else(|e| ...)`**: give me the `Ok`, or run a closure on the error to produce a fallback. No panic.

```rust
let f = File::open("x.txt").expect("x.txt should ship with the binary");
let f = File::open("x.txt").unwrap_or_else(|_| File::create("x.txt").unwrap());
```

## the ? operator

`?` after a `Result` unwraps the `Ok` inline, or **returns the `Err` early** from the enclosing function. Clean way to propagate: no `match`, no nesting.

```rust
fn read_name() -> Result<String, Box<dyn Error>> {
    let mut s = String::new();
    File::open("name.txt")?.read_to_string(&mut s)?;   // any ? bails out early on Err
    Ok(s)
}
```

`Box<dyn Error>` reads as "any kind of error": a catch-all return type so different error types can flow through the same `?`. `?` only works in a function that returns `Result` (or `Option`, or another type that implements `Try`).

## when to Result vs panic

Default to returning `Result`. Reserve `panic!` for truly unrecoverable situations.

- **Result** when failure is expected or the caller should decide how to respond: bad input, network errors, missing files.
- **panic! / unwrap / expect** in examples, prototypes, and tests, where handling errors would just add noise.
- **expect** is fine when you know something the compiler doesn't (a hardcoded valid value, a file guaranteed to exist). Document *why* in the message.
- **panic** when continuing would mean running in a broken or unsafe state. That's a bug for the caller to fix, not something to recover from.

## encode constraints into types

Best trick: make invalid states impossible to construct instead of validating everywhere. Wrap the constraint in a custom type whose constructor is the only way in.

```rust
pub struct Guess { value: i32 }

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess must be 1..=100, got {value}");
        }
        Guess { value }
    }
    pub fn value(&self) -> i32 { self.value }   // read-only; field stays private
}
```

Now any function taking a `Guess` gets a value that's already in range. The check lives in one place; the type carries the guarantee everywhere else.
