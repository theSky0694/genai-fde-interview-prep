# Start here: GenAI concepts from zero (for an experienced API engineer)

**Audience:** strong MuleSoft, Spring Boot, APIs and integration background; new to GenAI. No assumed knowledge of transformers, embeddings, vector DBs or LangGraph.

## The mental model: how a GenAI application grows
1. **LLM**: a model that predicts useful next tokens from the prompt. It is not your database and may produce plausible mistakes. Think of it as a text reasoning/generation service, not an authoritative system of record.
2. **Prompt and context**: instructions plus data sent with the request. The context window is finite; adding irrelevant text can hurt quality and cost.
3. **RAG**: before calling the model, search trusted documents for relevant passages; include those passages in the prompt and cite them. Like an API orchestration flow: fetch data, transform it, then invoke a downstream service.
4. **Chunk**: a small, retrievable unit of document text. Keep a heading, page, document ID and permissions with each chunk. Avoid splitting a policy exception away from its rule.
5. **Embedding**: a model-produced list of numbers representing semantic meaning. Similar phrases can be near one another even with different words. A vector database indexes and searches these numbers; it does not generate answers.
6. **BM25 / lexical search**: scores text using exact terms, their frequency and rarity. Excellent for account codes, policy IDs and proper nouns; it can miss paraphrases.
7. **Hybrid retrieval**: use both keyword and semantic search. Merge rankings (often reciprocal rank fusion, RRF) and optionally rerank with a stronger model.
8. **Evaluation**: test *retrieval* separately from *generation*. Recall@k asks whether a relevant passage appeared in the top k; groundedness checks whether the answer is supported by retrieved evidence.
9. **Tool calling**: the LLM proposes a structured call, but your application validates permissions, arguments, timeouts, retries and the tool result. A proposed tool call is not permission to execute.
10. **Agent / LangGraph**: an agent selects next actions based on state and observations; LangGraph represents steps as nodes, transitions as edges, and shared data as state. Start with deterministic workflows before introducing agent decisions.
11. **MCP**: a standard interface for exposing tools/resources to AI clients. Use when interoperability is valuable; direct Python functions may be simpler inside one service.
12. **Production**: auth, PII controls, logging without secrets, observability, cost, rate limits, failure handling, test sets and human review.

## Walk-through: why retrieval matters
A bank policy PDF says 'The daily transfer cap for Tier B is ₹75,000.' A user asks, 'How much may Tier B transfer today?'
- **Without RAG:** the model might guess based on generic knowledge.
- **With RAG:** ingestion extracts the policy, chunks it, records source metadata, indexes it. At query time retrieval finds the cap, the prompt instructs the model to use the excerpt, and the answer cites the policy.
- **Failure:** if the cap was in a table that extraction flattened incorrectly, the model cannot repair the missing fact. Fix extraction and retrieval first.

## Runnable mini-lab: keyword retrieval before embeddings
Create `mini_search.py` and run `python mini_search.py`. No GPU or API key needed.

```python
docs = [
    {"id": "policy-1", "text": "Tier B daily transfer limit is 75000 rupees."},
    {"id": "policy-2", "text": "Tier A daily transfer limit is 25000 rupees."},
    {"id": "policy-3", "text": "Reset your password in the employee portal."},
]
question = "Tier B transfer limit"
terms = set(question.lower().split())
ranked = sorted(
    docs,
    key=lambda d: len(terms & set(d["text"].lower().split())),
    reverse=True,
)
for hit in ranked[:2]:
    print(hit["id"], hit["text"])
```

**Expected:** policy-1 ranks first. **Why:** exact word overlap; this is a teaching baseline, *not BM25*. **Next experiment:** ask 'What is the maximum remittance for the second tier?' Exact overlap fails; semantic embeddings can help. Then test a query with a specific policy ID where exact matching is stronger.

## Learn in dependency order
| Order | Concept | Build in the project | Interview checkpoint |
|---|---|---|---|
| 1 | Python functions, dictionaries, FastAPI | /ask endpoint with hardcoded answer | Trace request/response |
| 2 | LLM, prompt, tokens, temperature | Call a model with explicit context | Explain hallucination |
| 3 | PDF extraction, chunking, metadata | Index a 3-page policy | Defend chunk boundaries |
| 4 | Embeddings, cosine similarity, lexical retrieval | Compare keyword vs semantic | Explain false positives |
| 5 | Hybrid search, RRF, reranking | Add ranked search results | Explain trade-offs |
| 6 | Grounded answers, citations, evaluation | Answer with source IDs | Diagnose wrong answer |
| 7 | Tools, LangGraph state/nodes/edges | Add retrieval and escalation flow | Defend agent vs workflow |
| 8 | MCP, permissions, observability, deployment | Secure tool gateway | Discuss enterprise failures |

## Outside explanations — blogs and practitioner tutorials (not only official docs)
- Pranay Joshi, **Understanding Chunking Strategies in RAG** (Medium, Feb 2026): https://medium.com/@jpranay7828/understanding-chunking-strategies-in-retrieval-augmented-generation-rag-a-complete-practical-a1cd8cb346d5 — compare splitting strategies and failure modes.
- Abhinav Singh, **Smarter Search Starts with Smarter Chunks** (Medium, Apr 2026): https://medium.com/@iamabhinav30/smarter-search-starts-with-smarter-chunks-55b9c0ca0b1d — tokenization, metadata and retrieval decisions.
- Lamhot Siagian, **The RAG Engineering Blueprint** (Medium, Feb 2026): https://medium.com/@lamhot.siagian/the-rag-engineering-blueprint-from-semantic-chunking-to-grounded-answers-0f3444b337c1 — end-to-end RAG and evaluation.
- Riad Bensalem, **My First AI Agent with LangGraph, Gemini & RAG** (LinkedIn, Apr 2025): https://www.linkedin.com/pulse/from-overwhelm-action-my-first-ai-agent-langgraph-gemini-bensalem-iqwue — approachable first project.
- Hiram Reis Neto, **Build a LangGraph Retrieval Agent** (LinkedIn, Apr 2025): https://www.linkedin.com/pulse/hands-on-how-build-langgraph-retrieval-agent-tutorial-hiram-reis-neto-iruff — retrieval + nodes + relevance grading (TypeScript example; translate ideas into Python).
- Ishan Rastogi, **Agentic AI Engineer — Implementation Foundations, Part 1** (LinkedIn, Jul 2026): https://www.linkedin.com/pulse/agentic-ai-engineer-implementation-foundations-rag-mcp-rastogi-tlxwc — concepts before orchestration.
- Saurav Prateek, **Building Agentic RAG from Scratch** (LinkedIn, Oct 2024): https://www.linkedin.com/pulse/building-agentic-rag-from-scratch-youtube-playlist-saurav-prateek-tfunc — progressive project episodes.

**Reading policy:** Blogs/Medium/LinkedIn are teaching resources, not necessarily evidence of interview questions or correctness. Cross-check API signatures against current official documentation and test code locally. Keep the original interview evidence citations separate.

## How to study each interview batch
Read the matching concept first, implement the project increment, explain it in plain words, *then* read the existing senior-level interview answer. Do not mark a topic ready just because you read it.
