# Model Context Protocol (MCP)

MCP is an open protocol developed by Anthropic that standardizes how applications provide tools, resources, and prompts to language models. It separates the concern of tool implementation from tool consumption: a tool provider implements the MCP server interface once, and any MCP-compatible host can use it without custom integration code.

## Why It Exists

Before MCP, every agent framework had its own way of defining tools, and tool providers had to write separate integrations for each. MCP is an attempt to create a universal adapter layer — analogous to LSP (Language Server Protocol) for language tooling, but for LLM tool connectivity.

## Core Concepts

- **Server**: exposes tools, resources, and prompts over the MCP protocol. Examples: a filesystem server, a database server, a web search server.
- **Client**: an MCP-aware application (host) that connects to servers and exposes their capabilities to a model. Examples: Claude Desktop, Claude Code, custom agent hosts.
- **Tools**: functions the model can call (analogous to function calling, but described in MCP's schema format).
- **Resources**: data the model can read (files, database records, API responses) without invoking a function.
- **Prompts**: reusable prompt templates exposed by the server.

## Transport

MCP supports two transport mechanisms:
- **stdio**: server runs as a subprocess, communicates over stdin/stdout. Simple and common for local tools.
- **HTTP + SSE**: server runs as a network service. Allows remote tools and multi-client scenarios.

## Adoption

As of 2025, MCP has seen significant adoption. Claude Desktop and Claude Code support it natively. Many third-party tool providers have published MCP servers. The major agent frameworks ([LangGraph](langgraph.md), [OpenAI Agents SDK](openai-agents-sdk.md)) have added MCP support.

In May 2026, Anthropic acquired Stainless — the company that had been building the official Anthropic SDKs and MCP server tooling used by hundreds of companies. The acquisition signals that Anthropic views SDK and MCP server quality as strategic infrastructure, not a commodity concern. Katelyn Lesse, Anthropic's Head of Platform Engineering, framed the rationale: "Agents are only as useful as what they can connect to."

## Practical Installation Guidance

Install MCPs selectively. A bloated tool list degrades decision quality — the more tools available, the harder it is for the model to select the right one for a given context. A practical set of high-value MCPs for coding workflows:

- **GitHub**: PRs, issues, code search
- **Context7**: live library documentation (avoids hallucinated API details from training data)
- **Sentry**: real error context with stack traces and surrounding events
- **Playwright**: browser automation and testing
- **PostgreSQL / Supabase**: direct database queries

Team-shared MCPs go in `.mcp.json` (checked into version control); personal ones in `~/.claude.json`.

## Tradeoffs

**Pros**: interoperability, ecosystem leverage, standardized discovery.
**Cons**: extra latency from the client-server hop (for remote servers), schema translation overhead, still maturing.

For tightly integrated tools used in a single application, direct function calling may be simpler. MCP shines when tools need to be reused across multiple agents or hosts.

## See Also

- [Tool Use](../concepts/tool-use.md)
- [Claude Code](claude-code.md)
- [Agentic Loop](../concepts/agentic-loop.md)
