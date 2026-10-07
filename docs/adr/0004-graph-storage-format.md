# ADR 0004: Graph storage format

- **Status:** Proposed (accept at the Epic 0 exit gate)
- **Issue:** #6 (Epic 0, task 0.6)
- **Brief:** §7–§9, §13, §16, §19, §44, §74, §75, §83–§87, §107, §131. **Audit:** §8.3 (two route vocabularies).

## Requirements (§83)

The format must be human-readable, diffable, versionable, deterministic, easy for Claude to edit, and easy to validate. No opaque binary database.

Studio adds two requirements of its own:
- **Claude never writes graph files directly** (ADR 0003 §4). The core writes them after validation. Humans may still hand-edit them, so they must stay pleasant to read.
- A **semantic diff** by element id (Epic 1.9) matters more than a line diff.

## Options

| | YAML (canonical subset) | JSON | Hybrid (YAML for humans, JSON for machine files) | SQLite |
|---|---|---|---|---|
| Readable / hand-editable | best (comments allowed, but see below) | noisy for nested contracts | mixed | no |
| Deterministic output | yes, with a canonical writer | yes | yes | no (binary) |
| Pitfalls | **implicit typing** (below) | none | two formats to maintain | rejected by §83 |
| Already a dependency | PyYAML (toolkit) | stdlib | both | stdlib |

### The YAML pitfall, measured

PyYAML (YAML 1.1) with `safe_load`, on values Studio will actually store:

```text
a: no          → False        (an "allow" flag silently flips)
b: on          → True
c: 1.20        → 1.2          (Minecraft 1.20 becomes 1.2: a wrong version)
d: 26.3        → 26.3 (float)
e: 0x10        → 16
f: 2026-10-07  → datetime.date
```

A loader with the implicit bool/int/float/timestamp resolvers removed reads all of these back **as strings**: `'no'`, `'on'`, `'1.20'`, `'26.3'`, `'0x10'`, `'2026-10-07'`. Studio uses that loader. Types then come **from the schema**, never from YAML's guessing.

## Decision

### 1. Format

- **YAML, restricted to a canonical subset:** block style, UTF-8, `\n` line ends, 2-space indent.
- **No** anchors, aliases, tags, multi-document files, flow collections longer than one line, or comments in machine-written files. A human can add comments; on the next core save they are dropped, with a warning in the UI. Long-lived explanation belongs in `description` fields, not comments.
- **Loader:** `yaml.SafeLoader` with the implicit bool/int/float/timestamp resolvers removed, so every scalar loads as a string.
- **Schema:** Pydantic v2 models are the source of truth. JSON Schema is exported from them (`schemas/*.schema.json`) for editors and for Claude's `--json-schema` output. The models coerce strings to the declared types (`"true"` → bool, `"40"` → int); a version is always a string.
- **Writer:** the core writes every file through one canonical serializer:
  - mapping keys in **schema field order**, with unknown/extension keys after them, sorted;
  - lists of elements sorted by `id`;
  - order-meaningful lists (rule actions, sync pipeline steps) kept in their given order;
  - strings quoted only when needed to round-trip as strings.
  
  Load → save of a canonical file is **byte-identical**. That is a test in Epic 1.9.
- JSON is still used for **machine-only** artifacts (recon output, test results, journal records, the context bundle), where nobody hand-edits and the toolkit already emits JSON.

### 2. File split (§44)

```text
<project>/
  project.yaml            identity, schema_version, autonomy mode, toolkit pin
  manifest.yaml           games (portable ids + exact versions), capabilities used, network flags
  graph/
    capabilities.yaml     capability nodes (WHAT): identity, provenance, surfaces
    integration.yaml      entities, components, systems, events, adapters, bridges; edges
    contracts.yaml        interaction contracts (§9)
    authority.yaml        per-property authority assignments (§13)
    synchronization.yaml  sync links + failure/watchdog/fallback (§16–§17, §37)
    strategies.yaml       per-relationship strategy choices + rationale (§11–§12)
    rules.yaml            declarative rules (§19, §112)
    registry.yaml         project-defined node kinds, edge types, strategy types (open registries)
```

- Each graph file starts with `schema_version` and `kind`.
- A project may split any file into a folder (`graph/rules/*.yaml`) once it grows. The loader treats a folder and a single file the same way.
- Dependencies (§76) are **derived** from edges, contracts and `generated_from` metadata at load time, not stored. A stored `dependencies.yaml` (§44) would go stale. If Claude or the user need to pin an extra dependency, it's an explicit `depends_on` edge.

### 3. Identifiers (§85)

- **Capability ids** are global and stable: `<namespace>.<name>[.<sub>]`, lowercase `[a-z0-9_]`, e.g. `minecraft.block`, `minecraft.tnt`, `l4d2.infected`. Namespace = game id (`minecraft`, `l4d2`, `skyrim`), the `mashup` namespace for capabilities exported from a project (`mashup.zombie_fortress.barricade`), or `user` for user-created ones. Capabilities are versioned independently with semver strings (§85).
- **Project element ids** (entities, adapters, contracts, rules, sync links, edges) use the same character set, unique within the project, with a kind prefix for readability: `contract.block_blocks_infected`, `rule.tnt_knocks_tank`, `edge.0f3a…`. Edges may use generated ids because humans address them by endpoints.
- **Ids never change.** Display text lives in `label`. A rename is a new id plus an `aliases:` entry on the element, so old references and `generated_from` headers keep resolving. The validator reports any reference that only resolves through an alias.
- **Game ids** are portable (`l4d2`, `minecraft`). Install paths are resolved locally and never stored in shareable files (§100); they live in Studio's per-machine state.

### 4. `schema_version` and migrations (§84)

- An **integer** `schema_version` in `project.yaml` and every graph file. It starts at `1`.
- Migrations are pure functions `vN → vN+1` over the loaded data, each with a fixture-based test (Epic 1.11).
- A file newer than the running Studio is refused with a clear message, never "best-effort" loaded.

### 5. Two route vocabularies (audit §8.3)

**Strategies** describe how a capability is implemented in a mashup:
- built-ins: `content_port`, `host_native_reimplementation`, `passthrough`, `embedded_library`, `full_reimplementation`, `headless_simulation`, `hybrid`, `custom`;
- the list is open (§107): projects register more in `registry.yaml`.

The toolkit KB's `route` describes how a mod reaches a game: `data | asset-only | loader-api | managed-patch | native-hook | reimplementation | decomp-recomp | passthrough | emulator | other`.

**Both are kept.** A strategy records the route(s) it uses (`routes: [loader-api]`). That lets the KB be searched by strategy, and lets field notes written from Studio fill in `route` correctly.

### 6. Example (illustrative; field names are finalized in Epic 1)

`graph/contracts.yaml`:

```yaml
schema_version: 1
kind: contracts
contracts:
  - id: contract.block_blocks_infected
    label: Minecraft blocks stop the infected
    source: {capability: minecraft.block}
    target: {entity: l4d2.infected}
    interaction: blocks_movement
    surfaces:
      - {surface: collision, need: required}
      - {surface: navigation, need: required}
      - {surface: ai, need: useful}
    events: [block.placed, block.destroyed]
    provenance:
      added_by: claude
      rationale: Block geometry changes traversable space, so navigation must update.
      state: proposed
```

`graph/authority.yaml`:

```yaml
schema_version: 1
kind: authority
assignments:
  - {subject: minecraft.block, property: collision, holder: game.l4d2}
  - {subject: minecraft.block, property: existence, holder: mashup_runtime}
  - {subject: minecraft.block, property: navigation, holder: game.l4d2}
```

Generated code header (§75), with comment syntax per language:

```text
// generated_from: contract.block_blocks_infected, rule.stone_indestructible
// graph_hash: 9c1e…  (hash of the referenced elements, for drift detection)
```

## Consequences

- Epic 1 implements: the Pydantic models, strict loader, canonical writer, validator, semantic diff, dependency engine and migrations, and exported JSON Schemas. The core's dependencies are PyYAML (shared with the toolkit) and Pydantic v2.
- A CI check fails if any file in `generated/` has a `generated_from` id that doesn't resolve (`IMPLEMENTATION_EPICS.md`, standing risks).
- Hand edits to graph YAML are supported, but the next core save canonicalizes them, and their comments are dropped with a warning.

## Open questions for the user

1. Is dropping comments on save acceptable, given that `description` fields hold lasting notes? (The alternative, ruamel.yaml round-trip, keeps comments but makes byte-identical output much harder.)
