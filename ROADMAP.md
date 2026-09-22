# Master Roadmap — September 2026 to March 2027

## Objective

Become interview-ready for GenAI Engineer / Forward Deployed Engineer roles by combining:

- production-oriented Python
- LLM and GenAI fundamentals
- RAG and retrieval engineering
- agentic systems and tool calling
- backend/API engineering
- GenAI system design
- customer/problem-solving skills expected from FDEs
- demonstrable, explainable GitHub projects
- repeated interview practice

This roadmap assumes strong existing enterprise integration/API experience and therefore does **not** follow a generic beginner-programming curriculum.

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

Build:
- model client abstraction
- structured-output application
- prompt evaluation experiments

Interview bar:
- explain how an LLM application works end-to-end
- discuss model selection and production trade-offs
- diagnose common prompt/application failures

---

## Stage 3 — Embeddings, Vector Search & RAG
**Window:** Nov 2026

Learn:
- embeddings and similarity
- cosine similarity
- chunking strategies
- metadata
- vector databases
- indexing
- semantic vs keyword search
- hybrid search
- reranking
- RAG pipeline design

Build progressively:
1. retrieval from scratch
2. basic RAG
3. metadata filtering
4. hybrid retrieval
5. reranking

Interview bar:
- design a RAG pipeline on a whiteboard
- justify chunking/retrieval decisions
- diagnose poor retrieval

---

## Stage 4 — Production RAG & Evaluation
**Window:** Nov–Dec 2026

Learn:
- retrieval metrics
- answer-quality evaluation
- groundedness
- synthetic test sets
- observability/tracing
- caching
- latency
- access control
- prompt injection considerations
- document ingestion pipelines
- production failure modes

Build:
- evaluated RAG API
- automated evaluation dataset
- tracing/metrics
- production-oriented ingestion pipeline

Interview bar:
- answer “How do you know your RAG system is good?”
- debug retrieval vs generation failures
- discuss enterprise security and scalability

---

## Stage 5 — Tool Calling, LangGraph & Agents
**Window:** Dec 2026–Jan 2027

Learn:
- tool/function calling
- agent loops
- state
- planning vs workflows
- LangGraph
- checkpoints
- human-in-the-loop
- memory patterns
- retries and failure recovery
- multi-agent trade-offs

Build:
- tool-using assistant
- stateful LangGraph workflow
- agent with guarded/reliable execution

Interview bar:
- distinguish workflow from agent
- explain when *not* to use agents
- design reliable agent execution

---

## Stage 6 — MCP & Agent Architecture
**Window:** Jan 2027

Learn:
- MCP concepts
- client/server architecture
- tools/resources/prompts
- security boundaries
- enterprise tool integration
- agent interoperability

Build:
- small MCP server
- agent consuming MCP tools

Interview bar:
- explain why MCP exists
- compare direct tool integrations with MCP
- discuss authentication/security/deployment implications

---

## Stage 7 — GenAI/FDE System Design
**Window:** Jan–Feb 2027

Learn and practise:
- multi-tenant GenAI systems
- API gateway and authentication
- model gateways
- queues and asynchronous processing
- caching
- vector infrastructure
- observability
- evaluation
- cost controls
- PII/data governance
- resilience
- scaling
- deployment patterns

FDE scenarios:
- ambiguous customer requirement → technical design
- prototype → production
- integration with existing enterprise systems
- debugging in customer environments
- communicating trade-offs to technical/non-technical stakeholders

Build:
- architecture case studies
- system-design documents
- implementation spikes

Interview bar:
- conduct 45–60 minute system-design discussions
- clarify requirements before designing
- defend trade-offs rather than reciting architectures

---

## Stage 8 — Portfolio & Interview Sprint
**Window:** Feb–Mar 2027

### Flagship projects

**Project A — Enterprise RAG Platform**
- ingestion
- hybrid retrieval
- reranking
- citations
- evaluation
- API
- observability
- security considerations

**Project B — Agentic Developer / Operations Assistant**
- tools
- LangGraph
- controlled execution
- MCP integration
- state/checkpointing
- auditability

Projects must have:
- clear problem statement
- architecture
- runnable code
- tests
- meaningful README
- design decisions/trade-offs
- known limitations

### Interview sprint

Practise repeatedly:
- Python coding
- GenAI fundamentals
- RAG
- agents
- system design
- FDE customer scenarios
- project deep dives
- behavioural/leadership stories

Every failed or weak answer goes into FAILED_QUESTIONS.md.

---

# Weekly operating model

A normal week should include:
- concept learning
- hands-on implementation
- one code review/refactor session
- interview questions
- one cumulative revision session

Do not wait until March to start interviewing practice. Interview questions begin during Stage 1.

# Definition of interview-ready

Interview-ready does **not** mean finishing every topic.

It means being able to:
1. code comfortably in Python under interview conditions;
2. explain LLM/RAG/agent fundamentals without memorised scripts;
3. build and debug a realistic GenAI service;
4. design production GenAI architectures and defend trade-offs;
5. deeply explain portfolio and professional projects;
6. handle ambiguous FDE/customer scenarios;
7. give concise senior-level behavioural examples;
8. recover well when an interviewer pushes beyond the first answer.

---

# Evidence-driven adjustments — 22 September 2026

Based on the first three reported interview experiences, apply these changes without discarding the stage structure:

- **Start a parallel DSA lane in Stage 1:** 2 problems/week initially; focus on arrays, strings, hash maps, two pointers/sliding window, stack/queue, binary search and core patterns. Kadane's algorithm is the first reported problem.
- **Expand Stage 1B:** SQLAlchemy engine/session/connection pooling joins FastAPI and async.
- **Pull retrieval intuition forward:** during Python learning, use document/query examples so Stage 3 is not the first exposure to retrieval.
- **Deepen Stage 3:** BM25, dense vs sparse, hybrid retrieval, RRF, reranking, structure-aware/parent-child chunking, context expansion, multi-hop retrieval and distributed evidence across long documents.
- **Add graph retrieval:** Neo4j/knowledge-graph fundamentals after baseline RAG; focus on when relationships justify a graph.
- **Thread production concerns through Stages 2–5:** cost, security, observability, traceability and evaluation are not postponed until Stage 7.
- **Stage 5 must include memory engineering:** short-term state, summaries, long-term/semantic memory, context compaction and failure diagnosis.
- **Stage 6 adds A2A:** compare MCP (tool/context interoperability) with A2A (agent interoperability).
- **Add an inference engineering module before/within Stage 7:** serving architecture, TTFT/latency/throughput, batching, KV cache, quantization, autoscaling and managed-vs-self-hosted benchmarking.
- **Add a compact ML fundamentals lane:** logistic regression/sigmoid, SVM/kernels and time-series differencing. Keep P2 until further evidence raises it.
- **Cloud-specific lane:** learn portable concepts first; map them to AWS Bedrock/AgentCore as a secondary implementation track.
