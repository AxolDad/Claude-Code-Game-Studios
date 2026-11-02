# Bevy — Breaking Changes

Last verified: 2026-11-02 | Engine: Bevy 0.20

Changes between Bevy versions, newest first. The 0.19 → 0.20 guide has 75 entries;
this file keeps the ones game code hits and groups them by risk. Entries that only
touch reflection internals, trait plumbing or the render world are omitted unless a
game with custom rendering would meet them (those sit in their own table). Older
versions are summaries: read the linked guide before migrating across one.

## 0.19 → 0.20 (Oct 2026 — POST-CUTOFF, HIGH RISK)

Guide: https://bevy.org/learn/migration-guides/0-19-to-0-20/

### HIGH RISK — Compile Breaks in Everyday Code

| Subsystem | Change | Details |
|-----------|--------|---------|
| Observers | `On<Add, A>` → `On<Add<A>>` | `Add`, `Insert`, `Discard`, `Remove`, `Despawn` carry the bundle as a generic. For dynamic components: `On<Add<()>>` with `.with_component(id)`. A custom `On<Foo, Bar>` becomes `impl EventPattern` |
| Picking | `On<Pointer<Press>>` → `On<PointerPress>` | Flat events: `PointerCancel`, `PointerOver`, `PointerEnter`, `PointerOut`, `PointerLeave`, `PointerPress`, `PointerRelease`, `PointerClick`, `PointerMove`, `PointerDragStart`, `PointerDrag`, `PointerDragEnd`, `PointerDragEnter`, `PointerDragOver`, `PointerDragLeave`, `PointerDragDrop`, `PointerScroll`. `press.pointer_id` → `press.pointer.id`; `press.pointer_location.position` → `press.pointer.position`; `pointer.location()` for the `Location`. No `Deref`. Generic handler: `fn f<E: PointerEvent>(event: On<E>)` |
| BSN | Scene references need `@` | `@scene_variable`, `@scene_function()`, `@{expr}` |
| BSN | Entity separators | `--` separates entities in `Children [ ... ]` and `bsn_list!`. `()` and `,` are deprecated. Only `bsn_list! { }` is accepted, not `[]` or `()` |
| BSN | Enums and values | `template_value(x)` no longer needed (deprecated). Enums no longer need `VariantDefaults` or `FromTemplate` if `Default + Clone`. All enum fields must be specified: `Foo::A { x: 1, y: 0 }` |
| Shaders | naga_oil dialect → WESL | See below. GLSL support removed; `shader_format_wesl` feature gone; SPIR-V passthrough unchanged. Plain `.wgsl` with no preprocessor directives keeps working |
| Queries | `iter_many*` yield `Result` | `iter_many`, `iter_many_mut`, `iter_many_unique`, `iter_many_unique_mut` yield `Result<Item, QueryEntityError>`. `.matched()` restores the old behaviour. `par_iter_many*` closures receive a `Result`. Before `sort*`: `.flat_map(Result::ok)` |
| States | `set_if_neq` → `set_if_different` | Deprecated wrapper remains on `NextState`; variant `PendingIfNeq` → `PendingIfDifferent`; `commands.set_state_if_neq` → `set_state_if_different` |
| Text | `FontSource::SansSerif` → `FontSource::sans_serif()` | Generic families use constructor functions over `GenericFontFamily` |
| Text | `EditableText` and `TextInput` separated | A working input needs both. `TextScroll` → `EditableText::viewport`. `resolve_font_source` → `text_font.font.resolve_font_family(fonts)?`. `Font::from_bytes` takes no family name |
| UI | `BorderRadius` fields are `CornerRadius` | Corners are 2D: `CornerRadius::circular(px(10.))` or `px(10.).into()`. Constructors are no longer `const`. `resolve_single_corner` removed |
| UI | `Val::Em`, `Val::Rem` added | `Val::resolve` and similar take `em_size, rem_size`. `FontSize::eval(viewport, RemSize(px))` |
| Math | `bevy_shape` and `bevy_curve` split from `bevy_math` | Prelude users unaffected. `bevy_math::primitives::*` moved to `bevy_shape`, `bevy_math::curve::*` → `bevy_curve::*`, `bevy_math::cubic_splines::*` → `bevy_curve::cubic_splines::*`. `Bounded2d`, `Bounded3d`, `BoundingVolume`, `IntersectsVolume` now in the prelude |
| ECS | `Name` from `&'static str` only | Runtime strings: `Name::new(s.to_owned())` |
| Render | `MeshTag(x)` → `MeshTag::new(x)` | Read with `.value` |
| Mesh | `MeshAabb::compute_aabb` → `get_aabb` | |
| Assets | `AssetLoadError` variants boxed | Matches on large variants need adjusting |
| UI | `UiCameraMapper::current_camera()` returns `Option` | |

WESL migration, from the guide:

```wgsl
// Before, naga_oil dialect (compile error in 0.20)
#import bevy_pbr::forward_io::VertexOutput
#import "shaders/util.wgsl"::hsv_to_rgb
#ifdef VERTEX_COLORS
@group(#{MATERIAL_BIND_GROUP}) @binding(0) var<uniform> color: vec4<f32>;

// After, WESL
import bevy_pbr::render::forward_io::VertexOutput;
import super::util::hsv_to_rgb;
@if(VERTEX_COLORS)
@group(constants::MATERIAL_BIND_GROUP) @binding(0) var<uniform> color: vec4<f32>;
```

Imports end in `;` and come first. Module names match their path
(`bevy_pbr::mesh_view_bindings` → `bevy_pbr::render::mesh_view_bindings`,
`bevy_pbr::prepass_utils` → `bevy_pbr::prepass::utils`). Conditionals are
`@if`/`@elif`/`@else`. `#define_import_path` is gone: `embedded://bevy_foo/bar.wesl`
is `bevy_foo::bar`, anything else is imported by asset path.

### HIGH RISK — Silent Behaviour or Visual Changes

These compile. Nothing tells you they changed.

| Subsystem | Change | Details |
|-----------|--------|---------|
| 3D | Screen space transmission is opt-in | `Camera3d` no longer requires `ScreenSpaceTransmission`. Glass and other transmissive materials render differently until it is added to the camera |
| Sprites | `Sprite` moved to `Mesh2d` + material | Same-Z draw order may change: give overlapping sprites distinct Z values. New `alpha_mode` field (default `Blend`, also `Opaque`, `Mask(f32)`). `Text2d` still uses the old backend |
| Tonemapping | `Tonemapping::None` is a full passthrough | `ColorGrading` and `DebandDither` no longer apply, negative channels no longer clamped. A `Camera3d` without `Hdr` using it renders differently. To turn off the tone curve only, use `Tonemapping::Linear`. `Camera2d` defaults to `Linear`, and with `Hdr` now runs the tonemapping pass |
| Text | `TextFont::default()` size is `FontSize::Rem(1.)` | Same as before at the default `RemSize` of 20, but now scales with it. `FontSize::Px(20.)` to fix the size |
| Text input | `Escape` releases focus and propagates | Ancestor or window `Escape` handlers now run on the same press that blurs the field. Two-step behaviour needs `input.original_event_target()` |
| Schedules | Built-in sets ordered weakly | `Render`, `RenderGraph`, `Core2d`, `Core3d`, UI sets in `ExtractSchedule`, and `UiSystems` in `PostUpdate` use `chain_weak`. Ordering survives only where tracked data access conflicts. A custom system relying on an earlier set through interior mutability, a channel, an atomic or `NonSend` state needs an explicit `.after(..)` |
| Color | `LinearRgba::with_luminance` no longer clamps | Components can leave `[0.0, 1.0]` |
| Windows | macOS activation follows `Window::focused` | Requested on startup only when a window has `focused: true` |
| Feathers | `FeathersColorPlane` vertical axis flipped | Now increases upward |
| Solari | `SolariLighting::restir` off by default | |
| Image processing | `compressed_image_saver` now means BCn/ASTC | The old Basis behaviour is `compressed_image_saver_universal`. JPEG joined the default processor extensions |

### MEDIUM RISK — Compile Breaks in Narrower APIs

| Subsystem | Change | Details |
|-----------|--------|---------|
| Exclusive systems | `&mut World` is a normal `SystemParam` | Allowed in any position. `WorldId` not allowed alongside it. `ExclusiveMarker`, `ExclusiveFunctionSystem` removed |
| Focus | `FocusCause::Auto` added | Exhaustive matches need an arm |
| Text | `TextReader` yields `(Entity, usize, TextLayoutItem)` | Supports inline boxes |
| Reflection | `to_dynamic` returns `Result` | `DynamicEnum::from` deprecated. `FromType` → `CreateTypeData` |
| Feathers | Value updates | `FeathersNumberInput`: insert `NumberInputValue::F32(v)`. Cursor types moved from `bevy_feathers::cursor` to `bevy_picking::cursor`; `custom_cursor` feature moved with them |
| Sprites | Internal material renamed | `bevy_sprite_render::SpriteMaterial` → `SpriteMeshMaterial`, `SpriteMaterialPlugin` → `SpriteMeshMaterialPlugin`. The old names now mean something else: `SpriteMaterial<M>` and `SpriteMaterialPlugin<M>` for custom sprite materials |
| Settings | `SettingsGroup` | Requires `Resource + Reflect + Default`, with `#[reflect(Default, SettingsGroup)]` |
| Paths | `Tonemapping`, `DebandDither` moved to `bevy_render::view` | Rust re-exports still work; reflected paths in saved data change |
| Schedules | `ScheduleBuildSettings.shuffle_seed` | Struct literals need the field. Only under the `debug` feature |
| Queries | `ComponentInfo::id()` removed | |

### LOW RISK — Deprecated, Still Compiles

See `deprecated-apis.md`: `ui::Interaction`, `ui::widgets::Button`, `template_value`,
`FilteredResources*`, `DeferredWorld::query`, `AssetId::invalid()`, `TypeIdMap`.

### Custom Rendering Only

Skip this table unless the game has its own render pipeline, phases or passes.

| Subsystem | Change | Details |
|-----------|--------|---------|
| Extraction | Generic over worlds | `SyncComponent<RenderApp>`, `#[extract_app(RenderApp)]` on `ExtractComponent`, `TemporaryRenderEntity::default()`, new `bevy_extract` crate |
| Materials | `UnpreparedBindGroup` → `BindGroupBuilder` | `#[derive(AsBindGroup)]` users unaffected |
| OIT | Premultiplied alpha, per-material opt-out | `MATERIAL_OIT_ENABLED`, `enable_oit`, `oit_draw` |
| Buffers | `ShaderBuffer` stores typed data | `new` takes `Vec<T>`, `set_data` removed, `extend` added |
| Depth | `DepthAttachment`, `ViewDepthTexture`, `ViewPrepassTextures::depth` changed | |
| Compositing | `ViewTarget::compositing_space` → `ResolvedCompositingSpace` | `SortedCamera::hdr` removed |
| Windows | `ExtractedWindows` → `Query<&ExtractedWindow>` | `WgpuWrapper` removed |
| UI | UI rendering is retained | Custom extraction of UI nodes affected |
| Shader utils | `bevy_pbr::utils::octahedral_*` → `bevy_render::utils::*` | `DownsampleShaders::general` is a `Handle<Shader>` |
| Atmosphere | Supports multiple cameras | `AtmosphereBuffer` component |

## 0.18 → 0.19 (Jun 2026 — POST-CUTOFF, HIGH RISK)

Guide: https://bevy.org/learn/migration-guides/0-18-to-0-19/

| Subsystem | Change | Details |
|-----------|--------|---------|
| ECS | `Resource` is a subtrait of `Component` | `#[derive(Component, Resource)]` not allowed. Broad queries such as `Query<EntityMut>` now conflict with resources: add `Without<IsResource>` |
| Scenes | `bevy_scene` → `bevy_world_serialization` | `Scene` → `WorldAsset`, `SceneRoot` → `WorldAssetRoot`, `DynamicScene` → `DynamicWorld`, `DynamicSceneRoot` → `DynamicWorldRoot`, `SceneInstanceReady` → `WorldInstanceReady`, `SceneSpawner` → `WorldInstanceSpawner`, `ScenePlugin` → `WorldSerializationPlugin`. `bevy_scene` is now the BSN crate |
| Features | `audio` and `ui` not implied by `2d`/`3d` | `default` is `2d, 3d, ui, audio` |
| Text | Parley replaced cosmic-text | `TextFont.font` is a `FontSource` (`.into()`), `font_size` is a `FontSize` (`FontSize::Px(35.)`). `system_font_discovery` feature |
| Lifecycle | `Replace` → `Discard` | `on_replace` → `on_discard` |
| Errors | `DefaultErrorHandler` → `FallbackErrorHandler` | `fallback_error_handler` |
| Audio | Rodio 0.22 | Formats are features: `vorbis` default; `wav`, `mp3`, `mp4`, `flac`, `aac`, `audio-all-formats`. Minimum Android API 26 |
| Shaders | `ShaderStorageBuffer` → `ShaderBuffer` | |
| Rendering | Render graph as systems; bloom in linear color | Bloom looks different. `Atmosphere`, `Skybox` moved to `bevy_light`; `Hdr` to `bevy_camera` |
| Lights | `shadows_enabled` → `shadow_maps_enabled` | |
| States | `DespawnOnEnter`/`DespawnOnExit` | Fire on same-state transitions |
| Widgets | `Core` prefix removed | `CoreScrollbarThumb` → `ScrollbarThumb` and similar. `FeathersPlugin` → `FeathersCorePlugin`. Feathers controls moved to BSN |

## 0.17 → 0.18 (Jan 2026 — POST-CUTOFF, HIGH RISK)

Guide: https://bevy.org/learn/migration-guides/0-17-to-0-18/

| Subsystem | Change | Details |
|-----------|--------|---------|
| Camera | `RenderTarget` is a component | Was a `Camera` field |
| UI | `BorderRadius` is a field on `Node` | Was a component. `LineHeight` became a component. `BorderRect` uses `Vec2` |
| Hierarchy | Detach renames | `clear_children` → `detach_all_children`, `remove_children` → `detach_children`, `remove_child` → `detach_child` |
| Features | Feature collections | `animation` → `gltf_animation`; `sprite_picking`, `ui_picking`, `mesh_picking` |
| Animation | `AnimationTarget` split | `AnimationTargetId` + `AnimatedBy` |
| Lights | `AmbientLight` resource → `GlobalAmbientLight` | `AmbientLight` is now a per-camera component |
| Gizmos | `Gizmos::cuboid` → `cube` | |
| States | `NextState::set` triggers on same state | Same-state transitions now fire `OnExit`/`OnEnter` |
| glTF | `GltfConvertCoordinates` | Coordinate conversion options changed |

## 0.16 → 0.17 (Sep 2025 — POST-CUTOFF, HIGH RISK)

Guide: https://bevy.org/learn/migration-guides/0-16-to-0-17/

| Subsystem | Change | Details |
|-----------|--------|---------|
| Events | `Event` split into `Message` and `Event` | `EventWriter`/`EventReader`/`Events` → `MessageWriter`/`MessageReader`/`Messages`; `add_event` → `add_message`; `send_event` → `write_message` |
| Observers | `Trigger` → `On` | `OnAdd` → `Add` and similar. `EntityEvent` carries an `entity` field. `world.trigger_targets` removed: `world.trigger(Explode { entity })` |
| Time | `Timer::finished` → `is_finished` | `paused` → `is_paused` |
| States | `StateScoped` → `DespawnOnExit` | |
| Sets | `*Systems` naming | `TransformSystem` → `TransformSystems`, `UiSystem` → `UiSystems`, `InputSystem` → `InputSystems`, `RunFixedMainLoopSystem` → `RunFixedMainLoopSystems` |
| Text | `JustifyText` → `Justify` | `Text2d` moved to `bevy_sprite` |
| Camera | `Hdr` split from `Camera` | Marker component |
| Windows | `Window` split into components | `CursorOptions` is its own component on the window entity |
| Assets | `Handle::Weak` → `Handle::Uuid` | |
| ECS | `Condition` → `SystemCondition` | `iter_entities` deprecated. `System::run` returns `Result` |

## 0.15 → 0.16 (Apr 2025 — before the cutoff)

Guide: https://bevy.org/learn/migration-guides/0-15-to-0-16/

| Subsystem | Change | Details |
|-----------|--------|---------|
| Queries | `Query::single` returns `Result` | `get_single` deprecated. `Query::many` → `get_many` |
| Hierarchy | `Parent` → `ChildOf` | `.parent()` accessor. `ChildBuilder` → `ChildSpawnerCommands`. Relationships introduced |
| Errors | Fallible systems | Systems may return `Result` |
| UI | Renames | `TargetCamera` → `UiTargetCamera`, `UiImage` → `ImageNode`, `PickingBehavior` → `Pickable` |
| Crates | `bevy_core` removed | `bevy_utils::HashMap` → `bevy_platform`. `no_std` support. Edition 2024 |

## Sources

- https://bevy.org/news/bevy-0-20/
- https://bevy.org/news/bevy-0-19/
- https://bevy.org/news/bevy-0-18/
- https://bevy.org/news/bevy-0-17/
- https://bevy.org/learn/migration-guides/0-19-to-0-20/
- https://bevy.org/learn/migration-guides/0-18-to-0-19/
- https://bevy.org/learn/migration-guides/0-17-to-0-18/
- https://bevy.org/learn/migration-guides/0-16-to-0-17/
- https://bevy.org/learn/migration-guides/0-15-to-0-16/
