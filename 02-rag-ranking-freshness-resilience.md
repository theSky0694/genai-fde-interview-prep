# RAG ranking, knowledge freshness and service resilience — from first principles

**Audience:** experienced MuleSoft/API developer beginning GenAI engineering.  
**Prerequisite:** read [GenAI concept foundations](00-genai-concept-foundations.md), especially chunks, embeddings, hybrid retrieval and evaluation.

This lesson connects three parts of one production RAG request:

```text
documents change
   -> ingestion updates the index
   -> retrieval finds candidate chunks
   -> reranking orders the best evidence
   -> LLM generates an answer
   -> service handles provider failure safely
```

## 1. Candidate retrieval versus reranking

A RAG system normally uses two search stages.

### Stage A: retrieve candidates quickly

A **bi-encoder** converts the query and every document chunk into vectors separately. Document vectors can be calculated during ingestion and stored. At request time, only the query vector is calculated, so searching many chunks is fast.

```text
query -> query vector
chunk -> stored chunk vector
similarity(query vector, chunk vector) -> candidate score
```

The model does not compare every query and document word together. Speed is the benefit; subtle relationships can be missed.

### Stage B: rerank a small candidate set carefully

A **cross-encoder** reads the query and one candidate together and predicts their relevance. It can inspect word-to-word interactions more carefully, but it must run for every query-candidate pair.

```text
(query + candidate 1) -> relevance score
(query + candidate 2) -> relevance score
...
```

Use fast retrieval for perhaps 20 candidates, then a cross-encoder to keep the best 5.

**MuleSoft analogy:** the bi-encoder is an indexed database lookup that narrows thousands of records. The cross-encoder is a more expensive validation/enrichment call applied only to the shortlist.

## 2. A tiny reranking lab

This no-model example teaches the shape of the pipeline:

```python
def retrieve_candidates(question: str, chunks: list[dict]) -> list[dict]:
    terms = set(question.lower().split())
    return sorted(
        chunks,
        key=lambda c: len(terms & set(c["text"].lower().split())),
        reverse=True,
    )[:3]

def rerank(question: str, candidates: list[dict]) -> list[dict]:
    # Teaching substitute for a cross-encoder:
    # reward chunks that contain the required subject and unit.
    def careful_score(chunk):
        text = chunk["text"].lower()
        return (
            2 * ("tier b" in text)
            + 2 * ("daily" in text)
            + 1 * ("limit" in text)
        )
    return sorted(candidates, key=careful_score, reverse=True)
```

**Expected:** for “What is the Tier B daily limit?”, a chunk that explicitly contains Tier B, daily and limit should move above a generic transfer-limit chunk.

This is not a real bi-encoder or cross-encoder. Replace the first function with vector or hybrid retrieval and the second with a reranking model after the control flow is understood.

## 3. Why duplicate chunks appear

Chunk overlap, repeated headers, document copies and keyword/vector fusion can return near-identical passages. Repetition wastes the model's context and can overemphasize one fact.

Start with exact source IDs and normalized text hashes:

```python
from hashlib import sha256

def deduplicate(chunks: list[dict]) -> list[dict]:
    seen = set()
    unique = []
    for chunk in chunks:
        normalized = " ".join(chunk["text"].lower().split())
        key = (chunk["document_id"], sha256(normalized.encode()).hexdigest())
        if key not in seen:
            seen.add(key)
            unique.append(chunk)
    return unique
```

Semantic near-duplicate detection can compare embeddings, but a high threshold may incorrectly merge distinct policy exceptions.

## 4. Knowledge freshness and document versions

RAG does not automatically know that a PDF changed. Ingestion must detect the update, create a new index version, validate it, and switch queries safely.

Store metadata with every chunk:

```python
chunk = {
    "chunk_id": "policy-7:v3:p4:c2",
    "document_id": "policy-7",
    "document_version": 3,
    "effective_from": "2026-11-01",
    "content_hash": "abc123",
    "tenant_id": "bank-A",
    "text": "Tier B daily limit is ₹75,000.",
}
```

A safe update flow:

1. calculate a content hash and skip unchanged documents;
2. parse and chunk the new version;
3. write it to a staging index;
4. run retrieval regression tests;
5. atomically promote the new index version;
6. retain rollback metadata;
7. remove or archive the old version according to policy.

**MuleSoft analogy:** this is blue-green deployment for knowledge. Build and test the new knowledge version before routing live traffic to it.

Do not delete the old chunks first. A failed reindex could leave the assistant with no evidence.

## 5. LLM-provider failure and graceful degradation

A **timeout** stops one request that takes too long. A **retry** makes another attempt when the failure may be temporary. A **circuit breaker** stops repeatedly calling a service that is already failing.

Circuit breaker states:

- **closed:** calls flow normally;
- **open:** calls are blocked for a cooling period;
- **half-open:** allow a small probe to see whether the service recovered.

```python
from dataclasses import dataclass
from time import monotonic

@dataclass
class CircuitBreaker:
    failures: int = 0
    opened_at: float | None = None
    threshold: int = 3
    cooldown_seconds: int = 30

    def allow(self) -> bool:
        if self.opened_at is None:
            return True
        return monotonic() - self.opened_at >= self.cooldown_seconds

    def record_success(self) -> None:
        self.failures = 0
        self.opened_at = None

    def record_failure(self) -> None:
        self.failures += 1
        if self.failures >= self.threshold:
            self.opened_at = monotonic()
```

For the learning project, degraded behavior could return retrieved passages and citations without generated prose:

```python
def degraded_response(question: str, hits: list[dict]) -> dict:
    return {
        "status": "generation_unavailable",
        "message": "Answer generation is temporarily unavailable.",
        "sources": [{"id": h["id"], "text": h["text"]} for h in hits[:3]],
    }
```

Only use a fallback model if its security, data residency, quality and cost have been approved. Sending bank data to an unapproved public provider is not graceful degradation.

## 6. Measurable experiments

### Ranking experiment

Use 30 questions with known supporting chunks.

Compare:

1. vector/keyword candidate order;
2. the same candidates after reranking and deduplication.

Measure Recall@10, MRR, duplicate rate, p95 latency and context tokens.

### Freshness experiment

Change five facts in a new corpus version. Confirm:

- all five new facts are retrievable after promotion;
- no query returns an obsolete version when the new version is effective;
- rollback restores the old version;
- unchanged documents are not re-embedded.

### Resilience experiment

Make the model client fail five times.

Confirm:

- each call has a timeout;
- retries are bounded and use backoff;
- the circuit opens after the configured threshold;
- blocked calls do not reach the provider;
- the response exposes sources without inventing an answer;
- recovery closes the circuit after a successful probe.

## 7. Trade-offs

| Decision | Benefit | Cost / risk |
|---|---|---|
| Retrieve more candidates | Higher chance of finding evidence | More reranking cost and noise |
| Cross-encoder reranking | Better query-passage comparison | Extra latency and compute |
| More chunk overlap | Preserves boundary context | Duplicate evidence and larger index |
| Immediate in-place update | Simple | Partial or empty index during failure |
| Versioned index promotion | Testable and reversible | More storage and orchestration |
| Retry | Recovers transient errors | Adds latency and can amplify outages |
| Circuit breaker | Protects provider and callers | Can reject calls during partial recovery |
| Fallback model | May preserve service | Security, quality and cost differences |

## 8. Debugging checklist

1. Verify the expected chunk exists in the active corpus version.
2. Check metadata and tenant filters.
3. Measure whether retrieval found it before inspecting generation.
4. Inspect candidate order before and after reranking.
5. Count exact and near duplicates.
6. Trace timeout, retry and breaker decisions separately.
7. Confirm a degraded response never presents retrieved text as a generated conclusion.
8. Record prompt, model and corpus versions as described in [Agent loops and repeatability](01-agent-loops-and-repeatability.md).

## 9. Interview summary

**Beginner:** “I use fast retrieval to find candidates, optional reranking to order the shortlist more carefully, versioned ingestion so updated documents can be tested and promoted safely, and timeouts plus a circuit breaker so an LLM outage does not create unlimited retries. If generation is unavailable, I return a clear status and approved source excerpts rather than inventing an answer.”

**Senior extension:** “I evaluate retrieval and reranking separately with Recall@K, MRR, duplicates, tokens and latency. Content hashes and blue-green index versions make updates idempotent and reversible. The generation client has end-to-end deadlines, retry classification, jittered backoff and a circuit breaker. Degraded modes respect tenant authorization and data residency.”

## 10. Practice task

Extend the [Enterprise Document Intelligence Assistant](PROJECT_ENTERPRISE_DOCUMENT_ASSISTANT.md) with:

- retrieve 10, deduplicate, rerank and keep 3;
- document version and content hash metadata;
- staging-to-active index promotion;
- a model-client timeout and circuit breaker;
- a source-only degraded response.

**Expected result:** the correct source ranks in the final top 3, updated facts replace old facts after promotion, three simulated provider failures open the circuit, and the service returns authorized citations without a fabricated answer.

**Readiness:** Untested until implemented and explained.
