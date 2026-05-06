---
title: Carrel Slot-Based Composition
status: draft
date: 2026-05-05
author: Dizan Vasquez
---

# Carrel Slot-Based Composition

## 1. Problem Statement

Carrel's current composition model is convention-driven: entries are grouped by capability type, and output files are placed at paths derived from those conventions. A `TypeZshConfig` entry named `zshrc` produces `config/zsh/zshrc.zsh`, which deploys to `<consumer>/.omp/config/zsh/zshrc.zsh`. This works for OMP configuration, where every output lands under `.omp/`. It does not work for OS tool configuration, where outputs must land at fixed OS paths (`~/.zshrc`, `~/.config/nvim/init.lua`).

Three root causes:

1. The convention controls both "which entries compose together" and "where the output lands." These are distinct concerns that should vary independently.
2. The core composer depends on conventions at runtime. It reads `entry.Type` and calls `Conventions[typ].DefaultPrimitive` to resolve the composition operator. `CapabilityType` conflates identity ("I am a rule") with behavior ("compose me with override").
3. The term "primitive" is ambiguous — it means "composition operator" but clashes with programming language primitives and OMP primitives.

This study introduces the **output slot** abstraction: a consumer-specific deployment target with a destination path, a composition mode, and member entries. Slots replace the implicit `(type, name)` grouping with an explicit, convention-agnostic model that handles both OMP and OS configuration uniformly.

## 2. Current State

### 2.1. Convention-driven composition

The compose function (`internal/composer/compose.go`) groups entries by `CapabilityType`, then sub-groups by name collision. For each `(type, name)` group, the effective composition mode is resolved by checking per-entry override, then falling back to the capability type's default mode from conventions. An `OutputFile` is produced with a destination path derived from the convention's directory and file naming function. For example, a rule named `no-push` in the `rules/` directory produces `rules/no-push.md`. The deployer prepends `<consumer>/.omp/` to make the path absolute.

This means:
- Entries of different types can never compose into the same output file.
- The destination path is rigidly derived from the type convention.
- The composer reads `entry.Type` on every entry and calls `Conventions[typ].DefaultMode` to resolve the composition mode. `CapabilityType` is both an identity tag and an implicit composition operator — type/operator conflation. The core should never read `entry.Type` to determine behavior.
- There is no way to say "this composed output lands at `~/.zshrc`."

### 2.2. OS config gap

OS tool conventions are defined in `internal/registry/conventions.go` for `TypeZshConfig`, `TypeNvimConfig`, `TypeWeztermConfig`, and `TypeP10kConfig`. They use the same `Dir`/`FileName` pattern as OMP types, producing paths like `config/zsh/zshrc.zsh` that are meaningless as deployment destinations.

The `carrel os-setup` and `carrel host-setup` commands are stubs that print placeholder text and return. No OS config deployment exists.

### 2.3. What works

- The entry registry model (UUID, type, source, relative path) is sound.
- Override and concatenation modes are correct and tested.
- The deployer handles collision policies and deployment claim recording correctly.
- Source scanning (`carrel scan`) discovers entries by convention and is independent of composition.

### 2.4. Glossary

**Source.** A directory or repository providing configuration. Registered in the registry with an alias, path, scope, and kind. A source contains files organized by convention.

**Entry.** A single configuration file from a source, registered in the registry. An entry has a UUID, a source (`SourceID`), a capability type (a scanning tag, never read by the core composer), a name, a relative path within the source, a content hash, a compose mode, and an optional final flag. Entries are source-side only — they represent source files, not deployed output.

**Consumer.** A target that receives composed configuration. Registered in the registry with an alias, path, kind (repo, container, host), and a deployment root. Consumers own slots; each consumer has its own set of slots.

**Deployment claim.** A record of a deployment attempt, stored in `deployments` and `deployment_entries`. The claim records which source entry was deployed to which path, with a content hash for verification.

**Convention.** The mapping from capability type to source directory layout, file naming, and default compose mode. Conventions live exclusively in the scanning layer (`carrel scan`) and default-slot generation. The core libraries are convention-agnostic — they never look up conventions at runtime.

**Slot.** A consumer-specific output target with a UUID, name, destination path (relative to the consumer's deployment root), an optional compose mode, and member entries via an `entry_slots` join table. Every slot belongs to exactly one consumer. Slots replace the implicit `(type, name)` grouping as the composition controller.

**Compose mode.** The operator governing how entries combine within a slot: `override` (deepest scope wins, unless final) or `concatenation` (append in source order). Formerly called "primitive" — renamed to avoid ambiguity.

## 3. Settled Commitments

These are decided before the study begins:

- **Entries carry types for scanning only.** `CapabilityType` remains on entries as a discovery and organization tag. The core composer and deployer never read `entry.Type` to determine behavior. The type is a scanning concept; the composition operator is `entry.ComposeMode`. This fixes the current conflation where `TypeRule` implies override and `TypeAppendSystem` implies concatenation. Scan owns the type→mode mapping; the core sees only the operator.
- **Override and concatenation are the compose modes.** These two modes remain. Reference and inheritance are registry/deployment concepts, not composition operators.
- **Slots are consumer-specific.** Every slot belongs to exactly one consumer (`slots.consumer_id` is NOT NULL). No universal slots. A consumer has its own set of slots. Two consumers that happen to need the same output structure get identical slots generated independently by convention-driven defaults. The core never resolves slot scope or inheritance — it queries slots WHERE consumer_id = ?.
- **Slots are registry objects.** Slots have UUIDs, stored in a `slots` table. An N:M join table (`entry_slots`) maps entries to slots. The convention system generates default slots and assignments per consumer at scan/bootstrap time. The core composes whatever slots and mappings exist.
- **Core/convention separation.** This is a project-level design principle. The core registry, composition, and deployment libraries are never aware of conventions. Conventions (capability types, directory layouts, file naming, source scopes, default compose modes) live exclusively in the scanning layer and CLI commands. The core API takes consumers, slots, entries, and mappings and processes them without interpreting what they represent. The composer never reads `entry.Type`, never looks up conventions, never resolves scope. The deployer does not switch on consumer kind.
- **Convention-driven scanning is preserved.** `carrel scan` discovers entries by walking source directories, matching conventions, and materializing the results as registry entries with fully-resolved fields (compose mode, relative path). Default slots and entry assignments can be generated per consumer.
- **Deployment claims are preserved.** The deployer still records `deployment_entries` with `(path, contentHash, sourceEntryID)`. Slot identity augments this.
- **Registry is the authority.** Slots and entry-to-slot mappings are stored in the registry, not in source manifests or derived at runtime.

## 4. Dimensions

Slots are consumer-specific registry objects with an N:M entry-to-slot mapping. These are settled designs — no viable alternatives exist under the core/convention separation principle. The remaining dimensions are:

### 4.1. Compose Mode Resolution

**Definition.** Where the effective compose mode comes from. The core composer cannot look up convention defaults. The mode must be available on the data the core receives.

#### Option 1: Entry-carried mode

Every entry has a mandatory `compose_mode` field. Scan populates it from the type's convention default. Slots do not carry a mode. The composer uses each entry's mode within that entry's slot — entries in the same slot may have different modes.

- **Properties:** `entries.compose_mode` is NOT NULL. An entry's mode is set once at scan time. The `compose_mode` column replaces the current `primitive_override` (nullable) with a mandatory value.
- **Pros:** Simple. No mode ambiguity. Scan owns the mapping from type to mode.
- **Cons:** Entries in the same slot can have different modes. A slot containing both a rule entry (override) and an APPEND_SYSTEM entry (concatenation) would apply different composition logic to different entries within the same slot — the output behavior is undefined for heterogeneous modes. The semantic meaning of the slot ("compose these entries") is weakened.

#### Option 2: Slot-carried mode

Every slot has a mandatory `compose_mode` field. All entries in a slot are composed with the slot's mode, regardless of what mode each entry might carry from its type. Scan sets the slot's mode from convention defaults when generating default slots. Entries do not carry a `compose_mode` field.

- **Properties:** `slots.compose_mode` is NOT NULL. One mode governs all entries in a slot. Per-entry mode configuration is no longer possible.
- **Pros:** Unambiguous. A slot's mode is its contract. All entries in a slot compose the same way.
- **Cons:** Loses the existing per-entry mode-override feature. A source that wants a specific rule to concatenate rather than override must use a separate slot.

#### Option 3: Entry-carried with slot override

Entries have a mandatory `compose_mode` field (populated by scan from conventions). Slots have an optional `compose_mode` field. If the slot specifies a mode, it overrides all entry modes within that slot. If `slots.compose_mode` is NULL, each entry's own mode applies.

- **Properties:** `entries.compose_mode` is NOT NULL. `slots.compose_mode` is nullable. Resolution: slot mode wins if set; otherwise per-entry.
- **Pros:** Flexible. Default behavior is per-entry. A slot can unify the mode for cases where heterogeneous modes are undesirable.
- **Cons:** Two layers of mode resolution. Debugging requires checking both the slot and the entry.

**Observations.** Option 2 is clearest but removes a useful escape hatch. Option 3 matches the established default-with-override pattern. Option 1 allows heterogeneous modes, which produces undefined output behavior. Option 3 is recommended.

### 4.2. Destination Path Resolution

**Definition.** How a slot's destination path is specified and resolved. The core deployer must not embed knowledge about consumer kinds.

#### Option 1: Consumer-provided deployment root

The slot stores a relative path (e.g., `rules/no-push.md`, `.zshrc`). The consumer carries a `deploy_root` field. The deployer resolves `<consumer.deploy_root>/<slot.dest_path>`. Bootstrap sets `deploy_root` per consumer kind: repo consumers use `<consumer.path>/.omp`, container consumers use `/home/dev`, host consumers resolve from environment.

- **Properties:** One new field on consumers (`deploy_root`). Slots are relative. Resolution is pure string concatenation — no kind-specific logic in the deployer.
- **Pros:** Clean separation: slots define relative paths; consumers define the root. Composable, testable, no switch-on-kind.
- **Cons:** A consumer that needs two roots (e.g., some files under `.omp/`, some elsewhere) needs two consumer registrations or per-slot path overrides.

#### Option 2: Slot stores absolute path

The slot stores an absolute path (e.g., `/home/dev/.zshrc`). No `deploy_root` needed on consumers. The deployer writes to the path directly.

- **Properties:** One path per slot, independent of consumer. No path resolution logic.
- **Pros:** Simplest deployer. Works for OS configs.
- **Cons:** Ties a slot to a specific environment. A slot with path `/home/dev/.zshrc` is only valid for container consumers. Slots cannot be shared across consumer kinds, which undermines convention-driven default generation.

**Observations.** Option 1 is the only option that allows convention-driven defaults to generate the same slot structure for different consumer kinds (each consumer resolves via its own `deploy_root`). Option 1 is recommended.

## 5. Design Options

Two composed options. Both use consumer `deploy_root` for destination resolution. Both satisfy core/convention separation. Option B is recommended.

| Dimension | A: Minimal | B: Full |
|---|---|---|
| Compose mode resolution | Entry-carried (option 1) | Entry with slot override (option 3) |
| Destination resolution | Consumer deploy_root | Consumer deploy_root |

### Option A: Minimal

**Concrete representation.** `slots(id, consumer_id, name, dest_path)`. `entries.compose_mode` (NOT NULL, replaces `primitive_override`). `entry_slots(entry_id, slot_id)`. Compose iterates slots, resolves member entries, uses `entry.compose_mode`. No slot-level mode. Convention-driven defaults generate slots and mode values at scan time.

**Files changed:**
- `internal/registry/schema.go` — schema v3: `slots`, `entry_slots`, `deploy_root` on consumers; `entries.compose_mode` NOT NULL (replaces `primitive_override`)
- `internal/registry/types.go` — Slot, EntrySlot types; Entry.ComposeMode (was PrimitiveOverride)
- `internal/registry/registry.go` — Slot CRUD; ResolveSlots(consumerID)
- `internal/registry/conventions.go` — `DefaultSlot`, `DefaultMode` per type (used by scan, not core)
- `internal/composer/compose.go` — iterate slots, resolve entry_slots, use entry.compose_mode; remove type-based grouping and convention lookups
- `internal/deployer/deploy.go` — use consumer.deploy_root
- `internal/scanner/discover.go` — set entry.compose_mode from convention default; generate default slots per consumer
- `cmd/carrel/main.go` — slot CRUD, os-setup/host-setup
- `cmd/carrel/bootstrap.go` — set deploy_root, generate default slots per consumer

**Pros.** Minimal new columns. Entry-level modes preserved. No slot-level mode to manage.

**Cons.** Entries in the same slot can have different modes. Heterogeneous modes within a slot produce undefined output behavior — a slot containing both override and concatenation entries has no clear contract.

### Option B: Full

**Concrete representation.** Same as Option A, plus: `slots.compose_mode` (nullable). If set, it overrides entry modes within that slot.

**Files changed:** Same as Option A, plus `internal/composer/compose.go` — mode resolution: `slot.compose_mode ?? entry.compose_mode`.

**Pros.** Entry-level modes work by default. Slot-level mode unifies heterogeneous slots when needed. Core logic is simple: one conditional.

**Cons.** Two layers of mode resolution. One additional nullable column.

## 6. Evaluation

| Criterion | A: Minimal | B: Full |
|---|---|---|
| Core/convention separation | Satisfied | Satisfied |
| OS config deployment | Yes | Yes |
| Cross-type composition | Yes (entry_slots, type ignored) | Yes |
| Composition mode clarity | Weaker (heterogeneous per slot) | Stronger (slot can unify) |
| Implementation cost | ~9 files | ~10 files |
| Schema complexity | 2 new tables, 2 new columns, 1 altered column | Same + 1 nullable column |
| Migration | primitive_override → compose_mode (NOT NULL) | Same |

**Recommendation.** Option B (Full).

Rationale:
1. The slot-level mode override solves the semantic ambiguity of Option A. A slot is a coherent output target; a slot-level mode makes that coherence explicit and enforceable.
2. Both options satisfy core/convention separation. The core takes data from the registry without convention lookups. `entry.Type` is never read by the composer.
3. The type/operator conflation is resolved: `CapabilityType` is a scanning tag; `ComposeMode` is the operator. Scan populates `ComposeMode` from type conventions; the core reads only `ComposeMode`.
4. Slots are always consumer-specific. No universal slots, no scope resolution, no nullable `consumer_id` — `WHERE consumer_id = ?` is the only query pattern. Convention-driven defaults generate identical slots per consumer, achieving the same result as "universal" without complicating the core.
5. The migration from `primitive_override` (nullable) to `compose_mode` (NOT NULL) is straightforward: existing entries without an override get their type's convention default materialized by the migration or by a re-scan.

## 7. Open Questions

1. **Default slot naming.** Should default slot names match entry names (slot `zshrc` for entries named `zshrc`), type names (slot `zsh-config` for TypeZshConfig entries), or be derived another way? The naming scheme must handle collisions: two entries of different types with the same name. What is the default slot for a rule named `context` and a context-file also named `context`?
2. **`primitive_override` to `compose_mode` migration.** Should the migration set `compose_mode` to the convention default for entries where `primitive_override` is NULL? Or should scan be re-run to recompute all mode values from current conventions?
3. **Backward compatibility of `carrel run`.** Currently `carrel run` composes and deploys all entries to `.omp/`. With slots, does it compose all slots for the consumer? Does it need a `--slot=<name>` filter? Does `carrel os-setup` compose only OS-type slots and `carrel run` compose only OMP-type slots, or is the split by slot names?
4. **`deploy_root` for existing consumers.** Existing repo consumers need `deploy_root = <consumer.path>/.omp`. Migration must backfill this. For future container/host consumers: `/home/dev` / `$HOME`.
5. **`final` flag semantics with slots.** If a slot carries its own mode, does the `final` flag on an entry still prevent deeper-scope entries from contributing? Or does the slot's mode override all per-entry flags?
6. **Deployment claims and slot identity.** Should `deployment_entries` gain a `slot_id` column? Enables querying "what slots produced this consumer's deployed files." Trade-off: more schema vs queryability.

## 8. Deliverables

This study outputs:
- A slot-based composition model with compose mode resolution, destination path resolution, and consumer-specific ownership.
- The analysis of type/operator conflation and its resolution via core/convention separation.

Downstream documents that consume this study:
- `carrel_arch-design.md` — updated to reflect the slot model, revised compose/deploy pipeline, and core/convention separation.
- Schema v3 specification — `slots`, `entry_slots`, `deploy_root`, and altered `entries.compose_mode`.
- Beads issues for implementation: schema migration, compose_mode materialization, slot CRUD, compose rewrite, deployer update, scan default generation, os-setup/host-setup.


## Appendix: Library Impact Summary

Changes required per package when implementing Option B.

### `internal/registry`

**Schema (schema.go):** Schema v3. New tables: `slots(id, consumer_id, name, dest_path, compose_mode)`, `entry_slots(entry_id, slot_id)`. New column on consumers: `deploy_root TEXT NOT NULL`. Altered column on entries: `compose_mode INT NOT NULL` replaces nullable `primitive_override`. `Conventions` map gains `DefaultSlot` and `DefaultMode` per type — used exclusively by the scanner, never imported by composer or deployer.

**Types (types.go):** New `Slot` and `EntrySlot` structs. `Entry.PrimitiveOverride` replaced by `Entry.ComposeMode` (mandatory). `Consumer` gains `DeployRoot` field. `ComposeMode` type alias for existing `Primitive` type — same values, renamed. Remove `CapabilityType` references from compose/deploy packages (types remain, but imports removed).

**Interface (registry.go):** New methods: `RegisterSlot`, `UpdateSlot`, `RemoveSlot`, `ResolveSlots(consumerID)`, `LinkEntrySlot`, `UnlinkEntrySlot`, `ResolveEntrySlots(slotID)`. Existing `ResolveEntries` and `RegisterEntry` unchanged. `RegisterConsumer` now requires `DeployRoot`.

**Dolt/Mem implementations:** Mirror the interface changes. `ResolveSlots` is `SELECT * FROM slots WHERE consumer_id = ?`. `ResolveEntrySlots` returns entries via the join table.

**Conventions:** `Conventions` map gains `DefaultSlot: "rule"` and `DefaultMode: ModeOverride` (or `ModeConcatenation`) per type. Used by scanner. Never imported by composer, deployer, or query.

### `internal/composer`

**Imports removed:** `github.com/dichodaemon/carrel/internal/registry` (convention types).

**Structs:** `OutputFile` gains `SlotID uuid.UUID` and `Mode ComposeMode`. `DestinationPath` is now the slot's resolved path (with deploy_root prefix).

**Compose function:** Signature changes: `Compose(consumer, slots, entrySlots)` — no more sources or entries parameters. Iterates slots. For each slot, resolves member entries from `entrySlots`. Determines effective mode: `slot.ComposeMode ?? firstEntry.ComposeMode`. Applies override or concatenation. No type grouping, no `CapabilityType` references, no convention lookups, no source scope ordering.

**Removed:** `effectivePrimitive`, `composeOverride`, `composeConcatenation`, `resolveWinner`, `convFileName`, `sortBySourceOrder`.

### `internal/deployer`

**Imports removed:** None.

**Structs:** `DeployResult` unchanged.

**Deploy function:** Destination paths come from `OutputFile.DestinationPath` (pre-resolved with `consumer.DeployRoot`). No path derivation. Collision detection and cleanup unchanged. Deployment claim recording gains `SlotID` on `DeploymentEntry`.

**Removed:** Any consumer-kind branching for path resolution.

### `internal/scanner`

**Discover function:** Unchanged (workspace discovery).

**New:** `GenerateDefaultSlots(reg, consumer) error` — creates default slots for a consumer based on convention `DefaultSlot` fields. `AssignDefaultEntries(reg, consumer) error` — assigns scanned entries to the consumer's default slots based on convention type→slot mappings. Entry assignment orders entries within each slot by source scope (universal before target-specific). This is convention logic living exclusively in the scanner.

**ScanSource:** Updated to set `entry.ComposeMode` from `convention.DefaultMode`. Updated to call `GenerateDefaultSlots` and `AssignDefaultEntries` after scanning if a consumer context is provided.

### `internal/authoring`

**AddEntry:** Sets `entry.ComposeMode` from convention default instead of leaving `PrimitiveOverride` nil. Otherwise unchanged.

**EditEntry, RemoveEntry, RenameEntry, UpdateEntryMeta:** Unchanged (operate on entries by UUID). Slot assignments managed separately via slot CRUD.

### `internal/query`

**ListFiles (sources):** Unchanged (lists entries with on-disk status).

**ListDeployed:** Gains `slot_id` in output if `deployment_entries` is extended.

**Plan:** Simplified — composes via slots rather than type grouping, but output format unchanged.

**Dependents/Feeds/Trace:** Unchanged at source/consumer level. May gain slot-level variants (e.g., `carrel trace --slot=<name>`).

**New:** `ListSlots(consumerID)`, `ListSlotEntries(slotID)` query methods.

### `internal/tui`

**Dashboard:** Gains a Slots tab showing consumer's slots, their member entries, and compose mode.