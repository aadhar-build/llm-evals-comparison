---
title: Framework → Article Mapping
parent: EU AI Act
nav_order: 1
---

# Framework → Article Mapping

> Which framework helps satisfy which EU AI Act obligation. Read across rows (by article) or down columns (by framework).

---

## The Master Mapping

| EU AI Act Requirement | RAGAS | DeepEval | promptfoo | Langfuse | inspect_ai | OpenAI Evals |
|---|---|---|---|---|---|---|
| **Art. 9** — Risk management system | ○ | ◑ | ● | ◑ | ● | ○ |
| **Art. 10** — Data governance | ◑ | ◑ | ○ | ● | ○ | ○ |
| **Art. 13** — Transparency | ○ | ◑ | ◑ | ● | ● | ◑ |
| **Art. 15** — Accuracy, robustness, cybersecurity | ◑ | ◑ | ● | ○ | ● | ○ |
| **Art. 17** — Quality management system | ○ | ● | ◑ | ● | ◑ | ○ |
| **Annex IV** — Technical documentation | ○ | ◑ | ◑ | ● | ● | ◑ |
| **Art. 40** — Harmonised standards alignment | ○ | ○ | ○ | ○ | ● | ○ |
| **Audit trail quality** | Low | Medium | Medium | High | High | Low |
| **EU data residency option** | Self-host | Self-host | Self-host | EU Cloud ✓ | Self-host | OpenAI infra |

**●** = directly addresses the requirement  
**◑** = partially addresses / supports with additional work  
**○** = does not address this requirement

---

## Article-by-Article Breakdown

### Article 9 — Risk Management System

**The requirement:** A documented, continuous, iterative risk management process throughout the AI system lifecycle. Must include systematic testing to identify foreseeable risks, evaluate risks from intended and reasonably foreseeable misuse, and adopt risk mitigation measures.

**The eval implication:** "Systematic testing" must be documented and structured. Manual spot-checks do not satisfy Article 9.

| Framework | What it contributes | Limitations |
|---|---|---|
| **inspect_ai** ● | AgentHarm (110 adversarial behaviors), HarmBench, CyberSecEval — peer-reviewed risk assessment benchmarks. `.eval` logs are audit-grade evidence of systematic testing. Lifecycle support: run pre-deployment, at model updates, and periodically. | Requires Python. High setup time. |
| **promptfoo** ● | OWASP LLM Top 10 coverage, automated adversarial case generation across 12+ harm categories. Structured JSON/HTML report as risk documentation. The YAML config is a machine-readable record of what was tested. | Report is developer-grade, not audit-grade. No published benchmarks with institutional authority. |
| **Langfuse** ◑ | Production monitoring enables continuous risk assessment post-deployment. Score thresholds can trigger alerts when quality drops — the "monitoring" component of Art. 9's lifecycle requirement. | Observes production behaviour only; does not run risk tests proactively. |
| **DeepEval** ◑ | Bias and toxicity metrics address specific risk categories. CI/CD integration means risk metrics are checked on every model or prompt update. | Not a dedicated risk testing framework; safety metrics are a subset of its metric library. |
| **RAGAS** ○ | No adversarial or safety coverage. | — |
| **OpenAI Evals** ○ | No structured risk testing capability. | — |

**Practical guidance:** For Article 9, run **inspect_ai AgentHarm** as your primary documented risk assessment before deployment. Use **promptfoo** for ongoing adversarial regression in CI. Include inspect_ai `.eval` logs in your risk management file.

---

### Article 10 — Data and Data Governance

**The requirement:** Training, validation, and testing data must be subject to appropriate data governance. Testing data must be relevant to the intended geographic, contextual, and behavioural scope. Bias and error detection required. Dataset characteristics must be documented.

**The eval implication:** Your evaluation dataset is a compliance artifact. It must be representative, documented, and versioned.

| Framework | What it contributes | Limitations |
|---|---|---|
| **Langfuse** ● | Production traces provide an auditable record of real-world inputs — the actual data distribution your system encounters. This is the most direct evidence that your evaluation reflects real deployment conditions. Data retention is configurable; GDPR-compliant EU Cloud or self-hosted. | Captures production data, not pre-deployment validation data. |
| **RAGAS** ◑ | `TestsetGenerator` creates synthetic evaluation data from your document corpus. Documenting the generation parameters (LLM used, diversity settings, test types) creates a partial dataset lineage record. | Synthetic data may not satisfy the "representative of deployment scope" requirement without validation against real usage. |
| **DeepEval** ◑ | Dataset import (JSONL) with metadata — evaluation datasets can be versioned and documented. Confident AI's dataset management features (optional cloud) add lineage tracking. | No automatic diversity or representativeness checking. |
| **promptfoo** ○ | No data governance features. | — |
| **inspect_ai** ○ | No dataset governance features — focuses on eval execution, not dataset management. | — |
| **OpenAI Evals** ○ | Registry model — datasets are public JSONL. No data governance features. | — |

**Practical guidance:** For Article 10, combine **Langfuse production traces** (real-world data distribution evidence) with documented **RAGAS evaluation datasets** (pre-deployment testing data). Keep versioned JSONL files alongside your model releases.

---

### Article 13 — Transparency and Provision of Information

**The requirement:** High-risk AI systems must be transparent to deployers. Technical documentation (Annex IV) and instructions for use must be provided. Where relevant, systems must indicate when they're producing AI-generated content.

**The eval implication:** Eval results contribute to the transparency obligation. Deployers have the right to know what the system was tested against and what limitations were found.

| Framework | What it contributes | Limitations |
|---|---|---|
| **Langfuse** ● | Full inference audit trail: every input, output, model version, latency, cost. Trace exports (JSONL, CSV) are machine-readable evidence of system behaviour. Deployers can request a trace export to understand what the system did in a given period. | Captures production behaviour; doesn't document pre-deployment testing methodology. |
| **inspect_ai** ● | `.eval` log files contain: full task configuration, all sample inputs and outputs, all scores, model attribution, timestamps. These are structured records that satisfy the "what did you test?" transparency requirement. | Batch evaluation only; no production trace capability. |
| **promptfoo** ◑ | YAML configuration is a human-readable, auditable record of what was tested and under what conditions. HTML/JSON report shows results. | Report format is developer-grade; not structured to regulatory standard. |
| **DeepEval** ◑ | pytest output + optional Confident AI dashboard provide test results as structured artifacts. Metric definitions and thresholds are version-controlled in code. | No equivalent to Langfuse's inference-level audit trail. |
| **OpenAI Evals** ◑ | Published evals in the registry are citable public artifacts — "we ran this publicly available eval." | Hosted on OpenAI infrastructure; data residency concerns for some use cases. |
| **RAGAS** ○ | Metric output is Python data; no structured documentation artifact. | — |

**Practical guidance:** For Article 13, include **inspect_ai `.eval` logs** in your Annex IV technical documentation, and use **Langfuse** for ongoing inference transparency. If deployers ask "what does this system do?", Langfuse traces let you show them.

---

### Article 15 — Accuracy, Robustness, and Cybersecurity

**The requirement:** High-risk AI systems must achieve appropriate accuracy for their intended purpose; be resilient against errors, faults, inconsistencies; and be secure against adversarial attacks that could alter their output or performance.

**The eval implication:** The "resilient against adversarial attacks" clause is a direct requirement to run adversarial testing.

| Framework | What it contributes | Limitations |
|---|---|---|
| **inspect_ai** ● | AgentHarm (adversarial), CyberSecEval (cybersecurity knowledge and risks), HarmBench (robustness). These are peer-reviewed published benchmarks — the most credible evidence of systematic robustness testing. | No LLM-specific accuracy metrics (faithfulness, relevancy). |
| **promptfoo** ● | OWASP LLM Top 10 coverage, automated attack generation — direct adversarial robustness testing across prompt injection, jailbreaks, PII extraction, indirect attacks. | Developer-grade report; no institutional benchmark authority. |
| **RAGAS** ◑ | Faithfulness and answer relevancy measure accuracy against ground truth. No robustness or adversarial coverage. | Accuracy metrics only; no adversarial testing. |
| **DeepEval** ◑ | Hallucination metric addresses accuracy. Robustness testing is limited. | No adversarial security testing. |
| **Langfuse** ○ | Observes accuracy degradation in production through score monitoring. Doesn't test for adversarial vulnerabilities. | — |
| **OpenAI Evals** ○ | Match-based evals measure accuracy on specific domains; no adversarial coverage. | — |

**Practical guidance:** For Article 15, run **promptfoo** for broad adversarial coverage and **inspect_ai CyberSecEval** for cybersecurity-specific evidence. Use **RAGAS/DeepEval** for accuracy baselines. The combination addresses all three sub-requirements (accuracy, robustness, security).

---

### Article 17 — Quality Management System

**The requirement:** Providers must implement a documented QMS covering: design and development strategy, data management, risk management, post-market monitoring, corrective action procedures, serious incident logging.

**The eval implication:** The QMS must document eval procedures — what metrics, what thresholds, what happens when thresholds are breached.

| Framework | What it contributes | Limitations |
|---|---|---|
| **DeepEval** ● | pytest integration makes eval thresholds explicit in version-controlled code. CI/CD gate means threshold breaches block deployment — the documented corrective action. Test results are CI artifacts retained with the build. | CI artifacts may not be retained long enough for multi-year QMS documentation. |
| **Langfuse** ● | Production scoring with configurable alerts implements the "post-market monitoring" component. Score trends over time are the QMS's continuous monitoring record. Trace retention is configurable for multi-year archival. | No pre-deployment quality gate capability. |
| **promptfoo** ◑ | Adversarial regression in CI implements part of the QMS testing procedure. YAML config is the documented test specification. | Not a full QMS tool; covers adversarial testing only. |
| **inspect_ai** ◑ | Periodic eval runs (pre-deployment, at model updates) contribute to the QMS's risk management cycle. `.eval` logs are the documented evidence. | No CI/CD integration; manual gate only. |
| **RAGAS** ○ | No QMS integration features. | — |
| **OpenAI Evals** ○ | No QMS integration features. | — |

**Practical guidance:** For Article 17, build your QMS eval layer around **DeepEval in CI** (pre-deployment quality gate with documented thresholds) and **Langfuse** (post-deployment monitoring with alert thresholds). Document your metric definitions and thresholds in version-controlled configuration files — these are your QMS procedures.

---

### Annex IV — Technical Documentation

**The requirement:** Annex IV specifies exactly what technical documentation must contain, including: general description and intended purpose, detailed description of elements and development process, information on monitoring and functioning, instructions for use, information on risks and mitigation measures, test results.

**The eval implication:** "Test results" in Annex IV means structured evidence of what was tested, when, with what methodology, and what the findings were.

| Framework | What it contributes | Format suitability |
|---|---|---|
| **inspect_ai** ● | `.eval` log files — self-contained JSON archives with configuration + all inputs/outputs/scores. Designed for archival and reproducibility. Directly citable in Annex IV documentation. | ● ● ● ● ● |
| **Langfuse** ● | Trace exports (JSONL/CSV) provide machine-readable evidence of system behaviour over the operational period. | ● ● ● ● ○ |
| **DeepEval** ◑ | pytest JUnit output + optional Confident AI dashboard. Less structured than inspect_ai logs but still machine-readable. | ● ● ● ○ ○ |
| **promptfoo** ◑ | JSON report of test results — structured but developer-facing. | ● ● ● ○ ○ |
| **OpenAI Evals** ◑ | Published evals in the registry are citable — running a published eval provides a reference to a public methodology. | ● ● ○ ○ ○ |
| **RAGAS** ○ | Python dict outputs — not structured for documentation archival. | ● ○ ○ ○ ○ |

**Practical guidance:** For Annex IV, use **inspect_ai `.eval` files** as the primary test evidence artifact. Include: eval task file (commit hash), model version, date of run, summary metrics, `.eval` log archive. Store alongside model release artifacts, not in a separate compliance folder — they decay when separated.

---

### Article 40 — Harmonised Standards

**The requirement:** AI systems built in accordance with harmonised standards adopted under the Act shall be presumed to conform to the Act requirements covered by those standards. The Commission publishes references to harmonised standards in the Official Journal.

**The eval implication:** As international AI safety standards are developed (ISO/IEC 42001, emerging AISI standards), alignment with those standards will create a "presumption of conformity" — the strongest compliance signal available.

| Framework | What it contributes |
|---|---|
| **inspect_ai** ● | The UK AI Safety Institute — which maintains inspect_ai — is directly involved in developing the international AI safety standards that will become harmonised standards under Article 40. Using inspect_ai now aligns your evaluation methodology with the standards as they're being written. |
| All others | No direct involvement in standards development. |

**Practical guidance:** This is a forward-looking benefit, not a current certification. But the risk of using a non-aligned methodology is real: if harmonised standards require specific eval procedures that you've never implemented, retrofitting is expensive. Starting with inspect_ai now is insurance against that risk.

---

## Red-Flag Matrix: What Each Framework Cannot Provide

Understanding gaps is as important as understanding coverage. For high-risk AI systems, these gaps are compliance risks:

| Framework | Critical gap for high-risk AI |
|---|---|
| **RAGAS** | No audit trail. No adversarial testing. Eval results not structured for regulatory documentation. |
| **DeepEval** | No production audit trail. No adversarial testing. Telemetry (PostHog) must be disabled for regulated deployments. |
| **promptfoo** | No audit-grade output format. No institutional authority. OpenAI ownership is a procurement concern for EU sovereignty requirements. |
| **Langfuse** | No pre-deployment adversarial testing. No published safety benchmarks. EU Cloud available but third-party data processor (DPA required). |
| **inspect_ai** | No production monitoring. No CI/CD integration. Steep setup. No RAG-specific metrics. |
| **OpenAI Evals** | Data processed by OpenAI. No audit trail. No adversarial testing. Not designed for application-specific compliance evidence. |

**No single framework is sufficient for Annex III high-risk AI compliance.** The minimum viable compliance stack requires at least inspect_ai (safety evidence) + Langfuse (production audit trail).

---

→ [Pre-deployment checklist](./high-risk-checklist.md) · [Back to EU AI Act](./README.md) · [Full matrix](../comparisons/full-matrix.md)
