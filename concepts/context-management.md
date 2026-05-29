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

A concrete example of retrieval-based output management for code: [Semble](../tools/semble.md) replaces the common grep-then-read-whole-file pattern with a hybrid semantic/BM25 search that returns only the relevant code snippets. At 2k tokens it achieves 94% recall; the grep+read baseline requires ~100k tokens for equivalent recall.

## Domain-Specific Semantic Context Layers

For agents operating in structured knowledge domains — analytics, data warehouses, legal, medical — the challenge is not only how much to load but what kind of content to load. Raw database schemas tell an agent the shape of data, not its meaning. An agent asked to write SQL can produce syntactically valid queries that return wrong answers because it doesn't know which metric definition is canonical, which tables should be joined and which will produce fan or chasm traps, or what business rules govern a particular dimension.

A **semantic context layer** addresses this by pre-structuring domain knowledge into a form that agents can query rather than infer. In analytics contexts this means: approved metric definitions (reusable canonical SQL replacing ad-hoc recalculation), joinable-column graphs with explicit trap annotations, and a business wiki of rules and context that narrows the interpretive space before the agent writes a single query. The agent queries the semantic layer on demand — "what does 'revenue' mean here?", "which tables join safely?" — rather than loading all of it upfront or reverse-engineering semantics from schema alone.

This is [Progressive Disclosure](progressive-disclosure.md) applied to domain knowledge: structured facts load first, full narrative context only when needed. A practical heuristic from production deployments: **tiered retrieval — structured facts first, full text only when needed** — works better than loading the complete business wiki at the start of each query.

Ktx (open-source, [github.com/Kaelio/ktx](https://github.com/Kaelio/ktx)) is a concrete implementation of this pattern for data warehouse agents. It ingests from dbt, LookML, and Metabase to build a unified semantic layer, exposes it via MCP tools and CLI, detects contradictions across sources, and runs read-only by design. The concept generalizes: any agent operating in a domain with formal semantics (approved definitions, known joinability constraints, documented business rules) benefits from this layer rather than attempting to infer semantics at query time.

## Context Window Sizes (as of 2025)

Leading models have context windows of 128k–1M+ tokens. While large windows reduce the frequency of compaction events, they don't eliminate the need for context management: performance degrades in very long contexts, and cost scales with tokens.

## See Also

- [Memory](memory.md)
- [Agentic Loop](agentic-loop.md)
- [Evaluation](evaluation.md)
- [Progressive Disclosure](progressive-disclosure.md) — Architectural pattern for loading agent context and capabilities on demand rather than all at once
- [Semble](../tools/semble.md) — Code search tool that returns ranked snippets instead of whole files, keeping coding-agent context lean
