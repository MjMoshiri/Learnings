---
topic: unity
status: wip
---

# csharp fundamentals

C-family syntax. Curly braces, semicolons, `if/else/switch`, `for/while`. Familiar if you know C/Java/JS.

- OO: classes, interfaces, polymorphism, encapsulation
- Strongly typed, compiler-enforced
- Cross-platform via .NET runtime
- GC'd, but value types exist via `struct`
- Exceptions: `try/catch/finally`. No checked exceptions, any method can throw anything
- Package manager: NuGet

## C#-specific stuff worth knowing early

- `var` = type inference. `const` = compile-time constant
- String interpolation: `$"hi {name}"`
- LINQ: `.Where(x => x > 5).ToList()` to query/transform collections
- Null: `??` for default, `?` suffix marks nullable
- Pattern matching: `is` checks, switch expressions on shape
- `async`/`await` for async ops
- `using` frees non-memory resources (files, sockets)
- Properties and indexers are language features, not a get/set naming convention
- Records: class or struct, immutability optional
- Extension methods: add methods to existing types
- Local functions: nested functions inside methods

## more syntax

- `IEnumerable` = base interface for anything iterable
- Index from end: `string last = names[^1];` // `^1` = last element
- Range slice: `int[] smallNumbers = numbers[0..5];` // 0 to 4
- LINQ query syntax (SQL-like, alt to method syntax):
  ```csharp
  var honorRoll = from student in Students
                  where student.GPA > 3.5
                  select student;
  ```
- `await foreach` = async iteration over `IAsyncEnumerable`
- C# 12+: `using` aliases work on any type, incl. tuples, pointers
- Explicit interface implementation: two interfaces with the same method name, disambiguate at the call site by casting to the interface

## functional flavors

- LINQ chains, lambdas, `Func<>`/`Action<>` delegates
- Records for immutable data
- Pattern matching, switch expressions
- Expression-bodied members (`=> expr`)
- Higher-order via delegates, no first-class functions

## naming conventions

- `PascalCase`: types (class/struct/interface/enum/delegate/record), namespaces, methods, properties, events, public fields, constants, local functions
- `camelCase`: locals, method params, private/internal fields with `_` prefix → `_workerQueue`
- Interfaces prefixed `I` → `IWorkerQueue`
- Attribute types end in `Attribute`
- Enums: singular noun, plural if flags
- Static private fields: `s_`, thread-static: `t_`
- Generic type params: `T`, or `TFoo` if descriptive
- Skip abbreviations unless widely known
- No double underscores (compiler-reserved)

## coding conventions

- Allman braces (open brace on new line, aligned)
- 4-space indent, no tabs
- File-scoped namespace: `namespace Foo;`
- `using` directives **outside** the namespace block
- Language keywords over runtime types: `string` not `System.String`, `int` not `Int32`. Prefer `int` over unsigned
- `var` only when the type is obvious from RHS (literal, `new`, cast). Use `var` for LINQ results
- String interpolation for short concat. `StringBuilder` in loops. Raw string literals `"""..."""` for multiline
- Collection expressions: `string[] vowels = ["a","e","i","o","u"];`
- `using` statement / declaration for `IDisposable`. New syntax: `using Font f = new(...);` (no braces)
- `&&`/`||` over `&`/`|` for short-circuit
- `Func<>`/`Action<>` over custom delegate types
- Concise `new`: `ExampleClass x = new();` when the type is known
- Object initializers: `new Foo { A = 1, B = 2 }`
- `required` properties instead of constructors when forcing init
- LINQ: meaningful query var names, `where` before other clauses, multiple `from` over `join` for inner collections
- Catch specific exceptions, never bare `Exception`
- Call static members via `ClassName.Member`

