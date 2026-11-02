# Bevy Navigation — Quick Reference

Last verified: 2026-11-02 | Engine: Bevy 0.20

## Bevy Does Not Provide Pathfinding

**Bevy 0.20 has no navmesh, navigation agent or pathfinding of any kind.** A search of
every `bevy*-0.20.1` crate, including examples, for navmesh and pathfinding finds
nothing. Do not write `NavMesh`, `NavigationAgent` or `find_path` as if Bevy had them.
The choice is a third-party crate (`PLUGINS.md`) or game code.

"Navigation" in Bevy's own docs and API means **UI focus navigation**, below.

## What Changed Since ~0.16 (LLM Cutoff)

- Automatic directional UI navigation: `AutoDirectionalNavigation` (0.18)
- `InputFocus` accessors `.get()`, `.set(entity, cause)`, `.clear()` (0.19)
- `FocusCause::Auto` added (0.20)

## Current API Patterns

### World Pathfinding: Third-Party

| Crate | Version | Checked | Bevy 0.20 |
|-------|---------|---------|-----------|
| `vleue_navigator` | 0.17.0 | 2026-11-02 | Yes, requires `bevy ^0.20` |
| `oxidized_navigation` | 0.12.0 | 2026-11-02 | **No**, latest requires `bevy ^0.15` |

```toml
[dependencies]
vleue_navigator = "0.17"
```

Its API was not reviewed for this reference. Read its own docs for the version in
`Cargo.lock`. Bevy's own pieces are limited to what a pathfinder needs as input: `Mesh`,
`Transform`, `FixedUpdate`.

### Tab Navigation

`TabGroup` and `TabIndex` from `bevy::input_focus::tab_navigation`. Add
`TabNavigationPlugin`: it is not in `DefaultPlugins`.

```rust
use bevy::{
    input_focus::tab_navigation::{TabGroup, TabIndex, TabNavigationPlugin},
    prelude::*,
    ui_widgets::Button,
};

fn main() {
    App::new()
        .add_plugins((DefaultPlugins, TabNavigationPlugin))
        .add_systems(Startup, setup)
        .run();
}

fn setup(mut commands: Commands) {
    commands.spawn(Camera2d);
    commands.spawn((
        Node::default(),
        TabGroup::new(0),
        children![(Button, Node::default(), TabIndex(0))],
    ));
}
```

`TabGroup::modal()` traps focus inside a dialog.

### Directional (Gamepad or Arrow Keys)

Add `DirectionalNavigationPlugin`, mark each node with `AutoDirectionalNavigation`, then
call `AutoDirectionalNavigator` from a system with a `CompassOctant`. Neighbours come from
screen position.

```rust
// fragment
use bevy::{
    input_focus::directional_navigation::DirectionalNavigationPlugin,
    math::CompassOctant,
    ui::auto_directional_navigation::{AutoDirectionalNavigation, AutoDirectionalNavigator},
};

app.add_plugins(DirectionalNavigationPlugin);
commands.spawn((Button, Node::default(), AutoDirectionalNavigation::default()));

fn navigate(mut navigator: AutoDirectionalNavigator, keys: Res<ButtonInput<KeyCode>>) {
    if keys.just_pressed(KeyCode::ArrowUp) {
        let _ = navigator.navigate(CompassOctant::North);
    }
}
```

The manual form is `DirectionalNavigationMap` and `DirectionalNavigation` in
`bevy::input_focus::directional_navigation`, for overrides and non-UI entities.

## Common Mistakes

- Treating `bevy_input_focus` navigation as world pathfinding. It moves focus between entities
- Adopting `oxidized_navigation` from older tutorials. It has no 0.20 release
- Assuming `Transform` movement avoids obstacles. Without a pathfinder and a collider crate nothing does
- `AutoDirectionalNavigator` without `DirectionalNavigationPlugin`

## Examples and Sources

`examples/ui/navigation/directional_navigation.rs`, `examples/ui/widgets/tab_navigation.rs`.
https://bevy.org/learn/migration-guides/0-17-to-0-18/
