# Design Study Definition

Definition for writing design study documents. A design study is a structured analysis of a problem space. It dissects a problem into sub-problems, explores alternative solutions for each, evaluates how choices compose, and converges toward a recommendation. Its value is in the reasoning -- the decomposition, the trade-offs, the framework for future decisions -- not just the conclusion.

---

## 1. Purpose

A design study exists to:

1. **Decompose** -- break a problem into independent dimensions or sub-problems that can be analyzed separately.
2. **Explore** -- identify and describe alternative solutions for each sub-problem, with concrete examples.
3. **Evaluate** -- compare alternatives against explicit criteria, including how choices in one dimension interact with choices in others.
4. **Recommend** -- converge toward a recommendation with traceable rationale, or document why a decision cannot yet be made (open questions).
5. **Preserve reasoning** -- capture the analytical framework so future maintainers can revisit decisions when circumstances change, without re-deriving the analysis from scratch.

A design study is NOT:

- A spec. A spec is a commitment document -- it records what will be built and what done looks like. A design study precedes commitment: it works through the problem space to inform a decision. See [Specification Document Reference](spec-doc_definition.md).
- An implementation plan. A plan covers how to build a decided-upon solution. A design study covers which solution to choose. See [Implementation Plan Reference](impl-plan_definition.md).
- A decision log. A decision log records conclusions ("we chose X because Y"). A design study records the analysis that produced the conclusion -- the dimensions, the alternatives, the evaluation. If the reasoning doesn't matter, a decision log entry is sufficient; if it does, write a design study.

---

## 2. Structure

Every design study follows this skeleton. Sections marked **(required)** must always be present. Sections marked **(when applicable)** are included when the problem warrants them.

### 2.1. Metadata (required)

```markdown
---
title: <Topic>
status: draft | review | resolved | archived
date: YYYY-MM-DD
author: <git user name>
---
```

| Field | Purpose |
|---|---|
| `title` | Short, descriptive name for the problem being studied. Matches the `#` heading. |
| `status` | Lifecycle stage (see section 4). |
| `date` | Creation date in `YYYY-MM-DD` format. Must be the actual calendar date when the document is created, not copied from another document. Must match the date prefix in the filename. Immutable after creation. |
| `author` | Who owns this study. Use `git config user.name`. |

**File naming:** `YYYY-MM-DD_<topic>_design-study.md`. The date is the creation date. The topic should match the title in kebab-case. E.g., a design study titled "Document Structure" created on 2026-04-09 is named `2026-04-09_document-structure_design-study.md`.

### 2.2. Problem Statement (required)

One to three paragraphs describing:

- The problem or question being analyzed.
- Why it matters -- what decision it informs, what is at stake.
- The scope of the analysis -- what dimensions are in play, what is excluded.

The problem statement sets up the decomposition. A reader who only reads this section should understand what the study is about and why it exists.

### 2.3. Current State (when applicable)

Description of the status quo: what exists today, what's working, what's broken. Include concrete artifacts (file trees, examples, metrics) rather than abstract descriptions.

This section grounds the analysis in reality. Without it, the study risks solving a theoretical problem that doesn't match the actual situation.

### 2.4. Dimensions (required)

The sub-problems or axes along which the design space is decomposed. Each dimension gets a subsection containing:

- **Definition** -- what the dimension is and why it's independent enough to analyze separately.
- **Options** -- the alternatives within this dimension. Each option is a numbered subsection with a descriptive title (e.g., `#### Option 1: Hierarchical single-file`). The number is scoped to its dimension (each dimension restarts at 1). Each option must include:
  - **Properties** -- neutral, factual observations about the option (structure, behavior, constraints). Not value judgments.
  - **Pros** -- advantages of this option relative to the others.
  - **Cons** -- disadvantages of this option relative to the others.
- **Observations** -- interactions with other dimensions, constraints, implications.
- **Conclusion** (when reached) -- the recommended choice within this dimension and the rationale.

Dimensions are the backbone of the study. A well-chosen decomposition makes the evaluation tractable; a poor one creates false independence or hides interactions.

### 2.5. Design Options (when applicable)

When dimensions interact in ways that matter, enumerate the composed options -- specific combinations of choices across dimensions. Each option should include:

- A concrete representation (directory tree, API shape, configuration example).
- Pros and cons specific to this combination, not just the sum of per-dimension pros and cons.

This section exists because the best choice in each dimension independently may not be the best combination. If dimensions are truly independent, this section can be omitted.

### 2.6. Evaluation (required)

Comparison of options against explicit criteria. Prefer a matrix or table format for scanability. The criteria should be stated before the evaluation, not derived from the results.

The evaluation should address:

- How each option performs on each criterion.
- Which trade-offs are decisive and why.
- A recommendation, or an explicit statement of what must be resolved before a recommendation can be made.

### 2.7. Open Questions (when applicable)

Numbered list of unresolved questions that affect the recommendation. Each question should state:

- The question itself.
- What decision it blocks or what dimension it affects.
- Candidate answers, if any.

When a question is resolved, move the answer to the relevant dimension or evaluation section and mark the question resolved (strikethrough with the resolution noted). A resolved design study should have zero open questions.

### 2.8. Deliverables (required)

Explicit list of what this study produces. A design study is not an end in itself -- it feeds other documents or actions. Name them:

- Which specs, doc-definitions, standards, or configuration changes will be created from the conclusions?
- Which existing documents will be updated?

This section is the contract between the study and the rest of the system. It prevents studies from becoming permanent drafts with no clear output.

---

## 3. Principles

### 3.1. Decompose Before Evaluating

The most common design study failure is jumping to options before establishing the dimensions. Listing four directory layouts and comparing them doesn't explain *why* those layouts differ. The dimensions (role, scope, locality) are what make the evaluation tractable and the reasoning transferable.

Always define the dimensions first. Then show how options are composed from choices within those dimensions.

### 3.2. Concrete Over Abstract

Every dimension, option, and evaluation point should be grounded in a concrete example -- a file tree, a filename, a configuration snippet, a real document from the repo. Abstract principles without concrete instances are untestable and unfalsifiable.

### 3.3. Interactions Are the Hard Part

Individual dimensions are easy to analyze. The value of a design study is in surfacing how choices interact: "if we choose role-first for directories, then scope must be encoded in filenames." Dedicate explicit attention to cross-dimension interactions, either within dimension observations or in a composed design options section.

### 3.4. Reasoning Over Conclusions

A spec records conclusions. A design study records reasoning. When writing, optimize for the reader who needs to understand *why*, not just *what*. If a future maintainer faces a similar problem in a different context, the decomposition and evaluation framework should be reusable even if the specific recommendation doesn't apply.

### 3.5. Converge or Explain Why Not

A design study should end with a recommendation or an explicit list of what prevents one. "We explored but didn't decide" is acceptable only if the open questions section explains exactly what's blocking. A study that ends without a recommendation and without open questions has failed.

---

## 4. Lifecycle

| Status | Meaning | What changes |
|---|---|---|
| `draft` | Actively working through the problem space. Dimensions may be added or restructured. | Everything. |
| `review` | Analysis is complete. Recommendation is stated. Ready for feedback. | Evaluation and recommendation may be refined based on feedback. Dimensions are stable. |
| `resolved` | Decisions have been extracted into their deliverables (specs, doc-definitions, standards, etc.). | Nothing -- study is frozen. Deliverables section should list the resulting documents with links. |
| `archived` | Deliverables have been implemented. The study is a historical record. | Nothing. Do not update. If circumstances change enough to revisit the decisions, write a new study. |

The transition from `resolved` to `archived` happens when the deliverables are implemented and live. An archived study is a historical record of how a decision was reached, not a living document.

**Key difference from specs:** A spec transitions through `approved` → `implemented` because the spec itself is the commitment that gets implemented. A design study transitions through `resolved` → `archived` because the study produces other documents (specs, doc-definitions, standards) that carry the commitment. The study's job is done when those documents exist.

---

## 5. Anti-Patterns

| Anti-pattern | Problem | Fix |
|---|---|---|
| **Options without dimensions** | Jumps to comparing four layouts without explaining what makes them different. Reader can't evaluate trade-offs or adapt to their context. | Define dimensions first. Show how options compose from choices within dimensions. |
| **Dimension without options** | Names a dimension ("scope") but doesn't enumerate concrete alternatives within it. | Each dimension needs at least two concrete options with examples. If there's only one viable choice, it's a constraint, not a dimension. |
| **Abstract evaluation** | "Option A is better for large repos." Better how? By what measure? | State criteria explicitly. Use a comparison matrix. Ground claims in concrete examples or evidence. |
| **Permanent draft** | Study stays in `draft` indefinitely with no deliverables produced. | The deliverables section forces accountability. If no deliverables emerge, either the study wasn't needed or the open questions need to be resolved. |
| **Study as spec** | Study includes acceptance criteria, implementation steps, or commitments. | A study recommends; a spec commits. Extract conclusions into a spec or doc-standard. The study references its deliverables. |
| **Missing interactions** | Dimensions are analyzed in isolation. The composed options section is absent despite significant cross-dimension effects. | If choosing X in dimension A constrains what's viable in dimension B, that interaction must be documented explicitly. |
| **Conclusion without trail** | States a recommendation without the reasoning chain from dimensions through evaluation. | Every recommendation should be traceable: "We chose X because of criterion Y, which follows from dimension Z." |

---

## 6. Relationship to Other Document Types

| Document type | Relationship |
|---|---|
| **Spec** | A design study may produce a spec as a deliverable. The spec records the committed decision; the study records the analysis that led to it. The spec's Design Decisions section can reference the study for full rationale. |
| **Doc-standard** | A design study may produce a doc-standard as a deliverable (e.g., a study of document organization produces a doc-standard for directory structure). |
| **Implementation plan** | A plan implements a spec, not a study. The chain is: study → spec (or doc-standard) → plan. |
| **Cookbook** | If a study surfaces practical findings along the way (tool quirks, workarounds), those belong in a cookbook entry, not in the study. |

---

## 7. Minimal Example

````markdown
---
title: Authentication Strategy
status: draft
date: 2026-04-09
author: Jane Smith
---

# Authentication Strategy

The API currently uses session cookies. Mobile clients and third-party integrations need stateless authentication. This study evaluates alternatives.

## 1. Dimensions

### 1.1. Token Format

#### Option 1: JWT

- *Properties:* Self-contained, no server-side lookup. Larger payload.
- *Pros:* Stateless. No shared backend state.
- *Cons:* Not revocable without a blocklist, which reintroduces server-side state.

#### Option 2: Opaque token

- *Properties:* Requires server-side lookup. Smaller payload.
- *Pros:* Immediately revocable.
- *Cons:* Requires shared state (Redis or equivalent) across API servers.

**Observation:** Interacts with revocation strategy (dimension 1.2). Choosing JWT constrains revocation to short-lived tokens or a blocklist.

### 1.2. Revocation Strategy

#### Option 1: Short-lived tokens + refresh

- *Properties:* Access tokens expire in minutes. No explicit revocation needed.
- *Pros:* No server-side revocation state.
- *Cons:* Compromised tokens valid until expiry.

#### Option 2: Server-side blocklist

- *Properties:* Immediate revocation via shared blocklist.
- *Pros:* Immediate revocation.
- *Cons:* Requires shared state across API servers.

### 1.3. Storage (client-side)

#### Option 1: HTTP-only cookie

- *Properties:* Browser-managed. CSRF protection needed. Not usable by native apps.
- *Pros:* Automatic cookie handling in browsers.
- *Cons:* Web-only.

#### Option 2: Authorization header

- *Properties:* Stateless. Works for all clients. Token must be stored by the client.
- *Pros:* Universal (web, mobile, third-party).
- *Cons:* Client responsible for secure storage.

## 2. Design Options

### Option A: JWT + short-lived + Authorization header

Stateless. No server-side revocation. Mobile-friendly. Risk: compromised tokens are valid until expiry.

### Option B: Opaque + blocklist + Authorization header

Revocable. Requires Redis or equivalent for token lookup and blocklist. More infrastructure.

## 3. Evaluation

| Criterion | A (JWT + short-lived) | B (Opaque + blocklist) |
|---|---|---|
| Stateless | Yes | No |
| Immediate revocation | No (wait for expiry) | Yes |
| Infrastructure cost | Low | Medium (Redis) |
| Mobile support | Yes | Yes |

**Recommendation:** Option A. The 5-minute expiry window is acceptable for this threat model. Eliminates shared state dependency.

## 4. Open Questions

1. ~~**Refresh token rotation policy.**~~ Resolved: rotate on every use, one-time-use refresh tokens.

## 5. Deliverables

- [ ] Spec: `2026-04-10_stateless-auth_spec.md` -- commits to JWT + short-lived + refresh rotation.
- [ ] Plan: implementation plan derived from the spec.
````


---

## Governing Standards

This definition conforms to the [Document Structure Standards](../standards/document-structure_standards.md). See section 2.1 for the file naming convention and date rules.