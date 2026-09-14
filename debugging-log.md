# Project360 — Debugging Log

The major bugs, experiments, fixes, and design decisions made while building Project360.

This is the development record. It focuses on what went wrong, how it was diagnosed, what changed, and what was ultimately kept, reverted, or removed.

Evaluation results are recorded separately in `eval_set.md`.

## Retrieval & Evidence Pipeline

**1. Wrong-record matching**

Problem: Change detection matched files by array position rather than filename.

Impact: A changed record could silently be compared against the wrong previous version.

Fix: Records are matched using a stable source reference / filename.

**2. `isNew` filtering hid evidence from analysis**

Problem: The `isNew` filter originally gated the analysis itself. Unchanged evidence — including Jira and structured project records — could therefore disappear from the comparison.

Initial fix: Jira was explicitly exempted.

Later discovery: Trackers, RAID logs, and status reports could still be filtered out when there was no new narrative evidence.

Final fix: Evidence was classified by content rather than a hardcoded filename rule:

* Narrative evidence — meeting minutes and emails — uses `isNew` to determine when it needs narrative reprocessing.
* Structured records — Jira, trackers, RAID logs, and status reports — remain available for comparison regardless of whether they changed.

This was verified against a 29-file OneDrive pull and 30 real emails.

**3. Historical chunks were truncated**

Problem: Retrieved chunks were limited to 400 characters, cutting off important rows in structured records.

Diagnosis: Retrieved content was measured directly and shown to end before the relevant data.

Fix: Increased the maximum chunk length to 1,500 characters.

This produced one of the largest retrieval improvements during testing.

**4. Relevant chunks were crowded out**

Problem: A low per-document retrieval cap meant important information in later chunks could be excluded.

Fix: Increased the per-source allowance and raised the total historical chunk limit from 8 to 10.

**5. Embeddings vs. keyword retrieval**

Problem: It was unclear whether embedding-based retrieval would outperform keyword search.

Experiment: Both approaches were tested against the same evidence after embedding-related rate limits were resolved.

Result: Embeddings often retrieved semantic but unhelpful content such as signatures, headers, or generic text. PostgreSQL full-text keyword search retrieved the relevant rows more reliably for the structured, tracker-heavy evidence.

Decision: Keep keyword search as the active retrieval method.

Embeddings remain available in the Supabase evidence store from the initial RAG backfill, but Project360 does not generate or use embeddings for its ongoing retrieval workflow.

New and changed evidence still goes through content hashing, change detection, chunking, and storage. Its embedding field can remain NULL because the current retrieval function does not depend on embeddings.

**6. Retrieval vocabulary gap**

Problem: Some relevant historical records used terminology such as "sign-off" or "confirm" that did not rank highly against the dominant project topics.

Experiment: Tried extracting search terms separately for each document instead of using the original global approach.

Result: No measurable improvement. It also introduced names as noisy search terms.

Decision: Reverted the experiment and documented the retrieval gap as a known limitation.

## Reasoning & Prompt

**7. Historical context was not used correctly**

Problem: The model sometimes ignored or misused retrieved historical context.

Fix: Added explicit instructions defining how prior context should influence the reasoning checks, including dedicated historical-context conditions.

**8. Findings were structurally incomplete**

Problem: Some findings contained empty source fields, missing names, or incomplete required information.

Fix: Added a structural quality checklist and concrete examples for each finding type.

Result: No structural defects were observed from prompt version 10 onward.

**9. Topic mention was treated as a tracked commitment**

Problem: The model sometimes treated a general discussion of a topic as proof that a specific commitment or action was already recorded.

Fix: Added an explicit rule distinguishing:

* a topic being discussed, from
* a specific action, decision, commitment, or tracked record.

The rule held during retesting.

**10. Routine administrative emails became findings**

Problem: Routine emails such as invoices, access renewals, and similar administrative messages were occasionally promoted to findings.

Fix: Added an unresolved-commitment test so routine administration is not treated as a project finding unless there is an actual unresolved action, decision, or commitment.

## Execution

**11. `undefined` in Jira success messages**

Problem: Success messages displayed `undefined` when no Jira status transition was required.

Fix: Added an explicit execution branch for findings that do not require a status change.

**12. New Jira descriptions were blank**

Problem: Some newly created Jira tickets received blank descriptions because the expected description field was not always populated.

Fix: Added a fallback using a field that is always present.

**13. Created Jira tickets lost their ticket key**

Problem: Newly created tickets did not retain their generated Jira key in the final workflow output.

Fix: After creation, the workflow writes the real Jira key back into the finding record.

**14. Manual Jira assignee handling (temporary)**

Decision: Automatic assignee mapping was not implemented in this build.

Reason: The test environment uses fictional names that don't map to real Jira user accounts, so there's no reliable way to auto-assign correctly. This is a limitation of the test setup, not a permanent design boundary.

## Alerts

**15. Telegram recipient constraint**

Problem: Telegram bots cannot initiate a private conversation with a user who has never interacted with the bot.

Current test approach: Alerts are sent to a known test recipient.

Production consideration: A DM-first onboarding flow or group fallback would be required for broader deployment.

## Feature Built and Removed

**16. Reminder / re-escalation system**

Problem: Unresolved findings appeared to need their own reminder and re-escalation mechanism.

Implementation: A complete reminder-tracking and nudge system was built and tested.

Result: 4 of 6 reminder items duplicated issues that the main pipeline was already detecting.

Decision: Removed the reminder system.

The main pipeline remains responsible for re-detecting unresolved field-level issues, while manual-record findings continue through the existing human workflow.

## Development Lessons

**1. Measure retrieval, don't assume it**

Retrieval problems were often invisible until the actual chunks, lengths, and ranking behaviour were inspected.

**2. Test alternatives against the real evidence**

Embedding retrieval appeared promising in theory but performed worse than keyword search on this particular evidence set.

**3. Keep complexity tied to demonstrated value**

The reminder system worked technically but duplicated capability already provided by the core pipeline. Removing it simplified the system without reducing the required outcome.

**4. Separate evidence classification from filenames**

The later `isNew` issue showed that workflow behaviour should depend on what the evidence is, not on a specific connector or hardcoded filename.

## Final Development Position

The resulting system keeps the components that demonstrated value:

* content hashing and change detection
* structured evidence storage
* historical chunk retrieval
* Gemini query expansion
* PostgreSQL keyword retrieval
* Gemini reasoning
* human approval
* Jira execution
* manual-record routing
* notifications

Embedding data from the original RAG backfill remains available in Supabase, leaving the option to revisit vector retrieval later without making it a dependency of the current Project360 workflow.
