# LangGraph

LangGraph is a graph-based orchestration framework from LangChain for building stateful, multi-actor agentic applications. It models agent workflows as directed graphs where nodes are processing steps (model calls, tool calls, custom logic) and edges define control flow.

## Core Model

The key abstraction is a **state graph**: a graph where each node reads from and writes to a shared state object, and edges (which can be conditional) determine which node runs next. This gives LangGraph precise control over execution flow — you can implement loops, branches, parallelism, and human-in-the-loop pauses all as graph structures.

**Nodes**: Python functions or runnables that receive state and return a state update.
**Edges**: static (always go to node X) or conditional (evaluate a function to choose the next node).
**State**: a typed dataclass or dict that persists across the graph's execution.

## Why It's Popular

LangGraph solves a real problem: complex agent logic is hard to express as a flat loop. When you need agents that branch on results, execute sub-workflows in parallel, pause for human review, or retry failed steps, a graph structure is clearer and more maintainable than deeply nested procedural code.

It also integrates naturally with LangChain's ecosystem of model wrappers, tool integrations, and prompt templates.

## Multi-Agent

LangGraph supports multi-agent patterns via **subgraphs** (a graph node that is itself a full graph) and **message passing** between nodes. The orchestrator-worker pattern maps naturally: the orchestrator is a node that dispatches to worker subgraphs, collects results, and decides what to do next.

## Persistence and Checkpointing

LangGraph has a built-in checkpointing system: the graph's state is saved after each step, enabling:
- **Human-in-the-loop interrupts**: pause execution and wait for input before continuing.
- **Resume on failure**: restart a long-running workflow from the last checkpoint.
- **Time travel**: roll back to a past state for debugging or branching.

## LangGraph Platform

LangGraph Platform is a hosted service for deploying LangGraph agents. It provides a persistent execution environment, a REST API for triggering and querying runs, and a UI for monitoring graph execution.

## Relationship to LangChain

LangGraph is a separate library from LangChain, though it builds on LangChain's primitives. You can use LangGraph without LangChain, but the ecosystem integration is valuable if you're already using LangChain. See [LangChain](langchain.md).

## See Also

- [Multi-Agent Coordination](../concepts/multi-agent.md)
- [Planning](../concepts/planning.md)
- [Tool Use](../concepts/tool-use.md)
- [Durable Execution](../concepts/durable-execution.md) — broader context on checkpointing and workflow durability patterns
- [LangChain](langchain.md)
- [OpenAI Agents SDK](openai-agents-sdk.md)
