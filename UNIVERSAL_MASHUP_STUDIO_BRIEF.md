# Universal Mashup Studio
## Comprehensive Claude Code Implementation Brief

**Status:** Authoritative product and architecture specification  
**Primary implementation agent:** Claude Code  
**Foundation:** `rehan-remade/universal-modder`  
**Primary target:** Windows PC game modding/mashup workflows  
**Generation model:** Local-first; no cloud generation credits required  
**Core principle:** The Universal Mashup Graph is the source of truth for mashup behavior and remains editable after Claude has generated the initial implementation.

---

# 1. Executive Summary

Build **Universal Mashup Studio**, a local-first application for creating, managing, editing, testing, sharing, and launching cross-game mashups.

The application must use the local capabilities of:

`https://github.com/rehan-remade/universal-modder`

as a foundation rather than replacing them.

The existing repository provides a valuable local modding toolkit: game discovery, engine detection, reverse-engineering workflows, engine playbooks, asset conversion, Windows automation, save backups, knowledge-base workflows, mashup strategies, testing concepts, and packaging/publish checks.

Retain those capabilities.

Do **not** depend on fal.ai, cloud asset-generation credits, or other paid external generation services.

Local generation is permitted.

Local ComfyUI, Blender, game-installed assets, user-owned game files, local converters, local scripts, and local IPC are all acceptable.

The repository currently supports multiple mashup approaches including content ports, passthrough with two games communicating locally, embedding a decompiled/reimplemented game as a library, full reimplementation/fusion, and headless guest-rule simulation. These should become implementation strategies within the new architecture rather than being discarded.

The resulting product should allow a user to say something such as:

> "Put Minecraft blocks into Left 4 Dead 2. Players should be able to place and destroy them, they should block zombies, affect navigation, interact with physics, and be saved."

Claude should:

1. Discover the participating games.
2. Analyze their engines and modding surfaces.
3. Discover the relevant capabilities of Minecraft blocks.
4. Discover the relevant interaction surfaces in L4D2.
5. Build a Universal Capability Graph.
6. Build a Mashup Integration Graph.
7. Select implementation strategies independently for each capability/relationship.
8. Establish authority and synchronization relationships.
9. Implement the mashup.
10. Generate and execute automated tests.
11. Produce a usable mashup project.
12. Expose the resulting graph to the user.
13. Allow the user to edit any reasonable aspect of the graph.
14. Translate graph/natural-language changes into implementation changes through Claude.
15. Support hot reload where possible and rebuild/restart where necessary.
16. Maintain version history and rollback.
17. Package the mashup as a shareable recipe/project without distributing game files.
18. Allow the resulting mashup to become a donor/capability source for future mashups.

The goal is not merely to visually insert content from one game into another.

The goal is to make imported functionality **participate in the host game's actual systems** wherever technically appropriate.

For example:

> Minecraft block → L4D2 collision → navigation → zombie AI → player interaction → physics → destruction → persistence.

---

# 2. Product Definition

Universal Mashup Studio is simultaneously:

1. A **game reconnaissance tool**.
2. A **mashup planning system**.
3. A **capability knowledge base**.
4. A **visual graph editor**.
5. A **natural-language mashup editor**.
6. A **Claude Code orchestration layer**.
7. A **mod/build system**.
8. A **runtime/bridge manager**.
9. A **testing system**.
10. A **project/library manager**.
11. A **version-control system for mashup architecture**.
12. A **sharing/package format**.
13. Eventually, a **multiplayer-capable mashup runtime**.

Do not reduce this to a GUI around shell commands.

The application itself must understand the mashup architecture.

---

# 3. Non-Negotiable Requirements

## 3.1 Local-first

The system must function without fal.ai or another paid cloud generation service.

Do not make external generation APIs a required dependency.

Local generation is allowed, including:

- ComfyUI
- Blender
- local AI models
- local scripts
- local asset converters
- game-native asset pipelines
- locally installed development tools

The existing repository explicitly supports local ComfyUI as an alternative to fal for asset workflows; preserve that principle and make cloud generation optional/non-required.

---

## 3.2 Preserve universal-modder functionality

Do not throw away useful existing functionality.

Preserve/integrate:

- `um scan`
- engine/version detection
- game reconnaissance
- engine playbooks
- reverse engineering workflows
- asset pipeline
- sprite tools
- 3D conversion/rendering
- Windows automation
- process management
- screenshots
- recording
- save backup/restore
- knowledge base
- mashup strategies
- verification workflows
- publish checks
- local runtime/bridge concepts

The existing project already organizes these into skills and a Python `um` CLI. The new application should orchestrate these capabilities rather than reinvent every tool.

---

## 3.3 No game files distributed

A shared mashup project must not package someone else's game installation.

Share:

- graph
- rules
- source code
- adapters
- configuration
- converters
- build instructions
- version requirements
- manifests
- project metadata
- user-created/local generated assets where legally appropriate
- capability definitions

Do not distribute:

- retail game executables
- copied proprietary game assets
- unauthorized game dumps
- decompiled game distributions
- other players' game installations

The recipient builds against their own installed games.

This is consistent with the repository's current publish guardrails.

---

# 4. Core Architectural Principle

## User intent → Claude reasoning → graph → implementation

The fundamental architecture is:

```text
User Intent
    ↓
Claude Analysis
    ↓
Universal Capability Graph
    ↓
Mashup Integration Graph
    ↓
Interaction Contracts
    ↓
Authority Graph
    ↓
Synchronization Graph
    ↓
Implementation Strategy Selection
    ↓
Adapters / Runtime / Mods / Bridges
    ↓
Build
    ↓
Automated Tests
    ↓
Real Game Verification
    ↓
Mashup Project
```

The graph is not merely documentation.

It is the **declarative source of truth** from which the implementation is derived.

Generated code is an implementation artifact.

---

# 5. The Universal Capability Graph

The Universal Capability Graph describes **what capabilities exist**, independent of one specific mashup.

A capability is not simply an asset.

Example:

```text
Minecraft Block
```

should be modeled as a capability containing potentially:

- visual representation
- physical volume
- placement
- destruction
- hardness
- collision
- support
- interaction
- inventory identity
- crafting identity
- sound
- particles
- state
- damage behavior
- world interaction
- persistence

The capability graph should represent semantic functionality rather than merely files/models.

---

# 6. Mashup Integration Graph

The Mashup Integration Graph describes how capabilities behave in a particular mashup.

Example:

```text
Minecraft Block
      │
      ▼
L4D2 Block Adapter
      │
      ├── Rendering
      ├── Collision
      ├── Physics
      ├── Navigation
      ├── Zombie AI
      ├── Player interaction
      ├── Destruction
      └── Persistence
```

The same capability can have completely different implementations in different games.

Example:

```text
Minecraft TNT
    │
    ├── Skyrim Adapter
    ├── L4D2 Adapter
    ├── GTA Adapter
    └── Future Host Adapter
```

---

# 7. Graph Layers

The underlying graph should support at least these conceptual layers:

## Layer 1 — Entities

Examples:

- Player
- Zombie
- Block
- Projectile
- Vehicle
- NPC
- Weapon
- World
- Item
- Terrain
- Explosion

## Layer 2 — Components

Examples:

- Transform
- Health
- Collision
- Inventory
- AI
- Physics body
- Navigation agent
- Renderer
- Audio source
- Damage receiver

## Layer 3 — Systems

Examples:

- Physics
- Navigation
- AI
- Damage
- Rendering
- Input
- Persistence
- Networking
- World generation
- Quest system

## Layer 4 — Capabilities

Examples:

- Minecraft block placement
- TNT explosion
- Skyrim spell casting
- L4D2 infected swarm
- GTA vehicle physics

## Layer 5 — Events

Examples:

- block.placed
- block.destroyed
- explosion.started
- explosion.finished
- entity.damaged
- player.interacted
- world.loaded
- world.saved

## Layer 6 — Rules

Declarative behavior created by the user or Claude.

## Layer 7 — Adapters

Concrete translations between systems.

## Layer 8 — Runtime bridges

IPC, shared memory, named pipes, WebSocket, local HTTP/control, shared GPU resources, or other appropriate mechanisms.

---

# 8. Graph Edge Types

Support rich relationships including:

- contains
- owns
- depends_on
- inherits
- implements
- renders_as
- collides_with
- damages
- heals
- controls
- spawns
- triggers
- translates_to
- feeds
- blocks
- modifies
- synchronizes_with
- mirrors
- persists_as
- authority_over
- reads_from
- writes_to
- overrides
- merges_with
- conflicts_with
- replaces
- falls_back_to
- generated_from
- adapted_by

The graph must permit users to create new edges.

Do not restrict the user to a fixed list of preconfigured connections.

---

# 9. Interaction Contracts

Every meaningful integration relationship should have an explicit contract.

Example:

```yaml
connection:
  id: minecraft-block-to-l4d2-zombie

source:
  capability: minecraft.block

target:
  entity: l4d2.infected

interaction:
  type: blocks_movement

required_surfaces:
  - collision
  - navigation
  - ai

authority:
  collision: l4d2
  navigation: mashup_runtime

strategy:
  collision: host_adapter
  navigation: host_adapter
  ai: host_adapter

events:
  - block.placed
  - block.destroyed
```

The contract is editable.

Claude can extend it when implementation analysis reveals missing surfaces.

---

# 10. Claude-Discovered Interaction Surfaces

Claude must not limit itself to the surfaces initially specified by the user.

If the user requests:

> "Put Minecraft blocks into L4D2."

Claude should reason about what is necessary.

For example:

```text
Rendering
Collision
Physics
Player interaction
Zombie interaction
Navigation
AI pathfinding
Destruction
Sound
Persistence
Networking
```

Claude should distinguish:

- required
- useful
- optional
- unsupported
- unnecessary

If Claude discovers a new required interaction surface during implementation, it may add it to the graph.

The application must display:

> Added by Claude

and explain:

> "L4D2 navigation integration was added because the new block geometry changes traversable space."

The user can:

- accept
- reject
- modify
- disable
- request implementation

---

# 11. Implementation Strategies

Implementation strategy is **not global to a mashup**.

Different capabilities and connections may use different strategies simultaneously.

Supported strategies should include:

## 11.1 Content Port

Translate guest behavior into native host behavior.

Example:

```text
Minecraft Block
→ L4D2 native world object
```

Use when host APIs are sufficient.

---

## 11.2 Host-Native Reimplementation

Recreate the donor behavior using the host's systems.

Example:

```text
Minecraft TNT behavior
→ L4D2 explosion/damage/physics APIs
```

---

## 11.3 Passthrough

Run multiple processes and synchronize them.

The existing repository already documents a passthrough architecture involving:

- guest simulation
- local state transport
- host injection
- collision back-channel
- timestamps
- watchdogs
- progressive implementation from a simple primitive to full content.

Use local IPC only.

Possible mechanisms:

- shared memory
- named pipes
- localhost WebSocket
- localhost HTTP/control
- UDP
- other appropriate local transport

---

## 11.4 Embedded Library

Where legally and technically appropriate, use a reimplementation/decompilation as a local library rather than running the complete donor game.

---

## 11.5 Full Reimplementation/Fusion

For maximum control.

Use where necessary.

---

## 11.6 Headless Guest Simulation

Reimplement donor rules as a simulation while using the host as the view.

This is especially useful for gameplay systems where the donor simulation is more important than rendering.

The repository already documents this pattern, including host entities mirroring guest simulation state and host features feeding the guest simulation.

---

## 11.7 Hybrid

A mashup can combine all of the above.

Example:

```text
Minecraft blocks
  rendering       → content port
  collision       → host adapter
  physics         → host physics
  redstone        → headless simulation
  special entities → passthrough
  persistence     → mashup runtime
```

---

## 11.8 Custom Adapter

Claude can create a new adapter architecture if the existing strategies are insufficient.

The graph must support:

```text
strategy.type = custom
strategy.definition = ...
```

---

# 12. Strategy Selection

Claude should automatically evaluate strategies.

For each relationship, evaluate:

- host API availability
- donor complexity
- fidelity requirements
- performance
- synchronization requirements
- latency
- persistence
- multiplayer requirements
- rendering requirements
- physics requirements
- licensing/ownership constraints
- implementation complexity
- maintainability
- robustness
- version stability

Claude should explain why it selected a strategy.

Example:

> Host Adapter selected because L4D2 exposes sufficient collision and damage surfaces. Passthrough was rejected because it would introduce unnecessary synchronization overhead.

The user can request:

> "Try another strategy."

Claude should then analyze alternatives.

---

# 13. Authority Graph

Authority must be fully customizable.

Authority is not a single property.

It can exist independently for:

- position
- rotation
- existence
- collision
- physics
- damage
- health
- AI
- navigation
- destruction
- inventory
- ownership
- interaction
- world state
- persistence
- rendering
- audio
- input
- networking
- synchronization

Example:

```text
Minecraft Block

placement authority:
    Minecraft

existence authority:
    Mashup Runtime

collision authority:
    L4D2

navigation authority:
    L4D2

damage authority:
    L4D2

persistence authority:
    Mashup Runtime
```

Users can change these relationships.

---

# 14. Immediate Authority Changes

When the user changes authority:

```text
Collision authority:
L4D2 → Mashup Runtime
```

Claude must immediately analyze and implement the consequences.

Pipeline:

```text
Graph changed
    ↓
Dependency analysis
    ↓
Affected modules identified
    ↓
Claude determines new architecture
    ↓
Generate/update code
    ↓
Build
    ↓
Run affected tests
    ↓
Hot reload if possible
    ↓
Restart if necessary
```

The UI must show progress.

---

# 15. Authority Conflict Detection

If two systems claim incompatible authority:

```text
Minecraft → position authority
L4D2 → position authority
```

do not silently select a winner.

Show:

> Authority conflict detected.

Offer:

- choose Minecraft
- choose L4D2
- choose Mashup Runtime
- bidirectional synchronization
- custom rule
- ask Claude to resolve

---

# 16. Synchronization Graph

Synchronization should be a first-class graph.

Represent:

- source
- target
- frequency
- latency target
- interpolation
- extrapolation
- authority
- timestamp
- reliability
- serialization
- batching
- fallback
- watchdog
- failure behavior

Example:

```yaml
sync:
  source: minecraft.player.transform
  target: l4d2.mashup_player.transform

  rate: 60hz
  authority: minecraft

  interpolation: enabled
  max_latency_ms: 50

  failure:
    watchdog_ms: 500
    fallback: freeze_and_detach
```

---

# 17. Runtime Failure Handling

Every passthrough/bridge connection needs failure behavior.

If a donor process crashes:

```text
Minecraft disconnected
        ↓
Bridge detects failure
        ↓
Graph-defined fallback
        ↓
Disable guest-controlled behavior
        ↓
Preserve host game where possible
```

Do not allow orphaned processes or uncontrolled input.

---

# 18. Natural Language Editing

The application must provide a natural-language interface.

Examples:

> "Make TNT knock Tanks back twice as far."

> "Don't let TNT damage the player."

> "Make snow blocks slow zombies."

> "Make Minecraft blocks destructible by L4D2 weapons."

> "Let Skyrim fire ignite Minecraft wooden blocks."

> "Make vehicles crush Minecraft blocks."

Claude should:

1. Interpret the request.
2. Identify affected graph nodes.
3. Create/update rules.
4. Identify required interaction surfaces.
5. Determine authority.
6. Select implementation strategies.
7. Implement.
8. Test.
9. Update graph status.

---

# 19. Declarative Rule System

Users must be able to create rules without writing game-specific code.

Example:

```text
WHEN
    Minecraft.TNT.explodes

AND
    target.type == L4D2.Tank

AND
    distance < 8m

THEN
    target.health -= explosion_damage
    target.rage += 25
    apply_impulse(target)
```

Rules should have:

- trigger
- conditions
- actions
- variables
- timing
- priority
- authority
- synchronization
- persistence
- conflict behavior

---

# 20. Reusable Rule Library

The application maintains a local rule library.

Users can save:

```text
TNT knocks enemies backward
Ice slows entities
Fire spreads to vegetation
Explosions destroy destructible objects
Blocks obstruct navigation
Magic damages undead
Vehicles crush blocks
```

A new mashup can choose:

> Add previously created rule

Claude evaluates compatibility and adapts it.

Rules can be shared with other users.

---

# 21. User-Created Capabilities

Users can create new capabilities.

Example:

```text
Frozen Minecraft Block

Properties:
    slows entities
    emits cold
    becomes slippery when melted
    damages fire-based entities
```

These become reusable capability definitions.

---

# 22. User-Created Systems

Users can also create systems that don't originate from a donor game.

Examples:

- universal weather
- cross-game destruction
- shared inventory
- universal magic
- cross-game vehicle system
- shared reputation
- cross-game crafting

Claude determines implementation.

---

# 23. Automatic Interaction Discovery

Claude should proactively analyze compatibility across donor capabilities.

Check:

- rendering
- collision
- physics
- damage
- AI
- navigation
- animation
- audio
- input
- inventory
- scripting
- quests
- world state
- persistence
- save/load
- networking
- multiplayer
- synchronization
- performance

Claude may propose additional connections.

It must not silently invent gameplay rules.

---

# 24. Claude Autonomy Modes

Implement three modes.

## Guided

Claude proposes architectural changes.

User approves.

## Automatic

Claude makes reasonable implementation decisions and reports them.

## Expert

Claude has broad architectural autonomy and performs larger graph changes automatically, while maintaining a detailed audit trail.

The user can change modes per project.

---

# 25. Graph Editing

The graph editor must support:

- creating nodes
- deleting nodes
- creating connections
- deleting connections
- changing edge types
- changing authority
- changing strategy
- changing synchronization
- editing conditions
- editing values
- editing rules
- enabling/disabling features
- replacing adapters
- selecting alternative strategies
- adding reusable rules
- creating capabilities
- creating systems
- viewing dependencies
- viewing implementation status
- viewing tests
- viewing Claude's reasoning summary

Do not artificially limit the number of editable features.

---

# 26. Beginner and Technical Views

Both views must use the same underlying graph.

## Visual/Beginner

Show:

```text
Minecraft
   │
   ├── Blocks ──────► L4D2
   │                    │
   │                    ├── Collision ✓
   │                    ├── Navigation ✓
   │                    ├── Zombies ✓
   │                    └── Physics ✓
```

Simple controls.

## Technical

Show:

```text
Capability
↓
Contract
↓
Adapter
↓
Host API
↓
Generated Module
↓
Runtime Bridge
↓
Test
```

Include:

- source files
- strategy
- authority
- transport
- synchronization
- dependencies
- events
- lifecycle
- performance
- implementation status

---

# 27. Explainability

Every important Claude-generated decision should be inspectable.

Example:

> Why was Passthrough selected?

Claude explains:

> "The host exposes insufficient APIs for native block simulation. Running the donor simulation preserves block behavior while exposing state to the host."

Also provide:

> Why was this interaction surface added?

> Why is this authority assigned here?

> Why did this module regenerate?

> Why did Claude reject this strategy?

---

# 28. Compatibility Matrix

Every capability should have a compatibility matrix.

Example:

```text
Minecraft Block → L4D2

Rendering          ✓
Collision          ✓
Physics            ✓
Player interaction ✓
Zombie interaction ✓
Navigation         ✓
Destruction        ⚠
Persistence        ✓
Networking         ⚠
```

Statuses:

- Fully integrated
- Partially integrated
- Unsupported
- Experimental
- Not applicable

Clicking a status shows the relevant graph and implementation details.

---

# 29. Incremental Regeneration

Do not rebuild everything for every change.

If the user changes:

```text
TNT → damages player = OFF
```

determine affected modules.

Potentially:

```text
TNT capability
Player damage adapter
Explosion integration
Tests
```

Do not regenerate:

- TNT renderer
- fuse logic
- zombie behavior
- unrelated systems

Use graph dependency analysis.

---

# 30. Versioning

Every meaningful graph state should be versionable.

Store:

- graph
- rules
- implementation manifest
- generated source state
- configuration
- test results
- game versions
- tool versions
- capability versions

Support:

- snapshots
- diff
- restore
- branch
- merge

---

# 31. Branching

Allow:

```text
Minecraft × L4D2
├── main
├── Zombie Fortress
├── TNT Mayhem
└── Experimental Physics
```

Branches must not restrict the underlying architecture.

A branch may become:

- another project
- a shared package
- a donor capability
- a release

---

# 32. Mashups as Donors

Any completed mashup may become a donor.

Example:

```text
Minecraft + L4D2
       ↓
Mashup Capability
       ↓
Skyrim
```

The user should be able to select:

> Use capability from another mashup.

The system should allow importing only a specific capability/rule/system rather than requiring the entire mashup.

---

# 33. Multiple Donors

A mashup may contain any reasonable number of donors.

Example:

```text
Minecraft
Skyrim
L4D2
GTA
Custom Capability Pack
Previous Mashup
```

Do not design around exactly two games.

---

# 34. Multiple Instances

The architecture should permit:

```text
Minecraft Instance A
Minecraft Instance B
L4D2 Instance A
Skyrim Instance A
```

but this is optional.

It should not be required for normal projects.

---

# 35. Input Ownership

Input is a graph resource.

Allow:

```text
Keyboard.W → Minecraft
Mouse → L4D2 Camera
F8 → Mashup Runtime
```

Support remapping and conflict detection.

---

# 36. Persistence

Every capability declares persistence requirements.

Example:

```text
Minecraft Block
    placement state → Mashup save
    location → Mashup save
    block type → Mashup save
```

Claude should integrate into the host's save/load mechanisms where possible.

If persistence is impossible, clearly mark it.

---

# 37. Performance Graph

Performance constraints are graph metadata.

Examples:

```text
Position synchronization: 60 Hz
Navigation updates: 10 Hz
Maximum latency: 16 ms
Fallback latency: 50 ms
```

Claude uses these constraints when selecting transport and implementation strategy.

---

# 38. Impossible/Unsupported Connections

The system must never falsely report success.

If something cannot be directly implemented:

```text
Direct implementation: Unsupported
```

Then suggest:

```text
Host Adapter: Possible
Headless Simulation: Possible
Passthrough: Possible
Experimental: Possible with limitations
```

The graph records the limitation.

---

# 39. Automated Testing

Tests must be generated from graph contracts.

For:

```text
Block → blocks zombie navigation
```

generate:

```text
1. Spawn zombie.
2. Establish navigation.
3. Place block.
4. Verify collision.
5. Verify navigation update.
6. Verify zombie reroutes.
7. Remove block.
8. Verify navigation recovers.
```

Test categories:

- unit
- adapter
- bridge
- synchronization
- runtime
- gameplay
- persistence
- multiplayer
- performance
- regression
- real-game integration

---

# 40. Test Status in Graph

Graph nodes should show:

```text
🟢 Implemented + verified
🟡 Implemented + partially verified
🔵 Experimental
🟠 Implemented + failing tests
🔴 Unsupported
⚪ Not applicable
```

---

# 41. Sandbox Development

Allow Claude to create temporary development/test environments.

Possible structure:

```text
Project/
    development/
    test/
    release/
```

Use isolated saves/configurations where possible.

The existing universal-modder workflow already emphasizes save backups and safe lab profiles; retain this behavior.

---

# 42. Game Reconnaissance

Before implementing a donor/host integration, Claude should use the repository's reconnaissance process.

Determine:

- installation
- engine
- engine version
- architecture
- managed/native
- loaders
- modding API
- save locations
- configuration
- logs
- anti-cheat
- online dependencies
- known modding route

The existing game-recon skill explicitly produces a modding plan containing this information. Integrate that workflow into project creation.

---

# 43. Knowledge Base Integration

Before beginning work:

```text
Search knowledge base.
```

After meaningful discoveries:

```text
Record:
    exact versions
    route
    engine behavior
    verification
    gotchas
```

The existing knowledge base is specifically designed to preserve this information for future agents.

Universal Mashup Studio should expose this knowledge to Claude automatically.

---

# 44. Project Structure

Each mashup must live in its own project directory.

Suggested structure:

```text
UniversalMashupProjects/
│
├── Minecraft_L4D2_ZombieFortress/
│   │
│   ├── project.yaml
│   ├── manifest.yaml
│   │
│   ├── graph/
│   │   ├── capabilities.yaml
│   │   ├── integration.yaml
│   │   ├── authority.yaml
│   │   ├── synchronization.yaml
│   │   ├── rules.yaml
│   │   ├── strategies.yaml
│   │   └── dependencies.yaml
│   │
│   ├── versions/
│   │
│   ├── branches/
│   │
│   ├── capabilities/
│   │
│   ├── rules/
│   │
│   ├── adapters/
│   │
│   ├── runtime/
│   │
│   ├── bridges/
│   │
│   ├── generated/
│   │
│   ├── assets/
│   │
│   ├── converters/
│   │
│   ├── tests/
│   │
│   ├── logs/
│   │
│   ├── artifacts/
│   │
│   ├── build/
│   │
│   └── README.md
```

Do not require every folder to exist if unused.

---

# 45. Project Manifest

Example:

```yaml
project:
  id: minecraft-l4d2-zombie-fortress
  name: Minecraft Zombie Fortress
  version: 0.1.0

games:
  - id: minecraft
    role: donor
    required_version: ...
  - id: l4d2
    role: host
    required_version: ...

capabilities:
  - minecraft.blocks
  - minecraft.tnt
  - l4d2.infected

runtime:
  mode: hybrid

generation:
  provider: local

network:
  official_services: false
```

---

# 46. Shareable Mashup Package

A shareable project should contain:

```text
manifest
capability graph
integration graph
authority graph
sync graph
rules
adapters
source
build instructions
game requirements
version requirements
dependency requirements
configuration
test specifications
```

It should not contain retail game files.

---

# 47. Import Workflow

Recipient imports a mashup:

```text
Import
 ↓
Read manifest
 ↓
Detect required games
 ↓
Fingerprint versions
 ↓
Check dependencies
 ↓
Check local capabilities
 ↓
Resolve differences
 ↓
Build locally
 ↓
Run tests
 ↓
Launch
```

If their game version differs:

```text
Version mismatch
```

Claude may attempt adaptation.

---

# 48. Compatibility and Migration

If a game updates:

```text
L4D2 version changed
```

the application should identify affected integrations.

Example:

```text
L4D2.Navigation API changed
    ↓
Affected:
    BlockNavigationAdapter
    ZombiePathAdapter
```

Claude can regenerate those components rather than rebuilding unrelated systems.

---

# 49. Launcher

A mashup may require:

- one game
- multiple games
- bridge processes
- runtime process
- helper processes
- server
- client
- overlay

The launcher must orchestrate these.

A single merged executable is **not required**.

A PowerShell/batch/native launcher is acceptable.

Example:

```text
launcher
    ↓
start runtime
    ↓
start guest
    ↓
start host
    ↓
wait for readiness
    ↓
connect bridges
    ↓
verify health
    ↓
launch gameplay
```

---

# 50. Runtime Process Manager

The application should monitor:

- process IDs
- launch state
- connection state
- bridge state
- crashes
- watchdogs
- logs
- synchronization health

The user should see:

```text
Minecraft        🟢
L4D2             🟢
Mashup Runtime   🟢
Bridge           🟢
Navigation Sync  🟢
```

---

# 51. Logs and Evidence

Every implementation run should produce artifacts.

Suggested:

```text
artifacts/
    2026-10-06/
        recon.json
        strategy.json
        build.log
        test-results.json
        screenshots/
        runtime/
        synchronization/
```

The existing mashup methodology emphasizes evidence artifacts and reproducible test runs; retain this philosophy.

---

# 52. Claude Implementation Journal

Maintain a machine-readable and human-readable record of Claude's decisions.

Example:

```text
2026-10-06
Added:
  L4D2 navigation interaction

Reason:
  Minecraft blocks modify traversable geometry.

Strategy:
  Host Adapter

Affected:
  BlockCollision
  BlockNavigation
  ZombiePathing

Tests:
  14 passed
  1 failed
```

---

# 53. Rollback Safety

Before risky changes:

- snapshot project state
- preserve previous graph
- preserve previous implementation
- preserve save data
- preserve generated configuration

The existing repository already provides save backup/diff/restore functionality; integrate it rather than replacing it.

---

# 54. Application Library

The main application should present projects as a library.

Example:

```text
MY MASHUPS

Minecraft × L4D2
Zombie Fortress
v0.8
🟢 Playable

Minecraft × Skyrim
Blockbound Skyrim
v0.4
🟡 Partial

Minecraft × L4D2 × Skyrim
Fantasy Apocalypse
v0.1
🔵 Experimental
```

Each project has:

- launch
- edit graph
- edit rules
- inspect capabilities
- view status
- rebuild
- test
- branch
- export/share
- import
- archive
- duplicate

---

# 55. Project Dashboard

Show:

```text
Games:
    Minecraft
    L4D2

Capabilities:
    27

Connections:
    83

Rules:
    14

Strategies:
    6

Authority conflicts:
    0

Tests:
    142 passed
    3 failed

Implementation:
    94% verified
```

The percentages should be meaningful, not arbitrary.

---

# 56. Graph Search

The user must be able to search:

> TNT

and see:

```text
Minecraft.TNT
    ↓
Explosion
    ↓
Damage
    ↓
Physics
    ↓
L4D2
```

Also:

> navigation

to find every system affected by navigation.

---

# 57. Dependency Visualization

Clicking a node should show:

```text
Minecraft Block
    │
    ├── Collision
    │     └── L4D2 Physics
    │
    ├── Navigation
    │     └── L4D2 AI
    │
    ├── Persistence
    │     └── Mashup Save
    │
    └── Rendering
          └── L4D2 Renderer
```

---

# 58. Conflict Resolution

When multiple donors affect the same host system, support:

- priority
- merge
- override
- coexist
- conditional rules
- custom resolver
- Claude-selected resolver

Example:

```text
Minecraft Explosion
        ↓
      L4D2 Damage

Skyrim Explosion
        ↓
      L4D2 Damage
```

The user can determine how they combine.

---

# 59. Capability Packs

Eventually support reusable capability collections.

Example:

```text
Minecraft Building Pack
    blocks
    placement
    breaking
    inventory
    crafting

Minecraft Explosives Pack
    TNT
    explosion
    fire
    destruction
```

Capability packs can be reused across mashups.

---

# 60. Sharing Rules and Capability Packs

Users can share:

- rules
- capabilities
- systems
- adapters
- complete mashups

without sharing game files.

A recipient can install a capability pack and use it against their own installed games.

---

# 61. Multiplayer Architecture

Design the graph for:

- local
- LAN
- internet
- client/server
- host authority
- peer authority
- dedicated server

Do not require full internet multiplayer implementation for the initial release.

The graph must not make future multiplayer impossible.

---

# 62. Dedicated Server Architecture

Eventually support:

```text
Mashup Server
    │
    ├── Capability simulation
    ├── Authority
    ├── Persistence
    ├── Synchronization
    └── Rule execution
```

Clients receive state.

The server should not necessarily render donor content.

---

# 63. Local-Only Communication

The system may use:

- localhost HTTP
- localhost WebSocket
- shared memory
- named pipes
- UDP
- local sockets
- other IPC

"No external API" means:

> No dependency on external cloud generation/services.

It does **not** mean that internal local IPC is forbidden.

Passthrough architecture specifically requires local communication. The existing repository documents localhost transport, shared memory, named pipes, and related mechanisms.

---

# 64. Asset Pipeline

The asset pipeline must remain local.

Support:

- extraction from user's installation
- conversion
- resizing
- format conversion
- sprite conversion
- 3D conversion
- rigging where local tools permit
- Blender workflows
- texture processing
- local ComfyUI
- procedural generation

Do not require fal.

The existing asset pipeline already provides game-format conversion workflows and should be reused.

---

# 65. Generated Assets

Generated assets should be stored inside the local project:

```text
assets/
    generated/
    converted/
    source/
```

Track:

- origin
- generator
- version
- parameters
- license/source metadata
- conversion process

---

# 66. Game Automation

Retain universal-modder's Windows automation.

Support:

- launch
- process management
- screenshots
- recording
- input
- logs
- process monitoring

The application should ask for user confirmation before potentially disruptive operations when appropriate.

---

# 67. Anti-Cheat / Online Safety

Do not connect to official online services in ways that bypass protections.

Respect the existing repository's rule:

- offline
- single-player
- or user-controlled servers

for experimental mashups.

Do not implement anti-cheat bypassing.

The repository's mashup skill explicitly establishes these boundaries.

---

# 68. Source of Truth Hierarchy

For any implementation decision, Claude should prioritize:

1. actual game behavior
2. actual installed game files
3. actual APIs/source/metadata
4. verified reverse-engineering results
5. project evidence
6. knowledge base
7. documented community information
8. assumptions

Do not invent game behavior.

---

# 69. Oracle-Based Development

For complex systems, create an oracle/reference implementation or test harness.

Example:

```text
Guest simulation
    ↓
Known input
    ↓
Known expected state
```

Then compare the implementation.

This is particularly important for:

- physics
- timing
- damage
- movement
- AI
- simulation
- deterministic rules

---

# 70. Vertical Slice Requirement

Do not attempt the entire mashup at once.

Start with the smallest meaningful interaction.

Example:

```text
Minecraft cube
      ↓
L4D2 renderer
```

Then:

```text
cube
 ↓
position
 ↓
collision
 ↓
physics
 ↓
real block
 ↓
zombie interaction
 ↓
navigation
```

This mirrors the existing passthrough methodology.

---

# 71. Claude Development Loop

For each project:

```text
1. Parse user intent.
2. Identify games.
3. Search knowledge base.
4. Recon games.
5. Fingerprint versions.
6. Identify capabilities.
7. Identify host surfaces.
8. Construct capability graph.
9. Construct integration graph.
10. Discover missing surfaces.
11. Propose interactions.
12. Select strategies.
13. Establish authority.
14. Establish synchronization.
15. Create contracts.
16. Build vertical slice.
17. Test.
18. Expand.
19. Validate.
20. Package.
21. Expose graph.
22. Wait for user modifications.
23. Incrementally regenerate.
```

---

# 72. User Modification Loop

After Claude finishes:

```text
User edits graph
        ↓
Graph diff
        ↓
Dependency analysis
        ↓
Affected capabilities
        ↓
Claude implementation
        ↓
Build/test
        ↓
Hot reload or restart
        ↓
Updated graph status
```

The project never becomes "locked."

---

# 73. Natural Language + Graph Are Equivalent Interfaces

A graph edit:

```text
TNT.damage.player = false
```

and natural language:

> "Don't let TNT hurt the player."

must produce the same underlying declarative state.

The graph remains authoritative.

---

# 74. No Hidden Configuration

Do not put important mashup behavior exclusively in generated source code.

If a behavior matters architecturally, represent it in the graph.

Generated code implements graph state.

---

# 75. Generated Code Metadata

Every generated module should identify:

```text
generated_from:
    graph nodes
    graph edges
    rules
    contracts
```

This allows incremental regeneration.

---

# 76. Dependency-Aware Regeneration

Every graph element should have dependencies.

When changed:

```text
changed node
    ↓
dependency traversal
    ↓
affected modules
    ↓
affected tests
```

Only regenerate what is required.

---

# 77. Hot Reload

Support hot reload where technically possible.

Examples:

- configuration
- rules
- numerical values
- graph routing
- certain adapters
- local runtime state

If hot reload isn't possible:

```text
Restart required
```

The application decides automatically.

---

# 78. Transactional Changes

A graph change should be treated as a transaction:

```text
Before
 ↓
Apply graph change
 ↓
Generate
 ↓
Build
 ↓
Test
 ↓
Commit if valid
```

If implementation fails:

```text
Rollback
```

unless the user explicitly chooses to keep the experimental state.

---

# 79. Experimental Mode

Allow users to intentionally maintain broken/experimental graph states.

For example:

```text
🔵 Experimental

Known failures:
    Navigation synchronization
    Persistence
```

Do not automatically delete experimental work.

---

# 80. Project Branches Must Preserve Experimentation

Users should be able to make dangerous changes on a branch without affecting the stable branch.

---

# 81. Application Technology

Choose a technology stack that supports:

- Windows
- rich graph editing
- desktop integration
- process management
- filesystem access
- subprocesses
- WebSockets/local IPC
- responsive UI
- large graphs
- future overlays

Do not choose a stack merely because it is fashionable.

Prioritize:

1. maintainability
2. graph performance
3. Windows integration
4. Claude orchestration
5. local process control
6. extensibility

If a web-based frontend is selected, package it as a desktop application with proper native process/filesystem integration.

---

# 82. Architecture Components

Recommended conceptual modules:

```text
UniversalMashupStudio
│
├── UI
│   ├── ProjectLibrary
│   ├── GraphEditor
│   ├── RuleEditor
│   ├── CapabilityBrowser
│   ├── TechnicalInspector
│   ├── ClaudeConsole
│   ├── TestViewer
│   └── RuntimeMonitor
│
├── Core
│   ├── GraphEngine
│   ├── CapabilityRegistry
│   ├── RuleEngine
│   ├── StrategyEngine
│   ├── AuthorityEngine
│   ├── SyncEngine
│   ├── DependencyEngine
│   ├── VersionEngine
│   └── CompatibilityEngine
│
├── Claude
│   ├── IntentParser
│   ├── Planner
│   ├── Implementer
│   ├── Reviewer
│   └── TestGenerator
│
├── UniversalModder
│   ├── Scanner
│   ├── ReverseEngineering
│   ├── AssetPipeline
│   ├── Automation
│   ├── Backup
│   ├── KnowledgeBase
│   └── PublishChecks
│
├── Runtime
│   ├── ProcessManager
│   ├── BridgeManager
│   ├── IPC
│   ├── Watchdog
│   └── Launcher
│
└── Sharing
    ├── PackageBuilder
    ├── Importer
    ├── Validator
    └── VersionMigrator
```

---

# 83. Graph Storage

Prefer a format that is:

- human-readable
- diffable
- versionable
- deterministic
- easy for Claude to edit
- easy for the application to validate

YAML or JSON is acceptable.

A hybrid approach is also acceptable.

Avoid making the graph dependent on an opaque binary database.

---

# 84. Graph Schema Versioning

Every graph must have:

```text
schema_version
```

The application must support migrations.

---

# 85. Capability Identity

Capabilities need stable IDs.

Example:

```text
minecraft.block
minecraft.tnt
l4d2.infected
skyrim.magic
```

Version them independently from projects.

---

# 86. Capability Provenance

Track where capabilities originated.

Example:

```yaml
origin:
  type: game
  game: minecraft
  version: ...
```

or:

```yaml
origin:
  type: mashup
  project: minecraft-l4d2
  capability: minecraft.tnt
```

or:

```yaml
origin:
  type: user_created
```

---

# 87. Rule Provenance

Track:

- author
- source project
- created date
- dependencies
- implementation history
- compatible games
- version

---

# 88. Claude Audit Trail

Claude changes must be auditable.

Record:

- request
- interpretation
- graph changes
- strategy changes
- authority changes
- files changed
- tests run
- results
- unresolved issues

Do not expose hidden chain-of-thought.

Store concise implementation rationale and decisions, not private reasoning traces.

---

# 89. User Confirmation Boundaries

The system may automatically implement graph changes inside the project.

However, preserve confirmation boundaries around potentially disruptive operations such as:

- installing game loaders
- modifying game installations
- driving input
- changing saves
- publishing/sharing

The existing universal-modder workflow already uses such safeguards.

---

# 90. Project Health

Each project gets a health status based on actual evidence.

Example:

```text
BUILD        🟢
GRAPH        🟢
TESTS        🟢
RUNTIME      🟢
PERSISTENCE  🟡
MULTIPLAYER  🔵
```

Do not calculate health solely from whether compilation succeeded.

---

# 91. Example: Minecraft Blocks in L4D2

The first major reference implementation should demonstrate:

```text
Minecraft Blocks
        ↓
L4D2
```

Required behavior:

- Steve/player can place blocks.
- Blocks appear correctly.
- Blocks occupy physical space.
- Player collides with them.
- Zombies collide with them.
- Zombies cannot simply walk through them.
- Navigation responds appropriately.
- Blocks can be destroyed where configured.
- Destruction affects gameplay.
- Physics behaves appropriately.
- State can persist where feasible.
- Host systems interact with blocks rather than merely rendering them.

The graph should expose every major interaction.

---

# 92. Example: Steve Uses TNT in L4D2

Required architecture:

```text
Minecraft TNT
    ↓
Fuse
    ↓
Explosion
    ├── L4D2 world
    ├── zombies
    ├── players
    ├── physics
    ├── destructible objects
    └── navigation
```

Each branch can use a different strategy.

Example:

```text
Explosion visuals
    → content port

Damage
    → host adapter

Physics
    → host physics

Block destruction
    → mashup runtime

Navigation
    → host adapter

Minecraft-specific fuse logic
    → headless simulation
```

---

# 93. Example: Skyrim + Minecraft

User:

> "Put Minecraft blocks in Skyrim and let them affect NPCs."

Claude should discover:

```text
Block
 ↓
Skyrim collision
 ↓
NPC navigation
 ↓
NPC pathfinding
 ↓
NPC interaction
 ↓
Physics
 ↓
Persistence
```

If appropriate, Claude should implement these surfaces rather than merely creating decorative cubes.

---

# 94. Example: Three Donors

```text
Minecraft
    blocks
    TNT

Skyrim
    magic

L4D2
    infected

        ↓

Universal Mashup

Minecraft TNT
    → Skyrim magic interaction
    → L4D2 damage
    → L4D2 physics

Minecraft blocks
    → Skyrim world
    → L4D2 navigation

Skyrim magic
    → L4D2 infected
```

Claude should identify possible interactions and propose them.

---

# 95. Example: Mashup as Donor

```text
Minecraft + L4D2
       ↓
Zombie Fortress Capability
       ↓
Skyrim + Zombie Fortress
```

The recipient project should be able to consume a capability without importing the original project wholesale.

---

# 96. Sharing

Sharing must be project/package based.

Example:

```text
ZombieFortress.mashup
```

contains the declarative and implementation information required to recreate the mashup against the recipient's own games.

The application should validate the recipient's installed games before building.

---

# 97. No Cloud Generation

The application must not require:

```text
FAL_KEY
```

or another cloud generation credential.

If the repository's fal skill exists, isolate or disable it.

Do not break unrelated local tooling merely because cloud generation is disabled.

The application should explicitly report:

```text
Asset generation:
Local only
```

---

# 98. Optional Local ComfyUI

If available:

```text
Local ComfyUI
```

may be used.

If unavailable:

- use existing assets
- use converters
- use procedural generation
- use placeholders
- ask the user when necessary

Do not make cloud generation a fallback.

---

# 99. Local Storage

All generated project state should remain local unless the user explicitly exports/shares it.

Do not silently upload:

- graph
- source
- assets
- game paths
- logs
- screenshots
- save data

---

# 100. Privacy

Game installation paths and user-specific information should remain local.

A shared project should use portable references rather than exposing absolute local paths.

Example:

```text
Game:
    Minecraft

Install path:
    resolved locally
```

not:

```text
C:\Users\John\Games\...
```

---

# 101. Build System

The project must be able to invoke:

- game-specific build tools
- loaders
- compilers
- Gradle
- MSBuild
- CMake
- Rust
- Python
- PowerShell
- Blender
- other discovered tools

through adapters.

Do not hard-code one game's build system into the entire application.

---

# 102. Engine Adapters

The application should have an engine integration layer.

Examples:

```text
Unity
Unreal
Source
Source 2
Bethesda
Minecraft
Native C++
.NET/XNA
Godot
GameMaker
etc.
```

Reuse universal-modder's existing engine playbooks rather than recreating them. The repository already includes extensive engine-specific routes.

---

# 103. Game Adapter Interface

Conceptually:

```text
GameAdapter

detect()
inspect()
install_mod_loader()
build()
launch()
stop()
capture()
read_logs()
inspect_runtime()
install_mod()
remove_mod()
```

Not every game needs to implement every method.

---

# 104. Capability Extraction

Claude should be able to analyze a donor and produce capability definitions.

For example:

```text
Minecraft
    Player
    Blocks
    Items
    Inventory
    Crafting
    Redstone
    TNT
    Fire
    Fluids
    Mobs
```

Only extract what is needed initially, then expand as required.

---

# 105. Progressive Graph Expansion

Do not attempt to understand every aspect of every game before starting.

Use:

```text
Requested capability
    ↓
Minimum required surfaces
    ↓
Implementation
    ↓
New discovery
    ↓
Graph expansion
```

This keeps the system practical.

---

# 106. Runtime Abstraction

The mashup runtime should provide common concepts where useful:

```text
Entity
Transform
Event
Capability
Rule
Authority
State
Synchronization
Persistence
Bridge
```

But do not force every game to conform to a generic abstraction when that would reduce fidelity.

The runtime should adapt to games, not force games into an artificial universal engine.

---

# 107. Strategy Engine Must Remain Extensible

New implementation strategies can be added later.

The graph should not assume a fixed finite list.

---

# 108. Capability Negotiation

When importing a shared mashup, compare:

```text
Required capability
vs.
Locally available capability
```

Example:

```text
Required:
L4D2.Navigation v3

Installed:
L4D2.Navigation v4
```

Claude can determine whether adaptation is safe.

---

# 109. Migration Engine

When versions differ:

```text
Old Adapter
    ↓
Analyze changes
    ↓
Generate migration
    ↓
Test
    ↓
Update project
```

---

# 110. User Overrides

If Claude recommends:

```text
Passthrough
```

the user must be able to say:

> "Use host-native reimplementation instead."

Claude should attempt it and update the graph.

---

# 111. Strategy Comparison

The technical view should allow:

```text
Strategy comparison

Host Adapter
  Fidelity: High
  Performance: High
  Complexity: Medium

Passthrough
  Fidelity: Very High
  Performance: Medium
  Complexity: High

Headless Simulation
  Fidelity: High
  Performance: High
  Complexity: High
```

These values must be evidence-based or clearly marked estimates.

---

# 112. Rule Priority

Rules need deterministic ordering.

Support:

- priority
- conditions
- phases
- before/after hooks
- conflict resolution

---

# 113. Determinism

Where simulation matters, provide deterministic modes where possible.

This is especially important for:

- multiplayer
- replays
- testing
- synchronization
- debugging

---

# 114. Event System

Create a common event abstraction.

Examples:

```text
entity.spawned
entity.destroyed
entity.damaged
block.placed
block.destroyed
player.interacted
explosion.started
explosion.finished
world.loaded
world.saved
game.connected
game.disconnected
```

Game adapters translate native events into these concepts.

---

# 115. Adapter Lifecycle

Adapters should support:

```text
initialize
connect
start
pause
update
sync
shutdown
recover
```

Where applicable.

---

# 116. Runtime Health

Each adapter reports:

```text
connected
healthy
latency
last update
authority
errors
```

---

# 117. Debugging

The technical view should provide:

- graph path tracing
- event tracing
- authority tracing
- synchronization tracing
- runtime logs
- adapter logs
- test artifacts
- screenshots
- state dumps

Example:

> Why didn't the zombie detect the block?

Trace:

```text
block.placed
 ↓
BlockAdapter
 ↓
Collision
 ↓
NavigationUpdate
 ↓
ZombiePathing
```

Show where the chain failed.

---

# 118. "Why Doesn't This Work?"

Provide a natural-language diagnostic interface.

User:

> "Why can't the Tank destroy this block?"

Claude examines:

- block health
- collision
- damage contract
- Tank attack event
- authority
- rule priority
- adapter state
- runtime logs

Then explains the actual missing link.

---

# 119. User Control vs Claude Control

Claude should do the difficult technical work.

The user controls:

- desired behavior
- graph relationships
- authority
- strategies
- rules
- priorities
- feature enablement
- project branching
- sharing
- final acceptance

---

# 120. Do Not Over-Constrain the User

Do not create artificial restrictions such as:

> "Minecraft must always be the donor."

A game can be:

- donor
- host
- guest
- simulation source
- runtime authority
- renderer
- client
- server

and these roles may differ by capability.

---

# 121. Do Not Force One Global Host

A mashup may effectively have multiple authorities.

Example:

```text
Minecraft:
    block simulation authority

L4D2:
    player combat authority

Mashup Runtime:
    cross-game rule authority

Dedicated Server:
    network authority
```

---

# 122. Project Roles

Represent roles per capability rather than assuming one global donor/host relationship.

---

# 123. Future Extensibility

Design for:

- more games
- more donors
- more capability packs
- more implementation strategies
- more runtime transports
- multiplayer
- dedicated servers
- live editing
- overlays
- community sharing
- reusable rule libraries
- automated compatibility migration

---

# 124. MVP Definition

The MVP should not attempt every possible game.

It should demonstrate the architecture with at least one meaningful mashup.

Recommended first vertical slice:

**Minecraft + Left 4 Dead 2**

Implement:

1. game detection
2. project creation
3. capability extraction
4. graph visualization
5. Minecraft block capability
6. L4D2 host integration
7. collision
8. basic placement
9. zombie interaction
10. graph editing
11. natural-language edit
12. Claude regeneration
13. automated test
14. project launch
15. save/project state
16. versioning
17. export/share

---

# 125. MVP Must Demonstrate Actual System Integration

Do not accept:

```text
Minecraft block model appears in L4D2.
```

as the primary success criterion.

Success must look more like:

```text
Player places block
        ↓
Block exists
        ↓
Player collides
        ↓
Zombie collides
        ↓
Zombie navigation responds
        ↓
Gameplay changes
        ↓
Block can be interacted with
        ↓
State persists
```

Where technically possible.

---

# 126. Second Vertical Slice

Add:

```text
Minecraft TNT
        ↓
L4D2
```

Demonstrate:

- fuse
- explosion
- visual effects
- damage
- physics
- world interaction
- destruction
- configurable rules
- authority
- graph editing

---

# 127. Third Vertical Slice

Demonstrate:

```text
Minecraft + L4D2
        ↓
Mashup becomes donor
        ↓
Skyrim
```

This validates the composability architecture.

---

# 128. Acceptance Criteria

The project is successful when a user can:

1. Select installed games.
2. Create a mashup project.
3. Ask Claude to combine capabilities.
4. Watch Claude analyze the games.
5. Inspect the resulting graph.
6. Edit the graph.
7. Create new connections.
8. Create rules using natural language.
9. Reuse saved rules.
10. Change authority.
11. Change implementation strategy.
12. Add a new interaction surface.
13. Have Claude implement the change.
14. Hot reload or rebuild automatically.
15. Run tests.
16. Inspect failures.
17. Roll back.
18. Branch.
19. Launch the mashup.
20. Save the project.
21. Export/share it.
22. Have another user import it against their own game installations.
23. Reuse a capability from a previous mashup.
24. Make a mashup itself a donor.

---

# 129. Quality Requirements

Prioritize:

- correctness
- transparency
- reversibility
- modularity
- extensibility
- local operation
- deterministic graph state
- incremental generation
- testability
- maintainability

Do not prioritize:

- making the architecture look simple at the expense of functionality
- forcing everything into one executable
- forcing every game into one generic engine model
- cloud generation
- unnecessary abstraction
- superficial visual integration

---

# 130. Important Distinction

Do not confuse:

```text
Visual presence
```

with:

```text
Gameplay integration
```

The application is specifically intended to enable the latter.

A block is not successfully integrated merely because it renders.

It should interact with whatever host systems are necessary to fulfill the requested behavior.

---

# 131. Important Distinction: Graph vs Code

The graph is authoritative.

Code is generated from the graph.

If code and graph disagree:

> The graph wins.

Claude must reconcile/rebuild the code.

---

# 132. Important Distinction: Claude vs User

Claude creates and implements.

The user owns the design.

Claude must not permanently lock architectural decisions.

Every meaningful implementation decision should be replaceable where technically feasible.

---

# 133. Important Distinction: Capability vs Implementation

Keep:

```text
WHAT
```

separate from:

```text
HOW
```

Example:

```text
Minecraft TNT
```

is the capability.

Its L4D2 implementation might be:

```text
host adapter
```

while its Skyrim implementation might be:

```text
reimplementation
```

and its GTA implementation might be:

```text
passthrough
```

The capability definition remains reusable.

---

# 134. Important Distinction: Donor vs Host

Do not make donor/host a rigid project-level relationship.

Represent relationships per capability/system.

---

# 135. Important Distinction: Local API vs Cloud API

Allowed:

```text
localhost
shared memory
named pipes
WebSocket
local HTTP
UDP
```

Not required:

```text
fal.ai
cloud asset generation
paid external generation APIs
```

---

# 136. Integration With Universal-Modder

Treat the repository as a toolkit/library of knowledge and executable workflows.

Do not simply copy every file into the new project.

Create an integration boundary.

Conceptually:

```text
Universal Mashup Studio
        │
        ▼
Universal Modder Adapter
        │
        ├── um scan
        ├── um win
        ├── um backup
        ├── um sprite
        ├── um render3d
        ├── um kb
        ├── publish checks
        └── existing skills/playbooks
```

Retain upstream compatibility where practical.

---

# 137. Upstream Changes

When possible:

- avoid destructive modifications to universal-modder
- create adapters/extensions
- isolate Studio-specific code
- document any fork-specific changes
- keep the ability to pull upstream improvements

---

# 138. Repository Strategy

If the new application is implemented as a companion project, prefer:

```text
universal-modder/
    existing toolkit

universal-mashup-studio/
    application
```

with a controlled integration layer.

If a monorepo is technically superior, preserve a clear boundary between:

```text
toolkit
application
runtime
project data
```

---

# 139. First Claude Code Task

Before implementing the full application, Claude must inspect:

- repository structure
- all existing skills
- CLI commands
- engine playbooks
- mashup skill
- knowledge base
- examples
- tests
- Windows tools
- packaging rules

Then produce:

```text
UNIVERSAL_MASHUP_STUDIO_REPO_AUDIT.md
```

containing:

- reusable components
- components requiring adaptation
- cloud-dependent components
- local-only components
- proposed integration points
- conflicts
- missing infrastructure
- risks

Do not begin blindly.

---

# 140. Second Claude Code Task

Build the core graph schema and validation system before building the full UI.

Deliver:

```text
capability graph
integration graph
authority graph
sync graph
rules
strategies
dependencies
versioning
```

with fixtures and tests.

---

# 141. Third Claude Code Task

Build a minimal graph editor.

It must support:

- nodes
- edges
- editing
- saving
- loading
- validation
- diff

before attempting the full application.

---

# 142. Fourth Claude Code Task

Implement Claude orchestration.

Claude should receive:

- user intent
- project graph
- game recon
- relevant source
- knowledge base
- affected graph nodes
- relevant implementation files
- test status

Do not send irrelevant project context for every change.

---

# 143. Fifth Claude Code Task

Implement incremental regeneration.

Test:

```text
Change one graph property
→ only affected modules regenerate
```

---

# 144. Sixth Claude Code Task

Implement runtime/project management.

Support:

- launch
- stop
- monitor
- logs
- bridges
- watchdog
- health

---

# 145. Seventh Claude Code Task

Implement first Minecraft/L4D2 vertical slice.

---

# 146. Eighth Claude Code Task

Implement rule library, versioning, branching, export/import.

---

# 147. Ninth Claude Code Task

Implement mashup-as-donor functionality.

---

# 148. Development Discipline

At every stage:

- build
- test
- document
- preserve working state
- record known limitations
- avoid speculative rewrites

Do not allow the application architecture to drift away from the graph model.

---

# 149. Definition of Done

A feature is not complete merely because:

```text
code compiles
```

It is complete when:

```text
graph representation exists
        +
implementation exists
        +
tests exist
        +
runtime behavior is verified
        +
failure behavior is understood
        +
user can inspect/edit it
```

---

# 150. Final Product Philosophy

Build Universal Mashup Studio as an **extensible local-first operating environment for cross-game capability composition**.

The user should eventually be able to think:

> "I want Minecraft's building system, L4D2's infected AI, Skyrim's magic, and a custom weather system to coexist."

and not have to manually determine:

- which API to call
- which game files to inspect
- which loader to install
- how collision works
- how navigation works
- how to synchronize processes
- how to serialize state
- which implementation strategy is appropriate
- which modules need regeneration

Claude should do that work.

But after Claude does it, the user should **own the architecture**.

The user must be able to open the graph and say:

> "Actually, I want Skyrim to control this."

or:

> "Use passthrough instead."

or:

> "Make this reusable."

or:

> "Add this rule."

or:

> "Remove that connection."

or:

> "Make the mashup itself a donor."

and Claude should turn those changes into a functioning implementation.

The final mental model should therefore be:

```text
                    UNIVERSAL MASHUP STUDIO

                         USER INTENT
                              │
                              ▼
                     ┌────────────────┐
                     │     CLAUDE     │
                     │ Analyze/Reason │
                     └───────┬────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │ UNIVERSAL CAPABILITY │
                  │        GRAPH         │
                  └──────────┬───────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │ MASHUP INTEGRATION   │
                  │        GRAPH         │
                  └──────────┬───────────┘
                             │
             ┌───────────────┼────────────────┐
             ▼               ▼                ▼
        Authority       Strategies      Synchronization
             │               │                │
             └───────────────┼────────────────┘
                             ▼
                  ┌──────────────────────┐
                  │   GENERATED SYSTEMS  │
                  │ Adapters / Bridges / │
                  │ Runtime / Game Mods  │
                  └──────────┬───────────┘
                             │
                             ▼
                       BUILD + TEST
                             │
                             ▼
                         RUN MASHUP
                             │
                             ▼
                       USER EDITS GRAPH
                             │
                             └───────────────┐
                                             │
                                             ▼
                                          CLAUDE
                                             │
                                             ▼
                                      NEW IMPLEMENTATION
                                             │
                                             ▼
                                     NEW MASHUP VERSION
                                             │
                                             ▼
                                      BECOMES A DONOR
```

**This loop is the product.**

Do not build a static mod generator around the graph.

Build a system in which the graph, Claude, the runtime, and the user continuously cooperate to create and evolve cross-game systems.

================================================================================
# 151. BUILT-IN NEW USER EDUCATION, HELP, AND ONBOARDING SYSTEM
================================================================================

## 151.1 Purpose

Universal Mashup Studio must include a built-in education and help system intended for people who have never used the application before.

This is a product feature, not merely external documentation.

The system must teach users:

- what a mashup is
- what a capability is
- what the Universal Mashup Graph represents
- how donor and host games work
- how Claude assists with implementation
- what interaction surfaces are
- how authority works
- how synchronization works
- how implementation strategies differ
- how rules work
- how to edit the graph
- how to create and reuse capabilities
- how to use previous mashups as donors
- how to test and troubleshoot projects
- how to package/share projects
- what the application's limitations and safety boundaries are

The documentation should be available inside the application and should be accessible contextually from relevant screens.

The user should never have to read the complete developer implementation brief to understand how to use the application.

---

## 151.2 Three Levels of Help

The application should provide three complementary levels.

### Level 1 — Quick Help

Short explanations shown next to controls.

Example:

> Authority determines which system has the final say over a property such as position, collision, damage, or persistence.

### Level 2 — Contextual Explanation

A longer explanation opened from the current screen.

Example:

> A collision connection tells the mashup that two objects must participate in the same physical interaction. Collision and AI navigation are not necessarily the same system; if you want an imported object to change enemy pathfinding, an additional navigation connection may be required.

### Level 3 — Complete Guide

A searchable, structured user guide containing tutorials, examples, terminology, workflows, troubleshooting, and advanced concepts.

---

## 151.3 First-Run Experience

On first launch, offer:

- Welcome
- What Universal Mashup Studio does
- Create a sample project
- Learn the basic graph concepts
- Create the user's first real project

Do not force a lengthy tutorial before allowing experienced users to proceed.

Offer:

- Start Tutorial
- Skip
- Don't show this again

The tutorial can always be reopened from Help/Learn.

---

## 151.4 Interactive First Mashup Tutorial

Provide a guided sample such as:

> Minecraft-style block in a host game.

The tutorial should walk the user through:

1. Selecting a host game.
2. Selecting a donor game.
3. Describing a desired capability.
4. Reviewing discovered capabilities.
5. Reviewing Claude's proposed interaction surfaces.
6. Viewing the generated graph.
7. Understanding a connection.
8. Understanding authority.
9. Selecting/accepting an implementation strategy.
10. Building the project.
11. Running tests.
12. Launching the result.
13. Making a natural-language change.
14. Viewing the resulting graph diff.
15. Rebuilding/reloading the affected components.

Use a safe sample project where possible rather than modifying the user's real games during the tutorial.

---

## 151.5 Context-Sensitive Education

Every major editor must provide a Help action.

Required contexts include:

- project dashboard
- game scanner
- capability browser
- graph editor
- connection editor
- authority editor
- synchronization editor
- strategy editor
- rule editor
- runtime monitor
- test results
- compatibility matrix
- project history
- package/export screen
- troubleshooting screen

Help must explain the concept in terms of the current project.

For example, in the Authority editor, do not merely define authority abstractly. Show an example from the user's actual graph.

---

# 152. COMPLETE NEW USER GUIDE CONTENT
================================================================================

The following is the baseline content specification for the in-application user guide.

--------------------------------------------------------------------------------
## Welcome to Universal Mashup Studio
--------------------------------------------------------------------------------

Universal Mashup Studio lets you combine capabilities from different games and build new experiences from them.

The important idea is that a mashup is not merely a visual combination.

If you import a Minecraft block into another game, the goal can be much more than displaying the block. The block can potentially participate in the host game's collision, physics, AI, navigation, damage, persistence, audio, interaction, and other systems.

You describe what you want.

Claude helps determine how to build it.

The Universal Mashup Graph records what the project is supposed to do.

The generated implementation makes that design real.

You can then test, inspect, change, and reuse it.

--------------------------------------------------------------------------------
## What Is a Mashup?
--------------------------------------------------------------------------------

A mashup is a project that combines capabilities from one or more source games with a host game or runtime.

Examples:

- Minecraft blocks in Left 4 Dead 2
- Minecraft construction in Skyrim
- Portal mechanics in another game
- L4D2-style infected behavior in a different host
- multiple donor games contributing different systems to one project

A project can have multiple donors.

A mashup can later become a donor itself.

--------------------------------------------------------------------------------
## What Is a Capability?
--------------------------------------------------------------------------------

A capability is something a game can do or provide.

A capability can be:

- an object
- a mechanic
- an AI behavior
- a physics behavior
- an inventory feature
- a crafting rule
- a world system
- a weapon
- a vehicle
- a projectile
- a building system
- a portal
- a weather system
- a persistence rule

A capability is broader than an asset.

For example, Minecraft TNT can be understood as:

- a placeable object
- a fuse
- an explosion event
- damage
- impulse
- physics
- destruction
- entity interaction

This decomposition lets the application connect the useful parts of a capability to host systems.

--------------------------------------------------------------------------------
## The Universal Mashup Graph
--------------------------------------------------------------------------------

The Universal Mashup Graph is the central model of the application.

It records:

- entities
- capabilities
- components
- systems
- events
- rules
- adapters
- runtime bridges
- dependencies
- authority
- synchronization
- implementation strategies
- relationships between all of them

The graph is not merely a diagram.

It is the declarative source of truth for the mashup.

Generated code is an implementation of that graph.

If the user changes the intended behavior, change the graph and let Claude update the implementation.

--------------------------------------------------------------------------------
## The Graph Is Layered
--------------------------------------------------------------------------------

Different graph views answer different questions.

Capability view:
> What can this game provide?

Integration view:
> How is the capability connected to the host?

Authority view:
> Who controls each property?

Synchronization view:
> How does state move between systems?

Dependency view:
> What depends on what?

Rule view:
> What should happen under each condition?

These are views of the same underlying project state.

--------------------------------------------------------------------------------
## Nodes and Connections
--------------------------------------------------------------------------------

Nodes represent things such as:

- Player
- Zombie
- Block
- Vehicle
- World
- Physics body
- Renderer
- AI
- Navigation
- Capability
- Rule
- Adapter
- Event

Connections describe relationships such as:

- renders_as
- collides_with
- damages
- controls
- spawns
- triggers
- blocks
- modifies
- synchronizes_with
- persists_as
- authority_over

You can inspect and create connections yourself.

Claude can also discover connections that you did not explicitly request.

--------------------------------------------------------------------------------
## Interaction Surfaces
--------------------------------------------------------------------------------

An interaction surface is a system that a capability needs to communicate with.

For example, an imported block may require:

- rendering
- collision
- physics
- player interaction
- AI
- navigation
- destruction
- persistence

Collision alone may not be enough for AI.

A zombie can physically collide with a block while its pathfinding system still believes the route is open.

If the user asks for zombies to navigate around blocks, Claude should therefore consider navigation as a separate interaction surface.

--------------------------------------------------------------------------------
## Claude Can Discover Missing Connections
--------------------------------------------------------------------------------

You do not need to know every system a capability must interact with.

If you say:

> Put Minecraft blocks in L4D2 and make them stop zombies.

Claude may discover that the project needs:

- block rendering
- block placement
- collision
- zombie interaction
- navigation integration

The application must show newly discovered requirements clearly.

Example:

> Added by Claude: Navigation obstacle integration.
>
> Reason: Physics collision prevents physical passage but does not necessarily update the host AI's pathfinding representation.

The user can accept, reject, or modify the proposal.

--------------------------------------------------------------------------------
## Implementation Strategies
--------------------------------------------------------------------------------

There is no single way to combine games.

The application supports several strategies.

### Content port

Adapt content into the host.

### Host-native reimplementation

Recreate the relevant behavior using host systems.

### Passthrough

Run a donor/guest process locally and communicate with the host.

### Embedded library

Use an appropriate local library implementation of the relevant behavior.

### Full reimplementation/fusion

Recreate the required system directly.

### Headless guest simulation

Reimplement rules without requiring the donor game's complete renderer.

### Hybrid

Combine multiple approaches.

The strategy can be selected independently for different capabilities and connections.

--------------------------------------------------------------------------------
## Why Strategies Can Be Mixed
--------------------------------------------------------------------------------

A single project might use:

- content port for rendering
- host physics for collision
- headless simulation for rules
- passthrough for a complex subsystem
- host persistence for saved state

This is often better than forcing the entire donor game into one architecture.

--------------------------------------------------------------------------------
## Authority
--------------------------------------------------------------------------------

Authority answers:

> Which system has the final say?

Authority can be different for different properties.

For a block:

- appearance: donor capability
- placement rules: donor-derived rule
- position: host physics
- collision: host physics
- navigation: host
- persistence: mashup runtime

If two systems both try to control the same property, the application should identify the conflict rather than silently choosing one.

--------------------------------------------------------------------------------
## Synchronization
--------------------------------------------------------------------------------

When systems communicate, they must synchronize state.

Synchronization may be:

- event-driven
- periodic
- continuous
- one-way
- bidirectional
- batched
- request/response

The project should specify appropriate behavior for each connection.

A failed bridge should also have a defined fallback.

--------------------------------------------------------------------------------
## Natural-Language Editing
--------------------------------------------------------------------------------

Natural language is a primary editing method.

Examples:

> Make wooden blocks destructible by zombies.

> Make stone blocks indestructible.

> Make TNT knock Tanks backward.

> Let vehicles crush blocks.

> Save placed blocks when the level ends.

> Use host physics as authority for block position.

Claude should translate these requests into graph and implementation changes.

--------------------------------------------------------------------------------
## Rules
--------------------------------------------------------------------------------

Rules describe behavior.

Examples:

> When a block is placed, create its host representation.

> When TNT reaches zero fuse time, trigger an explosion.

> When a zombie attacks a wooden block, apply damage.

Rules can be reused between projects.

--------------------------------------------------------------------------------
## Reusable Rules
--------------------------------------------------------------------------------

Save useful rules in the application.

Examples:

- Solid object blocks AI
- Explosive object damages nearby entities
- Fire spreads to flammable objects
- Vehicles crush destructible objects
- Objects persist across sessions

When reusing a rule, the application should check its dependencies and adapt it to the new host.

--------------------------------------------------------------------------------
## Simple and Technical Modes
--------------------------------------------------------------------------------

Simple mode should expose concepts such as:

- capabilities
- behaviors
- enabled features
- obvious connections

Technical mode should expose:

- graph nodes
- edge types
- interaction contracts
- authority
- synchronization
- adapters
- implementation strategies
- runtime bridges
- generated modules
- dependencies

Users should be able to move between modes without losing project state.

--------------------------------------------------------------------------------
## Creating a Project
--------------------------------------------------------------------------------

A beginner workflow is:

1. Create Mashup.
2. Select host.
3. Add donor.
4. Describe desired behavior.
5. Let Claude inspect the games.
6. Review discovered capabilities.
7. Review proposed graph.
8. Review important implementation choices.
9. Build.
10. Test.
11. Launch.
12. Iterate.

Start with the smallest useful version.

--------------------------------------------------------------------------------
## Game Reconnaissance
--------------------------------------------------------------------------------

Before implementation, the application should inspect local installations.

It can identify:

- installation location
- game version
- engine
- architecture
- modding route
- configuration
- save locations
- logs
- loaders
- relevant development interfaces

This information becomes part of project compatibility.

--------------------------------------------------------------------------------
## Why Game Versions Matter
--------------------------------------------------------------------------------

Game updates can change:

- memory structures
- APIs
- loaders
- file formats
- engine behavior
- modding interfaces

Projects should record compatibility information and warn when the user's installed version differs.

--------------------------------------------------------------------------------
## First Mashup Example: Minecraft Blocks in L4D2
--------------------------------------------------------------------------------

Start with one block.

Initial goal:

> Place one Minecraft block in L4D2.

Then add:

1. rendering
2. placement
3. collision
4. zombie obstruction
5. navigation
6. destruction
7. persistence

Do not attempt every Minecraft mechanic at once.

--------------------------------------------------------------------------------
## Why Collision Is Not Enough
--------------------------------------------------------------------------------

A collision object and an AI navigation obstacle may be separate representations.

Therefore:

Block
→ collision

does not automatically imply:

Block
→ navigation
→ zombie pathfinding

If AI must understand the object, explicitly integrate it.

--------------------------------------------------------------------------------
## TNT Example
--------------------------------------------------------------------------------

TNT can involve:

- placement
- fuse
- explosion
- damage
- impulse
- physics
- entity interaction
- block destruction
- sound
- effects
- persistence

When you request TNT, Claude should identify the required systems.

--------------------------------------------------------------------------------
## Map Destruction
--------------------------------------------------------------------------------

If you ask:

> Make TNT permanently destroy the host map.

Claude must determine whether the host supports runtime geometry changes.

If it does not, the application should say so.

A supported alternative might be:

- destructible proxy geometry
- removable collision
- debris
- persistent damage state

Do not claim unsupported behavior is working.

--------------------------------------------------------------------------------
## Multiple Donors
--------------------------------------------------------------------------------

A mashup can combine several sources.

Example:

Minecraft:
- construction

Portal:
- portals

L4D2:
- infected AI

Skyrim:
- host world

Project-specific rules can connect these capabilities.

--------------------------------------------------------------------------------
## Mashups as Donors
--------------------------------------------------------------------------------

A completed mashup can export capabilities.

For example:

Minecraft + L4D2
→ Constructible Barrier capability

That capability can then be used in another project.

Users should not need to import an entire mashup when they only want one capability.

--------------------------------------------------------------------------------
## Project Library
--------------------------------------------------------------------------------

Each mashup should be a separate project.

Example:

- Minecraft L4D2
- Minecraft Skyrim
- Portal L4D2
- Multi-Donor Fantasy
- Constructible Barrier Demo

Projects must be isolated from one another.

--------------------------------------------------------------------------------
## Project Dashboard
--------------------------------------------------------------------------------

The dashboard should show:

- project description
- host
- donors
- status
- compatibility
- graph version
- last build
- last test
- runtime state
- warnings
- recent changes

Useful project areas include:

- Overview
- Graph
- Capabilities
- Rules
- Strategies
- Authority
- Synchronization
- Runtime
- Tests
- Logs
- History
- Package

--------------------------------------------------------------------------------
## Project Status
--------------------------------------------------------------------------------

Useful states include:

- Ready
- Building
- Needs Attention
- Incompatible
- Runtime Error
- Out of Date
- Partial

A warning should explain what needs attention rather than simply showing a red icon.

--------------------------------------------------------------------------------
## Incremental Changes
--------------------------------------------------------------------------------

After initial generation, users can continue editing.

For example:

> Make wooden blocks destructible.

The application should determine which modules are affected and regenerate only those where possible.

Changes may require:

- live update
- hot reload
- reload
- rebuild
- game restart
- package reinstall

The application should tell the user what is required.

--------------------------------------------------------------------------------
## Snapshots and Rollback
--------------------------------------------------------------------------------

Before substantial changes, create a snapshot.

Projects should support:

- snapshots
- history
- diffs
- rollback
- branches

A failed experiment should not destroy a working project.

--------------------------------------------------------------------------------
## Runtime Monitor
--------------------------------------------------------------------------------

While a mashup runs, expose:

- host process
- guest processes
- bridge state
- object counts
- events
- warnings
- errors
- performance
- synchronization state

Users should be able to trace important events.

Example:

Player input
→ PlaceBlock
→ BlockCreated
→ RenderProxyCreated
→ CollisionProxyCreated
→ NavigationUpdated

If the chain stops, the user can see where.

--------------------------------------------------------------------------------
## Testing
--------------------------------------------------------------------------------

A feature should not be considered complete merely because the code compiles.

Tests should cover:

- capability behavior
- integration
- runtime behavior
- compatibility
- regression
- failure behavior

For a block:

- spawns
- renders
- collides
- obstructs zombies
- affects navigation
- can be destroyed
- removes collision when destroyed
- persists if requested

--------------------------------------------------------------------------------
## Troubleshooting
--------------------------------------------------------------------------------

If something fails:

1. Check project status.
2. Check compatibility.
3. Check graph changes.
4. Inspect runtime logs.
5. Inspect event tracing.
6. Check authority.
7. Check synchronization.
8. Check the affected adapter.
9. Run a minimal reproduction.
10. Roll back if necessary.

Ask Claude questions such as:

> Diagnose why zombies walk through this block.

> Show me which connection failed.

> What has authority over this object's position?

> What changed since the last working snapshot?

--------------------------------------------------------------------------------
## Performance
--------------------------------------------------------------------------------

Cross-game bridges can be expensive.

The application should help users identify:

- unnecessary per-frame synchronization
- excessive object counts
- expensive conversions
- excessive logging
- unnecessary polling
- repeated navigation rebuilds

Prefer event-driven or batched communication where appropriate.

--------------------------------------------------------------------------------
## Local-First Generation
--------------------------------------------------------------------------------

The application must not require fal.ai or paid cloud generation credits.

Local workflows can use:

- ComfyUI
- Blender
- local AI models
- local scripts
- local converters
- user-installed game assets

Cloud generation may not become a required runtime dependency.

--------------------------------------------------------------------------------
## Sharing Projects
--------------------------------------------------------------------------------

Share the project recipe, not the game.

A project package may contain:

- graph
- rules
- source
- adapters
- converters
- manifests
- tests
- metadata
- compatibility requirements
- build instructions

Recipients should use their own game installations.

Do not package retail game files, unauthorized proprietary assets, or game executables.

--------------------------------------------------------------------------------
## Compatibility Migration
--------------------------------------------------------------------------------

When a recipient has a different supported game version, the application should identify:

- what is incompatible
- which adapters are affected
- which capabilities remain valid
- whether Claude can migrate the project

Do not silently pretend compatibility.

--------------------------------------------------------------------------------
## Multiplayer and Online Boundaries
--------------------------------------------------------------------------------

Local IPC and appropriate user-hosted/offline scenarios are supported architectural targets.

The application must not be designed to bypass:

- anti-cheat
- DRM
- ownership checks
- official online-service restrictions

The user guide should explain this clearly without making the workflow unnecessarily complicated.

--------------------------------------------------------------------------------
## Common Beginner Mistakes
--------------------------------------------------------------------------------

### Starting too large

Begin with one capability and one interaction.

### Treating assets as mechanics

A model is not a physics object or gameplay system.

### Assuming collision equals AI

Navigation may require a separate integration.

### Ignoring authority

Two systems fighting over one property causes unstable behavior.

### Changing too much at once

Use incremental changes.

### Not creating snapshots

Save working versions before experiments.

### Ignoring updates

Check compatibility when a previously working project breaks after a game update.

--------------------------------------------------------------------------------
## How to Ask Claude Effectively
--------------------------------------------------------------------------------

Good request:

> Add Minecraft blocks to L4D2. Start with rendering, placement, collision, and zombie obstruction. Do not add persistence yet.

Better than:

> Merge Minecraft and L4D2.

For advanced users:

> Use L4D2 physics as authority for block transforms while retaining Minecraft-derived placement semantics.

Users should be encouraged to describe desired behavior first and implementation details second.

--------------------------------------------------------------------------------
## The Five Questions Every User Should Learn
--------------------------------------------------------------------------------

Whenever adding a capability, ask:

1. What do I want?
2. What does it need to interact with?
3. Who has authority?
4. How should state synchronize?
5. How will I test it?

These five questions capture most of the application's architecture.

--------------------------------------------------------------------------------
## Recommended Learning Path
--------------------------------------------------------------------------------

Project 1:
One imported visual object.

Project 2:
One interactive object.

Project 3:
Object with collision.

Project 4:
Object affecting AI.

Project 5:
Persistent object.

Project 6:
Reusable rule.

Project 7:
Two donors.

Project 8:
Passthrough capability.

Project 9:
Mashup as donor.

Project 10:
Multi-system mashup.

--------------------------------------------------------------------------------
## Glossary
--------------------------------------------------------------------------------

Capability:
A reusable semantic ability or system.

Donor:
A game or project providing a capability.

Host:
The primary runtime in which the mashup experience operates.

Graph:
The declarative representation of the mashup architecture.

Node:
An entity, capability, system, rule, adapter, or other graph object.

Edge:
A relationship between graph objects.

Interaction Surface:
A system a capability must communicate with.

Interaction Contract:
The explicit specification of how two graph elements communicate.

Authority:
Which system has the final say over a property.

Synchronization:
How state or events move between systems.

Adapter:
A translation layer between incompatible systems.

Bridge:
A runtime communication mechanism.

Rule:
A declarative statement describing behavior.

Strategy:
The implementation approach used for a capability or connection.

Passthrough:
A strategy in which systems/processes remain separate and communicate locally.

Headless:
A simulation without requiring the donor's complete visual runtime.

Capability Pack:
A reusable package of capability definitions, rules, adapters, and supporting implementation.

Snapshot:
A restorable project state.

Compatibility Fingerprint:
Information used to determine whether a project matches the installed game/tool environment.

--------------------------------------------------------------------------------
## The Simplest Mental Model
--------------------------------------------------------------------------------

For a new user, remember:

    WHAT DO I WANT?
           ↓
       CAPABILITY
           ↓
    WHAT DOES IT INTERACT WITH?
           ↓
    GRAPH CONNECTIONS
           ↓
    WHO HAS AUTHORITY?
           ↓
    HOW DOES IT SYNCHRONIZE?
           ↓
    HOW SHOULD IT BE IMPLEMENTED?
           ↓
       BUILD / TEST
           ↓
          PLAY
           ↓
        ITERATE

That is the core Universal Mashup Studio workflow.

--------------------------------------------------------------------------------
## Product Philosophy for New Users
--------------------------------------------------------------------------------

A mashup is not successful merely because content from one game appears in another.

The objective is meaningful functional integration.

A block should behave like a block.

An explosion should behave like an explosion.

A vehicle should participate in physics.

An AI creature should participate in the host's world.

A portal should actually connect spaces.

A crafting system should actually affect inventory and item creation.

The graph exists to make those relationships explicit.

--------------------------------------------------------------------------------
# 153. HELP SYSTEM UI REQUIREMENTS
================================================================================

The built-in guide must be searchable.

Support:

- full-text search
- table of contents
- bookmarks
- recently viewed topics
- contextual links
- "Learn more" links from UI
- glossary links
- examples
- beginner/advanced filtering

The same concept should be reachable from multiple relevant contexts.

For example, "Authority" should be reachable from:

- graph edge editor
- authority editor
- runtime trace
- troubleshooting
- glossary

---

## 153.1 Explain This

Every major technical object should offer:

> Explain this

Claude should generate a concise project-specific explanation using the current graph and project state.

Example:

> Explain this connection.

Response:

> This connection makes the imported block create a host collision proxy. L4D2 owns collision resolution, while the mashup runtime keeps the proxy synchronized with the imported block.

The explanation should be concise and should not expose hidden chain-of-thought.

---

## 153.2 Show Me

Where possible, help should include:

> Show me

which focuses the UI on the relevant graph nodes, opens the appropriate editor, or launches a safe tutorial.

---

## 153.3 Beginner and Advanced Explanations

The same concept can have:

### Beginner

> Authority means who gets the final say.

### Advanced

> Authority defines the authoritative writer for a property within a synchronization domain and determines how conflicting updates are resolved.

Do not overwhelm beginners with implementation details unless requested.

---

# 154. CLAUDE'S ROLE IN THE HELP SYSTEM
================================================================================

Claude should be able to answer questions about the current project.

Examples:

> Why does this connection exist?

> What happens if I disable this rule?

> Which systems are affected if I remove this node?

> Why does this project require a bridge?

> Why can't this capability be implemented natively?

> Which parts of this project will need rebuilding?

Claude should base answers on:

- current graph
- project state
- game reconnaissance
- compatibility information
- relevant source
- runtime/test results

Do not answer project-specific questions from generic documentation alone.

---

# 155. CONTEXTUAL HELP SHOULD FOLLOW THE USER
================================================================================

If the user selects a graph edge and opens Help, the application should explain that edge.

If the user selects a strategy, explain the strategy in relation to the selected capability.

If the user selects an authority conflict, explain the actual conflicting systems.

The help system should therefore be integrated with application state.

---

# 156. USER GUIDE MAINTENANCE
================================================================================

The user guide is part of the product and must evolve with the application.

When a major feature is implemented:

1. Add documentation.
2. Add contextual help.
3. Add glossary terms if necessary.
4. Add an example.
5. Update onboarding if appropriate.

Do not allow UI terminology and documentation terminology to diverge.

The graph terminology should be consistent throughout the product.

---

# 157. DOCUMENTATION ACCEPTANCE CRITERIA
================================================================================

The built-in guide is complete when a new user can:

1. Understand what a mashup is.
2. Create a project.
3. Add a host.
4. Add a donor.
5. Describe a capability.
6. Understand the basic graph.
7. Understand a connection.
8. Understand authority at a basic level.
9. Understand implementation strategies.
10. Review Claude's proposed changes.
11. Build a project.
12. Run a test.
13. Launch a project.
14. Make a natural-language change.
15. Understand why a rebuild/restart is required.
16. Find logs.
17. Diagnose a basic failure.
18. Create a snapshot.
19. Reuse a rule.
20. Understand project sharing.
21. Understand that game files are not distributed.
22. Find advanced graph editing.
23. Understand how a mashup can become a donor.

---

# 158. FINAL IMPLEMENTATION INSTRUCTION
================================================================================

Treat the built-in New User Education and Help System as a first-class product subsystem.

It is not an afterthought.

The application should ship with:

- onboarding
- interactive tutorial
- searchable guide
- contextual help
- project-aware explanations
- glossary
- examples
- troubleshooting guidance
- beginner/advanced views

The documentation should teach the same architecture that the implementation uses.

Most importantly:

    USER INTENT
        ↓
    CAPABILITIES
        ↓
    UNIVERSAL MASHUP GRAPH
        ↓
    INTERACTION SURFACES
        ↓
    AUTHORITY / SYNCHRONIZATION
        ↓
    IMPLEMENTATION STRATEGIES
        ↓
    GENERATED IMPLEMENTATION
        ↓
    TESTING
        ↓
    RUNTIME
        ↓
    ITERATION

This mental model should be visible throughout the application.

The application should make advanced cross-game engineering approachable without hiding the underlying architecture from users who want to understand and control it.
