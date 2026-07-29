---
topic: temporal
status: wip
tags: [distributed-systems, workflow-orchestration]
---

# Temporal advanced

### Retries: Activities vs Workflows

Activities are exportable functions (TypeScript) that return Promises, and they get a Retry Policy by default, covered in [basics.md](basics.md).

Workflows don't. No Retry Policy is attached to a Workflow Execution unless you set one explicitly in `WorkflowOptions`. Leave it unset and an unhandled exception inside Workflow code just fails the execution, no automatic retry. Makes sense once you think about it: a Workflow failure usually means the orchestration logic itself hit a case it can't handle, not a transient blip worth retrying blindly.

### Input/output shape: use an object

Prefer a single object for Workflow and Activity input/output over positional arguments. Add a field to an object later and old callers still work. Add a positional argument and you've broken the signature for everyone already calling it. This matters more for Temporal than most APIs because a Workflow's input is replayed from history for the life of the execution, sometimes years, so the shape has to survive code changes.

### Workflow ID Reuse Policy

Controls whether you can start a new Workflow Execution using a Workflow ID that already belongs to a **closed** execution.

- `WORKFLOW_ID_REUSE_POLICY_ALLOW_DUPLICATE` — start a new execution with that ID no matter how the previous one ended.
- `WORKFLOW_ID_REUSE_POLICY_ALLOW_DUPLICATE_FAILED_ONLY` — only allowed if the previous execution with that ID didn't complete successfully (failed, timed out, canceled, terminated).
- `WORKFLOW_ID_REUSE_POLICY_REJECT_DUPLICATE` — never reuse the ID, closed or not.

Try to start a Workflow with an ID the policy rejects and you get a `WorkflowExecutionAlreadyStartedError`.

### Workflow ID Conflict Policy

Same idea, different situation: what happens when you try to start with an ID that's currently **open** (still running), not closed. Reuse Policy doesn't apply here, Conflict Policy does. Lets you choose between failing the start call, attaching to the already-running execution, or terminating it and starting fresh.

### Retention Period

How long a Namespace keeps a Closed Workflow Execution's history and visibility data before deleting it. Set per Namespace. Once it's gone, you can't query, view, or replay that execution anymore, so it needs to outlive whatever debugging or audit window matters for that namespace.

### Workflow Execution Status

An execution is either **Open** or **Closed**. Closed splits into six terminal states:

- **Completed** — returned normally.
- **Continued-As-New** — the Workflow deliberately restarted itself with fresh history (the mechanism for avoiding the Event History limit below on long-running or looping Workflows).
- **Failed** — unhandled exception, no retry configured (or retries exhausted).
- **Canceled** — cancellation requested and honored.
- **Terminated** — killed from outside, forcibly, no chance for the Workflow code to react.
- **Timed Out** — exceeded its Workflow Execution Timeout.

### Logging

Don't use a third-party logger directly inside Workflow or Activity code. Use the SDK's replay-safe logger instead.

Inside Workflows, import `log` from `@temporalio/workflow`. Because a Workflow's state is reconstructed by replaying its whole event history, that log call would otherwise fire again on every replay. The SDK suppresses those duplicates automatically. Activities don't need this: a completed Activity is never re-executed, so there's no replay to suppress against. Get it via `activity.Context.current().log`.

By default the logger just writes to the Worker's stdout. Swap it out by building a `DefaultLogger` (wrap Winston or whatever you already use) and installing it with `Runtime.install({ logger })`. You keep the replay-safety guarantee either way, you're just choosing the format and destination.

### Timers

Durable Timers are tracked by the Cluster, not the Worker. If a Workflow crashes and the Worker restarts it, the timer doesn't reset, it keeps counting from when it first started, because that start time lives in the event history on the server side, not in the Worker's memory.

### Event History limits

If a Workflow Execution's Event History goes past 50K events (51,200 exactly), the execution may get terminated by the server. This is the real reason Continue-As-New exists: any Workflow that loops or runs long enough to approach that ceiling needs to periodically restart itself with a clean history instead of letting it grow unbounded.

### Sticky Execution

By default a Worker doesn't replay a Workflow's whole Event History on every Workflow Task. After the first task, the Worker caches the Workflow's state in memory, and the Cluster starts scheduling that execution's tasks on a **Sticky Queue**: a task queue private to that one Worker process. Subsequent tasks only carry the new events, so the Worker just applies the delta to its cached state. That's Sticky Execution, and it's on by default because full replays are expensive.

The fallback is what makes it safe. If the sticky Worker doesn't pick up a task within a timeout, the Cluster puts the task back on the original (shared) task queue, and whichever Worker grabs it does a full history replay to rebuild the state from scratch. So a Worker dying costs you one replay, not correctness.

### Determinism: what breaks it

Replay only works if the Workflow code takes the same path through the same history every time. The usual ways to break that:

- **Random numbers.** A different value on replay means a different branch.
- **Accessing or mutating external systems or state.** Network calls, DB reads, filesystem, environment variables, shared globals. Anything outside the history can differ between the original run and the replay. This is what Activities are for.
- **System time.** `new Date()` or `Date.now()` at replay time isn't the time of the original run.
- **Iterating over data structures with unknown ordering.** Iterate over a map or set with no ordering guarantee and the replay might visit entries in a different order than the original run.

The TypeScript SDK softens the first and third: Workflow code runs in a sandbox where `Math.random()` and `Date` are patched with deterministic, replay-safe versions. The other two are still on you.

### Testing: @temporalio/testing

The SDK ships a testing package, `@temporalio/testing`. The main tool is `TestWorkflowEnvironment`. Its time-skipping mode runs Workflows against a test server that fast-forwards time whenever the Workflow is just waiting, so a Workflow with a 30-day sleep in it finishes in milliseconds instead of a month. There's also a mock environment for unit-testing Activities in isolation, without a Worker or server.
