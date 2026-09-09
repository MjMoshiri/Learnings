---
topic: rust
status: wip
---

# testing

Notes from the Rust Book chapter on writing tests. A test fails when something inside it panics. `assert!`, `assert_eq!`, `assert_ne!` do most of the work.

## unit tests and #[cfg(test)]

Unit tests sit in the same file as the code, inside a module tagged `#[cfg(test)]`. Each test function gets `#[test]`. `#[cfg(test)]` tells the compiler to only build that module for `cargo test`. It's absent from a normal `cargo build`, so it adds nothing to the release binary.

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_adds() {
        assert_eq!(add(2, 2), 4);
    }
}
```

Unit tests can test private functions. The `tests` module is a child module, so `use super::*` reaches private items. That's the real contrast with integration tests, which only see the public API, and the reason unit tests live in the same file as the code.

## should_panic

Test that code panics when it should:

```rust
#[test]
#[should_panic(expected = "must be 1..=100")]
fn rejects_out_of_range() {
    Guess::new(200);
}
```

The `expected` substring guards against passing on the *wrong* panic. The test only passes if the panic message contains it.

## custom failure messages

Extra args to `assert!` become the panic message:

```rust
assert!(result.contains("Carol"), "greeting was {result}");
```

They're format args, shown when the test fails. Makes a red test explain itself instead of just saying the assertion failed.

## tests that return Result

A test can be `fn works() -> Result<(), String>` instead of panicking. Returning `Err` fails it. The payoff is using `?` inside the test body.

```rust
#[test]
fn it_works() -> Result<(), String> {
    if add(2, 2) == 4 { Ok(()) } else { Err(String::from("2+2 != 4")) }
}
```

Can't combine this with `#[should_panic]`. To assert something errs, use `assert!(value.is_err())`.

## doc tests

Code blocks in `///` doc comments run as tests too. `cargo test` runs them alongside unit and integration tests, which is why the output has three sections. `cargo test --doc` runs just these.

```rust
/// Adds two numbers.
///
/// ```
/// assert_eq!(my_crate::add(2, 2), 4);
/// ```
pub fn add(a: i32, b: i32) -> i32 { a + b }
```

Keeps documentation examples honest. A stale example fails the build.

## ignore

`#[ignore]` skips a test on normal `cargo test` runs. Good for slow or expensive ones. Run them with `cargo test -- --ignored`, or everything with `--include-ignored`.

## controlling the test runner

Everything after `--` goes to the test binary, not cargo:

- `cargo test -- --test-threads=1`: tests run in parallel on separate threads by default. Drop to 1 thread when they share state (files, env vars) and step on each other.
- `cargo test -- --show-output`: output from *passing* tests (`println!` etc.) is captured and swallowed by default. This flag shows it.
- `cargo test <name>`: run only tests whose name contains `<name>` (substring filter).

## integration tests

Integration tests live in a top-level `tests/` directory, next to `src/`:

```
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    ├── common
    │   └── mod.rs
    └── integration_test.rs
```

Each file in `tests/` compiles as its own crate. It can only touch the library's public API (`use my_crate::...`), exactly like an external consumer would. No `#[cfg(test)]` needed here; cargo only builds `tests/` for `cargo test`.

Run a single integration test file with `cargo test --test integration_test`.

## shared test helpers: common/mod.rs

Helper code shared across integration test files goes in `tests/common/mod.rs`, not `tests/common.rs`. A plain `tests/common.rs` counts as its own integration test crate and shows up in the output with 0 tests. The older `mod.rs` naming tells cargo it's a module for other tests to use, not a test file itself. Each integration test file pulls it in with `mod common;` and calls `common::setup()`.

This only works for library crates. A binary crate's `src/main.rs` functions can't be imported by `tests/` files, which is one reason the common pattern is a thin `main.rs` over a `lib.rs`.
