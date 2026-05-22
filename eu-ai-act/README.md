---
title: EU AI Act
nav_order: 5
has_children: true
permalink: /eu-ai-act/
---

# EU AI Act — Eval Framework Implications

> Eval framework choice is a compliance decision for high-risk AI systems, not just a technical one. This module makes the mapping explicit.

---

## Why This Module Exists

No other LLM eval comparison guide covers the EU AI Act. That gap matters because:

1. **The enforcement deadline is August 2026.** High-risk AI system obligations under the Act become enforceable for new systems from August 2026. Systems already deployed have until August 2027.

2. **Which framework you use determines which articles you can demonstrate compliance with.** An auditor asking "how did you evaluate your AI system's safety?" receives a materially different answer depending on whether you used inspect_ai (audit-grade logs, government-backed framework) or a homegrown script (no structured evidence).

3. **Most teams don't know which articles create eval obligations.** Articles 9, 10, 13, 15, and 17 all have implications for how you run evaluations — but they're not labelled "eval requirements." This module translates Act obligations into concrete framework choices.

---

## Enforcement Timeline

| Date | What becomes enforceable |
|---|---|
| **February 2025** | Prohibited AI practices (Article 5) — e.g., social scoring, subliminal manipulation |
| **August 2025** | General-Purpose AI rules (Articles 51–56) — transparency, copyright, systemic-risk models |
| **August 2026** | High-risk AI system obligations (Chapters III, V) — **the main deadline** |
| **August 2027** | High-risk systems already deployed before August 2026 |

**The August 2026 deadline is the one that matters for product teams** building AI systems in the Annex III high-risk categories. If your system is high-risk and you haven't built your eval infrastructure by then, you're non-compliant on day one of enforcement.

---

## Which Systems Are "High-Risk"? (Annex III)

The Act defines eight categories of high-risk AI systems. If your product falls into any of these, the full Chapter III obligations apply:

| Category | Examples |
|---|---|
| **Biometric identification** | Real-time facial recognition, emotion recognition systems |
| **Critical infrastructure** | AI managing energy grids, water systems, transport |
| **Education and vocational training** | Systems determining access to education, scoring assessments |
| **Employment** | CV screening, interview scoring, promotion/dismissal decisions |
| **Essential services** | Credit scoring, insurance risk assessment, benefits eligibility |
| **Law enforcement** | Predictive policing, evidence reliability assessment |
| **Migration and border control** | Asylum assessment, border crossing risk scoring |
| **Justice and democratic processes** | AI assisting judicial decisions, election-related AI |

**Practical note:** Many B2B SaaS products touch these categories indirectly. An "AI-powered HR tool" is in scope if it influences hiring decisions. A "loan origination assistant" is in scope even if a human makes the final call. When in doubt, assume Annex III applies and build accordingly — the downside of over-compliance is minimal; the downside of under-compliance is significant.

---

## The Four Articles That Create Eval Obligations

### Article 9 — Risk Management System

**What it requires:** A documented, iterative risk management system throughout the AI system's lifecycle. Must include: identification of foreseeable risks, evaluation of risks that materialise if system performs as intended, identification of risks from misuse.

**What this means for evals:** You must run structured adversarial tests (what risks exist?) and document the results. Ad-hoc manual testing is not sufficient evidence. A systematic red teaming run with structured output — promptfoo's OWASP LLM Top 10 coverage, or inspect_ai's AgentHarm benchmark — is what "systematic testing" looks like.

**Framework implication:** inspect_ai (audit-grade logs) or promptfoo (structured adversarial report) for pre-deployment; Langfuse for ongoing risk monitoring in production.

---

### Article 10 — Data and Data Governance

**What it requires:** Training, validation, and testing datasets must meet quality criteria. Testing must use data relevant to the intended geographic, contextual, and demographic scope. Bias testing required where relevant.

**What this means for evals:** Your evaluation dataset itself is a compliance artifact. It must be documented, versioned, and representative of the deployment context. Synthetic datasets (RAGAS TestsetGenerator) that aren't validated against real deployment data may not satisfy this requirement on their own.

**Framework implication:** Dataset provenance documentation in RAGAS or DeepEval eval runs; Langfuse traces as evidence of real-world data distribution.

---

### Article 13 — Transparency and Provision of Information

**What it requires:** High-risk AI systems must be sufficiently transparent that deployers can understand the system's capabilities and limitations and use it appropriately. Providers must supply technical documentation (Annex IV) and instructions for use.

**What this means for evals:** Your eval results are part of the transparency obligation. If a deployer asks "how was this system tested?", the answer must be documented and accessible. inspect_ai's `.eval` log format is purpose-built for this: it's machine-readable, timestamped, model-attributed, and archivable.

**Framework implication:** inspect_ai `.eval` logs in Annex IV documentation. Langfuse trace exports as evidence of production behaviour.

---

### Article 15 — Accuracy, Robustness, and Cybersecurity

**What it requires:** High-risk AI systems must achieve appropriate levels of accuracy, be robust against errors and inconsistencies, and be resilient against attempts to alter their use or performance through adversarial attacks.

**What this means for evals:** The "resilient against adversarial attacks" clause is a direct eval requirement. You must test adversarial robustness — not just output quality. inspect_ai's CyberSecEval and AgentHarm benchmarks, and promptfoo's adversarial attack coverage, directly address this.

**Framework implication:** promptfoo for attack surface coverage; inspect_ai for benchmarked robustness evidence.

---

### Article 17 — Quality Management System

**What it requires:** Providers must put a quality management system in place covering the entire lifecycle: strategy, procedures, data management, risk management, post-market monitoring, serious incident logging.

**What this means for evals:** The QMS must include documented eval procedures — what metrics, what thresholds, what happens when a threshold is breached. CI/CD-integrated eval (DeepEval in pytest) is the closest thing to a "documented eval procedure" that product teams have. Langfuse's production scoring is the "post-market monitoring" component.

**Framework implication:** DeepEval in CI as the QMS quality gate; Langfuse for post-market monitoring; inspect_ai for lifecycle safety evidence.

---

## The Non-Negotiable for High-Risk AI: Audit Trail

The single biggest implication of EU AI Act compliance for eval framework choice: **you need an audit trail**.

An audit trail means:
- Every eval run is logged with: date, model version, eval version, sample inputs, outputs, scores
- Logs are retained and retrievable
- The methodology can be reconstructed months later

| Framework | Audit trail quality | Notes |
|---|---|---|
| inspect_ai | ● ● ● ● ● | `.eval` format — timestamped, model-attributed, full I/O, archivable |
| Langfuse | ● ● ● ● ● | Production traces — complete inference records, configurable retention |
| DeepEval | ● ● ● ○ ○ | pytest output + Confident AI dashboard (optional cloud) |
| promptfoo | ● ● ● ○ ○ | HTML/JSON report — readable but not structured to regulatory standard |
| RAGAS | ● ● ○ ○ ○ | Metric outputs in Python dicts — you build the logging yourself |
| OpenAI Evals | ● ● ○ ○ ○ | Run logs in OpenAI-controlled infrastructure |

**For Annex III high-risk AI systems: inspect_ai + Langfuse are the minimum audit-grade stack.** Other frameworks contribute to compliance but don't produce self-contained audit evidence on their own.

---

## In This Module

- **[Framework Mapping](./framework-mapping.md)** — Which framework addresses which Act article, with practical guidance
- **[High-Risk Checklist](./high-risk-checklist.md)** — Pre-deployment eval checklist for Annex III systems; recommended stacks by risk tier

---

*This module reflects the Act as published in the Official Journal of the EU (OJ L 2024/1689). It is guidance, not legal advice. For formal conformity assessment, consult a notified body.*
