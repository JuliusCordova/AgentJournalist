# GitHub Copilot implementation workflow

## Context loading
1. Read `SPEC.md`, `docs/AGENT-CONTRACTS.md`, architecture and traceability.
2. Read relevant pattern packs in JuliusCordova/DevPattern. The repo owns business requirements; DevPattern provides reusable architecture guidance.
3. Confirm whether any attached v0.6 patch/ZIP has been integrated; do **not** treat unmerged local files as authoritative by default.

## Suggested Copilot entry points
- Chat prompt: `Use .github/prompts/implement-vertical-slice.prompt.md and implement FR-T07..T12 with offline tests.`
- Review: `Use .github/prompts/review-sdd.prompt.md to review the diff against CA/HA criteria.`
- Agent: `sdd-guardian` for specification-focused PR review, where Copilot custom agents are supported.

## Backlog by milestone
1. Contracts, fixtures, source/chunk persistence, ingestion safety, Pydantic validations.
2. Claim extraction, evidence linking, deterministic verdict rules, abstention.
3. TALLER dashboard, coverage/IRD-1 and Word export with review gate.
4. GraphRAG per-case, evidence-backed multi-hop comparison vs baseline.
5. SIX ADK specialist roles, REDACCION workflow and controls.
6. Cloud Run deployment, smoke tests, evals, sessions concurrency and budgets.

## Definition of Done per pull request
- FR/CA/HA IDs listed in PR summary.
- Test cases show normal, edge and adversarial paths.
- Traceable citations; no invented verified outcomes.
- Gates and ownership checked server side; failure and abstention handled.
- No secrets, sensitive fixtures or real-world accusation created as synthetic data.
- State which operations are simulated, implemented or deployed.
