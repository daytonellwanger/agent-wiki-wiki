# Multi-Agent Coordination

Multi-agent systems use multiple LLM instances — each with its own context, role, and tool access — to collaborate on a task. The motivation is specialization and parallelism: some tasks are too large for one context window, benefit from independent verification, or contain sub-tasks that can be done concurrently.

## Common Architectures

### Orchestrator + Subagents

An orchestrator agent decomposes a task, delegates sub-tasks to specialized subagents, and synthesizes results. The subagents may have narrower tool access and more specific instructions than the orchestrator.

This is the most common pattern in production. [LangGraph](../tools/langgraph.md) and [OpenAI Agents SDK](../tools/openai-agents-sdk.md) both provide primitives for it.

### Parallel Workers

Multiple agents work independently on different parts of a task simultaneously, with their results merged afterward. Useful when sub-tasks are truly independent (e.g., processing many documents in parallel).

### Debate / Verification

One agent produces a result; another reviews or critiques it. This can catch errors the first agent wouldn't catch itself, at the cost of additional latency and tokens.

### Peer Networks

Agents with equal standing collaborate, passing messages and negotiating. This pattern (popularized by [AutoGen](../tools/autogen.md)) is more flexible but harder to control and debug.

## Key Challenges

- **Context isolation**: each agent has its own context and doesn't automatically see what others have done. Explicit message-passing or shared state is required.
- **Error propagation**: a mistake by one subagent can cascade. The orchestrator needs to handle failures gracefully.
- **Coordination overhead**: spawning agents has latency and token cost; not every task benefits.
- **Trust boundaries**: when an orchestrator delegates to a subagent, deciding how much autonomy to grant — and how to validate results — is non-trivial.
- **Debugging**: tracing behavior across multiple agents is significantly harder than tracing a single agent.

## When to Use Multi-Agent

Use multi-agent when:
- The task genuinely exceeds one context window.
- Sub-tasks are parallel and independent.
- Independent verification would materially improve reliability.

Avoid it when a single capable agent would do — the coordination overhead is real.

## See Also

- [Agentic Loop](agentic-loop.md)
- [Planning](planning.md)
- [LangGraph](../tools/langgraph.md)
- [OpenAI Agents SDK](../tools/openai-agents-sdk.md)
- [AutoGen](../tools/autogen.md)
