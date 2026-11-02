# Phase 4: Test helpers on Bevy

Read only when the engine is Bevy. Every path here is under `tests/helpers/`, the
helper root in this skill's table.

Cargo compiles `tests/unit/main.rs` and `tests/integration/main.rs` as separate
crates and finds nothing else under `tests/`, so each crate that uses the helpers
declares them. A helper file no `mod` reaches is not compiled and nothing reports
it: `test-run.sh bevy` lists `uncompiled` files only under `tests/unit/` and
`tests/integration/`.

**In each `main.rs`:**

```rust
#[path = "../helpers/mod.rs"]
mod helpers;
```

**Module file** (`tests/helpers/mod.rs`):

```rust
//! Shared helpers for [Project Name] tests. Each test crate uses only some of them.
#![allow(dead_code)]

pub mod game_assertions;
pub mod game_factory;
```

**Base helper** (`tests/helpers/game_assertions.rs`):

```rust
/// Assert a value is within the inclusive range [min, max].
/// Use for any formula output that has defined bounds in a GDD.
#[track_caller]
pub fn assert_in_range(value: f32, min: f32, max: f32, label: &str) {
    assert!(
        (min..=max).contains(&value),
        "{label} {value:.2} is outside expected range [{min:.2}, {max:.2}]"
    );
}
```

**Factory helper** (`tests/helpers/game_factory.rs`):

```rust
use bevy::prelude::*;

/// An app with no window or GPU: add the plugins under test, then call `app.update()`.
pub fn make_app() -> App {
    let mut app = App::new();
    app.add_plugins(MinimalPlugins);
    app
}
```
