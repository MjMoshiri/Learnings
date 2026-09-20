---
topic: elixir
status: done
---

# fault tolerance

Anything can fail. Isolation first, then detection, then restart. Concurrency is the error-handling tool.

## the goal

Don't try to prevent every error. Contain it. Keep serving something. Recover without a human.

One isolated crash is better than a crash that takes the node down. Bugs that survive tests are often Heisenbugs: rare, state-dependent, hard to reproduce. The state may be corrupt. Kill the process, start another with a clean state. Log it. Keep going. That's let it crash.

## errors, exits, throws

Three runtime error types.

- **error**: bad match, bad arithmetic, missing function, `raise/1`. Functions that raise append `!` (`File.open!` vs `File.open`).
- **exit**: `exit(reason)`. Deliberate process death. Reason is any term.
- **throw**: `throw(value)`. Nonlocal return. Avoid it. There's no break/return. This is the hacky substitute.

Unhandled: the process dies. Other processes keep running.

`try/catch` can intercept. `catch type, value` is a pattern. `after` always runs (cleanup). Code in `try` is not tail-call optimized.

You rarely recover with `try`. Let the process die. Someone else restarts it.

## isolation is the default

No shared memory. A crash in one process doesn't corrupt another. Spawn a child, raise in the parent: the child still finishes.

The todo system already has this. One `Todo.Server` dying doesn't take the others with it. A dead database doesn't stop in-memory reads.

Isolation isn't enough. Clients still need the process that died. You have to notice and bring it back.

## links

`Process.link/1` or `spawn_link/1`. Two processes, bidirectional. If one dies abnormally, the other gets an exit signal and dies too, unless the reason is `:normal`.

Links transit. Crash one, take the tree.

A link is how death travels. Opposite of isolation.

**Trap exits** if you want the signal as a message instead of death:

```elixir
Process.flag(:trap_exit, true)
# then receive {:EXIT, from_pid, reason}
```

Error/throw reasons look like `{reason, stacktrace}`. `exit/1` reasons are the term you passed.

## monitors

Unidirectional. `Process.monitor(pid)` returns a ref. If the target dies, you get `{:DOWN, ref, :process, pid, reason}`. You don't die. `Process.demonitor(ref)` to stop.

`GenServer.call` uses a temporary monitor. If the server dies while you're waiting, the caller exits too. That's why a dead server takes its callers with it.

## supervisors

A supervisor starts children, watches them, restarts them. Workers do the actual work.

`Supervisor.start_link(children, strategy: :one_for_one)`. `:one_for_one` restarts the one that died.

Restart means a new process. New pid. Old state is gone.

That's why you register names. Pids go stale after a restart. A name doesn't.

```elixir
def start_link(_) do
  GenServer.start_link(__MODULE__, nil, name: __MODULE__)
end
```

`start_link`, not `start`. Supervisors need a link. Bidirectional: supervisor dies, children die. No orphans.

Child spec: how to start, what to do on death, an id. `use GenServer` injects `child_spec/1`. That's why `start_link/1` takes an ignored arg: the default spec calls `start_link(arg)`. Pass `[Todo.Cache]` and the supervisor calls `Todo.Cache.child_spec([])`, then `Todo.Cache.start_link/1`.

```elixir
defmodule Todo.System do
  def start_link do
    Supervisor.start_link([Todo.Cache], strategy: :one_for_one)
  end
end
```

`Todo.System.start_link()` is the whole system.

Callback-module form (`use Supervisor`, `init/1`): same result, more control. Use the list form unless you need the extra hook.

## link the workers too

Supervising only the cache isn't enough. Kill the cache, it restarts empty, and the old `Todo.Server` processes are still running, unreachable. Garbage.

Link the whole structure: cache ↔ servers, cache ↔ database, database ↔ workers. Crash anywhere, the tree dies, the supervisor starts a clean one. Wide blast radius.

Switch every `GenServer.start` to `start_link`.

## restart intensity

Default: 3 restarts in 5 seconds. Exceed it, the supervisor gives up and dies, taking children with it.

That's intentional. If restarting doesn't fix it, looping forever isn't a strategy. In a tree, the parent supervisor then restarts a larger piece.

`:kill` as an exit reason bypasses trapping. Unconditional death. Use it when you want to be sure.

## summary

- Failures happen. Isolate them. Restart. Keep serving.
- Let it crash: throw away maybe-corrupt state, start clean.
- Links propagate death. Trap exits to observe. Monitors watch without dying.
- A supervisor starts, watches, restarts. Names survive the new pid. `start_link` ties the tree together.
- Too many restarts in a window: the supervisor dies. That's how the failure moves up.
