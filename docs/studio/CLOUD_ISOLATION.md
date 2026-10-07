# Cloud-dependency isolation plan

**Status:** Approved (2026-10-07, Epic 0 exit gate). Issue: #2 (Epic 0, task 0.2). Brief §3.1, §97, §98, §99, §135. Audit §5, decisions D2 and D3.

**Goal:** Universal Mashup Studio runs with no cloud generation service, and never needs `FAL_KEY`. Isolation is done by **configuration at the Studio boundary**. No upstream file in `universal-modder` is deleted or edited (§137), and the toolkit keeps working unchanged for people who use it outside Studio.

**Decision (D3):** fal is **excluded** from Studio. It isn't an opt-in setting, and it is never a fallback (§98).

**Scope:** the requirement is that *Studio* doesn't use cloud generation. It is **not** to remove fal from `universal-modder`. The toolkit's fal tooling stays intact and usable outside Studio; Studio just never invokes or exposes it.

---

## 1. Every touchpoint and how Studio isolates it

| # | Touchpoint | Where | How Studio isolates it | Upstream change |
|---|---|---|---|---|
| T1 | `um fal` CLI group | `um/fal.py` (`um/cli.py` imports it at start-up) | `UniversalModderAdapter` has an **allow-list** of `um` groups: `scan`, `win`, `backup`, `kb`, `sprite`, `render3d`, `comfy`, `video`, `publish`. A request for `fal` is rejected before any process starts. The import at start-up is harmless: it needs no key and makes no network call (CI runs `um fal --help` keyless). | none |
| T2 | fal MCP server, Claude Code config | `.mcp.json` (project), plugin install `.mcp.json` | Studio's Claude sessions start with `--strict-mcp-config --mcp-config <studio-mcp.json>`. Claude Code then uses only the servers in Studio's file and ignores the project, user and plugin MCP configs. Studio's file has no fal entry. | none |
| T3 | fal MCP server, other agents | `mcp.json`, `.codex/config.toml`, `.cursor/mcp.json`, `.vscode/mcp.json`, `gemini-extension.json`, `opencode.json` | Not read by Studio: it drives Claude Code only. Listed so the audit is complete. | none |
| T4 | `fal-assets` skill | `skills/fal-assets/` and its copies in `.claude/skills/`, `.agents/skills/` | Defence in depth, three layers: (a) Studio sessions run in a **Studio-owned working directory**, not the toolkit clone, and get toolkit skills from a curated folder that leaves out `fal-assets` (§2); (b) the deny rule `Skill(skill:fal-assets)`; (c) the deny rules `mcp__fal` and `mcp__fal__*` in case any config still reaches the session. | none |
| T5 | fal guidance inside other skills | `mod-any-game` (7 mentions), `asset-pipeline` (6), `publish-mod` (4), `case-studies.md` (3), `showcase-video` (2), `mashup-mods` (1), `safety.md` (1) | Studio's appended system prompt (§3) overrides them: assets are local only. The skills stay readable; their non-fal content is the reason Studio loads them. | none |
| T6 | Example asset scripts | `examples/aoe2-de-civ/assets/gen.sh`, `examples/aoe2-de-civ/civ.py`, `examples/terraria-tmodloader/assets/make_art.sh` | Never executed by Studio. Examples are reference reading only (audit §2.6). | none |
| T7 | `FAL_KEY` in the environment | the user's shell or a `.env` | The process launcher **removes `FAL_KEY` and `FAL_KEY_FILE`** from the environment of every child process it starts (`um`, `claude`, build tools). A stray key on the machine therefore can't be picked up. | none |
| T8 | fal credit lint | `um/publish.py` (`fal_manifest.jsonl` → credit warning) | Kept. It only reads files, and it catches fal assets imported from elsewhere. | none |

## 2. What a Studio Claude session looks like

Studio launches the user's installed Claude Code CLI as a subprocess (ADR 0.5). Every launch includes:

```text
claude -p
  --output-format stream-json --verbose
  --strict-mcp-config --mcp-config <studio>/runtime/mcp.json      # T2: no fal
  --disallowedTools "mcp__fal" "mcp__fal__*" "Skill(skill:fal-assets)"   # T4 (b, c)
  --append-system-prompt-file <studio>/runtime/studio-rules.md    # T5, §3
  --add-dir <studio>/runtime/toolkit-skills                       # curated skills, T4 (a)
```

- **Working directory:** the mashup project folder, never the toolkit clone. In `-p` mode Claude Code connects the servers in a project's `.mcp.json` without a trust prompt, so the toolkit clone's `.mcp.json` must not be the session's project. `--strict-mcp-config` already blocks it; keeping the cwd elsewhere is the second layer.
- **Bare mode can't be used.** `--bare` would skip discovery, but it never reads the subscription login and needs an API key. Studio uses the user's Claude subscription (D7), so it relies on the flags above instead.
- **Curated toolkit skills:** at toolkit-pin time, Studio copies `skills/*` **except `fal-assets`** into `<studio>/runtime/toolkit-skills/.claude/skills/`, records the toolkit SHA, and refreshes the copy when the pin changes. These skills load through the `project` setting source, so Studio must not pass `--setting-sources` without `project`.
- **User-level plugins:** if the user has also installed the `universal-modder` plugin globally, its `fal-assets` skill could appear as `universal-modder:fal-assets`. The `Skill(skill:fal-assets)` deny rule matches a skill under any of its names; the verification test (§4) proves it.

## 3. Studio rules appended to every session (`studio-rules.md`, excerpt)

```text
Asset generation in Universal Mashup Studio is LOCAL ONLY.
- Never use fal, fal.ai, `um fal`, or any paid or remote generation service, even if a skill suggests it.
- Allowed, in this order: the user's own installed game assets (converted, never redistributed),
  local converters (`um sprite`, `um render3d`), local ComfyUI (`um comfy`, localhost only),
  procedural generation, clearly marked placeholders.
- If none fits, stop and ask the user. Cloud generation is never a fallback.
- Network use that is not generation (KB sync, web research, downloading a loader from its official
  release page) must be stated in the journal entry for the run.
```

## 4. Verification (becomes tests in Epic 3 / Epic 4)

| Test | Pass condition |
|---|---|
| `adapter_rejects_fal` | `UniversalModderAdapter.run("fal", ...)` raises before spawning a process |
| `child_env_has_no_fal_key` | with `FAL_KEY` set in the parent, every spawned child's environment lacks `FAL_KEY` and `FAL_KEY_FILE` |
| `session_has_no_fal_mcp` | a real Studio session's `system/init` event lists no MCP server named `fal` and no `mcp__fal*` tool, **even when the cwd contains a `.mcp.json` with fal** and the user has the toolkit plugin installed |
| `session_has_no_fal_skill` | `system/init` lists no `fal-assets` skill under any name; asking the session to "use the fal-assets skill" ends with a permission denial, never a fal call |
| `toolkit_works_without_key` | with no `FAL_KEY`: `um scan --list --json`, `um kb search x --json`, `um backup list x`, `um sprite --help`, `um render3d --help`, `um comfy --help`, `um publish check <fixture>` all exit as expected |
| `status_reports_local_only` | the UI and `mashup status --json` report `"asset_generation": "local-only"` |

## 5. User-visible reporting (§97, §99)

- The project dashboard shows **Asset generation: Local only**. The value comes from the Studio config constant; there is no toggle to change it.
- Each journal entry lists any network use in that run, by category: Claude model calls, KB sync, web research, tool downloads. Nothing is uploaded silently: Claude calls carry only the context the packer selected (ADR 0.5), and the journal records that it was sent.

## 6. Open questions

1. **Not settled by the docs; check with a spike:** whether `--setting-sources` filters `~/.claude/skills` and project `.claude/skills` the same way it filters `--add-dir` skills. The design doesn't depend on the answer, because the deny rule covers it; the spike just tells us which layer actually does the work.
2. Other generators the user installs locally later (e.g. local 3D or audio models) can be added as local providers. Anything that calls a remote API stays excluded by the same allow-list rule.
