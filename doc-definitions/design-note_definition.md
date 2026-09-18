# Design Note Definition

Reference for writing design note documents. A design note is the settled design for a scoped piece of work. It records the decisions, the conceptual rationale behind them, the shape of the interface and type changes, and the acceptance criteria that define done. It is written to be consumed directly by an implementation plan.

---

## 1. Purpose

A design note exists to:

1. **Decide** -- record the settled decisions for one piece of work, not a menu of options.
2. **Explain conceptually** -- the rationale is the heart of the note: why this approach, at the level of ideas, above the code.
3. **Shape the change** -- outline the interface and type changes each decision entails, structurally (not compilably).
4. **Commit** -- state the acceptance criteria that define done end-to-end.
5. **Feed a plan** -- give an implementer everything needed to sequence the work into an implementation plan.

A design note is NOT:

- A design study. A study decomposes a problem into interacting dimensions and evaluates alternatives systematically; its artifact is the reasoning framework. A note records self-contained decisions and their rationale. See [Design Study Definition](design-study_definition.md).
- An arch-design. An arch-design is the settled, living architecture of a whole component or subsystem. A note is scoped to one piece of work and is point-in-time. See [Architecture Design Definition](arch-design_definition.md).
- A brief. A brief frames work and records investigation findings; it makes no decisions and carries no acceptance criteria. A note is the commitment the brief precedes. See [Brief Definition](brief_definition.md).
- An implementation plan. A note says what and why; a plan sequences how. See [Implementation Plan Definition](impl-plan_definition.md).

---

## 2. When to Write One

Write a design note when:

- A piece of work has design decisions that need recording, with rationale, before implementation.
- The work is scoped -- one change, not a whole component's architecture.
- The decisions are self-contained: each has its own rationale, rather than interacting dimensions that need systematic comparison.

Do not write a design note when:

- The problem decomposes into interacting dimensions and the reasoning framework is itself worth preserving. Write a design study.
- The work is the settled architecture of a whole component or subsystem, maintained after shipping. Write an arch-design.
- There are no design decisions to make -- the work is framing or investigation. Write a brief.
- The scope is clear and you are ready to sequence implementation. Write a plan (referencing the note).

---

## 3. Structure

A design note has a fixed section order. Sections marked **(required)** must be present; sections marked **(when applicable)** are included when the work warrants them. A note is short -- up to a few pages. If a note grows into systematic alternatives analysis, it is becoming a design study; promote it.

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
| `status` | `draft` while the note is authored and revised; `accepted` once the decisions and acceptance criteria are settled. |
| `date` | Creation date in `YYYY-MM-DD` format. Must be the actual calendar date when the document is created. Must match the date prefix in the filename. Immutable after creation. |
| `author` | Who owns this note. Use `git config user.name`. |

**File naming:** `YYYY-MM-DD_<topic>_design-note.md`. The date is the creation date (immutable). The topic matches the title in kebab-case.

### 3.2. Problem (required)

One to three sentences: what is broken or what is being decided, scoped to this piece of work, and why it matters now. Lead with the problem, not the solution.

### 3.3. Context (when applicable)

Grounding for the decisions: the current state, code pointers, constraints, and anything a reader needs to evaluate the rationale without re-deriving it. Concrete over abstract -- name files, functions, types.

### 3.4. Decisions (required)

The core of the note. One entry per decision. Each entry has:

- **Decision** -- the settled choice, stated concretely.
- **Rationale** -- the conceptual explanation. This is the most important content: why this choice, at the level of ideas, above the code. What constraint, property, or fact makes it correct, and what would have to change for it to be revisited.
- **Alternatives** -- the rejected options, one line each with the deciding factor.

```markdown
### Decision: match note-off by note number, not frequency

**Decision:** Thread the MIDI note number through the allocator and match
note-off on it.

**Rationale:** Frequency is a derived rendering parameter, not identity. Using
it as the key breaks the moment tuning, transpose, or per-note pitch bend
exists -- a note-on and its note-off can then legitimately map to different
frequencies. The note number is the stable identity; frequency is computed
from it at note-on and carried for rendering.

**Alternatives:**

- Exact float match on frequency -- works today (deterministic), but
  structurally broken for tuning/MPE/microtuning; rejected for the future it
  forecloses.
```

Rationale is conceptual, not a code explanation. If a reader needs a full decomposition to understand why, the note has drifted into design-study territory.

### 3.5. Interface & Type Outline (required)

One short structural snippet per decision: the types, fields, signatures, and boundary changes the decision entails. Shape only -- field names, types, relationships -- not compilable code. The code is authoritative for syntax; the note is authoritative for shape and intent.

```markdown
### `Owner` (allocator.h)

struct Owner {
  bool active;
  uint8_t note;   // identity for note-off matching (was float freq)
  float freq;     // render parameter, computed at note-on
  uint8_t part;
  uint32_t serial;
};
```

### 3.6. Acceptance Criteria (required)

The end-to-end "done" contract: observable, checkable conditions that the work is complete. Write them as Given/When/Then or as checkable assertions. These are the note's commitment; the plan's Success Criteria are the finer-grained per-step verification that trace to them.

```markdown
- [ ] Given a note-off for a held note, the voice transitions to release and
      falls silent.
- [ ] Given two overlapping notes of the same pitch, each note-off releases
      exactly its own voice.
```

### 3.7. Approach (when applicable)

The order of work -- a short numbered list or paragraph. Not a plan: no task breakdown, no status table, no per-step verification. The plan sequences what the note decides.

### 3.8. Open Questions (when applicable)

Questions that must be answered before or during the work. Each states the question and what it blocks. An accepted note should have zero open questions -- resolve them and delete this section before marking `accepted`.

Open questions are for what cannot yet be verified, not for what has not yet been decided. A question may remain open only when the aspect cannot be verified at the time of writing (e.g., hardware not available, an external system not yet observable). Anything resolvable by design must be resolved before the note is written and folded into its regular structure -- open questions are not a backdoor for postponing design work.

---

## 4. Principles

### 4.1. Rationale is the Point

The decisions are only as good as their rationale. Optimize for the reader who needs to know *why*, not just *what* -- the conceptual reasoning is what survives contact with the code.

### 4.2. Scoped, Not Systemic

A note covers one piece of work. If the decisions add up to the architecture of a whole component, that is an arch-design. If they need a decomposition framework, that is a design study.

### 4.3. Frozen at Acceptance

A note is refined freely in `draft`; once `accepted`, it is frozen. A material change to a settled decision means a new note or promotion to an arch-design or design study -- not retrofitting this one.

### 4.4. Structural Code, Not Compilable Code

Interface outlines show shape and intent; the code is authoritative for syntax. A field rename does not stale the note; a removed type or a changed contract does.

---

## 5. Relationship to Other Document Types

| Document type | Relationship |
|---|---|
| **Brief** | A brief frames the work and records investigation; it may identify that a design note is needed. The note is the commitment the brief precedes. |
| **Design study** | A study reasons through interacting dimensions; its recommendation may be recorded as a design note. The note records the decision; the study records the analysis. |
| **Arch-design** | A note is scoped and point-in-time; an arch-design is whole-component and living. A note may propose a change that an arch-design later absorbs. |
| **Implementation plan** | A plan consumes the note as its companion: the note supplies the decisions, rationale, interface shape, and acceptance criteria; the plan sequences the work and verifies each step. |
| **Cookbook** | Practical findings along the way (tool quirks, workarounds) belong in a cookbook entry, not the note. |

---

## 6. Minimal Example

```markdown
---
title: Note-Off Matching by Note Identity
status: accepted
date: 2026-09-18
author: Jane Smith
---

# Note-Off Matching by Note Identity

## 1. Problem

Notes occasionally stick at sustain: a note-off reaches the allocator but
matches no held voice, so no release event is emitted and the voice keeps
sounding. The allocator matches note-off by exact float frequency.

## 2. Context

`engine/allocator.cc` `Allocator::NoteOff(part, freq_hz)` matches
`owner_[i].freq == freq_hz`. Callers compute `freq_hz` from
`engine::MidiNoteToFreq(note)` in `host/midi_io.cc` and `DrainMidi`
(`target/zephyr/cm33/src/main.cc`).

## 3. Decisions

### Decision: match note-off by note number, not frequency

**Decision:** Thread the MIDI note number through `PanelNoteOn/Off` →
`EngineControl::NoteOn/Off` → `Allocator::NoteOn/Off`; store it in `Owner`;
match note-off on `(part, note)`.

**Rationale:** Frequency is a derived rendering parameter, not identity.
Tuning, transpose, or per-note pitch bend breaks a frequency key outright --
the note-on and note-off can map to different frequencies. The note number is
the stable identity.

**Alternatives:**

- Exact float match on frequency -- deterministic today, but structurally
  broken for tuning/MPE/microtuning.

## 4. Interface & Type Outline

### `Owner` (allocator.h)

struct Owner {
  bool active;
  uint8_t note;   // identity for note-off matching
  float freq;     // render parameter, computed at note-on
  uint8_t part;
  uint32_t serial;
};

`NoteOff(int part, uint8_t note)` replaces `NoteOff(int part, float freq_hz)`.

## 5. Acceptance Criteria

- [ ] Given a note-off for a held note, the voice releases and falls silent.
- [ ] Given two overlapping notes of the same pitch, each note-off releases
      exactly its own voice.

## 6. Approach

1. Change the allocator owner/matching to use the note number.
2. Propagate the signature through engine control, panel, and the two MIDI
   transports.
3. Update the allocator/engine tests and the tools that call the old API.
```

---

## Governing Standards

This definition conforms to the [Document Structure Standards](../standards/document-structure_standards.md). See section 2.1 for the file naming convention and date rules.
