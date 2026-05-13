# Managed Agents

Managed Agents is a hosted REST API from Anthropic (currently in beta) that lets you run Claude-powered agents without operating any infrastructure. Anthropic provisions and manages the sandbox; you define agents and environments via REST, send tasks as events, and receive results over server-sent events (SSE).

## What Problem It Solves

The [Claude Agent SDK](claude-agent-sdk.md) runs the agent loop in your own process or container — you manage the runtime, the filesystem, the sandbox, and session storage. Managed Agents offloads all of that to Anthropic-hosted infrastructure. The tradeoff is less control: filesystem-based configuration (CLAUDE.md, `.claude/agents/`) does not apply, and agent definitions are expressed as REST payloads rather than markdown files.

## How It Works

Three core REST resources:

- **Agent definitions** — the agent's system prompt, tools, and behavior (analogous to a subagent definition file, but expressed as a JSON payload).
- **Environments** — container templates that define the execution sandbox (operating system, dependencies, disk image).
- **Sessions** — one session per task. You send events to a session and receive SSE in return.

When Claude needs a tool executed, it emits a tool-call event over SSE. Your application executes the tool and sends the result back as an event. This means **you own tool execution** — Claude triggers, you run, you return. Built-in tools (Read, Edit, Bash) are not automatically available the way they are in the Agent SDK.

Authentication requires the `managed-agents-2026-04-01` beta header on API requests.

## Key Differences from the Claude Agent SDK

| | Agent SDK | Managed Agents |
|---|---|---|
| Runs in | Your process / container | Anthropic-managed sandbox |
| Interface | Python/TS `query()` | REST API + SSE |
| Agent config | CLAUDE.md, `.claude/agents/` files, or programmatic | REST payload (JSON) |
| Tool execution | SDK executes built-in tools automatically | Claude triggers; you execute and return |
| Session state | JSONL on your filesystem | Anthropic-hosted event log |
| Infrastructure | You operate and scale | Anthropic operates |
| Status | Generally available | Beta |

## Recommended Use

Anthropic's suggested path: prototype locally with the Agent SDK (where filesystem conventions carry over from interactive Claude Code), then migrate to Managed Agents for production if you don't want to operate container infrastructure.

Managed Agents is a better fit when:
- You want to avoid managing sandboxes, container fleets, or session storage.
- Your agent definition is relatively stable (not depending on checked-in CLAUDE.md or `.claude/agents/` files).
- You are comfortable with a REST-based, tool-triggering model rather than an autonomous built-in-tools model.

The Agent SDK remains a better fit when:
- You are translating an existing Claude Code project that already uses CLAUDE.md and `.claude/agents/`.
- You need fine-grained control over tool execution environment (custom filesystems, local databases, etc.).
- You want to run agents on your own infrastructure for compliance or data-residency reasons.

## See Also

- [Claude Agent SDK](claude-agent-sdk.md)
- [Claude Code](claude-code.md)
- [Agentic Loop](../concepts/agentic-loop.md)
- [Tool Use](../concepts/tool-use.md)
