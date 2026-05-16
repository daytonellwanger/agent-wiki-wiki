# Pi

Pi is a minimalist, open-source coding agent harness written in TypeScript. Built by Mario Zechner (creator of libGDX) and released in late 2025, Pi prioritizes token efficiency and operator control over out-of-the-box features. Its core premise: frontier models trained with reinforcement learning need minimal scaffolding — extensive system prompts and framework overhead are a workaround for weaker models, not a requirement for capable ones.

The framework gained wide attention in January 2026 when OpenClaw, a multi-platform communication agent built on Pi's SDK, reached 145,000 GitHub stars in a single week. In April 2026, startup Earendil acquired Pi from Zechner and launched Lefos, a commercial cloud platform built on top of it.

**GitHub:** [badlogic/pi-mono](https://github.com/badlogic/pi-mono) · [earendil-works/pi](https://github.com/earendil-works/pi) (Earendil fork) · **Site:** [pi.dev](https://pi.dev/)

## What It Is

Pi is a **coding agent harness** — not a general-purpose orchestration framework. It runs from the terminal and automates software development tasks using a thin [agentic loop](../concepts/agentic-loop.md): stream LLM response → execute tools → append results → repeat. There is no built-in plan tracking, to-do system, or step limit; the model reasons about what to do next.

The system prompt ships at under 1,000 tokens (sometimes as few as ~200). Four built-in tools cover the core coding-agent surface:

1. `read` — files, images, directories, glob patterns, with line-range support
2. `write` — creates files and parent directories
3. `edit` — exact-string search/replace with unified diff output
4. `bash` — shell command execution with timeout handling

## Architecture

Pi is a TypeScript monorepo with enforced layered dependencies:

- **pi-ai** — Unified LLM API layer. Normalizes across four wire protocols (OpenAI Completions, OpenAI Responses, Anthropic Messages, Google Generative AI) and supports 15+ providers including Anthropic, OpenAI, Google, Azure, xAI, Groq, Ollama, and any OpenAI-compatible endpoint. Supports mid-session model switching and cross-provider context handoffs.
- **pi-agent-core** — The agent loop itself. Intentionally minimal.
- **pi-coding-agent** — The full runtime CLI: JSONL session persistence, context compaction, skills system, and extensions.
- **pi-tui** — Terminal UI with differential rendering and markdown display.

**Sessions** use an append-only DAG stored in `.jsonl` files. Each entry has a unique ID and optional parent ID, enabling session branching, history preservation, and cross-provider model switching mid-session.

**Extensions** provide 20+ lifecycle hooks: modify messages before they reach the LLM, block [tool calls](../concepts/tool-use.md), add custom tools and slash commands, register keyboard shortcuts, persist state into sessions. Extensions load via runtime TypeScript compilation — no pre-compilation needed. A community extension catalog had over 2,000 packages as of early 2026.

**Context engineering** uses progressive skill disclosure: skill content loads only when relevant triggers match, rather than dumping everything into the [context window](../concepts/context-management.md) upfront. Pi deliberately excludes MCP support — the author cites MCP servers that can add 13,700+ tokens to the context window as a concrete example of what Pi is designed to avoid.

## Design Philosophy

Pi makes a deliberate bet: the context window is the real bottleneck, and most framework overhead is counterproductive. Consequences:

- Minimal system prompt — no behavior encoded in tokens the model doesn't need
- No MCP, no sub-agents, no permission popups, no plan mode in the core — these can be added via extensions
- **Self-extension:** rather than installing pre-built tools, you ask the agent to write and load its own extensions, with hot reloading

The author's thesis (November 2025 blog post) is that RL-trained frontier models reason well enough that thin scaffolding outperforms thick scaffolding on coding tasks. Pi can be consumed as a full CLI or as individual packages — teams can take only the layers they need (e.g., just `pi-ai` for provider normalization) without adopting the full stack.

## Relation to Alternatives

- **vs. [Claude Code](claude-code.md):** Claude Code is a productized environment with editor integration, IDE plugins, MCP, hooks, and scheduled tasks. Pi is a hackable harness — multi-provider, no vendor-enforced behavior, no enterprise features out of the box.
- **vs. [LangGraph](langgraph.md):** LangGraph encodes workflow logic in a state machine; Pi delegates workflow judgment to the model. LangGraph is Python and general-purpose; Pi is TypeScript and coding-focused.
- **vs. [OpenAI Agents SDK](openai-agents-sdk.md):** Python-first and OpenAI-native. Pi is TypeScript, multi-provider, and treats context efficiency as a first principle.
- **vs. [AutoGen](autogen.md) / [CrewAI](crew-ai.md):** Those frameworks focus on [multi-agent coordination](../concepts/multi-agent.md) and role abstractions. Pi has no native sub-agent system; multi-agent patterns are built via extensions.

## See Also

- [Agentic Loop](../concepts/agentic-loop.md) — The perceive-reason-act cycle Pi implements
- [Tool Use](../concepts/tool-use.md) — How Pi's four built-in tools fit the broader pattern
- [Context Management](../concepts/context-management.md) — Pi's progressive skill disclosure and compaction approach
- [Claude Code](claude-code.md) — The productized coding agent Pi is most often compared to
- [LangGraph](langgraph.md) — Graph-based orchestration; a different philosophy on workflow encoding
- [OpenAI Agents SDK](openai-agents-sdk.md) — Python-native alternative from OpenAI
