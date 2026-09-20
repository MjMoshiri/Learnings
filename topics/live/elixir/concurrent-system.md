---
topic: elixir
status: wip
---

# concurrent system

One process per list. A cache that finds or starts them. Persistence as another process. The work is deciding what is a process.

## mix, briefly

`mix new todo`. Code under `lib/`. One module per file. Nested names map to folders: `Todo.Server` lives in `lib/todo/server.ex`. `iex -S mix` compiles and drops you in a shell with the project loaded.

That's enough mix. The rest is process design.

## one process per list

You already have `Todo.List` (pure data) and `Todo.Server` (one list, long-lived). Two ways to handle many lists:

1. One process, a map of lists inside it.
2. One server process per list.

The first serializes every user through one mailbox. The second lets lists run at the same time. That's the default Elixir move: if work can be independent, give it a process.

You still need something that creates those servers and finds them later. That's `Todo.Cache`. There's one. State is a map from list name to pid. `server_process(cache, name)` is a call: lookup, or start and insert.

```elixir
def handle_call({:server_process, name}, _, servers) do
  case Map.fetch(servers, name) do
    {:ok, pid} ->
      {:reply, pid, servers}

    :error ->
      {:ok, pid} = Todo.Server.start()
      {:reply, pid, Map.put(servers, name, pid)}
  end
end
```

Same name, same pid. Different name, different pid. A hundred thousand of these is a normal number. Processes are cheap.

## how to think about the graph

Draw the processes. Each box is sequential. Each arrow is a message.

- Many clients hit one cache. That's a bottleneck if `server_process` is slow. A map lookup plus optional spawn is microseconds, so it stays cheap. At 100ms it wouldn't.
- Once a client has a list pid, work on that list is concurrent with everything else. That's the scalable part.
- One list is still one process. A million clients editing Bob's list still go through Bob's mailbox, one at a time. No races on that list. That's why mutation lives in a process.

A process waiting on a message uses no CPU. Idle servers are free.

Need a critical section? Put it in a process. Callers send a request. Serialization is the mailbox.

## persistence

Another process: `Todo.Database`. `store` is a cast. `get` is a call. Data is `:erlang.term_to_binary` into a file named after the key. Ugly storage. Where the process sits matters more.

Register it under the module name so you don't pass the pid around. One instance.

Store from `Todo.Server` after each change. The server needs the list name in its state for that.

Don't load from disk in `init/1`. `GenServer.start` doesn't return until `init` finishes. A slow init blocks the cache, and everyone talks to the cache. Split it:

```elixir
def init(name) do
  {:ok, {name, nil}, {:continue, :init}}
end

def handle_continue(:init, {name, nil}) do
  list = Todo.Database.get(name) || Todo.List.new()
  {:noreply, {name, list}}
end
```

`init` returns fast. The caller is unblocked. `handle_continue` runs next, in the server, and can take its time.

## call vs cast, again

Cast: caller moves on. You don't know if the write happened. Availability up, consistency down.

Call: caller waits. You know. If the server is slow, the caller is stuck. That's also back pressure: the client can't produce work faster than the server can take it.

Casts pile up in the mailbox. Mailbox growth is memory growth. Enough of that and the VM dies.

A timed-out call still leaves the message in the server's mailbox. You gave up waiting. The work still happens later.

If you're not sure, start with a call.

## do you even need a process?

Need a process if:

- you manage long-lived state
- you reuse a resource (connection, file handle)
- you need a critical section

If none of those, it's a module. Run it in the caller. No bottleneck.

You could write files from `Todo.Server` itself. Same-list writes are already serialized. The catch is unbounded I/O: 100k clients, 100k disk ops.

Spawning a worker per request from the database process has the same problem. The mailbox stays short. Disk concurrency is still unbounded.

Pool: a few workers, requests hashed onto them. Same key, same worker (`:erlang.phash2(key, 3)`). Bounded concurrency, per-key serialization. That's the usual shape for a database.

## processes are services

Each process does one job.

- `Todo.Server`: one list
- `Todo.Cache`: name → pid
- `Todo.Database`: persist, internally a small pool, same key to the same worker

Independent until they need each other. Then call or cast.

## summary

- Independent work gets its own process. Shared mutable state gets one process that owns it.
- A process is sequential. That's the bottleneck and the lock.
- Draw the graph. The boxes with many arrows in are the ones that have to be cheap.
- Cast for responsiveness. Call for a guarantee, and for back pressure.
- Don't block a hot process in `init`. Continue after start returns.
