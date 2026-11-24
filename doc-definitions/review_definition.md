# Review Definition

Definition for writing review documents. A review is a point-in-time engineering assessment of one specific change -- a pull request, a branch, or a set of commits -- against a stated baseline. It explains the change to a reader who has never seen it, lists every issue the change would introduce or leave in place, ordered by priority and grouped by root cause, and recommends an action for each. Its value is in clarity: a solid engineer with no prior exposure to the change should be able to read any single issue and understand what goes wrong, why it matters, how the baseline handles it, how the change handles it, and what to do.

---

## 1. Purpose

A review exists to:

1. **Explain** -- give an unfamiliar reader enough background on the baseline and the change to follow every issue without opening the code.
2. **Diagnose** -- identify each problem by its root cause, not by the symptom or the file where it shows.
3. **Prioritize** -- rank the issues by impact and by when they must be dealt with, so the reader knows what blocks the change and what can follow.
4. **Recommend** -- state, for every issue, the action that resolves it and how to tell that it is resolved.

A review is NOT:

- **A report.** A report surveys a landscape and does not prescribe. A review is about one change and recommends actions. See [Report Definition](report_definition.md).
- **A design study.** A design study decomposes a problem space and evaluates alternatives before a solution exists. A review assesses a solution that already exists. A review may conclude that a design study is needed for one of its issues. See [Design Study Definition](design-study_definition.md).
- **A brief.** A brief frames work before commitment and is short by design. A review analyses a concrete change in whatever depth clarity requires. See [Brief Definition](brief_definition.md).
- **An arch-assessment.** An arch-assessment measures a component's dependency structure with metrics. A review assesses a change across correctness, capability, architecture, complexity and latency. See [Architecture Assessment Definition](arch-assessment_definition.md).
- **A code review.** Line-level comments belong on the pull request. A review collects what matters at the level of behaviour and design, and may cite code-review comments as one of its sources.
- **A living document.** A review is frozen once accepted. If the change evolves materially, write a new review.

---

## 2. When to Write One

Write a review when:

- A change is large or cross-cutting enough that its consequences are not visible from the diff, and a merge decision needs a shared, written basis.
- Several people need to agree on what blocks a change and what can follow.
- The findings from code review, testing, simulation and reading the code need to be consolidated into one place, without any one source dominating.

Do not write a review when:

- The change is small and its pull-request discussion is enough.
- You are choosing between designs that do not exist yet. Write a design study.
- You are surveying a landscape without recommending. Write a report.

---

## 3. Structure

Every review follows this skeleton. Sections marked **(required)** must always be present; sections marked **(when applicable)** are included when the change warrants them.

### 3.1. Metadata (required)

```markdown
---
title: <Topic>
status: draft | accepted
date: YYYY-MM-DD
author: <git user name>
subject: <PR number, branch and head commit under review>
baseline: <what the change is compared against, with commit>
---
```

| Field | Purpose |
|---|---|
| `title` | Short, descriptive name. Matches the `#` heading. |
| `status` | `draft` while the review is written and revised; `accepted` once it is settled and shared. Content freezes at `accepted`. |
| `date` | Creation date in `YYYY-MM-DD` format. Must be the actual calendar date when the document is created. Must match the date prefix in the filename. Immutable after creation. |
| `author` | Who wrote this review. Use `git config user.name`. |
| `subject` | The exact change under review, precise enough to check out. |
| `baseline` | The exact reference the change is compared against, precise enough to check out. |

**File naming:** `YYYY-MM-DD_<topic>_review.md`. Example: `2026-11-24_secondary-planner-mapping-integration_review.md`.

### 3.2. Bottom Line (required)

Half a page at most. What the change does, which issues block it, and the recommended actions in order. A reader who stops here must know whether the change can merge and what has to happen first.

### 3.3. Background (required)

What a reader needs to follow every issue without opening the code: what the affected component does, how the baseline does it, how the change does it, and the terms the issues use. Prefer one diagram per path over prose. Define each term once here and use it consistently afterwards.

### 3.4. Scope (required)

What was reviewed and what was not. A table or two short lists.

### 3.5. Sources (required)

Where the evidence came from: commits, files, test and simulation runs, code-review comments, other documents. Code-review comments are one source among others; they are cited from the issues they relate to, not given a section of their own.

### 3.6. Priority and Effort Scales (required)

The priority scale used to rank issues, with an impact test and the expected timing of action for each level, and the effort scale used for recommendations, with a written definition for each size. Recommended defaults:

| Priority | Impact | Action |
|---|---|---|
| **P0** | Unsafe behaviour, data corruption, or loss of a critical function in a configuration in use | Resolve before merge |
| **P1** | A capability lost or behaviour changed in a way testing shows; an architectural property weakened | Resolve before merge, or before the affected configuration is used, with a tracked ticket |
| **P2** | Debt that makes later defects more likely or maintenance costlier | Follow-up ticket |
| **P3** | Costs reading time only | Opportunistic |

Effort sizes are defined by scope, not by time. Each review states its definitions of **S**, **M** and **L**.

### 3.7. Configurations of Record (when applicable)

When the impact of an issue depends on where the code runs, one table of the deployment configurations the issues refer to: what runs in each and how it differs between the baseline and the change. Issues point to this table instead of repeating it.

### 3.8. Issues (required)

One subsection per priority level, highest first. Within a level, the most consequential issue first. Each issue is one root cause; symptoms of the same cause belong to the same issue. Every issue follows the same template, with the priority in the title:

```markdown
#### 6.1.1. P0 — <What goes wrong, stated as a consequence>

**Origin.** New, Pre-existing, or Partly pre-existing, with one sentence saying which part is which when it isn't obvious.

**The issue.** What goes wrong, when, and for whom, in the terms defined in Background.

**Why P0.** The priority test applied to this issue, naming the configurations it affects.

**On the baseline.** How the baseline handles the same concern, naming the mechanism.

**On the change.** What the change does instead, and what follows from it.

**Recommendation.** The action that resolves the issue, with alternatives where a decision is needed, the effort size, and what shows that it is resolved.

**Evidence.** Test, simulation or measurement results; related code-review comments; file and line references.
```

Statements that are derived but not observed are marked **[INFERENCE]**.

**Origin** takes one of three values, followed by a sentence when it isn't plain: **New** (the change introduces the issue), **Pre-existing** (the baseline already has it; the change leaves it in place or puts it on a new path), or **Partly pre-existing** (part of the cause is in the baseline, part is new; say which).

### 3.9. Validation Status (when applicable)

What testing, simulation and measurement show about the change as a whole: which runs exist, what they show, and what has not been run.

### 3.10. Recommended Actions (required)

Every recommendation from the issues, one line each, ordered by when it must happen (for example: before merge, before use in a given configuration, follow-up). Each line names its issue and effort.

### 3.11. Open Questions (when applicable)

Only what cannot be verified at the time of writing, each with who or what can answer it. Anything that can be settled by reading code or running an available test is settled before the review is accepted.

---

## 4. Principles

### 4.1. Written for a Newcomer

Every issue must be understandable by a solid engineer who has not seen the change. Background carries the shared context; issues use its terms. Length is not a constraint; clarity is.

### 4.2. One Issue per Root Cause

Group symptoms under the cause that produces them. An issue that would be fixed by the same change as another issue is the same issue.

### 4.3. Sorted by Priority

The reader meets the most important issue first. Priority appears in every issue title.

### 4.4. One Template

Every issue has the same parts in the same order. A reader who has read one issue knows where to look in all the others.

### 4.5. Evidence-Anchored

Every claim about behaviour is anchored to code, a run, or a measurement. Inferences are labelled.

### 4.6. Sources Are Not Structure

No source -- code-review comments, a test suite, a simulation campaign -- dictates the document's structure. Issues are organized by root cause and priority; sources are cited from within issues.

### 4.7. Frozen at Acceptance

A review is revised freely in `draft` and frozen at `accepted`. If the change evolves materially after acceptance, write a new review.

---

## 5. Anti-Patterns

| Anti-pattern | Problem | Fix |
|---|---|---|
| **Issue per symptom** | The same cause appears as several issues in different sections; the reader cannot tell which fix resolves what. | Merge symptoms under their root cause. |
| **Source-shaped review** | A section per code-review comment, test run or reviewer; the same issue is told several times. | Organize by root cause and priority; cite sources inside issues. |
| **Severity in an appendix** | Priority lives in a summary table far from each issue. | Put the priority in each issue's title and justify it in the issue. |
| **Missing baseline** | The issue describes only the change, so the reader cannot tell what is lost or gained. | State how the baseline handles the concern in every issue. |
| **Recommendation-free issue** | The issue explains a problem but leaves the action implicit. | Give every issue a recommendation with an effort size and a check. |
| **Expert-only prose** | Issues assume familiarity with the change's internals. | Move shared context into Background and define terms once. |
| **Multiple states in parallel** | Every comparison carries intermediate versions of the change, multiplying columns and caveats. | Compare the baseline with the change as it would merge; mention intermediate states only where they are the evidence. |

---

## 6. Relationship to Other Document Types

| Document type | Relationship |
|---|---|
| **Report** | A report may supply evidence a review cites. A review is about one change and recommends; a report surveys and does not. |
| **Design study** | A review may conclude that an issue needs a design study before it can be resolved. |
| **Design note** | A review may cite the design note behind the change and record where the change departs from it. |
| **Impl-plan** | The recommended actions of an accepted review may be turned into a plan. |
| **Brief** | A brief may frame the review work itself; the review holds the findings. |

---

## 7. Minimal Example

```markdown
---
title: Cache Eviction Rewrite Review
status: draft
date: 2026-07-01
author: Jane Smith
subject: PR #1234, branch jane/lru-eviction, head 1a2b3c4
baseline: master at 5d6e7f8
---

# Cache Eviction Rewrite Review

## 1. Bottom Line

PR #1234 replaces the cache's FIFO eviction with LRU. One P0 issue blocks it:
entries pinned by in-flight requests can now be evicted. Fix the pin check
(S), then merge; the two P2 issues can follow.

## 2. Background

The cache holds decoded tiles keyed by tile id. On master, eviction removes
the oldest insertion first and skips pinned entries. The PR tracks access
order instead ...

## 3. Scope

| In scope | Out of scope |
|---|---|
| Eviction policy and pinning | Serialization format |

## 4. Sources

1. PR #1234 diff, `5d6e7f8..1a2b3c4`
2. `cache_stress_test` runs on both commits

## 5. Priority and Effort Scales

[Priority table; S = one function, existing tests; M = one module, new tests;
L = crosses modules or needs a design decision.]

## 6. Issues

### 6.1. P0

#### 6.1.1. P0 — Pinned entries can be evicted mid-request

**Origin.** New.
**The issue.** ...
**Why P0.** ...
**On the baseline.** ...
**On the change.** ...
**Recommendation.** ... Effort: S. Resolved when `cache_stress_test` passes
with pinning enabled.
**Evidence.** ...

## 7. Recommended Actions

1. Before merge: restore the pin check (6.1.1, S).
2. Follow-up: ...
```

---

## Governing Standards

This definition conforms to the [Document Structure Standards](../standards/document-structure_standards.md). See section 2.1 for the file naming convention and date rules.
