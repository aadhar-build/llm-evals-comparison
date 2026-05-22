---
title: RAGAS
parent: Framework Profiles
nav_order: 1
---

# RAGAS

> The go-to framework for evaluating RAG pipelines — measures whether your retrieval and generation steps are actually working, not just whether the output sounds plausible.

**GitHub:** [explodinggradients/ragas](https://github.com/explodinggradients/ragas) · ~14k stars · Apache 2.0  
**Docs:** [docs.ragas.io](https://docs.ragas.io)

---

## At a Glance

| Dimension | Rating |
|---|---|
| Setup effort | ● ● ○ ○ ○ |
| Coding required | ● ● ● ○ ○ |
| RAG evaluation | ● ● ● ● ● |
| Conversational / general LLM | ● ● ○ ○ ○ |
| Red teaming | ○ ○ ○ ○ ○ |
| Production observability | ● ○ ○ ○ ○ |
| EU AI Act support | ● ○ ○ ○ ○ |
| Cost at scale | Medium (LLM-as-judge default) |

---

## Best For

- **RAG quality measurement** — evaluating whether retrieved context is relevant, whether the model uses it faithfully, and whether answers are grounded
- **Pre-production RAG testing** — catching retrieval failures, hallucinations, and low-precision contexts before they reach users
- **Synthetic test data generation** — building evaluation datasets from your own documents without manual labeling
- **Teams using LangChain or LlamaIndex** — RAGAS has first-class integrations with both

## Not Great For

- **Non-RAG use cases** — metrics are purpose-built for retrieval-augmented generation; applying them to conversational AI or agents produces misleading results
- **Production monitoring** — RAGAS is a batch evaluation tool, not a real-time observability layer; use Langfuse for production traces
- **Red teaming** — no adversarial or safety testing capabilities
- **Teams without LLM API budget** — default metrics use LLM-as-judge; at scale, cost adds up fast (see Cost section)

---

## Core Metrics

RAGAS organizes its metrics around the two stages of RAG: retrieval and generation.

### Retrieval Metrics
| Metric | What it measures | Reference needed? |
|---|---|---|
| Context Precision | Are relevant chunks ranked higher than irrelevant ones? | Yes (with reference) or No (without, uses response) |
| Context Recall | Does the retrieved context cover everything in the reference answer? | Yes |
| Context Entity Recall | Are named entities from the reference present in retrieved contexts? | Yes |
| NonLLM Context Precision | Same as Context Precision using Levenshtein distance instead of LLM judge | Yes |

### Generation Metrics
| Metric | What it measures | Reference needed? |
|---|---|---|
| Faithfulness | Is the answer grounded in the retrieved context? (hallucination detector) | No |
| Answer Relevancy | Is the answer actually responsive to the question? | No |
| Answer Correctness | Is the answer factually correct? | Yes |

**Practical note:** Faithfulness + Answer Relevancy are your first two metrics in almost every RAG project — they require no reference answers and catch the two most common failure modes.

---

## Tradeoffs vs. Alternatives

**vs. DeepEval**
Pick RAGAS when your primary concern is retrieval quality (are we fetching the right chunks?). Pick DeepEval when you want a general-purpose testing framework with CI/CD integration, or when you're evaluating non-RAG LLM behavior. For RAG projects, many teams use both: RAGAS for retrieval-specific metrics, DeepEval for test suite structure and CI integration.

**vs. LangSmith**
LangSmith bundles evaluation with tracing in a commercial product. RAGAS is evaluation-only and open source. If you're already on LangSmith, RAGAS metrics can run within it. If you're not, RAGAS + Langfuse is a fully open-source equivalent.

**vs. Braintrust**
Braintrust has stronger step-efficiency and CI/CD story for general LLM evals. RAGAS wins on RAG-specific depth (more retrieval metrics, better synthetic test generation).

---

## Integration Effort

**Time to first eval:** 30–60 minutes for a team with an existing RAG pipeline

**Minimum requirements:**
```python
pip install ragas
```

**Minimum working example:**
```python
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy
from datasets import Dataset

data = {
    "question": ["What is the Eiffel Tower?"],
    "answer": ["The Eiffel Tower is in Paris."],
    "contexts": [["The Eiffel Tower, located in Paris, was built in 1889."]],
}

result = evaluate(Dataset.from_dict(data), metrics=[faithfulness, answer_relevancy])
```

**Works with:** LangChain, LlamaIndex, raw OpenAI SDK, any Python RAG stack  
**LLM requirements:** Requires an LLM API for judge metrics (OpenAI, Anthropic, or any LiteLLM-compatible model)

---

## Cost at Scale

| Volume | Estimated cost (GPT-4o as judge) |
|---|---|
| 100 eval samples | ~$0.50–$2 |
| 1,000 eval samples | ~$5–$20 |
| 10,000 eval samples | ~$50–$200 |

*Varies significantly by metric. Faithfulness is the most expensive (multiple LLM calls per sample). NonLLM-based metrics (NonLLMContextPrecisionWithReference) cost nothing beyond compute.*

**Self-hosting:** RAGAS is open source; you can use any LLM as judge including local models via Ollama, reducing API cost to near-zero at the expense of judge quality.

---

## Synthetic Test Data Generation

A capability often overlooked: RAGAS can generate evaluation datasets from your own documents, which eliminates the bottleneck of manually labeling ground truth.

```python
from ragas.testset import TestsetGenerator

generator = TestsetGenerator(llm=your_llm, embedding_model=your_embeddings)
testset = generator.generate_with_langchain_docs(docs, testset_size=50)
```

This is particularly valuable for domain-specific RAG applications where general benchmarks don't apply.

---

## EU AI Act Relevance

**Moderate.** RAGAS contributes to compliance in these areas:

- **Article 9 (Risk management):** Provides documented, repeatable evaluation methodology for RAG systems. Faithfulness scores give quantitative evidence of hallucination risk management.
- **Article 13 (Transparency):** Metric definitions are publicly documented and reproducible — supports transparency claims about evaluation methodology.
- **Annex IV (Technical documentation):** Evaluation results stored in RAGAS `EvaluationResult` objects can be exported and included in technical documentation.

**Gaps:** No audit trail integration (no direct Langfuse/logging pipeline). No adversarial testing. No data governance features.

For EU high-risk AI systems using RAG, RAGAS should be part of a broader stack that includes Langfuse (audit trails) and inspect_ai or promptfoo (adversarial testing).

---

## Version Tracked

- **Current stable:** v0.4.x  
- **Last verified:** 2026-05-20  
- **Migration note:** v0.3 → v0.4 introduced breaking changes to the metric API. The legacy `LLMContextPrecisionWithReference` / `LLMContextPrecisionWithoutReference` classes are deprecated in v0.4 and scheduled for removal in v1.0. Migrate to the collections-based API (`from ragas.metrics.collections import ContextPrecision`).
- **NonLLM metrics:** `NonLLMContextPrecisionWithReference` is **not deprecated** — only the LLM-based legacy API is being phased out.
