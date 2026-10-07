# ADR 0004: Graph storage format

- **Status:** Accepted (2026-10-07, Epic 0 exit gate), with the comment-preservation requirement
- **Issue:** #6 (Epic 0, task 0.6)
- **Brief:** §7–§9, §13, §16, §19, §44, §74, §75, §83–§87, §107, §131. **Audit:** §8.3 (two route vocabularies).

## Requirements (§83)

The format must be human-readable, diffable, versionable, deterministic, easy for Claude to edit, and easy to validate. No opaque binary database.

Studio adds two requirements of its own:
- **Claude never writes graph files directly** (ADR 0003 §4). The core writes them after validation. Humans may still hand-edit them, so they must stay pleasant to read.
- A **semantic diff** by element id (Epic 1.9) matters more than a line diff.
- **Comments and formatting are kept** (user decision). Comments are documentation, not semantic state: the schema controls meaning, and YAML is the human-editable representation of it. The save pipeline must not be designed to discard comments.

## Options

| | YAML (canonical subset) | JSON | Hybrid (YAML for humans, JSON for machine files) | SQLite |
|---|---|---|---|---|
| Readable / hand-editable | best; comments kept (below) | noisy for nested contracts | mixed | no |
| Deterministic output | yes, with a canonical writer | yes | yes | no (binary) |
| Pitfalls | **implicit typing** (below) | none | two formats to maintain | rejected by §83 |
| Library | ruamel.yaml (round-trip) for graph files | stdlib | both | stdlib |

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

Studio's loader reads all of these back **as strings**: `'no'`, `'on'`, `'1.20'`, `'26.3'`, `'0x10'`, `'2026-10-07'`. Types then come **from the schema**, never from YAML's guessing.

### Comment preservation, measured

Prototype (ruamel.yaml 0.19.1, round-trip mode, with a resolver that makes every plain scalar a string, sketched in §1):

| Check | Result |
|---|---|
| Load → save an unchanged file with comments, flow mappings, a double-quoted string, `1.20`, `no`, a date, `0x10`, an empty value | **byte-identical** |
| Edit two values and insert a new key | comments, quoting and flow style elsewhere unchanged; only the edited lines differ |
| Reload after the edit | `version` → `'1.21'`, `allow` → `'yes'`: still strings |

So comment preservation and schema-authoritative types work together.

## Decision

### 1. Format

- **YAML** for every graph and project file: UTF-8, `\n` line ends, 2-space indent, block style by default. Short flow mappings (`{surface: collision, need: required}`) are allowed where they read better.
- **Not allowed** (the validator rejects them): anchors and aliases, custom tags, multi-document files. Comments, blank lines and quoting style are allowed anywhere and **preserved**.
- **Library:** `ruamel.yaml` in round-trip mode. It keeps comments, key order, quoting and flow/block style. The toolkit keeps using PyYAML; Studio's core adds ruamel.yaml.
- **Strict resolver:** every plain (unquoted) scalar resolves to a **string**, both when reading and when the writer decides whether a value needs quotes. YAML never guesses bool/int/float/null/date:

  ```python
  class StrictResolver(VersionedResolver):
      def resolve(self, kind, value, implicit):
          if kind is ScalarNode and implicit[0]:
              return super().resolve(kind, "plain", implicit)   # the str tag
          return super().resolve(kind, value, implicit)
  ```

- **Schema is authoritative:** Pydantic v2 models define every field's type. They coerce the loaded strings (`"true"` → bool, `"40"` → int, `""` → null where optional); a version is always a string. JSON Schema is exported from the models (`schemas/*.schema.json`) for editors and for Claude's `--json-schema` output.
- **Save pipeline:** the core loads the file as a round-trip document **and** as validated models. A change is applied to the models, validated, then written back **into the existing document tree** (edit in place, insert, remove), and the tree is dumped.
  - Unchanged content, including comments and formatting, is byte-identical. That is a test in Epic 1.9.
  - New elements go in at their canonical position: lists of elements sorted by `id`, keys in schema field order. Existing hand ordering is left alone, so a hand-edited file isn't reordered behind the user's back.
  - Removing an element removes the comment lines attached directly above it. Other comments stay. Removals are listed in the change's journal entry.
  - Order-meaningful lists (rule actions, sync pipeline steps) keep their given order.
- **`mashup fmt`** (opt-in) rewrites a file into canonical order and spacing, keeping comments. It never runs implicitly.
- **Comments are not semantic state.** Nothing in Studio reads meaning from a comment. Explanations that Studio must show in the UI (§27) live in `description` and `provenance.rationale` fields. Claude's proposals may include a comment for humans, but Claude doesn't write the file itself (ADR 0003 §4).
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

- Epic 1 implements: the Pydantic models, the strict round-trip loader and the in-place save pipeline, `mashup fmt`, the validator, semantic diff, dependency engine and migrations, and exported JSON Schemas. The core's dependencies are ruamel.yaml and Pydantic v2.
- Comment preservation gets dedicated tests: unchanged files are byte-identical; edits keep unrelated comments; removals report the comments they took with them.
- A CI check fails if any file in `generated/` has a `generated_from` id that doesn't resolve (`IMPLEMENTATION_EPICS.md`, standing risks).
- Hand edits to graph YAML, comments included, survive core saves.

## Resolved questions (2026-10-07)

1. Comments: **preserved where practical.** The serializer is designed to keep them, the schema controls meaning, and comments are documentation only.
