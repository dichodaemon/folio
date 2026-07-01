# Report Definition

Definition for writing report documents. A report is a point-in-time descriptive document. It surveys the current state of a topic — a research area, a codebase or group of codebases, a market, or findings on a particular question — and presents structured analysis. Its value is in the synthesis: organizing raw data into a framework, surfacing patterns, and drawing evidence-backed conclusions.

---

## 1. Purpose

A report exists to:

1. **Survey** — capture the current state of a landscape at a point in time.
2. **Structure** — organize findings around a taxonomy, dimensions, or analytical framework so the reader understands the lens before seeing the data.
3. **Analyze** — surface patterns, trade-offs, and notable divergences across the surveyed subject.
4. **Conclude** — synthesize observations into evidence-backed conclusions. Conclusions may state preference or assessment, but do not prescribe actions.

A report is NOT:

- **A brief.** A brief frames work before commitment. A report describes findings without framing next steps. See [Brief Definition](brief_definition.md).
- **A design study.** A design study decomposes a problem, evaluates alternatives, and recommends a decision. A report may inform a design study, but it does not make the recommendation. See [Design Study Definition](design-study_definition.md).
- **A reference.** A reference records settled facts intended to be maintained. A report is a point-in-time snapshot that is not updated as the landscape changes. See the document structure standards for reference placement.
- **A living document.** A report captures understanding at the time of writing. If the landscape changes, write a new report — do not retrofit the old one.

---

## 2. When to Write One

Write a report when:

- You need to survey a landscape — existing implementations, market options, research findings — before deciding how to proceed.
- You want to preserve the analysis that went into understanding a topic, so future readers can see what was known and when.
- The subject has enough breadth to benefit from structured organization and analytical synthesis, not just a flat list.

Do not write a report when:

- You are ready to recommend a decision. Write a design study that references the report as its evidence base.
- The subject is a single artifact or narrow fact. A reference entry is sufficient.
- The data will be actively maintained. Choose a living document type (reference, arch-design) instead.

---

## 3. Structure

Every report follows this skeleton. All sections are required unless marked otherwise.

### 3.1. Metadata (required)

```markdown
---
title: <Topic>
date: YYYY-MM-DD
author: <git user name>
---
```

| Field | Purpose |
|---|---|
| `title` | Short, descriptive name. Matches the `#` heading. |
| `date` | Creation date in `YYYY-MM-DD` format. Must be the actual calendar date when the document is created. Must match the date prefix in the filename. Immutable after creation. |
| `author` | Who wrote this report. Use `git config user.name`. |

No `status` field. A report does not have a lifecycle — it is written once and captures findings at a point in time. It is not updated as the landscape changes.

**File naming:** `YYYY-MM-DD_<topic>_report.md`. The date is the creation date. Example: `2026-06-29_ralph-implementations_report.md`.

### 3.2. Summary (required)

One paragraph abstract. The reader should understand what was surveyed, the scope, and the key takeaway without scrolling further.

```markdown
## 1. Summary

Comprehensive survey of every Ralph loop implementation on the Pi package registry
as of 2026-06-29. 22 packages matched; 16 are genuine implementations. Four
architectural families emerged: flat single-agent, external subprocess, multi-agent
session, and LLM chained. Only two implementations support per-agent model
selection.
```

### 3.3. Scope (required)

What is covered and what is excluded. Use a table or two short lists. Define the boundaries clearly — which items, time period, dimensions of analysis, and what was deliberately left out.

```markdown
## 2. Scope

| In scope | Out of scope |
|---|---|
| All Pi packages matching "ralph" on pi.dev as of 2026-06-29 | OMP or carrel internals |
| Multi-agent spawning, coordination, progress evaluation | Installation UX, documentation quality |
| Agent/model configuration mechanisms | Performance benchmarks |
```

### 3.4. Sources (required)

Where the data came from. Name specific URLs, repositories, versions, dates of access. This section exists so the reader can verify findings and reproduce the survey.

```markdown
## 3. Sources

1. Pi package registry: `https://pi.dev/packages?name=ralph` (accessed 2026-06-29)
2. Source repositories for each implementation:
   - pi-ralph-wiggum: `https://github.com/tmustier/pi-extensions` (branch: main)
   - ralpi: `https://github.com/mikefreno/ralpi` (branch: master)
   - ...
```

### 3.5. Findings (required)

The body of the report. Organization is determined by the subject, but must follow the "structure before listing" principle: establish the framework (taxonomy, dimensions, categories) before enumerating instances.

Common patterns:

- **Taxonomy + catalog**: Introduce classification dimensions, then present each surveyed item within that framework.
- **Comparison matrix**: Present dimensions and items in a structured table with per-cell analysis.
- **Thematic analysis**: Group findings by theme or pattern, with supporting evidence under each.

Content within Findings is **strictly factual and verifiable**:

- Every claim must be anchored to a specific artifact: a file path, code excerpt, version number, URL, or date.
- Pros and cons are valid when grounded in evidence ("37 contributors," "no tests in the repository," "exponential backoff with configurable retry count") — not when subjective ("better design," "cleaner code").
- Opinions, preferences, and assessments belong in Conclusions, not in Findings.

```markdown
## 4. Findings

### 4.1. Taxonomy

[Classification framework before individual entries...]

### 4.2. Implementation Details

#### 4.2.1. pi-ralph-wiggum

[Per-item analysis following the taxonomy...]
```

### 3.6. Conclusions (required)

Synthesis, evaluation, and perspective drawn from the findings. This is the one section where analysis and assessment belong. Conclusions may:

- Identify patterns and themes that cut across the surveyed items.
- Evaluate trade-offs with reasoned preference ("X outperforms Y on criterion Z based on the evidence in §4").
- Call out notable absences, gaps, or surprising results.
- Provide context for how the findings might inform downstream work.

Conclusions are **not prescriptive**. They do not say "we should use X" or "the team must adopt Y." Those belong in a design study or brief that references this report as its evidence base. Conclusions say "X is the most mature approach currently available" or "the landscape lacks a solution for Z."

```markdown
## 5. Conclusions

### 5.1. Cross-Cutting Patterns

[Patterns observed across multiple surveyed items...]

### 5.2. Notable Gaps

[What's missing from the landscape...]

### 5.3. Implications

[What the findings mean in context — without prescribing actions...]
```

---

## 4. Principles

### 4.1. Write Once

A report captures findings at a point in time. It is not updated as the landscape changes. If the landscape changes significantly enough that the conclusions would differ, write a new report and archive the old one.

### 4.2. Structure Before Listing

Establish the analytical framework — taxonomy, dimensions, categories — before enumerating findings. A reader should understand the lens through which the data is organized before encountering individual items. A flat list without structure is a data dump, not a report.

### 4.3. Factual Body

Everything outside Conclusions is anchored in verifiable evidence. File paths, code excerpts, version numbers, dates, URLs. Pros and cons are valid when grounded in objective facts — not when they are aesthetic judgments or unsubstantiated preferences.

### 4.4. Conclusions Are Analysis

Conclusions are the synthesis layer: patterns, trade-offs, assessments, and perspective. They may state preference if supported by the findings. They are not prescriptive — they inform decisions but do not make them. A conclusion that says "we should" has crossed into design study territory.

---

## 5. Anti-Patterns

| Anti-pattern | Problem | Fix |
|---|---|---|
| **Report as design study** | Ends with a specific recommendation or prescribed action. | Move the recommendation to a design study. Keep the report as the evidence base. Reference the report from the design study's Sources. |
| **Report as living reference** | Updated over time instead of capturing a snapshot. | Write a new report if the landscape changes. Archive the old one. |
| **Unsourced claims** | Body asserts evaluations or comparisons without citing specific artifacts. | Anchor every claim to a source. Save subjective assessment for Conclusions. |
| **Framework-free survey** | Data presented without an organizing taxonomy, dimensions, or analytical lens. | Establish the framework before enumerating findings. The taxonomy section earns its place. |
| **Conclusions untethered from findings** | States preferences or assessments not supported by the body. | Every conclusion must be traceable to specific findings. Cross-reference the relevant sections. |
| **Subjective pros/cons in body** | "Better API," "cleaner code," "more intuitive" without objective grounding. | Ground pros and cons in measurable facts: contributor count, test coverage, documented retry behavior, supported platforms. |

---

## 6. Relationship to Other Document Types

| Document type | Relationship |
|---|---|
| **Brief** | A report may be cited as a source in a brief. The brief frames work; the report provides the evidence landscape that motivated it. |
| **Design study** | A report may be the primary evidence base for a design study. The study's Sources section references the report; the study's evaluation draws on the report's findings and conclusions. |
| **Spec** | A report does not directly feed a spec. The chain is: report → design study → spec (or arch-design) → plan. |
| **Reference** | A report and a reference both capture facts. A reference is maintained; a report is a snapshot. When facts from a report prove durable, extract them into a reference. |
| **Cookbook** | Practical discoveries during research (tool quirks, workarounds) belong in a cookbook entry, not in the report. |

---

## 7. Minimal Example

```markdown
---
title: Widget Framework Landscape
date: 2026-07-01
author: Jane Smith
---

# Widget Framework Landscape

## 1. Summary

Survey of five open-source widget frameworks for the dashboard migration
project as of 2026-07-01. All five support React 19; three support server-side
rendering. Solid.js-based frameworks lead on bundle size (3-8 KB vs 15-40 KB
for React-based). None provide built-in accessibility audit tools.

## 2. Scope

| In scope | Out of scope |
|---|---|
| Open-source frameworks on npm with 1k+ weekly downloads | Proprietary or internal frameworks |
| Bundle size, SSR support, a11y tooling, TypeScript support | Runtime performance benchmarks |
| Latest stable release as of 2026-07-01 | Pre-release or beta versions |

## 3. Sources

1. npm registry: `https://www.npmjs.com` (accessed 2026-07-01)
2. Framework repositories:
   - widgetium: `https://github.com/widgetium/core` (v3.2.1)
   - dashkit: `https://github.com/dashkit/dashkit` (v2.0.4)
   - ...

## 4. Findings

### 4.1. Dimensions

The survey evaluates each framework along four dimensions:
- **Rendering strategy** — client-only vs. SSR-capable
- **Bundle cost** — minified + gzipped size of a minimal "hello world" widget
- **TypeScript support** — types bundled vs. DefinitelyTyped vs. none
- **Accessibility** — built-in audit tools, ARIA compliance, keyboard navigation

### 4.2. Per-Framework Analysis

#### 4.2.1. Widgetium (v3.2.1)

- **Rendering**: Client-only. No SSR primitives. (source: `docs/ssr.md` explicitly states "not supported")
- **Bundle size**: 18 KB gzipped (source: Bundlephobia, 2026-07-01)
- **TypeScript**: Bundled types, strict mode. (source: `package.json` `"types": "./dist/index.d.ts"`)
- **Accessibility**: No built-in audit tools. ARIA attributes documented per-component. (source: `docs/a11y.md`)
- **Pros**: Largest community (37 contributors, 2.1k GitHub stars). Comprehensive component library (47 widgets).
- **Cons**: No SSR path without wrapper. Bundle cost is 2-5x the Solid.js alternatives.

#### 4.2.2. DashKit (v2.0.4)

[Additional entries...]

## 5. Conclusions

### 5.1. SSR Is Still Emerging

Only three of five frameworks support SSR, and two of those describe it as
"experimental." For a dashboard that must render above-the-fold content
server-side, this narrows the field considerably.

### 5.2. Accessibility Is a Gap

No surveyed framework provides built-in accessibility audit tooling. Teams
must bring their own (axe-core, Lighthouse CI). This is a market gap, not a
differentiator among the options.

### 5.3. Solid.js Leads on Bundle Cost

The two Solid.js-based frameworks are 3-8 KB versus 15-40 KB for React-based
alternatives. For a widget embedding use case where every kilobyte counts,
this is the decisive dimension.
```
