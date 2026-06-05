---
topic: book-summaries
status: wip
tags: [systems, data, data-models]
---

# Ch 3 — Data models and query languages

### Declarative query languages

You state what data you want: the pattern, the conditions, the transformation. You don't say how to get it. The query optimizer picks the execution path, indexes, and join order. SQL is declarative.

### Impedance mismatch

The awkward translation layer between objects in your code and rows in a relational database. Your code thinks in nested objects and references; the DB thinks in flat tables and foreign keys.

### Denormalization

Denormalize stable facts, not fast-changing ones. If something rarely changes, copying it around is cheap. If it churns, every copy is another write you have to keep in sync. Duplicate the idea, not the data that moves.

> the most scalable approach may involve denormalizing some things and leaving others normalized. You will have to carefully consider how often the information changes and the cost of reads and writes (which might be dominated by outliers, such as users with many follows/followers in the case of a typical social network). Normalization and denormalization are not inherently good or bad—they simply represent trade-offs in terms of performance of reads and writes and implementation effort.

### Star and snowflake schemas

A fact table in the middle holds the events (sales, clicks, whatever), with dimension tables around it for the who/what/when/where. That's a star. A snowflake goes further and normalizes the dimensions into subdimensions, so a dimension table points to its own smaller tables.

### Schema-on-read vs schema-on-write

Schema-on-write is relational. The DB enforces structure when data goes in, like static typing. Schema-on-read is the document world. There's no enforcement on write; structure is assumed when you read, like dynamic typing.

Say you want to split `name` into `first_name` and `last_name`. Schema-on-read: start writing new documents in the new shape and handle the old shape in code at read time. Schema-on-write: run a migration, `ALTER TABLE`, backfill. One defers the work to read code, the other pays it up front.

### Graph queries: Cypher vs SQL

Cypher is declarative and reads close to the question you're asking. Find people born in the US who now live in Europe:

```cypher
MATCH
  (person) -[:BORN_IN]->  () -[:WITHIN*0..]-> (:Location {name:'United States'}),
  (person) -[:LIVES_IN]-> () -[:WITHIN*0..]-> (:Location {name:'Europe'})
RETURN person.name
```

The same query in SQL needs recursive CTEs to walk the variable-length `WITHIN` chains, and it runs several dozen lines. The graph model fits this; the relational one fights it.

Triple stores store everything as (subject, predicate, object), e.g. (Jim, likes, bananas). SPARQL is their query language.

### Event sourcing and CQRS

> The idea of using events as the source of truth and expressing every state change as an event is known as event sourcing. The principle of maintaining separate read-optimized representations and deriving them from the write-optimized representation is called command query responsibility segregation (CQRS)

### General Notes

Relational still dominates warehousing and analytics. The document model fits self-contained JSON where cross-document relationships are rare. Graph models fit the opposite case: everything related to everything, multi-hop queries. Dataframes bridge databases and ML's multidimensional arrays.

Every model can emulate the others, but awkwardly, like recursive SQL faking graph traversal. The lines are blurring anyway: relational DBs now have JSON columns, document DBs now do joins. And nonrelational models usually don't enforce schema, but the structure assumption doesn't vanish. It just goes implicit (on read) instead of explicit (on write).

back to [[non-functional-requirements]] for Ch 2.
