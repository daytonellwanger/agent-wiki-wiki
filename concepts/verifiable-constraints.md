# Verifiable Constraints

A verifiable constraint is a check on agent output that produces a deterministic pass/fail signal — no human judgment required. Tests pass or fail. The linter fires or it doesn't. The type checker either accepts the code or reports an error. The build either succeeds or it doesn't.

The practical importance of verifiability comes from how coding agents actually work. A useful mental model: agents are like blind squirrels running through a maze. They aren't reasoning toward a goal the way a human engineer does — they're sampling from a space of possible outputs. What shapes that space is the walls. A maze without walls produces random walks. A maze with well-placed walls produces convergence. Verifiable constraints are the walls.

This has a direct corollary: the value of a constraint scales with its verifiability. "Make the code readable" is unverifiable — there is no mechanical check that fires when readability fails. "All tests must pass" is verifiable — the build either passes or it doesn't. Unverifiable constraints get encoded in prompts and produce probabilistic compliance. Verifiable constraints get encoded in tooling and produce deterministic enforcement.

## Types of Verifiable Constraints

### Tests

Tests are the primary constraint. They are executable specifications: each test encodes one behavior the agent must produce, and the test suite as a whole defines the envelope the agent must stay within. Writing tests before writing code (test-driven development) is wall-placement done deliberately — the agent can only succeed by staying within the walls the tests define.

Anthropic's Claude Code documentation identifies this as the single highest-leverage practice: "Give Claude a way to verify its work — Claude performs dramatically better when it can verify its own work." Without tests, the agent is the only feedback loop and every mistake requires human attention. With tests, the agent can self-correct autonomously.

### Linters

Linters encode architectural intent as executable rules: naming conventions, module boundary enforcement, import restrictions, observability requirements, security checks. Unlike instructions in a rules file (which produce probabilistic compliance), a lint rule that blocks a commit is deterministic enforcement.

The key insight: linters can encode "how we build here" in a way that survives across sessions. Instructions drift. Rules don't. The lint error message itself becomes a corrective prompt — "use `logger.info({event: 'name'...})`" enables self-correction; "violation detected" does not. The error message must be actionable, not just indicative.

A recurring antipattern in LLM-generated codebases without architectural linting is **pattern inconsistency**: each new feature is implemented in a slightly different way rather than following the established conventions of the codebase. The agent encounters each feature in isolation, samples from its training distribution, and produces superficially correct but stylistically incoherent code — different error handling idioms, different data access patterns, different naming conventions across files added in different sessions. The resulting codebase is harder to review, refactor, and extend. Linters that encode the project's conventions (naming, module boundaries, error handling shape) catch this drift and force convergence, turning "how we build here" from a suggestion into a mechanical constraint.

### Type Checkers

Type systems apply statically, before any code runs, and catch a broad class of errors that tests may miss. A well-typed codebase constrains the solution space before the agent begins — many wrong implementations are simply unrepresentable. Strong type systems with explicit module signatures and interface contracts are among the most powerful passive constraints available.

### CI Gates

CI gates determine which constraints are hard walls (block the PR) versus soft suggestions (emit a warning). The choice matters: a warning the agent never sees is not a constraint. A gate that terminates the task after two CI cycles (as Stripe does) is. The distinction between "we have tests" and "tests must pass before merge" is the distinction between a suggestion and a wall.

A practical corollary for agentic workflows: move deterministic checks into pre-commit scripts and git hooks rather than relying on CLAUDE.md directives. Agents frequently skip listed steps in instruction files; a pre-commit hook that blocks the commit is enforced mechanically. The general principle: any check that can be made deterministic should be wired into the toolchain, not the prompt.

### Property-Based Tests

Property-based tests validate invariants across large or random input spaces rather than a fixed set of examples. Where unit tests say "given this input, expect this output," property-based tests say "for any input in this space, this invariant must hold." This provides stronger coverage guarantees and catches edge cases that example-based tests don't enumerate.

Property-based tests are particularly well-suited for encoding behavioral invariants that should hold universally — "no two traffic light directions are simultaneously green," "the account balance never goes negative" — rather than behaviors that only hold for specific inputs.

### Formal Contracts

Design-by-contract formalizes preconditions, postconditions, and invariants as executable assertions. The Agent Behavioral Contracts framework extends this to autonomous agent sessions: hard constraints that must never be violated, soft constraints that allow transient violations if remedied within a bounded recovery window, and governance policies over actions. The practical implication is a formal analog to CI gates, with mathematical bounds on behavioral drift rather than ad hoc thresholds.

Formal verification — using SMT solvers or model checkers to guarantee code satisfies a specification — is the strongest end of this spectrum, at the cost of significant specification overhead. Property-based testing sits between unit tests and formal verification.

## The Feedback Loop

Constraints only guide behavior when their output reaches the agent. The mechanism is a feedback loop: constraint fires, error signal is returned, agent revises, constraint re-runs. Several design decisions determine whether this loop is effective.

**Separation of generation and verification.** The agent that wrote the code should not be the agent that verifies it. A validator agent with a separate prompt and fresh context can fail the work cleanly. An agent reviewing its own output is biased toward passing it. This is the same insight behind the writer/reviewer pattern: Session A implements, Session B reviews.

**Error signal quality.** The constraint's output must be actionable. A stack trace that shows exactly which assertion failed, on which input, with the actual vs. expected values, enables rapid self-correction. An opaque error code does not. Investing in error message quality is investing in the feedback loop.

**Loop bounds.** An unbounded remediation loop can spin indefinitely. Hard limits on the number of correction cycles — analogous to Stripe's two-CI-cycle cap — prevent this. The agent either converges within the allowed budget or the task terminates for human review.

## The Harness Engineering Frame

Martin Fowler systematizes verifiable constraints under the term "harness engineering": the practice of building a structured layer of checks around an agent to make its behavior reliable.

Two orthogonal distinctions organize the design space:

**Feedforward vs. feedback.** Feedforward controls steer behavior before the agent acts — rules files, architectural documentation, type system configuration. Feedback controls detect issues after the agent acts — tests, linters, type checker runs, CI gates. An agent with only feedforward controls encodes rules but never learns whether they worked. An agent with only feedback controls keeps repeating the same mistakes. Both are required.

**Computational vs. inferential.** Computational controls (tests, linters, type checkers) are deterministic and run in milliseconds. Inferential controls (LLM-based code review) are probabilistic and run in seconds to minutes. Computational controls are the reliable enforcement layer. Inferential controls are useful for catching semantic issues but cannot be the only gate.

A practical technique for making inferential controls more reliable is **multi-model parallel review**: run several models (e.g., Claude, Codex, a specialized bugbot) against the same code independently, with context cleared between passes, and triage findings by severity. Bugs flagged by multiple independent models have substantially lower false-positive rates than single-model findings. This is the inferential-control analog of quorum voting: no individual model's output is trusted unconditionally, but convergent findings across models are high-signal. Tiered classification (critical / high / medium / low) lets reviewers focus effort on findings most likely to matter in production. This pattern directly addresses [LLM sycophancy](evaluation.md#llm-as-judge) — because each model reviews a fresh context, no model can be anchored or pressured by prior assessments.

Harness quality is partly a function of codebase structure: strongly typed languages with clear module boundaries afford richer harnesses. Technical debt compounds — harnesses are most needed where they're hardest to build.

## TDD as Constraint Discipline

Test-driven development maps directly onto the verifiable constraints pattern. Writing tests before implementation is a formal act of wall-placement. The agent receives the tests as a specification, executes, receives failure signals, and revises — the loop repeats until the walls are satisfied.

A three-phase workflow synthesizes the pattern: write a complete specification before touching code; write comprehensive tests before the agent implements; run linting in strict mode continuously. The more complete the specification and the closer the tests are to it, the more autonomously the agent can operate. Verifiable constraints are what enable autonomy, not what restrict it. An agent with well-specified walls can be trusted to run without supervision. An agent without walls cannot.

## Specs as the Primary Accountability Artifact

When code is generated by an LLM at a rate faster than humans can read it, the traditional expectation that engineers understand and are accountable for every line of code becomes impractical. One response is to shift where rigor lives: rather than reviewing generated code, engineers review and own the **specification** — a standardized document (often Markdown) that describes what the code must do — while tests verify that the generated code conforms to it.

In this model the spec becomes the "unit of knowledge" that humans read, discuss, version-control, and are accountable for. Code is treated more like compiled output than like a human-readable artifact. The accountability chain is: engineer owns spec → tests enforce spec → generated code satisfies tests. The practical workflow often looks like: write or iterate on a plan/spec file (with or without agent assistance), then hand the spec to the agent for implementation.

This reframes the open question about test correctness: if specs are the primary human artifact, then the quality of the spec is the real constraint on system correctness — a wrong spec produces code that passes all tests and still does the wrong thing. Rigor relocates, but it does not disappear. See the "Open Questions" section below.

One important caveat, raised in community discussion of this pattern: determining whether generated code fully conforms to a natural-language spec remains an unsolved problem. Tests provide partial verification, not complete conformance checking. The degree of organizational risk this represents depends on the quality and completeness of both the spec and the test suite — and on whether the organization has consciously accepted that tradeoff rather than assumed it away.

## Relationship to Reinforcement Learning

Verifiable constraints are not only a runtime engineering pattern — they are also the training substrate for current frontier coding models. Reinforcement Learning from Verifiable Rewards (RLVR) trains models using outcomes that can be checked without a human judge: code that passes or fails tests, math problems with correct or incorrect answers. Because the reward is binary and unambiguous, RLVR scales without expensive human labeling.

The practical consequence: models trained on RLVR objectives are substantially better at tasks where the goal can be specified as a verifiable outcome. Tasks that remain unreliable — open-ended design, ambiguous requirements — tend to be exactly those lacking clear verifiable signals. The agent and the training signal want the same thing: a wall to bump into. See [Evaluation](evaluation.md).

## Open Questions

- **Harness coherence**: as rules files, linters, and CI gates accumulate, they can contradict each other. There is no standard tooling to audit harness coherence.
- **Constraint coverage**: analogous to code coverage, there is no established metric for what fraction of desired behavior space is mechanically checked.
- **Functional behavior validation**: current test suites provide partial coverage of functional correctness; fully autonomous functional validation without human oversight remains unsolved.
- **Test correctness**: the remediation loop depends on the tests being correct. If the tests are wrong, the agent converges on code that satisfies incorrect specifications. Test correctness is underappreciated as a risk. In spec-driven workflows (where the spec is the primary human artifact), the risk propagates upstream: a wrong spec produces wrong tests, which produce confidently wrong code. The entire verification chain rests on spec quality.

## See Also

- [Evaluation](evaluation.md) — measuring agent performance, including outcome-based evaluation and RLVR
- [Agentic Loop](agentic-loop.md) — the perceive-reason-act cycle that constraints operate within
- [Planning](planning.md) — how agents decompose goals; constraint placement is a planning-level decision
