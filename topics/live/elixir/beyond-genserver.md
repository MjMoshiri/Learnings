---
topic: elixir
status: wip
---

# beyond genserver

Notes from Elixir in Action ch 10. Task, Agent, ETS. The APIs are thin. The point is what deserves a process, who owns state, and when a process becomes a bottleneck.

A process is not an object. A GenServer has a mailbox, private state, and a lifecycle. Nobody reaches in and mutates fields. They send messages. That's isolation. Not everything should be a permanent server.

## Task: do this, then die

A GenServer says: I exist. Send me things.

A Task says: give me a job. I'll do it. Then I'm gone.

It's a BEAM process, not a scheduled callback.

```elixir
task = Task.async(fn -> expensive_query() end)
do_something_else()
result = Task.await(task)
```

Need the result: `async` + `await`. Independent jobs run at the same time. Five two-second queries finish in about two seconds, not ten. If B doesn't depend on A, separate processes.

Don't need a return value: start a Task and let it exit. Metrics, a side effect, a background send.

Don't spawn from whatever happens to be running. Put important jobs under `Task.Supervisor`. Separate lifecycles. An HTTP request can reply "accepted" while the job retries, finishes, or notifies on its own. The request and the work shouldn't live and die together.

## Agent: GenServer that only holds state

Get, update, get, update. That's an Agent. GenServer underneath. Fine when that's all you need.

```elixir
{:ok, counter} = Agent.start_link(fn -> 0 end)
Agent.update(counter, fn x -> x + 1 end)
Agent.get(counter, fn x -> x end)
```

Operations are serialized because the Agent is a process. No new concurrency model.

You outgrow it fast. Idle timeout, arbitrary messages, termination hooks: you need `handle_info`. The book goes back to GenServer for that. When in doubt, use GenServer. You won't outgrow it. Agent is "I just need process-owned state." Don't obsess over it.

## ETS: the bottleneck lesson

One GenServer, a map, 10k clients doing get/put. Everything through one mailbox. Thousands of processes, and you serialized them anyway. That's a single-process bottleneck. No concurrency if everyone waits on one process.

ETS (Erlang Term Storage) is a fast in-memory table in the BEAM. Processes read and write it directly. Concurrent access. Ordinary Elixir terms. Caches, lookups, registries, counters.

Not Redis. In-memory, tied to the node. Data is copied in and out. A table has an owner. Owner dies, table usually dies. Keep a small supervised process that creates the table and stays alive. Clients don't ask that process for data. They hit ETS themselves.

Start with GenServer. Profile. If that process is the bottleneck and the state fits a table, then ETS. Not "ETS is faster, so use ETS." It's an optimization. Ordinary immutable data and processes until the numbers say otherwise.

## the questions

Need something concurrent?

- Job that finishes → Task
- State and behavior over time → GenServer
  - Just a state holder → maybe Agent
  - Shared-data bottleneck → ETS

Don't organize around classes. Organize around independent activities.

Who owns this state?
Who does this work?
How long should it live?
What happens if it crashes?
Who should crash with it?
Does everyone need to go through one process?

That's the chapter.
