# Zot

Zot is a minimal coding agent harness written in Go, distributed as a single static binary. Built by patriceckhart and released in May 2026 as open source, Zot targets developers who want a lightweight terminal agent with no runtime dependencies and broad LLM provider support. The author's stated motivation was to understand harness mechanics and build in a language they prefer — not to compete with established tools.

**Site:** [zot.sh](https://www.zot.sh)

## What It Is

Zot is a coding agent harness in the same category as [Pi](pi.md): it runs from the terminal, automates software development tasks via a thin [agentic loop](../concepts/agentic-loop.md), and is intentionally lightweight. Its distinguishing characteristics are:

- **Single Go binary**: no Docker, no Node runtime, no Python environment. The deployment surface is one file.
- **30+ provider support**: Anthropic, OpenAI, Google, DeepSeek, and local Ollama instances, with dynamic provider discovery rather than static configuration.
- **User-owned credentials**: no intermediary gateway; API keys go directly from user to provider.

Four built-in tools cover the core coding-agent surface:

1. `read` — file and directory reading
2. `write` — file creation
3. `edit` — search/replace editing
4. `bash` — shell command execution

Sessions are stored as JSONL transcripts, enabling portability, resumption, and branching.

## Execution Modes

Zot supports four distinct execution patterns:

- **Interactive TUI** — standard terminal UI for interactive use
- **Print mode** — pipeline-friendly output for scripting
- **JSON output** — structured output for programmatic consumers
- **RPC mode** — embeds the agent loop in other applications via JSON-RPC subprocess protocol

The RPC mode is unusual among open-source coding agent harnesses and makes Zot usable as a library or subprocess component rather than only as a standalone CLI.

## Key Features

**Session management.** Users can resume sessions with `-c`, fork branches at any past message, and auto-compact transcripts at 85% context utilization — comparable to Pi's append-only session DAG.

**Swarm.** Background subagents working simultaneously on independent tasks within the same session. This is Zot's built-in answer to [parallel agent coordination](../concepts/multi-agent.md) without requiring an external orchestration layer.

**Extensions.** Custom slash commands and tools register via JSON-RPC subprocess protocol — no core binary modification required.

**Jail mode.** Confines tool execution to the current directory, providing a lightweight filesystem boundary. `--no-yolo` adds confirmation prompts before tool execution.

## Design Philosophy

Zot makes similar bets to Pi: minimal dependencies, thin scaffolding, and user control over provider and credentials. The Go implementation is a deliberate choice rather than a default — several users in the HN launch thread explicitly cited TypeScript and JavaScript ecosystem complexity (supply-chain risks, heavy dependency trees) as a reason to prefer a compiled Go binary.

The creator's framing was explicitly not competitive: Pi (built by Mario Zechner) was cited as "quite possibly the best OSS tool." Zot exists as a different implementation of similar principles in a different language.

## Community Reception

At launch (May 2026), Zot received 61 points and 60 comments on HN. One early user (airbreather) described it as "best agent I have used so far by a country mile" and reported 3–5x productivity gains versus alternatives, praising fast startup ("no gateway fuckaround") and extensibility (added Gmail and web-browsing capabilities as custom extensions). The author addressed bug reports and feature requests (command history navigation) in the thread itself.

The discussion also surfaced a terms-of-service concern: Zot reportedly uses a flag that spoofs Claude Code requests, which some commenters noted may violate Anthropic's API terms and will break once Anthropic implements request signing. This is worth monitoring for users relying on Claude models.

## Relation to Alternatives

- **vs. [Pi](pi.md):** Both are minimal coding agent harnesses with multi-provider support and thin scaffolding. Pi is TypeScript, has a richer extension ecosystem (2,000+ community packages), and is backed by startup Earendil. Zot is Go, ships as a single binary, and has an RPC mode Pi lacks. Pi has no built-in multi-agent support (extensions required); Zot has a built-in swarm.
- **vs. [Claude Code](claude-code.md):** Claude Code is a productized Anthropic-native environment with IDE integration, hooks as middleware, and scheduled tasks. Zot is a vendor-neutral harness with no enterprise features and no lock-in.
- **vs. [Superset](superset.md):** Superset is an orchestration layer on top of CLI agents. Zot's swarm feature handles parallelism internally rather than requiring an external coordinator.

## See Also

- [Agentic Loop](../concepts/agentic-loop.md) — The perceive-reason-act cycle Zot implements
- [Tool Use](../concepts/tool-use.md) — How Zot's four built-in tools fit the broader pattern
- [Context Management](../concepts/context-management.md) — Session compaction and branching
- [Multi-Agent Coordination](../concepts/multi-agent.md) — Zot's swarm feature in context
- [Pi](pi.md) — The closest alternative: similar philosophy, TypeScript instead of Go
- [Claude Code](claude-code.md) — The productized coding agent Zot is most often compared to
