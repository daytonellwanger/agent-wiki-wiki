# Anthropic Client SDK

The Anthropic client SDK (`anthropic` for Python, `@anthropic-ai/sdk` for TypeScript) is the official low-level library for calling the Anthropic API directly. It provides access to the Messages API, tool use, streaming, prompt caching, batch processing, and the Files API — with no agentic loop or built-in tool execution.

## Relationship to the Claude Agent SDK

The Anthropic client SDK and the [Claude Agent SDK](claude-agent-sdk.md) are distinct packages with different abstraction levels:

| | Anthropic Client SDK | Claude Agent SDK |
|---|---|---|
| What it does | Direct Messages API access | Full agentic loop with built-in tools |
| Tool execution | You implement the loop | SDK executes tools autonomously |
| State management | You manage context | SDK handles context + compaction |
| Interface | Request/response | Async message stream |
| Startup overhead | None | ~12s (subprocess spawn) |
| Model support | Any Anthropic model | Claude models via Claude Code binary |

## Core Usage

```python
import anthropic

client = anthropic.Anthropic()
message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello"}],
)
```

For an agentic loop, you implement the tool-call cycle yourself: send a request, check if the response contains tool calls, execute them, append results, and send again until the model produces a final text response.

## Key Features

- **Streaming**: `client.messages.stream()` yields typed events as the model generates output.
- **Prompt caching**: mark content blocks with `cache_control` to cache them across requests, reducing cost and latency on repeated prompts.
- **Batch API**: submit up to 10,000 requests asynchronously; results are available within 24 hours at roughly half the standard price.
- **Tool use**: pass tool definitions and handle `tool_use` content blocks in the response loop.
- **Extended thinking**: enable `thinking` blocks for complex reasoning tasks.
- **Files API**: upload files (PDFs, images) for reuse across requests without re-uploading.

## When to Use It

Use the Anthropic client SDK instead of the [Claude Agent SDK](claude-agent-sdk.md) when:

- **Startup latency matters**: the client SDK has no subprocess overhead; each API call takes only network round-trip time.
- **You need fine-grained control**: exact prompt construction, per-request model selection, specific caching strategies, or custom streaming handling.
- **The agentic loop is simpler than Claude Code's**: for tasks that need a short, deterministic tool loop rather than the full Claude Code runtime.
- **You are integrating with an existing orchestration layer** (e.g., [LangGraph](langgraph.md)) that manages the loop itself.

## See Also

- [Claude Agent SDK](claude-agent-sdk.md)
- [Tool Use](../concepts/tool-use.md)
- [Context Management](../concepts/context-management.md)
- [Managed Agents](claude-managed-agents.md)
