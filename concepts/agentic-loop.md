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

## Structural Constraints in Coding Loops

When AI coding agents generate production code, two strategies exist for enforcing correctness invariants: behavioral gates (prompt instructions, checklists) and structural gates (compiler-enforced types, deterministic verification steps).

**Behavioral gates** rely on the model consistently following instructions across potentially thousands of lines of generated code. They degrade as codebases grow and as agents accumulate context. A well-placed instruction in a prompt can be silently ignored or overridden by downstream context.

**Structural gates** move enforcement into the substrate the loop cannot skip: the type system, the compiler, and deterministic tooling. One approach uses typed code generation to express invariants (e.g., multi-tenant authorization rules) as sealed types with unexported fields and constrained constructors, making it structurally impossible to bypass the invariant accidentally rather than just conventionally wrong. Another approach embeds a fixed sequence of verification steps — spec generation, test generation, compilation, type-checking, and audit — as required checkpoints in the loop. Each gate produces a binary pass/fail result; the loop cannot proceed if a gate fails.

The practical distinction: behavioral gates depend on model capability improving and on instructions surviving context accumulation; structural gates work independently of model capability and produce auditable artifacts rather than claims about model reliability.

This connects to [Evaluation](evaluation.md)'s treatment of verifiable rewards: tasks amenable to structural verification (does the code compile? do the types check?) are the same tasks where outcome-based evaluation is cleanest and where RLVR-trained models have improved most. The two patterns are complementary — structural gates make it easier to define the verifiable signal that both the harness and the training process depend on.

Structural constraints are not complete proofs of correctness. The human must still encode the right invariants upfront. The approach prevents *accidental* bypasses; a deliberately wrong invariant produces a wrong but type-safe result. The benefit is that a constraint in a type is reviewable, permanent, and enforced mechanically — whereas a constraint in a prompt decays.

## Emerging Alternatives to Sequential Execution

The standard agentic loop is fundamentally sequential: each forward pass reads a context window and produces a response; new inputs cannot arrive while the model is generating; the model cannot act while it is thinking. This creates bottlenecks when agents need to respond to new information mid-generation or parallelize reasoning and action.

A 2026 preprint from the Max Planck Institute for Intelligent Systems ("Multi-Stream LLMs", arXiv 2605.12460) proposes switching from instruction-tuning for sequential message formats to instruction-tuning for multiple parallel streams of computation — one per role (thought, input, output). Each forward pass reads from and writes to all streams simultaneously, with causal constraints that prevent outputs from influencing the inputs that produced them. The claimed benefits include: unblocking agents so they can think and act concurrently; improved monitorability (the thought stream is explicitly separate and inspectable); and enhanced security through cleaner role separation. Experiments used a Qwen 27B architecture; results showed maintained task quality with reduced end-to-end latency.

This is an early-stage preprint with limited community validation. The idea warrants attention because it addresses a structural bottleneck in every current agent framework, but the practical impact on agent harnesses (which are not yet instruction-tuned for multi-stream formats) remains to be seen.

## Key Invariants

- History is cumulative: the model sees everything that happened in the current session (up to context limits). This is both a strength (full context for reasoning) and a weakness (context window pressure, information overload).
- Each step is a model call: the agent's "thinking" is a forward pass, not persistent state. Anything the agent needs to remember must be in context or external memory.

## See Also

- [Planning](planning.md)
- [Tool Use](tool-use.md)
- [Memory](memory.md)
- [Context Management](context-management.md)
- [Multi-Agent Coordination](multi-agent.md)
- [Evaluation](evaluation.md)
