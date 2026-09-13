# Project360 — Project Continuity Agent

An AI agent that reads Outlook emails, OneDrive documents, and Jira tickets, and flags when they fall out of sync. It identifies missing actions, conflicting records, and decisions that may affect other teams. Once a person approves a finding, Project360 either makes the change automatically or routes it for manual editing.

## The problem

This was built for long-running, multi-team projects — tested against a simulated SAP transformation using entirely fictional project data.

Keeping decisions, commitments, and project records aligned is necessary, but much of the work is repetitive manual checking. Project managers and teams spend time cross-checking emails, meeting notes, documents, and tickets — work Project360 can complete in minutes so people can focus on decisions and delivery.

## What it does

Each run, Project360 checks a project's current communications and official records against each other for three types of gaps:

- **Capture** — an action, decision, or commitment appears, but nothing tracks it
- **Align** — records disagree or do not match the available evidence
- **Alert** — a change or risk in one team's work that could affect another team, flagged as a heads-up so that team can plan around it

Findings are posted to a Telegram group as review cards. Once approved:

- **Jira findings** → the change is made automatically
- **Excel or Word findings** → the required edit is flagged for manual editing

## Human in the Loop

Human approval is deliberate. Testing showed that LLMs can still miss information or produce incorrect findings, even after prompt revisions.

Project360 therefore identifies and proposes; a person decides and approves before any action is taken.

## Results

- ~95–100% reliable on concrete findings such as stale tickets and uncaptured commitments
- ~60–75% reliable on subjective findings such as whether someone should have been involved
- Zero structurally broken findings since a fix applied partway through the build

See `eval_set.md` for the evaluation methodology and `debugging-log.md` for the full development history.

## Where to look next

| File | What's in it |
| --- | --- |
| `architecture.md` | System architecture and diagram |
| `eval_set.md` | Evaluation set and answer key |
| `debugging-log.md` | Bugs, fixes, and removed features |

## Built with

n8n, Google Gemini, Supabase/Postgres, Jira API, Telegram.

## Build context

This prototype is self-hosted on my own machine using n8n Community edition and personal accounts.

Outlook and OneDrive evidence currently enters as plain-text files rather than live Microsoft 365 connections. The downstream retrieval and reasoning workflow is source-independent, so live connectors can be added without rebuilding the core logic.

Excel updates follow the same structured approach as Jira and could be automated through the Microsoft 365 connector. Word remains a deliberate human step because editing prose in context is more open-ended than changing a structured field.

Telegram is a stand-in for the messaging layer; the approval workflow could be connected to Microsoft Teams without changing the underlying logic.

The prototype uses fictional data, but a real deployment would involve confidential client and firm content. Google's Gemini API only excludes that data from training and human review once the API key sits on a billed Google Cloud account or Vertex AI, not the free tier — a production version of this agent would run exclusively on that billed tier.
