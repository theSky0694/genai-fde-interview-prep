# Batch 012 — Context quality and grounding diagnosis

**Collected:** 13 November 2026. **Readiness:** Untested.

## Evidence note

This batch adds one independently posted candidate account:

- **Source:** [Harshavardhan Reddy Bogati — LinkedIn profile](https://in.linkedin.com/in/harshavardhan-reddy-bogati-42423b334). The publicly indexed post is titled **“My First GenAI Engineer Interview Experience — 15 Tricky Questions I Faced”** and reports a GenAI Level 1 Engineer interview at an unnamed startup in Pune. Search access exposed the full question list through LinkedIn profile indexing, but did not expose a stable direct post URL or exact calendar date. A separate LinkedIn index displayed the post as **“1w”** during this collection. This batch therefore records neither an invented date nor a company name.
- **Firsthand status:** the author says they personally attended the interview.
- **Wording:** the two questions below are short paraphrases of the visible questions, not claimed verbatim transcripts.
- **Counting:** fifteen questions came from one account, so the independent-experience counter increases by **one**, from 16 to 17.

Most reported topics overlap earlier batches: RAG architecture, split chunks, agent loops, MCP, cosine similarity and production debugging. This batch selects only two closely related questions that add a deeper missing theme: **context utilization after retrieval succeeds**.

## Dependency order

1. Read [GenAI concept foundations](../00-genai-concept-foundations.md), especially tokens, context, RAG and evaluation.
2. Read [Context quality, grounding and lost-in-the-middle](../03-context-quality-and-grounding.md).
3. Use [RAG ranking, freshness and resilience](../02-rag-ranking-freshness-resilience.md) to separate retrieval from ranking.
4. Implement Reliability increment 3 in the [Enterprise Document Intelligence Assistant](../PROJECT_ENTERPRISE_DOCUMENT_ASSISTANT.md).
5. Practise the beginner answers first; attempt senior answers after running the experiments.

---

## 1. The correct information is in the prompt, but the model gives the wrong answer. Why, and where do you debug?

**Source:** paraphrased from the firsthand Pune startup report.  
**Roadmap:** Stage 4 — grounded generation and evaluation. **Priority:** P0.

### What problem this question tests

It tests whether you understand that successful retrieval does not prove successful generation. Interviewers want a layer-by-layer diagnosis rather than “try another prompt” or “use a bigger model.”

### Prerequisites from first principles

An LLM predicts output tokens from everything in its input. It does not perform an exact database read. The prompt may contain:

- system instructions;
- the user's question;
- conversation history;
- retrieved chunks;
- tool results;
- output-format rules.

The correct fact can lose influence when nearby text is irrelevant, conflicting or presented unclearly.

**MuleSoft analogy:** a downstream transformation can produce the wrong field even though the upstream API returned it correctly. You inspect mapping, precedence, filtering and payload version before blaming the source system.

### Exact project placement

```text
retrieval -> authorization -> deduplication -> reranking
          -> context assembly -> LLM generation -> citation validation
                                  ^ investigate here when evidence arrived
```

The application should trace source IDs and prompt positions without recording confidential document text.

### Annotated example

```python
def build_grounded_request(question: str, chunks: list[dict]) -> dict:
    evidence_blocks = [
        {
            "source_id": chunk["id"],
            "version": chunk["version"],
            "text": chunk["text"],
        }
        for chunk in chunks
    ]

    return {
        "instruction": (
            "Answer only from evidence. "
            "If it is insufficient or conflicting, say so. "
            "Cite source_id for every factual claim."
        ),
        "question": question,
        "evidence": evidence_blocks,
    }
```

**Input:**

```python
[
    {"id": "policy:p4", "version": 3,
     "text": "Tier B daily transfer limit is ₹75,000."},
    {"id": "policy:p4", "version": 2,
     "text": "Tier B daily transfer limit is ₹50,000."},
]
```

**Expected safe result:** the application detects conflicting versions before generation and keeps only the active version, or the model returns that the evidence conflicts. It must not silently choose one.

### Debugging order and why

1. **Confirm prompt inclusion.** Retrieval logs may show a chunk that a later token-budget step dropped.
2. **Inspect versions and conflicts.** Correct and obsolete values can both be present.
3. **Remove duplicates and irrelevant chunks.** Noise may weaken evidence use.
4. **Check instruction/evidence boundaries.** Document text must not be treated as system authority.
5. **Test evidence position.** Hold all text constant and move the required chunk.
6. **Freeze decoding and model.** Reproduce before changing variables.
7. **Validate citation support.** A plausible answer and valid-looking source ID are insufficient.

### Why this approach

It localizes the failure. Changing retrieval when the required evidence already reached the prompt can hide the real problem and make later regressions harder to explain.

### Alternatives and trade-offs

- **Stronger prompt:** cheap, but still probabilistic.
- **Fewer chunks:** reduces noise; risks omitting multi-document evidence.
- **Reranker:** improves ordering; adds latency.
- **Structured fact extraction:** easier to validate for tables and policy fields; requires a schema.
- **Extractive response:** highly grounded; less conversational.
- **Larger model:** may use context better; costs more and does not remove conflicts.
- **Human review:** suitable for consequential or conflicting cases; adds queue time.

### Beginner-friendly interview answer

“I separate retrieval from generation. First I prove the required passage actually reached the final prompt. Then I check duplicate, obsolete or conflicting chunks, the way evidence is labeled, its position, and whether the answer's citation really supports the claim. I reproduce the failure on a fixed example and change one factor at a time.”

### Senior-level interview answer

“I trace the evidence lifecycle from corpus version and retrieval rank through authorization, reranking, prompt inclusion and token position. With the request frozen, I run noise, conflict and position ablations, validate entailment between claims and citations, and compare models only after prompt assembly is ruled out. Release metrics separate evidence recall, supported-answer accuracy, refusal and citation validity.”

### Follow-ups with explained answers

**How do you prove retrieval succeeded?**  
Label the required source for each evaluation question and calculate Recall@K. Also verify that the source survived filtering and appeared in the final request, because retriever output alone is insufficient.

**Could temperature be the cause?**  
It can increase variation, but lowering it does not repair missing, contradictory or poorly structured evidence. Freeze it during diagnosis, then test separately.

**What if two policy versions conflict?**  
Resolve version precedence in application code using active version and effective date. Do not ask the model to infer authoritative policy.

**How do you validate citations?**  
For each factual claim, verify that the cited source ID was supplied and its text supports the claim. Begin with exact checks for structured facts, then add semantic or human evaluation where needed.

### Common mistakes

- Increasing top-k when the evidence already arrived.
- Changing prompt, model and retrieval together.
- Logging raw confidential prompts.
- Trusting a citation ID without checking its content.
- Letting the model decide which policy version is authoritative.
- Calling every wrong answer a hallucination without locating the failing stage.

### Practical exercise

Create ten fixed questions with known sources. For each, run a clean prompt, a prompt with ten irrelevant chunks and a prompt containing an obsolete conflicting chunk.

**Expected result:** a table of supported-answer accuracy, refusal, citation validity, input tokens and latency for all three conditions. Your version filter should prevent every obsolete value from reaching generation.

---

## 2. Why does a larger context window not necessarily improve a RAG system?

**Source:** paraphrased from the same firsthand Pune startup report.  
**Roadmap:** Stages 3–4 — retrieval, context assembly and evaluation. **Priority:** P0.

### What problem this question tests

It tests whether you understand the difference between **capacity** and **quality**. A model may accept more text while still struggling to select the right evidence from noise.

### Prerequisites from first principles

A context window is the request's token capacity. RAG should retrieve the evidence needed for this question; it should not use the context window as a replacement for search.

More context can cause:

- higher cost and latency;
- duplicate evidence;
- conflicting versions;
- relevant passages buried among noise;
- reduced output-token space;
- prompt-injection exposure from more documents.

**MuleSoft analogy:** raising the maximum payload size permits a larger message; it does not improve the quality of data mapping. Good orchestration selects the required systems and fields.

### Exact project placement

After reranking, add a context selector:

```python
def assemble_context(ranked, token_budget, token_count):
    selected = []
    used = 0

    for chunk in ranked:
        size = token_count(chunk["text"])
        if chunk["is_active"] and used + size <= token_budget:
            selected.append(chunk)
            used += size

    return {
        "chunks": selected,
        "used_tokens": used,
        "source_ids": [c["id"] for c in selected],
    }
```

**Expected behavior:** inactive versions are excluded; the highest-ranked active evidence fits within the budget; telemetry records selected source IDs and token use.

This simple greedy method is a learning baseline. A multi-hop question may require preserving several complementary chunks even if one ranks lower.

### Why this approach

It treats context as a limited, testable resource. Retrieval can cast a wider net, while context assembly selects a compact evidence set for generation.

### Alternatives and trade-offs

- **Fixed top-k:** simple and predictable; ignores variable chunk sizes and question complexity.
- **Token budget:** controls cost; may prefer short incomplete chunks.
- **Long-context model:** useful for inseparable documents; more expensive and still affected by noise.
- **Context compression:** reduces tokens; can remove exceptions.
- **Hierarchical retrieval:** finds section, then passage; adds retrieval steps.
- **Multi-hop retrieval:** collects complementary facts; requires stopping and evidence-sufficiency logic.
- **Summarize documents first:** reusable; summaries can lose provenance and details.

### Beginner-friendly interview answer

“A context window is capacity, not guaranteed understanding. Adding more chunks can add noise, duplicates and conflicts while increasing cost and latency. I retrieve enough candidates for recall, rerank them, select the smallest complete evidence set under a token budget, and evaluate answer quality as I vary evidence position and chunk count.”

### Senior-level interview answer

“I tune context assembly on a labeled set rather than maximizing tokens. I track Recall@K before assembly, required-evidence coverage after assembly, supported-answer accuracy, refusal, citation validity, p95 latency and cost. I use adaptive budgets for question complexity, preserve linked evidence for multi-hop cases, resolve versions deterministically and test position sensitivity.”

### Follow-ups with explained answers

**Should final-k always be small?**  
No. One-fact questions may need one chunk; comparison or multi-document questions may need several. Use an evidence-completeness criterion and a maximum budget.

**What is ‘lost in the middle’?**  
Research has shown that models can use evidence less reliably depending on its position in a long input, often struggling with relevant material buried in the middle. Test your chosen model rather than assuming the pattern or fix.

**Can a huge model context replace a vector database?**  
Sometimes for a small, stable corpus. At enterprise scale you still need permissions, freshness, source selection, deletion, cost control and citations.

**How do you choose a token budget?**  
Start from the provider limit, reserve tokens for instructions, question and output, then tune evidence tokens using accuracy, refusal, latency and cost on representative questions.

### Common mistakes

- Equating accepted tokens with reliable evidence use.
- Sending whole documents without authorization filtering.
- Counting chunks instead of tokens.
- Truncating the end without checking what was lost.
- Omitting output-token headroom.
- Evaluating only easy single-fact questions.

### Practical exercise

Use 20 labeled questions. Run final-k values 2, 5, 10 and 20, then repeat with the required evidence placed first, middle and last.

**Expected result:** a plot or table showing supported-answer accuracy, evidence coverage, tokens, cost and p95 latency. Select the smallest setting that meets the quality gate; document any position sensitivity.

---

## Combined project increment

Add these trace fields to `POST /answer`:

```json
{
  "corpus_version": "policies-v3",
  "retrieved_ids": ["a", "b", "c"],
  "selected_ids": ["a", "c"],
  "evidence_token_count": 812,
  "required_source_present": true,
  "conflict_detected": false,
  "citation_support_passed": true
}
```

Keep the trace internal and redacted. Do not return diagnostic internals to normal users.

### Validation gate

The increment passes when:

- every labeled required source can be traced into or out of the final prompt;
- obsolete versions never enter generation;
- all factual answers have a supporting supplied citation;
- unanswerable questions return insufficient evidence;
- the chosen context budget is justified with quality, latency and token measurements.

No readiness status changes until this lab is implemented and explained.
