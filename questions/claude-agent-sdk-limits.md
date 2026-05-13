# When Does the Claude Agent SDK Fall Short, and What Are the Alternatives?

The [Claude Agent SDK](../tools/claude-agent-sdk.md) wraps the Claude Code runtime and exposes its agentic loop programmatically: CLAUDE.md for context, subagents for delegation, skills for reusable workflows. This seems like a simple setup that works well for many agentic tasks. When would I want to use a different setup and why?

## Background

The SDK's design is covered in [Claude Agent SDK](../tools/claude-agent-sdk.md) and [How do you move an agentic Claude Code workflow to a programmable server environment?](claude-code-to-sdk.md). The properties that become constraints in certain contexts:

- The [agentic loop](../concepts/agentic-loop.md) is **open-ended**: the model decides what tools to call, in what order, and when to stop.
- Subagents support **one level of delegation**: subagents cannot spawn their own subagents.
- The SDK runs **Claude models only**.
- Each `query()` call incurs **~12s startup overhead** from spawning the CLI binary as a subprocess.
- The SDK is a **black box at the model-call level**: individual API requests are not directly accessible.

## Perspectives

### When you need explicit control flow → LangGraph

The SDK's agentic loop trusts the model to decide what to do next. This is powerful for open-ended tasks. It is the wrong abstraction when execution paths must be auditable, deterministic, or tightly constrained — regulatory pipelines, multi-step financial workflows, or anything where you need to guarantee that step B always follows step A regardless of what the model thinks.

[LangGraph](../tools/langgraph.md) fills this gap. It models workflows as directed graphs where control flow is code, not model output: branches, loops, parallel execution, and human-in-the-loop pauses are all explicit graph structures. The tradeoff is that you write more orchestration logic yourself. LangGraph's checkpointing system also supports resuming failed workflows and time-travel debugging — both unavailable in the SDK's agentic loop.

### When you need deeper or peer-based agent coordination

The SDK supports one level of subagent delegation only — subagents cannot spawn their own subagents, and there is no peer messaging between agents. Most tasks fit within one level, but complex pipelines with multiple independent stages can benefit from deeper hierarchies or peer collaboration.

[LangGraph](../tools/langgraph.md) supports deeper nesting via subgraphs (a node that is itself a full graph). [AutoGen](../tools/autogen.md) offers peer-based coordination where agents communicate as equals rather than in a strict hierarchy — well-suited for iterative refinement loops where a planner, coder, and critic collaborate without a fixed orchestrator. See [Multi-Agent Coordination](../concepts/multi-agent.md) for the architectural tradeoffs.

### When startup overhead or fine-grained API control matters → Anthropic client SDK

The SDK spawns the Claude Code CLI binary per `query()` call, incurring ~12 seconds of startup latency regardless of task complexity. Long-running containers amortize this, but it is a hard floor that makes the SDK unsuitable for low-latency request/response APIs.

More broadly, the SDK is opaque at the model-call level: you cannot inspect individual API requests, control exact caching behavior, or tune per-step prompts. When you need full control over every model call — specific token budgets, precise prompt engineering, custom streaming handling — the [Anthropic client SDK](../tools/anthropic-client-sdk.md) provides direct Messages API access with no subprocess overhead. The cost is that you implement the agentic loop, tool execution, and context management yourself.

### When you need multi-provider orchestration

The SDK runs only Claude models. Tasks that benefit from mixing providers — a fast model for triage, a powerful model for synthesis, a separate vision model for images — require a provider-agnostic framework. [LangGraph](../tools/langgraph.md) and [LangChain](../tools/langchain.md) both support multi-provider pipelines natively.

## Answer

The Claude Agent SDK is the right default for server-side agentic tasks using Claude, but four properties make it the wrong fit in specific contexts:

| Constraint | When it hurts | Alternative |
|---|---|---|
| Open-ended agentic loop | Auditable, deterministic pipelines | [LangGraph](../tools/langgraph.md) |
| One-level subagent nesting | Deep hierarchies or peer coordination | [LangGraph](../tools/langgraph.md) subgraphs, [AutoGen](../tools/autogen.md) |
| ~12s startup + opaque model calls | Low-latency APIs, fine-grained prompt control | [Anthropic client SDK](../tools/anthropic-client-sdk.md) |
| Claude-only models | Multi-provider pipelines | [LangGraph](../tools/langgraph.md) / [LangChain](../tools/langchain.md) |

The most common reason to reach outside the SDK is the need for **explicit workflow control**. The open-ended agentic loop is the SDK's defining feature and its primary constraint. When the model should decide how to proceed, use the SDK. When your code must decide, use LangGraph.

The second most common reason is **startup latency** for request/response APIs: the Anthropic client SDK with a hand-rolled loop avoids the subprocess overhead entirely, at the cost of reimplementing what the Agent SDK provides.

## See Also

- [Claude Agent SDK](../tools/claude-agent-sdk.md)
- [How do you move an agentic Claude Code workflow to a programmable server environment?](claude-code-to-sdk.md)
- [Anthropic client SDK](../tools/anthropic-client-sdk.md)
- [LangGraph](../tools/langgraph.md)
- [AutoGen](../tools/autogen.md)
- [Multi-Agent Coordination](../concepts/multi-agent.md)
- [Agentic Loop](../concepts/agentic-loop.md)
