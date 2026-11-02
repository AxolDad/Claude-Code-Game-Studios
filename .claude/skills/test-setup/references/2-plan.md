# /test-setup — Phase 2: Present Plan

> Part of `/test-setup`. `SKILL.md` names when to read this file; the settings it
> resolved and the rules under its later headings still apply here.

## Phase 2: Present Plan

Based on the engine detected and the existing state, present a plan:

```
## Test Setup Plan — [Engine]

I will create the following (skipping any that already exist):

tests/
  unit/           — Isolated unit tests for formulas, state, and logic   (Godot and Bevy)
  integration/    — Cross-system tests and save/load round-trips         (Godot and Bevy)
  smoke/          — Critical path test list (15-minute manual gate)
  README.md       — Test framework documentation

[Unity/Unreal: unit and integration tests go under the engine's test root
 instead — Unity Assets/Tests/EditMode|PlayMode/, Unreal
 Source/<Module>/Private/Tests/ — because neither compiles code in tests/]

production/qa/
  evidence/       — Screenshot and manual test sign-off records

[Engine-specific files — see per-engine details below]

.github/workflows/tests.yml  — CI: run tests on every push to main

Estimated time: ~5 minutes to create all files.
```

**On Bevy, take the file list from the script instead of writing it from this
template:** run `bash .claude/scripts/test-scaffold.sh bevy --dry-run`, list its
`WOULD_CREATE` rows as what you will create and its `EXISTS` rows as what you will
skip, and relay every `NEEDS` row. `NEEDS cargo_manifest Cargo.toml` and `NEEDS
library_crate src/lib.rs` mean the tests would not build or could not import the
game: name `/setup-engine` as the step that scaffolds them. If the script prints
nothing, say so and stop: there is no plan to approve.

Ask: "May I create these files? I will not overwrite any test files that
already exist at these paths."

**At `collaborative` and `guided`** — do not proceed without approval. These are
**new** files, and `automation-modes.md:81` gates new-file writes in `guided` too,
so the answer is the same in both modes. **At `autonomous`** — create them and log
the decision; do not block.

---
