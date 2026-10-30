# Agent loops and repeatable behaviour — from first principles

**Audience:** experienced API/integration developer beginning agent engineering.  
**Prerequisites:** [GenAI concept foundations](00-genai-concept-foundations.md), sections 1, 2, 9 and 10.

## 1. What makes an agent a loop?

A normal API flow follows routes written by a developer. An agent adds a model decision between steps:

```text
user request -> model chooses action -> application calls tool
             -> tool result returns -> model chooses next action -> ...
```

The LLM does not execute the tool itself. It returns a structured proposal such as:

```json
{"tool": "search_policy", "arguments": {"query": "Tier B transfer cap"}}
```

Python validates the proposal, calls the function, stores the observation, and asks the model what to do next.

**MuleSoft analogy:** imagine a Choice router whose next route is proposed by a nondeterministic service. The Mule flow must still validate the route, enforce permissions, handle errors, record state and stop after a bounded number of transitions.

## 2. Why an agent can repeat the same tool forever

A loop normally means the model has not received evidence that its previous action satisfied the task. Common causes are:

1. **The tool result is unclear.** It returns an empty string or a generic error rather than a typed outcome.
2. **State was not updated.** The next model call cannot see that the tool already ran.
3. **The prompt has no finish condition.** The agent knows which tool exists but not when to stop.
4. **A retry looks identical to the first attempt.** The same input produces the same failure.
5. **The tool is being used for the wrong job.** Search cannot answer because the document was never indexed.
6. **The graph routes every result back to the tool node.** This is application logic, not model intelligence.

A maximum-step guard prevents unlimited cost, but it does not diagnose the root cause.

## 3. A small bounded Python agent

```python
from dataclasses import dataclass, field

@dataclass
class AgentState:
    question: str
    attempts: int = 0
    previous_calls: set[tuple[str, str]] = field(default_factory=set)
    evidence: list[str] = field(default_factory=list)
    status: str = "running"

MAX_STEPS = 3

def run_search(state: AgentState, query: str, search) -> AgentState:
    call_key = ("search_policy", query.strip().lower())

    if call_key in state.previous_calls:
        state.status = "needs_review"
        return state

    state.previous_calls.add(call_key)
    state.attempts += 1
    state.evidence = search(query)

    if state.evidence:
        state.status = "ready_to_answer"
    elif state.attempts >= MAX_STEPS:
        state.status = "needs_review"

    return state
```

**Input:**

```python
AgentState(question="What is the Tier B limit?")
```

**Expected state after useful evidence:**

```text
status=ready_to_answer, attempts=1, evidence=["Tier B daily limit is ₹75,000"]
```

**Expected state after the same failed call is proposed twice:**

```text
status=needs_review
```

Why store `previous_calls`? A step limit catches any long run; duplicate-call detection catches one common failure earlier and provides a specific reason.

## 4. Why the same question can produce different answers

An LLM predicts the next token from a probability distribution. Generation settings may sample different tokens. Variation can also enter elsewhere:

- different retrieved chunks or ranking order;
- an updated document index;
- changing conversation history or agent memory;
- current time, external API data, or tool errors;
- different model/version or system prompt;
- concurrent state leakage or an unstable cache;
- an LLM judge that is itself variable.

**Temperature** controls how strongly generation favors high-probability choices. Lower temperature can reduce sampling variation, but it does not make the entire application deterministic.

**API analogy:** retrying a GET request can still return different data if the database changed, the load-balanced nodes have inconsistent caches, or request headers differ. You trace every dependency before blaming HTTP.

## 5. Build a repeatability trace

Record the inputs that can affect a run:

```python
trace = {
    "request_id": "req-104",
    "question": "What is the Tier B limit?",
    "prompt_version": "answer-v3",
    "model": "example-model-2026-10",
    "temperature": 0,
    "corpus_version": "policies-2026-10-30",
    "retrieved_ids": ["policy-7#p4"],
    "tool_calls": 1,
}
```

Do not log secrets or full sensitive documents. IDs, versions, hashes, timings and redacted outcomes are usually enough to compare runs.

## 6. Small repeatability experiment

Run the same 10 questions five times, giving 50 runs.

For each question measure:

- **route agreement:** percentage of runs that follow the same node sequence;
- **retrieval agreement:** overlap of retrieved chunk IDs;
- **answer agreement:** exact match for structured fields, or a reviewed semantic score for prose;
- **citation agreement:** percentage using the same supporting source;
- tool errors and latency.

Run two configurations:

1. fixed corpus and prompt, temperature 0;
2. temperature 0.7.

If configuration 1 still varies, compare retrieval IDs, state and tool traces before changing the model settings.

## 7. LangGraph placement

In LangGraph, add explicit terminal routes:

```text
START -> retrieve
retrieve -> answer       when evidence is sufficient
retrieve -> retry        when the query can be changed and attempts remain
retrieve -> human_review when call repeats or budget is exhausted
answer -> END
```

Store attempts, call fingerprints, evidence IDs and failure reason in typed graph state. A checkpointer supports resume after interruption; it must not become an unbounded memory dump.

## 8. Trade-offs

| Choice | Benefit | Cost / risk |
|---|---|---|
| Maximum steps | Hard cost and latency bound | May stop a recoverable task |
| Duplicate-call block | Stops obvious loops quickly | A repeated call can be valid if external state changed |
| Deterministic workflow | Easier to test and audit | Less flexible for open-ended tasks |
| Low temperature | More repeatable wording | Does not fix retrieval or state variation |
| Human review | Handles ambiguity and high-risk cases | Adds delay and operational workload |
| Multi-agent review | Different perspective | More model calls can add disagreement and cost |

## 9. Debugging checklist

1. Reproduce with the same prompt, model, corpus and state versions.
2. Compare node sequence and tool arguments.
3. Confirm observations were written into state.
4. Check whether the tool returned typed success, empty result, retryable error or permanent error.
5. Inspect retrieval IDs before comparing generated prose.
6. Verify cache keys include tenant, user-relevant permissions, corpus version and prompt version.
7. Add a regression case that proves the loop or inconsistency is detected.

## 10. Interview answers

**Beginner answer:** “An agent repeatedly lets a model choose an action, executes a validated tool call, and returns the observation. I prevent endless loops with explicit finish conditions, a step budget, duplicate-call detection and a review route. For inconsistent answers, I compare prompt, model, retrieval, state and tool traces rather than assuming temperature is the only cause.”

**Senior extension:** “I make termination and repeatability service-level properties. Typed state records attempts, call fingerprints and evidence; deterministic predicates own critical routing; every run records model, prompt and corpus versions. I measure route, retrieval, citation and answer agreement over repeated cases, then isolate the layer introducing variance.”

## 11. Practice task

Extend the [Enterprise Document Intelligence Assistant](PROJECT_ENTERPRISE_DOCUMENT_ASSISTANT.md):

- simulate an empty retrieval result;
- make a mock planner propose the same search twice;
- verify the second proposal routes to `needs_review`;
- run one answerable question five times;
- confirm the retrieved source ID remains identical.

**Pass criteria:** no run exceeds three tool calls; the duplicate is detected; the trace states why execution stopped; all five stable-corpus runs cite the same source.

**Readiness:** Untested until implemented and explained.
