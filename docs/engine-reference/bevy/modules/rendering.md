# Bevy Rendering — Quick Reference

Last verified: 2026-11-02 | Engine: Bevy 0.20

## What Changed Since ~0.16 (LLM Cutoff)

- WESL (`.wesl`) replaces the naga_oil dialect; GLSL support removed (0.20)
- `ScreenSpaceTransmission` is opt-in on `Camera3d` (0.20)
- `Sprite` uses `Mesh2d` + material; `alpha_mode` field; same-Z order may change (0.20)
- `Tonemapping::None` is a full passthrough; `Tonemapping::Linear` added and the `Camera2d` default (0.20)
- Render graph API removed: passes are systems in `Core2d` and `Core3d` (0.19)
- `shadows_enabled` → `shadow_maps_enabled` (0.19); `RenderTarget` is a component (0.18); `Hdr` is a component (0.17); `AmbientLight` resource → `GlobalAmbientLight` (0.18)
- New: contact shadows, vignette, lens distortion (0.19); mesh shaders (advanced, 0.20); Solari raytraced lighting (experimental, `bevy_solari` feature)

## Current API Patterns

### Custom 3D Material (WESL)

```rust
use bevy::{
    prelude::*, reflect::TypePath, render::render_resource::AsBindGroup, shader::ShaderRef,
};

const SHADER_ASSET_PATH: &str = "shaders/custom_material.wesl";

#[derive(Asset, TypePath, AsBindGroup, Debug, Clone)]
struct CustomMaterial {
    #[uniform(0)]
    color: LinearRgba,
    #[texture(1)]
    #[sampler(2)]
    color_texture: Option<Handle<Image>>,
}

impl Material for CustomMaterial {
    fn fragment_shader() -> ShaderRef {
        SHADER_ASSET_PATH.into()
    }
}

fn main() {
    App::new()
        .add_plugins((DefaultPlugins, MaterialPlugin::<CustomMaterial>::default()))
        .add_systems(Startup, setup)
        .run();
}

fn setup(
    mut commands: Commands,
    mut meshes: ResMut<Assets<Mesh>>,
    mut materials: ResMut<Assets<CustomMaterial>>,
) {
    commands.spawn((
        Mesh3d(meshes.add(Cuboid::default())),
        MeshMaterial3d(materials.add(CustomMaterial {
            color: LinearRgba::BLUE,
            color_texture: None,
        })),
    ));
    commands.spawn((
        Camera3d::default(),
        Transform::from_xyz(-2.0, 2.5, 5.0).looking_at(Vec3::ZERO, Vec3::Y),
    ));
}
```

### The Shader

From `assets/shaders/custom_material.wesl` at v0.20.1:

```wgsl
import bevy_pbr::render::forward_io::VertexOutput;
// relative import from the assets folder
import super::custom_material_import::COLOR_MULTIPLIER;

@group(constants::MATERIAL_BIND_GROUP) @binding(0) var<uniform> material_color: vec4<f32>;
@group(constants::MATERIAL_BIND_GROUP) @binding(1) var material_color_texture: texture_2d<f32>;
@group(constants::MATERIAL_BIND_GROUP) @binding(2) var material_color_sampler: sampler;

@fragment
fn fragment(mesh: VertexOutput) -> @location(0) vec4<f32> {
    return material_color
        * textureSample(material_color_texture, material_color_sampler, mesh.uv)
        * COLOR_MULTIPLIER;
}
```

| naga_oil (wrong in 0.20) | WESL |
|--------------------------|------|
| `#import bevy_pbr::forward_io::VertexOutput` | `import bevy_pbr::render::forward_io::VertexOutput;` |
| `#import "shaders/util.wgsl"::hsv_to_rgb` | `import super::util::hsv_to_rgb;` |
| `#ifdef VERTEX_COLORS` | `@if(VERTEX_COLORS)` (also `@elif`, `@else`) |
| `@group(#{MATERIAL_BIND_GROUP})` | `@group(constants::MATERIAL_BIND_GROUP)` |
| `#define_import_path my::module` | Removed: module name comes from the path |

The import needs the sibling `custom_material_import.wesl` from the Bevy repo; without it,
drop that line and the multiply. Plain `.wgsl` with no directives still loads, but cannot
use these imports. Language server: wgsl-analyzer.

### Custom Sprite Material

Implement `MaterialExtension2d`, add `SpriteMaterialPlugin::<M>::default()`, and put
`SpriteMaterial(handle)` on the sprite entity. Custom uniforms start at binding 20 to
avoid the sprite's own. Full version: `examples/shader/sprite_material.rs`.

### Camera and Light Components

```rust
// fragment
use bevy::{
    camera::Hdr, core_pipeline::tonemapping::Tonemapping, pbr::ScreenSpaceTransmission,
    post_process::bloom::Bloom,
};

commands.spawn((
    Camera3d::default(),
    Hdr,                                 // Not a Camera field, and not in the prelude
    Tonemapping::TonyMcMapface,
    Bloom::NATURAL,
    ScreenSpaceTransmission::default(),  // Needed for glass since 0.20
));
commands.spawn(DirectionalLight { shadow_maps_enabled: true, ..default() });
app.insert_resource(GlobalAmbientLight { brightness: 2000.0, ..default() });
```

Controllers are first-party (each behind a feature): `free_camera` →
`bevy::camera_controller::free_camera::{FreeCamera, FreeCameraPlugin}`; `pan_camera` →
`...::pan_camera::{PanCamera, PanCameraPlugin}`; `pan_orbit_camera` →
`...::pan_orbit_camera::prelude::*`, whose minimal example also wires up its own input plugin.

Under `render_dev_tools`, `RenderDebugOverlay` is in the default plugins and disabled. Enable
the keys (F1, F2): `RenderDebugOverlayKeybindings { enable_keybindings: true, ..Default::default() }`.

## Common Mistakes

- `SceneRoot`, `shadows_enabled`, `Camera { hdr: true, .. }`, or `AmbientLight` as a resource
- `#import` in a `.wesl` file. Imports use `import x::y;` and come first
- Glass looks opaque: add `ScreenSpaceTransmission` to the camera
- Overlapping sprites at one Z. Set distinct Z values
- Custom sprite material uniforms below binding 20

## Examples and Sources

`examples/shader/`, `examples/shader_advanced/` (`fullscreen_material`, `mesh_shader_intro`,
`custom_post_processing`), `examples/camera/`, `examples/3d/`.
https://bevy.org/news/bevy-0-20/, https://bevy.org/learn/migration-guides/0-19-to-0-20/
