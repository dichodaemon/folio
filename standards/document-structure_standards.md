# Document Structure Standards

Standards for organizing documents within a repository's `docs/` tree. Covers directory structure, file naming conventions, and placement rules. Applies to both shallow repos and deep monorepos.

For the analysis and rationale behind these decisions, see the companion design study: [Document Structure Design Study](../../../docs/workflow/design-studies/2026-04-09_document-structure_design-study.md).

---

## 1. Directory Structure

```
docs/
  doc-definitions/             # Defines formats for document types
  standards/                    # Conventions and standards for practices
  references/                  # Records facts, configurations, practical knowledge
  cookbooks/                   # Domain-scoped append-only recipe collections
  workflow/                    # Briefs, specs, plans, design studies, and arch-designs
    briefs/
    specs/
    plans/
    design-studies/
    arch-designs/
  archive/                     # Retired documents (preserving relative path)
```

### 1.1. Directories by Role

Each top-level directory corresponds to a document role. Role is the primary organizing axis.

| Directory | Role | What goes here | What does NOT go here |
|---|---|---|---|
| `doc-definitions/` | Doc-definitions | Documents that define formats and structural requirements for document types. Normative over document structure. | Documents that define conventions for practices or domains. |
| `standards/` | Standards | Documents that define conventions, standards, and best practices for practices or domains. Normative over how we work. | Document format definitions (those go in `doc-definitions/`). |
| `references/` | Reference | Reference material that records facts, configurations, and practical knowledge. Informative. | Cookbooks (they have their own directory). Component-level docs (they stay local). |
| `cookbooks/` | Reference (special form) | Domain-scoped, append-only recipe collections following the cookbook format. | Any document that isn't a cookbook. One file per domain, named `<domain>_cookbook.md`. |
| `workflow/` | Workflow | Artifacts of development workflows: briefs, specs, plans, design studies, arch-designs. Documents with a lifecycle. | Permanent reference material. Documents without a lifecycle. |
| `archive/` | Retired | Documents that have completed their lifecycle. | Active documents. Do not edit archived documents; write new ones instead. |

### 1.2. Cookbooks as a Separate Directory

Cookbooks are conceptually references (they record facts), but they get their own top-level directory because they have a distinctive format (append-only numbered entries), a naming convention that carries domain in the filename (`<domain>_cookbook.md`), and a growth pattern different from other references. This is a pragmatic choice that makes cookbooks instantly findable.

### 1.3. Workflow Subdirectories

Workflow documents are organized by document type:

| Subdirectory | Document type | Definition |
|---|---|---|
| `workflow/specs/` | Specifications | [spec-doc_definition.md](../doc-definitions/spec-doc_definition.md) |
| `workflow/briefs/` | Briefs | [brief_definition.md](../doc-definitions/brief_definition.md) |
| `workflow/plans/` | Implementation plans | [impl-plan_definition.md](../doc-definitions/impl-plan_definition.md) |
| `workflow/design-studies/` | Design studies | [design-study_definition.md](../doc-definitions/design-study_definition.md) |
| `workflow/arch-designs/` | Architecture designs | [arch-design_definition.md](../doc-definitions/arch-design_definition.md) |

### 1.4. Archive Structure

When a document is archived, it moves to `archive/` preserving its relative path within `docs/`. This maintains provenance -- you can tell where the document came from without opening it.

```
# Active
docs/workflow/specs/2026-04-07_auth-token-refresh_spec.md

# Archived
docs/archive/workflow/specs/2026-04-07_auth-token-refresh_spec.md
```

Empty directories in the archive are acceptable. Archived documents are looked up by name, not browsed.

---

## 2. File Naming

### 2.1. Pattern

Every document filename follows a single pattern:

```
[<date>_]<topic>_<doctype>.md
```

| Component | Required | Description | Example |
|---|---|---|---|
| `<date>` | Most workflow | Creation date in `YYYY-MM-DD` format. Present on workflow documents that are point-in-time artifacts (briefs, specs, plans, design studies). Absent on doc-definitions, standards, references, and arch-designs (which are living documents maintained indefinitely). | `2026-04-07` |
| `<topic>` | Yes | Kebab-case description of the document's subject. For cookbooks, this is the domain name. | `auth-token-refresh`, `omp-configuration`, `docker` |
| `<doctype>` | Yes | Document type from the closed suffix inventory (section 2.3). | `spec`, `plan`, `definition`, `cookbook` |

**Date convention:** The `<date>` component must be the actual calendar date when the document is created. Do not copy the date from an existing document or template. The date in the filename and the `date` field in the document's YAML metadata must match and must both reflect the true creation date. This date is immutable after creation -- it does not change when the document is updated.

### 2.2. Separator Rule

**Underscores separate structural elements. Hyphens separate words within an element.**

- `2026-04-07` -- hyphens within the date element (ISO 8601)
- `auth-token-refresh` -- hyphens within the topic element
- `2026-04-07_auth-token-refresh_spec.md` -- underscores between date, topic, and doctype

The last underscore-delimited segment is always the doctype. This is positional, not count-based -- a single-word topic like `cookbook_definition.md` is unambiguous.

### 2.3. Doctype Suffix Inventory

| Suffix | Role | Description |
|---|---|---|
| `_brief` | Workflow | Short orientation document that frames work before committing to a spec or plan |
| `_spec` | Workflow | Feature specification |
| `_plan` | Workflow | Implementation plan (new-feature or bug-fix) |
| `_design-study` | Workflow | Structured analysis of a problem space with alternatives and evaluation |
| `_arch-design` | Workflow | Settled internal architecture of a component, library, or subsystem |
| `_definition` | Doc-definitions | Defines the format and conventions for a document type |
| `_standards` | Standards | Defines conventions for a domain or practice |
| `_cookbook` | Reference | Append-only collection of domain-scoped recipes |
| `_reference` | Reference | Informative reference material |
| `_setup` | Reference | Installation and configuration record |

This set grows only when a genuinely new document type emerges. Before adding a suffix, check whether the document fits an existing type.

### 2.4. README Exception

READMEs do not follow the `<topic>_<doctype>.md` naming pattern. They are always named `README.md` -- no date prefix, no topic slug, no doctype suffix. This is a universal convention recognized by every developer, tool, and hosting platform. The fixed name is the file's identity.

READMEs live with the code they describe (see section 3.4), not in `docs/`. Their format is defined in [readme_definition.md](../doc-definitions/readme_definition.md).

### 2.5. Doctype Redundancy

The doctype appears in both the filename and the directory path (e.g., `workflow/specs/` contains `_spec` files). This redundancy is intentional -- a file moved, linked, or referenced outside its directory retains its identity. READMEs are exempt: they carry no doctype suffix because their fixed name already identifies them.

---

## 3. Placement Rules

### 3.1. Central vs Local

The `docs/` tree holds documents where centralization wins. Component-level documentation stays local to the component.

| Document scope | Where it lives | Examples |
|---|---|---|
| **Component** | In the component's directory | README next to source, API docs, component architecture |
| **Subsystem** | At the lowest common ancestor of the directories it spans | `rendering/docs/` for a doc spanning `rendering/scene_graph/`, `rendering/shader_pipeline/`, `rendering/layout_engine/` |
| **Cross-workspace or repo** | In the central `docs/` tree | Specs, plans, design studies, doc-definitions, standards, cross-cutting references, cookbooks |

The rule: **a document lives at the narrowest scope that contains everything it describes.**

### 3.2. Tool-Mandated Files

Files like `AGENTS.md`, `CLAUDE.md`, `SYSTEM.md`, and `.cursor/rules/*.mdc` have placement dictated by tooling conventions. Their path is their function -- moving them breaks auto-discovery. These are a parallel system. The `docs/` tree does not account for them and should not try to subsume them.

### 3.3. README Placement and Maintenance

READMEs are the primary component-level documentation. They follow specific rules:

1. **One per package.** Every directory with a BUILD file that exports headers or binaries consumed by other packages should have a `README.md`.
2. **At the package root.** The README sits next to the BUILD file, not in a subdirectory.
3. **Always current.** A README must reflect the current state of the code. When a change modifies the package's public surface (API, files, build targets, test coverage), the README is updated in the same commit. A stale README is a defect.
4. **No forward-looking content.** TODOs, improvement ideas, and future work belong in `TODO.md`, not the README.

For the full format specification, see [readme_definition.md](../doc-definitions/readme_definition.md).

### 3.4. Workflow Subdivision by Domain

Workflow directories (`briefs/`, `specs/`, `plans/`, `design-studies/`) start flat. Subdivide by domain when a directory exceeds ~30-50 files with active contributions from multiple domains.

When subdividing:

1. **Subdivide all workflow subdirectories together.** If `specs/` gets domain subdirectories, `briefs/`, `plans/`, and `design-studies/` should too. Inconsistent structure within `workflow/` is worse than slightly empty directories.
2. **Domain names match the repo's top-level structure.** In a monorepo with `frontend/`, `backend/`, `platform/` workspaces, the domain subdirectories mirror those names. In a single-purpose repo, domains might be functional areas or tool names.
3. **The trigger is observable friction, not anticipated growth.** Do not create domain subdirectories preemptively.


### 3.5. Cross-Reference Scope

Documents governed by a doc-definition must only reference documents within the same `docs/` tree or within the same repository. Do not link to documents that belong to another repository, another team's `docs/` tree, or a tool-mandated file in a different codebase (e.g., another project's `AGENTS.md`).

This rule exists because external references create invisible dependencies: when the target moves, renames, or disappears, every document that links to it silently breaks. Documents inside the same `docs/` tree move together; documents across repo boundaries do not.

When a document needs to mention an external resource (an upstream standard, a third-party API reference, a sibling repo's conventions), describe the concept inline and note where the authoritative source lives. Do not hyperlink it as though it were a stable part of this documentation set.

**Doc-definitions vs produced documents.** Doc-definition files (those in `doc-definitions/`) include a "Governing Standards" footer linking back to this standards document -- this helps agents and readers gather the full context for the documentation system. Documents *produced from* those definitions (actual READMEs, actual specs, actual cookbooks, etc.) must not include such a footer or reference this standards document. The definitions are the bridge between the standards and the produced documents; produced documents should not reach past their definition.
---

## 4. Quick Reference

**"Where does this document go?"**

| I have a... | It goes in... | Named... |
|---|---|---|
| Document format definition | `doc-definitions/` | `<topic>_definition.md` |
| Convention for a practice | `standards/` | `<topic>_standards.md` |
| Reference material | `references/` | `<topic>_reference.md` |
| Tool/environment setup record | `references/` | `<topic>_setup.md` |
| Domain cookbook | `cookbooks/` | `<domain>_cookbook.md` |
| Work-framing brief | `workflow/briefs/` | `YYYY-MM-DD_<topic>_brief.md` |
| Feature specification | `workflow/specs/` | `YYYY-MM-DD_<topic>_spec.md` |
| Implementation plan | `workflow/plans/` | `YYYY-MM-DD_<topic>_plan.md` |
| Design study | `workflow/design-studies/` | `YYYY-MM-DD_<topic>_design-study.md` |
| Architecture design | `workflow/arch-designs/` | `<topic>_arch-design.md` |
| Retired document | `archive/<original-path>/` | Same filename |
| Component README | With the component (not in `docs/`) | `README.md` -- see [readme_definition.md](../doc-definitions/readme_definition.md) |
