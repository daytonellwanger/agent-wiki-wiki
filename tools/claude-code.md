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

## See Also

- [Claude Code Subagents](claude-code-subagents.md)
- [Model Context Protocol (MCP)](mcp.md)
- [Tool Use](../concepts/tool-use.md)
- [Agentic Loop](../concepts/agentic-loop.md)
- [Context Management](../concepts/context-management.md)
- [Multi-Agent Coordination](../concepts/multi-agent.md)
