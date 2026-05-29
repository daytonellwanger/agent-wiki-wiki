# Durable Execution

Durable execution is the property of a workflow or agent pipeline that allows it to survive process crashes and resume from where it left off, rather than starting over. It is a prerequisite for production-grade long-running agents.

## Why It Matters for Agents

A standard agentic loop runs entirely in memory. If the process dies mid-task — due to a crash, a deployment, a network timeout, or a resource limit — all progress is lost. For tasks that take seconds or minutes, this is acceptable. For tasks that take hours, span dozens of tool calls, trigger expensive external operations (API calls, database writes, file uploads), or involve human-in-the-loop pauses, silent loss of progress is not.

Agent pipelines also need to handle failures gracefully: a tool call that returns an error should trigger a retry with backoff, not a full restart from the beginning. And in multi-agent systems, individual worker crashes should not corrupt shared state or produce duplicate side effects. These requirements together define the durable execution problem.

## The Core Mechanism: Checkpointing

The foundation of durable execution is checkpointing: after each step, the output is persisted to stable storage before the next step begins. If the process crashes, a new worker can read the checkpoint, skip the already-completed steps, and resume from the failure point.

This requires:
1. **Idempotency**: re-running a step that already completed must not produce duplicate side effects (double charges, double emails). Usually achieved by recording step outputs before executing side effects, then checking whether the output already exists before re-executing.
2. **Deterministic replay**: when resuming from a checkpoint, the workflow must reconstruct the same execution state from stored history. This is the central design constraint that distinguishes durable execution frameworks from generic job queues.
3. **Exactly-once dequeuing**: when a crashed worker's workflow is picked up by a new worker, only one worker should execute it. Usually achieved through database-level locking or leases.

## Implementation Approaches

### Dedicated Workflow Orchestrators

Purpose-built systems like **Temporal** and its open-source variant **Cadence** handle durability as a first-class concern. The orchestrator maintains a complete event history for every workflow; workers execute steps and emit events; the orchestrator replays events to reconstruct state after a crash. This architecture cleanly separates the durability concern from the business logic.

Tradeoffs:
- Operationally complex — Temporal typically runs against Cassandra for production-scale event history, which can require substantial infrastructure (12-node clusters reported for high-event-count workloads).
- A separate point of failure and deployment concern.
- Strong semantics and a mature ecosystem of SDKs, tooling, and patterns.

Lighter-weight alternatives in this category include **Restate** (uses a Kafka-backed journal; simpler ops), **Inngest** (managed service), and **Hatchet** (Postgres-backed, open-source, deliberately simpler than Temporal).

### Database-Backed Approaches

An alternative approach eliminates the dedicated orchestrator entirely and uses a general-purpose database as the coordination layer. Workers poll a job table, lock rows with `SELECT ... FOR UPDATE SKIP LOCKED`, checkpoint step outputs as records in the same database, and detect duplicate execution attempts via uniqueness constraints. Crash recovery works because the checkpoint records survive the worker crash; a new worker picks up the locked workflow after the lease expires.

The approach eliminates a separate orchestrator as a point of failure and co-locates workflow state with application data in a single system.

Job queue libraries like **Oban** (Elixir/Postgres) and **River** (Go/Postgres) take a related approach specifically for background jobs with retries.

Tradeoffs:
- Operationally simpler — fewer moving parts, SQL-based observability, no additional infrastructure.
- `SKIP LOCKED` queues work well at moderate scale but can degrade when worker count is high: dead-tuple accumulation from frequent updates can confuse the Postgres query planner and block vacuum from keeping up.
- Building the full feature set of a mature orchestrator on top of a database requires significant effort: retries with backoff, timeouts, cancellation, versioning, heartbeats, stuck-worker detection, fan-out/fan-in, long timers, and operator tooling all need to be built or assembled.
- Works well as a starting point and for moderate-throughput systems; high-throughput or complex orchestration logic usually pushes teams toward a dedicated tool.

### Workflow Frameworks with Built-in Durability

Some agent frameworks bundle durable execution directly. LangGraph's persistence layer (checkpointers) stores graph state between steps, supporting resumable multi-step agents. This is narrower in scope than a full workflow orchestrator but sufficient for single-agent pipelines where replay semantics are simple.

## Relevance to Agent Infrastructure

Durable execution is most important for agent workloads with these properties:

- **Long time horizon**: hours-long tasks, or tasks that pause waiting for human input, external events, or slow APIs.
- **Expensive intermediate steps**: LLM calls are billed per token; repeating 50 tool calls because step 51 failed wastes significant money.
- **Side effects**: any step that sends an email, charges a card, writes to an external database, or otherwise affects the world must not be re-executed on retry.
- **Multi-agent fan-out**: orchestrators spawning many parallel subagents need to track partial completion so that a partial failure doesn't require restarting the whole fan-out.
- **Compliance and auditability**: regulated domains may require an auditable record of every step the agent took, which checkpointing naturally provides.

For short, stateless agent calls where replay is cheap and side effects are reversible, the overhead of a durable execution layer may not be worth it. The decision point is usually: "can I afford to restart this workflow from scratch if it fails?"

## Key Design Concerns

**Versioning**: persisted checkpoints encode the workflow logic at the time of execution. If you deploy a new version of the workflow code while old executions are in-flight, replaying their checkpoints against the new code may produce incorrect state. This is one of the hardest operational problems in durable execution and is why systems like Temporal have explicit workflow versioning primitives.

**Observability**: workflow state stored in a database is queryable via SQL, which is convenient. But long-running workflows with hundreds of events per execution can produce large event histories; queries against them require attention to indexing.

**Idempotency keys**: external API calls within a workflow need idempotency keys to prevent double-execution on retry. This is a design discipline that must be applied call by call; no framework enforces it automatically.

## See Also

- [Agentic Loop](agentic-loop.md) — the fundamental execution pattern that durable execution extends
- [Multi-Agent Coordination](multi-agent.md) — fan-out/fan-in patterns that durable execution supports
- [Planning](planning.md) — long-horizon task decomposition that creates the need for checkpointing
- [LangGraph](../tools/langgraph.md) — graph-based orchestration with built-in persistence/checkpointing
