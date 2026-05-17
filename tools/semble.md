# Semble

Semble is an open-source code search library designed for AI agents. Its core claim: return the right code snippets rather than entire files, consuming ~98% fewer tokens than a grep-then-read workflow while matching the retrieval accuracy of much heavier approaches.

**GitHub:** [MinishLab/semble](https://github.com/MinishLab/semble)

## What It Is

Semble is a **retrieval tool for coding agents**, not an agent framework. It indexes a local directory or remote git repository and answers natural language queries with ranked code snippets. The agent receives only the relevant fragments, not entire files, which keeps the [context window](../concepts/context-management.md) lean.

The token efficiency claim is scoped: the baseline is "grep + read all matching files." Grep alone uses no tokens. The savings come from eliminating the subsequent file-read step that agents currently rely on to get useful context around a match. At 2k tokens, Semble reaches 94% recall; grep + read requires ~100k tokens for equivalent recall.

## Technical Approach

Semble uses a two-stage hybrid retrieval pipeline:

1. **Dual retrievers**: Semantic search via static embeddings (Model2Vec, using the `potion-code-16M` model) combined with BM25 lexical search for identifier matching.
2. **Fusion and reranking**: Reciprocal Rank Fusion merges the two result sets, then code-aware signals adjust rankings:
   - Adaptive weighting based on query type (keyword-heavy vs. conceptual)
   - Definition prioritization (function/class definitions ranked above usage sites)
   - Identifier stem matching
   - File coherence boosting (results from the same file cluster together)
   - Noise penalties for test files and legacy code

Static embeddings (via Model2Vec) are the key design choice for speed: no GPU required, no external API calls, and no model warm-up.

## Benchmarks

Evaluated against 1,250 queries across 63 repositories:

| System | NDCG@10 | Index time | Query time |
|---|---|---|---|
| CodeRankEmbed Hybrid | 0.862 | 57s | 16ms |
| **Semble** | **0.854** | **263ms** | **1.5ms** |

Semble indexes 218x faster than the leading code-specialized transformer and queries 11x faster, at a 0.9% accuracy cost.

## Integration

**As an MCP server** — works with Claude Code, Cursor, Codex, and OpenCode:
```
claude mcp add semble -s user -- uvx --from "semble[mcp]" semble
```

**As a CLI tool:**
```
semble search "authentication flow" ./my-project
semble search "save_pretrained" https://github.com/MinishLab/model2vec
```

**As a Python library:**
```python
from semble import SembleIndex
index = SembleIndex.from_path("./my-project")
results = index.search("save model to disk", top_k=3)
```

**Without MCP** — the tool can be referenced in `AGENTS.md` or `CLAUDE.md` as a bash command, making it accessible to agents that do not use MCP.

Text files (markdown, JSON) can be indexed alongside code using the `--include-text-files` flag.

## Limitations and Open Questions

- **Benchmarks measure retrieval accuracy only (NDCG), not end-to-end coding task performance.** Whether token savings translate to better or faster task completion in a full agent loop has not been independently measured. Agents trained heavily on grep-based workflows may distrust alternative tools and retry queries, negating the savings.
- **Token measurement assumes single-shot retrieval.** The 98% figure assumes the relevant snippet is found on the first query. Real agents issue multiple queries with varying parameters; the actual token savings depend on agent behavior.
- **No end-to-end agent benchmark yet.** The authors acknowledged this is on their roadmap.

## Relation to Alternatives

- **vs. grep + cat:** The traditional pattern for coding agents. Simple and universally trusted by models, but returns whole files, inflating context. Semble is a drop-in alternative for query-based lookups.
- **vs. full embedding pipelines (CodeRankEmbed, etc.):** Higher accuracy but 218x slower to index, 11x slower to query, require GPU or external API. Semble's static embeddings are the practical tradeoff for CPU-only local use.

## See Also

- [Context Management](../concepts/context-management.md) — The broader problem Semble addresses
- [Tool Use](../concepts/tool-use.md) — How Semble fits the agent tool-use pattern
- [Model Context Protocol (MCP)](mcp.md) — The protocol Semble uses for host integration
