---
topic: elixir
status: wip
---

# beyond genserver

Task, Agent, ETS. Thin APIs. What deserves a process, who owns state, when a process becomes a bottleneck.

A GenServer is an actor: mailbox, private state, lifecycle. Nobody mutates its fields. They send messages. That's isolation. Not everything should be a permanent server.

## Task: do this, then die

A GenServer says: I exist. Send me things.

A Task says: give me a job. I'll do it. Then I'm gone.

A Task is a BEAM process.

```elixir
task = Task.async(fn -> expensive_query() end)
do_something_else()
result = Task.await(task)
```

Need the result: `async` + `await`. Independent jobs run at the same time. Five two-second queries finish in about two seconds, not ten.

Don't need a return value: start a Task and let it exit. Metrics, a side effect, a background send.

Don't spawn from whatever happens to be running. Put important jobs under `Task.Supervisor`. An HTTP request can reply "accepted" while the job retries, finishes, or notifies on its own. The request and the work shouldn't live and die together.

## Agent: GenServer that only holds state

Get, update, get, update. That's an Agent. GenServer underneath. Fine when that's all you need.

```elixir
{:ok, counter} = Agent.start_link(fn -> 0 end)
Agent.update(counter, fn x -> x + 1 end)
Agent.get(counter, fn x -> x end)
```

Operations are serialized because the Agent is a process.

You outgrow it fast. Idle timeout, arbitrary messages, termination hooks: you need `handle_info`. Agent doesn't have it. When in doubt, use GenServer. You won't outgrow it. Agent is "I just need process-owned state."

## ETS: the bottleneck lesson

One GenServer, a map, 10k clients doing get/put. Everything through one mailbox. Thousands of processes, and you serialized them anyway. Single-process bottleneck.

ETS (Erlang Term Storage) is a fast in-memory table in the BEAM. Processes read and write it directly. Concurrent access. Ordinary Elixir terms. Caches, lookups, registries, counters.

In-memory, tied to the node. Data is copied in and out. A table has an owner. Owner dies, table usually dies. Keep a small supervised process that creates the table and stays alive. Clients don't ask that process for data. They hit ETS themselves.

Start with GenServer. Profile. If that process is the bottleneck and the state fits a table, then ETS. Ordinary immutable data and processes until the numbers say otherwise.

## the questions

Need something concurrent?

- Job that finishes → Task
- State and behavior over time → GenServer
  - Just a state holder → maybe Agent
  - Shared-data bottleneck → ETS

Who owns this state?
Who does this work?
How long should it live?
What happens if it crashes?
Who should crash with it?
Does everyone need to go through one process?
