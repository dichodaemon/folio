# Implementation Plan Definition

Reference for writing implementation plan documents. A plan is a working document. It answers: how are we building this, in what order, and what are the risks? It's most useful before and during the build. After the feature ships, it loses value quickly -- the code is the truth now. Don't try to keep it perfectly current post-ship; archive it instead.

---

## 1. Purpose

A plan exists to:

1. **Sequence** -- break the work into ordered steps so the implementer (human or agent) knows what to do next.
2. **Map** -- identify every file, function, and type that will be touched, so the blast radius is understood before the first edit.
3. **De-risk** -- surface non-obvious implementation choices, dependencies, and cleanup obligations before they become surprises mid-build.
4. **Track** -- provide a living checklist of what's done and what remains.

A plan is NOT:

- A spec. A spec covers *what* to build and *why*. A plan covers *how* to build it. Every plan should reference its companion spec (see [Specification Document Reference](spec-doc_definition.md)).
- A design document. If you're still choosing between approaches, you need a spec (with design decisions) or a design study (see [Design Study Definition](design-study_definition.md)), not a plan.
- An arch-design. An arch-design defines what to build (types, contracts, decomposition). A plan defines how to build it (task sequencing, file changes, status tracking). See [Architecture Design Definition](arch-design_definition.md).
- A post-implementation reference. After the feature ships, the code is the source of truth. Archive the plan; don't maintain it.

---

## 2. Two Templates

Plans fall into two categories with different structures. Use the one that matches your work.

| Template | Use when | Key sections |
|---|---|---|
| **New-feature plan** | Building something new from a spec | Spec link, status table, architecture, data model, solution breakdown, design decisions, success criteria |
| **Bug-fix plan** | Investigating and fixing a defect | Problem statement, root cause, diagnostic evidence, fix, validation, cleanup |

Both share universal patterns (section 2.3) but differ in structure because the work is fundamentally different: new features are designed forward from a spec; bug fixes are investigated backward from a symptom.

---

## 3. Structure: New-Feature Plan

### 3.1. Metadata (required)

```markdown
---
title: <Feature Name> -- Implementation Plan
status: draft | approved | issued | archived
date: YYYY-MM-DD
author: <git user name>
spec: <path-to-companion-spec>
arch-design: <path-to-companion-arch-design> (optional)
---
```

| Field | Purpose |
|---|---|
| `title` | Matches the spec title with "-- Implementation Plan" suffix. |
| `status` | `draft` while planning, `approved` when reviewed and ready for issue creation, `issued` when all tasks are converted to tracker issues, `archived` post-ship. |
| `date` | Creation date in `YYYY-MM-DD` format. Must be the actual calendar date when the document is created, not copied from another document. Must match the date prefix in the filename. Immutable after creation. |
| `author` | Who owns this plan. Use `git config user.name`. |
| `spec` | Relative path to the companion spec document. Every new-feature plan must have a companion spec, arch-design, or both. |
| `arch-design` | Optional. Relative path to the companion arch-design document. Every new-feature plan must have a companion spec, arch-design, or both. |

**File naming:** `YYYY-MM-DD_<feature-name>_plan.md`, using the same date and feature slug as the companion spec. E.g., `2026-04-07_build-log-notifier_plan.md`.

### 3.2. Implementation Status (required)

A table tracking completion of each sub-task. Start with a phase summary that names each phase and its dependency, then list tasks with two-level numbering where the first digit is the phase:

```markdown
**Phases:**
1. **Data model** -- types and fields that later steps depend on
2. **Logic** -- computations and validation using the data model (depends on phase 1)
3. **Integration** -- wiring into existing system (depends on phase 2)

| # | Task | Status |
|---|---|---|
| 1.1 | Define `TokenBucket` struct in rate_limiter module | Complete |
| 1.2 | Add `RateLimitPolicy` to config schema | Complete |
| 2.1 | Implement token consumption logic | Pending |
| 2.2 | Implement sliding window counter | Pending |
| 3.1 | Wire into HTTP middleware | Pending |
```

Tasks within the same phase can be parallelized (1.1 and 1.2 have no dependency between them). Tasks in later phases depend on earlier phases completing. The phase summary makes this explicit -- an agent or implementer resuming mid-plan reads the phase list to understand what's safe to pick up next.

Update this table as you work. It's the first thing a reviewer or resuming agent checks to understand where things stand.

### 3.3. Architecture (required)

Two subsections:

**3.3.1. Directory Layout** -- a table of files that will be created or modified:

```markdown
| File | Change |
|---|---|
| `src/rate_limiter/token_bucket.h` | New file: TokenBucket struct and helpers |
| `src/middleware/rate_limit_handler.cc` | Add rate-limiting middleware logic |
| `src/middleware/BUILD.bazel` | Add deps for new rate_limiter helpers |
```

**3.3.2. Dependency Graph** (when applicable) -- an ASCII diagram or prose description showing how components relate. Include this when the change touches multiple packages or introduces new inter-package dependencies.

### 3.4. Data Model Changes (when applicable)

One subsection per new or modified type, field, message, or schema. Include the exact definition (C++ struct, protobuf message, JSON schema) with inline annotations explaining non-obvious choices.

````markdown
#### `TokenBucket` struct (`rate_limiter/token_bucket.h`)

```cpp
struct TokenBucket {
  double capacity;        // Maximum tokens
  double refill_rate;     // Tokens per second
  double current_tokens;  // Available tokens (mutable)
  int64_t last_refill_ns; // Monotonic clock timestamp
};
```

Why monotonic clock: wall-clock adjustments (NTP, DST) would cause spurious token grants or starvation.
````

### 3.5. Solution Breakdown (when applicable)

Decomposes the spec's Solution into implementable pieces. One subsection per logical unit of work: a computation, a predicate, an integration point, a data flow path. This is the plan's counterpart to the spec's Solution section.

Each subsection should include:

- Function signature (or where the logic will live)
- Step-by-step description of the logic
- Edge cases and how they're handled
- **Dependencies:** what this step produces (types, helpers, interfaces) that later steps consume, and what it requires from earlier steps

When a subsection produces an artifact that other subsections depend on (e.g., a helper function, a type definition, a proto field), state the dependency explicitly. This is what drives the phase grouping in the status table -- if step A produces a type that step B consumes, they must be in different phases.

Keep this at the level of "what the code does," not "what the code looks like." Pseudocode or numbered steps are better than full implementations -- the code will diverge from copy-pasted snippets, creating a stale-plan problem.

### 3.6. Design Decisions (when applicable)

One subsection per non-obvious implementation choice. Each states:

- The decision
- What was considered
- Why this choice was made

These are *implementation* decisions, not *design* decisions. The spec's design decisions answer "what approach?"; the plan's design decisions answer "what implementation detail within that approach?"

Example: the spec decides "use a shell wrapper." The plan decides "define it in `~/.bashrc.d/` rather than `/usr/local/bin/` because the dev container rebuilds `/usr/local/` on restart."

### 3.7. Success Criteria (required)

Per-component checklist of observable pass/fail conditions. These are more granular than the spec's acceptance criteria -- they verify each implementation step, not just the end-to-end behavior.

```markdown
### TokenBucket struct
- [ ] `TokenBucket` compiles with no warnings under `-Wall -Werror`
- [ ] Unit test covers burst and steady-state token consumption

### HTTP middleware integration
- [ ] Middleware returns HTTP 429 when bucket is empty
- [ ] Middleware passes request through when tokens are available
```

When all success criteria are verified, the plan is ready for approval.


### 3.8. Document Staleness Audit (required)

Implementation work changes the codebase. Existing documents -- specs, arch-designs, READMEs, cookbooks -- may describe behavior or interfaces that the plan's changes invalidate. The plan must identify these documents and include tasks to update or archive them.

**Procedure:**

1. List every companion and related document (specs, arch-designs, READMEs) that describe the types, interfaces, or behaviors the plan modifies.
2. For each document, state whether the plan's changes invalidate any of its content.
3. If invalidated: add a task to the implementation status table (section 3.2) to update or archive the document. Place it in the last implementation phase or in a dedicated "Documentation" phase.
4. If not invalidated: state why (e.g., "spec describes the interface contract, which is unchanged; only the internal implementation changes").

````markdown
### Documents reviewed

| Document | Invalidated? | Action |
|---|---|---|
| `../specs/2026-04-13_pull-over-on-shoulder_spec.md` | Yes -- section 3 "Reference spline" decision is superseded. | Task 5.6: update spec design decisions table. |
| `../arch-designs/behavior-planner_arch-design.md` | No -- arch-design describes the ManeuverDescriptor contract, which is unchanged. | None. |
| `../../planning/sampling/README.md` | Yes -- describes ego-lane-only reference spline assumption. | Task 5.7: update README sampling overview. |
````
---

## 4. Structure: Bug-Fix Plan

### 4.1. Metadata (required)

```markdown
---
title: <Bug Summary> -- Fix Plan
status: draft | approved | issued | archived
date: YYYY-MM-DD
author: <git user name>
issue: <Jira ticket, GitHub issue URL, or path to issue document> (optional)
---
```

No `spec` field -- bug fixes typically don't have companion specs.

| Field | Purpose |
|---|---|
| `issue` | Optional. Reference to the tracking ticket (e.g., `PROJ-1234`, `https://github.com/org/repo/issues/42`, or a relative path to a local issue document). Links the fix to whatever triggered the investigation. |

**File naming:** `YYYY-MM-DD_<bug-summary>-fix_plan.md`. E.g., `2026-04-07_stale-cache-branch-switch-fix_plan.md`.

### 4.2. Problem Statement (required)

One paragraph describing:

- The observable symptom (what the user or system sees)
- The conditions under which it occurs
- The impact (severity, frequency)

Write in terms of observable behavior, not suspected cause. The cause comes next.

### 4.3. Root Cause (required)

The failure chain from trigger to symptom. Structure depends on how the investigation went:

**Simple root cause:** One subsection describing the code path that fails and why.

**Multi-hypothesis investigation:** Subsections for:
- Initial hypothesis (and why it was wrong, if applicable)
- What the evidence showed (logs, traces, repro steps)
- Actual root cause (the code path, the invariant violation, the missing check)

Include enough detail that a reader who wasn't present during the investigation can follow the reasoning. This is the most valuable section for future maintainers encountering similar symptoms.

### 4.4. Diagnostic Evidence (when applicable)

Verbatim log snippets, trace output, or metric screenshots that prove the root cause. Annotate them -- raw logs without context are noise.

```markdown
Timeout stuck at sentinel despite upstream response received:
```
[2026-04-07 14:23:01] response_latency_ms: 1e9    <- sentinel, never updates
[2026-04-07 14:23:01] upstream_responses: 1         <- response exists
[2026-04-07 14:23:01] request_id: 42
[2026-04-07 14:23:01] routing_group: 99             <- wrong group
```
Routing group 99 is the fallback group. The request's group reference
was assigned at dispatch time using a lookup that matched the wrong
entry due to a race between config reload and request routing.
```

### 4.5. Fix (required)

For each fix applied:

- **File and function** where the change is made
- **What changed** and why
- **Before/after** code diff or description

If multiple fixes are applied (e.g., a primary fix and a defensive fix), number them and state which is the root-cause fix and which are defensive.

### 4.6. Validation (required)

Observable conditions that confirm the fix works. Prefer before/after comparisons:

```markdown
- [ ] Before fix: `response_latency_ms` stuck at `1e9` when upstream has responded.
      After fix: `response_latency_ms` reflects actual round-trip time.
- [ ] Before fix: request times out despite successful upstream response.
      After fix: request completes within 2 seconds of upstream response arrival.
```

### 4.7. Cleanup (required)

Explicit list of diagnostic instrumentation (log statements, debug topics, temporary flags) added during investigation that must be removed or downgraded after validation.

```markdown
| File | Line | Action |
|---|---|---|
| `request_handler.cc:142` | `LOG_INFO() << "request_id: " ...` | Remove |
| `config_loader.cc:87` | `LOG_INFO() << "routing_group: " ...` | Downgrade to DEBUG |
```

Cleanup is a deliverable, not an afterthought. The plan is not complete until diagnostic instrumentation is resolved.

---

## 5. Universal Patterns

These apply to both templates:

- **Every change is tied to an exact file path and function name.** No "update the relevant module" -- name the file.
- **Validation is stated in terms of observable artifacts** (log lines, topic values, test output), not "it should work."
- **Cleanup is explicit.** Any instrumentation added during the work is tracked and resolved.
- **Inline code uses full signatures**, not pseudocode fragments. The reader should be able to grep the codebase and find the exact location.
- **The plan references its companion spec** (new-feature plans) or stands alone with a problem statement (bug-fix plans). It never duplicates the spec's objective or non-goals.

---

## 6. Lifecycle

### 6.1. Document Status

| Status | Meaning | What changes |
|---|---|---|
| `draft` | Planning phase. Structure may change. | Everything. |
| `approved` | Plan reviewed and accepted. Ready for issue creation. | Nothing in the plan content. Issue tracking handoff (section 6.2) begins. |
| `issued` | All tasks converted to issues in the issue tracker. | Nothing. Plan is frozen -- the issue tracker is now the execution authority. |
| `archived` | Feature shipped or bug fix merged. Code is now the truth. | Nothing. Do not update. If a future change invalidates the plan, write a new plan. |

### 6.2. Issue Tracking Handoff

When the plan transitions from `draft` to `approved`, convert it into trackable issues. The plan is a design artifact; the issue tracker is the execution artifact. They are not redundant -- the plan captures *why* and *how*; the issues track *what remains* and *who is doing it*.

**Procedure:**

1. **Create a master epic** for the entire plan. Title: the plan's title. Description: link to the plan file.
2. **Create one phase epic per phase** in the implementation status table (section 3.2). Title: the phase name. Link each phase epic as a child of the master epic (`parent-child`).
3. **Create one task issue per task** in the status table. Link each task as a child of its phase epic (`parent-child`).
4. **Wire `blocks` dependencies** between phase epics according to the dependency declarations in the phase summary (e.g., "depends on phase 1" becomes a `blocks` edge from phase 1's epic to phase 2's epic).
5. **Wire `blocks` dependencies between tasks** when the solution breakdown (section 3.5) declares explicit inter-task dependencies within a phase.
6. **Include staleness audit tasks** from section 3.8 as issues in the appropriate phase epic.
7. **Update the plan's status** to `issued`. The plan is now frozen.

After handoff, the issue tracker is the authority on what is done and what remains. Do not update the plan's status table -- it reflects the state at the time of handoff, not the current execution state.

### 6.3. Batch Issue Creation

When the issue tracker supports batch creation (e.g., `bd create --graph <plan.json>`), convert the plan's phases and tasks into a single graph file and create all issues in one operation. This avoids per-issue overhead and ensures the full dependency graph is visible from the start.

The graph file encodes:
- One node per epic (master + phases) and one node per task.
- `parent-child` edges wiring children to parents (child is `from_key`, parent is `to_key` — the child depends on the parent). Phase epics are children of the master epic; tasks are children of their phase epic.
- `blocks` edges between phase epics (from the phase dependency declarations).
- `blocks` edges between tasks (from the solution breakdown dependencies).

### 6.4. Lifecycle Transitions

```
draft ──> approved ──> issued ──> archived
  │                                  ▲
  │          (work happens via       │
  │           issue tracker)         │
  │                                  │
  └──────────────────────────────────┘
              (trivial plans may skip
               straight to archived)
```

The `in-progress` and `complete` statuses from the pre-issue-tracking workflow are no longer used. The issue tracker replaces the plan's status table as the progress-tracking mechanism.

---

## 7. Anti-Patterns

| Anti-pattern | Problem | Fix |
|---|---|---|
| **Plan without spec or arch-design** (new features) | No agreement on *what* to build; the plan is implementing assumptions. | Write a spec or arch-design first. The plan references it. |
| **Plan as spec** | Mixes "what to build" with "how to build it." Objective, non-goals, and acceptance criteria are embedded in the plan. | Split into spec (what/why) and plan (how). |
| **Pseudocode plan** | Solution breakdown uses invented syntax instead of real signatures and types. Implementer can't find the code. | Use real function names, real types, real file paths. |
| **Missing cleanup** | Debug logs and temporary instrumentation ship to production. | Add a cleanup section. Don't mark the work done until it's resolved. |
| **Tracking progress in the plan** | Plan status table diverges from issue tracker. Two sources of truth. | After `issued`, the issue tracker is authoritative. Do not update the plan's status table. |
| **Post-ship maintenance** | Updating the plan after the code has diverged. The plan becomes a second source of truth that contradicts the code. | Archive the plan. Write a new plan if a new change is needed. |
| **No validation criteria** | "Test that it works" instead of observable before/after conditions. | State what you'll observe, in what tool, with what values. |
| **Hypothesis-free debugging** | Bug-fix plan jumps to a fix without documenting the root cause investigation. | Document what you investigated, what you found, and why. The next person with similar symptoms needs this. |
| **Skipping issue handoff** | Plan approved but tasks never converted to issues. No dependency tracking, no parallelization visibility, no persistent memory across sessions. | Convert to issues immediately on approval (section 6.2). |
| **Missing staleness audit** | Plan changes interfaces described in existing docs. Docs go stale silently. | Always complete section 3.8 before approval. |

---

## Governing Standards

This definition conforms to the [Document Structure Standards](../standards/document-structure_standards.md). See section 2.1 for the file naming convention and date rules.
