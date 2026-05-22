---
title: Eval Framework RFP
parent: Templates
nav_order: 1
---

# Eval Framework Selection Scorecard

> A structured template for evaluating which LLM eval framework(s) to adopt. Fill this in before making a final recommendation to your team or leadership.

Copy this template. Score each framework you're evaluating. The weights are pre-set for three team types — adjust to match your context.

---

## Step 1 — Define Your Context

Fill these in first. They determine which weights to use.

| Field | Your answer |
|---|---|
| **Team type** | ☐ Startup / ☐ Growth-stage / ☐ Regulated enterprise |
| **Primary use case** | ☐ RAG / ☐ Conversational / ☐ Red teaming / ☐ Production monitoring / ☐ Safety/compliance |
| **EU AI Act scope?** | ☐ Yes — Annex III high-risk / ☐ Possibly / ☐ No |
| **Python team?** | ☐ Yes / ☐ Mixed / ☐ No (YAML preferred) |
| **CI/CD gate needed?** | ☐ Yes — on every PR / ☐ Yes — on releases only / ☐ No |
| **Data residency requirement?** | ☐ EU only / ☐ Self-host required / ☐ No constraint |
| **Who will maintain evals?** | ☐ Engineering / ☐ ML team / ☐ PM / ☐ Data team |

---

## Step 2 — Score Each Framework

Rate each criterion 1–5 for each framework you're evaluating.  
Then multiply by the weight column that matches your team type.

### Scoring Key

| Score | Meaning |
|---|---|
| 5 | Excellent — best-in-class for this criterion |
| 4 | Good — strong coverage, minor gaps |
| 3 | Adequate — meets minimum needs |
| 2 | Weak — requires significant workarounds |
| 1 | Poor — does not address this criterion |

---

### Criteria Weights by Team Type

| Criterion | Startup weight | Enterprise weight | EU-regulated weight |
|---|---|---|---|
| **1. Setup time to first eval** | 3× | 1× | 1× |
| **2. Metric coverage for your use case** | 2× | 3× | 2× |
| **3. CI/CD integration** | 2× | 3× | 2× |
| **4. Adversarial / red team coverage** | 1× | 2× | 3× |
| **5. Production observability** | 2× | 3× | 3× |
| **6. EU AI Act audit trail quality** | 0× | 1× | 4× |
| **7. Data residency / self-host option** | 0× | 2× | 4× |
| **8. Cost at your expected scale** | 3× | 2× | 1× |

---

### Scoring Sheet

Replace `[Framework A]` and `[Framework B]` with the frameworks you're evaluating. Add columns as needed.

| Criterion | Weight (your type) | [Framework A] score | [Framework A] weighted | [Framework B] score | [Framework B] weighted |
|---|---|---|---|---|---|
| 1. Setup time to first eval | | | | | |
| 2. Metric coverage | | | | | |
| 3. CI/CD integration | | | | | |
| 4. Adversarial coverage | | | | | |
| 5. Production observability | | | | | |
| 6. EU AI Act audit trail | | | | | |
| 7. Data residency / self-host | | | | | |
| 8. Cost at scale | | | | | |
| **Total** | | | | | |

---

## Step 3 — Qualitative Assessment

Scores don't capture everything. Answer these for each framework before deciding.

**Dealbreakers (any Yes = eliminate framework):**

| Question | [Framework A] | [Framework B] |
|---|---|---|
| Does it send data to a third party by default without opt-out? | | |
| Is it hosted outside the EU when EU data residency is required? | | |
| Does it require a skill set our team doesn't have and can't hire? | | |
| Is it abandoned or has it had no commits in 6+ months? | | |
| Does it have a vendor lock-in mechanism that's unacceptable? | | |

**Positive signals:**

| Question | [Framework A] | [Framework B] |
|---|---|---|
| Can we get a working eval running in under 2 hours? | | |
| Does the output format fit into our existing CI/reporting stack? | | |
| Is there an active community or commercial support option? | | |
| Does the documentation match what we actually need to do? | | |

---

## Step 4 — Pilot Evaluation

Before final decision, run a 1-week pilot with the top 1–2 candidates.

**Pilot protocol:**

1. **Select a representative eval task** — choose something you'll actually need to run in production, not a toy example
2. **Set a time budget** — max 4 hours to get a working eval running per framework
3. **Run the eval** — document exactly what you ran, what the output was, and what it cost
4. **Score the output** — does the result tell you something actionable?
5. **Estimate ongoing cost** — extrapolate from pilot to your expected eval frequency and volume

**Pilot output template:**

```
Framework: _______
Pilot task: _______
Time to working eval: _____ hours
Blocker(s) encountered: _______
Output format: _____ (useful / adequate / confusing)
Estimated cost per 100 samples at [judge model]: $_____
Estimated monthly cost at [X] eval runs/week: $_____
Team reaction: _______
Would we adopt this? Yes / No / Conditional on _______
```

---

## Step 5 — Decision Output

Fill in after scoring and pilot.

**Selected framework(s):** _______

**Rationale (2–3 sentences):** _______

**What we're NOT using and why:** _______

**Known gaps we're accepting:** _______

**Review trigger:** Re-evaluate if any of the following occur:
- [ ] Framework releases a major version with breaking changes
- [ ] Our use case expands significantly (e.g., from RAG to agentic)
- [ ] EU AI Act enforcement creates new audit requirements we can't satisfy
- [ ] Cost at scale exceeds $____/month
- [ ] A new framework emerges with 5k+ stars that addresses our gaps

---

## Reference: Framework Quick-Score Guide

Use this to calibrate your scores against the frameworks in this guide.

| Criterion | Strongest framework | Notes |
|---|---|---|
| Setup time | promptfoo, OpenAI Evals | YAML-first, no Python required |
| RAG metric coverage | RAGAS | Purpose-built retrieval metrics |
| CI/CD integration | DeepEval | Native pytest plugin |
| Adversarial coverage | promptfoo | OWASP LLM Top 10 + auto-generation |
| Production observability | Langfuse | Only production-capable framework |
| EU AI Act audit trail | inspect_ai | `.eval` logs, government-backed |
| Data residency (EU) | Langfuse | EU Cloud Frankfurt; self-host option |
| Cost at scale | promptfoo | Deterministic assertions are free |

→ [Full matrix](../comparisons/full-matrix.md) for the complete 6 × 10 comparison.

---

*This template is intentionally framework-agnostic. It works for any eval framework, not just the six covered in this guide.*
