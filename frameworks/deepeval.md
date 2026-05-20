# DeepEval

> LLM unit testing that works like pytest — write test cases, set pass/fail thresholds, run in CI/CD. The closest thing to test-driven development for LLM applications.

**GitHub:** [confident-ai/deepeval](https://github.com/confident-ai/deepeval) · ~16k stars · Apache 2.0  
**Docs:** [docs.confident-ai.com](https://docs.confident-ai.com)  
**Cloud product:** Confident AI (optional, commercial)

---

## At a Glance

| Dimension | Rating |
|---|---|
| Setup effort | ● ● ○ ○ ○ |
| Coding required | ● ● ● ○ ○ |
| RAG evaluation | ● ● ● ○ ○ |
| Conversational / general LLM | ● ● ● ● ○ |
| Red teaming | ● ● ○ ○ ○ |
| Production observability | ● ● ○ ○ ○ (via Confident AI cloud) |
| EU AI Act support | ● ● ○ ○ ○ |
| Cost at scale | Medium (LLM-as-judge default) |

---

## Best For

- **LLM unit testing in CI/CD** — DeepEval's pytest integration is the most mature in the space; test cases run as part of your existing test suite
- **General LLM quality regression testing** — catching output quality degradation when you change models, prompts, or retrieval parameters
- **Teams that want RAGAS metrics with better developer tooling** — DeepEval supports RAGAS-compatible metrics but wraps them in a more structured test framework
- **Red teaming at the individual-test level** — built-in attack modules for prompt injection, jailbreaks, and PII leakage (less comprehensive than promptfoo)
- **Multi-turn conversational evaluation** — explicit support for conversation history in test cases

## Not Great For

- **Deep RAG retrieval analysis** — RAGAS has more retrieval-specific metrics and better synthetic dataset generation for RAG-heavy workflows
- **Production monitoring without Confident AI** — the OSS version is batch-only; production observability requires the commercial Confident AI platform
- **Safety/adversarial at scale** — red teaming capabilities exist but are less mature than promptfoo's dedicated red teaming engine
- **Non-Python stacks** — evaluation is Python-only; no native TypeScript support for JS/TS LLM apps

---

## Core Metrics

DeepEval offers 20+ metrics across four categories. The most-used:

### LLM-as-Judge Metrics (G-Eval style)
| Metric | What it measures | RAG? | Reference? |
|---|---|---|---|
| G-Eval | Custom criterion evaluation — define any quality dimension in natural language | Optional | Optional |
| Answer Relevancy | Is the response relevant to the input? | Yes/No | No |
| Faithfulness | Is the response grounded in the provided context? | Yes | No |
| Contextual Precision | Are relevant contexts ranked higher? | Yes | Yes |
| Contextual Recall | Does retrieved context cover the reference? | Yes | Yes |
| Hallucination | Does the response contradict the provided context? | Yes | No |

### Deterministic Metrics (no LLM cost)
| Metric | What it measures |
|---|---|
| Exact Match | String equality check |
| BLEU / ROUGE | n-gram overlap with reference |
| JSON Correctness | Valid JSON output with optional schema |
| Tool Correctness | Did the LLM call the right tools in the right order? |

### Safety Metrics
| Metric | What it measures |
|---|---|
| Toxicity | Harmful content in output |
| Bias | Demographic bias in responses |
| PII Leakage | Personally identifiable information in output |

---

## The pytest Integration

This is DeepEval's strongest differentiator — test cases look and run like standard Python tests:

```python
from deepeval import assert_test
from deepeval.metrics import AnswerRelevancyMetric, FaithfulnessMetric
from deepeval.test_case import LLMTestCase

def test_rag_response():
    test_case = LLMTestCase(
        input="What causes hallucination in LLMs?",
        actual_output=your_rag_pipeline("What causes hallucination in LLMs?"),
        retrieval_context=["...retrieved chunks..."]
    )
    assert_test(test_case, [
        AnswerRelevancyMetric(threshold=0.7),
        FaithfulnessMetric(threshold=0.8)
    ])
```

Run with: `deepeval test run test_rag.py` or standard `pytest`.

**CI/CD:** GitHub Actions, CircleCI, GitLab CI — DeepEval produces JUnit-compatible XML output; failures block merges exactly like failing unit tests.

---

## Tradeoffs vs. Alternatives

**vs. RAGAS**
Pick DeepEval when you want pytest-style CI/CD integration and a broader metric library beyond RAG. Pick RAGAS when retrieval quality is your primary concern, or when you need synthetic test data generation. For RAG projects, many teams use RAGAS metrics *within* DeepEval's test framework — they're compatible.

**vs. promptfoo**
Pick DeepEval for quality regression testing against known-good baselines. Pick promptfoo when you need systematic adversarial testing across hundreds of attack variations. DeepEval's red teaming is suitable for "does this prompt fail this safety check"; promptfoo is suitable for "find novel attack vectors against this LLM."

**vs. Langfuse**
Different jobs. DeepEval is for pre-production testing. Langfuse is for production observability. Run DeepEval before deploy, Langfuse after. They're complementary, not competing.

---

## Integration Effort

**Time to first eval:** 20–40 minutes

```bash
pip install deepeval
deepeval login  # optional, for Confident AI dashboard
```

**Works with:** Any Python LLM stack (OpenAI SDK, LangChain, LlamaIndex, Anthropic SDK, raw HTTP)  
**LLM requirements:** Requires an LLM for judge metrics — OpenAI default, configurable to any model via `deepeval set-local-model`

---

## Cost at Scale

| Volume | Estimated cost (GPT-4o as judge) |
|---|---|
| 100 test cases | ~$1–$5 |
| 1,000 test cases | ~$10–$50 |
| 10,000 test cases | ~$100–$500 |

*G-Eval is the most expensive (multiple prompts per criterion). Deterministic metrics (Exact Match, JSON Correctness, BLEU) cost nothing.*

**Mitigation:** Use a cheaper judge model (GPT-4o-mini, Claude Haiku) for development runs; reserve GPT-4o-class judges for release-gate evaluations. DeepEval supports per-metric model configuration.

---

## Telemetry Note

DeepEval collects telemetry via **PostHog** by default (event names, metric names, Jupyter flag, anonymous UUID, public IP). This is opt-out:

```bash
export DEEPEVAL_TELEMETRY_OPT_OUT=1
```

Error reporting via **Sentry** is separate and opt-in:
```bash
export ERROR_REPORTING=1
```

Both are relevant for enterprise deployments with data residency requirements.

---

## EU AI Act Relevance

**Moderate-to-good** — the strongest non-safety-specialist framework for EU compliance work.

- **Article 9 (Risk management):** CI/CD integration enables continuous, documented quality monitoring throughout the development lifecycle — directly supports the requirement for ongoing risk management systems.
- **Article 10 (Data governance):** Test case datasets are versioned and auditable. Evaluation results are exportable.
- **Article 17 (Quality management):** pytest integration makes evaluations a formal part of the software development process — supports quality management system requirements.
- **Annex IV (Technical documentation):** Evaluation runs produce structured output (JSON/XML) suitable for technical documentation.

**Gaps:** No adversarial/red teaming beyond basic safety metrics. No audit trail for production inferences. No data residency controls in the OSS version.

For EU high-risk AI systems, DeepEval covers pre-production quality gates well. Pair with Langfuse for production audit trails and inspect_ai for safety conformity testing.

---

## Version Tracked

- **Current stable:** v1.x  
- **Last verified:** 2026-05-20  
- **Notable recent addition:** Confident AI cloud dashboard for test run history, regression tracking, and team collaboration (commercial, not required for OSS use)
