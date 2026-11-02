# Bevy — Version Reference

| Field | Value |
|-------|-------|
| **Engine Version** | Bevy 0.20.0 |
| **Installed at pin time** | NOT DETERMINED — `/setup-engine` §3 probes the toolchain (`cargo --version`, `rustc --version`) and the `bevy` entry in `Cargo.lock`, and records the result here. |
| **Release Date** | October 2026 (latest patch: 0.20.1, October 2026) |
| **Minimum Rust Version** | 1.97.1 (0.20.0 and 0.20.1) |
| **Project Pinned** | 2026-11-02 |
| **Last Docs Verified** | 2026-11-02 |
| **LLM Knowledge Cutoff** | May 2025 |

Last verified: 2026-11-02

## Knowledge Gap Warning

The LLM's training data likely covers Bevy up to ~0.16. Versions 0.17, 0.18, 0.19
and 0.20 were released after the cutoff and renamed or restructured events,
observers, scenes, resources, text, UI widgets and shaders; code written from
memory will often not compile. Always cross-reference this directory before
suggesting Bevy API calls.

## Installed-Version Gap Warning

The warning above is one-directional — it covers the **model** knowing less than
this pin. The reverse gap is real: this reference can sit **ahead of the version
the project actually builds**. `Cargo.lock` decides that version, not this file,
and a lock left on an older `bevy` emits compile errors against APIs this
reference cites correctly. **Check `Installed at pin time` above before trusting a
version-qualified claim** — `NOT DETERMINED` means the gap is unknown, not absent.

## No Engine Binary

Bevy is a Rust crate, not an installed program. There is no editor and no engine
executable, so `engine.path` does not apply. The toolchain is what gets probed:

| Question | Where to look |
|----------|---------------|
| Version declared | `Cargo.toml`: `bevy = "0.20"` (resolves to 0.20.1 on 2026-11-02) |
| Version in use | `Cargo.lock`: the `name = "bevy"` block |
| Latest published | https://crates.io/api/v1/crates/bevy |
| Toolchain | `cargo --version`, `rustc --version`. A `rustc` older than 1.97.1 is below the minimum for 0.20 |

Bevy's setup guide page still shows `bevy = "0.19"`. Do not read the version from it.

## Post-Cutoff Version Timeline

| Version | Release | Risk Level | Key Theme |
|---------|---------|------------|-----------|
| 0.17 | Sep 2025 | HIGH | `Event` split into `Message` and `Event`, `Trigger` becomes `On`, experimental headless widgets and Feathers, Solari, Rust hotpatching |
| 0.18 | Jan 2026 | HIGH | Cargo feature collections, `RenderTarget` component, first-party camera controllers, automatic directional UI navigation, font variations |
| 0.19 | Jun 2026 | HIGH | Resources as components, `bsn!` scenes, `bevy_scene` renamed `bevy_world_serialization`, Parley text, `EditableText`, render graph as systems, app settings |
| 0.20 | Oct 2026 | HIGH | BSN syntax changes, `On<Add<A>>`, WESL replaces the naga_oil dialect, flat pointer events, Feathers widgets, sprite materials, `Val::Em`/`Val::Rem`, mesh shaders |

## Before the Cutoff

0.16 (Apr 2025; `Query::single` returns `Result`, relationships, `ChildOf`,
`no_std`) predates the May 2025 cutoff, as does 0.15 (Nov 2024); the Knowledge Gap
Warning still applies to both.

Dates and minimum Rust versions from https://crates.io/api/v1/crates/bevy, checked
2026-11-02:

| Version | Published | Minimum Rust |
|---------|-----------|--------------|
| 0.15.0 | 2024-11-29 | 1.82.0 |
| 0.16.0 | 2025-04-24 | 1.85.0 |
| 0.16.1 | 2025-05-30 | 1.85.0 |
| 0.17.0 | 2025-09-30 | 1.88.0 |
| 0.17.3 | 2025-11-17 | 1.88.0 |
| 0.18.0 | 2026-01-13 | 1.89.0 |
| 0.18.1 | 2026-03-04 | 1.89.0 |
| 0.19.0 | 2026-06-19 | 1.95.0 |
| 0.19.1 | 2026-08-13 | 1.95.0 |
| 0.20.0 | 2026-10-08 | 1.97.1 |
| 0.20.1 | 2026-10-28 | 1.97.1 |

Migration guides and release notes: `breaking-changes.md` § Sources.
