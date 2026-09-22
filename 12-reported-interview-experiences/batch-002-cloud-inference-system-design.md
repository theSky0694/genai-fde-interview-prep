# Batch 002 — Cloud, Inference & GenAI System Design

Source: user-supplied reported interview experience, ingested 22 Sep 2026.

## High-priority question map

### Enterprise ingestion
Discuss connectors/events/batch ingestion, parsing, schema normalization, metadata/ACL preservation, deduplication, chunking, embedding/indexing, retries/idempotency, lineage, incremental updates and deletion propagation.

### Confidential documents / redaction
Defence in depth: classify data → enforce source ACLs during retrieval → redact/tokenize sensitive fields when appropriate → encrypt in transit/at rest → least-privilege identities → tenant isolation → audit access → control model/provider data handling → avoid sensitive trace/log leakage. Redaction is not a substitute for authorization.

### Legal-document chunking
Prefer structure-aware boundaries (document → chapter → clause → paragraph), preserve citations/page/section IDs, use parent-child retrieval, limited overlap, metadata filters and context expansion. Tables/footnotes/cross-references may require specialized parsing.

### RAG cost reduction
Measure first. Common levers: smaller embedding/generation models where quality permits, cache stable results, incremental rather than full re-indexing, route simple queries without expensive agent loops, retrieve/rerank before sending context, control top-k/context length, batch ingestion, summarize/cache reusable context, and use tiered model routing. Never optimize token cost while destroying retrieval recall.

### Temperature
Temperature changes sampling randomness; lower values generally make sampling less variable, but deterministic behaviour also depends on model/API semantics, seed support, model version, prompt, tools and infrastructure. Some reasoning models constrain temperature.

### MCP architecture
Host coordinates one or more clients; clients maintain connections to servers; servers expose capabilities such as tools/resources. MCP standardizes discovery/invocation/context exchange across integrations.

Resource: https://modelcontextprotocol.io/specification/2025-11-25/architecture

### A2A
A2A is an open protocol for communication/collaboration between agents. Distinguish it from MCP: MCP primarily standardizes agent/application access to tools/context; A2A focuses on agent-to-agent interoperability.

Resource: https://a2a-protocol.org/v1.0.0/

### Amazon Bedrock AgentCore
AgentCore is AWS infrastructure for building/deploying/operating agents, with services including Runtime and Gateway. Runtime supports framework/model flexibility; Gateway can expose tools through MCP. Note that Bedrock Agents Classic is now in maintenance mode for existing customers, so current preparation should emphasize AgentCore.

Resources: https://docs.aws.amazon.com/bedrock-agentcore/ ; https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/gateway-using.html

### Object storage vs SQL
Object storage: large immutable/semi-structured objects, documents, media, raw lake data, cheap durable storage. SQL: transactional structured data, constraints, joins, indexes and queryable relational state. A RAG ingestion architecture commonly stores originals in object storage and metadata/transactional state in SQL/search/vector systems.

### Data lake
Know object-storage-based central data layer, raw/curated zones, catalog/governance, ingestion (batch/stream), processing, query engines, security/lineage and lakehouse extensions.

### LLaMA architecture
Prepare decoder-only Transformer fundamentals plus LLaMA-specific choices (generation dependent): pre-normalization/RMSNorm, SwiGLU, RoPE, attention/KV-cache concepts; newer Llama generations can differ significantly, e.g. Llama 4 uses MoE and native multimodality. Always clarify which LLaMA generation the interviewer means.

Resource: https://huggingface.co/docs/transformers/main/en/model_doc/llama

### Benchmarking self-hosted inference vs AWS/Azure managed
Use the same model/precision/workload where possible. Measure quality plus TTFT, inter-token latency, end-to-end latency p50/p95/p99, throughput/tokens per second, concurrency, error rate, availability, GPU utilization, cold starts, max context, cost per request/token, operational effort, scaling behaviour and security/compliance. Load-test realistic prompt/output distributions.

### Inference microservice
API/auth → validation → admission/rate limiting → batching/scheduler → model runtime → streaming → metrics/tracing. Consider model loading, GPU memory, quantization, KV cache, timeouts/cancellation, autoscaling, graceful overload, model versioning and fallback.

### Quantization
Represent weights/activations at lower precision (e.g. 8/4-bit) to reduce memory and often improve inference efficiency, trading possible quality loss and hardware/kernel constraints. Compare PTQ vs QAT conceptually.

### Graph construction
Extract or ingest entities and relationships → normalize/entity-resolve → create nodes/edges/properties → attach provenance → index → query/traverse. In RAG, graphs help with multi-hop/relationship questions and can complement vector retrieval.

Resource: https://neo4j.com/docs/neo4j-graphrag-python/current/

### Observability / traceability
Trace each request across query transformation, retrieval, reranking, tools, prompts/model calls and output. Capture latency, tokens/cost, model/version, retrieved document IDs/scores, tool calls/errors, evaluation signals and correlation IDs while protecting sensitive data.

### CSV query solution
Clarify file size and query types. Small CSV: parse/dataframe. Large/repeated analytical workloads: ingest into analytical SQL/columnar engine. Natural-language querying should generate constrained/validated queries or code rather than embedding every cell blindly. Preserve schema/types and implement sandboxing/limits.

### Traditional ML topics
These reported questions add a genuine syllabus gap:
- Logistic regression + sigmoid
- SVM + kernel functions
- time-series fundamentals + first differencing

These should be prepared to interview depth, but not allowed to displace the core GenAI/FDE path unless further reports reinforce them.
