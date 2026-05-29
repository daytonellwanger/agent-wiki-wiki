# Computer Use

Computer use refers to agents that interact with graphical interfaces — web browsers, desktop applications, operating systems — the same way a human user would: by reading the screen and producing mouse clicks, keystrokes, and scroll actions.

## Why It Matters

Most software in the world has no API. Computer use makes that software accessible to agents, dramatically expanding what can be automated. It's particularly valuable for legacy software, web applications with no public API, and tasks that require navigating multi-step GUIs.

## How It Works

The agent receives a screenshot (or an accessibility tree representation) of the current screen state and produces an action — a click at coordinates, a keystroke sequence, a scroll. That action is executed, a new screenshot is taken, and the loop repeats. See [Agentic Loop](agentic-loop.md).

Accessibility-tree representations (which describe UI elements as structured text) are often more token-efficient than screenshots but not always available.

## Current State (2026)

Anthropic released a computer use capability for Claude in late 2024, enabling agents to operate desktop environments via screenshot + action. OpenAI has also invested in similar browser-automation capabilities via the Operator product.

Performance is improving rapidly. As of May 2026, Claude Opus 4.8 scores 84% on Online-Mind2Web, a web browser task benchmark. Performance still lags human accuracy on complex tasks, especially those requiring precise spatial reasoning or navigating visually dense interfaces.

## Key Challenges

- **Visual grounding**: mapping high-level intent ("click the submit button") to precise pixel coordinates is error-prone.
- **Slow feedback loops**: each step requires a screenshot, a model call, and an action execution. This is much slower than API-based tool calls.
- **Fragility**: small UI changes (button moves, page layout changes) can break agents that rely on visual landmarks.
- **Security**: an agent with mouse and keyboard access can be manipulated by malicious content on-screen (prompt injection via the UI). Sandboxing and careful permission scoping are essential.

## Agent-Authored Scripts: A Hybrid Approach for Production RPA

Pure computer use — running a model inference per action at execution time — is expensive and fragile at production scale. An alternative pattern: use an agent with computer use capability to *author* a deterministic Python script that encodes the automation, then execute that script directly on subsequent runs without further model inference.

This separates two concerns:

1. **Authoring phase**: the agent observes the GUI, reasons about the workflow, and writes a script capturing the exact sequence of actions.
2. **Execution phase**: the script runs deterministically, with no model involved. Model inference only re-enters when the script fails (e.g., a UI change breaks an assumption), triggering a self-healing loop that patches or rewrites the affected portion.

The practical benefit is latency, cost, and reliability at scale. Script execution is faster and cheaper than model-per-action inference, and errors are localized rather than cascading. Proponents claim this approach reaches 93–96% click accuracy in production versus 80–85% for pure computer use models.

The tradeoff: script maintenance. When UIs change substantially, the script may require significant revision. The self-healing loop partially addresses this, but complex layout changes may still require human intervention or full re-authoring. This pattern is most appropriate for workflows that are high-frequency, relatively stable, and run against the same application version repeatedly — such as EHR data entry, ERP report generation, or similar enterprise RPA targets.

## Use Cases

- Automating web research and form-filling
- Operating legacy enterprise software
- End-to-end testing of web applications
- Robotic process automation (RPA) replacement

## See Also

- [Tool Use](tool-use.md)
- [Agentic Loop](agentic-loop.md)
- [Planning](planning.md)
