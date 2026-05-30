# Overview

AI agents are systems that use language models to take sequences of actions toward a goal — perceiving inputs, reasoning about what to do, calling tools, and iterating until the task is done or a stopping condition is met. They represent the frontier of practical LLM deployment: harder to build reliably than a simple chatbot, but vastly more capable.

## Where Things Stand

The core building block is settled: a model in a loop with tools. What remains active and contested is everything around that loop — how to structure it, how to make it reliable, how to coordinate multiple agents, and how to evaluate whether it's working.

**Tool use** is mature. Every major model provider supports structured function calling, and the [Model Context Protocol (MCP)](tools/mcp.md) is emerging as a standard for connecting tools and resources to agents in a host-agnostic way.

**Memory** is still a patchwork. In-context history works for short tasks; anything longer needs external storage, summarization, or retrieval. No dominant pattern has emerged. See [Memory](concepts/memory.md).

**Multi-agent systems** are increasingly common — an orchestrator delegates to specialized subagents — but coordination overhead, error propagation, and debugging difficulty are real costs. See [Multi-Agent Coordination](concepts/multi-agent.md).

**Planning and reasoning** have improved dramatically with extended thinking / chain-of-thought. Models are better at decomposing tasks, but still fragile on long horizons. In May 2026, a general-purpose OpenAI reasoning model autonomously disproved a 79-year-old open problem in discrete geometry (the Erdős unit distance conjecture) using a 125-page reasoning trace — the first publicly documented case of AI autonomously solving a prominent open problem in mathematics. The result highlights cross-domain synthesis (algebraic number theory applied to combinatorial geometry) as a distinctive reasoning model strength, and signals that extended thinking at very large scale is becoming a meaningful capability axis. See [Planning](concepts/planning.md) and [Evaluation](concepts/evaluation.md).

**Coding agent reliability** crossed a meaningful threshold in late 2025. Reinforcement learning from verifiable rewards (RLVR) — training on outcomes like test pass/fail rather than human preference ratings — produced models that practitioners describe as moving from "often-work" to "mostly-work" for standard software engineering tasks. The improvement is most pronounced on tasks with clear verifiable outcomes; tasks requiring open-ended design or ambiguous specification remain unreliable. See [Evaluation](concepts/evaluation.md) for the connection between verifiable rewards and training. A complementary pattern: embedding deterministic verification gates (compilation, type-checking, tests) and type-system-enforced invariants directly into the coding loop, so that structural correctness is guaranteed by the substrate rather than delegated to prompt instructions. See the "Structural Constraints in Coding Loops" section of [Agentic Loop](concepts/agentic-loop.md).

**Computer use** — agents that directly operate GUIs and browsers — is early but moving fast, with Anthropic and OpenAI both investing heavily. See [Computer Use](concepts/computer-use.md).

**Inference speed** is an underappreciated variable in agentic loop design. Multi-step agents compound inference latency at every step, making per-call throughput a meaningful constraint for latency-sensitive workflows. The bottleneck for single-request inference is memory bandwidth, not compute — and as of mid-2026, software-layer optimizations (persistent kernels, custom GPU collectives, communication-computation overlap) are demonstrably closing the gap between commodity datacenter hardware and theoretical bandwidth limits. Agent builders should treat inference throughput as a configurable parameter, not a fixed constraint. See [Agentic Loop](concepts/agentic-loop.md).

**Evaluation** remains the hardest unsolved problem. Agents produce long, branching trajectories that are expensive to label and hard to score automatically. LLM-as-judge and trajectory-based evaluation are the current pragmatic approaches. See [Evaluation](concepts/evaluation.md).

## Key Open Questions

- How do you make long-horizon agents reliable without constant human oversight?
- What's the right abstraction for memory at scale?
- When should a system be one big agent vs. many small ones?
- How do you evaluate agent behavior efficiently and without systematic blind spots?
- How do you secure agents that have broad tool access?

## How to Use This Wiki

Start with the concept pages if you want to understand the field. Go to tool pages when evaluating or using specific frameworks. Check [log.md](log.md) to see what's changed recently.
