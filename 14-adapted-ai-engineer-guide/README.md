# Adapted AI Engineer interview guide

This is an **original study guide** informed by the topic coverage of [Nareshedagotti/AI-Engineer-Interview-QA](https://github.com/Nareshedagotti/AI-Engineer-Interview-QA), inspected 26 September 2026. That repository displayed no license. Its text, examples and answer prose have **not** been copied into this repository. Refer to the source for its full question bank.

Your existing interview batches and answers remain in [../12-reported-interview-experiences/](../12-reported-interview-experiences/). This collection adds a separate, personalized explanation layer. The source is a question bank, **not** an independently reported interview experience; it does not increase the evidence counters in `INTERVIEW_COVERAGE.md`.

## How to study this without overload

Read one cluster at a time. First give the short answer aloud, then explain the mechanism, one failure case, a trade-off and a measurement. Try the exercise only for topics that are weak. The main target is **GenAI Engineer / Agentic AI Engineer / FDE by March 2027**, drawing on nine years of enterprise integration and a locally hosted bank assistant using Python, FastAPI, LangGraph, tools and RAG.

| Guide | Priority now | Purpose |
|---|---|---|
| [Production Python and FastAPI](python-fastapi.md) | Now | Move from syntax to safe concurrent services and integrations |
| [RAG engineering](rag-engineering.md) | Stage 3–4, preview now | Explain retrieval decisions and prove grounded answers |
| [Agent workflows](agent-workflows.md) | Stage 5, preview now | Defend state, tools, checkpoints and reliability |
| [LLM application fundamentals](llm-applications.md) | Stage 2 | Model selection, context, prompting, cost and evaluation |
| [Transformer essentials](transformer-essentials.md) | One short session | Enough internals for application-focused interviews |

## Answer pattern for a senior interview

1. State the requirement and assumptions.
2. Describe the simple baseline.
3. Explain the engineering choice and an alternative.
4. Name a failure mode and mitigation.
5. State how you would measure it.
6. Tie it to an actual system you built **only where the detail is true and shareable**. Do not invent bank metrics, architecture or incidents.

A source topic is not proof it was asked in an interview. The examples below are illustrative exercises, not claims about the bank's internal implementation. Current readiness is **Untested** until you answer a question or implement its drill.

## Source topic map

| Original collection | Our focus | Deliberately deferred |
|---|---|---|
| [RAG Q&A](https://github.com/Nareshedagotti/AI-Engineer-Interview-QA/blob/main/RAG_QA.md) | Ingestion, chunking, hybrid retrieval, reranking, evaluation, security | Exhaustive algorithm catalog |
| [Agentic AI Q&A](https://github.com/Nareshedagotti/AI-Engineer-Interview-QA/blob/main/Agentic_AI_Interview_Questions.md) | Tool loop, workflow vs agent, LangGraph state, failure, human approval | Framework collecting and speculative self-improvement |
| [LLM Q&A](https://github.com/Nareshedagotti/AI-Engineer-Interview-QA/blob/main/LLM_Interview_Questions.md) | Context, prompting, model choice, inference trade-offs | Training mathematics unless a model-development role asks |
| [FastAPI Q&A](https://github.com/Nareshedagotti/AI-Engineer-Interview-QA/blob/main/FASTAPI_QA.md) | Validation, async, pooling, failures, streaming | Framework trivia |
| [Transformers Q&A](https://github.com/Nareshedagotti/AI-Engineer-Interview-QA/blob/main/TRANSFORMERS_QA.md) | Attention intuition and practical consequences | Derivations, training internals, MoE sharding |
