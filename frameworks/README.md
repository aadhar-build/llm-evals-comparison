---
title: Framework Profiles
nav_order: 2
has_children: true
permalink: /frameworks/
---

# Framework Profiles

Each framework profile follows the same structure so you can compare like-for-like.

## How to Read These Profiles

**At a Glance table** — Five-dot ratings (● = strong, ○ = weak) across 7 dimensions. Use for quick scanning.

**Best For / Not Great For** — Where the framework genuinely shines vs. where you'll hit friction. Written from usage experience, not the vendor's marketing page.

**Tradeoffs vs. Alternatives** — When to pick this over the closest competitor, and when not to.

**Integration Effort** — Realistic time-to-first-eval for a small team with an existing LLM application.

**Cost at Scale** — Open source is rarely free at scale. Includes LLM API costs when the framework uses LLM-as-judge by default.

**EU AI Act Relevance** — Which Article requirements this framework helps satisfy. Relevant for teams building high-risk AI systems under Annex III, or preparing for August 2026 enforcement.

**Version Tracked** — Framework versions move fast. Each profile notes when it was last verified.

---

## The Six Frameworks

| Framework | Primary strength | Stars | License |
|---|---|---|---|
| [RAGAS](./ragas.md) | RAG pipeline evaluation | ~14k | Apache 2.0 |
| [DeepEval](./deepeval.md) | LLM unit testing in CI/CD | ~16k | Apache 2.0 |
| [promptfoo](./promptfoo.md) | Red teaming and adversarial testing | ~21k | MIT |
| [Langfuse](./langfuse.md) | Production observability and tracing | ~28k | MIT / Commercial |
| [inspect_ai](./inspect-ai.md) | Safety evaluation, government-grade | ~2k | MIT |
| [OpenAI Evals](./openai-evals.md) | Benchmark registry, YAML-based | ~19k | MIT |

---

## What's Not Here

This guide covers the six frameworks above. Notable omissions and why:

- **LangSmith** — Excellent product, but commercial-first (LangChain Inc.). Langfuse is the open-source alternative with equivalent capabilities. Covered in comparisons where relevant.
- **Braintrust** — Strong step efficiency metrics and CI/CD integration. Excluded from v1 to keep scope manageable. Will be added in v2.
- **Arize Phoenix** — Good for ML observability teams. Overlap with Langfuse is high for LLM-specific use cases.
- **TruLens** — Predecessor to many patterns now in RAGAS and DeepEval. Less actively maintained.
- **Giskard** — Strong for ML model testing (bias, drift). Less LLM-native than others covered here.
