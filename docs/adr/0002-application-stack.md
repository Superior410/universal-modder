# ADR 0002: Application stack

- **Status:** Accepted **for validation** (2026-10-07). The core decision stands. The shell choice is settled by the §4 test, and passing it doesn't lock the architecture: later evidence can reopen this ADR.
- **Issue:** #4 (Epic 0, task 0.4)
- **Brief:** §81 (priorities, in order: maintainability, graph performance, Windows integration, Claude orchestration, local process control, extensibility), §82, §26, §117. **Audit:** D2, D7.

## Context and constraints

- **The core is headless.** Graph engine, project store, transactions, adapters, the Claude orchestrator, launcher and process manager all run without a window. They are reachable from a CLI (`mashup …`) and from tests. The UI is one client of the core and holds no logic (`IMPLEMENTATION_EPICS.md`, 0.4).
- **Windows first**, native, not WSL. Studio drives Windows games, PowerShell tools, Steam and the Minecraft and CurseForge launchers.
- The toolkit is Python 3.10+ with `uv`, and is called as a subprocess (D2). Its Windows tools are PowerShell 5.1 with embedded C# 5.
- Claude is reached by running the user's installed `claude` CLI as a subprocess (ADR 0003). The core must stream its JSON events to the UI.
- The UI must edit graphs of a few thousand nodes and stay responsive (Epic 5.3), with a beginner view and a technical view over the same data (§26). Overlays come later (§81).

## 1. Options for the core

| | Python 3.12 core (recommended) | TypeScript/Node core | Rust core |
|---|---|---|---|
| Fit with the toolkit | same language and tooling (`uv`, PyYAML); the toolkit's tests and conventions carry over | second ecosystem | second ecosystem |
| Graph engine needs (validation, diff, dependency traversal, migrations) | Pydantic v2 for schema + JSON Schema export; plain dict/graph code | Zod/TypeBox, equally fine | serde, fast but slower to change |
| Subprocess and Windows process control | `asyncio` subprocesses; Windows **Job Objects** via `pywin32` so child trees die with Studio (§17: no orphans) | `child_process`; Job Objects need a native addon | best-in-class, but more code |
| Maintainability for agent-written code (§81 #1) | high | high | medium |

**Core decision: Python 3.12**, managed by `uv`. It's packaged as a local service:
- exposes a JSON-RPC + event-stream API over a **WebSocket on `127.0.0.1`** with a per-launch random token (the toolkit's `safety.md` rule for any bridge);
- also exposes the `mashup` CLI over the same functions.

## 2. Options for the desktop shell

| | Tauri 2 + React | Electron + React (default if both pass §4) | PySide6 (Qt) | Plain browser tab |
|---|---|---|---|---|
| Windows integration | native window on WebView2 (preinstalled on Windows 11); sidecar processes; native dialogs | Chromium bundled, mature | native | none: no file dialogs, no tray, not a desktop app (§81 rules it out) |
| Installer size / memory | small | large (bundles Chromium) | medium | n/a |
| Graph editing UX | React ecosystem: **React Flow** for editable node graphs, **Cytoscape.js** or a WebGL renderer for very large overviews | same | `QGraphicsView`: fast, but every node editor, inspector and form is hand-built | same as Tauri |
| Future overlays (§81) | transparent always-on-top windows supported | supported | supported | no |
| Build toolchain | Node + Rust (the developer installs Rust; users don't) | Node | Python only | Node |
| Process control | delegated to the Python core either way | delegated | in-process | delegated |

**Shell decision: React + TypeScript UI. The host (Tauri 2 or Electron) is chosen by the §4 test.**
- The **Python core runs as a child process of the shell** (a sidecar in Tauri, a child process in Electron). The shell starts it, passes the token, and kills it on exit.
- React Flow is the editor for the beginner and technical views. A canvas/WebGL overview (Cytoscape.js) covers whole-project views if React Flow can't hold the target size.

**Rust rule (user decision):** Rust is allowed only as a build-only or native helper, and only if the test shows a concrete need. It is never part of the core. Tauri's shell is built with Rust, so: if Electron and Tauri both pass §4, **Electron is chosen** (no Rust anywhere). Tauri is chosen only if the test records a concrete advantage Electron can't match (e.g. installer size or memory on the user's PC, or sidecar process handling). The UI code is the same React app either way, so switching costs only the shell layer.

## 3. Why not a single Python desktop app (PySide6)

It has one language and no IPC, and Qt's scene graph is fast. It loses on §81 #1 and on graph UX: the node editor, inspectors, rule editor, diff views and search would all be custom widgets. React has mature, maintained graph-editing components. It also weakens the headless-core boundary, since UI and core would share a process and drift together.

## 4. Spike before acceptance (half a day, at the start of Epic 5, before 5.3)

| Check | Pass |
|---|---|
| React Flow with 3,000 nodes / 6,000 edges from a fixture graph, `onlyRenderVisibleElements` on | pan/zoom stays smooth on the user's PC; selecting a node updates the inspector in under 100 ms |
| Shell host, run for **both** Tauri 2 and Electron: start the Python core, token handshake, kill Studio from Task Manager | the core and every child process exit (Job Object); record installer size, idle memory and start time for each |
| Stream a `claude -p --output-format stream-json` run through the core to the UI | events appear live; cancelling sends SIGINT-equivalent and the run ends cleanly |
| Installer on the user's Windows 11 | installs per-user without admin; WebView2 present |

If React Flow fails the size test, the technical view switches to Cytoscape.js for large graphs. That changes component choice, not this ADR.

## Decision

- **Core:** Python 3.12 (`uv`), headless, with a `mashup` CLI and a localhost WebSocket API (token-protected).
- **Shell:** React + TypeScript, hosted by Electron unless the §4 test shows a concrete need for Tauri 2 (the only route by which Rust would enter, build-only). The core runs as a child process of the shell in both cases.
- **Graph UI:** React Flow, with Cytoscape.js as the large-graph fallback.
- **Process safety:** Windows Job Objects in the core, so children never outlive Studio.

## Consequences

- Epic 1 and Epic 2 are pure Python, testable on Linux CI and Windows CI.
- Epic 5 adds a Node toolchain for developers (plus Rust only if Tauri is chosen). Users get a normal per-user Windows installer.
- **Code signing** is not implemented in Epic 1. The installer and update layout keep it addable later: one signing step in the release script, stable publisher name, no self-modifying binaries.
- Every UI action maps to a core API call that is also a CLI command. This makes the UI scriptable and testable, and matches §73 (graph edit and natural language produce the same state).

## Resolved questions (2026-10-07)

1. Rust: only as a build-only/native helper when the test shows a concrete need; never in the core.
2. Code signing: not in Epic 1; keep the release layout compatible with adding it later.
