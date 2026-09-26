# Transformer essentials for an application engineer

[Guide home](README.md) · Original, focused explanation informed by the topics in the [source transformer collection](https://github.com/Nareshedagotti/AI-Engineer-Interview-QA/blob/main/TRANSFORMERS_QA.md). **Timebox:** about 60–90 minutes for the current GenAI Engineer/FDE target. **Readiness:** Untested.

The source contains deep architecture and training questions. Your main interview target is to build, evaluate and operate LLM applications. Study the three answers below first. Derivations, gradient mechanics, expert sharding and architecture research become relevant only if a particular job description emphasizes model development.

## 1. What does attention do?

**Short answer.** Attention lets each token build a representation using information from other relevant tokens in the allowed context. In a decoder-only generator, a causal mask prevents a token from looking ahead at future tokens while predicting the next one.

**Detailed explanation.** The model converts tokens into vectors and computes how strongly positions should influence one another. Query, key and value projections are learned roles: a query seeks information, keys are matched against it, and values contribute information to the resulting representation. Multiple attention heads can learn different relationships. Positional information is needed because token order matters. This is a *mechanism for mixing context*, not a guaranteed fact-checker or proof that every supplied passage will be used.

**Practical consequence.** Long context increases computation and may contain distractions. For a RAG application, retrieval quality, placement, formatting and evaluation matter even if the model technically accepts all passages.

**Follow-ups.**
- *Why can a model ignore a relevant retrieved passage?* It may be buried among distractors, conflict with stronger patterns, be truncated or be poorly formatted; inspect the actual model input and test evidence-use behavior.
- *Does attention weight tell you why an answer is true?* Not by itself. It is an internal signal, not a reliable claim-level explanation.
- *Why does token order matter?* “Bank approved the transfer” and “transfer approved the bank” contain similar tokens but mean different things; position encoding supplies order information.

**Trade-off.** Larger context can improve evidence coverage while raising latency/cost. Better selection and structure may beat simply increasing the window.

## 2. How do encoder, decoder and bi-encoder/cross-encoder roles relate to RAG?

**Short answer.** An encoder represents an input for understanding or search; a decoder generates tokens autoregressively. A bi-encoder embeds query and documents separately for scalable first-stage retrieval. A cross-encoder considers query and candidate together for more precise but slower reranking.

**Detailed explanation.** A vector retriever precomputes document embeddings, then embeds each query and searches for neighbors. This is efficient for a large corpus but loses some fine-grained query-document interaction. A cross-encoder scores each query/candidate pair jointly and can pay closer attention to exact relevance; because every pair needs a forward pass, use it after a fast retriever narrows the list. The generation model then consumes selected evidence and produces an answer. Different models can be used at each stage; there is no rule that the generator and embedding model must be from the same family. Query and document embeddings **within one retrieval index**, however, must be compatible.

**Follow-ups.**
- *Why not cross-encode every document?* Pairwise inference is expensive at corpus scale.
- *Can a reranker recover a missing gold document?* No; it only reorders supplied candidates.
- *What happens when the embedding model changes?* Rebuild/version the document index and compare retrieval on labeled queries before switching.

**Trade-off.** More reranking depth may improve quality but adds latency; tune candidate count against Recall@K, answer support and p95 time.

## 3. What should you know about long context and inference cost?

**Short answer.** Input tokens, output tokens, context length, model size, concurrency and serving strategy all affect latency and cost. Time to first token and generation throughput are distinct measurements. Quantization and batching change resource trade-offs and require quality testing.

**Detailed explanation.** A prompt with long retrieved passages spends compute before the first generated token. Decoder generation proceeds token by token; long outputs add wall-clock time. KV caching avoids recomputing previous attention state during generation, but consumes memory as concurrent requests and context lengths grow. Batching improves GPU utilization, though queueing can harm an individual interactive request. Quantization reduces weight memory and may enable a local model on available GPUs; it can alter accuracy, tool-call quality or throughput depending on hardware and implementation. Measure the complete workload instead of assuming one technique universally helps.

**Follow-ups.**
- *Does a larger context window mean you should stuff in every document?* No. Retrieval and context selection protect quality and latency.
- *How would you compare a local 8B model with a larger one?* Same task set and load profile; compare grounded success, valid tool calls, TTFT, throughput, p95 latency, GPU memory and operational cost.
- *Is temperature zero deterministic?* Not a complete guarantee; server implementation, parallelism, model/version and changing retrieval/tool state can change outcomes.

**Trade-off.** Higher throughput via batching may worsen per-request wait. Smaller/quantized models reduce resources but must be tested on difficult cases.

## Stop point

You can answer these three at an application/system-design level and return to Python, RAG and agents. Go deeper into scaled dot-product equations, positional variants or training objectives only when a target job specifically tests model internals.
