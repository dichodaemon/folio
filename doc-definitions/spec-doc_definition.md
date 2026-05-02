# Specification Document Definition

Reference for writing specification documents. A spec is a commitment document. It answers: what are we building, for whom, and what does done look like? It should survive the implementation and remain useful after the feature ships -- as context for future changes, as input for agents generating related code, and as a record of what was decided and why.

---

## 1. Purpose

A spec exists to:

1. **Align** -- ensure everyone (humans and agents) agrees on what is being built before code is written.
2. **Constrain** -- define what is in scope, what is out, and what done looks like.
3. **Record** -- capture the reasoning behind decisions so they can be revisited without re-deriving.
4. **Survive** -- remain accurate and useful after implementation, as living documentation for the feature.

A spec is NOT:

- An implementation plan (deprecated). Task sequencing and tracking are now handled by beads issue tracking. A spec covers *what* to build and *why*; beads issues cover *how* and *in what order*. See [Implementation Plan Reference](impl-plan_definition.md).
- A design study. If alternatives were evaluated, the spec records the winner and the rationale, not the full analysis. The analysis belongs in a design study. See [Design Study Definition](design-study_definition.md).
- An arch-design. An arch-design defines internal architecture: types, decomposition, and function contracts. For library and subsystem work where the internal architecture IS the interface, the arch-design subsumes the spec. For features where only the external contract matters, use a spec. See [Architecture Design Definition](arch-design_definition.md).
- A ticket or task tracker. It doesn't track who is doing what or when.

---

## 2. Structure

Every spec follows this skeleton. Sections marked **(required)** must always be present. Sections marked **(when applicable)** are included when the feature warrants them.

### 2.1. Metadata (required)

```markdown
---
title: <Feature Name>
status: draft | review | approved | implemented | superseded
date: YYYY-MM-DD
author: <git user name>
---
```

| Field | Purpose |
|---|---|
| `title` | Short, descriptive name. Matches the `#` heading. |
| `status` | Lifecycle stage. `draft` while writing, `review` when ready for feedback, `approved` when committed, `implemented` after shipping, `superseded` if replaced by a later spec. |
| `date` | Creation date in `YYYY-MM-DD` format. Must be the actual calendar date when the document is created, not copied from another document. Must match the date prefix in the filename. Immutable after creation. |
| `author` | Who owns this spec. Use `git config user.name`. |

**File naming:** `YYYY-MM-DD_<feature-name>_spec.md`, using lowercase kebab-case with a creation date prefix. The date is the creation date (not last-modified). The name should match the title slug -- e.g., a spec titled "VRU to Stopline" created on 2026-04-07 is named `2026-04-07_vru-to-stopline_spec.md`. The `_spec` suffix distinguishes it from the corresponding plan (`_plan.md`) and other document types.

### 2.2. Objective (required)

One to three sentences answering:

- **What** is being built or changed?
- **Why** -- what problem does it solve, what failure does it prevent, what capability does it enable?
- **For whom** -- who benefits?

This section is the single most important part of the spec. If a reader only reads this, they should understand the intent.

**Anti-patterns:**

- Starting with the solution instead of the problem.
- Describing the implementation approach here (that belongs in section 2.5).
- Being vague: "improve the system" tells the reader nothing.

### 2.3. Non-Goals (required)

Explicit list of things this spec does NOT address. Each item should be something a reasonable reader might expect to be in scope.

Non-goals prevent scope creep during implementation and set expectations for reviewers. They also signal to agents what NOT to build.

If there are genuinely no non-goals, state "None identified" rather than omitting the section -- the absence of the section is ambiguous; its presence with "None" is a deliberate statement.

### 2.4. Design Decisions (when applicable)

A table of key decisions made during specification, with rationale:

```markdown
| Decision | Choice | Rationale |
|---|---|---|
| Storage backend | SQLite | Single-file, no server dependency, sufficient for expected scale |
| API style | REST | Consumers are web clients; gRPC adds complexity with no benefit here |
```

Use this when the spec involves choices that aren't obvious from the solution description alone. Skip it for specs where the approach is straightforward and the objective implies the design.

If alternatives were evaluated, record the winning choice and the key reason it won. Do NOT reproduce the full alternatives analysis -- that belongs in a companion design study (see [Design Study Definition](design-study_definition.md)) or in the decision's rationale cell.

### 2.5. Solution (required)

The technical description of what will be built. This is the heart of the spec.

**Structure depends on complexity:**

- **Simple features:** A numbered list of behavior points or a prose description.
- **Multi-component features:** Subsections per component, each describing its responsibility, interfaces, and behavior.
- **API-heavy features:** Parameter tables, endpoint descriptions, data flow diagrams.

**What belongs here:**

- Observable behavior: what the user or system sees.
- Interfaces: APIs, CLI arguments, configuration options, file formats.
- Data flow: what goes in, what comes out, what is stored.
- Error handling: what happens when things fail.

**What does NOT belong here:**

- Implementation details that don't affect behavior (internal data structures, algorithm choice, code organization). Those belong in the arch-design or in beads issue descriptions.
- Code snippets longer than ~10 lines. If you need code to explain the design, the design may be too coupled to a specific implementation.

### 2.6. Acceptance Criteria (required)

A list of observable, testable conditions that define "done." Each criterion should be verifiable without reading the source code.

```markdown
- [ ] Given a pane log file at `/tmp/pane-1234.log`, the `read_pane` tool returns the last 100 lines.
- [ ] Given no matching pane, the tool returns an error listing available panes.
- [ ] The `/panes` command lists all discovered panes with PID, label, size, and last-modified time.
```

**Guidelines:**

- Write criteria as Given/When/Then or as checkable assertions.
- Each criterion maps to at least one test (unit, integration, or manual verification).
- Do not include implementation-level criteria ("function X is called with parameter Y"). Test observable outcomes.
- Include error cases and edge cases, not just the happy path.

### 2.7. Code Pointers (when applicable)

A labeled list of relevant source files, modules, or components that the implementation will touch or depend on.

```markdown
| File / Module | Relevance |
|---|---|
| `src/planning/lane_utils.h` | Lane geometry queries used by the new predicate |
| `src/planning/stop_line_tracker.cc` | Existing tracker to be extended |
| `src/msgs/planning.proto` | Message definition for stop line events |
```

This section serves two audiences: human reviewers who need to understand the blast radius, and agents who need to know where to look when implementing.

### 2.8. Open Questions (when applicable)

Numbered list of unresolved questions that must be answered before or during implementation.

Each question should state:

- The question itself.
- Why it matters (what decision it blocks or what behavior it affects).
- Candidate answers, if any.

Open questions are not a parking lot for vague concerns. Each one should be specific enough that someone could answer it in a sentence or a short investigation.

When a question is resolved, move the answer to the relevant section (Design Decisions, Solution, or Acceptance Criteria) and remove it from this list. An approved spec should have zero open questions.

### 2.9. References (when applicable)

Links to prior art, related specs, external documentation, or design studies that informed this spec.

---

## 3. Principles

### 3.1. Write for Three Readers

Every spec has three audiences:

1. **The human reviewer** who approves the design. They need the objective, non-goals, and acceptance criteria to evaluate whether the spec solves the right problem.
2. **The implementing agent** who generates code from the spec. It needs the solution, code pointers, and acceptance criteria to know what to build and how to verify it.
3. **The future maintainer** (human or agent) who changes the feature later. They need the objective, design decisions, and non-goals to understand why things are the way they are.

Write every section with all three in mind. If a section only serves one audience, it's probably in the wrong document.

### 3.2. Objective Before Solution

The most common spec failure is jumping to the solution before establishing the problem. A spec that starts with "add a gRPC endpoint that..." has already committed to an approach before the reader understands why.

Always write the objective first. Always write it in terms of the problem, not the solution.

### 3.3. Non-Goals Are Load-Bearing

Non-goals are not filler. They are decisions -- each one says "we considered this and chose not to do it." During implementation, non-goals prevent scope creep. After implementation, they explain why the feature doesn't do something a reader might expect.

### 3.4. Acceptance Criteria Are the Contract

The acceptance criteria are the spec's binding commitment. Everything else is context and rationale. If the implementation satisfies all acceptance criteria, the spec is fulfilled -- even if the solution section described a different approach.

This means: if a behavior matters, it must appear in the acceptance criteria. If it's only in the solution section, it's a suggestion, not a commitment.

### 3.5. Specs Are Living Documents

A spec's `status` changes over its lifecycle:

| Status | Meaning | Who can edit |
|---|---|---|
| `draft` | Work in progress. Incomplete, may change freely. | Author |
| `review` | Ready for feedback. Structure is stable, details may change. | Author + reviewers |
| `approved` | Committed. Implementation may proceed. | Author (with reviewer agreement for material changes) |
| `implemented` | Feature shipped. Spec is now reference documentation. | Anyone (corrections and clarifications only) |
| `superseded` | Replaced by a newer spec. Link to the replacement. | Anyone |

After `approved`, changes to acceptance criteria or non-goals require re-review. Changes to the solution section are allowed if acceptance criteria are unchanged.

### 3.6. One Spec Per Feature

A spec covers one feature, one change, or one coherent set of related changes. If you need "Part 1" and "Part 2," either the feature is too large (split it) or the spec is trying to be an implementation plan (separate the documents).

---

## 4. Anti-Patterns

| Anti-pattern | Problem | Fix |
|---|---|---|
| **Solution-first spec** | Jumps to implementation without establishing the problem. | Write the objective and non-goals before touching the solution section. |
| **Spec as implementation plan** | Includes task breakdowns, file-by-file change lists, or code organization decisions. | Split into spec (what) and beads issues (how). |
| **Acceptance-free spec** | No testable criteria for "done." | Add acceptance criteria before moving to `review`. |
| **Kitchen-sink non-goals** | Lists things nobody would expect to be in scope ("This spec does not address climate change"). | Each non-goal should be something a reasonable reader might assume is in scope. |
| **Abandoned open questions** | Questions listed but never resolved, even in `implemented` status. | Resolve before `approved`. If truly unanswerable, convert to a non-goal or a known limitation in the solution. |
| **Spec-as-journal** | Records the exploration process (alternatives tried, dead ends) instead of the conclusion. | Keep only the winning approach and its rationale. Move the analysis to a design study if the reasoning has reference value. |
| **Stale spec** | Implementation diverged but spec was never updated. | Update the spec when implementation deviates materially. An inaccurate spec is worse than no spec. |


---

## 5. Minimal Example

This example represents the floor -- a simple feature with a single component. Complex features will have longer solution sections, more acceptance criteria, and possibly design decisions and code pointers. The structure and tone are what matters.

````markdown
---
title: Build Log Notifier
status: approved
date: 2026-04-07
author: Jane Smith
---

# Build Log Notifier

## 1. Objective

Bazel builds in the dev container produce no notification when they finish. When a build takes more than a few minutes, the developer context-switches and misses the completion, wasting time. This spec defines a lightweight notifier that alerts the developer when a build finishes, with pass/fail status.

## 2. Non-Goals

- Parsing build output beyond exit code and elapsed time. Detailed error extraction is a separate concern.
- Supporting build systems other than Bazel.
- Providing a persistent build history or dashboard.

## 3. Design Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Notification mechanism | Desktop notification via `notify-send` | Already available in the dev container. No additional dependencies. Falls back to terminal bell if `notify-send` is absent. |
| Integration point | Shell wrapper around `bazel` | Non-invasive. Does not modify Bazel configuration or require a daemon. |

## 4. Solution

A shell function `bz` wraps `bazel` commands. It:

1. Records the start time.
2. Runs `bazel "$@"`, capturing the exit code.
3. Computes elapsed time.
4. Sends a desktop notification with:
   - Title: `Build passed` or `Build failed`
   - Body: the bazel command and elapsed time (e.g., `bazel build //my_project:all -- 3m42s`)
5. If `notify-send` is not available, emits a terminal bell (`\a`) and prints the summary to stderr.
6. Returns the original exit code so it can be used in scripts and pipelines.

The function is defined in `~/.bashrc.d/bz.sh` and available in all shell sessions inside the dev container.

## 5. Acceptance Criteria

- [ ] `bz build //my_project:all` on a successful build sends a notification with title "Build passed" and body containing the command and elapsed time.
- [ ] `bz build //invalid:target` on a failed build sends a notification with title "Build failed".
- [ ] `bz test //common:utils_test` works for test commands, not just build.
- [ ] The exit code of `bz` matches the exit code of the underlying `bazel` command.
- [ ] When `notify-send` is absent, a terminal bell is emitted and the summary is printed to stderr.
- [ ] Elapsed time is formatted as `Xm Ys` (e.g., `3m 42s`), not raw seconds.

## 6. References

- `notify-send` man page: https://man.archlinux.org/man/notify-send.1
````

---

## Governing Standards

This definition conforms to the [Document Structure Standards](../standards/document-structure_standards.md). See section 2.1 for the file naming convention and date rules.