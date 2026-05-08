# Cookbook Definition

Reference for writing cookbook documents. A cookbook is a flat collection of standalone, numbered entries — also called recipes — each one a self-contained tip, workaround, procedure, or finding discovered during development. It is not a tutorial, not a guide, not a reference manual. Each entry exists because someone hit a problem, found the answer, and wrote it down so the next person (or agent) doesn't have to re-derive it.

---

## 1. Purpose

A cookbook exists to:

1. **Capture** — record practical findings that don't belong in a spec, plan, or reference doc but are too useful to lose.
2. **Answer quickly** — each entry should take a reader from "I have this problem" to "here's the command" in under 30 seconds.
3. **Accumulate** — entries are appended over time as new findings emerge. The document grows but never needs restructuring.

A cookbook is NOT:

- A reference manual. If the topic needs subsections, tables of options, or exhaustive coverage, it belongs in a dedicated reference doc.
- A tutorial or guide. If the content requires sequential reading or builds on previous sections, it belongs in a skill or a spec.
- A plan or spec. If the content describes what to build or how to build it, it belongs in those documents (see [Specification Document Reference](spec-doc_definition.md), [Implementation Plan Reference](impl-plan_definition.md)).

---

## 2. Structure

### 2.1. Document Header (required)

```markdown
# <Domain> Cookbook

Miscellaneous findings, tips, and workarounds discovered during development.
```

**File naming:** `<domain>_cookbook.md`, using lowercase with underscores. The domain is the working context the cookbook serves. Examples: `docker_cookbook.md`, `git_cookbook.md`, `omp_cookbook.md`.

**Title:** `# <Domain> Cookbook`, using the domain name in natural case. The title must match the filename's domain slug. Examples: `# Docker Cookbook`, `# Git Cookbook`, `# OMP Cookbook`.

The subtitle is always the single line above. This consistency makes cookbooks immediately recognizable across directories.

### 2.2. Entry Format (required)

Each entry (also called a recipe) is a numbered section separated from its neighbors by horizontal rules:

```markdown
---

## N. Entry Title

<body>

---
```

| Element | Requirement |
|---|---|
| Separator (`---`) | Before and after every entry. Creates visual breaks and makes entries scannable. |
| Heading (`## N. Title`) | Sequential integer. Title is a short noun phrase describing the topic, not the problem. E.g., "Local .gitignore equivalent" not "How to exclude files without modifying .gitignore." |
| Body | See section 2.3. |

Numbering is append-only. Entries are never deleted or renumbered. If an entry becomes obsolete, mark it deprecated (see section 2.6) — do not remove it. This keeps external references (other docs, agent context, bookmarks) stable.
### 2.3. Entry Body (required)

Every entry body follows the same three-part structure, in order:

1. **Context** — one to three sentences explaining *when* you'd need this and *why* the obvious approach doesn't work. This is the "I have this problem" hook.
2. **Solution** — the command, configuration, code snippet, or procedure that solves it. Fenced code blocks for anything executable. Inline code for paths and identifiers.
3. **Explanation** (when applicable) — why the solution works, caveats, alternatives considered, or gotchas. Only include this when the solution is non-obvious or has edge cases.

Do not add sub-headings within an entry. If an entry needs sub-headings, it has outgrown the cookbook format and should be extracted into its own document.

### 2.4. Code Blocks

- Use fenced code blocks with a language tag when the content is executable: ` ```bash `, ` ```python `, etc.
- Use bare fenced blocks (no language tag) for paths, output examples, or non-executable content.
- Inline annotations within code blocks (comments explaining specific lines) are preferred over prose explanations after the block.

### 2.5. Tables

Tables are allowed within entries when listing structured information (file contents, directory layouts, option mappings). Keep them short — a table with more than 8 rows suggests the entry should be a reference doc instead.


### 2.6. Deprecation

When an entry becomes obsolete (tool removed, API changed, better approach documented elsewhere), mark it deprecated by adding a bold notice immediately after the heading:

```markdown
## 3. Querying the old metrics endpoint

**Deprecated:** Metrics endpoint was removed in v2.4. See entry 12 for the replacement.

<original body preserved below>
```

Keep the original body intact — it may still be useful as historical context or for teams on older versions. The deprecation notice tells the reader to stop here; the preserved body tells them what used to work and why it no longer does.
---

## 3. Principles

### 3.1. One Problem, One Entry

Each entry addresses exactly one thing. If an entry says "and also," it should be two entries. The reader is scanning headings to find their problem — a combined entry hides one of them.

### 3.2. Concrete Over Abstract

Every entry should contain at least one code block, command, file path, or configuration snippet that the reader can copy and use. An entry that explains a concept without showing how to apply it belongs in a skill or reference doc, not a cookbook.

### 3.3. Context Before Solution

Never start an entry with a command. Always state when and why first. A command without context is a magic incantation — the reader can't adapt it to their situation and can't tell when it's the wrong answer.

### 3.4. Self-Contained Entries

Each entry must be understandable without reading any other entry. No "as described in entry 2 above." If two entries share context, repeat the minimum necessary context in each.

### 3.5. Append-Only Growth

New entries go at the bottom with the next sequential number. Do not insert entries between existing ones or reorganize by topic. The document is a log of findings, not a taxonomy.

If the cookbook grows beyond ~15 entries, consider whether a subset of entries share enough domain context to justify a dedicated reference doc. Extract them as a group, replace them with a single entry pointing to the new doc, and continue appending.

---

## 4. Scope: One Cookbook Per Domain

Each domain gets at most one cookbook. A domain is a working context — a project area (`frontend`), a cross-cutting concern (`general`), or a tool (`git`, `bazel`, `omp`). The domain name drives both the filename and the title (see section 2.1).

The choice of domain granularity is a judgment call. Start broad (one cookbook per directory). Split into tool-specific cookbooks only when a single cookbook accumulates enough entries around one tool that scanning past them becomes friction for readers looking for unrelated entries.

---

## 5. Anti-Patterns

| Anti-pattern | Problem | Fix |
|---|---|---|
| **Entry without code** | Reader can't act on it. Becomes a vague note. | Add the command, snippet, or path. If there's nothing concrete to show, it's not a cookbook entry. |
| **Multi-topic entry** | Heading matches one problem, body covers two. Second problem is invisible to a scanner. | Split into separate numbered entries. |
| **Tutorial entry** | Entry assumes sequential reading or builds on previous steps. | Extract to a skill or guide. Replace with a one-entry summary pointing to it. |
| **Reference entry** | Entry exhaustively lists options, flags, or configuration keys. | Extract to a reference doc. Keep a short cookbook entry for the most common case. |
| **Stale entry** | Tool, path, or API described no longer exists. | Mark deprecated with a notice after the heading (see section 2.6). Do not delete or renumber. |
| **Entries without context** | Starts with a command. Reader doesn't know when to use it. | Add 1-2 sentences before the solution explaining the problem. |

---

## 6. Minimal Example

````markdown
# OMP Cookbook

Miscellaneous findings, tips, and workarounds discovered during development.

---

## 1. Checking if an OMP extension loaded

The extension list isn't shown in the UI by default. To verify your extension was discovered and loaded at startup:

```bash
# From within OMP, use the /status command or check the LSP status
/status
```

Look for your extension name in the "Extensions" section. If it's missing, check:
- The file is in `~/.omp/agent/extensions/` or `<cwd>/.omp/extensions/`
- The file exports a default factory function
- The filename doesn't collide with a disabled extension in `config.yml`

Extensions are discovered at startup only — a restart is required after adding new files.

---

## 2. Preventing git hooks from running on the host

Pre-commit hooks in the project repo require Python dependencies only available inside the Docker container. Running `git commit` on the host fails with import errors.

```bash
# Skip hooks for a single commit (use sparingly)
git commit --no-verify -m "message"

# Permanent fix: run git operations inside the container
docker exec -u $(whoami) dev-container bash -c 'cd /workspace && git commit -m "message"'
```

The `--no-verify` flag is a workaround, not a solution. The hooks exist for a reason — prefer running inside the container.

---
````

---

## Governing Standards

This definition conforms to the [Document Structure Standards](../standards/document-structure_standards.md). See section 2 for file naming conventions.