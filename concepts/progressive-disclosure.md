# Progressive Disclosure

Progressive disclosure is a design principle that reveals complexity incrementally rather than all at once — loading only what is needed at each step and making deeper capability available on demand. In agentic AI, it is a core technique for managing what an agent loads into its own context window about its capabilities and reference materials.

## What the Agent Loads

Agents that load all available tools, documentation, and reference material upfront face attention dilution (the model's focus degrades as the window fills), instruction interference (behavioral guidelines get buried), and tool schema bloat (a naive setup can accumulate 50,000+ tokens of JSON before reasoning begins). Progressive disclosure addresses this by loading context on demand rather than all at once.

**Agent Skills**. The Agent Skills standard (released December 2025, adopted across Claude Code, Gemini CLI, and others) implements a three-tier architecture. At the top level, only skill metadata — name and description (~80–100 tokens per skill) — is loaded at startup. When a user request matches a skill, the full instruction file (~2,000 tokens) is fetched from the filesystem and loaded into context. Supporting resources and executable scripts are loaded only if referenced during execution; script output, not the script itself, enters the context. Anthropic's benchmarks show this reduces token consumption by ~85% compared to loading all tools upfront.

**Index-First Loading**. Provide the agent with a structured index describing available resources; it fetches only the files relevant to the current subtask. This parallels how hierarchical disclosure works in UX: a table of contents is always visible, chapters only when needed.

**Phase-Based Loading**. Structure complex tasks into phases — research, planning, execution, review — and load context appropriate to each phase, releasing or swapping rather than accumulating. This is progressive disclosure applied to the lifecycle of a single task rather than to a menu of capabilities.

### Tradeoffs

**Reliability of implicit activation**. Context-side progressive disclosure that depends on the model recognizing when to load capabilities is not perfectly reliable. Evaluations found skills activated in only 44% of test cases under fully implicit triggering. Explicit fallback instructions in configuration (e.g., "Before starting any task, identify which docs are relevant and read them first") improve consistency at low cost.

**Security**. Capabilities loaded on demand can be a vector for prompt injection — malicious instructions injected into skill content or external resources get loaded into the context window when the skill activates.

## See Also

- [Context Management](context-management.md) — Managing context window limits, compaction, and retrieval
- [Tool Use](tool-use.md) — How agents invoke external tools and APIs
- [Claude Code](../tools/claude-code.md) — Production example with Agent Skills and deferred tool schemas
