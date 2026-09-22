# Session Log

## 22 September 2026 — Interview HQ setup + first reported-question ingestion

### Completed
- Created centralized Interview HQ repository structure.
- Established Sep 2026 → Mar 2027 GenAI/FDE roadmap.
- Added readiness tracker and failed-question system.
- Added evidence-driven reported-interview workflow.
- Ingested 3 reported interview experiences (~65 raw prompts, 32 normalized themes).
- Prepared initial answer/reference notes across RAG, retrieval, LangGraph, graph retrieval, cloud/inference/system design, Python/FastAPI/database, agents/memory and DSA.
- Reweighted roadmap using the first evidence batch.

### Evidence-driven emphasis
**P0:** Python backend/FastAPI/async; RAG/retrieval architecture; hybrid search/BM25/RRF/reranking; chunking/large-document retrieval; LangGraph/agent architecture; GenAI production/system design.

**P1:** embeddings/vector concepts; MCP; graph/Neo4j; inference/quantization; observability; SQLAlchemy/pooling; memory; DSA.

**P2 currently:** A2A; AWS-specific implementation; traditional ML fundamentals. Promote if further interview evidence supports it.

### Roadmap changes
- Added parallel DSA lane starting Stage 1.
- Expanded Stage 1B with SQLAlchemy pooling.
- Deepened Stage 3 retrieval engineering.
- Added graph retrieval.
- Threaded cost/security/observability/evaluation earlier.
- Added memory engineering to Stage 5.
- Added A2A comparison to Stage 6.
- Added inference engineering module.
- Added compact ML-fundamentals lane.
- Kept AWS-specific learning secondary to portable architecture fundamentals.

### Current position
Stage 1A — Python Foundations.

Reported-question readiness is currently **Untested**, not Red. No honest readiness percentage is available until a baseline diagnostic is performed.

### Next session
Run a representative ~15-question baseline diagnostic drawn from the ingested interview questions. Ask one question at a time without teaching during the test. Record each result as Green / Amber / Red, then update coverage and failed-question files. After baseline, begin Stage 1A Python while maintaining the parallel interview-question/DSA lanes.

### Continuity rule
GitHub is the canonical state. At the end of meaningful sessions, consolidate decisions, progress, readiness evidence, failed questions and next actions here rather than relying on chat history.
