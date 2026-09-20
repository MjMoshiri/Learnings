---
topic: elixir
status: wip
---

# data abstractions

Higher-level types (`Fraction`, `User`, `Money`) are maps plus a module. The module's functions create, change, and query the data. Clients can still see the whole structure.

## modules as abstractions

Data is separate from code. Call `Fraction.new/2`, then pass that value back into `Fraction.add/2`. The client shouldn't care that a fraction is two integers in a map.

```elixir
defmodule Fraction do
  def new(a, b), do: %{a: a, b: b}

  def add(f1, f2) do
    new(
      f1.a * f2.b + f2.a * f1.b,
      f1.b * f2.b
    )
  end

  def value(f), do: f.a / f.b
end
```

`__MODULE__` is the current module as an atom. Use it when you construct a struct so you don't hardcode the name:

```elixir
def new(a, b), do: %__MODULE__{a: a, b: b}
```

The catch: the structure is always visible. A client can read `f.a` even if you never meant them to. Elixir has no private fields. Convention is the only wall. Don't depend on the shape from outside the module.

## maps, structs, records

**Maps** group fields. Open. Any key. Nested data, one-off shapes.

**Structs** are named maps. `defstruct` in a module. The struct's name is the module. Underneath it's a map with a `__struct__` key.

```elixir
defmodule Fraction do
  defstruct a: nil, b: nil

  def new(a, b), do: %__MODULE__{a: a, b: b}
end

%Fraction{a: 1, b: 2}
# same as %{__struct__: Fraction, a: 1, b: 2}
```

Defaults come from `defstruct`. Missing keys are `nil` unless you set them. `@enforce_keys [:a, :b]` fails construction if those are missing. Only checked at build time, not on updates. Doesn't validate values.

```elixir
%Fraction{a: 1}            # b is nil, unless enforced
%{f | a: 3}                # update, compile-time field check
%Fraction{oops: 1}         # KeyError, unknown field
```

**Struct vs map:**

- `is_map(%Fraction{})` is true. `is_struct/1` / `is_struct(f, Fraction)` if you need the tag.
- `%Fraction{}` in a pattern only matches Fraction. `%{}` matches any map, including structs.
- You can't add keys. Field list is fixed at compile time.
- Structs don't pick up map protocols. No `Enum.each` on a struct. No `struct[:a]` unless you impl Access.
- Dot access works: `f.a`. That's the intended way in.
- `Map.from_struct/1` drops `__struct__` and gives a plain map.

Use a struct when the shape belongs to a module. Use a map when the keys aren't known in advance, or when you want Enumerable/Access for free.

**Records** are Erlang tuples tagged in the first element. `Record.defrecord`. Elixir used records; structs replaced them. Reach for records only when talking to Erlang code that still uses them.

```elixir
require Record
Record.defrecord(:user, name: "john", age: 27)
# {:user, "john", 27}
```

## inspect

`Kernel.inspect/2` returns a string. `IO.inspect/2` prints that string and returns the original value, so it slots into a pipe.

Both go through the `Inspect` protocol. IEx uses it too. Default inspect dumps every field.

That's the visibility problem again. Implement Inspect if you don't want clients staring at the internals:

```elixir
defimpl Inspect, for: Fraction do
  def inspect(%Fraction{a: a, b: b}, _opts) do
    "#Fraction<#{a}/#{b}>"
  end
end
```

Or derive it and pick fields:

```elixir
@derive {Inspect, only: [:id, :name]}
defstruct [:id, :name, :password]
```

Hidden fields print as `#User<...>`. The `#` prefix means "this is not valid Elixir, don't paste it back". That's also how functions and pids print.

`inspect` is for debugging. `to_string` / interpolation go through `String.Chars`, a different protocol. Tuples have Inspect. They don't have String.Chars, so `"#{ {1, 2} }"` raises. Use `"#{inspect(tuple)}"`.

## put_in and Access

Nested data. You don't walk it by hand.

```elixir
users = %{"john" => %{age: 27}, "meg" => %{age: 23}}

put_in(users["john"].age, 28)
update_in(users["john"].age, &(&1 + 1))
get_in(users["john"].age)
pop_in(users["john"].age)
```

Macro form takes a path expression. Function form takes a list of keys. Those keys can be Access accessors:

```elixir
get_in(user, [:languages, Access.all(), :name])
# ["elixir", "c"]

update_in(user, [:languages, Access.all(), :name], &String.upcase/1)
```

Accessors:

- `Access.all()` every list element
- `Access.at(i)` / `Access.at!(i)` one index
- `Access.key(k)` / `Access.key!(k)` map or struct field (`key` has a default, `key!` raises)
- `Access.elem(i)` tuple index
- `Access.filter/1`, `Access.find/1`, `Access.slice/1`

Access is a **behaviour**, not a protocol. Maps and keyword lists implement it. Structs don't. `map[:k]` returns nil on miss. `struct[:k]` raises `UndefinedFunctionError` (`User.fetch/2` is missing). Use `struct.k`.

`nil[:anything]` is `nil`. Nested `[]` won't raise on a hole. `map.key` will.

## protocols

Polymorphism: the function is generic, the type picks the impl at runtime.

```elixir
defprotocol Size do
  def size(data)
end

defimpl Size, for: BitString do
  def size(binary), do: byte_size(binary)
end

defimpl Size, for: Map do
  def size(map), do: map_size(map)
end

defimpl Size, for: Tuple do
  def size(tuple), do: tuple_size(tuple)
end
```

`Size.size("hello")` dispatches on the type of the argument. That's why structs exist: the `__struct__` tag is what protocols look at. A bare map and a `%User{}` are different types to a protocol, even though both are maps.

`defimpl` inside the struct's module can omit `:for`.

```elixir
defmodule User do
  defstruct [:name]

  defimpl Size do
    def size(_user), do: 1
  end
end
```

Fallback: `@fallback_to_any true` on the protocol, then `defimpl Size, for: Any`. Structs can `@derive [Size]` to opt into that Any impl without writing one.

A type with no impl raises `Protocol.UndefinedError`.

## built-in protocols

- **Enumerable**: `Enum`, `Stream`. Lists, maps, ranges, MapSet. Not structs, unless you impl it.
- **Collectable**: the other direction. `Enum.into`, `for` with `into:`.
- **Inspect**: `Kernel.inspect`, `IO.inspect`, IEx.
- **String.Chars**: `to_string`, interpolation, `IO.puts`.
- **List.Chars**: `to_charlist`.
- **JSON.Encoder**: JSON encoding (1.18+). Derive it, prefer `only:`.
- **IEx.Info**: what `i/1` prints in iex.

Access is not in this list.

## summary

A module is the abstraction. Its functions create, manipulate, and query data. Clients can see the whole structure. They shouldn't rely on its shape.

Maps group fields. Structs are named maps tied to a module.

Polymorphism is protocols. The protocol is the interface the generic code calls. `defimpl` is the per-type body.
