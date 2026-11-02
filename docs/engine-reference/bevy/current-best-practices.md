# Bevy — Current Best Practices

Last verified: 2026-11-02 | Engine: Bevy 0.20

Practices that are **new or changed** since the model's training data (~0.16).
Rust blocks whose first line is `// fragment` are not complete programs; the others
are. Blocks with `#[test]` are complete test files for `tests/`.

## Project Layout

Required: game code in `src/lib.rs` and a thin `src/main.rs`, so `tests/` can import
the game. A binary-only crate cannot be imported by an integration test. Assets live
in `assets/`, which is `AssetServer`'s root.

```rust
// fragment
// src/main.rs
use bevy::prelude::*;
use my_game::GamePlugin;

fn main() -> AppExit {
    App::new().add_plugins((DefaultPlugins, GamePlugin)).run()
}
```

```rust
// fragment
// src/lib.rs
use bevy::prelude::*;

pub mod combat;

pub struct GamePlugin;

impl Plugin for GamePlugin {
    fn build(&self, app: &mut App) {
        app.add_plugins(combat::plugin);
    }
}
```

Split systems into plugins, one per game system. A plain function is a plugin:
`pub fn plugin(app: &mut App) { .. }`, used as `app.add_plugins(combat::plugin)`.
Tests add the one plugin they need instead of `DefaultPlugins`.

## Cargo

```toml
[dependencies]
bevy = "0.20"

# Enable a small amount of optimization in the dev profile.
[profile.dev]
opt-level = 1

# Enable a large amount of optimization in the dev profile for dependencies.
[profile.dev.package."*"]
opt-level = 3
```

Features are collections since 0.18: `default` is `2d`, `3d`, `ui`, `audio`. `2d` and
`3d` do not imply `ui` or `audio`; a `default-features = false` build must name what
it wants. `bevy_feathers` is not in `default`. `dev` is `debug`, `bevy_dev_tools`,
`render_dev_tools` and `file_watcher`. `cargo run --features bevy/dynamic_linking`
speeds up iteration and is not for shipping.

## Testing

`App::new()` with no plugins is enough to run systems. `app.update()` runs one frame,
and the first call also runs `Startup`.

```rust
use bevy::prelude::*;

#[derive(Component)]
struct Health(i32);

fn apply_poison(mut targets: Query<&mut Health>) {
    for mut health in &mut targets {
        health.0 -= 1;
    }
}

#[test]
fn poison_tick_reduces_health_by_one() {
    let mut app = App::new();
    app.add_systems(Update, apply_poison);
    let target = app.world_mut().spawn(Health(5)).id();

    app.update();

    assert_eq!(app.world().get::<Health>(target).map(|h| h.0), Some(4));
}
```

Headless with time, input and states. `MinimalPlugins` is `TaskPool`, `FrameCount`,
`Time`, `ScheduleRunner` and (with its feature) `CiTesting`. It has no `StatesPlugin`,
`InputPlugin` or `AssetPlugin`: add what the test needs.

```rust
use bevy::{prelude::*, state::app::StatesPlugin, time::TimeUpdateStrategy};
use core::time::Duration;

#[derive(States, Debug, Clone, Copy, Default, Eq, PartialEq, Hash)]
enum GameState {
    #[default]
    Menu,
    Playing,
}

#[test]
fn next_state_applies_on_following_update() {
    let mut app = App::new();
    app.add_plugins((MinimalPlugins, StatesPlugin))
        .insert_resource(TimeUpdateStrategy::ManualDuration(Duration::from_millis(16)))
        .insert_resource(ButtonInput::<KeyCode>::default())
        .init_state::<GameState>();

    app.world_mut()
        .resource_mut::<NextState<GameState>>()
        .set(GameState::Playing);
    app.update();

    assert_eq!(*app.world().resource::<State<GameState>>().get(), GameState::Playing);
}
```

`TimeUpdateStrategy::FixedTimesteps(n)` runs the fixed loop exactly `n` times per
`app.update()`. `app.add_message::<T>()` is required before a system uses
`MessageReader<T>` or `MessageWriter<T>` in a test with no plugin registering it.

Test files: `tests/unit/main.rs`, `tests/integration/main.rs`, and `#[cfg(test)]`
modules in `src/`. A module is `tests/unit/[system].rs` or `tests/unit/[system]/mod.rs`
declared with `mod [system];` in that `main.rs`. **Anywhere else it is not compiled
and `cargo test` still exits 0.** Naming: file `[system].rs`, function
`[scenario]_[expected]`, e.g. `combat.rs` → `base_attack_returns_expected_damage()`.

Source: `tests/how_to_test_systems.rs`, `tests/how_to_test_apps.rs` in the Bevy repo.

## Schedules

| Schedule | Use |
|----------|-----|
| `Startup` | One-time setup. Runs on the first update, after the first `StateTransition` |
| `First` → `PreUpdate` → `StateTransition` → `RunFixedMainLoop` → `Update` → `SpawnScene` → `PostUpdate` → `Last` | Per-frame order |
| `FixedUpdate` | Fixed-rate game logic, zero or more times per frame. `Res<Time>` here is `Time<Fixed>` |
| `RunFixedMainLoop` with `RunFixedMainLoopSystems::{BeforeFixedMainLoop, AfterFixedMainLoop}` | Accumulate input before, interpolate after |

`.chain()` is strict ordering. `.chain_weak()`, `.before_weak()`, `.after_weak()` order
only where data access conflicts, for better parallelism. Bevy's built-in render and
UI sets use weak ordering since 0.20: put a real ECS dependency between systems
rather than sharing state through a channel or atomic.

Enable the `debug` feature and set `ScheduleBuildSettings.shuffle_seed` to expose
order-dependent bugs. With `bevy_dev_tools`, `bevy::dev_tools::states::log_transitions`
logs state changes. A panic in a system is caught and turned into an error passed to
`FallbackErrorHandler` (the default handler panics again).

## Messages and Observers

The training-data `Event` split in two in 0.17:

| Need | Use |
|------|-----|
| Buffered, read once per frame by a system | `#[derive(Message)]`, `MessageWriter::write`, `MessageReader::read`, `app.add_message` |
| Run immediately on trigger | `#[derive(Event)]`, `commands.trigger(..)`, `app.add_observer(..)` |
| Target one entity, optionally bubble up | `#[derive(EntityEvent)]` with an `entity` field, `.observe(..)`, `#[entity_event(propagate, auto_propagate)]` |
| React to component lifecycle | `On<Add<A>>`, `On<Insert<A>>`, `On<Discard<A>>`, `On<Remove<A>>`, `On<Despawn<A>>` |

```rust
use bevy::prelude::*;

#[derive(Component)]
struct Mine;

#[derive(EntityEvent)]
struct Explode {
    entity: Entity,
}

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_observer(on_add_mine)
        .add_systems(Startup, setup)
        .run();
}

fn setup(mut commands: Commands) {
    let mine = commands.spawn(Mine).observe(on_explode).id();
    commands.trigger(Explode { entity: mine });
}

fn on_add_mine(add: On<Add<Mine>>) {
    info!("mine added: {}", add.entity);
}

fn on_explode(explode: On<Explode>, mut commands: Commands) {
    commands.entity(explode.entity).despawn();
}
```

Wrong (the 0.17 to 0.19 form, a compile error now):

```rust
// fragment
fn on_add_mine(add: On<Add, Mine>) {}
```

Observers accept conditions: `app.add_observer(on_click.run_if(in_state(GameState::Playing)))`.

## Queries and Fallible Systems

Systems may return `Result`, so `?` works in them. `.context(..)` and `.with_context(..)`
come from `ContextExt` in the prelude. `Query::single` returns `Result`; `Single<&T>` as
a parameter skips the system when there is no unique match. `iter_many` yields `Result`:

```rust
// fragment
fn log_total(list: Res<Roster>, health: Query<&Health>) {
    // .matched() keeps only entities the query matches, the pre-0.20 behaviour
    let total: i32 = health.iter_many(&list.0).matched().map(|h| h.0).sum();
    info!("total health {total}");
}
```

`commands.despawn_all::<With<Enemy>>()` despawns by filter;
`commands.despawn_all_where::<&Health, ()>(|h| h.0 <= 0)` checks a value.

## States

```rust
use bevy::prelude::*;

#[derive(States, Debug, Clone, Copy, Default, Eq, PartialEq, Hash)]
enum AppState {
    #[default]
    Menu,
    InGame,
}

#[derive(Component)]
struct MenuUi;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .init_state::<AppState>()
        .add_systems(OnEnter(AppState::Menu), spawn_menu)
        .add_systems(Update, start.run_if(in_state(AppState::Menu)))
        .run();
}

fn spawn_menu(mut commands: Commands) {
    commands.spawn((MenuUi, Node::default(), DespawnOnExit(AppState::Menu)));
}

fn start(keys: Res<ButtonInput<KeyCode>>, mut next: ResMut<NextState<AppState>>) {
    if keys.just_pressed(KeyCode::Enter) {
        next.set_if_different(AppState::InGame);
    }
}
```

`set` always fires `OnExit`/`OnEnter`, even for the state you are already in (since
0.18), and `DespawnOnExit`/`DespawnOnEnter` fire too (since 0.19). Use
`set_if_different` to skip same-state transitions. `add_computed_state` and
`add_sub_state` exist; see `examples/state/`.

## Scenes: `bsn!`

Scenes are code-defined with `bsn!` and `bsn_list!`. **The `.bsn` asset format does not
exist yet.** The crate docs say Bevy "does not currently have support for `.bsn` files"
and the 0.20 notes list it as future work. Never present it as the scene format.

```rust
use bevy::{prelude::*, text::FontSourceTemplate, ui_widgets::Button};

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_systems(Startup, scene.spawn())
        .run();
}

fn scene() -> impl SceneList {
    bsn_list! { Camera2d -- @ui() }
}

fn ui() -> impl Scene {
    bsn! {
        Node { width: percent(100), height: percent(100), column_gap: px(5), }
        Children [
            @button("Ok")
            on(|_event: On<PointerPress>| println!("Ok pressed!"))
            --
            @button("Cancel")
            BackgroundColor(Color::srgb(0.4, 0.15, 0.15))
        ]
    }
}

fn button(label: &str) -> impl Scene {
    bsn! {
        Button
        Node { width: px(150), height: px(65), border: px(5), }
        BackgroundColor(Color::srgb(0.15, 0.15, 0.15))
        Children [
            Text(label)
            TextFont { font: FontSourceTemplate::Handle("fonts/FiraSans-Bold.ttf"), font_size: px(33.0), }
        ]
    }
}
```

| Rule | Detail |
|------|--------|
| References | `@scene_function()`, `@variable`, `@{expr}` |
| Separator | `--` between entities |
| Patching | `Node { width: px(10), }` sets one field. No `..default()` inside `bsn!` |
| Different syntax | Inside `bsn!`: `Text(label)`, `font_size: px(33.0)`. In plain Rust: `Text::new(..)`, `font_size: FontSize::Px(33.0)` |
| Types in `bsn!` | Derive `Default + Clone` (preferred), or `FromTemplate` when a field needs spawn context such as `Handle` or `Entity`. Not both |
| Spawning | `scene.spawn()` as a system, `commands.spawn_scene(..)`, `commands.queue_spawn_scene(..)` (waits for dependencies), `world.spawn_scene(..)` returns `Result` |
| Ready | `On<Ready>` fires for each entity in a scene after it and its children have spawned |
| Scene components | `#[derive(SceneComponent)]` with `fn scene() -> impl Scene`; spawned only through scene APIs |

glTF is not BSN. It still spawns through `WorldAssetRoot` and fires
`On<WorldInstanceReady>`. World round-trips use `bevy_world_serialization`.

## Shaders: WESL

Write new shaders as `.wesl`. Plain `.wgsl` with no preprocessor directives keeps
working; the naga_oil `#import` and `#ifdef` dialect does not. GLSL is gone.
`modules/rendering.md` has the syntax. The shader loader accepts `spv`, `wgsl`, `wesl`.

## UI: Widgets

`bevy_ui_widgets` is in `default` through `ui`, and `DefaultPlugins` adds
`UiWidgetsPlugins`. Widgets are headless: behaviour with no styling, and external
state: the app updates the state component in response to `ValueChange`.
`Hovered` and `Pressed` replace `Interaction`. `modules/ui.md` has the example.

**Feathers** (`bevy_feathers` feature, `FeathersPlugins`) is an editor-oriented toolkit.
Its crate docs say it is "deliberately not intended" for game UI. Use it for tools and
inspectors, not the game's HUD.

## Data Files

`assets/data/`: JSON (validated by hooks), RON and TOML (NOT validated by hooks). Prefer
JSON where a hook's validation matters. Gameplay values go in data files, not constants.

## Command Line

| Purpose | Command |
|---------|---------|
| Parse and type check (`commands.smoke`) | `cargo check` |
| Tests (`commands.test`) | `cargo test --no-fail-fast` |
| Release build (`commands.build`) | `cargo build --release` |
| Toolchain probe | `cargo --version`, `rustc --version` |

`cargo test` exiting 0 does not prove tests ran: an undeclared test file is not
compiled, and a filter matching nothing also exits 0. `.claude/scripts/test-run.sh bevy`
prints `uncompiled <path>` for unreached test files and flags a zero-test run. `cargo
check` needs no GPU and runs nothing.

Linux CI needs: `g++ pkg-config libx11-dev libasound2-dev libudev-dev libxkbcommon-x11-0 libwayland-dev libxkbcommon-dev`.

### Screenshot Capture

Enable the `bevy_ci_testing` feature and name a RON file in `CI_TESTING_CONFIG`:

```ron
(
    events: [
        (60, ScreenshotAndExit),
    ],
)
```

```bash
CI_TESTING_CONFIG=screenshot.ron cargo run --features bevy/bevy_ci_testing
```

Writes `./screenshot-60.png`, relative to the working directory, and exits 0. The path
is fixed to `./screenshot-<frame>.png`. Copy it to `production/qa/evidence/`. Needs a
GPU and a window; without them the result is `NOT VERIFIED`.

| Case | Behaviour |
|------|-----------|
| `CI_TESTING_CONFIG` unset | Reads `ci_testing_config.ron` from the working directory |
| File missing or unreadable | Warns, uses an empty config, **and the app never exits by itself** |
| Malformed RON | Panics |
| `NamedScreenshot("x")` then a later `AppExit` | **Do not use.** Observed on 0.20.1: exits 0, logs "Took a screenshot", writes no file, because the app exits before the save finishes. Use `ScreenshotAndExit` |
| `setup: (fixed_frame_time: Some(0.03))` | Optional. Sets `TimeUpdateStrategy::ManualDuration`, for repeatable frames |
| `CiTestingConfig` resource already inserted | Used instead of the file |

Bevy's docs recommend this plugin only during testing. Keep the feature out of release
builds. The in-code way:

```rust
// fragment
use bevy::render::view::screenshot::{save_to_disk, Screenshot};

fn capture(mut commands: Commands) {
    commands
        .spawn(Screenshot::primary_window())
        .observe(save_to_disk("screenshot.png"));
}
```

## Sources

- https://bevy.org/news/bevy-0-20/
- https://bevy.org/learn/migration-guides/0-19-to-0-20/
- https://bevy.org/learn/quick-start/getting-started/setup/ (profile settings only; its version line is stale)
- Bevy 0.20.1 source: `crates/bevy_dev_tools/src/ci_testing/`, `crates/bevy_scene/src/lib.rs`, `crates/bevy_feathers/src/lib.rs`, `tests/`, `examples/`
