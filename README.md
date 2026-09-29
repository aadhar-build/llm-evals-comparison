---
title: Home
nav_order: 1
description: "A PM-authored, vendor-neutral decision guide for choosing between RAGAS, DeepEval, promptfoo, Langfuse, inspect_ai, and OpenAI Evals."
permalink: /
---

# LLM Evals Comparison

> A PM-authored, vendor-neutral decision guide for choosing between LLM evaluation frameworks.

This is a selection guide, not a benchmarking paper or a feature matrix. It is organised around the situations teams are actually in: what they are building, what stage they are at, and what they have to comply with. The aim is to answer "which framework should we use?" in under five minutes, without reading a 14-metric comparison table first.

Frameworks covered: RAGAS, DeepEval, promptfoo, Langfuse, inspect_ai and OpenAI Evals.

---

## Quick pick

| Your situation | Start here |
|---|---|
| Building a RAG pipeline, need to measure retrieval quality | [RAGAS](./frameworks/ragas.md) |
| Writing LLM tests in CI/CD, want pytest-style eval assertions | [DeepEval](./frameworks/deepeval.md) |
| Red teaming an LLM, testing adversarial prompts | [promptfoo](./frameworks/promptfoo.md) |
| Need production monitoring and observability for LLM traces | [Langfuse](./frameworks/langfuse.md) |
| EU AI Act compliance, safety evaluation, government context | [inspect_ai](./frameworks/inspect-ai.md) |
| Contributing evals to a shared benchmark registry | [OpenAI Evals](./frameworks/openai-evals.md) |
| Need to pick between RAGAS and DeepEval | [RAGAS vs DeepEval](./comparisons/ragas-vs-deepeval.md) |
| Need to pick between promptfoo and inspect_ai | [promptfoo vs inspect_ai](./comparisons/promptfoo-vs-inspect-ai.md) |
| Want all 6 frameworks side-by-side | [Full matrix](./comparisons/full-matrix.md) |
| 3-person startup, where do I even start? | [By team type](./decision-guide/by-team-type.md) |
| We need an eval stack, not just one tool | [By lifecycle stage](./decision-guide/by-lifecycle-stage.md) |
| Building a system subject to EU AI Act | [EU AI Act module](./eu-ai-act/) |
| Need to choose an eval framework for my org | [Eval framework RFP](./templates/eval-framework-rfp.md) |
| Shipping an AI feature, need a pre-launch eval plan | [Pre-launch eval plan](./templates/pre-launch-eval-plan.md) |

---

## Why this exists

The LLM eval comparisons I could find had one of four problems. They were written by a vendor, so they favour that vendor's product. They were written by engineers, so they are feature tables with no help on making the decision. They leave out inspect_ai, which Anthropic and Google DeepMind use for safety evals and which almost never appears in comparisons. Or they ignore the EU AI Act entirely, even though the Act's August 2026 deadline for high-risk systems turns framework choice into a compliance question.

This guide is written from a product team's point of view. The question is not "which framework has the most metrics?" but "given our team, our use case and our risk profile, which one should we start with on Monday?"

---

## What's here

### Framework profiles

Each framework gets the same profile: what it is good for, what it is not good for, setup effort, cost model, EU AI Act relevance, and how it compares to the alternatives. See [frameworks/](./frameworks/).

### Decision guides

Three guides, each cutting the same six frameworks a different way:

- [By use case](./decision-guide/by-use-case.md): RAG, conversational AI, red teaming, production monitoring, safety audit
- [By team type](./decision-guide/by-team-type.md): startup, ML platform team, regulated enterprise
- [By lifecycle stage](./decision-guide/by-lifecycle-stage.md), which makes the case that you probably need more than one framework as the product matures

See [decision-guide/](./decision-guide/).

### Head-to-head comparisons

- [RAGAS vs DeepEval](./comparisons/ragas-vs-deepeval.md), the most common choice teams face
- [promptfoo vs inspect_ai](./comparisons/promptfoo-vs-inspect-ai.md), since both do red teaming
- [Full matrix](./comparisons/full-matrix.md), all 6 frameworks against 10 decision criteria

See [comparisons/](./comparisons/).

### EU AI Act module

Which frameworks satisfy which Act requirements. Maps Articles 9, 10, 13, 15 and 17 and Annex IV to concrete framework choices, and includes a five-phase pre-deployment checklist for Annex III high-risk AI systems.

- [EU AI Act overview](./eu-ai-act/README.md): enforcement timeline, Annex III categories, the articles that create eval obligations
- [Framework to article mapping](./eu-ai-act/framework-mapping.md): master mapping table plus a red-flag matrix of compliance gaps
- [High-risk AI checklist](./eu-ai-act/high-risk-checklist.md): five-phase pre-deployment checklist and recommended stacks by risk tier

See [eu-ai-act/](./eu-ai-act/).

### Templates

Working documents you can copy and use as they are.

- [Eval framework RFP](./templates/eval-framework-rfp.md): scoring template for choosing eval frameworks for your org, weighted by team type
- [Pre-launch eval plan](./templates/pre-launch-eval-plan.md): metrics, baselines, thresholds and a sign-off gate before an AI feature ships

See [templates/](./templates/).

---

## Scope

This guide covers frameworks for evaluating LLM outputs and behaviours. It does not cover LLM benchmark leaderboards (MMLU, HumanEval and the like), model selection or comparison, or data labelling and annotation platforms.

Evals move fast. [CHANGELOG.md](./CHANGELOG.md) tracks framework versions.

---

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md). The most useful contributions are framework version updates, new use-case examples, and corrections to the EU AI Act mapping.

---

*Maintained by [@aadhar-build](https://github.com/aadhar-build). Not affiliated with any framework vendor.*
