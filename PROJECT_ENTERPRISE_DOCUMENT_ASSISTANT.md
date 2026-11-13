# Enterprise Document Intelligence Assistant — Learning Project

Build incrementally: (1) Python keyword search over synthetic policies, (2) FastAPI /search, (3) PDF extraction with page and heading metadata, (4) BM25 and embedding retrieval, (5) hybrid search and RRF, (6) grounded LLM answers with citations, (7) a labeled evaluation set, (8) LangGraph retrieval/review workflow, (9) permissions, tracing and Docker.

For each increment: define the problem in simple terms; explain prerequisites; run a tiny example; inspect expected output; identify one failure; compare an alternative; practice beginner and senior interview answers.

Start with the runnable mini-search code in [GenAI concept foundations](00-genai-concept-foundations.md). Then use the **Beginner-first concept and project bridge** appended to each of the nine original interview batches. Never use real customer data in experiments. Apply authorization checks before model context assembly. Blogs, Medium and LinkedIn tutorials are listed in the foundations guide alongside official docs; treat blog posts as teaching references rather than verified interview evidence.

## Reliability increment 1 — bounded agents and repeatable runs

After the basic retrieval/review workflow works, complete [Agent loops and repeatable behaviour](01-agent-loops-and-repeatability.md) and [Batch 010](12-reported-interview-experiences/batch-010-2026-10-30-agent-loops-repeatability.md).

Add these fields to graph or workflow state:

- attempt count and maximum tool-call budget;
- canonical fingerprints of prior tool calls;
- retrieved and cited source IDs;
- explicit stop reason;
- prompt, model and corpus versions;
- node route and redacted tool outcomes.

### Required lab

1. Mock an empty retrieval result.
2. Make the planner propose the same query twice.
3. Reject the duplicate and route to human review.
4. Run 10 fixed questions five times against a frozen corpus.
5. Compare route, retrieval, citation and structured-fact agreement.

**Expected result:** no run exceeds three tool calls; duplicate calls have a visible stop reason; answerable questions consistently cite the same authorized source and return the same structured facts.

## Reliability increment 2 — ranking, document updates and provider outages

Next complete [RAG ranking, freshness and service resilience](02-rag-ranking-freshness-resilience.md) and [Batch 011](12-reported-interview-experiences/batch-011-2026-11-06-rag-ranking-freshness-resilience.md).

Add:

- retrieve 20 candidates, deduplicate, rerank and keep 5;
- document ID, version, effective date and content hash;
- staging and active index versions with a promotion gate;
- model-call timeout, classified retry and circuit-breaker state;
- an authorization-filtered source-only degraded response.

### Required lab

1. Evaluate retrieval with and without reranking on 30 labeled questions.
2. Change five source facts and reject an intentionally incomplete staged index.
3. Promote the corrected index and prove the new facts are active.
4. Make a fake model fail five times and verify the circuit opens.
5. Confirm degraded responses contain source excerpts and never claim a generated answer.

**Expected result:** the required source ranks in the final top five; duplicate and token counts are recorded; bad index versions never serve traffic; repeated provider failures stop new calls; source-only responses remain tenant-safe.

Keep every readiness status **Untested** until the matching lab runs and the design can be explained.


## Reliability increment 3 — context quality and grounded generation

Next complete [Context quality, prompt grounding and lost-in-the-middle](03-context-quality-and-grounding.md) and [Batch 012](12-reported-interview-experiences/batch-012-2026-11-13-context-quality-grounding.md).

Add:

- traceable source IDs from retrieval through final prompt assembly;
- active document version filtering before generation;
- a configurable evidence-token budget with output headroom;
- duplicate and conflict detection;
- structured evidence blocks with source IDs;
- citation-support validation after generation.

### Required lab

1. Create 20 questions with known evidence, including multi-document, conflicting-version and unanswerable cases.
2. Compare final-k values 2, 5, 10 and 20.
3. Place required evidence first, middle and last.
4. Inject ten irrelevant chunks and an obsolete conflicting chunk.
5. Record required-evidence coverage, supported-answer accuracy, refusal, citation validity, tokens and p95 latency.

**Expected result:** obsolete evidence never reaches generation; every factual claim cites a supplied supporting source; unanswerable cases refuse; the chosen context budget is justified by measured quality, cost and latency.
