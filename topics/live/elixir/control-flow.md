---
topic: elixir
status: wip
---

# control flow

Branching in Elixir is matching a value against a shape, then running the first clause that fits. Function arguments are patterns. Recursion is the loop.

Docs: [patterns and guards](https://elixir.hexdocs.pm/patterns-and-guards.html)

## pattern matching

`=` is a match, not assignment. Left side is a pattern, right side is a term. Variables in the pattern bind to the matching subterms. No fit: `MatchError`.

```elixir
{a, b} = {1, 2}
# a is 1, b is 2
```

`_` is the anonymous variable. Matches anything, never binds, cannot be read.

```elixir
{_, {hour, _, _}} = date_time = :calendar.local_time()
# hour is bound, the rest is ignored
# date_time holds the whole value
```

The same name twice in one pattern means those positions must be equal:

```elixir
{amount, amount, amount} = {127, 127, 127}  # ok
{amount, amount, amount} = {127, 127, 1}    # MatchError
```

That's not the pin. Pin (`^x`) is for a variable already bound outside the pattern. Same name in one pattern is a same-value constraint.

Numbers in patterns compare strictly: `1` does not match `1.0`. Tuples match on size. Lists: `[head | tail]`. Empty list is a separate clause. Maps match a subset: `%{name: n} = %{name: "meg", age: 23}` works. `%{}` matches every map.

## binaries and `<>`

Binaries split by bit width:

```elixir
<<a::4, b::4>> = <<155>>
# a is 9, b is 11
# 155 is 10011011: first nibble 1001, second 1011
```

`<>` concatenates binaries. In a pattern it only works as a prefix match:

```elixir
"hello " <> rest = "hello world"
# rest is "world"
```

Suffix match (`rest <> " world"`) is not a valid pattern.

## guards

Patterns check shape. Guards check extra conditions. `when` after a clause. Restricted set: comparisons, `and`/`or`/`not`, arithmetic, `is_*` type checks, a handful of data functions. No side effects. The VM runs them cheaply.

A guard must return `true`. Truthy is not enough. `when head` fails if `head` is `"x"`. Write `when head != nil`.

If a guard would raise, the clause doesn't match.

```elixir
def empty_map?(map) when map_size(map) == 0, do: true
def empty_map?(map) when is_map(map), do: false
```

`%{}` matches any map, so emptiness needs a guard.

`defguard` / `defguardp` for reusable ones.

## branching

Multiclause functions first. That's the default way to branch. First clause that matches all arguments runs.

```elixir
defmodule ListHelper do
  def sum([]), do: 0
  def sum([head | tail]), do: head + sum(tail)
end
```

Then the expressions:

- `if` / `unless`: boolean, two-way. `unless` is `if not`.
- `cond`: first truthy condition. Use when you're testing expressions, not matching a value.
- `case`: match one value against patterns (and guards).
- `with`: chain matches. If any left-hand pattern fails, jump to `else`, or return the failed value.

```elixir
case File.read(path) do
  {:ok, body} -> body
  {:error, reason} -> reason
end

with {:ok, a} <- fetch_a(),
     {:ok, b} <- fetch_b(a) do
  {:ok, a + b}
else
  {:error, _} = err -> err
end
```

## recursion and tail calls

No `for` loop. Recursion is the loop.

BEAM does tail-call optimization. If the last thing a function does is a call, that's a jump, not a stack push. Long loops don't grow the stack.

The `sum` above is not tail recursive. `head + sum(tail)` has to come back to do the add.

Tail version keeps the running total in an argument:

```elixir
def sum(list), do: sum(list, 0)
defp sum([], acc), do: acc
defp sum([head | tail], acc), do: sum(tail, acc + head)
```

The recursive call is the last expression. Use this when the list can be long.

## Enum, Stream, Collectable

Most loops you write are higher-order functions.

**Enum** is eager. `Enum.map` walks the whole thing and returns a list.

**Stream** is a lazy enumerable. Each step is a recipe. Nothing runs until you pull, usually by dumping into Enum or a Collectable. Compose `Stream.map |> Stream.filter` without building intermediate lists.

**Collectable** is the other side of Enumerable: something you can pour values into. `Enum.into`, the last stage of a comprehension.

Comprehensions (`for`) iterate, filter, transform, and can join enumerables. They collect into a list by default, or into anything Collectable via `into:`.

```elixir
for x <- 1..5, rem(x, 2) == 1, do: x * x
# [1, 9, 25]
```
