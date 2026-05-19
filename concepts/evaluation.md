# Evaluation

Evaluation is the practice of measuring agent performance — determining whether an agent reliably accomplishes its goals, where it fails, and whether a change improves things. It's widely regarded as the hardest unsolved problem in production agent development.

## Why Agents Are Hard to Evaluate

Standard NLP evaluation assumes a fixed input-output pair: given this prompt, the correct output is this string. Agents don't work that way. They produce **trajectories** — sequences of actions, tool calls, and observations — and the same goal can be accomplished via many different valid trajectories. Evaluating whether a trajectory was good is expensive, ambiguous, and doesn't reduce to a single score easily.

## Approaches

### Outcome-Based Evaluation

Measure only the final result: did the agent accomplish the task? This is the cleanest signal but is blind to efficiency, safety, and near-misses. It also requires a verifiable ground truth, which isn't always available.

**Examples**: did the code pass the tests? Is the file in the right place? Did the query return the right answer?

### Trajectory Evaluation

Evaluate the steps taken, not just the final outcome. Useful for catching unsafe intermediate actions (e.g., deleting the wrong file en route to success) or identifying where agents fail on long-horizon tasks.

Trajectory evaluation requires either human labelers or automated heuristics. Both are expensive to build and maintain.

### LLM-as-Judge

Use a second language model to score the first model's outputs. Scales cheaply, but the judge model has its own biases, calibration issues, and blind spots. Works best when combined with human review to calibrate the judge.

### Human Evaluation

Ground truth, but expensive and slow. Best reserved for final validation, high-stakes decisions, or calibrating automated evaluators.

## Benchmarks

Several benchmarks have emerged specifically for agentic tasks:
- **SWE-bench**: software engineering tasks (fix this GitHub issue in this repo). Widely used to evaluate coding agents.
- **WebArena** / **WorkArena**: web browsing and UI interaction tasks.
- **GAIA**: general assistant tasks requiring tool use and multi-step reasoning.
- **Terminal-Bench**: command-line and shell task completion. Tests whether an agent can accomplish real terminal workflows end-to-end.
- **MCP Atlas**: evaluates an agent's ability to correctly invoke and chain MCP tools across multi-step tasks. As [MCP](../tools/mcp.md) adoption has grown, dedicated benchmarks for it have emerged alongside the broader agentic benchmark landscape.

Benchmark scores are informative but not sufficient — models can overfit to benchmark distributions, and benchmark tasks often don't represent your specific production use case.

## Verifiable Rewards and Model Training

Outcome-based evaluation is not only a measurement tool — it is increasingly a training signal. **Reinforcement Learning from Verifiable Rewards (RLVR)** trains models using outcomes that can be checked without a human judge: code that passes or fails tests, math problems with correct or incorrect answers, tool calls that return success or error. Because the reward is binary and unambiguous, RLVR scales without expensive human labeling.

The practical consequence for agent builders: models trained on RLVR objectives (including recent coding-focused variants from Anthropic and OpenAI) are substantially better at tasks where the goal can be specified as a verifiable outcome. This aligns tightly with the outcome-based evaluation approach — tasks amenable to RLVR training are, almost by definition, also the tasks easiest to evaluate in production. As of late 2025, RLVR-trained models showed a noticeable improvement in coding agent reliability: practitioners and commentators described coding agents crossing from "often-work" to "mostly-work" for standard software engineering tasks. The tasks that remain unreliable tend to be those lacking clear verifiable signals — open-ended design, ambiguous requirements, multi-stakeholder coordination — which are also the hardest to evaluate.

## Practical Guidance

- Define what "success" means before building. If you can't specify it, you can't evaluate it — and the agent can't reliably pursue it.
- Treat task specification as infrastructure. An agent operating on vague goals will produce vague results regardless of model quality or framework sophistication. The inability to write a clear eval is often a signal that the task itself is underspecified, not that evaluation is hard.
- Build a small labeled eval set from real user tasks early. Don't wait until the product is deployed.
- Use outcome-based evaluation as the primary signal; add trajectory evaluation where safety matters.
- LLM-as-judge is a useful accelerant but needs periodic human calibration.

## See Also

- [Planning](planning.md)
- [Agentic Loop](agentic-loop.md)
- [Multi-Agent Coordination](multi-agent.md)
