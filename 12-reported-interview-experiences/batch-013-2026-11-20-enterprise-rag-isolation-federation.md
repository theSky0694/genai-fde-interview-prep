# Batch 013 — Enterprise RAG isolation and federated sources

**Collected:** 20 November 2026. **Readiness:** Untested.

## Evidence note

This batch adds one independently posted candidate account:

- **Source:** [Vivek Kumar — Teradata Cloud AI Engineer interview](https://www.linkedin.com/posts/vivek7282_opentowork-hiring-softwareengineer-activity-7500497980702195712-JVFe).
- **Firsthand:** the author states that he personally completed four eliminatory rounds for a Cloud AI Engineer role at Teradata.
- **Date limitation:** LinkedIn displayed **“2mo Edited”** when verified on 20 November 2026 and exposed no exact date. This is newly discovered older evidence.
- **Reported scope:** DSA, Kubernetes, rate limiting, vector-store internals, data isolation, internal-document RAG, extension to communication channels and Confluence, integrations and security.
- **Wording:** the questions below are transparent paraphrases of listed design prompts.
- **Counting:** one candidate account counts as one independent experience. The counter increases from 17 to 18.

Most topics overlap earlier batches. The two themes taught deeply here are tenant-safe retrieval and federated RAG across enterprise systems.

## Dependency order

1. [GenAI concept foundations](../00-genai-concept-foundations.md).
2. [RAG ranking, freshness and resilience](../02-rag-ranking-freshness-resilience.md).
3. [Enterprise RAG isolation and federation](../04-enterprise-rag-isolation-federation.md).
4. Reliability increment 4 in the [project guide](../PROJECT_ENTERPRISE_DOCUMENT_ASSISTANT.md).
5. Beginner answers first; senior answers after the lab.

---

## 1. How would you enforce security and tenant isolation in vector-search RAG?

**Source:** paraphrased from Teradata Round 2.  
**Roadmap:** Stages 4 and 7. **Priority:** P0.

### What it tests

Whether you treat RAG as a secure data system, rather than a chatbot with a prompt saying “do not reveal confidential information.”

### Prerequisites

Authentication identifies the caller. Authorization decides what that identity may access. Vector similarity measures relevance, not permission. Metadata filters constrain candidates before content is read.

**MuleSoft analogy:** an authenticated Process API caller still needs scopes and resource authorization downstream.

### Project placement

```text
JWT validation
 -> Principal(user, tenant, roles)
 -> authorized vector query
 -> authorized source fetch
 -> reranking
 -> prompt assembly
 -> answer and citation validation
```

Unauthorized chunks must never enter reranking, prompts, shared caches or logs.

### Annotated code

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Principal:
    user_id: str
    tenant_id: str
    roles: frozenset[str]

def retrieve(question: str, principal: Principal):
    if "policy-reader" not in principal.roles:
        raise PermissionError("missing policy-reader role")

    return vector_store.search(
        query=question,
        top_k=10,
        filters={
            "tenant_id": principal.tenant_id,
            "allowed_roles": {"contains": "policy-reader"},
            "status": "active",
        },
    )
```

**Expected:** a bank-a caller sees no bank-b chunk even if it scores higher.

### Why this approach

Pre-filtering minimizes exposure. Rechecking at source fetch adds defense in depth. Tenant identity comes from verified claims.

### Alternatives and trade-offs

- Shared index plus filters is efficient but increases filter-bug blast radius.
- Tenant namespaces strengthen separation but add operations.
- Dedicated deployments isolate best but cost more.
- Post-filtering is too late.
- Prompt rules are not authorization.

### Beginner answer

“I verify the user, derive tenant and roles from trusted claims, and filter the vector query before content reaches the model. I authorize again at source fetch, scope caches by tenant and permission version, redact logs and test cross-tenant attacks.”

### Senior answer

“I enforce isolation at storage, retrieval, source fetch, prompt and cache boundaries. I choose shared metadata, namespaces or dedicated indexes by regulatory and blast-radius needs, then test explicit foreign IDs, semantic near-matches, permission changes, deletion, concurrent caches and trace redaction.”

### Follow-ups

**Why filter before search?**  
It prevents foreign chunks from becoming candidates, reranker inputs or log content.

**Can tenant ID come from the body?**  
No. Use verified token/session claims.

**What belongs in a cache key?**  
Tenant, user or permission scope, corpus version, retrieval configuration and normalized query.

**What if permissions change?**  
Invalidate caches and update ACL metadata; recheck source permission at fetch for sensitive data.

### Common mistakes

Authentication mistaken for authorization; post-retrieval filtering; client tenant IDs; unscoped caches; full prompt logging; no negative tests; stale restricted chunks.

### Exercise

Create two tenants with ten similarly worded policies. Run valid, explicit foreign-ID, semantic near-match and concurrent-cache tests.

**Expected:** zero foreign IDs in retrieval, prompt and response traces. Removing the filter must make the negative test fail, proving the test is meaningful.

---

## 2. Design internal-document RAG and extend it to communication channels and Confluence-like systems

**Source:** paraphrased from Teradata Round 3.  
**Roadmap:** Stages 5–7. **Priority:** P0.

### What it tests

Whether you can integrate heterogeneous systems without treating every source as an interchangeable vector database.

### Prerequisites

**Federated retrieval** queries several sources. Policies use hybrid text retrieval; tickets use structured queries; wiki pages use source search plus fetch; messages use authorized channel search; databases use parameterized APIs or SQL.

**MuleSoft analogy:** a Process API orchestrates System APIs and produces a canonical response without forcing all systems into one database.

### Project placement

```text
question + principal
 -> source router
 -> policy node
 -> ticket node
 -> message/wiki node
 -> normalize authorized evidence
 -> completeness check
 -> grounded or partial answer
```

Begin with deterministic routing.

### Annotated state

```python
from typing import Literal, TypedDict

class State(TypedDict):
    question: str
    tenant_id: str
    required_sources: list[Literal["policy", "ticket", "message", "wiki"]]
    evidence: list[dict]
    errors: list[str]
    deadline_ms: int

def route_sources(state: State) -> State:
    q = state["question"].lower()
    sources = ["policy"]
    if "status" in q or "owner" in q:
        sources.append("ticket")
    if "discussed" in q or "decision" in q:
        sources.extend(["message", "wiki"])
    state["required_sources"] = list(dict.fromkeys(sources))
    return state
```

Each connector returns source type, ID, version, tenant, retrieval time and authorization decision.

**Expected:** a question about the policy governing ticket 42 and the related decision selects policy, ticket and message/wiki nodes and cites each successful source.

### Why this approach

Source-native queries preserve structure, freshness and permissions. Typed evidence gives generation a stable contract. Explicit errors prevent silent incompleteness.

### Alternatives and trade-offs

- One vector index is simple but weak for freshness, ACLs and deletion.
- Live queries are fresh but slower.
- Hybrid indexing plus live volatile lookup adds orchestration.
- LLM routing is flexible but unpredictable.
- Deterministic routing is testable but needs maintained rules.
- Parallel calls reduce latency but may waste calls.
- Sequential calls are adaptive but slower.

### Beginner answer

“I keep every enterprise system behind an authorized connector and use source-native retrieval. LangGraph carries caller identity, required sources, evidence and errors. I normalize authorized results, preserve source IDs and versions, and label partial answers when a required source fails.”

### Senior answer

“I separate routing, connector execution, normalization, evidence sufficiency and generation. Identity, deadline and trace context propagate to every call. Stable documents may be indexed and volatile records fetched live. I measure routing, evidence recall, freshness, partial-failure correctness, authorization violations, latency and tool cost.”

### Follow-ups

**Why not embed ticket status?**  
Embeddings help semantic descriptions but are poor for exact current status, assignee or SLA. Query structured fields.

**What if a source times out?**  
Respect the end-to-end deadline and return a labeled partial or retryable result when that source is required.

**Call everything in parallel?**  
Only when independently necessary. Avoid waste and rate-limit pressure.

**How do you handle prompt injection in messages?**  
Treat retrieved text as untrusted evidence, not instructions. Tools and permissions remain enforced in code.

### Common mistakes

Embedding structured status; losing source ACLs; unrestricted tool choice; hiding partial failures; no deadline propagation; missing timestamps; calling every source for every question.

### Exercise

Add synthetic policy, ticket and message connectors. Create 30 questions: ten single-source, ten two-source and ten with a timeout or permission denial.

**Expected:** at least 90% routing accuracy, zero unauthorized evidence, correct partial labels and complete citations.

## Study plan

Read the concept lesson; build two tenants and negative tests; add typed connectors; implement routing; inject timeout and denial; record metrics; speak beginner answers. Readiness remains Untested.
