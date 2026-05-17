# When Should You Use a Subagent Rather Than Having CLAUDE.md Reference a Separate Instruction File?

Claude Code offers two ways to encode a specialized procedure: as a [subagent](../tools/claude-code-subagents.md) definition that the main session delegates to, or as a separate markdown file that CLAUDE.md references and Claude reads on demand. Both solve the context-bloat problem — neither loads the instructions unless they're needed. The question is what else subagents add.

## Background

[CLAUDE.md](../tools/claude-code.md) is a per-project markdown file read at the start of every Claude Code session. It can point Claude to other files: "when the user asks you to do X, read `instructions/x.md` and follow it." Claude then fetches and reads that file at the moment it becomes relevant, keeping the instructions out of the initial context. This is a lightweight form of [progressive disclosure](../concepts/progressive-disclosure.md) that requires no additional tooling.

A [subagent](../tools/claude-code-subagents.md) is a separate agent instance spawned via the Agent tool. It also loads its instructions on demand — its system prompt is only in play when the subagent is invoked. But it differs from the referenced-file approach in several structural ways: it runs in its own fresh context window, it can have its own tool allowlist and model, and it returns only a final result to the parent.

## Perspectives

### Both approaches achieve on-demand instruction loading; subagents add context isolation

The progressive disclosure argument favors both approaches equally — neither loads instructions until they're needed. The meaningful difference is what happens during execution. When Claude follows a referenced markdown file, it does the work itself in the main conversation: every file read, search result, and intermediate tool call accumulates in the main context. A subagent doing the same work keeps all that intermediate output inside its own context window, handing back only the result. For procedures that produce a lot of noisy intermediate output — log analysis, broad codebase searches, documentation retrieval — this isolation is the primary reason to use a subagent. See [Context Management](../concepts/context-management.md).

### Subagents can enforce tool restrictions; referenced files cannot

A subagent definition can specify `tools: Read, Bash` to limit which tools it can call, regardless of what the parent session allows. A referenced markdown file is just instructions — it cannot technically constrain the parent Claude's tool access. If a procedure should not be able to write files, or should be sandboxed to a specific set of operations, only a subagent can enforce that boundary.

### Subagents can use a different model; referenced files cannot

A subagent can be pinned to a specific model — Haiku for cheap exploration, Opus for careful reasoning — without changing the main session's model. With a referenced file, the main session's model does the work at its full cost.

### Subagents can run in parallel; referenced files run sequentially in the main thread

Multiple subagents can run concurrently in the background. Work done by reading a referenced markdown file is sequential in the main thread. For independent subtasks, parallelism can be a significant speedup.

### Referenced files are simpler and keep the parent in the loop

The referenced-file approach has real advantages: no new file type to learn, no delegation boundary to reason about, and the parent session sees all intermediate work — which makes iterative refinement easier. If a procedure requires frequent back-and-forth or the parent needs to act on intermediate results, keeping the work in the main thread is the right call. Subagents are one-shot: they receive a prompt, do their work, and return a result. They cannot ask the parent questions mid-task (in foreground mode, clarifying questions surface to the user, not the parent agent).

## Answer

Both approaches solve progressive disclosure. Use the referenced-file approach when the procedure is simple, the parent needs to see intermediate results, or iterative back-and-forth is expected. Use a subagent when any of these are true:

- **Output volume**: The procedure produces verbose intermediate output (search hits, test results, document fetches) that would crowd the main context.
- **Tool or model specificity**: The procedure should be restricted to certain tools, or run more cheaply or carefully on a different model.
- **Parallelism**: Multiple instances of the procedure could run concurrently to save time.
- **Hard encapsulation**: You want a strict boundary — the procedure can't accidentally call tools it shouldn't, and its internal state can't interfere with the main conversation.

A useful frame: a referenced markdown file is an *instruction Claude reads and follows*. A subagent is a *capability Claude delegates to*. The delegation boundary is the difference.

## See Also

- [Claude Code](../tools/claude-code.md) — What CLAUDE.md is and how it is read
- [Claude Code Subagents](../tools/claude-code-subagents.md) — Subagent architecture, tool restrictions, model selection, and when to use them
- [Progressive Disclosure](../concepts/progressive-disclosure.md) — The design principle behind on-demand context loading
- [Context Management](../concepts/context-management.md) — Managing context window limits and what accumulates in the main thread
