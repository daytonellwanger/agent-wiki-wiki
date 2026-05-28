# If CLAUDE.md Should Be Short and Focused on Preventing Mistakes, How Does Claude Build a Rich Understanding of Project Context?

The advice for [CLAUDE.md](../tools/claude-code.md) is strict: every line should prevent Claude from making a mistake; anything else should be removed. But surely telling Claude about project architecture, goals, and design history would help it make better decisions. Why does the advice push in the opposite direction?

## Background

[CLAUDE.md](../tools/claude-code.md) is Claude Code's per-project instruction file, read unconditionally at the start of every session. Anthropic's own CLAUDE.md reportedly contains only build commands, test invocations, and a "Gotchas" section — a handful of lines, not a project encyclopedia.

The concern driving the short-CLAUDE.md advice is [attention dilution](../concepts/progressive-disclosure.md): when the always-loaded context grows, behavioral constraints get buried and the model's ability to follow them degrades. A bloated CLAUDE.md doesn't make Claude smarter about the project; it makes it less reliable about the things that most need correcting. See [Context Management](../concepts/context-management.md) for the underlying mechanism.

## Perspectives

### CLAUDE.md has a specific, narrow job

CLAUDE.md corrects Claude's *default behavior* where it would otherwise make recurring mistakes — things the code itself doesn't communicate. "Use pnpm, not npm." "Run tests with this exact command." "Never modify `config/prod.yaml` directly." Architecture and goals are different in kind: they help Claude understand the system, but they don't correct a behavioral default that would otherwise cause repeated failures.

The "keep it short" advice is really about recognizing this distinction. Load what only CLAUDE.md can deliver (behavioral corrections); let the codebase itself carry the architectural knowledge.

### Project context reaches Claude through other, more appropriate channels

Claude Code is a filesystem agent — it can read any file in the project. Architecture docs, READMEs, design notes, and inline comments are all accessible. When Claude reads a file to complete a task, it encounters this context naturally, where it's relevant. Front-loading it all into CLAUDE.md adds it to every session whether or not it's needed, which is the bloat pattern [progressive disclosure](../concepts/progressive-disclosure.md) is designed to avoid.

For richer structured context, CLAUDE.md can reference other files: "When the user asks about the data pipeline, read `docs/pipeline-architecture.md`." This is [index-first loading](../concepts/progressive-disclosure.md) — Claude fetches only what the current task requires.

For deep domain-specific work, [specialist subagents](../tools/claude-code-subagents.md) can carry extensive pre-loaded knowledge. The progressive disclosure page documents production deployments where individual specialist specs run 900+ lines, with the majority being domain facts, code patterns, and debugging tables — not behavioral instructions. The key difference from CLAUDE.md is that a specialist agent loads this context only when specifically invoked, not at every session start.

### Short means each remaining line carries real weight

If CLAUDE.md contains both "never force-push main" and three paragraphs of architectural context, the model cannot easily distinguish the load-bearing constraints from the background information. A short file makes every remaining line feel mandatory — it signals to the model that CLAUDE.md is high-stakes, not optional background reading.

## Answer

The "keep it short" advice applies specifically to CLAUDE.md's role as the unconditionally-loaded startup file. Its job is behavioral correction — the things Claude would otherwise get wrong by default. Architecture, goals, and design context belong in a different layer:

- In the codebase itself (READMEs, comments, docs), where Claude reads them as needed during task execution
- In files CLAUDE.md references on demand ("when the user asks about X, read `docs/x.md`")
- In [specialist subagent](../tools/claude-code-subagents.md) specifications for tasks with complex domain requirements

The answer to "how does Claude understand my project?" is not "put it all in CLAUDE.md" — it's to structure the project so a capable filesystem agent can discover what it needs when it needs it. Short CLAUDE.md and deep project understanding are not in tension; they target different layers.

## See Also

- [Claude Code](../tools/claude-code.md) — CLAUDE.md role and best practices
- [Progressive Disclosure](../concepts/progressive-disclosure.md) — On-demand context loading and the three-tier architecture
- [Context Management](../concepts/context-management.md) — Attention dilution and why less context can mean better performance
- [Claude Code Subagents](../tools/claude-code-subagents.md) — Specialist agents that pre-load deep domain context on demand
- [When Should You Use a Subagent Rather Than Having CLAUDE.md Reference a Separate Instruction File?](subagent-vs-claude-md.md) — Choosing between on-demand instruction files and delegation
