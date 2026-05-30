# OpenRouter

OpenRouter is a unified API gateway that routes requests to 400+ language models across multiple providers (Anthropic, OpenAI, Google, Meta, Mistral, and many others) through a single endpoint. It acts as an abstraction layer between agent code and model providers, handling provider-specific API differences, failover, and cost/latency optimization.

## What Problem It Solves

Without a routing layer, agent code must manage multiple provider SDKs, API key sets, and response formats. When models are updated or a better option emerges, switching requires code changes. OpenRouter provides a single OpenAI-compatible endpoint so agents can switch or try models by changing a model ID string, not by rewriting integration code.

## Key Features

- **Unified API**: OpenAI-compatible endpoint; minimal changes needed to existing code.
- **Model breadth**: 400+ models, including open-weights models on third-party inference providers.
- **Provider-level failover**: Automatically retries failed requests against alternative providers serving the same model.
- **Intelligent routing**: Routes by cost, latency, or quality (via "meta" models that select an appropriate backend for a given prompt).
- **API key management**: Mint sub-keys with per-key spend limits and expiry. Useful for exposing AI features to external users without sharing root credentials.
- **Billing caps**: Hard spend limits at the account and key level — most direct providers do not offer this. Valuable for public-facing applications where abuse could generate unexpected costs.
- **Enterprise features**: Workspaces, zero-data-retention policies, spend management, and guardrails.
- **Free tier**: A set of 20+ models available at no cost (used by some open-source projects to enable free end-user access without requiring an API key).

## Tradeoffs

**Cost markup.** OpenRouter charges a roughly 5% surcharge over provider list prices. For exploratory development or teams actively switching between models, this is negligible. For high-volume production agentic systems running a fixed set of models, the overhead can become significant and may push teams toward direct provider APIs.

**Indirect provider access.** OpenRouter is not a provider — it routes to other providers. Usage leaderboards and token volume stats reflect only traffic routed through OpenRouter, not total usage across the industry. This makes OpenRouter's model popularity rankings an unreliable signal of actual adoption; a single high-volume application can dominate the rankings. See [Evaluation](../concepts/evaluation.md) for more on this failure mode.

**Market consolidation risk.** OpenRouter is most valuable during a period of rapid model release and experimentation. If the market consolidates around a small number of providers, the switching-cost benefit shrinks and the surcharge becomes harder to justify. The value proposition depends partly on continued provider fragmentation.

**VC-backed and proprietary.** OpenRouter is not open-source. The "open" in the name refers to access to many models, not to the software itself. Some practitioners have noted concern about consumer-unfriendly changes under VC incentives; self-hosted alternatives exist but are not widely adopted.

## When to Use OpenRouter

- **Exploration and early development**: Lowest-friction way to try many models before committing to a provider.
- **Multi-provider agent pipelines**: When the workflow benefits from routing different task types to different models.
- **Cost and reliability management**: When billing caps, failover, and spend tracking are needed without building that infrastructure in-house.
- **Exposing AI capabilities externally**: API key management with per-key limits simplifies secure delegation to users or downstream systems.

## When to Prefer Direct Provider Access

- **High-volume production**: The 5% surcharge and an extra network hop add up at scale.
- **Provider-specific features**: Prompt caching, fine-tuning, batching, and proprietary tuning options are often unavailable or degraded through intermediaries.
- **Latency-sensitive loops**: An extra hop adds latency; for [agentic loops](../concepts/agentic-loop.md) that compound per-step latency, this matters.

## Growth and Scale

As of May 2026, OpenRouter processes weekly token volumes of approximately 25 trillion tokens (up from 5 trillion six months prior), projecting over one quadrillion tokens annually. The developer base is reported at 8 million. These figures come from the company's own Series B announcement and should be treated as marketing claims; the actual proportion of total LLM API traffic they represent is unknown.

## See Also

- [Agentic Loop](../concepts/agentic-loop.md) — Inference latency compounds across loop steps; routing adds one hop
- [Context Management](../concepts/context-management.md) — Managing costs across many model calls
- [Evaluation](../concepts/evaluation.md) — Usage-based leaderboard limitations (OpenRouter's model rankings as an unreliable signal)
- [Tool Use](../concepts/tool-use.md) — Multi-provider tool calling patterns
