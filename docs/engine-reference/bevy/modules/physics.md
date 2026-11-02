# Bevy Physics — Quick Reference

Last verified: 2026-11-02 | Engine: Bevy 0.20

## Bevy Does Not Provide Physics

**Bevy 0.20 has no physics engine: no `RigidBody`, `Collider`, joints, gravity, collision
events or character controller.** A search of every `bevy*-0.20.1` crate finds "rigid
body" and "collider" only in `examples/showcase/breakout.rs` (hand-written) and one doc
comment in `bevy_ecs`. Do not write `RigidBody::Dynamic` without a crate that provides it.
The choice is a third-party crate (`PLUGINS.md`) or game code.

## Third-Party Physics

| Crate | Version | Checked | Bevy 0.20 |
|-------|---------|---------|-----------|
| `avian3d`, `avian2d` | 0.8.0 | 2026-11-02 | Yes, requires `bevy ^0.20` |
| `bevy_rapier3d`, `bevy_rapier2d` | 0.37.0 | 2026-11-02 | Yes, requires `bevy ^0.20` |

```toml
[dependencies]
avian3d = "0.8"
```

Their APIs were not reviewed for this reference. Read their own docs for the version in
`Cargo.lock`.

## What Changed Since ~0.16 (LLM Cutoff)

- `RunFixedMainLoopSystem` → `RunFixedMainLoopSystems` (0.17)
- `bevy_shape` split out of `bevy_math`; `Bounded2d`, `Bounded3d`, `BoundingVolume`, `IntersectsVolume` now in the prelude (0.20)
- `MeshRayCast` lives in `bevy_picking`, under the `mesh_picking` feature

## Current API Patterns

These are what Bevy has for writing simple movement and queries by hand.

### Fixed Timestep

```rust
use bevy::prelude::*;

#[derive(Component)]
struct Velocity(Vec3);

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .insert_resource(Time::<Fixed>::from_hz(60.0))
        .add_systems(FixedUpdate, integrate)
        .run();
}

// In FixedUpdate, Res<Time> is Time<Fixed>
fn integrate(time: Res<Time>, mut bodies: Query<(&Velocity, &mut Transform)>) {
    for (velocity, mut transform) in &mut bodies {
        transform.translation += velocity.0 * time.delta_secs();
    }
}
```

`FixedUpdate` runs zero or more times per frame. For smooth rendering and no lost input,
split the work: accumulate input in `RunFixedMainLoopSystems::BeforeFixedMainLoop`, simulate
in `FixedUpdate`, interpolate in `RunFixedMainLoopSystems::AfterFixedMainLoop` using
`Time<Fixed>::overstep_fraction`. See `examples/movement/physics_in_fixed_timestep.rs`.

### Ray Cast Against Meshes

`MeshRayCast` is a system parameter, in the prelude under the `mesh_picking` feature. It
tests rendered meshes, so it is no replacement for colliders.

```rust
use bevy::prelude::*;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_systems(Update, cast)
        .run();
}

fn cast(mut ray_cast: MeshRayCast) {
    let ray = Ray3d::new(Vec3::new(0.0, 5.0, 0.0), Dir3::NEG_Y);
    for (entity, hit) in ray_cast.cast_ray(ray, &MeshRayCastSettings::default()) {
        info!("{entity} at {} normal {}", hit.point, hit.normal);
    }
}
```

### Bounding Volumes

Pure math, no world. You supply the broad phase. In the prelude; the module moved to
`bevy_shape` in 0.20, so `bevy::math::bounding` is the old path.

```rust
// fragment
let a = Aabb3d::new(Vec3::ZERO, Vec3::ONE);
let b = BoundingSphere::new(Vec3::new(1.5, 0.0, 0.0), 1.0);
let touching = a.intersects(&b);

let ray_cast = RayCast3d::from_ray(Ray3d::new(Vec3::NEG_X * 5.0, Dir3::X), 10.0);
let distance: Option<f32> = ray_cast.aabb_intersection_at(&a);
```

Also `AabbCast3d`, `BoundingSphereCast`, and the 2D forms `Aabb2d`, `BoundingCircle`,
`RayCast2d`, `AabbCast2d`, `BoundingCircleCast`.

### Debug Drawing

`Gizmos` draws wireframes in a system: `gizmos.cube(transform, color)`,
`gizmos.sphere(position, radius, color)`, `gizmos.line(a, b, color)`.

## Common Mistakes

- Writing `RigidBody`, `Collider` or `CollisionEvent` with no physics crate in `Cargo.toml`
- Moving by `delta_secs()` in `Update` and expecting deterministic physics. Use `FixedUpdate`
- `gizmos.cuboid` (renamed `cube` in 0.18)
- Adding a physics crate from an older tutorial at an old version. It will pull a second Bevy
- Using `.unwrap()` on `Query::single`. It returns `Result`

## Examples and Sources

`examples/movement/physics_in_fixed_timestep.rs`, `examples/ecs/fixed_timestep.rs`,
`examples/3d/mesh_ray_cast.rs`, `examples/math/bounding_2d.rs`.
https://bevy.org/learn/migration-guides/0-19-to-0-20/
