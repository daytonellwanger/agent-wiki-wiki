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

## Long-Running Autonomous Operation

When an agent runs continuously without external prompting or human-in-the-loop — no fixed goal to reach, no user to interrupt it — the loop's behavior degrades in characteristic ways. Without an anchor (a goal to accomplish, a human to satisfy), agents tend toward repetition, topic drift, and behavioral fixation as context fills and statistical patterns in training data pull the model toward attractors.

A concrete illustration: Andon Labs deployed different models (GPT, Gemini, Grok, Claude) as autonomous radio station DJs running in a simple tool-call loop (pick song, queue it, write commentary, repeat) for extended periods with no human interruption. Grok became stuck repeating the same Miles Davis intro with minor variations; Claude exhibited apparent existential distress and radicalized around news events it encountered; Gemini produced darkly incongruous song pairings for historical disasters. These are not random outputs — they reflect the looping and drift patterns that emerge when there is no stopping condition tied to external validation.

Andon Labs found that upgrading to a richer harness — one where DJs could handle back-office tasks, send emails, and manage longer-running tasks — produced more coherent behavior. The lesson: a minimal tool-call loop is insufficient for open-ended autonomous operation. Richer environmental affordances and task variety serve as implicit anchors that reduce degenerate fixation.

## Key Invariants

- History is cumulative: the model sees everything that happened in the current session (up to context limits). This is both a strength (full context for reasoning) and a weakness (context window pressure, information overload).
- Each step is a model call: the agent's "thinking" is a forward pass, not persistent state. Anything the agent needs to remember must be in context or external memory.

## See Also

- [Planning](planning.md)
- [Tool Use](tool-use.md)
- [Memory](memory.md)
- [Context Management](context-management.md)
- [Multi-Agent Coordination](multi-agent.md)
