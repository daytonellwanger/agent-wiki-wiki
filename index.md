# Index

## Concepts

- [Agentic Loop](concepts/agentic-loop.md) — The fundamental perceive-reason-act cycle that all agents run
- [Tool Use](concepts/tool-use.md) — How agents invoke external tools and APIs (function calling, MCP)
- [Memory](concepts/memory.md) — Mechanisms for retaining information across turns and sessions
- [Planning](concepts/planning.md) — How agents decompose goals and reason about multi-step tasks
- [Multi-Agent Coordination](concepts/multi-agent.md) — Architectures where multiple agents collaborate
- [Context Management](concepts/context-management.md) — Managing context window limits, compaction, and retrieval
- [Progressive Disclosure](concepts/progressive-disclosure.md) — Revealing complexity incrementally to the context window
- [Computer Use](concepts/computer-use.md) — Agents that interact with GUIs and desktop environments
- [Evaluation](concepts/evaluation.md) — Measuring agent performance: trajectories, outcomes, and benchmarks

## Tools

- [Model Context Protocol (MCP)](tools/mcp.md) — Open protocol for connecting tools and resources to LLMs
- [Claude Code](tools/claude-code.md) — Anthropic's agentic coding CLI and IDE extension
- [Claude Code Subagents](tools/claude-code-subagents.md) — The Agent tool system for spawning specialized child agents within a Claude Code session
- [Claude Agent SDK](tools/claude-agent-sdk.md) — Python/TypeScript SDK for running Claude Code's agent loop programmatically in headless or server contexts
- [Managed Agents](tools/claude-managed-agents.md) — Anthropic-hosted REST API for running agents without operating container infrastructure
- [OpenAI Agents SDK](tools/openai-agents-sdk.md) — OpenAI's Python SDK for building agents with tools and handoffs
- [LangGraph](tools/langgraph.md) — Graph-based stateful orchestration framework from LangChain
- [LangChain](tools/langchain.md) — Foundational LLM application framework with broad integrations
- [AutoGen](tools/autogen.md) — Microsoft's conversational multi-agent framework
- [CrewAI](tools/crew-ai.md) — Role-based multi-agent framework organized around crew personas
- [Anthropic Client SDK](tools/anthropic-client-sdk.md) — Low-level Python/TypeScript library for direct Anthropic Messages API access
- [Pi](tools/pi.md) — Minimalist open-source coding agent harness prioritizing token efficiency and operator control

## Questions

- [How do you move an agentic Claude Code workflow to a programmable server environment?](questions/claude-code-to-sdk.md) — Translating CLAUDE.md, subagents, and skills to the Claude Agent SDK or Managed Agents
- [When does the Claude Agent SDK fall short, and what are the alternatives?](questions/claude-agent-sdk-limits.md) — Structural limits of the SDK's agentic loop and what frameworks fill the gaps
- [When should you use a subagent rather than having CLAUDE.md reference a separate instruction file?](questions/subagent-vs-claude-md.md) — Both load instructions on demand; subagents add context isolation, tool restrictions, model selection, and parallelism
