# Agent approval gates and MCP transport boundaries

This lesson supports [Batch 014](12-reported-interview-experiences/batch-014-2026-11-27-agent-approval-mcp-transport.md) and the [Enterprise Document Intelligence Assistant](PROJECT_ENTERPRISE_DOCUMENT_ASSISTANT.md).

## Dependency order

1. Read [GenAI concept foundations](00-genai-concept-foundations.md).
2. Read [Agent loops and repeatability](01-agent-loops-and-repeatability.md).
3. Understand a normal Python function and an HTTP API.
4. Build the read-only FastAPI assistant before adding side-effecting tools.
5. Add approval and remote MCP only after the local tool path is tested.

## 1. Why agent actions need approval gates

An LLM predicts text. A tool converts a model-produced request into an action such as sending an email, changing a ticket or deleting a document. The LLM may misunderstand the user, select the wrong record or follow malicious text retrieved from a document.

A **human-in-the-loop approval gate** pauses a workflow before a risky action. A person sees a stable proposal, accepts or rejects it, and the workflow resumes from saved state.

**MuleSoft analogy:** this is a long-running orchestration with a human task. The flow persists a correlation ID and payload, waits for an external decision, then resumes idempotently. It must not replay earlier side effects.

### Risk tiers

| Tier | Examples | Default |
|---|---|---|
| Read-only | search policy, fetch ticket | execute after authorization |
| Reversible write | add draft comment, create draft ticket | policy-dependent approval |
| External or sensitive write | send email, publish message, change access | approval required |
| Destructive | delete data, revoke access, transfer funds | strong approval plus additional controls |

Risk is determined by code and policy, not by the model.

### State model

A proposed action should contain:

- proposal ID;
- requester and tenant;
- tool name and validated arguments;
- human-readable impact summary;
- risk tier;
- creation and expiry time;
- policy version;
- status: pending, approved, rejected, expired or executed;
- idempotency key.

The approved object is immutable. If arguments change, create a new proposal and ask again.

### Minimal Python design

```python
from dataclasses import dataclass
from enum import Enum
from uuid import uuid4

class Status(str, Enum):
    PENDING = "pending"
    APPROVED = "approved"
    REJECTED = "rejected"
    EXECUTED = "executed"

@dataclass(frozen=True)
class ToolProposal:
    proposal_id: str
    tenant_id: str
    requester_id: str
    tool: str
    arguments: dict
    risk: str
    idempotency_key: str

def propose(tool: str, arguments: dict, principal) -> ToolProposal:
    validated = TOOL_SCHEMAS[tool].model_validate(arguments).model_dump()
    risk = RISK_POLICY.classify(tool, validated)
    return ToolProposal(
        proposal_id=str(uuid4()),
        tenant_id=principal.tenant_id,
        requester_id=principal.user_id,
        tool=tool,
        arguments=validated,
        risk=risk,
        idempotency_key=str(uuid4()),
    )
```

Validation occurs before showing the proposal. Authorization is checked again immediately before execution because permissions can change while approval is pending.

### FastAPI approval endpoints

```python
@app.post("/proposals/{proposal_id}/approve")
async def approve(proposal_id: str, principal=Depends(current_principal)):
    proposal = await store.get_for_update(proposal_id)
    require_approver(principal, proposal)
    await store.transition(proposal_id, "pending", "approved")
    queue.publish({"proposal_id": proposal_id})
    return {"proposal_id": proposal_id, "status": "approved"}

async def execute_approved(proposal_id: str):
    proposal = await store.get_for_update(proposal_id)
    if proposal.status == "executed":
        return proposal.result                 # idempotent replay
    assert proposal.status == "approved"
    authorize_again(proposal)
    result = await TOOLS[proposal.tool](**proposal.arguments)
    await store.mark_executed(proposal_id, result)
    return result
```

In production, the external tool should also accept an idempotency key, or execution and state transition need a transactional outbox pattern.

### LangGraph placement

```text
understand request
 -> choose tool
 -> validate and authorize
 -> classify risk
 -> low risk: execute
 -> high risk: persist proposal -> interrupt
 -> human decision
 -> reject: explain and end
 -> approve: reauthorize -> execute once -> audit
```

A LangGraph interrupt is a workflow pause, not the approval policy itself. Durable storage, authorization, expiry and idempotency remain application responsibilities.

### Failure cases

- **Process restarts while waiting:** resume from a database/checkpointer, not process memory.
- **Approval is duplicated:** atomic status transition accepts only the first decision.
- **Worker crashes after the external action:** use the same idempotency key when retrying.
- **Arguments change:** invalidate approval and create a new proposal.
- **Permission changes:** reauthorize before execution.
- **Retrieved prompt asks for bypass:** tool gateway ignores document instructions and enforces policy.

## 2. MCP and why transport choice matters

The **Model Context Protocol (MCP)** standardizes how an AI application discovers and invokes tools, reads resources and uses reusable prompts.

- **Host:** the user-facing AI application.
- **Client:** the host-side component maintaining a connection to one MCP server.
- **Server:** exposes tools, resources and prompts.
- **Tool:** an operation that may compute or cause an effect.
- **Resource:** data the client can read.
- **Prompt:** a reusable prompt template.

MCP standardizes the integration boundary. LangGraph orchestrates workflow state and branching. They solve different problems and can be used together.

**MuleSoft analogy:** MCP resembles a standard connector contract between an orchestration layer and capability providers. LangGraph resembles the process orchestration that decides which connector to call and what happens next.

### stdio transport

With **stdio**, the host starts the server as a child process and exchanges protocol messages over standard input and output.

Use it when:

- client and server run on one trusted machine;
- building local developer tools;
- avoiding an exposed network port;
- process lifecycle can be owned by the host.

Trade-offs:

- simple local deployment;
- process isolation;
- awkward for shared remote services;
- logs must go to stderr so protocol output is not corrupted;
- authentication is usually inherited from the local environment.

### Streamable HTTP transport

With **Streamable HTTP**, the server is reachable over HTTP and can support remote, shared or independently deployed capabilities.

Use it when:

- server and host deploy separately;
- many clients need one service;
- API gateway, workload identity, rate limits and centralized observability are needed;
- network latency and partial failure are accepted.

Trade-offs:

- familiar enterprise operations and scaling;
- requires TLS, authentication, authorization, origin/network controls, timeouts and quotas;
- introduces network retries and duplicate-request risk;
- sessions and streaming require explicit lifecycle design.

### Transport does not replace security

For either transport:

1. validate tool arguments;
2. derive identity from a trusted channel;
3. authorize action and resource;
4. enforce tenant isolation;
5. limit time, output and concurrency;
6. log metadata without secrets;
7. use idempotency for side effects;
8. require approval for risky actions.

### Project topology

Start local:

```text
FastAPI host -> MCP client -> local stdio document-tool server
```

Then move the same tool contract behind a remote boundary:

```text
FastAPI host -> MCP client -> HTTPS gateway -> Streamable HTTP MCP server
```

The LangGraph nodes should depend on a small client interface, not transport details:

```python
from typing import Protocol

class DocumentTools(Protocol):
    async def search(self, query: str, tenant_id: str) -> list[dict]: ...
    async def create_ticket(self, title: str, tenant_id: str,
                            idempotency_key: str) -> dict: ...

async def search_node(state, tools: DocumentTools):
    state["evidence"] = await tools.search(
        query=state["question"],
        tenant_id=state["principal"].tenant_id,
    )
    return state
```

A local adapter and a remote MCP adapter can implement the same protocol. Tests can use a fake adapter.

## Progressive lab

### Step A: local read-only tool

Expose document search through stdio. Verify 20 queries return the same IDs as the direct Python implementation.

### Step B: remote read-only tool

Deploy the same contract over Streamable HTTP. Add identity propagation, tenant checks, a 2-second timeout and trace IDs.

Measure:

- success rate;
- p50 and p95 latency;
- unauthorized calls blocked;
- duplicate calls;
- protocol/schema errors.

### Step C: risky write

Add create_ticket as a side effect. Route it through the proposal/approval state machine. Kill and restart the process while approval is pending, then submit the decision twice.

**Expected result:** the workflow resumes after restart, one ticket is created, duplicate approval does not duplicate the ticket, and the audit record links request, proposal, approver and result.

## Interview checklist

A strong explanation separates:

- model suggestion from policy decision;
- workflow pause from durable persistence;
- approval from authorization;
- MCP contract from LangGraph orchestration;
- local transport convenience from remote service requirements;
- successful tool response from exactly-once business effect.

Readiness remains **Untested** until the lab passes and the design can be explained without notes.
