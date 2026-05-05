# CrewAI

CrewAI is a Python framework for building role-based multi-agent systems. It models agents as members of a "crew" — each with a defined role, goal, backstory, and set of tools — that collaborate on tasks.

## Core Model

The primary abstractions are:

- **Agent**: a defined persona with a role (e.g., "Senior Research Analyst"), a goal, a backstory (context that shapes behavior), and tools.
- **Task**: a specific objective assigned to one or more agents, with an expected output format.
- **Crew**: a collection of agents and tasks, along with a process that defines how they work together.
- **Process**: how the crew executes — sequential (tasks run in order), hierarchical (a manager agent routes tasks), or parallel.

## Design Philosophy

CrewAI is explicitly designed around the metaphor of a human team. Role descriptions and backstories are used to shape agent behavior through natural language — the "Senior Researcher" agent is told to be thorough and critical; the "Writer" agent is told to produce clear, engaging prose. This makes it accessible and intuitive for non-experts.

## Tools

CrewAI integrates with a library of built-in tools (web search, code execution, file read/write, etc.) and supports custom tools. Tools are assigned to specific agents rather than being globally available, which mirrors how human teams have specialized capabilities.

## When to Use CrewAI

CrewAI works well for workflows that naturally decompose into defined roles — research + synthesis, planning + execution, writing + review. The explicit role/persona model can improve output quality for tasks that benefit from distinct "voices" or expertise perspectives.

For complex, dynamic workflows with conditional branching or precise control flow, [LangGraph](langgraph.md) is typically more appropriate.

## Relationship to Alternatives

CrewAI is more opinionated about structure than [AutoGen](autogen.md) (which is more open-ended) but simpler to get started with than [LangGraph](langgraph.md) (which requires explicit graph design). It occupies a "medium structure, high accessibility" niche.

## See Also

- [Multi-Agent Coordination](../concepts/multi-agent.md)
- [Tool Use](../concepts/tool-use.md)
- [LangGraph](langgraph.md)
- [AutoGen](autogen.md)
