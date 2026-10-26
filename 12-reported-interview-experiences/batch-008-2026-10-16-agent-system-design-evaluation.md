# Batch 008 — Agent system design, evaluation and India GenAI breadth

**Collected:** 16 October 2026. **Readiness:** Untested.

## Evidence note

This batch uses three independently posted candidate reports:

1. [IDFC FIRST Bank Gen AI Engineer report](https://www.glassdoor.com/Interview/IDFC-FIRST-Bank-Interview-Questions-E2883495.htm), posted **13 October 2026**, describing a September process in Noida. The candidate reports DSA followed by coverage of RAG, deep learning, ML/AI, transformer architecture, fine-tuning, Agentic AI, MCP, microservices and FastAPI. The source lists topics rather than exact technical wording, so Questions 3–5 are explicitly **inferred drills**, not claimed verbatim questions.
2. [Fireworks AI interview reports](https://www.glassdoor.com/Interview/Fireworks-AI-Interview-Questions-E9514416.htm). A **14 October 2026** Research candidate reports two AI-free coding rounds plus an agent-design case study. Question 2 comes from a separate **17 August 2026** AI Engineer report on the same page and is newly discovered older evidence.
3. [Anonymous startup GenAI Engineer account](https://www.reddit.com/r/leetcode/comments/1x1rrbm/just_had_the_fastest_genai_engineering_interview/), posted **15 October 2026**. The candidate reports receiving a multi-agent architecture problem with roughly ten minutes to design it. No company name is supplied, so none is inferred.

These three reports increase the independent-experience counter by three. They are useful signals, not a representative hiring-market sample.

---

## 1. Design a multi-agent system in ten minutes

**Source:** Anonymous startup GenAI Engineer account; close paraphrase. **Roadmap:** Stages 5 and 7. **Priority:** P0.

**What it tests:** Rapid requirements framing, orchestration boundaries and production trade-offs under ambiguity.

**Senior answer outline:**
- Spend the first minute fixing the goal, users, inputs, success metric, latency, cost, data sensitivity and allowed actions.
- Begin with the smallest deterministic workflow. Introduce specialist agents only when tasks require different context, tools, permissions or parallel exploration.
- Use a typed shared state and explicit coordinator. Define each worker's input/output schema, deadline, retry policy and completion contract.
- Put external effects behind idempotent tools with authorization, timeouts and audit events. Add loop/step/token budgets and a human gate for consequential writes.
- Evaluate end-to-end task success plus routing accuracy, tool errors, latency, cost and correlated failures; provide a degraded single-agent or rules-based path.

**Trade-off:** Parallel workers can improve coverage and latency, but increase cost, coordination overhead and correlated-error risk.

**Follow-up / drill:** Deliver a ten-minute design for a bank dispute-investigation workflow: two minutes for requirements, four for architecture, two for failures/security and two for evaluation.

---

## 2. Extend an LLM system to a new dataset and optimize its latency

**Source:** Fireworks AI, AI Engineer report dated 17 August 2026; direct question combines dataset extension, evaluation, model choice and latency. Newly discovered older evidence. **Roadmap:** Stages 4 and 7. **Priority:** P0.

**What it tests:** Evaluation design, distribution shift, defensible model selection and production optimization.

**Senior answer outline:**
- Characterize the new dataset: schema, language, domain, label definition, quality, imbalance, privacy and expected shift from the original distribution.
- Freeze a representative test set with important slices and failure cases. Choose task metrics plus business harm, calibration, latency and cost; do not rely only on an LLM judge.
- Re-establish simple baselines, then change one layer at a time: prompt, retrieval/index, model or fine-tuning. Select the smallest model meeting the acceptance threshold.
- Diagnose latency into retrieval, queueing, prefill, decoding, tool calls and network time. Apply caching, batching, smaller contexts, reranking limits, streaming and model routing where measurements justify them.
- Shadow-run, canary and monitor slice regressions, drift, cost and rollback criteria.

**Trade-off:** Aggressive caching or smaller contexts reduce latency and cost but can serve stale evidence or remove necessary information.

**Follow-up / drill:** Take the bank-assistant evaluation set, add one new product family and present the test slices, acceptance gates and p95 latency budget.

---

## 3. Design a production RAG service behind FastAPI and microservices

**Source:** IDFC FIRST Bank Gen AI Engineer report; **inferred drill** from the explicitly listed RAG, microservices and FastAPI topics—not interviewer wording. **Roadmap:** Stages 3, 4 and 7. **Priority:** P0.

**What it tests:** Whether RAG knowledge can be turned into an operable service.

**Senior answer outline:**
- Separate offline ingestion/versioning from online query serving. Parse, classify, chunk, embed and index asynchronously; publish an immutable corpus/index version.
- Keep the synchronous path bounded: authenticate, normalize query, retrieve, rerank, assemble context, generate, validate and cite.
- Use FastAPI with async I/O for network-bound dependencies, bounded connection pools, request deadlines and cancellation. Move CPU-heavy parsing and long ingestion to workers.
- Define stable contracts between retrieval and generation rather than splitting every step into a microservice. Propagate tenant, trace, corpus version and idempotency metadata.
- Support abstention and degraded retrieval; measure Recall@K, groundedness, p95 latency, empty retrieval, timeout rate and per-request cost.

**Trade-off:** More services provide independent scaling and ownership, but add network failures, contract evolution and operational burden.

**Follow-up / drill:** Implement one FastAPI endpoint with a total timeout budget, typed response citations and simulated vector-store/model failures.

---

## 4. Where should MCP sit in an enterprise agent architecture?

**Source:** IDFC FIRST Bank report; **inferred drill** from MCP and Agentic AI topics. **Roadmap:** Stage 6. **Priority:** P1.

**What it tests:** Protocol knowledge connected to authorization and service ownership.

**Senior answer outline:**
- Use MCP as a standardized discovery/invocation boundary when multiple compatible clients need reusable tools, resources or prompts.
- Keep domain logic and authorization in underlying services. The MCP server validates schemas and identity propagation but must not become the sole security boundary.
- Offer narrow business tools rather than arbitrary database or HTTP access. Separate read tools from writes; use approvals, scopes, allowlists and idempotency keys.
- Enforce timeouts, result-size limits, sanitization, audit logging and protocol/tool versioning. Treat tool output as untrusted data, not new instructions.
- Prefer direct SDK/API integration for a single tightly coupled low-latency application where MCP adds no interoperability value.

**Trade-off:** MCP improves portability and discovery but introduces another trust boundary, protocol lifecycle and observability layer.

**Follow-up / drill:** Draw both direct-call and MCP versions of a bank policy lookup and compare identity propagation, latency, failure ownership and audit evidence.

---

## 5. Choose between prompting, RAG and fine-tuning for a transformer system

**Source:** IDFC FIRST Bank report; **inferred drill** from transformer architecture and fine-tuning topics. **Roadmap:** Stages 2–4. **Priority:** P1.

**What it tests:** Model understanding without defaulting to the most expensive adaptation.

**Senior answer outline:**
- Use prompting for behavior expressible through instructions and examples; use RAG for private, current or auditable knowledge; use fine-tuning for stable repeated behavior, style, format or tool policy that prompting cannot reliably deliver.
- Fine-tuning changes weights but does not provide a trustworthy mechanism for frequently changing facts. RAG improves freshness and citations but adds retrieval failure modes and latency.
- Establish an evaluation set first. Compare a prompt baseline, RAG and only then PEFT/fine-tuning using quality, hallucination, latency, cost and maintainability.
- Explain transformer relevance at interview depth: attention builds contextual token representations; inference costs depend on context/prefill and autoregressive decoding; architecture informs limits but does not choose the product solution alone.
- Combine approaches when justified—for example, fine-tune structured tool behavior while retrieving current policy evidence.

**Trade-off:** Fine-tuning can reduce prompt size and stabilize behavior, but creates data, training, evaluation, deployment and rollback obligations.

**Follow-up / drill:** Classify ten requirements from the bank assistant into prompt, RAG, fine-tuning or deterministic code and defend each choice.

---

## Study order

1. Practise Question 1 as a strict ten-minute whiteboard exercise.
2. Implement the failure-aware FastAPI slice from Question 3.
3. Use Question 2 to turn that implementation into an evaluation and latency story.
4. Review Questions 4 and 5 as architecture decisions, not framework definitions.

No readiness status changes until Aakash answers or implements the drills.


## Beginner-first concept and project bridge (26 Oct 2026)

**Read first:** [Concept foundations](../00-genai-concept-foundations.md). The existing interview answers above are the *destination*, not the starting point.

### Concept explained — Multi-agent design, evaluation and enterprise boundaries
**Multi-agent** means multiple specialized decision-making components, not automatically higher accuracy. Begin with one deterministic pipeline. Split roles only when separate context, tools or responsibilities provide measurable value. **Evaluation** requires a dataset and clear metric; a demo that 'looks good' is not evidence of reliability. **Latency** includes retrieval, reranking, model and network time.

### Project exercise — learn by building
Measure a baseline /ask request: retrieval milliseconds, model milliseconds and total latency. Add a review node only for unsupported answers. Compare one workflow with a hypothetical planner+researcher+reviewer design; list the extra calls and failure paths.

### Interview follow-ups, with reasoning
Why might a single agent outperform three? Less orchestration overhead and fewer cascading errors. When is human approval required? Before consequential writes or uncertain high-impact decisions.

### Learning checkpoint
Explain the idea without jargon; run or design the exercise; show the expected observation and one failure mode; only then attempt the senior-level questions above. Do not change readiness to 'ready' merely because this was read.

**Practitioner explanation:** [Read the blog / LinkedIn tutorial](https://www.linkedin.com/pulse/building-human-in-the-loop-agents-langgraph-complete-roadmap-g8a0c). This is a teaching resource, **not** evidence that an interviewer asked a question. Also use the additional Medium and LinkedIn reading list in the concept foundations guide.
