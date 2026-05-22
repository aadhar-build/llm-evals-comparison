---
title: promptfoo vs inspect_ai
parent: Comparisons
nav_order: 2
---

# promptfoo vs inspect_ai

> Both do red teaming and adversarial testing. The difference is who the output is for and what it needs to prove.

---

## The One-Line Version

**promptfoo** finds vulnerabilities before your users do — fast, YAML-driven, developer-facing.  
**inspect_ai** proves a system was safely evaluated — rigorous, audit-grade, government-backed.

The question isn't which is better. It's which audience needs to be convinced. For a developer iteration loop: promptfoo. For a regulator or enterprise auditor: inspect_ai.

---

## At a Glance

| Dimension | promptfoo | inspect_ai |
|---|---|---|
| Maintained by | Community (acquired by OpenAI, March 2026) | UK AI Safety Institute (gov) |
| Primary audience | Developers, security teams | Safety researchers, compliance teams |
| Configuration | YAML-first, no Python required | Python (required) |
| Setup time | 15–30 minutes | 1–3 hours |
| Output format | HTML/JSON report, web UI | `.eval` structured log (audit-grade) |
| Adversarial coverage | ● ● ● ● ● | ● ● ● ● ○ |
| Audit trail quality | ● ● ○ ○ ○ | ● ● ● ● ● |
| EU AI Act evidence value | ● ● ● ○ ○ | ● ● ● ● ● |
| Multi-turn agentic eval | ● ○ ○ ○ ○ | ● ● ● ● ○ |
| Published safety benchmarks | ✗ | ✓ (inspect_evals) |
| CI/CD integration | ✓ (native) | Manual |
| Self-host capable | ✓ | ✓ |
| Institutional authority | None | UK AISI |
| GitHub stars | ~21k | ~2k |
| License | MIT | MIT |

---

## What promptfoo Does Better

### Speed and breadth of adversarial coverage

promptfoo's red team engine is the most comprehensive automated attack generation available in open source. From a single YAML config, it generates hundreds of adversarial test cases across:

- **Prompt injection** (direct and indirect via RAG context)
- **Jailbreaks** (role-play, encoding, DAN variants)
- **PII extraction** (direct ask, indirect inference)
- **Harmful content** (12 categories including violence, CSAM detection, financial crime)
- **OWASP LLM Top 10** — full coverage with plugin-per-category
- **Indirect injection** — adversarial content embedded in retrieved documents

```yaml
redteam:
  purpose: "Customer-facing loan application assistant"
  plugins:
    - owasp:llm
    - pii:direct
    - pii:indirect
    - harmful:financial-crime
    - harmful:hate
  strategies:
    - jailbreak
    - jailbreak:tree
    - prompt-injection
```

This generates 200–500 test cases automatically. A team with no security background can run this in 30 minutes and get a structured vulnerability report.

inspect_ai's `AgentHarm` benchmark covers 110 adversarial behaviors across 11 harm categories — excellent coverage for published benchmarks, but it's a fixed set. promptfoo's generation is open-ended and customisable per application context.

### Developer experience

promptfoo is designed to be used continuously during development, not just at release gates:

- No Python required for basic use — YAML config and `npx promptfoo redteam run`
- Web UI comparison table — immediate visual output
- CI/CD integration — runs as a pipeline step with pass/fail exit codes
- Multi-model comparison — run the same attack suite against GPT-4o vs Claude simultaneously

The iteration loop is fast enough to run on every PR that touches prompt logic.

### Cost

promptfoo's attack generation uses an LLM once (to create the test cases), then re-runs the same cases deterministically. A full red team campaign costs $1–10 depending on scope and judge model. Re-running against a new model version is essentially free.

---

## What inspect_ai Does Better

### Audit-grade output

This is the core difference. promptfoo produces a developer-readable report. inspect_ai produces an `.eval` log — a structured JSON archive containing:

- Full configuration (model, task, parameters)
- Every sample: input, model output, scorer output, score
- Timestamps, model attribution
- Reproducible: run the same `.eval` file against the same commit six months later

```bash
inspect view  # opens the log in a browser-based viewer
inspect list runs  # all previous eval runs
```

This log format is purpose-built for inclusion in regulatory documentation. An auditor can open an `.eval` file and reconstruct exactly what was tested, when, with what model, and what the result was. promptfoo's JSON output is readable but not structured to this standard.

### Institutional authority

inspect_ai is maintained by the UK AI Safety Institute — the government body responsible for frontier AI safety evaluations. Anthropic, Google DeepMind, and Meta AI have used inspect_ai evals as part of their pre-deployment safety testing. The UK AISI publishes evaluation reports built on inspect_ai.

This matters for a specific, important reason: **EU AI Act Article 40 references harmonised standards**. The AISI is involved in developing the international AI safety standards that will eventually be those harmonised standards. Using inspect_ai positions your evaluation methodology to be aligned with whatever those standards become. Using promptfoo does not.

### inspect_evals: peer-reviewed published benchmarks

The companion repo `inspect_evals` contains evaluations that have been reviewed and published by the UK AISI:

| Eval | What it tests |
|---|---|
| **AgentHarm** | 110 adversarial behaviors, 11 harm categories |
| **HarmBench** | Broad harmful behavior across multiple attack vectors |
| **CyberSecEval** | Cybersecurity knowledge, vulnerability generation risks |
| **GAIA** | Multi-step tool-use reasoning (Level 1–3) |
| **SWE-bench Verified** | Real GitHub issue resolution by agents |

Running AgentHarm against your model and including the `.eval` log in your technical documentation is the most defensible safety evidence currently available to product teams. The benchmarks appear in Anthropic and DeepMind pre-deployment reports.

### Multi-turn agentic evaluation

inspect_ai gives the model access to real tools — web browser, bash, file operations — and evaluates whether it uses them correctly across multi-turn scenarios. This is what you need for evaluating AI agents that take real-world actions.

```python
@task
def agentic_safety_eval():
    return Task(
        dataset=csv_dataset("safety_scenarios.csv"),
        solver=[use_tools(web_browser(), bash()), generate()],
        scorer=model_graded_fact()
    )
```

promptfoo's agentic evaluation is limited. Multi-turn adversarial scenarios are possible but not first-class.

---

## The EU AI Act Decision Point

This is where the two frameworks diverge most sharply in practice.

| Context | Recommendation |
|---|---|
| Startup, pre-launch, no regulatory pressure | promptfoo — fast, comprehensive, no Python required |
| Growth-stage company with enterprise customers | promptfoo in dev + inspect_ai before major releases |
| EU-regulated industry (fintech, health, gov) | inspect_ai primary — audit logs in Annex IV docs |
| Preparing EU AI Act conformity assessment | inspect_ai mandatory — Article 9 risk management evidence |
| Government or defence contractor | inspect_ai — institutional authority required |
| Publishing safety research | inspect_ai — comparable to frontier lab reports |

The EU AI Act's Article 9 requires a documented risk management system. "We ran promptfoo" is evidence of adversarial testing. "We ran AgentHarm via inspect_ai and here are the `.eval` logs" is evidence of systematic safety evaluation using the same methodology as Anthropic and DeepMind. For a regulator, those are not equivalent.

---

## Versioning

**promptfoo:** Semantic versioning. Current stable: v0.121.x. Upgrade normally.  
**inspect_ai:** No semver. Pin to a specific commit for reproducible evals.

```
# requirements.txt — for audit reproducibility
git+https://github.com/UKGovernmentBEIS/inspect_ai.git@<commit-hash>
```

This matters: if you're including inspect_ai eval results in Annex IV technical documentation, you need to prove the methodology didn't change between runs. The commit hash is your proof.

---

## Can You Use Both?

Yes, and it's the recommended pattern for regulated teams:

```
promptfoo: continuous red teaming during development
           → runs on every PR, catches new vulnerabilities fast
           → developer-facing report, no regulatory weight

inspect_ai: structured safety evaluation at release gates
            → runs AgentHarm + domain-specific evals before deploy
            → .eval logs included in Annex IV documentation
```

This gives you fast feedback (promptfoo) and audit-grade evidence (inspect_ai) without needing to choose.

---

## The Verdict

| If your question is… | Answer |
|---|---|
| "Find vulnerabilities before launch, fast" | promptfoo |
| "Prove this system is safe to a regulator" | inspect_ai |
| "Run OWASP LLM Top 10 coverage" | promptfoo |
| "Run AgentHarm (110 published behaviors)" | inspect_ai |
| "I need CI/CD adversarial gate, no Python" | promptfoo |
| "We're preparing an EU AI Act conformity assessment" | inspect_ai |
| "3-person startup, first red team ever" | promptfoo |
| "Enterprise selling into regulated EU markets" | Both |

---

→ [promptfoo profile](../frameworks/promptfoo.md) · [inspect_ai profile](../frameworks/inspect-ai.md) · [Full matrix](./full-matrix.md) · [Back to comparisons](./)
