# Enterprise RAG isolation and federated source access

**Audience:** experienced API/integration developer beginning GenAI.  
**Read first:** [GenAI concept foundations](00-genai-concept-foundations.md), then [RAG ranking, freshness and resilience](02-rag-ranking-freshness-resilience.md).  
**Project:** [Enterprise Document Intelligence Assistant](PROJECT_ENTERPRISE_DOCUMENT_ASSISTANT.md).

## 1. The two problems

Enterprise RAG must solve:

1. **Isolation:** one tenant must never retrieve another tenant's evidence.
2. **Federation:** information may remain in document stores, ticket systems, databases and communication platforms instead of being copied into one index.

RAG quality is irrelevant if authorization is wrong.

**MuleSoft analogy:** a Process API may orchestrate Salesforce, SAP and a database, while still propagating identity and applying resource policy at every System API.

## 2. Prerequisites

- A **tenant** is an organization or isolated customer boundary.
- **Authentication** proves who the caller is.
- **Authorization** decides which resources the caller may access.
- A **vector index** retrieves semantically similar chunks.
- **Metadata filtering** restricts candidates by tenant, document or classification.
- **Federated retrieval** queries several source systems and combines authorized evidence.

## 3. Prompt instructions are insufficient

A prompt saying “show only this tenant's documents” is guidance, not access control.

```text
Unsafe: search every tenant -> send results to model -> ask model to hide data
Safe: authenticate -> authorize -> filter retrieval -> rerank authorized chunks
      -> assemble prompt -> generate -> validate citations
```

Unauthorized text must never enter model context, logs or shared caches.

## 4. Minimal FastAPI boundary

```python
from dataclasses import dataclass
from fastapi import Depends, FastAPI, HTTPException

app = FastAPI()

@dataclass(frozen=True)
class Principal:
    user_id: str
    tenant_id: str
    roles: frozenset[str]

def current_principal() -> Principal:
    # Teaching stub. Production validates a signed token.
    return Principal("user-17", "bank-a", frozenset({"policy-reader"}))

@app.get("/search")
def search(q: str, principal: Principal = Depends(current_principal)):
    if "policy-reader" not in principal.roles:
        raise HTTPException(403, "not authorized")

    return vector_store.search(
        query=q,
        filters={
            "tenant_id": principal.tenant_id,
            "allowed_roles": {"contains": "policy-reader"},
            "status": "active",
        },
        top_k=10,
    )
```

**Expected:** a bank-a caller cannot retrieve a bank-b chunk even when that chunk has the highest similarity.

Real filter syntax depends on the vector database.

## 5. Defense in depth

Use independent controls:

- dedicated collections or namespaces where stronger isolation is required;
- tenant and ACL filters during retrieval;
- authorization again when fetching full source text;
- tenant-scoped cache keys;
- least-privilege service credentials;
- redacted logs;
- source IDs and authorization decisions in traces;
- negative tests that deliberately target another tenant;
- deletion and permission propagation to indexes and caches.

Derive tenant identity from verified claims, never a client-supplied field.

## 6. Federated sources

Different sources need different tools:

| Source | Appropriate access |
|---|---|
| Policy PDFs | hybrid document retrieval |
| Tickets | structured status/filter query |
| Wiki pages | source-native search plus page fetch |
| Team messages | authorized channel search |
| Transaction database | parameterized SQL or governed API |

Normalize results only after authorization. Required evidence fields are source type, source ID, tenant, version, retrieval time, text and authorization decision.

A LangGraph state can hold question, principal, required sources, evidence, errors, deadline and stop reason. Nodes query sources; deterministic edges handle missing permission, timeout and partial results.

## 7. Partial failure and freshness

If documents succeed but tickets time out, do not imply a complete answer. Return a labeled partial result, retryable status or escalation.

Record source freshness. A vector entry can be older than its source platform. For high-value facts, fetch the current record by ID before answering.

## 8. Measurable lab

Create two synthetic tenants, each with ten similar policies, five tickets and five messages. Test:

1. valid same-tenant query;
2. explicit other-tenant ID;
3. semantically close other-tenant chunk;
4. shared cache under concurrent users;
5. one source timeout;
6. stale indexed content.

Measure unauthorized evidence, recall, source selection, cache leakage, partial-result labeling, p95 latency and version agreement.

**Acceptance:** zero unauthorized chunks in retrieval, prompts and responses; every partial response identifies missing sources; no cross-user cache result.

## 9. Alternatives and trade-offs

- **Shared collection plus filters:** efficient; filter defects have a larger blast radius.
- **Collection per tenant:** stronger separation; more operational overhead.
- **Copy everything into one index:** simple query; harder freshness, deletion and permissions.
- **Query sources live:** freshest; higher latency and availability dependence.
- **Hybrid:** index stable text and fetch volatile records live; more orchestration.
- **Agent source selection:** flexible; less predictable. Start deterministic.

## 10. Common mistakes

- authorizing after vector search;
- trusting client tenant IDs;
- cache keys without tenant and permission version;
- credentials inside prompts;
- confidential prompt logging;
- confident answers after required-source failure;
- stale copied ACLs;
- happy-path-only evaluation.

## 11. Interview answers

**Beginner:** “I propagate verified identity into every retrieval call and filter by tenant and permissions before text reaches the model. For several systems, I query each through an authorized connector, preserve source/version metadata and clearly report partial failure.”

**Senior:** “I enforce isolation at storage, retrieval, source fetch, prompt and cache boundaries. Each connector applies source-native ACLs and least privilege. I test cross-tenant negatives, cache leakage, stale permissions, partial failure and version drift while tracing decisions without logging confidential content.”

## 12. Further learning

Teaching references, separate from interview evidence:

- [OWASP LLM Sensitive Information Disclosure](https://genai.owasp.org/llmrisk/llm02-sensitive-information-disclosure/) — threat vocabulary.
- [Microsoft multitenancy guidance for Azure AI Search](https://learn.microsoft.com/azure/architecture/guide/multitenant/service/ai-search) — shared versus dedicated isolation.
- [Pinecone multitenancy](https://docs.pinecone.io/guides/index-data/implement-multitenancy) — namespace example.
- [LangGraph Graph API](https://docs.langchain.com/oss/python/langgraph/graph-api) — state and conditional routing.

Verify API syntax against the provider actually used.
