---
title: Decision Guide
nav_order: 3
has_children: true
permalink: /decision-guide/
---

# Decision Guide

> Three structured guides for answering "which eval framework(s) should we use?" Skip what's not relevant to your situation.

---

## How to Choose a Guide

**If you have a specific problem to solve** → [By Use Case](./by-use-case.md)  
Examples: "I need to evaluate my RAG pipeline", "I want to red team my chatbot", "I need production monitoring"

**If you're unsure where to start** → [By Team Type](./by-team-type.md)  
Examples: "We're a 3-person startup", "We're an enterprise ML platform team", "We're subject to EU AI Act"

**If you're building an eval strategy, not just picking one tool** → [By Lifecycle Stage](./by-lifecycle-stage.md)  
The insight: prototype, pre-production, and production monitoring each need different frameworks. Most teams are missing at least one stage.

---

## The 5-Minute Decision

**Answer these three questions:**

**1. What are you evaluating?**
- RAG pipeline → RAGAS
- Conversational / agentic LLM → DeepEval
- Adversarial robustness → promptfoo
- Production traffic → Langfuse
- Safety for high-risk AI → inspect_ai

**2. What stage are you at?**
- Active development → RAGAS or DeepEval (Stage 1)
- Pre-launch → add promptfoo (Stage 2)
- Live in production → add Langfuse (Stage 3)
- Preparing for compliance audit → add inspect_ai (Stage 4)

**3. What's your regulatory context?**
- No regulatory pressure → promptfoo for pre-launch adversarial; skip inspect_ai for now
- EU-regulated / high-risk AI → inspect_ai is the most defensible choice for compliance evidence

---

## Guides

### [By Use Case](./by-use-case.md)
Covers: RAG quality, conversational AI, red teaming, agentic systems, production monitoring, EU AI Act compliance, benchmark research, CI/CD gates, multi-model comparison.

For each use case: primary recommendation, runner-up, and when to pick each.

---

### [By Team Type](./by-team-type.md)
Covers: 3-person RAG startup, enterprise ML platform team, regulated EU company, research/safety team.

For each team type: recommended stack, what to skip, and upgrade triggers.

---

### [By Lifecycle Stage](./by-lifecycle-stage.md)
Covers: Stage 0 (prototype), Stage 1 (development evals), Stage 2 (adversarial pre-production), Stage 3 (production observability), Stage 4 (compliance audit).

The key insight: you need different tools at different stages. Most teams only have Stage 1.

---

## Quick Reference Table

| Situation | Framework | Why |
|---|---|---|
| RAG retrieval accuracy | RAGAS | Purpose-built retrieval metrics |
| LLM CI test suite | DeepEval | pytest-native, threshold-based |
| Adversarial pre-launch | promptfoo | OWASP LLM Top 10, auto-generated attack cases |
| Production tracing | Langfuse | Live inference monitoring, EU Cloud available |
| EU AI Act evidence | inspect_ai | Audit-grade logs, AISI institutional backing |
| Public benchmark contribution | OpenAI Evals | YAML-only, no code required |
| Startup, start here | RAGAS + Langfuse | Maximum signal, minimal setup |
| Enterprise regulated, start here | inspect_ai + Langfuse EU | Compliance-first stack |

---

→ [Back to main README](../README.md) | [Framework profiles](../frameworks/)
