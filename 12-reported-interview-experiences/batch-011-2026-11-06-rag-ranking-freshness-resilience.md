# Batch 011 — RAG ranking, document freshness and LLM-service resilience

**Collected:** 6 November 2026. **Readiness:** Untested.

## Evidence note

This batch uses two independently posted reports:

1. [Barclays Pune — Software Engineer, GenAI interview experience](https://takeuforward.org/community/interview-experiences/barclays-pune-ai-genai-engineer-interview-experience). The public page displayed **“23h ago” when verified on 6 November 2026**; the site did not expose an exact calendar timestamp, so none is invented. The anonymous candidate reports a 75-minute assessment, two technical rounds and a hiring-manager round. The report explicitly lists questions about LLM unavailability, document updates, reranking, access control, circuit breakers, retries, idempotency, FastAPI and production RAG.
2. [Bharath Reddy — AI Engineer interview on RAG trade-offs](https://www.linkedin.com/posts/bharath-reddy-35a405195_i-interviewed-an-ai-engineer-recently-his-activity-7508347304744005632-QdDm). LinkedIn displayed **“1w” when verified on 6 November 2026** and did not expose an exact publication date. This is a first-person interviewer account. It explicitly includes hybrid search, bi-encoder versus cross-encoder, and duplicate retrieval. The post promotes a paid interview kit; only the visible interview claims are used.

These are two reports, regardless of question count. The reported-experience counter increases by two. They are useful signals, not a representative hiring-market sample.

## Dependency order

1. Read [GenAI concept foundations](../00-genai-concept-foundations.md), sections 3–8.
2. Work through [RAG ranking, freshness and service resilience](../02-rag-ranking-freshness-resilience.md).
3. Reuse tracing and bounded retries from [Agent loops and repeatability](../01-agent-loops-and-repeatability.md).
4. Implement the project increment in [Enterprise Document Intelligence Assistant](../PROJECT_ENTERPRISE_DOCUMENT_ASSISTANT.md).
5. Practise the interview answers below.

---

## 1. What is the difference between a bi-encoder and a cross-encoder, and where does reranking fit?

**Source:** direct question in the Bharath Reddy interviewer account; Barclays independently reports a reranking question.  
**Roadmap:** Stage 3 — retrieval and ranking. **Priority:** P0.

### What it tests

Whether you understand why production RAG often uses a fast search stage followed by a slower, more accurate ranking stage.

### Prerequisite concept in simple language

A **bi-encoder** encodes the question and each chunk separately. Chunk vectors can be stored before users ask questions. Similarity search is therefore fast.

A **cross-encoder** reads the question and one candidate chunk together. It can judge their relationship more carefully, but it must run once per candidate at request time.

**API analogy:** the bi-encoder is a database index that cheaply produces 20 candidates. The cross-encoder is a slower rules/validation service called only for those 20.

### Project placement

```text
question
 -> hybrid/vector retrieval: top 20 candidates
 -> remove duplicates
 -> cross-encoder reranker: best 5
 -> authorization check and prompt assembly
 -> answer with citations
```

Authorization should be applied before any unauthorized content reaches the model. Depending on the store, tenant filtering should also occur during retrieval.

### Small example

```python
def final_context(question, retriever, reranker):
    candidates = retriever(question, top_k=20)
    unique = deduplicate(candidates)
    scored = reranker.rank(question, unique)
    return scored[:5]
```

**Expected input/output:**

```text
Input question: What is the Tier B daily transfer limit?
Retriever: 20 candidate chunks, including 4 duplicates
Deduplication: 16 unique chunks
Reranker: correct policy page moves from rank 7 to rank 1
Output: 5 chunks for the answer prompt
```

The `deduplicate` function and a runnable teaching substitute are in the linked concept lesson.

### Why this approach

The expensive model sees only a shortlist. This balances scale and precision. It also gives two separate places to evaluate: candidate recall and final ranking.

### Alternatives and trade-offs

- **Vector search only:** simple and fast; may rank subtle relevance poorly.
- **Keyword/BM25 only:** strong for exact policy IDs and names; weak for paraphrases.
- **Hybrid retrieval:** combines exact and semantic matches; fusion can create duplicates.
- **Larger top-k:** improves candidate recall but raises reranking latency and context noise.
- **No reranker:** reasonable if the corpus is small and evaluation shows no benefit.

### Beginner-friendly interview answer

“A bi-encoder creates separate vectors for the query and documents, so it is fast enough to retrieve from a large index. A cross-encoder reads the query and each candidate together, so it ranks relevance more carefully but costs more. I retrieve perhaps 20 candidates, deduplicate them, rerank, and pass the best few to the LLM.”

### Senior-level answer

“I treat retrieval as candidate generation and the cross-encoder as precision ranking. I tune retrieve-k and final-k on a labeled set, monitor Recall@K before reranking and MRR or nDCG after it, and include latency, duplicate rate and context tokens. Hybrid retrieval is valuable for identifiers plus paraphrases, but I fuse by stable source identity and enforce tenant filters before context assembly.”

### Follow-ups with explained answers

**Why not use the cross-encoder on every chunk?**  
Its cost grows with the number of query-chunk pairs. A million-chunk corpus would require an impractical number of model evaluations per question.

**What if reranking lowers answer quality?**  
Confirm the required passage was in the candidate set, inspect whether the reranker demoted it, and compare answer metrics. The reranker may be trained for a different language or domain.

**How do you reduce duplicate retrieval?**  
First merge exact source/chunk IDs and normalized text hashes. For near-duplicates, use a high semantic-similarity threshold and preserve distinct versions or policy exceptions.

**Which metric belongs to which stage?**  
Recall@K checks whether candidate retrieval found the required evidence. MRR/nDCG evaluate ordering. Grounded answer correctness evaluates generation.

### Common mistakes

- Saying cross-encoders generate embeddings for the whole vector index.
- Adding a reranker without measuring the baseline.
- Passing all retrieved duplicates to the LLM.
- Measuring only final prose and not candidate recall.
- Applying authorization only after prompt assembly.

### Practical exercise

Use 30 questions with known source chunks. Compare hybrid retrieval alone with retrieve-20, deduplicate, rerank-to-5.

**Expected result:** a table with Recall@20, MRR@5, duplicate rate, p95 latency, context tokens and answer accuracy. Keep the reranker only if quality gain justifies latency.

---

## 2. How do you handle document updates in a production RAG system?

**Source:** direct question in the Barclays Pune candidate account.  
**Roadmap:** Stages 3, 4 and 7 — ingestion, evaluation and operations. **Priority:** P0.

### What it tests

Whether you see RAG as an evolving data product rather than a one-time vector upload.

### Prerequisite concept in simple language

The LLM does not read your latest document automatically. RAG answers from whatever chunks are currently in the active index. Updating a PDF therefore requires an ingestion workflow:

```text
detect change -> parse -> chunk -> embed -> stage -> test -> promote
```

Store a document ID, version, effective date and content hash with each chunk.

**MuleSoft analogy:** treat knowledge like a versioned API deployment. You test a new revision before routing production traffic to it and retain a rollback path.

### Project placement

```python
chunk = {
    "chunk_id": "policy-7:v3:p4:c2",
    "document_id": "policy-7",
    "document_version": 3,
    "content_hash": "abc123",
    "effective_from": "2026-11-01",
    "tenant_id": "bank-A",
    "text": "Tier B daily limit is ₹75,000.",
}
```

Build version 3 in a staging collection. Run the fixed retrieval tests. Switch an `active_index_version` pointer only after the tests pass.

### Small pseudocode

```python
def ingest_if_changed(document, active_hash, build_index, validate, promote):
    new_hash = sha256(document.bytes).hexdigest()
    if new_hash == active_hash:
        return "unchanged"

    candidate_version = build_index(document)
    if not validate(candidate_version):
        return "rejected"

    promote(candidate_version)
    return "promoted"
```

**Expected behavior:** unchanged PDFs are skipped; a valid new version becomes active; a version failing retrieval tests never replaces the current index.

### Why this approach

Staging prevents partial indexes from serving traffic. Content hashes make ingestion idempotent. Explicit versions let every answer trace show which knowledge it used.

### Alternatives and trade-offs

- **In-place update:** simpler and cheaper; risks mixed versions or an empty index during failure.
- **Blue-green collections:** testable and reversible; requires extra storage.
- **Full rebuild:** consistent; expensive for large corpora.
- **Incremental update:** faster; harder to detect deletions and dependency changes.
- **Keep historical versions:** useful for “as of date” questions; increases storage and filtering complexity.

### Beginner-friendly interview answer

“I store version and hash metadata for each document. When content changes, I create chunks and embeddings in a staging index, run retrieval regression tests, and atomically promote the version. I do not delete the active version first. Every answer trace records the corpus version so stale answers can be diagnosed.”

### Senior-level answer

“I separate source change detection, parsing, chunking, embedding and publication. Content-addressed ingestion is idempotent; blue-green index versions support acceptance tests and rollback. Queries filter by tenant, effective date and active version. I monitor ingestion lag, failed documents, stale-hit rate, deleted-document leakage and retrieval regressions.”

### Follow-ups with explained answers

**What happens when a document is deleted?**  
Create a tombstone or deletion event, build a candidate index without the document, test it, then promote. Verify caches and replicas no longer return the removed source.

**Should every edit trigger full re-embedding?**  
No. Hash chunks or sections and embed only changed content when the embedding model and parsing rules are unchanged. A model or chunking change may require a full rebuild.

**How do you prevent answers during partial ingestion?**  
Queries use only the active immutable version. Staging data is invisible until validation and promotion complete.

**How do you test freshness?**  
Maintain questions tied to changed facts, assert the active version returns new evidence, and measure ingestion-to-availability delay.

### Common mistakes

- Overwriting the live index while it serves traffic.
- Omitting document/corpus version from traces.
- Re-embedding unchanged files on every run.
- Forgetting deletion and cache invalidation.
- Mixing old and new policy versions without an effective-date rule.

### Practical exercise

Change five facts in the synthetic policy corpus. Build a new staged version and intentionally make one expected source missing.

**Expected result:** validation rejects the bad version; the current index remains available. After fixing it, promotion makes all five new facts retrievable, and rollback restores the previous version.

---

## 3. What happens when the LLM API goes down?

**Source:** direct question in the Barclays Pune candidate account, repeated as a system-design constraint.  
**Roadmap:** Stage 7 — production reliability. **Priority:** P0.

### What it tests

Whether you can apply distributed-system reliability patterns without hiding failures or leaking data through an unsafe fallback.

### Prerequisite concept in simple language

- A **timeout** bounds how long one call may wait.
- A **retry** makes another attempt for a temporary failure.
- **Exponential backoff with jitter** spreads retries instead of creating a traffic spike.
- A **circuit breaker** temporarily blocks calls after repeated failures.
- **Graceful degradation** returns a limited, clearly labeled service.

**MuleSoft analogy:** this is the same distinction among connector response timeout, Until Successful, reconnection strategy and a fallback flow. The GenAI-specific question is what limited response remains safe and useful.

### Project placement

The document assistant can still retrieve authorized sources while generation is unavailable:

```python
async def answer(question, retriever, llm_client, breaker):
    hits = retriever.search(question)

    if not breaker.allow():
        return source_only_response(hits)

    try:
        result = await llm_client.generate(
            question=question,
            context=hits,
            timeout_seconds=8,
        )
        breaker.record_success()
        return result
    except TransientProviderError:
        breaker.record_failure()
        return source_only_response(hits)
```

**Expected degraded output:**

```json
{
  "status": "generation_unavailable",
  "message": "Answer generation is temporarily unavailable.",
  "sources": [{"id": "policy-7:p4", "text": "Tier B daily limit is ₹75,000."}]
}
```

This response exposes only already-authorized evidence and does not pretend that an LLM answer was produced.

### Why this approach

It bounds latency, avoids a retry storm and keeps the system honest. Retrieval-only degradation may still help users while preventing fabricated answers.

### Alternatives and trade-offs

- **Immediate failure:** simplest and safest; poor user experience.
- **Bounded retry:** recovers transient problems; increases latency.
- **Circuit breaker:** protects both systems; may reject calls during partial recovery.
- **Fallback model:** maintains generation; may violate quality, cost or data-residency requirements.
- **Queue for later:** useful for asynchronous reports; unsuitable for an interactive response unless the UI supports status polling.

### Beginner-friendly interview answer

“I use an end-to-end timeout, retry only classified transient failures with bounded backoff, and open a circuit breaker when the provider is unhealthy. The service returns a clear degraded status and authorized citations instead of inventing an answer. A fallback model is used only if security, data residency and quality are approved.”

### Senior-level answer

“I separate request deadlines from per-attempt timeouts and budget retries within the remaining deadline. I retry rate limits or transient 5xx failures where idempotent, not validation or authorization errors. The breaker publishes health metrics and uses limited half-open probes. Degraded behavior is contract-tested, tenant-safe and observable; fallback routing respects model approval, residency, cost and corpus confidentiality.”

### Follow-ups with explained answers

**Should every failure be retried?**  
No. Bad input, authentication failure, policy denial and invalid tool schemas need correction. Retrying them wastes capacity. Retry only failures likely to change.

**What is the danger of synchronized retries?**  
Many instances retrying at the same interval can overload a recovering provider. Jitter randomizes the delay.

**Can retrieval results be returned directly?**  
Yes, if they are authorization-filtered and clearly labeled as source excerpts. Do not convert them into an unsupported conclusion.

**How would you test the breaker?**  
Use a fake client that fails repeatedly. Assert the configured number of provider calls, open state, blocked calls, degraded response and successful recovery probe.

### Common mistakes

- Infinite retries inside a user request.
- Treating timeouts and retries as the same control.
- Failing over confidential content to an unapproved provider.
- Serving cached answers without tenant and corpus-version keys.
- Returning a normal 200 answer that hides degraded generation.
- Omitting provider, attempt and breaker state from traces.

### Practical exercise

Make the fake model fail five times, then succeed.

**Expected result:** three failed provider calls open the breaker; subsequent requests do not call the provider and return source-only responses; after the cooldown, one successful probe closes the circuit.

---

## Study plan for this batch

1. **30 minutes:** read [RAG ranking, freshness and service resilience](../02-rag-ranking-freshness-resilience.md).
2. **45 minutes:** implement retrieve → deduplicate → rerank.
3. **45 minutes:** add document hashes and staged index promotion.
4. **45 minutes:** add a fake LLM client, timeout and circuit breaker.
5. **30 minutes:** record experiment results and speak the three beginner answers.
6. Attempt senior answers only after the code path and expected failures are understood.

No readiness status changes until the exercises are implemented and explained.
