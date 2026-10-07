# Universal Mashup Studio — Implementation Epics

Companion to the master brief (`UNIVERSAL_MASHUP_STUDIO_BRIEF.md`). The brief is the product spec; this file is the build order.
Each epic lists issues with acceptance criteria. Claude Code should open one GitHub issue per bullet under "Issues", label it with the epic, and work them in order. **No epic starts until the previous epic's exit gate passes.**

---

## Ground rules for every issue

1. **Graph first.** If a behavior matters, it exists in the graph schema before it exists in code (brief §74, §131).
2. **Definition of done is scoped to the epic.** Early epics (no game runtime yet) are done at: schema/representation + implementation + tests + docs. From Epic 7 on, the full §149 definition applies, including real-game verification.
3. **No game files in the repo, fixtures, or test artifacts.** Fixtures use synthetic data.
4. **No cloud generation dependency.** `FAL_KEY` must never be required for any test, build, or launch path.
5. **Every issue updates the user guide** for any user-facing concept it introduces (brief §156). Help is built alongside features, not at the end.
6. **Record decisions** in `docs/adr/NNNN-title.md` when choosing between real alternatives.

---

## Epic 0 — Audit and foundational decisions

Goal: understand what exists before building anything (brief §139).

Issues:
- **0.1 Repository audit** → `UNIVERSAL_MASHUP_STUDIO_REPO_AUDIT.md` covering every item in §139 (skills, `um` CLI commands, engine playbooks, mashup skill, KB, tests, Windows tooling, publish checks). Classify each: reuse as-is / wrap / adapt / cloud-dependent / out of scope.
- **0.2 Cloud-dependency isolation plan** — list every fal/cloud touchpoint and how it gets disabled without breaking local tooling (§97).
- **0.3 ADR: repository layout** — companion repo vs monorepo (§138). Must preserve the ability to pull upstream `universal-modder` changes.
- **0.4 ADR: application stack** — evaluate against §81 priorities. Constraint to respect: the core should be callable headless (CLI + tests) so the UI is a client of the core, not where logic lives. Lean to evaluate: Python core reusing `um`, desktop shell (Tauri or Electron) over a local IPC/HTTP boundary.
- **0.5 ADR: how the app invokes Claude** — Claude Code headless mode vs Claude Agent SDK; context-packing strategy (§142); cost/latency; what happens offline. Note honestly in the ADR that "local-first" applies to generation and game data — Claude itself is a remote model call.
- **0.6 ADR: graph storage format** — YAML vs JSON vs hybrid, file split, ID scheme, `schema_version` (§83–85).

Exit gate: audit + all ADRs merged and reviewed by the user.

---

## Epic 1 — Graph core (no UI)

Goal: the source of truth exists, validates, diffs, and answers dependency questions (§140).

Issues:
- **1.1 Schema: node/edge model** — node kinds (entity, component, system, capability, event, rule, adapter, bridge) and an *open* edge-type registry (§8: users can add edge types).
- **1.2 Schema: interaction contracts** (§9), including `added_by: claude|user` + rationale + accept/reject state (§10).
- **1.3 Schema: authority graph** — per-property authority (§13) + conflict detection that reports, never auto-resolves (§15).
- **1.4 Schema: synchronization graph** (§16) incl. failure/watchdog/fallback (§17) and performance metadata (§37).
- **1.5 Schema: strategies** — per-relationship, extensible, `custom` type allowed (§11, §107).
- **1.6 Schema: rules** — trigger/conditions/actions/priority/phase (§19, §112); provenance (§87).
- **1.7 Capability identity + provenance** — stable IDs, independent versioning, origin `game|mashup|user_created` (§85–86). Mashup-as-donor must be representable *now* even though it's built in Epic 11.
- **1.8 Validator** — referential integrity, schema version, authority conflicts, orphan nodes, unsupported-status consistency.
- **1.9 Deterministic serialization + semantic diff** — stable ordering so git diffs are meaningful.
- **1.10 Dependency engine** — given a changed node/edge, return affected nodes, generated modules, and tests (§76). Generated-code metadata format (`generated_from`, §75) defined here.
- **1.11 Schema migrations** framework (§84).
- **1.12 Fixtures** — synthetic Minecraft-block→L4D2 and TNT graphs written by hand as test data.

Exit gate: CLI can `validate`, `diff`, and `affected <node>` against fixtures; full test suite green.

---

## Epic 2 — Project store, versioning, transactions

Goal: nothing destructive can happen before rollback exists. (Moved earlier than the brief's ordering on purpose — §78 transactions depend on it.)

Issues:
- **2.1 Project layout + manifest** (§44–45), portable game references, no absolute paths in shareable files (§100).
- **2.2 Snapshots / restore / diff** (§30) — git-backed is acceptable if it stays invisible to beginners.
- **2.3 Branches** (§31, §80).
- **2.4 Transaction wrapper** — apply → generate → build → test → commit-or-rollback, with "keep as experimental" (§78–79).
- **2.5 Implementation journal + audit trail** — concise rationale only, no reasoning traces (§52, §88).
- **2.6 Artifacts directory** per run (§51).

Exit gate: a scripted fake change can be applied, fail a fake test, and roll back cleanly.

---

## Epic 3 — universal-modder integration boundary

Goal: Studio calls the toolkit through one adapter, never by reaching into its internals (§136–137).

Issues:
- **3.1 `UniversalModderAdapter`** wrapping `um scan`, `um win`, `um backup`, `um kb`, asset tools, publish checks — structured results, not scraped text where avoidable.
- **3.2 Recon → graph** — recon output becomes host/donor nodes + compatibility fingerprint (§42).
- **3.3 KB read before work / write after discovery** hooks (§43).
- **3.4 Save backup integration** for every action that touches saves (§53).
- **3.5 `GameAdapter` interface** (§103) with a stub implementation and capability flags for unimplemented methods.
- **3.6 Confirmation boundaries** — loader install, game-dir modification, input driving, save changes, publishing all require explicit user confirmation (§89).

Exit gate: scanning the user's machine produces a valid project graph with real host/donor nodes.

---

## Epic 4 — Change pipeline + Claude orchestration

Goal: a graph edit or NL request becomes a scoped implementation change (§142–143).

Issues:
- **4.1 Context packer** — given affected set (from 1.10), assemble only the relevant graph slice, recon, KB entries, source files, test status.
- **4.2 Intent → graph diff** — NL request produces a *proposed* graph diff that the user reviews; graph edit and NL produce identical state (§73).
- **4.3 Autonomy modes** — guided / automatic / expert, per project (§24).
- **4.4 Generator contract** — Claude writes code only into `generated/` + adapters with `generated_from` headers; reconcile when code and graph disagree (graph wins).
- **4.5 Incremental regeneration test** — changing one property regenerates only the affected modules (§143). This is the epic's core test.
- **4.6 Hot-reload vs restart classifier** (§77).
- **4.7 Explainability endpoints** — "why this strategy / surface / authority / regeneration" answered from journal + graph (§27).

Exit gate: against a fixture project, an NL request round-trips to a reviewed graph diff, scoped regeneration, and passing tests.

---

## Epic 5 — Desktop shell + minimal graph editor

Goal: a usable editor over the core (§141). Keep it minimal.

Issues:
- **5.1 Shell** per ADR 0.4, talking to the core over its API only.
- **5.2 Project library + dashboard** with evidence-based health (§54–55, §90). No percentages that aren't computed from tests.
- **5.3 Graph canvas** — create/delete/edit nodes and edges, save/load, validation errors inline, diff view. Must stay responsive at a few thousand nodes.
- **5.4 Beginner and technical views over the same graph** (§26).
- **5.5 Claude console** — NL input, proposal review (accept/reject/modify), progress of the change pipeline.
- **5.6 Help system skeleton** — quick help tooltips, contextual help panel, searchable guide, glossary, "Explain this" wired to 4.7 (§151–155). Content grows with each later epic.

Exit gate: a user can open a project, edit the graph, ask for an NL change, review it, and see validation and diff — no game running yet.

---

## Epic 6 — Runtime, launcher, process manager

Goal: run multi-process mashups safely (§49–50, §144).

Issues:
- **6.1 Launcher** — ordered start, readiness checks, health verification.
- **6.2 Process manager** — PIDs, crash detection, no orphaned processes or stuck input on exit (§17).
- **6.3 Local IPC bridge library** — at least one transport (named pipes or localhost WebSocket) with timestamps + watchdog + graph-defined fallback.
- **6.4 Runtime monitor UI** — per-process/bridge status, event trace (§117).
- **6.5 Test harness from contracts** — generate test plans from interaction contracts (§39); test status flows back onto graph nodes (§40).

Exit gate: two dummy processes bridged locally; killing one triggers the declared fallback and the monitor shows it.

---

## Epic 7 — Vertical slice 1: Minecraft blocks in L4D2

Goal: prove *gameplay integration*, not visual presence (§124–125, §130). Follow §70 step-by-step; each step is its own issue and must be verified in-game before the next.

Issues:
- **7.0 Recon both games + strategy ADR for the slice.** Expected outcome to validate, not assume: L4D2 side is likely host-native reimplementation (VScript and/or SourceMod on a local/listen server), with Minecraft acting as the *semantic and asset* donor from the user's own install — not passthrough. Passthrough is a poor first slice here.
- **7.1 One cube renders** at a fixed position.
- **7.2 Player placement** (raycast → snapped grid position).
- **7.3 Player collision.**
- **7.4 Infected collision.**
- **7.5 Navigation** — infected path *around* blocks, not just bump into them. Recon must determine which nav-blocking mechanism L4D2 actually supports at runtime; record the finding in the KB. Gate the issue on an automated reroute test (§39 example).
- **7.6 Destruction** (configurable per block type) with collision + nav recovery on removal.
- **7.7 Persistence** within what the host allows; mark unsupported parts honestly (§36, §38).
- **7.8 NL edit on the live slice**, e.g. "make stone blocks indestructible" → scoped regen → retest.
- **7.9 Guide + interactive tutorial** built from this slice (§151.4), using a safe lab profile.

Constraints: offline / local server only, no VAC-protected online play with modified content (§67).

Exit gate: the §125 chain verified in-game with recorded evidence artifacts.

---

## Epic 8 — Rules, NL editing depth, rule library

- **8.1 Rule engine runtime** — deterministic ordering, phases, conflict resolution (§58, §112).
- **8.2 Rule editor UI** (trigger/conditions/actions).
- **8.3 Local rule library** — save, browse, re-apply with compatibility check + adaptation (§20).
- **8.4 User-created capabilities and systems** (§21–22).
- **8.5 "Why doesn't this work?" diagnostics** over traces + rules + authority (§118).

## Epic 9 — Vertical slice 2: TNT in L4D2

- Per §126; each branch (fuse, explosion, damage, impulse, block destruction, nav update) as a separate issue with its own strategy, and at least one deliberate authority change mid-slice to exercise §14.

## Epic 10 — Sharing and import

- **10.1 Package builder** — recipe only; publish checks from universal-modder block game files (§46, §3.3).
- **10.2 Importer** — manifest → detect games → fingerprint → capability negotiation → local build → tests (§47, §108).
- **10.3 Version-mismatch detection + scoped migration** (§48, §109).

## Epic 11 — Mashup as donor

- **11.1 Export a single capability/rule/system from a project** without the whole project (§32).
- **11.2 Vertical slice 3** — capability from Minecraft×L4D2 consumed by a Skyrim project (§127).
- **11.3 Capability packs** (§59–60).

## Later (design must not preclude; don't build yet)

Multiplayer and dedicated server (§61–62), multiple instances of one game (§34), overlays, community sharing.

---

## Standing risks to track as issues

- **Scope**: the brief describes years of work. Hold the line on exit gates.
- **Engine reality vs. graph promises**: the graph must be able to say "unsupported" for anything recon can't verify (§38). Never let a status show green from compilation alone (§90).
- **Game updates** silently breaking adapters — fingerprint on every launch, not just import.
- **Generated code drift** — CI check that every file in `generated/` has valid `generated_from` metadata pointing at existing graph elements.
- **Upstream divergence** from universal-modder — keep fork-specific changes listed in one file.
