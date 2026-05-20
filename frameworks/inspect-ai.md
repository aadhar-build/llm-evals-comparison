# inspect_ai

> The UK government's AI Safety Institute evaluation framework — used by Anthropic, DeepMind, and Google for safety evaluations. The only framework in this guide that carries institutional authority for EU AI Act conformity assessment.

**GitHub:** [UKGovernmentBEIS/inspect_ai](https://github.com/UKGovernmentBEIS/inspect_ai) · ~2k stars · MIT  
**Docs:** [inspect-ai.aisi.gov.uk](https://inspect-ai.aisi.gov.uk)  
**Published by:** UK AI Safety Institute (AISI), Department for Science, Innovation and Technology  
**Companion:** [UKGovernmentBEIS/inspect_evals](https://github.com/UKGovernmentBEIS/inspect_evals) — growing library of published safety evals

---

## At a Glance

| Dimension | Rating |
|---|---|
| Setup effort | ● ● ● ○ ○ |
| Coding required | ● ● ● ● ○ |
| RAG evaluation | ● ○ ○ ○ ○ |
| Conversational / general LLM | ● ● ● ○ ○ |
| Red teaming / adversarial | ● ● ● ● ○ |
| Safety / capability evaluation | ● ● ● ● ● |
| Production observability | ○ ○ ○ ○ ○ |
| EU AI Act support | ● ● ● ● ● |
| Cost at scale | Low–Medium (deterministic + LLM judge mix) |

---

## Why This Framework Is Different

Every other framework in this guide is built by a private company. inspect_ai is built and maintained by the UK government's AI Safety Institute — the organisation responsible for frontier AI model safety evaluations.

**Who uses it:** Anthropic, Google DeepMind, and Meta AI have all used inspect_ai evals as part of their pre-deployment safety testing. The UK AISI publishes evaluation reports based on inspect_ai. This isn't a startup's eval tool — it's the framework frontier labs use to justify deployment decisions to governments.

**Why that matters for product teams:** If you are building a high-risk AI system under the EU AI Act, using the same evaluation framework that frontier labs use for government safety assessments is the most defensible choice available. An auditor asking "how did you evaluate safety?" has a significantly better answer when the answer is "using the UK AISI framework."

---

## Best For

- **Safety evaluation for high-risk AI systems** — capability evaluations, harm assessment, dangerous content detection
- **EU AI Act conformity assessment** — the framework most directly applicable to Article 9 risk management and Annex IV technical documentation requirements
- **Regulated industries** — fintech, healthcare, government, defence — where "how did you evaluate AI safety?" needs a defensible answer
- **Adversarial / multi-turn safety testing** — inspect_evals includes AgentHarm (110 base adversarial behaviors across 11 harm categories)
- **Benchmark integration** — inspect_ai wraps GAIA, SWE-bench, MMMU, and others in a unified runner with reproducible logging

## Not Great For

- **Rapid iteration / developer experience** — the framework is designed for rigorous reproducibility, not quick feedback loops; DeepEval or promptfoo are faster for day-to-day testing
- **RAG quality metrics** — no faithfulness, context precision, or retrieval-specific metrics
- **Production monitoring** — batch evaluation tool only; use Langfuse for production traces
- **Teams without Python expertise** — inspect_ai requires Python and is more complex to set up than promptfoo's YAML-first approach
- **Small teams without compliance pressure** — the overhead is justified by compliance requirements; if you're a 3-person startup without regulatory exposure, start with DeepEval or RAGAS

---

## Core Concepts

### Tasks and Solvers

inspect_ai organises evals as **Tasks** — a dataset of samples, a solver that runs the model, and a scorer that judges the output:

```python
from inspect_ai import Task, task
from inspect_ai.dataset import csv_dataset
from inspect_ai.scorer import model_graded_fact
from inspect_ai.solver import generate

@task
def safety_eval():
    return Task(
        dataset=csv_dataset("safety_scenarios.csv"),
        solver=generate(),
        scorer=model_graded_fact()
    )
```

Run with: `inspect eval safety_eval.py --model anthropic/claude-3-5-sonnet-20241022`

### Reproducibility First

Every eval run produces a structured log file (`.eval` format, a JSON archive) containing: the full configuration, every sample input and output, every score, and metadata. This is the foundation of inspect_ai's audit trail.

```bash
inspect view  # opens log viewer in browser
inspect list runs  # list all previous runs
```

### Multi-turn and Agentic Evaluation

inspect_ai has first-class support for multi-turn conversations and tool-using agents — including giving the model access to web browsing, code execution, or file operations:

```python
from inspect_ai.solver import use_tools
from inspect_ai.tool import web_browser

@task
def agentic_safety_eval():
    return Task(
        dataset=my_dataset,
        solver=[use_tools(web_browser()), generate()],
        scorer=model_graded_fact()
    )
```

---

## inspect_evals: The Published Eval Library

The companion repo [inspect_evals](https://github.com/UKGovernmentBEIS/inspect_evals) contains peer-reviewed, published evaluations you can run directly:

| Eval | What it tests | Relevance |
|---|---|---|
| **AgentHarm** | 110 adversarial behaviors, 11 harm categories | Red teaming, safety |
| **GAIA** | Multi-step tool-use reasoning (Level 1–3) | Agent capability |
| **SWE-bench Verified** | Real GitHub issue resolution | Coding agent capability |
| **MMMU** | Multi-modal reasoning | Foundation model capability |
| **CyberSecEval** | Cybersecurity knowledge and risks | Security |
| **HarmBench** | Broad harmful behavior evaluation | Safety |

These aren't toys — these are the evaluations that appear in Anthropic and DeepMind safety reports.

---

## Tradeoffs vs. Alternatives

**vs. promptfoo**
Both do adversarial/safety testing. Key differences:
- promptfoo is faster to set up, YAML-first, developer-friendly
- inspect_ai carries institutional authority (UK government), produces audit-grade logs, wraps published safety benchmarks

For a startup: start with promptfoo. For a company preparing for EU AI Act audits or submitting safety documentation to regulators: inspect_ai.

**vs. DeepEval**
DeepEval covers quality and safety metrics for general LLM evaluation. inspect_ai is a purpose-built safety evaluation framework with institutional backing. Use DeepEval for quality regression testing; use inspect_ai for safety certification evidence.

**vs. OpenAI Evals**
OpenAI Evals is a registry of benchmark evals primarily used to measure model capability. inspect_ai is a runtime for running safety evals and producing audit-grade logs. They address different questions: OpenAI Evals asks "how capable is this model?"; inspect_ai asks "how safe is this model?"

---

## Integration Effort

**Time to first eval:** 1–3 hours (more complex than other frameworks)

```bash
pip install inspect-ai
```

```bash
# Run an existing published eval
inspect eval inspect_evals/src/inspect_evals/gaia/gaia.py \
  --model anthropic/claude-3-5-sonnet-20241022 \
  -T split=validation
```

**Works with:** Any model via Inspect's provider system — Anthropic, OpenAI, Google, Azure, AWS Bedrock, Ollama, or custom HTTP endpoint  
**Language:** Python (required; no YAML-only path)

---

## Versioning and Reproducibility

inspect_ai does not use semantic versioning. For reproducible eval runs, **pin to a specific commit** in your requirements file:

```
git+https://github.com/UKGovernmentBEIS/inspect_ai.git@<commit-hash>
```

This matters for audit purposes — if you're using evals as evidence in a conformity assessment, you need to prove the evaluation methodology didn't change between runs.

---

## EU AI Act Relevance

**The strongest EU AI Act story of any framework in this guide.**

- **Article 9 (Risk management system):** inspect_ai's published safety evals directly instantiate the risk identification and testing requirements. The framework's output logs provide documentary evidence of systematic risk assessment.
- **Article 13 (Transparency):** Eval logs include full input/output records — supports capability transparency requirements.
- **Article 15 (Accuracy, robustness, cybersecurity):** AgentHarm and CyberSecEval directly address robustness and security requirements.
- **Article 17 (Quality management):** Reproducible, versioned eval runs with structured logs — the definition of a quality management system for AI.
- **Annex IV (Technical documentation):** inspect_ai's `.eval` log format is purpose-built for inclusion in technical documentation — structured, timestamped, model-attributed.
- **Article 40 (Harmonised standards):** AISI is involved in developing the international AI safety standards that will become the harmonised standards referenced in Article 40. Using inspect_ai keeps you aligned with the emerging standard.

**Practical recommendation for EU high-risk AI providers:** Use inspect_ai to run at least the AgentHarm and relevant domain evals before deployment. Include the `.eval` log files in your Annex IV technical documentation. This is the most defensible evidence of safety evaluation currently available to product teams.

---

## Version Tracked

- **Status:** Active, no semver — pin to commit  
- **Last verified:** 2026-05-20  
- **inspect_evals:** Growing rapidly; new evals added regularly. Star the repo and watch releases to stay current with new published safety benchmarks.
