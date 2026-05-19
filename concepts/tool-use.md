# Tool Use

Tool use (also called function calling) is the ability of a language model to invoke external functions and APIs as part of generating a response. It is the primary mechanism by which agents act on the world.

## How It Works

The model is given a list of tool definitions — name, description, and parameter schema (typically JSON Schema). When the model decides a tool is needed, it outputs a structured tool call instead of (or alongside) text. The host application executes the call, returns the result, and the model continues reasoning with that result in context.

Most providers support **parallel tool calls**: the model can request multiple tool invocations in a single turn, which are executed concurrently before the next model call.

## Tool Design

Tool design matters enormously. A tool is effectively a natural-language API — the model reads the description and schema to decide when and how to use it. Good tools are:

- **Narrow in scope**: do one thing, do it clearly.
- **Well-described**: the description should say when to use the tool, not just what it does.
- **Predictable**: side effects should be explicit so the model (and human reviewers) can reason about them.

## Model Context Protocol (MCP)

[MCP](../tools/mcp.md) is a standard protocol for connecting tools and resources to agents. Rather than hard-coding tool integrations into each agent, MCP lets tool providers expose a standard interface that any compliant host can consume. It's increasingly the standard layer for tool connectivity.

## Tradeoffs and Failure Modes

- **Tool selection errors**: the model calls the wrong tool, or fails to call one when it should.
- **Schema mistakes**: the model produces malformed arguments; robust implementations validate and re-prompt.
- **Cascading errors**: in long agentic loops, a bad tool call early can send the agent off-track with no natural recovery point.
- **Security**: tools that write files, send emails, or call external APIs are a significant attack surface. See [Evaluation](evaluation.md) for notes on testing.

## Reliability Engineering

The gap between a naive tool-calling loop and a production-grade one is mostly not about the model — it is about the execution harness. Experimental evidence from Forge (a reliability framework for self-hosted models) illustrates the scale of these gaps:

- Across 97 configurations, adding structured guardrails lifted an 8B model's accuracy from 53% to 99% on a 26-scenario agentic eval — without changing the model or weights.
- Without retry mechanisms, error recovery scores 0% across all tested models. This is not a capability gap — it is an architectural absence. The model cannot recover from malformed output if the harness doesn't detect the failure and re-prompt.
- Serving backend choice mattered more than model choice in some configurations: the same 12B weights running in prompt-injection mode vs. native function-calling mode showed a 75-percentage-point accuracy difference across backends.

### Guardrail Patterns

A reliability layer for tool-calling typically includes some combination of:

- **Rescue parsing** — recover from malformed tool calls using heuristic or model-based extraction before giving up
- **Retry nudges** — on failure, re-prompt with targeted guidance toward correct tool usage rather than generic retries
- **Step enforcement** — ensure the agent follows required workflow sequences (optional; trading flexibility for reliability)
- **Context management** — VRAM- or token-budget-aware compaction to prevent context overflow from disrupting long agentic runs
- **Mode anchoring** — injecting a synthetic "respond" tool to prevent models from exiting tool-calling mode prematurely when tools are still available

These patterns are composable and can be applied as middleware inside a custom orchestration loop without replacing the underlying model or framework.

## See Also

- [Model Context Protocol (MCP)](../tools/mcp.md)
- [Planning](planning.md)
- [Agentic Loop](agentic-loop.md)
- [Multi-Agent Coordination](multi-agent.md)
