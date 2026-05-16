# Context Management

Context management is the challenge of keeping the agent's context window populated with the right information — and not too much of it. As agents run longer tasks, context fills up with tool results, conversation history, and intermediate reasoning, and decisions must be made about what to keep.

## Why It Matters

Every token in context costs money and latency. More importantly, large contexts degrade model performance: models can lose track of information buried deep in a long context, and the most relevant information may be drowned out by noise. Context management is thus both an economic and a quality concern.

## Strategies

### Truncation

Simply drop the oldest messages when the context exceeds a threshold. Fast and simple, but loses potentially important history.

### Summarization (Compaction)

Summarize older portions of the conversation or tool output history into a compact representation, then replace the raw history with the summary. Retains more signal than truncation but requires an additional model call to generate the summary.

Claude Code uses a compaction approach: when the context window fills, a summary of the conversation is generated and substituted for the raw history.

### Retrieval-Augmented Memory

Instead of keeping everything in context, store history externally and retrieve relevant pieces on demand. This scales to arbitrarily long histories but introduces retrieval latency and the risk of missing important context. See [Memory](memory.md).

### Structured State

Some agent frameworks maintain a structured state object (e.g., a typed dataclass or JSON blob) that is updated and summarized at each step, rather than relying on the raw message history. This is more predictable and compact but requires upfront schema design.

## Tool Output Management

Tool calls can return very large outputs (e.g., a full file, a large API response). Good practice:
- Truncate or summarize large tool outputs before adding them to context.
- For file content, include only the relevant sections.
- Store large outputs externally and give the agent a reference it can use to retrieve specific parts.

## Context Window Sizes (as of 2025)

Leading models have context windows of 128k–1M+ tokens. While large windows reduce the frequency of compaction events, they don't eliminate the need for context management: performance degrades in very long contexts, and cost scales with tokens.

## See Also

- [Memory](memory.md)
- [Agentic Loop](agentic-loop.md)
- [Evaluation](evaluation.md)
- [Progressive Disclosure](progressive-disclosure.md) — Architectural pattern for loading agent context and capabilities on demand rather than all at once
