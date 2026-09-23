# Batch 004 — RAG Follow-up Interview Drill

Source: user-supplied LinkedIn interview-preparation post, ingested 23 Sep 2026.

> Evidence note: this is an interview-preparation post, not a first-person interview report. It strengthens preparation relevance but is not counted as an additional independent interview experience.

## 1. Why RAG instead of fine-tuning?

**30-second answer:** I choose RAG when the problem is primarily knowledge access rather than changing model behavior. If knowledge is private, frequently updated, large, or answers need citations/provenance, retrieval keeps knowledge outside model weights and lets us update the corpus without retraining. Fine-tuning is more appropriate when I need to change behavior, style, task performance or response format consistently. They are complementary rather than mutually exclusive.

**Follow-ups:** When would you fine-tune? Can RAG and fine-tuning coexist? What are RAG's latency/failure costs?

## 2. How did you decide chunk size?

**30-second answer:** I don't start with an arbitrary token count. I inspect document structure and query patterns, choose boundaries that preserve semantic units, then tune size/overlap against retrieval metrics and downstream answer quality. I also preserve section/page/parent metadata so a small retrieval chunk can expand to useful surrounding context.

**Trade-off:** Smaller chunks can improve retrieval precision but lose context; larger chunks preserve context but may introduce noise and consume context-window budget.

## 3. Which retrieval method did you use?

Compare:
- sparse/BM25 — exact terms, IDs, names, jargon;
- dense/vector — semantic similarity and paraphrases;
- hybrid — complementary lexical + semantic recall;
- metadata filters — constrain by tenant, date, type, ACL, product, etc.;
- reranking — improve precision after broad candidate retrieval.

**Strong project answer:** explain the data/query failure modes that motivated the selected combination rather than naming a fashionable retriever.

## 4. How did you evaluate retrieval quality?

With labeled relevant documents/chunks:
- **Precision@K:** fraction of top-K retrieved items that are relevant.
- **Recall@K:** fraction of all known relevant items found in top K.
- **MRR:** rewards placing the first relevant result early.
- **NDCG@K:** measures ranking quality when relevance can be graded and position matters.

Also evaluate contextual relevance and downstream groundedness separately. A good final answer does not prove retrieval was good—the model can sometimes answer despite poor evidence.

Resources:
- https://learn.microsoft.com/azure/architecture/ai-ml/guide/rag/rag-information-retrieval
- https://learn.microsoft.com/en-us/azure/foundry/concepts/evaluation-evaluators/rag-evaluators

## 5. How did you reduce hallucinations?

Treat hallucination as a system problem:
1. improve retrieval recall;
2. rerank for precision;
3. filter/validate evidence;
4. assemble focused context;
5. explicitly require evidence-grounded answers and citations;
6. detect insufficient evidence and abstain/clarify;
7. evaluate groundedness and regression-test failures.

A better prompt alone cannot repair missing/wrong retrieval.

## 6. What if the correct document is not retrieved?

First identify whether failure is query understanding, filtering, indexing/chunking or ranking. Depending on the cause:
- rewrite/expand/decompose query;
- hybrid retrieval;
- broaden candidate count;
- reconsider filters/thresholds;
- search alternate source/index;
- retrieve related parent/adjacent chunks;
- ask a clarifying question;
- abstain with insufficient evidence.

Do not blindly lower thresholds: that can increase noise.

## 7. How do you debug a plausible but wrong RAG answer?

Trace each stage with the same request:

**Query understanding → Retrieval → Fusion → Reranking → Context assembly → Generation → Validation**

Ask:
1. Was the query interpreted correctly?
2. Was the required evidence indexed?
3. Did retrieval return it?
4. Did fusion/reranking demote it?
5. Was it dropped during context packing?
6. Did the model contradict/misread valid context?
7. Did validation/evaluation fail to catch it?

Persist retrieved document IDs/scores, reranking results, prompt/context, model/version and trace/correlation ID (subject to privacy/security policy). Convert the incident into a regression test.

## Interview pattern

The important lesson is that interviewers can start with “What is RAG?” and immediately move into design justification, evaluation and debugging. Preparation must therefore cover the complete retrieval lifecycle rather than memorized definitions.

## Authoritative resources

- Microsoft RAG architecture/design: https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/rag/rag-solution-design-and-evaluation-guide
- Microsoft information retrieval guidance: https://learn.microsoft.com/azure/architecture/ai-ml/guide/rag/rag-information-retrieval
- Microsoft RAG evaluators: https://learn.microsoft.com/en-us/azure/foundry/concepts/evaluation-evaluators/rag-evaluators
- Azure hybrid search/RRF: https://learn.microsoft.com/azure/search/hybrid-search-ranking
