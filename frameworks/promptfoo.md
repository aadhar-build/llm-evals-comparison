---
title: promptfoo
parent: Framework Profiles
nav_order: 3
---

# promptfoo

> The most capable open-source tool for red teaming LLMs and running systematic adversarial tests — finds failure modes before your users do.

**GitHub:** [promptfoo/promptfoo](https://github.com/promptfoo/promptfoo) · ~21k stars · MIT  
**Docs:** [promptfoo.dev](https://promptfoo.dev)  
**Note:** Acquired by OpenAI in March 2026. Remains MIT-licensed open source.

---

## At a Glance

| Dimension | Rating |
|---|---|
| Setup effort | ● ● ○ ○ ○ |
| Coding required | ● ○ ○ ○ ○ (YAML-first) |
| RAG evaluation | ● ● ○ ○ ○ |
| Conversational / general LLM | ● ● ● ○ ○ |
| Red teaming / adversarial | ● ● ● ● ● |
| Production observability | ○ ○ ○ ○ ○ |
| EU AI Act support | ● ● ● ○ ○ |
| Cost at scale | Low–Medium (config-driven, judge model optional) |

---

## Best For

- **Red teaming LLMs** — systematic adversarial testing across hundreds of attack categories: prompt injection, jailbreaks, PII extraction, OWASP LLM Top 10
- **Prompt regression testing** — comparing outputs across model versions, prompt variants, or configuration changes before deploying
- **No-code eval setup** — YAML configuration means non-engineers can write and run evals without Python
- **CI/CD prompt quality gates** — assertion-based pass/fail pipeline integration
- **Multi-model comparison** — run the same prompt suite against GPT-4o, Claude, Gemini simultaneously and compare outputs side-by-side

## Not Great For

- **RAG-specific metric depth** — no faithfulness, context precision, or retrieval-specific metrics; use RAGAS for RAG quality
- **Production monitoring** — no trace collection or real-time observability; it's a pre-production testing tool
- **Teams that need Python-native testing** — DeepEval's pytest integration is more natural for Python-heavy teams
- **Long-horizon agentic evaluation** — tool call quality and multi-step agent behavior aren't first-class concerns

---

## How It Works

promptfoo is configuration-first. A basic eval is a YAML file:

```yaml
# promptfooconfig.yaml
prompts:
  - "Summarize this document: {{document}}"

providers:
  - openai:gpt-4o
  - anthropic:claude-3-5-sonnet-20241022

tests:
  - vars:
      document: "The Eiffel Tower was built in 1889..."
    assert:
      - type: contains
        value: "1889"
      - type: llm-rubric
        value: "Summary is accurate and under 100 words"
      - type: not-contains
        value: "I cannot"
```

Run with: `npx promptfoo eval`

The output is a web UI comparison table showing each model's response against every assertion.

---

## Red Teaming

promptfoo's red teaming engine is its primary differentiator. It generates adversarial test cases automatically across:

| Attack Category | What it tests |
|---|---|
| Prompt injection | Does the model follow injected instructions in user input? |
| Jailbreaks | Does the model comply with policy-violating requests? |
| PII extraction | Does the model leak personal information it was given? |
| Harmful content | Does the model generate dangerous or illegal content? |
| Indirect injection | Does RAG context contain adversarial instructions the model follows? |
| OWASP LLM Top 10 | Full coverage of the OWASP LLM security taxonomy |

```yaml
redteam:
  purpose: "Customer service assistant for a bank"
  plugins:
    - owasp:llm
    - harmful:violence
    - pii:direct
  strategies:
    - jailbreak
    - prompt-injection
```

This generates hundreds of adversarial test cases automatically and scores pass/fail against each.

---

## Tradeoffs vs. Alternatives

**vs. DeepEval**
Pick promptfoo when your primary concern is adversarial robustness and safety. Pick DeepEval when you want pytest-style quality regression tests. DeepEval has basic safety metrics; promptfoo has a dedicated red teaming engine an order of magnitude more comprehensive.

**vs. inspect_ai**
Both do adversarial/safety testing. The key difference:
- **promptfoo** is developer-facing, YAML-driven, CI/CD-first, works with any LLM
- **inspect_ai** is government-grade, carries UK AISI institutional authority, and maps directly to EU AI Act conformity assessment requirements

For a startup or mid-size product team: promptfoo. For a regulated EU company preparing for AI Act audits: inspect_ai (or both, targeting different audiences).

**vs. RAGAS**
Different jobs entirely. Use RAGAS for retrieval quality in RAG systems. Use promptfoo for adversarial robustness testing of any LLM-powered feature.

---

## Integration Effort

**Time to first eval:** 15–30 minutes

```bash
npm install -g promptfoo
promptfoo init
promptfoo eval
```

**Works with:** Any LLM provider via provider config — OpenAI, Anthropic, Google, Azure, Bedrock, Ollama, or any HTTP endpoint  
**Language:** YAML config + optional JavaScript/Python for custom providers and assertions  
**No Python required** for basic use

---

## Cost at Scale

| Mode | Cost |
|---|---|
| Assertion-only evals (contains, regex, JSON schema) | Free — no LLM calls for assertions |
| LLM-rubric assertions (llm-rubric type) | ~$0.001–$0.01 per assertion depending on judge model |
| Red team generation | ~$1–$10 per red team run depending on scope |

promptfoo is significantly cheaper than DeepEval or RAGAS at scale because many assertions are deterministic (no LLM judge needed). The red team generation step uses an LLM, but it's a one-time cost per campaign.

---

## EU AI Act Relevance

**Good — particularly strong for Article 9 risk management.**

- **Article 9 (Risk management system):** Red teaming directly maps to the Act's requirement to identify and test for foreseeable risks. promptfoo's OWASP LLM Top 10 coverage gives structured, documentable evidence of adversarial risk testing.
- **Article 13 (Transparency):** The YAML configuration creates a human-readable, auditable record of what was tested and what the acceptance criteria were.
- **Annex IV (Technical documentation):** promptfoo's HTML/JSON output reports are structured enough to include in technical documentation.

**Key limitation for EU compliance:** promptfoo provides no audit trail for production inferences and has no EU data residency controls. It's a pre-production testing tool — pair with Langfuse for production audit trails and inspect_ai for formal conformity assessment evidence.

**OWASP mapping contribution:** The community is actively mapping promptfoo plugins to OWASP LLM Top 10 categories (see open issue #6900). Once complete, this will make the EU AI Act → OWASP → promptfoo plugin chain fully traceable.

---

## Version Tracked

- **Current stable:** v0.121.x  
- **Last verified:** 2026-05-20  
- **Acquisition note:** OpenAI acquired promptfoo in March 2026. The repo remains MIT-licensed and community-maintained. Watch for changes to the commercial offering post-acquisition, but the core CLI is unaffected.

---

→ [promptfoo vs inspect_ai](../comparisons/promptfoo-vs-inspect-ai.md) · [Full matrix](../comparisons/full-matrix.md) · [By Use Case: Red teaming](../decision-guide/by-use-case.md#red-teaming--adversarial-testing) · [Back to Framework Profiles](./)
