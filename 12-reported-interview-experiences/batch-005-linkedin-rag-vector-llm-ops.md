# Batch 005 — LinkedIn interview reports: document RAG, vector search, and production LLM APIs

**Collected:** 26 September 2026. **Readiness:** Untested; these are practice prompts, not evidence that Aakash has answered them.  
**Evidence:** Two public firsthand accounts (one candidate, one interviewer) and one explicitly secondhand report. LinkedIn displayed relative ages (approximately 3, 5, and 7 months respectively) at collection; exact publication dates were not exposed, so none is asserted below. No company is attributed to the firsthand accounts.

## Source register and evidence limits

| ID | Public source | Nature of report | Use here |
|---|---|---|---|
| S1 | [Nikhil Naganur, PDF/RAG architecture interview](https://www.linkedin.com/posts/nikhil-naganur-b765aa220_genai-rag-datascientist-activity-7465037876569907200-UcCz) | Interviewer describes asking a candidate about text, images, tables, extraction, linkage and retrieval. Firsthand interviewer account; company unnamed. | Question 1 is a careful paraphrase, not a transcript. |
| S2 | [Sandhya Sahani, two AI/ML Engineer interviews](https://www.linkedin.com/posts/sandhya-sahani-50848035a_ai-machinelearning-interviewexperience-activity-7450781558023622656-QK2u) | Candidate reports two senior/mid-level interviews; employer names and which round asked each question are not supplied. | Question 2 combines closely related vector DB/project questions she lists. |
| S3 | [Arunabha Sarkar, reported Infosys GenAI Developer interview](https://www.linkedin.com/posts/arunabha-sarkar-alex_infosysinterview-genaiinterview-llmengineering-activity-7427924490401103872-e3y0) | Author says a friend attended; secondhand, not independently confirmed with the candidate or employer. | Question 3 groups reported rate-limit, monitoring, cache, streaming and data-protection topics. |

These are anecdotes, not a representative sample. S1 and S2 can contribute one independent report each after deduplication; S3 remains in a separate secondhand category. The subjects overlap existing batches, so this batch adds *worked interview answers and drills*, not new normalized themes or automatic priority changes.

---

## 1. Design RAG for PDFs containing text, tables and images

**Source:** S1. **Roadmap:** Stages 3–4; extends TxnGuard ingestion. **Priority:** P0 because retrieval engineering was already P0, not because of a single post.

**Interview prompt (paraphrased):** A PDF has prose, tables and images. How would you extract and store each element, preserve its relationship to a page/section, and retrieve the right evidence for a user question?

### 30-second answer

“I would separate ingestion from retrieval. Parse the PDF into typed elements with page coordinates and stable source IDs; use OCR for scanned text, table extraction for structured cells, and image captions or multimodal embeddings only when visual content matters. Store the original asset and a searchable representation, linking both to document version, page, section and access policy. At query time, use keyword plus semantic retrieval and reranking, then expand to the parent table, figure or surrounding section. Return the original page/asset with a precise citation and test retrieval and answer grounding separately.”

### Detailed answer

1. **Clarify the task.** Are users asking about textual policy, numerical tables, diagrams, or all three? What counts as an acceptable citation? What are document volume, update frequency, access controls, languages, latency and on-premise constraints? These determine whether multimodal indexing is justified.
2. **Extract with provenance.** Detect born-digital versus scanned pages. Parse text blocks and headings; OCR scans and image text; extract tables into rows/cells while preserving headers, units, merged cells and footnotes; retain image crops and nearby captions. A parser's output is a hypothesis: measure it against a manually inspected sample, especially columns, reading order and page breaks.
3. **Create typed, stable records.** For each element keep `document_id`, immutable `version_id` or content hash, `page_number`, `element_id`, `element_type`, bounding box, section path, neighboring/parent IDs, source URI, ACL/tenant, extraction method and confidence. Store original PDF and crops in protected object storage. A vector index is a search aid, not the canonical source.
4. **Index according to content.** Chunk prose along headings/paragraphs while retaining section context. Serialize tables with headers and units attached to each relevant row or group; keep a pointer to the complete table. Caption images or embed them with a vision model only if actual visual retrieval is needed. Index identifiers and exact terms for BM25 as well as semantic vectors; route numerical queries to structured table search/calculation rather than trusting an embedding to preserve quantities.
5. **Retrieve and assemble evidence.** Classify the query only if it measurably helps. Apply ACL and document-version filters **before** results reach the model. Retrieve candidates from lexical and vector indexes, fuse/rerank, then expand a matched row/chunk to its parent table or section. Budget context so headers, units and footnotes are not cut off. Return the original page crop/table alongside the answer where the UI permits.
6. **Generate with a support contract.** Require each factual claim or number to cite an element/page and abstain or ask for clarification when evidence is missing or conflicting. Do not claim a citation proves correctness merely because the cited page exists.
7. **Evaluate stages separately.** Build labeled queries for prose, tables, images, cross-page evidence, scanned pages and access-controlled content. Track extraction fidelity, evidence Recall@K, reranker quality, citation-to-claim support, numeric exactness, answer correctness, latency and cost. Include negative queries and document revisions/deletions.

### Trade-offs to defend

| Choice | When it helps | Cost or failure mode |
|---|---|---|
| OCR all pages vs only scans | Uniform text path may simplify indexing | More cost and OCR errors on clean digital PDFs; page classifier can misroute |
| Flatten tables as prose vs preserve structure | Simple semantic search over small, plain tables | Loses headers, units, merged cells and exact row relationships |
| Image captions vs multimodal embeddings | Captions are cheap and searchable using text infrastructure | Captions omit details; embeddings add model/storage complexity and need their own evaluation |
| Small chunks vs parent-child retrieval | Small chunks improve focused retrieval | Evidence may lose context; expansion increases tokens |
| Vector only vs hybrid retrieval | Dense retrieval handles paraphrases | Exact regulation IDs, amounts and rare names often need lexical search |
| Parse at query time vs offline | Query-time parsing accommodates fresh documents | Expensive and high latency; offline needs versioning/reindexing discipline |

### Likely follow-ups and answers

**Q: How do you avoid returning a table row with the wrong column heading?**  
A: Keep a structured table representation and link every cell/row to its header path, units, footnotes and parent table. Retrieve a candidate row but present the parent context to the model. Add tests where two adjacent columns have similar values and where merged headers or page-spanning tables occur.

**Q: What if the PDF is updated?**  
A: Ingest an immutable new version, reparse/reindex changed content, switch the active version atomically, and invalidate cached answers. Preserve version IDs in citations for audit. Propagate deletions and permission changes to search indexes and caches; otherwise stale content can remain accessible.

**Q: How would you measure success?**  
A: Start with a small hand-labeled set containing the exact supporting element and expected answer. Measure extraction, evidence retrieval and final grounded answer separately. If evidence is absent from top-K, fix parsing/index/retrieval; if present but the answer is wrong, inspect context assembly, prompting or model behavior.

**Q: Would LangGraph improve this?**  
A: Use a deterministic ingestion pipeline and simple retrieval API first. Add a stateful workflow when queries genuinely need routing, multi-step retrieval or human review; measure whether it improves task success enough to justify latency, trace complexity and operational overhead.

**Common weak answer:** “Upload the PDF, chunk it, embed it and let the LLM handle images.” That skips extraction, table structure, provenance, ACLs and measurement.

**Hands-on drill:** Create three sample PDFs: clean prose, a scanned page, and a table with units/footnotes. Produce typed element JSON, inspect it manually, and answer six questions with page-level evidence; log parsing and retrieval failures.

---

## 2. Defend your vector database and retrieval design

**Source:** S2. **Roadmap:** Stages 3–4 and project deep dive. **Priority:** P0/P1 within the existing retrieval emphasis.

**Interview prompt (paraphrased):** Why did you choose your vector database, how does vector search work in your project, and what alternatives did you reject?

### 30-second answer

“I would first define the workload: corpus size and churn, top-K latency, concurrent queries, metadata/ACL filters, exact keyword needs, deployment constraints and operational skills. Embeddings map queries and chunks into the same vector space; an index retrieves nearby candidates, often approximately. I would benchmark candidate stores with representative queries and filters, then compare recall, latency, update/deletion behavior, hybrid search, cost and operational complexity. My choice is workload-specific, and I would use BM25 or SQL where exact matching or structured filters dominate.”

### Detailed answer

- **Work backward from requirements.** A few thousand static chunks can work with an in-process exact search or PostgreSQL plus pgvector. A large, frequently updated, multi-tenant corpus may need specialized ANN indexing, scaling and operational controls. The product name is the conclusion, not the justification.
- **Explain the retrieval path.** Ingestion normalizes/chunks documents and records model/version metadata. Query embedding uses the *same compatible embedding model and preprocessing*. The index uses a similarity metric appropriate to the model; exact KNN compares every vector, while ANN trades some recall for latency/memory. Filter by tenant/ACL and version, get candidates, optionally combine lexical search and rerank, then assemble cited context.
- **Benchmark on your data.** Label query-to-evidence pairs, including exact IDs, paraphrases and no-answer cases. Compare Recall@K and ranking quality, p50/p95 latency under concurrent load, index build/update time, memory/storage, filter correctness, deletion propagation and cost. ANN parameters are tuned against an exact-search baseline. Never equate “top cosine score” with answer correctness.
- **Operationalize.** Version embeddings and indexes so model changes do not silently mix spaces. Specify retry/idempotency for ingestion, backups, reindex strategy, observability and authorization. In a bank, source ACL enforcement and deletion behavior may matter more than a tiny latency difference.

### Trade-offs to defend

| Option | Reason to choose | Reason to reconsider |
|---|---|---|
| FAISS/local index | Small controlled corpus, prototype, low serving complexity | Persistence, multi-user updates, filtering and operations may need more engineering |
| PostgreSQL + pgvector | Existing Postgres operations and joins/ACL metadata; moderate workload | Benchmark ANN/filter performance and scaling at actual corpus/concurrency |
| Dedicated vector store | Larger scale, managed indexing, filtering and operations features | Extra system, cost, network boundary and data governance work |
| Lexical/BM25 | Exact terms, IDs, codes and rare names | Misses semantic paraphrases |
| Dense vectors | Paraphrases and semantic similarity | Can blur numbers, negation and exact identifiers |
| Hybrid + rerank | Better coverage across query types | More latency/cost and more failure points; prove gain on labeled queries |

### Likely follow-ups and answers

**Q: If cosine similarity is high, is the answer relevant?**  
A: No. It scores vector closeness, which can retrieve topically similar but wrong versions, amounts or obligations. Inspect evidence labels, apply metadata constraints, rerank where useful and validate grounded answers.

**Q: What changes if you switch embedding models?**  
A: Dimensions and vector geometry may differ. Build a versioned new index, dual-run retrieval against the same evaluation set, then switch atomically. Do not mix old document vectors with new query vectors by accident.

**Q: How do you enforce access control?**  
A: Carry source permissions into indexed metadata, filter candidates before any model context or logs expose content, and verify tenant separation with adversarial tests. Recheck permissions at fetch time for sensitive content and propagate revocation to caches.

**Q: Why not put everything in a vector database?**  
A: Transactions, exact filters and aggregations belong in a structured store. A vector index finds candidate evidence; it should not replace source-of-truth records or authorization logic.

**Common weak answer:** “We chose X because it is fast and popular.” The interviewer wants workload assumptions and measured trade-offs.

**Hands-on drill:** Benchmark BM25, dense and hybrid retrieval on 30 bank-style queries (IDs, paraphrases, amounts, policy sections) and report Recall@5, p95 latency and failures.

---

## 3. Operate an LLM-backed API under rate limits and failures

**Source:** S3, a **secondhand** Infosys report. **Roadmap:** Stages 1B, 2 and 7. **Priority:** P1 as useful production integration practice; do not count it as an independently verified candidate report.

**Interview prompt (synthesized from closely related reported topics):** How would your FastAPI GenAI service handle model rate limits, failures, monitoring, streaming, caching and sensitive data?

### 30-second answer

“I would set per-user and global concurrency/token budgets, use timeouts and bounded retries with exponential backoff and jitter for retryable failures, honor provider retry guidance, and return a clear error or safe fallback when the deadline expires. Every request has a trace ID and metrics for latency, token use, rate limits, failure type and cost, with sensitive content excluded from logs. Streaming needs disconnect and cancellation handling. Cache only where the request, permissions, model/prompt version and data freshness make reuse safe.”

### Detailed answer

1. **Budget at the boundary.** Authenticate the caller, enforce tenant quotas and input limits, then estimate model-token demand and available capacity. Queue or reject when saturated rather than creating an unbounded backlog. Separate interactive deadlines from background jobs.
2. **Classify failure before retry.** Retry transient network/5xx and rate-limit responses if the operation is safe and the deadline allows; honor `Retry-After` when supplied. Use capped exponential backoff with jitter and a retry budget. Fail fast for validation/auth errors. If the model provider may have processed a request before the connection failed, account for duplicate charges/actions; tool side effects need idempotency keys and reconciliation.
3. **Limit cascading failures.** Use bounded concurrent calls, circuit breaking or temporary shedding where warranted, and model fallback only after checking quality, data-residency and cost constraints. In an on-prem bank deployment, a local fallback can still overload the same GPU server: use per-model capacity measures.
4. **Observe without leaking.** Emit request/trace IDs and spans for retrieval, model and each tool. Track p50/p95 latency, time-to-first-token, queue time, tokens, cost, 429/5xx/timeouts, retry count, tool errors and grounded-answer quality. Redact or avoid prompts, account identifiers, retrieved confidential text and secrets in logs/traces. Audit high-impact actions separately.
5. **Stream deliberately.** Streaming improves perceived latency, not necessarily total compute cost. Propagate client cancellation, close upstream streams, handle partial outputs and define what happens if an error occurs after the first token. Buffer or validate structured output before committing actions; never execute a tool merely because an incomplete streamed fragment resembles a call.
6. **Cache only valid reuse.** A cache key may include tenant/permission scope, normalized query, prompt/model versions, retrieval-index/document versions and relevant parameters. Responses to personalized or rapidly changing questions may be uncacheable. Set TTL/invalidation and measure hit rate *and* stale/unauthorized response risk.

### Trade-offs to defend

| Decision | Benefit | Risk |
|---|---|---|
| More retries | Better recovery from transient failures | Tail latency, duplicate cost, retry storms |
| Global queue | Smoothes spikes | Long waits; needs per-tenant fairness and deadlines |
| Fallback model | Availability | Different quality, behavior and data-handling rules |
| Streaming | Faster first visible token | Partial failures, cancellation and output-validation complexity |
| Response caching | Lower cost/latency | Stale or cross-user data if key/ACL scope is wrong |
| Detailed traces | Faster debugging | Sensitive-data exposure unless minimized and access-controlled |

### Likely follow-ups and answers

**Q: What would you do on HTTP 429?**  
A: Honor retry guidance, apply bounded backoff with jitter and a deadline, reduce concurrency or queue load, and expose a retryable response if the deadline cannot be met. Measure 429s by model and tenant; do not let every worker retry simultaneously.

**Q: Can you retry a tool call that writes to an account?**  
A: Only with an idempotency or deduplication contract and confirmation of the prior outcome. Distinguish a model retry from a side-effecting business operation; a network timeout does not prove the write failed.

**Q: How would you show this worked in production?**  
A: Run load and failure-injection tests, report p95 latency, success rate, queue saturation, token cost, retry amplification and tenant isolation; inspect traces for a sample of failed and degraded requests.

**Q: Would you cache a RAG response?**  
A: Only if authorization and document versions are part of the validity contract. Often caching retrieval or embeddings is safer and easier to invalidate than caching a full answer.

**Common weak answer:** “Add a retry decorator, log the prompt, and cache by user question.” That can create storms, expose confidential data and serve stale or unauthorized answers.

**Hands-on drill:** Stub a model endpoint that returns 429, 500, a slow stream and a partial stream. Build a FastAPI proxy with bounded retries, deadlines, cancellation, trace IDs and non-sensitive metrics. Demonstrate an idempotent tool action and an explicit cache invalidation test.

---

## Study order and progress rule

1. **This week:** Question 3 pairs naturally with current Python/FastAPI work; implement its failure-handling drill in a small service.
2. **RAG stage:** Questions 1 and 2 become design reviews and labeled evaluations for TxnGuard.
3. **Interview practice:** Explain the 30-second answer first, then defend one trade-off and one failure case without notes.

Do not mark a question interview-ready until Aakash has answered it and defended the trade-offs. The report sources describe prompts; all answer architectures, drills and follow-up answers above are preparation material written for this repository, not claims that those precise responses were given in the interviews.
