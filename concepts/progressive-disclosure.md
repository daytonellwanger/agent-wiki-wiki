# Progressive Disclosure

Progressive disclosure is a design principle that reveals complexity incrementally rather than all at once — loading only what is needed at each step and making deeper capability available on demand. In agentic AI, it is a core technique for managing what an agent loads into its own context window about its capabilities and reference materials.

## What the Agent Loads

Agents that load all available tools, documentation, and reference material upfront face attention dilution (the model's focus degrades as the window fills), instruction interference (behavioral guidelines get buried), and tool schema bloat (a naive setup can accumulate 50,000+ tokens of JSON before reasoning begins). Progressive disclosure addresses this by loading context on demand rather than all at once.

Single-file manifests (CLAUDE.md, .cursorrules) are the simplest implementation — but they fail beyond modest codebases. A 1,000-line prototype can be fully described in a single file; a 100,000-line system cannot. As projects grow, agents lose coherence across sessions and developers spend increasing time resolving routine errors that a well-structured knowledge architecture would prevent. The three-tier pattern below is the practical response.

**Agent Skills**. The Agent Skills standard (released December 2025, adopted across Claude Code, Gemini CLI, and others) implements a three-tier architecture. At the top level, only skill metadata — name and description (~80–100 tokens per skill) — is loaded at startup. When a user request matches a skill, the full instruction file (~2,000 tokens) is fetched from the filesystem and loaded into context. Supporting resources and executable scripts are loaded only if referenced during execution; script output, not the script itself, enters the context. Anthropic's benchmarks show this reduces token consumption by ~85% compared to loading all tools upfront.

**Index-First Loading**. Provide the agent with a structured index describing available resources; it fetches only the files relevant to the current subtask. This parallels how hierarchical disclosure works in UX: a table of contents is always visible, chapters only when needed.

**Phase-Based Loading**. Structure complex tasks into phases — research, planning, execution, review — and load context appropriate to each phase, releasing or swapping rather than accumulating. This is progressive disclosure applied to the lifecycle of a single task rather than to a menu of capabilities.

### What Specialist Agents Should Contain

The three-tier architecture raises a design question for the middle tier: what should a specialist agent's specification actually contain? The answer is weighted strongly toward domain knowledge rather than behavioral instructions. In a production deployment on a 108K-line codebase (283 sessions, 16,522 agent turns), over half of each specialist agent specification's content was domain facts, code patterns, formulas, and debugging tables — not behavioral instructions. One specialist (a networking agent) was 915 lines, ~65% domain knowledge, embedding the full determinism theory for the subsystem.

The argument for pre-loading rather than relying on retrieval: complex domains require complete mental models. Partial knowledge causes subtle, hard-to-detect bugs — an agent that retrieves individual facts about a determinism protocol may still miss edge cases that a fully pre-loaded agent catches by construction. Retrieval supplies relevant snippets; pre-loaded specialist context supplies the synthesized, cross-subsystem view that complex debugging requires.

A practical signal for what to codify: *"If you explained it twice, write it down."* When domain context is repeatedly re-derived or re-explained across sessions, it belongs in a specialist agent specification rather than in on-demand documentation.

A secondary payoff: pre-loaded context shifts the communication pattern. When agents already carry deep domain knowledge, human prompts can stay short and substantive — one production deployment found 80% of prompts were ≤100 words, a workload profile that is only achievable when developers aren't spending prompts re-explaining the system.

### Tradeoffs

**Reliability of implicit activation**. Context-side progressive disclosure that depends on the model recognizing when to load capabilities is not perfectly reliable. Evaluations found skills activated in only 44% of test cases under fully implicit triggering. Explicit fallback instructions in configuration (e.g., "Before starting any task, identify which docs are relevant and read them first") improve consistency at low cost.

**Security**. Capabilities loaded on demand can be a vector for prompt injection — malicious instructions injected into skill content or external resources get loaded into the context window when the skill activates.

## See Also

- [Context Management](context-management.md) — Managing context window limits, compaction, and retrieval
- [Tool Use](tool-use.md) — How agents invoke external tools and APIs
- [Claude Code](../tools/claude-code.md) — Production example with Agent Skills and deferred tool schemas
