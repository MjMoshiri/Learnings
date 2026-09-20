---
topic: elixir
status: wip
---

# basics

Elixir is Erlang with better syntax. Compiles to BEAM bytecode. Same VM that ran Ericsson phone switches.

## everything is an expression

`if`, `case`, `cond`, function bodies: they all return a value. No statements. The last expression in a function is the return.

## the VM

**BEAM** is the Erlang VM. One scheduler thread per core, preempting lightweight processes by reduction count. You don't pick which core a process runs on.

**mix** is the build tool. `iex` is the REPL. `elixir file.exs` runs a script. Those are the three ways in.

At runtime a module name is an atom that maps to a `.beam` file on disk. `String` is the atom `:"Elixir.String"`. In iex:

```elixir
AnAtom == Elixir.AnAtom
# true
```

Capitalized identifiers are aliases. They expand to `Elixir.` plus the name.

## modules and functions

Code lives in modules. `defmodule`, `def` (public), `defp` (private).

`alias Foo.Bar` lets you write `Bar` instead of `Foo.Bar`.

Default args use `\\`:

```elixir
def greet(name \\ "world"), do: "hello #{name}"
```

Functions are first-class. Capture with `&String.upcase/1`. Anonymous functions close over their environment:

```elixir
fn x -> x + y end
```

The pipe: `x |> f() |> g()` is `g(f(x))`. Left-hand side becomes the first argument.

## macros

`defmacro`. A lot of the language is macros: `if`, `def`, `defmodule`. They run at compile time and rewrite code. Don't reach for them until you have a real syntax problem.

## attributes, docs, types

Module attributes are compile-time. `@pi 3.14159` is a constant. `@moduledoc` and `@doc` feed **ExDoc**. `@spec` is a type spec. **Dialyzer** reads those specs and does success typing: it won't prove your code is right, it proves it isn't obviously wrong.

## data is immutable

Nothing mutates in place. A function returns a new version. Structural sharing, not a copy of everything.

Dynamically typed. The type is whatever the value is.

## primitives

Numbers, atoms, binaries. That's the important set.

**Atoms** are named constants. `:ok`, `:error`, `true`, `false`, `nil`. Cheap to compare. Don't generate them from user input; the atom table is not a thing you want to grow forever.

There is no Boolean type. `true` and `false` are atoms. There is no null. Use `nil`. The only falsy values are `false` and `nil`. Everything else is truthy, including `0` and `""`.

There is no string type. Use binaries (`"hello"`, UTF-8). Charlists (`~c"hello"`) exist because Erlang used lists of code points. You almost never want those.

`#{}` interpolates inside binaries: `"2 + 2 = #{2 + 2}"`.

Dates: `~D[2026-09-08]`. Calendar types on top of the primitives: `Date`, `Time`, `NaiveDateTime`, `DateTime`.

## collections

**Tuples** `{:ok, value}`. Small, fixed size, stored together. Pattern match the shape.

**Lists** are linked lists. `[head | tail]`. `hd/1` and `tl/1`. Prepend is cheap, nth-element is not. Don't use them as arrays.

**Maps** `%{key => value}`, or `%{key: value}` when keys are atoms.

**Keyword lists** are lists of `{atom, value}` tuples. Function options are usually keyword lists: `[timeout: 5000, retries: 3]`.

**Range** `1..10`. Enumerable.

**MapSet** is a set, implemented with a map.

## equality

`==` is weak. `1 == 1.0` is true. `===` is strict. `1 === 1.0` is false. Same split for `!=` and `!==`.
