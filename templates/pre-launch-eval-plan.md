---
title: Pre-Launch Eval Plan
parent: Templates
nav_order: 2
---

# Pre-Launch Eval Plan Template

> Write this before your team starts instrumenting anything. The plan defines what "good enough" means before you build the scaffolding to measure it.

---

## How to Use This Template

1. **Fill in Sections 1–4 before writing any eval code.** These sections define what you're measuring and why — decisions that are hard to reverse once infrastructure is built.
2. **Fill in Sections 5–6 after your first eval run.** Baselines require an actual run; thresholds are set relative to baselines.
3. **Get sign-off before shipping.** Section 7 is the gate. If the plan isn't signed off, the feature isn't launched.
4. **Archive this document alongside your model release artifacts.** For EU AI Act Annex IV, this document is part of your technical documentation.

---

## Document Header

| Field | Value |
|---|---|
| **Feature name** | |
| **Product / system** | |
| **Author** | |
| **Date** | |
| **Version** | v1.0 |
| **EU AI Act scope** | ☐ Annex III high-risk / ☐ Out of scope / ☐ Uncertain — flag for legal review |
| **Status** | ☐ Draft / ☐ In review / ☐ Approved / ☐ Active / ☐ Superseded |

---

## Section 1 — Feature Overview

**What is this feature?**
*2–3 sentences. What does it do, who uses it, what decisions does it affect?*

> Example: An AI assistant that processes employee CVs and surfaces a shortlist of candidates for hiring managers. The assistant uses RAG over the company's job description corpus to generate relevancy scores and a short narrative for each applicant. Hiring managers make the final decision; the assistant does not decide.

___

**Intended users:**

___

**Deployment context:**
*(geography, languages, user types, integration points)*

___

**Is a human in the loop?**
☐ Yes — human makes all final decisions  
☐ Partial — human reviews high-impact decisions only  
☐ No — system acts autonomously

If Partial or No: **what is the maximum consequence of a wrong output?**

___

---

## Section 2 — What We're Measuring

List every quality dimension you care about. For each, answer: what does "good" look like for this feature?

| Quality dimension | Why it matters for this feature | How we'll measure it |
|---|---|---|
| Faithfulness / grounding | | |
| Answer relevancy | | |
| Retrieval precision | | |
| Retrieval recall | | |
| Hallucination rate | | |
| Bias / fairness | | |
| Adversarial robustness | | |
| Latency (P50 / P99) | | |
| Cost per query | | |
| [Domain-specific metric] | | |

**Metrics we explicitly decided NOT to measure and why:**

| Metric skipped | Reason |
|---|---|
| | |

*Documenting skipped metrics is as important as documenting chosen ones. An auditor will ask.*

---

## Section 3 — Evaluation Dataset

**Dataset source:**
☐ Synthetic (generated from corpus) — tool used: _______  
☐ Real user queries (historical) — collection period: _______  
☐ Expert-authored — authored by: _______  
☐ Hybrid — describe: _______

**Dataset size:** _____ samples

**Dataset coverage:**

| Dimension | How it's represented |
|---|---|
| Geographic scope (if relevant) | |
| Language(s) | |
| User type / persona | |
| Edge cases | |
| Demographic diversity (if relevant) | |

**Dataset version:** v____  
**Storage location:** _______  
**Access control:** _______

**Known gaps in this dataset:**

___

**Plan to expand or update dataset:**
*(e.g., "add real user queries after 2 weeks in production")*

___

---

## Section 4 — Framework Selection

**Primary eval framework(s):**

| Framework | Role | Why chosen over alternatives |
|---|---|---|
| | | |
| | | |

**Frameworks considered and rejected:**

| Framework | Reason not chosen |
|---|---|
| | |

**Eval run location:**
☐ Local (developer machine)  
☐ CI/CD pipeline (framework: _______)  
☐ Dedicated eval infrastructure  
☐ Third-party cloud (provider: _______, data residency: _______)

**LLM judge model:**
Model: _______  
Provider: _______  
Data sent to provider: ☐ Inputs only / ☐ Inputs + outputs / ☐ Inputs + outputs + context

*If EU data residency required: confirm judge model provider is EU-compliant or use a self-hosted judge.*

---

## Section 5 — Baseline

*Complete after first eval run. Do not set thresholds before you have a baseline.*

**Baseline run date:** _______  
**Model version evaluated:** _______  
**Eval framework version:** _______

| Metric | Baseline score | Notes |
|---|---|---|
| | | |
| | | |
| | | |

**Baseline interpretation:**
*Is this baseline acceptable? What does it reveal about the current model state?*

___

**Comparison to previous version (if applicable):**

| Metric | Previous | Baseline (current) | Delta |
|---|---|---|---|
| | | | |

---

## Section 6 — Pass/Fail Thresholds

Set thresholds based on baseline + acceptable tolerance. Every threshold needs an owner and a defined consequence.

| Metric | Threshold (min/max) | Rationale | Breach consequence | Owner |
|---|---|---|---|---|
| Faithfulness | ≥ ___ | | ☐ Block deploy / ☐ Alert / ☐ Log | |
| Answer relevancy | ≥ ___ | | ☐ Block deploy / ☐ Alert / ☐ Log | |
| Hallucination rate | ≤ ___ | | ☐ Block deploy / ☐ Alert / ☐ Log | |
| Adversarial pass rate | ≥ ___ | | ☐ Block deploy / ☐ Alert / ☐ Log | |
| Latency P99 | ≤ ___ms | | ☐ Block deploy / ☐ Alert / ☐ Log | |
| [Domain metric] | | | | |

**Threshold review process:**
*Who has authority to change a threshold, and what approval is required?*

___

**Threshold documentation location:**
*(thresholds should live in version-controlled code, not just this doc)*

```python
# Example: thresholds.py — version controlled alongside eval code
THRESHOLDS = {
    "faithfulness": 0.85,
    "answer_relevancy": 0.75,
    "hallucination_rate_max": 0.10,
    "adversarial_pass_rate": 0.90,
}
```

---

## Section 7 — Production Monitoring Plan

*What happens after launch?*

**Monitoring framework:** _______  
**Deployment:** ☐ Cloud (provider: _____, region: _____) / ☐ Self-hosted

**Sampling rate for production scoring:**
*Scoring every inference is expensive. What % of production traffic will be scored?*  
☐ 100% (feasible only at low volume)  
☐ ___% random sample  
☐ 100% of [specific condition, e.g. "user thumbs-down signals"]

**Metrics monitored in production:**

| Metric | Sampling strategy | Alert threshold | Alert recipient |
|---|---|---|---|
| | | | |

**Data retention period:** _______  
*(For EU AI Act Annex III: recommend minimum 3 years)*

**Incident definition:**
*At what failure rate or severity does an event become a reportable incident?*

___

**Re-evaluation trigger:**
*When will you run a full eval suite again (not just production monitoring)?*

☐ On every model/prompt update  
☐ Monthly  
☐ Quarterly  
☐ When production metric drops below _______  
☐ Other: _______

---

## Section 8 — Sign-Off

All parties must sign off before the feature ships to production.

| Role | Name | Decision | Date | Notes |
|---|---|---|---|---|
| **Product Manager** | | ☐ Approved / ☐ Rejected / ☐ Conditional | | |
| **Tech Lead / Engineering** | | ☐ Approved / ☐ Rejected / ☐ Conditional | | |
| **Data / ML Lead** (if applicable) | | ☐ Approved / ☐ Rejected / ☐ Conditional | | |
| **Compliance / Legal** (if EU AI Act scope) | | ☐ Approved / ☐ Rejected / ☐ Conditional | | |
| **Security** (if adversarial testing required) | | ☐ Approved / ☐ Rejected / ☐ Conditional | | |

**Conditions for conditional approvals:**

___

**Ship decision:** ☐ Ship / ☐ Hold — reason: _______

---

## Appendix A — Eval Run Log

Track every eval run against this plan. Update after each run.

| Run date | Model version | Framework version | Key metric scores | Outcome | Notes |
|---|---|---|---|---|---|
| | | | | ☐ Pass / ☐ Fail | |
| | | | | ☐ Pass / ☐ Fail | |

---

## Appendix B — Checklist

Quick reference before sign-off meeting.

**Section 1–2 (definition):**
- [ ] Feature described in plain language a non-ML stakeholder can understand
- [ ] Every measured metric has a stated reason for inclusion
- [ ] Skipped metrics are documented with rationale

**Section 3 (dataset):**
- [ ] Dataset is version-controlled and retrievable in 12 months
- [ ] Dataset coverage documented — no obvious demographic or scenario gaps
- [ ] If EU AI Act scope: dataset representativeness reviewed against deployment geography

**Section 4 (framework):**
- [ ] Framework selection is documented with alternatives considered
- [ ] Data sent to LLM judge is documented — no surprise PII in context
- [ ] If EU data residency required: judge model provider confirmed compliant

**Section 5–6 (baseline + thresholds):**
- [ ] Baseline established from an actual eval run, not estimated
- [ ] Every threshold has a breach consequence defined
- [ ] Thresholds are in version-controlled code, not only in this doc

**Section 7 (monitoring):**
- [ ] Production monitoring is instrumented and tested before launch
- [ ] Alert recipients are confirmed and have acknowledged their role
- [ ] Data retention period is set and meets any applicable regulatory requirement

**Section 8 (sign-off):**
- [ ] All required sign-offs obtained
- [ ] Conditional approvals have conditions documented and tracked

---

*For EU AI Act Annex III high-risk AI systems: this document, along with eval run logs and `.eval` archive files, constitutes the "test results" section of your Annex IV technical documentation. Store alongside model artifacts.*

→ [EU AI Act high-risk checklist](../eu-ai-act/high-risk-checklist.md) · [Eval Framework RFP](./eval-framework-rfp.md) · [Decision Guide](../decision-guide/)
