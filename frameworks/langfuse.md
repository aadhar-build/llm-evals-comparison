# Langfuse

> Production observability for LLM applications — traces every inference, scores outputs, and gives you the audit trail that pre-production testing tools can't provide.

**GitHub:** [langfuse/langfuse](https://github.com/langfuse/langfuse) · ~28k stars · MIT (self-host) / Commercial (cloud)  
**Docs:** [langfuse.com/docs](https://langfuse.com/docs)  
**Cloud:** EU Cloud at [cloud.langfuse.com](https://cloud.langfuse.com) · US Cloud at [us.cloud.langfuse.com](https://us.cloud.langfuse.com)  
**Company:** Langfuse GmbH, Berlin, Germany

---

## At a Glance

| Dimension | Rating |
|---|---|
| Setup effort | ● ● ○ ○ ○ |
| Coding required | ● ● ○ ○ ○ (SDK decorator pattern) |
| RAG evaluation | ● ● ● ○ ○ (scoring, not metrics) |
| Conversational / general LLM | ● ● ● ● ○ |
| Red teaming / adversarial | ○ ○ ○ ○ ○ |
| Production observability | ● ● ● ● ● |
| EU AI Act support | ● ● ● ● ○ |
| Cost at scale | Free (self-host) / Usage-based (cloud) |

---

## Best For

- **Production LLM monitoring** — tracing every inference, input, output, latency, and cost in a live application
- **Human review pipelines** — annotating traces with scores, building labeling queues for quality teams
- **Eval + observability in one system** — linking offline evaluation results back to production traces
- **Teams with EU data residency requirements** — EU Cloud is hosted in Germany; GDPR-compliant by design
- **Multi-step and agentic tracing** — traces nest arbitrarily; a single user request that spawns 5 LLM calls shows as a unified trace tree
- **Online evaluation** — automatically scoring production outputs using LLM-as-judge or rule-based scorers

## Not Great For

- **Pre-production red teaming** — Langfuse observes what happens in production; use promptfoo or inspect_ai to find vulnerabilities before deploy
- **Deep RAG metric analysis** — Langfuse can score RAG outputs, but it doesn't compute faithfulness or context precision natively; use RAGAS to generate scores and send them to Langfuse
- **One-off quality checks without a running application** — Langfuse requires an instrumented application generating traces; it's not a batch eval CLI
- **Teams that want zero cloud dependency** — self-hosting requires Docker Compose or Kubernetes (feasible, but non-trivial)

---

## How It Works

Langfuse works via an SDK decorator that wraps your LLM calls and sends traces to the Langfuse backend:

```python
from langfuse.decorators import observe, langfuse_context
from langfuse.openai import openai  # drop-in OpenAI client

@observe()
def my_rag_pipeline(question: str) -> str:
    # Every LLM call inside here is automatically traced
    context = retrieve_documents(question)
    response = openai.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": f"Use this context: {context}"},
            {"role": "user", "content": question}
        ]
    )
    return response.choices[0].message.content
```

Every call produces a trace with: input, output, latency, token usage, cost, model used — visible in the Langfuse dashboard.

**Scoring:** Attach scores to any trace programmatically or via the UI:
```python
langfuse_context.score_current_trace(
    name="faithfulness",
    value=0.87,
    comment="Grounded in retrieved context"
)
```

This is how RAGAS scores get linked back to production traces.

---

## Trace Hierarchy

Langfuse traces nest naturally for multi-step and agentic workflows:

```
Trace: user_question
  └── Span: retrieve_documents (latency: 120ms)
  └── Generation: gpt-4o call (tokens: 450, cost: $0.002)
      └── Score: faithfulness = 0.87
      └── Score: answer_relevancy = 0.91
```

For agent harnesses with subagents, each agent's work appears as a nested span. This is the foundation for the EU AI Act audit trail use case.

---

## Tradeoffs vs. Alternatives

**vs. LangSmith**
Both are LLM observability platforms. Langfuse is open source (self-host for free) and German-based (strong EU data residency story). LangSmith is commercial-only and US-based. Feature parity is high; the decision usually comes down to data residency requirements and vendor preference. Langfuse's self-hosting option is a strong differentiator for regulated industries.

**vs. DeepEval / RAGAS**
Complementary, not competing. DeepEval and RAGAS run offline evals before deployment. Langfuse traces what actually happens in production and can apply the same scoring logic to live traffic. Typical stack: RAGAS/DeepEval pre-deploy → Langfuse post-deploy.

**vs. Braintrust**
Braintrust has stronger step-efficiency metrics for agentic evals. Langfuse has a better self-hosting story and stronger EU compliance positioning. Both support datasets, experiments, and scoring.

---

## Integration Effort

**Time to first trace:** 20–30 minutes

```bash
pip install langfuse
```

```python
import os
os.environ["LANGFUSE_PUBLIC_KEY"] = "..."
os.environ["LANGFUSE_SECRET_KEY"] = "..."
# EU Cloud
os.environ["LANGFUSE_HOST"] = "https://cloud.langfuse.com"
```

**Works with:** OpenAI SDK (drop-in wrapper), LangChain, LlamaIndex, LiteLLM, Anthropic SDK, raw HTTP via API  
**Languages:** Python, TypeScript/JavaScript, other languages via REST API

---

## Cost at Scale

| Option | Cost |
|---|---|
| Self-hosted (Docker Compose) | Free — requires a server with ~2GB RAM |
| Self-hosted (Kubernetes, Helm) | Free — requires K8s cluster |
| Cloud (Hobby tier) | Free up to 50k observations/month |
| Cloud (Pro) | $29/month + usage beyond free tier |
| Cloud (Enterprise) | Custom pricing, SSO, audit logs, dedicated support |

Self-hosting is genuinely viable for most teams — Langfuse provides a maintained Docker Compose setup and Helm chart.

---

## EU AI Act Relevance

**Strong — Langfuse's most important compliance differentiator in the eval ecosystem.**

- **Article 9 (Risk management):** Production tracing provides continuous, automated risk monitoring throughout the operational lifecycle — not just at deploy time. Incident detection through score thresholds (e.g., alert when faithfulness drops below 0.7).
- **Article 10 (Data governance):** Traces create an auditable record of every inference, including inputs, outputs, and metadata. Data retention is configurable.
- **Article 13 (Transparency):** Trace data can support user-facing transparency requirements — you can demonstrate what information the AI used to generate a response.
- **Article 17 (Quality management):** Online scoring pipelines implement continuous quality monitoring as required by quality management system obligations.
- **Annex IV (Technical documentation):** Trace exports (JSONL, CSV) provide machine-readable evidence of system behavior over time.

**EU data residency:** The EU Cloud instance (cloud.langfuse.com) is hosted in Germany (AWS eu-central-1). Langfuse GmbH is a German company subject to GDPR. A Data Processing Agreement (DPA) is available at [langfuse.com/security/dpa](https://langfuse.com/security/dpa) — applicable to all subscription tiers including free.

**Self-hosting for maximum data control:** For Annex III high-risk AI systems where data sovereignty is non-negotiable, self-hosting ensures no data leaves your infrastructure.

---

## Version Tracked

- **Current stable:** v3.x  
- **Last verified:** 2026-05-20  
- **Self-host note:** Docker Compose setup is well-maintained. The `docker-compose.yml` in the repo is the recommended starting point. Helm chart for Kubernetes is available in the `langfuse/langfuse-k8s` repo.
