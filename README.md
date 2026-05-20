# LLM Evals Comparison

> A PM-authored, vendor-neutral decision guide for choosing between LLM evaluation frameworks.

This is not a benchmarking paper or a feature matrix. It's a **selection guide** — structured around real team situations, lifecycle stages, and compliance needs. Skip the 14-metric comparison tables. Answer "which framework should we use?" in under 5 minutes.

**Frameworks covered:** RAGAS · DeepEval · promptfoo · Langfuse · inspect_ai · OpenAI Evals

---

## Quick Pick

| Your situation | Start here |
|---|---|
| Building a RAG pipeline, need to measure retrieval quality | [RAGAS](./frameworks/ragas.md) |
| Writing LLM tests in CI/CD, want pytest-style eval assertions | [DeepEval](./frameworks/deepeval.md) |
| Red teaming an LLM, testing adversarial prompts | [promptfoo](./frameworks/promptfoo.md) |
| Need production monitoring and observability for LLM traces | [Langfuse](./frameworks/langfuse.md) |
| EU AI Act compliance, safety evaluation, government context | [inspect_ai](./frameworks/inspect-ai.md) |
| Contributing evals to a shared benchmark registry | [OpenAI Evals](./frameworks/openai-evals.md) |
| Need to pick between RAGAS and DeepEval | [RAGAS vs DeepEval](./comparisons/ragas-vs-deepeval.md) |
| 3-person startup, where do I even start? | [By Team Type](./decision-guide/by-team-type.md) |
| We need an eval stack, not just one tool | [By Lifecycle Stage](./decision-guide/by-lifecycle-stage.md) |

---

## Why This Exists

Every existing LLM eval comparison is either:
- **Vendor-authored** (biased toward their product)
- **Engineer-authored** (feature tables, no decision framing)
- **Incomplete** (inspect_ai — used by Anthropic, DeepMind, Google for safety evals — is missing from almost every comparison)
- **EU AI Act blind** (no resource maps framework choice to compliance requirements, despite August 2026 enforcement)

This guide is written from a product team's perspective. The question isn't "which framework has the most metrics?" It's "given our team, our use case, and our risk profile, which framework should we start with on Monday?"

---

## What's Here

### Framework Profiles
Each framework has a consistent profile covering: best-for, not-great-for, setup effort, cost model, EU AI Act relevance, and how it compares to alternatives.

→ [frameworks/](./frameworks/)

### Decision Guides
Three structured guides for choosing:

- **[By Use Case](./decision-guide/by-use-case.md)** — RAG, conversational AI, red teaming, production monitoring, safety audit
- **[By Team Type](./decision-guide/by-team-type.md)** — startup / ML platform team / regulated enterprise
- **[By Lifecycle Stage](./decision-guide/by-lifecycle-stage.md)** — the insight that you probably need multiple frameworks at different stages

→ [decision-guide/](./decision-guide/)

### Head-to-Head Comparisons
- [RAGAS vs DeepEval](./comparisons/ragas-vs-deepeval.md) — the most common choice question
- [promptfoo vs inspect_ai](./comparisons/promptfoo-vs-inspect-ai.md) — both do red teaming; which one?
- [Full matrix](./comparisons/full-matrix.md) — all 6 × 10 decision criteria

→ [comparisons/](./comparisons/)

### EU AI Act Module
Which frameworks satisfy which Act requirements. Includes an eval checklist for Annex III high-risk AI systems.

→ [eu-ai-act/](./eu-ai-act/) *(coming in Session 5)*

### Templates
- [Eval Framework RFP](./templates/eval-framework-rfp.md) — scorecard for evaluating eval frameworks for your org
- [Pre-Launch Eval Plan](./templates/pre-launch-eval-plan.md) — eval strategy template before shipping an AI feature

→ [templates/](./templates/) *(coming in Session 6)*

---

## A Note on Scope

This guide covers frameworks for evaluating LLM **outputs and behaviors**. It does not cover:
- LLM benchmark leaderboards (MMLU, HumanEval, etc.)
- Model selection or comparison
- Data labeling or annotation platforms

Evals move fast. See [CHANGELOG.md](./CHANGELOG.md) for framework version tracking.

---

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md). The most useful contributions are: framework version updates, new use-case examples, and corrections to the EU AI Act mapping.

---

*Maintained by [@aadhar-build](https://github.com/aadhar-build). Not affiliated with any framework vendor.*
