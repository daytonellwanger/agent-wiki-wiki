# Multi-Agent Coordination

Multi-agent systems use multiple LLM instances — each with its own context, role, and tool access — to collaborate on a task. The motivation is specialization and parallelism: some tasks are too large for one context window, benefit from independent verification, or contain sub-tasks that can be done concurrently.

## Common Architectures

### Orchestrator + Subagents

An orchestrator agent decomposes a task, delegates sub-tasks to specialized subagents, and synthesizes results. The subagents may have narrower tool access and more specific instructions than the orchestrator.

This is the most common pattern in production. [LangGraph](../tools/langgraph.md) and [OpenAI Agents SDK](../tools/openai-agents-sdk.md) both provide primitives for it. [Claude Code Subagents](../tools/claude-code-subagents.md) is a well-documented, production-grade implementation: the main session delegates to named child agents (Explore, Plan, General-purpose, or custom) each running in its own context window with scoped tool access.

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

## Task Decomposition: Narrow and Parallel vs. Sequential

A key design decision when applying multi-agent systems to a new problem is whether the task structure calls for narrow-and-parallel exploration or sequential execution.

Generic single-stream coding agents work well for sequential tasks like feature development: one context window follows a logical chain from requirements to implementation. They fail at tasks that require simultaneous exploration across many independent hypotheses — the agent can only pursue one thread at a time, and the search space is too large.

Cloudflare's Project Glasswing (May 2026) is a concrete example: applying Anthropic's Mythos Preview model to vulnerability discovery across a large codebase. A single-agent approach would sequentially investigate one vulnerability class at a time. Instead, Cloudflare built a seven-stage harness running approximately 50 concurrent agents, each scoped to a narrow vulnerability class:

1. **Recon** — architecture mapping and task queue generation
2. **Hunt** — parallel agents investigating specific vulnerability classes
3. **Validate** — independent adversarial review to reduce noise (see Debate/Verification above)
4. **Gapfill** — re-scanning under-explored areas
5. **Dedupe** — collapsing duplicate findings
6. **Trace** — cross-repository analysis to determine actual exploitability
7. **Report** — structured output generation

The Validate stage is particularly notable: independent agents adversarially review findings produced by the Hunt agents before those findings are promoted. This is the Debate/Verification pattern applied within a larger pipeline, not as the whole system.

The lesson generalizes: when the search space has many parallel hypotheses to explore simultaneously (security auditing, large-scale code review, competitive intelligence), a narrow-and-parallel harness will outperform a sequential agent even if the sequential agent is more sophisticated. The harness's shape should match the task's shape.

## When to Use Multi-Agent

Use multi-agent when:
- The task genuinely exceeds one context window.
- Sub-tasks are parallel and independent.
- Independent verification would materially improve reliability.
- The task has many simultaneous hypotheses to explore rather than a single sequential chain (narrow-and-parallel vs. sequential).

Avoid it when a single capable agent would do — the coordination overhead is real.

## See Also

- [Agentic Loop](agentic-loop.md)
- [Planning](planning.md)
- [Claude Code Subagents](../tools/claude-code-subagents.md)
- [LangGraph](../tools/langgraph.md)
- [OpenAI Agents SDK](../tools/openai-agents-sdk.md)
- [AutoGen](../tools/autogen.md)
