# Log

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
