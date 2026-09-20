---

topic: elixir
status: done
-----------

# building a distributed system

A distributed BEAM system is mostly the same process model stretched across machines. Processes stay isolated. They send messages. The pid may just happen to live on another node.

## nodes are just BEAMs

A node is a named BEAM instance. Connect several and you have a cluster.

```elixir
Node.connect(:node2@localhost)
```

Message passing works across nodes. That's the important abstraction: don't design around machines, design around processes.

Remote communication is less reliable, though. Messages cross a network. Nodes crash. Connections disappear. Use timeouts. Prefer calls when you need confirmation.

## find the process, not the machine

Before sending a message, you need to know which process owns the resource.

Locally, `Registry` handled that. Across the cluster:

* `:global` → one process under a cluster-wide name
* `:pg` → multiple processes in a cluster-wide group

```elixir
{:global, {Todo.Server, "Bob"}}
```

Every node can now find the process responsible for Bob. Clients don't care where it physically runs.

Global registration requires coordination across the cluster, so it's convenient, not infinitely scalable.

## survive node failure

Finding the process isn't enough. If Bob's process lived on node A and A dies, another node can start a replacement—but the data also has to survive.

The chapter's simple solution: replicate writes to every node.

```text
write
 ├─ node A
 ├─ node B
 └─ node C
```

Then a surviving node can recreate the process from its local copy.

Processes are disposable. Durable state has to live somewhere recoverable.

## distribution does not remove distributed problems

The BEAM makes distribution convenient. It doesn't make networks reliable.

A remote node not responding might mean:

* it crashed
* the network broke
* the network is slow
* the node is overloaded

You can't really know. That's why timeouts and failure handling matter.

Worst case: **network partition**. Both halves of the cluster stay alive and accept writes independently. Now you have split brain and conflicting state.

Erlang gives you tools to detect disconnections. What to do about them is still your architecture.

## same philosophy, larger scale

The useful mental model:

```text
single node:
process → message → process

cluster:
process → network → process
```

Monitors still detect failure. Supervisors still recover processes. Processes still own state.

The machine becomes another failure boundary.

## summary

* A node is a BEAM instance; connected nodes form a cluster.
* Processes and message passing remain the core abstraction across machines.
* `:global` finds one process cluster-wide; `:pg` groups many.
* Don't depend on a process's machine. Depend on its identity/responsibility.
* Replicate or otherwise persist state if another node must recover it.
* Remote calls need timeouts and acknowledgements.
* Distribution improves availability and scaling, but partitions, consistency, and partial failure are still your problem.
