# README Definition

Reference for writing README documents. A README is the entry point for a library or package -- the first thing a developer or agent reads when encountering the code. It answers three questions: what does this package do, how do I use it, and where do I go for more information. A README is a living document: it stays current with the code it describes, or it becomes a liability.

---

## 1. Purpose

A README exists to:

1. **Orient** -- tell the reader what the package is, what problem it solves, and who consumes it.
2. **Inventory** -- list the files and their roles so the reader knows what they're looking at without opening every file.
3. **Instruct** -- show how to depend on, include, and call the package's API.
4. **Test** -- document what the tests cover and how to run them.
5. **Link** -- connect the package to its standards, specs, plans, and related packages.

A README is NOT:

- A spec or plan. If the content describes what to build or how to build it, it belongs in `docs/workflow/` (see [spec-doc_definition.md](spec-doc_definition.md), [impl-plan_definition.md](impl-plan_definition.md)).
- A design study. If the content analyzes alternatives and trade-offs, it belongs in `docs/workflow/design-studies/` (see [design-study_definition.md](design-study_definition.md)).
- A TODO list. Unresolved improvement ideas belong in a `TODO.md` file, not in the README. A README describes what the code *is*, not what it should become.
- A tutorial. If the content requires sequential reading or builds on previous sections, it belongs in a skill or a guide.

---

## 2. File Naming and Placement

### 2.1. File Name

Always `README.md`. No date prefix, no topic slug, no doctype suffix. The name is fixed by universal convention -- every developer and tool on the planet knows what `README.md` means and where to look for it.

### 2.2. Placement

One `README.md` per library or package, at the package root -- the same directory as the `BUILD` or `BUILD.bazel` file. This is the narrowest scope that contains everything the README describes.

```
my_package/
  BUILD
  README.md          <-- here
  my_module.h
  my_module.cc
  my_module_test.cc
```

Do not place READMEs inside subdirectories of a package unless the subdirectory is itself a separate package with its own BUILD file. Do not place READMEs in `docs/` -- they live with the code they describe. See the [Document Structure Standards](../standards/document-structure_standards.md) section 3.1 for the central-vs-local rule.

### 2.3. When to Create One

Every package that has a BUILD file and exports headers or binaries consumed by other packages should have a README. Internal-only test helpers or generated code directories may skip it.

---

## 3. Structure

A README has a fixed section order. Sections marked "required" must always be present. Sections marked "when applicable" are included when the package has relevant content and omitted otherwise -- do not include empty placeholder sections. Use horizontal rules (`---`) between top-level sections for visual separation.

### 3.1. Title and Lead Paragraph (required)

```markdown
# <Package Name>

<One to three sentences: what problem the package solves and why it
exists. What it replaces or what gap it fills. Who consumes it.>
```

The H1 title is the package's display name. Use short, recognizable names: `String Utils`, `LRU Cache`, `Config Parser`. The title does not need to match the directory path verbatim, but it should be unambiguous.

The lead paragraph is the most important text in the document. A reader who reads only this paragraph should know whether this package is relevant to them. Lead with *why* the package exists (the problem, the gap, what broke before), then *what* it does about it. The *how* (architecture, encoding schemes, algorithmic design) belongs in the Design section, not the lead.

### 3.2. Contents (required)

```markdown
## Contents

| File | Description |
|---|---|
| `module.h/.cc` | One-line description of what this file provides. |
| `module_test.cc` | Unit tests for module. |
```

A table mapping file names to one-line descriptions. This is the package's inventory -- the reader can scan it to understand the package's scope without opening any file. Group related files on a single row when they form a natural pair (e.g., `module.h/.cc`).

For small packages (1-3 files), a bulleted list is acceptable instead of a table.

### 3.3. API (when applicable)

````markdown
## API

All declarations are in namespace `<namespace>`.

```cpp
#include "<path>/module.h"

// Key types and function signatures.
```
````

Show the public interface in a code block: type definitions, function signatures, and brief inline comments. This is not an exhaustive header dump -- show the types and functions a caller needs to know about. Precede the code block with the include directive and namespace.

Omit this section for packages that don't export a programmatic API (e.g., binaries, test data, configuration).

### 3.4. Usage (required)

````markdown
## Usage

### Build dependency

```python
deps = [
    "//path/to:target",
]
```

### Include path

```cpp
#include "path/to/module.h"
```

### Example

```cpp
// Annotated, runnable code showing the common usage pattern.
```
````

Three subsections:

1. **Build dependency** -- the Bazel `deps` entry a consumer adds to their BUILD file.
2. **Include path** -- the `#include` directive.
3. **Example** -- a short, annotated code sample showing the primary usage pattern. The example should be copy-pasteable (with appropriate setup) and demonstrate the API's intended calling convention. For packages with two distinct usage modes (e.g., convenience API vs optimized API), show both.

### 3.5. Dependencies (required)

````markdown
## Dependencies

### Internal

| Package | Path | Purpose |
|---|---|---|
| Config Parser | [`src/config/parser`](../src/config/parser/README.md) | YAML/JSON config file parsing. |
| Logging | `shared/logging` | Structured logging with level filtering. |

### External

| Package | Version |
|---|---|
| nlohmann/json | 3.11 |
| spdlog | -- |
````

Two subsections:

1. **Internal** -- direct dependencies on packages within the same organization. Three columns: **Package** (display name), **Path** (directory path, linked to its README when in the same project; plain text when in a sibling project), and **Purpose** (one-line statement of what the dependency provides to this package). If the dependency is in the same project (same repo or workspace), include a link to its README in the Path column. If it is in a sibling project, list the name and path without a link.
2. **External** -- direct dependencies on third-party libraries. Include the version when the package pins or requires a specific version. Use `--` when no specific version is required.

List only direct dependencies that a consumer transitively inherits by depending on this package. Test-only dependencies (e.g., Google Test, test helpers) are not listed -- they do not affect consumers. Transitive dependencies are not listed -- they are documented in their own packages' READMEs.

### 3.6. Build Targets (when more than two targets)

```markdown
## Build Targets

| Target | Type | Description |
|---|---|---|
| `:types` | `cc_library` | Shared data types and constants. |
| `:parser` | `cc_library` | Config file parser implementation. |
| `:validator` | `cc_library` | Schema validation rules. |
| `:test` | `cc_test` | Unit tests. |
```

A table of the Bazel targets this package exports, with the target type (`cc_library`, `cc_test`, `cc_binary`, `py_library`, etc.) and a one-line description of what each provides. This is what a consumer scans to decide which target to add to their `deps`.

Include this section only when the package defines more than two targets. For packages with one or two targets, the Usage section's build dependency snippet already identifies the target -- a separate table adds no information. List only targets intended for external consumption; internal-only targets (e.g., test helpers not used outside the package) may be omitted.

### 3.7. Design or Architecture (when applicable)

```markdown
## Design

Narrative explanation of the internal structure...
```

Use this section when the package has non-trivial internal architecture: layers, pipelines, client/server splits, or algorithmic structure that a caller needs to understand. Use subsections (`###`) for distinct layers or components. Include contract and precondition callouts where relevant.

Consider Mermaid diagrams when they clarify relationships or flows that are hard to follow in prose alone -- layer dependencies, pipeline stage sequences, or data flow between components. A diagram should supplement the text, not replace it: the prose states the facts, the diagram makes the structure visible at a glance.

Omit this section for simple packages where the API section tells the whole story (e.g., a single-struct POD types package).

### 3.8. Testing (required)

````markdown
## Testing

Tests cover:

- **Category** -- what this test category verifies.
- **Category** -- what this test category verifies.

Run from the dev container:

```bash
docker exec -u "$(id -un)" <container> bash -c \
  "cd /stack/<workspace> && bazel test //path/to:target_test"
```
````

Two parts:

1. **Coverage summary** -- a bulleted list of what the tests cover (parity, edge cases, error paths, performance, etc.). This tells a future contributor what's protected and what gaps exist.
2. **Run command** -- the exact shell command to run the tests, including the `docker exec` wrapper when running from outside the container. A contributor should be able to copy-paste this and get results.

### 3.9. Reference Documents (required)

```markdown
## Reference Documents

| Document | Purpose |
|---|---|
| [`<topic>_spec.md`](path/to/spec.md) | Design specification for this package. |
| [`<topic>_design-study.md`](path/to/design-study.md) | Alternatives analysis and trade-offs. |
| [`dependency/README.md`](path/to/dependency/README.md) | Upstream dependency. |
```

A table linking to documents a reader might need after the README: specs, plans, design studies, upstream dependency READMEs, TODO files. Links point upstream -- to documents this package depends on or was designed from. Do not list downstream consumers; they are responsible for linking back to their own dependencies. Do not list tool-mandated files (e.g., `AGENTS.md`) -- those are discovered by tooling, not by readers following document links. All references should be to documents within the same repository.

### 3.10. Table of Contents (when applicable)

For READMEs with more than five sections or significant depth (subsections with their own subsections), add a Table of Contents after the lead paragraph and before the first section:

```markdown
## Table of contents

- [1. Contents](#contents)
- [2. API](#api)
- [3. Usage](#usage)
- [4. Dependencies](#dependencies)
- [5. Build Targets](#build-targets)
- [6. Design](#design)
- [7. Testing](#testing)
- [8. Reference Documents](#reference-documents)
```

Use explicit `<a id="...">` anchors on headings when heading text contains backticks, parentheses, or special characters that different Markdown renderers handle inconsistently.

Short READMEs (five or fewer sections, no deep nesting) do not need a TOC.

---

## 4. Maintenance

### 4.1. Keep It Current

A README must reflect the current state of the code. When you change the package's API, files, build targets, or test coverage, update the README in the same commit. A stale README is worse than no README -- it actively misleads.

### 4.2. What Triggers an Update

| Change | README update |
|---|---|
| Add/remove/rename a file | Update Contents table. |
| Change public API | Update API section and Usage example. |
| Add/remove a build target | Update Usage build dependency and Build Targets table (if present). |
| Add/change test coverage | Update Testing section. |
| Add/remove a dependency on another package | Update Dependencies section. |

### 4.3. Review Discipline

README accuracy should be checked during code review. If a PR changes the package's public surface and the README is not updated, that is a review finding.

---

## 5. Principles

### 5.1. Orient First, Detail Second

The lead paragraph and Contents table are the most important parts. A reader who scans only those two sections should know what the package does and what files it contains. Everything else is progressive disclosure.

### 5.2. Show, Don't Describe

Every README should contain at least one code block that a reader can copy and use. An API section without code is a description; an API section with a code block is documentation.

### 5.3. One Package, One README

Each BUILD-file-scoped package gets exactly one README. If a directory contains multiple packages (multiple BUILD files), each gets its own README in its own directory. If a README needs to describe code across multiple directories, the code should be reorganized so the README's scope matches a single package.

### 5.4. No Forward-Looking Content

A README describes what the code *is*, not what it will become. TODOs, improvement ideas, and future work belong in `TODO.md`. A README that mixes current documentation with aspirational notes misleads the reader about the package's actual capabilities.

---

## 6. Anti-Patterns

| Anti-pattern | Problem | Fix |
|---|---|---|
| **No Contents table** | Reader must open every file to understand scope. | Add a file-to-description table. |
| **No Usage section** | Reader can't figure out how to depend on the package. | Add build dep, include path, and example. |
| **No Testing section** | Reader doesn't know what's tested or how to run tests. | Add coverage summary and run command. |
| **Missing Build Targets on complex package** | Package has many targets but no inventory; reader guesses which target to depend on. | Add a Build Targets table when the package defines more than two targets. |
| **TODO list in README** | Mixes documentation of current state with future work. Reader can't tell what works now. | Move to `TODO.md`. |
| **Stale API signatures** | Code changed but README wasn't updated. Reader writes code against a ghost API. | Update README in the same commit as the API change. |
| **No lead paragraph** | Reader has to read the whole document to understand what the package does. | Add 1-3 sentences at the top. |
| **Exhaustive header dump** | API section reproduces the entire header file. Reader drowns in detail. | Show only the types and functions a caller needs. |
| **Downstream dependent references** | Reference Documents table lists packages that *consume* this package. Creates a maintenance burden: every new consumer requires a README update in the consumed package. | Reference Documents links upstream (dependencies, standards, specs). Downstream consumers are responsible for linking back to their dependencies, not the other way around. |
| **README in wrong location** | README is not next to the BUILD file, or is inside `docs/`. | Move to the package root. |

---

## 7. Minimal Example

A README for a small CPU utility package (two targets, so no Build Targets section):

````markdown
# String Utils

Configuration parsers and log formatters across the codebase
duplicate string trimming, splitting, and case-conversion logic. This
package extracts those operations into a single shared library.

---

## Contents

| File | Description |
|---|---|
| `string_utils.h/.cc` | Trim, split, and case-conversion functions. |
| `string_utils_test.cc` | Unit tests. |

---

## API

All functions are in namespace `mylib`.

```cpp
#include "mylib/utils/string_utils.h"

/// Trims leading and trailing whitespace. Returns a view into the input.
std::string_view Trim(std::string_view input);

/// Splits on delimiter. Empty segments are preserved.
std::vector<std::string> Split(std::string_view input, char delimiter);

/// Converts to lowercase (ASCII only).
std::string ToLower(std::string_view input);
```

---

## Usage

### Build dependency

```python
deps = [
    "//mylib/utils:string_utils",
]
```

### Include path

```cpp
#include "mylib/utils/string_utils.h"
```

### Example

```cpp
#include "mylib/utils/string_utils.h"

const auto trimmed = mylib::Trim("  hello  ");  // "hello"
const auto parts = mylib::Split("a,b,,c", ',');  // {"a", "b", "", "c"}
```

---

## Dependencies

### Internal

None.

### External

None.

---

## Testing

Tests cover:

- **Trim** -- empty, whitespace-only, leading, trailing, both, no whitespace.
- **Split** -- empty input, single element, consecutive delimiters, trailing delimiter.
- **ToLower** -- ASCII letters, digits (unchanged), empty string.

Run from the dev container:

```bash
docker exec -u "$(id -un)" dev-container bash -c \
  "cd /workspace/myproject && bazel test //mylib/utils:string_utils_test"
```

---

## Reference Documents

None.
````

---

## Governing Standards

This definition conforms to the [Document Structure Standards](../standards/document-structure_standards.md). See section 3.1 for the central-vs-local placement rule that governs READMEs.
