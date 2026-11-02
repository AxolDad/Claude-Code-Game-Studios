---
paths:
  - "assets/shaders/**"
  - "**/*.gdshader"
  - "**/*.{wesl,wgsl}"
  - "Assets/**/*.{shader,hlsl,cginc,compute,shadergraph}"
  - "Shaders/**"
  - "**/*.{usf,ush}"
---

# Shader Code Standards

All shader files, in every engine, must follow these standards to maintain
visual quality, performance, and cross-platform compatibility.

## Naming Conventions
- File naming: `[type]_[category]_[name].[ext]`
  - `spatial_env_water.gdshader` (Godot)
  - `SG_Env_Water` (Unity Shader Graph)
  - `M_Env_Water` (Unreal Material)
  - `env_water.wesl` (Bevy)
- Use descriptive names that indicate the material purpose
- Prefix with the engine's shader type — Godot `spatial_`, `canvas_`, `particles_`,
  `post_`; Unity `SG_` (Shader Graph); Unreal `M_`, `MI_` (instance), `MF_` (function).
  Bevy has no type prefix: lowercase with underscores

## Code Quality
- All uniforms/parameters must have descriptive names and appropriate hints
- Group related parameters (Godot: `group_uniforms`, Unity: `[Header]`, Unreal: Category, Bevy: a `var<uniform>` struct)
- Comment non-obvious calculations (especially math-heavy sections)
- No magic numbers — use named constants or documented uniform values
- Include authorship and purpose comment at the top of each shader file

## Performance Requirements
- Document the target platform and complexity budget for each shader
- Use appropriate precision: `half`/`mediump` on mobile where full precision isn't needed
- Minimize texture samples in fragment shaders
- Avoid dynamic branching in fragment shaders — use `step()`, `smoothstep()` and
  `mix()` (Godot, GLSL, WGSL, WESL) or `lerp()` (HLSL: Unity, Unreal)
- No texture reads inside loops
- Two-pass approach for blur effects (horizontal then vertical)

## Cross-Platform
- Test shaders on minimum spec target hardware
- Provide fallback/simplified versions for lower quality tiers
- Document which render pipeline the shader targets (Forward/Deferred, URP/HDRP, Forward+/Mobile/Compatibility)
- Do not mix shaders from different render pipelines in the same directory

## Variant Management
- Minimize shader variants — each variant is a separate compiled shader
- Document all keywords/variants and their purpose
- Use feature stripping where possible to reduce build size
- Log and monitor total variant count per shader

## Bevy: WESL and WGSL

Bevy 0.20 removed the naga_oil dialect. Check `docs/engine-reference/bevy/` for
current module paths before importing one.

- New shaders are WESL (`.wesl`). A plain `.wgsl` file with no preprocessor
  directives keeps working
- `#import`, `#ifdef`, `#define_import_path` and `#{NAME}` are the old dialect: a
  file using them must be translated to WESL and renamed `.wesl`
- Imports come first in the file and end with a semicolon:
  `import super::util::hsv_to_rgb;`
- Module names match the shader's path in its crate:
  `bevy_pbr::render::forward_io::VertexOutput`, not `bevy_pbr::forward_io`
- `@if`, `@elif` and `@else` replace `#ifdef`, on whole declarations, struct
  members, function parameters, imports and statements
- Int and UInt shader defs read as `constants::NAME`:
  `@group(constants::MATERIAL_BIND_GROUP) @binding(0)`
- GLSL support is gone. SPIR-V passthrough is unchanged
