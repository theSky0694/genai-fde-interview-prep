# Batch 014 — Human approval gates and MCP transport boundaries

**Collected:** 27 November 2026. **Readiness:** Untested.

## Evidence note

This batch adds one independently posted candidate account:

- **Source:** [Som Raj — Deloitte GenAI Engineer L2 technical interview](https://medium.com/@somraj0611/deloitte-genai-engineer-interview-experience-l2-technical-round-45-minutes-questions-asked-45c5626f3644).
- **Firsthand:** the author says he recently attended the Deloitte GenAI Engineer L2 technical round and describes a 45-minute discussion.
- **Date limitation:** Medium’s indexed result displayed **“20 hours ago”** when verified on 27 November 2026 and identified the month as November 2026. The accessible index did not expose an exact calendar publication date, so none is invented here.
- **Reported scope:** business outcomes, end-to-end RAG and evaluation, agent architecture and tool choice, human approval, Azure deployment, Python/API development, and MCP architecture and transports.
- **Wording:** the two questions taught below are short questions shown directly in the indexed article. Explanations and follow-ups are original study material.
- **Counting:** one candidate account counts as one independent experience, regardless of question count. The experience counter increases from 18 to 19.

RAG evaluation, project defence and tool choice overlap earlier batches. This batch teaches two non-duplicative implementation drills: durable approval for risky actions and the stdio versus Streamable HTTP MCP boundary.

## Dependency order

1. [GenAI concept foundations](../00-genai-concept-foundations.md).
2. [Agent loops and repeatability](../01-agent-loops-and-repeatability.md).
3. [Agent approval and MCP transport lesson](../05-agent-approval-and-mcp-transport.md).
4. Implement the lab in the [Enterprise Document Intelligence Assistant](../PROJECT_ENTERPRISE_DOCUMENT_ASSISTANT.md).
5. Practise the beginner answer before the senior answer.

---

## 1. “How do you handle human approval and control risky actions in an agentic workflow?”

**Source wording:** exact short question from the reported Agentic AI section.  
**Roadmap:** Stages 5 and 7. **Priority:** P0.

### What problem it tests

An interviewer wants to know whether you understand that an LLM may propose an action but must not become the authorization system. Risky actions need policy checks, durable pause/resume, a clear approval payload, idempotent execution and an audit trail.

### Prerequisites from first principles

An **agent** is an application loop that lets a model choose an action, observes the result and decides what to do next. A **tool** is a typed function or API available to that loop. A **side effect** changes the outside world, for example sending an email or updating a ticket.

A **human approval gate** pauses before a risky side effect. The approver reviews the exact frozen action, not merely a vague chat message.

**MuleSoft analogy:** imagine an order orchestration that reaches a manual fraud-review step. The flow persists its correlation ID and state, waits outside the running process, then resumes once. The approval does not replace API authorization, input validation or idempotency.

### Where it fits in the running project

Add a create_ticket tool to the document assistant:

```text
POST /assist
 -> retrieve evidence
 -> model proposes create_ticket
 -> validate schema
 -> authorize caller
 -> classify risk
 -> persist pending proposal
 -> LangGraph interrupt
 -> POST /proposals/{id}/approve
 -> reauthorize
 -> execute with idempotency key
 -> persist result and audit
```

Read-only search may execute immediately after authorization. External writes require approval.

### Annotated implementation

```python
from typing import Literal, TypedDict

class AgentState(TypedDict):
    question: str
    tenant_id: str
    proposed_tool: dict | None
    proposal_id: str | None
    approval: Literal["pending", "approved", "rejected"] | None
    result: dict | None

async def approval_gate(state: AgentState):
    call = state["proposed_tool"]
    validated = TOOL_SCHEMAS[call["name"]].model_validate(call["args"])
    authorize_proposal(state["tenant_id"], call["name"], validated)

    if RISK_POLICY.requires_human(call["name"], validated):
        proposal = await proposals.create_once(
            tenant_id=state["tenant_id"],
            tool=call["name"],
            args=validated.model_dump(),
        )
        state["proposal_id"] = proposal.id
        state["approval"] = "pending"
        return interrupt({
            "proposal_id": proposal.id,
            "tool": proposal.tool,
            "arguments": proposal.args,
            "impact": describe_impact(proposal),
        })

    return await execute_tool_once(call)
```

- model output is schema-validated;
- policy code decides risk;
- the persisted proposal freezes the arguments;
- the interrupt returns reviewable data;
- execute_tool_once uses an idempotency key.

On resume:

```python
async def resume_after_decision(state: AgentState):
    proposal = await proposals.get_for_update(state["proposal_id"])

    if proposal.status == "rejected":
        return {**state, "approval": "rejected", "result": None}

    require_status(proposal, "approved")
    authorize_again(proposal)          # permissions may have changed
    result = await tool_gateway.execute(
        proposal.tool,
        proposal.args,
        idempotency_key=proposal.id,
    )
    await proposals.mark_executed_once(proposal.id, result)
    return {**state, "approval": "approved", "result": result}
```

**Expected input:** “Create a P1 ticket containing this diagnosis.”  
**Expected pause output:** proposal ID, ticket fields, impact and expiry.  
**Expected final output:** one ticket ID after approval, or a clear rejection without a tool call.

### Why this approach

It creates a hard boundary between probabilistic reasoning and deterministic control. Durable state survives restarts. Frozen arguments prevent approving one action and executing another. Reauthorization handles changed permissions. Idempotency makes retries safe.

### Alternatives and trade-offs

- **Approve every tool:** safest-looking but slow and causes approval fatigue.
- **Approve by risk tier:** practical, but the policy needs ownership and tests.
- **Approve an entire plan:** fewer prompts, but later actions may differ from what the user saw.
- **Approve every exact action:** stronger control, with more latency.
- **In-memory pause:** easy demo, but loses work on restart.
- **Database/checkpointer:** reliable, with operational complexity.
- **Model decides risk:** flexible, but too nondeterministic for the final policy decision.

### Beginner-friendly interview answer

“I let the model propose a tool call, but application code validates it, checks authorization and assigns a risk tier. For a risky write, I save the exact action, pause the LangGraph workflow and show it to an authorized person. After approval I check permissions again and execute with an idempotency key, so retries cannot create the action twice. I record who requested, approved and executed it.”

### Senior-level answer

“I use a policy-enforced tool gateway and a durable proposal state machine: pending, approved or rejected, expired, then executed. Approval binds to immutable normalized arguments and a policy version. Resume is driven by an authenticated endpoint or event, followed by reauthorization and an atomic or idempotent side effect. I test process death, duplicate decisions, stale permissions, argument tampering, expiry, prompt injection and audit redaction. Metrics include approval latency, override rate, false escalation and duplicate-effect count.”

### Follow-up questions and explained answers

**Is LangGraph interrupt enough?**  
No. It supplies a pause/resume mechanism. You still need durable storage, authenticated approval, authorization, expiry, idempotency and auditing.

**What if the tool arguments change after approval?**  
The approval is invalid. Create a new immutable proposal and request approval again.

**How do you avoid duplicate writes after a worker crash?**  
Send a stable idempotency key to the downstream service. Store the external result against the proposal. If the downstream system lacks idempotency, use a transactional outbox or reconciliation strategy.

**Should the approver be the requester?**  
Policy decides. High-risk operations may require separation of duties, where the requester cannot approve their own action.

**What should appear on the approval screen?**  
Exact target, normalized arguments, data sensitivity, predicted impact, requesting identity, expiry and a plain-language diff or preview.

### Common mistakes

- treating prompt instructions as authorization;
- saving state only in memory;
- approving a description while executing mutable arguments;
- no reauthorization after a long wait;
- retrying a side effect without idempotency;
- letting retrieved documents set the risk tier;
- logging full secrets or document content;
- approving every read and causing fatigue.

### Practical exercise

Implement create_ticket with pending, approved, rejected and executed states. Run five tests:

1. approve normally;
2. reject;
3. restart while pending;
4. submit approval twice;
5. revoke permission before approval.

**Expected result:** normal approval creates one ticket; rejection creates none; restart resumes; duplicate approval still creates one; revoked permission blocks execution. Readiness stays Untested until all five pass.

---

## 2. “What is the difference between stdio and Streamable HTTP transport in MCP?”

**Source wording:** exact short question from the reported MCP section.  
**Roadmap:** Stage 6. **Priority:** P1.

### What problem it tests

The interviewer is checking whether you can choose a deployment boundary, not merely expand the acronym MCP. You should explain process topology, identity, networking, failure handling and when each transport fits.

### Prerequisites from first principles

The **Model Context Protocol (MCP)** is a standard connection between an AI application and capability providers.

- the **host** is the AI application;
- a **client** connects the host to one MCP server;
- the **server** exposes tools, resources and prompts;
- a **transport** carries protocol messages.

**stdio** uses a child process’s standard input and output. **Streamable HTTP** uses an HTTP endpoint and supports a remote service boundary.

**MuleSoft analogy:** stdio resembles invoking a locally packaged connector or command on the same worker. Streamable HTTP resembles calling a separately deployed System API through a network and gateway. The business capability may be the same, but lifecycle, security, scaling and failure modes differ.

### Where it fits in the running project

First expose document search through a local MCP server:

```text
FastAPI/LangGraph host
 -> MCP client
 -> child Python process over stdio
 -> document search tool
```

Then deploy the same contract remotely:

```text
FastAPI/LangGraph host
 -> MCP client
 -> TLS/API gateway
 -> Streamable HTTP MCP server
 -> document search tool
```

LangGraph still owns state and branching. MCP supplies the tool integration contract.

### Annotated project interface

Keep orchestration independent of transport:

```python
from typing import Protocol

class SearchTools(Protocol):
    async def search(self, query: str, tenant_id: str) -> list[dict]:
        ...

async def retrieve_node(state, tools: SearchTools):
    # The node does not know whether tools uses stdio or HTTP.
    hits = await tools.search(
        query=state["question"],
        tenant_id=state["principal"].tenant_id,
    )
    return {**state, "evidence": hits}
```

Create two adapters:

```python
class LocalMCPTools:
    async def search(self, query, tenant_id):
        return await stdio_session.call_tool(
            "search", {"query": query, "tenant_id": tenant_id}
        )

class RemoteMCPTools:
    async def search(self, query, tenant_id):
        with trace_timeout(seconds=2):
            return await http_mcp_session.call_tool(
                "search", {"query": query, "tenant_id": tenant_id}
            )
```

The remote adapter must propagate authenticated identity through a trusted mechanism; accepting tenant_id from an untrusted model argument is insufficient.

### Why this approach

A common interface keeps business workflow tests stable while deployment evolves. Start with stdio for local development and move to Streamable HTTP when independent deployment, shared access or centralized scaling is justified.

### Alternatives and trade-offs

| Decision | stdio | Streamable HTTP |
|---|---|---|
| Topology | same machine, child process | remote or separately deployed service |
| Setup | simple local lifecycle | network, TLS and service operations |
| Sharing | usually one host-managed process | many clients can share a server |
| Identity | local process/environment boundary | explicit authentication and authorization |
| Scaling | scale with host processes | scale server independently |
| Failures | process exit, pipe corruption | timeout, network partition, retries, rate limits |
| Best fit | local IDE/desktop/private helper | enterprise/shared capability |

stdio is not automatically secure: a compromised local process or inherited credential can still be dangerous. HTTP is not automatically enterprise-ready: it needs TLS, authentication, authorization, timeouts, quotas and observability.

### Beginner-friendly interview answer

“stdio lets the host launch an MCP server locally and exchange messages through the process input and output. It is convenient for local tools and development. Streamable HTTP connects to an HTTP MCP server, so the server can deploy and scale separately and serve remote clients. That adds network latency and requires authentication, TLS, authorization, timeouts, retries and rate limits. MCP defines the tool boundary; LangGraph can still orchestrate the workflow.”

### Senior-level answer

“I choose transport from the trust and deployment boundary. stdio gives host-owned process lifecycle and no listening port, but needs strict environment, credential and stdout discipline. Streamable HTTP supports independently scaled shared services and gateway controls, while adding distributed-system failure modes, session lifecycle, retry/idempotency concerns and workload identity. I hide transport behind a typed client, enforce tenant authorization server-side, set deadlines and output limits, trace calls, and contract-test both adapters.”

### Follow-up questions and explained answers

**Does Streamable HTTP mean the tool is stateless?**  
No. HTTP transport does not guarantee stateless behavior. The server may maintain protocol sessions or application state. Keep business state explicit and durable.

**Can logs be printed to stdout with stdio?**  
Avoid it because stdout carries protocol messages. Send diagnostics to stderr or structured external logging.

**Where should authorization run?**  
At the server/tool boundary using trusted caller identity. The host can precheck for user experience, but the capability provider must enforce its own policy.

**How do retries affect tools?**  
Reads are generally safe to retry within a deadline. Writes need idempotency keys and clear retry rules because a lost response does not prove the action failed.

**Is MCP a replacement for LangGraph?**  
No. MCP standardizes access to tools, resources and prompts. LangGraph models workflow nodes, state, branches, loops and interrupts.

### Common mistakes

- saying HTTP is only “faster” or stdio only “older”;
- mixing MCP with agent orchestration;
- trusting model-supplied tenant identity;
- no remote timeout or retry budget;
- retrying writes without idempotency;
- writing logs to stdout in a stdio server;
- exposing a remote MCP server without authentication;
- tying every LangGraph node directly to one transport library.

### Practical exercise

Run the same 20 search requests through a local stdio adapter and a remote Streamable HTTP adapter. Add one unauthorized tenant request, one 3-second server delay and one forced connection drop.

Record:

- result parity;
- p50 and p95 latency;
- timeout behavior;
- unauthorized requests blocked;
- duplicate tool executions;
- trace IDs present.

**Expected result:** identical authorized document IDs, zero cross-tenant data, bounded timeout failure, no duplicate side effects and traceable calls. Explain why the remote version is operationally heavier even when its Python call looks similar.

## Study plan

1. Read the concept lesson.
2. Implement approval states and failure tests.
3. Run the same read-only tool through both transports.
4. Record evidence in the project README.
5. Speak the beginner answers in 60 seconds each.
6. Add senior details only after the labs pass.

Readiness remains **Untested**.
