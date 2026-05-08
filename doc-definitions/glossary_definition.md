# Glossary Definition

Reference for writing glossary documents. A glossary is a single-file, living lexicon of project-specific vocabulary. It defines the terms, concepts, and names that have precise meaning within a codebase and cannot be looked up in a general reference. It exists so that contributors — human and agent — share the same vocabulary and understand the boundaries between related concepts.

---

## 1. Purpose

A glossary exists to:

1. **Disambiguate** — settle which word means what when multiple words are used for the same thing, or one word means different things to different people.
2. **Define** — give each term a tight, opinionated definition so that "slot" or "bead" or "folio" is not left to interpretation.
3. **Relate** — show how terms connect to each other, so that understanding one leads naturally to understanding its neighbors.
4. **Alert** — flag terms that have been ambiguous in the past, with the resolution that was chosen.

A glossary is NOT:

- A general programming dictionary. Terms like "middleware," "serialization," or "event loop" do not belong even if the project uses them extensively. Before adding a term, ask: is this concept unique to this project's domain, or would a developer already know it? Only the former belongs.
- A reference manual. If a term needs a page of explanation, code examples, configuration tables, or exhaustive coverage, it belongs in a dedicated reference document (see `_reference` and `_setup` definitions). The glossary entry should be a one-sentence pointer into that deeper document.
- A comment or design doc. The glossary does not argue for why a term was chosen or document alternatives that were considered. That belongs in a design study or arch-design.

---

## 2. When to Write One

Write a glossary when:

- The project has domain-specific terms that contributors keep misusing or asking about.
- Multiple names exist for the same concept and the team needs to settle on one.
- An agent or new contributor cannot be expected to infer the vocabulary from code alone.

Do not write a glossary when:

- The vocabulary is small enough that inline comments or a README suffice. A glossary is overhead for a codebase with fewer than ~5 project-specific terms.
- The terms are all standard programming concepts. A glossary that defines "singleton" and "idempotent" adds noise, not signal.

---

## 3. Structure

A glossary has a fixed section order. All sections are required unless marked otherwise. The document is living — sections grow over time as terms are added, relationships emerge, and ambiguities are resolved.

### 3.1. Document Header (required)

```markdown
# Glossary
```

The heading is always `# Glossary`. No subtitle, no description line. The file name and heading together are sufficient to identify the document.

**File naming:** `glossary.md`. No topic prefix, no date prefix, no doctype suffix. This is a singleton per `docs/` directory — there is exactly one glossary, and its fixed name signals that.

**Placement:** `docs/glossary.md`. At the docs root, not in a subdirectory. The glossary describes the vocabulary of its parent directory and all descendants:

```
<scope-root>/
  docs/
    glossary.md      ← defines vocabulary for <scope-root>/ and everything under it
    workflow/
    references/
  src/
  ...
```

The scope is determined by the `docs/` directory's parent. If `docs/` is at the repo root, the glossary covers the whole repo. If `docs/` is inside a component directory, it covers that component and its subtree.

### 3.2. Terms Section (required)

```markdown
## Terms

**Term Name**:
One sentence defining what the term IS. Start with the category or nature of the thing, then its distinguishing characteristic.
_Avoid_: alias1, alias2
```

Rules for terms:

- **Be opinionated.** When multiple words exist for the same concept, pick the best one as the canonical term and list the others under _Avoid_. The decision is recorded; the debate is closed.
- **One sentence max.** Define what the term IS, not what it does, how it works, or why it was chosen. If the definition spills into a second sentence, the concept likely belongs in a reference document instead.
- **Alphabetical order.** Terms are listed alphabetically. New terms are inserted in order, not appended at the end. This is a reference document, not a log.
- **Only project-specific terms.** General programming concepts (timeouts, error types, utility patterns) do not belong even if the project uses them. Before adding a term, confirm: would a competent developer unfamiliar with this project know this term? If yes, it does not belong.
- **Group under subheadings** when natural clusters emerge (e.g., `### Data Model`, `### Pipeline Stages`). A flat list is fine when all terms belong to a single cohesive domain.

### 3.3. Relationships (required)

```markdown
## Relationships
```

How the defined terms connect to each other. Each relationship uses bold term names and expresses cardinality where obvious:

```markdown
- A **Slot** contains one or more **Sources**
- A **Source** is resolved into a **Bead** at deployment time
- A **Folio** references the **Carrel** that produced it
```

Rules:

- **Use bold term names** so relationships are scannable independently of the terms section.
- **Express cardinality** where it is clear and stable: "one or more," "exactly one," "zero or more."
- **Only include relationships between glossary terms.** Do not relate glossary terms to general concepts or to terms that are not defined in the glossary.
- **A term without any relationship to other glossary terms** is fine — do not force a relationship. But if a term sits in isolation, ask whether it is too general to belong.

### 3.4. Example Dialogue (required)

```markdown
## Example Dialogue
```

A short conversation between a developer and a domain expert that demonstrates how the terms interact naturally and clarifies boundaries between related concepts:

```markdown
> **Dev:** "When a **Customer** places an **Order**, do we create the **Invoice** immediately?"
> **Domain expert:** "No — an **Invoice** is only generated once a **Fulfillment** is confirmed."
```

Rules:

- **Two to four exchanges.** Enough to show terms in use, not enough to become a tutorial.
- **Use bold for every glossary term** on first mention in the dialogue.
- **Test a boundary.** The dialogue should clarify a distinction that the definitions alone might leave ambiguous. If every question has an obvious answer from the definitions, the dialogue is not pulling its weight.
- **Keep it realistic.** Write the way people actually talk about the system, not the way documentation reads.

### 3.5. Flagged Ambiguities (required)

```markdown
## Flagged Ambiguities
```

Terms that have been used ambiguously in the past, with the resolution that was chosen:

```markdown
- "account" was used to mean both **Customer** and **User** — resolved 2026-04-15: these are distinct concepts. **Customer** is the billing entity; **User** is the login identity.
```

Rules:

- **Record the resolution,** not just the conflict. A flagged ambiguity without a resolution is a complaint, not a decision.
- **Include the resolution date.** This tells readers whether the resolution is recent (still settling) or established.
- **Remove entries when the ambiguity is no longer live.** If the old usage has been fully purged from code, comments, and documentation, the flag is noise. An ambiguity that no one remembers is not an ambiguity.
- **A term that appears here must also appear in the Terms section.** The flag references the canonical definition; it does not replace it.

---

## 4. Principles

### 4.1. Opinionated and Decisive

A glossary that says "this term is sometimes called X, sometimes called Y" has failed. Pick one. List the others under _Avoid_. The glossary is where the vocabulary debate ends.

### 4.2. Tight Definitions

One sentence. Define what the thing IS. "A `Slot` is a named configuration target in the carrel deployment model" — not "A `Slot` handles the deployment of configuration to target workspaces and is configured via YAML files in the `.omp/` directory..."

If a term genuinely needs more than one sentence to define, it may be too broad for a glossary. Split it or move it to a reference document and point to it from the glossary.

### 4.3. Living but Disciplined

The glossary is updated as vocabulary evolves — it is not a point-in-time artifact. But every addition or change must meet the same bar: is this a project-specific term that contributors are likely to misunderstand? Vacuuming in every noun phrase from the codebase dilutes the glossary into uselessness.

### 4.4. Terms First, Relationships Second

Define terms before relating them. A relationship that references an undefined term is a broken link. When adding a new term, check whether it creates new relationships with existing terms and update the Relationships section in the same change.

### 4.5. Scope Discipline

A glossary at `docs/glossary.md` covers the parent of `docs/` and all its descendants. A term that describes something outside that scope — a concept from a sibling component, an upstream service, a different repository — does not belong. If contributors in this scope regularly need to reference external concepts, document them in a reference doc, not the glossary.

---

## 5. Anti-Patterns

| Anti-pattern | Problem | Fix |
|---|---|---|
| **Glossary as general dictionary** | Defines "middleware," "singleton," "event loop." | Remove every term a competent developer would already know. Restrict to project-specific concepts. |
| **Multi-sentence definitions** | Definition becomes a mini-essay. Readers can't scan it. | Cut to one sentence. Move supporting detail to a reference doc; link to it. |
| **Missing _Avoid_ aliases** | Multiple names for the same concept coexist in code and conversation. | Pick the canonical term. List every known alias under _Avoid_. |
| **Stale terms** | Term was removed from the codebase but remains in the glossary. | Remove the term. The glossary is living — it reflects the current vocabulary, not the historical one. (Flagged ambiguities about removed terms stay; they document decisions.) |
| **Relationships without the terms** | Relationships reference terms not defined in the glossary. | Either define the term or remove the relationship. |
| **Dialogue that restates definitions** | Every question has an obvious answer from the Terms section. The dialogue adds no new understanding. | Rewrite the dialogue to test a boundary the definitions leave ambiguous. |
| **Terms without boundaries** | Every term is defined in isolation; no relationships, no dialogue, no ambiguities flagged. | Add relationships. If no terms relate to each other, the glossary may be too broad or too sparse. |

---

## 6. Minimal Example

```markdown
# Glossary

## Terms

**Carrel**:
The deployment engine that resolves sources and provisions configuration into target workspaces.
_Avoid_: config-manager, provisioner

**Folio**:
The documentation framework that defines formats, standards, and workflow artifacts.
_Avoid_: doc-system, doc-framework

**Slot**:
A named configuration target within a carrel deployment, backed by one or more resolved sources.
_Avoid_: target, destination

**Source**:
A versioned unit of configuration (skill, rule, command, extension) registered in the carrel registry.

## Relationships

- A **Carrel** deploys configuration into **Slots**
- A **Slot** is backed by one or more **Sources**
- A **Source** is registered in the **Carrel** registry
- A **Carrel** may produce artifacts consumed by a **Folio**

## Example Dialogue

> **Dev:** "Is a **Slot** the same thing as a **Source**?"
> **Domain expert:** "No. A **Source** is the configuration artifact — a skill file, a rule, a command. A **Slot** is the target where those get deployed. One **Slot** can pull from multiple **Sources**."
> **Dev:** "And the **Carrel** is what wires them together?"
> **Domain expert:** "Right. The **Carrel** resolves **Sources** from the registry and provisions them into **Slots** at deploy time."

## Flagged Ambiguities

- "target" was used to mean both **Slot** and the output directory — resolved 2026-05-01: **Slot** is the named configuration target; the filesystem path is the "output path" and is not a glossary term.
```

---

## Governing Standards

This definition conforms to the [Document Structure Standards](../standards/document-structure_standards.md). The produced file `glossary.md` is exempt from the `<topic>_<doctype>.md` naming convention — it is a singleton with a fixed name, parallel to the README exception described in section 2.4 of the standards.
