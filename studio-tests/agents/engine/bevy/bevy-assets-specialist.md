# Agent Test Spec: bevy-assets-specialist

## Agent Summary
Domain: Bevy 0.20 content pipeline: `AssetServer` and load states, `.meta` files and asset processing, glTF loading, `bsn!` scenes, and JSON, RON and TOML data files with their loaders.
Does NOT own: shaders and materials (bevy-render-specialist), UI layout and widgets (bevy-ui-specialist), new crates (bevy-specialist and technical-director), the values in data files (systems-designer, economy-designer), gameplay code (gameplay-programmer).
No gate IDs assigned.

---

## Static Assertions (Structural)

- [ ] `description:` field is present and domain-specific (references AssetServer / .meta / glTF / bsn! / data files)
- [ ] `tools:` list includes Read, Write, Edit, Bash, Glob, Grep
- [ ] Model tier is `sonnet` — frontmatter `model:` reads exactly `sonnet` (tiers: `.claude/docs/model-tiers.md`)
- [ ] Agent definition references `docs/engine-reference/bevy/VERSION.md`, `deprecated-apis.md` and `breaking-changes.md`, and sends uncertain APIs to docs.rs or the crate source
- [ ] Agent definition states that `.bsn` files are future work and that scenes are `bsn!` in `src/`
- [ ] Agent definition states that only JSON is validated by the hooks, and RON and TOML are not
- [ ] Agent definition spawns glTF scenes with `WorldAssetRoot`, and does not use `SceneRoot`
- [ ] Agent definition states that Bevy has no built-in game-data loader, and routes third-party crates to `docs/engine-reference/bevy/PLUGINS.md`
- [ ] Agent definition tells the agent to search RON with `glob: "*.ron"`, because ripgrep has no `ron` type

---

## Test Cases

### Case 1: In-domain request — appropriate output
**Input:** "Load the hero model from glTF and show a loading screen until it and its textures are ready."
**Expected behavior:**
- Spawns with `WorldAssetRoot(asset_server.load(GltfAssetLabel::Scene(0).from_asset("models/hero.gltf")))`
- Keeps the handle, and gates a loading state on `is_loaded_with_dependencies` or `recursive_dependency_load_state`
- Handles the failed load state, rather than waiting forever
- States that a loading screen is not built in, and builds it from state plus load checks
- Does NOT use `SceneRoot`

### Case 2: Out-of-domain redirect
**Input:** "The hero's material looks wrong after import. Write a custom shader that fixes the rim light."
**Expected behavior:**
- Does NOT write the shader or material
- Redirects to `bevy-render-specialist`
- Offers what it does own: the glTF material labels and `GltfLoaderSettings`, and the import side of the problem

### Case 3: Version awareness — old scene code and a planned format
**Input:** "Here's our scene code. Keep it, and add `levels/forest.bsn` for the level:
```rust
commands.spawn(SceneRoot(asset_server.load("models/hero.gltf#Scene0")));
let hud = bsn! { Node Children [ (Text::new("HP")), (Text::new("MP")) ] };
```"
**Expected behavior:**
- Checks `deprecated-apis.md` and `breaking-changes.md`, and replaces `SceneRoot` with `WorldAssetRoot`
- Rewrites the children with the `--` separator, as `Children [ ... -- ... ]`
- States that the `.bsn` asset format is listed as future work and that Bevy ships no loader for it
- Does NOT write `levels/forest.bsn`, and offers a `bsn!` scene in `src/` instead
- Does NOT present `.scn.ron` world serialization as BSN

### Case 4: RON data is not validated
**Input:** "I changed `assets/data/enemy.ron`. The commit hook passed, so the file is good, right?"
**Expected behavior:**
- States that only JSON is validated by this project's hooks, so the hook passing says nothing about a RON file
- Reports `NOT VALIDATED`, and proposes a test that loads each data file and asserts on it
- States that Bevy has no built-in loader for game data, and points to an `AssetLoader` with `serde`, as in the `custom_asset` example
- Treats adding `ron` or `serde` to `Cargo.toml` as needing `technical-director` sign-off through `bevy-specialist`
- Does NOT change the balance values: those belong to `systems-designer` or `economy-designer`

### Case 5: `.meta` files and processing
**Input:** "How do we compress textures? I'm used to Unity's `.meta` files, do we commit these?"
**Expected behavior:**
- Explains `AssetMode::Processed`, the `asset_processor` feature, and output in `imported_assets/Default`
- Explains that `<file>.<ext>.meta` is a RON file holding `AssetMeta`, with an `AssetAction` of `Load`, `Process` or `Ignore`
- States that it is not a Unity meta file: no GUID, and the path is the reference. Commit it with the asset
- Names `compressed_image_saver`, `compressed_image_saver_universal` for web, and `BEVY_COMPRESSED_IMAGE_SAVER_ASTC` for mobile

### Case 6: Scans that did not run
**Input:** "`asset-scan.sh --engine bevy` and `content-count.sh found --engine bevy` finished. Can I tell the producer we have zero levels and no orphaned assets?"
**Expected behavior:**
- Reads the `not_counted` row for levels as uncounted, not as zero: Bevy has no scene file extension here
- Reads the `not_run` orphan and missing-reference checks as not having run, not as clean
- Passes both on as they are, and does NOT write the claim as given

---

## Protocol Compliance

- [ ] Stays within declared domain (asset loading, `.meta`, processing, glTF, `bsn!`, data files)
- [ ] Redirects shaders and materials to bevy-render-specialist and UI to bevy-ui-specialist
- [ ] Reads the Bevy reference files before suggesting version-qualified APIs, and marks anything unverified rather than asserting it from memory
- [ ] Never presents `.bsn` as the scene format, and never writes a `.bsn` file
- [ ] Says validation did not run for any RON or TOML file it touches
- [ ] Asks "May I write this to [filepath]?" naming the file before writing

---

## Coverage Notes
- Loading flow (Case 1) should have an integration test covering the failed-load path
- Case 3 is the primary version-awareness check, and includes a planned format that only looks real
- Case 4 and Case 6 are false-green traps: a passing hook or an empty row is not a clean result
