---

topic: elixir
status: wip
-----------

# working with components

An OTP application is the unit Elixir uses to package a system component: modules, processes, dependencies, startup behavior, and configuration.

A Mix project normally produces one OTP application.

## application startup

Applications with processes define an application callback:

```elixir
def start(_, _) do
  Todo.System.start_link()
end
```

Starting the application therefore starts its supervision tree.

```text
OTP application → top supervisor → system processes
```

Library applications don't need a callback or supervision tree; they simply expose reusable modules.

## dependencies

Dependencies are declared in `mix.exs`:

```elixir
{:poolboy, "~> 1.5"}
```

`mix deps.get` fetches them and `mix.lock` pins exact versions for reproducible builds.

Dependencies are themselves OTP applications and may contribute processes through `child_spec`.

The chapter replaces the custom database pool with Poolboy without changing callers:

```text
caller → Todo.Database → Poolboy → workers
```

The important point is keeping implementation behind a stable interface.

## composing the system

The supervision tree becomes the system architecture:

```elixir
[
  Todo.Metrics,
  Todo.ProcessRegistry,
  Todo.Database,
  Todo.Cache,
  Todo.Web
]
```

Plug/Cowboy adds HTTP in the same way: as another supervised component.

HTTP remains an adapter:

```text
request → decode → domain API → response
```

The core system doesn't know or care about HTTP.

Each request runs in its own process. Requests remain concurrent until they reach a shared state-owning process, which serializes access.

## calls vs casts

Casts give responsiveness but no confirmation.

Calls give confirmation but make latency depend on downstream work.

For longer operations:

```text
request → queue → accepted
                  ↓
              process later
```

Choose based on consistency and latency requirements.

## configuration

Runtime-specific values belong in `runtime.exs`, often sourced from OS environment variables:

```elixir
config :todo, http_port: ...
```

Read them with:

```elixir
Application.fetch_env!(:todo, :http_port)
```

Keep the three meanings separate:

```text
Mix env         → dev / test / prod
application env → OTP configuration
OS env          → process environment variables
```

Prefer runtime configuration for deployment-specific values rather than baking them in at compile time.

## summary

* OTP applications are the component/deployment boundary.
* Application startup means starting the supervision tree.
* Dependencies are applications and can inject supervised processes.
* Stable APIs isolate callers from implementation changes.
* HTTP is just another adapter around the domain system.
* Calls vs casts trade confirmation against latency.
* Deployment-specific configuration belongs at runtime.
