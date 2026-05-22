---
title: High-Risk AI Checklist
parent: EU AI Act
nav_order: 2
---

# Pre-Deployment Eval Checklist for High-Risk AI Systems

> For AI systems in EU AI Act Annex III categories. Complete before first production deployment and before each significant model update.

---

## Before Using This Checklist

**Step 0: Confirm your system is Annex III high-risk.**

Check the eight categories: biometric identification, critical infrastructure, education, employment, essential services, law enforcement, migration/border control, justice/democratic processes.

If you're not sure: assume high-risk and proceed. The cost of over-compliance is a few extra eval runs; the cost of under-compliance is formal enforcement action.

**This checklist is for:** AI providers (companies that develop and place the system on the market).  
**Deployers** (companies that operate a third-party AI system) have separate obligations under Chapter III, Section 3 — this checklist doesn't cover those.

---

## Phase 1 — Pre-Development (Before You Build)

### 1.1 Risk identification

- [ ] **Document the intended purpose** — specific use case, intended users, deployment context, geographic scope
- [ ] **Identify foreseeable misuse scenarios** — what could go wrong if the system is used outside its intended purpose?
- [ ] **Identify foreseeable risks** — list risks categorised by: probability of occurrence, severity of harm, number of people potentially affected
- [ ] **Define risk acceptance criteria** — what metric score constitutes "acceptable" for each risk category?

*Article 9 requires this to be documented and iterative. Create a living risk register.*

---

### 1.2 Eval dataset planning

- [ ] **Define the evaluation dataset scope** — what user types, geographic contexts, languages, and use case variants must the dataset cover?
- [ ] **Document dataset construction methodology** — how samples were selected, synthetic vs. real, sources used
- [ ] **Assess representativeness** — does the dataset reflect the actual deployment demographic and behavioural context?
- [ ] **Check for bias in the evaluation set** — are any demographic groups systematically underrepresented?

*Article 10 requires testing data to be relevant to the intended deployment scope. Dataset methodology is a compliance artifact.*

---

## Phase 2 — Development Evals

### 2.1 Quality baseline (DeepEval or RAGAS)

For RAG systems:
- [ ] **Faithfulness** ≥ threshold defined in risk register (typically 0.85 for high-risk applications)
- [ ] **Context precision** ≥ threshold
- [ ] **Context recall** ≥ threshold
- [ ] **Answer relevancy** ≥ threshold

For general LLM systems:
- [ ] **Answer relevancy** ≥ threshold
- [ ] **Hallucination rate** ≤ threshold
- [ ] **Bias score** within acceptable range

*Use DeepEval's pytest integration so threshold violations block CI. Keep threshold values in version-controlled config.*

```python
# thresholds.py — version controlled alongside model artifacts
THRESHOLDS = {
    "faithfulness": 0.85,
    "context_precision": 0.80,
    "answer_relevancy": 0.75,
    "hallucination_max": 0.10,
}
```

---

### 2.2 Bias and fairness testing

- [ ] **Define demographic dimensions relevant to your use case** — e.g., gender, age, nationality, language, disability status
- [ ] **Run DeepEval BiasMetric** across demographic variants of your test cases
- [ ] **Document metric scores per demographic group** — not just aggregate scores
- [ ] **Define acceptable disparity threshold** — e.g., no demographic group may score more than 15% below the overall average
- [ ] **If bias found:** document the finding, the root cause analysis, and the mitigation taken

*Article 10 and Article 15 both touch on bias. Aggregate metrics hide demographic disparities — always disaggregate.*

---

### 2.3 Adversarial robustness (promptfoo)

- [ ] **Define the adversarial threat model** — who is the adversary? What are they trying to do?
- [ ] **Run promptfoo OWASP LLM Top 10 suite**
- [ ] **Run promptfoo with harm categories relevant to your domain:**
  - Employment decisions → `harmful:discrimination`, `pii:direct`
  - Financial services → `harmful:financial-crime`, `pii:indirect`
  - Healthcare → `harmful:medical-advice-without-disclaimer`, `pii:medical`
  - Government → `harmful:politics`, `prompt-injection`
- [ ] **Prompt injection test** — can user input override system instructions?
- [ ] **Indirect injection test** — if system uses RAG, can adversarial content in retrieved documents affect behaviour?
- [ ] **Document all findings** — vulnerabilities found, severity, remediation taken, re-test result

```yaml
# promptfooconfig.yaml — version controlled
redteam:
  purpose: "HR screening assistant for employment decisions"
  plugins:
    - owasp:llm
    - harmful:discrimination
    - pii:direct
    - harmful:hate
  strategies:
    - jailbreak
    - prompt-injection
```

*Article 9 (risk management) and Article 15 (robustness against adversarial attacks). The YAML file is your documented test specification.*

---

## Phase 3 — Pre-Deployment Safety Evaluation

### 3.1 Structured safety evaluation (inspect_ai)

This is the audit-grade step. The output of this phase goes into your Annex IV technical documentation.

- [ ] **Pin inspect_ai to a specific commit:**
  ```
  git+https://github.com/UKGovernmentBEIS/inspect_ai.git@<commit-hash>
  ```
- [ ] **Run AgentHarm benchmark** (from inspect_evals):
  ```bash
  inspect eval inspect_evals/src/inspect_evals/agentharm/agentharm.py \
    --model <your-model> \
    --log-dir ./eval-logs/pre-deploy/$(date +%Y%m%d)/
  ```
- [ ] **Run domain-relevant benchmarks** from inspect_evals:
  - CyberSecEval for any system with internet access or code execution
  - HarmBench for general safety baseline
- [ ] **Write custom inspect_ai Tasks for domain-specific risks** identified in Phase 1:
  ```python
  @task
  def employment_fairness_eval():
      return Task(
          dataset=csv_dataset("employment_fairness_scenarios.csv"),
          solver=generate(),
          scorer=model_graded_fact()
      )
  ```
- [ ] **Archive `.eval` log files** — these are your compliance artifacts
- [ ] **Document summary findings** from each eval run in your risk register

*Articles 9, 13, 15, 17, Annex IV. The `.eval` logs are the primary documentary evidence for conformity assessment.*

---

### 3.2 Annex IV documentation package

Compile the following before deployment. This is the "test results" section of your Annex IV technical documentation:

- [ ] **Eval run summary table:**

  | Eval | Date | Model version | inspect_ai commit | Score | Pass/fail |
  |---|---|---|---|---|---|
  | AgentHarm | YYYY-MM-DD | model-v1.2 | abc1234 | 0.87 | Pass |
  | CyberSecEval | YYYY-MM-DD | model-v1.2 | abc1234 | 0.91 | Pass |
  | Domain eval | YYYY-MM-DD | model-v1.2 | abc1234 | 0.83 | Pass |

- [ ] **Archived `.eval` log files** — stored alongside model artifacts, not just in a compliance folder
- [ ] **promptfoo adversarial report** (JSON) — link from risk register
- [ ] **Bias testing results** — disaggregated by demographic dimension
- [ ] **Dataset documentation** — scope, methodology, representativeness assessment
- [ ] **Risk register** — current state, including any residual risks and accepted mitigations

---

## Phase 4 — Production Monitoring

### 4.1 Langfuse instrumentation

- [ ] **Instrument all LLM calls** with `@observe()` decorator or SDK integration
- [ ] **Set up EU-compliant data handling:**
  - EU Cloud (cloud.langfuse.com, AWS eu-central-1) — sign DPA
  - Self-hosted — verify infrastructure is in EU and access controls are documented
- [ ] **Attach quality scores to production traces** — minimum: faithfulness and answer relevancy on sampled traffic (10–20%)
- [ ] **Configure alert thresholds:**
  ```python
  # Alert when faithfulness drops below 0.75 on rolling 24-hour window
  # Alert when error rate exceeds 5%
  # Alert when latency p99 exceeds threshold
  ```
- [ ] **Configure data retention** — minimum retention for compliance: 3 years recommended (check your sector's specific requirements)

*Article 17 (post-market monitoring) and Article 9 (continuous risk management).*

---

### 4.2 Incident handling

- [ ] **Define "serious incident" threshold** — at what failure rate or severity does an event become an Art. 73 serious incident requiring notification?
- [ ] **Create incident response procedure** — who is notified, what investigation is triggered, what remediation steps
- [ ] **Verify Langfuse alerting reaches the right people** — on-call engineer and compliance owner

*Article 73 requires providers to report serious incidents to market surveillance authorities.*

---

## Phase 5 — Ongoing (Per Model Update)

### 5.1 Re-evaluation triggers

Re-run the full Phase 2–3 checklist when:

- [ ] Model version changes (including fine-tuning runs)
- [ ] System prompt changes substantively
- [ ] Retrieval corpus changes by more than 20%
- [ ] Significant new use case or user type is added
- [ ] Adversarial technique is discovered that your current suite didn't catch
- [ ] A production incident reveals a failure mode not covered by existing evals

### 5.2 Periodic review

- [ ] **Quarterly:** Review Langfuse quality trends — is performance stable or drifting?
- [ ] **Every 6 months:** Re-run inspect_ai safety evals against latest model version
- [ ] **Annually:** Review and update risk register; re-assess dataset representativeness

---

## Recommended Stacks by Risk Tier

### Tier 1 — Highest risk (employment, credit, law enforcement, biometrics)

Full stack. No shortcuts.

```
Development:   RAGAS or DeepEval (quality baseline) + DeepEval bias metrics
Adversarial:   promptfoo (broad coverage) + inspect_ai (audit-grade safety)
Production:    Langfuse self-hosted (data sovereignty) with 3-year retention
Documentation: inspect_ai .eval logs + Langfuse exports in Annex IV package
```

**Expected setup time:** 2–4 weeks for first compliant deployment.  
**Ongoing cost (1k production queries/day):** inspect_ai eval runs quarterly ($20–50/run) + Langfuse self-hosted (infra cost only).

---

### Tier 2 — High risk (education access, essential services, critical infrastructure)

Full stack with some flexibility on tooling choices.

```
Development:   RAGAS or DeepEval (quality baseline)
Adversarial:   promptfoo (OWASP LLM Top 10 minimum) + inspect_ai AgentHarm
Production:    Langfuse (EU Cloud or self-hosted)
Documentation: inspect_ai .eval logs in Annex IV package
```

**Expected setup time:** 1–2 weeks.

---

### Tier 3 — Moderate risk (education tools, general service assistants near Annex III)

Lighter stack; escalate to Tier 2 if classification changes.

```
Development:   DeepEval (CI/CD quality gate)
Adversarial:   promptfoo (pre-launch red team, minimum 1 campaign)
Production:    Langfuse (EU Cloud acceptable)
Documentation: promptfoo JSON report + DeepEval CI artifacts
```

**Expected setup time:** 3–5 days.

---

## What This Checklist Doesn't Cover

- **Deployer obligations** — if you are operating a third-party AI system (not building it), you have separate obligations under Art. 26. This checklist is for providers.
- **GPAI model compliance** — if you are deploying a general-purpose AI model (not a specific-purpose system), the Chapter V rules apply instead.
- **Notified body conformity assessment** — for some Annex III categories, a third-party conformity assessment by a notified body is required. This checklist prepares you for that assessment; it doesn't replace it.
- **Sector-specific regulation** — AI in medical devices (MDR), financial services (DORA), or aviation operates under additional sector rules that interact with the AI Act.

---

*This checklist reflects EU AI Act OJ L 2024/1689. It is practical guidance, not legal advice. Engage a notified body and legal counsel for formal conformity assessment.*

→ [Framework mapping](./framework-mapping.md) · [Back to EU AI Act module](./README.md)
