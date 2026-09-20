---
topic: elixir
status: wip
---

# supervision trees

Notes from Elixir in Action ch 9. Ch 8 gave you a supervisor. Now shrink the blast radius. Start workers from supervisors, not from other workers. Recover locally. Escalate only if that fails.

## start workers from supervisors

The ch 8 tree was fully linked. Crash anywhere, the whole system restarts. Correct, no orphans. Too coarse. A database glitch killed the cache. A dead list killed every worker.

That happened because workers started other workers. Cache started the database. Database started the pool. The pids lived in worker state. Death followed the links.

Fix: put independent services next to each other under the supervisor.

```elixir
Supervisor.start_link(
  [Todo.ProcessRegistry, Todo.Database, Todo.Cache],
  strategy: :one_for_one
)
```

Kill the database. Cache keeps serving. The supervisor traps exits. `:one_for_one` restarts only the child that died.

Children start synchronously, in list order. Dependencies first. Registry before anything that registers. `init/1` still has to be fast. Slow init stalls every sibling after it. `handle_continue` for the slow part.

Don't dump every process under one supervisor. Restart intensity is shared. Too many children, one noisy process trips the whole tree. Give each service its own supervisor. Database pool fails to recover: that supervisor dies, the parent restarts the database, cache is left alone.

## you can't keep the pid

A supervised process can restart. New pid. Anything you stored is stale.

Atoms as names aren't enough. You want `{:database_worker, 1}`, `{Todo.Server, "Bob"}`. That's `Registry`. Unique keys: one process per name. Duplicate keys: pub/sub.

Register in `init`. Lookup right before the call. The registry links to registrants and drops the entry when they die. Restart, the successor registers under the same key.

**Via tuples** wire this into GenServer:

```elixir
{:via, Registry, {Todo.ProcessRegistry, {Todo.DatabaseWorker, id}}}
```

Pass it as `name:` on start, and as the target of `call`/`cast`. Clients pass an id, not a pid.

Stale pid still happens. Lookup, then crash, then send. The call fails, the client (a `Todo.Server`) dies. Recovery is concurrent. A brief inconsistency is normal. Don't hide it.

## the tree is the system

A supervision tree is how the system starts, stops, and recovers. Supervisors are service managers. Workers are the services.

Look at the tree to see what an error does. Worker dies: its supervisor restarts it. Restart doesn't fix it: intensity exceeded, that supervisor dies, the parent restarts a larger piece. Local first. Then wider. Then the node.

Stop a subtree: tell the parent to terminate that child. Descendants go with it. Stop the whole system: kill the top supervisor.

Graceful shutdown runs `terminate/2`, but only if the GenServer traps exits. Set the trap in `init`. `:shutdown` in the child spec is how long the supervisor waits (default 5s for workers, infinity for supervisors). Then `:kill`.

Restart options in the spec:

- permanent (default): always restart, even on `:normal`
- temporary: never restart. HTTP/TCP handlers. Socket is already dead.
- transient: restart only on abnormal exit. One-off jobs.

Strategies:

- `:one_for_one`: restart the one that died
- `:one_for_all`: kill everyone, start everyone. Tight coupling in all directions.
- `:rest_for_one`: kill younger siblings, restart them. Younger depends on older.

Only OTP-compliant processes go directly under a supervisor (`GenServer`, `Supervisor`, `Registry`, later `Task`/`Agent`). `spawn_link` from a worker is fine. Not from the supervisor.

## start on demand

Database workers are a fixed pool. You know the children up front. To-do servers are not. They appear when someone asks.

`DynamicSupervisor`. Start it empty. `DynamicSupervisor.start_child(sup, spec)` later. Serialized in the supervisor process. No race on "is it running?"

`Todo.Cache` becomes that supervisor. `server_process(name)` always tries `start_child`. Two outcomes count as success:

```elixir
{:ok, pid}                              # started
{:error, {:already_started, pid}}       # name taken, here's the live one
```

The second is GenServer registration failing before `init`. Same name, existing pid. You wanted that process anyway.

Every request hitting the supervisor is a bottleneck. Starting a child that immediately dies because the name is taken is wasted work. Fine for now. Distributed registry in ch 12.

To-do servers are `restart: :temporary`. Crash, don't restart. Next `server_process` starts a new one. A corrupt list can't take down the cache by burning restart intensity.

Why supervise them at all? Isolation: one list dying doesn't touch the others. Shutdown: stop `Todo.Cache`, every list dies. Supervision is not only restart.

## let it crash, for real

Worker code should state the intent. Not a pile of `try`. Unexpected errors go to the supervisor. That's intentional programming.

Two cases you handle yourself:

1. **Error kernel.** Processes the system cannot run without, state you cannot rebuild. Keep them tiny. Split: one process holds state, another does the work. The holder is too simple to crash. You can `try` in `handle_*` and roll back to the old state (immutability). Still supervise them. `:kill` cannot be trapped.

2. **Expected errors.** File missing (`:enoent`) is a miss. Return `nil`. Permission denied is not expected. Let the match fail. `File.write!` on store: if you can't persist, the worker has failed. Fail fast. Restarting may not help. Then the supervisor gives up and the system stops. Right call if you can't write.

Restart throws the mailbox away and starts with empty state. Some in-flight requests fail. The new process is clean. That's the point.

State does not survive. If you need it, persist it yourself, outside the process. Restore in `handle_continue`. Persist after the full transformation, not in the middle. That's when the state is consistent.

Careful: if the crash was caused by bad state and you persist that state, restart loads the poison and dies again. If you can afford it, start clean and take the dependents down with you.

## summary

- Workers start other workers: death is coarse. Supervisors start workers: death is local.
- Don't keep pids of supervised processes. Registry + via. Lookup late.
- The tree is start order, shutdown, and recovery policy. Separate services, separate supervisors.
- `DynamicSupervisor` for children you don't know in advance. Temporary when on-demand start is enough.
- Handle the error if you know what to do. Otherwise let it crash. Recover locally, then escalate.
