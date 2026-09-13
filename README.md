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

Project360 does **not act on its own**.

Testing showed that an LLM can still miss information or produce incorrect findings, even after improving the prompts. So Project360 follows a simple rule:

**The agent identifies and proposes. A person reviews and approves.**

Nothing is updated, actioned, or escalated until a human approves the finding.

## Results

Project360 performed strongest when checking **clear, concrete evidence** and was less reliable when a finding required **subjective judgement**.

* **~95–100% reliable** when identifying clear issues, such as **outdated Jira records or commitments missing from project records**
* **~60–75% reliable** when making judgement-based assessments
* **100% structurally complete** since a fix was introduced partway through the build

For the full evaluation methodology, see `eval_set.md`.
For the development, debugging, and iteration history, see `debugging-log.md`.

## Build Context

Project360 is currently a **self-hosted prototype** running on my own machine using n8n Community Edition and personal accounts.

To keep the prototype simple, some parts are represented differently from how they would work in production:

* **Outlook & OneDrive:** Evidence currently enters as plain-text files rather than live Microsoft 365 connections. The core retrieval and reasoning logic is source-independent, so live connectors could be added later without rebuilding the workflow.
* **Excel:** Excel updates follow the same structured approach as Jira and could be automated using an Excel node.
* **Microsoft Word:** Word updates remain a deliberate human step because editing written content in context is more open-ended than changing a structured field.
* **Telegram:** Telegram currently acts as the approval and notification layer. The same workflow could be connected to Microsoft Teams without changing the underlying logic.

The prototype uses **fictional firm and client data**. A real deployment would involve confidential client and firm content.
For confidential or production use, Google provides **paid-service data protections** for Gemini. When the Gemini API is used through a project with an active Google Cloud billing account, or through Vertex AI, prompts and responses are **not used to improve or train Google's models** under the applicable paid-service terms.
