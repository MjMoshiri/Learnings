---
topic: temporal
status: wip
tags: [distributed-systems, workflow-orchestration]
---

# Temporal basics

### What it needs to run

- **Persistence DB**: history, mutable state, task queues. Production options: PostgreSQL, MySQL, Cassandra. SQLite is dev-only, used by the CLI dev server.
- **Visibility**: lets you list/search/filter workflow executions by status, custom search attributes, etc. Basic visibility can run on the same SQL DB but it's limited (no custom search attributes, weak filtering). Elasticsearch gives you advanced visibility; expect it in production.
- **Cluster**: not just "the server." It's a set of internal services. Notably a Frontend Service that accepts client/SDK requests (gRPC, optionally TLS) and routes them to the right backend service (history, matching, etc).

### The core pieces

**Workflow.** A durable function defining orchestration logic: the steps, which activities get called, in what order, with what branching. Must be deterministic, same code plus the same event history replayed always produces the same result. That's why workflow code can't do direct I/O, call `random()`, or read the wall clock directly; all of that goes through the SDK's deterministic APIs. Workflow state isn't stored anywhere directly, it's reconstructed by replaying the event history through the workflow code.

**Activity.** A single unit of work. All I/O and non-determinism belongs here: calling an external API, writing to a database, sending an email, LLM invocations, anything that can fail or come back different on a retry. This is exactly what makes Temporal usable for durable AI applications: the non-deterministic LLM call happens inside an Activity, so it never violates Workflow determinism.

**Worker.** A process that hosts the actual Workflow and Activity implementations. It polls task queues for work, executes whatever matches, and reports the result back to the server. Pull-based long polling, not server push: a worker blocks on a poll call (~60s), and the moment it returns, work or not, it polls again. Scaling horizontally just means running more worker processes pointed at the same task queue. Workers aren't hot-reloaded, restart them after deploying any code change to Workflow or Activity code.

### Deployment options

- **CLI dev server** (`temporal server start-dev`): single binary, in-memory SQLite, starts instantly, ships with a Web UI. Local dev and quick experiments only, nothing survives a restart or handles real load.
- **Docker Compose** (`temporalio/docker-compose` repo): full stack as separate containers, server, real database (Postgres/MySQL), Elasticsearch, Web UI. Good for local integration testing or shared dev/staging. Not a production recommendation on its own.
- **Self-hosted**: you deploy Temporal Server's services yourself (frontend, history, matching, worker) on your own infra, and own the database and Elasticsearch cluster. Full control, but you own all the ops: scaling, upgrades, on-call, backups. Makes sense with a platform team or compliance/data-residency requirements that rule out a third party.
- **Temporal Cloud**: managed SaaS. Temporal runs the server, database, and Elasticsearch; you just run Workers and point them at your namespace. No infra to run, pay per action/GB. Default recommendation for production. Migrating an app to or from Temporal Cloud needs very little code change, since Workers just point at a different endpoint/namespace.

### Task Queues

A named, lightweight queue that workers long-poll. Not pre-provisioned anywhere, it's created implicitly the first time something references its name. Same name used on both sides: the client uses it to start a workflow or schedule an activity, workers use it to know what to pick up. You can (and often should) route specific work to specific worker pools with different task queues, e.g. a GPU-only activity gets its own queue so only GPU workers poll it.

### Namespaces

A unit of isolation within a cluster, roughly a separate database or tenant. Fully isolated: separate task queues, visibility data, retention policies. Typical use: splitting dev/staging/prod, or isolating tenants, inside the same physical cluster.

### Registering activities and workflows

Activities are plain functions or methods in your worker code. You register them explicitly with a Worker instance when you construct it (API differs per SDK: an activities module in TypeScript, `worker.RegisterActivity(...)` in Go, a list of functions in Python). Workflows get registered the same way. A worker only ever executes the workflow types and activity functions it's been told about.

### Activity Options

Configuration attached when a workflow calls an activity:

- `StartToCloseTimeout` — max time for one attempt.
- `ScheduleToCloseTimeout` — max time across all retries combined.
- `ScheduleToStartTimeout` — max time it can sit in the queue before a worker picks it up.
- `HeartbeatTimeout` — for long-running activities that report progress.
- `RetryPolicy` — covered below.
- `TaskQueue` override — send this activity to a different worker pool than the one the workflow runs on.

You must set at least one of `StartToCloseTimeout` or `ScheduleToCloseTimeout`. Temporal won't let an activity run with no timeout at all.

### Proxy Activities

Workflow code can't call an activity function directly (that would call arbitrary, non-deterministic code from inside deterministic code, and doesn't make sense across process boundaries anyway, since the activity might run on a different worker). Instead the SDK gives you a proxy. TypeScript: `proxyActivities<typeof activities>(options)` returns typed functions that look like normal async calls but actually schedule an ActivityTask on the task queue and await the result. Go: `workflow.ExecuteActivity(ctx, ActivityFn, args...)`. Activity Options get attached here, per call site.

### Activity Retry Policy

Temporal retries failed activities automatically by default (1s initial interval, 2.0 backoff coefficient, capped at 100x the initial interval, unlimited attempts unless you cap it). Don't rely on the default, set your own on purpose.

Fields that matter:

- `InitialInterval` — wait before the first retry.
- `BackoffCoefficient` — multiplier applied each subsequent retry. 2.0 means 1s, 2s, 4s, 8s...
- `MaximumInterval` — cap so backoff doesn't grow forever.
- `MaximumAttempts` — 0 means unlimited.
- `NonRetryableErrorTypes` — error types where retrying can't help. A declined credit card should never retry. A network timeout should.

Say you're charging a customer:

```typescript
const { chargeCustomer } = proxyActivities<typeof activities>({
  startToCloseTimeout: '30s',
  retry: {
    initialInterval: '1s',
    backoffCoefficient: 2.0,
    maximumInterval: '60s',
    maximumAttempts: 5,
    nonRetryableErrorTypes: ['CardDeclinedError'],
  },
});
```

If `chargeCustomer` fails, the server schedules a retry after `initialInterval`, then doubles the wait each time up to `maximumInterval`, and gives up either once `maximumAttempts` is hit or immediately if the thrown error's type is in `nonRetryableErrorTypes`. A declined card fails once and stops. A network blip gets five tries with growing backoff before Temporal gives up on it too.

### Commands: how replay actually produces work

This is the mechanism that makes the whole Workflow/replay model work.

When a worker processes a Workflow Task, it runs the workflow function deterministically against the current event history. Every SDK call inside that function with a side effect (`ExecuteActivity`, `StartTimer`, `SignalExternalWorkflowExecution`, `CompleteWorkflowExecution`, and so on) doesn't actually execute anything itself. It gets translated into a Command (`ScheduleActivityTask`, `StartTimer`, etc), and the batch of commands produced by that run is what the worker sends back to the server as the result of completing the Workflow Task. The server appends those as new Events in the workflow's history.

So the workflow function's real job, every time it runs, is: look at the history so far, decide what commands to emit next. Durability falls out of this for free, replaying the same deterministic code against the same history always produces the same commands, so the workflow can resume from a crash anywhere in its execution and land in exactly the same place. That's the whole pitch: you don't build your own retry logic, crash-recovery bookkeeping, or "did this already run" checks. Temporal owns that.

### Web UI

Main debugging/observability tool. Where you go to see current and recent Workflow Executions.

For a given execution, it shows:

- Workflow Type
- Task Queue
- start/close time
- namespace
- inputs and outputs
- full event history
