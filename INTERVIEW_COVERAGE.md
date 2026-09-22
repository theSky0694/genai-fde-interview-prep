# Interview Evidence & Coverage Dashboard

Last updated: 22 September 2026

This dashboard answers: **Based on the interview evidence collected so far and our roadmap, where do we stand?**

## Measures

### 1. Syllabus Readiness
How much of the planned curriculum has reached interview-ready status.

### 2. Reported-Question Coverage
For ingested reported questions:
- **Green:** can answer without material help
- **Amber:** partial / needs hints / incomplete
- **Red:** cannot yet answer adequately
- **Untested:** not yet tested

### 3. Evidence-Based Topic Pressure
Tracks which topics recur across reported experiences. It uses report count, recency, company spread and target-role relevance. It is used to **reweight preparation**, not to make unsupported claims about market-wide frequency.

### 4. Practical Evidence
Whether a topic is backed by code, tests, architecture work or a project that can be defended.

## Baseline

Reported interview experiences ingested: **0**

There is not enough interview-experience evidence yet to reweight the roadmap. Current preparation remains at **Stage 1A — Python Foundations**.

## Current emphasis

| Topic | Roadmap status | Report evidence | Practical evidence | Current priority |
|---|---|---|---|---|
| Python foundations | Learning | None yet | Not yet | P1 |
| Production Python / FastAPI | Not started | None yet | Not yet | P1 |
| LLM fundamentals | Not started | None yet | Not yet | P1 |
| RAG / retrieval | Not started | None yet | Not yet | P1 |
| RAG evaluation | Not started | None yet | Not yet | P1 |
| Agents / tool calling | Not started | None yet | Not yet | P1 |
| MCP | Not started | None yet | Not yet | P2 |
| GenAI system design | Not started | None yet | Not yet | P1 |
| FDE scenarios | Not started | None yet | Not yet | P1 |
| Behavioural / leadership | Not started | None yet | Not yet | P1 |

Priorities above are initial roadmap priorities, not LinkedIn-derived conclusions.

## Reweighting rule

After each batch of reported interview questions:

1. normalize and deduplicate questions;
2. update report counts and company spread;
3. test/map current readiness;
4. identify syllabus gaps;
5. raise/lower topic emphasis where evidence warrants;
6. update INTERVIEW_TRACKER.md and, when material, ROADMAP.md;
7. record what changed and why.

This prevents the roadmap from becoming static while also preventing a single anecdotal post from hijacking preparation.
