# Brief Definition

Reference for writing brief documents. A brief is a short orientation document. It frames a piece of work -- the objective, the sources, the constraints, the approach -- so that anyone picking it up (human or agent) can start without a preamble conversation. It precedes commitment: you write a brief before deciding whether the work needs a design note, a plan, or neither.

---

## 1. Purpose

A brief exists to:

1. **Orient** -- tell the reader what the work is about, why it matters, and what the boundaries are.
2. **Scope** -- define what's in and what's out before effort is spent on detailed design or implementation.
3. **Source** -- identify where the relevant information lives (code, documents, external systems, people).
4. **Propose** -- outline an approach or process, without committing to the level of detail a design note or plan requires.

A brief is NOT:

- A design note. A design note is a commitment document with settled decisions, rationale, interface shape, and acceptance criteria for a scoped piece of work. A brief is pre-commitment. See [Design Note Definition](design-note_definition.md).
- An implementation plan. A plan breaks work into sequenced tasks with a status table and success criteria. A brief frames the work that a plan would sequence. See [Implementation Plan Definition](impl-plan_definition.md).
- A design study. A design study decomposes a problem into dimensions, evaluates alternatives, and converges on a recommendation. A brief may identify that a design study is needed, but it does not perform the analysis. See [Design Study Definition](design-study_definition.md).
- A living document. A brief captures the understanding at the time of writing. If the work evolves significantly, write a design note or plan -- do not retrofit the brief into something it was not designed to be.

---

## 2. When to Write One

Write a brief when:

- You need to frame a piece of work before deciding whether it warrants a design note, a design study, or direct implementation.
- You're orienting an agent or collaborator on a task that has multiple information sources and a non-obvious scope.
- The work is too small for a design note but too structured for a conversation.

Do not write a brief when:

- The scope is already clear and you're ready to commit. Write a design note.
- The work is a known bug with a known fix. Write a bug-fix plan (or just fix it).
- You're choosing between alternatives. Write a design study.

---

## 3. Structure

A brief has a fixed section order. All sections are required unless marked otherwise. Briefs are short by design -- up to five pages. If a section grows beyond a few paragraphs, the document likely wants to be a design note or a design study.

### 3.1. Metadata (required)

```markdown
---
title: <Topic>
status: draft | accepted
date: YYYY-MM-DD
author: <git user name>
---
```

| Field | Purpose |
|---|---|
| `title` | Short, descriptive name. Matches the `#` heading. |
| `date` | Creation date in `YYYY-MM-DD` format. Must be the actual calendar date when the document is created. Must match the date prefix in the filename. Immutable after creation. |
| `author` | Who wrote this brief. Use `git config user.name`. |
| `status` | `draft` while the brief is authored and revised; `accepted` once it is settled and shared. |

A brief's content is refined freely in `draft` and freezes at `accepted` (see §4.2). Briefs written before this change carry no `status` field and are treated as accepted.

**File naming:** `YYYY-MM-DD_<topic>_brief.md`. E.g., `2026-04-21_api-gateway-architecture_brief.md`.

### 3.2. Objective (required)

One to three sentences answering: what is this work about, and why does it matter now? Lead with the outcome, not the activity.

```markdown
## 1. Objective

Document the current and planned architecture of the API gateway layer so that
the team has a shared reference for design decisions through Q3 2026.
```

### 3.3. Scope (required)

What's in and what's out. Use a table or two short lists. The purpose is to prevent scope creep before work begins.

```markdown
## 2. Scope

| In scope | Out of scope |
|---|---|
| Current state of the secondary stack | Performance benchmarking |
| Planned state for end of Q2 and Q3 2026 | Implementation of planned changes |
| Component-level descriptions | Line-by-line code documentation |
```

### 3.4. Sources (when applicable)

Where the relevant information lives. Name specific files, branches, URLs, documents, or people. This section exists so that the person (or agent) doing the work knows where to look without asking.

```markdown
## 3. Sources

1. The codebase (branch: `user/api-gateway-docs`)
2. Google Docs: <list of document titles or URLs>
3. Confluence pages: <list of page titles or URLs>
```

### 3.5. Approach (when applicable)

How the work will proceed. Not a detailed plan -- just the shape of the process. A few numbered steps or a short paragraph.

```markdown
## 4. Approach

1. Gather facts from the sources into a staging directory as markdown.
2. Draft the architecture document following the doc-definition for the target format.
3. Review with the team and iterate.
```

### 3.6. Constraints (when applicable)

Non-obvious limitations, dependencies, or decisions that affect the work. Tooling restrictions, access limitations, format requirements.

### 3.7. Open Questions (when applicable)

Questions that must be answered before or during the work. Each question should identify who or what can answer it.

Open questions are for what cannot yet be verified, not for what has not yet been decided. A question may remain open only when the aspect cannot be verified at the time of writing (e.g., hardware not available, an external system not yet observable). Anything resolvable by design work must be resolved before the brief is written and folded into its regular structure — open questions are not a backdoor for postponing design work.

---

## 4. Principles

### 4.1. Short by Design

A brief is up to five pages. It frames; it does not analyze, design, or specify. If the document grows beyond that, it is evolving into a different document type. Promote it explicitly -- create the design note or design study and reference the brief as the origin.

### 4.2. Frozen at Acceptance

A brief captures the understanding at the time of writing. Its content is iterated and refined while in `draft`; once `accepted`, it is frozen and not updated as the work progresses. Progress tracking belongs in a plan; design evolution belongs in a design note or design study. A brief that is continuously updated after acceptance has become a living document without the structure to support it — if the framing changes materially, write a new brief or promote to the successor document; do not retrofit.

### 4.3. Pre-Commitment

A brief does not commit to a solution, an architecture, or a timeline. It frames the problem and the approach. Commitments belong in design notes (what to build) and plans (how to build it).

---

## 5. Anti-Patterns

| Anti-pattern | Problem | Fix |
|---|---|---|
| **Brief as living document** | Brief is updated throughout the project, accumulating decisions and status. | Promote to a design note or plan. Reference the brief as the origin. |
| **Brief with acceptance criteria** | Brief defines what "done" looks like in detail. | That's a design note. Write a design note. |
| **Brief with task breakdown** | Brief lists sequenced implementation steps with status tracking. | That's a plan. Write a plan. |
| **Brief with alternatives analysis** | Brief evaluates multiple approaches in depth. | That's a design study. Write a design study. |
| **Multi-page brief** | Brief grows beyond five pages because the scope is too large. | Split into a brief (framing) and a design note or design study (detail). |

---

## 6. Minimal Example

```markdown
---
title: API Gateway Architecture
date: 2026-04-21
author: Jane Smith
---

# API Gateway Architecture

## 1. Objective

Document the current and planned architecture of the API gateway layer so that
the team has a shared reference for design decisions through Q3 2026.

## 2. Scope

| In scope | Out of scope |
|---|---|
| Current state of the secondary stack | Performance benchmarking |
| Planned state for end of Q2 and Q3 2026 | Implementation of planned changes |
| Component-level descriptions | Line-by-line code documentation |

## 3. Sources

1. The codebase (branch: `user/api-gateway-docs`)
2. Google Docs: architecture overview, gateway design doc
3. Confluence pages: API gateway status, Q2 planning

## 4. Approach

1. Export Google Docs and Confluence pages as markdown into a staging directory.
2. Read the codebase alongside the exported sources; extract facts into structured markdown.
3. Draft the architecture document per the doc-definition.
4. Review with the team and iterate.

## 5. Constraints

- Google Docs OAuth is not yet operational. Docs must be manually exported.
- The agent can read the codebase directly but cannot access Google Docs or Confluence natively.

## 6. Open Questions

- What doc-definition should the architecture document follow? (Answer: to be created during this work.)
- Which Confluence pages are authoritative vs. stale? (Answer: ask the team lead.)
```

---

## Governing Standards

This definition conforms to the [Document Structure Standards](../standards/document-structure_standards.md). See section 2.1 for the file naming convention and date rules.
