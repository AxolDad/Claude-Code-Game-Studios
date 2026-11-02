# Bevy Networking — Quick Reference

Last verified: 2026-11-02 | Engine: Bevy 0.20

## Bevy Does Not Provide Game Networking

**Bevy 0.20 has no replication, client-server model, rollback, RPC layer or socket
abstraction for gameplay.** A search of every `bevy*-0.20.1` crate for sockets and
replication finds only unrelated matches. The choice is a third-party crate
(`PLUGINS.md`) or game code on top of `std::net` or a transport crate.

## Current API Patterns

### Replication: Third-Party

| Crate | Version | Checked | Bevy 0.20 |
|-------|---------|---------|-----------|
| `lightyear` | 0.31.0 | 2026-11-02 | Yes, requires `bevy_app` and `bevy_ecs ^0.20` |
| `bevy_replicon` | 0.45.5 | 2026-11-02 | Yes, requires `bevy ^0.20` |

```toml
[dependencies]
bevy_replicon = "0.45"
```

Their APIs were not reviewed for this reference. Read their own docs for the version in
`Cargo.lock`. Choosing one is an architecture decision: record it as an ADR.

### What Bevy Does Have

These sit near networking. **None of them replicates game state.**

| Feature | What it is | Not |
|---------|------------|-----|
| `bevy_remote` | Bevy Remote Protocol: JSON-RPC 2.0 for inspecting and altering a running app's ECS | Not a player transport |
| `http`, `https` | `WebAssetPlugin`: `AssetServer` loads from URLs | Not an HTTP client for game requests |
| `FixedUpdate`, `Time<Fixed>` | The fixed loop most netcode schedules against | Not lockstep or prediction |

### Bevy Remote Protocol (Tooling)

Feature `bevy_remote`. `RemotePlugin` provides the protocol and no transport;
`RemoteHttpPlugin` adds HTTP, by default on port 15702. Not available on wasm.

```rust
use bevy::{
    prelude::*,
    remote::{http::RemoteHttpPlugin, RemotePlugin},
};

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_plugins(RemotePlugin::default())
        .add_plugins(RemoteHttpPlugin::default())
        .run();
}
```

Clients `POST` JSON to the root URL:

```json
{
  "method": "world.get_components",
  "id": 0,
  "params": {
    "entity": 4294967298,
    "components": ["bevy_transform::components::transform::Transform"]
  }
}
```

A client can alter the world. Keep this feature out of shipped builds.

### Web Assets

```toml
[dependencies]
bevy = { version = "0.20", features = ["https"] }
```

```rust
// fragment
let image: Handle<Image> = asset_server.load("https://example.com/icon.png");
```

## Common Mistakes

- Treating `bevy_remote` as multiplayer. It is a remote control for tools
- Inventing `bevy::net`. There is no such module
- Shipping with `bevy_remote` on
- Adopting a networking crate at a version that does not match. Check crates.io first: the requirement must say `0.20`
- Calling a Bevy example "networked". `examples/remote/` is the protocol above

## Examples and Sources

`examples/remote/`: `server`, `client`, `app_under_test`, `integration_test`.
`crates/bevy_remote/src/lib.rs`, `crates/bevy_remote/src/http.rs` at
https://github.com/bevyengine/bevy/tree/v0.20.1
