# Batch 010 — Agent loop control and repeatable behaviour

**Collected:** 30 October 2026. **Readiness:** Untested.

## Evidence note

This batch uses one independently posted candidate account:

- [Naresh Edagotti — AI Engineer Interview Questions: Evaluation, Reliability, Scalability](https://www.linkedin.com/posts/naresh-edagotti-6a71a1233_i-had-langgraph-on-my-resume-the-interviewer-activity-7508343805452955649-jX7C). The public LinkedIn page displayed **“1w” when verified on 30 October 2026**; LinkedIn did not expose an exact calendar date, so no more precise publication date is claimed. The author says LangGraph was on his résumé and reports five interviewer questions concerning loops, inconsistent answers, evaluation, workflow design and scaling. The post also promotes a paid interview kit. Only the visible first-person interview claims are used.

This is one report, regardless of the number of questions. It increases the reported-experience counter by one. It is useful preparation evidence, not evidence of market-wide frequency.

The evaluation, orchestration and scaling questions substantially overlap [Batch 009](batch-009-2026-10-23-langgraph-evaluation-project-defence.md) and earlier system-design batches. This batch teaches the two less-developed failure modes instead of repeating all five.

## Study dependencies

1. Read [GenAI concept foundations](../00-genai-concept-foundations.md), sections 1, 2, 9 and 10.
2. Work through [Agent loops and repeatable behaviour](../01-agent-loops-and-repeatability.md).
3. Implement the exercises in the [Enterprise Document Intelligence Assistant](../PROJECT_ENTERPRISE_DOCUMENT_ASSISTANT.md).
4. Then practise the answers below.

---

## 1. Your agent keeps calling the same tool and never finishes. What do you investigate, and how do you prevent it?

**Source wording:** close paraphrase of the first reported question.  
**Roadmap:** Stage 5 — tools and LangGraph. **Priority:** P0.

### What the interviewer is testing

Whether you understand that an “agent” is a controlled software loop, can diagnose state and tool failures, and can place hard bounds around cost and side effects.

### Prerequisites from first principles

A tool call is a structured request proposed by an LLM. For example:

```json
{"tool": "search_policy", "arguments": {"query": "Tier B transfer limit"}}
```

Your Python service validates this request, runs the function, saves the result in state, and calls the model again. If the model cannot see a useful observation or has no finish condition, it may propose the same call again.

**MuleSoft analogy:** a Until Successful scope that retries forever because every response is treated as retryable. The fix is more than setting a retry count: classify the error, update variables, stop on permanent failure, and make write operations idempotent.

### Where it fits in the running project

Use this flow in the document assistant:

```text
question
 -> retrieve policy passages
 -> evidence found? -> answer with citation
 -> empty but query can improve? -> rewrite once and retry
 -> same call or budget exhausted? -> human review
```

Relevant state:

```python
from typing import TypedDict

class AssistantState(TypedDict):
    question: str
    attempts: int
    prior_call_keys: set[str]
    evidence: list[dict]
    stop_reason: str | None
```

The model may suggest a query. Deterministic Python decides whether that proposal is allowed.

### Small implementation

```python
import json
from hashlib import sha256

MAX_TOOL_CALLS = 3

def call_key(tool: str, arguments: dict) -> str:
    canonical = json.dumps(arguments, sort_keys=True)
    return sha256(f"{tool}:{canonical}".encode()).hexdigest()

def allow_tool_call(state: AssistantState, tool: str, arguments: dict) -> bool:
    key = call_key(tool, arguments)

    if key in state["prior_call_keys"]:
        state["stop_reason"] = "duplicate_tool_call"
        return False

    if state["attempts"] >= MAX_TOOL_CALLS:
        state["stop_reason"] = "tool_budget_exhausted"
        return False

    state["prior_call_keys"].add(key)
    state["attempts"] += 1
    return True
```

**Expected input and output:**

```text
first search_policy({"query": "Tier B limit"})  -> allowed=True, attempts=1
same call again                               -> allowed=False
stop_reason                                   -> duplicate_tool_call
```

The fingerprint uses a canonical JSON representation so argument order does not make identical calls look different.

### Why this approach

- A hard budget guarantees a cost and latency ceiling.
- Duplicate detection catches a common loop earlier.
- A stop reason makes the trace debuggable.
- Deterministic code owns the safety rule; the LLM cannot talk its way around it.

### Alternatives and trade-offs

- **Only a maximum-step limit:** simple and essential, but detects the loop late and does not explain it.
- **Prompt instruction saying “do not repeat”:** useful guidance, but not an enforcement control.
- **Allow repeat after a delay:** appropriate for polling a changing external job; include job ID and next allowed time in the key.
- **Human review:** useful for ambiguous or consequential tasks, but adds delay and reviewer workload.
- **Multi-agent reviewer:** can inspect a plan, but adds model calls and may repeat the same failure.

For an API that writes data, also use an idempotency key. Preventing a loop does not undo a duplicate payment or ticket creation.

### Beginner-friendly interview answer

“An agent is a loop where the model proposes an action, Python calls the tool, and the result goes back into state. I first check the trace: did the tool return useful data, was the result stored, and does the graph have a route to finish? I add a maximum step count, detect repeated tool name plus arguments, classify retryable and permanent errors, and route exhausted cases to review.”

### Senior-level answer

“I inspect the repeated transition, normalized tool arguments, observation and state diff. Root causes usually include an empty or malformed observation, missing state update, an unconditional graph edge, an underspecified completion condition, or a tool incapable of satisfying the request. I enforce a tool-call budget and duplicate fingerprint in code, use typed outcomes such as success/empty/retryable/permanent, require idempotency for side effects, and measure loop rate, calls per successful task, abandonment and cost.”

### Follow-up questions with answers

**Is every repeated call wrong?**  
No. Polling a long-running job or retrying after a transient timeout can be valid. The state must show what changed: attempt number, backoff, job status or refreshed credentials. An identical call with no new condition is suspicious.

**Where should the stop rule live in LangGraph?**  
Store attempts and call fingerprints in graph state. Use a conditional edge after planning and after the tool result. Route to tool, answer, retry or review through deterministic predicates.

**How do retries differ from loops?**  
A retry is bounded and triggered by a classified transient failure, normally with backoff or changed conditions. A loop has no reliable progress or termination proof.

**What metric would you monitor?**  
Start with duplicate-call rate, tool calls per completed task, percentage hitting the budget, and human-review rate. Slice by tool and error class.

### Common mistakes and debugging

- Increasing the limit without finding the cause.
- Treating every tool error as retryable.
- Failing to include the previous observation in state.
- Fingerprinting raw JSON where key order changes.
- Letting the model decide whether it exceeded a security or cost limit.
- Retrying non-idempotent writes without a request key.

### Practical exercise

In the document assistant, mock search so the first request returns an empty list. Make the planner propose the same query twice.

**Pass criteria:**

- the first call is recorded;
- the second identical call is rejected;
- execution ends as `needs_review`;
- `stop_reason` is `duplicate_tool_call`;
- no more than one actual search function invocation occurs.

---

## 2. The same query produces different answers although the code did not change. What causes this, and how do you debug it?

**Source wording:** close paraphrase of the second reported question.  
**Roadmap:** Stages 4, 5 and 7 — evaluation, agent state and production observability. **Priority:** P0.

### What the interviewer is testing

Whether you can separate normal probabilistic generation from uncontrolled variation in retrieval, prompts, state, tools, deployment versions and concurrency.

### Prerequisites from first principles

An LLM generates text one token at a time. It assigns probabilities to possible next tokens. Sampling settings such as temperature can choose different tokens across runs.

The full application has more moving parts:

```text
question
 -> conversation/state
 -> retrieval and ranking
 -> prompt assembly
 -> model generation
 -> optional tools
 -> final answer
```

Variation at any step can change the result. Setting temperature to zero may make generation more stable, but it cannot stabilize a changing document index or leaked conversation state.

**API analogy:** two identical HTTP requests can return different results because of database updates, cache differences, feature flags, time-based logic or different service versions. You compare the complete request context and downstream trace.

### Where it fits in the running project

Attach reproducibility metadata to every `/ask` response:

```python
from pydantic import BaseModel

class RunMetadata(BaseModel):
    request_id: str
    prompt_version: str
    model_version: str
    corpus_version: str
    retrieved_ids: list[str]
    route: list[str]
    tool_calls: int
```

This metadata is not shown as normal answer content, but it is stored in a redacted trace and can be returned in a debug environment.

### Small implementation

```python
def compare_runs(a: RunMetadata, b: RunMetadata) -> list[str]:
    differences = []
    for field in (
        "prompt_version",
        "model_version",
        "corpus_version",
        "retrieved_ids",
        "route",
        "tool_calls",
    ):
        if getattr(a, field) != getattr(b, field):
            differences.append(field)
    return differences
```

**Expected examples:**

```text
same prompt/model/corpus; retrieved_ids differ -> investigate retrieval tie-breaking or index
retrieved_ids same; route differs             -> inspect state and routing condition
everything same; answer wording differs       -> inspect sampling/seed and judge semantic equivalence
```

### Why this approach

It localizes variation before you change the system. Comparing only final strings confuses harmless wording differences with changes in evidence or behavior.

### Alternatives and trade-offs

- **Exact string equality:** excellent for structured fields; too strict for natural-language wording.
- **LLM-as-a-judge:** can compare meaning, but the judge can vary and costs money.
- **Reference facts/citation assertions:** deterministic and strong for known cases, but require labeled data.
- **Fixed random seed:** useful where supported, but provider/model implementations may still change.
- **Caching the final answer:** improves repeatability and cost, but can serve stale or unauthorized data unless the key includes tenant, permissions, corpus and prompt versions.

### Beginner-friendly interview answer

“I would not assume temperature is the only cause. I compare the prompt and model versions, retrieved document IDs, conversation state, tool results and route for both runs. If the evidence differs, I debug retrieval or the index. If all inputs are the same but wording differs, I reduce sampling and evaluate factual and citation consistency rather than requiring identical prose.”

### Senior-level answer

“I define repeatability at several layers: route agreement, retrieval overlap, citation agreement, structured-fact agreement and semantic answer agreement. Each trace records prompt/model/corpus versions, permissions, retrieval IDs, tool observations and state transitions. I replay a fixed test set, isolate the earliest divergent step, verify cache and tenant keys, and distinguish acceptable linguistic variation from unstable evidence or business decisions.”

### Follow-up questions with answers

**Does temperature zero guarantee the same output?**  
No. It reduces sampling variation, but retrieval order, service versions, concurrency, provider implementation and external tools can still vary.

**What should be exactly deterministic?**  
Authorization, schema validation, budget enforcement, idempotency, routing for critical rules, and structured calculations should be code-driven. Natural-language wording may vary while supported facts and citations remain stable.

**How would you test repeatability?**  
Run each fixed question five times against a frozen corpus. Compare routes, retrieved IDs, citations, structured facts and answer meaning. Report disagreement by layer.

**Can a cache create cross-user errors?**  
Yes. A cache key based only on question text can return one tenant’s answer to another. Include tenant/permission scope, corpus version, prompt version and relevant conversation state, or avoid caching sensitive outputs.

### Common mistakes and debugging

- Setting temperature to zero and declaring the problem solved.
- Comparing only final answer strings.
- Omitting model, prompt and corpus versions from traces.
- Sharing mutable state across concurrent requests.
- Using a cache key that excludes authorization scope.
- Calling variation a hallucination without checking whether retrieval changed.

### Practical exercise

Choose 10 synthetic bank-policy questions. Run each five times with a frozen corpus and temperature zero.

Record:

- route;
- retrieved IDs;
- cited IDs;
- extracted structured answer, such as `{"limit_rupees": 75000}`;
- latency and tool-call count.

**Pass criteria:**

- 100% tenant and citation correctness;
- 100% structured-fact agreement for answerable cases;
- at least 95% route agreement;
- any remaining prose variation is supported by the same evidence;
- the earliest divergent step is identifiable from the trace.

---

## Recommended weekly study plan

1. **30 minutes:** read [Agent loops and repeatable behaviour](../01-agent-loops-and-repeatability.md).
2. **45 minutes:** implement call fingerprints, budget and stop reasons.
3. **45 minutes:** add run metadata and the five-run repeatability test.
4. **20 minutes:** explain both beginner answers aloud without notes.
5. **20 minutes:** answer the follow-ups and show the recorded evidence.

Do not change readiness from **Untested** until the exercises run and the answers can be explained.
