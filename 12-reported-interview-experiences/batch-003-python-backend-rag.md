# Batch 003 — Python, Backend, RAG & Agents

Source: user-supplied reported MNC GenAI/Python interview experience, ingested 22 Sep 2026.

### Async: when and when not?
Use async for high-concurrency I/O-bound work where dependencies expose non-blocking APIs: network calls, async DB drivers, parallel retrieval/model calls. It improves utilization while tasks wait. Do not expect async to speed CPU-bound work; use processes/workers/appropriate compute. Avoid async when dependencies are blocking or complexity provides no concurrency benefit.

Resource: https://fastapi.tiangolo.com/async/

### FastAPI benefits
Python type-hint-driven request/response validation, OpenAPI generation, async support, dependency injection, good performance and strong Pydantic integration. In interviews, connect features to production concerns rather than simply listing them.

Resource: https://fastapi.tiangolo.com/

### Database connection string
A DB URL normally identifies dialect/driver, credentials, host/port and database plus options. Credentials should come from secrets/config, not source control.

Resource: https://docs.sqlalchemy.org/en/21/tutorial/engine.html

### Multiple users and DB connections
Do not create a permanent DB connection per application user. Create a process-level Engine/pool; each request/session checks out a connection for a short unit of work/transaction and returns it. Size pool/overflow/timeouts for workload and database limits.

### SQLAlchemy connection pool
The Engine owns/uses a connection pool; QueuePool is typical for many server DB dialects. Connections are checked out, used, and returned for reuse. Understand pool_size, max_overflow, timeout, recycling/pre-ping and transaction/session lifecycle.

Resource: https://docs.sqlalchemy.org/en/21/core/engines.html

### Deterministic/consistent LLM output
Start with deterministic application design: stable model/version, stable prompts/context/order, constrained structured output, low/appropriate sampling, fixed tools and deterministic retrieval. Use provider seed only when supported and understand it may not guarantee bit-identical results. For factual applications, consistency should come from grounding/validation/evaluation rather than temperature alone.

### Conversation memory
Useful categories: short-term/thread state, sliding-window/recent messages, summaries, semantic/episodic long-term memory, structured user/task state. Keep only information needed for the current decision; retrieve long-term memories on demand.

### Long history
Use bounded recent history + structured state + selective retrieval of older relevant turns + summaries/checkpoints. Avoid repeatedly sending the entire transcript.

### Doesn't summarization increase cost?
Yes. Summarization is worthwhile only if its one-time/periodic cost saves more repeated input tokens or improves context quality across later turns. Summarize incrementally, cache summaries and trigger compaction by thresholds rather than summarizing every request.

### Agent gives a wrong answer: identify and correct
Trace the execution first: user input → routing/planning → retrieval/tool arguments → tool output → state → prompt → model output. Classify whether failure is retrieval, tool, reasoning, stale data, prompt, policy or generation. Reproduce with a test case, fix the responsible component, add the case to an evaluation/regression suite, and monitor the relevant metric. Do not respond by blindly changing the prompt.

### Maximum subarray sum
Expected approach: Kadane's algorithm.

Maintain:
- current_best_ending_here = max(x, current + x)
- global_best = max(global_best, current_best_ending_here)

Initialize from the first element so all-negative arrays work.

Complexity: O(n) time, O(1) extra space.

Follow-ups to prepare:
- return start/end indices
- explain why resetting works
- all-negative input
- empty-input contract
- brute force vs Kadane
