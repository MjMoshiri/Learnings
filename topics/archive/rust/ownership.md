---
topic: rust
status: wip
---

# ownership

Notes from the Rust By Practice ownership chapter. It all comes down to one question: for every piece of heap memory, exactly one variable is responsible for freeing it. That's the owner. No GC, no manual free. When the owner goes out of scope, Rust frees the memory. To keep "exactly one owner" true, Rust moves ownership instead of copying.

A String is a small stack handle (ptr, len, cap) pointing at heap bytes that can grow. Same shape for Vec, Box, HashMap: owning handle on the stack, data on the heap. That split is what the rules below are protecting.

## move

`let y = x` on a String copies the handle (ptr/len/cap) into y but NOT the heap bytes. Both would point at the same allocation, and both would try to free it at scope end. That's a double free. So Rust moves ownership to y and marks x invalid. Using x after is a compile error (E0382 borrow of moved value).

```rust
let x = String::from("Hello world");
let y = x;            // ownership moves to y, x is dead
println!("{}", x);    // error: borrow of moved value
```

Same when passing into and returning from functions. Everything is by value by default, but "by value" means the handle moves, not that the heap is copied.

```rust
fn give() -> String {
    let s = String::from("hi");
    s            // ownership moves OUT to the caller
}                // s was moved, nothing freed here
```

Passing in moves ownership in. Want it back? Return it. That awkwardness is why borrowing exists.

## copy vs clone

Copy: implicit, on `let y = x`, cheap bitwise copy, both stay valid. Only for types with nothing on the heap.

Clone: explicit `.clone()`, can be expensive (allocates and deep-copies), both stay valid.

Shortcut: does dropping this value free anything on the heap?
- No (i32, bool, char, &str, unit, fixed arrays of Copy types) -> Copy
- Yes (String, Vec, Box, HashMap) -> not Copy, it moves, need .clone() for a real second copy

A tuple is Copy only if every element is Copy. `(1, 2, (), String::from("x"))` isn't, because of the String. Swap it for a &str literal and the whole tuple becomes Copy.

## borrowing

Reference with `&`: look at someone's data without taking ownership. Caller keeps the value.

Rule: many readers (`&`) XOR one writer (`&mut`). Never both at once. That's the borrow checker.

- `&String` (shared borrow): read only
- `&mut String` (exclusive borrow): read AND write. You're the only one with access, so you can read it, branch on it, then mutate it.

```rust
fn append(s: &mut String) {
    if s.is_empty() { s.push_str("x"); }   // read, decide
    s.push_str("!");                        // then write
}
```

To pass something to be mutated, three things line up:
1. the variable is `let mut s`
2. call site passes `&mut s`
3. parameter is `s: &mut String`

The `&mut` at the call site is deliberate. In Rust you can always see at the call where a value might be mutated. Python/Java let a function silently mutate an object you passed. Rust doesn't.

While a `&mut` is alive it's the only path to the data. No second `&mut`, no `&` at the same time.

## nll (non-lexical lifetimes)

A borrow doesn't live until the end of its block. It lives until its last use. That's NLL (non-lexical lifetimes).

So two `&mut` to the same value are fine as long as their uses don't overlap in time.

```rust
let r1 = &mut s;
r1.push_str("world");   // r1's last use, r1 is done here
let r2 = &mut s;        // fine, r1 already dead
r2.push_str("!");
```

Add a later use of r1 and it breaks. That stretches r1's life past where r2 is created, so now two `&mut` overlap and the borrow checker rejects it (cannot borrow s as mutable more than once).

```rust
println!("{}", r1);   // revives r1, now r1 and r2 overlap -> error
```

Same rule explains why a shared borrow followed by a mutation is fine: the `&` is done being used before the `&mut` begins.

## mut and move are independent

`let mut s1 = s;` still MOVES out of s (s is dead after). The mut only describes the new binding s1. Mutability is a property of the binding, not the value, and it can change when ownership moves. The original s never needed to be mut.

To mutate through a borrow, the OWNER must be mut (can't lend write access you don't have). In the move case, only the new owner needs mut and the old binding is gone.

## box

`Box::new(5)` puts a value on the heap and gives back a stack pointer that owns it. The minimal String pattern: tiny stack handle, one owned heap value. Use `*` to reach the value inside.

Box is not Copy and does move. A separate Box is separate data:

```rust
let x = Box::new(5);
let mut y = Box::new(5);   // separate heap allocation
*y = 4;                    // changes only y
assert_eq!(*x, 5);         // x untouched
```

Why Box matters later: big data without copying it on the stack, recursive types, trait objects (Box<dyn Trait>).

## partial move

Moving one field out of a compound value leaves the rest usable but kills the whole.

```rust
let t = (String::from("hello"), String::from("world"));
let _s = t.0;            // moves t.0 out
println!("{:?}", t);     // error: t partially moved
println!("{:?}", t.1);   // fine, t.1 was never moved
```

To avoid moving fields out, borrow during destructuring with `ref` or `&`:

```rust
let (ref s1, ref s2) = t;   // s1, s2 are &String, t still owns everything
// or, idiomatic:
let (s1, s2) = &t;
```

A plain binding moves. `ref` binds a reference instead. Destructuring `&t` gives references to each field automatically.

## &str vs String

Two types that both look like "a string".

`"Hello world"` is `&'static str`: a reference into read-only bytes baked into the binary, lives the whole program. You don't own it, can't grow or change it. Just a pointer + length. Copy, because copying it is just duplicating a pointer.

`String::from("Hello world")` allocates fresh heap memory, copies the bytes in, gives you an owned String. Growable, mutable, freed when the owner drops.

Use &str for reading text (most params should be &str). Use String when you need to own or build up text. Literal -> owned with String::from(...) or "...".to_string(). Owned -> borrowed for free with &s.

## char vs String

A `char` is a single Unicode scalar value, 4 bytes, single quotes: `'a'`. Copy, fixed size. A String is a growable UTF-8 buffer, double quotes. `'a'` and `"a"` are different types. Iterating a string with `.chars()` yields char values.

## the unit type ()

`()` is the unit type and its only value. Default return type: a function with no `-> T` returns `()`. Carries no data, takes no space, is Copy. Showed up as a Copy element in `(1, 2, (), "hello")`, and it's why a function returning nothing returns `()`, which breaks a `let s = f(...)` that expected a real value back.

## the one recurring question

Did ownership move, and if so, what's still valid? Keep the original usable with one of three:
- clone (copy the heap data)
- borrow with & or ref (don't transfer)
- use a Copy type (nothing on the heap to transfer)
