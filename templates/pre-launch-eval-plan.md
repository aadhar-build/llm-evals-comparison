---
title: Pre-Launch Eval Plan
parent: Templates
nav_order: 2
---

# Pre-launch eval plan template

> Write this before your team starts instrumenting anything. The plan defines what "good enough" means before you build the scaffolding to measure it.

---

## How to use this template

1. Fill in sections 1 to 4 before writing any eval code. They define what you are measuring and why, and those decisions are hard to reverse once the infrastructure exists.
2. Fill in sections 5 and 6 after your first eval run. A baseline needs a real run, and thresholds are set relative to the baseline.
3. Get sign-off before shipping. Section 8 is the gate. If the plan is not signed off, the feature does not launch.
4. Archive this document with your model release artifacts. Under EU AI Act Annex IV it is part of your technical documentation.

---

## Document header

| Field | Value |
|---|---|
| **Feature name** | |
| **Product / system** | |
| **Author** | |
| **Date** | |
| **Version** | v1.0 |
| **EU AI Act scope** | ☐ Annex III high-risk / ☐ Out of scope / ☐ Uncertain, flag for legal review |
| **Status** | ☐ Draft / ☐ In review / ☐ Approved / ☐ Active / ☐ Superseded |

---

## Section 1: feature overview

**What is this feature?**
*Two or three sentences. What does it do, who uses it, and what decisions does it affect?*

> Example: An AI assistant that processes employee CVs and surfaces a shortlist of candidates for hiring managers. The assistant uses RAG over the company's job description corpus to generate relevancy scores and a short narrative for each applicant. Hiring managers make the final decision; the assistant does not decide.

___

**Intended users:**

___

**Deployment context:**
*(geography, languages, user types, integration points)*

___

**Is a human in the loop?**
☐ Yes, a human makes all final decisions  
☐ Partial, a human reviews high-impact decisions only  
☐ No, the system acts on its own

If Partial or No: **what is the maximum consequence of a wrong output?**

___

---

## Section 2: what we are measuring

List every quality dimension you care about. For each one, say what "good" looks like for this feature.

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

**Metrics we decided not to measure, and why:**

| Metric skipped | Reason |
|---|---|
| | |

*Documenting skipped metrics is as important as documenting chosen ones. An auditor will ask.*

---

## Section 3: evaluation dataset

**Dataset source:**
☐ Synthetic (generated from corpus), tool used: _______  
☐ Real user queries (historical), collection period: _______  
☐ Expert-authored, authored by: _______  
☐ Hybrid, describe: _______

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

## Section 4: framework selection

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

*If EU data residency is required, confirm the judge model provider is EU-compliant or use a self-hosted judge.*

---

## Section 5: baseline

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

## Section 6: pass/fail thresholds

Set thresholds from the baseline plus an acceptable tolerance. Every threshold needs an owner and a defined consequence.

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
# Example: thresholds.py, version controlled alongside eval code
THRESHOLDS = {
    "faithfulness": 0.85,
    "answer_relevancy": 0.75,
    "hallucination_rate_max": 0.10,
    "adversarial_pass_rate": 0.90,
}
```

---

## Section 7: production monitoring plan

*What happens after launch?*

**Monitoring framework:** _______  
**Deployment:** ☐ Cloud (provider: _____, region: _____) / ☐ Self-hosted

**Sampling rate for production scoring:**
*Scoring every inference is expensive. What share of production traffic will be scored?*  
☐ 100% (feasible only at low volume)  
☐ ___% random sample  
☐ 100% of [specific condition, e.g. "user thumbs-down signals"]

**Metrics monitored in production:**

| Metric | Sampling strategy | Alert threshold | Alert recipient |
|---|---|---|---|
| | | | |

**Data retention period:** _______  
*(For EU AI Act Annex III, a minimum of 3 years is recommended)*

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

## Section 8: sign-off

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

**Ship decision:** ☐ Ship / ☐ Hold, reason: _______

---

## Appendix A: eval run log

Track every eval run against this plan. Update after each run.

| Run date | Model version | Framework version | Key metric scores | Outcome | Notes |
|---|---|---|---|---|---|
| | | | | ☐ Pass / ☐ Fail | |
| | | | | ☐ Pass / ☐ Fail | |

---

## Appendix B: checklist

Quick reference before sign-off meeting.

**Sections 1 and 2 (definition):**
- [ ] Feature described in plain language a non-ML stakeholder can understand
- [ ] Every measured metric has a stated reason for inclusion
- [ ] Skipped metrics are documented with rationale

**Section 3 (dataset):**
- [ ] Dataset is version-controlled and retrievable in 12 months
- [ ] Dataset coverage documented, with no obvious demographic or scenario gaps
- [ ] If EU AI Act scope: dataset representativeness reviewed against deployment geography

**Section 4 (framework):**
- [ ] Framework selection is documented with alternatives considered
- [ ] Data sent to the LLM judge is documented, with no surprise PII in context
- [ ] If EU data residency required: judge model provider confirmed compliant

**Sections 5 and 6 (baseline and thresholds):**
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

*For EU AI Act Annex III high-risk systems, this document plus the eval run logs and `.eval` archive files make up the "test results" section of your Annex IV technical documentation. Store them with the model artifacts.*

Related: [EU AI Act high-risk checklist](../eu-ai-act/high-risk-checklist.md), [eval framework RFP](./eval-framework-rfp.md), [decision guide](../decision-guide/)
