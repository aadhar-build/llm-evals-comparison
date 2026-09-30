---
title: Eval Framework RFP
parent: Templates
nav_order: 1
---

# Eval framework selection scorecard

> A structured template for evaluating which LLM eval framework(s) to adopt. Fill this in before making a final recommendation to your team or leadership.

Copy this template and score each framework you are considering. The weights are preset for three team types. Adjust them to match your context.

---

## Step 1: define your context

Fill these in first. They determine which weights to use.

| Field | Your answer |
|---|---|
| **Team type** | ☐ Startup / ☐ Growth-stage / ☐ Regulated enterprise |
| **Primary use case** | ☐ RAG / ☐ Conversational / ☐ Red teaming / ☐ Production monitoring / ☐ Safety/compliance |
| **EU AI Act scope?** | ☐ Yes, Annex III high-risk / ☐ Possibly / ☐ No |
| **Python team?** | ☐ Yes / ☐ Mixed / ☐ No (YAML preferred) |
| **CI/CD gate needed?** | ☐ Yes, on every PR / ☐ Yes, on releases only / ☐ No |
| **Data residency requirement?** | ☐ EU only / ☐ Self-host required / ☐ No constraint |
| **Who will maintain evals?** | ☐ Engineering / ☐ ML team / ☐ PM / ☐ Data team |

---

## Step 2: score each framework

Rate each criterion from 1 to 5 for each framework, then multiply by the weight column that matches your team type.

### Scoring key

| Score | Meaning |
|---|---|
| 5 | Excellent. Best in class for this criterion |
| 4 | Good. Strong coverage with minor gaps |
| 3 | Adequate. Meets minimum needs |
| 2 | Weak. Needs significant workarounds |
| 1 | Poor. Does not address this criterion |

---

### Criteria weights by team type

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

### Scoring sheet

Replace `[Framework A]` and `[Framework B]` with the frameworks you are scoring. Add columns as needed.

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

## Step 3: qualitative assessment

Scores miss things. Answer these for each framework before deciding.

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

## Step 4: pilot

Before the final decision, run a one-week pilot with the top one or two candidates.

**Pilot protocol:**

1. Pick a representative eval task, something you will actually run in production rather than a toy example
2. Set a time budget of at most 4 hours per framework to get a working eval running
3. Run the eval and write down exactly what you ran, what came out, and what it cost
4. Score the output. Does the result tell you something you can act on?
5. Estimate ongoing cost by extrapolating from the pilot to your expected eval frequency and volume

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

## Step 5: decision

Fill in after scoring and pilot.

**Selected framework(s):** _______

**Rationale (two or three sentences):** _______

**What we are not using, and why:** _______

**Known gaps we are accepting:** _______

**Review trigger:** Re-evaluate if any of the following occur:
- [ ] Framework releases a major version with breaking changes
- [ ] Our use case expands significantly (e.g., from RAG to agentic)
- [ ] EU AI Act enforcement creates new audit requirements we can't satisfy
- [ ] Cost at scale exceeds $____/month
- [ ] A new framework emerges with 5k+ stars that addresses our gaps

---

## Reference: framework quick-score guide

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

The [full matrix](../comparisons/full-matrix.md) has the complete 6 by 10 comparison.

---

*This template is framework-agnostic on purpose. It works for any eval framework, not only the six covered in this guide.*
