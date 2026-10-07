# ADR 0003: How Studio invokes Claude

- **Status:** Proposed (accept at the Epic 0 exit gate)
- **Issue:** #5 (Epic 0, task 0.5)
- **Brief:** §14, §18, §24, §27, §52, §73, §88, §89, §142. **Audit:** D3, D7. **Plan:** `docs/studio/CLOUD_ISOLATION.md`.
- **Sources** (Claude Code docs, read 2026-10-07): [CLI reference](https://code.claude.com/docs/en/cli-reference), [Run Claude Code programmatically](https://code.claude.com/docs/en/headless), [Agent SDK overview](https://code.claude.com/docs/en/agent-sdk/overview), [Permissions](https://code.claude.com/docs/en/permissions), [Skills](https://code.claude.com/docs/en/skills).

## Being honest about "local-first"

Studio is local-first for **generation, game data and project state** (§3.1, §99). **Claude is not local.** Every Claude call sends a prompt and context to Anthropic's servers and uses the user's Claude subscription. Studio says so in the UI and records each call in the journal. It also sends only the context the packer selected for that change (§3).

## Constraint from D7: the user's Claude subscription

The user wants Studio to use their Claude subscription, not an API key. Three facts from the docs decide the design:
1. **`claude -p` (headless mode) uses the same login as interactive Claude Code**, so a signed-in subscription works.
2. **`--bare` mode can't be used with a subscription.** It "never reads OAuth credentials" and needs `ANTHROPIC_API_KEY` or an `apiKeyHelper`.
3. **The Agent SDK note:** "Unless previously approved, Anthropic does not allow third party developers to offer claude.ai login or rate limits for their products, including agents built on the Claude Agent SDK. Use the API key authentication methods… instead."

## Options

### A. Drive the user's installed Claude Code CLI as a subprocess (recommended)

Studio runs `claude -p … --output-format stream-json` with the user's own installation and login. Studio **never handles, stores or offers** a login. Before the first call it checks `claude auth status`, which reports `authMethod` (`claude.ai`, `api_key`, …), and asks the user to sign in to Claude Code themselves if needed.

- Running `claude -p` from a script is the documented use of headless mode.
- Fine-grained control comes from flags: MCP (`--strict-mcp-config`, `--mcp-config`), tool rules (`--allowedTools`, `--disallowedTools`), permission routing (`--permission-prompt-tool`, `--permission-prompts`), prompts (`--append-system-prompt-file`), sessions (`--session-id`, `--resume`), structured output (`--json-schema`), limits (`--max-turns`, `--max-budget-usd`), and live events (`stream-json`, `system/init`, `system/api_retry`).
- The core language doesn't matter: a subprocess works from Python (ADR 0002).

### B. Claude Agent SDK (Python) inside the core

It gives richer callbacks (`canUseTool`, hooks as Python functions, native message objects). But per fact 3, an app built on the SDK may not offer claude.ai login without Anthropic's approval; the documented route is an API key. That conflicts with D7, so it is rejected for now. It becomes the natural choice if Studio switches to API-key billing (see Consequences).

### C. Claude API directly (Client SDK) with a hand-written tool loop

This loses Claude Code's tools, permissions, sessions, skills, plugins and hooks, and still needs an API key. Rejected.

## Decision: Option A

### 1. Session shape

One Claude Code session per **change transaction** (Epic 2.4), with a fresh `--session-id` UUID recorded in the journal. The cwd is the project folder (never the toolkit clone, see the isolation plan).

```text
claude -p "<task prompt>"
  --output-format stream-json --verbose
  --session-id <uuid>
  --strict-mcp-config --mcp-config <studio>/runtime/mcp.json
  --disallowedTools "mcp__fal" "mcp__fal__*" "Skill(skill:fal-assets)" <confirmation-class denials, §2>
  --allowedTools <per-autonomy-mode allow list, §4>
  --permission-prompt-tool mcp__studio__approve
  --append-system-prompt-file <studio>/runtime/studio-rules.md
  --add-dir <studio>/runtime/toolkit-skills
  --max-turns <N>
  [--json-schema <schema>]        # for structured steps, e.g. intent → graph diff
  [--model <user's choice>]
```

`<studio>/runtime/mcp.json` lists exactly one server: **`studio`**, a local stdio MCP server run by the Python core. It gives Claude typed access to the graph and the toolkit, and it is the permission host.

### 2. Confirmation boundaries (§89) are enforced by Studio, not by prompt text

| Action class | How Claude can do it | Who decides |
|---|---|---|
| Read or write project files (`graph/`, `generated/`, `adapters/`, `tests/`) | built-in Read/Edit, inside the project's working directory | autonomy mode (§4) |
| Run `um scan`, `kb search`, builds, tests in the project sandbox | `studio` MCP tools (`studio__run_um`, `studio__build`, `studio__test`) | autonomy mode |
| **Install a loader, write to a game folder, drive input, change saves, publish/share, open a KB PR** | **only** through `studio` tools (`studio__install_loader`, `studio__deploy_to_game`, `studio__drive_input`, `studio__restore_saves`, `studio__publish`) | **the user, every time.** The tool blocks until the UI confirms; the core takes a `um backup` before any save or game-folder change |
| The same actions by other routes (Bash `reg`, writing into a Steam or `.minecraft` folder) | **denied**: game and launcher folders are outside the working directories, so edits there prompt; the prompt goes to `mcp__studio__approve`, which **always denies** paths outside the project | policy |

Shell rules can't be perfect; the docs warn about [Bash rule limits](https://code.claude.com/docs/en/permissions). So the design doesn't rely on denying commands. Game-folder side effects happen only inside Studio's own tools, the session can't reach game folders through the file tools, and the approval tool refuses the rest. Launch rules are enforced in `studio__launch` itself and aren't left to Claude: `-insecure` for every Valve game (D4), private/LAN servers only for modded multiplayer (D10).

### 3. Context packing (§142)

The packer (Epic 4.1) writes `context/<transaction-id>/` into the project before the call, and the prompt points Claude at it:
- `graph-slice.yaml`: the changed elements plus their dependency closure (Epic 1.10), never the whole graph by default;
- `recon.json` for the games involved; `kb.md` with the top KB notes (`um kb search --json`), each trimmed to setup, route and gotchas;
- `files.txt`: the generated and adapter files carrying `generated_from` ids in the slice (§75);
- `tests.json`: the current status of the affected tests;
- `request.md`: the user's intent, autonomy mode, and the constraints that apply (D3, D4, D10).

Claude can still read more of the project with its file tools, and the journal records what was read. Nothing outside the project, the curated skills and the toolkit KB is offered.

### 4. Autonomy modes (§24) map to permissions

| Mode | Graph changes | Code in `generated/`, `adapters/`, `tests/` | Builds, tests | Confirmation-class actions |
|---|---|---|---|---|
| Guided | proposed as a diff (`--json-schema`) → user approves → applied by the core | after the user approves the diff | auto | user, every time |
| Automatic | applied, then reported | auto | auto | user, every time |
| Expert | larger changes applied, then reported | auto | auto | user, every time |

No mode turns off the confirmation class. The graph is always written by the **core** after schema validation, never by Claude editing YAML directly. Claude returns graph changes as structured output (`--json-schema`), which keeps §73 true: natural language and graph edits go through the same validated path.

### 5. Progress, cost, limits

- **Live progress:** the core forwards `stream-json` events to the UI as "Claude is reading X / editing Y / running tests". `system/api_retry` events with `rate_limit` show as "waiting for your Claude usage limit".
- **Cost:** with a subscription there's no per-token bill. `total_cost_usd` is still recorded as the docs' client-side **estimate**, labelled as such, so heavy changes are visible. `--max-turns` caps every run, and a transaction can be cancelled from the UI (SIGINT first, then terminate).
- **Version floor:** Studio checks `claude --version` at start and requires a version that has every flag used here (`--permission-prompts` needs v2.1.259 or later). It reads the `capabilities` array in `system/init` where available.

### 6. Offline or not signed in

| Works | Doesn't |
|---|---|
| Open projects; edit, validate and diff the graph; rules; branches and snapshots; rollback; build; run existing tests; launch; backups; KB search (local clone) | natural-language edits, regeneration, "explain this" (§27, it needs Claude), "why doesn't this work" (§118) |

A graph edit made offline is saved and shows **"implementation pending"**. When Claude is available again, Studio offers to regenerate the affected modules.

## Consequences

- Each user needs Claude Code installed and signed in. Studio's onboarding checks this and explains it.
- **If Studio is ever distributed to other people:** each user still brings their own Claude Code installation and login, and Studio never offers a login. Offering subscription login as a feature of a distributed product would need Anthropic's approval (fact 3). Otherwise Studio supports an **API-key mode**: the user's own `ANTHROPIC_API_KEY` or `apiKeyHelper`. That mode can also use `--bare` and the Agent SDK. The session builder is written so the auth mode is one setting.
- Epic 4 builds the `studio` MCP server (stdio, run by the core) and the approval tool before any code-generating session runs.

## Open questions for the user

1. Model default: whatever your Claude Code is set to, or pin one per project?
2. A per-transaction turn cap: start at 40 turns and adjust?
