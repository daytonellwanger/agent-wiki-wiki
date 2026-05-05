# Computer Use

Computer use refers to agents that interact with graphical interfaces — web browsers, desktop applications, operating systems — the same way a human user would: by reading the screen and producing mouse clicks, keystrokes, and scroll actions.

## Why It Matters

Most software in the world has no API. Computer use makes that software accessible to agents, dramatically expanding what can be automated. It's particularly valuable for legacy software, web applications with no public API, and tasks that require navigating multi-step GUIs.

## How It Works

The agent receives a screenshot (or an accessibility tree representation) of the current screen state and produces an action — a click at coordinates, a keystroke sequence, a scroll. That action is executed, a new screenshot is taken, and the loop repeats. See [Agentic Loop](agentic-loop.md).

Accessibility-tree representations (which describe UI elements as structured text) are often more token-efficient than screenshots but not always available.

## Current State (2025)

Anthropic released a computer use capability for Claude in late 2024, enabling agents to operate desktop environments via screenshot + action. OpenAI has also invested in similar browser-automation capabilities via the Operator product.

Performance is improving rapidly but still lags human accuracy on complex tasks, especially those requiring precise spatial reasoning or navigating visually dense interfaces.

## Key Challenges

- **Visual grounding**: mapping high-level intent ("click the submit button") to precise pixel coordinates is error-prone.
- **Slow feedback loops**: each step requires a screenshot, a model call, and an action execution. This is much slower than API-based tool calls.
- **Fragility**: small UI changes (button moves, page layout changes) can break agents that rely on visual landmarks.
- **Security**: an agent with mouse and keyboard access can be manipulated by malicious content on-screen (prompt injection via the UI). Sandboxing and careful permission scoping are essential.

## Use Cases

- Automating web research and form-filling
- Operating legacy enterprise software
- End-to-end testing of web applications
- Robotic process automation (RPA) replacement

## See Also

- [Tool Use](tool-use.md)
- [Agentic Loop](agentic-loop.md)
- [Planning](planning.md)
