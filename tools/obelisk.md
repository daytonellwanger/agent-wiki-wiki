# Obelisk

Obelisk is an open-source durable workflow system designed for AI agent workloads. Its distinguishing feature is first-class support for SQLite as a workflow store, using Litestream to replicate the database file to object storage — so compute is disposable but workflow state persists.

## What It Does

Obelisk maintains an execution log in a local database. When a workflow or agent pipeline needs to resume after a crash, it replays the log to reconstruct state and retry failed activities. This is the standard deterministic-replay model (cf. Temporal, Restate), but applied to a per-worker SQLite file rather than a centralized event store.

## Storage Backends

- **SQLite + Litestream**: Each worker writes to a local SQLite file. Litestream streams changes asynchronously to S3-compatible object storage. Compute units are disposable; state survives as long as the backup reaches object storage. Suited for bursty, experimental, or lower-stakes workloads.
- **PostgreSQL**: For deployments requiring higher availability, broader scalability, or stronger durability guarantees.

The author's framing: "SQLite is the right default; Postgres is the right choice when you need higher availability, broader shared scalability, or other deployment properties."

## Deployment Model

The intended topology is a fleet of small, isolated compute units — micro-VMs or containers — each running one Obelisk worker with its own SQLite database. Observers can pull the SQLite file directly for inspection and debugging. This model favors fault isolation and cost efficiency over high availability.

## Key Limitation

Litestream replication is asynchronous. If a compute volume is destroyed before the latest writes are replicated, those writes are lost. This is acceptable for many AI agent use cases but is not a substitute for synchronous durability when that property is required.

## Context

The project was discussed on Hacker News with 323 points and 188 comments. The discussion surfaced substantial community experience with SQLite at scale (including multi-million-MAU apps) as well as skepticism about concurrent write limits and type system weaknesses relative to Postgres. The consensus in the thread: SQLite is a reasonable default for single-writer, naturally-partitioned workflows; shared-database systems are necessary when concurrency or availability requirements grow.

## See Also

- [Durable Execution](../concepts/durable-execution.md) — the broader pattern Obelisk implements, including SQLite and Postgres variants
- [Agentic Loop](../concepts/agentic-loop.md) — the execution model that durable execution extends
