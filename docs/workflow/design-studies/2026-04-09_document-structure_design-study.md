---
title: Document Structure
status: resolved
date: 2026-04-09
author: user
---

# Document Structure

---

## 1. Problem Statement

The configuration standards (UC1.3) define where agent configuration lives (rules, skills, commands) but not where the documents those configurations act on are stored. As the repository accumulates specs, plans, design studies, cookbooks, references, doc-definitions, and standards, the lack of an organizing principle creates friction: documents land in ad hoc locations, naming is inconsistent, and finding a document requires knowing where someone happened to put it.

This matters because the problem scales with repository complexity. A shallow repo can absorb disorganization; a deep monorepo with dozens of workspaces and hundreds of markdown files cannot. The design must work for both.

This study analyzes: directory structure (role and scope as organizing axes), file naming conventions (separator rules, doctype suffixes, date prefixes), the locality-vs-sparsity trade-off for monorepos, and the boundary between the `docs/` tree and tool-mandated files (AGENTS.md, CLAUDE.md). It does not cover the content or formatting of individual document types -- those are defined by their respective doc-definitions.

---

## 2. Current State

### 2.1. What exists today

```
docs/
  workflow/
    specs/
      2026-04-07_feature-alpha-spec.md
    plans/
      2026-04-07_feature-alpha-plan.md
      2026-04-08_stale-cache-branch-switch-fix-plan.md
  other/
    agents/
      general/
        spec_doc-reference.md
        impl_plan-reference.md
        cookbook-reference.md
        omp-configuration-reference.md
        omp-configuration-standards.md
        dev-environment-setup.md
      applied/
        frontend_cookbook.md
    cookbooks/
      wezterm_cookbook.md
    old_workflow/
      (legacy specs, plans, and misc documents)
```

### 2.2. Problems with the current structure

1. **No organizing principle.** `other/` is a catch-all. `agents/general/` vs `agents/applied/` vs `cookbooks/` are ad hoc distinctions that emerged from usage rather than design.
2. **Mixed concerns.** `agents/general/` contains doc-definitions that define document formats (spec definition, plan definition, cookbook definition) and standards that define conventions (OMP configuration standards) alongside reference docs that describe tools (dev-environment setup, OMP configuration reference). These have different roles.
3. **Cookbooks live in two places.** `other/cookbooks/wezterm_cookbook.md` and `other/agents/applied/frontend_cookbook.md`. The `agents/` prefix on one but not the other has no clear rationale.
4. **Audience is implicit.** Nothing in the path signals whether a document is for agents, humans, or both.
5. **`old_workflow/` is a graveyard.** Documents that predate the current conventions sit in a flat directory with no lifecycle signal.

---

## 3. Dimensions

### 3.1. Role

How does the document relate to the repository?

| Role | Verb | Description | Examples |
|---|---|---|---|
| **Doc-definitions** | "how documents are structured" | Defines formats and structural requirements for document types. Other documents must conform to these. Changes here propagate to all conforming documents. | spec-doc_definition.md, impl-plan_definition.md, cookbook_definition.md, design-study_definition.md |
| **Standards** | "how we do things" | Defines conventions, standards, and best practices for practices and domains. | document-structure_standards.md, omp-configuration_standards.md |
| **Reference** | "what exists / how it works" | Reference material for humans and agents. Records facts, configurations, and practical knowledge. Does not impose structure on other documents. | omp-configuration_reference.md, dev-environment_setup.md, cookbooks |
| **Workflow** | "artifacts of work" | By-products of development workflows. Created, used, and eventually archived as work progresses. The repository's operational history. | specs, plans, design studies |
| **Other** | catch-all | Documentation that doesn't fit the above. Should be rare -- if this bucket grows, a new role is needed. | Legacy docs, one-off investigations |

**Observations:**

- Doc-definitions, Standards, and Reference are all "reference" in the informal sense, but they differ in a critical way: Doc-definitions documents are normative over document formats (they define what's correct for other documents), Standards documents are normative over practices and domains, and Reference documents are informative (they record what's true). A Reference document may itself contain normative statements about user behavior ("you must run git inside the container") -- but it does not govern the structure of other documents in the tree.
- Workflow documents have a lifecycle (draft -> in-progress -> complete -> archived). Doc-definitions, Standards, and Reference documents are maintained indefinitely or until superseded.
- Cookbooks are Reference: they record findings. They don't define how other documents should look. The cookbook *definition* is a Doc-definition: it defines how cookbooks must be structured.

**Conclusion:** Five roles identified: Doc-definitions (normative over document formats), Standards (normative over practices/domains), Reference (informative), Workflow (artifacts of work), and Other (catch-all). Role is the primary organizing axis for the `docs/` tree. The directory name encodes the role.

### 3.2. Scope

What part of the codebase or work does the document apply to?

| Scope | Description | Examples |
|---|---|---|
| **Repository** | Applies to the entire repo. Not specific to any project, tool, or subdirectory. | Spec definition, plan definition, cookbook definition |
| **Domain** | Applies to a specific working context: a project (`frontend`), a tool (`wezterm`, `omp`, `git`), or a technology area (`simulation`). | frontend_cookbook.md, wezterm_cookbook.md, omp-configuration_reference.md |
| **Feature / task** | Applies to a single feature or task. Tied to a specific piece of work. | A spec, a plan |

**Observations:**

- Scope is mostly orthogonal to role. A Doc-definitions document can be repo-wide (spec definition) or domain-scoped (a domain-specific definition). A Standards document can be repo-wide (document-structure standards) or domain-scoped (a domain-specific coding standard). A Workflow document is almost always feature/task-scoped.
- The current directory structure partially encodes scope (`agents/general/` vs `agents/applied/`) but inconsistently.
- Cookbooks are always domain-scoped by definition (per cookbook reference, section 4).

#### 3.2.1. Locality vs Sparsity

In a shallow repo, all documents can live in a central `docs/` tree without friction -- there's only one level of navigation. In a deep monorepo (e.g., my-monorepo with 15+ workspaces, 500+ markdown files, and `docs/` directories scattered across `backend/`, `services/`, `platform/`, etc.), two competing forces emerge:

**Locality:** Place documentation close to the code it documents. A component README lives next to the component's source. A workspace-level design doc lives in that workspace's `docs/` directory. Benefits: co-discoverable with the code, maintained by the same people who own the code, visible during code review of that directory.

**Sparsity:** The more documents are distributed, the harder they are to find across the repo. A developer looking for "how do we write specs?" has no idea which workspace's `docs/` directory to search. Centralization makes cross-cutting documents findable but disconnects them from the code they describe.

**Interaction with role:**

| Role | Locality pressure | Rationale |
|---|---|---|
| Doc-definitions | Low (centralize) | Definitions apply across the repo. Scattering them into workspaces makes them invisible to other workspaces. A developer in `backend/` shouldn't have to know that the spec format is defined in `frontend/docs/`. |
| Standards | Low (centralize) | Standards apply across the repo. Scattering them into workspaces makes them invisible to other workspaces. |
| Reference | Mixed | Repo-wide references (OMP configuration) centralize naturally. Domain-scoped references (component READMEs) belong with the code. Cookbooks centralize because they're cross-cutting by nature, even when domain-scoped. |
| Workflow | Low (centralize) | Specs and plans often span multiple workspaces (a feature touches `backend/`, `frontend/`, and `shared/`). Placing them in one workspace's `docs/` is arbitrary. A central `workflow/` tree avoids the question. |
| Component docs | High (localize) | READMEs, API docs, and architecture docs for a specific library or component belong next to that code. These are the documents that should NOT be centralized. |

**Conclusion:** Scope is secondary to role for the central `docs/` tree. The `docs/` tree holds documents where centralization wins. Component-level documentation stays local to the component (narrowest containing scope rule). Scope is encoded in the filename (topic portion), not in the directory path -- except for workflow directories, which may add domain subdirectories when they grow large enough (see section 5).

### 3.3. Intended Audience

Who reads this document?

| Audience | Description |
|---|---|
| **Agents** | Written primarily for AI agents. Uses direct instructions, structured formats, agent-readable conventions. |
| **Humans** | Written primarily for human readers. Uses prose, visual formatting, context that assumes human background knowledge. |
| **Both** | Useful to both. Most documents fall here in practice. |

**Observations:**

- Audience is the weakest organizing dimension. Almost every document in this repo is useful to both agents and humans. The distinction matters most for tone and formatting, not for placement.
- Pure agent-only documents are rare -- mostly agent configuration (rules, skills, SYSTEM.md), which already live in `.omp/` and `.claude/`, not in `docs/`.
- Pure human-only documents are also rare in a repo where agents are first-class contributors.
- Encoding audience in the directory path would create artificial splits. A cookbook is useful to both; forcing it into an `agents/` or `humans/` bucket adds noise without helping anyone find it.

**Conclusion:** Audience should influence writing style (per-document), not directory placement. Drop it as a structural axis.

#### 3.3.1. Tool-Mandated Files

While audience does not drive directory structure, certain files have placement dictated by tooling conventions. These files exist at fixed paths because agents and their harnesses expect them there -- not because of an organizational design choice.

| File | Required path | Discovery mechanism | Tool(s) |
|---|---|---|---|
| `CLAUDE.md` | `<cwd>/.claude/CLAUDE.md` or `<dir>/CLAUDE.md` | Auto-discovered by depth-based walk (up to 20 ancestors). Content merged into system prompt when agent works in or under that directory. | OMP, Claude Code |
| `AGENTS.md` | `<dir>/AGENTS.md` | Auto-discovered by depth-based walk. Same merge behavior as `CLAUDE.md`. Broadest cross-tool reach. | OMP, Claude Code, Codex CLI |
| `SYSTEM.md` | `<cwd>/.omp/SYSTEM.md` | Discovered at startup. Replaces entire system prompt (see OMP configuration reference, section 2.12). | OMP |
| `.cursor/rules/*.mdc` | `<cwd>/.cursor/rules/` | Auto-discovered at startup by Cursor. | Cursor |

**Key properties:**

- **Path is the contract.** Moving `AGENTS.md` out of the directory it applies to breaks auto-discovery. These files cannot be reorganized into a central `docs/` tree -- their placement IS their function.
- **Harness registry update.** When a new `CLAUDE.md` or `AGENTS.md` is added, the corresponding harness picks it up automatically (no registration step). When removed, the harness stops injecting it. This is pure convention-based discovery.
- **Scope is encoded by depth.** A root-level `CLAUDE.md` applies to the entire repo. A `frontend/AGENTS.md` applies only when the agent works within `frontend/`. This depth-scoped injection is the mechanism by which tool-mandated files achieve locality without a central index.
- **Not documentation.** These files are configuration masquerading as documentation. They instruct the agent, not the reader. They belong to the `.omp/` / `.claude/` / tool-configuration layer described in the configuration standards, not to the `docs/` tree. The `docs/` structure designed here is for documents that humans and agents read by choice, not files that tools inject by convention.

**Implication for design:** Tool-mandated files are a parallel system. The `docs/` tree does not need to account for them, and should not try to subsume them. They live where they live because the tools require it.


**Conclusion:** File naming uses the `[<date>_]<topic>_<doctype>.md` pattern. Underscores separate structural elements; hyphens separate words within an element. Date prefix for workflow documents only. Doctype suffix from a closed inventory. See subsections 3.4.1-3.4.5 for full specification.

### 3.4. File Naming

How are document filenames structured?

#### 3.4.1. Anatomy

Every document filename follows a single pattern:

```
[<date>_]<topic>_<doctype>.md
```

| Component | Required | Description | Example |
|---|---|---|---|
| `<date>` | Workflow only | Creation date in `YYYY-MM-DD` format. Present on workflow documents (specs, plans) because they are point-in-time artifacts. Absent on doc-definitions, standards, and references, which are maintained indefinitely -- a creation date would go stale and mislead. | `2026-04-07` |
| `<topic>` | Yes | Kebab-case description of the document's subject matter. The author's judgment call; no length or abbreviation rules. For cookbooks, this is the domain name. | `auth-token-refresh`, `omp-configuration`, `wezterm` |
| `<doctype>` | Yes | The document type from a closed set of suffixes. Makes documents greppable by type and signals the format to expect. | `spec`, `plan`, `definition`, `cookbook`, `reference`, `standards`, `setup` |

#### 3.4.2. Separator Rule

**Underscores separate structural elements. Hyphens separate words within an element.**

This is the single rule that resolves all separator ambiguity:

- `2026-04-07` — hyphens within the date element (ISO 8601)
- `feature-alpha` -- hyphens within the topic element
- `2026-04-07_feature-alpha_spec.md` -- underscores between date, topic, and doctype

The last underscore-delimited segment is always the doctype. This is positional, not count-based -- a single-word topic like `cookbook_definition.md` is unambiguous.

#### 3.4.3. Doctype Suffix Inventory

The closed set of recognized suffixes:

| Suffix | Role | Description |
|---|---|---|
| `_spec` | Workflow | Feature specification |
| `_plan` | Workflow | Implementation plan (new-feature or bug-fix) |
| `_design-study` | Workflow | Structured analysis of a problem space with alternatives and evaluation |
| `_definition` | Doc-definitions | Defines the format and conventions for a document type |
| `_standards` | Standards | Defines conventions for a domain or practice |
| `_cookbook` | Reference | Append-only collection of domain-scoped recipes |
| `_reference` | Reference | Informative reference material |
| `_setup` | Reference | Installation and configuration record |

This set grows only when a genuinely new document type emerges. Before adding a suffix, check whether the document fits an existing type.

#### 3.4.4. Interaction with Directory Structure

The filename carries information that the directory path does not:

- **Doctype** is always in the filename, even though the directory may also signal it (e.g., `workflow/specs/` contains `_spec` files). The redundancy is intentional -- a file moved, linked, or referenced outside its directory retains its identity.
- **Date** appears only in workflow filenames. The directory (`workflow/specs/`) already signals the role; the date signals the temporal ordering within that role.
- **Topic** is the primary differentiator within a directory. It should be specific enough to distinguish the document from its siblings without needing to open it.

#### 3.4.5. Current Files Under This Convention

| Current filename | Proposed filename | Change |
|---|---|---|
| `2026-04-07_feature-alpha-spec.md` | `2026-04-07_feature-alpha_spec.md` | Hyphen before `spec` -> underscore |
| `2026-04-08_stale-cache-branch-switch-fix-plan.md` | `2026-04-08_stale-cache-branch-switch-fix_plan.md` | Hyphen before `plan` -> underscore |
| `spec_doc-reference.md` | `spec-doc_definition.md` | Underscore within topic → hyphen; suffix → `_definition` |
| `impl_plan-reference.md` | `impl-plan_definition.md` | Underscore within topic → hyphen; suffix → `_definition` |
| `cookbook-reference.md` | `cookbook_definition.md` | Hyphen before suffix → underscore; suffix → `_definition` |
| `omp-configuration-reference.md` | `omp-configuration_reference.md` | Hyphen before suffix → underscore |
| `omp-configuration-standards.md` | `omp-configuration_standards.md` | Hyphen before suffix → underscore |
| `dev-environment-setup.md` | `dev-environment_setup.md` | Hyphen before suffix → underscore |
| `frontend_cookbook.md` | `frontend_cookbook.md` | Already conforms |
| `wezterm_cookbook.md` | `wezterm_cookbook.md` | Already conforms |
---

## 4. Design Options

### 4.1. Option A: Role as top-level directory

```
docs/
  doc-definitions/               # Doc-definitions: normative document formats
    spec-doc_definition.md
    impl-plan_definition.md
    cookbook_definition.md
    design-study_definition.md
  standards/                      # Standards: conventions and best practices
    omp-configuration_standards.md
    document-structure_standards.md
  references/                    # Reference: informative documents
    omp-configuration_reference.md
    dev-environment_setup.md
    frontend_cookbook.md
    wezterm_cookbook.md
  workflow/                      # Workflow: artifacts of work
    specs/
      2026-04-07_feature-alpha_spec.md
    plans/
      2026-04-07_feature-alpha_plan.md
    design-studies/
      2026-04-09_document-structure_design-study.md
  archive/                       # Other / retired
    old_workflow/
      ...
```

**Pros:**
- Role is immediately visible from the path.
- Clear rule: "If it defines how documents are structured, it's in `doc-definitions/`. If it defines conventions for a practice or domain, it's in `standards/`. If it records facts or knowledge, it's in `references/`. If it's a work artifact, it's in `workflow/`."
- Workflow documents already live in `workflow/` -- no change needed there.

**Cons:**
- `doc-definitions/` / `standards/` vs `references/` distinction requires judgment. Is `omp-configuration_standards.md` (standard) vs `omp-configuration_reference.md` (reference) obvious to a newcomer?
- Cookbooks all land in one flat directory regardless of domain. With many cookbooks, domain becomes the next needed split.
- Doesn't encode scope. `frontend_cookbook.md` and `wezterm_cookbook.md` sit next to each other with no signal that they serve different domains.

### 4.2. Option B: Role as top-level, scope as second level

```
docs/
  doc-definitions/              # Doc-definitions
    spec-doc_definition.md
    impl-plan_definition.md
    cookbook_definition.md
    design-study_definition.md
  standards/                    # Standards
    omp/
      configuration_standards.md
    document-structure_standards.md
  references/                   # Reference
    omp/
      configuration_reference.md
    tools/
      dev-environment_setup.md
    cookbooks/
      frontend_cookbook.md
      wezterm_cookbook.md
  workflow/                     # Workflow
    specs/
      2026-04-07_feature-alpha_spec.md
    plans/
      2026-04-07_feature-alpha_plan.md
    design-studies/
      2026-04-09_document-structure_design-study.md
  archive/
    ...
```

**Pros:**
- Both role and scope are encoded in the path.
- Cookbooks group naturally under `references/cookbooks/`.
- Repo-wide doc-definitions sit at the top of `doc-definitions/`; domain-scoped standards nest under their domain in `standards/`.

**Cons:**
- Deeper nesting. `docs/references/cookbooks/frontend_cookbook.md` is verbose.
- The `omp/` domain appears under both `standards/` and `references/`, splitting related OMP documents.
- Workflow documents are already scoped by type (specs/, plans/) -- adding a domain level inside those might be premature.

### 4.3. Option C: Scope as top-level, role as convention

```
docs/
  repo/                         # Repository-scoped
    doc-definitions/
      spec-doc_definition.md
      impl-plan_definition.md
      cookbook_definition.md
      design-study_definition.md
    standards/
      document-structure_standards.md
    ...
  omp/                          # Domain: OMP
    configuration_reference.md
    configuration_standards.md
  tools/                        # Domain: terminal tools
    dev-environment_setup.md
  cookbooks/                    # All cookbooks (domain encoded in filename)
    frontend_cookbook.md
    wezterm_cookbook.md
  workflow/                     # Workflow artifacts
    specs/
      ...
    plans/
      ...
    design-studies/
      ...
  archive/
    ...
```

**Pros:**
- Domain-scoped documents cluster by domain, which is how you look for them ("I need OMP docs").
- Cookbooks stay in one place; the filename carries the domain.
- Repo-wide doc-definitions have a clear home.

**Cons:**
- Role is not visible in the path. You'd need to open a file to know if it defines a document format or records facts.
- `cookbooks/` is a role-based directory alongside scope-based directories -- mixed organizing principles.
- Workflow could arguably be a scope (`feature/task`) or a role. Treating it as top-level alongside scopes is hybrid.

### 4.4. Option D: Flat role directories, minimal nesting

```
docs/
  doc-definitions/             # Doc-definitions: how documents are structured
    spec-doc_definition.md
    impl-plan_definition.md
    cookbook_definition.md
    design-study_definition.md
  standards/                   # Standards: conventions and best practices
    omp-configuration_standards.md
    document-structure_standards.md
  references/                  # Reference: what exists, how it works
    omp-configuration_reference.md
    dev-environment_setup.md
  cookbooks/                   # Reference (special form): domain-scoped recipes
    frontend_cookbook.md
    wezterm_cookbook.md
  workflow/                    # Workflow: specs, plans, and design studies
    specs/
      ...
    plans/
      ...
    design-studies/
      ...
  archive/                     # Retired documents
    ...
```

**Pros:**
- Shallow. Maximum two levels of nesting for any document.
- Role is encoded in the top-level directory name. `doc-definitions/` is clearly normative over document formats. `standards/` is clearly normative over practices. `references/` is clearly informative.
- Cookbooks get their own top-level directory because they have a distinct format (append-only, numbered entries) and their own naming convention (`<domain>_cookbook.md`). Domain is in the filename, not the path.
- `workflow/` stays as-is. Specs and plans are already well-organized.
- `archive/` replaces `other/old_workflow/` with a clearer name.

**Cons:**
- Cookbooks are conceptually a subset of `references/` (they record facts) but get their own directory. This is a pragmatic choice (they have a distinctive format and naming) rather than a principled one.
- If `references/` grows large, there's no scope-based subdivision -- but the current size doesn't warrant it.

---

## 5. Evaluation

| Criterion | A | B | C | D |
|---|---|---|---|---|
| Role visible in path | Yes | Yes | No | Yes |
| Scope visible in path | No | Yes | Yes | Partial (filename) |
| Shallow nesting | Yes | No | Moderate | Yes |
| Cookbooks well-placed | Flat in references/ | Nested in references/cookbooks/ | Top-level cookbooks/ | Top-level cookbooks/ |
| OMP docs together | No (split by role) | No (split by role) | Yes | No (split by role) |
| Workflow unchanged | Yes | Yes | Yes | Yes |
| Intuitive for newcomers | Moderate | Moderate | Moderate | High |
| Scales to more domains | Poorly (flat) | Well | Well | Moderately (filename-based) |
| Monorepo-friendly | Moderate | Moderate | High | Moderate |
| Locality/sparsity clear | No | Partial | Yes (scope-based) | No |

**Trade-off:** Options A/B/D prioritize role (the normative vs informative distinction). Option C prioritizes scope (domain clustering). The question is: when you're looking for a document, do you think "I need a standard" (role-first) or "I need OMP docs" (scope-first)?

**Scale matters.** In a shallow repo, this question barely matters -- every directory is one click away. In a deep monorepo, the answer depends on document role:

- For Doc-definitions, Standards, and Workflow documents, you think role-first: "where are the document format definitions?" or "where are the specs?" These are few, cross-cutting, and must be centrally findable.
- For Reference documents (component READMEs, architecture docs), you think scope-first: "what docs exist for this component?" These belong near the code (locality wins).
- Cookbooks and cross-cutting references land in the middle -- they're domain-scoped but not tied to a specific component directory.

This suggests that role-first for the central `docs/` tree is correct, **combined with** the convention that component-level documentation stays in-tree with the component (never centralized into `docs/`). The `docs/` tree holds what must be findable across the repo; everything else stays local.

**Interaction with tool-mandated files:** The central `docs/` tree and the tool-mandated file system (AGENTS.md, CLAUDE.md) serve complementary purposes. `docs/` is for documents people seek out. Tool-mandated files are injected by the harness based on directory depth. A workspace might have both `frontend/AGENTS.md` (tool-mandated, auto-injected) and `docs/cookbooks/frontend_cookbook.md` (central, sought out). They don't compete -- they serve different access patterns.


**Workflow subdivision by domain.** Workflow directories (`specs/`, `plans/`, `design-studies/`) start flat. As a repository grows, the question arises: should they subdivide by domain? The answer depends on the design option and on a general growth policy.

*Structural question: where does domain appear?*

In options A, B, and D, workflow lives under a central `workflow/` tree organized by document type. Subdivision means adding a domain level below the doctype: `workflow/specs/frontend/`. The workflow tree stays centralized; domain is a filter within it.

Option C is fundamentally different. Its scope-first principle could place workflow artifacts under their domain: `frontend/workflow/specs/` rather than `workflow/specs/frontend/`. This distributes workflow documents by scope, which is consistent with C's organizing principle but breaks the central findability that workflow documents benefit from (see section 3.2.1 -- workflow has low locality pressure). A hybrid is possible (central `workflow/` with domain subdirectories), but that concedes the scope-first principle for workflow documents, making C behave like A/B/D in this area.

| Option | Subdivision path | Consistent with organizing principle? |
|---|---|---|
| A | `workflow/specs/frontend/` | Yes -- role-first, domain is secondary |
| B | `workflow/specs/frontend/` | Yes -- role-first with scope as second level |
| C (distributed) | `frontend/workflow/specs/` | Yes -- scope-first, but loses central findability |
| C (centralized) | `workflow/specs/frontend/` | No -- concedes to role-first for workflow |
| D | `workflow/specs/frontend/` | Yes -- role-first, domain is secondary |

*Growth policy: when to subdivide.*

Domain subdirectories are overhead at small scale (empty structure, deeper paths) and essential at large scale (50+ files in a single directory become unscannable). The trigger should be observable friction, not anticipated growth:

1. **Start flat.** Date-prefixed filenames provide temporal ordering. The topic portion of the filename carries domain identity implicitly (`auth-token-refresh` is recognizably backend). At low file counts, this is sufficient.
2. **Subdivide when scanning breaks down.** When a workflow directory accumulates enough files that scanning past unrelated domains becomes friction -- roughly 30-50 files with active contributions from multiple domains -- introduce domain subdirectories.
3. **Subdivide all doctype directories together.** If `specs/` gets domain subdirectories, `plans/` and `design-studies/` should too, even if they're smaller. Inconsistent structure within `workflow/` is worse than slightly empty directories.
4. **Domain names match the repo's top-level structure.** In a monorepo with `frontend/`, `backend/`, `platform/` workspaces, the domain subdirectories mirror those names. In a single-purpose repo, domains might be functional areas or tool names instead.

This policy applies regardless of which design option is chosen (with the caveat that Option C's distributed variant doesn't have a central `workflow/` to subdivide).

**Ranking and recommendation.**

Ranking: **D > A > B > C.** Option D is the recommended structure.

*1st: Option D -- Flat role directories, minimal nesting.* Best balance across repo sizes. Role is visible in the path. Nesting is shallow -- two levels max for any document. The separation of cookbooks into their own top-level directory is pragmatic rather than principled (they're conceptually references), but it earns its keep: cookbooks have a distinctive format, a naming convention that carries domain in the filename, and a growth pattern (append-only) different from other references. Giving them their own directory makes them instantly findable without adding depth.

For shallow repos, D is trivially navigable -- five top-level directories, each with a handful of files. No empty structure, no premature taxonomy. For monorepos, D works because the central `docs/` tree only holds documents where centralization wins (per section 3.2.1). Component docs stay local. The flat structure doesn't encode domain in the path, but the filename carries it (`omp-configuration`), and the workflow subdivision policy handles growth when it arrives.

The weakness is that D splits domain-related documents by role: `omp-configuration_standards.md` in `standards/` and `omp-configuration_reference.md` in `references/`. But this is the correct split -- these documents serve different purposes. A reader looking for "the OMP configuration standard" and a reader looking for "the OMP configuration reference" are asking different questions.

*2nd: Option A -- Role as top-level directory.* A is D without the separate `cookbooks/` directory. More principled (cookbooks are references) but less practical -- as cookbooks accumulate, they mix with documents that have a completely different format and growth pattern. You'd likely evolve into D anyway, making A a starting point that creates a future restructure.

*3rd: Option B -- Role as top-level, scope as second level.* The most informative path structure -- both role and scope are visible. But it pays for that with depth (`docs/references/cookbooks/frontend_cookbook.md`) and premature taxonomy. Domain subdirectories in the central tree would mostly be sparse, because the locality principle (section 3.2.1, Q6) pushes domain-scoped documents to their component directories, not to the central tree. B is the right answer if the central tree is expected to hold hundreds of documents across many domains -- unlikely given the locality principle.

*4th: Option C -- Scope as top-level, role as convention.* Weakest for two reasons. First, role is invisible in the path -- you can't tell whether a document in `omp/` is a standard or a reference without opening it. Second, it mixes organizing principles: `cookbooks/` and `workflow/` are role-based directories alongside scope-based directories like `omp/` and `tools/`. C's strength is domain clustering, but the locality principle undercuts it: if domain clustering matters, the documents should be with the code (local), not in a central domain directory. The resolved open questions (Q2 workflow subdivision, Q6 component docs boundary) both weaken C's case -- they establish that cross-cutting documents are better served by role-first organization and that domain-scoped documents belong local to the code.

**Conclusion:** Option D. The recommended `docs/` tree structure is:

```
docs/
  doc-definitions/             # Defines formats for document types
  standards/                   # Defines conventions and best practices
  references/                  # Records facts, configurations, practical knowledge
  cookbooks/                   # Domain-scoped append-only recipe collections
  workflow/                    # Specs, plans, and design studies
    specs/
    plans/
    design-studies/
  archive/                     # Retired documents (preserving relative path)
```

Combined with:

- File naming: `[<date>_]<topic>_<doctype>.md` (section 3.4).
- Component docs stay local to the component, not in `docs/` (section 3.2.1, Q6).
- Tool-mandated files (AGENTS.md, CLAUDE.md) are a parallel system, not part of `docs/` (section 3.3.1).
- Workflow directories start flat; subdivide by domain when scanning breaks down (~30-50 files).
- Archives preserve relative paths.

---

## 6. Open Questions

1. ~~**Naming: `doc-definitions/` and `standards/` vs `reference/` naming.**~~ Resolved: `doc-definitions/` for normative document-format definitions, `standards/` for normative conventions over practices and domains, `references/` for informative material. Avoids the level-of-analysis trap where "normative" and "informative" confuse document-governing normativity with operational normativity. `doc-definitions/` is unambiguous: it contains definitions for document formats. `standards/` is unambiguous: it contains standards for practices. `references/` is the natural complement. File suffix for doc-definitions: `_definition` (e.g., `cookbook_definition.md`), distinguishing the document that defines a format from documents that conform to it. File suffix for standards: `_standards` (e.g., `omp-configuration_standards.md`). File naming convention: underscores separate structural elements (date, topic, doctype), hyphens separate words within an element (see section 3.4).
2. ~~**Workflow scope subdivision.**~~ Resolved: start flat; subdivide by domain when a workflow directory exceeds ~30-50 files with multiple active domains. Subdivision path is `workflow/<doctype>/<domain>/` for options A/B/D. Option C has a structural choice between distributed (`<domain>/workflow/`) and centralized (`workflow/<doctype>/<domain>/`); the centralized form concedes scope-first for workflow. Growth policy: subdivide all doctype directories together; domain names mirror the repo's top-level structure. See section 5 evaluation for full analysis.
3. ~~**Where do investigation/debugging documents go?**~~ Resolved: these are bug-fix plans. Documents like `stale-cache-investigation.md`, `config-reload-race-condition.md`, and `auth-token-refresh-failure-fix.md` all follow the problem -> investigation -> root cause -> fix -> validation structure defined in the bug-fix plan template (see impl-plan_definition.md, section 4). They predate the formal plan definition and lack its metadata and naming convention, but their content maps directly. They belong in `workflow/plans/` with the `_plan` doctype suffix.
4. ~~**Archive lifecycle.**~~ Resolved: preserve the relative path. A document archived from `workflow/specs/old_spec.md` moves to `archive/workflow/specs/old_spec.md`. This maintains provenance (you can tell where the document came from without opening it), makes archiving a simple move operation with no renaming, and mirrors the live tree structure. The doctype suffix in the filename provides redundant type information, but path preservation is cheaper than reconstruction. Empty directories in the archive are acceptable -- archived documents are looked up by name, not browsed.
5. ~~**Configuration standards update.**~~ Resolved: Option D selected. UC1.3 in `omp-configuration_standards.md` needs a `docs/` section showing the Option D layout (`doc-definitions/`, `standards/`, `references/`, `cookbooks/`, `workflow/`, `archive/`). Tracked as a deliverable in section 7.
6. ~~**Monorepo component docs boundary.**~~ Resolved: a document lives at the narrowest scope that contains everything it describes. Three cases: (a) **Component-scoped:** describes one component's interface, behavior, or internal design -- lives in that component's directory (README, API docs). The test: would a reader find it by navigating to that component? (b) **Subsystem-scoped:** describes a concern spanning several directories within a workspace (e.g., rendering pipeline architecture across `rendering/scene_graph/`, `rendering/shader_pipeline/`, `rendering/layout_engine/`) -- lives at the lowest common ancestor (`rendering/docs/` or `rendering/README.md`), not in any single subdirectory and not in the central tree. (c) **Cross-workspace or repo-scoped:** spans workspaces or is about the repo itself -- lives in the central `docs/` tree. The rule is "narrowest containing scope," not "closest to the most important component."
7. ~~**Tool-mandated file maintenance.**~~ Resolved: out of scope for this study. Tool-mandated files were analyzed in section 3.3.1 to establish that they're a parallel system the `docs/` tree doesn't account for. Prescribing maintenance conventions for them (e.g., a checklist item when renaming workspaces: "check for AGENTS.md and CLAUDE.md and move them") is a concern for the configuration standards document, not for document structure. Noted as a potential addition to omp-configuration_standards.md.


---

## 7. Deliverables

- [x] Standards: `document-structure_standards.md` -- directory structure and file naming conventions for the `docs/` tree (created).
- [x] Update: UC1.3 in `omp-configuration_standards.md` updated to include the `docs/` section in the recommended layout tree.
- [x] Rename and migrate: existing files moved and renamed to conform to the naming convention and directory structure. Executed via companion spec/plan (`2026-04-09_document-structure-migration_spec.md` / `2026-04-09_document-structure-migration_plan.md`).
- [x] Doc-definition: `design-study_definition.md` -- defines the design study document type (created).