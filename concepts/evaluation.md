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

Benchmark scores are informative but not sufficient — models can overfit to benchmark distributions, and benchmark tasks often don't represent your specific production use case.

## Practical Guidance

- Define what "success" means before building. If you can't specify it, you can't evaluate it.
- Build a small labeled eval set from real user tasks early. Don't wait until the product is deployed.
- Use outcome-based evaluation as the primary signal; add trajectory evaluation where safety matters.
- LLM-as-judge is a useful accelerant but needs periodic human calibration.

## See Also

- [Planning](planning.md)
- [Agentic Loop](agentic-loop.md)
- [Multi-Agent Coordination](multi-agent.md)
