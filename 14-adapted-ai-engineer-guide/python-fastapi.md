# Production Python and FastAPI — interview answers

[Guide home](README.md) · Inspired by the topics in the [source FastAPI collection](https://github.com/Nareshedagotti/AI-Engineer-Interview-QA/blob/main/FASTAPI_QA.md). Original explanations and examples. **Roadmap:** Stage 1B. **Readiness:** Untested.

## 1. When does async help a GenAI API?

**Short answer.** Async helps a server serve other requests while this request waits for non-blocking I/O: model calls, database queries or tool APIs. It does not accelerate CPU-heavy parsing or embedding work by itself. The entire call path must cooperate; a blocking client inside `async def` can stall the event loop.

**Detailed explanation.** A FastAPI route awaiting an async HTTP client releases the event loop during network wait. Several requests can therefore be in flight on one worker. If the route calls a synchronous SDK or runs PDF/OCR processing directly, that work occupies the worker. Move blocking I/O to a suitable thread pool or use a supported async client; CPU-intensive work belongs in a bounded process/worker service or job queue. Async also requires **backpressure**: unlimited simultaneous calls can exhaust model slots, sockets, memory or provider quota. Use explicit concurrency limits, timeouts and cancellation handling.

**Bank-style example.** An assistant might retrieve from a search service and call a local model. Parallelize independent reads when permissions and request deadlines permit; do not run a write tool concurrently with an unvalidated decision.

**Trade-offs.** Sync code is simpler and can be adequate at modest load. Async increases concurrency for I/O, but introduces cancellation, pooling and race considerations. Compare p95 latency, throughput, queue length and downstream saturation under load rather than claiming async is always faster.

**Follow-ups.**
- *Can you use `requests.get()` inside `async def`?* It works functionally but blocks the event loop. Prefer an async client or isolate the blocking call.
- *Does `asyncio.gather` make three model calls free?* No. It may reduce wall-clock wait, while increasing token cost, GPU contention and rate-limit pressure. Bound concurrency.
- *What if the client disconnects?* Cancel work where safe, close streams, and avoid assuming an in-flight business write was rolled back.

**Drill.** Implement a route calling two stub APIs with configurable delays. Load-test sync blocking, proper async and bounded async versions; compare p95 latency and downstream concurrency.

## 2. How would you validate and version an LLM API?

**Short answer.** Define explicit request/response schemas, reject invalid or excessive input at the boundary, authenticate and authorize the caller, version externally visible contracts, and keep model or prompt versions separately traceable.

**Detailed explanation.** A request schema should specify required fields, allowed ranges (such as top-K), size limits, document IDs and stable error shapes. Pydantic validates shape; domain checks must still verify that the user may access the requested document and that a cited source exists. Separate transport schemas from internal LangGraph state so workflow changes do not accidentally break clients. Version the public API when response semantics or fields change. Track model, prompt, retriever and index versions in traces for regression analysis.

**Trade-offs.** Strict schemas catch mistakes early but may make client evolution harder; additive optional fields and explicit migration windows help. An apparently valid JSON response from an LLM is not proof that its facts or proposed actions are safe.

**Follow-ups.**
- *Why are type hints not enough?* They aid tooling, but runtime validation and domain authorization need explicit enforcement.
- *What would you return for an unsupported document ID?* A stable, non-leaking error; avoid revealing whether another tenant's document exists.
- *How do you handle schema changes in tool outputs?* Contract-test the wrapper, version the tool schema and fail clearly on missing required fields rather than silently accepting partial data.

**Drill.** Define a `/answer` request with question, document scope and top-K limits; test malformed JSON, oversized input, forbidden scope and stale document version.

## 3. How do database sessions and connection pools work with multiple users?

**Short answer.** A pool manages a limited number of reusable connections; a request borrows a connection/session for a short unit of work and returns it. Do not keep one persistent connection per user or share a mutable session across requests.

**Detailed explanation.** The engine/pool is normally created at application startup. A request-scoped session performs a transaction, commits or rolls back, then closes reliably even when an exception occurs. Pool size plus overflow must fit the database capacity across *all* app workers. Long-running LLM calls should not hold a database transaction open while waiting; read needed state, release the session, then make the model call, and revalidate before any later write. For a side-effecting tool, use idempotency and clear transaction boundaries.

**Trade-offs.** Bigger pools can improve concurrency until the DB saturates. Smaller pools protect it but increase waiting. Async DB drivers are useful only if the rest of the stack and workload justify them.

**Follow-ups.**
- *Why not create a new engine per request?* It defeats pooling and can exhaust connections.
- *What happens on an exception?* Roll back if needed and return the connection; do not leave an open transaction.
- *How do you size the pool?* Consider all replicas/workers, DB connection limit, observed checkout wait and query latency; load-test with realistic traffic.

**Drill.** Simulate 100 concurrent requests against a pool of 10, measure wait time, and ensure exception paths return connections.

## 4. How do you make a model/tool API reliable?

**Short answer.** Give every call a deadline, classify failures, retry only safe transient operations with bounded exponential backoff and jitter, constrain concurrency, and expose metrics and trace IDs without logging secrets or sensitive prompts.

**Detailed explanation.** Distinguish invalid input/auth failures, 429 quota, transient 5xx/network faults, slow responses and semantically invalid outputs. Honor server retry guidance where present. A request deadline governs the entire chain: queuing, retrieval, model and tools. Retrying an LLM read may duplicate cost; retrying a business write can duplicate the action unless the tool has an idempotency key and reconciliation. Use a circuit breaker or load shedding when sustained failures justify it. Treat tool outputs as untrusted data, validate required fields and signal partial responses.

**Trade-offs.** Retries improve transient success but increase tail latency and load. Fallback models improve availability only if quality and data-residency constraints allow them. Rich logs help diagnosis but can expose bank data; use metadata, sampled/redacted traces and controlled audit stores.

**Follow-ups.**
- *What do you do on 429?* Respect retry guidance, reduce concurrency and stop once the request deadline or retry budget is spent.
- *What if a tool times out after submitting a payment?* Do not assume failure; query by idempotency key and reconcile the authoritative state.
- *What do you monitor?* p50/p95 latency, time-to-first-token, queue depth, tokens/cost, 429/5xx, retries, tool failure categories and task success.

**Drill.** Stub 429, 500, timeout and partial JSON responses; show bounded retries and no duplicate side effects.

## 5. How does streaming change the API contract?

**Short answer.** Streaming improves time to first visible token. It complicates cancellation, partial errors, moderation/validation and client behavior. It does not guarantee lower total latency or cost.

**Detailed explanation.** Define event types such as token, citation, completed and error. Once bytes have been sent, the HTTP status cannot be changed to reflect a later model failure, so send an explicit terminal error event and do not mark the answer complete. On disconnect, close the upstream stream and release capacity. Never execute a side-effecting tool from an incomplete streamed fragment. Buffer structured tool-call arguments until parsing and authorization pass.

**Trade-offs.** Streaming suits interactive UX. Non-streaming is simpler when strict schema validation or all-or-nothing output matters. A hybrid approach can stream explanatory text while holding actions behind a validated execution boundary.

**Follow-ups.**
- *Can you cache a partial stream?* Usually not as a completed answer; record completion state and version scope.
- *How do you measure streaming?* Track first-token time, completion time, cancellation rate and partial-failure rate.
- *What if the client reconnects?* Decide whether to resume from persisted events or start a new idempotent request; do not silently duplicate work.

**Drill.** Build a stub streamed endpoint and tests for disconnect, upstream timeout after three tokens and malformed final tool arguments.

## Practice order

Start with questions 1 and 4, because they connect your API integration experience to Python. Then implement question 2; study pooling and streaming when the service needs them. Give an answer aloud before reading the detail.
