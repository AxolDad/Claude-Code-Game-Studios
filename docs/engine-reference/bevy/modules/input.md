# Bevy Input — Quick Reference

Last verified: 2026-11-02 | Engine: Bevy 0.20

## What Changed Since ~0.16 (LLM Cutoff)

- `InputSystem` → `InputSystems` (0.17)
- `InputFocus` methods: `.get()`, `.set(entity, cause)`, `.clear()` (0.19). `FocusCause::Auto` added (0.20)
- Input features: `keyboard`, `mouse`, `gamepad`, `touch`, `gestures`
- `Escape` in a text input releases focus and propagates (0.20)
- Pointer events are flat: `On<PointerPress>`, not `On<Pointer<Press>>` (0.20)
- Rumble requests are messages: `MessageWriter<GamepadRumbleRequest>` (0.17)

## What Bevy Provides

Raw state for keyboard, mouse, gamepad, touch and gestures, plus focus dispatch and
pointer picking.

**Not provided:** an action-mapping layer. `bevy_input` has no actions, bindings or
rebinding (source files: `axis`, `button_input`, `common_conditions`, `gamepad`,
`gestures`, `keyboard`, `mouse`, `touch`). Write a small mapping resource, or see
`PLUGINS.md`.

## Current API Patterns

### Poll Input

```rust
use bevy::{
    input::{common_conditions::input_just_pressed, mouse::AccumulatedMouseMotion},
    prelude::*,
};

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_systems(
            Update,
            (
                read_keyboard,
                read_mouse,
                read_gamepads,
                pause.run_if(input_just_pressed(KeyCode::Escape)),
            ),
        )
        .run();
}

fn read_keyboard(keys: Res<ButtonInput<KeyCode>>) {
    if keys.pressed(KeyCode::KeyW) {
        info!("forward");
    }
}

fn read_mouse(buttons: Res<ButtonInput<MouseButton>>, motion: Res<AccumulatedMouseMotion>) {
    if buttons.just_pressed(MouseButton::Left) {
        info!("mouse delta {}", motion.delta);
    }
}

fn read_gamepads(gamepads: Query<(Entity, &Gamepad)>) {
    for (entity, gamepad) in &gamepads {
        if gamepad.just_pressed(GamepadButton::South) {
            info!("{entity} jump");
        }
        if let Some(x) = gamepad.get(GamepadAxis::LeftStickX) {
            info!("{entity} stick {x}");
        }
    }
}

fn pause() {
    info!("paused");
}
```

| Device | API |
|--------|-----|
| Keyboard, physical | `Res<ButtonInput<KeyCode>>`: `pressed`, `just_pressed`, `just_released` |
| Keyboard, logical | `Res<ButtonInput<Key>>`, `Key::Character("?".into())` (`bevy::input::keyboard::Key`) |
| Mouse | `Res<ButtonInput<MouseButton>>`, `AccumulatedMouseMotion`, `AccumulatedMouseScroll` (`bevy::input::mouse`) |
| Gamepad | Gamepads are entities: `Query<(Entity, &Gamepad)>`, `gamepad.get(..)` returns `Option<f32>`, `left_stick()` returns `Vec2` |
| Touch | `Res<Touches>`: `iter_just_pressed`, `iter_just_released`, `iter_just_canceled`, `iter` |
| Conditions | `input_pressed`, `input_just_pressed`, `input_just_released`, `input_toggle_active` (`bevy::input::common_conditions`) |

### Rumble

```rust
// fragment
use bevy::input::gamepad::{GamepadRumbleIntensity, GamepadRumbleRequest};

rumble.write(GamepadRumbleRequest::Add {
    gamepad: entity,
    intensity: GamepadRumbleIntensity::strong_motor(0.1),
    duration: Duration::from_secs(1),
});
```

### Focus

`Res<InputFocus>` tracks the focused entity. `FocusedInput<KeyboardInput>` is an entity
event dispatched to it and bubbles. UI focus navigation: `modules/navigation.md`.

### Fixed Timestep

`FixedUpdate` may run zero times in a frame, so `just_pressed` read there can miss a
press. Accumulate in `RunFixedMainLoop` under `RunFixedMainLoopSystems::BeforeFixedMainLoop`
(`examples/movement/physics_in_fixed_timestep.rs`).

### Tests

```rust
// fragment
app.insert_resource(ButtonInput::<KeyCode>::default());
app.world_mut()
    .resource_mut::<ButtonInput<KeyCode>>()
    .press(KeyCode::Space);
app.update();
```

## Common Mistakes

- Reading `just_pressed` in `FixedUpdate`. See above
- Inventing an `ActionState` or `InputMap` in plain Bevy. They come from a third-party crate
- `gamepad.get(..).unwrap()`: it returns `Option`
- `Res<Gamepads>`. Gamepads are entities
- Text input eating keys: check `InputFocus` before treating a key as a game action

## Examples and Sources

`examples/input/`: `gamepad_input`, `gamepad_rumble`, `touch_input`.
`tests/how_to_test_apps.rs`.
https://bevy.org/learn/migration-guides/0-19-to-0-20/
