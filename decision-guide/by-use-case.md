# By Use Case

> Match your specific use case to the right framework. Each section below names the primary recommendation, the runner-up, and when you'd pick each one.

---

## Use Case Index

| Use case | Primary | Runner-up |
|---|---|---|
| [RAG quality measurement](#rag-quality-measurement) | RAGAS | DeepEval |
| [Conversational AI / chatbots](#conversational-ai--chatbots) | DeepEval | RAGAS (for RAG-backed chatbots) |
| [Red teaming / adversarial testing](#red-teaming--adversarial-testing) | promptfoo | inspect_ai |
| [Agentic / multi-step systems](#agentic--multi-step-systems) | inspect_ai | DeepEval |
| [Production monitoring](#production-monitoring) | Langfuse | — |
| [EU AI Act compliance](#eu-ai-act-compliance) | inspect_ai | Langfuse |
| [Benchmark contribution / research](#benchmark-contribution--research) | OpenAI Evals | inspect_evals |
| [CI/CD quality gates](#cicd-quality-gates) | DeepEval | promptfoo |
| [Multi-model comparison](#multi-model-comparison) | promptfoo | inspect_ai |

---

## RAG Quality Measurement

**Use this when:** You have a retrieval-augmented generation pipeline and you need to measure whether your retrieval is actually working, not just whether the LLM output sounds coherent.

### Primary: RAGAS

RAGAS is purpose-built for RAG evaluation. Its core metrics map directly to the two-stage RAG problem:

**Retrieval quality:**
- `context_precision` — are the top-k retrieved chunks actually relevant?
- `context_recall` — did you retrieve all the chunks needed to answer the question?

**Generation quality:**
- `faithfulness` — is the answer grounded in the retrieved context, or is it hallucinating?
- `answer_relevancy` — does the answer actually address the question?

These four metrics together tell you whether your RAG pipeline works. Coherent-sounding outputs with low faithfulness scores = hallucination with fluency. That's the pattern RAGAS is designed to catch.

```python
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy, context_precision, context_recall

result = evaluate(
    dataset=my_eval_dataset,
    metrics=[faithfulness, answer_relevancy, context_precision, context_recall]
)
```

**Version note:** If you're on v0.3, the v0.4 migration involves breaking changes to the metric API. Start on v0.4 if onboarding fresh.

→ [Full RAGAS profile](../frameworks/ragas.md)

---

### Runner-up: DeepEval

DeepEval has faithfulness and contextual relevancy metrics and can evaluate RAG pipelines. The trade-off: RAGAS has more retrieval-specific depth (context recall is a RAGAS-native metric not replicated well elsewhere), while DeepEval gives you better CI/CD integration.

**When to pick DeepEval over RAGAS for RAG:** When your team already uses DeepEval for non-RAG eval and you want a unified framework rather than running two separate eval stacks. You can pipe RAGAS scores into DeepEval assertions for unified CI reporting.

---

## Conversational AI / Chatbots

**Use this when:** Your LLM product is a chatbot, assistant, or any system where the primary quality dimension is response quality on open-ended questions — not retrieval accuracy.

### Primary: DeepEval

DeepEval's `GEval` metric is the most flexible tool for conversational quality evaluation. You define a rubric in plain English, and a judge model evaluates outputs against it.

```python
from deepeval.metrics import GEval
from deepeval.test_case import LLMTestCase, LLMTestCaseParams

correctness_metric = GEval(
    name="Correctness",
    criteria="Determine whether the actual output is factually correct based on the expected output",
    evaluation_params=[LLMTestCaseParams.INPUT, LLMTestCaseParams.ACTUAL_OUTPUT],
    threshold=0.7
)
```

GEval handles cases where string matching fails — open-ended responses, multiple valid answers, paraphrased correct answers.

**Also useful for chatbots:**
- `AnswerRelevancyMetric` — does the response address the question?
- `HallucinationMetric` — is anything in the response unsupported by the provided context?
- `BiasMetric` — does the response show gender/racial/political bias?

→ [Full DeepEval profile](../frameworks/deepeval.md)

---

### Runner-up: RAGAS (for RAG-backed chatbots)

If your chatbot uses retrieval to ground its answers, add RAGAS to measure the retrieval layer. RAGAS and DeepEval are complementary here: DeepEval handles the conversational quality; RAGAS handles the retrieval quality.

---

## Red Teaming / Adversarial Testing

**Use this when:** You want to find what adversarial inputs — prompt injections, jailbreaks, PII extraction attempts — can make your system do before your users do.

### Primary: promptfoo

promptfoo has the most comprehensive red teaming engine of any framework in this guide. It automatically generates adversarial test cases across attack categories and scores pass/fail.

```yaml
redteam:
  purpose: "Customer service chatbot for a bank"
  plugins:
    - owasp:llm        # Full OWASP LLM Top 10 coverage
    - pii:direct       # Direct PII extraction attempts
    - harmful:violence # Harmful content generation
    - jailbreak        # Jailbreak variants
  strategies:
    - jailbreak
    - prompt-injection
```

**Attack categories covered:**
- Prompt injection (direct and indirect via RAG context)
- Jailbreaks (role-play, DAN, encoded prompts)
- PII extraction
- Harmful content generation (12 categories)
- OWASP LLM Top 10

**Time to first red team:** 30–60 minutes. YAML-first, no Python required.

**Cost:** ~$1–$10 per campaign (one-time LLM cost to generate attack cases; subsequent runs reuse generated cases).

→ [Full promptfoo profile](../frameworks/promptfoo.md)

---

### Runner-up: inspect_ai

inspect_ai's `AgentHarm` benchmark (from the companion `inspect_evals` repo) covers 110 adversarial behaviors across 11 harm categories. It's less about broad attack surface coverage and more about systematic, peer-reviewed harm evaluation with audit-grade output.

**Pick inspect_ai over promptfoo for red teaming when:**
- You need the output to be documentable for regulatory purposes
- You're running established published benchmarks (AgentHarm, HarmBench) for comparability
- Your context is government, defence, or heavily regulated

→ [Full inspect_ai profile](../frameworks/inspect-ai.md)

---

## Agentic / Multi-step Systems

**Use this when:** Your LLM system uses tools (web browsing, code execution, API calls), executes multi-step tasks, or coordinates multiple agents.

### Primary: inspect_ai

inspect_ai has first-class support for agentic evaluation. You can give the model access to tools and evaluate whether it uses them correctly across multi-turn interactions.

```python
from inspect_ai.solver import use_tools
from inspect_ai.tool import web_browser, bash

@task
def agentic_eval():
    return Task(
        dataset=my_scenarios,
        solver=[use_tools(web_browser(), bash()), generate()],
        scorer=model_graded_fact()
    )
```

**inspect_evals library includes:**
- **GAIA** — multi-step tool-use reasoning across 3 difficulty levels
- **SWE-bench Verified** — real GitHub issue resolution with tool access

These benchmarks evaluate whether an agent can plan, use tools, recover from errors, and complete long-horizon tasks — which is exactly what you need to measure for agentic systems.

→ [Full inspect_ai profile](../frameworks/inspect-ai.md)

---

### Runner-up: DeepEval

DeepEval has multi-turn conversation evaluation and task completion metrics. For simpler agentic cases (structured tool-use with verifiable outputs), DeepEval's assertions framework is sufficient. For complex, long-horizon agentic evaluation, inspect_ai has more depth.

---

## Production Monitoring

**Use this when:** You have an LLM system in production and you need to understand what's actually happening — quality, cost, latency, edge cases — on real user traffic.

### Primary: Langfuse

Langfuse is the only framework in this guide purpose-built for production observability. The others are batch evaluation tools; Langfuse traces live traffic.

**What Langfuse gives you:**
- Every inference: input, output, latency, token count, cost, model used
- Session replay: see the full conversation a user had
- Score attachment: run an LLM judge on a sample of production traffic and attach scores to traces
- Alerting: trigger alerts when faithfulness drops below threshold
- Dataset building: export low-scoring traces to build your next golden dataset

**Key differentiator for EU deployments:** EU Cloud at cloud.langfuse.com (AWS eu-central-1, Frankfurt, Germany). GDPR-compliant. DPA available. Langfuse GmbH is a German company. Self-hosting available for full data sovereignty.

```python
from langfuse.decorators import observe

@observe()
def production_pipeline(user_query: str) -> str:
    # Everything inside is traced automatically
    ...
```

**Cost:** Free up to 50k observations/month (Cloud Hobby). Self-host for free beyond that.

→ [Full Langfuse profile](../frameworks/langfuse.md)

---

## EU AI Act Compliance

**Use this when:** Your AI system may be subject to EU AI Act obligations — particularly if it touches Annex III use cases (credit decisions, biometric identification, employment screening, critical infrastructure, etc.).

### Primary: inspect_ai

The EU AI Act strongest story in this guide. inspect_ai is maintained by the UK AI Safety Institute, which is involved in developing the international AI safety standards referenced in EU AI Act Article 40. Using inspect_ai aligns your evaluation methodology with the emerging regulatory standard.

**EU AI Act article coverage:**
- **Art. 9 (Risk management):** Safety evals (AgentHarm, HarmBench, CyberSecEval) directly instantiate risk identification requirements
- **Art. 13 (Transparency):** `.eval` logs include full input/output records
- **Art. 15 (Accuracy, robustness, cybersecurity):** AgentHarm and CyberSecEval directly address these requirements
- **Art. 17 (Quality management):** Reproducible, versioned eval runs with structured logs
- **Annex IV (Technical documentation):** `.eval` format is purpose-built for inclusion in technical documentation

**Practical recommendation:** Run AgentHarm and relevant domain evals before deployment. Include `.eval` log files in your Annex IV technical documentation. This is the most defensible evidence of safety evaluation available to product teams.

→ [Full inspect_ai profile](../frameworks/inspect-ai.md)

---

### Strong complement: Langfuse

For Article 10 (data governance) and continuous Article 17 (quality management) obligations — production monitoring and audit trail — pair inspect_ai with Langfuse EU Cloud or self-hosted.

→ [Full Langfuse profile](../frameworks/langfuse.md)

---

## Benchmark Contribution / Research

**Use this when:** You want to contribute reusable evaluations to a public registry, run established benchmarks, or produce research-grade evaluation evidence.

### Primary: OpenAI Evals

OpenAI Evals is a registry model, not a product evaluation tool. The value proposition: your eval becomes publicly available, runnable by anyone against any model, and associated with your GitHub handle.

**What you contribute:**
- A JSONL dataset of samples with expected answers
- A YAML config describing the eval type and matching method

No Python required. A domain expert — PM, lawyer, doctor, compliance officer — can write a valid OpenAI Eval without engineering support.

**PM-relevant opportunity (as of 2026-05-20):**
- EU AI Act Articles 9–17 practical application — genuinely unclaimed territory
- Multi-agent coordination patterns (correct vs. incorrect architectures)
- RAG system design decisions

→ [Full OpenAI Evals profile](../frameworks/openai-evals.md)

---

### Runner-up: inspect_evals

For research-grade benchmark contribution to a peer-reviewed library: the `inspect_evals` companion repo accepts contributions. These are held to a higher standard than the OpenAI Evals registry (peer review by UK AISI staff), but carry institutional credibility proportional to that bar.

---

## CI/CD Quality Gates

**Use this when:** You want LLM quality to be a first-class part of your CI/CD pipeline — PRs fail if quality drops below a threshold.

### Primary: DeepEval

DeepEval is the only framework in this guide with native pytest integration. Write LLM test cases the same way you'd write unit tests; fail the CI build on metric threshold violations.

```bash
# In your CI pipeline
deepeval test run tests/llm_quality_tests.py
```

**What makes DeepEval CI-first:**
- pytest plugin — runs with `pytest` or `deepeval test run`
- Threshold-based assertions — test fails if metric < threshold
- JUnit-compatible output — integrates with any CI system
- Parallel test execution — doesn't serialize your pipeline

→ [Full DeepEval profile](../frameworks/deepeval.md)

---

### Runner-up: promptfoo

promptfoo's assertion-based eval (`contains`, `regex`, `json-schema`, `llm-rubric`) runs as a CI step. For adversarial regression testing specifically — "did this PR make the model more vulnerable to prompt injection?" — promptfoo is the right CI gate. For general quality metrics, DeepEval is more complete.

---

## Multi-model Comparison

**Use this when:** You're choosing between models (GPT-4o vs. Claude vs. Gemini) for a specific application, or evaluating whether a model upgrade improves quality.

### Primary: promptfoo

promptfoo runs the same prompt suite against multiple providers in parallel and renders a side-by-side comparison table.

```yaml
providers:
  - openai:gpt-4o
  - anthropic:claude-3-5-sonnet-20241022
  - google:gemini-1.5-pro

tests:
  - vars:
      document: "{{sample_document}}"
    assert:
      - type: llm-rubric
        value: "Summary is accurate and under 100 words"
```

```bash
npx promptfoo eval --view
```

The web UI shows every model's output on every test case with pass/fail per assertion. This is the fastest way to answer "is model B better than model A for our specific use case?"

→ [Full promptfoo profile](../frameworks/promptfoo.md)

---

### Runner-up: inspect_ai

For research-grade multi-model comparison on established benchmarks (GAIA, MMMU, SWE-bench), inspect_ai provides comparable output across runs. The `.eval` format makes it straightforward to compare multiple models on the same eval task with full reproducibility.

---

→ Back to [Decision Guide](./README.md) | [By Team Type](./by-team-type.md) | [By Lifecycle Stage](./by-lifecycle-stage.md)
