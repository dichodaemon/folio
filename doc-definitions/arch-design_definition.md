# Architecture Design Definition

Definition for writing architecture design documents. An arch-design is the settled internal architecture of a component, library, or subsystem. It records the types, function decomposition, contracts, and structural decisions that an implementer needs to build the system and a maintainer needs to understand it. Its value is in the settled result -- the design as decided -- not in the reasoning that produced it.

---

## 1. Purpose

An arch-design exists to:

1. **Specify** -- define the internal architecture: types, function signatures, contracts, decomposition, and structural relationships.
2. **Constrain** -- state what is in scope, what is out, and what done looks like.
3. **Survive** -- remain accurate and useful after implementation, as the living architectural reference for the component.
4. **Orient** -- give an implementer (human or agent) everything needed to build the system without a preamble conversation.

An arch-design is NOT:

- A design study. A design study explores alternatives and converges on a recommendation. An arch-design records the result of that convergence -- the settled design, without alternatives. See [Design Study Definition](design-study_definition.md).
- A spec. A spec defines observable behavior and external contracts. An arch-design defines internal architecture: types, decomposition, and function contracts. For library and subsystem work where the internal architecture IS the interface, the arch-design subsumes the spec. For features where only the external contract matters, use a spec instead. See [Specification Document Definition](spec-doc_definition.md).
- An implementation plan. A plan sequences work into tasks with status tracking. An arch-design defines what to build; a plan defines how to build it in what order. See [Implementation Plan Definition](impl-plan_definition.md).
- A code dump. Code in the document is structural -- it shows the shape, fields, relationships, and behavioral contracts of the design. It is not required to be compilable. The implementation is authoritative for exact syntax; the arch-design is authoritative for architectural intent.

---

## 2. Structure

Every arch-design follows this skeleton. Sections marked **(required)** must always be present. Sections marked **(when applicable)** are included when the component warrants them.

### 2.1. Metadata (required)

```markdown
---
title: <Component Name>
status: draft | review | approved | implemented | stale | superseded
date: YYYY-MM-DD
author: <git user name>
design-study: <path-to-companion-design-study> (optional)
---
```

| Field | Purpose |
|---|---|
| `title` | Short, descriptive name. Matches the `#` heading. |
| `status` | Lifecycle stage (see section 4). |
| `date` | Creation date in `YYYY-MM-DD` format. Must be the actual calendar date when the document is created, not copied from another document. Must match the date prefix in the filename. Immutable after creation. |
| `author` | Who owns this document. Use `git config user.name`. |
| `design-study` | Optional. Relative path to the companion design study that produced this architecture. Present when a study exists, absent otherwise. |

**File naming:** `YYYY-MM-DD_<component-name>_arch-design.md`. E.g., `2026-05-01_behavior-planner_arch-design.md`.

### 2.2. Objective (required)

One to three sentences answering:

- **What** component or subsystem is being designed?
- **Why** -- what problem does it solve, what capability does it enable?
- **For whom** -- who uses this component (callers, consumers, downstream systems)?

Lead with the problem, not the solution.

### 2.3. Non-Goals (required)

Explicit list of things this arch-design does NOT address. Each item should be something a reasonable reader might expect to be in scope.

Non-goals prevent scope creep during implementation, set expectations for reviewers, and signal to agents what NOT to build.

If there are genuinely no non-goals, state "None identified" rather than omitting the section.

### 2.4. System Context (required)

Where this component sits in the broader system. Covers:

- Which layers or subsystems it interacts with.
- Key boundaries and interface types between layers.
- Internal subsystems and their responsibilities.

This section is prose with optional Mermaid diagrams. It answers "where does this live and what does it talk to?" before the reader dives into type definitions.

### 2.5. Design Decisions (when applicable)

A summary table of key architectural decisions with rationale:

```markdown
| Decision | Choice | Rationale |
|---|---|---|
| Coordinator ownership | Component state | Behavior planner is pure library; must not own GPU resources |
| Selection input | Pre-aggregated ManeuverResult | Decouples selection from bucket-level pipeline details |
```

Each row captures the winning choice and the key reason it won. Do NOT reproduce the full alternatives analysis -- that belongs in the companion design study. If no design study exists, the rationale cells should be self-contained.

### 2.6. Architecture (required)

The structural decomposition of the component. This is the heart of the document.

**Content:**

- Subsystem or module decomposition -- what the parts are and how they relate.
- Data flow -- what goes in, what comes out, what is stored, what flows between subsystems.
- Diagrams (Mermaid) for lifecycle, initialization sequences, per-cycle flow, or other temporal structure.

**What belongs here:**

- The shape of the system: which modules exist, which depend on which, how data flows between them.
- Lifecycle descriptions: initialization sequence, per-cycle flow, shutdown, error recovery.
- Diagrams that clarify structure or temporal ordering.

**What does NOT belong here:**

- Individual type definitions (those go in section 2.7).
- Individual function contracts (those go in section 2.8).
- Alternatives or tradeoffs (those belong in the design study).

### 2.7. Types (required)

Every struct, enum, variant, and type alias that constitutes the component's architecture. One subsection per type or per logical group of related types.

Each type definition includes:

- The structural definition (fields, variants, defaults) in the implementation language.
- Inline annotations explaining non-obvious fields or constraints.

**Structural, not compilable.** The definitions show the shape -- field names, types, relationships, invariants. The implementation may add fields, reorder, adjust qualifiers, or rename. The arch-design is authoritative for the architecture's structure and intent; the code is authoritative for exact syntax.

### 2.8. Contracts (required)

Function signatures with behavioral contracts. One subsection per public function, callback, or interface method.

Each contract includes:

- The function signature.
- **Preconditions** -- what must be true before the call.
- **Postconditions** -- what is guaranteed after the call returns.
- **Error semantics** -- what the function returns or does on failure.

Contracts define the promises the architecture makes to its callers and between its subsystems. They are the verifiable commitments of the design.

### 2.9. Lifecycle (when applicable)

How the component behaves over time. Include when the component has non-trivial initialization, per-cycle or per-request flow, state management, or shutdown behavior.

**Content:**

- Initialization sequence (what is created, in what order, what depends on what).
- Per-cycle or per-request flow (the main operational loop).
- State transitions and their triggers.
- Shutdown or cleanup behavior.

Mermaid sequence diagrams for initialization and flowcharts for operational cycles are encouraged.

### 2.10. Acceptance Criteria (required)

A list of observable, testable conditions that define "done." Each criterion should be verifiable without reading the source code.

**Guidelines:**

- Write criteria as Given/When/Then or as checkable assertions.
- Each criterion maps to at least one test (unit, integration, or manual verification).
- Do not include implementation-level criteria ("function X is called with parameter Y"). Test observable outcomes.
- Include error cases and edge cases, not just the happy path.

### 2.11. Code Pointers (required)

File layout and directory structure for the component. A table of files that will be created or modified, with their purpose.

```markdown
| File | Purpose |
|---|---|
| `fallback/behavior/plan_behavior.h` | PlanBehavior function signature |
| `fallback/behavior/lane_topology.h` | LaneIndex, LaneInfo types |
| `fallback/behavior/maneuvers/lane_follow.h` | MakeLaneFollowManeuver factory |
```

This section serves two audiences: human reviewers who need to understand the blast radius, and agents who need to know where to look when implementing.

### 2.12. Open Questions

Open questions may exist while the document is in `draft` status. Each question should state:

- The question itself.
- Why it matters (what decision it blocks or what type/contract it affects).
- Candidate answers, if any.

**This section must not exist when the document reaches `review` or any later status.** When a question is resolved, incorporate the answer into the relevant section (Types, Contracts, Architecture, Design Decisions) and delete the question. When all questions are resolved, delete this section entirely. A document in `review` with an Open Questions section is a contradiction.

---

## 3. Principles

### 3.1. Architecture Over Implementation

The arch-design records the settled architecture: the types, decomposition, and contracts that define the system's structure. It does not record implementation details that don't affect the architecture (loop structure, variable names, optimization choices). If it compiles, it's probably too detailed. If it doesn't communicate the shape of the system, it's too abstract.

### 3.2. Settled, Not Explored

An arch-design states decisions. It does not explore alternatives, weigh tradeoffs, or preserve rejected options. That work belongs in the companion design study. Every statement in the arch-design should be a commitment, not a recommendation.

If you find yourself writing "Option A vs Option B," you need a design study, not an arch-design.

### 3.3. Structural Code

Code in the Types and Contracts sections shows the shape of the design: field names, types, relationships, signatures, error returns. It is structural, not compilable. The implementation may diverge in syntax while preserving the architecture.

This means: a field reordering, a `const` qualifier change, or a renamed parameter does not make the document stale. A new callback, a removed type, or a changed error path does.

### 3.4. Living Document

The arch-design is updated when the implementation reveals that the architecture needs to change. It remains accurate after shipping and serves as the architectural reference for future maintainers. Updates should be rare and deliberate -- if the architecture changes fundamentally, that's a new design study, not an update to this document.

### 3.5. Self-Contained for Implementation

An implementer should be able to build the component from the arch-design and the plan alone. The design study provides rationale for reviewers and future maintainers but is not required for implementation. If the implementer needs to read the design study to understand what to build, the arch-design is incomplete.

### 3.6. Acceptance Criteria Are the Contract

The acceptance criteria are the arch-design's binding commitment. If the implementation satisfies all acceptance criteria, the arch-design is fulfilled. If a behavior matters, it must appear in the acceptance criteria.

---

## 4. Lifecycle

| Status | Meaning | What changes |
|---|---|---|
| `draft` | Actively working through the design. Types and contracts may change. Open questions may exist. | Everything. |
| `review` | Architecture is complete. No open questions. Ready for feedback. | Types and contracts may be refined based on feedback. Structure is stable. |
| `approved` | Committed. Implementation may proceed. | Author (with reviewer agreement for material changes to types, contracts, or acceptance criteria). |
| `implemented` | Code matches the document. Arch-design is the living reference. | Anyone (corrections, clarifications, and architecture updates that reflect code changes). |
| `stale` | Code has diverged from the document. Types, contracts, or structural decisions no longer match the implementation. | Must be updated to `implemented` or `superseded`. A stale arch-design is a call to action. |
| `superseded` | Replaced by a newer arch-design. Link to the replacement in the metadata. | Nothing. |

**Key transitions:**

- `draft` to `review`: all open questions resolved, Open Questions section deleted.
- `review` to `approved`: reviewer sign-off. Material changes after approval require re-review.
- `approved` to `implemented`: code matches the architecture. Acceptance criteria pass.
- `implemented` to `stale`: detected divergence between code and document (manual or automated).
- `stale` to `implemented`: document updated to match code.
- Any status to `superseded`: a new arch-design replaces this one.

---

## 5. Anti-Patterns

| Anti-pattern | Problem | Fix |
|---|---|---|
| **Arch-design as design study** | Explores alternatives, weighs tradeoffs, preserves rejected options. | Move the analysis to a design study. The arch-design records only the settled result. |
| **Compilable code** | Types section contains copy-pasteable code that must exactly match the implementation. | Code is structural. Show the shape, not the syntax. The implementation is authoritative for exact code. |
| **Arch-design as plan** | Includes task breakdowns, sequencing, or status tracking. | Split into arch-design (what to build) and plan (how to build it). |
| **Missing contracts** | Types are defined but function signatures and behavioral promises are absent. | The contracts section is required. Every public function and callback needs preconditions, postconditions, and error semantics. |
| **Stale document kept at `implemented`** | Code diverged but status was never updated. Readers trust a document that no longer reflects reality. | Transition to `stale`. Update or supersede. |
| **Open questions in review** | Document is marked `review` but still has unresolved questions. | Resolve all questions and delete the section before advancing to `review`. |
| **Abandoned arch-design** | Document stays in `draft` indefinitely, never reaching `review`. | The acceptance criteria section forces accountability. If the architecture can't be settled, the design study isn't done. |

---

## 6. Relationship to Other Document Types

| Document type | Relationship |
|---|---|
| **Design study** | An arch-design may be produced from a design study. The study records the reasoning; the arch-design records the settled result. The `design-study` metadata field links them. When no study exists, the Design Decisions section provides inline rationale. |
| **Spec** | For library and subsystem work where the internal architecture IS the interface, the arch-design subsumes the spec. For features where only the external contract matters (CLI tools, API endpoints, configuration changes), use a spec instead. Both document types can coexist for complex features with distinct external and internal concerns. |
| **Implementation plan** | A plan implements an arch-design, a spec, or both. The plan references the arch-design for types, contracts, and decomposition. The plan sequences the work; the arch-design defines what the work produces. |
| **Brief** | A brief may identify that an arch-design is needed. The brief frames the work; the arch-design (possibly after a design study) settles the architecture. |
| **Cookbook** | If an arch-design surfaces practical implementation findings (tool quirks, workarounds, patterns), those belong in a cookbook entry, not in the arch-design. |

---

## 7. Minimal Example

````markdown
---
title: Rate Limiter
status: approved
date: 2026-05-01
author: Jane Smith
---

# Rate Limiter

## 1. Objective

The API gateway has no request throttling. A misbehaving client can saturate backend services, degrading availability for all clients. This arch-design defines the internal architecture of a token-bucket rate limiter that the HTTP middleware calls on every request.

## 2. Non-Goals

- Distributed rate limiting across multiple gateway instances. This design is per-instance.
- Dynamic rate limit configuration (hot reload). Limits are set at startup.
- Per-endpoint rate limits. All endpoints share one bucket per client.

## 3. System Context

The rate limiter is a library consumed by the HTTP middleware layer. It sits between request parsing (upstream) and handler dispatch (downstream).

- **Middleware to rate limiter**: `RateLimitDecision CheckRate(ClientID)`.
- **Rate limiter to storage**: internal `TokenBucket` map, no external dependency.

## 4. Architecture

Two components:

- **TokenBucket** -- per-client token bucket with monotonic-clock refill. Pure value type.
- **RateLimiter** -- owns a map of `ClientID` to `TokenBucket`. Thread-safe. Exposes `CheckRate`.

Per-request flow: middleware extracts `ClientID` from the request header, calls `CheckRate`, and either forwards the request or returns HTTP 429.

## 5. Types

### TokenBucket

```cpp
struct TokenBucket {
  double capacity;         // Maximum tokens
  double refill_rate;      // Tokens per second
  double current_tokens;   // Available tokens
  int64_t last_refill_ns;  // Monotonic clock timestamp
};
```

Why monotonic clock: wall-clock adjustments (NTP, DST) would cause spurious token grants or starvation.

### RateLimitDecision

```cpp
enum class RateLimitDecision { kAllow, kReject };
```

## 6. Contracts

### CheckRate

```cpp
RateLimitDecision CheckRate(ClientID client);
```

- **Precondition**: `client` is a valid, non-empty identifier.
- **Postcondition on kAllow**: one token consumed from the client's bucket. Request may proceed.
- **Postcondition on kReject**: no token consumed. Caller should return HTTP 429.
- **Thread safety**: safe to call concurrently for different clients. Serialized per client.

## 7. Acceptance Criteria

- [ ] Given a client with available tokens, `CheckRate` returns `kAllow` and decrements the token count.
- [ ] Given a client with zero tokens, `CheckRate` returns `kReject` without modifying the bucket.
- [ ] Given a client whose bucket has been idle for `capacity / refill_rate` seconds, the bucket is full regardless of previous consumption.
- [ ] Given two concurrent requests for different clients, neither blocks the other.
- [ ] Given no prior requests from a client, `CheckRate` creates a bucket with `capacity` tokens.

## 8. Code Pointers

| File | Purpose |
|---|---|
| `src/rate_limiter/token_bucket.h` | TokenBucket struct |
| `src/rate_limiter/rate_limiter.h` | RateLimiter class, CheckRate |
| `src/middleware/rate_limit_handler.cc` | Middleware integration |
````

---

## Governing Standards

This definition conforms to the [Document Structure Standards](../standards/document-structure_standards.md). See section 2.1 for the file naming convention and date rules.
