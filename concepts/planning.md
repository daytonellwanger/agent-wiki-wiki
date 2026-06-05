# Planning

Planning refers to how an agent reasons about what to do — decomposing a goal into steps, deciding which actions to take, and adapting when things go wrong. It is one of the core competencies that separates capable agents from reactive chatbots.

## ReAct

ReAct (Reason + Act) is the most common pattern: the model interleaves reasoning traces ("I should look up X before trying Y") with tool calls. This makes the agent's logic auditable and gives the model a scratchpad to work through complex sub-problems before committing to an action.

Most modern agent frameworks use a ReAct-style loop by default, even if they don't name it as such.

## Chain-of-Thought and Extended Thinking

Chain-of-thought prompting encourages the model to reason step-by-step before answering. Extended thinking (available in some models, including Claude) gives the model a dedicated reasoning phase that happens before the visible response is generated. This substantially improves performance on tasks requiring multi-step reasoning or careful planning.

The scale at which extended reasoning can operate has grown substantially. In May 2026, an OpenAI reasoning model produced a 125-page summarized chain of thought while autonomously disproving a 79-year-old open problem in discrete geometry — a qualitatively different scale from what has been publicly documented in standard coding or QA benchmarks. Consistent with this, Anthropic has described its Mythos model as operating at similarly extended reasoning depths. This points to a pattern where the frontier on hard, open-ended reasoning tasks is defined not by model architecture changes but by the length and structure of the reasoning trace the model is allowed to develop.

## Task Decomposition

For complex goals, agents often benefit from explicit decomposition: breaking the task into a list of sub-tasks before beginning execution. This can be done in a single planning step at the start, or dynamically as sub-tasks are discovered during execution.

**Plan-then-execute**: generate a full plan upfront, then execute each step. Clear, but brittle when the world doesn't match the plan.

**Dynamic planning**: update the plan at each step based on observations. More robust, but harder to track and audit.

## Failure Modes

- **Horizon collapse**: the agent solves a sub-problem well but loses track of the overall goal.
- **Overplanning**: spending too many steps in planning rather than acting.
- **Irreversible errors**: taking an action that makes the original goal impossible to achieve (e.g., deleting a file needed later).
- **Stuck loops**: repeatedly calling the same tool with the same arguments when it fails. In open-ended autonomous operation (no fixed goal, no human-in-the-loop), a related but distinct form appears: thematic or behavioral fixation, where the agent drifts into a repetitive attractor state — not because a tool is failing, but because there is no external anchor pulling it out. See [Agentic Loop](agentic-loop.md) for more on this failure mode in long-running autonomous contexts.

## Input Quality Is a Prerequisite

An agent's planning is only as good as the goal it receives. Vague or ambiguous task descriptions don't become precise once they enter an agent — they produce vague plans, vague tool calls, and vague outputs. This mirrors the Theory of Constraints insight that bottlenecks require "predictable, high-quality inputs": you don't fix slow legal review by adding more lawyers; you fix it by ensuring complete documentation arrives at that stage. The same logic applies to agents — speeding up execution doesn't help if the goal specification is the real constraint.

In practice this means the human-facing interface (how tasks are specified, what context is provided, how success is defined) is often more important to agent performance than any architectural choice inside the agent. Agents deployed in organizational workflows that have ambiguous upstream processes will reproduce those ambiguities, not resolve them. See [Evaluation](evaluation.md) for the related principle that you must define success before you can measure it.

A related but distinct consideration is **session-level prompt coherence**: individual prompts may each be clear, but the aggregate of all steering given during a coding session may still form an incoherent specification. Reactive prompts — "it doesn't work, try again," or "that's not quite right, redo it" — add steering without adding intent, and their sum does not accumulate into a meaningful specification. A useful diagnostic: if the full sequence of prompts given to an agent could not be synthesized into a coherent description of the desired outcome, the agent is unlikely to converge on one. Purposeful prompts that progressively refine a consistent intent compose into something like an executable specification; reactive course-corrections without a unifying view compound ambiguity. This complements the "Specs as the Primary Accountability Artifact" pattern in [Verifiable Constraints](verifiable-constraints.md), where the coherence of the human-authored specification is the root constraint on all downstream correctness.

## The Execution-Judgment Gap

A recurring observation in both research and practitioner deployments is that current agents are far more capable at **execution** — carrying out well-specified tasks — than at **judgment**: choosing which problems matter, setting direction, deciding what to work on. Anthropic's 2026 analysis of Claude's role in its own development pipeline put this plainly: Claude "excels at executing well-specified tasks but struggles with high-level judgment about which problems matter." The paper used the term "research taste" for this judgment layer and measured it directly — Claude's suggested experimental direction beat human researcher judgment 64% of the time (April 2026), up from 51% six months earlier. A 64% win rate represents a modest and growing edge, not a clear human advantage, but the gap between executing a defined task and setting the right direction remains large.

The practical consequence for agent design: the closer the task specification is to a complete executable description of the work, the more reliable the agent's planning becomes. The further upstream the agent must operate — the more it must decide what to do rather than how to do it — the more unreliable its behavior becomes. This is the planning-layer version of the "Input Quality Is a Prerequisite" insight: human judgment should push as close to the task definition as possible, leaving execution to the agent. See [Evaluation](evaluation.md) for the connection between this gap and RLVR training — models improve most on tasks where the success criterion is binary and verifiable, not on tasks requiring open-ended direction-setting.

## Human-in-the-Loop

For high-stakes or high-uncertainty tasks, agents should pause and ask for human confirmation before irreversible actions. This is a planning-level decision: the agent needs to recognize when uncertainty is high enough to warrant interruption. See [Agentic Loop](agentic-loop.md).

## See Also

- [Agentic Loop](agentic-loop.md)
- [Tool Use](tool-use.md)
- [Multi-Agent Coordination](multi-agent.md)
- [Evaluation](evaluation.md)
