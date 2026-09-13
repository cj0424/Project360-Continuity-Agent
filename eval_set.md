# Project360 — Evaluation Set

This document defines how Project360 was evaluated and records the results of those tests.

The evaluation uses a fixed set of fictional project evidence designed to test **Capture, Align, and Alert** findings across Jira, documents, trackers, and notifications. Testing covers retrieval, reasoning, regression behaviour, and the complete human-in-the-loop execution flow.

For the development and debugging history behind these results, see `debugging-log.md`.

---

## Executive Summary

**Final verdict: PASS**

Project360 completed the full workflow from evidence ingestion through historical retrieval, reasoning, finding generation, human approval, and execution/manual routing.

The strongest results were on **concrete field-level findings**, where the agent consistently identified direct contradictions or missing records. More interpretive findings, particularly cross-document relationships, showed greater run-to-run variation.

The final locked run processed **14 findings with zero execution errors**.

---

## 1. Baseline Evaluation

The same fixed three-week evidence set was run four times before the RAG layer and later prompt refinements.

| Run | Total | Align | Capture | Alert |
| --- | ----: | ----: | ------: | ----: |
| 1   |     9 |     5 |       3 |     1 |
| 2   |    11 |     6 |       3 |     2 |
| 3   |    10 |     7 |       2 |     1 |
| 4   |    11 |     7 |       3 |     1 |

The same evidence produced different finding counts between runs. Direct field-to-field contradictions were consistently detected; softer interpretive findings varied.

This established **subjective reasoning variance** as an important evaluation consideration.

---

## 2. Retrieval Validation

The historical retrieval layer was tested through regression, persistence, and answer-key checks.

### Regression checks

**7/7 passed**

Previously fixed retrieval and pipeline defects remained silent after later changes.

### Persistence checks

**3/3 passed**

Historical evidence remained available and usable after ingestion and subsequent processing.

### Retrieval approach

Project360 currently retrieves historical context using:

**Gemini query expansion → PostgreSQL full-text keyword search → relevant historical chunks**

Embeddings are retained in the Supabase evidence store from the initial RAG backfill, but they are **not used by the current Project360 retrieval workflow**.

The ongoing evidence process still performs content hashing, change detection, chunking, and storage. New evidence does not require an embedding because the active retrieval path is keyword-based.

Testing showed that keyword search performed better than embedding retrieval on the structured, tracker-heavy evidence set.

### Known retrieval boundary

Keyword retrieval can still miss evidence when the vocabulary used in a record differs significantly from the search terms generated from the current evidence.

This remains a bounded retrieval-scope limitation.

---

## 3. Reasoning & Precision Tests

The reasoning layer was tested against several recurring failure modes.

### Historical context

The prompt was refined to explicitly require historical evidence to be considered where relevant.

### Topic vs. commitment

The agent was instructed not to treat a general topic or discussion as proof that a specific tracked commitment exists.

The refined rule held during retesting.

### Routine administrative noise

Routine emails such as invoices, access renewals, and similar administrative messages were tested to ensure they were not incorrectly promoted to findings.

The refined filtering successfully suppressed these cases.

### Record-vs-record checking

The agent was tested on contradictions between project records, rather than only narrative evidence.

### Finding structure

A structural checklist and examples were added to the prompt.

From version 10 onward:

**100% structural completeness**

No broken findings with missing source fields, missing names, or incomplete required structure were observed in the validated runs.

---

## 4. Final End-to-End Run

The final locked configuration used:

* Prompt version **v12**
* Original search-term extraction approach
* Current keyword-based retrieval
* Human approval before execution

### Final findings

**14 total**

| Finding type    | Count |
| --------------- | ----: |
| Jira            |    10 |
| Excel / tracker |     3 |
| Notification    |     1 |

### Execution results

| Action                       |               Result |
| ---------------------------- | -------------------: |
| Jira status updates          |         7 successful |
| Jira ticket creation         |         3 successful |
| Excel/tracker manual routing |         3 successful |
| Alert delivery               |         1 successful |
| **Total**                    | **14/14 successful** |

**14/14 findings completed with zero execution errors.**

---

## 5. Reliability Summary

| Test / Measure                         |                   Result |
| -------------------------------------- | -----------------------: |
| Structural completeness                |           100% from v10+ |
| Regression suppression                 |                      7/7 |
| RAG persistence                        |                      3/3 |
| Structural sanity checks               |                      6/6 |
| Week 4 answer key                      |                    3.5/4 |
| Week 5 answer key                      | 6/8 clean hits + 3 bonus |
| End-to-end execution                   |                    14/14 |
| Concrete field-level findings          |                 ~95–100% |
| Interpretive / cross-document findings |                  ~60–75% |
| Overall blended estimate               |                     ~85% |

These figures are based on the fixed fictional evaluation set and should be treated as evaluation results for this project rather than general production accuracy claims.

---

## 6. Known Limitations

### 1. Subjective run-to-run variance

The same evidence can produce different results when findings depend on interpretation rather than direct field-level contradictions.

### 2. Keyword retrieval ceiling

Keyword search performs well on the current structured evidence but can miss semantically related evidence expressed with different terminology.

### 3. Bounded retrieval scope

Historical context is limited to the retrieved evidence passed into the reasoning step.

### 4. Manual Jira assignee handling (temporary)

Assignee mapping is currently manual, since the test environment uses fictional names that don't map to real Jira accounts. This is a temporary limitation of the test setup — with real Jira user accounts, ticket creation and updates would be fully automated, including the assignee.

### 5. Alert recipient onboarding

Telegram bots cannot initiate conversations with users who have not first interacted with the bot. The current test flow therefore uses a known recipient; production onboarding would require a DM-first or fallback approach.

### 6. Generated wording

Findings prioritise factual correctness and structured evidence over perfectly natural wording.

---

## 7. Final Verdict

**PASS**

Project360 demonstrated a complete, functioning human-in-the-loop pipeline with strong performance on concrete project-record inconsistencies and successful end-to-end execution.

The main remaining limitations are interpretive variance, bounded keyword retrieval, conservative Jira assignment, and operational onboarding constraints.
