# Batch 001 — RAG, Retrieval, LangGraph & GraphRAG

Source: user-supplied reported interview experiences, ingested 22 Sep 2026.

## Evidence summary

This batch strongly clusters around retrieval engineering: embeddings, chunking, hybrid retrieval, BM25, RRF, multi-stage retrieval, distributed evidence across pages, graph retrieval, LangGraph orchestration and MCP design choices.

## Interview answers

### What are embeddings?
**30-second answer:** An embedding is a dense numerical vector representing an item such as text so that semantic relatedness can be measured geometrically. Similar meanings tend to map to nearby vectors, which enables semantic retrieval, clustering and recommendation.

**Direction vs magnitude:** Direction generally carries the semantic pattern used by similarity measures such as cosine similarity. Magnitude is model- and normalization-dependent; do not claim it universally represents “importance.” With normalized vectors, magnitude is fixed and cosine similarity effectively compares direction.

**Resources:** OpenAI embedding model docs: https://developers.openai.com/api/docs/models/text-embedding-3-large

### What is RAG and how does it work?
**30-second answer:** Retrieval-Augmented Generation retrieves external evidence relevant to a query and supplies that evidence as context to a generative model. A production flow is typically ingest → parse → chunk → enrich metadata → embed/index → query understanding → retrieve → fuse/rerank → context assembly → grounded generation → citations/evaluation.

**Why it exists:** It lets applications use current/private/domain data without encoding all of it into model weights and provides evidence that can be inspected.

**Resources:** Microsoft RAG overview: https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview

### Chunking strategies
Know fixed-size/token chunks, overlap, sentence/paragraph chunks, recursive splitting, semantic chunking, and structure-aware/hierarchical chunking. For legal/long structured documents prefer preserving headings, clauses, page/section identifiers and parent-child relationships rather than blindly slicing every N tokens.

Chunking is an optimization problem: too large hurts retrieval precision/context budget; too small loses surrounding meaning.

### 10 MB PDF: relevant information distributed across pages/partitions
Do not solve this by increasing top-k alone.

A strong design:
1. parse document structure and preserve document/page/section/parent IDs;
2. create retrieval-sized child chunks while retaining parent/section metadata;
3. run hybrid retrieval (lexical + dense);
4. fuse candidates (e.g. RRF);
5. rerank;
6. expand around strong hits using adjacent chunks, parent section, cross-references or graph relationships;
7. deduplicate and pack context under the token budget;
8. generate only from supplied evidence with citations;
9. evaluate retrieval separately from generation.

For questions requiring evidence scattered across sections, query decomposition/multi-hop retrieval may retrieve each fact independently and then combine evidence.

**Resources:** Microsoft retrieval guidance: https://learn.microsoft.com/azure/architecture/ai-ml/guide/rag/rag-information-retrieval

### Why BM25 along with vector retrieval?
BM25 is strong for exact lexical signals—identifiers, product names, error codes and rare terms. Dense retrieval captures semantic similarity when wording differs. Their failure modes are complementary, so hybrid retrieval often improves recall.

Do not add raw BM25 and cosine scores directly: their scales are not inherently comparable. Fuse ranked lists using RRF or normalize/calibrate scores, then optionally apply a reranker.

**Resources:** Elasticsearch RRF: https://www.elastic.co/docs/reference/elasticsearch/rest-apis/reciprocal-rank-fusion ; Microsoft RRF: https://learn.microsoft.com/azure/search/hybrid-search-ranking

### What is RRF?
Reciprocal Rank Fusion combines ranked result lists using rank positions rather than incomparable raw scores. A common form is sum(1/(k + rank)) across lists. Documents ranked highly by multiple retrievers rise naturally.

### Dense vs sparse retrieval
Sparse retrieval represents/query-matches using sparse term features and is strong for exact lexical matching; BM25 is the classic example. Dense retrieval maps text to learned vectors and retrieves by vector similarity, capturing semantic relationships. Hybrid systems use both.

### Why multiple retrieval nodes?
Use separate nodes when retrieval sources/strategies have different latency, failure handling, filters or routing requirements—for example vector search, lexical search, SQL and graph traversal. This makes routing, observability, retries and evaluation explicit. Do not split nodes merely for architectural decoration.

### Why LangGraph?
A defensible answer is control, state and reliability—not “because agents are popular.” LangGraph is useful when a workflow needs explicit multi-step state, conditional routing, durable execution/checkpointing, retries, human intervention or inspectable boundaries between retrieval/tool/reasoning steps.

**Resources:** LangGraph learning docs: https://docs.langchain.com/oss/python/learn ; LangGraph design guide: https://docs.langchain.com/oss/javascript/langgraph/thinking-in-langgraph

### What are LangGraph nodes?
Nodes are discrete units of work in a graph. They receive current state, perform an LLM/data/action/user-input step, and return state updates and/or routing decisions. Edges/control flow determine what runs next.

### Why Neo4j?
Use Neo4j when relationships are first-class retrieval evidence: multi-hop relationships, dependency chains, entity neighborhoods, hierarchies and connected facts that vector similarity alone may miss. Do not use a graph database just because GraphRAG exists; for simple independent document lookup, vector/keyword retrieval may be simpler.

**Resources:** Neo4j GraphRAG docs: https://neo4j.com/docs/neo4j-graphrag-python/current/ ; Neo4j GraphRAG guide: https://neo4j.com/developer/genai-ecosystem/

### Complete retrieval workflow
Query → intent/query analysis → optional decomposition/routing → parallel dense/BM25/graph/structured retrieval → metadata/security filtering → candidate fusion → reranking → context expansion → deduplication/context packing → generation with evidence → citations → tracing/evaluation.

### Why didn't I use MCP?
MCP solves standardized context/tool integration; it is not required merely because an application has tools. A good answer: direct in-process/API integrations were simpler when tools were application-specific and controlled by one service. MCP becomes more valuable when tools/resources must be reusable/discoverable across multiple hosts/agents or when standard interoperability outweighs added protocol/operational complexity.

**Resources:** MCP architecture: https://modelcontextprotocol.io/specification/2025-11-25/architecture
