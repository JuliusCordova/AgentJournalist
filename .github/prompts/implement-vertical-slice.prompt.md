---
description: Implement one SDD-traceable AgentJournalist vertical slice with tests
---
Act as a senior implementation engineer following `SPEC.md`, `docs/AGENT-CONTRACTS.md` and `docs/TRAZABILIDAD-TALLER.md`.

Implement the smallest end-to-end capability requested in this chat, focusing first on **TALLER**:
1. Article input with strict size limits.
2. Extract atomic verifiable claims; exclude opinions.
3. Retrieve authorized local document chunks with genuine page/offset provenance.
4. Verify individual claims and abstain when missing/contradictory; do not fabricate citations.
5. Return Pydantic contract to FastAPI; React can display per-claim statuses and source links.

Before coding: list impacted FR/CA IDs, planned files, fixtures and acceptance tests. Use simple interfaces for ADK/Vertex integration; offline tests must work with stubbed model adapter. Never invent results or call live Google APIs without configuration.

After coding: show evidence of tests and unresolved gaps. Do not mark GraphRAG, probabilities, Azure or deployment 'done' merely because interfaces exist.
