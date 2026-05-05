# Agentic Loop

The agentic loop is the fundamental execution pattern of an AI agent: a repeated cycle of perception, reasoning, action, and observation that continues until the task is complete or a stopping condition is met.

## The Basic Loop

```
while not done:
    observation = perceive(environment)
    thought = reason(goal, history, observation)
    action = decide(thought)
    result = execute(action)
    history.append(observation, thought, action, result)
```

In practice: the model receives a prompt (including the goal, conversation history, and any new observations), produces a response that may include tool calls, the tool calls are executed, results are added to history, and the loop repeats.

## Stopping Conditions

An agent stops when:
1. It produces a final answer (no tool calls, just a response).
2. It calls a designated "stop" tool or produces a structured stop signal.
3. A maximum step count or token budget is exhausted.
4. A human interrupt is triggered.

Well-designed agents should stop gracefully and report partial results if they hit a budget limit, rather than failing silently.

## Human-in-the-Loop

Agents can pause the loop to request human input. This is important for:
- Irreversible or high-stakes actions (deleting data, sending emails, deploying code).
- Ambiguous goals where the agent needs clarification before proceeding.
- Cases where the agent detects it's stuck or uncertain.

The decision of when to interrupt is itself a planning problem. Interrupting too often defeats the purpose of automation; never interrupting creates risk. See [Planning](planning.md).

## Loop Depth

Agents can be nested: a subagent runs its own agentic loop inside a step of the parent agent's loop. This is how [multi-agent systems](multi-agent.md) work. Depth increases capability but also complexity and debugging difficulty.

## Key Invariants

- History is cumulative: the model sees everything that happened in the current session (up to context limits). This is both a strength (full context for reasoning) and a weakness (context window pressure, information overload).
- Each step is a model call: the agent's "thinking" is a forward pass, not persistent state. Anything the agent needs to remember must be in context or external memory.

## See Also

- [Planning](planning.md)
- [Tool Use](tool-use.md)
- [Memory](memory.md)
- [Context Management](context-management.md)
- [Multi-Agent Coordination](multi-agent.md)
