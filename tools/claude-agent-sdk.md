# Claude Agent SDK

The Claude Agent SDK is a Python and TypeScript library that exposes the same agent loop, built-in tools, and context management that power the [Claude Code](claude-code.md) CLI as a programmable, embeddable runtime. It is the primary path for running Claude Code-style agentic flows in headless, server-side, or automated contexts.

## Relationship to Other SDKs

The Claude Agent SDK is **distinct** from the standard Anthropic client SDK (`anthropic` / `@anthropic-ai/sdk`):

| | Anthropic Client SDK | Claude Agent SDK |
|---|---|---|
| What it does | Direct Messages API access | Full agentic loop with built-in tools |
| Tool execution | You implement your own loop | SDK executes tools autonomously |
| State management | You manage context | SDK handles context + compaction |
| Interface | Request/response | Async message stream |

The Agent SDK is architecturally a process-spawning wrapper around the Claude Code CLI binary. The Python SDK spawns the binary as a subprocess; the TypeScript SDK bundles it as an optional npm dependency.

**Packages**:
- Python: `pip install claude-agent-sdk`
- TypeScript: `npm install @anthropic-ai/claude-agent-sdk`

(The package was renamed from `claude-code-sdk` / `@anthropic-ai/claude-code` in mid-2025; the main options type was renamed `ClaudeCodeOptions` → `ClaudeAgentOptions`. No other code changes were required for migration.)

## Core API: `query()`

The entire SDK surface centers on `query()`, which returns an async iterator of typed message objects as the agent reasons, calls tools, receives results, and produces a final answer.

```python
from claude_agent_sdk import query, ClaudeAgentOptions

async for message in query(
    prompt="Find and fix the bug in auth.py",
    options=ClaudeAgentOptions(
        allowed_tools=["Read", "Edit", "Bash"],
        permission_mode="acceptEdits",
        cwd="/my-project",
    )
):
    print(message)
```

Key options on `ClaudeAgentOptions` / `Options`:

| Option | Description |
|---|---|
| `allowed_tools` | Which built-in tools the agent may call |
| `permission_mode` | `"default"` (prompt on writes), `"acceptEdits"`, `"bypassPermissions"` |
| `system_prompt` | String, or `{"type": "preset", "preset": "claude_code"}` |
| `setting_sources` | Which filesystem sources to load: `"project"`, `"user"`, `"local"` |
| `agents` | Programmatic subagent definitions (see [Subagents](#subagents)) |
| `skills` | `"all"`, `[]`, or a list of skill names |
| `hooks` | In-process callback hooks |
| `mcp_servers` | MCP server connections |
| `model` | Model alias or full model ID |
| `max_turns` | Maximum agent turns before stopping |
| `cwd` | Working directory for filesystem operations |

## Claude Code Conventions in the SDK

The SDK intentionally mirrors the filesystem conventions of interactive Claude Code. A project already configured for Claude Code requires minimal changes to run programmatically.

### CLAUDE.md

Loaded from the filesystem via `setting_sources`:
- `"project"` → loads `<cwd>/CLAUDE.md`, parent-directory CLAUDE.md files, and `.claude/rules/*.md`
- `"user"` → loads `~/.claude/CLAUDE.md`
- `"local"` → loads `CLAUDE.local.md` and `.claude/settings.local.json`

Default (when omitted) is `["user", "project", "local"]`, matching CLI behavior.

**Multi-tenancy warning**: three sources load regardless of `setting_sources`: managed policy settings, `~/.claude.json` global config, and auto-memory at `~/.claude/projects/<project>/memory/`. For isolated deployments, use `setting_sources=[]` and set `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`.

### Subagents

Files in `.claude/agents/` load when `setting_sources` includes `"project"`. Subagents can also be defined programmatically via the `agents` parameter — programmatic definitions take precedence over filesystem ones with the same name:

```python
from claude_agent_sdk import AgentDefinition

agents={
    "code-reviewer": AgentDefinition(
        description="Expert code reviewer for quality and security.",
        prompt="You are a code review specialist...",
        tools=["Read", "Grep", "Glob"],
        model="sonnet",
    )
}
```

The `Agent` tool must be in `allowed_tools` for Claude to invoke subagents. Subagents run in fresh context windows; they cannot spawn their own subagents.

See [Claude Code Subagents](claude-code-subagents.md) for the full subagent model.

### Skills

Skills are `SKILL.md` files in `.claude/skills/<name>/` directories. There is no programmatic API for registering skills — they must exist on the filesystem. The `skills` option controls which are enabled per call:

```python
skills="all"          # enable all discovered skills
skills=["pdf-parser"] # enable only listed skills
skills=[]             # disable all skills
```

**Limitation**: the `allowed-tools` frontmatter in `SKILL.md` (which works in the CLI) is ignored by the SDK. Tool access is controlled exclusively via `allowed_tools`.

### System Prompt

The full Claude Code system prompt is not the SDK default. To get it:

```python
system_prompt={"type": "preset", "preset": "claude_code"}
# or append to it:
system_prompt={"type": "preset", "preset": "claude_code", "append": "Always use type hints."}
```

The preset injects per-session dynamic context (cwd, git status, platform, etc.) into the system prompt. For cross-session prompt cache reuse, set `exclude_dynamic_sections=True` — this moves dynamic values into the first user message instead, making the system prompt stable and cacheable.

### Hooks

Filesystem hooks in `.claude/settings.json` load via `setting_sources`. In addition, programmatic in-process callbacks can be registered via the `hooks` parameter. Callbacks return `{}` to allow or `{"decision": "block", "reason": "..."}` to block a tool call.

## Deployment Patterns

The SDK is designed for server-side use. Anthropic documents four production deployment patterns:

1. **Ephemeral sessions** — new container per task, destroyed on completion. Best for one-off operations (bug fixes, document translation).
2. **Long-running sessions** — persistent containers handling multiple queries over their lifetime. Best for proactive agents (monitoring, chat bots).
3. **Hybrid sessions** — ephemeral containers hydrated with history from a database. Best for intermittent, multi-session tasks.
4. **Single containers** — multiple SDK processes in one container for agents that must share a filesystem.

The SDK does not provide HTTP server functionality — you build that wrapper. Recommended sandbox providers include Modal, Cloudflare Sandboxes, E2B, Fly Machines, and Vercel Sandbox.

**Container minimums**: 1 GiB RAM, 5 GiB disk, 1 CPU. Python 3.10+ or Node.js 18+.

**Session storage**: sessions are JSONL files at `~/.claude/projects/<encoded-cwd>/<session-id>.jsonl`. For cross-host resumption, copy the file or implement a `SessionStore` adapter. The Python SDK offers a `ClaudeSDKClient` for stateful multi-query sessions.

## Key Limitations

- **~12s startup overhead per `query()` call** — the SDK spawns the CLI binary as a subprocess. Long-running container deployments amortize this cost.
- **Skills are filesystem-only** — cannot be injected at runtime.
- **Subagents cannot nest** — one level of delegation only.
- **Agent teams are CLI-only** — multi-agent topologies with direct peer messaging are unavailable.
- **Python SDK: no `outputStyle` option** — output styles must be configured via `settings.local.json` or the TypeScript SDK.

## Managed Agents (Alternative)

For production use without operating container infrastructure, [Managed Agents](claude-managed-agents.md) is a hosted alternative where Anthropic manages the sandbox. The tradeoff: filesystem-based configuration (CLAUDE.md, `.claude/agents/`) does not apply — agent definitions are expressed as REST payloads.

## See Also

- [Claude Code](claude-code.md)
- [Claude Code Subagents](claude-code-subagents.md)
- [Managed Agents](claude-managed-agents.md)
- [Agentic Loop](../concepts/agentic-loop.md)
- [Tool Use](../concepts/tool-use.md)
- [Context Management](../concepts/context-management.md)
