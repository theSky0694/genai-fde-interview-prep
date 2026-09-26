# LLM application fundamentals — interview answers

[Guide home](README.md) · Original explanations informed by the topics in the [source LLM collection](https://github.com/Nareshedagotti/AI-Engineer-Interview-QA/blob/main/LLM_Interview_Questions.md). **Roadmap:** Stage 2 and production stages. **Readiness:** Untested.

## 1. What is a context window, and what happens when it fills?

**Short answer.** The context window bounds the tokens the model can consider for a request, including instructions, history, retrieved evidence, tool results and output budget. More context costs memory/latency and can add distraction; fitting text does not guarantee the model uses it correctly.

**Detailed explanation.** Count all message roles and anticipated output, not just the user question. A tool-heavy agent can consume budget quickly with verbose JSON or logs. Rank and trim retrieved evidence, summarize or structure older conversation, and retain critical task state separately. If information is omitted, the model cannot cite it; if too much is included, relevant evidence can be buried. A larger window helps some tasks but is not a replacement for good retrieval and evaluation.

**Trade-offs.** More evidence can improve recall while increasing latency/cost and conflicting context. Aggressive compaction saves tokens but can erase conditions. Measure evidence coverage and answer quality at several budgets.

**Follow-ups.**
- *What should never be summarized away?* Authorization state, unresolved obligations, exact identifiers, tool outcomes and user corrections critical to the task.
- *How do you handle a large tool result?* Extract task-relevant fields with source references; retain full output in protected storage for recovery when justified.
- *Would you always choose the largest context model?* No; compare cost, latency and measured quality for the workload.

**Drill.** Run 20 questions with 2k, 8k and 32k context budgets. Report evidence coverage, support and latency.

## 2. When do you use prompting, RAG or fine-tuning?

**Short answer.** Start with prompting for behavior and output format; use RAG when the task needs current/private source knowledge and citations; consider fine-tuning for persistent behavior or domain patterns after evaluation shows prompting/RAG is insufficient. These approaches can be combined.

**Detailed explanation.** A bank policy updated weekly should usually be retrieved from the authoritative version, not memorized through training. A stable repetitive extraction format may improve with labeled examples or fine-tuning, but still needs validation. Fine-tuning does not provide automatic citations, current knowledge or access control. Before any training, define a gold dataset, baseline prompt/RAG performance, data governance and operational budget. Avoid choosing a technique because a question contains the phrase “reduce hallucination”; diagnose whether the model lacked evidence, ignored evidence or failed a task format.

**Trade-offs.** Prompting is fast to iterate but uses context and may be brittle. RAG has ingestion/retrieval complexity but supports updates and provenance. Fine-tuning needs curated data, training/evaluation and rollout control; it may lower per-request prompt burden or improve behavior, yet can overfit or forget general capabilities.

**Follow-ups.**
- *Can RAG fully stop hallucination?* No. The model can misread correct evidence or cite irrelevant text; evaluate claim-level support.
- *Would you fine-tune on confidential policies?* Only with an authorized data and retention design; retrieval is generally easier to update and revoke.
- *What if the answer format is wrong but evidence is correct?* Try schema-constrained output and validation before training.

**Drill.** Compare baseline prompting, few-shot prompting and RAG for a small versioned policy set, including a changed policy and no-answer question.

## 3. How do temperature and top-p affect an application?

**Short answer.** They alter sampling from model token probabilities, affecting variability. Lower values often make outputs more repeatable, but temperature zero does not guarantee full determinism or factual correctness.

**Detailed explanation.** Temperature reshapes relative token probabilities; top-p restricts selection to a cumulative-probability set. Exact behavior can differ by model/server implementation. For structured extraction or tool arguments, constrain output with a schema and validate in code rather than depending on a low temperature. For creative drafting, some variation can be useful. Other sources of variability include provider updates, parallelism, tool results and changing retrieval context.

**Trade-offs.** Low variability can improve reproducibility but may repeatedly produce the same wrong answer. Higher variability can generate ideas but complicates evaluation. Tune against task metrics, not aesthetics alone.

**Follow-ups.**
- *Would temperature zero make a bank action safe?* No. Authorization, state checks and approval remain in deterministic code.
- *Should you tune both temperature and top-p aggressively?* Change one factor at a time in an evaluation to understand its effect.
- *How do you reproduce a failure?* Record model/version, prompt, parameters, retrieval IDs, tool outputs and timing within privacy rules.

**Drill.** Run the same 20 extraction prompts multiple times under two settings; measure schema validity, variance and correctness.

## 4. How would you choose a model for an enterprise assistant?

**Short answer.** Define task quality, data residency, latency, throughput, context, tool/structured-output behavior, cost and operations. Benchmark representative tasks on candidate models, including failure cases, rather than selecting by parameter count or leaderboard alone.

**Detailed explanation.** In a locally hosted setting, compare Qwen/Gemma/MiniMax variants only if approved and available, with the same prompts, tools and retrieval. Record task success, unsupported claims, valid tool arguments, TTFT, tokens/sec, GPU memory and concurrent throughput. Quantization can reduce memory and improve deployment feasibility, but quality/latency effects depend on model, precision and hardware. Route easy tasks to cheaper models only when measured quality remains acceptable; keep fallback behavior and data boundary explicit.

**Trade-offs.** A larger model may improve difficult reasoning but costs more GPU memory/latency. A smaller model can work well when retrieval and tool contracts are strong. Managed APIs reduce hosting effort but may not meet data-residency requirements. The correct choice depends on workload and governance.

**Follow-ups.**
- *What if one model wins a public benchmark but fails your tool calls?* Prefer task-specific evaluation for the deployment.
- *How do you test quantization?* Compare the same labeled tasks and load profile at each precision, including rare edge cases and structured outputs.
- *How do you handle model upgrades?* Version prompts/evals, shadow test, canary, monitor regressions and keep rollback.

**Drill.** Make a 30-task benchmark with RAG, tool selection and structured extraction; report quality, p95 latency and GPU footprint for two model configurations.

## 5. What does “evaluation” mean for an LLM application?

**Short answer.** Define success for the user task, then measure components and end-to-end outcomes separately. Include labeled expected evidence, deterministic schema/action checks, human-reviewed samples and repeated runs where nondeterminism matters.

**Detailed explanation.** A RAG system needs parser/retrieval/grounding metrics; an agent needs tool choice, permissions, state transition and task completion. A single average score can hide a dangerous rare failure. Stratify by task type, tenant, language, long documents, no-answer cases and updated sources. Record input and system versions so a regression can be reproduced. LLM judges can help scale review, but compare them to human decisions on a sample and inspect disagreements.

**Trade-offs.** A small gold set is quick but can overfit and miss production variation. Large synthetic sets are scalable but may not represent real failures. Combine a stable regression set with sampled real-world feedback under privacy controls.

**Follow-ups.**
- *What metric would you put on a dashboard?* Task success plus safety violations and p95 latency/cost, broken down by failure class; the exact mix follows the use case.
- *How do you decide whether a new prompt is better?* Compare on the same cases and budget, including uncertainty and regression failures.
- *What if the answer sounds good but source support is missing?* Fail grounding, even if a judge likes its fluency.

**Drill.** Define a 20-case eval manifest with question, expected evidence, rubric and forbidden actions; run it after every change.
