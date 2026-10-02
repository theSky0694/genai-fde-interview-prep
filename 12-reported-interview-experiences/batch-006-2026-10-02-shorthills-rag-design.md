# Batch 006 — ShortHills AI RAG assignment and architecture defence

**Collected:** 2 October 2026. **Source post:** 6 September 2026. **Readiness:** Untested.

## Evidence note

This batch comes from one public, firsthand candidate account: [Chandan Yadav — ShortHills AI campus interview experience](https://www.linkedin.com/posts/chandan-yadav-89aaa3253_shorthillsai-interviewexperience-campusplacement-activity-7502231606145990656-9g_o). The author identifies the company as **ShortHills AI**, the role as **Associate AI Product Engineer**, and describes a three-round India campus process. LinkedIn displayed a relative date; the activity identifier resolves to 6 September 2026. The prompts below are careful paraphrases of the reported topics, not verbatim interviewer wording.

One account contributes **one independent report**, not six. It reinforces existing RAG, chunking, embedding, system-design and DSA themes; it is not evidence of market-wide frequency.

---

## 1. Design document intelligence for 1,000+ documents

**Reported context:** Practical GenAI/RAG assignment. **Roadmap:** Stages 3–4. **Priority:** P0.

**Prompt (paraphrased):** Design an end-to-end system that parses more than 1,000 documents and returns grounded answers.

**What it tests:** Requirement clarification, ingestion, retrieval, grounding and production thinking.

**Senior answer outline:**
- Clarify document types, update frequency, languages, ACLs, query volume, latency and citation requirements.
- Use an idempotent ingestion pipeline: object storage as source of truth; parse/OCR; typed chunks with document/version/page/section metadata; versioned embeddings; retry and dead-letter handling.
- Apply ACL and active-version filters before retrieval. Combine lexical and dense candidates only if evaluation proves the gain; rerank and expand to parent context.
- Generate from bounded evidence, cite source/page, and abstain when evidence is insufficient.
- Evaluate extraction, Recall@K/MRR, citation support, answer correctness, p95 latency and cost separately.

**Trade-off:** Simple batch ingestion is easier to operate; event-driven incremental ingestion improves freshness but adds ordering, replay and deletion complexity.

**Follow-up / drill:** Sketch the ingestion and query paths, then explain how an updated or deleted policy becomes visible without leaking its old version.

---

## 2. Justify structure-aware chunking

**Reported context:** Assignment implementation and final architecture discussion. **Roadmap:** Stage 3. **Priority:** P0.

**Prompt (paraphrased):** Why use structure-aware chunking, and when would it beat fixed-size chunks?

**What it tests:** Whether chunking choices follow document structure and measured retrieval quality.

**Senior answer outline:**
- Preserve headings, paragraphs, table headers, lists and section ancestry so a chunk retains meaning and provenance.
- Use small searchable child chunks with parent expansion when precision and context conflict.
- Keep overlap modest and purposeful; excessive overlap duplicates results and tokens.
- Fixed windows remain sensible for uniform text, a baseline, or documents with unreliable structure.
- Compare approaches on labeled queries using evidence Recall@K, MRR, context completeness, token use and latency—not intuition alone.

**Trade-off:** Structure-aware parsing improves coherence but is parser-dependent, document-specific and more expensive to test.

**Follow-up / drill:** Take one bank policy with a table and nested headings; compare fixed 500-token chunks against section/parent-child chunks on ten gold-evidence questions.

---

## 3. Defend the embedding model selection

**Reported context:** Final-round technical decision review. **Roadmap:** Stages 2–3. **Priority:** P1.

**Prompt (paraphrased):** Why did you choose this embedding model?

**What it tests:** Workload-based model selection rather than tool-name recall.

**Senior answer outline:**
- Define languages, domain vocabulary, maximum useful input length, deployment/data-residency limits, throughput and vector-storage budget.
- Benchmark candidates on the real corpus with gold query–evidence pairs, exact identifiers, paraphrases and no-answer cases.
- Measure Recall@K/MRR, latency, throughput and memory; also test metadata filtering and reranking.
- Record model, preprocessing and index versions. Re-embed into a new index and dual-run before migration; never mix incompatible vector spaces.
- Retain lexical search for IDs, account/product codes and exact policy terms.

**Trade-off:** Larger embeddings may improve semantic retrieval but increase latency, storage and serving cost; domain fit can matter more than dimensions.

**Follow-up / drill:** Benchmark two locally deployable embedding models plus BM25 on 30 bank-style queries and write a one-page selection decision.

---

## 4. Choose the database and vector-search boundary

**Reported context:** Final-round database-choice discussion. **Roadmap:** Stages 3–4 and 7. **Priority:** P0.

**Prompt (paraphrased):** Which database would you choose for this RAG system, and why?

**What it tests:** Storage boundaries, scale assumptions, authorization and operations.

**Senior answer outline:**
- Do not force one store to do everything: original documents belong in durable object storage; workflow/status and structured metadata in a transactional database; embeddings in pgvector, a local index, or a dedicated vector service according to scale.
- For a modest controlled corpus, Postgres plus pgvector can simplify joins, ACL metadata and operations.
- A dedicated vector engine becomes attractive when measured corpus/concurrency, ANN tuning, filtered search or horizontal scaling demands it.
- Benchmark filtered Recall@K, p95 latency, update/deletion behavior, backup/reindex operations and cost.
- Treat the vector index as derived data, never the authorization or document source of truth.

**Trade-off:** Fewer services reduce operational burden; specialized stores can improve scale/features but add network, governance and recovery complexity.

**Follow-up / drill:** Defend pgvector first, then state the measurable thresholds that would trigger migration to a dedicated vector store.

---

## 5. Improve a weak retrieval pipeline

**Reported context:** Final-round discussion of retrieval flow, trade-offs and possible improvements. **Roadmap:** Stages 3–4. **Priority:** P0.

**Prompt (paraphrased):** Relevant documents exist, but answers are poor. How would you diagnose and improve the pipeline?

**What it tests:** Stage-wise debugging rather than random prompt changes.

**Senior answer outline:**
- Reproduce with a labeled trace and determine whether parsing, chunking, filtering, candidate retrieval, reranking, context assembly or generation failed.
- If gold evidence is absent from top-K, inspect extraction and ACL/version filters, then test query rewriting, lexical+dense retrieval, metadata constraints and chunk design.
- If evidence is retrieved but ranked low, tune fusion/reranking on held-out queries.
- If good evidence reaches the model but the answer is wrong, fix context truncation, instructions, citations/abstention or model choice.
- Protect a regression set and compare quality, latency and cost for every change.

**Trade-off:** Query expansion and reranking can raise recall/precision but add model calls, tail latency and failure points.

**Follow-up / drill:** Instrument a trace with retrieved IDs, scores, filters, reranker order, assembled context and citations; diagnose five deliberately seeded failures.

---

## 6. Find the first non-repeating character

**Reported wording:** The post gives the example input `aabbcd` and expected output `c`. **Roadmap:** Stage 1A parallel DSA lane. **Priority:** P1.

**What it tests:** Hash-map counting, order preservation and clean complexity analysis.

**Senior answer outline:** Make one pass to count characters and a second pass through the original string to return the first count of one. This is O(n) time and O(k) space for k distinct characters. State behavior for an empty string and when no unique character exists.

```python
def first_unique_char(value: str) -> str | None:
    counts: dict[str, int] = {}
    for char in value:
        counts[char] = counts.get(char, 0) + 1
    return next((char for char in value if counts[char] == 1), None)
```

**Trade-off:** `collections.Counter` is shorter; the explicit map demonstrates the underlying approach. A fixed-size frequency array is possible only with a clearly bounded alphabet.

**Follow-up / drill:** Return the index instead of the character, then explain how the design changes for a character stream where a second pass is unavailable.

---

## Study order

1. Practise Questions 1 and 5 as a single 10-minute whiteboard discussion.
2. Implement Question 6 without assistance.
3. Use Questions 2–4 as defence prompts for each decision in TxnGuard.

Do not mark any entry ready until Aakash answers it and defends at least one trade-off.
