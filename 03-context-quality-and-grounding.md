# Context quality, prompt grounding and “lost in the middle”

**Audience:** experienced API/integration developer beginning GenAI.  
**Use after:** [GenAI concept foundations](00-genai-concept-foundations.md).  
**Project:** [Enterprise Document Intelligence Assistant](PROJECT_ENTERPRISE_DOCUMENT_ASSISTANT.md).

## 1. The problem

A model can receive the correct passage and still answer incorrectly. It does not execute a deterministic database lookup. It predicts output tokens from instructions, the question and all supplied context. Relevant evidence can be obscured by irrelevant chunks, conflicting versions, weak formatting, a poor prompt, or a model that does not reliably use evidence in a long input.

A larger context window means the API accepts more tokens. It does **not** mean every token receives equal attention or that adding text improves the answer.

**MuleSoft analogy:** increasing a message-size limit does not improve routing. If a flow receives duplicate, stale and conflicting payloads without clear precedence, a larger payload makes the transformation harder.

## 2. Prerequisites

- A **token** is a small unit of text processed by the model.
- The **context window** is the maximum total tokens available for instructions, conversation, retrieved evidence and output.
- **Retrieval** selects candidate evidence from the corpus.
- **Generation** uses the prompt and evidence to produce the response.
- **Grounding** means the answer is supported by supplied evidence.
- **Context utilization** describes whether the model actually uses the right supplied evidence.

Keep retrieval and generation failures separate:

| Observation | Likely investigation |
|---|---|
| Required chunk never retrieved | extraction, chunking, embedding, query or filters |
| Required chunk retrieved but omitted from final prompt | deduplication, reranking or token budget |
| Required chunk is in prompt but answer is wrong | prompt structure, conflicts, placement, model behavior |
| Answer cites a chunk that does not support it | generation and citation validation |

## 3. A small grounded-answer contract

Do not send an unstructured wall of text. Give each excerpt an ID and a clear rule:

```python
def build_prompt(question: str, chunks: list[dict]) -> str:
    evidence = "\n\n".join(
        f"[SOURCE {c['id']}]\n{c['text']}" for c in chunks
    )
    return f"""You answer only from EVIDENCE.
If the evidence does not support an answer, return INSUFFICIENT_EVIDENCE.
For every factual statement, cite its source ID.

QUESTION:
{question}

EVIDENCE:
{evidence}

OUTPUT:
answer followed by source IDs
"""
```

**Expected input:**

```python
question = "What is the Tier B daily transfer limit?"
chunks = [
    {"id": "policy-v3:p4", "text": "Tier B daily transfer limit is ₹75,000."},
    {"id": "faq:p2", "text": "Transfers may require additional verification."},
]
```

**Expected output:**

```text
The Tier B daily transfer limit is ₹75,000. [policy-v3:p4]
```

A prompt contract improves clarity but cannot guarantee correctness. Validate structured output and citations in application code.

## 4. Why more context can hurt

More chunks can introduce:

1. **noise:** irrelevant text competes with the useful passage;
2. **conflict:** old and new policies disagree;
3. **position effects:** evidence in the middle of a long prompt may be used less reliably;
4. **instruction conflict:** retrieved text may contain text that looks like a command;
5. **cost and latency:** more input tokens increase both;
6. **truncation:** important content or output space may be cut off.

This does not mean “always use fewer chunks.” It means retrieve broadly enough for recall, then select and structure the smallest evidence set that completely supports the answer.

## 5. Context assembly in the project

```text
question
  -> retrieve 20 candidates
  -> authorization filter
  -> deduplicate
  -> rerank
  -> select evidence under token budget
  -> detect version conflicts
  -> build structured prompt
  -> generate
  -> validate citations and support
```

A teaching selector:

```python
def select_context(ranked_chunks, token_budget, estimate_tokens):
    selected = []
    used = 0

    for chunk in ranked_chunks:
        size = estimate_tokens(chunk["text"])
        if used + size > token_budget:
            continue
        selected.append(chunk)
        used += size

    return selected
```

Production selection must also preserve linked evidence needed for multi-document questions. A greedy top-rank selector may omit a second required fact.

## 6. Diagnose “correct evidence, wrong answer”

Use a fixed failing example and inspect these layers in order:

1. **Prompt inclusion:** prove the exact required text reached the provider.
2. **Permissions and redaction:** confirm preprocessing did not remove the fact.
3. **Conflicts:** search the prompt for another value or older version.
4. **Prompt contract:** make evidence and output rules explicit.
5. **Placement:** test the required passage at the beginning, middle and end.
6. **Context size:** remove irrelevant chunks without removing required evidence.
7. **Decoding:** use a stable factual setting; do not assume temperature fixes grounding.
8. **Model comparison:** only after holding the prompt and evidence constant.
9. **Citation support:** verify the cited source contains the claimed value.

Do not log confidential prompt contents in production. Record redacted IDs, versions, hashes, ranks and token counts.

## 7. Measurable lab

Create 30 questions with known evidence:

- 10 with one required chunk;
- 10 with two required chunks;
- 5 with a conflicting obsolete chunk;
- 5 unanswerable questions.

For every question run:

- relevant evidence only;
- relevant evidence plus 10 irrelevant chunks;
- relevant evidence placed first, middle and last;
- relevant plus obsolete evidence with and without explicit version metadata.

Measure:

- supported-answer accuracy;
- correct refusal rate;
- citation-support rate;
- input tokens;
- p95 latency;
- performance by evidence position.

**Expected result:** a comparison table showing whether noise, position or conflict changes behavior. A useful acceptance gate is 100% correct refusal on the five unanswerable cases and no unsupported citations.

## 8. Alternatives and trade-offs

- **Send every retrieved chunk:** highest apparent coverage; more noise, cost and conflict.
- **Fixed small final-k:** predictable; may omit evidence for multi-hop questions.
- **Reranking:** improves ordering; adds latency and another model dependency.
- **Extractive answer:** quotes evidence directly; less flexible but easier to verify.
- **Structured fact extraction before generation:** strong for policies and tables; requires schemas.
- **Long-context model:** useful for documents that cannot be separated; still needs evaluation.
- **Context compression or summaries:** saves tokens; can remove qualifications or provenance.

## 9. Common mistakes

- Concluding the retriever failed because the final answer was wrong.
- Measuring only end-to-end answer quality.
- Treating a large context window as reliable memory.
- Sending duplicate or obsolete chunks.
- Placing document text in the instruction section.
- Trusting a citation because its ID exists without checking textual support.
- Changing model, prompt and retriever simultaneously.

## 10. Interview answers

**Beginner version:**  
“A correct retrieved chunk does not guarantee a correct answer because retrieval and generation are separate stages. I first verify the exact chunk entered the prompt, then check conflicting or irrelevant context, prompt structure, placement and citation support. I test these on a fixed dataset rather than simply increasing top-k or context length.”

**Senior version:**  
“I instrument the full context-assembly path: source version, retrieval rank, rerank score, authorization decision, prompt inclusion, token position and citation support. I reproduce the failure with a frozen prompt, run position/noise/conflict ablations, and change one component at a time. The release gate includes answer correctness, refusal, evidence recall, citation entailment, latency and token cost.”

## 11. Further learning

These are teaching references, not interview-evidence sources:

- [Lost in the Middle](https://arxiv.org/abs/2307.03172) — primary research demonstrating that long-context performance can depend on where relevant information appears.
- [Anthropic: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — practical discussion of selecting and structuring context; useful for agent state and tool results.
- [Pinecone: Retrieval-Augmented Generation](https://www.pinecone.io/learn/retrieval-augmented-generation/) — approachable retrieval-to-generation walkthrough; useful before implementing vector search.
- [RAG ranking, freshness and resilience](02-rag-ranking-freshness-resilience.md) — repository lesson for candidate retrieval, reranking, versioning and outages.

Test all advice on your own corpus and model. No single prompt pattern is universally reliable.
