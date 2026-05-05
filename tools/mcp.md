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

## Tradeoffs

**Pros**: interoperability, ecosystem leverage, standardized discovery.
**Cons**: extra latency from the client-server hop (for remote servers), schema translation overhead, still maturing.

For tightly integrated tools used in a single application, direct function calling may be simpler. MCP shines when tools need to be reused across multiple agents or hosts.

## See Also

- [Tool Use](../concepts/tool-use.md)
- [Claude Code](claude-code.md)
- [Agentic Loop](../concepts/agentic-loop.md)
