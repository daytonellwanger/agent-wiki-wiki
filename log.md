# Log

## [2026-05-04]

**Source:** Initial population — seeded from model knowledge (knowledge cutoff: August 2025)

**Technical:** Pages created: overview.md, index.md, concepts/agentic-loop.md (new), concepts/tool-use.md (new), concepts/memory.md (new), concepts/planning.md (new), concepts/multi-agent.md (new), concepts/context-management.md (new), concepts/computer-use.md (new), concepts/evaluation.md (new), tools/mcp.md (new), tools/claude-code.md (new), tools/openai-agents-sdk.md (new), tools/langgraph.md (new), tools/langchain.md (new), tools/autogen.md (new), tools/crew-ai.md (new).

**Summary:** Initial wiki population covering the state of AI agents as of mid-2025. The core agent loop is settled: LLM + tools in a loop, with ReAct-style reasoning. The major open questions are memory architecture at scale, reliable long-horizon planning, evaluation methodology, and multi-agent coordination overhead. MCP has emerged as a meaningful standard for tool connectivity, and computer use (GUI-operating agents) is a fast-moving frontier. The dominant frameworks divide into: graph-based orchestration (LangGraph), conversation-based multi-agent (AutoGen), role-based multi-agent (CrewAI), and provider-native (OpenAI Agents SDK, Claude Code). Evaluation remains the hardest unsolved problem — no widely-adopted standard for scoring agent trajectories exists.
