---
topic: book-summaries
status: wip
tags: [systems, data, serialization, schema-evolution]
---

# Ch 5 — Encoding and evolution

### Compatibility

- Backward compatible: new code reads old data.
- Forward compatible: old code reads new data.

Forward is harder. Old code has to ignore fields it never knew about.

### JSON Schema

JSON, XML, and CSV are textual and everywhere but don't pin down structure. JSON Schema is a JSON document describing the shape of other JSON: required fields, types, constraints. Validates input, doubles as docs.

`additionalProperties` is the evolution footgun. Defaults to `true`, so any unnamed field passes through. Set it `false` and the validator rejects undeclared fields, killing forward compatibility: new code adds a field, old validators reject the data. Leave it `true` and typos slip by as extra properties. Neither default is free.

`patternProperties` validates keys by regex instead of naming each one. Fits dynamic keys, like a map keyed by user id where every value matches the same shape.

`pg_jsonschema` is a Supabase Postgres extension. Adds `json_matches_schema` and `jsonb_matches_schema`, so you validate a jsonb column inside a CHECK constraint. Schema-on-read column gets a schema-on-write guardrail without moving the data out.

### Protocol Buffers

Binary encoding driven by a `.proto` schema. Every field carries a tag number, not its name.

```proto
message Person {
  string user_name = 1;
  int64 favorite_number = 2;
  repeated string interests = 3;
}
```

The `= 1`, `= 2`, `= 3` are tags. On the wire each field is tag + type + value; the name never appears. That's where the compactness comes from, and it sets the evolution rules:

- Rename a field freely. The tag is the identity, the name is for humans.
- Add a field with a new tag. Old code skips the unknown tag (forward); new code reading old data uses a default (backward).
- Never reuse or change a tag number. That's the one move that breaks everything.
- A new field can't be required, since old data never carries it. proto3 dropped `required` for this.

`repeated` means zero or more, i.e. a list. Tags 1 to 15 encode in one byte, so give the low numbers to the fields you touch most.

### Avro

Avro drops tag numbers. The encoded bytes are just values concatenated, with nothing marking boundaries, names, or types. You can't parse Avro without the schema that wrote it.

That forces two schemas:

- Writer's schema: what the data was encoded with.
- Reader's schema: what the consuming code expects.

They needn't match versions. At decode time Avro lines them up by field name:

- In both: decode, converting type if needed.
- Writer's only: ignore it.
- Reader's only: fill the declared default.

Order doesn't matter since matching is by name. That's how Avro gets both directions without tags.

The catch: the reader needs the writer's schema, and where it comes from depends on the setting:

- One big file of many records (Hadoop dump): write the schema once at the head.
- A database written over time: tag each record with a schema version, keep a registry.
- Records over a network: negotiate the version on the handshake.

Why give up tags? Dynamically generated schemas. Dump a table to Avro and the schema falls out of the columns. Rename a column, regenerate, done. Protobuf would have you assigning and guarding tag numbers by hand forever. Avro fits the case where nobody writes the schema by hand.

### Why schema-based binary encodings earn their keep

> binary encodings based on schemas are also a viable option. They can be much more compact than the various "binary JSON" variants, since they can omit field names from the encoded data.

What you get:

- Compact, because field names don't ride along.
- The schema is documentation that can't drift, since decoding needs it current.
- A schema registry checks a change for forward/backward compatibility before it ships.
- Code generation gives compile-time type checking in statically typed languages.

Net: schema evolution buys the flexibility of schema-on-read JSON plus stronger guarantees and better tooling. Keep the number of formats small so ops stay simple.

### Modes of dataflow

Encoding matters because data crosses between processes, one side encoding while the other decodes. Three modes:

- Through a database: writer encodes, a later reader decodes, maybe with different code.
- Through service calls (RPC, REST): client encodes a request, server decodes and encodes a response, client decodes it.
- Through async messages (event-driven, actors): sender encodes, recipient decodes, usually with a broker between.

### RPC, REST, and location transparency

RPC dresses a network call as a local function call. Location transparency is the promise: the caller doesn't care which machine runs the code. The abstraction leaks, because a network call isn't a local one. It can be slow, time out, or vanish with no reply, and a retry might run the work twice unless the call is idempotent. REST is the looser, more explicit style over HTTP. RPC frameworks use the schema formats above to encode request and response.

### Service discovery and service meshes

Once many services call each other, two problems show up. Service discovery turns a service name into a current network address, since instances come and go. A service mesh pushes cross-cutting concerns (routing, retries, TLS, timeouts, metrics) into a sidecar proxy beside each service, so the calling code stays plain.

### Durable execution

Workflow engines persist a workflow's progress so it survives a crash and resumes where it stopped instead of restarting. Temporal calls a task an activity; others call them durable functions. Durable execution is now a common way to get transactionality across services, where one logical operation spans several services and has to either finish or unwind cleanly.

### Event-driven architecture and the actor model

Event-driven systems send messages instead of calling each other directly. A sender publishes an encoded message, a broker holds it, a recipient decodes and handles it. Sender and recipient are decoupled in time and identity; neither blocks on the other.

The actor model is one way to structure this. An actor is an isolated unit with private state that talks only by async messages and handles one at a time, so no shared-memory locking inside it. Distributed actor frameworks (Akka, Erlang/OTP, Orleans) spread actors across nodes, so message passing becomes encoded network traffic. The same compatibility rules apply, since a message sent by one version gets read by another.

### General notes

The encoding choice isn't a detail. It decides whether you can run rolling upgrades, which decides whether you ship small and often instead of rare and risky. Get backward and forward compatibility right and different code versions coexist safely, whichever way the data flows. Language-specific encodings trap you in one language and usually break compatibility. Textual formats (JSON, XML, CSV) are portable but loose about types. Binary schema formats (Protocol Buffers, Avro) are compact with clear compatibility rules, at the cost of not being human-readable until decoded.

back to [[storage-and-retrieval]] for Ch 4.
