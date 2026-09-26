# Agent workflows — interview answers

[Guide home](README.md) · Original explanations informed by the topics in the [source agentic AI collection](https://github.com/Nareshedagotti/AI-Engineer-Interview-QA/blob/main/Agentic_AI_Interview_Questions.md). **Roadmap:** Stages 5–7. **Readiness:** Untested.

## 1. What makes an agent different from a workflow or one LLM call?

**Short answer.** A single call transforms input into output. A workflow follows an explicit path with conditional branches. An agent can choose among actions and tools based on intermediate observations, repeating until it reaches a stop condition. Use only as much autonomy as the task requires.

**Detailed explanation.** A known approval process—retrieve a record, validate, seek approval, write—is best represented as a deterministic workflow even if an LLM extracts information within a step. An open-ended investigation may require the model to decide whether to search, inspect a database, ask for clarification or stop. The agent loop is observe → decide → call a permitted tool → validate the result → update state → repeat. Its developer still owns the action space, budget, permissions, timeout and success criteria. “Agentic” does not absolve us from integration contracts.

**Trade-offs.** More autonomy handles variable paths but raises latency, cost and debugging surface. A fixed flow is easier to audit and test. A hybrid can let the model choose *which read* to perform while deterministic code governs writes.

**Follow-ups.**
- *When would you not use an agent?* If the path is known, latency is strict, the task is high-impact or a simple query/retrieval suffices.
- *What stops infinite loops?* Step and token budgets, deadlines, repeated-action detection, state-based completion and explicit failure outcomes.
- *Can a workflow contain an LLM?* Yes. LLM use alone does not make the whole system autonomous.

**Drill.** Draw a deterministic policy lookup and a tool-selecting investigation. Define where a model decides versus where code enforces rules.

## 2. How would you structure a LangGraph application?

**Short answer.** Define typed state, small nodes with clear inputs/outputs, explicit routing, checkpoints where durable resumption matters, and traceable tool boundaries. Keep business side effects idempotent and outside speculative reasoning.

**Detailed explanation.** State can hold request ID, user/tenant, current task, evidence IDs, tool observations, proposed action, approval status, budgets and errors. Nodes might validate request, retrieve, assess evidence, choose next step, execute permitted tool, verify result and finalize. Conditional edges route based on typed outcomes, not ambiguous free-form text. A checkpoint stores progress so an interrupted run can resume, but resumption may repeat a node: tool writes need idempotency keys and authoritative outcome checks. State should contain minimal sensitive data and references to protected records rather than entire confidential documents. The graph is a control-flow tool; it does not automatically make an LLM reliable.

**Trade-offs.** A graph gives explicit state and branching but adds operational complexity. A simple function or queue-driven workflow can be better for a small fixed path. Persisting every state improves recovery and auditability but adds storage, retention and privacy responsibilities.

**Follow-ups.**
- *What is a node versus an edge?* A node performs a step; an edge chooses the next step based on state.
- *What happens if the process crashes after a bank API write but before checkpointing?* On resume, reconcile by idempotency key or transaction ID before replaying; never assume checkpoint and external write were atomic.
- *How do you debug wrong output?* Walk the trace: routing decision, retrieved evidence, tool inputs/outputs, state transitions and model response.

**Drill.** Build a small graph with retrieval, validation, human approval and a stub write tool. Kill it between write and checkpoint; prove no duplicate write after resume.

## 3. How do you design a safe tool integration?

**Short answer.** Treat a model-proposed call as a request, not authority. Expose narrow typed tools, authenticate and authorize in the tool layer, validate arguments, set timeouts and idempotency, classify errors and log auditable outcomes.

**Detailed explanation.** A tool schema should make the action and required parameters clear. Split read-only from side-effecting tools; avoid a broad “execute any HTTP request” tool when a narrow account lookup will do. The model can select a tool, but code checks the user's permission, tenant, destination, parameter limits and approval status. The wrapper handles pagination, missing fields, partial results and provider changes explicitly. Tool outputs are data and may be malicious or stale. For writes, record an idempotency key, expected state/version and authoritative result; a timeout is an unknown outcome, not proof of failure. Trace the proposal, validation and final state without exposing secrets.

**Trade-offs.** Narrow tools restrict flexibility but improve reliability and auditability. Generic tools are quicker to prototype but expand the attack surface. Human approval slows execution but is appropriate for irreversible/high-impact actions.

**Follow-ups.**
- *What if the tool returns HTTP 200 with a missing field?* Validate the response contract and surface a typed partial/incomplete result; do not let the model treat it as complete evidence.
- *Can you retry a write?* Only with idempotency and reconciliation.
- *How do you prevent prompt injection from a retrieved page calling a tool?* The page has no authority; enforce allowed actions and destinations in code.

**Drill.** Write a typed `lookup_account` read tool and `create_case` write tool; test unauthorized tenant, missing field, duplicate request and ambiguous timeout.

## 4. When do you use memory, and what belongs there?

**Short answer.** Use short-term state for the active task; persist only durable facts or preferences with a clear owner, consent and update policy. Memory is a product requirement, not a default place to dump every conversation.

**Detailed explanation.** Thread state may contain current goal, prior tool results and unresolved questions. As it grows, compact it with explicit task facts and links back to source records. Long-term memory should have a specific purpose, schema, provenance, expiry and correction path. Retrieval-based memory is useful when relevant past facts must be selected, but an embedding store is not automatically an appropriate store for every sensitive item. Separate user preferences, task state and authoritative business records; do not let a remembered statement override a current permission check or fresh source of truth.

**Trade-offs.** Longer history may help continuity while increasing cost, stale assumptions and data exposure. Summaries save tokens but can omit a critical condition. Structured state is reliable for known fields but less flexible for open-ended context.

**Follow-ups.**
- *What if memory conflicts with a new user correction?* Treat the correction as current and update/supersede the old record with provenance.
- *What if a summary omits a tool failure?* Persist critical status as structured state, not only prose summary.
- *How do you evaluate memory?* Test recall, incorrect recall, stale facts, deletion, cross-user isolation and effect on task success.

**Drill.** Run a multi-turn task with conflicting updates; compare full history, summary and structured state under a fixed token budget.

## 5. When would you use multiple agents or AutoGen instead of one graph?

**Short answer.** Start with one workflow/agent. Add specialized agents only when separable roles, different permissions or parallel independent work produce a measured benefit. Framework choice follows the communication, state and safety needs; it is not a substitute for evaluating the task.

**Detailed explanation.** A planner and executor can be useful when plans need review before actions. Independent research or code-review subtasks may run in parallel, but the coordinator must merge conflicting evidence and enforce budgets. Multi-agent conversation increases tokens, latency and ambiguous ownership. AutoGen is one ecosystem for agent interaction; LangGraph is the primary framework in this roadmap because its explicit state and control flow fit auditable enterprise workflows. Explain concepts and trade-offs, then learn another framework deeply only if a target job requires it.

**Trade-offs.** Specialist agents may improve modularity and parallelism. They also create coordination failures, repeated work and difficult traces. A single agent with well-designed tools may outperform a team for straightforward tasks.

**Follow-ups.**
- *How would you compare architectures?* Hold task set, model, tools and budgets constant; compare task success, cost, latency, interventions and failure classes.
- *What if two agents disagree?* Use source-backed evidence and a deterministic resolution or escalation rule; do not assume majority vote is truth.
- *What is the smallest useful multi-agent case?* Two independent read-only analyses with a verifier or human adjudicator, if measured benefit warrants it.

**Drill.** Solve the same five tasks with a single graph and planner/executor variant. Compare traces and costs, not just final answer fluency.

## 6. How do you evaluate an agent in production?

**Short answer.** Define task success and safety outcomes, inspect trajectories and tool calls, run repeatable scenarios with failures, and monitor latency/cost. A fluent final answer is insufficient.

**Detailed explanation.** Build a small gold set of tasks with initial state, allowed actions, expected evidence or state transition and prohibited actions. Record whether the agent chose the right tool, supplied valid arguments, respected permissions, recovered from partial failure and stopped appropriately. Replay or stub external systems for deterministic regression checks; repeat stochastic model runs to estimate variability. Measure successful completion, unsafe action rate, unnecessary tool calls, retries, total tokens and user intervention. Review real failures by first broken boundary rather than changing prompts blindly.

**Trade-offs.** End-to-end success is intuitive but hides where failure occurred. Step-level metrics diagnose issues but can be over-engineered. Simulated environments are reproducible yet may miss production variation; pair them with carefully governed trace review.

**Follow-ups.**
- *What if the final answer is correct but it called an unauthorized tool?* Fail the safety evaluation; outcome alone is not sufficient.
- *Can an LLM judge evaluate all traces?* Use it as one signal, calibrate against humans and deterministic checks, especially for sensitive actions.
- *What should a rollback mean?* Compensating business actions may not fully reverse impact; define approval and reconciliation before execution.

**Drill.** Create 10 tasks, including missing evidence, tool 200/partial data, timeout after write, prompt injection and cancelled run; record trajectory and outcome metrics.
