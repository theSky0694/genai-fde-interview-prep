# Repository working rules

This repository supports Aakash's transition from senior integration engineering to GenAI Engineer / FDE roles. Keep additions concise, source-linked, and useful for implementation or interview practice.

## Weekly research digest

- Add a dated file under `13-genai-engineering-research/` for worthwhile new papers, research reports, and firsthand engineering analyses on RAG, agent workflows, tool integration, LangGraph, AutoGen, and coding agents.
- Prefer primary papers and firsthand engineering accounts. Verify the publication or revision date, link the original source, and label preprints and author-reported benchmark results as such.
- For each item include the finding, why it matters for an integration developer building enterprise GenAI systems, and one concrete experiment or implementation takeaway.
- Aim for 4–6 strong items; publish fewer when the evidence is thin. Do not add filler, unsupported performance claims, or items already covered in an earlier digest.
- Use research to propose a measurable project or interview drill. Change the roadmap only when evidence or prerequisite structure warrants it.

## Weekly reported interview questions

- Search public first-person interview experiences on LinkedIn, Medium, company blogs, forums, and other accessible sites for GenAI Engineer, Agentic AI Engineer, Python/GenAI Engineer, and FDE roles, with India relevance where possible.
- Record the source URL, author or site, publication date, company/role if reported, and whether wording is exact, paraphrased, or inferred from an interview-preparation post. Never present a generic prep post as a firsthand interview report.
- Do not invent company attribution, interviewer wording, or question frequency. Avoid paywalled or inaccessible text that cannot be verified. Attribute short paraphrases and link the original rather than copying long passages.
- Deduplicate against `12-reported-interview-experiences/QUESTION_INDEX.md` and prior batches. For each new question, add what it tests, a compact answer outline, a practical exercise or follow-up, and its roadmap stage.
- Update counts and priorities only for independent reports, separating reports from preparation posts. Do not imply a representative hiring-market sample.
- Preserve `Untested` readiness until Aakash has actually answered or implemented a task. Keep the weekly selection small enough to study alongside a full-time job.

## Git hygiene

- Read the existing roadmap, question index, and relevant batch before editing.
- Do not overwrite user notes or change readiness status without evidence.
- Use dated, descriptive commits and keep source links intact.

## Beginner-first concept and project pedagogy (mandatory)

The learner is an experienced MuleSoft/API integration developer, **not yet fluent in GenAI**. Weekly content must teach, not merely summarize or test. Never assume knowledge of embeddings, retrieval, tokenization, transformers, RAG, agent state, LangGraph, evaluation, or MCP without introducing prerequisites or linking to an accessible lesson.

For each chosen interview theme, organize learning as: (1) what problem it solves and why, (2) first-principles concept explanation with simple example and familiar API/integration analogy where apt, (3) exact placement in a progressive Python/FastAPI document-RAG/agent project and data flow, (4) small annotated implementation with expected input/output, (5) design choices and trade-offs, (6) beginner then senior interview answers, (7) follow-up questions **and explained answers**, (8) mistakes/debugging, (9) an achievable practice task and validation criteria. Explain terminology before using it. Show the journey from concept to code to project to interview, not just interview Q&A. Prefer fewer thoroughly explained questions to a long shallow list. Cross-link reusable concept lessons, project walkthroughs, and weekly question batches. Preserve all existing notes; do not falsely mark exercises complete.

## Mandatory beginner-first teaching standard (26 October 2026)

- Aakash is an experienced integration/API developer but a beginner in GenAI concepts. Do not assume familiarity with embeddings, tokens, transformers, retrieval metrics, LangGraph or MCP.
- Every new interview batch must provide (or cross-link to) a plain-language prerequisite explanation, a worked example, a step in the progressive [Enterprise Document Intelligence Assistant](PROJECT_ENTERPRISE_DOCUMENT_ASSISTANT.md), executable/annotated code where applicable, expected output, a failure case, trade-offs, beginner answer, senior answer and answered follow-ups. Prioritize depth over question count.
- Maintain [concept foundations](00-genai-concept-foundations.md) and progressively improve **existing** batches, not just future ones. Preserve original reported question wording and evidence attribution.
- Teaching sources must not be restricted to official docs: use accessible, relevant **Medium, LinkedIn, independent engineering blogs, practical tutorials**, plus official documentation for API correctness. Include the URL and why each source helps. Do not mislabel teaching posts as firsthand interview reports or blindly trust blog claims.
- Organize learning concept-first -> project implementation -> debugging/evaluation -> interview defence. Keep readiness 'Untested' until demonstrated.
