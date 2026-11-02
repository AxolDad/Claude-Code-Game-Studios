# Bevy — Third-Party Crates

Last verified: 2026-11-02 | Engine: Bevy 0.20

Bevy has no physics, pathfinding, game networking or input mapping. Plugins here are
crates from crates.io, not engine add-ons.

## How to Use This Guide

Every row was checked against the crates.io API on 2026-11-02, reading the Bevy
requirement of that exact release. **Only the version and its Bevy requirement were
checked. Their APIs were not reviewed**, so this file holds no third-party code. Read
the crate's docs for the version in `Cargo.lock`, and do not write its API from memory.

**Status:**
- **Confirmed**: latest release requires Bevy 0.20
- **Not on 0.20**: latest release requires an older Bevy, so adding it pulls a second, incompatible Bevy into the build

A crate not listed here is unverified, not rejected. See the check below. Purposes are
taken from each crate's crates.io description.

## Physics

Bevy has none (`modules/physics.md`).

| Crate | Version | Released | Requires | Status | Purpose |
|-------|---------|----------|----------|--------|---------|
| `avian3d` | 0.8.0 | 2026-10-20 | `bevy ^0.20` | Confirmed | ECS-driven 3D physics engine |
| `avian2d` | 0.8.0 | 2026-10-20 | `bevy ^0.20` | Confirmed | ECS-driven 2D physics engine |
| `bevy_rapier3d` | 0.37.0 | 2026-10-10 | `bevy ^0.20` | Confirmed | 3D physics engine in Rust, Bevy plugin |
| `bevy_rapier2d` | 0.37.0 | 2026-10-10 | `bevy ^0.20` | Confirmed | 2D physics engine in Rust, Bevy plugin |

Official: https://crates.io/crates/avian3d, https://crates.io/crates/bevy_rapier3d

## Navigation

Bevy has no pathfinding (`modules/navigation.md`).

| Crate | Version | Released | Requires | Status | Purpose |
|-------|---------|----------|----------|--------|---------|
| `vleue_navigator` | 0.17.0 | 2026-10-27 | `bevy ^0.20` | Confirmed | Navmesh plugin |
| `oxidized_navigation` | 0.12.0 | 2024-12-25 | `bevy ^0.15` | **Not on 0.20** | Navmesh generation plugin |

Official: https://crates.io/crates/vleue_navigator

## Networking

Bevy has no game networking (`modules/networking.md`).

| Crate | Version | Released | Requires | Status | Purpose |
|-------|---------|----------|----------|--------|---------|
| `lightyear` | 0.31.0 | 2026-10-21 | `bevy_app`, `bevy_ecs ^0.20` | Confirmed | Server-client networking library |
| `bevy_replicon` | 0.45.5 | 2026-10-31 | `bevy ^0.20` | Confirmed | Server-authoritative replication |

Official: https://crates.io/crates/lightyear, https://crates.io/crates/bevy_replicon

## Input

Bevy has no action mapping (`modules/input.md`).

| Crate | Version | Released | Requires | Status | Purpose |
|-------|---------|----------|----------|--------|---------|
| `leafwing-input-manager` | 0.22.0 | 2026-10-15 | `bevy ^0.20.0` | Confirmed | Action-input keybindings |
| `bevy_enhanced_input` | 0.27.2 | 2026-10-26 | `bevy ^0.20` | Confirmed | Input manager, inspired by Unreal's Enhanced Input |

Official: https://crates.io/crates/leafwing-input-manager, https://crates.io/crates/bevy_enhanced_input

## Audio

Bevy has no buses or effects (`modules/audio.md`).

| Crate | Version | Released | Requires | Status | Purpose |
|-------|---------|----------|----------|--------|---------|
| `bevy_kira_audio` | 0.27.1 | 2026-10-13 | `bevy_app`, `bevy_ecs`, `bevy_reflect ^0.20` | Confirmed | Plugin for dynamic audio |
| `bevy_seedling` | 0.8.0 | 2026-08-09 | `bevy_app`, `bevy_ecs ^0.19.0` | **Not on 0.20** | Firewheel audio engine integration |

Official: https://crates.io/crates/bevy_kira_audio

## Tools, Assets and Tweening

| Crate | Version | Released | Requires | Status | Purpose |
|-------|---------|----------|----------|--------|---------|
| `bevy_egui` | 0.43.1 | 2026-10-10 | `bevy_app`, `bevy_ecs ^0.20.0` | Confirmed | Egui integration |
| `bevy_asset_loader` | 0.28.1 | 2026-10-13 | `bevy ^0.20.0` | Confirmed | Plugin for asset loading |
| `bevy_tweening` | 0.17.0 | 2026-11-01 | `bevy ^0.20` | Confirmed | Tweening animation plugin |

Official: https://crates.io/crates/bevy_egui, https://crates.io/crates/bevy_asset_loader, https://crates.io/crates/bevy_tweening

## On-Demand Check

Do this before adding any crate, listed or not. Versions move quickly.

1. Latest version: `https://crates.io/api/v1/crates/<name>`
2. Its dependencies: `https://crates.io/api/v1/crates/<name>/<version>/dependencies`
3. Find `bevy`, or `bevy_app`/`bevy_ecs` for crates depending on sub-crates. The requirement must admit the version in `Cargo.lock`
4. Older than that: the crate is not usable yet. Do not pin Bevy down to fit it without an ADR

## Quick Decision Guide

| Need | Bevy provides | Crate |
|------|---------------|-------|
| Rigid bodies, colliders | Nothing | `avian3d`, `avian2d`, `bevy_rapier3d`, `bevy_rapier2d` |
| Ray cast against meshes only | `MeshRayCast` | None needed |
| Pathfinding | Nothing | `vleue_navigator` |
| UI focus movement | `bevy_input_focus` | None needed |
| Multiplayer | Nothing | `lightyear`, `bevy_replicon` |
| Rebindable actions | Nothing | `leafwing-input-manager`, `bevy_enhanced_input` |
| Play sounds, spatial panning | `bevy_audio` | None needed |
| Buses and effects | Nothing | `bevy_kira_audio` |
| Game UI widgets | `bevy_ui_widgets` | None needed |
| Debug panels | Feathers | `bevy_egui` |
| Loading screens | `AssetServer` | `bevy_asset_loader` |
| Tweens | `bevy_animation` curves | `bevy_tweening` |

---

**Last Updated:** 2026-11-02
**Engine Version:** Bevy 0.20.0
**Sources:** https://crates.io/api/v1/crates (per-crate version and dependency endpoints)
**LLM Knowledge Cutoff:** May 2025
