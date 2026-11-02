# Bevy — Deprecated and Replaced APIs

Last verified: 2026-11-02 | Engine: Bevy 0.20

If an agent suggests any API in the "Deprecated" column, it MUST be replaced with
the "Use Instead" column. Bevy often removes an API outright rather than deprecating
it first, so many rows below no longer compile at all. "Since" is the release that
changed it.

## Deprecated in 0.20 (Still Compiles, Warns)

| Deprecated | Use Instead | Since | Notes |
|------------|-------------|-------|-------|
| `ui::Interaction` | `picking::hover::Hovered`, `ui::Pressed` | 0.20 | Source note: should be removed after 0.20 |
| `ui::widgets::Button` | `ui_widgets::Button` | 0.20 | Headless: fires `Activate`, no styling |
| `template_value(x)` in `bsn!` | `x` | 0.20 | Still in the prelude |
| `()` or `,` between entities in `bsn!` | `--` | 0.20 | |
| `NextState::set_if_neq` | `NextState::set_if_different` | 0.20 | Deprecated wrapper |
| `FilteredResources`, `FilteredResourcesMut` | `QueryParamBuilder`, `IsResource`, `ResourceEntities` | 0.20 | Resources are entities since 0.19 |
| `DeferredWorld::query` | `query_state.query_mut(&mut world)` | 0.20 | |
| `AssetId::invalid()`, `AssetId::INVALID_UUID` | `Option<AssetId>` | 0.20 | |
| `DynamicEnum::from` | `to_dynamic` (now returns `Result`) | 0.20 | |
| `TypeIdMap` | `TypeIdHashMap`, `TypeIdIndexMap` | 0.20 | |

## Replaced in 0.20 (Compile Error)

| Deprecated | Use Instead | Since | Notes |
|------------|-------------|-------|-------|
| `On<Add, A>` | `On<Add<A>>` | 0.20 | Also `Insert`, `Discard`, `Remove`, `Despawn` |
| `On<Pointer<Press>>` | `On<PointerPress>` | 0.20 | Every pointer event is flat. `press.pointer.id`, `press.pointer.position` |
| `query.iter_many(list)` yielding items | `query.iter_many(list).matched()`, or handle the `Result` | 0.20 | |
| `FontSource::SansSerif` | `FontSource::sans_serif()` | 0.20 | |
| `MeshAabb::compute_aabb` | `get_aabb` | 0.20 | |
| `MeshTag(x)` | `MeshTag::new(x)` | 0.20 | |
| `TextScroll` | `EditableText::viewport` | 0.20 | |
| `resolve_font_source` | `text_font.font.resolve_font_family(fonts)?` | 0.20 | |
| `bevy_feathers::cursor` | `bevy_picking::cursor` | 0.20 | |
| `bevy_sprite_render::SpriteMaterial` | `SpriteMeshMaterial` | 0.20 | `SpriteMaterial<M>` now names the custom sprite material component |
| `shader_format_wesl` feature | No feature needed | 0.20 | GLSL support removed |

## Replaced in 0.19

| Deprecated | Use Instead | Since | Notes |
|------------|-------------|-------|-------|
| `SceneRoot` | `WorldAssetRoot` | 0.19 | glTF scenes still spawn this way |
| `Scene`, `DynamicScene` | `WorldAsset`, `DynamicWorld` | 0.19 | |
| `DynamicSceneRoot` | `DynamicWorldRoot` | 0.19 | |
| `SceneInstanceReady` | `WorldInstanceReady` | 0.19 | BSN has its own `Ready` event |
| `SceneSpawner` | `WorldInstanceSpawner` | 0.19 | |
| `#[derive(Component, Resource)]` | Pick one | 0.19 | `Resource` is a subtrait of `Component` |
| `TextFont { font_size: 35.0, .. }` | `font_size: FontSize::Px(35.0)` | 0.19 | Inside `bsn!`: `font_size: px(35.0)` |
| `TextFont { font: handle, .. }` | `font: handle.into()` | 0.19 | `FontSource` |
| `shadows_enabled` | `shadow_maps_enabled` | 0.19 | `PointLight`, `DirectionalLight`, `SpotLight` |
| `Replace`, `on_replace` | `Discard`, `on_discard` | 0.19 | |
| `DefaultErrorHandler` | `FallbackErrorHandler` | 0.19 | |
| `ShaderStorageBuffer` | `ShaderBuffer` | 0.19 | |
| `FeathersPlugin` | `FeathersCorePlugin` | 0.19 | |
| `CoreScrollbarThumb`, `CoreSliderDragState` | `ScrollbarThumb`, `SliderDragState` | 0.19 | |

## Replaced in 0.17 and 0.18

| Deprecated | Use Instead | Since | Notes |
|------------|-------------|-------|-------|
| `AmbientLight` as a resource | `GlobalAmbientLight` | 0.18 | `AmbientLight` is a per-camera component |
| `Camera { target, .. }` | `RenderTarget` component | 0.18 | |
| `BorderRadius` as a component | `Node { border_radius, .. }` | 0.18 | |
| `AnimationTarget` | `AnimationTargetId` + `AnimatedBy` | 0.18 | |
| `clear_children`, `remove_children`, `remove_child` | `detach_all_children`, `detach_children`, `detach_child` | 0.18 | |
| `Gizmos::cuboid` | `Gizmos::cube` | 0.18 | |
| `EventReader`, `EventWriter`, `Events` | `MessageReader`, `MessageWriter`, `Messages` | 0.17 | `#[derive(Message)]` |
| `app.add_event::<T>()` | `app.add_message::<T>()` | 0.17 | |
| `Trigger<OnAdd, A>` | `On<Add<A>>` | 0.17, 0.20 | `On<Add, A>` was the 0.17 to 0.19 form |
| `trigger.target()` | `add.entity` | 0.17 | `EntityEvent` carries an `entity` field |
| `world.trigger_targets(e, id)` | `world.trigger(Explode { entity })` | 0.17 | |
| `Timer::finished`, `Timer::paused` | `is_finished`, `is_paused` | 0.17 | |
| `StateScoped` | `DespawnOnExit` | 0.17 | `DespawnOnEnter` also exists |
| `TransformSystem`, `UiSystem`, `InputSystem` | `TransformSystems`, `UiSystems`, `InputSystems` | 0.17 | |
| `JustifyText` | `Justify` | 0.17 | |
| `Camera { hdr: true, .. }` | `Hdr` component | 0.17 | |
| `Condition` | `SystemCondition` | 0.17 | |
| `Handle::Weak` | `Handle::Uuid` | 0.17 | |

## Replaced in 0.16 (Thinly Covered Before the Cutoff)

| Deprecated | Use Instead | Since | Notes |
|------------|-------------|-------|-------|
| `query.get_single()` | `query.single()` | 0.16 | Returns `Result`. Do not `.unwrap()` it: `let Ok(x) = query.single() else { return };` or `Single<&T>` as the param |
| `Parent` | `ChildOf` | 0.16 | `.parent()` |
| `query.many(..)` | `query.get_many(..)` | 0.16 | |
| `TargetCamera`, `UiImage`, `PickingBehavior` | `UiTargetCamera`, `ImageNode`, `Pickable` | 0.16 | |
| `bevy_utils::HashMap` | `bevy_platform` collections | 0.16 | |

## Patterns (Not Just APIs)

| Deprecated Pattern | Use Instead | Why |
|--------------------|-------------|-----|
| Polling `Interaction` in a system | `Hovered` + `Has<Pressed>`, or `observe(\|_: On<Activate>\| ..)` on a `ui_widgets::Button` | `Interaction` is on its way out |
| Hand-written `#import` and `#ifdef` in shaders | WESL `import ...;` and `@if(...)`, in `.wesl` files | The naga_oil dialect is gone |
| Writing scenes as `.bsn` files | `bsn!` in code | The `.bsn` asset format does not exist yet. FUTURE work per the 0.20 notes |
| `Query<EntityMut>` with no filter | Add `Without<IsResource>` | Resources are entities now and conflict |
| `Tonemapping::None` to turn off the tone curve | `Tonemapping::Linear` | `None` is now a full passthrough |
| Sprites sharing a Z and relying on draw order | Give overlapping sprites distinct Z values | Backend moved to `Mesh2d`; same-Z order may differ |
| `Feathers` for game UI | `bevy_ui` with `bevy_ui_widgets`, styled yourself | Crate docs: deliberately not intended for game UI |

## Sources

- https://bevy.org/learn/migration-guides/0-19-to-0-20/
- https://bevy.org/learn/migration-guides/0-18-to-0-19/
- https://bevy.org/learn/migration-guides/0-17-to-0-18/
- https://bevy.org/learn/migration-guides/0-16-to-0-17/
- https://bevy.org/learn/migration-guides/0-15-to-0-16/
