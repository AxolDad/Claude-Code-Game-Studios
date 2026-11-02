# Bevy Configuration — setup-engine reference

Lookup tables and scaffold files for `/setup-engine`, split out of the skill body
so they cost nothing on invocations that are not Bevy. Read this file only when
the chosen engine is Bevy.

Referenced from Sections 4, 5, 5.5 and 7.5 of `/setup-engine` by the names B1, B2,
B3, B4 and B5. Bevy has no language choice, so each subsection has one variant.

**In this file:**

- B1. CLAUDE.md Technology Stack Template
- B2. Naming Conventions
- B3. Engine Specialists Routing
- B4. Scaffold Files
- B5. Screenshot Capture

---

### B1. CLAUDE.md Technology Stack Template

```markdown
- **Engine**: Bevy [version]
- **Language**: Rust
- **Build System**: Cargo
- **Asset Pipeline**: Bevy `AssetServer` loading from `assets/`
```

---

### B2. Naming Conventions

- Types (structs, components, resources, plugins): PascalCase (e.g., `PlayerController`)
- Variables/functions/systems: snake_case (e.g., `move_speed`)
- Events and messages: PascalCase past tense types (e.g., `HealthChanged`), as in Bevy's own `WindowResized`, `CursorMoved` and `ScreenshotCaptured`
- Files: snake_case (e.g., `player_controller.rs`)
- Scenes: snake_case; scenes are defined in code with `bsn!`, and the `.bsn` file format is listed as future work in the 0.20 notes, so never name a `.bsn` file as the scene format
- Constants: SCREAMING_SNAKE (e.g., `MAX_HEALTH`)

> **This list and the `naming` enum table in Section 5.5.1 of the skill must
> agree, value for value.** They are written into two different files —
> `technical-preferences.md` from here, `project.yaml` from there — and Section
> 5.5.3 compares them and **stops the skill** on any mismatch.

---

### B3. Engine Specialists Routing

```markdown
## Engine Specialists
- **Primary**: bevy-specialist
- **Language/Code Specialist**: bevy-ecs-specialist (all .rs files)
- **Shader Specialist**: bevy-render-specialist (.wesl and .wgsl files, materials, cameras)
- **UI Specialist**: bevy-ui-specialist (bevy_ui, Feathers, widgets, picking, input focus)
- **Additional Specialists**: bevy-assets-specialist (asset loading and processing, glTF, BSN, data files)
- **Routing Notes**: Invoke primary for architecture, plugin structure, Cargo features, performance and cross-cutting review. Invoke ECS specialist for systems, schedules, observers, states, queries and Rust patterns. Invoke render specialist for materials, shaders, cameras and render features. Invoke UI specialist for all interface implementation. Invoke assets specialist for loading, processing, glTF, BSN and data files.

### File Extension Routing

| File Extension / Type | Specialist to Spawn |
|-----------------------|---------------------|
| Game code (.rs files) | bevy-ecs-specialist |
| Shader / material files (.wesl, .wgsl) | bevy-render-specialist |
| UI / screen files (.rs modules using bevy_ui, Feathers) | bevy-ui-specialist |
| Scene / prefab / level files (`bsn!` scenes in .rs, .gltf, .glb) | bevy-assets-specialist |
| Data files (assets/data/: .json, .ron, .toml) | bevy-assets-specialist |
| Native extension / plugin files (Cargo.toml, Cargo features, third-party crates) | bevy-specialist |
| General architecture review | bevy-specialist |
```

---

### B4. Scaffold Files

Section 7.5 writes these exactly, each only if absent. This crate layout was built
and tested with `cargo test` on Bevy 0.20.1 before it was recorded here.

| Placeholder | Value |
|-------------|-------|
| `<major.minor>` | `engine.version` from Section 3, major and minor only (`0.20.0` → `0.20`) |
| `<crate>` | the brief's working title, else the repo directory name, as lowercase snake_case: letters, digits and underscores, starting with a letter. If that gives a Rust keyword or `bevy`, `core`, `std` or `test`, append `_game` |

**`Cargo.toml`:**
```toml
[package]
name = "<crate>"
version = "0.1.0"
edition = "2024"

[dependencies]
bevy = "<major.minor>"

[profile.dev]
opt-level = 1

[profile.dev.package."*"]
opt-level = 3
```

**`src/lib.rs`:**
```rust
//! Game code lives in the library crate so tests can import it.
use bevy::prelude::*;

/// The game's root plugin.
pub struct GamePlugin;

impl Plugin for GamePlugin {
    fn build(&self, _app: &mut App) {}
}
```

**`src/main.rs`:**
```rust
use bevy::prelude::*;
use <crate>::GamePlugin;

fn main() {
    App::new().add_plugins((DefaultPlugins, GamePlugin)).run();
}
```

**`assets/.gitkeep`:** empty.

---

### B5. Screenshot Capture

What `commands.run` leads to later, so Section 5.5 can state it: `/dev-story` writes a
RON file holding `(events: [(60, ScreenshotAndExit)])` and runs
`CI_TESTING_CONFIG=<file>.ron cargo run --features bevy/bevy_ci_testing`, adding
`--bin <name>` when the project has more than one binary. The file name is fixed to
`./screenshot-<frame>.png`, so it is then moved to
`production/qa/evidence/[story-slug]/[NN]-[what-it-shows].png`. Exit 0 is not
evidence: check the PNG exists, and never pair a screenshot event with a fixed
`AppExit`. The game needs a camera for anything to be drawn, and a GPU and a window to
run at all. The full procedure is in `.claude/docs/run-and-observe.md`.
