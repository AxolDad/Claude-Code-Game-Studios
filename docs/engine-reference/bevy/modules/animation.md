# Bevy Animation — Quick Reference

Last verified: 2026-11-02 | Engine: Bevy 0.20

## What Changed Since ~0.16 (LLM Cutoff)

- `AnimationTarget` split into `AnimationTargetId` + `AnimatedBy` (0.18)
- Feature `animation` renamed `gltf_animation` (0.18)
- `SceneRoot` → `WorldAssetRoot`; `SceneInstanceReady` → `WorldInstanceReady` (0.19)
- `MorphWeights` restructured; `AnimationTargetId` algorithm changed (0.19)
- Curves: `bevy_math::curve::*` → `bevy_curve::*` (0.20; prelude users unaffected)

## What Bevy Provides

Clips, graphs with blending and masks, additive blends, crossfades, morph targets,
animation events, curve-based clips from code, glTF import.

**Not provided:** an animation state machine. `AnimationTransitions` is described in
its source as "an unstable temporary API" that may be replaced by a state machine. No
editor or timeline tool. Sprite-sheet animation is game code driving
`Sprite::texture_atlas`; see `examples/2d/sprite_animation.rs`.

## Current API Patterns

### Play a glTF Clip

The `AnimationPlayer` lives on a descendant the glTF spawned, not on your root entity.

```rust
use bevy::{prelude::*, world_serialization::WorldInstanceReady};

const GLTF_PATH: &str = "models/animated/Fox.glb";

#[derive(Component)]
struct AnimationToPlay {
    graph_handle: Handle<AnimationGraph>,
    index: AnimationNodeIndex,
}

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_systems(Startup, setup)
        .run();
}

fn setup(
    mut commands: Commands,
    asset_server: Res<AssetServer>,
    mut graphs: ResMut<Assets<AnimationGraph>>,
) {
    let (graph, index) = AnimationGraph::from_clip(
        asset_server.load(GltfAssetLabel::Animation(2).from_asset(GLTF_PATH)),
    );
    let graph_handle = graphs.add(graph);

    commands
        .spawn((
            AnimationToPlay { graph_handle, index },
            WorldAssetRoot(asset_server.load(GltfAssetLabel::Scene(0).from_asset(GLTF_PATH))),
        ))
        .observe(play_when_ready);
}

fn play_when_ready(
    ready: On<WorldInstanceReady>,
    mut commands: Commands,
    children: Query<&Children>,
    to_play: Query<&AnimationToPlay>,
    mut players: Query<&mut AnimationPlayer>,
) {
    let Ok(animation) = to_play.get(ready.entity) else {
        return;
    };
    for child in children.iter_descendants(ready.entity) {
        if let Ok(mut player) = players.get_mut(child) {
            player.play(animation.index).repeat();
            commands
                .entity(child)
                .insert(AnimationGraphHandle(animation.graph_handle.clone()));
        }
    }
}
```

### Crossfade

Start every animation through `AnimationTransitions` once you use it: it manages all of
them and "will get confused" if they start directly on the player (example comment).

```rust
// fragment
let mut transitions = AnimationTransitions::new();
transitions
    .play(&mut player, index, Duration::ZERO)
    .repeat();
commands.entity(entity).insert(transitions);
```

### Graph and Events

| Need | API |
|------|-----|
| Many clips | `AnimationGraph::from_clips(clips)` returns `(graph, Vec<AnimationNodeIndex>)` |
| Add to a graph | `graph.add_clip(clip, weight, parent)`, `add_blend(weight, parent)`, `add_additive_blend` |
| Mask bones | `add_clip_with_mask`, `add_target_to_mask_group(target, group)` |
| Control | `player.play(i)` returns `&mut ActiveAnimation`: `.repeat()`, `.set_speed(s)`, `.seek_to(t)`, `.is_finished()` |
| Whole player | `stop(i)`, `stop_all()`, `pause_all()`, `all_finished()`, `animation_mut(i)` |

Events are observed:

```rust
// fragment
#[derive(AnimationEvent, Clone)] // bevy::animation::AnimationEvent
struct Footstep;

clip.add_event(0.5, Footstep);
app.add_observer(|_step: On<Footstep>| info!("step"));
```

### Clips from Code

```rust
// fragment
use bevy::animation::{animated_field, AnimatedBy, AnimationTargetId};

let target = AnimationTargetId::from_name(&name);
clip.add_curve_to_target(
    target,
    AnimatableCurve::new(animated_field!(Transform::translation), curve),
);
// The target entity needs both: `target` and `AnimatedBy(player_entity)`
```

## Common Mistakes

- Querying `AnimationPlayer` on the root entity. It is on a descendant
- Forgetting `AnimationGraphHandle` on the player's entity
- `SceneRoot`, `SceneInstanceReady`, `Scene` (renamed in 0.19), and `On<Ready>` (BSN's event, not glTF's)
- Calling `player.play` directly while `AnimationTransitions` is on the same entity
- Disabling default features and losing `gltf_animation`

## Examples and Sources

`examples/animation/`: `animated_mesh`, `animated_mesh_control`, `animated_mesh_events`,
`animated_transform`, `animation_events`, `animation_graph`, `animation_masks`,
`morph_targets`, `eased_motion`.
https://bevy.org/learn/migration-guides/0-17-to-0-18/,
https://bevy.org/learn/migration-guides/0-18-to-0-19/
