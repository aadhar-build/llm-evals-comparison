# Changelog

Tracks material changes to framework profiles as the eval ecosystem evolves.

## 2026-05-22 — v1.0.0 (Templates, cross-linking, launch-ready)

**Added**
- `templates/eval-framework-rfp.md` — weighted scoring template (8 criteria × 3 team-type weight sets), pilot protocol, dealbreaker checklist, decision output format
- `templates/pre-launch-eval-plan.md` — 8-section working document: feature scope, metrics, dataset, framework selection, baseline, thresholds, production monitoring, sign-off gate; Annex IV archival note
- `templates/README.md` — templates index

**Changed**
- `README.md` — quick-pick table expanded to 14 rows covering all content; EU AI Act and Templates sections updated (removed "coming soon" caveats); full content listing for all five modules
- Framework profiles — cross-links added to RAGAS, DeepEval, promptfoo, inspect_ai pointing to relevant comparison pages and decision guide use-case anchors

**Repo is now complete.** All 7 planned sessions delivered.

---

## 2026-05-22 — v0.5.0 (EU AI Act module)

**Added**
- `eu-ai-act/README.md` — enforcement timeline, Annex III categories, four articles that create eval obligations (Art. 9/10/13/15/17), audit trail requirements per framework
- `eu-ai-act/framework-mapping.md` — master mapping table (6 frameworks × 7 Act requirements); article-by-article breakdown with practical guidance; red-flag matrix of what each framework cannot provide
- `eu-ai-act/high-risk-checklist.md` — 5-phase pre-deployment checklist; bias testing procedures; inspect_ai run templates; Annex IV documentation package spec; recommended stacks for Tier 1/2/3 risk systems

**Key content shipped:**
- "No single framework is sufficient for Annex III compliance" — minimum stack is inspect_ai + Langfuse
- Full Article 40 (harmonised standards) analysis — inspect_ai is the only framework whose maintainer is involved in developing those standards
- Red-flag matrix: what each framework *cannot* provide for high-risk AI — the compliance gaps most teams don't know about
- Concrete Annex IV documentation package template — what files to archive, alongside which artifacts

---

## 2026-05-22 — v0.4.0 (Head-to-head comparisons)

**Added**
- `comparisons/ragas-vs-deepeval.md` — deep comparison: retrieval metrics, CI/CD integration, cost, when to use both together; includes "what RAGAS does that DeepEval doesn't" and vice versa
- `comparisons/promptfoo-vs-inspect-ai.md` — developer speed vs audit-grade output; EU AI Act decision point; when to use both
- `comparisons/full-matrix.md` — all 6 frameworks × 10 criteria; dimension deep-dives; recommended stacks; "what's missing from all of them"
- `comparisons/README.md` — index page

**Key content shipped:**
- "RAGAS vs DeepEval are not competitors — most RAG teams use both" structured clearly with concrete integration pattern
- EU AI Act decision point for promptfoo vs inspect_ai: `We ran AgentHarm via inspect_ai` carries regulatory weight that `We ran promptfoo` does not
- Full matrix includes "what no single framework covers" section (business metrics, multilingual depth, fine-grained attribution)

---

## 2026-05-22 — v0.3.0 (Decision guides)

**Added**
- `decision-guide/by-lifecycle-stage.md` — maps framework choice to dev → pre-production → production → compliance stages; recommended stacks for startup, enterprise, regulated EU
- `decision-guide/by-team-type.md` — team-context guides for 3-person RAG startup, enterprise ML platform, regulated EU company, research/safety team
- `decision-guide/by-use-case.md` — 9 use cases with primary recommendation, runner-up, and when to pick each
- `decision-guide/README.md` — 5-minute decision framework and guide index

**Key content shipped:**
- "Most teams need a stack, not a single framework" — lifecycle stage mapping makes this concrete
- Recommended stacks by context: minimum viable eval, full stack (EU market), regulated enterprise
- Quick diagnostic: 5 questions to determine what stage and what frameworks you need

---

## 2026-05-20 — v0.2.0 (Complete framework coverage)

**Added**
- Framework profiles: promptfoo, Langfuse, inspect_ai, OpenAI Evals
- promptfoo: OWASP LLM Top 10 red teaming coverage; OpenAI acquisition note (March 2026, remains MIT)
- Langfuse: EU Cloud data residency detail (cloud.langfuse.com, AWS eu-central-1, Germany); DPA link
- inspect_ai: UK AISI institutional authority section; inspect_evals library; no-semver pin-to-commit guidance
- OpenAI Evals: PM contribution opportunity section; YAML+JSONL two-file model

**All 6 framework profiles now complete.**

**In progress (upcoming sessions)**
- Decision guides: by-use-case, by-team-type, by-lifecycle-stage
- Head-to-head comparisons: RAGAS vs DeepEval, promptfoo vs inspect_ai
- EU AI Act module

---

## 2026-05-20 — v0.1.0 (Initial release)

**Added**
- Repository scaffold with full structure
- README with quick-pick decision table
- Framework profiles: RAGAS, DeepEval
- frameworks/README.md — how to read profiles
- CONTRIBUTING.md

---

## Framework Version Reference

*Last verified: 2026-05-20*

| Framework | Current stable | Notes |
|---|---|---|
| RAGAS | v0.4.x | v0.3 → v0.4 migration required; breaking changes in metric API |
| DeepEval | v1.x | Confident AI cloud integration added in recent releases |
| promptfoo | v0.121.x | Acquired by OpenAI March 2026; remains MIT-licensed OSS |
| Langfuse | v3.x | Self-host option available; EU Cloud at cloud.langfuse.com |
| inspect_ai | Active | No semver; pin to a commit for reproducible evals |
| OpenAI Evals | Active | YAML-first format; no code required for basic evals |
