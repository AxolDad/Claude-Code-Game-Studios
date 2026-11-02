# Phase 6: Parse check and capture on Unity, Unreal and Bevy

Read only when the engine is Unity, Unreal or Bevy. Report the command, its exit code
and any error lines. No executable: `parse NOT VERIFIED — engine binary not
available`.

## Unity

**Parse check:** the `commands.smoke` command (`-batchmode -quit -projectPath .
-logFile -`). Exit 1 with `error CS…` lines means a script does not compile.

**Capture** needs `Assets/Scripts/ScreenshotOnArg.cs`, given word for word in
`run-and-observe.md` (named in `SKILL.md`, Phase 6). When the project has none
(look for `Assets/**/ScreenshotOnArg.cs`), ask "May I write
`Assets/Scripts/ScreenshotOnArg.cs`?" and write it exactly as given before
launching. Declined: nothing can capture, so report
`Run result: NOT VERIFIED — ScreenshotOnArg.cs not written`.

## Unreal

**Parse check:** build the editor target. A compile error fails it (`Result:
Failed`, non-zero exit).

- Windows: `"<UE root>/Engine/Binaries/DotNET/UnrealBuildTool/UnrealBuildTool.exe" <Project>Editor Win64 Development -Project="<absolute path>/<Project>.uproject"`
- Linux: `"<UE root>/Engine/Build/BatchFiles/Linux/Build.sh" <Project>Editor Linux Development -Project=…`
- macOS: `"<UE root>/Engine/Build/BatchFiles/Mac/Build.sh" <Project>Editor Mac Development -Project=…`

The commands are from Epic's documentation, as recorded in
`docs/engine-reference/unreal/current-best-practices.md` ("Command Line").

**Capture:** launch straight into the map the story touched, as
`run-and-observe.md` gives it.

## Bevy

**Parse check:** `commands.smoke` (`cargo check`), and `cargo check --tests` when the
story wrote tests: `cargo check` alone does not compile test code. Exit 101 with
`error[E…` lines means the code does not compile. A first build compiles all of
Bevy and can outlast the Bash tool's 10-minute limit, so run it in the background
and wait for it. No `cargo`: `parse NOT VERIFIED — cargo not available`. Neither
command covers a test file no `mod` line reaches: Phase 6's "Is the test there?" does.

**Capture** is `run-and-observe.md`'s Bevy procedure. Write the RON file to
`production/qa/evidence/[story-slug]/capture.ron` (no ask: it records how the
evidence was made, as the stamp does), then:

```bash
rm -f screenshot-60.png
CI_TESTING_CONFIG=production/qa/evidence/[story-slug]/capture.ron timeout 1800 cargo run --features bevy/bevy_ci_testing
mv screenshot-60.png production/qa/evidence/[story-slug]/[NN]-[what-it-shows].png
```

Never edit `Cargo.toml` to enable the feature. **Exit 0 is not the evidence, the file
is:** no PNG means `Run result: NOT VERIFIED — no screenshot written`. No GPU, no
display or a timeout is `NOT VERIFIED` too, with the messages `run-and-observe.md`
lists. macOS has no `timeout`: use `gtimeout` from `coreutils`, which asks first.
