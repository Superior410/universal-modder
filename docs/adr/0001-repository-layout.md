# ADR 0001: Repository layout

- **Status:** Proposed (accept at the Epic 0 exit gate)
- **Issue:** #3 (Epic 0, task 0.3)
- **Brief:** §136, §137, §138. **Audit:** §2.7, §8, §10, decisions D2, D8, D11.

## Context

Studio needs a home for its code (core, desktop shell, runtime, tests, ADRs) that:
1. keeps the toolkit `universal-modder` pullable from upstream (`rehan-remade/universal-modder`) with no merge conflicts (§137);
2. keeps a clear boundary between **toolkit**, **application**, **runtime** and **project data** (§138);
3. fits D2: Studio calls the toolkit as a **separate process**, never as an imported library.

Facts from the audit that bear on this:
- This repo's CI runs `um publish check .` over **every file**, plus `test_skill_copies_match`. Studio code here would be linted by toolkit rules, and Studio would need the same skill-copy discipline.
- Upstream moves fast: the fork fell 5 commits behind within a day, and every one of those commits touched `knowledge/`.
- `bin/um` is a **bash** launcher. On native Windows (Studio's primary target, §81) it needs Git Bash. `uv run --project <toolkit> python -m um …` runs the same code with no bash, so that is how Studio invokes the toolkit.
- Studio will carry a desktop UI with its own toolchain (ADR 0002). Toolkit users don't need any of it.

## Options

### A. Companion repository `universal-mashup-studio` (recommended)

```text
universal-mashup-studio/            ← new repo (Studio)
  core/          Python: graph engine, project store, adapters, CLI `mashup`
  shell/         desktop UI (ADR 0002)
  runtime/       launcher, process manager, bridge library (Epic 6)
  docs/adr/      ADRs (moved here from the fork when this is accepted)
  docs/studio/   brief, build order, audit, plans
  tests/
  toolkit.lock   pinned universal-modder repo URL + commit SHA

universal-modder/                   ← this fork, kept as close to upstream as possible
  (unchanged toolkit)
  FORK_CHANGES.md                   the only list of fork-specific changes (§137)
```

- **Toolkit pin:** `toolkit.lock` records the URL (the fork) and an exact commit SHA. In development the toolkit is a sibling clone; on users' machines Studio keeps a managed clone under its data folder, at the pinned SHA. Updating the pin is a reviewed commit, and CI runs Studio's adapter tests against it.
- **Upstream sync:** the fork stays a near-mirror. Fast-forwarding from upstream is routine because Studio adds nothing to it. Toolkit improvements Studio needs (e.g. `publish check --json`, Steam `buildid` in `um scan`) go **upstream as PRs**. Until they merge, Studio parses the current output (D2), so it never waits on upstream.
- **Pros:** toolkit CI never sees Studio code; Studio CI gets its own rules (Windows job, Node/Rust toolchain); separate release cadence; the §138 boundary is enforced by the repo split itself; the license and plugin metadata stay as they are.
- **Cons:** two repos to manage; cross-repo changes take two PRs; **the user has to create the new repo** (this session can't create repositories).

### B. Monorepo: a `studio/` folder inside this fork

- **Pros:** one clone; brief, audit and code side by side; atomic changes across toolkit and Studio.
- **Cons:**
  - toolkit CI (`um publish check .`) lints every Studio file, so its UI and build files would need exclusions;
  - Studio dependencies (UI toolchain) would sit in a repo installed as an agent plugin by other people;
  - fork-specific files would multiply, so every upstream sync touches a bigger surface;
  - `.claude/skills` copy-parity rules would apply if Studio ever ships skills;
  - "Studio imports toolkit internals" stays one `import` away, against D2.

### C. Studio as a subfolder, toolkit as a git submodule *of Studio*

This is a variant of A. The submodule SHA is the pin. It's rejected because submodules are easy to get wrong on Windows and in fresh clones, while `toolkit.lock` plus an explicit fetch says the same thing more plainly. It can be reconsidered later at no cost: the lock file and a submodule pin record the same thing.

## Decision

**Option A: a companion repository**, `universal-mashup-studio`, with this fork kept as a near-mirror of upstream plus `FORK_CHANGES.md`.

Until the new repo exists, Epic 0 documents stay in this fork under `docs/` and move with the first Studio commit, keeping their history in the move commit's message.

## Consequences

- The user creates an empty repository (suggested: `Superior410/universal-mashup-studio`, private or public as they prefer) and adds it to the session. Epic 1 starts there.
- The fork's `docs/adr/`, `docs/studio/` and the three root Studio documents move to the new repo. The fork's `main` then returns to upstream plus `FORK_CHANGES.md`.
- The toolkit is invoked as `uv run --project <toolkit> python -m um <group> …` (works on native Windows without bash), with `FAL_KEY` removed from the environment (0.2).
- Project data (§44) lives **outside both repos**, in a user-chosen projects folder (default `Documents\UniversalMashupProjects`). Studio's own state lives under `%LOCALAPPDATA%\UniversalMashupStudio`. That name is deliberately different from the old prototype's `MashupStudio` (D11), so the two can't be confused or merged by accident.

## Open questions for the user

1. The new repository's name and visibility (public like the toolkit, or private while it's early).
2. Should the projects folder default to `Documents\UniversalMashupProjects` (§44's name) or somewhere you prefer?
