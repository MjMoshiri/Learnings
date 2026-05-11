# Python vs Rust API Benchmark

Two HTTP APIs with the same endpoints (`/hello`, `/fib/{n}`), one in FastAPI, one in Actix Web. Both in Docker with matching CPU and memory limits. Bombardier as the load generator, three runs averaged.

Single core, 50 connections: Rust about 11x faster on requests/sec and latency. Dual core, 200 connections: gap shrinks to 5–6x, partly because the test setup hit a network ceiling.

The number is not the lesson. The lesson is that fair language benchmarks need pinned resources, identical workload, and honest acknowledgement of what hit the ceiling first.
