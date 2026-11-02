# Agent Test Spec: bevy-render-specialist

## Agent Summary
Domain: Bevy 0.20 rendering: WESL and WGSL shaders, translation from the naga_oil dialect, materials for meshes, sprites and full-screen passes, cameras and tonemapping, render features, and render-world code.
Does NOT own: UI layout and widgets (bevy-ui-specialist), asset loading and texture processing setup (bevy-assets-specialist), system ordering outside the render world (bevy-ecs-specialist), gameplay code (gameplay-programmer), visual direction (art-director).
No gate IDs assigned.

---

## Static Assertions (Structural)

- [ ] `description:` field is present and domain-specific (references materials / WESL / naga_oil / SpriteMaterial)
- [ ] `tools:` list includes Read, Write, Edit, Bash, Glob, Grep
- [ ] Model tier is `sonnet` — frontmatter `model:` reads exactly `sonnet` (tiers: `.claude/docs/model-tiers.md`)
- [ ] Agent definition references `docs/engine-reference/bevy/VERSION.md`, `deprecated-apis.md` and `breaking-changes.md`, and sends uncertain APIs to docs.rs or the crate source
- [ ] Agent definition has a naga_oil to WESL translation covering `#import`, `#ifdef` and `#{MATERIAL_BIND_GROUP}`
- [ ] Agent definition distinguishes `SpriteMaterial<M>` from `SpriteMeshMaterial`
- [ ] Agent definition tells the agent to search shaders with `glob: "*.{wesl,wgsl}"`, because ripgrep has no `wesl` type
- [ ] Agent definition states there is no shader graph or built-in particle crate, and routes third-party crates to `docs/engine-reference/bevy/PLUGINS.md`

---

## Test Cases

### Case 1: In-domain request — appropriate output
**Input:** "Make a custom 3D material with a tint color and a texture."
**Expected behavior:**
- Writes a Rust material deriving `AsBindGroup`, `Asset` and the rest, with `#[uniform(0)]`, `#[texture(1)]` and `#[sampler(2)]`
- Implements `Material`, adds `MaterialPlugin::<M>::default()`, and spawns with `MeshMaterial3d`
- Writes a `.wesl` shader under `assets/shaders/` whose imports end in semicolons and come first
- Binds with `@group(constants::MATERIAL_BIND_GROUP)`, not a literal group number, and matches the binding numbers to the Rust attributes
- Offers `ExtendedMaterial` as the alternative if the material should keep `StandardMaterial` lighting
- Does NOT write GLSL or HLSL

### Case 2: Out-of-domain redirect
**Input:** "Build the HUD with a health bar and a pause button."
**Expected behavior:**
- Does NOT build the layout or widgets
- Redirects to `bevy-ui-specialist`
- Notes that it would own a custom `UiMaterial` shader for the bar, if the HUD needs one

### Case 3: Version awareness — pasted naga_oil shader
**Input:** "This worked on 0.19. Why does it fail now?
```wgsl
#import bevy_pbr::forward_io::VertexOutput
#import "shaders/util.wgsl"::hsv_to_rgb
@group(2) @binding(0) var<uniform> tint: vec4<f32>;
@fragment
fn fragment(in: VertexOutput) -> @location(0) vec4<f32> {
#ifdef VERTEX_COLORS
    return in.color * tint;
#endif
    return tint;
}
```"
**Expected behavior:**
- States that the naga_oil dialect is gone and shaders with preprocessor directives must be translated and renamed to `.wesl`
- Rewrites the imports as `import bevy_pbr::render::forward_io::VertexOutput;` and `import super::util::hsv_to_rgb;`, noting the changed module path
- Replaces `#ifdef` with `@if(VERTEX_COLORS)`, attached to whole statements or declarations
- Replaces the literal group with `constants::MATERIAL_BIND_GROUP`
- Does NOT suggest a `shader_format_wesl` feature, which no longer exists

### Case 4: Sprite materials and the renamed type
**Input:** "I want a custom shader on my sprites. I found `bevy_sprite_render::SpriteMaterial` in an old post, do I edit that?"
**Expected behavior:**
- Explains the name trap: the old internal `SpriteMaterial` is now `SpriteMeshMaterial`, and `SpriteMaterial<M>` is the new user-facing component
- Implements `MaterialExtension2d`, adds `SpriteMaterialPlugin::<M>::default()`, and inserts `SpriteMaterial<M>` on the sprite
- Uses `sample_final_color` or `sample_sprite_texture` from `bevy_sprite_render::sprite_mesh::functions`
- Warns that draw order at equal Z may change, and sets explicit Z on sprites that can overlap

### Case 5: Camera defaults that changed
**Input:** "After upgrading, my glass no longer refracts, and I set `Tonemapping::None` to remove the curve but the colors are off."
**Expected behavior:**
- Explains that `ScreenSpaceTransmission` is opt-in now and adds it to the `Camera3d`
- Explains that `Tonemapping::None` is a full passthrough, and uses `Tonemapping::Linear` to remove the curve only
- Checks `breaking-changes.md` rather than diagnosing from memory

### Case 6: No built-in solution
**Input:** "Set up the particle system for our spell effects, and is there a shader graph for the artist?"
**Expected behavior:**
- States that the 0.20.1 crate set has no particle or VFX crate, and no shader graph
- Offers a material or compute approach, or sends the crate choice through `docs/engine-reference/bevy/PLUGINS.md`, and says a crate not listed there is unverified
- Does NOT invent a `bevy_particles` API
- Treats adding a crate as a `bevy-specialist` and `technical-director` decision

---

## Protocol Compliance

- [ ] Stays within declared domain (shaders, materials, cameras, render features, render-world code)
- [ ] Redirects UI, asset loading and gameplay work to the owning agent
- [ ] Reads the Bevy reference files before suggesting version-qualified APIs, and marks anything unverified rather than asserting it from memory
- [ ] Produces WESL with no `#import`, `#ifdef` or `#{...}`
- [ ] Does not present unmeasured GPU timings as fact
- [ ] Asks "May I write this to [filepath]?" naming the file before writing

---

## Coverage Notes
- Material and sprite cases (1, 4) should have a retained screenshot in `production/qa/evidence/`; shader output is not automated
- Case 3 is the primary version-awareness check
- Case 4 verifies the agent does not confuse the two `SpriteMaterial` types, which a search of old posts would mix up
