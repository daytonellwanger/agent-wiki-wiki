# Claude Code Subagents

Claude Code's subagent system lets a main Claude Code session delegate focused subtasks to specialized child agents — each running in its own context window with its own system prompt, tool restrictions, and (optionally) its own model. The mechanism is exposed through the **Agent tool** (called the Task tool prior to v2.1.63; the old name still works as an alias).

## What a Subagent Is

A subagent is a separate agent instance spawned by the main session. Key properties:

- Runs in a **fresh context window** — it does not inherit the parent's conversation history or tool results.
- Receives only the parent's system prompt (defined in the subagent's configuration) plus the specific prompt the parent passes at invocation time.
- Returns its final message as the Agent tool result; all intermediate tool calls and outputs stay isolated inside the subagent.

The practical benefit is context hygiene: a subagent doing extensive file search, log analysis, or documentation retrieval doesn't pollute the main conversation with noisy intermediate output; it hands back only what matters.

## Creating Custom Subagents

### File-based definition

Subagents are Markdown files with YAML frontmatter. The frontmatter configures the agent; the Markdown body becomes its system prompt.

```markdown
---
name: code-reviewer
description: Expert code reviewer. Use proactively after code changes.
tools: Read, Grep, Glob, Bash
model: sonnet
---

You are a senior code reviewer. When invoked, analyze the code and provide
specific, actionable feedback on quality, security, and best practices.
```

Only `name` and `description` are required. The `description` is particularly important: Claude reads it to decide when to automatically delegate to this subagent.

### Where to store them

Files are scoped by location. Higher-priority locations shadow lower-priority ones when names conflict.

| Location | Scope | Priority |
|---|---|---|
| Managed settings (org admin) | Organization-wide | 1 (highest) |
| `--agents` CLI flag (JSON) | Current session only | 2 |
| `.claude/agents/` | Current project | 3 |
| `~/.claude/agents/` | All user projects | 4 |
| Plugin `agents/` directory | Where plugin is enabled | 5 (lowest) |

Project subagents (`.claude/agents/`) can be checked into version control so teams share and improve them together.

### The `/agents` command

Running `/agents` opens an interactive interface for creating, editing, listing, and deleting subagents — including a guided wizard that generates the system prompt via Claude. From the command line, `claude agents` lists all configured subagents grouped by source.

### CLI-defined subagents

Pass subagents as JSON at startup for ephemeral, session-scoped definitions:

```bash
claude --agents '{
  "code-reviewer": {
    "description": "Expert code reviewer. Use proactively after code changes.",
    "prompt": "You are a senior code reviewer...",
    "tools": ["Read", "Grep", "Glob", "Bash"],
    "model": "sonnet"
  }
}'
```

## Invoking Subagents

### Automatic delegation

Claude reads each subagent's `description` field and decides when to delegate. No special syntax required. The description quality is the primary lever for tuning delegation behavior.

### Explicit invocation

Three escalating levels of explicit control:

1. **Natural language**: "Use the code-reviewer subagent to check my recent changes." Claude decides whether to delegate.
2. **@-mention**: `@"code-reviewer (agent)"` guarantees that subagent runs for the current task. The @-mention appears in the same typeahead as file references.
3. **Session-wide**: `claude --agent code-reviewer` starts a session where the main thread itself runs as that subagent (its system prompt replaces the default Claude Code system prompt entirely).

To set a session-wide agent as a project default, add `"agent": "code-reviewer"` to `.claude/settings.json`.

### Foreground vs. background

- **Foreground** (default): blocks the main conversation until complete. Permission prompts and clarifying questions pass through to the user.
- **Background**: runs concurrently. Permissions are pre-approved before launch; anything not pre-approved is auto-denied. The user can press **Ctrl+B** to background a running task, or ask Claude to "run this in the background". To disable background tasks entirely, set `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1`.

## Forked Subagents (Experimental)

Enabled via `CLAUDE_CODE_FORK_SUBAGENT=1` (requires v2.1.117+). A fork is a subagent that inherits the **full conversation history** of the parent rather than starting fresh. This trades input isolation for context continuity — useful when a side task would otherwise need extensive re-explanation.

Key differences from named subagents:

| | Fork | Named subagent |
|---|---|---|
| Context | Full parent conversation history | Fresh, only what's passed in the prompt |
| System prompt | Same as parent | From the subagent's definition file |
| Prompt cache | Shared with parent (cheaper) | Separate cache |
| Permissions | Surfaces prompts in terminal | Pre-approved before launch |

When fork mode is enabled, Claude uses forks wherever it would otherwise use the general-purpose subagent. Named subagents (like Explore) are unaffected. A fork cannot spawn further forks.

## Persistent Memory

Setting `memory: user|project|local` in a subagent's frontmatter gives it a persistent directory that survives across conversations. The subagent's system prompt is automatically augmented with instructions to read and write `MEMORY.md` (first 200 lines or 25 KB is injected at startup). This lets a subagent accumulate project-specific knowledge, patterns, and architectural notes over time.

## Lifecycle Hooks

Hooks can be defined in two places:

- **Subagent frontmatter**: `PreToolUse`, `PostToolUse`, and `Stop` hooks that are active only while the subagent runs. At runtime, `Stop` is converted to `SubagentStop`.
- **`settings.json`**: `SubagentStart` and `SubagentStop` hooks in the main session, matched by agent type name.

A common pattern is a `PreToolUse` hook on `Bash` to validate commands against an allowlist before execution.

## Agent Teams (Related, Experimental)

Subagents are distinct from **agent teams**, which are enabled via `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`. The key differences:

| | Subagents | Agent teams |
|---|---|---|
| Communication | Report back to main agent only | Teammates message each other directly |
| Coordination | Main agent manages all work | Shared task list with self-coordination |
| Context | Each subagent is fresh per invocation | Each teammate is a persistent independent session |
| Resumption | Via session + agent ID (experimental) | Each teammate is already a full session |
| Experimental? | No — generally available | Yes — experimental, known limitations |

Use subagents for focused, self-contained tasks where only the result matters. Use agent teams when workers need to communicate, challenge each other, or coordinate on shared state.

## When to Use Subagents

Use a subagent when:
- The task produces verbose output (logs, test results, search hits) that would crowd the main context.
- You want to enforce specific tool restrictions or run a task in a stricter permission mode.
- The work is genuinely self-contained and can summarize its result.
- You want to route a task to a faster/cheaper model (e.g., Haiku for exploration).

Stick with the main conversation when:
- The task needs frequent back-and-forth or iterative refinement.
- Multiple phases share significant context (planning → implementation → testing).
- Latency matters — subagents start fresh and may need time to gather their own context.

### Code Review Subagents

Code review is one of the most effective uses of the subagent pattern. A well-designed code review subagent uses read-only tools to prevent reviewer bias, reviews in a fresh context with no implementation history, and includes explicit "Do NOT flag" sections to reduce noise. Group findings by severity.

Claude Code ships a `/code-review` skill that the Anthropic team is treating as the primary built-in review mechanism. According to Boris Cherny (Anthropic), it supports effort levels — `low`, `medium`, `high`, `xhigh`, `max`, and an `ultra` mode — where higher effort levels catch more issues at higher cost. The `ultra` mode is designed to catch >99% of bugs at approximately $3–20 per review. A common effective pattern: `/code-review xhigh --fix` covers the large majority of what automated review can address.

The writer/reviewer pattern (one session implements, a second fresh session reviews) is the strongest form of this: the reviewer subagent has no knowledge of the implementation decisions, so its review is structurally independent rather than biased by having written the code. See [Verifiable Constraints](../concepts/verifiable-constraints.md) for the connection to multi-model parallel review.

## SDK Usage

In the Claude Agent SDK (Python/TypeScript), subagents are defined programmatically using the `agents` parameter in `query()` options. The `Agent` tool must be included in `allowedTools` for Claude to invoke them. Programmatically defined agents take precedence over filesystem-based ones with the same name.

See the [Subagents in the SDK documentation](https://code.claude.com/docs/en/agent-sdk/subagents) for full API reference.

## See Also

- [Claude Code](claude-code.md)
- [Multi-Agent Coordination](../concepts/multi-agent.md)
- [Context Management](../concepts/context-management.md)
- [Tool Use](../concepts/tool-use.md)
- [Model Context Protocol (MCP)](mcp.md)