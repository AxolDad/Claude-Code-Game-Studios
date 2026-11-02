# /test-setup — Phase 6: Post-Setup Summary

> Part of `/test-setup`. `SKILL.md` names when to read this file; the settings it
> resolved and the rules under its later headings still apply here.

## Phase 6: Post-Setup Summary

After writing all files, report:

```
Test infrastructure created for [engine].

Files created:
- tests/README.md
- tests/unit/ and tests/integration/ (Godot) — or the engine's test root:
  Unity Assets/Tests/EditMode/ and PlayMode/, Unreal Source/<Module>/Private/Tests/,
  Bevy tests/unit/main.rs and tests/integration/main.rs
- tests/smoke/critical-paths.md
- production/qa/evidence/.gitkeep
[engine-specific files]
- .github/workflows/tests.yml

Next steps:
1. [Engine-specific install step, e.g., "Install GdUnit4 via the Asset Store (AssetLib before Godot 4.7)"; Bevy has none]
2. Write your first test in the engine's test root:
   tests/unit/[system]/[system]_[feature]_test.gd (Godot),
   Assets/Tests/EditMode/[System]Tests.cs (Unity),
   Source/<Module>/Private/Tests/[System]Test.cpp (Unreal), or
   tests/unit/[system].rs with `mod [system];` in tests/unit/main.rs (Bevy)
3. Run `/qa-plan sprint` before your first sprint to classify stories and set
   test evidence requirements. At `workflow: minimal` — the default — skip
   this: there are no sprints. At `qa.level: minimal`, also the default, tests
   are waived, so `/dev-story` writes none. Raise `qa.level` with `/settings`
   to have it write them
4. `/smoke-check` before every QA hand-off

Gate note: /gate-check Technical Setup → Pre-Production now requires:
- the engine's test root: tests/unit/ and tests/integration/ (Godot),
  Assets/Tests/EditMode/ and Assets/Tests/PlayMode/ (Unity),
  Source/<Module>/Private/Tests/ (Unreal), or
  tests/unit/main.rs and tests/integration/main.rs (Bevy)
- .github/workflows/tests.yml
- At least one example test file
Run /test-setup and write one example test before advancing.
(At `workflow: minimal` that gate requires only the engine.)

Verdict: **COMPLETE** — test framework scaffolded and CI/CD wired up.
```

On Bevy, compile what was scaffolded before giving the verdict: `timeout 1800 cargo
check --tests`. `cargo check` alone does not compile test code. Exit 0 adds `Tests
compile: cargo check --tests exit 0` to the block. A non-zero exit is INCOMPLETE, quoting the
first `error` line. A timeout or a missing `cargo` is also INCOMPLETE, as `Tests NOT
COMPILED — [reason]`: nothing has shown the scaffold builds. With a `NEEDS` row open,
print `Tests NOT COMPILED — [the NEEDS row]` and do not run it.

On Bevy, a `FAILED` row, a `NEEDS` row or a script that printed nothing is not
COMPLETE. List the rows in place of the block's file list and end with
`Verdict: **INCOMPLETE** — [the rows]`, and for `NEEDS` name `/setup-engine` as the
next step.

---
