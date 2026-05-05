# Memory

Memory refers to how agents retain and retrieve information across turns, sessions, or tasks. It's one of the least-settled areas of agent architecture — the right approach varies significantly by use case.

## Types of Memory

### In-Context Memory

The simplest form: everything the agent needs is in the current context window. Works well for short, self-contained tasks. Breaks down when tasks are long, sessions span multiple interactions, or information must survive across users.

### External Memory (Retrieval)

Information is stored outside the model and retrieved on demand — typically via vector similarity search (RAG), keyword search, or structured database queries. The model sees only the retrieved snippets, not the full store.

**Strengths**: scales to large information sets, can be updated without retraining.
**Weaknesses**: retrieval quality determines answer quality; failure modes are hard to predict.

### Episodic Memory

A log of past actions, observations, and outcomes — analogous to a human's memory of specific events. Useful for agents that need to avoid repeating mistakes, recall past decisions, or learn from experience within a session or across sessions.

### Procedural / Instruction Memory

Stored instructions, preferences, or workflows that shape how the agent behaves. Often implemented as a system prompt that is updated over time, or a small database of user preferences retrieved at session start.

## Key Challenges

- **What to store**: not everything is worth remembering; deciding what to write to memory is a hard problem.
- **Context window pressure**: retrieved memory consumes tokens; poor retrieval wastes them on irrelevant content.
- **Staleness**: external memory can go out of date; agents need strategies for detecting and handling stale information.
- **Compaction**: for long-running agents, conversation history must be summarized or truncated without losing critical information. See [Context Management](context-management.md).

## Current Practice

Most production agents use a combination: in-context memory for the current task, retrieval (RAG) for domain knowledge, and either a session summary or a key-value store for cross-session persistence. There is no widely-adopted standard; this area is actively evolving.

## See Also

- [Context Management](context-management.md)
- [Agentic Loop](agentic-loop.md)
- [Tool Use](tool-use.md)
