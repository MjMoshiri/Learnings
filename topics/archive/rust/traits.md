---
topic: rust
status: wip
---

# traits

Notes from the Rust Book chapter on generics and traits. A trait defines shared behavior: a set of method signatures a type promises to implement. Close to interfaces in other languages, with some extra tricks.

## monomorphization

Generics cost nothing at runtime. At compile time Rust **monomorphizes**: for every concrete type a generic is used with, the compiler stamps out a specialized copy of the code. `Vec<i32>` and `Vec<String>` become two separate compiled versions. You write one function, the binary contains one per type, and each call is as fast as if you'd written it by hand. The price is compile time and binary size, not speed.

## defining and implementing

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}

impl Summary for Article {
    fn summarize(&self) -> String {
        format!("{}, by {}", self.title, self.author)
    }
}
```

## traits as parameters

Two syntaxes for "accept anything that implements this trait":

```rust
// impl Trait: short, reads well for simple cases
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}

// trait bound: the full form, needed when things get specific
pub fn notify<T: Summary>(item: &T) {
```

`impl Trait` is sugar for the trait bound. They diverge when you take two parameters: `(a: &impl Summary, b: &impl Summary)` lets the two be *different* concrete types, while `<T: Summary>(a: &T, b: &T)` forces both to be the *same* type.

## multiple bounds and where

Require more than one trait with `+`:

```rust
pub fn notify<T: Summary + Display>(item: &T) {
```

When the bounds pile up, inline syntax gets unreadable. Move them into a `where` clause:

```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
```

Same meaning, but the signature stays legible: types up front, constraints listed below.

## blanket implementations

You can implement a trait for *every type that satisfies a bound*:

```rust
impl<T: Display> ToString for T {
    // any type that implements Display gets to_string() for free
}
```

That's why `3.to_string()` works: the standard library has this exact blanket impl. Integers implement `Display`, so they get `ToString` without anyone writing `impl ToString for i32`. The standard library uses blanket implementations heavily; when a method shows up on your type "out of nowhere", a blanket impl is usually the reason.
