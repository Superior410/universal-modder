# Universal Mashup Studio — Repository Audit

Issue: #1 (Epic 0, task 0.1). Brief: `UNIVERSAL_MASHUP_STUDIO_BRIEF.md` §139. Build order: `IMPLEMENTATION_EPICS.md`.

## 0. Scope and method

| | |
|---|---|
| Audited tree | `Superior410/universal-modder` @ `baff1e5` (fork of `rehan-remade/universal-modder`), **in sync with upstream main**. The first pass was at `6c02e77`, 5 commits behind; the difference was knowledge-base notes only, so no finding changed. |
| Tests | `uv run --with pytest pytest -q tests`: **76 passed, 1 skipped**, 1 Pillow deprecation warning (`Image.getdata`, removed in Pillow 14, 2027-10). `um kb check --index`: 74 notes pass. Run on Linux; Windows-only paths are not exercised. |
| Method | Read every `um/*.py` module's command surface and library functions, both PowerShell tools, all 10 skills, the mashup skill and the Source and Minecraft playbooks in full, the agent and plugin configs, the hooks, CI, and the KB notes closest to the first vertical slice. File paths are cited for each claim. |

Classification used throughout:
- **reuse as-is**: call it unchanged.
- **wrap**: call it through the Studio adapter, which adds structured I/O and error handling.
- **adapt**: needs a change. It goes upstream if generally useful, otherwise into a Studio extension.
- **cloud-dependent**: needs a paid or remote generation service. It must be isolated (§97).
- **out of scope**: not used by Studio.

---

## 1. Summary

1. **The repository is a toolkit plus a body of knowledge, not a runtime.** It has an excellent recon CLI, Windows automation, backups, a publish lint and a 74-note knowledge base. The brief's mashup "strategies" (§11) exist here as **documentation** (`skills/mashup-mods/SKILL.md`, Patterns 1–5) and as **one-off example code** (`examples/minecraft-gta5-passthrough`). There is no reusable bridge, IPC, watchdog, launcher or process-manager library. The brief (§3.2, §11.3, §136) assumes "local runtime/bridge concepts" can be orchestrated. They exist only as concepts, so Epic 6 builds them from scratch, using the example as a reference design.
2. **Most of `um` is importable as a library, but it exits the process on errors.** Every module routes failures through `um.common.die()` → `sys.exit()` (`um/common.py:78`). Several commands print text and give no structured return (`publish.check` prints and returns an exit code; `backup.restore` prints and requires `yes=True`). The Studio adapter must either catch `SystemExit` and capture stdout, or get small upstream changes that add structured results. Recommendation: **wrap first, upstream later** (§7).
3. **Cloud generation is wired into every agent config and into the core skill loop.** The fal MCP server is pre-registered in seven config files. `mod-any-game` step 6 tells agents to "generate with `um fal`". Nothing *requires* `FAL_KEY` for any non-fal command: `um scan/win/backup/kb/sprite/render3d/publish/comfy/video` all work without it, and CI runs every group's `--help` without a key. Isolation is a config and instruction problem, not a code-surgery problem (§5, input to 0.2).
4. **Recon has three gaps that matter for the Minecraft × L4D2 slice.** VAC is not detected (it isn't file-based). Minecraft Java installs from the official launcher are not discovered by `um scan --list`. No game **build version** is reported for Source or Minecraft. Version fingerprinting (§42, §47, §48) needs all three (§9).
5. **The knowledge base already holds the closest prior art to Epic 7:** `knowledge/games/portal-2/portalcraft-minecraft-inside-portal-2.md`, a working Minecraft passthrough into a Source 1 game. It documents VScript-spawned invisible solid boxes for Minecraft blocks (`prop_dynamic`, `solid 2`, `rendermode 10`, `SetSize`), Source↔Minecraft unit and axis mapping, BSP voxelisation and `-insecure` launching. Also `knowledge/games/left-4-dead/left-4-dead-infected-and-tank-in-minecraft.md` (Source model, animation and sound converters, the reverse direction). Neither covers **L4D2 navigation-mesh blocking**, which remains the slice's main unknown.

---

## 2. Inventory and classification

### 2.1 `um` CLI (`um/`, Python ≥3.10, deps Pillow/numpy/PyYAML; launcher `bin/um` uses `uv`)

Entry point `um/cli.py` imports all ten groups at start-up (`GROUPS`, line 10). Each module exposes `register(sub)`. Library functions sit beside the CLI handlers.

| Group | Commands | Library surface (what an adapter can call) | Platform | Network | Class |
|---|---|---|---|---|---|
| `scan` (`um/scan.py`, 758 lines) | `scan <game>`, `--list`, `--json` | `all_games()`, `resolve_game()`, `scan(query) -> dict` (engine, confidence, evidence, anti-cheat, loaders, mod folders, saves, exes, ranked routes, warnings, playbook path) | all (Steam/Epic/Xbox discovery on Win/WSL/Linux/macOS) | none | **wrap** + **adapt** (gaps §9) |
| `win` (`um/win.py`, 534) | `setup`, `ps`, `kill <pid>`, `launch`, `shot`, `record`, `drive`, `reg get/set` | functions shell out to `um/ps1/*.ps1` and ffmpeg | Windows / WSL only (dies elsewhere, line 44) | `setup` downloads ffmpeg from GitHub (SHA-256 checked) | **wrap** |
| `backup` (`um/backup.py`, 169) | `create`, `list`, `diff`, `restore` | `create() -> Path`, `snapshots()`, `diff() -> dict`, `restore()` (prints; needs `yes`; auto pre-restore snapshot) | all | none | **wrap** (save-data safety, §53). Not suitable as the *project* versioning engine (zip snapshots of folders, sha1 manifests) |
| `kb` (`um/kb.py`, 484) | `search`, `show`, `new`, `check`, `index`, `sync`, `pr` | `resolve_root()`, `search(root, terms, game, engine, route) -> list[dict]`, `check_note()`, `new_note()`, `build_index()`; `knowledge/index.json` is a machine-readable index (74 entries) | all | `sync` reads GitHub (`UM_KB_REPO`, default `rehan-remade/universal-modder`); `pr` pushes a branch and opens a PR | **wrap** (`search`/`show`/`check`/`new`); `pr` behind a confirmation boundary (§89) |
| `publish` (`um/publish.py`, 127) | `check <mod> [--game]` | `check() -> int` (prints FAIL/WARN lines; no structured result) | all | none | **adapt** (needs a structured result for the package builder, Epic 10) |
| `sprite` (`um/sprite.py`, 447) | cutout, trim, fit, pixelate, outline, palette, sheet, slice, frames, team mask, seamless, info… | pure Pillow/numpy functions | all | none | **reuse as-is** (asset pipeline, §64) |
| `render3d` (`um/render3d.py` + `um/blender/render_sprites.py`) | render a GLB to sprite frames from a game camera preset | `render()` shells out to Blender | all (needs Blender) | none | **reuse as-is** |
| `comfy` (`um/comfy.py`, 309) | status, checkpoints, txt2img, run workflow | `status()`, `txt2img()`, `generate()`; writes a JSONL record per output | all | **localhost only** (`127.0.0.1:8188` default, `COMFYUI_URL`) | **reuse as-is**: the sanctioned local generator (§98) |
| `video` (`um/video.py`, 580) | probe, contact, first-frame, beats, mux, compile (EDL) | ffmpeg wrappers | all (ffmpeg) | none | **reuse as-is** for evidence and showcase; low priority for Studio |
| `fal` (`um/fal.py`, 505) | recipes (sprite, texture, PBR, model3d, rig, sfx, music, voice, video), search, schema, price, upload | REST to `queue.fal.run`, `rest.fal.ai`, `v3.fal.media`, `api.fal.ai` | all | **remote, paid** | **cloud-dependent** → isolate |

Shared helpers in `um/common.py`: WSL↔Windows path mapping (`to_win`, `to_posix`), `data_dir()` (`$UM_HOME` or `~/.universal-modder`), `run()`, `die()`, `emit()`. **Reuse**, and note `die()` (above).

### 2.2 Windows tooling (`um/ps1/`)

| File | What it does | Class |
|---|---|---|
| `WinDrive.ps1` (256) | Line-per-command stdin protocol for one game window: focus, rect, click, drag, key, hold, rel mouse, type, wheel, scan-code mode, resize client area, `idle` (seconds since the user's last input). Sends input **only while the target is foreground**. C# 5 embedded, compiled by PowerShell 5.1. | **wrap**: the input-driving surface behind the §89 confirmation boundary. `idle` is the right primitive for "is the user at the PC?" |
| `ProcLoopback.ps1` (179) | WASAPI per-process loopback audio capture, timestamped raw f32 | **reuse as-is** (evidence recording) |

### 2.3 Skills (`skills/*/SKILL.md`, copied to `.claude/skills` and `.agents/skills`; `tests/test_um.py::test_skill_copies_match` fails if the copies differ)

| Skill | Role for Studio | Class |
|---|---|---|
| `mod-any-game` (+ `references/`) | The 10-step loop (intake → recon → route → lab → source of truth → vertical slice → assets → verify → showcase → publish → field note). It maps onto the brief's §71 loop. | **adapt**: Studio wants the loop, minus step 6's fal default and with the graph as the journal (§8) |
| `mashup-mods` | Patterns 1–5 = content port, passthrough, embedded decomp, reimplement/fuse, headless guest rules. Exactly the §11 strategy set, plus guardrails. | **reuse as-is** as Claude context for strategy selection (§12); source text for strategy descriptions in the graph |
| `game-recon` | Produces `MODDING_PLAN.md` (install, engine, anti-cheat, saves/config/logs, community route, chosen route, lab plan, unknowns) | **wrap**: Epic 3.2 turns its output into graph nodes; `MODDING_PLAN.md` becomes an artifact |
| `reverse-engineering` | Decompilers, data-format round trips, live memory, RenderDoc | **reuse as-is** (Claude context) |
| `game-automation` | Launch/see/drive; scripted scenes; **in-game JSON-lines agent bridge** pattern (`examples/terraria-tmodloader/reference/AgentBridge.cs`) | **reuse as-is**; the bridge pattern is the template for test oracles in Epic 6.5 |
| `asset-pipeline` | Learn the target format, 2D/3D→sprite, textures, guest Unity asset conversion | **reuse**, but strip the fal-first framing (6 fal mentions) |
| `publish-mod` | Lint, packaging per platform, README, versioning | **wrap** (Epic 10); 4 fal mentions (credits for fal assets) |
| `share-field-notes` | KB search before work, note at the end, PR with permission | **wrap**: §43 read-before / write-after hooks |
| `showcase-video` | Repeatable takes, record, EDL edit | **out of scope** for the MVP (optional later) |
| `game-research-websearch` | Archive-aware web research | **reuse as-is** (Claude context; network, but not generation) |
| `fal-assets` | fal MCP / `um fal` recipes | **cloud-dependent** → exclude from Studio sessions |

### 2.4 Engine playbooks (`skills/mod-any-game/references/engines/`, 13 files)

`unity`, `unreal`, `godot`, `source`, `native`, `dotnet-xna`, `bethesda`, `big-frameworks`, `misc-engines`, `minecraft`, `genie-aoe2`, `retro-decomp` (plus `safety.md`, `case-studies.md`). `um scan` returns the playbook path for the detected engine (`report["playbook"]`).
- **Class: reuse as-is** as the knowledge behind the engine layer (§102). They are prose, not code: the `GameAdapter` implementations (Epic 3.5) are new code that *cites* them.
- For the slice: `source.md` covers VScript (`scripts/vscripts/*.nut`), SourceMod/Metamod on servers you run, `-insecure` for local testing. `minecraft.md` covers Fabric/NeoForge, the version-triple lock, a separate launcher profile with its own `gameDir`, and the rule that Minecraft assets are downloaded per user and never redistributed.

### 2.5 Knowledge base (`knowledge/`)

- 74 notes, with a schema enforced by `um kb check` (front matter: `kind, title, game, engine, route, status, date, agents`; required sections setup/route/verification/gotchas; no secrets, no pasted decompiles, size caps). The `route` vocabulary (`data | asset-only | loader-api | managed-patch | native-hook | reimplementation | decomp-recomp | passthrough | emulator | other`) overlaps the brief's strategy list but is **not the same set** (§8).
- `knowledge/index.json` is machine-readable, which suits the context packer (Epic 4.1).
- Directly relevant notes: `games/portal-2/portalcraft-minecraft-inside-portal-2.md`, `games/gta-v/minecraft-passthrough.md`, `games/left-4-dead/left-4-dead-infected-and-tank-in-minecraft.md`, `games/minecraft/bloons-td-6-in-minecraft.md` (headless guest sim, Pattern 5), `techniques/oracles-how-agents-know-a-mod-works.md`, `techniques/reading-source-engine-models-and-animations.md`, `techniques/driving-real-games-safely.md`.
- **Class: wrap** (read) and **wrap behind confirmation** (write, PR).

### 2.6 Examples (`examples/`)

| Example | Value to Studio | Class |
|---|---|---|
| `minecraft-gta5-passthrough` | The only real passthrough implementation: Fabric mod (`HostLink`, `SharedMemory`, `WorldBridge`, mixins), ScriptHookV ASI + ReShade compositor, **WebSocket on 127.0.0.1 + named shared memory**, a **fake host** (`host/fakehost.py`) and a fake D3D11 "GTA" (`gta/tests/fakegta.cpp`) used as synthetic oracles, plus `place_test.py` / `tnt_test.py` | **reference design** for Epic 6 (bridge, watchdog, synthetic host). Not reusable as a library: transport, framing and coordinate mapping are hard-coded per game pair |
| `terraria-tmodloader` | `reference/AgentBridge.cs` (JSON-lines agent bridge), `InModRecorder.cs`; assets were fal-generated | reference for test bridges; assets illustrate fal provenance |
| `aoe2-de-civ` | Data-mod pipeline, Blender render to SLD sprites; `assets/gen.sh` calls fal | reference only; **contains fal-generated assets** |

### 2.7 Tests and CI

- `tests/test_um.py`: 48 test functions (76 test cases after parametrization) covering scan engine detection (Unity Mono/IL2CPP, Unreal, Godot, GameMaker/RPG Maker, managed PE, VDF, Steam registry, known-game matching, online-only matching), sprites, video EDL, fal client (mocked), publish secret patterns, KB new/check/search/index, backup concurrency and restore round trips, ComfyUI (mocked), the PATH hook, and skill-copy parity.
- **Not covered:** `scan` for Source or Minecraft installs; `win launch/drive/shot/record` (Windows only); `publish check --game` hash matching at scale.
- CI (`.github/workflows/test.yml`, ubuntu): pytest, `um <group> --help` for all ten groups, `um kb check --index`, **`um publish check .`** on the whole repo. Any Studio file added to this repo is linted by the publish check: absolute user paths warn, secrets fail.
- **Class: reuse as-is.** Studio tests run alongside these (layout decided in ADR 0.3).

### 2.8 Packaging rules and guardrails

- `um publish check` (`um/publish.py`). **FAIL:** files byte-identical to the game install (size then sha1), API keys (fal, Anthropic, OpenAI, GitHub, AWS, private keys), `.env` files. **WARN:** decompiler fingerprints in code, engine archives over 5 MB, absolute user paths, no README, fal manifest without credit. This maps directly onto §3.3 and §100.
- `skills/mod-any-game/references/safety.md`: the bright line on online/anti-cheat, ownership and redistribution, kill by PID, bind bridges to `127.0.0.1` with a token, ask before driving input, installing loaders, changing the registry or publishing. It maps onto §67 and §89.
- **Class: reuse as-is** (rules) and **adapt** (`publish.check` structured output).

### 2.9 Agent, plugin and hook configs

| File | Contains | Class |
|---|---|---|
| `.mcp.json`, `mcp.json`, `.codex/config.toml`, `.cursor/mcp.json`, `.vscode/mcp.json`, `gemini-extension.json`, `opencode.json` | the fal MCP server (`https://mcp.fal.ai/mcp`, bearer `$FAL_KEY`) | **cloud-dependent** |
| `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `.codex-plugin`, `.cursor-plugin`, `.agents/plugins` | plugin metadata; keywords include `fal` | out of scope (distribution metadata) |
| `hooks/hooks.json`, `hooks/add-to-path.sh`, `.claude/settings.json` | SessionStart hook puts `bin/um` on PATH | **reuse as-is** (Claude sessions launched by Studio get `um`) |
| `AGENTS.md`, `CLAUDE.md`, `GEMINI.md` | agent instructions; "Start here" points to the mod-any-game loop and lists fal as a core tool | **adapt** via a Studio-level instruction overlay, not by editing upstream |

---

## 3. Reusable components (as-is or via a thin wrapper)

- **Recon:** `um.scan.scan()` / `all_games()` (structured dict), the `game-recon` skill, engine playbooks, and the `KNOWN` + `ENGINES` route tables in `um/scan.py`.
- **Save safety:** `um.backup` create/diff/restore with automatic pre-restore snapshots.
- **Windows automation:** `um win` launch/ps/kill-by-PID/shot/record/drive/reg, `WinDrive.ps1` (foreground-only input, `idle`), `ProcLoopback.ps1`.
- **Knowledge:** `um.kb` search/show/check/new, `knowledge/index.json`, the oracles technique note.
- **Local asset pipeline:** `um sprite`, `um render3d` (Blender), `um comfy` (localhost ComfyUI).
- **Release safety:** `um publish check`, `safety.md`.
- **Strategy knowledge:** `mashup-mods` Patterns 1–5 and guardrails.
- **Test discipline:** synthetic hosts (`fakehost.py`, `fakegta.cpp`), JSON-lines agent bridge, evidence-artifact journaling, circuit breaker.

## 4. Components requiring adaptation

| Component | Why | Proposed change | Where |
|---|---|---|---|
| `um.common.die()` everywhere | exits the process; unusable inside a long-running Studio core | Adapter calls `um` as a **subprocess with `--json`** where available, or in-process with `SystemExit` caught. Upstream PR later: raise a `UmError` and keep `die()` for CLI only. | adapter now, upstream later |
| `publish.check()` | prints; returns only an exit code | add `check(..., report=True) -> {fails, warns, files}` and `--json` | upstream PR (generally useful) |
| `scan()` | no game build/version; no VAC; L4D2 not in `KNOWN`; Minecraft Java not discoverable via `--list` | add Steam `buildid` from `appmanifest_*.acf`; add a Valve multiplayer/VAC note for `source`/`source2` hits; add `left 4 dead 2` to `KNOWN` (VScript + SourceMod on a local/listen server); Minecraft launcher discovery (official launcher `launcher_profiles.json`, `.minecraft`, Prism/MultiMC/CurseForge instances) | upstream PRs; Studio fingerprinting can extend locally first |
| `backup.restore()` | prints, needs `yes`; no dry-run object | adapter uses `diff()` for the preview the UI shows, then `restore(yes=True)` after user confirmation | adapter |
| `mod-any-game` / `AGENTS.md` loop | step 6 and the tool table default to fal; the journal is `MODLOG.md`, not a graph | Studio instruction overlay for its Claude sessions: graph is the journal of record, assets local-only, fal skill and MCP excluded. `MODLOG.md` stays as the human-readable feed for field notes. | Studio (no upstream edit) |
| `asset-pipeline`, `publish-mod` | fal-first wording | read through the overlay; nothing to change upstream | Studio |
| Passthrough example code | per-pair, hard-coded transport | extract the *ideas* (timestamps, watchdog, shared-memory ring + JSON control, synthetic host) into Studio's bridge library (Epic 6.3); don't import the example | Studio |

## 5. Cloud-dependent components (input to issue 0.2)

| Touchpoint | Path | Required by any non-fal path? |
|---|---|---|
| `um fal` group | `um/fal.py` | No. Imported at CLI start-up but needs `FAL_KEY` only when a fal command runs (`fal_key()`, line 75). `um fal --help` works keyless (CI proves it). |
| fal MCP server | the 7 config files in §2.9 | No, but Claude Code launched in this repo picks up the project `.mcp.json` (subject to the user's project-server approval), and the plugin install path carries the same server. With no key, the connection fails noisily; with a key, an agent that sees fal tools may use them. |
| `fal-assets` skill (×3 copies) | `skills/`, `.claude/skills/`, `.agents/skills/` | No |
| fal guidance inside other skills | `mod-any-game` (7 mentions), `asset-pipeline` (6), `publish-mod` (4), `case-studies.md` (3), `showcase-video` (2), `mashup-mods` (1), `safety.md` (1) | No; instructional only |
| Example asset scripts | `examples/aoe2-de-civ/assets/gen.sh`, `examples/terraria-tmodloader/assets/make_art.sh`, `examples/aoe2-de-civ/civ.py` | No (examples) |
| fal credit lint | `um/publish.py` (`fal_manifest.jsonl` → credit warning) | No; harmless, keep |

**Other network use, not generation (allowed, but must be visible per §99):** `um kb sync` (reads GitHub), `um kb pr` (pushes and opens a PR; already `--yes`-gated), `um win setup` (downloads ffmpeg, checksum-verified), and the `game-research-websearch` skill.

**Conclusion for 0.2:** no upstream code needs deleting. Isolation means three things: (a) Studio launches Claude sessions with an MCP config that omits fal and a skill set that omits `fal-assets`; (b) the Studio adapter never exposes `um fal`; (c) the UI reports "Asset generation: Local only" from a config flag. This still has to be verified against how Claude Code merges project `.mcp.json` with explicit `--mcp-config`. That is an open question for ADR 0.5.

## 6. Local-only components

Everything in §3, plus `um comfy` (localhost), `um video`, both PowerShell tools, the Blender script, all skills except `fal-assets`, all playbooks, and the KB when read from the local clone (`kb.local_root()` prefers the repo's own `knowledge/` and only syncs from GitHub when no local copy exists).

---

## 7. Proposed integration points

Studio talks to the toolkit through one `UniversalModderAdapter` (Epic 3.1). It maps onto the brief's §82 `UniversalModder` module:

| §82 module | Backed by | Mode | Structured? | Confirmation (§89) |
|---|---|---|---|---|
| Scanner | `um scan --json`, `um scan --list --json` | subprocess | yes | no |
| KnowledgeBase | `um kb search --json`, `kb show`, `kb check`, `kb new`; `knowledge/index.json` direct read | subprocess / file read | yes (search, index) | `kb pr` → **yes** |
| Backup | `um backup create/list/diff`; `restore --yes` | subprocess | `diff` yes; others parse output | `restore` → **yes** |
| Automation | `um win launch/ps/kill/shot/record`; `drive` | subprocess (Windows) | partial | `drive`, `launch` with loaders, `reg set` → **yes** |
| AssetPipeline | `um sprite`, `um render3d`, `um comfy` | subprocess | partial | no |
| PublishChecks | `um publish check` (structured after upstream PR) | subprocess | after adapt | packaging/sharing → **yes** |
| ReverseEngineering | skill + playbooks as Claude context only | context | n/a | n/a |

The adapter pins the toolkit version (`um --version`, git SHA) into each project's manifest, so builds are reproducible (§30 "tool versions").

**`GameAdapter` (§103) vs what exists:**

| Method | Existing support |
|---|---|
| `detect()` / `inspect()` | `um scan` (+ gaps in §4) |
| `install_mod_loader()` | none; playbook prose only. Must be confirmation-gated |
| `build()` | none generic; per-game (Gradle in the MC example, MSVC in Portalcraft) |
| `launch()` / `stop()` | `um win launch` / `um win kill <pid>` |
| `capture()` | `um win shot` / `record` (beware the frozen-capture gotcha in the oracles note) |
| `read_logs()` | none generic; log paths are listed in the `mod-any-game` table |
| `inspect_runtime()` | none generic; pattern = in-mod JSON-lines bridge (`AgentBridge.cs`) |
| `install_mod()` / `remove_mod()` | none generic |

---

## 8. Conflicts between the brief and the repository

1. **Fal is the toolkit's default asset route; the brief forbids depending on it** (§3.1, §97 vs `AGENTS.md` tools list, `mod-any-game` step 6, plugin keywords, seven MCP configs). Resolution: the Studio overlay (§5). No upstream deletion is needed, which keeps §137 satisfied.
2. **"The repository supports passthrough, embedded library, fusion, headless sim"** (brief §1, §11.3, §11.6). In code it supports one passthrough example. The rest are documented patterns. The brief's Epic 6 ("bridge library") is net-new work, and estimates should reflect that.
3. **Two route vocabularies.** KB `route` = `data | asset-only | loader-api | managed-patch | native-hook | reimplementation | decomp-recomp | passthrough | emulator | other`. Brief strategies = `content_port | host_native_reimplementation | passthrough | embedded_library | full_reimplementation | headless_simulation | hybrid | custom`. They describe different things: *how a mod reaches the game* vs *how a capability is implemented in a mashup*. Keep both and map them, so the KB stays searchable by strategy. Decide in ADR 0.6 and issue 1.5.
4. **Journal of record.** The repo's convention is a free-form `MODLOG.md`. The brief makes the graph plus a structured journal authoritative (§52, §74, §131). Resolution: the graph and journal are authoritative, and Studio renders a `MODLOG.md` from the journal so `um kb new` keeps working.
5. **Project-level donor/host.** KB notes use `game` + `games_also` (one primary game). The brief wants per-capability roles (§120–122). This only affects how Studio *writes* field notes, which can name the primary host.
6. **`um publish check .` in CI would lint Studio files.** This is fine and desirable. But Studio docs that quote example user-profile paths (the brief's own §100 example does) will raise WARN lines. These are not failures; keep them out of code files.
7. **"No external API"** (§63, §135) vs `um kb sync`, `kb pr`, `win setup`, and Claude itself. None of these is a generation service, and the brief allows them. Still, §99 requires they are never silent: the UI should show when the network is touched.

## 9. Missing infrastructure (net-new for Studio)

Grouped by the epic that owns it:
- **Epic 1:** graph schema, validator, deterministic serializer, semantic diff, dependency engine, migrations. Nothing exists. `um` uses PyYAML, so YAML tooling is already a dependency.
- **Epic 2:** project store, snapshots, branches, transactions, structured journal, artifacts. `um backup` covers save folders only.
- **Epic 3:** `UniversalModderAdapter`, `GameAdapter` interface, recon → graph mapping. Also version fingerprinting: Steam `buildid`, Minecraft version + loader + API triple, and a VAC/online flag for Valve multiplayer titles. Minecraft install discovery.
- **Epic 4:** context packer, intent → graph diff, generator contract (`generated_from`), hot-reload classifier.
- **Epic 5:** the desktop app (no UI exists).
- **Epic 6:** launcher, process manager with PID tracking and orphan cleanup, bridge/IPC library (shared memory + JSON control, timestamps, watchdog, fallback), runtime monitor, contract-driven test harness.
- **Epic 7:** an L4D2 `GameAdapter` (VScript + SourceMod on a local listen server, `-insecure`), Minecraft as semantic and asset donor (converters run on the user's install), and **a verified L4D2 nav-blocking mechanism**. This is not documented anywhere in the KB.

## 10. Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| L4D2 has no runtime nav-mesh blocking that rebuilds around arbitrary spawned blocks | medium | high (§125 requires "zombie navigation responds") | Epic 7.0 recon is a gate. Candidates to verify: `func_nav_blocker` / nav-blocking entities and VScript nav APIs on a listen server. Record the result in the KB; mark Unsupported honestly if it fails (§38) |
| fal MCP still loaded in Claude sessions Studio launches | high | medium (cost and noise, violates §97) | ADR 0.5 picks the invocation mode with explicit MCP and skill control; add a test that a Studio-launched session lists no fal tools |
| `die()` / `sys.exit` inside the long-running core | high if imported | high (core crashes) | subprocess boundary first; upstream `UmError` later |
| Upstream drift (the fork fell 5 commits behind within a day) | certain | low today | ADR 0.3: Studio code in separate paths; fork-specific changes listed in one file; regular fast-forward |
| Skill-copy test and publish check catch Studio files unexpectedly | medium | low | keep Studio skills (if any) in the same `skills/` → copies convention, or outside it entirely (ADR 0.3) |
| VAC: a modded L4D2 joins a secured server | low (with gating) | severe (account ban) | the L4D2 adapter always launches with `-insecure`; the launcher refuses otherwise; scan gains a VAC/online flag |
| Minecraft version-triple lock breaks shared projects | high over time | medium | the manifest records the exact MC + loader + API triple; the importer checks it (Epic 10) |
| Windows-only paths are untested in CI | high | medium | add a Windows CI job when the Studio adapter lands (Epic 3) |
| Pillow 14 removes `Image.getdata` (2027-10) | certain, distant | low | upstream one-line fix in the test |
| Scope (brief = years of work) | certain | high | exit gates in `IMPLEMENTATION_EPICS.md` |

---

## 11. Inputs for the remaining Epic 0 issues

- **#2 (0.2, cloud isolation):** §5 lists every touchpoint. The key open question is how Claude Code combines a project's `.mcp.json` with Studio's own MCP config, and whether skill directories can be excluded per session.
- **#3 (0.3, repo layout):** constraints found. The publish check and skill-copy test run over the whole repo in CI. The fork is fast-forwardable today. Toolkit code is import-hostile (`die()`), which favors a subprocess boundary and makes a companion repo cheap.
- **#4 (0.4, stack):** the toolkit is Python 3.10+ with `uv`; Windows tooling is PowerShell 5.1 + C# 5. A Python core can reuse `um` directly. The UI choice is independent of that.
- **#5 (0.5, invoking Claude):** sessions must load `um` on PATH (the existing SessionStart hook does this), the mashup and recon skills and the playbooks, but not fal. Confirmation boundaries must be enforced by the app (§7 table), not left to skill prose.
- **#6 (0.6, graph format):** PyYAML is already a dependency. Two route vocabularies need a mapping (§8.3). Stable dotted IDs match the KB's existing `game` and `engine` keys (`um scan` engine keys: `source`, `java`, `creation`, …).

---

## 12. Decisions from review (2026-10-07)

| # | Decision | Effect |
|---|---|---|
| D1 | **Don't assume more exists than the code shows; build what is needed.** | Runtime pieces the brief attributes to the repository (bridge/IPC, watchdog, launcher, process manager, strategy implementations beyond docs) are planned as net-new Studio work, using the examples only as reference designs. |
| D2 | **Call the toolkit as a separate command-line process.** | `UniversalModderAdapter` (Epic 3.1) runs `bin/um …` as a subprocess, using `--json` where it exists and parsing stdout or exit codes where it doesn't. Studio never imports `um` in-process, so `die()`/`sys.exit()` can't take the core down. Upstream structured-output PRs (e.g. `publish check --json`) are optional improvements, not prerequisites. |
| D3 | **fal is excluded from Studio entirely** (not an opt-in), isolated through configuration (§5), with no upstream deletion. | Studio-launched Claude sessions get an MCP config without fal and a skill set without `fal-assets`; the adapter doesn't expose `um fal`; Studio works with no `FAL_KEY`. Details in issue 0.2. |
| D4 | **Any modded Valve/Steam game is launched with `-insecure`, always.** | The launcher adds `-insecure` for every Source / Source 2 game and **refuses to launch** a modded Valve game without it; it isn't a user-toggleable setting. This covers L4D2. `-insecure` is a Valve-engine flag. Steam games on other engines don't have it; for those, the existing rules apply (offline, single-player or a user-run server; never inject past anti-cheat). |
| D5 | **Minecraft Java is discovered from the official launcher folder** (`%APPDATA%\.minecraft` on Windows), which may hold several installed versions. | Studio discovery reads `launcher_profiles.json` (installations: name, `lastVersionId`, `gameDir`) and `versions/<id>/<id>.json`, and lists each version or installation as a separate selectable donor. One project pins one exact version + loader + API triple. |
| D6 | **Game build versions are read locally first; AI is only a fallback.** | See below. |
| D7 | **Studio's in-app Claude uses the user's Claude subscription** (Claude Code signed in with their account), not an API key. | ADR 0.5 designs around a signed-in Claude Code session driven by the app. |
| D8 | **Repository layout is decided in ADR 0.3**, with no preference given up front. | ADR 0.3 compares a companion repo with a folder inside the fork. |

### What the user's machine actually has (read-only check, 2026-10-07)

`%APPDATA%\.minecraft` (official launcher, **Microsoft Store edition**: `launcher_*_microsoft_store.*` files):
- **3 launcher installations** in `launcher_profiles.json`:
  - "Mashup: My mash-up": version `26.2`, its own `gameDir` under `%LOCALAPPDATA%\MashupStudio\minecraft\…`, custom JVM args;
  - the default "Latest release" (resolves to `26.3`);
  - "Latest snapshot" (resolves to `26.4-snapshot-3`).
- **6 version folders** on disk: `26.2`, `26.3`, `26.3-pre-1`, `26.3-pre-2`, `26.4-snapshot-2`, `26.4-snapshot-3`.
- **No Fabric or NeoForge versions installed** (no `fabric-loader-*` or `neoforge-*` folder in `versions/`).
- `versions/26.3/26.3.json` reads locally as `id 26.3`, `type release`, `releaseTime 2026-09-15`, Java 25 (`java-runtime-epsilon`). This confirms D6 works for Minecraft with no network.

Consequences:
- Discovery lists **installations** (what the user picks in the launcher) and resolves each to a concrete version folder, because `latest-release`/`latest-snapshot` move when the launcher updates.
- A Studio project pins a concrete id (e.g. `26.3`), never `latest-*`.
- A loader install (Fabric) into this launcher is a §89 confirmation step. `minecraft.md` warns that `fabric-installer -launcher microsoft_store` fails on this launcher edition, so Studio adds the installation to `launcher_profiles.json` itself, after `um backup`.
- **An earlier Studio prototype already wrote to this machine:** the "Mashup: My mash-up" installation points at a `MashupStudio` data folder, and `servers.dat.modforge-backup` sits beside `servers.dat`. Before Epic 2 defines project storage, ADR 0.3 needs to decide whether to reuse, migrate or ignore that folder.

### How build versions are read locally (D6)

Everything below is a file read on the user's machine. No network and no model call.

| Source | Where | Gives | Status in `um` today |
|---|---|---|---|
| Steam games (L4D2, Skyrim, Portal 2…) | `steamapps/appmanifest_<appid>.acf` | `buildid` (Steam's build number for the installed depot), `LastUpdated` | the file is already parsed by `steam_games()` (`um/scan.py:134`), but `buildid` is dropped. A one-field addition |
| Source 1 games | `<game>/<mod>/steam.inf` (L4D2: `left4dead2/steam.inf`) | `PatchVersion`, `ClientVersion`, `ServerVersion` | not read; to verify on the user's L4D2 install in Epic 3 |
| Windows executables (any engine) | the PE `VERSIONINFO` resource of the main exe | `FileVersion`, `ProductVersion` | `pe_info()` reads only arch and managed flag; add a version-resource read |
| Epic Games Store | `ProgramData/Epic/EpicGamesLauncher/Data/Manifests/*.item` | `AppVersionString` | already parsed by `epic_games()`, field dropped |
| Xbox / Game Pass | `<install>/appxmanifest.xml` | `<Identity Version=…>` | not read |
| Minecraft Java | `.minecraft/versions/<id>/<id>.json` | `id` (e.g. `26.3`, or `fabric-loader-0.19.5-26.3`), `type`, `releaseTime`, `inheritsFrom` (base game version for loader profiles) | not read |
| Minecraft mods | `<gameDir>/mods/*.jar` → `fabric.mod.json` / `META-INF/neoforge.mods.toml` | loader and API versions (the version triple) | not read |
| Anything else | SHA-256 of the main executable | an exact fingerprint even with no readable version | not computed |

**AI fallback:** when only a hash or an opaque build number is available, Claude may map it to a human-readable version name (e.g. "Steam build 1234567 = the 2.2.x update"). It records the mapping as an **estimate** with its source. A fingerprint mismatch is always decided from the local data, never from the AI's guess. These reads land in Epic 3.2 (recon → graph) as Studio code, plus optional upstream PRs to `um scan` for the Steam and Epic fields it already parses.
