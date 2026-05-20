# Changelog

Tracks material changes to framework profiles as the eval ecosystem evolves.

## 2026-05-20 — v0.1.0 (Initial release)

**Added**
- Repository scaffold with full structure
- README with quick-pick decision table
- Framework profiles: RAGAS, DeepEval
- frameworks/README.md — how to read profiles
- CONTRIBUTING.md

**In progress (upcoming sessions)**
- Framework profiles: promptfoo, Langfuse, inspect_ai, OpenAI Evals
- Decision guides: by-use-case, by-team-type, by-lifecycle-stage
- Head-to-head comparisons: RAGAS vs DeepEval, promptfoo vs inspect_ai
- EU AI Act module

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
