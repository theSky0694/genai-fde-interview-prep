# Batch 009 — LangGraph coding, inference fundamentals and evidence-led project defence

**Collected:** 23 October 2026. **Readiness:** Untested.

## Evidence note

This batch uses three independently posted reports:

1. [EPAM Systems Senior AI Engineer report](https://www.glassdoor.com/Interview/EPAM-Systems-Interview-Questions-E15544.htm), posted **22 October 2026**, for a recruiter-led process in Hyderabad. The candidate reports LangGraph coding and deep checks on transformer architecture, KV cache, self-attention and LSTM-versus-transformer differences. The source also says “abstractor method coding”; that ambiguous phrase is deliberately excluded rather than reinterpreted.
2. [Wells Fargo AI Engineer report](https://static.glassdoor.sg/Interview/Wells-Fargo-Interview-Questions-E8876.htm), posted **22 October 2026**, for Bengaluru. The candidate reports two two-person panel rounds, with most discussion focused on a project from their previous organization. No unreported project topic or exact question is inferred.
3. [Bharath Reddy's interviewer account](https://www.linkedin.com/posts/bharath-reddy-35a405195_i-interviewed-an-ai-engineer-recently-his-activity-7502908588672303104-hfBa), published roughly **late September 2026** and newly discovered this week. The post quotes a sequence about improving RAG accuracy, proving improvement, choosing metrics and creating a test dataset. It also promotes a paid interview kit, so only the visible firsthand interviewer claims are used.

These three independent accounts increase the reported-experience counter by three. They are useful preparation evidence, not a representative market sample.

---

## 1. Implement a small stateful LangGraph workflow live

**Source:** EPAM Senior AI Engineer report; inferred exercise from the reported “LangGraph coding” topic, not claimed verbatim wording. **Roadmap:** Stage 5. **Priority:** P0.

**What it tests:** Whether graph terminology translates into correct state, routing and failure behavior.

**Senior answer outline:**
- Define a typed state containing the user request, validated inputs, retrieved evidence, tool result, attempt count, error and final response.
- Keep nodes narrow and deterministic where possible: validate, retrieve/call tool, assess result, generate, approve or stop.
- Route with explicit predicates rather than asking the model to decide every transition.
- Bound retries and preserve idempotency for external writes. Checkpoint only the state required for safe resume, excluding secrets and unnecessary payloads.
- Attach trace IDs, node latency, token use and outcome to every transition; make the terminal failure state explicit.

**Trade-off:** LangGraph adds resumability and observable control flow, but a plain function or SDK loop is easier for short linear tool use.

**Follow-up / drill:** Implement a four-node bank policy assistant with one retry, a human-approval interrupt and resume after simulated retrieval failure.

---

## 2. Explain self-attention, LSTM versus transformer, and the KV cache as one inference story

**Source:** EPAM Senior AI Engineer report; close paraphrase of listed topics. **Roadmap:** Stage 2 and inference extension. **Priority:** P1.

**What it tests:** Conceptual depth tied to deployment consequences rather than memorized definitions.

**Senior answer outline:**
- An LSTM processes tokens recurrently and carries compressed hidden state; this encodes sequence order naturally but limits training parallelism and makes long-range information harder to preserve.
- A transformer processes token representations with attention, allowing each position to combine information from other positions. Positional information is added separately.
- Self-attention computes queries, keys and values; attention weights select which prior representations contribute to the current one.
- During autoregressive decoding, cache earlier keys and values so they are not recomputed for every new token. Memory then grows with layers, sequence length, batch and KV-head dimensions.
- Connect this to production: long contexts reduce concurrency; continuous batching, paging, prefix reuse, eviction and GQA/MQA affect latency and capacity.

**Trade-off:** Transformers parallelize training and handle long-range dependencies well, but attention/context and KV-cache memory create serving costs.

**Follow-up / drill:** Explain why doubling context length can reduce concurrent users on the same GPU, without deriving attention kernels.

---

## 3. Improve RAG accuracy—and prove which component caused the improvement

**Source:** Bharath Reddy interviewer account; direct-question sequence condensed. **Roadmap:** Stages 3 and 4. **Priority:** P0.

**What it tests:** Experimental discipline across retrieval and generation.

**Senior answer outline:**
- Begin with labeled failure categories: ingestion/parsing, retrieval miss, poor ranking, context assembly, stale/contradictory evidence, unsupported generation or evaluation error.
- Freeze a representative evaluation set and current baseline before changing anything.
- Measure retrieval independently with Recall@K, MRR or nDCG, then answer quality with correctness, groundedness/citation support, completeness and abstention behavior.
- Change one variable at a time—chunking, embedding, hybrid search, reranking, prompt or model—and report slice-level confidence intervals where practical.
- Confirm improvements against latency, cost and regressions; use online feedback only after an offline acceptance gate.

**Trade-off:** Better retrieval metrics do not guarantee better answers; adding reranking or larger contexts can improve recall while increasing latency or distracting generation.

**Follow-up / drill:** Compare two chunking strategies on 30 bank-policy questions with known supporting passages; report retrieval and answer metrics separately.

---

## 4. Build a trustworthy RAG evaluation dataset

**Source:** Bharath Reddy interviewer account; direct question. **Roadmap:** Stage 4. **Priority:** P0.

**What it tests:** Ground-truth design and evaluation leakage control.

**Senior answer outline:**
- Define the intended users, supported query types, corpus version and expected answer/abstention contract.
- Sample real, privacy-safe questions across products, languages, difficulty and frequency; add adversarial, unanswerable, stale-policy and multi-document cases.
- For each question record acceptable answers, supporting document/page/chunk IDs, required facts and known distractors. Have a domain reviewer adjudicate ambiguous items.
- Split by source document, time or product family to avoid near-duplicate leakage. Version questions and evidence with the corpus.
- Track coverage and label disagreement. Keep a small locked acceptance set separate from development and prompt tuning.

**Trade-off:** Synthetic questions scale cheaply but often mirror the generator and miss real ambiguity; production questions are realistic but require privacy controls and costly labeling.

**Follow-up / drill:** Create ten evaluation records from one bank-policy PDF, including two unanswerable and two multi-section questions, with page-level evidence.

---

## 5. Defend one production AI project end to end

**Source:** Wells Fargo Bengaluru AI Engineer report; inferred drill from the reported project-centred panels. **Roadmap:** Stage 7 / FDE narrative. **Priority:** P0.

**What it tests:** Ownership, architecture judgment, evidence and communication across technical and leadership panels.

**Senior answer outline:**
- Start with user problem, constraints and measurable outcome—not the framework list.
- Explain the architecture and data flow, then justify where deterministic services, RAG, model judgment, tools and human approval sit.
- State scale and ownership precisely: users, request volume, latency target, models, deployment boundary, what you personally designed and what the team owned.
- Present one production failure: signal, root cause, containment, permanent fix and what monitoring or process changed.
- Cover security, audit, evaluation, cost and alternatives rejected. Finish with current limitations and the next improvement.

**Trade-off:** Excessive implementation detail loses leadership interviewers; vague claims fail technical follow-ups. Use a layered two-minute summary with deeper evidence ready.

**Follow-up / drill:** Prepare a two-minute and ten-minute version of the 1,000-user bank TUI agent story, including one metric, one failure and one disputed design decision.

---

## Study order

1. Prepare the bank-agent project defence first; it turns Aakash's strongest experience into reusable interview evidence.
2. Implement the small LangGraph workflow.
3. Create the ten-record RAG evaluation set and use it to practise Questions 3 and 4.
4. Keep the transformer/KV-cache answer to interview depth rather than GPU-kernel depth.

No readiness status changes until Aakash answers or implements the drills.
