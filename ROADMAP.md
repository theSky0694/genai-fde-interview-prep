# Master Roadmap — September 2026 to March 2027

## Objective

Become interview-ready for GenAI Engineer / Agentic AI Engineer / Forward Deployed Engineer roles by combining:

- production-oriented Python and backend/API engineering
- LLM and GenAI fundamentals
- RAG and retrieval engineering
- agentic AI: tool use, state, planning, memory and reliable execution
- LangGraph, MCP and interoperability fundamentals
- agent/RAG evaluation, observability, security and governance
- GenAI system design
- customer/problem-solving skills expected from FDEs
- demonstrable, explainable GitHub projects
- repeated interview practice

This roadmap assumes strong existing enterprise integration/API experience and does **not** follow a generic beginner-programming curriculum.

**Core progression:** Python → Backend/FastAPI → LLM fundamentals → RAG → Production RAG → Agentic AI → MCP/interoperability → Production GenAI/System Design → Portfolio/Interviews.

Agentic AI extends the roadmap rather than replacing its foundations. Python, APIs, LLM fundamentals and retrieval remain prerequisites.

---

## Stage 1 — Python for GenAI/FDE
**Window:** Sep–Oct 2026

### 1A — Python foundations
Learn:
- Python execution model and syntax
- variables and core types
- list, tuple, set, dict
- conditions and loops
- functions, arguments, scope
- comprehensions
- exceptions
- modules and packages
- classes, dataclasses and OOP
- type hints
- JSON and file handling
- virtual environments and dependency management

Build:
- small transformation exercises
- JSON/config processor
- typed mini application

Interview bar:
- explain Python collections and mutability
- write clean functions without assistance
- choose appropriate data structures
- explain exceptions, scope and common Python idioms

### 1B — Production Python
Learn:
- HTTP clients
- REST/API consumption
- Pydantic
- async/await
- FastAPI
- SQLAlchemy engine/session/connection pooling
- logging
- environment/configuration management
- pytest
- basic packaging

Build:
- production-style FastAPI service
- external API integration
- validation, error handling and tests

Interview bar:
- explain sync vs async
- design and implement an API
- discuss validation, failures, testing and observability

**Parallel DSA lane:** start with 2 problems/week: arrays, strings, hash maps, two pointers/sliding window, stack/queue, binary search and core patterns.

---

## Stage 2 — LLM Fundamentals & Prompting
**Window:** Oct 2026

Learn:
- transformer intuition
- tokens and tokenization
- embeddings
- context windows
- inference
- temperature/top-p
- system/user/tool messages
- structured output
- prompt engineering
- model selection
- latency/cost/quality trade-offs
- hallucinations and limitations
- foundations of function/tool calling

Build:
- model client abstraction
- structured-output application
- prompt evaluation experiments
- first schema-constrained tool call

Interview bar:
- explain an LLM application end-to-end
- discuss model selection and production trade-offs
- distinguish plain generation, structured output and tool calling
- diagnose common prompt/application failures

---

## Stage 3 — Embeddings, Vector Search & RAG
**Window:** Nov 2026

Learn:
- embeddings and cosine similarity
- chunking strategies
- metadata and indexing
- vector databases
- semantic vs keyword search
- dense vs sparse retrieval
- BM25
- hybrid search
- RRF
- reranking
- structure-aware and parent-child chunking
- context expansion
- multi-hop/distributed-evidence retrieval
- graph retrieval fundamentals and when relationships justify a graph
- RAG pipeline design

Build progressively:
1. retrieval from scratch
2. basic RAG
3. metadata filtering
4. hybrid retrieval
5. reranking
6. structure-aware/context-expanded retrieval

Interview bar:
- design a RAG pipeline on a whiteboard
- justify chunking/retrieval decisions
- diagnose poor retrieval
- explain hybrid/fusion/reranking trade-offs

---

## Stage 4 — Production RAG & Evaluation
**Window:** Nov–Dec 2026

Learn:
- Precision@K, Recall@K, MRR and NDCG intuition
- answer-quality evaluation and groundedness
- synthetic/labeled test sets
- observability/tracing
- caching and latency
- access control and confidential data
- prompt injection considerations
- document ingestion pipelines
- production failure modes
- cost controls and model routing

Build:
- evaluated RAG API
- automated evaluation dataset
- tracing/metrics
- production-oriented ingestion pipeline

Interview bar:
- answer “How do you know your RAG system is good?”
- debug retrieval vs generation failures
- discuss enterprise security, cost and scalability

---

## Stage 5 — Agentic AI, Tool Calling & LangGraph
**Window:** Dec 2026–Jan 2027

This is now a **core track**, not an optional extension.

### 5A — From LLM calls to agents
Learn the progression:
**LLM call → structured output → tool calling → tool loop → stateful workflow → agent**

Learn:
- function/tool schemas
- tool selection and execution
- tool-result feedback loops
- deterministic workflows vs agentic decision-making
- agent loops and stopping conditions
- routing
- planning vs workflows
- when **not** to use an agent

### 5B — LangGraph and reliable orchestration
Learn:
- nodes, edges and shared state
- conditional routing
- checkpoints/durable execution
- retries/timeouts/failure recovery
- human-in-the-loop and approval gates
- interrupt/resume patterns
- idempotent tool execution
- auditability

### 5C — Agent memory
Learn:
- thread/short-term state
- sliding windows
- summaries/context compaction
- structured task state
- semantic/episodic long-term memory
- selective memory retrieval
- memory write/read policies
- memory failure modes

### 5D — Planning and multi-agent fundamentals
Learn:
- router patterns
- planner/executor pattern
- reflection/critique concepts without over-engineering
- supervisor/worker patterns
- single-agent vs multi-agent trade-offs
- delegation and communication boundaries

Do **not** learn multiple frameworks for framework collecting. LangGraph remains the primary implementation framework; other frameworks are conceptual comparison only.

Build progressively:
1. deterministic tool-using workflow
2. tool-loop agent
3. stateful LangGraph workflow
4. RAG tool inside the agent
5. SQL/API tools
6. checkpointing + human approval
7. memory-enabled workflow
8. guarded/reliable agent with failure handling

Interview bar:
- distinguish workflow from agent
- explain when not to use an agent
- design state and tool boundaries
- diagnose a wrong agent answer from trace → routing → retrieval/tool → state → model
- explain memory choices
- defend single-agent vs multi-agent architecture

---

## Stage 6 — MCP, A2A & Agent Architecture
**Window:** Jan 2027

Learn:
- MCP concepts and client/server architecture
- tools/resources/prompts
- discovery and invocation
- direct tools vs MCP
- authentication/security boundaries
- reusable enterprise tool integration
- MCP operational trade-offs
- A2A fundamentals
- MCP vs A2A
- interoperability vs application-specific orchestration

Build:
- small MCP server
- LangGraph agent consuming MCP tools
- one reusable tool exposed both conceptually/directly and via MCP for comparison

Interview bar:
- explain why MCP exists
- justify direct integration vs MCP
- explain MCP vs A2A
- discuss authentication, authorization, deployment and trust boundaries

---

## Stage 7 — Production Agentic AI & GenAI/FDE System Design
**Window:** Jan–Feb 2027

### Agent production engineering
Learn:
- agent tracing and step-level observability
- tool-call metrics and failure classification
- agent evaluation and trajectory evaluation
- task-success metrics
- deterministic regression tests around nondeterministic models
- tool permissions/least privilege
- prompt injection and tool-abuse risks
- approval gates for high-impact actions
- retries, budgets, loop limits and timeouts
- idempotency and duplicate-action prevention
- model/tool fallback
- latency and cost budgets
- state persistence and recovery

### Inference engineering
Learn:
- serving architecture
- TTFT, inter-token latency, throughput
- batching
- KV cache
- quantization
- autoscaling
- managed vs self-hosted benchmarking

### GenAI/FDE system design
Practise:
- multi-tenant GenAI/agent systems
- API gateway/authentication
- model gateways
- queues and asynchronous processing
- caching
- vector infrastructure
- observability/evaluation
- PII/data governance
- resilience/scaling/deployment
- agent security and governance

FDE scenarios:
- ambiguous customer requirement → technical design
- deterministic workflow vs agent decision
- prototype → production
- integration with enterprise systems
- debugging customer environments
- communicating trade-offs to technical/non-technical stakeholders

Build:
- architecture case studies
- system-design documents
- implementation spikes
- agent evaluation/observability dashboard or report

Interview bar:
- conduct 45–60 minute system-design discussions
- clarify requirements before designing
- defend why an agent is/isn't appropriate
- design safe, observable and recoverable agent systems
- defend trade-offs rather than reciting architectures

---

## Stage 8 — Portfolio & Interview Sprint
**Window:** Feb–Mar 2027

### Flagship evolution

**Project A — TxnGuard: Enterprise RAG → Agentic RAG Platform**

Progressively evolve the same system:
**RAG → advanced retrieval → evaluated RAG → RAG tool → LangGraph agent → multi-tool agent → MCP-enabled integration → production API**

Target capabilities:
- ingestion
- hybrid retrieval + reranking
- citations/evaluation
- FastAPI
- RAG tool
- SQL/structured-data tool
- controlled tool execution
- LangGraph state/checkpointing
- human approval where appropriate
- memory only where justified
- MCP integration
- tracing/auditability
- security/guardrails
- tests and failure scenarios

**Project B — Agentic Developer / Operations Assistant**
- repository/operations tools
- LangGraph orchestration
- controlled execution
- MCP integration
- state/checkpointing
- approval gates
- observability/evaluation
- auditability

Projects must have:
- clear problem statement
- architecture
- runnable code
- tests
- meaningful README
- design decisions/trade-offs
- evaluation
- known limitations
- explicit explanation of where deterministic workflow is preferred over agent autonomy

### Interview sprint
Practise repeatedly:
- Python coding + DSA
- GenAI/LLM fundamentals
- RAG/retrieval
- agentic AI/LangGraph
- tool calling/MCP/A2A
- agent debugging/evaluation
- production/system design
- FDE customer scenarios
- project deep dives
- behavioural/leadership stories

Every failed or weak answer goes into FAILED_QUESTIONS.md.

---

# Weekly operating model

A normal week includes:
- concept learning
- hands-on implementation
- one code review/refactor session
- interview questions
- one cumulative revision session

During Stages 3–7, prefer **evolving TxnGuard** over repeatedly creating disconnected demo projects.

Do not wait until March for interviews. Interview practice begins during Stage 1.

# Scope guardrails

To keep the March 2027 target realistic:
- LangGraph is the primary agent framework.
- Do not spend weeks learning CrewAI/AutoGen/etc. unless a target role specifically requires one.
- Multi-agent is learned after reliable single-agent workflows.
- Agentic AI does not replace RAG, Python or backend fundamentals.
- Prefer deterministic workflows when the execution path is known.
- Every new agent capability must be testable, observable and explainable.
- Cloud-specific products remain secondary to portable architecture concepts.

# Definition of interview-ready

Interview-ready does **not** mean finishing every topic.

It means being able to:
1. code comfortably in Python under interview conditions;
2. explain LLM/RAG/agent fundamentals without memorised scripts;
3. build and debug a realistic GenAI service;
4. build a tool-using stateful agent and explain every architectural decision;
5. evaluate/debug agent trajectories, tools, retrieval and generation separately;
6. design production GenAI/agent architectures and defend trade-offs;
7. deeply explain portfolio and professional projects;
8. handle ambiguous FDE/customer scenarios;
9. give concise senior-level behavioural examples;
10. recover well when an interviewer pushes beyond the first answer.

---

# Evidence-driven adjustments — 22–24 September 2026

The first reported interview batches already reinforce the expanded agentic track:

- **Python/backend remains foundational:** FastAPI, async and SQLAlchemy/pooling stay early.
- **Retrieval remains P0:** BM25, dense/sparse, hybrid retrieval, RRF, reranking, structure-aware chunking, context expansion and multi-hop retrieval.
- **Agentic AI is promoted to a core track:** tool loops, LangGraph state/routing, memory, failure diagnosis, human-in-the-loop and reliable execution.
- **MCP is deepened; A2A remains fundamental/interview depth:** interoperability follows tool-calling fundamentals rather than preceding them.
- **Production concerns move earlier:** cost, security, observability, traceability and evaluation are threaded through RAG and agent stages.
- **Agent evaluation is explicitly added:** task success, trajectory/tool-call inspection, regression cases and failure classification.
- **Inference engineering remains before/within system design:** serving, TTFT/latency/throughput, batching, KV cache, quantization, autoscaling and managed-vs-self-hosted benchmarking.
- **Parallel DSA continues:** initially 2 problems/week.
- **Compact ML fundamentals remain P2:** logistic regression/sigmoid, SVM/kernels and time-series differencing.
- **Cloud-specific lane remains secondary:** portable concepts first; AWS Bedrock/AgentCore can be mapped afterward.

The March 2027 target remains unchanged. The additional Agentic AI scope is absorbed mainly by deepening Stages 5–7 and evolving the same flagship project rather than adding months of separate framework study.
