---

topic: elixir
status: wip
-----------

# running the system

Production Elixir is about packaging the BEAM system, running it, and observing it when something goes wrong.

## releases

An Elixir app runs inside BEAM. Starting the system means:

```text
compile code → start BEAM → start OTP apps
```

For production, use an OTP release:

```bash
mix release
```

A release packages your compiled app, dependencies, configuration, and Erlang runtime. The server does not need your source code or a separate Elixir install.

Docker, Kubernetes, systemd, etc. are just ways to run that release.

## inspect the live system

A running BEAM node is intentionally observable.

You can connect with a remote IEx shell and inspect:

```elixir
Process.list()
Process.info(pid)
```

Processes, memory, message queues, GenServer state, supervisors, and applications can all be examined while the system keeps running.

## debugging is observation

Traditional breakpoints don't fit thousands of concurrent processes very well.

BEAM systems lean toward:

```text
logs → metrics → inspection → tracing
```

Use `IO.inspect` / `dbg` during development, `Logger` in production, and tracing when you need to see messages, function calls, or state changes.

Tracing is powerful but expensive, so use it narrowly.

## summary

* `mix release` packages the production system.
* BEAM is part of the architecture, not just a runtime.
* Production nodes can be inspected while running.
* Debug concurrent systems by observing them rather than freezing them.
* Individual processes may fail; supervisors recover them while the system stays alive.
