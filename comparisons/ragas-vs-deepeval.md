---
title: RAGAS vs DeepEval
parent: Comparisons
nav_order: 1
---

# RAGAS vs DeepEval

> The most common comparison question in LLM evaluation. Short answer: they solve different problems. Longer answer below.

---

## The One-Line Version

**RAGAS** measures whether your RAG pipeline is working — retrieval quality, grounding, faithfulness.  
**DeepEval** tests whether your LLM outputs meet a quality bar — pass/fail assertions, CI/CD integration, multi-metric test suites.

They are frequently framed as competitors. In practice, most teams with RAG products use both: RAGAS for the metrics that matter, DeepEval for the test framework that runs them in CI.

---

## At a Glance

| Dimension | RAGAS | DeepEval |
|---|---|---|
| Primary purpose | RAG pipeline quality measurement | LLM output testing (pass/fail) |
| Test runner | No native runner — library of metrics | pytest plugin — native CI/CD |
| RAG-specific depth | ● ● ● ● ● | ● ● ● ○ ○ |
| General LLM coverage | ● ● ○ ○ ○ | ● ● ● ● ○ |
| Conversational / multi-turn | ○ ○ ○ ○ ○ | ● ● ● ○ ○ |
| Synthetic test data generation | ● ● ● ● ○ | ● ● ○ ○ ○ |
| CI/CD integration | Manual | Native (pytest) |
| Setup complexity | Medium | Medium |
| LLM judge required | Yes (most metrics) | Yes (most metrics) |
| Self-host capable | Yes | Yes |
| EU AI Act relevance | Low | Low–Medium |
| GitHub stars | ~14k | ~16k |
| License | Apache 2.0 | Apache 2.0 |

---

## What RAGAS Does That DeepEval Doesn't

### Retrieval-specific metrics

RAGAS has metrics that don't exist in DeepEval:

**Context Recall** — "Did you retrieve all the information needed to answer the question?" This requires a ground-truth answer and measures whether your retrieval step missed relevant information. DeepEval has no equivalent.

**Context Precision** — "Of the chunks you retrieved, how many were actually relevant?" Measures retrieval signal-to-noise. DeepEval's contextual precision metric approximates this but with different semantics.

**Context Entity Recall** — "Did your retrieved chunks contain the named entities in the ground-truth answer?" A more targeted retrieval completeness signal useful for knowledge-intensive queries.

These three metrics are the reason RAG teams reach for RAGAS even when they're already using DeepEval for everything else.

### Synthetic test generation

RAGAS's `TestsetGenerator` produces synthetic question/context/answer triples from your document corpus — it reads your documents and generates evaluation samples automatically.

```python
from ragas.testset import TestsetGenerator

generator = TestsetGenerator.from_langchain(
    generator_llm=ChatOpenAI(model="gpt-4o"),
    critic_llm=ChatOpenAI(model="gpt-4o-mini"),
    embeddings=OpenAIEmbeddings()
)
testset = generator.generate_with_langchain_docs(documents, test_size=50)
```

This is the fastest path from "I have a document corpus" to "I have an eval dataset." DeepEval has a dataset synthesis feature but it's less mature and less retrieval-aware.

---

## What DeepEval Does That RAGAS Doesn't

### pytest integration

DeepEval is built as a pytest plugin. Tests run with `pytest` or `deepeval test run`. Results integrate with any CI system that understands JUnit output.

```python
# tests/test_llm_quality.py
from deepeval import assert_test
from deepeval.test_case import LLMTestCase
from deepeval.metrics import AnswerRelevancyMetric, FaithfulnessMetric

def test_rag_response():
    test_case = LLMTestCase(
        input="What is the cancellation policy?",
        actual_output=pipeline("What is the cancellation policy?"),
        retrieval_context=retrieved_chunks
    )
    assert_test(test_case, [
        AnswerRelevancyMetric(threshold=0.8),
        FaithfulnessMetric(threshold=0.85)
    ])
```

RAGAS has no native CI integration. You call it in a script and check the output manually. To use RAGAS in CI, you wrap it yourself — many teams use DeepEval as the test harness and pipe RAGAS scores in as custom metrics.

### Metrics beyond RAG

DeepEval covers use cases that RAGAS doesn't address:

| Metric | DeepEval | RAGAS |
|---|---|---|
| G-Eval (custom rubric) | ✓ | ✗ |
| Hallucination | ✓ | ✗ (faithfulness is the approximate) |
| Toxicity | ✓ | ✗ |
| Bias | ✓ | ✗ |
| Summarization quality | ✓ | ✗ |
| Role adherence | ✓ | ✗ |
| JSON correctness | ✓ | ✗ |
| Task completion | ✓ | ✗ |
| Multi-turn conversation | ✓ | ✗ |

If your LLM product does anything beyond RAG — a chatbot, an agent, a summariser, a classifier — DeepEval covers it. RAGAS doesn't.

### Threshold-based pass/fail

DeepEval's central design principle is the threshold: every metric has a minimum score, and the test fails if the actual score falls below it. This makes LLM quality a binary gate — the build passes or fails — which is what CI/CD requires.

RAGAS returns raw scores. You decide what to do with them. For a CI gate, you'd write the threshold logic yourself.

---

## Cost Comparison

Both frameworks use LLM judges for most metrics, so cost scales with:
- Number of samples
- Number of metrics evaluated
- Judge model choice (GPT-4o vs GPT-4o-mini vs local)

| Scenario | RAGAS | DeepEval |
|---|---|---|
| 100 samples, 3 metrics, GPT-4o judge | ~$1–3 | ~$1–3 |
| 100 samples, 3 metrics, GPT-4o-mini judge | ~$0.10–0.30 | ~$0.10–0.30 |
| 1,000 samples, 5 metrics, GPT-4o-mini | ~$1–3 | ~$1–3 |
| NonLLM context precision (RAGAS only) | Free | N/A |

**Cost reduction strategies:**
- **RAGAS:** Use `NonLLMContextPrecisionWithReference` for binary precision cases — no LLM call. Note: this class is still active in v0.4 despite deprecation confusion in older docs.
- **DeepEval:** Run expensive metrics (GEval, Hallucination) only on a sample of production traffic; run fast deterministic metrics (JSON correctness, regex) on every test case.
- **Both:** Use a cheaper judge (GPT-4o-mini, Claude Haiku) at threshold-setting time, reserve GPT-4o for close calls.

---

## Version Compatibility

RAGAS v0.3 → v0.4 was a breaking migration: the metric import paths, dataset format, and several metric names changed. If you're starting fresh, install v0.4 directly. If you're upgrading an existing integration, budget a sprint for migration.

DeepEval v1.x is broadly stable. Major version bumps are less frequent and less disruptive.

Both frameworks evolve fast. Pin to a specific minor version in your requirements file and upgrade on a schedule, not continuously.

---

## When to Pick RAGAS

Choose RAGAS as your primary framework when:

- **You're building a RAG product** and need to measure retrieval quality (context recall, context precision) specifically
- **You need synthetic test data** from your document corpus before you have real user queries
- **You want the most complete set of retrieval metrics** and are willing to build your own CI harness around them
- **Your team is Python-native** and comfortable with a library rather than a framework

Don't use RAGAS alone if you need a CI/CD quality gate — you'll end up writing plumbing that DeepEval already provides.

---

## When to Pick DeepEval

Choose DeepEval as your primary framework when:

- **You need CI/CD integration now** — tests that fail the build when quality drops
- **Your product isn't purely RAG** — conversational AI, agents, summarisation, classification
- **You want one framework for everything** — G-Eval's custom rubric handles cases RAGAS can't
- **Your team already uses pytest** — DeepEval tests look and feel like unit tests
- **You need multi-turn conversation evaluation** — DeepEval supports it natively

---

## When to Use Both

This is the most common real-world pattern for RAG products:

```
RAGAS:   measures context_recall, context_precision, faithfulness (retrieval layer)
DeepEval: runs those scores as assertions in pytest CI + adds answer_relevancy, hallucination (output layer)
```

Concretely:
1. Run RAGAS evaluate() on your dataset → get metric scores as a dict
2. Pass those scores into DeepEval test cases as custom metrics or use DeepEval's built-in RAG metrics
3. Use DeepEval's threshold assertions to gate CI

This gives you RAGAS's retrieval depth inside DeepEval's test runner. It's more setup than picking one, but it's the most complete coverage.

---

## The Verdict

| If your question is… | Answer |
|---|---|
| "Is my retrieval working?" | RAGAS |
| "Are my LLM outputs good enough?" | DeepEval |
| "I need CI/CD quality gates" | DeepEval |
| "I need synthetic test data" | RAGAS |
| "I have a chatbot, not a RAG system" | DeepEval |
| "I want the most complete RAG coverage" | RAGAS + DeepEval |
| "We're a 3-person team, pick one" | Start with RAGAS if RAG product; DeepEval if not |

---

→ [RAGAS profile](../frameworks/ragas.md) · [DeepEval profile](../frameworks/deepeval.md) · [Full matrix](./full-matrix.md) · [Back to comparisons](./)
