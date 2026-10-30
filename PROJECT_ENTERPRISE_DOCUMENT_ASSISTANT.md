# Enterprise Document Intelligence Assistant — Learning Project

Build incrementally: (1) Python keyword search over synthetic policies, (2) FastAPI /search, (3) PDF extraction with page and heading metadata, (4) BM25 and embedding retrieval, (5) hybrid search and RRF, (6) grounded LLM answers with citations, (7) a labeled evaluation set, (8) LangGraph retrieval/review workflow, (9) permissions, tracing and Docker.

For each increment: define the problem in simple terms; explain prerequisites; run a tiny example; inspect expected output; identify one failure; compare an alternative; practice beginner and senior interview answers.

Start with the runnable mini-search code in [GenAI concept foundations](00-genai-concept-foundations.md). Then use the **Beginner-first concept and project bridge** appended to each of the nine original interview batches. Never use real customer data in experiments. Apply authorization checks before model context assembly. Blogs, Medium and LinkedIn tutorials are listed in the foundations guide alongside official docs; treat blog posts as teaching references rather than verified interview evidence.

## Current reliability increment — bounded agents and repeatable runs

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

**Expected result:** no run exceeds three tool calls; duplicate calls have a visible stop reason; answerable questions consistently cite the same authorized source and return the same structured facts. Keep readiness **Untested** until the lab is run and explained.
