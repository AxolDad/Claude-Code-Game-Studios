# /setup-engine — Section 3: Look Up Current Version

> Part of `/setup-engine`. `SKILL.md` names when to read this file; the settings it
> resolved and the rules under its later headings still apply here.

## 3. Look Up Current Version

Once the engine is chosen:

- If version was provided, use it
- If no version provided, use WebSearch to find the latest stable release:
  - Search: `"[engine] latest stable version [current year]"`
  - Confirm with the user: "The latest stable [engine] is [version]. Use this?"
- **On Bevy, the source is crates.io, not a search:** WebFetch
  `https://crates.io/api/v1/crates/bevy` and read `crate.max_stable_version`, then
  confirm with the user the same way. crates.io answers 403 to a request with no
  `User-Agent`, so a failed fetch says nothing about the version: fall back to
  WebSearch, and never read the version from Bevy's setup guide page, which lags
  behind the release.

### Check the chosen version against what is actually installed

A version found by web search is what exists, not what the developer has. Pinning
one they cannot run means every later instruction — build, test, smoke, the
engine reference docs — targets an engine that is not on the machine.

Probe for the binary (Bash), and treat failure as unknown, never as absent:

| Engine | Probe |
|--------|-------|
| Godot | `godot --version`, else look for `godot`/`Godot_v*` on PATH or in the platform's usual install location (macOS: the binary inside the bundle, `Godot.app/Contents/MacOS/Godot`) |
| Unity | The Hub's editor directory, then that editor's `-version` — `C:/Program Files/Unity/Hub/Editor/<version>/Editor/Unity.exe` on Windows, `/Applications/Unity/Hub/Editor/<version>/Unity.app/Contents/MacOS/Unity` on macOS, `/home/<user>/Unity/Hub/Editor/<version>/Editor/Unity` on Linux. Not bare `Unity`: on `PATH` it may be Unity's separate CLI, which rejects `-version` |
| Unreal | The launcher's install directory (`UE_<version>` folders under `C:/Program Files/Epic Games/` on Windows); elsewhere, ask where the engine was built or installed. The editor is not on `PATH` by default |
| Bevy | `cargo --version` and `rustc --version`. Bevy is a crate, so there is no engine binary. Compare `rustc` with the chosen version's minimum: `version.rust_version` from `https://crates.io/api/v1/crates/bevy/<version>` (same 403 caveat as above) |

Keep the executable the probe found (for Godot, whether bare `godot --version`
ran — that is, `godot` is on `PATH`): Section 5.5.1 writes it into `commands.*`
and `engine.path`. On Bevy there is nothing to keep: `commands.*` run `cargo` and
`engine.path` does not apply.

Then:
- **Match** — say so in one line and continue.
- **Mismatch** — state both versions and ask, offering three options: pin the
  installed version, pin the newer one and upgrade later, or pin the newer one
  deliberately. Record the answer; do not pick silently.
- **Not found / probe failed** — report `installed version NOT DETERMINED` and
  continue with the chosen version. Do **not** report this as "no engine
  installed": a probe that could not run has not established absence.

**On Bevy, "installed" means the toolchain.** `rustc` at or above the minimum is the
match. Below it is the mismatch: state both and ask, offering `rustup update`, a
Bevy version whose minimum this toolchain meets, or pinning this version anyway
and updating Rust later. `cargo` or `rustc` not found is `installed version NOT
DETERMINED`. A minimum that could not be fetched is also `NOT DETERMINED` in the row
below, because the toolchain's version alone does not show it is new enough. A met
minimum reads `rustc 1.97.1, cargo 1.97.1 — meets Bevy 0.20.0's minimum 1.97.1`.

Whatever the outcome, write it into Section 7's `VERSION.md` as an
`Installed at pin time` row, so a later reader can tell a deliberate
version-ahead pin from an accident.

---
