# Log

## [2026-05-17] — Input quality as a planning and evaluation prerequisite

**Source:** HN post "I don't think AI will make your processes go faster" (https://news.ycombinator.com/item?id=48168221); article at frederickvanbrabant.com/blog/2026-05-15-i-dont-think-ai-will-make-your-processes-go-faster/. Score: 411, ~300 comments.

**Technical:** Pages updated: concepts/planning.md (new "Input Quality Is a Prerequisite" section), concepts/evaluation.md (Practical Guidance section expanded with two new bullets).

**Summary:** A widely-upvoted piece arguing that AI doesn't make organizational processes faster because the bottleneck is rarely code execution speed — it's upstream ambiguity (unclear requirements, vague specs, organizational coordination). Drawing on the Theory of Constraints, the author's point is that speeding up a non-bottleneck step has diminishing returns; you need "predictable, high-quality inputs" at the actual constraint. The HN discussion broadly confirmed this, with practitioners noting that senior engineering time is dominated by coordination and buy-in, not coding; that "AI is unveiling how the bureaucracy is the slow part"; and that Amdahl's Law makes optimizing the fast part increasingly irrelevant as the slow parts dominate. Added a new "Input Quality Is a Prerequisite" section to the Planning page: vague goals produce vague agent plans regardless of model quality, and the human-facing task specification interface is often more important than any architectural choice inside the agent. Also sharpened the Evaluation practical guidance to make the connection explicit — the inability to write a clear eval is often a signal that the task itself is underspecified, not that evaluation is inherently hard.

## [2026-05-17] — Semble: code search for agents

**Source:** Show HN: Semble – Code search for agents that uses 98% fewer tokens than grep (https://news.ycombinator.com/item?id=48169874); GitHub repo MinishLab/semble.

**Technical:** Pages created: tools/semble.md (new). Pages updated: index.md (Semble added under Tools), concepts/context-management.md (Semble added as concrete example in Tool Output Management section and in See Also).

**Summary:** Added a tool page for Semble, an open-source Python library that gives coding agents retrieval-based code search as an alternative to grep-then-read-whole-file. The core technique is a two-stage hybrid pipeline: static embeddings (Model2Vec, CPU-only) for semantic matching combined with BM25 for identifier matching, fused via Reciprocal Rank Fusion and reranked with code-aware signals (definition prioritization, identifier stem matching, file coherence boosting, noise penalties for tests/legacy). Against 1,250 queries across 63 repositories it achieves NDCG@10 of 0.854 vs. 0.862 for the leading code-specialized transformer, while indexing 218x faster (263ms vs. 57s) and querying 11x faster (1.5ms vs. 16ms). The token efficiency claim — 94% recall at 2k tokens vs. ~100k tokens for grep+read — is for retrieval only; the HN discussion surfaced that end-to-end agent benchmarks do not yet exist and real savings depend on agent query behavior. Semble ships as an MCP server (one command to add to Claude Code, Cursor, Codex, OpenCode) or as a CLI/Python library. Added a concrete example of retrieval-based output management to the context-management page.

## [2026-05-17] — The agentic loop is simple

**Technical:** Pages created: opinions/the-agentic-loop-is-simple.md (new). Pages updated: index.md (new Opinions section with first entry).

**Summary:** Added the first opinion page: the agentic loop itself is simple. Prompted by Amp's published walkthrough showing a functional code-editing agent in under 400 lines. The core claim is that the mechanism — an LLM, a loop, and a tool-calling protocol — is not complex; it is a communication convention between model and executor. The genuine engineering difficulty in production agents (editor integration, system prompt tuning, multi-agent coordination, latency, reliability) lives at the product layer, not in the loop. The counterarguments acknowledge that simple to implement is not simple to get right, and that tool interface design involves real judgment even when the loop itself is straightforward.

## [2026-05-16] — Subagent vs. referenced instruction file

**Technical:** Pages created: questions/subagent-vs-claude-md.md (new). Pages updated: index.md (new question entry).

**Summary:** Added a new question: when should you use a subagent rather than having CLAUDE.md reference a separate markdown instruction file that Claude reads on demand? Both approaches achieve progressive disclosure — neither loads instructions until they're needed. The real differentiators are context isolation (a subagent's intermediate tool calls stay out of the main thread; a referenced file's work accumulates there), enforceability (a subagent can restrict its tool allowlist; a referenced file cannot), model selection (subagents can run on a cheaper or more powerful model), and parallelism (subagents can run concurrently; referenced file work is sequential). The referenced-file approach has its own advantages: simpler, no delegation boundary, and the parent sees all intermediate results — which makes it better when iterative refinement is expected. The practical frame: a referenced markdown file is an instruction Claude reads and follows; a subagent is a capability Claude delegates to.

## [2026-05-16] — Progressive Disclosure

**Technical:** Pages created: concepts/progressive-disclosure.md (new). Pages updated: index.md (new Progressive Disclosure entry under Concepts), concepts/context-management.md (Progressive Disclosure added to See Also), tools/claude-code.md (Progressive Disclosure added to See Also).

**Summary:** Added a concept page for Progressive Disclosure — the architectural pattern for loading agent context and capabilities on demand rather than all at once. The core problem it addresses is context bloat: agents that load all tools and documentation upfront face attention dilution, instruction interference, and tool schema bloat (50,000+ tokens of JSON before reasoning begins). The page covers three main patterns: the Agent Skills three-tier architecture (metadata always loaded, full instructions only on activation, resources only when referenced), which cuts token consumption by ~85%; index-first loading, where an agent receives a structured index and fetches only relevant files; and phase-based loading, where context is swapped per task phase rather than accumulated. Key tradeoffs: implicit activation is unreliable without explicit fallback instructions (44% activation rate in Vercel evals), and on-demand content loading is a prompt-injection vector.

## [2026-05-16]

**Technical:** Pages created: tools/pi.md (new). Pages updated: index.md (new Pi entry under Tools), tools/claude-code.md (Pi added to See Also).

**Summary:** Added a tool page for Pi, an open-source coding agent harness built by Mario Zechner and released in late 2025. Pi takes a deliberate minimalist position: a system prompt under 1,000 tokens, four built-in tools (read/write/edit/bash), no MCP, no sub-agents, no plan mode — all by design. Its core bet is that RL-trained frontier models need thin scaffolding, not thick orchestration, and that the context window is the real bottleneck. Pi supports 15+ LLM providers through a unified API layer and allows self-extension via runtime-compiled TypeScript hooks. The framework gained wide visibility in January 2026 when OpenClaw (a multi-platform communication agent built on Pi's SDK) went viral. In April 2026, startup Earendil acquired Pi and launched Lefos, a cloud platform built on top of it. Pi occupies a distinct niche from Claude Code (productized, Anthropic-native) and LangGraph (Python, state-machine orchestration) — it is a hackable harness for teams that want multi-provider support and full control over agent behavior.

## [2026-05-13]

**Technical:** Pages created: questions/claude-agent-sdk-limits.md (new), tools/anthropic-client-sdk.md (new). Pages updated: index.md (new question entry, new Anthropic Client SDK tool entry).

**Summary:** Added a new question: when does the Claude Agent SDK fall short, and what are the alternatives? The answer identifies four structural constraints: (1) the open-ended agentic loop is wrong for auditable, deterministic pipelines — LangGraph is the alternative; (2) one-level subagent nesting rules out deep hierarchies or peer-based coordination — LangGraph subgraphs or AutoGen fill the gap; (3) ~12s-per-query subprocess startup plus opaque model calls make the SDK unsuitable for low-latency APIs or fine-grained prompt control — the Anthropic client SDK (direct Messages API) is the right tool there; (4) Claude-only model support rules out multi-provider pipelines — LangGraph/LangChain handle those. Added a new Anthropic Client SDK tool page to cover the direct Messages API library, which the wiki previously described only in contrast to the Agent SDK.

## [2026-05-11]

**Source:** Official Claude Agent SDK documentation at code.claude.com/docs/en/agent-sdk; Managed Agents documentation at platform.claude.com/docs/en/managed-agents; Claude Code headless documentation at code.claude.com/docs/en/headless.

**Technical:** Pages created: questions/claude-code-to-sdk.md (new), tools/claude-agent-sdk.md (new), tools/claude-managed-agents.md (new). Pages updated: index.md (three new entries under Tools, new Questions section populated).

**Summary:** Added a new question: how do you move an agentic Claude Code workflow to a server/programmatic context? The answer centers on the Claude Agent SDK — a Python/TypeScript library that wraps the Claude Code runtime and exposes it via a `query()` async iterator. The SDK intentionally mirrors Claude Code's filesystem conventions (CLAUDE.md, `.claude/agents/`, `.claude/skills/`), so an existing project largely carries over. The key differences: subagents can also be defined programmatically at runtime, the full Claude Code system prompt must be opted into explicitly, and each `query()` call incurs ~12 seconds of subprocess startup overhead. For production deployments without managing container infrastructure, Managed Agents is a hosted REST alternative — though it does not support filesystem-based configuration and requires rebuilding agent definitions as REST payloads. For simple CI/CD scripting, `claude -p` (headless CLI) requires no new code at all. Two new tool pages cover the SDK and Managed Agents in depth.

## [2026-05-06]

**Source:** Official Claude Code documentation at code.claude.com/docs/en/sub-agents and code.claude.com/docs/en/agent-sdk/subagents; official agent teams documentation at code.claude.com/docs/en/agent-teams.

**Technical:** Pages created: tools/claude-code-subagents.md (new). Pages updated: tools/claude-code.md (subagents bullet expanded, See Also extended), concepts/multi-agent.md (Claude Code Subagents linked in Orchestrator + Subagents section, See Also extended), index.md (new Claude Code Subagents entry added).

**Summary:** Claude Code has a fully documented, production-grade subagent system built around the Agent tool (renamed from Task tool in v2.1.63). The main session can delegate subtasks to child agents that each run in an isolated context window, keeping verbose intermediate output out of the main conversation. Custom subagents are defined as Markdown files with YAML frontmatter and can be scoped to a project (checked into version control), a user, an org, or a single session. The description field is the primary mechanism: Claude reads it to decide when to delegate automatically. Notable capabilities include per-subagent model selection (route exploration to Haiku, heavy analysis to Opus), tool allowlists/denylists, permission mode overrides, persistent memory across sessions, lifecycle hooks, MCP server scoping, and worktree isolation for parallel file edits. Foreground and background execution modes let the user continue working while a subagent runs. A newer experimental "fork" mode (v2.1.117+) lets a subagent inherit the full parent conversation history — trading context isolation for continuity, and reusing the parent's prompt cache to reduce cost. Agent teams, a separate experimental feature, extends this with peer-to-peer messaging and a shared task list for workflows requiring inter-agent coordination; subagents remain the right tool for focused, self-contained delegation.

## [2026-05-04]

**Source:** Initial population — seeded from model knowledge (knowledge cutoff: August 2025)

**Technical:** Pages created: overview.md, index.md, concepts/agentic-loop.md (new), concepts/tool-use.md (new), concepts/memory.md (new), concepts/planning.md (new), concepts/multi-agent.md (new), concepts/context-management.md (new), concepts/computer-use.md (new), concepts/evaluation.md (new), tools/mcp.md (new), tools/claude-code.md (new), tools/openai-agents-sdk.md (new), tools/langgraph.md (new), tools/langchain.md (new), tools/autogen.md (new), tools/crew-ai.md (new).

**Summary:** Initial wiki population covering the state of AI agents as of mid-2025. The core agent loop is settled: LLM + tools in a loop, with ReAct-style reasoning. The major open questions are memory architecture at scale, reliable long-horizon planning, evaluation methodology, and multi-agent coordination overhead. MCP has emerged as a meaningful standard for tool connectivity, and computer use (GUI-operating agents) is a fast-moving frontier. The dominant frameworks divide into: graph-based orchestration (LangGraph), conversation-based multi-agent (AutoGen), role-based multi-agent (CrewAI), and provider-native (OpenAI Agents SDK, Claude Code). Evaluation remains the hardest unsolved problem — no widely-adopted standard for scoring agent trajectories exists.
