---
title: By Lifecycle Stage
parent: Decision Guide
nav_order: 3
---

# By Lifecycle Stage

> The key insight most eval guides miss: **you don't pick one framework**. You need different tools at different stages of development. This guide maps the right framework(s) to each stage.

---

## The Core Insight

LLM eval is not a one-time exercise. The questions you're answering at each stage are fundamentally different:

| Stage | The question | Right tool |
|---|---|---|
| **0 — Prototype** | "Does this idea even work?" | Informal / vibes |
| **1 — Development** | "Does my output quality meet the bar I defined?" | RAGAS, DeepEval |
| **2 — Pre-production** | "What can an adversary do to this?" | promptfoo, inspect_ai |
| **3 — Production** | "Is quality holding in the real world?" | Langfuse |
| **4 — Compliance/audit** | "Can I prove this system is safe?" | inspect_ai, OpenAI Evals |

Most teams have Stage 1. Most are missing Stages 2 and 3. Almost no team without regulatory pressure has Stage 4.

---

## Stage 0 — Prototype

**Goal:** Validate that your approach is directionally correct before investing in eval infrastructure.

**What "eval" looks like here:**
- Manual inspection of 20–50 outputs
- Spot-checking with domain experts
- A/B prompts with human preference ratings

**Frameworks useful here:** None formally. A notebook with a judge model (GPT-4o or Claude rating outputs 1–5) is sufficient.

**When to move to Stage 1:** When you've decided the approach works and you're iterating on prompt design or retrieval configuration. Once you're making changes that require you to verify "did this get better or worse?", you need measurement — not vibes.

---

## Stage 1 — Development Evals

**Goal:** Systematic measurement of output quality during active development. Catch regressions before they reach production.

### For RAG systems → RAGAS

RAGAS is purpose-built for RAG pipelines. The metrics it provides — context precision, context recall, faithfulness, answer relevancy — directly measure whether the retrieval step is working and whether the generation is grounded in retrieved context.

```bash
pip install ragas
```

**Typical eval loop:**
1. Define a golden dataset: 50–200 question/ground-truth/context triples
2. Run your RAG pipeline on each sample
3. Score with RAGAS metrics
4. Set thresholds: e.g., faithfulness > 0.85, context precision > 0.75
5. Run in CI on every retrieval or prompt change

**Watch for:** The v0.3 → v0.4 migration was breaking. If onboarding, start with v0.4 directly.

**Cost:** ~$0.50–$5 per 100 samples with GPT-4o as judge (most metrics require an LLM judge). NonLLM-based context precision is free but limited to binary cases.

→ [Full RAGAS profile](../frameworks/ragas.md)

---

### For general LLM (conversational, agents, summarization) → DeepEval

DeepEval provides pytest-style LLM unit testing. You define test cases with inputs, expected outputs (or rubrics), and thresholds. Tests fail if metrics fall below threshold — same pattern as unit tests.

```bash
pip install deepeval
```

**Typical eval loop:**
```python
from deepeval.metrics import GEval, AnswerRelevancyMetric
from deepeval.test_case import LLMTestCase
from deepeval import assert_test

def test_customer_support_response():
    test_case = LLMTestCase(
        input="How do I cancel my subscription?",
        actual_output=run_my_llm("How do I cancel my subscription?"),
        expected_output="Cancellation is available in Settings > Account > Cancel."
    )
    assert_test(test_case, [AnswerRelevancyMetric(threshold=0.8)])
```

**What DeepEval does well:** Multi-metric testing in one framework, pytest CI integration, G-Eval (custom rubric) for open-ended tasks.

**What to watch for:** PostHog telemetry is on by default (opt-out: `deepeval login --telemetry-opt-out`). Enterprise deployments should review the data privacy docs before onboarding.

→ [Full DeepEval profile](../frameworks/deepeval.md)

---

### Can I use RAGAS and DeepEval together?

Yes. The common pattern:
- **RAGAS** for retrieval-specific metrics: context precision, context recall, faithfulness
- **DeepEval** as the test framework: wraps RAGAS scores into CI-runnable assertions

You can pass RAGAS metric scores into DeepEval test cases. This gives you retrieval-quality metrics (RAGAS) inside a structured test runner (DeepEval).

---

## Stage 2 — Pre-Production Adversarial Testing

**Goal:** Find what an adversary — or a badly worded production prompt — can make your system do before your users find it.

This stage is almost universally skipped by teams without a security mandate. That's a mistake: prompt injection, PII leakage, and jailbreak vulnerabilities are real and discoverable before launch.

### For systematic red teaming → promptfoo

promptfoo generates adversarial test cases automatically across attack categories: prompt injection, jailbreaks, PII extraction, indirect RAG injection, OWASP LLM Top 10.

```yaml
# promptfooconfig.yaml
redteam:
  purpose: "Customer support chatbot for a fintech app"
  plugins:
    - owasp:llm
    - pii:direct
    - harmful:financial-crime
  strategies:
    - jailbreak
    - prompt-injection
```

```bash
npx promptfoo redteam run
```

**Time to first red team run:** 30–60 minutes. No Python required.

**Cost:** ~$1–$10 per campaign depending on attack scope and model used for generation.

→ [Full promptfoo profile](../frameworks/promptfoo.md)

---

### For EU-regulated, government, or high-stakes contexts → inspect_ai

promptfoo and inspect_ai both do red teaming. The difference:
- **promptfoo** produces a developer-facing pass/fail report
- **inspect_ai** produces audit-grade structured logs (`.eval` format) with full input/output attribution, suitable for regulatory documentation

If you're building for a regulated industry (fintech, healthtech, government) or preparing for EU AI Act conformity assessment, the output of your red teaming needs to be documentable. inspect_ai's log format is purpose-built for this.

```bash
pip install inspect-ai
inspect eval inspect_evals/src/inspect_evals/agentharm/agentharm.py \
  --model anthropic/claude-3-5-sonnet-20241022
```

→ [Full inspect_ai profile](../frameworks/inspect-ai.md)

---

### Should I run both?

If you have the capacity: yes. A practical split:
- **promptfoo** for fast iteration and broad attack surface coverage during development
- **inspect_ai** for a structured, documentable safety eval run before deployment

For most teams, starting with promptfoo in Stage 2 is the pragmatic choice. Add inspect_ai when compliance requirements create the need for audit-grade evidence.

---

## Stage 3 — Production Observability

**Goal:** Continuous measurement of output quality on real production traffic. Catch drift, degradation, and edge cases that your pre-launch eval dataset didn't include.

### → Langfuse

Langfuse traces every LLM inference in production: inputs, outputs, latency, token cost, model used. You can attach scores to any trace — either automatically (using an LLM judge on a sample) or via human annotation pipelines.

```python
from langfuse.decorators import observe

@observe()
def my_llm_pipeline(user_message: str) -> str:
    # All LLM calls inside are automatically traced
    ...
```

**What Langfuse gives you that pre-production eval doesn't:**
- Real user inputs, not synthetic datasets
- Longitudinal quality trends (is faithfulness drifting down over 30 days?)
- Cost visibility per query, per user, per session
- Input that feeds back into your Stage 1 golden dataset

**EU data residency:** Langfuse's EU Cloud (cloud.langfuse.com) is hosted in Germany (AWS eu-central-1). GDPR-compliant, DPA available. Self-hosting also available for full data sovereignty.

→ [Full Langfuse profile](../frameworks/langfuse.md)

---

## Stage 4 — Compliance and Safety Audit

**Goal:** Produce documented evidence that your AI system was evaluated against safety and compliance requirements. Needed for EU AI Act conformity assessment, enterprise procurement security reviews, and regulated industry certifications.

### → inspect_ai (primary)

inspect_ai's `.eval` log format is designed for this purpose: structured, timestamped, model-attributed, and archivable. The UK AISI (which maintains inspect_ai) is involved in developing the international AI safety standards that will become the harmonised standards referenced in EU AI Act Article 40. Using inspect_ai keeps your evaluation methodology aligned with the emerging regulatory standard.

**Recommended evals to run pre-deployment for high-risk AI systems:**

| Eval | From inspect_evals | What it covers |
|---|---|---|
| AgentHarm | ✓ | 110 adversarial behaviors, 11 harm categories |
| HarmBench | ✓ | Broad harmful behavior |
| CyberSecEval | ✓ | Security risks |
| Domain-specific | Write custom Task | Your specific regulatory context |

**Pin to commit:** inspect_ai has no semver. For audit reproducibility, pin to a specific commit in your requirements file.

→ [Full inspect_ai profile](../frameworks/inspect-ai.md)

---

### → OpenAI Evals (for registry-based benchmark evidence)

If your domain has publicly available benchmark evals (or you can contribute one), running against the OpenAI Evals registry provides citable, reproducible evidence of model quality on published benchmarks.

This is particularly relevant for EU AI Act Annex IV (Technical Documentation): a published eval in the registry that tests compliance-relevant knowledge is citable evidence of pre-deployment evaluation.

→ [Full OpenAI Evals profile](../frameworks/openai-evals.md)

---

## Recommended Stacks by Context

### Minimum viable eval stack (startup, RAG product)
```
Stage 1: RAGAS                    → retrieval quality measurement
Stage 2: promptfoo (basic)        → red team before first public launch
Stage 3: Langfuse                 → production tracing and cost visibility
```

### Full eval stack (growth-stage, EU market)
```
Stage 1: RAGAS + DeepEval         → metric depth + CI regression
Stage 2: promptfoo + inspect_ai   → broad adversarial + audit-grade safety
Stage 3: Langfuse                 → production observability (EU Cloud)
Stage 4: inspect_ai               → pre-deployment safety certification
```

### Regulated enterprise (fintech, healthtech, govtech)
```
Stage 1: DeepEval                 → pytest-integrated CI eval suite
Stage 2: inspect_ai               → government-grade safety eval with audit logs
Stage 3: Langfuse (self-hosted)   → full data sovereignty, GDPR-compliant tracing
Stage 4: inspect_ai + OpenAI Evals → conformity assessment documentation
```

---

## Quick Diagnostic

If you're not sure which stage you're at:

1. **Can you answer "what metric determines if my LLM output is good enough?"** — If not, you're at Stage 0. Don't pick a framework yet; define the metric first.

2. **Do you have a golden dataset of 50+ examples with expected outputs?** — If not, start there before instrumenting anything.

3. **Have you run a single adversarial test against your system?** — If not, add promptfoo before shipping. It takes 30 minutes and often finds something.

4. **Do you know what your production outputs look like?** — If you're guessing based on your test set, you need Langfuse. Real traffic is always more interesting than synthetic datasets.

5. **Is your system classified as high-risk under EU AI Act Annex III?** — If yes (loan decisions, biometric identification, safety-critical infrastructure, etc.), inspect_ai Stage 4 is not optional.

---

→ Back to [Decision Guide](./README.md) | [By Use Case](./by-use-case.md) | [By Team Type](./by-team-type.md)
