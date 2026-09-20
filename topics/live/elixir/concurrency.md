---
topic: elixir
status: wip
---

# concurrency

A process is a lightweight concurrent unit. Isolated, no shared memory. Messages are how they talk. A server is a process that loops forever and keeps state in the loop's arguments.

## processes and the scheduler

A BEAM process starts small, a couple of KB. Own heap, own mailbox, own garbage collection. Crash one, the others keep going. You can run a lot of them.

The VM runs one scheduler thread per core. Each scheduler picks runnable processes. Preemption is by reduction count: function calls (and some BIFs) count as reductions. Hit the quota, you're swapped out. A busy loop doesn't pin a core and starve everyone else. You don't pick which core a process runs on.

## spawn and self()

`spawn/1` takes a zero-arg function. Returns a pid. The caller continues immediately.

```elixir
pid = spawn(fn -> IO.puts("in the other process") end)
```

`spawn/3` is `spawn(Module, :function, [args])`. Same thing, useful when you don't want a closure.

`self()` is the current process's pid. The iex shell is a process. That's why `self()` in iex is a pid you can send to.

A pid is a term. Put it in a tuple, a map, a list. Send it around so others can reply.

## send

```elixir
send(pid, {:an, :arbitrary, :term})
```

Any term. Fire and forget. Returns the message, not a confirmation the other side got it. If the process is dead, the message is dropped.

Erlang form: `pid ! message`. Same thing.

The message lands in the target's mailbox. Mailbox is a queue. Nothing is processed until the process `receive`s.

## receive and after

```elixir
receive do
  {:an, :arbitrary, :term} -> :matched
  other -> other
end
```

Selective. Walks the mailbox, takes the first message that matches a clause. Non-matching messages stay. That's useful, and also how you leak a mailbox: keep receiving only one shape, everything else piles up.

No matching message: the process waits.

```elixir
receive do
  {:an, :arbitrary, :term} -> :ok
after
  5_000 -> :timeout
end
```

`after` is milliseconds. `after 0` checks the mailbox and moves on if nothing matches. No `after` means wait forever.

## request-response is a pattern

Send is async. If you want a reply, send your pid, then receive:

```elixir
send(server, {:get, self()})
receive do
  {:response, value} -> value
end
```

Synchronous calls are this plus a timeout. Later OTP wraps it. The VM does not.

## stateful server

A server process lives a long time (or forever) and handles messages. Endless recursion. State is the argument.

```elixir
defmodule Counter do
  def start do
    spawn(fn -> loop(0) end)
  end

  def value(pid) do
    send(pid, {:value, self()})
    receive do
      {:response, n} -> n
    end
  end

  def inc(pid, n \\ 1), do: send(pid, {:inc, n})

  defp loop(count) do
    new_count =
      receive do
        {:value, caller} ->
          send(caller, {:response, count})
          count

        {:inc, n} ->
          count + n
      end

    loop(new_count)
  end
end
```

`start/0` returns a pid. Clients talk to that pid. `loop/1` never returns. Tail recursive, so the stack doesn't grow. The next state is whatever the receive clause returns.

The state is private. Other processes can't read it. They send a message and wait for a reply. Isolation: no shared memory, so no locks.

## register

Pids get annoying to pass around. Register a local name:

```elixir
pid = Counter.start()
Process.register(pid, :counter)

send(:counter, {:inc, 1})
Counter.value(:counter)
```

`send/2` accepts a registered name. Names are atoms, unique on the node. A process can have one. Die and the name is freed.

`Process.whereis(:counter)` gives the pid or `nil`. `Process.registered()` lists them.

Use this when there's one of something: a cache, a config holder. Don't register every short-lived worker.

## summary

- A BEAM process is a lightweight concurrent unit. Isolated. No shared memory.
- Communication is async messages. Request-response is a pattern you build on top.
- A server process runs a long time and handles messages. Endless recursion is the engine.
- Private state lives in the arguments of that recursion.
