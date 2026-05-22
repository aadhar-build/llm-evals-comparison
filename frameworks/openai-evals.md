---
title: OpenAI Evals
parent: Framework Profiles
nav_order: 6
---

# OpenAI Evals

> A YAML-first benchmark registry where you contribute domain-specific evaluations that become part of a shared, publicly reusable test suite — no code required for most contributions.

**GitHub:** [openai/evals](https://github.com/openai/evals) · ~19k stars · MIT  
**Docs:** [github.com/openai/evals/blob/main/docs/build-eval.md](https://github.com/openai/evals/blob/main/docs/build-eval.md)

---

## At a Glance

| Dimension | Rating |
|---|---|
| Setup effort | ● ● ○ ○ ○ |
| Coding required | ○ ○ ○ ○ ○ (YAML + JSONL, no code for basic evals) |
| RAG evaluation | ● ○ ○ ○ ○ |
| Conversational / general LLM | ● ● ● ○ ○ |
| Red teaming / adversarial | ● ● ○ ○ ○ |
| Production observability | ○ ○ ○ ○ ○ |
| EU AI Act support | ● ● ○ ○ ○ |
| Benchmark contribution | ● ● ● ● ● |
| Cost at scale | Low (model API cost only) |

---

## What Makes This Different

OpenAI Evals is less a tool for evaluating *your* application and more a registry for contributing *reusable* evaluations that anyone can run against any model. The value proposition is:

1. **Your eval becomes publicly available** — useful for the community, visible to OpenAI, and associated with your name
2. **No code required** — a JSONL dataset + a YAML config is sufficient for most contributions
3. **Model-graded evals included** — you can specify that a judge model evaluates responses, not just string matching

This makes OpenAI Evals uniquely well-suited for product managers and domain experts who want to contribute evaluations from their area of expertise without writing Python.

---

## Best For

- **Contributing domain-specific benchmark evals** — if you have expertise in EU AI Act compliance, RAG quality patterns, or agentic behavior, you can codify that knowledge as a reusable eval
- **Testing model knowledge of specific domains** — regulatory frameworks, industry standards, technical accuracy in niche areas
- **Model-graded quality evaluations** — using a judge model to evaluate factual accuracy or reasoning quality
- **Teams that want to contribute to the public eval ecosystem** — each merged PR to openai/evals is a publicly visible contribution with your GitHub handle attached
- **Simple comparison testing** — does model A or model B handle this class of prompt better?

## Not Great For

- **Application-specific quality regression** — OpenAI Evals is a registry, not a test framework. DeepEval integrates better into your CI/CD pipeline for ongoing quality monitoring
- **Production observability** — no tracing or real-time capabilities
- **RAG pipeline evaluation** — no retrieval-specific metrics; RAGAS is the right tool
- **Deep adversarial testing** — promptfoo and inspect_ai have more sophisticated red teaming capabilities

---

## How It Works

### The Two-File Model

A complete eval is two files:

**1. Dataset (JSONL):** One sample per line
```jsonl
{"input": [{"role": "user", "content": "Is a customer support chatbot for loan applications considered high-risk under the EU AI Act?"}], "ideal": "Yes"}
{"input": [{"role": "user", "content": "Does Article 9 of the EU AI Act require ongoing risk monitoring?"}], "ideal": "Yes"}
```

**2. Config (YAML):**
```yaml
eu-ai-act-knowledge:
  id: eu-ai-act-knowledge
  description: Tests model knowledge of EU AI Act requirements for AI product teams
  metrics:
    - accuracy

eu-ai-act-knowledge/test:
  class: evals.elsuite.basic.match:Match
  args:
    samples_jsonl: eu-ai-act-knowledge/test.jsonl
```

Run with: `oaieval gpt-4o eu-ai-act-knowledge`

That's it. No Python required.

### Model-Graded Evals

For more nuanced quality evaluation, you can use a judge model:

```yaml
my-eval/test:
  class: evals.elsuite.modelgraded.classify:ModelGradedClassify
  args:
    samples_jsonl: my-eval/test.jsonl
    eval_type: cot_classify
    modelgraded_spec: fact
```

The judge model evaluates each response against the expected answer and produces a score. This is more expensive but handles open-ended responses where string matching fails.

---

## Tradeoffs vs. Alternatives

**vs. DeepEval**
Pick OpenAI Evals when you want to contribute to a public registry or need a simple YAML-based eval without Python. Pick DeepEval when you need pytest integration, CI/CD pipelines, or the full metric library (faithfulness, hallucination, etc.).

**vs. promptfoo**
Both support YAML-first configuration. promptfoo is better for pre-production testing of your own application with assertion-based pass/fail. OpenAI Evals is better for contributing standardised benchmarks to the public ecosystem.

**vs. inspect_ai**
OpenAI Evals asks "how does this model score on this benchmark?" inspect_ai asks "is this model safe to deploy?" They operate at different levels of rigor and institutional authority.

---

## Integration Effort

**Time to first eval:** 20–45 minutes to write a dataset; ~5 minutes to run

```bash
pip install evals
```

```bash
oaieval gpt-4o my-eval
```

**Works with:** OpenAI models natively; other models via completion API compatibility  
**Language requirements:** Python for setup and running; JSONL + YAML for eval authoring (no Python needed for the eval itself)

---

## Cost at Scale

Cost is essentially the model API cost — no additional eval infrastructure fees.

| Volume | Estimated cost (GPT-4o) |
|---|---|
| 100 samples, match eval | ~$0.01–$0.05 |
| 100 samples, model-graded | ~$0.50–$2 |
| 1,000 samples, match eval | ~$0.10–$0.50 |

Model-graded evals cost more because the judge model processes both the original response and the grading prompt.

---

## PM-Specific Opportunity: Contributing Domain Evals

For a product manager with domain expertise, OpenAI Evals is the lowest-friction path to a meaningful open-source contribution. The contribution that lands well:

1. **Pick a domain you know** — EU AI Act requirements, RAG system design, product prioritisation, agentic workflow safety
2. **Write 50–100 samples** — questions with clear correct answers in your domain
3. **Submit a PR** — the `build-eval.md` guide walks through exactly what's needed

A merged eval in `openai/evals` is visible on your GitHub profile, directly demonstrates domain expertise, and is cited by anyone who runs that eval against a model.

**Example domains with open gaps (as of 2026-05-20):**
- EU AI Act Articles 9–17 practical application
- Multi-agent coordination patterns (correct vs. incorrect architectures)
- RAG system design decisions (when to use reranking, when not to)
- AI product metrics definition accuracy

---

## EU AI Act Relevance

**Moderate.** Primarily indirect.

- **Annex IV (Technical documentation):** A published eval in the registry that tests AI Act compliance knowledge is citable evidence that the development team evaluated model knowledge of applicable regulations.
- **Article 17 (Quality management):** Benchmark evals run as part of pre-deployment checks contribute to documented quality management processes.

**Gap:** No audit trail for production inferences, no adversarial safety testing, no data residency controls. For direct EU AI Act compliance work, inspect_ai or Langfuse provide more substantive evidence.

---

## Version Tracked

- **Status:** Active  
- **Last verified:** 2026-05-20  
- **Context note:** OpenAI acquired promptfoo in March 2026, consolidating some of their eval tooling investments. The openai/evals registry remains active and accepting community contributions.
