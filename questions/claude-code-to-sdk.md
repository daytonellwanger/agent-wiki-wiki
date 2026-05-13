# How do you move an agentic Claude Code workflow to a programmable server environment?

You've built an agentic flow in Claude Code — a CLAUDE.md with project instructions, custom subagents in `.claude/agents/`, skills in `.claude/skills/`, and hooks wired up in `.claude/settings.json`. It works interactively. Now you want to invoke that same flow programmatically: from a server, an API handler, a CI job, or any headless context. What's the translation?

## Background

[Claude Code](../tools/claude-code.md) surfaces its agentic behavior through a set of filesystem-based configuration primitives: CLAUDE.md files provide the model with project-specific context, [subagents](../tools/claude-code-subagents.md) in `.claude/agents/` define specialized child agents, skills in `.claude/skills/` package reusable workflows, and hooks in `.claude/settings.json` intercept tool calls at runtime. In interactive use, these are driven by a human in a terminal or IDE. Shifting to programmatic invocation means replacing that human-driven loop with code.

The key question is: does this configuration transfer, or do you have to rebuild it?

## Perspectives

### Use the Claude Agent SDK: the configuration transfers

The primary path is the **[Claude Agent SDK](../tools/claude-agent-sdk.md)** — a Python and TypeScript library that wraps the same Claude Code runtime and exposes it via a `query()` function. It intentionally mirrors the filesystem conventions: CLAUDE.md, `.claude/agents/`, and `.claude/skills/` load automatically (controlled by a `settingSources` option), so a project already configured for interactive Claude Code requires minimal changes to run programmatically.

What changes:
- The **agentic loop** shifts from CLI-managed to SDK-managed — you consume a typed message stream instead of watching terminal output.
- **Subagents** can be defined either by leaving `.claude/agents/` files in place or by passing them as `AgentDefinition` objects at runtime — useful when you need to inject prompts without touching the filesystem.
- **System prompt**: the full Claude Code system prompt is not the default in the SDK; you must opt in with `system_prompt={"type": "preset", "preset": "claude_code"}`.
- **Skills**: still filesystem-only (no programmatic registration), but the `skills` option controls which are active per call.

A minimal translation of an existing Claude Code project:

```python
async for message in query(
    prompt="...",
    options=ClaudeAgentOptions(
        cwd="/my-project",
        setting_sources=["project"],  # loads CLAUDE.md, subagents, skills, hooks
        system_prompt={"type": "preset", "preset": "claude_code"},
        allowed_tools=["Read", "Edit", "Bash", "Agent"],
        skills="all",
    )
):
    ...
```

The main constraint is startup overhead: because the SDK spawns the Claude Code binary as a subprocess, each `query()` call incurs ~12 seconds of process startup time. This matters for latency-sensitive request/response APIs.

### Use headless CLI (`claude -p`): no new dependencies

The simplest option requires no new dependencies — the same Claude Code CLI gains a non-interactive mode with the `-p` flag:

```bash
claude -p "Fix the bug in auth.py" --allowedTools "Read,Edit,Bash" --output-format stream-json
```

This is appropriate for CI/CD pipelines, one-shot automation scripts, and cases where all you need is structured output. The `--bare` flag disables CLAUDE.md, hooks, skill discovery, and auto-memory — recommended for deterministic scripted runs. There is no way to supply programmatic hooks or runtime agent definitions; everything must be in files or passed as prompt text.

### Use Managed Agents: skip the infrastructure

**[Managed Agents](../tools/claude-managed-agents.md)** is a hosted REST API (currently in beta) where Anthropic provisions and manages the sandbox. You define agents and environments via REST, send tasks as events, and receive results over SSE — without running any infrastructure. The filesystem conventions do not apply; agent configuration is expressed as REST payloads. This path is structurally different from translating a Claude Code project — it's closer to rebuilding the agent definition in a new API.

## Answer

The standard translation path is the **[Claude Agent SDK](../tools/claude-agent-sdk.md)**. It is purpose-built for this transition: it runs the same agent loop as Claude Code, respects the same filesystem conventions (CLAUDE.md, `.claude/agents/`, `.claude/skills/`), and adds programmatic control over every configuration surface.

The key gaps vs. interactive Claude Code to plan for:

- **~12s per-query startup** — the SDK spawns the CLI binary per call. For server deployments, the standard mitigation is keeping a warm container that handles multiple queries over its lifetime.
- **Skills are filesystem-only** — they cannot be injected at runtime; they must exist as `SKILL.md` files on disk.
- **Subagents cannot nest** — subagents cannot spawn their own subagents (one level of delegation only).
- **Agent teams are CLI-only** — multi-agent setups with direct peer-to-peer messaging are unavailable.
- **Multi-tenancy requires explicit isolation** — by default, `query()` loads user-level settings and auto-memory from the host filesystem, which can leak between tenants. Use `setting_sources=[]` and `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1` for isolated multi-tenant deployments.

For production deployments where you don't want to operate container infrastructure, the alternative is [Managed Agents](../tools/claude-managed-agents.md). The tradeoff: you lose filesystem-based configuration and rebuild agent definitions as REST payloads.

For CI/CD or simple one-shot automation, `claude -p` is simpler than the SDK and requires no additional code.

## See Also

- [Claude Agent SDK](../tools/claude-agent-sdk.md)
- [Managed Agents](../tools/claude-managed-agents.md)
- [Claude Code Subagents](../tools/claude-code-subagents.md)
- [Claude Code](../tools/claude-code.md)
- [Agentic Loop](../concepts/agentic-loop.md)
- [Multi-Agent Coordination](../concepts/multi-agent.md)
