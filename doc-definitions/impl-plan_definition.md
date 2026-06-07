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

- A spec. A spec covers *what* to build and *why*. A plan covers *how* to build it. Every plan should reference its companion document (see [Specification Document Reference](spec-doc_definition.md)).
- A design document. If you're still choosing between approaches, you need a spec (with design decisions) or a design study (see [Design Study Definition](design-study_definition.md)), not a plan.
- An arch-design. An arch-design defines what to build (types, contracts, decomposition). A plan defines how to build it (task sequencing, file changes, status tracking). See [Architecture Design Definition](arch-design_definition.md).
- A brief. A brief documents investigation, analysis, or root cause findings. A plan references a brief; it does not duplicate its content.
- A post-implementation reference. After the feature ships, the code is the source of truth. Archive the plan; don't maintain it.

---

## 2. Structure

### 2.1. Metadata (required)

```markdown
---
title: <Topic> -- Implementation Plan
status: draft | approved | issued | archived
date: YYYY-MM-DD
author: <git user name>
spec: <path-to-companion-spec> (optional)
arch-design: <path-to-companion-arch-design> (optional)
brief: <path-to-companion-brief> (optional)
issue: <Jira ticket, GitHub issue URL, or path to issue document> (optional)
---
```

| Field | Purpose |
|---|---|
| `title` | Topic with "-- Implementation Plan" suffix. |
| `status` | `draft` while planning, `approved` when reviewed and ready for issue creation, `issued` when all tasks are converted to tracker issues, `archived` post-ship. |
| `date` | Creation date in `YYYY-MM-DD` format. Must be the actual calendar date when the document is created, not copied from another document. Must match the date prefix in the filename. Immutable after creation. |
| `author` | Who owns this plan. Use `git config user.name`. |
| `spec` | Optional. Relative path to the companion spec document. |
| `arch-design` | Optional. Relative path to the companion arch-design document. |
| `brief` | Optional. Relative path to the companion brief (e.g., investigation results, root cause analysis, benchmark findings). |
| `issue` | Optional. Reference to the tracking ticket (e.g., `PROJ-1234`, `https://github.com/org/repo/issues/42`, or a relative path to a local issue document). Links the plan to whatever triggered the work. |

Every plan must have at least one companion document: `spec`, `arch-design`, `brief`, or `issue`. The plan never duplicates the companion's content -- it references it. Investigation artifacts (root cause analysis, diagnostic evidence, hypothesis logs) belong in a brief, not in the plan.

**File naming:** `YYYY-MM-DD_<topic>_plan.md`. E.g., `2026-04-07_build-log-notifier_plan.md`.

### 2.2. Implementation Status (required)

A table tracking completion of each sub-task. Start with a phase summary that names each phase and its dependency, then list tasks with two-level numbering where the first digit is the phase:

```markdown
**Phases:**
1. **Data model** -- types and fields that later steps depend on
2. **Logic** -- computations and validation using the data model (depends on phase 1)
3. **Integration** -- wiring into existing system (depends on phase 2)

| # | Task | Status |
|---|---|---|
| 1.1 | Define `TokenBucket` struct in `src/rate_limiter/token_bucket.h` | Complete |
| 1.2 | Add `RateLimitPolicy` to config schema in `src/config/rate_limit.proto` | Complete |
| 1.3 | Verify: `bazel build //src/rate_limiter:token_bucket //src/config:rate_limit_proto` succeeds | Complete |
| 2.1 | Implement token consumption logic in `src/rate_limiter/token_bucket.cc` | Pending |
| 2.2 | Write test: burst and steady-state consumption in `src/rate_limiter/token_bucket_test.cc` | Pending |
| 2.3 | Verify: `bazel test //src/rate_limiter:token_bucket_test` passes | Pending |
| 2.4 | Implement sliding window counter in `src/rate_limiter/sliding_window.cc` | Pending |
| 2.5 | Write test: window rollover and edge reset in `src/rate_limiter/sliding_window_test.cc` | Pending |
| 2.6 | Verify: `bazel test //src/rate_limiter:sliding_window_test` passes | Pending |
| 2.7 | Verify: `bazel build //src/rate_limiter/...` succeeds | Pending |
| 3.1 | Wire into HTTP middleware in `src/middleware/rate_limit_handler.cc` | Pending |
| 3.2 | Verify: `bazel build //src/middleware/...` succeeds | Pending |
```

Tasks within the same phase can be parallelized (1.1 and 1.2 have no dependency between them). Tasks in later phases depend on earlier phases completing. The phase summary makes this explicit -- an agent or implementer resuming mid-plan reads the phase list to understand what's safe to pick up next.

Update this table as you work. It's the first thing a reviewer or resuming agent checks to understand where things stand.

**Task descriptions.** Each task row must name the target file(s) and the change. The solution breakdown (section 2.5) must have a subsection for each task or group of tasks, cross-referenced by task number.

**Verify tasks.** Every implementation task that produces a testable artifact must be followed by a "Verify:" task naming the exact command and observable pass/fail condition. Verify tasks are phase gates -- they block the phase epic from closing and transitively block all subsequent phases. Every phase must end with a "Verify:" task that confirms the build succeeds for all targets touched by that phase. This is the minimum gate; additional verification (tests, parity checks) may be added.

**Test tasks.** For every task that changes production logic, add a separate task for writing or updating the unit test that covers that logic. Test-writing (new coverage for new behavior) is a distinct task from test-updating (making existing tests compile against changed APIs).

### 2.3. Architecture (required)

Two subsections:

**2.3.1. Directory Layout** -- a table of files that will be created or modified:

```markdown
| File | Change |
|---|---|
| `src/rate_limiter/token_bucket.h` | New file: TokenBucket struct and helpers |
| `src/middleware/rate_limit_handler.cc` | Add rate-limiting middleware logic |
| `src/middleware/BUILD.bazel` | Add deps for new rate_limiter helpers |
```

**2.3.2. Dependency Graph** (when applicable) -- an ASCII diagram or prose description showing how components relate. Include this when the change touches multiple packages or introduces new inter-package dependencies.

### 2.4. Interface Changes (when applicable)

One subsection per new, modified, or removed interface element: types, fields, structs, functions, methods, or signatures. Include the exact definition with inline annotations explaining non-obvious choices.

**Types and fields:** C++ structs, protobuf messages, JSON schemas, enum variants. Show the definition and annotate non-obvious choices.

**Functions and methods:** New or changed signatures. State what changed, why, and which callers need updating. For removed functions, list the replacement (if any) and confirm zero remaining callers.

Group by file when a single file has both type and function changes -- the reader should not have to cross-reference two subsections for the same contract.

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

#### `ConsumeTokens` (new) (`rate_limiter/token_bucket.h`)

```cpp
/// @brief Attempt to consume `count` tokens from the bucket.
/// @return true if tokens were available and consumed; false if the bucket
///         is empty (no tokens consumed, caller should reject the request).
bool ConsumeTokens(TokenBucket& bucket, int count, int64_t now_ns);
```

Called by `RateLimitHandler::HandleRequest`. Refills tokens based on elapsed time before checking availability.

#### `EvaluateBoundaryOffsets` (removed) (`behavior_vocabulary/lane_topology.h`)

Replaced by `Spline2DEvaluateProjection` on raw boundary splines in `EnumerateCorridors`. Zero remaining callers after task 3.3.
````

### 2.5. Solution Breakdown (when applicable)

Decomposes the work into implementable pieces. One subsection per logical unit of work: a computation, a predicate, an integration point, a data flow path.

Each subsection should include:

- Function signature (or where the logic will live)
- Step-by-step description of the logic
- Edge cases and how they're handled
- **Dependencies:** what this step produces (types, helpers, interfaces) that later steps consume, and what it requires from earlier steps
- **Done condition:** the observable check that confirms this step is correct, cross-referencing the "Verify:" task in the status table (section 2.2) that enforces it

When a subsection produces an artifact that other subsections depend on (e.g., a helper function, a type definition, a proto field), state the dependency explicitly. This is what drives the phase grouping in the status table -- if step A produces a type that step B consumes, they must be in different phases.

Keep this at the level of "what the code does," not "what the code looks like." Pseudocode or numbered steps are better than full implementations -- the code will diverge from copy-pasted snippets, creating a stale-plan problem.

### 2.6. Design Decisions (when applicable)

One subsection per non-obvious implementation choice. Each states:

- The decision
- What was considered
- Why this choice was made

These are *implementation* decisions, not *design* decisions. The spec's design decisions answer "what approach?"; the plan's design decisions answer "what implementation detail within that approach?"

Example: the spec decides "use a shell wrapper." The plan decides "define it in `~/.bashrc.d/` rather than `/usr/local/bin/` because the dev container rebuilds `/usr/local/` on restart."

### 2.7. Success Criteria (required)

Per-component checklist of observable pass/fail conditions. These are more granular than the spec's acceptance criteria -- they verify each implementation step, not just the end-to-end behavior.

```markdown
### TokenBucket struct
- [ ] `TokenBucket` compiles with no warnings under `-Wall -Werror`
- [ ] Unit test covers burst and steady-state token consumption

### HTTP middleware integration
- [ ] Middleware returns HTTP 429 when bucket is empty
- [ ] Middleware passes request through when tokens are available
```

When all success criteria are verified, the plan is ready for approval. Each criterion must trace to a specific "Verify:" task in the status table (section 2.2). Criteria not represented as Verify: tasks are unenforceable -- they will not be checked during execution.

### 2.8. Document Staleness Audit (required)

Implementation work changes the codebase. Existing documents -- specs, arch-designs, READMEs, cookbooks -- may describe behavior or interfaces that the plan's changes invalidate. The plan must identify these documents and include tasks to update or archive them.

**Procedure:**

1. List every companion and related document (specs, arch-designs, briefs, READMEs) that describe the types, interfaces, or behaviors the plan modifies.
2. For each document, state whether the plan's changes invalidate any of its content.
3. If invalidated: add a task to the implementation status table (section 2.2) to update or archive the document. Place it in the last implementation phase or in a dedicated "Documentation" phase.
4. If not invalidated: state why (e.g., "spec describes the interface contract, which is unchanged; only the internal implementation changes").

````markdown
### Documents reviewed

| Document | Invalidated? | Action |
|---|---|---|
| `../specs/2026-04-13_pull-over-on-shoulder_spec.md` | Yes -- section 3 "Reference spline" decision is superseded. | Task 5.6: update spec design decisions table. |
| `../arch-designs/behavior-planner_arch-design.md` | No -- arch-design describes the ManeuverDescriptor contract, which is unchanged. | None. |
| `../../planning/sampling/README.md` | Yes -- describes ego-lane-only reference spline assumption. | Task 5.7: update README sampling overview. |
````

### 2.9. Cleanup (when applicable)

Explicit list of diagnostic instrumentation (log statements, debug topics, temporary flags) added during the work that must be removed or downgraded after validation.

```markdown
| File | Line | Action |
|---|---|---|
| `request_handler.cc:142` | `LOG_INFO() << "request_id: " ...` | Remove |
| `config_loader.cc:87` | `LOG_INFO() << "routing_group: " ...` | Downgrade to DEBUG |
```

Cleanup is a deliverable, not an afterthought. The plan is not complete until diagnostic instrumentation is resolved. Include cleanup tasks in the implementation status table (section 2.2).

---

## 3. Conventions

- **Every change is tied to an exact file path and function name.** No "update the relevant module" -- name the file.
- **Validation is stated in terms of observable artifacts** (log lines, topic values, test output), not "it should work."
- **Cleanup is explicit.** Any instrumentation added during the work is tracked and resolved (section 2.9).
- **Inline code uses full signatures**, not pseudocode fragments. The reader should be able to grep the codebase and find the exact location.
- **The plan references its companion document** (spec, arch-design, brief, or issue). It never duplicates the companion's objective, non-goals, or investigation findings.

---

## 4. Lifecycle

### 4.1. Document Status

| Status | Meaning | What changes |
|---|---|---|
| `draft` | Planning phase. Structure may change. | Everything. |
| `approved` | Plan reviewed and accepted. Ready for issue creation. | Nothing in the plan content. Issue tracking handoff (section 4.2) begins. |
| `issued` | All tasks converted to issues in the issue tracker. | Nothing. Plan is frozen -- the issue tracker is now the execution authority. |
| `archived` | Feature shipped or fix merged. Code is now the truth. | Nothing. Do not update. If a future change invalidates the plan, write a new plan. |

### 4.2. Issue Tracking Handoff

When the plan transitions from `draft` to `approved`, convert it into trackable issues. The plan is a design artifact; the issue tracker is the execution artifact. They are not redundant -- the plan captures *why* and *how*; the issues track *what remains* and *who is doing it*.

**Procedure:**

1. **Create a master epic** for the entire plan. Title: the plan's title. Description: link to the plan file.
2. **Create one phase epic per phase** in the implementation status table (section 2.2). Title: the phase name. Link each phase epic as a child of the master epic (`parent-child`).
3. **Create one task issue per task** in the status table. Link each task as a child of its phase epic (`parent-child`).
4. **Wire `blocks` dependencies** between phase epics according to the dependency declarations in the phase summary (e.g., "depends on phase 1" becomes a `blocks` edge from phase 1's epic to phase 2's epic).
5. **Wire `blocks` dependencies between tasks** when the solution breakdown (section 2.5) declares explicit inter-task dependencies within a phase.
6. **Include staleness audit tasks** from section 2.8 as issues in the appropriate phase epic.
7. **Update the plan's status** to `issued`. The plan is now frozen.

After handoff, the issue tracker is the authority on what is done and what remains. Do not update the plan's status table -- it reflects the state at the time of handoff, not the current execution state.

### 4.3. Batch Issue Creation

When the issue tracker supports batch creation (e.g., `bd create --graph <plan.json>`), convert the plan's phases and tasks into a single graph file and create all issues in one operation. This avoids per-issue overhead and ensures the full dependency graph is visible from the start.

The graph file encodes:
- One node per epic (master + phases) and one node per task.
- `parent-child` edges wiring children to parents (child is `from_key`, parent is `to_key` -- the child depends on the parent). Phase epics are children of the master epic; tasks are children of their phase epic.
- `blocks` edges between phase epics (from the phase dependency declarations).
- `blocks` edges between tasks (from the solution breakdown dependencies).

### 4.4. Lifecycle Transitions

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

## 5. Anti-Patterns

| Anti-pattern | Problem | Fix |
|---|---|---|
| **Plan without companion document** | No agreement on *what* to build or *why*; the plan is implementing assumptions. | Write a spec, arch-design, or brief first. The plan references it. |
| **Plan as spec** | Mixes "what to build" with "how to build it." Objective, non-goals, and acceptance criteria are embedded in the plan. | Split into spec (what/why) and plan (how). |
| **Investigation in the plan** | Root cause analysis, diagnostic evidence, and hypothesis logs embedded in the plan instead of a brief. | Write a brief for the investigation. The plan references it via the `brief` metadata field. |
| **Pseudocode plan** | Solution breakdown uses invented syntax instead of real signatures and types. Implementer can't find the code. | Use real function names, real types, real file paths. |
| **Missing cleanup** | Debug logs and temporary instrumentation ship to production. | Add a cleanup section (section 2.9). Don't mark the work done until it's resolved. |
| **Tracking progress in the plan** | Plan status table diverges from issue tracker. Two sources of truth. | After `issued`, the issue tracker is authoritative. Do not update the plan's status table. |
| **Post-ship maintenance** | Updating the plan after the code has diverged. The plan becomes a second source of truth that contradicts the code. | Archive the plan. Write a new plan if a new change is needed. |
| **No validation criteria** | "Test that it works" instead of observable before/after conditions. | State what you'll observe, in what tool, with what values. Encode as Verify: tasks. |
| **Skipping issue handoff** | Plan approved but tasks never converted to issues. No dependency tracking, no parallelization visibility, no persistent memory across sessions. | Convert to issues immediately on approval (section 4.2). |
| **Missing staleness audit** | Plan changes interfaces described in existing docs. Docs go stale silently. | Always complete section 2.8 before approval. |

---

## Governing Standards

This definition conforms to the [Document Structure Standards](../standards/document-structure_standards.md). See section 2.1 for the file naming convention and date rules.
