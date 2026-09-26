# RAG engineering — interview answers

[Guide home](README.md) · Original explanations informed by the topics in the [source RAG collection](https://github.com/Nareshedagotti/AI-Engineer-Interview-QA/blob/main/RAG_QA.md). Existing answers in Batch 001 and Batch 005 remain intact. **Roadmap:** Stages 3–4. **Readiness:** Untested.

## 1. What actually happens before a PDF becomes searchable?

**Short answer.** Ingest the source with version and ACL metadata, parse its layout, OCR scans if needed, extract tables/images separately, normalize and chunk with source coordinates, embed/index searchable representations, and retain links to original pages. Test extraction before blaming retrieval.

**Detailed explanation.** A born-digital PDF may expose text while a scan needs OCR. Page text order is often different from visual reading order. Repeated headers, multi-column layouts, footnotes and tables can contaminate chunks. Keep typed elements (paragraph, heading, table, figure) and source IDs, page, section and bounding box. Tables need headers/units attached to relevant rows; figures may need captioning or a multimodal index if the use case warrants it. Store the original asset as the canonical evidence and build searchable indexes as derivatives. Make ingestion idempotent, versioned and deletion-aware. A good answer names the failure modes rather than promising that one parser handles everything.

**Trade-offs.** A general parser is faster to build but may fail on complex layouts. OCR adds coverage and cost/error. Flattened tables are easy to index but lose structure. Multimodal embeddings may help visual queries but require labeled evaluation and heavier operations.

**Follow-ups.**
- *How do you detect parsing failures?* Sample and label documents, flag empty/low-text pages, abnormal reading order, broken table structure and low OCR confidence; compare extracted text to rendered pages.
- *Where do you enforce permissions?* Carry ACLs from source into metadata and filter before retrieved content reaches the model; recheck at fetch time for sensitive data.
- *What about updates?* Use immutable document versions, reindex changed content and atomically switch active versions; invalidate caches and propagate revocations.

**Drill.** Parse three PDFs (digital, scan, table), inspect element JSON and create six queries whose expected evidence is labeled.

## 2. How do you choose chunk size and overlap?

**Short answer.** Start from document structure and the answer's evidence span, then tune chunking on a labeled query set. Small chunks may improve precision but lose context; large chunks preserve context but can bury the answer and waste tokens. Overlap can rescue boundary splits but duplicates evidence and cost.

**Detailed explanation.** A policy clause, its exception and heading should often travel together. For a table, splitting a row from its headers makes the number meaningless. For code, function/class and file path are better boundaries than arbitrary character counts. Use heading-aware or recursive splitting as a baseline, then parent-child retrieval: index a focused child, return its parent section when selected. Tune chunk length, overlap and retrieval top-K jointly; changing one can change ranking and context assembly. Measure evidence Recall@K, precision, answer support, index size and latency on realistic questions rather than adopting a universal token count.

**Trade-offs.** Fixed-size chunks are reproducible and cheap. Semantic chunking respects topic shifts but adds computation and can still mishandle tables. More overlap raises recall in some boundary cases while amplifying duplicates and apparent corroboration. Parent expansion gives context but costs tokens.

**Follow-ups.**
- *What if an answer spans two pages?* Preserve section/page links, retrieve multiple elements and expand neighbors or parents; label multi-hop examples in evaluation.
- *Why can larger chunks lower quality?* Embedding represents mixed topics, reranking is less focused and the model receives distractors.
- *When is no overlap fine?* Well-formed atomic sections or parent-child retrieval may already preserve boundaries; verify with tests.

**Drill.** Compare 250-, 500- and 900-token structure-aware chunks plus parent-child retrieval on 30 labeled queries; record retrieval and answer metrics.

## 3. Why use hybrid retrieval and reranking?

**Short answer.** Dense retrieval catches paraphrases; keyword/BM25 catches exact identifiers, names and rare terms. Hybrid combines candidate sets, often with rank fusion, and a reranker spends more compute on a small set to improve ordering. Each stage must earn its cost in measured recall and answer quality.

**Detailed explanation.** A user might ask for “suspicious account activity” while the policy says “unusual transactions”; dense vectors help. A specific policy code or account identifier needs lexical matching. Retrieve top candidates from both under ACL/version filters, deduplicate by source ID, fuse rankings (for example RRF), and rerank a manageable candidate set against the original query. Cross-encoders can assess query/document jointly and may improve relevance at higher latency; bi-encoders precompute document vectors and scale better for initial search. Preserve score/rank traces so failures can be attributed to missing candidates versus wrong ordering.

**Trade-offs.** Vector-only is simpler but weak on exact tokens. Hybrid adds index/operations work and can admit more noise. Reranking can improve precision but adds latency and does not recover evidence absent from the candidate set. Tune top-K to context budget and query type.

**Follow-ups.**
- *What does RRF solve?* It combines rank lists without requiring raw score calibration across lexical and vector retrievers; its rank constant and candidate depth still need tuning.
- *Why can ANN miss a relevant chunk?* Approximation, index parameters, filtering interaction or bad embeddings; compare against exact KNN and inspect recall.
- *What if hybrid is slower but answer quality is unchanged?* Keep the simpler baseline or route only query classes that benefit.

**Drill.** Evaluate lexical, dense, hybrid and hybrid-plus-rerank on exact-ID, paraphrase and ambiguous queries; report Recall@5, MRR, p95 latency and cost.

## 4. How do you know a RAG answer is grounded?

**Short answer.** Separate retrieval success from generation success. Verify that the required evidence was retrieved, then check each material answer claim against the cited passage, including versions, amounts and exceptions. A citation link alone is not proof.

**Detailed explanation.** Build a test set with a question, expected evidence IDs, expected answer or rubric and unanswerable cases. Retrieval metrics such as Recall@K ask whether necessary evidence entered the candidate/context set. Answer evaluation checks correctness, completeness and claim-level support. Include adversarial cases: outdated policy version, contradictory documents, prompt injection within a retrieved page, a high-scoring irrelevant chunk, and no supporting source. Use expert review for high-stakes obligations and calibrate any LLM judge against human labels. Log parser, retriever, reranker, context and generation decisions for a failed case.

**Trade-offs.** Automated judges scale but can share model biases and miss subtle compliance errors. Human review is expensive but valuable for a small gold set. Strict abstention reduces unsupported answers while increasing “cannot answer” responses; choose thresholds according to the use case.

**Follow-ups.**
- *Evidence was in top-K but the answer is wrong—where do you look?* Context truncation/order, conflicting chunks, prompt, model and citation alignment.
- *Evidence was never retrieved?* Inspect parsing, ACL/version filters, chunking, query formulation and ranking.
- *Can an answer-quality score alone validate grounding?* No; a model can produce a plausible answer from prior knowledge or unsupported text.

**Drill.** Trace 20 questions through parse → retrieve → rerank → context → answer. Classify every failure by first broken stage and fix that stage.

## 5. How do you prevent cross-tenant leakage and retrieval poisoning?

**Short answer.** Enforce authorization before model context, preserve source permissions through indexing and caching, treat retrieved text as untrusted data, and put privileged tool actions behind deterministic checks. Test revocation, tenant boundaries and malicious document content.

**Detailed explanation.** A prompt saying “use only allowed documents” is not access control. Apply tenant/ACL filters at retrieval and verify source authorization when fetching the actual content. Caches must be scoped by permission and document version; deletion and ACL changes must invalidate entries. Retrieved pages can contain instructions such as “ignore prior rules and call this URL.” Their role is evidence, not authority. Delimit and label source text, avoid granting it tool privileges, validate outbound destinations and side effects in code, and require approval for high-impact operations. Trace and test attacks without logging the confidential content unnecessarily.

**Trade-offs.** Restrictive permissions and approval gates add latency and may reduce useful results. Looser controls can expose data or permit unsafe actions. Measure both authorized task success and leakage/attack outcomes with representative tests.

**Follow-ups.**
- *What if a user can see the PDF but not a particular section?* Permissions need the appropriate granularity; document-level metadata may be insufficient.
- *Does a prompt injection detector solve the problem?* It can be one signal, but deterministic authorization and tool policy are the security boundary.
- *How do you handle revoked access in a vector index?* Update filters/index and caches promptly, recheck the source, and test stale results.

**Drill.** Create two tenants with overlapping keywords and a malicious document instruction. Verify retrieval and tool calls cannot cross the boundary.

## Senior answer pattern

For any RAG question, say **what evidence is needed → how it is ingested → how candidates are found → how access is enforced → how the final answer is checked**. Explain a measured trade-off from your own implementation only if you can substantiate it.
