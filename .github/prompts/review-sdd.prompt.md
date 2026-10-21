---
description: Review a PR against AgentJournalist SDD, traceability, security, and evaluation
---
Review changed files against `SPEC.md`, `docs/AGENT-CONTRACTS.md`, and `docs/TRAZABILIDAD-TALLER.md`.
Report findings in severity order with file and line, requirement/criterion ID, reproducible failure and proposed fix.
Specifically test:
- Unsupported/fabricated citations or document pages.
- Using a source's absence as proof of falsehood; treating press releases as independent confirmation.
- Fake numeric falsity probabilities or conflating IRD with truth.
- Permissions/gates bypass, especially TALLER SYSTEM G2 vs human approvals.
- Cross-session data leakage and stale artifact versions.
- SSRF, external prompt injections, secrets and unbounded requests.
- GraphRAG unsupported edges and missing RAG baseline comparison.
- DOCX export changing factual content or evading final human review.
- CI/eval/report claims inconsistent with actual executed tests.
State explicitly if code is not yet deployed or validated.
