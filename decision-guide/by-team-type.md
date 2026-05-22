---
title: By Team Type
parent: Decision Guide
nav_order: 2
---

# By Team Type

> Framework choice is as much about team context as technical requirements. A 3-person startup and a regulated enterprise asking "which LLM eval framework?" have almost nothing in common. This guide cuts by team type.

---

## Team Type Overview

| Team type | Primary constraint | Start here |
|---|---|---|
| [3-person RAG startup](#3-person-rag-startup) | Speed — minimal infra overhead | RAGAS → Langfuse |
| [Enterprise ML platform team](#enterprise-ml-platform-team) | Scale and standardisation | DeepEval + RAGAS + Langfuse |
| [Regulated EU company](#regulated-eu-company) | Compliance and audit trail | inspect_ai + Langfuse (EU Cloud) |
| [Research or safety team](#research-or-safety-team) | Rigor and reproducibility | inspect_ai |

---

## 3-Person RAG Startup

**Context:** Small team, moving fast, building an LLM product (likely RAG-based). No dedicated ML engineer. Engineering resources are precious. Every tool must justify its setup cost.

**Constraints:**
- Time-to-value in hours, not days
- No dedicated infra budget
- OpenAI or Anthropic API as the sole LLM dependency (no custom model)
- No regulatory pressure yet

### Recommended Stack

**Step 1: RAGAS for retrieval quality (Days 1–2)**

RAGAS tells you whether your RAG pipeline is actually working — not just whether the LLM can talk fluently, but whether it's using retrieved context correctly. This is the metric gap that kills RAG products in production.

```bash
pip install ragas
```

Start with three metrics: `faithfulness`, `answer_relevancy`, `context_precision`. If these three look healthy (>0.8), your retrieval layer is working. Set them as thresholds in a pytest fixture and run them on every PR that touches your retrieval or prompt logic.

**Cost:** ~$1–3 per 100 samples with GPT-4o as judge. At 3 eval runs/week, that's under $10/month.

**Step 2: Langfuse for production tracing (Days 3–5)**

Once you're live, you need to see what's actually happening. Langfuse's free tier (50k observations/month) covers most early-stage products. The `@observe()` decorator wraps your pipeline in a few lines.

```python
from langfuse.decorators import observe

@observe()
def rag_pipeline(question: str) -> str:
    ...
```

The dashboard shows you: which queries are failing, what the cost-per-query is, and which user sessions have low LLM judge scores.

**Note:** Use Langfuse Cloud's EU endpoint (cloud.langfuse.com) from day one if you have any EU users — it's free and avoids a data migration later.

**Step 3: promptfoo red team before your first public launch (Day before launch)**

Before you open the product to users: 30 minutes with promptfoo to generate adversarial test cases.

```bash
npm install -g promptfoo
promptfoo redteam run
```

You will almost certainly find something. The most common early-stage findings: prompt injection via user input, unexpected behavior when context is empty, PII in outputs when the retrieval step returns documents containing personal data.

### What to skip (for now)

- **DeepEval** — adds overhead without adding metric coverage that RAGAS doesn't give you for a RAG product. Add it when you start building non-RAG features.
- **inspect_ai** — the institutional-grade safety framework is overkill for a startup without regulatory pressure. Revisit when you have compliance requirements or when preparing for enterprise sales.
- **OpenAI Evals** — useful for contributing benchmarks, not for instrumenting your own product.

### Upgrade trigger

Add DeepEval to the stack when: your product grows beyond RAG (conversational, agentic, summarization) and you need multi-metric CI assertions that RAGAS doesn't cover.

Add inspect_ai when: you're selling to enterprise or regulated industries, or when you start preparing for EU AI Act compliance.

---

## Enterprise ML Platform Team

**Context:** Internal ML platform team managing 20–100+ models in production. Mix of RAG, fine-tuned, and agentic systems. Engineering team with ML expertise. Eval infrastructure needs to serve multiple product teams.

**Constraints:**
- Eval needs to be standardised — different product teams need to use the same framework
- CI/CD integration is non-negotiable
- Cost at scale matters — you're running thousands of eval samples per day
- Data residency requirements likely (enterprise clients, GDPR)

### Recommended Stack

**Core testing layer: DeepEval**

For a platform team, DeepEval's pytest integration is the right foundation. It standardises the eval interface across product teams: every team writes LLMTestCase objects with the same metric structure, and CI gates on the same threshold mechanism.

```python
# Shared platform wrapper — each product team implements this interface
from deepeval.test_case import LLMTestCase
from deepeval.metrics import AnswerRelevancyMetric, FaithfulnessMetric

class PlatformEval:
    def run(self, input: str, output: str, context: list[str]) -> dict:
        test_case = LLMTestCase(input=input, actual_output=output, retrieval_context=context)
        metrics = [AnswerRelevancyMetric(threshold=0.7), FaithfulnessMetric(threshold=0.8)]
        # returns metric results for CI assertion
```

**Retrieval layer: RAGAS**

Run RAGAS as the retrieval-specific metric provider. Pipe RAGAS scores into DeepEval test cases for unified CI reporting.

**Cost at scale:** At 1,000 samples/day, LLM-judge-based metrics cost $5–50/day depending on the judge model and metric count. Mitigation: run full eval suite weekly, fast deterministic subset (regex, JSON schema assertions via promptfoo) on every PR.

**Production observability: Langfuse (self-hosted)**

At platform team scale, cloud billing is non-trivial. Langfuse's self-hosted option (Docker Compose or Helm) eliminates per-observation cost and gives you full data control for GDPR compliance. The Helm chart (`langfuse/langfuse-k8s`) deploys on your existing Kubernetes cluster.

```yaml
# Helm install on existing k8s cluster
helm repo add langfuse https://langfuse.github.io/langfuse-k8s
helm install langfuse langfuse/langfuse
```

**Pre-production red teaming: promptfoo**

Run promptfoo as part of the pre-production release gate for every model update. The YAML config is version-controlled alongside your model artifacts. Product teams can extend the attack config for their specific contexts.

**Adversarial depth: inspect_ai (for high-risk models)**

For models used in consequential decisions (credit scoring, hiring support, medical information), add inspect_ai as a release gate. The `.eval` log files become part of your model release documentation. This isn't needed for every model — reserve it for high-stakes deployments.

### Operational recommendations

- **Eval versioning:** Store RAGAS/DeepEval metric configs in your MLOps artifact store alongside model weights. When re-evaluating a model version, use the config that was active at release time — not the current version.
- **Cost dashboards:** Instrument eval LLM judge calls in Langfuse. Eval infrastructure spend belongs in your cost visibility stack.
- **Golden dataset management:** A shared repo of golden datasets (per domain, per team) prevents every team from defining their own ground truth differently. DeepEval supports dataset import from JSONL.

---

## Regulated EU Company

**Context:** Building AI systems in fintech, healthtech, legaltech, or government supply chains. Subject to EU AI Act (enforceably August 2026). Products may be classified as high-risk under Annex III. Procurement requires security and compliance documentation.

**Constraints:**
- Eval must produce audit-grade evidence, not just a pass/fail report
- Data residency: EU only (no US-hosted tooling for PII or inference data)
- Traceability: every model decision must be reconstructable
- Article 9 (risk management), Article 17 (quality management), Annex IV (technical documentation) create specific eval obligations

### Non-negotiable choices

**Production observability: Langfuse EU Cloud or self-hosted**

For any system processing personal data or consequential decisions:
- **EU Cloud** (cloud.langfuse.com) — AWS eu-central-1 (Frankfurt), German company (Langfuse GmbH), DPA available. Free tier through Pro tier all covered. Use if you want managed infrastructure.
- **Self-hosted** — full data sovereignty, no data leaves your infrastructure. Use if your data classification prohibits third-party processors.

Do **not** use LangSmith for regulated EU deployments — US-hosted, no EU-only option.

**Pre-deployment safety eval: inspect_ai**

inspect_ai is the framework used by Anthropic, Google DeepMind, and the UK AI Safety Institute for safety evaluations. Its `.eval` log format is purpose-built for regulatory documentation.

**Why inspect_ai specifically:**
- Audit-grade logs: full input/output records, model attribution, timestamps
- inspect_evals includes AgentHarm (110 adversarial behaviors), CyberSecEval, HarmBench
- UK AISI is involved in developing harmonised standards under EU AI Act Article 40 — using inspect_ai aligns with the emerging standard
- No semver: pin to a commit in your requirements file so eval methodology is reproducible and documentable

```bash
# Pin for audit reproducibility
git+https://github.com/UKGovernmentBEIS/inspect_ai.git@<commit-hash>
```

**Development evals: DeepEval**

DeepEval's pytest integration makes eval results CI artifacts, not manual reports. For a compliance context, every eval run should produce a machine-readable artifact stored alongside model release documentation.

**Important:** Disable PostHog telemetry before deploying in a regulated environment:
```bash
deepeval login --telemetry-opt-out
```

### Article → Framework mapping (quick reference)

| EU AI Act requirement | Framework that addresses it |
|---|---|
| Art. 9 — Risk management system | inspect_ai (safety evals), promptfoo (adversarial testing) |
| Art. 10 — Data governance | Langfuse (inference audit trail) |
| Art. 13 — Transparency | Langfuse (trace data), DeepEval (metric documentation) |
| Art. 17 — Quality management | DeepEval (CI-integrated regression), Langfuse (production scoring) |
| Annex IV — Technical documentation | inspect_ai `.eval` logs, OpenAI Evals registry entries |

→ Full mapping in [EU AI Act module](../eu-ai-act/framework-mapping.md) *(coming in Session 5)*

### The hard recommendation

If you are a regulated EU company and your product touches Annex III use cases (credit scoring, biometric identification, CV screening, critical infrastructure), your eval stack is not optional — it's an obligation. The practical minimum:

1. inspect_ai safety eval run before every production deployment, with `.eval` logs archived
2. Langfuse (EU Cloud or self-hosted) for continuous production tracing
3. DeepEval in CI for quality regression testing

This stack is also your most defensible answer to the question "how did you evaluate your AI system?" in a conformity assessment.

---

## Research or Safety Team

**Context:** Academic or frontier AI research team, AI safety organisation, or government-adjacent body. Primary concern: rigorous, reproducible evaluation that stands up to peer review and regulatory scrutiny.

**Constraints:**
- Reproducibility — the same eval must produce the same result months later
- Institutional credibility — "we ran our own tests" is less credible than "we used the same framework as the UK AISI"
- Multi-model comparison — evaluating capabilities and risks across different models
- Long-horizon and agentic evaluation — multi-turn, tool-using scenarios

### Primary framework: inspect_ai

inspect_ai is built for exactly this context. Key properties that matter for research use:

**Reproducibility:** Every eval run is a self-contained `.eval` log archive containing the full configuration, all samples, all outputs, all scores. Run the same eval on the same commit six months later — the output is comparable.

**inspect_evals companion library:** Peer-reviewed, published evaluations including:
- **AgentHarm** — 110 adversarial behaviors across 11 harm categories (red teaming benchmark)
- **GAIA** — multi-step tool-use reasoning (Level 1–3 difficulty)
- **SWE-bench Verified** — real GitHub issue resolution
- **MMMU** — multimodal reasoning
- **CyberSecEval** — cybersecurity knowledge and risks
- **HarmBench** — broad harmful behavior evaluation

These are the evaluations that appear in Anthropic and DeepMind safety reports. Running them gives your results direct comparability with published frontier lab safety assessments.

**Multi-provider support:** inspect_ai connects to Anthropic, OpenAI, Google, Azure, AWS Bedrock, Ollama, or any custom HTTP endpoint — compare models on the same eval suite.

**Agentic evaluation:** First-class support for multi-turn and tool-using agents including web browsing, code execution, and file operations.

**Versioning for research:** Pin to a specific commit hash. Cite the commit in your paper or report. This is the equivalent of citing a specific version of a measurement instrument.

```
# requirements.txt
git+https://github.com/UKGovernmentBEIS/inspect_ai.git@<commit-hash>
```

### When to add other frameworks

- **OpenAI Evals:** Contribute domain-specific evaluations to the public registry. Published evals are citable and associated with your institution's GitHub handle.
- **promptfoo:** Rapid hypothesis testing for new attack categories before formalising them in inspect_ai tasks.
- **Langfuse:** If running a live AI system as part of research (user studies, deployed prototypes), Langfuse provides trace collection infrastructure.

### Practical note on inspect_ai setup

inspect_ai requires Python and has a steeper learning curve than YAML-first tools. Expect 2–4 hours to run your first custom Task. The documentation at [inspect-ai.aisi.gov.uk](https://inspect-ai.aisi.gov.uk) is the authoritative source.

→ [Full inspect_ai profile](../frameworks/inspect-ai.md)

---

→ Back to [Decision Guide](./README.md) | [By Use Case](./by-use-case.md) | [By Lifecycle Stage](./by-lifecycle-stage.md)
