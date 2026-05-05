# AutoGen

AutoGen is an open-source framework from Microsoft Research for building multi-agent conversational systems. Its core idea is that complex tasks can be solved by having multiple agents — each with different roles, capabilities, and instructions — converse with each other.

## Core Model

In AutoGen, agents are **conversational agents**: they send and receive messages. An agent can be an LLM-backed agent, a human proxy (which surfaces messages to a real user), a code-executing agent, or a custom agent with any logic. Agents collaborate by passing natural-language messages, with each agent acting on incoming messages according to its instructions and capabilities.

### Key Agent Types

- **AssistantAgent**: an LLM-backed agent that produces messages and can suggest code.
- **UserProxyAgent**: proxies for a human; can execute code and relay results back into the conversation.
- **GroupChat**: a coordinator that routes messages among a set of agents in a multi-agent conversation, with configurable speaker selection.

## What It's Good At

AutoGen excels at workflows where multiple specialized agents need to collaborate in an open-ended way — e.g., a planner agent, a coder agent, and a critic agent iterating together on a software task. The conversational model is flexible and natural for iterative refinement loops.

It also has strong code-execution support: agents can write Python, have it executed in a sandbox, observe the output, and iterate.

## AutoGen v0.4 Rewrite

Microsoft significantly redesigned AutoGen in v0.4 (late 2024 / early 2025), introducing an **event-driven, actor-model architecture**. Agents are now actors that communicate via typed messages over an event bus, rather than a synchronous conversation. This enables distributed execution and better scalability.

The new architecture introduced:
- **AgentChat**: a higher-level API that preserves the original conversational programming model on top of the actor core.
- **Core**: the low-level actor runtime, suitable for building custom agent types.

## Relationship to Alternatives

Compared to [LangGraph](langgraph.md), AutoGen is more flexible for open-ended agent conversations but less suited for precise, predetermined execution graphs. Compared to [OpenAI Agents SDK](openai-agents-sdk.md), it's more framework-agnostic (supports multiple model providers) and more research-oriented.

## See Also

- [Multi-Agent Coordination](../concepts/multi-agent.md)
- [Planning](../concepts/planning.md)
- [LangGraph](langgraph.md)
- [OpenAI Agents SDK](openai-agents-sdk.md)
