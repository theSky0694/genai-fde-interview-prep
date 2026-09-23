# Interview Evidence & Coverage Dashboard

Last updated: 22 September 2026

## Current snapshot

**Reported experiences ingested:** 3  
**Interview-preparation posts ingested:** 1  
**Raw prompts/questions:** ~72  
**Normalized themes:** 32  
**Current roadmap stage:** 1A — Python Foundations

### What the evidence changed

The original roadmap direction was correct, but these interviews show that several topics need **earlier and deeper emphasis**:

1. **Retrieval engineering is now P0.** Do not learn RAG as a simple “embed → vector DB → LLM” pipeline. Prepare hybrid search, BM25, RRF, reranking, metadata, context expansion, multi-hop/distributed evidence and evaluation.
2. **Python Stage 1B is P0.** Async/FastAPI/database pooling appeared directly and is foundational for production GenAI.
3. **Agents must be explainable architecturally.** LangGraph nodes/state/routing, failure diagnosis, memory and design justification matter more than framework syntax.
4. **Production/system design starts earlier.** Cost, observability, security, inference serving and source routing should be threaded into projects rather than deferred to Stage 7.
5. **Add a light DSA track now.** One reported interview included Kadane; coding cannot wait until the final sprint.
6. **Add targeted ML fundamentals.** Logistic regression, SVM and time-series appeared, but evidence is currently too thin to divert major time from GenAI.
7. **AWS-specific material remains secondary unless more target JDs/reports reinforce it.** Learn AgentCore/Bedrock concepts after portable architecture fundamentals.

## Evidence x readiness

| Topic | Reports | Readiness | Practical evidence | Priority |
|---|---:|---|---|---|
| RAG/retrieval architecture | 3/3 | Untested | None yet | P0 |
| Hybrid/BM25/RRF/reranking | 2/3 | Untested | None yet | P0 |
| Chunking/long docs | 2/3 | Untested | None yet | P0 |
| LangGraph/agents | 3/3 | Untested | Existing professional context, not repo-tested | P0 |
| Python backend/FastAPI/async | 1/3 | Learning path not reached | None yet | P0 |
| GenAI production/system design | 2/3 | Untested | Professional architecture experience to map | P0 |
| Embeddings/vector concepts | 2/3 | Untested | None yet | P1 |
| MCP/A2A | 2/3 / 1/3 | Untested | Prior exposure, not tested here | P1/P2 |
| Graph/Neo4j | 2/3 | Untested | None yet | P1 |
| Inference/quantization | 1/3 | Untested | None yet | P1 |
| SQLAlchemy/pooling | 1/3 | Not started | None yet | P1 |
| DSA | 1/3 | Untested | None yet | P1 |
| Traditional ML | 1/3 | Untested | None yet | P2 |

## Where we stand

At this point **coverage cannot honestly be expressed as a percentage** because we have not tested you against these questions yet. “Untested” is intentionally different from “Red.”

The new RAG follow-up post reinforces the existing P0 emphasis on retrieval design, evaluation and debugging, but does not count as an independent interview experience.

The next useful measurement is a baseline diagnostic using a representative subset of these questions. After that, this dashboard can report Green / Amber / Red / Untested counts and drive weekly emphasis.

## Reweighting policy

After each batch:
1. normalize/deduplicate;
2. count independent experiences and company spread where known;
3. map to roadmap;
4. test current readiness where practical;
5. update priorities;
6. modify roadmap only when evidence or prerequisite structure warrants it.

Anecdotal reports guide preparation; they are not treated as a statistically representative hiring-market dataset.
