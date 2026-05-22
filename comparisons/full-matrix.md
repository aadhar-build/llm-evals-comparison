---
title: Full Comparison Matrix
parent: Comparisons
nav_order: 3
---

# Full Comparison Matrix

> All 6 frameworks × 10 decision criteria. Use this to compare across dimensions, not to find one winner — the right answer is almost always a combination.

---

## The Matrix

| Criterion | RAGAS | DeepEval | promptfoo | Langfuse | inspect_ai | OpenAI Evals |
|---|---|---|---|---|---|---|
| **Setup effort** | Medium | Medium | Low | Low | High | Low |
| **Coding required** | Python | Python | YAML (no code) | Python (decorator) | Python | YAML + JSONL |
| **RAG evaluation** | ● ● ● ● ● | ● ● ● ○ ○ | ● ● ○ ○ ○ | ● ● ○ ○ ○ | ● ○ ○ ○ ○ | ● ○ ○ ○ ○ |
| **Red teaming / adversarial** | ○ ○ ○ ○ ○ | ● ● ○ ○ ○ | ● ● ● ● ● | ○ ○ ○ ○ ○ | ● ● ● ● ○ | ● ● ○ ○ ○ |
| **Production observability** | ○ ○ ○ ○ ○ | ○ ○ ○ ○ ○ | ○ ○ ○ ○ ○ | ● ● ● ● ● | ○ ○ ○ ○ ○ | ○ ○ ○ ○ ○ |
| **EU AI Act support** | ● ○ ○ ○ ○ | ● ● ○ ○ ○ | ● ● ● ○ ○ | ● ● ● ● ○ | ● ● ● ● ● | ● ● ○ ○ ○ |
| **LLM judge required** | Yes (most) | Yes (most) | Optional | Optional | Yes (most) | Yes (model-graded) |
| **Self-host option** | Yes | Yes | Yes | Yes (Docker/Helm) | Yes | Yes |
| **CI/CD integration** | Manual | Native (pytest) | Native | N/A | Manual | Manual |
| **Cost at scale** | Low–Med | Low–Med | Low | Free–Usage | Low–Med | Low |

---

## Reading the Matrix

**● ● ● ● ●** = best-in-class for this criterion  
**○ ○ ○ ○ ○** = not designed for this use case  
Ratings are relative within this set of 6, not against the entire ecosystem.

"LLM judge required" means: the core use of the framework requires LLM API calls for evaluation (not just for the model under test). This is relevant for cost estimation and for offline/air-gapped environments.

---

## Dimension Deep-Dives

### RAG Evaluation

| Framework | What it measures | Key gap |
|---|---|---|
| **RAGAS** | Context precision, context recall, faithfulness, answer relevancy | No CI/CD runner; no non-RAG metrics |
| **DeepEval** | Contextual relevancy, faithfulness, answer relevancy | No context recall; weaker retrieval depth than RAGAS |
| **promptfoo** | Basic output matching, LLM-rubric | No retrieval-specific metrics |
| **Langfuse** | Scores you define and attach | Doesn't compute retrieval metrics natively; use with RAGAS |
| **inspect_ai** | Task-completion scoring | No RAG-specific metrics |
| **OpenAI Evals** | Match-based, model-graded | No retrieval metrics |

→ For RAG: **RAGAS** is the only choice with purpose-built retrieval metrics. Use DeepEval as the CI harness around RAGAS scores.

---

### Red Teaming / Adversarial

| Framework | Coverage | Audit value |
|---|---|---|
| **promptfoo** | OWASP LLM Top 10, auto-generated attacks, 12+ harm categories | Developer report |
| **inspect_ai** | AgentHarm (110 behaviors), HarmBench, CyberSecEval | Audit-grade `.eval` logs |
| **DeepEval** | Bias, toxicity, hallucination checks | Quality metrics, not security |
| **OpenAI Evals** | Model-graded safety checks | Benchmark registry |
| **RAGAS** | None | — |
| **Langfuse** | None (observation only) | — |

→ For red teaming: **promptfoo** for coverage and speed; **inspect_ai** for regulatory documentation.

---

### Production Observability

| Framework | What it captures | Key feature |
|---|---|---|
| **Langfuse** | Every inference: input, output, latency, tokens, cost | Live trace collection + scoring |
| **DeepEval** | Batch eval results | No live trace |
| **RAGAS** | Batch eval scores | No live trace |
| **promptfoo** | Pre-production test results | No production capability |
| **inspect_ai** | Eval run logs | Batch only |
| **OpenAI Evals** | Benchmark run results | Batch only |

→ For production: **Langfuse** is the only choice. None of the others are designed for live traffic.

---

### EU AI Act Support

| Framework | Articles addressed | Audit trail | Data residency |
|---|---|---|---|
| **inspect_ai** | Art. 9, 13, 15, 17, Annex IV, Art. 40 | ● ● ● ● ● `.eval` logs | Self-host |
| **Langfuse** | Art. 9, 10, 13, 17, Annex IV | ● ● ● ● ● trace exports | EU Cloud (Frankfurt) |
| **promptfoo** | Art. 9, 13, Annex IV | ● ● ○ ○ ○ HTML/JSON report | Self-host |
| **DeepEval** | Art. 13, 17 | ● ● ○ ○ ○ pytest output | Self-host |
| **OpenAI Evals** | Annex IV (citable benchmarks) | ● ○ ○ ○ ○ run logs | OpenAI infra |
| **RAGAS** | Art. 17 (indirect) | ● ○ ○ ○ ○ metric outputs | Self-host |

→ For EU AI Act: **inspect_ai** for safety evidence; **Langfuse** for production audit trail. Both are needed for Annex III high-risk AI systems.

---

### Cost at Scale

All frameworks using LLM judges have costs that scale with: samples × metrics × judge model cost.

| Framework | Cost model | 1,000 samples estimate |
|---|---|---|
| **RAGAS** | LLM judge per metric | $5–30 (GPT-4o); $0.50–3 (GPT-4o-mini) |
| **DeepEval** | LLM judge per metric | $5–30 (GPT-4o); $0.50–3 (GPT-4o-mini) |
| **promptfoo** | LLM for attack generation (one-time) + deterministic assertions | $0.01–0.10 (regex/JSON); $10–50 (full red team) |
| **Langfuse** | Free self-host; usage-based cloud | Free–$29/mo + observations |
| **inspect_ai** | LLM judge per sample | $5–30 (GPT-4o); $0.50–3 (GPT-4o-mini) |
| **OpenAI Evals** | Model API cost only | $0.10–5 depending on eval type |

**Cost reduction strategies:**
- Use GPT-4o-mini or Claude Haiku as judge — 10–20× cheaper, acceptable quality for most metrics
- Sample production traffic (10–20%) for scoring in Langfuse; don't judge every inference
- promptfoo: deterministic assertions (regex, JSON schema) cost nothing; reserve LLM rubric for complex cases
- RAGAS: `NonLLMContextPrecisionWithReference` is free for binary precision cases

---

### CI/CD Integration

| Framework | Integration | How |
|---|---|---|
| **DeepEval** | Native | `pytest` or `deepeval test run`; JUnit output |
| **promptfoo** | Native | `npx promptfoo eval --ci`; exit code on assertion failure |
| **RAGAS** | Manual | Wrap in script; assert on metric thresholds yourself |
| **Langfuse** | N/A | Production tool — not a CI tool |
| **inspect_ai** | Manual | Wrap `inspect eval` in CI step; parse `.eval` log |
| **OpenAI Evals** | Manual | `oaieval` CLI; parse output |

→ For CI gates: **DeepEval** is the only framework with native pytest integration. promptfoo works well for adversarial regression gates specifically.

---

## Recommended Stacks

Rather than picking one, most production teams need a combination. Here are the most common patterns:

### Minimum viable (RAG startup)
```
RAGAS → measure retrieval quality
Langfuse → trace production traffic
promptfoo → red team before first launch
```

### Standard (growth-stage product team)
```
RAGAS + DeepEval → eval depth + CI/CD runner
Langfuse → production observability
promptfoo → adversarial regression in CI
```

### Compliance-first (regulated EU company)
```
DeepEval → CI/CD quality regression
inspect_ai → safety eval + audit-grade logs
Langfuse (EU Cloud / self-hosted) → production audit trail
```

### Research / frontier safety
```
inspect_ai → primary eval framework
inspect_evals → published benchmarks (AgentHarm, GAIA, etc.)
OpenAI Evals → contributing domain benchmarks to public registry
```

---

## The Framework Nobody Mentions: What's Missing from All of Them

No single framework in this guide provides:

- **End-to-end coverage** from development through production — you need at least two
- **Business-level metrics** (task deflection rate, user satisfaction, cost-per-query) — these live in your product analytics stack
- **Fine-grained attribution** (which retrieved chunk caused the hallucination?) — emerging in RAGAS but not mature anywhere
- **Multilingual eval depth** — most benchmarks and metrics assume English; non-English RAG products are under-served

These gaps are worth knowing before you design your eval strategy.

---

→ [RAGAS vs DeepEval](./ragas-vs-deepeval.md) · [promptfoo vs inspect_ai](./promptfoo-vs-inspect-ai.md) · [Decision Guide](../decision-guide/) · [Framework Profiles](../frameworks/)
