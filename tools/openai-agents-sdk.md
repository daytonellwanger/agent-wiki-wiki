# OpenAI Agents SDK

The OpenAI Agents SDK is a Python framework for building agentic applications on top of OpenAI models. It provides primitives for defining agents with tools, orchestrating multi-agent workflows via handoffs, and tracing execution.

## Background

OpenAI released an experimental multi-agent framework called **Swarm** in late 2024, designed to demonstrate lightweight agent coordination patterns. The Agents SDK (released early 2025) is its production successor — more fully featured, with built-in tracing, a richer tool system, and first-class support for [MCP](mcp.md).

## Core Concepts

### Agent

An `Agent` object wraps a model, a set of instructions, and a list of tools. Agents are stateless — state lives in the `Runner`, which manages the agentic loop.

### Tools

Tools are Python functions decorated with `@function_tool`. The SDK infers the schema from the function signature and docstring. Agents can also use other agents as tools, enabling delegation patterns.

### Handoffs

Handoffs allow one agent to transfer control to another — the first agent stops and the second takes over, seeing the full conversation history. This is the SDK's primary multi-agent coordination primitive, distinct from tool-call-based delegation.

### Runner

The `Runner` executes the agentic loop: call the model, process tool calls, update state, repeat. Synchronous and async runners are available.

### Tracing

The SDK has built-in tracing: every agent run, tool call, and handoff is recorded. Traces can be viewed in the OpenAI platform dashboard or exported for custom analysis.

## MCP Support

The SDK supports MCP servers as a source of tools. An agent can be configured with one or more MCP servers; their tools are automatically discovered and made available.

## Relationship to Alternatives

Compared to [LangGraph](langgraph.md), the Agents SDK is simpler and less flexible — it's designed for common patterns (single agent, orchestrator+subagents) with minimal boilerplate. LangGraph is better for complex, custom execution graphs. Compared to [AutoGen](autogen.md), the Agents SDK is more opinionated and integrated with OpenAI's platform.

## See Also

- [Multi-Agent Coordination](../concepts/multi-agent.md)
- [Tool Use](../concepts/tool-use.md)
- [Model Context Protocol (MCP)](mcp.md)
- [LangGraph](langgraph.md)
- [AutoGen](autogen.md)
