# Multi-Agent Coordination

Multi-agent systems use multiple LLM instances — each with its own context, role, and tool access — to collaborate on a task. The motivation is specialization and parallelism: some tasks are too large for one context window, benefit from independent verification, or contain sub-tasks that can be done concurrently.

## Common Architectures

### Orchestrator + Subagents

An orchestrator agent decomposes a task, delegates sub-tasks to specialized subagents, and synthesizes results. The subagents may have narrower tool access and more specific instructions than the orchestrator.

This is the most common pattern in production. [LangGraph](../tools/langgraph.md) and [OpenAI Agents SDK](../tools/openai-agents-sdk.md) both provide primitives for it. [Claude Code Subagents](../tools/claude-code-subagents.md) is a well-documented, production-grade implementation: the main session delegates to named child agents (Explore, Plan, General-purpose, or custom) each running in its own context window with scoped tool access.

**Trigger tables** are a practical pattern for automatic specialist routing. Rather than requiring the developer or orchestrator to manually select which specialist handles each task, a trigger table maps file-modification patterns to agents: changes to network/sync files route to a `network-protocol-designer`; changes to coordinate or camera code route to a `coordinate-wizard`; architecture review requests route to a `systems-designer`. The table encodes institutional knowledge about domain boundaries — which parts of the codebase require which kind of expertise — making specialist selection automatic and consistent. Redundant encoding (developers consult the trigger table before making changes, not just after) further reinforces correct routing.

### Parallel Workers

Multiple agents work independently on different parts of a task simultaneously, with their results merged afterward. Useful when sub-tasks are truly independent (e.g., processing many documents in parallel).

A practical implementation pattern for parallel coding agents is git worktrees: each agent gets its own branch and working directory, preventing interference and allowing results to be reviewed independently before merging. Tools like [Superset](../tools/superset.md) operationalize this pattern by layering a management UI on top of CLI agents — monitoring status, surfacing diffs, and handling PRs across many concurrent worktrees.

### Debate / Verification

One agent produces a result; another reviews or critiques it. This can catch errors the first agent wouldn't catch itself, at the cost of additional latency and tokens.

**Internalizing debate into a single model.** The compute cost of multi-agent debate is real — multiple model invocations, long transcripts, high token counts. A 2025 research direction called Latent Agents (IMAD — Internalized Multi-Agent Debate) addresses this by distilling multi-agent debate into a single model through a two-stage post-training pipeline: (1) supervised fine-tuning on debate transcripts to teach the model the debate structure, followed by (2) reinforcement learning with dynamic reward scheduling and progressive length clipping to force the model to internalize reasoning rather than verbalize it. The resulting models match or exceed explicit multi-agent debate on GSM8K, MMLU-Pro, and Big-Bench Hard while consuming only 6–21% of the tokens (5–16x efficiency gain). A notable mechanistic finding: the internalization process creates distinct agent-specific subspaces — interpretable directions in activation space corresponding to different reasoning perspectives (e.g., Chain-of-Thought vs. Self-Critique vs. Program-of-Thought). These subspaces can be steered, making it possible to suppress malicious or undesired agent personas via negative steering with minimal performance impact. The implication for practitioners: when deployment constraints make multi-agent debate impractical (latency, cost), post-training distillation may preserve most of the accuracy benefit at a fraction of the inference cost.

### Peer Networks

Agents with equal standing collaborate, passing messages and negotiating. This pattern (popularized by [AutoGen](../tools/autogen.md)) is more flexible but harder to control and debug.

## Key Challenges

- **Context isolation**: each agent has its own context and doesn't automatically see what others have done. Explicit message-passing or shared state is required.
- **Error propagation**: a mistake by one subagent can cascade. The orchestrator needs to handle failures gracefully.
- **Coordination overhead**: spawning agents has latency and token cost; not every task benefits.
- **Trust boundaries**: when an orchestrator delegates to a subagent, deciding how much autonomy to grant — and how to validate results — is non-trivial.
- **Debugging**: tracing behavior across multiple agents is significantly harder than tracing a single agent.

## Task Decomposition: Narrow and Parallel vs. Sequential

A key design decision when applying multi-agent systems to a new problem is whether the task structure calls for narrow-and-parallel exploration or sequential execution.

Generic single-stream coding agents work well for sequential tasks like feature development: one context window follows a logical chain from requirements to implementation. They fail at tasks that require simultaneous exploration across many independent hypotheses — the agent can only pursue one thread at a time, and the search space is too large.

Cloudflare's Project Glasswing (May 2026) is a concrete example: applying Anthropic's Mythos Preview model to vulnerability discovery across a large codebase. A single-agent approach would sequentially investigate one vulnerability class at a time. Instead, Cloudflare built a seven-stage harness running approximately 50 concurrent agents, each scoped to a narrow vulnerability class:

1. **Recon** — architecture mapping and task queue generation
2. **Hunt** — parallel agents investigating specific vulnerability classes
3. **Validate** — independent adversarial review to reduce noise (see Debate/Verification above)
4. **Gapfill** — re-scanning under-explored areas
5. **Dedupe** — collapsing duplicate findings
6. **Trace** — cross-repository analysis to determine actual exploitability
7. **Report** — structured output generation

The Validate stage is particularly notable: independent agents adversarially review findings produced by the Hunt agents before those findings are promoted. This is the Debate/Verification pattern applied within a larger pipeline, not as the whole system.

A second concrete example is FuzzingBrain V2 (May 2026), an academic multi-agent system built on Google's OSS-Fuzz that takes a different approach to the same narrow-and-parallel problem. Rather than LLM-based adversarial review as the verification gate, it uses automated fuzzer reproduction: a reported vulnerability is only promoted if a fuzzer can reproduce it. This is the [Verifiable Constraints](verifiable-constraints.md) pattern applied at the output boundary — mechanically checking every finding before it leaves the pipeline, rather than relying on LLM judgment. In real-world deployment the system identified 29 zero-day vulnerabilities across 12 open-source projects, all confirmed and fixed by maintainers. The two designs illustrate a fork in how verification can be implemented in security agent pipelines: LLM-as-adversarial-reviewer (Glasswing's Validate stage) versus automated ground-truth execution (FuzzingBrain's fuzzer gate). The fuzzer approach is more mechanical and tamper-resistant but requires a runnable artifact; the LLM approach is more general but inherits the [sycophancy mitigations](evaluation.md) that apply to any LLM-as-judge design.

The lesson generalizes: when the search space has many parallel hypotheses to explore simultaneously (security auditing, large-scale code review, competitive intelligence), a narrow-and-parallel harness will outperform a sequential agent even if the sequential agent is more sophisticated. The harness's shape should match the task's shape. And when feasible, grounding the verification gate in a mechanical, executable signal — rather than LLM judgment alone — produces more reliable noise reduction.

A third concrete implementation is Anthropic's own open-source vulnerability discovery harness (github.com/anthropics/defending-code-reference-harness, June 2026), which follows the same seven-stage shape — recon, find, verify, dedupe, report, patch — with ASAN crash detection as the verification gate. Its distinguishing contribution relative to Glasswing and FuzzingBrain is the security infrastructure around the agents rather than the pipeline shape itself: each autonomous agent runs in a gVisor-isolated container with an egress allowlist restricted to the Claude API. See the "What Practitioners Actually Do" section of [Agentic Loop](agentic-loop.md) for detail on this sandboxing pattern. The harness also ships a set of interactive [Claude Code](../tools/claude-code.md) skills (`/threat-model`, `/vuln-scan`, `/triage`, `/patch`) for human-in-the-loop use alongside the fully autonomous pipeline — demonstrating a tiered autonomy design where the same workflow can run interactively or autonomously depending on operator preference.

## When to Use Multi-Agent

Use multi-agent when:
- The task genuinely exceeds one context window.
- Sub-tasks are parallel and independent.
- Independent verification would materially improve reliability.
- The task has many simultaneous hypotheses to explore rather than a single sequential chain (narrow-and-parallel vs. sequential).

Avoid it when a single capable agent would do — the coordination overhead is real.

## See Also

- [Agentic Loop](agentic-loop.md)
- [Planning](planning.md)
- [Claude Code Subagents](../tools/claude-code-subagents.md)
- [LangGraph](../tools/langgraph.md)
- [OpenAI Agents SDK](../tools/openai-agents-sdk.md)
- [AutoGen](../tools/autogen.md)
- [Superset](../tools/superset.md) — desktop orchestration layer for parallel coding agents using git worktrees
