# Claude Code

Claude Code is Anthropic's agentic coding tool — a CLI (and IDE extension) that runs Claude directly in the developer's environment, with access to the local filesystem, shell, and external tools via MCP.

## What It Does

Claude Code is an agent built on Claude that specializes in software engineering tasks: writing and editing code, running tests, reading documentation, searching codebases, executing shell commands, and managing git. It operates as a full agentic loop with tools, not just a code-completion autocomplete.

Key capabilities:
- Read, write, and edit files across the local filesystem
- Execute shell commands and interpret results
- Search code (grep, find, AST search)
- Run tests and iterate on failures
- Manage git (commit, branch, PR creation via `gh`)
- Connect to external tools via [MCP](mcp.md)
- Supports hooks: shell commands triggered on events (e.g., after each tool call)

## Architecture

Claude Code runs as a CLI process that wraps the Claude API. Each conversation is an agentic loop: the model sees the conversation history and tool results, produces tool calls or a response, and the loop continues. The host (Claude Code itself) handles permission prompts, tool execution, and context management (including compaction when context fills).

## Differentiating Features

- **Permission model**: the user approves or denies individual tool calls, with configurable auto-approval for safe operations. This gives human oversight without requiring constant interruption.
- **CLAUDE.md**: a per-project (or per-user) markdown file that provides the model with project-specific instructions, conventions, and context. Read at the start of each session.
- **Subagents**: Claude Code supports spawning subagents via the **Agent tool** (renamed from Task tool in v2.1.63) for parallelism and context isolation. Custom subagents are defined as Markdown files with YAML frontmatter in `.claude/agents/` or `~/.claude/agents/`. See [Claude Code Subagents](claude-code-subagents.md) for full coverage.
- **Extended thinking**: Claude Code can use Claude's extended thinking mode for complex reasoning tasks.

## IDE Integration

Available as extensions for VS Code and JetBrains IDEs, embedding the CLI experience inside the editor with a panel UI.

## Enterprise Adoption Notes

The token pricing model creates unpredictable cost exposure at enterprise scale. In a notable instance, Microsoft granted thousands of employees access to Claude Code in December 2025 as a pilot for its Experiences and Devices group (Windows, Microsoft 365, Outlook, Teams, Surface). By May 2026, the pilot had consumed Microsoft's entire projected 2026 AI spend, and the company announced it would wind down Claude Code licenses by June 30, 2026 — the end of Microsoft's fiscal year — in favor of GitHub Copilot CLI.

The stated rationale was "strategic consolidation" and better fit with Microsoft's internal repositories and security requirements. The financial driver is widely understood to be fiscal-year-end cost containment. Anthropic models remain accessible to Microsoft developers through Copilot CLI, Microsoft 365 apps, and the existing partnership; the change is to the dedicated Claude Code license rather than to model access generally.

The HN discussion around the announcement identified this as a broader enterprise risk pattern with agentic AI tools: unsupervised agentic workflows consume tokens at a qualitatively different rate than supervised human-in-the-loop use, and the cost curve is difficult for enterprises to predict or control without deliberate tooling for monitoring and governance. One developer in the thread reported spending approximately $40,000 on Claude tokens over three months, attributing the excess to the model spawning numerous subagents that got stuck. Another commenter noted that Claude's token usage per task is higher than comparable agents — a deliberate quality trade-off that enterprises may not want when operating at scale.

## See Also

- [Claude Code Subagents](claude-code-subagents.md)
- [Model Context Protocol (MCP)](mcp.md)
- [Tool Use](../concepts/tool-use.md)
- [Agentic Loop](../concepts/agentic-loop.md)
- [Context Management](../concepts/context-management.md)
- [Progressive Disclosure](../concepts/progressive-disclosure.md) — Design principle behind Agent Skills and on-demand context loading
- [Multi-Agent Coordination](../concepts/multi-agent.md)
- [Pi](pi.md) — Minimalist open-source alternative; multi-provider, no vendor-enforced behavior
