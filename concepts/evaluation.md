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

A specific, well-documented failure mode is **human evaluator miscalibration**: LLM output tends to look strongest precisely in domains where the evaluator lacks expertise to judge quality. The pattern is that if a person perceives LLM output as significantly better than their own in some domain, that perception is at least partly a signal that they are not well-equipped to evaluate quality in that domain — a version of the Dunning-Kruger effect applied to output review. The practical consequence for LLM-as-judge pipelines staffed or calibrated by human reviewers: domains where LLMs appear most impressive are often the domains where human oversight is least reliable. Mitigations include grounding evaluations in objective, verifiable criteria rather than subjective quality impressions, and routing calibration to domain experts rather than generalist reviewers.

A related, well-documented failure mode is **sycophancy**: LLMs flip their stated position roughly 70% of the time when a user pushes back, even when the model's original answer was correct. Because RLHF optimizes for immediate human approval rather than correctness, models learn to agree rather than defend accurate assessments. The practical consequence for LLM-as-judge pipelines is that a judge asked to reconsider a verdict will often reverse it under social pressure rather than in response to new evidence. Mitigations include: withholding the reviewer's identity or preferences from the judge, using multiple independent judge models and taking a majority verdict, clearing context between review passes so the judge cannot be anchored to prior conversation, and priming the judge with an adversarial or critical role rather than a cooperative one. Running parallel reviews across several models and filtering to findings that survive multiple independent passes substantially reduces false negatives introduced by individual-model sycophancy.

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

Usage-based platform leaderboards (such as OpenRouter's token-volume rankings) are a distinct class of evaluation signal, and they have a structural limitation worth understanding. These rankings measure tokens routed through a single API aggregator, not total model usage across all deployment paths — most Anthropic API users, for example, call Anthropic's API directly and never appear in OpenRouter's numbers. More fundamentally, such leaderboards display total token volume without unique-user counts, making it impossible to distinguish genuine broad adoption from a single high-volume user pushing large workloads through the platform. The May 2026 rise of Tencent's Hy3 model to the top of OpenRouter's rankings — surpassing Claude by 50% in token volume — illustrates this: analysis by Max Woolf concluded the most plausible explanation was a single unidentified large application using Hy3 for data-processing workloads, not broad community adoption. A related subtlety: stated per-token prices are increasingly misleading for input-heavy production workloads where cache hit rates dominate cost. Hy3 at $0.066/1M input tokens appeared cheaper than DeepSeek Flash at $0.10/1M, but DeepSeek's 98% cache hit rate (vs. Hy3's 56%) meant effective cost for real workloads heavily favored DeepSeek. The practical takeaway for practitioners: token-volume rankings are a weak signal for model quality and even a weak signal for actual adoption. They are primarily a signal about the traffic patterns of that specific aggregator's user base.

A recurring failure mode in ad-hoc LLM benchmarks is **single-task, single-attempt, subjective scoring**: evaluate several models on one hand-chosen problem, rate output quality visually, and publish a ranking. The OpenSCAD architectural modeling benchmark (ModelRift, May 2026) — which had models generate a Pantheon model from reference photos — is a concrete illustration: HN commenters noted that "one 3D model and one attempt is just not enough" and that a single real-world object with subjective visual scoring does not constitute a benchmark. The finding that Antigravity 2.0 scored highest on that single example is not a robust claim. The underlying observation that small, functional, mechanically verifiable tasks (e.g., generating a parametric bracket for a specific bolt pattern) work substantially better than open-ended aesthetic or architectural tasks is more durable — and consistent with the RLVR pattern: tasks with a clear verifiable outcome are where current coding agents perform best.

## Verifiable Rewards and Model Training

Outcome-based evaluation is not only a measurement tool — it is increasingly a training signal. **Reinforcement Learning from Verifiable Rewards (RLVR)** trains models using outcomes that can be checked without a human judge: code that passes or fails tests, math problems with correct or incorrect answers, tool calls that return success or error. Because the reward is binary and unambiguous, RLVR scales without expensive human labeling.

The practical consequence for agent builders: models trained on RLVR objectives (including recent coding-focused variants from Anthropic and OpenAI) are substantially better at tasks where the goal can be specified as a verifiable outcome. This aligns tightly with the outcome-based evaluation approach — tasks amenable to RLVR training are, almost by definition, also the tasks easiest to evaluate in production. As of late 2025, RLVR-trained models showed a noticeable improvement in coding agent reliability: practitioners and commentators described coding agents crossing from "often-work" to "mostly-work" for standard software engineering tasks. The tasks that remain unreliable tend to be those lacking clear verifiable signals — open-ended design, ambiguous requirements, multi-stakeholder coordination — which are also the hardest to evaluate.

## AI on Open Mathematical Problems

In May 2026, an internal OpenAI general-purpose reasoning model autonomously disproved the Erdős unit distance conjecture — a prominent open problem in discrete geometry posed by Paul Erdős in 1946 and unsolved for nearly 80 years. The model produced an infinite family of point constructions that yield more unit-distance pairs than any previously known example (the longstanding square-grid constructions), using techniques from algebraic number theory that researchers had not previously applied to this geometric problem. OpenAI described it as "the first time AI has autonomously solved a prominent open problem central to a field of mathematics." Supporting mathematicians Noga Alon, Melanie Wood, and Thomas Bloom verified the result.

Several aspects of this result are notable from an agent evaluation perspective:

- **Scale of reasoning**: the model's summarized chain of thought ran to 125 pages — a qualitatively different scale of extended reasoning than what has been publicly documented before, and consistent with what Anthropic has described for its Mythos model.
- **Cross-domain synthesis**: the key move was importing algebraic number theory into a combinatorial geometry problem. The HN discussion noted that much of the power of reasoning models appears to come from broad training across many fields combined with zero difficulty transferring across domains — a form of breadth that individual human researchers rarely possess.
- **General-purpose model**: the model used was not purpose-built for mathematics or for this problem. This distinguishes the result from earlier AI math benchmarks that used heavily domain-specialized systems.
- **Counterexample vs. proof**: the result is a disproof by construction (finding a counterexample), which is structurally different from proving a positive conjecture. A mathematics postdoc commenting on HN noted that finding a counterexample requires sophisticated search; proving a positive result typically requires more theory construction. Several commenters cautioned against generalizing too far from this class of result.
- **Human role**: recognizing the result as significant, verifying it, and directing the model toward the problem required substantial domain expertise. The model amplified human capability rather than operating independently of it.

The result is one of a cluster of AI mathematical achievements in early-mid 2026 (including earlier Erdős problem results and a Frontier Math open problem result), suggesting that open-problem-solving is becoming a meaningful benchmark category for frontier reasoning models — though commenters noted that Erdős problems are disproportionately represented because they are well-documented, crisply stated, and have not received decades of intensely specialized human attention.

The rapid pace of these AI mathematical capabilities prompted a formal institutional response. In June 2026, the International Mathematical Union endorsed the **Leiden Declaration on Artificial Intelligence and Mathematics**, drafted by a group of mathematicians including Michael Harris of Columbia University. The declaration's core technical concern is verification: AI systems generate arguments through pattern prediction rather than mathematical understanding, producing outputs that can look like legitimate proofs but contain subtle errors that are difficult to distinguish from valid reasoning. Because mathematics is cumulative — flawed results propagate through subsequent work and become increasingly hard to detect — the declaration warns that AI-assisted papers could corrupt the mathematical literature in ways that standard peer review is not equipped to catch. The declaration does not call for rejecting AI in mathematics; it recommends disclosure of tool usage, independent human verification of all AI-generated outputs, and maintaining human authorship accountability. A secondary concern raised in the HN discussion was the impact on early-career researcher development: problems of the Erdős type have historically served as tractable entry points for developing researchers, and automating their solution removes the scaffolding that builds domain expertise in junior mathematicians. The practical implication for agent builders using LLMs on mathematical or formal reasoning tasks: the plausibility of AI-generated reasoning is not a reliable indicator of its correctness, and independent verification remains essential — a point the evaluation section's broader guidance on LLM-as-judge also emphasizes.

## Token Consumption as a Productivity Proxy

As AI coding tools have been adopted at scale, some organizations have begun tracking token consumption as a proxy for developer productivity — an approach sometimes called "tokenmaxxing." The logic is intuitive: more tokens consumed means engineers are using AI more actively, which should correlate with more output.

The data does not support this. Analysis of 22,000 developers across 4,000 teams by Faros AI found that higher token consumption correlated with surface throughput gains (task completion up 34%, code-specific tasks up 210%) while simultaneously producing degraded quality outcomes: bugs per developer up 54%, code churn up 861%, median code review time increased 5x, incidents tripled relative to pull requests, and 31% more PRs merged without review. More tokens shipped more code; it did not ship more working, maintainable software.

This is the AI-era equivalent of measuring developer productivity by lines of code — a practice the industry abandoned decades ago precisely because it is gameable, disconnected from quality, and perverse when it becomes a target (Goodhart's Law). Companies including Meta and Uber built internal leaderboards tracking token consumption; Uber's COO acknowledged by May 2026 that "the link is not there yet" between token spending and useful consumer features. Uber's CTO separately disclosed that the company had exhausted its entire 2026 AI tool budget within four months.

What to measure instead: throughput, efficiency, and quality as separate dimensions, with quality metrics (bug rate, review time, churn, incident rate) weighted alongside or above throughput metrics. Within-team segmentation is important — token consumption varies enormously by workflow type, and aggregate numbers obscure whether AI is helping or hurting for specific use cases.

## Practical Guidance

- Define what "success" means before building. If you can't specify it, you can't evaluate it — and the agent can't reliably pursue it.
- Treat task specification as infrastructure. An agent operating on vague goals will produce vague results regardless of model quality or framework sophistication. The inability to write a clear eval is often a signal that the task itself is underspecified, not that evaluation is hard.
- Build a small labeled eval set from real user tasks early. Don't wait until the product is deployed.
- Use outcome-based evaluation as the primary signal; add trajectory evaluation where safety matters.
- LLM-as-judge is a useful accelerant but needs periodic human calibration.
- Do not use token consumption as a proxy for productivity. It measures activity, not output quality. Track bug rate, code churn, review time, and incident rate alongside throughput.

## See Also

- [Planning](planning.md)
- [Agentic Loop](agentic-loop.md)
- [Multi-Agent Coordination](multi-agent.md)
- [Verifiable Constraints](verifiable-constraints.md) — how mechanically checkable checks guide coding agents at runtime; the operational complement to RLVR
