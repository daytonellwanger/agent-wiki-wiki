# The Agentic Loop Itself Is Simple

A functional agent — even one capable of editing code — can be built in under 400 lines of code. The core mechanism is not a research breakthrough; it is an LLM, a loop, and a tool-calling protocol.

## Context

The [agentic loop](../concepts/agentic-loop.md) is the perceive-reason-act cycle at the heart of every agent. [Tool use](../concepts/tool-use.md) is the mechanism by which agents take actions. Together, these two ideas are sufficient to explain how a capable agent works. Amp's engineering team published a walkthrough demonstrating this with a minimal code-editing agent implementation.

## Reasoning

The core loop has three moving parts: maintain a conversation history, send a prompt to the model, and execute whatever tools the model requests. That's it. A working code-editing agent needs a file reader, a file writer, a shell executor, and a loop that feeds results back to the model. The protocol for tool use is a communication convention, not a complex algorithm.

## Counterarguments

Simple to implement is not the same as simple to get right. A loop that works in demos can fail in production due to context pressure, poor stopping conditions, error handling gaps, or model behavior that is hard to anticipate. The loop's simplicity can also be misleading: the hard problems (reliability, safety, latency) are real — they just live elsewhere.

There is also a version of this view that undersells the difficulty of tool design. The loop may be simple, but deciding which tools to expose, and how to design their interfaces so the model uses them correctly, involves significant judgment.

## See Also

- [Agentic Loop](../concepts/agentic-loop.md) — The perceive-reason-act cycle in detail
- [Tool Use](../concepts/tool-use.md) — How agents invoke external tools
